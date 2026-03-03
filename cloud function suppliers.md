https://github.com/spring-cloud/stream-applications/tree/main/functions/supplier/tcp-supplier
- 여기에 있는것들이 Supplier들을 정의하고 이걸 다른데에서 인젝트받아서 쓰는 구조.
- 다른데에서 -> 다른 함수들하고 연결해서 쓴다는 의미.. 겠지?

위 예제들 중 [TCP Supplier](https://github.com/spring-cloud/stream-applications/tree/main/functions/supplier/tcp-supplier)를 보면 Supplier를 정의하기 위해 내부에서 IntegrationFlow를 사용하는 것을 볼 수 있다. 

이말인 즉슨 SI를 SCF에서 활용할수 있다는 뜻..
