
`@SpringBootApplication`이 붙어있는 곳에서 `@ComponentScan`을 광범위하게 잡아버려서, 예를들어 basepackage가 테스트 콘피그를 포함하게 하고, 테스트콘피그에 mock 관련 설정이 있다면,

test 클래스에서 아무리 mock관련 설정 클래스를 ComponentScan의 excludeFilter로 제거를 해줘도 말을 듣지 않는다.