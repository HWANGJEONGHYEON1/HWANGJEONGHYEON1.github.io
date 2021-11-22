---
layout: post
title:  "싱글톤에 대하여(멀티스레드)"
date:   2021-11-22 19:22:21 +0900
categories: java, singleton
---
# 멀티스레드에서 싱글톤

- 기존에 회사 로직 중 컨트롤러에서 new ResponseEntity로 보내는 작업을 리팩토링을 진행하였다( 컨트롤러에서 굳이 에러처리와 new로 생성하는 부분)


## 기조 소스

```java
    @RequestMapping(value = "/api/roles", method = {RequestMethod.POST})
    public ResponseEntity<UserInfo> createRoles(@RequestBody @Valid RoleParam roleParam,
                                                        BindingResult bindingResult) throws NotValidParamException {

        if (bindingResult.hasErrors()) {
            throw new NotValidParamException(bindingResult.getAllErrors().get(0).getDefaultMessage());
        }

        return new ResponseEntity(roleManageService.getUserInfo(roleParam));
    }
```

## 리팩토링 공통화 ResponseObject

```java

@JsonInclude(JsonInclude.Include.NON_DEFAULT)
@Getter
public class ResponseObject<T> {
    private T data;
    private Object meta;
    private ResponseLink links = null;
    private ResponsePagination pagination = null;
    private List<ResponseError> errors = null;

}

public class ResourceConverter {
    public static <T> ResponseObject<T> toResponseObject(T data){
        ResponseObject<T> responseObject = new ResponseObject<>();
        responseObject.setData(data);
        return responseObject;
    }

    public static <T, U> ResponseObject<T> toResponseObject(T data, U meta){
        ResponseObject<T> responseObject = toResponseObject(data);
        responseObject.setMeta(meta);
        return responseObject;
    }
}

  @RequestMapping(value = "/api/roles", method = {RequestMethod.POST})
    public ResponseEntity<ResponseObject> createRoles(@RequestBody @Valid RoleParam roleParam,
                                                        BindingResult bindingResult) throws NotValidParamException {

        return ResponseConverter.toResponseObject(roleManageService.getUserInfo(roleParam));
    }
}

```

- 설명
  - 기존 에러를 던지는 로직은 서비스 단으로 옮기고 ControllerAdvice에서 처리할 수 있도록 만들었다.
  - 무수히 많은 컨트롤러에서 위의 코드식으로 컨트롤러에서는 ResponseObject만 리턴해줄 수 있게 만들었다.
     - 사실 컨트롤러에서는 어떤 파라미터를 가공하여 서비스에 넘기는것도 괜찮다라는 생각은 가지고 있지만, 팀의 문화를 따르기로 결정했다.

## 의문점
- 현재는 toResponseObject라는것은 결국에 ResponseObject를 새로만들어서 사용자의 요청이 올 때마다 객체를 생성하는 구조이다.
- `그렇다면 괜찮은 코드인가?`
- 생각해 볼 수 있는 것
  - `ResourceConverter, ResponseObject를 빈으로 등록해서 싱글톤임을 보장해보면 어떨까?`

## 싱글톤
- 전역변수를 사용하지 않고 객체를 하나만 생성하도록 해준다.
- 생성자가 여러차례 호출되어도 하나임을 보장
- `한번만 객체를 생성하여 메모리 낭비를 줄여준다` < 내가 생각했던 포인트이다.
- 주의점
  - 상태를 가진 객체를 Singleton으로 만들면 안된다 => 어플리케이션 내에서 한개의 인스턴스가 존재하고, 이를 전역에서 접근한다면 다른 스레드에서 객체의 상태를 변경시킬 여지가 있다. 상태가 공유된다는것은 매우 위험하기 때문에 무상태성 으로 유지해야한다.
 
## 예시
- 요청1과 요청2가 있을 때 동시에 같은 toResponse가 왔을 때 지금 생성된 객체는 하나인대, 요청1 호출 되었지만, 동시에 2도 요청되어서 요청2에 대한 값을 1이 받을 수 있다.


## 해결
- 해결이라기보다는 현재 api 사내에서만 사용되기 때문에 문제가 될 수 없다고 판단하여 static 그대로 가기로 했지만 만약에 엄청난 양의 서비스라면 공통 모듈을 두지도 않고 굳이 response 오브젝트를 줘야하나? 리스트면 리스트 그대로 주고 성공한다면 200, 에러처리를 잘 했다면 그에따른 에러처리를 주지 않을까 생각한다.



