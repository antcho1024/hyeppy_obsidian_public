<font color="#c0504d">Multi Instance</font> 를 구현하기 위한 기술
![[Pasted image 20240710154311.png]]
👉 최소 2대의 서버 필요
하드웨어 <font color="#c0504d">서버 1개당 1개의 인스턴스가 실행</font>되고 공유 스토리지(SAN, NAS)가 필요.

👍 장점
**높은 처리량과 고가용성**
- 고가용성 및 장애 조치 : 한 노드에 장애가 발생해도 다른 노드에서 서비스가 지속 가능 (Failover, Failback 기능)
- 확장성 : 필요에 따라 노드를 추가하여 시스템의 처리 용량을 확장 가능
- 부하분산 

https://myalpaca.tistory.com/17


![[Pasted image 20240710164833.png]]
