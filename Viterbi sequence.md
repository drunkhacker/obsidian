Graphhopper에서 mapmatching 시 쓰는 알고리즘 분석

`MapMatching.java` 에 있음

```java 
private List<SequenceState<State, Observation, Path>>
computeViterbiSequence(List<ObservationWithCandidateStates> timeSteps)
```

timestpes는 어떻게 뽑아오지?

같은 파일의 `MatchResult match(List<Observation> observations)` 함수에서 만듬. 
```java
List<ObservationWithCandidateStates> timeSteps = createTimeSteps(filteredObservations, snapsPerObservation);

```

filteredObservations = observation들을 적당히 filter한 것들 (error sigma * 2 이상 벌어진 log들만 취함)
snapsPerObservation = 각 observation에 대해 findCandidateSnaps()를 거쳐 나온 애들 
```java

    public List<Snap> findCandidateSnaps(final double queryLat, final double queryLon) {
        double rLon = (measurementErrorSigma * 360.0 / DistanceCalcEarth.DIST_EARTH.calcCircumference(queryLat));
        double rLat = measurementErrorSigma / DistanceCalcEarth.METERS_PER_DEGREE;
        Envelope envelope = new Envelope(queryLon, queryLon, queryLat, queryLat);
        for (int i = 0; i < 50; i++) {
            envelope.expandBy(rLon, rLat);
            List<Snap> snaps = findCandidateSnapsInBBox(queryLat, queryLon, BBox.fromEnvelope(envelope));
            if (!snaps.isEmpty()) {
                return snaps;
            }
        }
        return Collections.emptyList();
    }
```

findCandidateSnaps()는 일단 좌표 주변에 있는 node?들을 다 가져오는 느낌.. car model이나 priority등은 따지지 않는것 같다.

즉, computeViterbiSequence()의 인자인 timesteps는 query 된 lat, lng에 대해 지도상 가능한 snap들을 가져오고 나서 그것을 가지고 viterbi seq를 구함.

### Viterbi Sequence 알고리즘 

주어진 observation들에 대해서, priority Q에 처음 observation에서 도달가능한 snap들을 집어넣고 다음의 과정을 수행:

> Q에서 하나 가져옴 
> 가져온 element는 observation과 그의 snap 중 하나임.
> 가져온 element의 observation에서 next observation(timestep)의 snap들로 가는 route를 계산함
>     이때 router가 쓰이고 custom profile, model이 개입함
>      이를 s -> n1, s-> n2, ... 라 하자. 줄여서 s -> nx
> 계산된 s -> nx 중 n에 대해 확률을 업데이트할 여지가 있는 애들은 업데이트 (Relaxation 비슷) 해준다.
> 
> **Relaxation 과정**
> s -> nx 중 path가 검색된것 들 중에 기존에 알고있는 probability 보다 좀 더 나아진 확률이면 업데이트 함.
> 검색된 s->n 집합을 iterate하면서 n에 해당하는 (즉 next timestep의 snap) priority가 좀 더 나아졌으면 업데이트해준다. 
> 업데이트는 기존 q에서 n에 해당하는 element들을 삭제하고, 새 element를 만들어서 넣어준다. (새로 만드는 element는 지워진 놈과 내용이 같음. 단지 priority만 다름)
> 

확률(이 값이 Q의 priority로 사용됨)은 어떻게 계산되지?

**초기값**
```java
label.minusLogProbability = probabilities.emissionLogProbability(distance) * -1.0;
```

$\text{emissionLogProbability} = \log(N(0, \sigma^{²})(\text{distance}))$ 여기서 σ = 50 으로 잡아놓음. (변경가능) 위 코드의 distance는 snap과 실제 관측 지점 사이의 직선거리

**의미**
관측지점과 snap의 거리가 멀면 멀수록 emission prob가 낮아짐


**업데이트(Relaxation) 과정**
```java
double transitionLogProbability = probabilities.transitionLogProbability(path.getDistance(), linearDistance);
double minusLogProbability = qe.minusLogProbability - probabilities.emissionLogProbability(to.getSnap().getQueryDistance()) - transitionLogProbability;

```

