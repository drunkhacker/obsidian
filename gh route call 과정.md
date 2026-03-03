/Users/drh/.gradle/caches/modules-2/files-2.1/com.graphhopper/graphhopper-core/9.1/7e5e262f94cf8e9ed77648e46261f832b75dff09/graphhopper-core-9.1-sources.jar!/com/graphhopper/routing/Router.java

`protected GHResponse routeVia(GHRequest request, Solver solver) `

```java
List<Snap> snaps = ViaRouting.lookup(encodingManager, request.getPoints(), solver.createSnapFilter(), locationIndex,  
        request.getSnapPreventions(), request.getPointHints(), directedEdgeFilter, request.getHeadings());
```

snaps는 points를 edge위로 snap한것. 
snap은 closest node, closest edge 가 있음. 

snaps 만든 이후에 `ViaRouting.calcPaths` 호출 

```java
public static Result calcPaths(List<GHPoint> points, QueryGraph queryGraph, List<Snap> snaps, DirectedEdgeFilter directedEdgeFilter, PathCalculator pathCalculator, List<String> curbsides, boolean forceCurbsides, List<Double> headings, boolean passThrough)
```

incomingedge = NO_EDGE

buildEdgeRestrictions() 호출 

```java
private static EdgeRestrictions buildEdgeRestrictions(  
        QueryGraph queryGraph, Snap fromSnap, Snap toSnap,  
        double fromHeading, double toHeading, int incomingEdge, boolean passThrough,  
        String fromCurbside, String toCurbside, DirectedEdgeFilter edgeFilter) {  
    EdgeRestrictions edgeRestrictions = new EdgeRestrictions();
```

curbside 및 heading 처리

heading의 경우에는 headingResolver를 만들어서 fromSnap, toSnap에 해당하는 heading과 가깝지 **않은** edge를 찾아 edgeRestrictions의 unfavoredEdges에 등록함. 
가깝지 않은 = snap의 closest node에 연결된 edge들 중 edge의 각도가 원하는 heading과 100°이상 차이나는 edge

그 후에 sourceOut, targetIn edge 세팅 
근데, forceCurbsides가 false이면 sourceOut, targetIn은 둘 다 ANY_EDGE


그리고 난 후 ViaRouting#calcPaths에서 `pathCalculator.calcPaths()`호출 

```java
PathCalculator pathCalculator = solver.createPathCalculator(queryGraph);
```
이고, solver는 
Router.java#route() 함수에서 createSolver로 만들어짐.
```java
public GHResponse route(GHRequest request) {
...
            Solver solver = createSolver(request);

...
```

createsolver는 CH, LM enabled flag들을 가지고 CH혹은 LM혹은 flexsolver를 만듬. 
우리 경우는 아마 flex.

`FlexSolver`는 Router.java에 있음..

Flexsolver의 `getAlgoOpts()`를 보면,
- profile에 turncost가 있는경우는 EDGE_BASED, 아니면 NODE_BASED로 traverse하게 돼있음
- roundtrip의 경우에는 A* 알고리즘을 쓰게 돼있음 
	- 아닌 경우는 request(GHRequest)의 getAlgorithm() 호출하여 algorithm으로 세팅함


FlexSolver가 만들어주는 path calculator는 `FlexiblePathCalculator`

FlexiblePathCalculator.java#calcPaths()

기본알고리즘이 뭐지!?!? = A* bi
RoutingAlgorithmFactorySimple 을 보면 createAlgo()함수에서 AlgorithmOptions에 전달한 option string이 비어있으면 그냥 astar bi 를 쓴다고 돼있음.

참고: virtual edge

In GraphHopper, a virtual edge refers to an edge that is dynamically created during the routing process to handle situations where the start or end points of a route (or waypoints in between) do not exactly match the existing nodes in the road network graph. Virtual edges are temporary constructs used to connect the exact GPS location (e.g., the start, end, or via point of a route) to the nearest node or edge in the graph.

```java
private List<Path> calcPaths(int from, int to, EdgeRestrictions edgeRestrictions, RoutingAlgorithm algo) 
```

