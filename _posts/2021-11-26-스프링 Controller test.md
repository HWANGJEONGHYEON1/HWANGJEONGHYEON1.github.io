---
layout: post
title:  "스프링 컨트롤러 테스트"
date:   2021-11-26 15:22:21 +0900
categories: test, spring
---

# 스프링 컨트롤러 테스트
> mock을 이용하여 반환 되는 값과 반환되는 리스트의 사이즈가 1인것을 검증하고 싶었고, 예외가 발생했을 떄 커스텀한 예외가 잘 떨어지도록 작성했다.

## SpringController test
- junit4
- MockMvc

## Mock 한 이유
- 객체를 테스트하려면 테스트 대상 객체가 메모리에 올라가 있어야한다.
- 생성하는데 복잡한 절차가 필요하고 다른 sw에 도움이 필요한 객체도 있다.
- 실제 객체와 비슷한 가짜 객체를 만들어 테스트에 필요한 기능만 가지도록 목킹을 한다.
- @WebMvcTest, @AutoConfigureMockMvc
  - @Controller, @RestController가 설정된 클래스들을 찾아 메모리에 생성한다. 
- 서블릿 컨테이너가 구동되고 DispathcerServlet 객체가 메모리에 올라가야하는데, 서블릿 컨테이너를 목킹하여 간단한게 컨트롤러 테스트를 할 수 있따.


## 테스트코드

```java
    @Test
    public void id_조회시_사이즈가_1인_리스트() throws Exception {
        mockMvc.perform(get("/common/users?searchType=id&keyword=11"))
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.data[0].user.id", is("11")))
                .andExpect(jsonPath("$.data[0].user.userName", is("테스트")))
                .andExpect(jsonPath("$.length()", is(1))) // 컨트롤러에서 만든 리스트를 반환한 길이
                .andDo(print());
    }
    
    // id = "" 로 넣은 이유는 다른 서비스에서 해당 api를 호출했을 때, 모든 정보를 노출시킬 수 있도록 하는 로직이 있다. 그것을 방지하고자 예외로직을 추가(회사 내부로직이기 때문에 노출시킬 수 없다)
    @Test
    public void usercd로_조회시_사이즈가_2이상일경우_에러() throws Exception {
        mockMvc.perform(get("/common/users?searchType=id&="))
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                // 커스텀한 예외가 잘 터지는 확인
                .andExpect(result -> Assertions.assertThat(result.getResolvedException().getClass().isAssignableFrom(ListSizeExceedByUserCdException.class)).isTrue())
                .andExpect(status().is4xxClientError());
    }
```

## 도메인 로직 테스트
- 나중에 같이 도메인로직도 테스트를 같이 올려야겠다.



