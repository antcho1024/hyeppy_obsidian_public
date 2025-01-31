---
sticker: emoji//1f644
---
- [[#TPR (Tibero Reformance Repository)|TPR (Tibero Reformance Repository)]]
- [[#사용해보기|사용해보기]]
	- [[#사용해보기#TPR 기능 활성화|TPR 기능 활성화]]
	- [[#사용해보기#수집된 정보 조회|수집된 정보 조회]]
	- [[#사용해보기#스냅샷 수동 저장|스냅샷 수동 저장]]
	- [[#사용해보기#TPR report 사용해보기|TPR report 사용해보기]]
- [[#Trouble Shooting|Trouble Shooting]]

### TPR (Tibero Reformance Repository) 
성능 정보를 주기적으로 자동으로 수집하는 기능

**- 스냅샷 저장 기능**
  TPR은 `_VT_JCNTSTAT`, `V$SYSTEM_EVENT`, `V$SQLSTATS`,`V$SGASTAT` 를 주기적으로 테이블에 저장함
  👉 이 스냅샷을 이용해서 report를 만든게 <font color="#c0504d">TPR report</font> !
**- 세션상태 저장 기능**
  TPR은 1초에 한번씩 현재 RUNNING 상태인 세션 정보를 Shared Cache에 위피하는 큐에 1시간 동안 저장
  - `V$ACTIVE_SESSION_HISTORY` 동적뷰로 조회 가능
  - 큐 전체 용량 2/3가 채워졌거나 1시간이 지난 데이터는 샘플링하여`_TPR_ACTIVE_SESSION_HISTORY` 테이블에 저장됨 (`DBA_HIST_ACTIVE_SESS_HISTORY` 동적뷰로도 조회 가능)
  
---
### 사용해보기
#### TPR 기능 활성화
- `$TB_SID.tip` 파일 수정
```Shell
vi $TB_HOME/config/$TB_SID.tip

# 스냅샷 기능
TIBERO_PERFORMANCE_REPOSITORY=Y

# 세션상태 저장 기능
ACTIVE_SESSION_HISTORY=Y

# 추가 설정?
TOTAL_SHM_SIZE=2GB # 2GB 이상
```
이 외의 추가 설정(스냅샷 저장주기, 보유 기간, TPR report Top 갯수 등등)은 정복하기 책 P.288 참고

*위 파일 수정 시 tibero 재부팅하기 → tbdown tbboot*
⛔ERROR : 하단 트러블 슈팅 참고

#### 수집된 정보 조회
- 스냅샷 저장하는 테이블, 뷰
- 세션상태 저장하는 테이블, 뷰

#### 스냅샷 수동 저장
TPR 설정시 정해진 주기에 자동으로 수집됨. 그치만 수동으로 현재 시점의 스냅샷을 저장하고 싶다면!
```Shell
# 스냅샷 생성
SQL > EXEC DBMS_TPR.CREATE_SNAPSHOT();
```

#### TPR report 사용해보기
- 스냅샷 → TPR report   ⭕ 가능
```sql
-- 스냅샷 조회
SELECT SNAP_ID,
	TO_CHAR(BEGIN_INTERVAL_TIME, 'YYYY/MM/DD HH24:MI:SS') BEG,
	TO_CHAR(END_INTERVAL_TIME, 'YYYY/MM/DD HH24:MI:SS') END
FROM _TPR_SNAPSHOT
ORDER BY SNAP_ID DESC;

-- 시간 형식을 설정
ALTER SESSION SET NLS_DATE_FORMAT='YYYY/MM/DD HH24:MI:SS';

-- REPORT 생성
-- _TPR_SNAPSHOT의 'BEGIN_INTERVAL_TIME'이 아래 범위에 포함하는 모든 스냅샷 사용
EXEC DBMS_TPR.REPORT_TEXT('2024/08/06 15:50:00', '2024/08/06 15:59:00');
EXEC DBMS_TPR.REPORT_TEXT(sysdate, sysdate-1);

-- report 조회 아래 폴더에 .txt 파일로 존재
cd /tibero/tibero7/instance/tibero
```

- 세션정보 → TPR report   ❌ 아직 불가능

👍 <span style="background:#fff88f">report 뽑기 성공!</span>
![[TPR -20240806163108683.webp]]


https://technet.tmaxsoft.com/upload/download/online/tibero/pver-20160406-000002/tibero_admin/apm.html
→ TPR Report 문서

---
### Trouble Shooting
- $TB_SID.tip 수정 후 tbboot 시 SHM 사이즈 에러
<font color="#c0504d">에러 내용</font>
```Shell
[tibero@tibero-svr: ~]#tbboot
iparam condition check failed. name:ACTIVE_SESSION_HISTORY, value: 1
*** Tibero initialization parameter (tip) file failure:
Error (-7200) occurred while processing parameter 'ACTIVE_SESSION_HISTORY' and value '1'    
(ACTIVE_SESSION_HISTORY TOTAL_SHM_SIZE is too small to use active session history funtionality. Set TOTAL_SHM_SIZE more than 2GB.)..
Tip file path = /tibero/tibero7/config/tibero.tip
```

<font color="#c0504d">해결방안</font>
```Shell
vi $TB_HOME/config/$TB_SID.tip
TOTAL_SHM_SIZE=3GB # 2-> 3 으로 수정 후
tbboot
```
