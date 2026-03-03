custom auth provider를 넣기위해, 
OAuth2AuthorizationServerConfigurer에서 tokenEndpoint를 customize하는 부분에서 provider를 집어넣어볼 수 있다. 
provider가 여기서 auth manager에 대한 참조를 할 수 있으므로, http.getSharedObject를 사용하고 싶지만, 그것은 불가능.

왜냐면 http.build가 되기 전에는 shared object가 접근되는것이 authenticationManagerBuilder (및 무슨 어쩌구 Context하나..)밖에 없음. 그런데, OAuth2AuthorizationServerConfigurer가 configure(httpsecurity)를 부르면서 각종 shared object를 세팅하게 돼있음.
이때 tokengenerator 및 authenticationmanager를 세팅함.

정확히는, OAuth2AuthorizationServerConfigurer가 다시 OAuth2TokenEndpointConfigurer를 부르고, 그 안에서 init, configure 두 메소드가 있는데,
init 같은 경우는 http.build() 이전에 불림. 이때, OAuth2TokenEndpointConfigurer의 init에 의해 tokengenerator는 생성이 돼있음.
또한, authenticationProvidersConsumer (즉, provider들을 추가로 더 더할수 있는 곳)이 init에서 불리게 돼있음.

그러나, authentication manager는 httpSecurity의 beforeConfigure 메소드에서 생성되게 돼있다.
beforeConfigure는 http.build() 과정에서 내부적으로 불리게 돼있음. build과정은,
beforeInit -> init -> beforeConfigure -> configure -> performBuild
 순으로 진행되고, 따라서 authenticationProvidersConsumer는 init에서 불리기 때문에 auth manager에 접근할 수 가 없다.

아니면 authmanagerbuilder를 가지고 만들어서 세팅해줄순있음.

또한,

`@EnableWebSecurity`가 `@EnableGlobalAuthentication`을 부르고, 이는 `AuthenticationConfiguration`를 기본으로 임포트하므로, `@Bean`으로 AuthenticationConfiguration를 주입받아 거기서 만들어주어도 됨. 
그런데 이렇게 만들어진 기 생성된 bean은 나중에 
HttpSecurityConfiguration이 httpSecurity() 메소드를 통해 httpSecurity bean을 만들어낼때 parentAuthenticationManager로 세팅되게 됨. (authenticationConfiguration.getAuthenticationManager를 부르기 때문이고, bean을 만들면서 이미 authenticationConfiguration 내부에 AuthenticationManager가 세팅되어있어서 getAuthenticationManager는 해당 AuthenticationManager를 리턴함.)

사실 authmanager 어느쪽에 provider를 붙여도 실제 auth에 큰 차이는 없다. 
(authmananger가 authenticate에 실패하면 parent로 옮겨가서 다시 시도하기 때문..)

manager를 만드는 두 메소드는 각각 
HttpSecurityConfiguration.java의 httpSecurity()와 
AuthenticationConfiguration.java의 getAuthenticationManager() 참고.

----

ProviderManager 를 한번 들여다보면 조을듯