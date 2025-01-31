---
sticker: emoji//1f48b
---
**수행되어야 할 일련의 연산 모음**

### 트랜잭션 특징
- 원자성 : 모두 반영 되던가, 아예 반영이 되지 않던가 
	ex) 일부만 반영 되는 경우는 없어야함
	
 - 일관성 : 트랜잭션이 진행되는 동안에 데이터베이스가 변경 되더라도  **업데이트된 데이터베이스로 트랜잭션이 진행되는것이 아니라**, **처음에 트랜잭션을 진행 하기 위해 참조한 데이터베이스로 진행**
	 🐤 A 트랜잭션 처리 도중에 B 트랜잭션이 진행되더라도 A는 B를 참고?하지 않음
	참고 : <font color="#2DC26B">트랜잭션 격리 수준(Isolation Level)</font> [[동시성 제어]] 
		 - **READ UNCOMMITTED**: 트랜잭션이 커밋되지 않은 변경 사항도 읽을 수 있습니다.
		- **READ COMMITTED**: 트랜잭션이 커밋된 변경 사항만 읽을 수 있습니다.
		- **REPEATABLE READ**: 트랜잭션이 시작될 때 읽은 데이터가 트랜잭션이 끝날 때까지 동일하게 유지됩니다.
		- **SERIALIZABLE**: 가장 높은 격리 수준으로, 트랜잭션이 완전히 독립적으로 실행됩니다.
	
- 독립성 : 다른 트랜잭션의 연산에 끼어들 수 없음

- 영구성 : 결과는 영구적으로 반영

![[Pasted image 20240624171539.png]]

### 롤백

``` SQL
START TRANSACTION; -- 트랜잭션 시작
insert into members values(5, '쿠', '크다스 동생', '크라운제과', '?', '대한민국'); -- 데이터 수정
select * from members; -- 수정 상태 보여줌
ROLLBACK -- 트랜잭션을 취소하고 START TRANSACTION 실행 전 상태로 롤백함
```

### 자동 커밋
DDL문(CREATE, DROP, ALTER, RENAME, TRUNCATE) 은 자동으로 커밋되기 때문에 롤백 대상이 아님

```SQL
-- 트랜잭션 시작 
INSERT INTO employees (emp_id, emp_name) VALUES (1, 'John Doe'); 

-- DDL 문 실행 
CREATE TABLE test_table (id NUMBER); 
-- INSERT 문도 자동으로 커밋됨

ROLLBACK; -- 롤백 불가
```

