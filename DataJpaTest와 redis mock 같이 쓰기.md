```kotlin
@DataJpaTest
@AutoConfigureDataRedis  
@EnableRedisMockTemplate
```


@DataRedisTest도 있는데 @DataJpaTest와 같이 쓸수가 없다. 근데 DataRedisTest를 까보면 
```
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@BootstrapWith(DataRedisTestContextBootstrapper.class)  
@ExtendWith(SpringExtension.class)  
@OverrideAutoConfiguration(enabled = false)  
@TypeExcludeFilters(DataRedisTypeExcludeFilter.class)  
@AutoConfigureCache  
@AutoConfigureDataRedis  
@ImportAutoConfiguration
```

이렇게 돼있는데 여기서 중요해보이는건 `@AutoConfigureDataRedis` 이거인듯하여 집어넣으면 될듯.

