---
layout: post
title:  "에러 발생 시 teams 알림"
date:   2022-04-21 18:20:21 +0900
categories: docker
---

# 실제로는 배포 실패 시 마이크로 팀즈로 배포실패알람
> 방치되어있던 프로젝트이지만 인사정보를 업데이트 시켜주는 중요한 프로젝트, 어느순간부터 에러가 발생하여 배치가 돌지 않는 문제가 발생 => 알람 설정

## 필요한 teams 카드 포맷
> https://docs.microsoft.com/en-us/microsoftteams/platform/task-modules-and-cards/cards/cards-format?tabs=adaptive-md%2Cconnector-html

> https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/what-are-webhooks-and-connectors 

## 소스
- 현재는 컨트롤러에서 try catch로 다 작업되어있어 각각 정의해줬지만 @ControllerAdvice를 통해 응답스팩을 바꾸고 한번에 처리가능할 것 같다.
- 각 어느 환경에서 에러가나는지 @Value 어노테이션을 통해서 값을 가져오고 title에 합쳐주는 식으로 명시
- TeamsUtil 클래스에서 teams로 알람을 보내는대, 구현하고 보니 굳이 응답을 기다리지 않아도 되므로 async로 하면 어떨까 생각한다.
- 현재 RestTemplate가 deprecated되어 있으므로 WebClient로 대체해야한다. 현재 프로젝트에서는 버전이 낮으므로 가능.

```java

@Getter
public class TeamsMessage {

    @JsonProperty("@type")
    private String type = "MessageCard";

    @JsonProperty("@context")
    private String context = "http://schema.org/extensions";

    private static final String themeColor = "0076D7";
    private String summary;
    private List<Sections> sections;

    public TeamsMessage(List<Sections> section) {
        this.sections = section;
        this.summary = section.get(0).getActivityTitle();
    }
}

@Getter
public class Sections {

    private String activityTitle = "BATCH-ERROR";
    private List<Facts> facts = new ArrayList<>();
    private boolean markdown = true;

    public Sections(String active) {
        this.activityTitle = activityTitle + "(" + active + ")" ;
    }

    public void makeFacts(Map<String, Object> factMap) {
        factMap.keySet()
                .stream()
                .map(key -> new Facts(key, String.valueOf(factMap.get(key))))
                .forEach(fact -> facts.add(fact));
    }
}

@Getter
@AllArgsConstructor
public class Facts {
    private String name;
    private String value;
}

@Component
@Slf4j
public class TeamsUtil {
    private RestTemplate restTemplate;
    private HttpHeaders headers;

    @Value("${spring.profiles}")
    private String profile;

    @Value("${error.notification.teams-url}")
    private String sendMessageErrorUrl;

    private ObjectMapper mapper = new ObjectMapper();

    @PostConstruct
    public void init() {
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        factory.setReadTimeout(5000); // 읽기시간초과, ms
        factory.setConnectTimeout(3000); // 연결시간초과, ms

        HttpClient httpClient = HttpClientBuilder.create()
                .setMaxConnTotal(100) // connection pool 적용
                .setMaxConnPerRoute(5) // connection pool 적용
                .build();

        factory.setHttpClient(httpClient); // 동기실행에 사용될 HttpClient 세팅

        restTemplate = new RestTemplate(factory);

        headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.setContentType(new MediaType("application", "json", StandardCharsets.UTF_8));
    }

    public String sendErrorMessage(Map<String, Object> teamsValue) {
        teamsValue.put("date", LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-mm-dd hh:mm:ss")));
        return requestHttpHookCall(teamsValue, sendMessageErrorUrl);
    }

    private String requestHttpHookCall(Map<String, Object> teamsValue, String sendMessageErrorUrl) {
        Sections section = new Sections(profile);
        section.makeFacts(teamsValue);
        List<Sections> sections = Arrays.asList(section);

        TeamsMessage message = new TeamsMessage(sections);
        HttpEntity<String> request;

        try {
            request = new HttpEntity<>(mapper.writeValueAsString(message), headers);
            ResponseEntity<String> result = restTemplate
                    .exchange(sendMessageErrorUrl, HttpMethod.POST, request, String.class);
            log.debug("teams sendMessage result={}", result);
            return result.getBody();
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            log.error("Error found During sending teams message-parse {}", e.getMessage());
            return null;
        }
    }
}

```

## json

```
{
  "themeColor": "0076D7",
  "sections": [
    {
      "activityTitle": "BATCH-ERROR(local)",
      "facts": [
        {
          "name": "date",
          "value": "2022-11-26 03:11:08"
        },
        {
          "name": "message",
          "value": "logWriter4Rollback :: rollback for invalid sync row count ( \n### Error updating database.  Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolationException: Duplicate entry 'KEY' )"
        },
        {
          "name": "statusCode",
          "value": "500"
        }
      ],
      "markdown": true
    }
  ],
  "@type": "MessageCard",
  "@context": "http://schema.org/extensions"
}
```


## 참고 
- http://jsonviewer.stack.hu/
- https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/what-are-webhooks-and-connectors