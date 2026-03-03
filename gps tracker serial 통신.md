https://www.scribd.com/document/483909326/Concox-GPS-Trackers-Configuration-Tool-Instruction-Mac
AT^GT_CM=SERVER,0,120.24.248.12,3455,0# 이런거 쓰면 SMS 명령어를 보낼 수 있는듯.

SSCOM v5 이용하니까 됨. windows 필요.
https://github.com/m2mlorawan/sscom5

port는 OPEN_CON 이용.![[IMG_4564.jpg]]
![[IMG_4564.jpg]]


기본적으로 sms command로만 되던거를 OPEN_CON 포트로 AT^GT_CM=xxxx 형식으로 보낼 수 있음.
하지만 일반 AT command 같은 경우는 잘 되지 않음..(ex: AT+CNUM)

driver 필요.
[러시아 버젼 웹 페이지 링크](https://wless.ru/technology/?action=details&id=790&pf=tech&pf_id=1&prod=35&tech=2&type=167)

----
electron 으로 gui 만들어서 하자.
[serial 통신 참고 자료](https://it-jm.tistory.com/28)



가상 serial 포트 만들기
https://reactivemusic.net/?p=20622 - 근데 안된다는듯?
https://apple.stackexchange.com/a/50403 이런 얘기도 있다.
socat으로 해보기. 이게 되는듯하다 : [[osx에서 가상 serial port 만들기]] 


https://serialport.io/docs/api-parser-readline 로 해보자.
간단 테스트:
```Javascript
const {SerialPort} = require('serialport');  
const {ReadlineParser} = require('@serialport/parser-readline');  
  
  
// SerialPort.list().then(r => {  
//         r.forEach(port => {  
//             console.log(port.path)  
//             console.log(port.serialNumber)  
//             console.log("")  
//         })  
//     }  
// )  
  
const serialPort = new SerialPort({path: '/dev/ttys023', baudRate: 19200});  
  
const parser = serialPort.pipe(new ReadlineParser());  
parser.on('data', console.log);  
serialPort.write('Hello there this is serial port test');
```
반대편에는 echo 로 쏜다.


[AT Commander](https://www.npmjs.com/package/at-commander ) 라는 패키지가 어느정도 래핑을 잘 해놓은듯하여 이걸 쓰면 좀 편리할듯.

프로그램이 해야할 것들:
기기 세팅
- APN 셋업 (AT^GT_CM=APN,XXX#)
	- GPRSSET 이랑 같이 보자
- 전화번호 확인 (AT+CNUM)
	- 심카드가 있는지부터 확인해야할듯
- 접속 server 주소 세팅 (GTCM)
	- AT^GT_CM=SERVER,0,120.24.248.12,3455,0#
- power report 세팅 (GT_CM)
	- "POW_REPORT,ON#" 
- ADC offset 세팅 (GT_CM)
	- ADC OFFSET,60#
	- ADC OFFSET#
- 비밀번호 세팅
	tracker password setup
	![[Pasted image 20230808170744.png]]



서버 통신
- tracker 등록
- 로그인 잘되는지 확인
- power report 들어오는지 확인
- gps (또는 LBS라도..) 들어오는지 확인
