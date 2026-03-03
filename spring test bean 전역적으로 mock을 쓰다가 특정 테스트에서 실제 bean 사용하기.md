
셋업:

실제 service bean
MyServiceImpl : MyService


test code:
mock bean 
```kotlin
@Configuration
class MyMockTestConfig {
    @TestConfiguration
    class MockMyServiceTestConfig {
        @Bean
        @ConditionalOnMissingBean(name = ["myServiceTestImpl"])
        @Primary
        fun myServiceMockClient(): MyService {
            return mockk(relaxed = true)
        }
    }

}


// 특정 테스트
@SpringBootTest  
@Import(MyServiceRealTest.MyConfig::class)  
class MyServiceRealTest {
    
    @Order(Ordered.HIGHEST_PRECEDENCE)
    class MyConfig {
        @Bean
        @Primary
        fun myServiceTestImpl(
            findRoleUseCase: FindRoleUseCase,
            findTenantUseCase: FindTenantUseCase
        ): MyService  = MyServiceImpl(findRoleUseCase, findTenantUseCase)
    }

}

```


MyMockTestConfig 설명:
Configuration안에 test configuration을 넣은 이유는 test configuration이 top level로 있으면 자동으로 pick되지 않으므로. 

- `@Primary` annotation은 `MyServiceImpl`보다 우위에 있기위해.
- `@ConditionalOnMissingBean`이 중요한데, 특정 테스트의 경우 Import를 하면서 Config class가 Ordered.HIGHEST_PRECEDENCE 를 차지하면서 미리 다른 bean을 등록해버렸을때 중복된 타입의 bean을 로딩하지 않기 위해 써놨음.

이런 셋업이면, MyServiceRealTest의 경우에는 MyConfig inner class가 `@Import` 를 통해 로딩될 때 가장 높은 우선순위로 로딩되면서 `MyService`타입의  myServiceTestImpl bean이 `@Primary`로 등록되고, 실제 코드에 있는 bean을 덮어쓴다. (이때 아마 allow bean overriding이 돼있어야하긴할듯)

MyServiceRealTest 외의 테스트는 myServiceTestImpl이 로딩되지 않으므로 myServiceMockClient가 로딩되어 mocking을 쓸수 있게 됨.

