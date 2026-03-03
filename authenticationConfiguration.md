이 클래스는 spring security 셋업에서 하나밖에 존재하지 않고, 여기서 authenticationmanager 혹은 authenticationManagerBuilder를 가져오게 되어있다. 

AuthenticationManagerBuilder같은 경우는 build()를 부를때마다 새 ProviderManager를 생성하게 되어있다.

그럼 궁금한거: authmgrbuilder

http security. build 를 하며는, 
beforeConfigure() 메소드에서 (beforeconfigure메소드는 AbstractConfiguredSecurityBuilder.doBuild에서 불리게 되어있음) httpSecurity 오브젝트에 authenticationManager가 이미 셋업되어있지 않으면, AuthenticationManagerBuilder를 가져와서 build를 호출한다.

따라서 SecurityFilterChain을 만들 때 httpSecurity를 inject받으면 (httpsecurity는 prototype bean이므로 inject 받을때마다 생성) build시 새 auth manager(provider manager)가 나온다.




