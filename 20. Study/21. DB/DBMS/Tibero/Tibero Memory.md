---
sticker: emoji//1f976
---
cpu가 데이터를 처리하기 위해서는 디스크에 있는 데이터를 사용하기 위해서는 데이터를 메모리에 올려야함. 그런 와중에 자주 사용하는 데이터나 실행계획을 캐싱해서 성능을 높일 수 있음
### 메모리 구조
![[Tibero Memory-20241106172421298.webp]]

#### 공유 메모리 TSM (Tibero Shared Memory)
모든 프로세스, 세션이 공유함. 한 인스턴스 내에 한개임.(인스턴스 시작될 때 생성됨)
크기 설정은 *TOTAL_SHM_SIZE*로 설정. (최대 16TB)
- 성능과 관련된 부분
##### databse buffer cache
- <span style="background:#fff88f">최근에 사용된 데이터 블록을 저장하는 메모리 영역</span>
	- 테이블과 인덱스의 데이터
	- 디스크에서 필요한 데이터를 메모리로 읽어올 때 데이터 블록 단위로 올려놓음 (데이터 블록: 데이터가 저장되고 관리되는 최소 단위)
- 크기는 *DB_CACHE_SIZE*로 설정. (권장 크기는 싱글 인스턴스는 *TOTAL_SHM_SIZE* 의 2/3, TAC 인스턴스는 1/2)
- TSM 영역이기 때문에 A 블록을 올려 놓으면 다른 애도 이 A를 씀! 없으면 올려서 쓰고~

##### redo log buffer
- 작업 내용을 저장하는 메모리 영역. **Redo Log**!
- 기본크기:*10MB*, *LOG_BUFFER* 
- 순서. DML(insert, update, delete) 작업 수행 > <span style="background:#fff88f">변경사항 Redo Log Buffer에 기록</span> > LGWR(백그운드 프로세스 Log Writer)가 주기적으로 디스크에 저장
	- 어떤시점? 트랜잭션 커밋 시점, Redo Log Buffer가 일정량 차올랐을 때, 체크포인트가 발생할 때

##### Shared Cache - PP Cache (Physical Plan)
최근에 사용한 <font color="#c0504d">SQL의 명령문</font>, <font color="#c0504d">구문 분석(파싱) 정보</font>, <font color="#c0504d">실행계획</font>정보를 저장한다
PP Cache에 저장된 SQL에 저장된 파싱정보 재사용 > *소프트 파싱*
PP Cache에 존재하지 않은 경우, 해당 SQL을 다시 파싱하여 실행계획 생성 > *하드 파싱*
(파싱: DB가 SQL을 이해하고 분석해서 실행계획을 만드는 전체 과정)


##### Shared Cache - DD Cache
최근에 수행한 SQL이 사용한 <span style="background:#fff88f">오브젝트(테이블, 컬럼, 사용자, 권한) 의 Data Dictionary 정보</span>를 저장
- SQL을 파싱하고 실행 계획을 생성하는 과정에서 필요한 메타데이터를 저장
- 데이터베이스 내의 객체 정보(테이블, 인덱스, 사용자 권한 등)를 담고 있는 일종의 참조 정보 집합
- <font color="#c0504d">실제 데이터가 아닌 실제 데이터를 빠르게 참조할 수 있도록 하는 메타데이터</font>!

##### System Area
전역변수 공간, Worker Thread 관리 공간, 세션정보 관리 공간으로 구성됨

#### 독립적인 메모리 PGA (Process Global Area)
각 프로세스마다 개별적으로 존재. (프로세스 내의 스레드끼리는 하나의 PGA를 공유.)
##### Process Memory 
(크기 고정)
- 프로세스가 동작하는데 필요한 공간으로 사용
##### SQL Info Area 
(크기 고정)
- 커서 정보 영역. SQL문 처리하는데 사용
##### Session Info Area 
(크기 고정)
- 로그온 및 세션 관련 정보가 저장됨
##### SQL Work Area 
- SQL 수행 시 Sorting, Merge 조인, Hash 조인의 작업 공간으로 필요 시 할당되어 사용되는 확장 가능
- 위와 같이 사용되는데 크기가 충분하지 않으면 Temp tablespace를 사용하여 성능이 저하됨


![[Tibero Memory-20241106182639745.webp]]TSM + PGA = Memory Target 
정의 파라미터 : MEMORY_TARGET, TOTAL_SHM_SIZE 