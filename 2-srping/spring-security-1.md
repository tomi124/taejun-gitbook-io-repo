# Spring Security 에서 인증 과정



![](https://media.vlpt.us/images/suker80/post/311c1f26-8645-47bf-84e6-c0abb81e3bf8/image.png)

#### Spring security 에서의 인증절차 <a href="#spring-security" id="spring-security"></a>

인증 필터는 여러가지가 있는데 그 중 가장 대표적이고 보편적이게 사용하는것이 name과 password를 사용해서 UsernamePasswordFilter를 이용 하는 것이다.

#### 1. UsernamePasswordAuthenticationFilter <a href="#1-usernamepasswordauthenticationfilter" id="1-usernamepasswordauthenticationfilter"></a>

필터에 진입하게 되면 정해진 url인지와 HTTP 메소드인지 검증한다. ( 디폴트는 /login , Method = Post)

UsernamePasswordAuthenticationFilter 에서는 입력받은 username과 password를 가지고

UsernamePasswordAuthenticationToken을 생성해낸다.

생성해낸 토큰을 AuthenticationManager를 이용해서 검증을 하게 된다.

#### 2. AuthenticationManager <a href="#2-authenticationmanager" id="2-authenticationmanager"></a>

authenticationManager는 인터페이스로 구현이 되어있으며 이를 상속받아 구현 한 구현체로 인증을 진행하게 된다.

AuthenticationManagerBuild 클래스 에서는 spring security 기본 provider와 authenticationManager를 설정을 하게 된다.

![](https://media.vlpt.us/images/suker80/post/56fdc539-f1bb-409b-89d3-a652ada3b391/image.png)

위의 코드에서는 AuthencationProvider를 AuthenticationManager의 구현체인 ProviderManager에게 등록하고 있다.

![](https://media.vlpt.us/images/suker80/post/959fc9c3-3d8d-4e47-a557-70b6cbcedf1a/image.png)

인증을 수행할 provider들을 추가 하는 코드이다.

#### 3. ProviderManager <a href="#3-providermanager" id="3-providermanager"></a>

ProviderManager는 인증을 위임 받은 클래스이다.

설정에서 추가한 Provider들을 가지고 현재 Authencation 객체를 검증하게 된다.

Provider는 현재 토큰이 현재 인증 방식과 맞는 인증방식인지 검증하게 되고 맞다면 토큰을 검증하게 된다.

Providers에는

* LdapAuthenticationProvider
* AnonymousAuthenticationProvider
* RememberMeAuthenticationProvider
* OAuthAuthenticationProvider
* DaoAuthenticationProvider

등이 있다.

providerManager에서는 현재 토큰이 provider를 서포트 하면 authenticate 인증을 수행하게 된다.

![](https://media.vlpt.us/images/suker80/post/370c1a4e-59b4-4b2f-a368-32c091c4f415/image.png)

UsernamePasswordAuthenticationToken은 DaoAuthenticationProvider를 서포트하므로 여기서 인증이 마저 수행된다.

#### 4. AuthenticationProvider <a href="#4-authenticationprovider" id="4-authenticationprovider"></a>

앞서 DaoAuthenticationProvider에서는 username으로 서버에 저장되어 있는 유저 객체를 찾게 된다.

#### 5. UserDetailsService <a href="#5-userdetailsservice" id="5-userdetailsservice"></a>

UserDetailsService에서는 각 서버 방식에 맞게 username으로 userDetails를 구현한 객체를 찾게 된다.

#### 6. UserDetails <a href="#6-userdetails" id="6-userdetails"></a>

UserDetails는 사용자에 대한 이름과 비밀번호 정보와 만료가 되었는지 , 등등을 검증을 하게 된다.

#### 7. AuthenticationProvider <a href="#7-authenticationprovider" id="7-authenticationprovider"></a>

찾은 userDetails 객체를 리턴을 하게 되고 만약 찾지를 못했다면 인증이 실패하고 exception을 생성한다.

#### 8. ProviderManager <a href="#8-providermanager" id="8-providermanager"></a>

다시 providermanager로 돌아오게 되고 인증에 성공했다는 event를 생성하게 된다.

#### 9. UsernamePasswordAuthenticationFilter <a href="#9-usernamepasswordauthenticationfilter" id="9-usernamepasswordauthenticationfilter"></a>



![](https://media.vlpt.us/images/suker80/post/fc86d73d-6619-4587-91ba-447d24efae61/image.png)

인증에 성공하면 필터로 돌아오게 되고

이 필터에서 successfulAuthentication 메서드를 호출하게 된다.

successfulAuthentication 메서드 에서는

인증에성공한 토큰을

SecurityContext에 담게된다.

그 후 인증에 성공했다면 AuthenticaionSuccessHandler를 호출 하게 된다.



출처 : [https://velog.io/@suker80/Spring-Security-%EC%97%90%EC%84%9C-%EC%9D%B8%EC%A6%9D-%EA%B3%BC%EC%A0%95](https://velog.io/@suker80/Spring-Security-%EC%97%90%EC%84%9C-%EC%9D%B8%EC%A6%9D-%EA%B3%BC%EC%A0%95)
