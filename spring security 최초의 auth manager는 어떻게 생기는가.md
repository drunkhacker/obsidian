AuthenticationConfiguration에서 getAuthenticationManager()를 호출할 때 없으면 만들게 되어있다.
해당구현을 보면, AuthenticationMangerBuilder의 build를 부르게 돼있다. 
그러면, AuthenticationMangerBuilder는 어디서 나오는가? - 위 getAuthenticationManager 호출 시 없으면 만듬. 그 후에 authmanagerbuilder의 build 함수가 호출(AbstractSecurityBuilder) 되고, 그 후 configure method에서 (AbstractConfiguredSecurityBuilder) SecurityConfigurer를 돌게 되어있음. 
configurer 중 InitializeAuthenticationProviderManagerConfigurer 가 AuthenticationProvider 빈을 하나 가져와서 등록하게 돼있음. 

구조를 보니 여러 provider가 있으면 동작을 안하게 되어있다..