- edgeRestriction에 있던 unfavoredEdges를 queryGraph에 등록해줌.
	- 어떻게 쓰이는지는 모르겠다
	- queryGraph에 등록해주면 이게 다시 `edge`의 unfavored flag를 켬
- curbside가 force된게 아니라면, algo.calcPaths(from, to) 만 부름.
	- force됐으면 in, out edge도 같이 넣어서 다른 함수를 부름
	- 근데 결국엔 다 같은거로 옴..
- runAlgo() 함수 호출
	- AbstractBidirAlgo.java
	- AStarBidirection.java 
	- AStarBidirection.java


AbstractBidirAlgo#runAlgo()
```java
protected void runAlgo() {  
    while (!finished() && !isMaxVisitedNodesExceeded() && !isTimeoutExceeded()) {  
        if (!finishedFrom)  
            finishedFrom = !fillEdgesFrom();  
  
        if (!finishedTo)  
            finishedTo = !fillEdgesTo();  
    }  
}
```

AbstractNonCHBidirAlgo#fillEdgesFrom() -> fillEdges()

```java
    private void fillEdges(SPTEntry currEdge, PriorityQueue<SPTEntry> prioQueue, IntObjectMap<SPTEntry> bestWeightMap, boolean reverse) {
        EdgeIterator iter = edgeExplorer.setBaseNode(currEdge.adjNode);

```


FlexiblePathCalculator가 routing algorithm instance를 생성할 때
```java 
RoutingAlgorithm algo = algoFactory.createAlgo(queryGraph, weighting, algoOpts);
```
로 하는데 createAlgo는 첫번째 인자로 Graph를 받고, queryGraph는 사실 graph의 interface를 구현함.


참고: queryGraph와 graph의 차이
QueryGraph는 Graph를 단순하게 wrapping한거고, virtualedge와 virtualnode를 세팅하려고 만든 클래스. 주석에는 `A class which is used to query the underlying graph with real GPS points
라고 돼있음.

queryGraph는 언제 만들어지지? - Router#routeVia()에서         
```java
QueryGraph queryGraph = QueryGraph.create(graph, snaps);
```
QueryGraph.create()는 내부에서 snaps에 있는 점/edge들을 virtual로 등록해주는것 같다.


unfavor edge의 쓰임:
CustomWeighting에서 calcEdgeWeight()에서 쓰임.
edge가 unfavor면 headingPenaltySecond를 더해주는 것으로 동작함.
CustomWeighting#calcEdgeWeight()
```java 
    public double calcEdgeWeight(EdgeIteratorState edgeState, boolean reverse) {
        double priority = edgeToPriorityMapping.get(edgeState, reverse);
        if (priority == 0) return Double.POSITIVE_INFINITY;

        final double distance = edgeState.getDistance();
        double seconds = calcSeconds(distance, edgeState, reverse);
        if (Double.isInfinite(seconds)) return Double.POSITIVE_INFINITY;
        // add penalty at start/stop/via points
        if (edgeState.get(EdgeIteratorState.UNFAVORED_EDGE)) seconds += headingPenaltySeconds;
        double distanceCosts = distance * distanceInfluence;
        if (Double.isInfinite(distanceCosts)) return Double.POSITIVE_INFINITY;
        return seconds / priority + distanceCosts;
    }

```



테스트 :

from: 1.313966172801256, 103.84007955767403
to: 1.3728612637245141, 103.8937503630811
from heading: 45

snaps:
EDGE, 3490377 62:339420-951050 snap: [1.314019, 103.840033], query: [1.313966,103.84008]
EDGE, 3490378 1039807:560252-2260094 snap: [1.372827, 103.893749], query: [1.372861,103.89375]

...
팁:
```java
snaps.get(1).closestEdge.get(encodingManager.getIntEncodedValue("osm_way_id"))
```
osm way id 추출할 수 있음.


시작점에서 "제일" 가까운 snap을 찾고 그 snapped point에서 뻗어나가는 edge들 중 heading 방향이 다른 애들은 unfavor로 하고 나중에 그걸 turn cost를 더해서 weight를 계산한다.
탐색에서 제외하진 않음.


