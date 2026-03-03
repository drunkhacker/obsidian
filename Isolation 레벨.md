Dirty read
- 동시에 TX들이 들어올때 다른 TX가 커밋도 하지 않은 것을 읽을 수 있는 현상

Phantom read
- Tx1 이 range query를 수행중에 tx2가 row를 insert 혹은 지워서 tx1이 똑같은 range query를 수행했을때 tx2가 만들거나 삭제한 row가 나타나거나 사라지는 현상

Non-repeatable read
- tx1이 query를 수행중에 tx2가 특정 row를 변경했을때 tx1이 다시 query 수행하면 똑같은 값이 나오지 않게 되는 현상


참고: https://jennyttt.medium.com/dirty-read-non-repeatable-read-and-phantom-read-bd75dd69d03a

isolation level들
- postgresql은 default가 **Read Committed** 

Read uncommitted : 
- dirty read, phantom, non-repeatable 다 발생

Read Committed:
- dirty read는 발생 방지
- phantom, non-repeatable은 발생 할 수 있음

Repeatable read:
- dirty, non-repeatble read 발생 방지
- phantom read는 발생 가능
	- 특정 row를 고치는것에 대한 보호는 되는데 , row가 추가, 삭제되는 것에 대한 보호는 안됨
	- 아마 row lock을 쓰나?

Serializable
- concurrent한 tx를 허용하지 않음