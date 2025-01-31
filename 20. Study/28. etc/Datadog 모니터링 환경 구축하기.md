*구축 요약!*
- Datadog 환경 서버 필요함 (모니터링 한 데이터를 수집하는 서버)
  이렇게 부르는지는 몰겠으나 이 문서에서는 그냥 **Repo DB**라고 부르겠삼
- 모니터링 대상 서버에 DB구축, 모니터링 Agent 설치 필요함
  **관제 DB** 라고 부르겠삼



### Datadog 설치
Datadog 설치 서버 
IP: 192.168.56.102
id/pw: root/root

내 local VM에 띄워져 있는 Red hat 서버이구 네트워크는 외부에서 접속 가능으로 뚫어놓은 상태 (외부망 뚫어놓아서 가능한지는 몰겠삼 확인필요함 일단 참고!)

1. Datadog Free Trial 신청
	[datadog](https://www.datadoghq.com/)에 접속하여 free trial 신청 > 입력 폼 입력
2. 모니터링 대상 스택 > 필수 아님
3. 운영중인 서버 갯수: 걍 5개 정도 함
![[Datadog 모니터링 환경 구축하기-20250122101051964.webp]]


4. Repo DB 서버가 Red Hat 이므로 OS 선택해서 설치 명령문 복사
   (설치할 Repo DB 서버 OS에 맞게 선택 필요!)

5. 명령어 붙여넣기 (Repo DB 서버 SSH에)
   다 끝난뒤에 쓰고 있는거라 캡쳐본이 없으나
   그냥 복붙하면 됐었음!
  
  6. 다 되면 다시 홈페이지 들어가보면 다 됐다고 떠 있음!![[Datadog 모니터링 환경 구축하기-20250122101538980.webp]]
     정상 설치 안될 경우 Finish 비활성화 되어 있음

끝! 

들어가보기
https://us5.datadoghq.com/ 
(지역: us5-Central)
id: haesoo_cho@tmax.co.kr
pw: tmax1111@


참고 [블로그](https://m.blog.naver.com/techtrip/222048427611)

---
### Agent 설치 (Window, Oracle)
이제 모니터링 대상 DB를 정하고 관제 DB 에 Agent 를 붙여야함.

모니터링 대상 환경은 내 로컬 Window에 있는 Oracle 21c (Muti-tanant)임. 

1. Oracle 설치
2. Oracle Client 설치 (필수인지 확인 필요. 문서를 이것저것 봐서 찾아봐야함)
3. Visual Studio 2017 설치 (필수인지 확인 필요. 문서를 이것저것 봐서 찾아봐야함)

4. Oracle Database에 Datadog 사용자 생성

Window + R > cmd 창 열기 > `sqlplus / as sysdba` 
```SQL
CREATE USER c##datadog IDENTIFIED BY &password CONTAINER = ALL ;
# pw 입력하라고 나오면 원하는 pw 입력. 난 datadog 했음
ALTER USER c##datadog SET CONTAINER_DATA=ALL CONTAINER=CURRENT;
```

5. 사용자에게 데이터베이스에 대한 액세스 권한 부여
위 과정에서 계정이 바뀌었으면 다시 `sqlplus / as sysdba` 로 sysdba 계정으로 로그인 필요

```SQL
grant create session to c##datadog ;
grant select on v_$session to c##datadog ;
grant select on v_$database to c##datadog ;
grant select on v_$containers to c##datadog;
grant select on v_$sqlstats to c##datadog ;
grant select on v_$instance to c##datadog ;
grant select on dba_feature_usage_statistics to c##datadog ;
grant select on V_$SQL_PLAN_STATISTICS_ALL to c##datadog ;
grant select on V_$PROCESS to c##datadog ;
grant select on V_$SESSION to c##datadog ;
grant select on V_$CON_SYSMETRIC to c##datadog ;
grant select on CDB_TABLESPACE_USAGE_METRICS to c##datadog ;
grant select on CDB_TABLESPACES to c##datadog ;
grant select on V_$SQLCOMMAND to c##datadog ;
grant select on V_$DATAFILE to c##datadog ;
grant select on V_$SYSMETRIC to c##datadog ;
grant select on V_$SGAINFO to c##datadog ;
grant select on V_$PDBS to c##datadog ;
grant select on CDB_SERVICES to c##datadog ;
grant select on V_$OSSTAT to c##datadog ;
grant select on V_$PARAMETER to c##datadog ;
grant select on V_$SQLSTATS to c##datadog ;
grant select on V_$CONTAINERS to c##datadog ;
grant select on V_$SQL_PLAN_STATISTICS_ALL to c##datadog ;
grant select on V_$SQL to c##datadog ;
grant select on V_$PGASTAT to c##datadog ;
grant select on v_$asm_diskgroup to c##datadog ;
grant select on v_$rsrcmgrmetric to c##datadog ;
grant select on v_$dataguard_config to c##datadog ;
grant select on v_$dataguard_stats to c##datadog ;
grant select on v_$transaction to c##datadog;
grant select on v_$locked_object to c##datadog;
grant select on dba_objects to c##datadog;
grant select on cdb_data_files to c##datadog;
grant select on dba_data_files to c##datadog;
```


6. View 생성, 권한 부여
```SQL
CREATE OR REPLACE VIEW dd_session AS
SELECT /*+ push_pred(sq) push_pred(sq_prev) */
  s.indx as sid,
  s.ksuseser as serial#,
  s.ksuudlna as username,
  DECODE(BITAND(s.ksuseidl, 9), 1, 'ACTIVE', 0, DECODE(BITAND(s.ksuseflg, 4096), 0, 'INACTIVE', 'CACHED'), 'KILLED') as status,
  s.ksuseunm as osuser,
  s.ksusepid as process,
  s.ksusemnm as machine,
  s.ksusemnp as port,
  s.ksusepnm as program,
  DECODE(BITAND(s.ksuseflg, 19), 17, 'BACKGROUND', 1, 'USER', 2, 'RECURSIVE', '?') as type,
  s.ksusesqi as sql_id,
  sq.force_matching_signature as force_matching_signature,
  s.ksusesph as sql_plan_hash_value,
  s.ksusesesta as sql_exec_start,
  s.ksusesql as sql_address,
  CASE WHEN BITAND(s.ksusstmbv, POWER(2, 04)) = POWER(2, 04) THEN 'Y' ELSE 'N' END as in_parse,
  CASE WHEN BITAND(s.ksusstmbv, POWER(2, 07)) = POWER(2, 07) THEN 'Y' ELSE 'N' END as in_hard_parse,
  s.ksusepsi as prev_sql_id,
  s.ksusepha as prev_sql_plan_hash_value,
  s.ksusepesta as prev_sql_exec_start,
  sq_prev.force_matching_signature as prev_force_matching_signature,
  s.ksusepsq as prev_sql_address,
  s.ksuseapp as module,
    s.ksuseact as action,
    s.ksusecli as client_info,
    s.ksuseltm as logon_time,
    s.ksuseclid as client_identifier,
    s.ksusstmbv as op_flags,
    decode(s.ksuseblocker,
        4294967295, 'UNKNOWN', 4294967294, 'UNKNOWN', 4294967293, 'UNKNOWN', 4294967292, 'NO HOLDER', 4294967291, 'NOT IN WAIT',
        'VALID'
    ) as blocking_session_status,
    DECODE(s.ksuseblocker,
        4294967295, TO_NUMBER(NULL), 4294967294, TO_NUMBER(NULL), 4294967293, TO_NUMBER(NULL),
        4294967292, TO_NUMBER(NULL), 4294967291, TO_NUMBER(NULL), BITAND(s.ksuseblocker, 2147418112) / 65536
    ) as blocking_instance,
    DECODE(s.ksuseblocker,
        4294967295, TO_NUMBER(NULL), 4294967294, TO_NUMBER(NULL), 4294967293, TO_NUMBER(NULL),
        4294967292, TO_NUMBER(NULL), 4294967291, TO_NUMBER(NULL), BITAND(s.ksuseblocker, 65535)
    ) as blocking_session,
    DECODE(s.ksusefblocker,
        4294967295, 'UNKNOWN', 4294967294, 'UNKNOWN', 4294967293, 'UNKNOWN', 4294967292, 'NO HOLDER', 4294967291, 'NOT IN WAIT', 'VALID'
    ) as final_blocking_session_status,
    DECODE(s.ksusefblocker,
        4294967295, TO_NUMBER(NULL), 4294967294, TO_NUMBER(NULL), 4294967293, TO_NUMBER(NULL), 4294967292, TO_NUMBER(NULL),
        4294967291, TO_NUMBER(NULL), BITAND(s.ksusefblocker, 2147418112) / 65536
    ) as final_blocking_instance,
    DECODE(s.ksusefblocker,
        4294967295, TO_NUMBER(NULL), 4294967294, TO_NUMBER(NULL), 4294967293, TO_NUMBER(NULL), 4294967292, TO_NUMBER(NULL),
        4294967291, TO_NUMBER(NULL), BITAND(s.ksusefblocker, 65535)
    ) as final_blocking_session,
    DECODE(w.kslwtinwait,
        1, 'WAITING', decode(bitand(w.kslwtflags, 256), 0, 'WAITED UNKNOWN TIME',
        decode(round(w.kslwtstime / 10000), 0, 'WAITED SHORT TIME', 'WAITED KNOWN TIME'))
    ) as STATE,
    e.kslednam as event,
    e.ksledclass as wait_class,
    w.kslwtstime as wait_time_micro,
    c.name as pdb_name,
    sq.sql_text as sql_text,
    sq.sql_fulltext as sql_fulltext,
    sq_prev.sql_fulltext as prev_sql_fulltext,
    comm.command_name
FROM
  x$ksuse s,
  x$kslwt w,
  x$ksled e,
  v$sql sq,
  v$sql sq_prev,
  v$containers c,
  v$sqlcommand comm
WHERE
  BITAND(s.ksspaflg, 1) != 0
  AND BITAND(s.ksuseflg, 1) != 0
  AND s.inst_id = USERENV('Instance')
  AND s.indx = w.kslwtsid
  AND w.kslwtevt = e.indx
  AND s.ksusesqi = sq.sql_id(+)
  AND decode(s.ksusesch, 65535, TO_NUMBER(NULL), s.ksusesch) = sq.child_number(+)
  AND s.ksusepsi = sq_prev.sql_id(+)
  AND decode(s.ksusepch, 65535, TO_NUMBER(NULL), s.ksusepch) = sq_prev.child_number(+)
  AND s.con_id = c.con_id(+)
  AND s.ksuudoct = comm.command_type(+)
;

GRANT SELECT ON dd_session TO c##datadog ;
```


7. Agent 설치 
[Agnet 설치 명령어](https://app.datadoghq.com/account/settings/agent/latest?platform=overview&_gl=1*jhmxwh*_gcl_aw*R0NMLjE3MzczMzI4NzIuQ2owS0NRaUEtYUs4QmhDREFSSXNBTF8tSDlrTERaWHZJTlQwLVlZNUlaeXlaWGVvMTV5WUZ5OVNqZHV3aFo4WnQ1cGZ6QjdnTW5LLUVFOGFBbFdBRUFMd193Y0I.*_gcl_au*MTA5MDU3NzI4Ni4xNzMwMTc2MjU1*_ga*MjAxNzQwNjMyOC4xNzIwNTAyMzQ4*_ga_KN80RDFSQK*MTczNzUwODE0OC4xMC4wLjE3Mzc1MDgxNDguMC4wLjEyMTE3MDU5MA..*_fplc*M2ZCYnElMkZlVm5BWiUyQjZiU0EyQzRqNnRSR0xkUDRzTTJZSElsJTJCSGlaOG5pcmVoR0duJTJCNDdSWU9VRFhuYVhGT2UlMkZ0ciUyQk1Ed1NkVmRFTUtFOEY3cTZ0bEYxS3RrampmcExJMHUlMkYzUnFFWHlDb0JFWjZ1Rk9ib0hCcWR0cDk3TWclM0QlM0Q.) 해당 링크에서 관제 DB OS에 맞게 선택하여 명령어 복사 > cmd 창에 붙여넣기
위 링크에 접속이 불가능하다면
[datadog 사이트](https://us5.datadoghq.com/ )여기에 로그인 후 (로그인 정보 상단에 있음) > 왼쪽 메뉴 > Integration > Agent 가면 있음!
현재 기준으로는 아래꺼임!
```shell
Start-Process -Wait msiexec -ArgumentList '/qn /i "https://s3.amazonaws.com/ddagent-windows-stable/datadog-agent-7-latest.amd64.msi" APIKEY="f9ab84993b39790bd30c227786706b42" SITE="us5.datadoghq.com"'
```

8. Agent 설정
이 과정은 <span style="background:#fff88f">Repo DB 서버 (Datadog 설치한 서버)</span>에서 수행하는 것임!

/etc/datadog-agent/conf.d/oracle.d/conf.yaml 파일을 생성해서 설정 옵션을 작성해줘야함.
```shell
vi /etc/datadog-agent/conf.d/oracle.d/conf.yaml
# 파일내용 작성 (하단 참고)
```

- 양식
```yaml
init_config:
instances:
  - server: '<HOSTNAME_1>:<PORT>'
    service_name: "<CDB_SERVICE_NAME>" # The Oracle CDB service name
    username: 'c##datadog'
    password: 'ENC[datadog_user_database_password]'
    dbm: true
    tags:  # Optional
      - 'service:<CUSTOM_SERVICE>'
      - 'env:<CUSTOM_ENV>'
  - server: '<HOSTNAME_2>:<PORT>'
    service_name: "<CDB_SERVICE_NAME>" # The Oracle CDB service name
    username: 'c##datadog'
    password: 'ENC[datadog_user_database_password]'
    dbm: true
    tags:  # Optional
      - 'service:<CUSTOM_SERVICE>'
      - 'env:<CUSTOM_ENV>'

```

- 내가 적은거
```yaml
init_config:
instances:
  - server: 192.168.207.131:1521
    service_name: xe 
    username: c##datadog
    password: datadog
    dbm: true
```
 *설명*
- server: 관제 DB 서버의 IP, Port
  나는 내 local 에 설치 되어 있는 oracle이므로 내 local IP 입력했고
  Port는 Oracle 기본 Port임. 변경한적 없으면 저거 일 것임
- service_name: 데이터베이스 이름 
  오라클 설치할때 변경하지 않았으면 orcl 혹은 xe 일 가능성이 높음 
  확인하는 쿼리 `SELECT NAME FROM V$DATABASE;`
- username, password : 사용자 이름, 비번 
  *4. Oracle Database에 Datadog 사용자 생성*에서 정의한 이름 비번
- dbm: Database Monitoring 기능 활성화하는 옵션

9. Agent 다시 시작 
   (Red Hat 기준) 다른 OS에 관해서는 [링크](https://docs.datadoghq.com/ko/agent/configuration/agent-commands/?tab=agentv6v7#start-stop-and-restart-the-agent)참고
   이 과정도 <span style="background:#fff88f">Repo DB 서버 (Datadog 설치한 서버)</span>에서 수행하는 것임!
```
sudo systemctl restart datadog-agent
```

![[Datadog 모니터링 환경 구축하기-20250122105614471.webp]]
![[Datadog 모니터링 환경 구축하기-20250122105647687.webp]]



끝!
[datadog](https://us5.datadoghq.com/ ) 로그인 > 왼쪽 메뉴 > APM 메뉴 > Database Monitoring 에 조회 되는지 확인!

참고 문서: [Datadog Docs](https://docs.datadoghq.com/ko/database_monitoring/setup_oracle/selfhosted/?tab=multitenant#%EC%82%AC%EC%9A%A9%EC%9E%90%EC%97%90%EA%B2%8C-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%EC%97%90-%EB%8C%80%ED%95%9C-%EC%95%A1%EC%84%B8%EC%8A%A4-%EA%B6%8C%ED%95%9C-%EB%B6%80%EC%97%AC)

### Agent 설치 (Ubuntu, Oracle)
- 실습, 작성 필요


### Agent 설치 (Ubuntu, PostgreSQL)

postgres랑 연결하기
postgres 설치: [[PostgreSQL 설치하기]]

- 실습, 작성 필요