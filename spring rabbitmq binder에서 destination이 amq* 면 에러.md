[[rabbitmq 에서 amq.* queue name은 internal reserved]]와 연관 

anonymous queue name을 앞에 destination 이름을 붙여서 만들기 때문. 해결하려면 queueNameGroupOnly 옵션을 켜면 됨.

[내부코드](https://github.com/spring-cloud/spring-cloud-stream-binder-rabbit/blob/0ce832f2bc8e5e9be60bc98a38f26ce3bb5239b0/spring-cloud-stream-binder-rabbit-core/src/main/java/org/springframework/cloud/stream/binder/rabbit/provisioning/RabbitExchangeQueueProvisioner.java)
