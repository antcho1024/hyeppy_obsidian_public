데이터베이스 내에 저장되는 모든 엔터티

#### Table
table
#### View
하나 이상의 테이블로부터 유도된 <font color="#c0504d">가상의 테이블</font>
- SELECT 쿼리를 기반으로 생성
- 데이터 접근 단순화, 보안 강화 ← 사용 이유
  ex) 특정 열만 포함하는 뷰를 생성하여 사용자가 모든 데이터를 보지 못하게 함, 접근 가능한 사람을 지정 가능
- 👍 기본 테이블 데이터 업데이트시 → 적용됨 (실시간 상태 반영)
```SQL
CREATE VIEW active_employees AS
SELECT employee_id, name, department
FROM employees
WHERE status = 'active';
```

😀 <span style="background:rgba(160, 204, 246, 0.55)">장점</span> 
- 원하는 부분만 가져와서 사용할 수 있다.
- 복잡한 쿼리를 단순화해서 사용 가능하다.
- 데이터의 보안이 용이하다.
- 사용자가 데이터를 관리하기가 쉽다.
- 논리적 독립성을 제공한다.

😑 <span style="background:rgba(160, 204, 246, 0.55)">단점</span>
- 인덱스를 구성할 수 없다.
- 한번 정의된 뷰는 수정이 불가하다.
- 삽입, 갱신, 삭제 연산에 많은 제약이 있다

#### 인덱스
[[Index]]
#### Sequence
자동 증가 컬럼 생성 방법
(다른 DB에는 컬럼 자체에 옵션이 있으나 오라클에는 존재하지 않음 )
1. <span style="background:#fdbfff">Max(컬럼) + 1 </span>
   해당 방법도 많이 사용
   ```SQL
   insert into memos(num,name) values((select max(num)+1 from memos), '세종대왕');
   ```
   ✔️ 참고 : 시퀀스는 컴퓨터를 부팅 또는 다른 사람의 파일을 가지고 와서 사용할 때, 
   파일의 수정사항에 대해서  내부에 가지고 있는 시퀀스가 다름으로 시퀀스의 번호가 random 으로 나누어질 수 있습니다.  그렇기 때문에 새롭게 모든 시퀀스를 삭제하고 순차적으로 진행해야 합니다.    
   이러한 이유로 시퀀스 없는 상태에서 자동 증가값 구현 하는 사람들이 많으며 시퀀스 번호로 일련번호를 넣는것을 선호하지 않습니다. 그래서 그룹 함수의 max 함수를 사용하여 번호를 삽이하는 형태로 구현을 선호
   
2. <span style="background:#fdbfff">시퀀스</span>
   유니크한 숫자 값을 생성하기 위한 객체![[Pasted image 20240705155442.png]]
   - INCREMENT BY : 시퀀스 실행 시 증가시킬 값
   - START WITH : 시퀀스의 시작값이다. (MINVALUE과 같거나 커야 한다)
   - MINVALUE : 시퀀스가 시작되는 최솟값이다.
   - MAXVALUE : 시퀀스가 끝나는 최댓값이다.
   - NOCYCLE | CYCLE : NOCYCLE (반복안함), CYCLE(시퀀스의 최댓값에 도달 시 최솟값 1부터 다시시작)
   - NOCACHE | CACHE : NOCACHE(사용안함), CACHE(캐시를 사용하여 미리 값을 할당해 놓아서 속도가 빠르며, 동시 사용자가 많을 경우 유리)
   - NOORDER | ORDER : NOORDER(사용안함), ORDER(요청 순서로 값을 생성하여 발생 순서를 보장하지만 조금의 시스템 부하가 있음)
🚩생성
```SQL
CREATE SEQUENCE emp_seq 
		INCREMENT BY 1 
		START WITH 1;
```

🚩사용
```SQL
# 기본
SELECT emp_seq.NEXTVAL FROM dual

# table에 Insert 시
INSERT INTO emp(empno, ename, job) VALUES (emp_seq.NEXTVAL , 'TIGER' , 'ANALYST')
```
**시퀀스명.NEXTVAL**을 사용하여 일련번호를 생성.
시퀀스를 실행할 때마다 값이 증가하니 주의. (증가된 값을 다시 내릴 수 없음)

*참고 : **시퀀스명.CURRVAL**을 사용하여 현재 시퀀스 순번을 갖고 옴. 
 - CURRVAL은 여러번 실행해도 순번은 증가하지 않음
 - INSERT, SELECT 등 쿼리문에 사용 시 NEXTVAL과 함께 사용 (아니면 에러남)
 
🚩수정
🚩삭제

#### 트리거
- **정의**: 특정 이벤트가 발생할 때 자동으로 실행되는 SQL 코드 블록입니다.
- **구성**: INSERT, UPDATE, DELETE 등의 이벤트에 반응하도록 설정됩니다.
- **용도**: 데이터 무결성을 유지하거나 자동화된 작업을 수행합니다. 예를 들어, 직원 정보가 업데이트될 때 변경 로그를 기록하는 트리거를 생성할 수 있습니다.

```SQL
CREATE TRIGGER log_employee_update
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
  INSERT INTO employee_log (employee_id, update_time)
  VALUES (:OLD.employee_id, SYSDATE);
END;

```
#### 프로시저
#### 함수
#### Package
#### Cursor
#### 사용자 (User) 및 역할 (Role)
#### Synonym
#### Tablespace
[[Tablespace]]
