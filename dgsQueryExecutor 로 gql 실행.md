gql 요청 스트링
```
mutation createChargerTransfer($input: CreateChargerTransferInput!) {
	createChargerTransfer(input: $input) {
		// 실행 결과에 대해서 꺼내올 projection들
		id
		...
	}
}


```

```
dgsQueryExecutor.execute(gqlQuery, mapOf("input" to gqlInputMap))
```

gqlInputMap은 `mapOf<String, Any>` type. 변환은 [[jackson으로 일반 클래스 map으로 변환하기]]을 참고하기.

이렇게 하면 결과로, 
![[Pasted image 20230726121801.png]]
이런게 나와서, jsonPath로 data.createChargerTransfer 를 참고하면 바로 결과 오브젝트에 접근할 수 있음.

