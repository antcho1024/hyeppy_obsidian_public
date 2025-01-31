---
sticker: emoji//1f61d
---
- [[#파티션 테이블|파티션 테이블]]
	- [[#파티션 테이블#장단점|장단점]]
	- [[#파티션 테이블#ex) 예시|ex) 예시]]
	- [[#파티션 테이블#🔔 실전 상황|🔔 실전 상황]]
	- [[#파티션 테이블#파티션 테이블 종류|파티션 테이블 종류]]
	- [[#파티션 테이블#파티션 프루닝(Partition Pruning)|파티션 프루닝(Partition Pruning)]]
- [[#서브 파티션|서브 파티션]]

## 파티션 테이블
특정 컬럼값을 기준으로 데이터를 분할해 저장해놓은 테이블
- 논리적인 테이블은 1개
- 물리적으로는 N개

### 장단점
👉 장점
- <font color="#c0504d">Select Query Performance가 향상</font>될 수 있다. (Table Full Scan이 필요한 조회 쿼리)
- <font color="#c0504d">디스크 장애 시 해당 파티션만 영향을 받으므</font>로 데이터의 훼손 가능성이 감소하고 가용성이 향상
- 개별 Partition 단위의 관리가 가능 (DML, Load, Import, Export, Exchange 등
- 논리적으로는 하나의 테이블이기 때문에 개발되어 있는 쿼리문을 변경할 필요가 없음
- <font color="#c0504d">조인 시</font> 파티션 간의 병렬 처리 및 파티션 내에서의 <font color="#c0504d">병렬 처리를 수행</font>  
- 데이터 액세스 범위를 줄여 성능을 향상하고 테이블의 파티션 단위로 <font color="#c0504d">디스크의 I/O를 분산해 부하를 감소</font>

👉 단점
- 오버헤드 발생 (오버헤드 : 특정 작업을 수행하기 위해 필요한 추가적인 자원)
	- 파티션 키 컬럼의 값이 변경 되면 다른 파티션으로 이동해야함
	- 데이터 insert 시 연산 발생
	- join 비용 증가
	- (이해 못함) 파티션에 기준이 되는 것이 컬럼의 일부일 때 일부를 기준으로 파티션을 구성할 수 없으므로 이에 해당하는 오버헤드 컬럼이 있어야 함.

### ex) 예시
```SQL
CREATE TABLE sales (
    sale_id NUMBER,
    sale_date DATE,
    amount NUMBER
)
PARTITION BY RANGE (sale_date) (
    PARTITION p1 VALUES LESS THAN (TO_DATE('2023-01-01', 'YYYY-MM-DD')),
    PARTITION p2 VALUES LESS THAN (TO_DATE('2023-07-01', 'YYYY-MM-DD')),
    PARTITION p3 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD'))
);
```

### 🔔 실전 상황

**<span style="background:rgba(240, 200, 0, 0.2)">파티션 테이블로 지정하기 좋은 상황</span>**
- 데이터의 양이 많고 INSERT가 지속적으로 일어나는 테이블 
	ex) log 테이블 -> 데이터가 많고 지속적으로 insert됨. 기간 별로 파티셔닝해 로그 조회 속도 를 빠르게 하고 필요 없는 데이터의 파티션만 제거해서 효율적으로 관리
- 일부 데이터가 손상되더라도 나머지 데이터 사용이 가능해야 하는 테이블
- DB 장애 시 복구를 최대한 빨리 해야 하는 테이블
  
<span style="background:rgba(240, 200, 0, 0.2)">키선정</span>
- Primary Key와 같이 데이터들의 구분이 지어지지 않는 컬럼은 피해야 한다.
- 데이터들이 어디에 들어가 있는지 직관적으로 판단이 가능한 컬럼(날짜 컬럼)
- I/O 병목을 줄일 수 있는 데이터 분포도가 양호한 컬럼
  
 **<span style="background:rgba(240, 200, 0, 0.2)">주의</span>**
- 여러 파티션에 대한 조회는 한 테이블로 구성하였을 경우보다 효율이 떨어짐
```SQL
# 아래와 같은 테이블이 있다고 가정했을 때
CREATE TABLE orders (
    order_id INT,
    order_date DATE,
    customer_id INT,
    amount DECIMAL(10, 2)
)
PARTITION BY RANGE (order_date) (
    PARTITION p1 VALUES LESS THAN (TO_DATE('2022-01-01', 'YYYY-MM-DD')),
    PARTITION p2 VALUES LESS THAN (TO_DATE('2023-01-01', 'YYYY-MM-DD')),
    PARTITION p3 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD'))
);

# 인덱스 생성시 `order_date`를 기준으로 파티셔닝되어야 함
CREATE INDEX idx_order_id ON orders (order_id)
PARTITION BY RANGE (order_date) (
    PARTITION p1 VALUES LESS THAN (TO_DATE('2022-01-01', 'YYYY-MM-DD')),
    PARTITION p2 VALUES LESS THAN (TO_DATE('2023-01-01', 'YYYY-MM-DD')),
    PARTITION p3 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD'))
);

```

<span style="background:rgba(240, 200, 0, 0.2)">파티션 숫자 결정</span>

| 파티션이 많은 것이 속도가 빠름                                                             | 파티션이 적은 것이 속도가 빠름                                                       |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| <font color="#c0504d">Composite Primary Key의 전체</font>를 사용할 경우                | 검색조건에서<font color="#c0504d"> Composite Primary Key의 일부분</font>만을 사용할 경우 |
| <font color="#1f497d">OLTP</font> 작업                                          | <font color="#1f497d">Batch, OLAP</font> 작업                             |
| 너무 많을 경우 각 Partition의 value range를 체크하므로 parsing time이 길어지고 관리대상이 많아지는 단점이 존재 | Partition 효과를 볼 수 없                                                     |

### 파티션 테이블 종류
- ##### Range Partitioning
	- 일, 월, 분기 등 특정 컬럼의 정렬 값을 기준으로 분할하는 방식으로 논리적인 범위의 분산에 효율적
	- 관리가 용이하며 이력 데이터에 적합
	- 파티션을 결정하는 컬럼을 명시하여야 하며 MAXVALUE값은 NULL값을 포함
	- 범위가 포함하는 데이터의 양이 일정하지 않은 경우 특정 파티션에 대해 데이터가 편중될 수 있음
- ##### Hash Partitioning
	- 데이터의 균등 분할을 통해 성능을 향상하고자 하는 경우에 효율적
	- 파티션 키에 해시함수를 적용한 결과 값이 같은 레코드를 같은 파티션 세그먼트에 저장해 두는 방식 - Row와 파티션 간의 매핑을 사용자가 제어할 수 없음
	- 파티션 키의 해시 값에 의해 데이터가 다수의 파티션에 분배되며 균등한 분배를 위해서는 파티션 개수를 명시하여야 하며 파티션의 수는 2의 거듭제곱의 수로 지정
	- NULL값은 첫 번째 파티션에 위치함
	- 고객 ID처럼 변별력이 좋고 데이터 분포가 고른 컬럼을 파티션 키 컬럼으로 선정해야 효과적
- ##### List Partitioning
	- 파티션 컬럼을 명시, 키 컬럼 값을 기준으로 파티션 하는 방식 컬럼의 구체적인 값들에 대해 파티션을 명확하게 컨트롤하고자 할 때 효율적
	- 연관되지 않은 데이터, 순서에 맞지 않는 데이터의 그루핑을 쉽게 할 수 있음
	- 명시되지 않은 값을 가진 Row는 Insert가 불가능
	- 여러 컬럼으로 파티션 키를 생성할 수 없고 오직 하나의 컬럼만 가능
	- 각 파티션에 대해 모든 파티션 키는 반드시 문자로 LIST 되어야 하며 파티션 값의 List는 4K까지 가능
	- 파티션 키의 값은 64K-1을 초과할 수 없고 NULL 값을 포함한 어떠한 값이라도 한 번만 명시 가능
- ##### Composite Partitioning
	- Range + List, Range + Hash 파티션 등의 조합으로 구성하며 주 파티션과 서브파티션으로 이루어짐
	- 이력 데이터와 온라인 데이터의 복합적인 성격을 지닌 데이터의 분할에 용이하며 병렬 DML 작업에 뛰어난 수행성능을 보장
	- 파티션 및 서브 파티션 단위의 관리 작업 수행이 가능
	- Hash 파티셔닝의 경우 스토리지 스트라이핑으로 인해 디스크 점핑이 발생할 수 있으므로 충분한 검토 후 적용
	- 단 이렇게 구성할 경우 파티션을 나누는 기준이 두개가 되기 때문에 파티션의 갯수가 너무 많아지게 됨 (주 파티션의 갯수 X 서브파티션의 갯수) 인덱스의 경합이 너무 심해져서 잘 사용하지는 않음

_***🍒<font color="#c0504d"> Range -> Hash -> List 순으로 많이 사용</font>***_


### 파티션 프루닝(Partition Pruning)
쿼리를 최적화하는 기술로,
쿼리가 <font color="#c0504d">특정 파티션만 접근하도록</font> 하여 <font color="#c0504d">불필요한 파티션을 스캔하지 않도록 해줌</font>

🐤 즉, 쿼리에 파티션 키 조건을 포함 시켜 특정 파티션만 접근하게 함으로써 성능을 최적화 하게함

ex)
아래와 같은 테이블이 있을 때 (*order_date* 로 파티셔닝이 되어 있음 )
```SQL
CREATE TABLE orders (
    order_id INT,
    order_date DATE,
    customer_id INT,
    amount DECIMAL(10, 2)
)
PARTITION BY RANGE (order_date) (
    PARTITION p1 VALUES LESS THAN (TO_DATE('2022-01-01', 'YYYY-MM-DD')),
    PARTITION p2 VALUES LESS THAN (TO_DATE('2023-01-01', 'YYYY-MM-DD')),
    PARTITION p3 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD'))
);
```

✔️ 파티션 프루닝 가능한 쿼리
```SQL
SELECT * FROM orders WHERE order_date >= TO_DATE('2022-01-01', 'YYYY-MM-DD') AND order_date < TO_DATE('2023-01-01', 'YYYY-MM-DD');
```
(where 절에 *order_date*  조건을 추가함)

❌전체 파티션 접근하게 하는 쿼리
```SQL
SELECT * FROM orders WHERE customer_id = 12345;
```


---

## 서브 파티션 
ex)
```SQL
CREATE TABLE orders (
    order_id INT,
    order_date DATE,
    customer_id INT,
    region VARCHAR(50),
    amount DECIMAL(10, 2)
)
PARTITION BY RANGE (order_date)
SUBPARTITION BY LIST (region)
(
    PARTITION p1 VALUES LESS THAN (TO_DATE('2023-01-01', 'YYYY-MM-DD')) 
    (
        SUBPARTITION sp1 VALUES ('North'),
        SUBPARTITION sp2 VALUES ('South'),
        SUBPARTITION sp3 VALUES ('East'),
        SUBPARTITION sp4 VALUES ('West')
    ),
    PARTITION p2 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD')) 
    (
        SUBPARTITION sp5 VALUES ('North'),
        SUBPARTITION sp6 VALUES ('South'),
        SUBPARTITION sp7 VALUES ('East'),
        SUBPARTITION sp8 VALUES ('West')
    )
);


```

