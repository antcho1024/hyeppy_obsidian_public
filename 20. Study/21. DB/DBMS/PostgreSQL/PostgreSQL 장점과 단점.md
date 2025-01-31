### PostgreSQL 소개
객체 관계형(object-relational) 데이터베이스 관리 시스템(DBMS) 

참고: [[RDBMS vs NoSQL]]
(🐤PostgreSQL은 SQL로 관리가 가능한 ORDBMS인데, 동시에 NoSQL처럼 다양한 객체를 저장할 수 있는 기술을 제공함 ➡️ RDBMS와 NoSQL의 장점을 일부 겸비한 데이터베이스)

RDBMS와 똑같이 테이블이지만, 컬럼에 기본 데이터 타입(`INT`, `VARCHAR`, `DATE`, `FLOAT`) 말고도 다양한 타입 가능 (객체 타입)
👉 객체 타입이란,  ??

JSON / JSONB(Binary JSON) 데이터를 저장, 조회, 처리 지원
```SQL
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    profile JSONB -- JSONB 타입으로 프로필 데이터를 저장
);

-- 데이터 삽입
INSERT INTO users (name, profile)
VALUES ('Alice', '{"age": 30, "city": "Seoul"}');

-- 특정 JSON 필드 값에 따라 필터링
SELECT name
FROM users
WHERE profile->>'city' = 'Seoul';

-- JSON 필드 조회
SELECT profile->>'age' AS age
FROM users;
```

사용자가 객체 타입을 정의
```SQL
-- 주소 객체 타입 정의
CREATE TYPE address_type AS (
    street VARCHAR(100),
    city VARCHAR(50),
    postal_code VARCHAR(20)
);

-- 학생 테이블 생성 (주소를 객체 타입으로 저장)
CREATE TABLE student (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    age INT,
    address address_type -- 객체 타입을 컬럼으로 사용
);

-- 학생 데이터 삽입
INSERT INTO student (name, age, address)
VALUES (
    'Alice', 
    20, 
    ROW('123 Main St', 'Seoul', '12345') -- 주소 객체 생성
);

-- 주소의 도시(city) 필드만 조회
SELECT name, address.city
FROM student;

name   | city
-------|------
Alice  | Seoul
```


> [!여러운 말로..]
> 
사용자 정의 객체와 테이블 접근 방식을 결합하여 보다 복잡한 데이터 구조를 구축하는 엔터프라이즈급 오픈소스 객체 관계형(object-relational) 데이터베이스 관리 시스템(DBMS)
확장성과 SQL 규정 준수를 위해 관계형 및 비관계형 쿼리를 위한 SQL과 JSON을 모두 지원
PostgreSQL은 고급 데이터 유형과 성능 최적화 기능을 지원하며, 이는 보통 Oracle 및 SQL Server와 같은 고가의 상용 데이터베이스에서만 사용할 수 있는 기능

![[PostgreSQL 장점과 단점-20250108104313769.webp]]
개발자들 사이에서 가장 인기 있는 DBMS

#### 장점
> [!수평 vs 수직 확장성]
>  - <font color="#c0504d">수평 확장성(horizontal scaling)</font>: 수평 확장성은 시스템의 성능과 용량을 늘리기 위해 **여러 노드(서버)를 추가하는 방식**으로, 이를 통해 각 서버 간의 부하를 분산시키고, 고 가용성과 내구성을 향상할 수 있다. 수평 확장은 서버의 수를 늘려 처리 능력을 향상하는 방식이므로, 분산 시스템 및 클러스터링에 적합하다.    수평 확장성이 분산 시스템에 적합하고 확장 가능성이 높지만, 관리 및 구성 복잡성이 증가할 수 있다.
>  - <font color="#c0504d">수직 확장성(vertical scaling)</font>: 수직 확장성은 **기존 서버의 하드웨어 자원을 업그레이드**하여 시스템의 성능과 용량을 늘리는 방식이다. 예를 들어, CPU, RAM, 디스크 공간 등의 업그레이드를 통해 서버의 처리 능력을 향상시키는 것이다. 수직 확장은 한계가 있을 수 있으며, 일정 수준 이상의 업그레이드가 어려울 수 있다.


##### 1) 수직적 확장성 ??
- 단일 노드에서 성능을 최대한 활용하도록 설계되어 있음
	- 복제, 샤딩을 통해 고용량 트래픽을 처리할 수 있음 ??


