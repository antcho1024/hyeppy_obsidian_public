**Vacuum이 하는 일 미리보기** 
- 불필요한 쓰레기 데이터 정리
- 정리된 공간을 압축(진공)해서 디스크 공간의 효울성 높임
- 오래된 트랜잭션 ID를 정리

정리하면,
1. 공간 재활용 (권장. 안하면 공간 효율 ⬇️)
	1. 오래된 이전 버전 레코드 삭제 작업을 통한 공간 확보   `Vacuum` or `Vacuum Freeze`
	2. 오래된 이전 버전 레코드 삭제 작업 후에 공간 압축      `Vacuum Full`
2. XID Frozen (필수) > 작업 이름: Anti-wraparound Vacuum, 위 명령어 시 같이 수행됨.

**Vaccum이 필요한 이유**
- 이전 데이터를 보관하기 때문

#### MVCC
[[Oracle vs PostgreSQL 차이점]] MVCC 내용참고 필요. (같은 내용은 적지 않음)

- 트랜잭션 ID(XID): 트랜잭션마다 1씩 증가

위 페이지를 읽어보면 *쿼리가 시작된 시점의 XID와 같거나 작은 데이터 버전을 읽는 것*이 MVCC의 핵심 (xmin 을 말하는듯)

🙋**특징**
1. 이전 버전의 데이터를 테이블 블록 내에 저장함
2. 레코드(행) 별로 트랜잭션 ID (XID)를 관리함.
   행마다 4바이트 정수로 XID를 같이 저장하는데 
   4바이트 정수로 저장할 경우, 최댓값이 대략 43억임.
   ⬇️
   그럼 그 이후엔?
   회전함. > 그럼 *쿼리가 시작된 시점의 XID와 같거나 작은 데이터 버전을 읽는 것* 으로 가시성을 판단한다고 했는데, 회전 후 10번이 회전 전 42억 8000번 데이터를 읽을 수 있는지 없는지 판단할 수 없게 됨.
   ⬇️
   XID를 3개의 유형으로 정함
   - Bootstrap XID: initdb() 시에 할당되는 XID. 값은 1
   - <font color="#c0504d">Frozen XID</font>: Anti-wraparound Vacuum을 위해 적용하는 XID. <font color="#c0504d">값은 2</font>
   - <font color="#4f81bd">Normal XID</font>: 트랜잭션이 사용하는 XID. <font color="#4f81bd">값은 3부터 시작</font>
	
	무슨 말이냐 하믄, 1바퀴 회전 전에 이전 XID를 Frozen XID로 변경.
	회전 이후에 42억 얼마얼마 던 것들을 다 2로 변경함.
	그럼 회전 이후에 3부터 증가하니까 이전 데이터들보다 항상 큼.
	이 작업을  **Anti-wraparound Vacuum** 이라고함.


#### 수행 방법, 실습 보기
`Vacuum` `Vacuum Freeze` `Vacuum Full`
오른쪽으로 갈 수록 큰 작업으로 왼쪽 작업을 포함함.
- **Vacuum**:              쓰레기 데이터 삭제 (+ Frozen 처리도 가능)
- **Vacuum Freeze**: 쓰레기 데이터 삭제 + 오래된 데이터 Frozen 처리
- **Vacuum Full**:      쓰레기 데이터 삭제 + 테이블 새로 작성 + Frozen 처리도 + 공간 압축.

```SQL
vacuum <테이블 명> # 수행시, 테이블과 테이블에 딸린 인덱스에 대한 Vacuum 작업 수행됨.
```
![[Vacuum-20250114111601908.webp|527]]

![[Vacuum-20250114111809055.webp|527]]

![[Vacuum-20250114111831033.webp|527]]

정리,
- PostgreSQL MVCC는 행마다 XID를 관리, 이전 버전도 블록내에 저장되어 존재함
- 이전 버전이 존재하므로 정리해주는 Vacuum이 필요
- XID를 4바이트로 순환하여 사용하기 때문에, Anti-wraparound Vacuum이 필요함


