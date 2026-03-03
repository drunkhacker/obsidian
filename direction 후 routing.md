
A -> B

ViaRouting이 A를 snap한 A' 에서 찾을 것임..
driver position들을 A까지 오게 하고, driver position + A까지 맵매칭하여, 그럼 알아서 snap이 되겠지? 맵매칭한 마지막 점이 snapped point고, 거기부터 B까지 가는걸 찾기.

driver positions + A가 타당하니? --- 
용처가 뭐야?
- inuse나 pickup route일때 driver 위치에서 목적지 (pickup point or dest)까지 걸리는 route 제대로 뽑기
-..


inuse나 pickup일때 routing 뽑기

- driver 위치가 업데이트 되면 firebase에 업데이트 되고, 라이더 앱에 신호가 감.
- 그렇다면 마지막 driver 위치는 왠만하면 dynamodb에 있는것과 같을것.
- 아니면? 
	- 아닌 경우를 대비하여,
	- routing server 입장에서 dynamodb에서 꺼낸 마지막 위치와,
	- 요청 들어온 위치간의 gap을 참고하여, 
	- gap이 작으면 (혹은 같으면) 그것까지 넣어서 map matching
	- 아니면 그냥 요청들어온 포인트부터 바로 routing. (no mapmatching, no direction.)
		- 실패케이스. 수집 필요
