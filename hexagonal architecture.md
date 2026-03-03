port and adapter 패턴이라고도 불림.
지금 onion 쪽의 패키지 구조가 이와 비슷하다.
결국엔 layer segregation pattern.

- port는 interface
- adapter는 port를 사용하여 application과 interaction을 하는 존재.
	- 클라이언트의 요청을 server로 인터페이스를 통해 대리하는 존재.


domain layer
application layer
outer layer

outer layer = infra + interface
application layer = usecase
domain layer = service + domain obj

https://engineering.linecorp.com/ko/blog/port-and-adapter-architecture
