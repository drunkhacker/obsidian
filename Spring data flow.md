
dataflow server - Flink의 job manager에 대응되는듯 
skipper - Flink의 task manager에 대응되는듯 

job = array of (task)


paritioning using rabbitmq
https://seroter.com/2017/09/05/surprisingly-simple-messaging-with-spring-cloud-stream/


spring data flow는 Flink와 다르게 별도 execution engine을 제공한다기보다는 stream을 연결하는데에 주안점을 둔 듯. 

장점으로는 익숙한 프레임워크, control 가능한 아키텍처 및 인프라 세팅 정도 
단점으로는 기능이 빈약하다 정도일것.

예를들어 Flink에서 제공하는 stateful stream processing을 하려면 별도의 persistence layer를 붙여서 구현해야함. Flink에서는 내부적으로 data stream에서 들어오는 데이터의 key field를 잡아서 key별로 스트림 구성 + 해당하는 스트림 인스턴스 별로 관리되는 state를 이용하여 로직 개발이 가능함. spring data flow에서도 해당 기능이 빈약하다는 것을 인지하고 있고 그것에 대한 대안들을 언급하긴 함. (별도 persist layer및 로직 구현 등으로 풀어내기) 

인프라 관리의 어려움을 고민해봤을때는 명확하게 다 드러나는 환경하에서 개발하는 것이 좀 더 장점이 많다고 여겨짐. 익숙한 프레임워크 (거꾸로 말하면 Flink 등의 새 엔진의 learning curve) + controllability 확보가 기능의 빈약함으로 인한 추가 구현 burdensome보다 중요해보임. 추가 구현해야하는 부분들이 엄청 크진 않아서 감내할 만한 정도로 보임. 


----
onion T1 data 스트림 처리를 위한 플로우 추가 
- 현재는 data packet을 stdout에 떨구면 fluent -> airflow 태스크 정의 들을 통해 table 형태로 athena에서 쓸수 있게끔 s3에 올리는 작업까지 수행
- s3에 올라간 데이터들은 athena속도 때문에 실시간 이벤트 처리에 사용하긴 힘듬
- 1안: collector쪽에 rabbitmq exchange로 쏘는 것을 만들자
	- 이미 파싱된것 활용 가능
	- collector가 꼭 있어야함
```
┌────────┐      ┌─────────┐              ┌──────────┐
│ MQTT   │      │collector│              │          │
│ broker ├──────►         ├─┬───grpc────►│  battery │
│        │      │         │ │            │  svc     │
└────────┘      └─────────┘ │            │          │
                            │            └──────────┘
                            │
              ──────────────▼───────
                RabbitMQ
              ──────────────────────
```
- 2안: mqtt 브로커쪽에서 바로 가져와서 rabbitmq exchange로 쏘는것을 만들자
	- 추가 파싱 필요.
	- 데이터 스트림 작업 쪽을 분리할 수 있음 (그렇게 중요한가?)
	- 기존 collector 코드에 변화없음 
```
      ┌────────┐      ┌─────────┐              ┌──────────┐
      │ MQTT   │      │collector│              │          │
      │ broker ├──────►         ├─────grpc────►│  battery │
      │        │      │         │              │  svc     │
      └────┬───┘      └─────────┘              │          │
           │                                   └──────────┘
      ┌────▼───┐
      │ spring │
      │ data   │
      │ flow   │
      └────┬───┘
           │       ──────────────────────
           └──────►  RabbitMQ
                   ──────────────────────

```

Spring data flow에 기존에 있는 source들을 프로그램에서 사용하는 방법??

spring data flow server 등등이 또 또는건 원하는 것이 아님 . 그럴거면 flink 쓰지..
별도의 stream processor 가 있고 그게 각종 소스들을 폴링하여 stream을 새로 생성해놓는 정도면 됨.
그 스트림을 stream processor 내부에서만 써도 되고.. 아니면 필요하다면 rmq로 노출해도 되고. 



https://spring.io/projects/spring-cloud-stream-applications

https://github.com/spring-cloud/stream-applications/tree/main/applications/source/jdbc-source
https://github.com/spring-cloud/stream-applications/tree/main/functions/supplier/jdbc-supplier


https://github.com/spring-cloud/spring-cloud-function

https://github.com/spring-cloud/spring-cloud-function/blob/main/spring-cloud-function-samples/function-sample-pojo/src/main/resources/application.properties

https://velog.io/@kshired/Spring-Cloud-Function%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%B4-Spring-Application%EC%9D%84-AWS-Lambda%EC%97%90-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0

https://velog.io/@zenon8485/Spring-Cloud-Stream-%EC%A0%95%EB%A6%AC


```
In conclusion, Spring Cloud Function and Spring Cloud Stream are two distinct projects that serve different purposes. While Spring Cloud Function is a framework for building standalone, functional microservices, Spring Cloud Stream is a framework for building event-driven microservices that process and emit streams of data. Spring Cloud Stream builds on Spring Cloud Function to fully support a functional programming model.
```


splitting이 뭐지?
