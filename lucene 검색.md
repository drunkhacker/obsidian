
analyzer
custom하게 만들었음:

```kotlin
this.tokenFilter(EdgeNGramFilterFactory::class.java)  
    .param("minGramSize", "$min")  
    .param("maxGramSize", "$max")  
    .param("preserveOriginal", "true")
```

edgengram filter는 뭐하나? = edge ngram을 generate함 (edge는 keyword 처음부터 시작한다는 뜻)
https://solr.apache.org/guide/6_6/filter-descriptions.html#FilterDescriptions-EdgeN-GramFilter

LuceneAnalysisConfigurer를 config할 때 token filter를 여러개 먹일 수 있는데, 앞에서 부터 하나씩 적용된다고 보면 된다.
```kotlin
private fun LuceneAnalysisComponentParametersStep.setEdgeNgram(min: Int = 2, max: Int = 20) =  
    this.tokenFilter(EdgeNGramFilterFactory::class.java)  
        .param("minGramSize", "$min")  
        .param("maxGramSize", "$max")  
        .param("preserveOriginal", "true")
```

preserve original은 ngram의 마지막에 keyword를 그대로 출력할것인지 (즉, max length로 짤리지 않는)를 지정 

ASCIIFoldingFilterFactory = 비 아스키문자를 ascii로 reduce함 
https://solr.apache.org/guide/6_6/filter-descriptions.html#FilterDescriptions-ASCIIFoldingFilter

SnowballPorterFilterFactory =  keyword를 어근으로 바꿔줌. 
>Reduces a word to it’s root in a given language. (eg. protect, protects, protection share the same root). Using such a filter allows searches matching related words.



lucence의 simpleQueryString은 쿼리 스트링을 만들어서 검색을 할 수 있게 한다.
