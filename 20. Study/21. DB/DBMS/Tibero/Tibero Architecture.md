---
sticker: emoji//1f4af
---

**아키텍쳐 **
- [[#Instance + Database|Instance + Database]]
	- [[#Instance + Database#Instance|Instance]]
		- [[#Instance#프로세스|프로세스]]
		- [[#Instance#메모리|메모리]]
	- [[#Instance + Database#Database|Database]]
		- [[#Database#논리 저장구조|논리 저장구조]]
		- [[#Database#물리 저장구조|물리 저장구조]]
	- [[#Instance + Database#SQL 처리 과정|SQL 처리 과정]]
		- [[#SQL 처리 과정#select 과정|select 과정]]
		- [[#SQL 처리 과정#update, delete 과정|update, delete 과정]]
		- [[#SQL 처리 과정#insert 과정|insert 과정]]



---

![[Tibero Architecture-20240909110926976.webp]]
## Instance + Database
### Instance
[[Instance]]참고
tbboot 하면 생성되고 
tbdown 하면 사라짐

사용자가 SQL 던지면 워커프로세스가 DB의 Data files에서 데이터 갖고 와서 메모리에 올려서 처리하고 데이터 갖고 와서 사용자 보여줌!

#### 프로세스
[[Tibero Process]]참고

#### 메모리 
[[Tibero Memory]]참고


### Database
[[Tibero Storage]]참고
#### 논리 저장구조
#### 물리 저장구조

---
### SQL 처리 과정
#### select 과정
1. Client > Listener 요청
2. Listener는 프로세스, 스레드 할당
3. 파싱작업 (워커스레드 진행?)
	1. 공유 메모리 pp 캐시 확인 (이미 만들어진 실행계획 있는지)
		1. 있으면 소프트 파싱
		2. 없으면 하드 파싱 (새로 분석, 실행계획 생성)
	2. 공유 메모리 dd 캐시 확인 (SQL 문에서 참조하는 테이블과 열에 대한 메타데이터(구조 정보)를 확인)  
4. Database Buffer Cache 확인 (찐 데이터 들어있는 영역)
	1. 요청한 데이터 있으면 갖고 와
	2. 없으면 저장소 가서 데이터 읽어와
		1. 디스크에 있는 데이터 파일을 통해 데이터 갖고옴.
		2. 데이터 블록을 읽어와서 Database Buffer Cache에 먼저 적재.
		3. 클라이언트 요청에 맞게 데이터 추출
		   
*데이터 블록과 데이터 파일*: [[Tablespace]]하단 참고
**data block**  < extent < segment < **datafile** < tablespace

#### update, delete 과정
흰색은 동일 / 주황은 달라지는 거

1. Client > Listener 요청
2. Listener는 프로세스, 스레드 할당
3. 파싱작업 (워커스레드 진행?)
	1. 공유 메모리 pp 캐시 확인 (이미 만들어진 실행계획 있는지)
		1. 있으면 소프트 파싱
		2. 없으면 하드 파싱 (새로 분석, 실행계획 생성)
	2. 공유 메모리 dd 캐시 확인 (SQL 문에서 참조하는 테이블과 열에 대한 메타데이터(구조 정보)를 확인)  
4. Database Buffer Cache 확인 (찐 데이터 들어있는 영역)
	1. 요청한 데이터 있으면 갖고 와
	2. 없으면 저장소 가서 데이터 읽어와
		1. 디스크에 있는 데이터 파일을 통해 데이터 갖고옴.
		2. 데이터 블록을 읽어와서 Database Buffer Cache에 먼저 적재.
5. <font color="#fac08f">데이터 변경 작업 수행</font>
6. <font color="#fac08f">Redo Log Buffer에 변경 사항 기록</font>
7. <font color="#fac08f">롤백을 대비해서 Undo 데이터를 생성하고 Undo 테이블스페이스에 저장함.</font>
8. <font color="#fac08f">변경사항 커밋 (Log Writer(LGWR) 프로세스가  Redo Log 파일로 기록. 메모리 > disk)</font>
<font color="#fac08f">   커밋시, Redo Log Buffer 꽉찼을때, 그냥 주기적으로, 체크포인트 발생시</font>
	1. 데이터는 데이터 파일에
	2. redo로그버퍼는 리두로그 파일에 적재


#### insert 과정
흰색은 동일 / 주황은 동일 / 파랑은 달라지는거
1. Client > Listener 요청
2. Listener는 프로세스, 스레드 할당
3. 파싱작업 (워커스레드 진행?)
	1. 공유 메모리 pp 캐시 확인 (이미 만들어진 실행계획 있는지)
		1. 있으면 소프트 파싱
		2. 없으면 하드 파싱 (새로 분석, 실행계획 생성)
	2. 공유 메모리 dd 캐시 확인 (SQL 문에서 참조하는 테이블과 열에 대한 메타데이터(구조 정보)를 확인)  
4. Database Buffer Cache 확인 <font color="#92cddc">(insert 할 테이블의 데이터 블록이 있는지 확인 - 데이터를 삽입할 공안 확보를 위해)</font>
	1. <font color="#92cddc">데이터 블록이 있으면 공간도 있는지 확인</font>
		1. <font color="#92cddc">공간이 있으면 메모리상에서 삽입</font> 
	2. <font color="#92cddc">없으면 저장소 가서 데이터블록 읽어와</font>
		1. <font color="#92cddc">갖고 온 데이터 블록에 삽입할 공간이 있으면 새데이터 삽입</font>
		2. <font color="#92cddc">공간이 없으면 새로운 데이터 블록 할당해서 메모리에 추가</font>
5. <font color="#fac08f">데이터 변경 작업 수행</font>
	1. <font color="#92cddc">이 데이터 블록은 Dirty 상태(메모리상의 변경 사항이 아직 디스크에 반영되지 않은 상태)</font>
	2. <font color="#92cddc">INSERT 작업으로 삽입된 데이터는 Database Buffer Cache에 먼저 기록되며, 이 변경 사항은 추후 DB Writer 프로세스에 의해 디스크의 데이터 파일로 기록</font>
6. <font color="#fac08f">Redo Log Buffer에 변경 사항 기록</font>
7. <font color="#fac08f">롤백을 대비해서 Undo 데이터를 생성하고 Undo 테이블스페이스에 저장함.</font>



