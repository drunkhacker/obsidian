[yidongnan](https://github.com/yidongnan/grpc-spring-boot-starter)
[lognet](https://github.com/LogNet/grpc-spring-boot-starter)

두 종류가 있고, lognet이 지금 tada와 onion에서 쓰는 것.

https://github.com/LogNet/grpc-spring-boot-starter/issues/144 이런 글이 있는것보면 [yidongnan](https://github.com/yidongnan/grpc-spring-boot-starter)것이 좀 더 기능이 많은듯하다.

현재 시점 (2023.07.29)에서는 yidongnan이 좀 더 star나 fork가 많음.
lognet 3k vs yidongnan 2.1k

lognet starter는 test 목적을 위해 netty server를 띄우지 않고 in-process server를 띄우는 옵션을 제공한다.
```yaml
 grpc:
    enabled: false (1)
    inProcessServerName: myTestServer (2)
```


yidongnan껏도 [testing할때 inprocess 서버를 지원](https://yidongnan.github.io/grpc-spring-boot-starter/en/server/testing.html)한다.
```
@SpringBootTest(properties = {
        "grpc.server.inProcessName=test", // Enable inProcess server
        "grpc.server.port=-1", // Disable external server
        "grpc.client.inProcess.address=in-process:test" // Configure the client to connect to the inProcess server
        })
```