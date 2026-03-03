ㄱflink가 제공할 수 있는 이점 :

데이터 소스들 연계하여 데이터를 조작하고 평가하여 새 데이터를 만들어내는 것을 엔진이 제공하는 api 위에서 편하게 할 수 있음.

historical하게 이전 데이터포인트를 [window 컨셉](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/operators/windows/)으로 볼수도 있음.

각 데이터 포인트가 들어왔을때 [stateful](https://nightlies.apache.org/flink/flink-docs-master/docs/concepts/stateful-stream-processing/)하게 작업을 할 수도 있음.

현재 operation alert에서 필요한 것들:

1.  no op suspicious battery swap
    1.  swap 기록 없이 배터리 id가 패킷에서 바뀐 케이스
    2.  stream: packet
        1.  패킷들에서 bms id가 갑자기 달라지는 케이스 검출 가능 (key by vid)
        2.  근데 bms id들이 swap 기록에 있는지 체크해야하는데 그게 외부 데이터베이스를 갔다오는건데 이걸 스트림 처리하기가 어려움.
        3.  → change batter id 이벤트들을 모아서 flatmapper에 넘길때 mapper안에서 db access
	        1. 근방에 있던 swap 기록이 없으면 alert 발생
	        2. 물론 sync io라 기분이 나쁨. 발전할 수 있는 방향이?
2.  battery connect ok count
    1.  connect ok count가 1,2인 경우 (≠3)
    2.  keyed stream 이용하면 될듯
3.  unauth charging
5.  abnormal odo
	1. 
6.  no battery swap after distance

- 공통으로 들어가는 로직이 last transaction odometer 디비에서 가져오는거 . 근데 이거 redis에도 캐싱중. 어떻게 할까?
	- battery tx 발생시 이벤트 발생이 가장 깔끔한듯..
- vin - term id mapping은 어디서 가져올까?
	- table 등록을 하는게 나을듯.



## 로컬 환경 개발 + 디버깅
?? 조사필요


---
### 참고 링크들 
https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/dev/datastream/operators/asyncio/
https://medium.com/amarpandey/asynchronous-io-using-apache-flink-6c2d4770508a

timing에 대한 내용: https://medium.com/amarpandey/asynchronous-io-using-apache-flink-6c2d4770508a
https://jess-m.tistory.com/31

gradle 기본 셋업: https://nightlies.apache.org/flink/flink-docs-master/docs/dev/configuration/overview/
k8s 위에서의 사용: https://nightlies.apache.org/flink/flink-docs-master/docs/deployment/resource-providers/native_kubernetes/


- apache beam이라는것도 있음. 좀 더 추상화 레벨이 높은 데이터 파이프라인 구축을 위한 것으로 보임
	- https://jaemunbro.medium.com/gcp-dataflow-apache-beam-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-a4f5f09b98d1
flink example kotlin  - https://github.com/rockmkd/flink-examples-kotlin/blob/master/pom.xml