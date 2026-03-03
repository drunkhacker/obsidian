- Lidar 센서는 빛 반사를 이용하여 x,y,z,R(신호세기)를 측정한다.  
- 차량에 장착되는 라이다는 360도 회전하고 (x,y 커버), 16,32,64,128개의 vertical 한 레이저 배열을 이용하여 (channel) z 축을 커버한다.
- lidar를 이용하여 위치 추정을 하려면 IMU + GPS + lidar 값 세개가 필요함.
	- IMU는 inertial measurement unit.  roll, pitch, yaw, 차량의 움직임을 측정하여 lidar 정렬을 보정
	- GPS는 전역 좌표계로 정렬하기 위해서 필요

- lidar 데이터를 가지고 HDMAP을 그리려면 데이터 보정 작업이 들어가야함 
	- SLAM이 대표적
		- SLAM 알고리즘 상 중요한 부분이 왔던 곳을 또 왔을때 인식하고 그걸 가지고 기존의 측정치들을 보정하는 과정
- 서로다른 lidar 센서를 썼을 때 보정?

- 필요한 보정정보

| 항목                                         | 설명                                             | 예시                                                               |
| ------------------------------------------ | ---------------------------------------------- | ---------------------------------------------------------------- |
| **1. 센서 Extrinsic Calibration**            | 라이다 센서의 위치와 방향 (센서 → 차량 기준 좌표계 변환)             | `T_sensor_to_vehicle`예: x = 0.2m, y = 0.0m, z = 1.8m / yaw = 90° |
| **2. 센서 Intrinsic Calibration**            | 라이다 내부 각 채널의 발사 각도 및 보정값 (레이저별 tilt, 시간 오프셋 등) | 대부분 제조사 제공예: channel 1 = -15°, channel 64 = +2°                  |
| **3. 차량 Extrinsic Calibration (optional)** | 차량 좌표계와 월드 좌표계의 관계 (SLAM이나 GPS로 추정됨)           | `T_vehicle_to_world`                                             |
| **4. 타이밍 정보 (Timestamping)**               | 각 포인트의 수집 시각 정보, 동기화 정확도                       | UTC timestamp or ROS time예: GPS와 LiDAR 시간 동기 여부                  |
| **5. IMU / GPS 데이터**                       | 차량의 위치와 자세 (pose), 보정용                         | 위치 정합 시 필수: `x, y, z, roll, pitch, yaw`                          |
| **6. 기온/습도/노이즈 정보**                        | 센서 외부 조건이 반사값, 거리 측정에 영향을 줌                    | 고급 센서에서는 자동 보정                                                   |
| **7. 포인트 클라우드 형식 명세**                      | `.pcd`, `.las`, `.bin`, `.npy` 등 구조 명세         | 필드: x, y, z, intensity, ring, timestamp 등                        |
| **8. 캘리브레이션 타겟/결과**                        | 실측 또는 시뮬레이션 기반 정합 결과                           | 보정 매트릭스, 잔차(RMSE), 오차 시각화                                        |

statement) 서로 다른 lidar를 가지고는 자율주행을 못한다?

- HD Map을 만들 때 쓴 lidar vs 실제 주행에 센서값을 넣어주는 lidar가 있음. 
- 근데 어쨌든 map을 만들때 SLAM등을 통해 world 좌표계 기준으로 넣잖아
- 그러고나면 HDMap이 있다고 치면, 그걸 가지고 다른 lidar가 쓰는거야뭐 다시 좌표변환이니 가능하지 않나?
- 하지만 HDMap이 제대로 안돼있을거고, 주행에 사용하는 lidar가 역시나 world 좌표계로 매핑될때 제대로 안돼있을 가능성이 큼.
- 만약 완벽히 동일한 셋업을 가지고, train + 주행을 한다면 HDMap을 굳이 worldmap으로 변환할 이유가 없음. 동일 좌표계 처리가 가능할듯.
- and 차고가 다르다면 거리 계산에 distortion이 생기지 않을까? (근데 그게 그냥 affine 변환 아닌가?)

SLAM 알고리즘 구현체[Cartographer](https://github.com/cartographer-project/cartographer)
인풋: IMU, GPS, lidar

자율주행의 두가지 분파(?)
- Lidar + HDMap 조합 (Waymo)
	- HDMap과 센서를 통해 인지, 계획, 실행함. 
		- 정밀 센서를 통해 위치를 인식하여 지물이 뭐가 있고 등등을 파악하기 때문에 HDMap의 역할이 매우 중요하다.
	- 근데 HDMap 유지보수가 상당히 비쌈..
- 카메라 + 인지기능 조합 (Tesla)
	- 사람이 하는 운전은 시각과 판단에만 의존하니 카메라와 AI엔진으로만 한다.

HDMap은 어디에 쓰이나?
- Waymo 방식의 자율주행에서 거의 모든 phase에 쓰임.
- HDMap은 자율주행에서 crucial한 역할.
- route planning에도 쓰이고, 현재 surrounding에 대한 perception 및 object classification에 사용.
- route planning: trivial
- perception: lidar 센서 값과 장애물에 대한 모습등을 파악하여 현재 내가 world 상에 정확히 어딨는지 판단
- object classification: 신호등이 어딨는지, 지금 lidar가 보는 장애물이 무엇인지를 이미 HDMap에 있는 annotated된 정보의 도움으로 빠르게 인식


 

참고: 
- [The Basics of LiDAR - Light Detection and Ranging - Remote Sensing](https://www.neonscience.org/resources/learning-hub/tutorials/lidar-basics)
- [Lidar Data Processing for Autonomous Systems](https://www.youtube.com/watch?v=30BAocUSrJc)
- [Understanding SLAM Using Pose Graph Optimization | Autonomous Navigation, Part 3](https://www.youtube.com/watch?v=saVZtgPyyJQ)
- [3D lidar SLAM: The Basics](https://www.kudan.io/blog/3d-lidar-slam-the-basics/)
- [Apollo x Udemy - HDMaps](https://developer.apollo.auto/devcenter/coursevideo.html?target=2_26)
