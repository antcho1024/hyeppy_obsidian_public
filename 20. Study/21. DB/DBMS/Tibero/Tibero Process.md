---
sticker: emoji//1f47f
---
# 프로세스
- [[#Listener|Listener]]
- [[#Worker Process|Worker Process]]
	- [[#Worker Process#Foreground Worker Process|Foreground Worker Process]]
	- [[#Worker Process#Background Worker Process|Background Worker Process]]
- [[#Background Process|Background Process]]
	- [[#Background Process#Monitor Process|Monitor Process]]
	- [[#Background Process#Tibero Manager Process|Tibero Manager Process]]
	- [[#Background Process#Parallel Execution Worker Process|Parallel Execution Worker Process]]
	- [[#Background Process#Recovery Worker Process|Recovery Worker Process]]
	- [[#Background Process#Database Writer Process|Database Writer Process]]
	- [[#Background Process#Agent Process|Agent Process]]



![[Tibero Process-20240725164519372.webp]]

---
## Listener
  <font color="#c0504d">클라이언트의 새로운 접속 요청을 받아</font> 이를 유휴한 <font color="#c0504d">Worker Process에 할당</font>
  - 독립적으로 기동/종료 할 수 없고<font color="#4f81bd"> 티베로 DB와 같이 기동/종료됨</font>
  - Listener를 강제 종료해도 <font color="#4f81bd">Moniter Process에 의해 다시 기동</font>됨
  ![[Tibero Process-20240725173535592.webp]] 
> [!새로운 요청이 이루어지는 순서]
> 1. 클라이언트가 Listener에게 접속 요청
> 2. Listener가 현재 유휴한 Worker Thread가 있는 Worker Process를 찾아 해당 Control Thread에게 클라이언트 접속을 요청
> 3. Listener의 요청을 받은 Control Thread는 자신에게 속한 Worker Thread의 상태를 검사하여 현재 유휴한 Worker Thread에게 클라이언트 접속을 할당
> 4. 할당된 Worker Thread는 클라이언트와 인증 절차를 거친 후 세션을 시작

👉 **리스너 포트 설정** (윈도우는 불가)
- $TB_SID.tip파일 →  `LISTNER_PORT`, `EXTRA_LISTNER_PORTS`  : 이럴경우 재기동 필요
- 동적으로 추가 가능
```SQL
ALTER SYSTEM LISTNER ADD PORT 8799;
ALTER SYSTEM LISTNER DELETE PORT 8799;
```


---
## Worker Process
![[Tibero Process-20240807142219055.webp]]
<font color="#c0504d">클라이언트와 실제로 통신</font>을 하며 사용자의 요구 사항을 처리하는 프로세스
### Foreground Worker Process 
Listener를 통해 들어온 <font color="#c0504d">온라인 요청을 처리</font>하는 프로세스
![[Tibero Process-20240807143438945.webp|296]]
1개 Worker Process = 1개 <font color="#9bbb59">Control Thread</font> + 10개의 <font color="#9bbb59">Worker Thread</font>

**Control Thread (CTHR)**
- 워커 프로세스 마다 한개씩 존재하며, <font color="#9bbb59">Worker Thread</font> 를 생성
- 클라이언트가 Listener에게 접속 요청을 하면
  Listener가 현재 유휴한 <font color="#9bbb59">Worker Thread</font>가 있는 Worker Process를 찾아 해당 <font color="#9bbb59">Control Thread</font>에게 클라이언트 접속을 요청을 하면
  <font color="#9bbb59">Control Thread</font>가 현재 유휴한 <font color="#9bbb59">Worker Thread</font>에게 클라이언트 접속을 할당
**Worker Thread (WTHR)**
- <font color="#9bbb59">Worker Thread</font> 1개당 Session 1개 
  (클라이언트와 1:1로 통신하며 클라이언트가 보내는 메시지를 받아 처리 → DBMS 대부분의 일 처리(SQL Parsing, 최적화, 수행 등))
- dedicated 모드 방식 (Session과 Thread가 1:1관계) 타 DB의 경우 Shared 모드 방식도 있음

<span style="background:#fff88f">COUNT 관련 정리..</span>
- MAX_SESSION_COUNT          : Woker Thread 최대 갯수 (얘만 직접 설정하는 것을 권장)
- MAX_BG_SESSION_COUNT    : 위 세션중 Background Worker Thread 갯수
- \_WTHR_PER_PROC                  : Process 1개당 Worker Thread 갯수
- WTHR_PROC_CNT                 : Worker Process 갯수
→ Listner를 통해 붙을 수 있는 최대 세션 갯수 = 
   MAX_SESSION_COUNT -  MAX_BG_SESSION_COUNT = WTHR_PROC_CNT * \_WTHR_PER_PROC
→  \_WTHR_ PER_PROC * N = MAX_BG_SESSION_COUNT

### Background Worker Process
<font color="#c0504d">Internal Task나 Job Scheduler에 등록된 배치 작업을 실행</font>하는 프로세스
리스너 업무에 관련된 프로세스가 아님

---
## Background Process

### Monitor Process (MONP)
- 티베로 기동 시 제일 먼저 생성, 종료 시 가장 마지막에 종료됨 
- Listener를 포함한 다른 프로세스를 생성
- 주기적으로 모든 프로세스 의 상태를 모니터링
- 교착상태(Deadlock)도 감시
### Tibero Manager Process (MGWP)
- 클라이언트 세션의 영향(세션 풀)을 받지 않고 티베로에 원활히 접속하기 위해 시스템 관리 용도로 예약된 Worker Process
- Worker Process와 비슷한 기능을 하지만 오직<font color="#c0504d"> sys의 요청을 처리</font>
- <font color="#c0504d">sys가 접속 요청</font>을 하면 Listener를 거치지 않고 스페셜 포트를 통해 Tibero Manager Process를 할당
- Foreground Worker Process처럼 1개의 Control Thread 와 N개의 Worker Thread 가 있음, 안에 리스너도 있음
### Parallel Execution Worker Process (PEWP)
(추가 공부 필요)
- 세션에서 병렬 쿼리 수행 시 할당되는 프로세스
- 병렬 쿼리 수행 세션 (QC query coordinator) 1개당 PEWP 1개 할당 
   (Multi-Cursor의 경우 PEWP 여러개 할당 가능)
- 1개의 병렬쿼리 : PEWP 내 여러개 Parallel Execution Thread러 처리 
- 파리미터 : PEP_PROC_CNT
 🐥 ex) 워커 스레드가 1개의 세션(SQL)을 처리하고 있는데 만약 테이블 10개를 읽어와야 한다면 10개를 하나하나 순서대로 읽지 않고  동시에 할 수 있게 함 
(이 판단은 옵티마이저가 실행계획을 짠 후 병렬 판단)
- 메인 Thread : 작업을 분리해주는 역할, 전달해주는 Thread - 절반 (ex. 8개)
- 작업을 처리하는 Thread - 절반 (ex. 8개)
### Recovery Worker Process (RCWP) 
- Crash Recovery, Instance Recovery, Media Recovery를 수행하는 Thread를 모아 놓은 프로세스
- 해당 Thread 로는 LNW(Log Network Writer), LNR(Log Network Reader), SMR(Standby Managed Recovery) 등이 있음

### Database Writer Process (DBWR)
Buffer Cache와 Redo Log Buffer에서 변경된 내용을 디스크에 기록하는 일과 관련된 Thread를 모아 놓은 프로세스
- *LGWR* (Log Writer Thread)
  Log Writer Thred의 약어로 Redo Log Buffer의 내용을 Redo 로그 파일에 쓰는Thread.
  Write시점
	- Commit 발생
	- 로그 스위치 발생
	- 책참고..귀찮아..
- *BLKW* (Block Writer Thread)
- *LOGA*
  Log Archiver Thread 약어로 로그 파일의 아카이브 작업을 수행하는 Thread.  Log Switch 또는 요청시 Redo 로그 파일에서 Archive Log를 생성한다.
  🐤Redo Log가 Storage 중 Redo Log Files에 저장되잖아. > 근데 꽉차거나 로그 스위치 시에! 이 스레드가 아키아브 로그 파일로 복사해서 저장해. > Redo Log Files에서는 지워. > 그럼 아카이브 로그는 나중에 복구작업에 쓸 수 있는거야.
- *CKPT* 
- *FBWR* 
   (자세한건 책 참고)
  
### Agent Process (AGNT)
시스템 유지를 위해 티베로 내부의 주기적인 작업 수행 
Job 을 담당함
(자세한건 책 참고)
- 실제 작업은 Worker Thread 임
- 일을 시키는 일만 함
- 만약 모든 Worker Thread가 풀이면 Job Fail
