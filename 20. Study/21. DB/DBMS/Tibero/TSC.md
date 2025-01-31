---
sticker: emoji//1f921
---
**목차**
- [[#TSC (Tibero Standby Cluster)|TSC (Tibero Standby Cluster)]]
			- [[#LNW, LNR, SMR은 모두 RCWP(Recovery Worker Process)에 속해 있음|LNW, LNR, SMR은 모두 RCWP(Recovery Worker Process)에 속해 있음]]
			- [[#로그 전송 방식|로그 전송 방식]]
			- [[#제약 사항|제약 사항]]
	- [[#TSC (Tibero Standby Cluster)#Primary DB|Primary DB]]
			- [[#동작 모드 (REPLICATION_MODE)|동작 모드 (REPLICATION_MODE)]]
			- [[#기동시키기|기동시키기]]
	- [[#TSC (Tibero Standby Cluster)#Standby DB|Standby DB]]
			- [[#기동시키기|기동시키기]]
			- [[#Read Only 모드|Read Only 모드]]
			- [[#TAC-TSC|TAC-TSC]]
			- [[#Standby Redo Log Group|Standby Redo Log Group]]
			- [[#Cascade TSC|Cascade TSC]]
			- [[#Snapshot Standby|Snapshot Standby]]
	- [[#TSC (Tibero Standby Cluster)#TSC 역할 전환|TSC 역할 전환]]

---
##  TSC (Tibero Standby Cluster)
Primary 데이터베이스의 변경 사항을 실시간으로 Standby 데이터베이스에 복제
![[TSC-20240819164206190.webp]]
- 데이터 전송 : Primary DB -> Standby DB
- 물리적으로 떨어진 공간에서도 가능
- 데이터 전송 단위 : Transaction 

##### LNW, LNR, SMR은 모두 RCWP(Recovery Worker Process)에 속해 있음
🧵LNW(Log Network Writer) 
- 데이터 전송  Thread (Primary DB -> Standby DB)
- Primary DB에서 기동 되는 Thread
- <font color="#4f81bd">Standby를 9개까지 구성이 가능</font>하며, 각 Standby 마다 LNW가 하나씩 기동
🧵LNR(Log Network Reader)
- 데이터 받음
- Standby DB에서 기동 되는 Thread
- 기본 DB에서의 LGWR 역할 수행
> [!LGWR 역할]
> redo log를 Log 파일에 쓰는 Thread
> Buffer Cache, Redo Log Buffer (MEMORY) ➡️ DISK 에 저장
> 참고 :  [[CheckPoint]]

🧵SMR(Standby Managed Recovery)
- Standby DB에서 기동 되는 Thread
- Redo 로그 파일이나 Archive 로그 파일에서 <font color="#c0504d">Redo 로그를 읽어 Standby를 복구</font>하는 Thread
> [!"복구한다"의 의미]
> Redo Log 내용을 Database에 적용한다.
>  Standby DB를 Primary DB와 같은 상태로 최신화 한다.
##### 로그 전송 방식 
*Primary에서 설정, Standby DB마다 다르게 설정 가능*
🚠 LGWR SYNC
Primary에서 LGWR가 Redo 로그를 <font color="#4f81bd">Redo 로그 파일에 쓸 때 LNW가 Redo 로그를 Standby로 보냄</font>.
- 가장 자주 Redo 로그를 보냄 
 -> Standby로 데이터가 보호 확률 ⬆️
 -> Standby의 성능이 좋지 못하면 Primary의 성능에 영향을 줄 수 있음
- 따라서 Standby를 Primary와 비슷한 수준으로 구축할 것을 권함.
- Standby의 성능이 좋지 못하면 Primary의 온라인 Redo 로그 파일이나 Archive 로그 파 일에서 Redo 로그를 읽어 전송할 수 있으므로 Primary를 Archivelog 모드로 운영할 것을 권장.

🚠 LGWR ASYNC 
Primary에서 LNW가 <font color="#4f81bd">임의의 시점에 온라인 Redo 로그 파일에서 Redo 로그를 읽어서 Standby로 전 송</font>. 
기본적으로 온라인 Redo 로그 파일을 읽어서 전송하지만 Standby가 따라오지 못하는 경우 Archive 로그 파일에서 읽을 수도 있으므로 Primary를 Archivelog 모드로 운영할 것을 권장.

🚠 ARCH ASYNC
LOGA가 Archive 로그 파일을 만든 후 LNW에게 알려주면, <font color="#4f81bd">LNW는 Archive 로그 파일을 읽어</font>
<font color="#4f81bd">Standby로 전송</font>한다. Primary는 반드시 Archivelog 모드로 동작해야 한다. 그렇지 않으면 Primary는 정상적으로 기동되지만 Standby는 아무런 동작도 하지 못한다.

> [!Archivelog 모드로 설정 권장]
> 어떤 모드(LGWR SYNC, LGWR ASYNC, ARCH ASYNC)로 설정하더라도 TSC를 안전하게 구성하려면 *Primary를 Archivelog 모드로 설정*하는 것을 권장
> 
>🦞Why?
> ▶️ 1, 2번째 방법으로 하다가 Standby가 못받았어. 그럼 최후수단으로 아카이브 로그 파일을 사용해서 로그 파일을 전송하려고해.
> 근데 Primary가 Archivelog 모드가 아니면 아카이브 로그 파일이 생성되지 않기 떄문에 전송할 수가 없어
>▶️ 3번째 방법은 아카이브 로그 파일에서 전송하기인데 저거로 설정하면,, 정상기동은 되지만 아것도 전송X임.

#####  제약 사항
- Primary DB, Standby DB 둘의 컴퓨팅 환경이 같아야 함 (CPU Bus Size Endianness, OS 등)
- $TB_SID.tip 파일의 DB_LOCK_SIZE, DB_NAME 이 동일 해야 함
- Primary Redo 로그가 남지 않는 Direct Path Insert Direct Path Load는 지원 X
- 테이블 스페이스와 파일의 상태를 변경하는 DDL 중 일부도 허용하지 않음 *(책 P. 528 참고)*
	-> 이렇게 되었을 경우 다시 Primary DB 백업하여 Standby 구성 해야 함

---
### Primary DB
##### 동작 모드 (REPLICATION_MODE)
- PROTECTION 모드
<font color="#4f81bd">LGWR SYNC로 설정된 Standby가 하나 이상</font> 있어야 하고, Primary는 LGWR SYNC인 Standby 중에서 <font color="#4f81bd">성공했다는 응답을 하나 이상</font> 받아야 진행할 수 있음
LGWR SYNC 구성으로 RECOVERY 모드로 기동된 Standby가 하나도 없는 상태에서 Primary를 기동하면 에러가 발생한다.
-> LGWR SYNC인 <font color="#c0504d">모든 Standby가 실패 하면 Primary도 같이 실패</font>하고 더 이상 진행하지 않음.
	(진행하지 않는다 의미 : Primary 노드가 더 이상 트랜잭션 처리를 진행하지 않는다)

- AVAILABILITY 모드 
<font color="#4f81bd">LGWR SYNC로 설정된 Standby가 하나 이상</font> 있어야 하고, Primary는 LGWR SYNC인 Standby
중에서 <font color="#c0504d">성공했다는 응답이 없으면 Standby와 동기화를 포기하고 계속 진행</font>한다.
LGWR SYNC 구성으로 RECOVERY 모드로 기동된 Standby가 하나도 없는 상태에서 Primary를 기동하면 에러가 발생한다.

- PERFORMANCE 모드
<font color="#4f81bd">로그 전송 방식에 제한이 없</font>고<font color="#c0504d"> Standby의 성공과 무관하게 Primary는 계속 진행</font>한다. 
따라서 Primary의 성능에 가장 영향을 주지 않는다.

##### 기동시키기
```Shell
$TB_SID.tip

LOG_REPLICATION_MODE=PERFORMANCE
LOG_REPLICATION_DEST_1="192.169.187.129:8633 LGWR ASYNC"
LOG_REPLICATION_DEST_2="192.169.187.130:8633 ARCH ASYNC"
```
- LOG_REPLICATION_MODE     : Primary 동작 모드 (Default : UNPROTECTED)
- LOG_REPLICATION_DEST_N  : Standby 주소/포트, 전송 방식 설정


---
### Standby DB
##### 기동시키기
1. 티베로 설치
   `Create Database` 는 수행하지 않는다.
2. **$TB_SID.tip** 파일 수정
   DB_NAME,  DB_BLOCK_SIZE 를 Primary 것과 같도록 설정
3. Primary 백업 파일 -> Standby로 복사
   Primary의 데이터 파일 / 컨트롤 파일 / Redo 로그 파일 / passwd 파일
   이때 복사한 컨트롤 파일이 Standby $TB_SID.tip 파일의 CONTROL_FILES에 있는지 확인 필요
> [!데이터 파일 / 컨트롤 파일 / Redo 로그 파일 / passwd 파일]
> - 데이터 파일: 데이터 동기화를 위해서  (초기 데이터 파일)
> - 컨트롤 파일: 데이터베이스의 구조와 메타데이터(데이터 파일 경로, Redo 로그 파일 정보, DB 이름, SCN 등) > Standby의 초기 구성은 Primary의 컨트롤 파일을 기반으로 설정
> - Redo Log 파일: 발생한 모든 변경 사항
> - Passwd 파일: 데이터베이스 인증과 관련된 정보

4. 파일 경로 변환
   Primary에서 Standby로 복사한 DB 파일의 디렉토리 경로가 Primary와 다른 경우,
   Standby $TB_SID.tip 파일 STANDBY_FILE_NAME_CONVERT에 경로 변환을 설정하고, Standby MOUNT 모드로 기동(ibboot-t MOUNT)한 후 Alter Database를 수행하면 변환된 경로가 컨트롤 파일에 반영됨.
```Shell
Standby $TB_SID.tip

STANDBY_FILE_NAME_CONVERT="Primary 절대경로, Standby 절대경로"

$ tbboot-t MOUNT
Change core dump dir to /tibero/tibero7/bin/prof.
Listener port- 8629
Tibero 7
TmaxData Corporation Copyright (c) 2008-. All rights reserved.
Tibero instance started up (MOUNT mode).

$ tbsql sys/tibero
Connected to Tibero.

ALTER DATABASE STANDBY CONTROLFILE: Database altered.

# 자세한 코드는 책 P.532 참고
```

5. Standby RECOVERY 모드로 기동
   Standby RECOVERY 모드로 기동 : `$ tbboot -t RECOVERY`
   Standby를 한 번이라도 NORMAL 모드로 기동하면 더 이상 Standby 기능을 할 수 없음
	Normal 모드 = Read Only 모드
##### Read Only 모드
`Recovery 모드`  : Standby DB 는 기본적으로 복구 모드로 동작 
 ➡️ 1. Redo Log 기록, 2. 복구
 ➡️ 사용자 접근 제한됨 (참고 : sys 계정은 접근 가능 - 일부 동작만)

`Read Only 모드`
<font color="#4f81bd">복구하는 과정을 중단하지 않고</font> Read Only 모드로 전환하여 사용자의 <font color="#4f81bd">Read Only 접근을 허용</font>
(Standby가 LGWR SYNC 모드로 설정되어 Primary와 동기화 되어 있는 경우라면 최근에 Commit 된 내용을 Standby에서도 볼 수 있음)
```Shell
$ tbboot -t RECOVERY

$ tbsql sys/tibero

ALTER DATABASE OPEN READ ONLY CONTINUE RECOVERY: 

ALTER DATABASE STANDBY:
```
(접근도 가능, 복구도 가능)

##### TAC-TSC
Standby도 TAC 구성이어야 함

🙋복구는 1개의 노드에서만 진행 가능.
↓
**기본 TSC**
기본적으로 Standby는 1 Node TAC로 구성.

**멀티노드 TSC**
- 1개의 노드      : Recovery 모드로 기동
- N-1개의 노드 : Read Only 모드로 기동
(노드를 최대 Primary 노드 개수만큼 구성 가능)

```Shell
# Standby TAC의 STB_SID.tip
$ STANDBY_ENABLE_TAC_MULTINODE=Y

# Standby TAC에서 복구 담당 노드
$ tbboot-t RECOVERY

# Standby TAC에서 Read Only 모드 노드
# RECOVERY 모드로 기동하고 나서 Read Only 모드로 전환해야함
$ tbboot-t RECOVERY
$ tbsql sys/tibero
ALTER DATABASE OPEN READ ONLY CONTINUE RECOVERY:
```
Standby에서 복구를 담당하는 노드는 Primary $TB_SID.tip 파일에서 설정한 노드이며, 복구는 한 노드가 담당
👉 Primary $TB_SID 파일에
LOG_REPLICATION_MODE와 LOG_REPLICATION_DEST_N를 동일하게 설정해야 함


설정 옵션
```Shell
# Primary 모든 노드의 $TB_SID.tip 에 아래 변수 설정
 LOG_REPLICATION_MODE
 LOG_REPLICATION_DEST_N

# LGWR PROXY 방식
# 살아 있는 노드가 다운된 노드의 Redo 로그를 대신 Standby로 전송
STANDBY_ENABLE_LOG_RECOVERY=Y 
```


##### Standby Redo Log Group 
이해 못함
책 P. 536 참고


##### Cascade TSC
Primary에 여러 Standby를 연결한 경우,
Primary가 여러 Standby로 Redo 정보를 보내는 데 부담이 될 수 있음!
⬇️
*Redo Log 전달*
Primary가 모든 Standby에 전달 ❌
Primary → Cascade Standby → 다른 Standby ⭕

설정 제약
```Shell
# Cascade Standby $TB_SID.tip 파일
LOG_REPLICATION_MODE =  AVAILABILTY | PERFORMANCE 만  가능
LOG_REPLICATION_DEST_N = LGWR ASYNC | ARCH ASYNC 만 가능
```


##### Snapshot Standby
- Redo 정보를 계속 받아 저장 ⭕
- 복구 ❌
- 독자적으로 DDL/DML을 할 수 있는 Standby

🐤 Redo Log를 저장은 하되 DB에 반영은 하지 않음 + DDL/DML 가능 (테스트 용도로 보통 사용)
    Snapshot Standby -> Standby 변경 시점에
     - Snapshot Standby 되기 직전의 DB로 돌아가
     - DDL/DML 변셩사항 무시
     - 그동안 저장했던 Redo Log 반영 (복구)

**Snapshot Standby -> Standby**
책 참고
**Standby -> Snapshot Standby** 
책 참고

### TSC 역할 전환
Standby ◀️▶️ Primary 전환!

*용어 미리보기*
- Switchover: Primary 종료시 사용
- Failover: Standby를 종료 후 Primary로 기동시 사용

*계획 하에 Primary 종료, Standby 중 하나를 Primary로 전환할 때*

1. Primary를 SWITCHOVER 모드로 종료
Primary가 NORMAL 모드로 종료 + 모든 Redo 로그를 Standby에게 전송 후 종료
```Shell
$ tbdown -t SWITCHOVER
```

2. Standby를 Primary로 전환
```Shell
$ tbdown
$ tbboot-t FAILOVER # Primary 기동 (NEw Primanry)
```
   동시에 기존 Primary 는 Standby로 기동하고 싶다면? 
```Shell
# NEW PRIMARY
# $TB_SID.tip 파일
LOG_REPLICATION_MODE, LOG_REPLICATION_DEST_N에 New_Standby 추가
$ tbboot-t FAILOVER 

# NEW STANDBY
$ tbboot-t RECOVERY
```


*예상치 못하게 현재 Primary가 비정상적으로 종료된 경우*
-> Standby 중 하나를 Primary로 전환 해야함
```Shell
$ tbdown
$ tbboot-t FAILOVER # Primary 기동 (NEw Primanry)
```

위와 차이점은,
기존에 사용하던 <font color="#4f81bd">Primary는 SWITCHOVER 모드로 종료되지 않았기 때문에</font> 새로운 Primary가 운영된 후에는 <font color="#4f81bd">더 이상 어떤 용도(Primary 나 Standby)로도 사용될 수 없음</font>


### CM Observer
*이 전까지의 내용은 Primary장애 발생시, DBA가 관여하여 Standby Failover함.*

**CM Observer**
Primary와 Standby의 CM과 주기적으로 통신 ➡ Primary 장애 발생 시 ➡  Standby를 Primary로 자동 Failover

**요구사항**
- CM Observer는 관제 대상이 되는 Primary와 Standby(CM, 인스턴스)와 독립 된 서버에서 동작 (Primary, Standby 서버 장애시 영향 X)
- 안정적으로 유지(Troubleroot 환경)되어야 함.