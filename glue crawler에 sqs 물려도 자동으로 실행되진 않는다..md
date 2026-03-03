chatgpt가 틀림.
crawler에 s3이벤트를 꽂아주는 sqs를 물려놓아도 sqs에는 메시지가 들어간다해도 자동으로 crawler를 trigger하지는 않는다.

참고자료:
https://aws.amazon.com/ko/about-aws/whats-new/2021/10/aws-glue-crawlers-amazon-s3-notifications/
```
크롤러를 실행할 때마다 SQS 대기열에 새 이벤트가 있는지 검사하고, 찾은 결과가 없으면 크롤러가 중지됩니다.
```

https://devnote.tech/2023/01/having-glue-crawler-crawl-based-on-events-with-sqs/
```
This new feature does not trigger Glue crawlers crawling process.
```

따라서 crawler를 그냥 주기적으로 돌리면서 sqs에 별 이벤트가 없으면 안하는 식으로 넘어가야함.

[[glue crawler가 sqs에 접근하려면]] 세팅은 여전히 필요한듯
