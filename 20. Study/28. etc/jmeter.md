---
sticker: emoji//1f911
---
## 설치 및 실행

- 설치
https://jmeter.apache.org/download_jmeter.cgi
Binaries > zip 파일 다운로드

- 실행
cmd -> 압축푼 폴더 아래 bin 폴더로 이동 -> jmeter 입력 후 엔터

![[jmeter-20240731162808006.webp]]
*경고들 : 플러그인을 찾기 위해 패키지 스캐닝을 사용하는 기능이 더 이상 권장 X, CLI 권장, 테스트 요구사항에 맞게 Java Heap 메모리를 늘리라는 권장*


ex)DB tibero 연결 및 테스트 
## Database 연결 및 테스트

### 1. 초기 구성
![[jmeter-20240731174807369.webp]]Test Plan 하위에 위 사진과 같이 구성을 만들어줌
- JDBC Connection Configuration
	`Add > Config Elment > JDBC Connection Configuration`
     -- 오라클 DB 접속을 위한 JDBC 접속정보 설정  
- CSV Data Set Config  
	`Add > Config Elment > CSV Data Set Config`  
     -- Insert, Update 등의 테스트를 할때 필요한 데이터 파일을 지정
 - Thread Group
	 `Add > Thread (User) > Thread Group`
     -- 몇개의 동시유저를 띄워서 몇번 반복 할 지 등을 설정
 - JDBC Request
	 `Add > Sampler > JDBC Request`
     -- JDBC 를 통해서 실행할 SQL을 여기에서 설정
- Summary Report
	`Add > Listener > Summary Report`
     -- 실행정보(데이터건수, 에러율, TPS 등)를 실시간으로 볼 수 있음

### 2. JDBC 라이브러리를 등록
![[jmeter-20240731180118512.webp]]
Test Plan > 하단 Browse > lib 폴더 > {tibero}.jar 파일 넣기
> [!jar 파일 얻는 법]
> Tibero 설치 파일에서 (tar 압축 푼 형태에서)
> TmaxData > tibero7 > client > lib > jar > tibero7-jdbc.jar 7

### 3. JDBC Connection Configuration
database JDBC 접속하기 위한 정보 설정
![[jmeter-20240731180845653.webp]]
- <font color="#4f81bd">Variable Name for created pool</font> 항목에 적당한 이름을 입력
	→ 이 이름은 뒤에 <font color="#4f81bd">JDBC Request</font> 설정할 때  같은 이름으로 넣어줘야 함
- <font color="#4f81bd">Max Number of Connections</font> 는 커넥션풀(Connection Pool) 갯수
	Thread(동시사용자)를 100개로 지정하더라도 이 갯수를 10개로 지정하게 되면, DB와의 직접연결은 10개만 생기고 100개의 Thread 가 10개의 커넥션풀을 나눠서 사용하게 됨
	이 값에 0 을 입력하면, Thread 갯수에 맞춰서 커넥션풀 갯수를 자동으로 같게 맞춰줌
- <font color="#4f81bd">Database Connection Configuration</font> 항목에는 JDBC 접속정보를 입력
	- url : `jdbc:tibero:thin:@{IP}:{Port}:{DbName}` 
         ex) jdbc:tibero:thin:@192.168.56.101:8629:tibero
	- JDBC Driver class : `com.tmax.tibero.jdbc.TbDriver`

### 4. CSV Data Set Config
테스트 할 때 필요한 데이터 파일을 생성 후! 지정!
- 생성
![[jmeter-20240731181353981.webp|214]]
구냥 대충 이따구로 만들어줌

- 넣어주기
  ![[jmeter-20240731181512439.webp]]
FileName : 경로와 이름 적어주기 ex) C:\TEMP\test_insert.csv
Variable Names : 사용할 변수 명 적기 ex) 첫번째 컬럼 변수 c1, 두번째 컬럼 변수 c2

### 5. JDBC Request
JDBC 를 통해서 실제로 실행할 SQL 설정
![[jmeter-20240731181945375.webp]]

- <font color="#4f81bd">Variable Name of Pool declared in JDBC Connection Configuration</font>  :<font color="#4f81bd"> JDBC Connection Configuration</font> 에서 지정한 이름을 똑같이 적어줌.
- <font color="#4f81bd">Query Type</font> 선택
	조회 : Select Statement
	Insert/Update/Delete : Update Statement
- <font color="#4f81bd">Query</font> : 항목에 실행할 SQL을 입력
	변수 :  ${변수명} 
	이 변수명은 아래 <font color="#4f81bd">Variable Names</font> 항목에도 입력해줍니다.
	*주의 : 세미콜론 넣지 않기

### 6. 실행 (상단에바에 실행버튼)
### 7. Summary Report




참고
https://jack-of-all-trades.tistory.com/442
