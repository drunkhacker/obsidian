추가적인 role이 더 필요하다.
sqs에 access policy를 추가해주자.
```json
{
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::999637940583:role/service-role/AWSGlueServiceRole-OnionLakeFormationCralwer-prod"
      },
      "Action": "SQS:*",
      "Resource": "*"
    }
```

참고자료
https://aws.amazon.com/ko/blogs/big-data/run-aws-glue-crawlers-using-amazon-s3-event-notifications/
https://github.com/aws-samples/aws-glue-crawler-utilities/blob/main/blog_resources/s3_event_notifications_sqs_queue_access_policy.json


