---
sticker: emoji//1f60e
---

*목차*
[[#Tablespace|Tablespace]]
[[#👍 역할  **(데이터를 논리적으로 구분 / 관리 용이성 / 성능을 최적화)**|👍 역할  **(데이터를 논리적으로 구분 / 관리 용이성 / 성능을 최적화)**]]
[[#➡️ 예시➡️ 예시]]
[[#종류|종류]]
[[#참고: 오라클에서의 논리적 단위|참고 : 오라클에서의 논리적 단위]]

### Tablespace

![[Pasted image 20240708101148.png]]

![[Tablespace-20240909143617692.webp]]

- 🙋 **Tablespace : 논리적 저장소 단위**
- Data File : 데이터가 저장되는 물리적인 공간 *(.dbf .ora 등)*
오라클 에서는 <font color="#4f81bd">테이블이 저장될 공간(테이블스페이스)을 먼저 만들고 나서 테이블을 생성</font>

### 👍 역할  **(데이터를 논리적으로 구분 / 관리 용이성 / 성능을 최적화)**
- 용도에 따라 tablespace를 나누어 저장해 관리 용이성을 높임
  ex) 중요한 데이터를 저장하는 테이블스페이스는 고속 디스크에, 덜 중요한 데이터를 저장하는 테이블스페이스는 저속 디스크에 저장 / 자주 액세스하는 데이터를 특정 테이블스페이스에 저장하여 성능을 최적화
- Disk 용량 정의
  ex) 많은 데이터가 쌓일 게시판 테이블은 기본용량 100메가 자동확장 10메가로 테이블스페이스를 만들어서  그곳에 게시판 테이블을 만들어 쓰면 게시판 데이터는 그곳에 100메가까지 데이터가 저장되고 용량 초과시 자동적으로 10메가단위로 테이블 스페이스의 크기는 확장
- 이 테이블 스페이스의 사람들은 여기에 저장해??
- 


### ➡️ 예시
```sql
-- 테이블스페이스 생성
CREATE TABLESPACE sales_data
DATAFILE 'sales_data1.dbf' SIZE 100M, -- data file 지정
         'sales_data2.dbf' SIZE 100M;

-- 테이블 생성 시 특정 테이블스페이스 지정
CREATE TABLE sales (
    sale_id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
) TABLESPACE sales_data;
```


### 종류
**<font color="#4f81bd">System TableSpace</font>**
  오라클 데이터베이스를 생성할 때 자동으로 생기며 오라클 데이터베이스의 기동을 위해 꼭 필요한 테이블스페이스
  - Control File
  - 딕셔너리 파일 저장
      → 오라클 서버의 모든 정보를 저장하고 있는 아주 중요한 테이블이나 뷰를 말하며 이 파일이 손상될경우 DB 접근이 불가능해 지기 떄문에 아주 중요
  - 저장 피로시저, 패키지, 트리거 등

**<font color="#4f81bd">SYAUX  tablespace</font>**
Oracle10g 버전부터 등장한 기능으로 주로 Oracle서버의 성능 튜닝을 위한 데이터들이 저장

**<font color="#4f81bd">일반 tablespace</font>**
가장 일반적으로 많이 사용되는 Tablespace로 관리자가 필요에 의해서 만드는 Tablespace, DBA뜻대로 얼마든지 만들고 지울 수 있음

### 참고: 오라클에서의 논리적 단위
  ![[Pasted image 20240708102646.png]]
  data block  < extent < segment < tablespace

🐥 Table 과의 관계
- data block 
    데이터를 저장하는 가장 작은 논리적 단위
    (테이블의 <font color="#c0504d">각 행</font> : <font color="#c0504d">1개 이상의 데이터 블록</font>에 저장)
- extent 
  테이블이 데이터를 추가하면, 필요한 만큼의 익스텐트를 할당받아 데이터를 저장
- segment 
  얘는 어댑터 같은 것
  ex) 테이블, 인덱스, 클러스터 등
	segment에 포함이 되느냐 안되느냐! → 데이터 저장이 되느냐 안되느냐로 판단함
	뷰         : 저장안됨 → 세그먼트 X
	테이블  : 저장됨    → 세그먼트 O



---
### Auto Extend
**Auto Extend**는 Tablespace의 데이터 파일이 **자동으로 확장**되도록 설정하는 옵션