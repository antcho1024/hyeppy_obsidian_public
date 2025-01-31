---
sticker: emoji//1f921
---
Instance : 메모리 구조와 백그라운드 프로세스의 집합

#### Database와 Instance의 관계
- 1 : 1 관계     |    단일 인스턴스
- 1 : N 관계    |   [[RAC (Real Application Clusters)]]
- N : 1관계     |   Multitenant - PDB, CDB (일반적이진 않음)
#### DB Instance 구현방식
*< Multi-Instance >*
##### <font color="#c0504d">1️⃣ 다중 서버 다중 인스턴스</font>
여러 서버(노드)에서 각각 독립적인 인스턴스를 실행하며, 이 인스턴스들은 하나의 데이터베이스를 공유
👍 장점 : 고가용성, 부하 분산, 장애 복구
ex) [[RAC (Real Application Clusters)]]

> [!구성예시]
>   -  Node 1: Instance 1
>   - Node 2: Instance 2
>   - Node N: Instance N
>   - Shared Storage: Data Files, Control Files, Log Files

##### <font color="#c0504d">2️⃣ 단일 서버 다중 인스턴스</font>
단일 서버에서 여러 인스턴스를 실행하여 각 인스턴스가 독립적인 메모리 구조와 백그라운드 프로세스를 갖습니다. 서버의 자원을 공유하며, 각 인스턴스는 개별적인 데이터베이스를 관리
👍 장점 : 자원 효율성, 독립적인 데이터베이스 관리.
ex) 한 서버에서 여러 Oracle 인스턴스 또는 PostgreSQL 인스턴스 실행.

> [!구성예시]
>   - Server: Instance 1, Instance 2, ..., Instance N

*< Single-Instance >*
##### <font color="#c0504d">3️⃣ 단일 서버 단일 인스턴스</font>
단일 서버에서 하나의 인스턴스만 실행하며, 이 인스턴스가 단일 데이터베이스를 관리합니다. 가장 일반적인 데이터베이스 설정 방식
👍 장점 : 간단한 설정 및 관리.
ex) 대부분의 MySQL, PostgreSQL, 단일 인스턴스 Oracle 데이터베이스.

> [!구성예시]
>   - Server: Instance 1