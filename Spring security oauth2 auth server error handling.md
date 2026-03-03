OAuth2AuthorizationEndpointFilter 혹은 OAuth2TokenEndpointFilter를 보면 `doFilterInternal` 안에서 try - catch 로 감싸서 `authenticationManager.authenticate()`를 호출하는 부분이 있다.

이때 `OAuth2AuthenticationException` 을 잡아서 `authenticationFailureHandler`로 처리를 하는데, 이는 authorizationServerConfigurer의 errorResponseHandler 로 세팅해줄수있다.

다만, OAuth2AuthenticationException의 타입밖에 처리가 안된다. 
(옛날버젼에서는 일반적인 exception 타입도 처리가 됐었는데..)

구버전 구현을 보면, 
DefaultWebResponseExceptionTranslator.java 에서 exception을 translate하게 돼있다.
기본구현은 여기서 throwableAnalyzer 라는것을 사용하는데, 이는 신버젼(?)의 exceptionTranslationFilter에도 적용이돼있다. 이 exception translator를 어떻게 잘 변형하면 .. 될거 같은데.

