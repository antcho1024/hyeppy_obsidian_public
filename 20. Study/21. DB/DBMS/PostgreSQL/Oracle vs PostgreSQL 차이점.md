### MVCC모델
#### MVCC 이란?
데이터를 **버전 관리**하여 동시성을 제공하는 방식
읽기 작업은 쓰기 작업을 블로킹하지 않고 쓰기 작업은 읽기 작업을 블로킹하지 않아야한다.

> [!블로킹(blocking)]
> 데이터베이스 시스템에서 한 트랜잭션(Transaction)이 특정 리소스(데이터 또는 테이블 등)에 대한 잠금(lock)을 보유하고 있을 때, 다른 트랜잭션이 그 리소스를 사용하려고 시도하면서 **대기 상태**에 놓이는 상황

예를 들어
- A 트랜잭션에서 쓰기 작업 중일 때 B 트랜잭션에서 <font color="#c0504d">읽기 가능</font>
- A 트랜잭션에서 쓰기 작업 중일 때 B 트랜잭션에서 <font color="#c0504d">쓰기 불가능</font>
스냅샷마냥 특정 시점의 데이터를 읽게 되는데 이를 구현하는 방식이 DBMS 마다 다름.

이 MVCC가 없다면, 동시성 제공을 안해준다면 어떻게 되는지는 [블로그](https://retto9522.tistory.com/60) 참고

##### Oracle
<span style="background:#fff88f">UNDO 세그먼트를 생성해둠</span>
쓰기 작업(UPDATE, DELETE, INSERT 등)을 수행할 때, 기존 데이터의 이전 상태를 **Undo Segment**에 저장
트랜잭션이 실행 중일 때, 다른 트랜잭션이 과거 버전을 요청하면 Undo Segment에 저장된 데이터를 참조하여 이전 상태를 제공

##### PostgreSQL
<span style="background:#fff88f">블록 내에 이전 레코드를 저장함.</span>

PostgreSQL는 데이터를 행 수준에서 관리함
👉 데이터의 각 행에는 <font color="#c0504d">xmin</font>, <font color="#c0504d">xmax</font> 이라는 메타데이터를 함께 저장함.
- **xmin**: 해당 행을 생성한 트랜잭션 ID
- **xmax**: 해당 행을 삭제하거나 업데이트한 트랜잭션 ID.
	PostgreSQL에서 **데이터를 수정시** Delete + Insert로 처리함
	그래서 기존 행의 **xmax** 값에 현재 트랜잭션 ID를 기록하고, 새로운 행에는 새로운 **xmin** 값을 기록

⬇️
이 두 값을 통해 어떤 트랜잭션이 어떤 데이터에 접근 가능할지를 판단해줌.

**트랜잭션 읽기 동시성 제공**

| **xmin**             | **읽기**                                                      | **xmin 상황 설명**                             |
| -------------------- | ----------------------------------------------------------- | ------------------------------------------ |
| null                 | x                                                           | 이런경우 존재x                                   |
| 현재 transaction 보다 큼  | <font color="#c0504d">읽기 불가능</font>                         | 특정 트랜잭션이 데이터를 생성했지만<br>아직 commit 하지 않은 경우. |
| 현재 transaction 보다 작음 | <font color="#4f81bd">읽기 가능</font>                          | 예전에 생성됨.                                   |
| **xmax**             | **읽기**                                                      | **xmax 상황 설명**                             |
| null                 | <font color="#4f81bd">읽기 가능</font>                          | 수정된 적 없는 데이터                               |
| 현재 transaction 보다 큼  | <font color="#4f81bd">읽기 가능</font> <br>(아직 커밋되기 <br>전일 경우민) | 다른 트랜잭션에서 수정중인데이터                          |
| 현재 transaction 보다 작음 | <font color="#c0504d">읽기 불가능</font>                         | 다른데에 이미 수정된 상태로 존재함.                       |
*읽기 가능 , 읽기 가능 조합만 읽기 가능!*
근데 여기에 commit이 되었다는 조건도 들어가야하는거 같긴한데..? 일단 넘어가


```text
예시

(1) Transaction A (ID=1) 이 데이터 삽입함 
test=100  (xmin=1, xmax= Null)

(2) Transaction B (ID=2) 가 데이터 수정함 (커밋 완료)
test=100  (xmin=1, xmax= 2)
test=200  (xmin=2, xmax= Null)

(3) Transaction C (ID=3) 가 데이터 조회하려고 함
test=100  (xmin=1, xmax= 2)       > 조회 불가
test=200  (xmin=2, xmax= Null)    > 조회 가능

test=200  (xmin=2, xmax= 4)       > 다른 트랜잭션에서 수정중임. 조회 가능

```


PostgreSQL에서 MVCC는 <font color="#9bbb59">데이터 행의 여러 버전을 생성</font>하기 때문에, 쓸모없는 데이터 버전이 쌓임 > 이를 관리하기 위해 **Vacuum 프로세스**가 필요 [[Vacuum]]
*Undo 세그먼트를 사용하지 않기에 복잡성은 낮췄지만 Vacuum 이 일을 해줘야한다!*

### Shared Pool 존재 여부
Oracle 에는 존재함. (파싱한 SQL, 데이터 틱셔너리에서 조회한 데이터, 쿼리 결과 등 저장해두는 캐시)
PostgreSQL에는 존재하지 않음.
대신 프로세스 레벨에서 SQL 정보를 공유하는 기능을 제공함.
하나의 프로세스에서 같은 SQL을 여러번 수행하면 최초 1회만 하드 파싱. > 3장 옵티마이저 동작원리 참고


