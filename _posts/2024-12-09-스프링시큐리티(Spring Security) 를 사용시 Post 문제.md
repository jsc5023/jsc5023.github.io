# Spring Security에서 POST 요청이 처리되지 않는 문제 해결: 회원가입 구현 중 배운 점

회원가입 기능을 구현하던 중 예상치 못한 문제가 발생했습니다. 회원가입 버튼을 눌렀을 때 컨트롤러 메서드가 호출되지 않아 디버깅이 되지 않았습니다. 원인을 찾는 과정에서 Spring Security 설정에서 POST 요청 처리를 명시적으로 추가해야 한다는 점을 깨달았고, 이를 해결하는 과정을 공유합니다.

---

## 문제 상황

회원가입을 구현하는 과정에서 **`@PostMapping("/security/sign-up")`** 메서드가 호출되지 않았습니다. 브라우저 개발자 도구의 Network 탭을 확인해도 요청이 컨트롤러에 도달하지 않았습니다. 초기 코드 구성은 아래와 같았습니다:

### 컨트롤러 코드
```java
@PostMapping("/security/sign-up")
public String signUp(HttpServletRequest request, Model model) {
    String userId = request.getParameter("userId");
    String password = request.getParameter("password");
    // 회원가입 로직 (내용 생략)
    return "redirect:/security/login";
}
```

### Spring Security 설정
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(HttpMethod.GET, "/security/*").permitAll()
                .anyRequest().authenticated())
            .build();
}
```

**원인:** Spring Security가 POST 요청을 기본적으로 보호하며, 명시적으로 허용하지 않은 POST 요청은 차단됩니다. 저는 GET 요청만 허용하도록 설정했기 때문에 POST 요청이 처리되지 않았던 것입니다.

---

## 해결 과정

문제를 해결하기 위해 아래와 같은 단계를 진행했습니다.

### 1. POST 요청 허용 추가
Spring Security 설정에서 POST 요청도 명시적으로 허용하도록 추가합니다.

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .csrf(AbstractHttpConfigurer::disable) // CSRF 비활성화
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(HttpMethod.GET, "/security/login", "/security/sign-up").permitAll() // GET 허용
                .requestMatchers(HttpMethod.POST, "/security/sign-up").permitAll() // POST 허용
                .anyRequest().authenticated()) // 나머지는 인증 필요
            .build();
}
```

### 2. 설정 테스트
1. 브라우저에서 회원가입 버튼을 클릭합니다.
2. POST 요청이 컨트롤러 메서드에 정상적으로 도달하는 것을 확인합니다.
3. Network 탭에서 200 응답 코드와 데이터 전송 결과를 확인합니다.

### 3. 결과
Spring Security 설정을 수정한 후, 회원가입 POST 요청이 정상적으로 처리되었습니다.

---

## 배운 점

이번 문제를 통해 다음과 같은 중요한 점을 배웠습니다:

- **Spring Security는 POST 요청과 GET 요청을 별도로 관리한다.**
  - POST 요청은 명시적으로 허용하지 않으면 차단된다.
- **보안 설정은 기능 구현 초기부터 명확히 정의해야 한다.**
  - 예상치 못한 차단이 발생하지 않도록 보안 정책을 설계해야 한다.
- **문제를 해결하며 배운 점을 기록하고 공유하자.**

---

## 코드 요약

### 최종 Spring Security 설정
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .csrf(AbstractHttpConfigurer::disable) // CSRF 비활성화
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(HttpMethod.GET, "/security/login", "/security/sign-up").permitAll()
                .requestMatchers(HttpMethod.POST, "/security/sign-up").permitAll()
                .anyRequest().authenticated())
            .build();
}
```

### 회원가입 컨트롤러
```java
@PostMapping("/security/sign-up")
public String signUp(HttpServletRequest request, Model model) {
    String userId = request.getParameter("userId");
    String password = request.getParameter("password");
    // 회원가입 로직 처리
    return "redirect:/security/login";
}
```

---

## 마무리

이번 경험을 통해 Spring Security 설정의 중요성을 다시 한번 깨달았습니다. 앞으로도 보안 설정에 대한 이해를 기반으로 더 안전한 애플리케이션을 개발할 수 있을 것 같습니다.

