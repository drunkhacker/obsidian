테스트 하고자하는 컴포넌트들만 올려서 integration test를 빠르게 하자는게 목표
`@SpringBootTest`는 모든 컴포넌트를 다 올리기 때문에 기동이 느리다. (메모리도 많이 먹는듯)

WebMvcTest 나 DataJpaTest 등이 그런 슬라이스들에 속함.

참고: https://reflectoring.io/spring-boot-test/ 


위 링크에는 application context 로딩시 프로퍼티를 다르게 줌으로써 테스트 시 원하지 않는 동작을 끌 수 있는 예제를 소개함. 예를 들면 `@EnableScheudle`같은 경우는 `@Configuration`에 `@ConditionalOnProp`등을 먹여서 제어할 수 있음. 그러면 테스트 할때는 prop을 제어해서 끄면 됨.

