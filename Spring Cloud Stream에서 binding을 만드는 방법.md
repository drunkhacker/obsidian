1. Function이 bean으로 정의되어있으면 \<function name\>-{in, out}-\<index\>  으로 정의
	1. 그리고 이걸 다시 간단한 이름으로 재정의 할 수 있음 아래 예제 참고
	```
	spring.cloud.stream.function.bindings.uppercase-in-0=input
	```
2. explicit하게 function 없이도 정의 가능
   이렇게 되면 stream bridge 로만 접근하겠다는 의도가 됨.
	2. output-bindings 
	3. input-bindings


[참고자료](https://docs.spring.io/spring-cloud-stream/docs/current/reference/html/spring-cloud-stream.html#_functional_binding_names)
