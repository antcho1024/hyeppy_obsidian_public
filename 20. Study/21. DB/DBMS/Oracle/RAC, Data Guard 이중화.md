참고 개념 : 
- [[RAC (Real Application Clusters)]]  /  TAC
- Active DataGuard (고가용성 Data Replication) / TSC


개념을 요약하자면
- **RAC**
  여러 Node (instance)에서 동시에 하나의 데이터베이스를 공유
  구성요소 : node N개, shared storage
  → <font color="#c0504d">instance node를 여러개 두기</font>
- **Active DataGuard**
  주 데이터베이스의 변경 사항을 실시간으로 스탠바이 데이터베이스에 복제
  Database 이중화.
> [!역할?]
> - 읽기 전용 쿼리
>   스탠바이 데이터베이스는 읽기 전용 모드로 운영되므로, 주 데이터베이스는 데이터 입력과 업데이트에 집중하고, 스탠바이 데이터베이스는 사용자가 조회하는 작업을 처리
> - Failover



🐥 <font color="#c0504d">두 기능을 같이 사용한다면?</font>
방법 1: 클러스터 이중화 (두 개의 클러스터) 👍
> [!구성]
> [ Primary Cluster ]
> - Primary Database
> - instance A (Node A)
> - Instance B (Node B)
> 
> [ Standby Cluster ]
> - Standby Database
> - Instance C (Node C)
> - Instance D (Node D)
> 

방법 2 : 단일 클러스터 내에서 데이터베이스 이중화
> [!구성]
> [ Single Cluster ]
> - Primary Database
> - Standby Database
> - Instance A (Node A)
> - Instance B (Node B)
> - Instance C (Node C) 

권장되고 많이 사용되는 방법 : 1번! (출처가 gpt임 확인은 필요)
why?
1. 고가용성 및 재해 복구 : 한 클러스터에 장애가 발생 시 다른 클러스터가 운영을 지속 가능/불가능
2. 부하 분산 부분에서 1번이 더 효율적



![[Pasted image 20240711155932.png]]
-> 나중에 이쁘게 다시 정리하자..