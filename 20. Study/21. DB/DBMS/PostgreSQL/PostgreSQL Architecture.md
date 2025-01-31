- [[#아키텍처 주요 구성 요소|아키텍처 주요 구성 요소]]
	- [[#아키텍처 주요 구성 요소#Shared Memory|Shared Memory]]
	- [[#아키텍처 주요 구성 요소#Process 유형|Process 유형]]
	- [[#아키텍처 주요 구성 요소#데이터베이스 구조|데이터베이스 구조]]
	- [[#아키텍처 주요 구성 요소#Vacuum|Vacuum]]


![[PostgreSQL Architecture-20250108102153425.webp]]
##### 아키텍처 주요 구성 요소
- 공유 메모리
- 백그라운드 프로세스
- 데이터 파일

###### Shared Memory
- Shared Buffer > *databse buffer cache*
- WAL 버퍼 > *redo log buffer*

###### Process 유형
- <span style="background:#fff88f">Postmater (Daemon) 프로세스</span>
  Postmaster Process는 PostgreSQL을 <font color="#c0504d">기동할 때 가장 먼저 시작되는 프로세스</font>이다. 초기 기동 시에 <font color="#c0504d">복구 작업, Shared 메모리 초기화 작업, 백 그라운드 프로세스 구동 작업</font>을 수행한다. 또한, <font color="#c0504d">클라이언트 프로세스의 접속 요청이 있을 때 Backend Process를 생성</font>한다.![[PostgreSQL Architecture-20250109100654147.webp]]
pstree 명령어로 프로세스 간의 관계를 확인하면, <font color="#c0504d">Postmaster 프로세스가 모든 프로세스의 부모 프로세스</font>임.
```Shell
[postgres@localhost pg11]$ pstree -p 20936  
postgres(20936) /home/postgres/pg11/bin/postgres -D /data/pg11  
                ┬─postgres(20937) postgres: logger  
                ├─postgres(20939) postgres: checkpointer  
                ├─postgres(20940) postgres: background writer  
                ├─postgres(20941) postgres: walwriter  
                ├─postgres(20942) postgres: autovacuum launcher  
                ├─postgres(20943) postgres: stats collector  
                └─postgres(20944) postgres: logical replication launcher
```

- Background Process

| Process 명                                        | 수행하는 일                                                                                   |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| logger                                           | 에러 메세지 및 다양한 정보를 로그 파일에 기록 한다.                                                           |
| checkpointer                                     | 체크포인트 발생 시, dirty 버퍼를 파일에 기록한다.                                                          |
| wirter                                           | 주기적으로 dirty 버퍼를 파일에 기록한다.                                                                |
| wal writer                                       | WAL Buffer 내용을 WAL 파일에 기록한다.                                                             |
| <font color="#c0504d">autovacuum launcher</font> | Vacuum이 필요한 시점에 autovacuum worker를 fork한다.                                               |
| archiver                                         | Achive log 모드일 때, WAL 파일을 지정된 디렉토리에 복사한다.                                                |
| stats collector                                  | 세션 수행 정보 (pg_stat_activity)와 테이블 사용 통계 정보 (pg_stat_all_tables) 와 같은 DBMS 사용 통계 정보를 수집한다. |

Autovacuum laucher를 제외한 나머지 프로세스들은 Oracle에도 있는 것들

- <span style="background:#fff88f">Backend Process</span>
  *Oracle의 Backend Process, Tiebro의 Worker Thread*
  Backend Process의 최대 개수는 max_connections 파라미터로 설정하며, 기본 설정 값은 100이다. Backend Process는<font color="#c0504d"> 사용자 프로세스 쿼리 요청을 수행한 후, 결과를 전송하는 역할</font>을 수행한다. 쿼리 수행에 필요한 몇 가지 메모리 구조가 필요한데 이것을 통칭해서 로컬메모리(*PGA*)라고 한다. 로컬메모리와 관련된 주요 파라미터는 다음과 같다.
  
    . <font color="#4f81bd">work_mem 파라미터</font>
  정렬 작업, Bitmap 작업, Hash조인과 Merge 조인 작업 시에 사용되는 공간이다. 기본 설정값은 4MB이다.
    . <font color="#4f81bd">  maintenance_work_mem 파라미터</font>
  Vacuum 및 Create Index 작업 시에 사용되는 공간이다. 기본 설정값은 64MB이다.
    . <font color="#4f81bd">temp_buffers 파라미터</font>
  Temporary 테이블을 저장하기 위한 공간이다. 기본 설정값은 8MB이다.
  
- <span style="background:#fff88f">Client Process</span>


###### 데이터베이스 구조

***Database 관련 사항***
PostgreSQL은 여러 데이터베이스로 구성됨. > **Database Cluster**
**Database Cluster**는 **하나의 서버**이며, 그 안에 여러 **데이터베이스**가 **논리적으로 분리**되어 있는 구조

- `initdb()`
PostgreSQL 설치 후, 데이터베이스 클러스터를 초기화 명령어.
수행시  <font color="#9bbb59">template0</font>, <font color="#9bbb59">template1</font>, <font color="#9bbb59">postgres</font> 데이터베이스가 생성됨.
> [!템플릿 데이터베이스란?]
> 새로운 데이터베이스를 생성할 때 사용되는 템플릿
> 예를 들어, 매번 새 데이터베이스에 동일한 설정이나 테이블이 필요하면 템플릿 사용.

![[PostgreSQL Architecture-20250113102111852.webp]]
- 템플릿
	- template0
	  초기 상태 그대로의 템플릿 데이터베이스
	  **수정이 불가능**한 상태로 유지
	  사용자가 직접 데이터를 추가하거나 변경할 수 없음
	- template1
	  사용자가 새로운 데이터베이스를 생성할 때 **기본적으로 복제되는 템플릿**
	  initdb() 수행 직후에 template0 과 template1 데이터베이스의 테이블 목록은 같다. 단 template1의 데이터베이스에는 사용자가 필요한 오브젝트를 생성할 수 있음
	  예를 들어, 매번 새 데이터베이스에 동일한 설정이나 테이블이 필요하다면, **template1**에 미리 만들어 두기.

- 데이터베이스 생성하기

```SQL
CREATE DATABASE mydb;
```
새 데이터베이스를 생성할 때, 기본적으로 **template1**을 복제해서 데이터베이스가 만들어짐

장점
- 매번 새 데이터베이스를 생성할 때 공통적으로 필요한 테이블, 함수, 설정 등을 다시 만들 필요가 없음
- **template1**에 필요한 오브젝트를 미리 정의해 두면 모든 새 데이터베이스가 동일한 초기 상태를 가짐

> [!왜 여러개의 Database를 두는지?]
> PostgreSQL은 **데이터베이스 단위로 분리**하여 데이터를 관리하는 방식이며, 데이터베이스 간 독립성을 중시.
> *Tibero에서의 스키마 같은 역할*인데, PostgreSQL 내 Database에도 스키마도 있음
> 글고 *Tibero내에서 다른 스키마끼리도 join 가능*하지만,  PostgreSQL 내 타 데이터베이끼리는 Join 불가 (완전 독립 그잡채)
> 
> 🐤내 생각인데 PostgreSQL 은 좀 더 유연하게 쓰여서? 1개의 클러스터에 여러 Database를 허용..? 말로 쓰니 이상하네..

![[PostgreSQL Architecture-20250110164051341.webp]]


***Tablespace 관련 사항***
1. initdb() 수행 직후에 pg_default, pg_global Tablespace 가 생성된다.
2. 테이블 생성 시에 Tablespace를 지정하지 않으면 pg_default Tablespace 에 저장된다.
3. 데이터베이스 클러스터 레벨에서 관리되는 테이블은 pg_global Tablespace 에 저장된다.
4. pg_default Tablespace의 물리적 위치는 $PGDATA\base 이다.
5. pg_global Tablespace의 물리적 위치는 $PGDATA\global 이다.
6. 1개의 Tablespace를 여러 개의 데이터베이스가 사용할 수 있다. 이때, Tablespace 디렉토리 내에 Database별 서브 디렉토리가 생성된다. 이때 서브 디렉토리의 이름은 Database의 OID가 된다.
7. 사용자 Tablespace를 생성하면 $PGDATA\tblspc 디렉토리에 사용자 Tablespace와 관련된 심볼릭 링크가 생성된다.

**pg_global 테이블스페이스**
pg_global 테이블스페이스는 '데이터베이스 클러스터' 레벨에서 관리 해야 할 데이터들을 저장하는 용도의 테이블스페이스
예를 들어, pg_database 테이블과 같은 유형의 테이블들은 어떤 데이터베이스에서 접속해서 조회하더라도 동일한 정보를 제공
![[PostgreSQL Architecture-20250113102704984.webp]]

***Table 관련 사항***
1. 테이블 별로 3개의 파일이 존재한다.
2. 1개는 테이블 데이터를 저장하기 위한 파일이다. 파일명은 테이블의 OID이다.
3. 1개는 테이블 여유 공간을 관리하기 위한 파일이다. 파일명은 OID_fms 이다.
4. 1개는 테이블 블록의 visibility를 관리하기 위한 파일이다. 파일명은 OID_vm 이다.
5. 인덱스는 vm 파일이 없다. 즉, OID, OID_fsm 2개의 파일로 구성된다.

Database, Template, Tablespace, Table 이론 내용 실제로 그렇게 동작하는지 실습, 설명은 [해당링크](https://bitnine.tistory.com/549) 참고

물리적 구성에서
테이블스페이스 하위에 데이터베이스(템플릿 포함) 존재함.
![[PostgreSQL Architecture-20250113102550245.webp]]


###### Vacuum 
[[Vacuum]]
