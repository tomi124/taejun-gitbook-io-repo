# HttpSecurity, WebSecurity의 차이



> HttpSecurity와 WebSecurity의 차이점에 대해 찾아본 내용을 정리한 글입니다.

```java
@Override
public void configure(WebSecurity web) throws Exception {
    web.ignoring().antMatchers("/health", "/health/**");
}
@Override
public void configure(HttpSecurity http) throws Exception {
     http.csrf().disable()
        .authorizeRequests()
        .antMatchers("/health", "/health/**").permitAll()
        .anyRequest().authenticated();
}
```

### 1) WebSecurity <a href="#1-websecurity" id="1-websecurity"></a>

antMatchers에 파라미터로 넘겨주는 endpoints는 Spring Security Filter Chain을 거치지 않기 때문에 '인증' , '인가' 서비스가 모두 적용되지 않는다.

또한 Security Context를 설정하지 않고, Security Features(Secure headers, CSRF protecting 등)가 사용되지 않는다.

Cross-Site-Scripting(XSS), content-sniffing에 대한 endpoints 보호가 제공되지 않는다.

일반적으로 로그인 페이지, public 페이지 등 인증, 인가 서비스가 필요하지 않은 endpoint에 사용한다.

​

아래의 경우처럼 WebSecurity, HttpSecurity에 모두 설정을 한 경우 WebSecurity가 HttpSecurity보다

우선적으로 고려되기 때문에 HttpSecurity에 인가 설정은 무시된다.

```java
@Override
public void configure(WebSecurity web) throws Exception {
    web
        .ignoring()
        .antMatchers("/publics/**");
}

@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .antMatchers("/publics/**").hasRole("USER") // no effect
        .anyRequest().authenticated();
}
```

### 2) HttpSecurity <a href="#2-httpsecurity" id="2-httpsecurity"></a>

antMatchers에 있는 endpoint에 대한 '인증'을 무시한다.

Security Filter Chain에서 요청에 접근할 수 있기 때문에(요청이 security filter chain 거침) 인증, 인가 서비스와

Secure headers, CSRF protection 등 같은 Security Features 또한 사용된다.

취약점에 대한 보안이 필요할 경우 HttpSecurity 설정을 사용해야 한다.

### 참고 <a href="#undefined" id="undefined"></a>

[https://ravthiru.medium.com/springboot-security-configuration-using-httpsecurity-vs-websecurity-1a7ec6a23273](https://ravthiru.medium.com/springboot-security-configuration-using-httpsecurity-vs-websecurity-1a7ec6a23273)\
[https://stackoverflow.com/questions/56388865/spring-security-configuration-httpsecurity-vs-websecurity](https://stackoverflow.com/questions/56388865/spring-security-configuration-httpsecurity-vs-websecurity)

출처 : [https://velog.io/@gkdud583/HttpSecurity-WebSecurity%EC%9D%98-%EC%B0%A8%EC%9D%B4](https://velog.io/@gkdud583/HttpSecurity-WebSecurity%EC%9D%98-%EC%B0%A8%EC%9D%B4)\
