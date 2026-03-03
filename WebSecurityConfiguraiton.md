springSecurityFilterChain 이라는 메소드에서 filter chain bean들을 가지고 filter를 만들어준다.

각 filter chain은 endpoint matcher와 filter로 구성

endpoint matcher는 아마 security matcher와 requestMatcher로 구성되는거 같은데 차이점이 ..뭐지?
https://github.com/spring-projects/spring-security/issues/12950

[동영상 설명](https://www.youtube.com/watch?v=PczgM2L3w60)을 보면 security matcher가 matching pattern을 제한해서 securityFilterChain 을 여러개 정의해놓고 쓸때 다른 패스에 대해 서로 다른 security filter chain을 적용할 수 있게 한다고 돼있음.