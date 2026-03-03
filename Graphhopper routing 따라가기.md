
graphhopper.route()
Router.route(GHRequest)
Router.routeVia(GHRequest, solver)
여기서 GHRequest가 해체되는데, 좀 더 살펴볼 필요가 있음
- snap 가져옴
- edgefilter만듬 (solver를 통해서. 근데 solver도 만들때 ghrequest참고해서 만듬)
- solver만듬 (CH disable을 가정해보면)
	- createFlexSolver()
	- FlexSolver생성.
- solver로 부터 PathCalculator만듬 ---- (1)
	- createPathCalculator()
	- algorithmFactory = RoutingAlgorithmFactorySimple 
	- 
- ViaRouting.calcpaths 호출
- 결론적으로 GHRequest가 여기서 해체되고 profile등은 위에 만든 다른 인스턴스로 숨어들어갔을거라 보는게 맞음.
```java
ViaRouting.calcPaths(
List<GHPoint> points, QueryGraph queryGraph, List<Snap> snaps, DirectedEdgeFilter directedEdgeFilter, PathCalculator pathCalculator, List<String> curbsides, boolean forceCurbsides, List<Double> headings, boolean passThrough
)
```
내부에서 for loop 돌면서, 
- edgerestriction생성
- pathCaculator.calcPaths(fromSnap, toSnap)

pathCalculator.calcPaths
pathCalculator는 Flex가 있고 CH가 있음. (Flex를 보자)

FlexiblePathCalculator는 애초에 어디서 생성되냐? - 위의 (1) 부분

FlexiblePathCalculator.calcPaths(from, to, edgeRestrictions)
- unfavor edge 설정 
- createAlgo()호출 후 
	- algoFactory는 위에 있는 RoutingAlgorithmFactorySimple 라고 보면 됨.
	- RoutingAlgorithmFactorySimple.createAlgo()를 보면 
		- algoStr이 비어있으면 기본으로 ASTAR_BI.
		- 
- algo.calcPaths 호출

createAlgo()
- algoFactory로부터 가져옴...

결국엔 AStarBidirection.calcPaths() 호출
AStarBidirection는 
- EdgeToEdgeRoutingAlgorithm interface을 구현 
- AbstractBidirAlgo 를 상속

AbstractBidirAlgo의 calcPath는 runAlgo() 호출
runAlgo()는 bidir를 굴림. 
- fillEdgesFrom() - fillEdges() 호출
- fillEdgesTo() - fillEdges() 호출

AbstractNonCHBidirAlgo.fillEdges(currEdge)
- iter = edgeExplorer.setBaseNode(currEdge.adjNode) 하면서 다음 edge 탐색 
- iter 돌면서
	- calcWeight()
		- GHUtility.calcWeightWithTurnWeight(weighting)

결국엔 특정 profile에서 길을 가고 못가고를 판단하는것은
- edgeExplorer가 가능한 edge를 뽑아주느냐
- calcWeight()가 가능한 weight값을 주느냐

- edgeExplorer는 Algo생성자에서 graph.createEdgeExplorer 로 생성
	- 흠.. 여기는 일단 특별할 구석이 없어보임 (profile때문에 달라질 것이 없단 뜻)
- weighting 역시 생성자에서 주입
- weighting은 결국에 FlexiblePathCalculator() 생성자에서 들어오고
- 그 위에 Router.createWeighting()에서 만들어지고, 
- createWeighting()은
	- weightingFactory.createWeighting() 호출
- weightingFactory는 사실
	- graphhopper.createWeightingFactory()
		- DefaultWeightingFactory()만듬


DefaultWeightingFactory.createWeighting()
- weightingStr == custom이면 (현재 TADA가 custom 씀)
- mergedCustomModel이라는 것을 주어진 profile.custom model과 request.custommodel을 합쳐서 만듬. <- Request마다 조금씩 다르게 뭔갈 할 수 있는 장치인듯하다.
- CustomModelParser.createWeighting(encodingManager, turnCostProvider, mergedCustomModel)

CustomModelParser
CustomWeighting

CustomWeighting.calcEdgeWeight()
- 이건 model parameter에 따라 weighting을 계산하는 함수
- 이중 핵심은 POSITIVE_INFINITY가 나오는 경우를 찾는것:
	- edgeToPriorityMapping이 prio = 0이면 
	- edgeToSpeedMapping이 speed=0이면
	- edge.distance * distanceinfluence가 inf면
- 세번째는 당연한 소리인거같고, 1,2가 문젠데..
- 결국엔 edgeToPriorityMapping, edgeToSpeedMapping은 모두 model parser가 준비해서 가져다줌


CustomModelParser.createWeightingParameters()
CustomWeightingHelper를 캐시에서 가져옴 (혹은 새로 만듬)

custom은 딱히 edge의 차량 통행 가능성에 관한것들을 보는거 같진 않음
.. 그럼 edge explorer인가?



