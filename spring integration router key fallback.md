메시지의 타입에 따라 라우팅을 결정하는 구조를 짰다고 하자.
method invoking router를 이용해서 method는 Object::getClass 를 써서 진행하는데, 

subflowmapping에 의도한 몇개의 클래스를 집어넣고, defaultSubflowMapping에 집어넣으면 될것 같지만, (switch case default 처럼), 아마 subflowmapping으로 안들어오고 자꾸 다른 routing을 시도할 것이다. 

이유:

router 메소드의 결과로 subflow 매핑을 짜면 각 경우를 나타내는 값들은 org/springframework/integration/router/AbstractMappingMessageRouter.java의 channelMappings 에 key로 지정이 된다.

메시지가 오고 그것을 어떤 라우팅으로 보낼지는 determineTargetChannels가 결정하는데 
메시지를 router 메소드에 집어넣고 나온 결과값이 channelMappings에서 key로 존재하는지 뒤져보고 없으면 defaultSubflow로 빠질것 같지만,
그 전에, 

addChannelKeyToCollection 함수를 불러서 내가 정의한것 외에 추가적으로 라우팅을 정의하려는 짓을 한다! 
이 과정에서 addChannelFromString 를 부르고, 해당 메소드는 channelKeyFallback 속성이 true면, 라우터 메소드의 결과값을 바로 채널로 삼아서 해당하는 채널로 메시지를 보내려하는데, 이때 채널이 정의되어 있지 않으면 터진다.

만약 channelKeyFallback이 false라면, 아무짓도 안하고 addChannelFromString에서 리턴.역시나 addChannelKeyToCollection에서도 아무 사이드 이펙트가 일어나지 않음.
그러케 되면 determineTargetChannels에서 해당 메시지에 대한 target channel이 없다는 뜻의 빈 리스트를 리턴하고, deafultoutputchannel로 보내게 되는데, 
deafult output channel은 default subflow mapping을 정의하면 그 친구의 input으로 정의되게 돼있어서 의도된 동작을 한다.

또는 defaultsubflowmapping말고 defaultoutputchannel을 subflow def 단에서 정의해도 됨.

channelKeyFallback = false로 세팅하고 싶다면, noChannelKeyFallback() 메소드를 쓸것.
