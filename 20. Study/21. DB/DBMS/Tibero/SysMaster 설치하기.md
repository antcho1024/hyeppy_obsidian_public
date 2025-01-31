---
sticker: emoji//1f64d
---
목차
- SysMaster DB 8 설치
- TPM Agent 설치
- UI 에서 연결
- 외부 접속 뚫기

### SysMaster DB 8
- 스펙 : [[Tibero 설치하기]] 와 동일
- 설치 방법 : **<font color="#ff0000">도커</font>**, k8s, HyperCloud
- 파일 다운로드 : https://ims.tmaxsoft.com/tody/ims/issue/issueView.do?issueId=327465 

##### 사전 준비

Red Hat 등록
```Shell
sudo subscription-manager register --username=antcho1024 --password=W5bst8gfDCDMaff
```

파일 압축해제 관련
```Shell
sudo yum install epel-release
sudo yum install p7zip

ls -l sysmaster-db-8.1.4-release.7z.*   # 파일 확인 압축 2개, md5 2개 

# (선택 사항) MD5 체크섬 - 다운로드한 파일의 무결성을 확인 
md5sum -c sysmaster-db-8.1.4-release.7z.001.md5
md5sum -c sysmaster-db-8.1.4-release.7z.002.md5

# 압축해제
7za x sysmaster-db-8.1.4-release.7z.001
```

Docker 설치
```Shell
# 필수 패키지 설치
sudo dnf -y install yum-utils device-mapper-persistent-data lvm2

# Docker CE 저장소 추가
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Docker CE 설치
sudo dnf install docker-ce --nobest -y

# Docker 서비스 시작 및 활성화
sudo systemctl start docker
sudo systemctl enable docker

# 설치 확인
docker --version
```

Docker Compose 설치
```Shell
# Docker Compose 바이너리 다운로드
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 실행 권한 부여
sudo chmod +x /usr/local/bin/docker-compose

# 설치 확인
docker-compose --version
```

##### SysMaster 설치

```Shell
docker load -i sysmaster-db-{version}.tar
vi .env # 설정 확인 및 변경
docker-compose up -d
```

확인
![[Pasted image 20240723170248.png]]

접속
{IP}:{CLIENT_PORT}
```
http://192.168.56.102/80
admin/admin
```

![[SysMaster 설치하기-20240723171446159.webp]]
또리링~


### TPM 설치
- Tibero 설치된 VM으로 이동

환경변수 설정
```Shell
vi $TB_HOME/config/$TB_SID.tip

# 추가
SQL_STAT_HISTORY=Y
SQL_STAT_HISTORY_THRESHOLD=100
SQL_STAT_HISTORY_QSIZE=10

# 설명
SQL_STAT_HISTORY=Y              # SQL 실행 정보 생성 여부, Y 필수
SQL_STAT_HISTORY_THRESHOLD=100  # SQL 실행 정보 생성 기준 실행 시간 임계치 100 가능 (예: 100msec 이상 수행된 SQL에 대해서만 실행 정보 생성) - 동적 수정 가능
SQL_STAT_HISTORY_QSIZE=10       # 각 세션마다 저장할 SQL 실행 정보 개수
```

TPM Agent 설치 및 기동

```Shell
# 압축해제
tar -xvzf tpmagent_centos7_8.1.4.tar.gz

# config 파일 수정
vi /tibero/tpmagent_dist/config.cfg

# 주석 달지 않은 변수들은 필수는 아님 installation guide 참고
ID R008             # Instance ID 
IP 192.168.56.102   # SysMaster DB 8 서버에 접속할 수 있는 IP 주소
PORT 8292           # TPM Agent가 SysMaster DB 8 서버의 COLLECTOR_PORT로 연결 포트
LOG ./              # 로그 파일 생성 디렉터리
VERBOSE 3           # 로그 레벨
FREQ 1000           # 수집주기
FROM_CHARSET cp949
TO_CHARSET utf-8
LOG_ROTATE_TIME_INTERVAL 24
LOG_ROTATE_FILE_SIZE 50
MAX_LOG_FILE_NUMBER 7
BOOT_WITH_AUTO_DOWN_CLEAN true

# 코어 덤프 파일 최대 사이즈 설정
ulimit -c unlimited

# 실행
./tpmagent
```

<font color="#ff0000">Trouble Shooting</font>
- installation guide에는 환경변수 설정하는 창이 있는데 Tibero 설치 할 때 설정한 것과 같은 거 같음 (생략가능?)

에러내용
```shell
[tibero@tibero-svr tpmagent_dist]$ ./tpmagent
./tpmagent: error while loading shared libraries: libtpmstat.so: cannot open shared object file: No such file or directory
```
![[SysMaster 설치하기-20240724131210405.webp]]

해결
- 라이브러리 설치 확인, 경로 확인
```shell
find / -name libtpmstat.so 2>/dev/null
# /tibero/tibero7/client/lib/libtpmstat.so

# 환경변수 아래 추가 
export LD_LIBRARY_PATH=/tibero/tibero7/client/lib:$(pwd):$LD_LIBRARY_PAT 
```

내 생각
라이브러리 경로가 원래(Tibero 설치 때) 잘 되어 있었는데`. set.sh` 파일 실행하면서 다른 경로로 임시로 바뀐 것 같다.
다시 경로 똑바로 하고 나니 성공!
![[SysMaster 설치하기-20240724132114407.webp]]


### SysMaster UI 에서 연결
http://192.168.56.102/sysmaster_db/dashboard → Registered Instance → Create
![[SysMaster 설치하기-20240724132446299.webp|325]]

지금은 서비스 내렸는데 올리면 다시 사진 첨부~

### 외부 접속 뚫기
SysMaster 설치한 VM에 
NAT 네트워크 > 포트 포워딩 추가!
![[SysMaster 설치하기-20240814111343340.webp|584]]
- 호스트 IP에는 내 윈도우 컴퓨터 찐 IP 쓰기
- 호스트 포트는 내 윈도우 컴퓨터에서 사용할 포트

이렇게 하니까
[http://192.168.207.131:30080](http://192.168.207.131:30080/)
여기로 접속 가능!

수민님한테 링크 드리고 테스트 해본 결과 외부(회사내)에서 접속 가능!\

정리하자면,
<font color="#4f81bd">192.168.56.102:80</font> → 호스트 전용 네트워크로 VM 끼리 소통 가능하고 내부에서만 접속 가능한 링크
<font color="#4f81bd">192.168.207.131:30080</font> → 외부에서 접속 가능 (Vm끼리 소통 불가능)

둘다 뚫어놓음!