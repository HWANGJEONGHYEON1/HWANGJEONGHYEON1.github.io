---
layout: post
title:  "SpringSecurity정보 Controller에서 받기"
date:   2021-05-08 01:15
categories: spring,security
tags: [security, jwt]
---
### Spring-Security 구현 - 2

* 프로젝트에 Spring Security, JWT 구현한다.
* 프로젝트 진행 중 Mustache에서 security-taglibs를 쓰는것을 찾기 힘들어 컨트롤러에서 데이터를 넘겨는것으로 대체

```java
    @GetMapping("/")
    public String index(Authentication auth, Model model) {
        if (auth != null) {
            UserDetails userDetails = (UserDetails)auth.getPrincipal();
            final String userName = userDetails.getUsername();
            final String role = userDetails.getAuthorities().stream().map(r -> String.valueOf(r)).collect(Collectors.joining(","));
            model.addAttribute("userName", userName);
            model.addAttribute("role", role);
        }

        return "index";
    }
```

* 이 외 다른방법
```java
@GetMapping("/index")
public void index(Pricipal principal) {
    System.out.println("username = " + principal.getName());
}
```

```java
@GetMapping("/index")
public void index(Pricipal principal) {
    Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    UserDetails userDetails = (UserDetails)principal;
    String username = principal.getUsername();
    String password = principal.getPassword();
}
```

```java
@Controller
public class IndexController {
    @GetMapping("/index")
    public void index(@AuthenticationPrincipal Member member) {
        System.out.println("username = " + member.getUsername());
    }
}

@Entity
public Member implements UserDetails {
    // ...
}

@Service
public class UserSevice implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return memberRepository.findByUsername(username);
    }
}
```