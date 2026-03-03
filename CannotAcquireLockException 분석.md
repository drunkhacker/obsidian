[[Isolation 레벨]] 참고하자.

onion에서 `updateVehicleStatusByOnionData` 부르다가 accquire lock 에러가 뜸.
해당 메소드는 repeatable read tx, timeout =5 sec으로 돼있음.
결국에는 FindVehicleUseCase 를 부르는데, 이게 걍 vin으로 찾는거.

에러메시지는 
```
##### PSQLException

ERROR: could not serialize access due to concurrent update
```

update 하려는 놈: update vehicle by data . odometer나 bat level 등등을 변경함.
근데 해당 uc 안에서 checkNoBatterySwapAfterDistanceTraveledUseCase를 부르는데 얘는 coroutinescope으로 다른 쓰레드 (Dispatchers.io)에서 돔..
-> 틀린 말인게, runblocking 이전의 콜들은 호출하는 thread를 그대로 타고 감..

checkNoBatterySwapAfterDistanceTraveledUseCase 안에서 findbyvin을 할때 non-repeatable read 방지 세팅 들고 들어가니까, update vehicle by data가 변경하는게 방지되어야할듯.

그러려면 둘중 하나는 터져야하는거 아닌가?

실험해볼것: 어차피 checkUsecase가 실제로 vehicle을 변경할 일은 없으니 로드된 vehicle entity를 넣어주는게 어떨까.