$\text{transitionLogProbability} = \log(E(|\text{path거리}-\text{직선거리}|)$
- path 거리  = s->n 으로의 route 거리
- 직선거리 = s가 속한 timestep -> n이 속한 timestep간의 직선거리
-  $E(\beta) = {1 \over \beta}  e^{(-x/\beta)}$
![[Pasted image 20241218120052.png]]
(참고로 $\lambda = {1 \over \beta}$)

의미: 
- $\beta$ 는 2로 고정 (코드에)
- 직선거리와 routing 거리의 차이가 $x$
- $\beta$가 커질수록 위 그림에서 그래프가 누운쪽으로 감
- $x$가 커질수록 위 그림에서 오른쪽으로 감 
- 직선거리와 routing 거리의 차이가 클 수록 transition prob 가 낮아짐


업데이트 과정에서의 priority 계산의 의미
```java 
-(probabilities.emissionLogProbability(to.getSnap().getQueryDistance()) + transitionLogProbability)
```
`to.getSnap().getQueryDistance()`는 s->n에서 n(snap)과 next timestep간의 직선거리

(앞에 부호 -가 붙은건 priority q에서 확률이 큰게 먼저 나오게 하려고 한 것임)

log값 두개를 더했으니 사실은 곱 -> 즉, n(snap) 이 실제로 next timestep 관측지점에서 방출(emission)될 확률 X (s->n 으로의 천이 확률).
근데 그걸 왜 qe.minusLogProbability에서 또 빼지?? 확률을 또 곱한단 의미인데.. 
chatGPT에게 물어보니, '지금까지의' 누적확률을 업데이트하기위해서라고 한다. 즉, 시작점부터 n까지 오는 천이확률을 누적하기 위해서 곱하는 것.


> Without logs, the Viterbi algorithm involves multiplying probabilities:
> $P(new)=P(old)×P(emission)×P(transition)$
>
> **In Short:**
> 
> - **`qe.minusLogProbability`**: cumulative negative log probability so far.
> - **`probabilities.emissionLogProbability(...)`**: returns $\log(P(\text{emission}))$ , typically a negative number.
> - **`transitionLogProbability`**: is $\log(P(\text{transition}))$, also negative.
> 
> You update the cumulative probability by adding the logs of the new probabilities. Since you're storing negative logs, the formula for the new cumulative negative log probability is:
> 
> $-\log P(\text{new}) = -\log P(\text{old}) - \log P(\text{emission}) - \log P(\text{transition})$
> 
> Hence the code uses subtraction. The old minusLogProbability isn't replaced; it's **extended** by combining (adding) the logs of the new probabilities—just written in a way that subtracting a negative number effectively adds to the cumulative cost.


viterbi sequence의 주 main loop은 마지막 timestep의 snap들까지 도달하게 되면 끝난다.
그리고 q의 element들을 집어넣을때 back reference를 세팅하여 나중에 경로로 뽑아낼 수 있도록 한다.


### Mapmatching 과정 중 viterbi sequence 전 과정들 설명

graphhopper의 mapmatching 구현은 [이 논문](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/map-matching-ACM-GIS-camera-ready.pdf)을 거의 그대로 구현한 것 같음.

전처리 과정
- observation을 filtering함, 논문과 코드 구현에 따르면 $2\sigma_z$ 보다 더 떨어진 점들만 살림. 
- $\sigma_z$는 GPS 측정 오차의 표준편차임. 
	- 논문에서는 오차 자체도 normal distribution을 따른다고 치고 MAD 방식을 통해 구했음. 
	- GH 코드에서는 세팅 가능하게 하였음.
		- GH7에서는 50, GH9에서는 10. 작은 값일수록 측정오차가 작다고 가정하는 꼴.

snap 포인트들 계산
- 관측 gps로부터 envelop영역을 점점 키워나가면서 도로에 snap되는 점들을 찾아온다.
	- envelop 영역을 키워나가는 방법은 $\sigma_z$만큼의 해당하는 위경도 delta를 동서남북으로 확장하면서 만듬
- snap 점들을 찾아오는 방법
	- locationIndex.query()에 현재 envelop을 넣어서 호출하면 해당 bbox(=envelop)안에 있는 모든 edge를 iterate해준다.
	- 각 edge마다 다음의 작업을 수행,
		- edge와 gps 포인트간의 거리 계산
		- 거리계산은 edge를 이루는 base, adj node간의 거리를 먼저 보고 그게 충분히 만족스러우면 그걸로 거리를 갈음.
		- 아니라면 edge의 geometry(base + pillar)를 가져와서 계산
			- gps 관측점의 수선의 발이 edge 위로 떨어지는거 같으면, crossing point를 구하고 그것을 통해 distance 계산. 즉, 점과 edge(선분)사이의 거리 계산
			- gps 관측점의 수선의 발이 edge 위로 안떨어지면, edge의 pillar 노드와의 점과 점 거리를 계산 
			- 이 과정을 edge의 geometry를 돌면서 만족할만한(거리가 epsilon보다 작을때) 것이 나오면 탐색 끝.
- Snap 구조체
	- queryPoint : GPS 관측점
	- queryDistance : GPS 관측점과 snapped point간 직선거리
	- wayIndex: snap point가 edge의 point list내 몇번째 geometry edge에 위치해있는가?
	- closestNode : 가장 가까운 노드 번호 
	- closestEdge: 가장 가까운 edge
	- snappedPoint: 실제 snap된 좌표 
	- snappedPosition: snap된 좌표가 edge 위인지 tower혹은 pillar node인지.



snap들이 준비되면 그걸가지고 query graph를 만드는데, query graph는 기존 base graph의 데이터를 건들지 않고 레이어를 하나 올리는거라고 보면 됨.

여기서의 중요과정은 snap들을 가지고 virtual edge를 만드는 과정임:

- 각 snap들마다 tower노드에 떨어진 snap을 제외하고 edge를 쪼개서 새 virtual node와 새 virtual edge를 만들어줌. 이때 만들어진 새 node의 좌표는 `Snap.snappedPoint`이다. 
- 그리고 이 과정에서 각 snap의 edge를 검사하여 edge의 base node가 adj 보다 크면 edge 표현을 반대로된걸 가져와서 사용한다 (이것이 실제로 엣지의 방향을 바꾸거나 하는건 아니다). 이는 나중에 edge split할때 처리과정을 간단하게 하기위함. (왜 그런진 디테일은 모르겠음)

### emission probability 개선방안 
현재 emission prob은 snap과 observation과의 거리만 사용하고 있음. 
[이 논문](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/map-matching-ACM-GIS-camera-ready.pdf)에도 같은 방법을 썼는데 이 논문에서의 변은 통상 GPS 정보값에 direction이 같이 오지 않으니 좌표값만 썼다는 것.
다만 우리 같은 케이스는 direction도 같이 받아오고 있으므로 해당 값을 사용하여 candidate snap들을 좀 더 걸러낼 수 있을거 같음. 이 아이디어는 [이 논문](https://www.mrt.kit.edu/z/publ/download/hummel2006b.pdf)에서 언급한 것이고, gps 데이터에 써있는 direction과 road의 각도의 차이를 random variable로 세팅하였음. 
- 이 random variable은 Normal distribution을 따른다고 가정하였음
- sigma는 어떻게 구할진 딱히 언급이 없다.

snap을 계산할 때 이미 full point list를 가져와서 crossing point등을 계산하는 부분이 있으므로 해당 부분을 활용해보면 어떨까 함. `Snap`구조체에 direction을 추가하는것. 

각도의 차이는 범위가 0부터 180($\pi$)까지일것. 
그리고 기존의 emission probability 계산 값 (즉, routing ~ direct distance 차이값을 이용한)에 변화를 크게 주지 않기로 하자. (흠) .. 대충 sigma를 $\pi/4$ 로 잡아보자 . 의미는 없다.

근데 mean이 0이려면 범위가 -180부터 180이어야함. 


