`@SpringBootTest` 어노테이션을 쓰고 난 다음에도 여러 auto configure를 붙일 수 있다.
SpringBootTest의 기본 web env가 mock인데 .. 이렇게 되면 기동되는 servlet container가 모킹되므로 이를 테스트하기 위해서는 mock mvc가 필요함. -> `@AutoConfigureMockMvc` 사용하여 
MockMvc 객체를 주입받아 쓸것.
