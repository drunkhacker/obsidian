```
1. 평균 주행거리 (주 사용 시간대)
2. 충전 시점 (충전 할 때의 SOC, 언제 충전하는지 - 아침에 또는 밤에)

Friday, April 5th
```

- 히스토그램을 쓰자.
- 분석기간은 3월 한달로 하자.

## 항목별 viz 고민
### 일 평균 주행거리
- 히스토그램 분포로 3월 한달 각 기기의 주행거리의 평균을 보여준다

### 주 사용 시간대
- scatter plot
- x축은 24시간 (캄보디아 기준시각)
- y축은 시간대별 주행거리 

### 충전 시점 + 시간
- scatter plot
- x축은 24시간  (충전 시각, 캄보디아 기준)
- y축은 충전 시간 
- 일단 개별 데이터 포인트들을 다 뿌려보자.

### 속도?


## 데이터 프로세싱
- 뽑을 데이터들
	- 일평균 주행거리
		- 데이터 형태
			- id, date, odometer
	- 시간대별 주행거리
		- id, date, hour, odo
	- 충전 시작시점 및 시간
		- id, start soc, start time, end soc, end time
	- 충전중임을 어떻게 파악?
		- 연속된  soc의 상승 - 데이터포인트 찍어봐야알겠는데?
		- 하트비트 3분마다, 5개 연속(15분) soc 상승 추세가 보이면
		- 상승추세?
			- monotonic increase (with error)
			- error = soc 2 미만 
		- 
- 주의사항
	-  충전 시점의 경우 하루가 넘어가는 경우가 있기 때문에 연결에 대한 고민.
	- how?
		- 같은 id의 두개의 연속된 충전 span prev, next의 
		- 시간간격(next.start - prev.end)이 epsilon 이하이며 &&
		- soc 차이가 (next.start_soc - prev.end_soc) epsilon이하.
	- 하루 단위 데이터 처리할 때 하루 시각 끝에 충전 timespan이 있으면 일단 분리.

- ### table 설계
- x_series_daily_odometer
- x_series_hourly_odometer
- x_series_charge

