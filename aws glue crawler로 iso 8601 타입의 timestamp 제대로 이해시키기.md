
crawler 로 일단 s3 지정해서 테이블 만듬
테이블의 json serde 수정 
그리고 나서 crawler 돌려보기
테이블의 column def 수정 
근데 crawler가 table def를 update가 있어도 못건들게 해야함. 
![[Pasted image 20231115190955.png]]

참고자료
https://dasoldasol.github.io/aws/aws-glue-dt/
https://stackoverflow.com/a/63398790
"Enable Update all new and existing partitions with metadata from the table"
