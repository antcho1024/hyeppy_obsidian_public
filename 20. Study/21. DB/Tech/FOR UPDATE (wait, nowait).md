---
sticker: emoji//1f3ca-200d-2642-fe0f
---
오라클 데이터베이스에서는 선택된 행들에 대하여 <font color="#c0504d">배타적인 LOCK</font>을 설정할 수  
있는 기능
보통 SELECT 문이랑 같이 사용됨 (<font color="#4f81bd">SELECT 문을 통해 조회된 데이터에 대해 수정할 것임을 명시</font>)

☘️ 옵션 종류
-  <span style="background:#fff88f">FOR UPDATE with no option</span>
   LOCK을 획득하기까지 무한정 기다림
- <span style="background:#fff88f">FOR UPDATE NOWAIT (= WAIT 0)</span>
   LOCK을 획득하지 못하면 ORA-00054와 함께 바로 실패
```SQL
-- SESSION1 이 LOCK을 점유하고 있는 상황 가정
V901:SESSION2> select ename  from scott.emp where empno=7900 for update nowait;  

ERROR at line 1:  
ORA-00054: resource busy and acquire with NOWAIT specified
```
- <span style="background:#fff88f">UPDATE WAIT integer(0 ~ 4294967295, second)</span>
  WAIT 다음 주어지는 정수 만큼 동안 LOCK을 획득하기 위해 재시도 (그만큼 기다렸다가)
  -> LOCK을 획득하지 못하면 ORA-30006
  -> 숫자 설정 x 혹은 값 범위 넘어갈 경우 ORA-30005
- <span style="background:#fff88f">FOR UPDATE</span>
  해당 행들에 배타 잠금(Exclusive Lock)을 설정
```SQL
SELECT column1, column2
FROM table_name
WHERE condition
FOR UPDATE;
```
- <span style="background:#fff88f">FOR UPDATE OF</span>
  특정 열에 대한 업데이트 의도를 명시
```SQL
SELECT a.column1, b.column2
FROM table_a a, table_b b
WHERE a.id = b.id
FOR UPDATE OF a.column1, b.column2;
```
`table_a`의 `column1`과 `table_b`의 `column2`를 업데이트할 의도를 나타내며, 해당 열이 포함된 행들에 잠금을 설정



🚩 번외, Table 잠금
```SQL
LOCK TABLE employees IN EXCLUSIVE MODE;
```
FOR UPDATE는 SELECT 문을 통해 조회된 데이터에 대해서만 잠군다면,
이는 테이블 전체에 배타 잠금을 설정하는 강력한 잠금