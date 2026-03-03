파티션을 MSCK repair table로 로딩하면 너무 오래걸린다. [자세한 설명](https://athena.guide/articles/msck-repair-table/)

보통 partition을 가져올때 Athena가 GetPartition이라는 명령을 내부적으로 때리는데 partition이 너무 많을 경우 문제가 된다.

https://docs.aws.amazon.com/athena/latest/ug/partition-projection.html
