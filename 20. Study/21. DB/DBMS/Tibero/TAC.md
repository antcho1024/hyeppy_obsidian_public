---
sticker: emoji//1f5e8-fe0f
---
해당 개념에 대해서는 [[RAC (Real Application Clusters)]]에도 정리해두었다 참고!

- [[#TAC(Tibero Active Cluster)|TAC(Tibero Active Cluster)]]
	- [[#TAC(Tibero Active Cluster)#공유 디스크 종류|공유 디스크 종류]]
	- [[#TAC(Tibero Active Cluster)#TAC 구성모듈|TAC 구성모듈]]
		- [[#TAC 구성모듈#CWS(Cluster Wait-Lock Service)|CWS(Cluster Wait-Lock Service)]]
		- [[#TAC 구성모듈#GWA(Global Wait-Lock Adapter)|GWA(Global Wait-Lock Adapter)]]
		- [[#TAC 구성모듈#CCC(Cluster Cache Control)|CCC(Cluster Cache Control)]]
		- [[#TAC 구성모듈#GCA (Global Cache Adapter)|GCA (Global Cache Adapter)]]
		- [[#TAC 구성모듈#MTC(MessageTransmission Control)|MTC(MessageTransmission Control)]]
		- [[#TAC 구성모듈#INC(Inter-Node Communication)|INC(Inter-Node Communication)]]
		- [[#TAC 구성모듈#NMS(Node Membership Service)|NMS(Node Membership Service)]]
	- [[#TAC(Tibero Active Cluster)#TAC 프로세스|TAC 프로세스]]
		- [[#TAC 프로세스#Active Cluster Control Thread|Active Cluster Control Thread]]
		- [[#TAC 프로세스#Diagnostic Thread|Diagnostic Thread]]
		- [[#TAC 프로세스#Cluster Message Processor Thread|Cluster Message Processor Thread]]
		- [[#TAC 프로세스#Asynchronous Thread|Asynchronous Thread]]
		- [[#TAC 프로세스#Garbage Collector|Garbage Collector]]
		- [[#TAC 프로세스#Reconfigurator|Reconfigurator]]
		- [[#TAC 프로세스#Node Manager|Node Manager]]
	- [[#TAC(Tibero Active Cluster)#TAC 모니터링|TAC 모니터링]]
- [[#TBCM|TBCM]]


![[TAC-20240830111326044.webp]]

## TAC(Tibero Active Cluster)
여러 대의 노드(Instance)가 공유 디스크를 기반으로 데이터 파일을 공유

실행 중인 모든 인스턴스는 <font color="#c0504d">공유된 데이터에 대해 트랜잭션을 수행</font>하며, 공유된 데이터에 대한 접근은 데이터의 일관성과 정합성 유지를 위해 상호 통제 하에 이뤄진다. 
변경된 <span style="background:#fff88f">데이터 블록</span>은 노드 사이를 연결하는 <font color="#c0504d">고속 사설망 (nterconec)을 통해 주고받기 때문에 하나의 Bufer Cache를 사용하는 것처럼 동작</font>한다. 
운영 중에 한 노드가 멈추더라도 동작 중인 <font color="#c0504d">다른 노드들이 서비스를 지속적으로 처리</font>한다. 
큰 업무를 여러 작은 업무로 나누어 여러 노드에 분산하여 수행할 수 있기 때문에 <font color="#c0504d">업무 처리 시간이 단축</font>된다.

### 공유 디스크 종류
- RAW Device
- Shared Cluster File System
- TAS(Tibero Active Storage) > (3rd Party Cluster S/W가 필요 없음) [[TAS]]
  3rd Party Cluster S/W: 다른 회사에서 만든 클러스터링 소프트웨어로, 여러 서버(컴퓨터)를 하나처럼 묶어서 서버 중 하나가 고장 나도 서비스가 멈추지 않게 해줌
 
### TAC 구성모듈
<span style="background:#fdbfff">이 아래 모듈들이 CM 안에 있는건지 그림만 그렇게 나온건지 확인 필요</span>
#### CWS(Cluster Wait-Lock Service)
- 기존 **Wiock(Wait Lock)이 클러스터 내에서 동작할 수 있도록** 구현된 모듈
	*Wlock(Wait Lock)*: 데이터나 리소스를 여러 서버에서 동시에 사용하려 할 때 발생하는 잠금 기능. 한 서버가 특정 데이터를 사용 중이라면, 다른 서버는 기다리게(Wait) 해서 충돌을 방지
- DLM(Distributed Lock Manage)이 내장되어 있음. 
	*DLM(Distributed Lock Manager)*: **클러스터 전체에서 잠금을 관리**. 여러 서버가 데이터에 접근할 때, **누가 먼저 사용할지 결정해 주고, 다른 서버들은 기다림**.
- Wlock은 GWA(GIobal Walt Lock Adapter)를 통해 CNS(중앙관리시스템)에 접근하여 데이터 접근과 잠금을 요청하거나 기다림.
- 관련 Thread
   WATH: 잠금을 기다리는 서버들을 모니터링하여 상태를 관리
   WLGC: 잠금을 해제하거나 갱신하는 역할
   WPCF: 잠금 충돌을 관리하고 조정

#### GWA(Global Wait-Lock Adapter)
Wlock 이 CWS를 사용할 수 있도록 인터페이스 역할을 하는 모듈. 

#### CCC(Cluster Cache Control)
- Buffer Cache의 데이터 블록을 클러스터 환경에서 안전하고 효율적으로 사용할 수 있도록 관리하는 모듈
	*클러스터 환경에서는 여러 노드가 동일한 데이터를 공유하거나 동시에 접근할 수 있으므로, 데이터 **일관성을 유지**하기 위해 CCC가 중요한 역할을 함*
	- 예를 들어, 한 노드가 데이터를 수정하면, 해당 데이터를 다른 노드에서 읽을 때 최신 상태로 제공하기 위해 CCC 간의 통신, 협력함
- DLM이 내장
  데이터 블록에 여러 노드가 접근할 때 잠금(lock)을 걸어 충돌을 방지함
- Buffer Cache에서는 Global Cache Adapter(GCA)를 통해 CCC에 접근
- 관련된 Thread
   CATH: 클러스터 내에서 데이터 접근을 요청하는 트랜잭션을 모니터링하고 관리
   CLGC: 데이터 블록을 캐시에 가져오거나 갱신하는 작업을 수행
   CRCF: 데이터 접근 시 발생할 수 있는 충돌을 감지하고 조정하는 역할

#### GCA (Global Cache Adapter)
- Buffer Cache에서 CCC를 사용할 수 있도록 인터페이스 역할을 하는 모듈
- CCC의 Lock-Down Event에 맞춰 데이터 블록이나 Redo 로그를 디스크에 저장하는 기능
  ***CCC의 Lock-Down Event**란, 데이터 블록이나 Redo 로그가 여러 노드에서 동시에 접근되지 않도록 잠금(lock)을 걸어야 하는 상황*
- **DBWR가 Global Write를 요청**하거나 **CCC에서 DBWVR에게 Block Wite을 요청**하는 *인터페이스*를 제공
	- **DBWR가 Global Write를 요청**
		- DBWR: <font color="#c0504d">실제로 기록 작업</font> 수행. (Database Buffer Cache > Disk에 기록) > 실제 기록은 얘가 하지만, 데이터 일관성 유지, 데이터 안정성을 위해서 기록작업 <font color="#c0504d">요청을 하고 해야함</font>!
		- GCA: DBWR가 요청한 Global Write를 CCC로 전달
		- CCC: GCA로부터 DBWR의 요청을 받은 CCC는 **디스크 기록 요청을 승인**하고, 데이터가 일관성 있게 관리되도록 **잠금 및 데이터 동기화**를 수행 (DBWR가 안전하게 데이터를 기록할 수 있도록 환경을 조성)
	- **CCC에서 DBWVR에게 Block Wite을 요청**
		- CCC에서 DBWR아 너가 좀 데이터 써줘! 왜? 일관성 유지를 위해서지~
		- 일관성을 위해 <font color="#c0504d">특정 데이터 블록을 디스크에 기록할 필요가 있을 때</font>
			- 예를 들어, 데이터가 수정되었고 다른 노드가 이 데이터에 접근하려고 할 때, CCC는 해당 데이터 블록을 디스크에 안전하게 기록(Block Write)하도록 요청하여 최신 상태를 보장하려함.
		- DBWR아 데이터 써줘~ > GCA 전달 > DBWR 오케이쓸게~ 썼음 > CCC의 일관성 유지작업
			- CCC의 일관성 유지 작업: 상태를 확인하고, 다른 노드가 해당 데이터 블록을 읽거나 수정할 수 있도록 **잠금을 해제**하거나 **접근을 허용**하는 등 데이터 일관성을 유지하는 후속 작업을 수행
- CCC에서는 GCA를 통해 아래 데이터 주고 받음
	- *CR Block*: 특정 시점의 일관된 데이터 블록으로, 읽기 작업에서 읽기 일관성을 유지하기 위해 사용됨 
		- ex) 트랜잭션이 특정 시점의 데이터를 읽어야 할 때, 다른 트랜잭션이 데이터를 수정하더라도 그 시점의 데이터 상태를 유지해야 하기때문에, 이때 CR Block이 제공되어 특정 시점의 스냅샷을 읽는 것처럼 작동하여, 일관성 있는 데이터를 조회할 수 있게 함
		  👉 CCC는 **클러스터 환경에서 CR Block을 각 노드에 전달**하여, 여러 노드가 동일한 시점의 데이터를 읽을 수 있도록 함
	- *Global Dirty Block*: 다른 노드에서 수정되었지만 아직 디스크에 기록되지 않은 데이터 블록으로, 클러스터 간 동기화를 위해 필요
	- *Current Block*: 최신 상태의 데이터 블록으로, 쓰기 작업을 위해 사용됨

#### MTC(MessageTransmission Control)
- 노드간 통신 <font color="#c0504d">관리</font> 모듈
- **MTC는 클러스터의 서버 간 메시지를 안정적으로 주고받도록 돕는 관리자 역할**입니다. 메시지 손실이나 순서 오류가 발생하지 않도록 관리하고, 다른 모듈들이 안전하게 통신할 수 있게 지원

	노드 간의 통신 메시지의 손실과 Out-Of-Order 문제를 해결하는 모듈이다. 문제를 해결하기 위해 Retransmission Queue와 Out-Of-Order Message Queue를 관리한다. GMC(General Message Control)을 제공하여 CWS/CCC 이외의 모듈에서 노드 간의 통신이 안전하게 이루 어지도록 보장한다. 현재 IIC(nter-=nstance Cal), DDD(Distributed Deadlock Detection. AWM(Automatic Workload Management)에서 노드 간의 통신을 위해 GMC를 사용하고 있다.
#### INC(Inter-Node Communication)
노드 간의 **네트워크 연결을 담당**하는 모듈이다.
INC를 사용하는 사용자에게 네트워크 토폴로 지와 프로토콜을 투명하게 제공하며 TCP와 UDP 등의 프로토콜을 지원한다.
#### NMS(Node Membership Service)
<span style="background:#fdbfff">노드마다 있는지 / 클러스터 단위에 있는지 확인 필요</span>
**NMS**는 클러스터에 포함된 **서버(노드)들의 정보**를 관리하는 모듈입니다. 각 서버의 ID, IP 주소, 포트 번호, 서버의 가중치(서버가 얼마나 바쁜지)를 관리

CM으로부터 전달받은 노드 정보(노드 ID, IP Address, Port, Incarnation Number)와 노드 가중치(노드 Workload)를 관리하는 모듈
주요 기능
- 노드 멤버십의 조회: 현재 클러스터에 어떤 서버들이 있는지 확인
- 노드 추가, 삭제 기능: 새로운 서버가 추가되거나, 기존 서버가 제거될 때 정보를 업데이트
- 관련된 Thread: NMGR: 서버들의 멤버십 정보를 지속적으로 확인하고 관리하는 스레드


📌 위 모듈들 중, 일관성을 관리하는 모듈은: **CWS, CCC**인데 기능이 헷갈린다면?
👇👇👇👇

**CWS (Cluster Wait-Lock Service)**
- 주요 역할: 잠금 관리(Locking).
- 설명: 클러스터 내에서 *여러 노드가 데이터에 접근할 때 충돌을 방지*하기 위해 잠금을 관리 역할
- 사용 목적: *노드 간의 데이터 일관성을 유지*하기 위해 *잠금(lock)을 통해 접근을 제어*합니다.
- 어떤 상황에서 사용되나요?
    - 예를 들어, 하나의 노드가 특정 데이터 블록을 수정하고 있을 때, 다른 노드가 동시에 접근하여 데이터를 읽거나 수정하려고 하면 충돌이 발생할 수 있습니다.
    - CWS는 이런 상황에서 잠금을 걸어 다른 노드가 기다리도록(Wait) 하여 데이터 일관성을 유지합니다.

**CCC (Cluster Cache Control)**
- 주요 역할: 캐시 일관성 관리(Cache Coherency).
- 설명: CCC는 노드 간 데이터 캐시의 일관성을 유지하는 역할을 합니다. *여러 노드가 같은 데이터 블록을 캐시에 보관하고 있을 때, 데이터가 변경되면 이를 각 노드의 캐시에도 반영해 동일한 최신 데이터를 볼 수 있도록 관리*합니다.
- 사용 목적: *노드마다 캐시 된 데이터가 최신 상태로 유지되도록 하여, 캐시 일관성을 보장*합니다.
- 어떤 상황에서 사용되나요?
    - 예를 들어, 한 노드에서 데이터 블록이 수정되었을 때, 다른 노드의 캐시에서도 해당 데이터 블록이 최신 상태로 반영되어야 합니다.
    - CCC는 Global Dirty Block 등의 개념을 사용해 변경된 데이터가 다른 노드에서도 반영될 수 있도록 관리합니다.


### TAC 프로세스
기존 티베로 + <span style="background:#fff88f">ACSD(Active Cluster Service Daemon) Process</span>

**ACSD**
ACSD Process  :  10개의 Thread(ACCT, NMGR, DIAG, WRCE, CRCF, WLGG, CLGC, WATH, CATH,
CMPT) 존재. -> 각 Thread는 *7개 그룹*으로 분류됨

👇👇👇👇
#### Active Cluster Control Thread
1️⃣ ACCT Thread
  1. ACCT는 <font color="#c0504d">클러스터 간의 메시지 통신을 담당</font>하는 Thread이다.
     보내고 받고!  자신의 노드에서 외부로 전송할 메시지를 수집하여, 원격 노드로 전달합니다. 이를 통해 각 노드 간의 상태를 동기화하고, 데이터 일관성을 유지!
  2. CWS/CCC의 요청 처리
     클러스터 내의 원격 노드에서 CWS/CCC의 Lock Operation과 Reconfiguration 요청을 받아 CMPT Thread에게 전달하거나, 자기 노드의 세션에서 전송해야 할 메세지를 받아 원격 노드로 전송하는 Thread
  3. ACSD Process의 메인 Thread로서 나머지 Thread를 감독하는 역할을 한다.

#### Diagnostic Thread
1️⃣  DIAG Thread
  클러스터에서 서버들 간 요청을 주고받다가 문제가 생기면, 그 상황을 기록(덤프)하는 역할

#### Cluster Message Processor Thread
1️⃣  CMPT Thread
  CMPT는 <font color="#c0504d">다른 노드에서 보낸 메시지를 처리</font>하는 Thread 
  CMPT는 ACF_CMPT_CNT 초기화 파라미터에 설정된 값만큼 공통 풀을 생성한다. 
  CMPT는 ACCT로부터 메시지를 받아 다음과 같은 일을 수행한다
	- CR Block Request를 받으면 주어진 스냅샷에 해당하는 CR Block을 생성하고 요청자에게 전송한다.
	- Current Block Request를 받으면 Local Block Cache에 존재하는 Current Block을 읽어서 요청자에게 전송한다.
	- Global Write Request를 받으면 해당 데이터 블록이 Dirty이면 BLKW에게 디스크 쓰기를 지시한다.
	- MTC의 Iter-Instance Call Request를 받아 처리한다. 
	- MLD(Master Lookup Directory) Lookup/Remove Request를 받아 처리한다.

#### Asynchronous Thread
1️⃣  WATH Thread
2️⃣  CATH Thread
WATH, CATH는 CWS/CCC에서 세션을 담당하는 Worker Thread가 처리해야 할 비동기적(즉, 나중에 처리해도 되는) 작업을 대신 수행 하는 Thread이다.
🌳특징
- CATH는 BAST를 맞거나 스스로 잠금을 설정할 때 캐시된 Lock Mode를 Downgrade하기 전에 BLKW로부터 Disk Write Notification이나 LOGW로부터 Log Flush를 기다린 후 Master에게 Lock Downgrade를 통보한다.
- BLKW가 Master로부터 받은 Global Write Request을 처리한 후 CATH에게 알려주면 CATH는 Master에게 Write Done Notify를 보낸다.
- WATH와 CATH는 Shadow Resource Block를 Reclaim하기 전에 Master에게 MC Lock에 대한 제거 요청을 보낸 후 이에 대한 응답을 받아서 처리한다.

#### Garbage Collector
1️⃣  WLGC Thread
2️⃣  CLGC Thread
주기적으로 <font color="#c0504d">Lock Resource을 관리하고 타임아웃을 체크하여 불필요한 리소스 정리하는 Thread</font>

🌳역할
- 잠금 리소스 관리 및 타임아웃 체크: 잠금 리소스를 주기적으로 관리하고, 타임아웃이 발생한 잠금 요청이 있는지 확인합니다.
- Deadlock 감지(DDD): 타임아웃된 잠금 요청을 확인할 때, 교착 상태(Deadlock)가 발견되면 DDD를 통해 문제를 해결합니다.
- MTC 메시지 재전송: 메시지가 손실되거나 지연된 경우, MTC의 Retransmission Queue에 쌓인 메시지를 다시 전송하여 통신 문제를 해결합니다.
- TSN 동기화: WIGC(Thread)는 TSN(Tibero System Number)를 주기적으로 동기화하여 클러스터 내에서 일관된 트랜잭션 번호를 유지합니다.
- 리소스 확보(Resource Block Reclaiming): 잠금 리소스를 위한 메모리가 부족해지면 불필요한 리소스를 제거하여 메모리를 확보합니다.
 
#### Reconfigurator
1️⃣  WRCF Thread
2️⃣  CRCF Thread
NMGR로부터 노드 Join과 Leave Event를 받아 CWS/CCC Lock Remastering/Reconfiguration 실행
🐥 클러스터에 새로운 노드가 참여하거나 떠나갈 경우!
CWS/CCC 재구성! 재할당!

#### Node Manager
1️⃣  NMGR Thread가 있다.
TBCM과 통신하여 노드 Join과 Leave Event를 받아 처리하며 노드 멤버십을 관리한다. 또한 WRCF와 CRCF에 의해 수행되는 CWS/CCC Reconfiguration을 통제(Suspend/Resume)한다.
🐥역할
- 노드 멤버십 관리: 클러스터에 어떤 노드가 포함되어 있는지 관리하며, 노드가 들어오거나 나갈 때 상태를 업데이트합니다.
- 노드 Join/Leave 이벤트 처리: TBCM과 통신하여 노드가 추가되거나 삭제될 때 이벤트를 받아 처리합니다.
- Reconfigurator 제어: 노드의 상태가 변경될 때 Reconfigurator(WRCF와 CRCF)가 재구성을 수행하도록 통제(Suspend/Resume)합니다.


### TAC 모니터링
방법 1. `$TB_ HOME/scripts/ cm_stat.sh`
cm_stat.sh 스크립트 파일을 통해 노드 및 인스턴스의 상태를 모니터링(TAC에 참여한 노드의 현재 상태, IP 정)

방법 2. **글로벌 뷰(Global View)**
싱글인스턴스에서 V$SESSION 로 세션정보를 조회하듯
GV$SESSION을 사용하면 TAC에 있는 모든 인스턴스의 세션 정보를 통합해서 볼 수 있음
```SQL
SELECT * FROM GV$SESSION;
```

방법3. `cmictl show service` -- name 서비스이름
CM이 기동된 이후에는 cmrctl show 명령어로 서비스, DB 등의 리소스에 대한 상태를 확인할 수 있다.


## TBCM 
Tibero Cluster Manager (P.448 참고)
클러스터에 등록된 리소스들에 대한 모니터링과 상태 변화에 따른 필요한 동작들을 수행
같은 클러스터에 속하게 된 CM들 간에는 네트워크, 공유 디스크를 통해 주기적으로 자신이 살아있음을 알리는 Heartbeat를 주고받아 클러스터 멤버십 구성을 확인함

CM에서 클러스터에 등록된 구성요소: 리소스
- File(Cluster File)
- Network(Public IP, Private IP)
- Cluster
- Service
- DB
- AS(Active Storage)
- VIP


**번 외, TAC와 ACID** from GPT
Tibero TAC에서 **ACID 원칙**(Atomicity, Consistency, Isolation, Durability)을 보장하기 위해 존재하는 모듈과 스레드들은 다음과 같이 연결될 수 있습니다:

---

### 1. **Atomicity (원자성)**  
트랜잭션이 **완전히 수행되거나 전혀 수행되지 않는 것**을 보장합니다.
#### 관련 모듈 및 스레드:
- **CWS (Cluster Wait-Lock Service)**:
  - 트랜잭션이 동일 데이터에 접근할 때 잠금을 관리하여 트랜잭션 중단 시 데이터의 불완전 상태를 방지합니다.
  - **WLGC Thread**: 잠금을 해제하거나 갱신하여 트랜잭션 종료 시 잠금을 적절히 정리합니다.
- **DBWR (Database Writer)**:
  - 변경된 데이터를 데이터 파일로 기록하며, 트랜잭션이 완료되지 않은 경우 기록하지 않습니다.

---

### 2. **Consistency (일관성)**  
트랜잭션 수행 전후의 데이터가 항상 **일관된 상태**를 유지해야 합니다.
#### 관련 모듈 및 스레드:
- **CCC (Cluster Cache Control)**:
  - 데이터 블록의 변경 사항을 디스크에 기록하거나 다른 노드의 캐시에 동기화하여 데이터 일관성을 유지합니다.
  - **CRCF Thread**: 데이터 접근 시 충돌을 감지하고 조정하여 데이터 일관성을 보장합니다.
  - **Global Dirty Block** 관리: 변경된 데이터가 다른 노드에서도 최신 상태로 반영되도록 처리합니다.
- **GCA (Global Cache Adapter)**:
  - CCC와 협력하여 데이터 블록과 Redo 로그의 디스크 기록 요청을 처리하며, 일관성을 유지합니다.

---

### 3. **Isolation (고립성)**  
트랜잭션이 **서로 간섭하지 않도록** 격리합니다.
#### 관련 모듈 및 스레드:
- **CWS (Cluster Wait-Lock Service)**:
  - 트랜잭션 간의 동시성 문제를 방지하기 위해 잠금을 관리합니다.
  - **WATH Thread**: 대기 중인 트랜잭션을 모니터링하여 잠금 상태를 효율적으로 관리합니다.
  - **WLGC Thread**: 잠금 리소스를 해제하거나 갱신하여 교착 상태를 방지합니다.
- **MTC (MessageTransmission Control)**:
  - 노드 간 트랜잭션 메시지 손실이나 순서 오류를 방지하여 트랜잭션이 격리된 상태에서 실행되도록 보장합니다.

---

### 4. **Durability (지속성)**  
트랜잭션이 성공적으로 완료되면 **결과가 영구적으로 저장**됩니다.
#### 관련 모듈 및 스레드:
- **DBWR (Database Writer)**:
  - 데이터 변경 사항을 데이터 파일에 안전하게 기록합니다.
- **Redo Log Buffer와 GCA**:
  - 트랜잭션 변경 사항을 **Redo 로그**에 기록하여 장애 발생 시 복구를 지원합니다.
- **Garbage Collector (WLGC, CLGC)**:
  - 불필요한 리소스를 정리하며, TSN 동기화를 통해 트랜잭션의 일관된 상태를 유지합니다.

---

### 요약 연결 구조

1. **CWS**:
   - 잠금 관리 → Atomicity, Isolation 보장.
2. **CCC**:
   - 데이터 동기화 및 잠금 → Consistency 보장.
3. **GCA**:
   - Redo 로그 및 데이터 블록 기록 → Durability, Consistency 보장.
4. **DBWR**:
   - 디스크 기록 → Durability 보장.
5. **MTC**:
   - 노드 간 메시지 동기화 → Isolation, Consistency 보장.
6. **Garbage Collector (WLGC, CLGC)**:
   - 자원 정리 및 TSN 동기화 → Consistency, Durability 보장.

---

TAC의 설계는 **ACID 원칙을 유지하며 클러스터 환경에서 데이터의 일관성과 가용성을 극대화**하는 데 초점이 맞춰져 있습니다.