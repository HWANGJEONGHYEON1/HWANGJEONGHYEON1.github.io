---
layout: post
title:  "스프링 클라우드 GatewayProperties 주입하기"
date:   2022-11-29 18:20:21 +0900
categories: kafka
---

> Spring-cloud-config를 통하여 도메인 설정들을 가져왔다. 도메인 관리는 DB를 통해 하는것이 좋다 판단하여 변경

## GatewayProperties
- org.springframework.cloud.gateway.config;
- 스프링 클라우드에서 해당 객체를 사용하여 필터 또는 라우트 해주는 역할
- 최초에는 config-server에 설정되어 있는 git에서 해당 객체를 가져왔었다.

```java

spring:
  cloud:
    gateway:
      routes:
        # ==================================================================
        - id: mypage
          uri: https://mypage.~.com
          predicates:
            - Path=/api/mypage/**
          filters:
            - RewritePath=/api/mypage/(?<segment>.*), /$\{segment}
```
- spring-config-server를 사용하면
- Fetching config from server at : http://localhost:8888로 설정 파일들을 불러와 GatewayProperties에 값을 세팅해준다.
- `위의 작업을 없애고 디비에서 불러오게 만든다.`


### 테이블 생성
- predicate와 filter는 나중에 여러개가 들어올 가능성이 있으므로 1:N 관계로 생성
```sql
CREATE TABLE `tb_domain_info` (
  `domain_seq` bigint(20) unsigned NOT NULL,
  `route_id` varchar(50) NOT NULL,
  `route_uri` varchar(300) NOT NULL,
  `use_yn` char(1) NOT NULL COMMENT,
  `reg_id` varchar(20) NOT NULL COMMENT,
  `reg_dt` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '등록일시',
  `chg_id` varchar(20) DEFAULT NULL COMMENT '수정자',
  `chg_dt` datetime DEFAULT NULL COMMENT '수정일시',
  PRIMARY KEY (`domain_seq`),
  KEY `idx_tb_domain_info_1` (`domain_seq`,`route_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='도메인 설정';
 
 
CREATE TABLE `tb_predicate_info` (
  `predicate_seq` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'predicates 시퀀스',
  `domain_seq` bigint(20) unsigned NOT NULL COMMENT '도메인 시퀀스',
  `route_predicate` varchar(300) NOT NULL COMMENT '도메인 predicate',
  PRIMARY KEY (`predicate_seq`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='도메인 설정';
 
 
CREATE TABLE `tb_filter_info` (
  `filter_seq` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'filter 시퀀스',
  `domain_seq` bigint(20) unsigned NOT NULL COMMENT '도메인 시퀀스',
  `route_filter` varchar(300) NOT NULL COMMENT '도메인 filter',
  PRIMARY KEY (`filter_seq`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='도메인 설정';
 
```

### 데이터 삽입
- JPA를 사용하여 넣었고 나머지 코드는 생략
```java

@Transactional
public void createGateway(List<GatewayCreateDto> requestDtos) {
    for (GatewayCreateDto requestDto : requestDtos) {
        GatewayRoute gatewayRoute = GatewayRoute.create(requestDto.getRouteId(), requestDto.getRouteUri(), requestDto.getRegId());

        requestDto.getFilters()
                .forEach(filter -> gatewayRoute.addFilter(new Filter(filter)));

        requestDto.getPredicates()
                .forEach(predicate -> gatewayRoute.addPredicate(new Predicate(predicate)));
        gatewayRouteRepository.save(gatewayRoute);
    }
}
```

### config 클라이언트에서 디비에 적재 된 데이터 가져오기
```java

@Slf4j
@Configuration
@RequiredArgsConstructor
@Getter
public class PropertiesConfig {

    private final GatewayProperties gatewayProperties;

    // 스프링 로드 시점에 최초로 설정들을 셋팅
    // 만약 디비의 값이 변경된다면 /servicemanager/refresh를 호출해줌으로써 현재 서비스중에 환경설정을 업데이트한다.
    @PostConstruct
    @EventListener(RefreshScopeRefreshedEvent.class)
    public void gatewayRoutes() {
        log.info(" ===== gatewayProperties 셋팅 =====");
        List<RouteDefinition> routes = new ArrayList<>();
        getDomains();
        domainDtos
                .forEach(domainDto -> {
                    try {
                        routes.add(setRouteDefinition(domainDto));
                    
                    } catch (URISyntaxException e) {
                        throw new RuntimeException(e);
                    }
                });
        gatewayProperties.setRoutes(routes);
    

    // GatewayProperties는 RouteDefinition를 의존하고 있어 해당 객체들의 값을 셋팅해준다.
    // 하드 코딩 된 부분은 api의 응답스펙을 바꿈으로써 입맞에 맞게 해결하면 된다.
    private RouteDefinition setRouteDefinition(DomainDto domainDto) throws URISyntaxException {

        Map<String, String> filterArgs = new LinkedHashMap<>();
        Map<String, String> predicateArags = new LinkedHashMap<>();

        RouteDefinition routeDefinition = new RouteDefinition();

        List<FilterDefinition> filterDefinitions = new ArrayList<>();
        FilterDefinition filterDefinition = new FilterDefinition();
        filterDefinition.setName("RewritePath");
        filterArgs.put("_genkey_0", "/api/" + domainDto.getRouteId() + "/(?<segment>.*)");
        filterArgs.put("_genkey_1", "/$\\{segment}");
        filterDefinition.setArgs(filterArgs);
        filterDefinitions.add(filterDefinition);

        routeDefinition.setFilters(filterDefinitions);

        List<PredicateDefinition> predicateDefinitions = new ArrayList<>();
        PredicateDefinition predicateDefinition = new PredicateDefinition();
        predicateDefinition.setName("Path");
        predicateArags.put("_genkey_0", "/api/" + domainDto.getRouteId() + "/**");
        predicateDefinition.setArgs(predicateArags);
        predicateDefinitions.add(predicateDefinition);

        routeDefinition.setPredicates(predicateDefinitions);
        routeDefinition.setUri(new URI(domainDto.getRouteUri()));
        routeDefinition.setId(domainDto.getRouteId());
        Route.builder(routeDefinition);
        return routeDefinition;
    }

    // WebClient를 통해서 적재된 데이터를 가져온다. 
    public void getDomains() {
        String uri = "http://localhost:8081/gateways";
        HttpHeaders httpHeaders = new HttpHeaders();
        domainDtos = Objects.requireNonNull(restWebClient.request(uri, httpHeaders, null, HttpMethod.GET, new ParameterizedTypeReference<ResponseObject<List<DomainDto>>>() {
        }).block()).getData();
    }    
}
```