
[LW Locks in PostgreSQL](https://chat.openai.com/share/0c2228bb-85a0-4be0-a22b-be67bcdc04ab)
- postgresql의 lw lock 이란 무엇인가 

[index creation status](https://chat.openai.com/share/aaae1daa-0c99-4fb8-9867-c9436395aad5)
- reindex status및 create index 의 status 조회에 대한 질답
- index bloat에 관한 질답 
- `pgstattuple` 에서 보이는 `leaf_fragmentation` 해석에 관한 질답

[what is wait event in postgres?](https://chat.openai.com/share/401c45e9-b4d4-4290-9140-99a27f703df3)
- wait event의 의미와 그 종류에 대한 질답
- client read wait의 의미
	- client read wait를 일으키는 쿼리에 대한 조사
- `pg_locks`로부터 table locking 횟수 얻기 
- `pg_toast`가 포함하는 정보들

[how mysql implement mvcc?](https://chat.openai.com/share/3a4d5a05-7bd0-4bbf-8a42-447c9571625c)
- mysql의 mvcc 구현 방향에 대한 질문
- postgresql 와 mysql의 delete operation 차이

[MySQL vs PostgreSQL Bloat](https://chat.openai.com/share/31bfb360-20ab-4df0-923c-e001cc01a9d9)
- mysql에 bloat 비슷한게 있는지 물어보는 질문
- table frag 및 공간활용에 대한 innodb 의 potential한 문제점들
- `OPTIMIZE TABLE`명령어에 대한 동시실행성 여부

[Check Vacuum Status](https://chat.openai.com/share/c7de68c4-6d5f-4b47-81a6-c6255f0d398f)
- vacuum 및 reindex 상태 조회에 대한 쿼리 질문

[query auto vacuum + auto analyze threshold](https://chat.openai.com/share/353963e6-39a0-4e93-a3f7-b365c20a69e3)
- auto analyze및 auto vacuum threshold 조회 및 설정에 관한 질답

[Autovacuum During Publication-Subscription](https://chat.openai.com/share/9d831c43-b2f5-4113-beab-5b349f9735bb)
- postgresql에서 pub-sub 이 있을때 autovacuum 동작에 관한 질답

[Querying Current XID in PostgreSQL](https://chat.openai.com/share/2d6e39b7-aea8-4c63-ab9a-3ebbf4fb8dba)
- current txid 조회에 대한 질답 
- txid wraparound 가 발생했는지 조회하는 방법 (근데 답변이 이상함)

[how do i query vacuum_Freeze_min_age in postgresql?](https://chat.openai.com/share/8c4b18c2-ba17-4c71-8472-f21b572e6ef8)
- `vacuum_freeze_min_age` 세팅 조회 방법
- table의 age 검사
- remaining txid 체크하는 방법
- index에도 age가 있는지? 

[Same PID for Transactions](https://chat.openai.com/share/65fdb810-7ea8-475c-a6e9-5d3dd7bd75ce)
- 같은 tx안에 있는 query들이 같은 pid를 갖는가

[Unindexed Foreign Key Locking](https://chat.openai.com/share/43176db5-a389-4d66-81d0-03abf7407907)
- foreign key에 index가 걸려있을때의 락 동작에 대한 질문
- foreign key column이 index가 안걸려있을때 table이 lock되는 예제 
	- 관련하여 파티션 돼있을때 파티션들이 모두 락이 걸리는지에 대한 질문

[AccessShareLock in PostgreSQL](https://chat.openai.com/share/f6c610a1-8c8c-4b8f-9ecf-f714e3b9db3f)
- postgresql에서 access share lock의 의미
- parition된 테이블에서 특정 컬럼의 값으로 조회할 시 partiton 전체에 락이 걸리는지에 대한 질문
	- 쿼리 플랜에 따라 달렸다..

[Index Entries in PostgreSQL](https://chat.openai.com/share/18f4321b-5276-4bbd-b981-26f0365c5956)
- 'index entry' 가 무얼 의미하는지?
- index type과, index cardinality를 조회하는 법 
- index의 index entry 갯수를 조회하는 법
	- 근데 direct한 방법은 없음
- index entry는 어떻게 구성돼있는가?
- low cardinality 인덱스에 대한 검색 시 부하

[Foreign Key Index Impact](https://chat.openai.com/share/a376e4ba-eb5a-473f-8480-3d1b7844b5e2)
- foreign key에 index가 안걸려있을때의 potential한 퍼포먼스 저하이슈들

[Covering Index in Databases](https://chat.openai.com/share/b482a240-e375-43dd-9e9f-3b84e9eb8f97)
- covering index의 의미

[Find queries using index.](https://chat.openai.com/share/a13c055a-a9e5-4f7a-92be-768d8fc228c0)
- 특정 인덱스를 사용하는 쿼리들을 찾을수있는지?
	- 딱히 없음. plan을 보시요.
- `pg_stat_statements`에서 슬로우 쿼리 가져오는 법
	- stat reset하는 법
	- 특정 db에 대한 쿼리들만 가져오는 법

[Resetting pg_stat_statements in PostgreSQL](https://chat.openai.com/share/22a8e6a0-5f26-4a1a-b166-d8599f4be6b4)
- `pg_stat_statements_reset` 함수를 사용했을때의 side effect가 없는지?

[PostgreSQL Shared Blocks Explanation](https://chat.openai.com/share/6ce40de5-997e-4987-bcac-a9df81f51187)
- shared block이란 무엇인가?
- `pg_stat_statements` view에서 shared block hit count의 의미
- postgresql 의 query plan cache 존재 여부
- explain과 explain analyze의 차이
- 쿼리가 특정인덱스만 타도록 할수 없는가?
	- 기본적으로는 없음.
	- **`pg_hint_plan`** 따위를 쓰면 가능 
- plan explain에서 cost와 width의 의미
- bitmap heap scan, bitmap index scan이란 무엇인가

[PostgreSQL Bitmap Index Optimization](https://chat.openai.com/share/15b3362c-e5f3-462a-8b09-dc7dfc780c04)
- bitmap index scan을 위해 bitmap을 구성 시 index 읽는 작업이 최적화 돼있는가?

[Lossy Bitmaps in PostgreSQL](https://chat.openai.com/share/02691b59-f33d-480e-b17e-93c939f51964)
- bitmap index scan시 bitmap이 lossy하게 구성되는지 여부를 알수있는 방법은?
- explain시에 나오는 shared hit, exact , heap blocks, "Rows Removed by Filter" 등등의 의미

[Downsides of Multi-Column Indexes](https://chat.openai.com/share/e57c412e-459f-4d98-8b96-60d7f228cc4c)
- postgresql에서 multi column index의 단점은?
- b-tree multi col index가 실제적으로 저장되는 방식은?