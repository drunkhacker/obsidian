reflect 비슷하게 쓰는것 같음.
`val gqlInputObj = objectMapper.convertValue(gqlInput, Map::class.java)`

이런식으로 convertValue를 활용하면 된다.
