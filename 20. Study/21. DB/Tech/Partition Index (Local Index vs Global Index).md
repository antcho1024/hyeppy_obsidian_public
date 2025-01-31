---
sticker: emoji//1f9be
---
🔺 **[[Partition Table]]에는 다른 방식으로 인덱스를 걸어줘야 한다**

## Local Index (주로 사용)
<font color="#c0504d">파티션마다 독립적으로 생성</font>됨. ( 파티션 내에서만 유효 )

- 로컬 인덱스는 테이블 파티션 키 컬럼을 똑같이 인덱스로 구성
- 테이블의 파티션 구조가 바뀐다거나 파티션이 삭제가 된다고 하더라도 인덱스 재생성이 필요가 없어 오라클에서 알아서 자동으로 관리
- 👍 파티션별 데이터 접근에 최적화

ex) `idx_emp_name` 인덱스는 각 파티션(`p1`, `p2`, `p3`)에 대해 별도로 생성됨
```SQL
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(100),
    department_id INT,
    hire_date DATE
)
PARTITION BY RANGE (hire_date) (
    PARTITION p1 VALUES LESS THAN (TO_DATE('2020-01-01', 'YYYY-MM-DD')),
    PARTITION p2 VALUES LESS THAN (TO_DATE('2021-01-01', 'YYYY-MM-DD')),
    PARTITION p3 VALUES LESS THAN (TO_DATE('2022-01-01', 'YYYY-MM-DD'))
);

CREATE INDEX idx_emp_name ON employees (name) LOCAL;
```


## Global Index
파티션을 무시하고 <font color="#c0504d">전체 테이블을 대상으로 생성되는 인덱스</font> (하나의 인덱스가 테이블 전체 데이터를 관리)
👍 파티션 키와 상관없는 데이터 접근에 유리

ex)
```SQL
CREATE INDEX idx_global_emp_name ON employees (name) GLOBAL;
```

### 비교 요약

|        |                     Local Index                      |                         Global Index                         |
| :----: | :--------------------------------------------------: | :----------------------------------------------------------: |
|   구조   |                    파티션 별로 독립된 인덱스                    |                    테이블 전체를 대상으로 하는 단일 인덱스                    |
| 관리 용이성 |              파티션 추가/삭제 시 인덱스도 자동으로 관리됨               | **<font color="#4f81bd">*</font>** 파티션 관리 시 인덱스 관리가 복잡할 수 있음 |
|   컬럼   | <font color="#c0504d">파티션 키 컬럼을 인덱스로 지정</font>해주어야 함 |  파티션 키 컬럼 외에 <font color="#c0504d">다른 컬럼들도 인덱스 지정 가능</font>  |
|   성능   |   <font color="#c0504d">특정 파티션에 대한 쿼리</font>에 최적화됨   |       <font color="#c0504d">파티션 키와 상관없는 쿼리</font>에 유리함       |
| 사용 사례  |                   파티션 키를 사용한 범위 쿼리                   |                   전체 데이터 범위에서 키 검색이 필요할 때                    |
**<font color="#4f81bd">*</font>** ex)  log 를 저장하는 Partition Table 에서 시간이 지나 필요 없는 데이터들을 제거하기 위해 파티션 DROP을 수행하게 되는 경우: 
→ Global Index 로 구성되어 있다면 필요 없는 테이블 파티션을 DROP 한 뒤에 인덱스가 깨짐.
→ 이렇게 인덱스가 깨져버리면 Invaild 상태가 됨
```
ORA-01502: index or partition of such index is in usable state tips  
ORA-01502: 인덱스 또는 인덱스의 분할 영역은 사용할 수 없는 상태입니다.
```
→ 이 인덱스를 다시 사용하기 위해서는 인덱스 리빌딩 작업을 다시 해줘야 하는 등 번거로움 존재.
하지만 로컬 인덱스는 각각의 파티션마다 하나씩 걸려 있기 때문에 파티션을 DROP 할 때 함께 제거되고 남아있는 파티션에는 전혀 지장이 없음.

🐤 파티션이 추가, 삭제, 병합되면 global 인덱스도 이에 대한것을 반영하도록 업데이트 해야함.


### (Local, Global) Prefixed Index
파티션된 테이블에서 인덱스를 생성할 때 <font color="#8064a2">파티션 키</font>를 포함 할 수도 있고 포함하지 않을 수도 있음.
이때, <span style="background:rgba(240, 200, 0, 0.2)"><font color="#8064a2">파티션 키</font>를 인덱스의 첫번째 컬럼으로 포함 한 것</span> : (Local, Global) Prefixed Index

🍒 사용 권장? (이해 못했음..)
- Table Partition Key를 Index로 설정할 경우  Local Prefixed Index를 사용
- PK 컬럼은 Table Partition Key를 첫 번째 컬럼으로 하는 Local Prefixed Index를 사용
- Non-partition 컬럼에 대한 빈번한 배치 작업 수행 시 Local Nonprefixed Index를 사용
- Non-partition 컬럼에 대한 Unique Index 설정 시 Global Index를 사용 (OLTP)
https://coding-factory.tistory.com/841#google_vignette
why? 
- **Local Prefixed Index**: 파티션 키를 포함한 인덱스로, 각 파티션에 대해 독립적으로 관리되어 쿼리 성능이 향상됨.
- **Local Nonprefixed Index**: 파티션 키와 무관한 컬럼에 대해 생성되며, 빈번한 배치 작업 시 효율적임.
- **Global Index**: 파티션 키가 아닌 컬럼에 대해 유니크 인덱스를 설정할 때 사용하며, 전체 파티션에 걸쳐 고유성을 보장함.