#### Vacuum 과 Lock
**Vacuum**: 이전 레코드 삭제
- 가벼운 작업으로 다른 작업(SELECT, INSERT, UPDATE 등)과 함께 실행 가능함.
  "여기 잠깐 쓰레기를 치울게요. 여러분은 계속 작업하세요!"
```SQL
# 세션1
SELECT * FROM t1;

# 동시에 세션2
VACUUM t1;

# 둘 다 정상 실행됨. 충돌 x
```

**Vacuum Full**: Table Compact 하게 만듦
- Vacuum Full은 테이블을 완전히 재작성하는 작업
  성능 최적화에는 좋지만, 작업 중에 테이블이 완전히 잠금(락)
```SQL
# 세션1
SELECT * FROM t1; > "ACCESS SHARE" 락을 걸어 테이블에 읽기 권한을 확보

# 동시에 세션2
VACUUM t1; 

# Vacuum Full이 **대기 상태**에 빠집니다. 테이블을 독점적으로 잠글 수 없음
```
따라서 이 명령어는 신중히 사용해야함

> [!CLUSTER > PG_REPACK 익스텐션]
> Vacuum Full을 꼭 사용해야한다면 Cluster > PG_REPACK 익스텐션을 사용하면 덜 강력한 락을 사용함 


#### Vacuum 과 Redo Log
- **Vacuum**:
    - "기존 레코드를 지우는 작업"으로 Redo 로그가 발생 X

- **Vacuum Freeze**:
	- 이 작업은 각 레코드의 `xmin` 값을 업데이트하므로 **Redo 로그**가 많이 발생
	      
- **Vacuum Full**:
    - 테이블을 완전히 새로 만드는 작업.
    - 테이블 크기를 줄이고 Redo 로그가 많이 발생.

#### Age
데이터베이스, 테이블, 레코드는 나이를 갖고 있음
![[Vacuum-20250114201038714.webp]]
![[Vacuum-20250114201104528.webp]]
트랜잭션 발생때마다 나이가 1살씩 증가함.

나이순: 데이터베이스 > 테이블 > 레코드

**vacuum_freeze_min_age 파라미터**
![[Vacuum-20250114201228959.webp]]
이 파라미터의 값보다 나이가 많은 레코드를 XID Frozen 대상으로 하겠음.

**"Age"속성**
- 데이터베이스, 테이블, 레코드 별로 나이를 관리한다.
- 사용자 데이터베이스 생성 시의 나이는 template1 데이터베이스의 나이와 같다. 이는, templatel 데이터베이스를 복제하기 때문이다.
- 테이블 생성 시의 나이는 1살이다.
- 레코드 입력 시의 나이도 살이다.
- 트랜잭션이 발생할 때마다 데이터베이스, 테이블, 레코드의 나이가 살씩 증가한다.
- 즉, 나이는 '현재 XID - 생성 시점의 XID'로 계산된다.
- 나이가 오래된 테이블과 레코드는 Vacuum 대상이다.
- Vacuum을 수행한 후에는 테이블과 레코드의 나이가 어려진다.

> 책 퀴즈 풀어보기 (?) 더 이해된담에 풀어야할듯 몰겠삼 ㅋ

#### Autovacuum
Vacuum 자동 수행기능
- autovacuum 파라미터로 설정함
- 완전히 off 할 수 없음
  off 해도 Anti-wraparound Vacuum 은 동작함.
  XID Frozen 을 적절한 시점에 수행하지 못하면 DB를 사용하지 못할정도의 문제가 발생하기 때문
  *9.6에서 해결?*

- autovacuum_freeze_max_age 파라미터
  테이블의 나이가 해당 파라미터 값 보다 크면 Anti-wraparound Vacuum 이 수행됨

> 책 퀴즈 풀어보기 (?) 더 이해된담에 풀어야할듯 몰겠삼 ㅋ

#### Visibility Map

#### HOT(Heap Only Tuple)