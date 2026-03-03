1. real time traffic이 반영이 되나?
	1. 반영이 된다면? - 그게 sane 할까? 테스트시에 좀 빡셀거 같은데
	2.  -> 매번 정해진 결과가 나오진 않아서 테스트가 오히려 힘들수도
2. mapmatch 후 direction을 할건가?
	1. 사용량 보고 결정..
3. 이것도 profile가지고 하면 안되나 
4. 요청하는 쪽에서 route engine 선택이 가능하게끔 세팅 필요
	1. 그러면 grpc -> route -> nb 로 가야하나? 너무 path가 길지 않나?
		1. delay가 어떻게 늘어날지 고민 필요
	2. 근데 사실 그게 관리측면에서는 이로울듯 
5. mapmatch 시에 wayid 같이 줄 수 있나?
	1. toll gate pass time 추정에 필요
	2. 비슷한 논리로 estimate 돌릴때 (즉 product search할때) path detail에 time 껴서 줄 수 있는지 
6. 적용 지역?
	1. grpc controller에서 정해서? 
		1. 호출부에서 책임. ride 마다 다르게 할 수도 있음
		2. gradual한 파급효과를 보려면 이게 낫겠다 
	2. 아니면 내부적으로 config로 정해서?
	