---
sticker: emoji//1f49b
---
### 제약조건이란 
테이블에 <font color="#c0504d">데이터를 작성(삽입/갱신) 할 때 조건에 맞지 않는 데이터를 입력 시키지 않기 위한</font> 구조
테이블의 열에 제약을 거는 것으로 데이터 베이스 측은 <font color="#1f497d">데이터의 무결성</font>을 가질 수 있음

### 무결성 제약조건 종류 5가지
- <span style="background:#d3f8b6">기본키 (primary key) 제약</span> : 컬럼 값은 반드시 존재 + 유일 (NOT NULL + UNIQUEm)
- <span style="background:#d3f8b6">고유키 (Unique Key) 제약</span> : 유일한 값. 중복금지 (UNIQUEm)
- <span style="background:#d3f8b6">NOT NULL 제약</span>
- <span style="background:#d3f8b6">체크 (CHECK) 제약</span> : 해당 컬럼에 입력할 수 있는 값의 범위나 조건 지정
- <span style="background:#d3f8b6">외부 참조키 (Foregin Key) 제약</span> : 다른 테이블의 <font color="#c0504d">기본 키</font> 또는 <font color="#c0504d">고유 키</font> 참조 하는 제약

### 제약 조건을 정의하는 2가지 방법

- 컬럼 제약 (열 제약)
```SQL
CREATE TABLE TEST_TABLE
(
	ID VARCHAR2(10)    PRIMARY KEY,
	NAME VARCHAR2(10)  NOT NULL,
	TEL VARCHAR2(10)   UNIQUE,
	AGE NUMBER(2)      CHECK(AGE BETWEEN 18 AND 65),
	DEPT_CD CHAR(2)    REFERENCES DEPT_TABLE(DEPT_CD)
};
```

- 테이블 제약

``` SQL
CREATE TABLE TEST_TABLE(
	ID1 VARCHAR2(5),
	ID2 VARCHAR2(10),
	NAME VARCHAR2(10),
	TEL VARCHAR2(10),
	AGE NUMBER(2),
	CHAR(2),
	DEFT_CD,
	
	CONSTRAINT cons_p1 PRIMAEY KEY(ID1, ID2),
	CONSTRAINT cons_u1 UNIQUE(TEL),
	CONSTRAINT cons_c1 CHECK(AGE BETWEEN 18 AND 65), 
	CONSTRAINT cons_f1 FOREIGN KEY (DEPT_CD)
					   REFERENCES DEPT_TABLE(DEPTO_CD)
);
```


*참고 : 제약을 추가  및 삭제*

``` SQL
ALTER TABLE 테이블명 ADD CONSTRAINT 제약명 대상컬럼(컬럼명);

# 예시
ALTER TABLE test ADD CONSTRAINT const1 unique(col1);
```

``` SQL
ALTER TABLE 테이블명 DROP CONSTRAINT 제약명;

# 예시
ALTER TABLE test DROP CONSTRAINT const1;
```

*참고 : 제약 활성화/ 비활성화*

``` SQL
ALTER TABLE 테이블명 [ENABLE/DISABLE] CONSTRAINT 제약명;
```


### 그 외의 제약 조건

- **DEFERRABLE**: 제약 조건의 검사를 트랜잭션 종료 시점까지 연기할 수 있는 옵션입니다.

```SQL
CREATE TABLE employees (
    emp_id NUMBER PRIMARY KEY,
    emp_name VARCHAR2(100),
    dept_id NUMBER,
    CONSTRAINT emp_dept_fk FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
											    DEFERRABLE INITIALLY DEFERRED
);
```

        
- **INITIALLY IMMEDIATE**: 제약 조건을 즉시 검증하도록 설정합니다. 이는 기본 설정입니다.

```SQL
CREATE TABLE employees (
    emp_id NUMBER PRIMARY KEY,
    emp_name VARCHAR2(100),
    dept_id NUMBER,
    CONSTRAINT emp_dept_fk FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
												INITIALLY IMMEDIATE
);

```
