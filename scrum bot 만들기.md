## 요건

### 스크럼과 질문
스크럼은 질문세트를 지정할 수 있다.
질문 세트는 여러개의 질문을 포함한다.
질문에는 답을 할 수 있다. 
답은 여러 형태 중 선택 가능하다.
- 일반 response
	- 일반 response에는 사진을 붙일수도 있다.
- single choice
- multiple choice
- 그냥 뜨는 메세지

### 스크럼과 참여자
스크럼에 참여할 사람들을 지정할 수 있다.

### 스크럼과 시간
스크럼을 할 요일을 지정할 수 있다.
스크럼을 안하는 특수한 날을 지정할 수 있다.
휴일인 사람은 스크럼을 안할 수 있다.
스크럼은 시작시각과 끝내는 시각이 있다.

### 스크럼과 통계 
스크럼이 끝나면 참여율을 표시한다.
스크럼이 끝나면 안한 사람을 알 수 있다.

### 특수기능
(MVL) 스크럼 참여자는 아직 안한 사람을 찾아낼 수 있다.

### 전달매체
- Slack
- ... 나중에 더 많이


## 모델링

Team
- (has many) Member
- (has a) Platform

Scrum
- Schedule
- (has a) Delivery
- (has many) Member
- (has a) Questionnaire
- (has a) Result
- (belongs to) Team

Delivery
- method?
- channel?
- ...

Result
- (belongs to) Scrum
- (has many) Response
- (has a) Summary

Response
- (belongs to) Question
- (belongs to) Answer
- (belongs to) User

Answer
- (belongs to) User
- (belongs to) QuestionLog
- (belongs to) 

Questionnaire
- (has many) Question

Question
- type


Schedule
- [modeling example](https://chatgpt.com/c/67926f30-9a84-800d-b8cb-bb541061ee67)