> [!Sharding]
> 데이터를 여러 서버(노드)로 나누어 저장하고 처리
> ex)   사용자 ID 1~1000번은 서버 A에 저장, 사용자 ID 1001~2000번은 서버 B에 저장
> *tibero는 불가능*
> 
> **Citus**
> PostgreSQL에서 Sharding을 쉽게 구현할 수 있도록 도와주는 확장 도구
> PostgreSQL의 기존 기능을 확장해서, PostgreSQL을 분산형 데이터베이스로 만들어 줌
> 자동 샤딩 처리, 분산 쿼리 처리 등




##### 2) 사용자 정의 데이터 유형 지원
- JSON, XML, H-Store 등 다양한 데이터 유형을 지원
- PostgreSQL은 NoSQL 기능을 강력하게 지원하는 몇 안 되는 관계형 데이터베이스 중 하나이기 때문에 그 많은 데이터 유형을 지원
- 사용자가 직접 데이터 유형을 정의할 수 있음

NoSQL 참고: [[RDBMS vs NoSQL]]



##### 3)  쉽게 통합 가능한 서드파티 도구
> [!서드 파티]
> 데이터베이스 시스템(DBMS) 자체의 기본 기능이나 구성 요소가 아닌, 외부에서 제공되거나 개발된 소프트웨어, 라이브러리, 도구 등을 의미
> 
> Oracle, Tibero도 지원함

기본 PostgreSQL 설치 위에 플러그인처럼 추가하여 사용할 수 있음. > 매우 유연, 강력(확장성(Extension)을 중시하는 구조로 설계됨)

**E.G.**
- ClusterControl: 데이터베이스 관리, 모니터링, 확장.
- pgBackRest: 백업 및 복구 시스템.
- DB Data Directive: - 데이터 비교 및 동기화.
- Citus: 데이터베이스의 수평적 확장을 가능하게 하는 확장.
- pg_stat_statements: 쿼리 성능 분석 도구.
- 
**사용 방법**
```SQL
CREATE EXTENSION pg_stat_statements;
```

##### 4) 오픈소스 및 커뮤니티 중심 지원

##### 5) 그 외
- LAMP 스택 옵션으로 웹사이트와 웹 애플리케이션을 실행
- WAL(미리 쓰기 로그)로 데이터베이스의 내결함성 향상
- 지리적 개체를 지원하므로 위치 기반 서비스 및 지리 정보 시스템을 위한 지리 공간 데이터 저장소로 사용 가능
- 사용하기 쉽기 때문에 많은 교육이 필요하지 않음
- 임베디드 및 엔터프라이즈에서 PostgreSQL을 사용할 때 간편한 유지 및 관리


결론, Tibero / Oracle 와 다르거나 성능이 좋은거
- 다양한 데이터 타입
- 확장 기능 (커스터마이징, 기능 추가)
- **스트리밍 복제(Streaming Replication)** 기능으로 마스터-슬레이브 구조를 구현 (Oracle의 Data Guard 같은..)

| 특징                     | PostgreSQL 스트리밍 복제                                   | Oracle Data Guard                 |
| ---------------------- | ---------------------------------------------------- | --------------------------------- |
| **복제 방식**              | WAL(Log) 스트리밍을 통해 실시간 복제                             | Redo 로그를 사용한 실시간 복제               |
| **구성 방식**              | 마스터-슬레이브 구조                                          | Primary-Standby 구조                |
| **읽기 가능한 복제본**         | 슬레이브에서 읽기(SELECT) 가능 (`hot_standby = on`)            | Active Data Guard에서 읽기 가능         |
| **자동 장애 조치(Failover)** | 수동으로 슬레이브 승격 (도구 사용 가능, 예: Patroni)                  | Fast-Start Failover로 자동 승격 가능     |
| **복제본 동기화 방식**         | **동기식 또는 비동기식** 선택 가능                                | **동기식, 비동기식, 혼합(최적화)** 지원         |
| **복제본 수**              | 여러 슬레이브 노드 지원                                        | 최대 30개의 Standby 지원                |
| **관리 도구**              | 기본 PostgreSQL 설정 + 외부 도구 (예: `pgpool-II`, `Patroni`) | Data Guard Broker                 |
| **라이선스**               | 오픈소스, 무료                                             | Oracle Enterprise Edition + 추가 비용 |
- 샤딩 ? (Tibero 없음?)