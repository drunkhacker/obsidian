https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Migration-Guide

인증부분 -> [spring authorization server](https://github.com/spring-projects/spring-authorization-server)가 따로 생김.
> This project replaces the Authorization Server support provided by Spring Security OAuth.

[Auth server ref](https://docs.spring.io/spring-authorization-server/reference/)
버젼 문제:
spring sec 5.8.9 랑 같이 쓰려면 버젼이 0.4.5 여야한다.
`implementation("org.springframework.security:spring-security-oauth2-authorization-server:0.4.5")`
기능에 제약이 있는지는 확인 해봐얄듯

관련 문서:
https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-07

[configuration customzing](https://docs.spring.io/spring-authorization-server/reference/configuration-model.html#customizing-the-configuration)섹션에 각종 customize에 대한 내용이 나와서 이걸 읽어보고 [core model](https://docs.spring.io/spring-authorization-server/reference/core-model-components.html)읽어보는게 좋을듯

OAuth2AuthorizationService: [공식문서 설명](https://docs.spring.io/spring-authorization-server/reference/core-model-components.html#oauth2-authorization-service)이 좀 부족하긴한데.. 


custom grant type들 구현하기
https://docs.spring.io/spring-authorization-server/reference/guides/how-to-ext-grant-type.html
https://glenmazza.net/blog/entry/spring-auth-server-custom-grant
예제코드: https://github.com/Basit-Mahmood/spring-authorization-server-password-grant-type-support/tree/master/SpringAuthorizationServer/src/main/java/pk/training/basit/oauth2/authentication
복수개의 custom granter들 등록하는 방법 : https://github.com/Basit-Mahmood/spring-authorization-server-password-grant-type-support/blob/master/SpringAuthorizationServer/src/main/java/pk/training/basit/configuration/AuthorizationServerConfiguration.java#L91

- AuthenticationConverter: HTTP request -> Authentication
- AuthenticationProvider:  Authentication -> Authentication (but with access token)
기존 `AuthorizationServerEndpointsConfigurer`가 authorizationServerSecurityFilterChain내에서 `OAuth2AuthorizationServerConfigurer`를 사용하는 것으로 바뀐듯 

OAuth2Authentication이 deprecated됨. 이제 직접 OAuth2AccessToken및 그걸 감싸는 OAuth2AccessTokenAuthenticationToken 등을 만들어야함.
TokenEnhancer를 통한 converting도 deprecated, tokenGenerator를 사용하여 생성하고, tokenGenerator빈을 세팅할때 jwtEncoder를 세팅해줄 수 있음.

tokenGenerator는 어떻게 세팅해야 jwt가 나오나?
https://docs.spring.io/spring-authorization-server/reference/core-model-components.html#oauth2-token-generator
RegisteredClient의 속성을 SELF_CONTAINED 로 하면 JWT.

security filter chain을 여러개 등록할 수 있다.
https://www.danvega.dev/blog/multiple-spring-security-configs
