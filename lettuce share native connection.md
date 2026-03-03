
Spring Data Redis에 있는 `setShareNativeConnection(boolean)`이라는 함수가 기본적으로 `true`로 세팅되게 돼있는데 이는 connection 풀을 무시하고 native connection 하나만 가지고 커넥션을 하게끔 만든다. [관련글](https://stackoverflow.com/a/63720409) 

LettuceConnectionFactory [문서](https://docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/connection/lettuce/LettuceConnectionFactory.html)에 보면, 
	The shared native connection is never closed by `LettuceConnection`, therefore it is not validated by default on `getConnection()`. Use `setValidateConnection(boolean)`to change this behavior if necessary. If `shareNativeConnection` is true, a shared connection will be used for regular operations and a `LettuceConnectionProvider` will be used to select a connection for blocking and tx operations only, which should not share a connection. If native connection sharing is disabled, new (or pooled) connections will be used for all operations.


애초에 Redis가 single threaded라 커넥션 풀을 맺어봤자 필요가 없다는 [이야기](https://parkcheolu.tistory.com/192)도 있다.

여기서 궁금증: 멀티 node인 경우에는 어떻게 되는가? 

