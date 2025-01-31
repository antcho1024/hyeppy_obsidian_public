---
sticker: emoji//1f315
---
# 설치

### <span style="background:#fff88f">사전 설치 목록</span>
- <font color="#0070c0">VM (CentOS7) 위에 설치</font>
  Virtual Box 설치
- <font color="#0070c0">Linux 설치</font>
  tibero지원 가능 os (테크넷 - 설치 메뉴얼에서 조회 가능)
  ex) Red Hat Enterprise Linux 8.1 설치 > https://developers.redhat.com/products/rhel/download
- <font color="#0070c0">JDK 설치 </font>
  ex) jdk 1.8 https://www.oracle.com/java/technologies/downloads/#java8-windows
  설치 후 확인
```
C:\Users\haesoo>java -version
java version "1.8.0_411"
Java(TM) SE Runtime Environment (build 1.8.0_411-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.411-b09, mixed mode)
```


### <span style="background:#fff88f">사전 작업</span>
- <font color="#0070c0">vm 에서 linux 설치 </font>
  새로만들기 > iso 추가, 등등 설정 
  메모리 : 8-16기가 (8192MB)
  CPU : 2개 이상
  Disk : 크게 잡아도 실제로 파일 크기를 그만큼 잡지는 않는다고 함 (일단 40기가 잡아봤음)
- <font color="#0070c0">네트워크 설정</font>
  (NAT  네트워크 사용. VM 끼리 연결되려면 호스트전용 네트워크 같이 사용 해야함 -> [[호스트 전용 네트워크 + NAT 모드]])
  1. virtual box에서 IP 입력해준다.
     어댑터 1을 NAT 모드로 설정 / 포트 포워딩을 아래와 같이 설정
     참고 : NAT -> 	[[NAT 모드]]
```plaintext
이름 : ssh 접속 / 프로토콜 : TCP / 호스트 IP : (공란) / 호스트 포트 : 22 / 게스트 IP : (공란) / 게스트 포트 : 22
이름 : DB 접속 / 프로토콜 : TCP / 호스트 IP : (공란) / 호스트 포트 : 1234 / 게스트 IP : (공란) / 게스트 포트 : 8629
```
	  
2. /etc/sysconfig/network-scripts/ifcfg~~ 에서 IP 수정
```Shell
BOOTPROTO=static       # 고정 IP로 변경
IPADDR=10.0.2.15       # VirtualBox NAT 네트워크에서 사용할 수 있는 주소     
NETMASK=255.255.255.0 
GATEWAY=10.0.2.2
DNS1=8.8.8.8
```
3. 반영 
   `systemctl restart network` / `systemctl restart NetworkManager`


- <font color="#0070c0">기타 시스템 설정</font> 
```Shell
vi /etc/sysctl.conf
kernel.shmmni= 4096
kernel.shmmax=4294967296
kernel.shmall=1048576
kernel.sem = 10000 32000 10000 10000
fs.file-max=6815744
net.ipv4.ip_local_port_range=1024 65500
net.core.rmem_max = 4194304
net.ipv4.tcp_rmem=4194304
net.ipv4.tcp_wmem=1048576

sysctl -p

vi /etc/security/limits.conf
tibero soft  nofile 65536
tibero hard nofile 65536
tibero soft nproc 2047
tibero hard nproc 16384

# 방화벽 해제
systemctl stop firewalld
systemctl disable firewalld
```

### <span style="background:#fff88f">Tibero 설치</span>
- <font color="#0070c0">설치시 필요한 SW 요구사항 확인</font>
```Shell
java -version                      # 1.5.17 ~ 1.9 사이
cat /proc/meminfo | grep MemTotal  # 책 참고
free -m                            
df -k
```

- (root 계정에서) <font color="#0070c0">dba 그룹, tibero 계정, /tibero 디렉토리 생성</font>
```Shell
groupadd -g 1024 dba
useradd -d /tibero tibero -g dba -u 1024  # 책과 root 를 맞추기 위해서 명령어 약간 다름
cat /etc/passwd # ~ tibero:x:1024:1024::/tibero:/bin/bash 
passwd tibero
```

- <font color="#0070c0">tibero 계정 환경 변수 설정</font>
```Shell
su - tibero
id    # uid=1024(tibero) gid=1024(dba) groups=1024(dba) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# 아래 TB_HOME은 오류방지를 위해 pwd 로 경로 확인 후 넣기
vi ~/.bash_profile
export TB_HOME=/tibero/tibero7
export TB_SID=tibero
export TB_PROF_DIR=$TB_HOME/bin/prof
export PATH=.:$TB_HOME/bin:$TB_HOME/clinet/bin:$PATH
export LD_LIBRARY_PATH=$TB_HOME/lib:$TB_HOME/client/lib:$LD_LIBRARY_PATH
export TB_SQLPATH=$TB_HOME/config

source ~/.bash_profile

env | grep TB   # 확인
# TB_SQLPATH=/tibero/tibero7/config
# TB_HOME=/tibero/tibero7
# TB_PROF_DIR=/tibero/tibero7/bin/prof
# TB_SID=tibero
```

- <font color="#0070c0">tibero 설치 tar이랑 license 파일 업로드</font>
```Shell
[tibero@antcho: ~]#pwd
/tibero
[tibero@antcho: ~]#ls
license.xml  tibero7-bin-FS02_PS01-linux64_3.10-272145-20240519221306.tar.gz
```

- <font color="#0070c0">압축해제</font>
```Shell
tar -xvzf tibero7-bin-FS02_PS01-linux64_3.10-272145-20240519221306.tar.gz

# 확인
[tibero@antcho: ~]#ls -l
total 639068
-rw-r--r--. 1 tibero dba       485 Jul 12 15:33 license.xml
drwxr-xr-x. 9 tibero dba      4096 Jul 12 15:35 tibero7
-rw-r--r--. 1 tibero dba 654389879 Jul 12 15:33 tibero7~.tar.gz
```

- <font color="#0070c0">license</font>
```
mv license.xml /tibero/tibero7/license
ls -l /tibero/tibero7/license/license.xml
```
![[Tibero 설치하기-20240719092553165.webp]]
(사진 - 경로 다름 사진은 참고만!!)

- <font color="#0070c0">실행</font>
```
cd $TB_HOME/config
ls -l
gen_tip.sh
```
![[Pasted image 20240712154735.png]]

- <font color="#0070c0">$TB_SID.tip 수정</font>
위를 실행한 결과 $TB_SID.tip (tibero.tip)으로 생성됨
나한테 맞게 수정해주기
(보통은 TB_SID 와 DB_NAME을 동일하게 설정하지만, 실습을 위해 DB_NAME을 다르게 설정해줌)
+메모리 수정?

```Shell
vi $TB_SID.tip

DB_NAME=tibero_db        # DB 이름 확인!!(여기서 쓴 DB 이름 아래서 넣어줘야함.)
LISTENER_PORT=8629
CONTROL_FILES="/tibero/tibero7/database/tibero_db/c1.ctl"
DB_CREATE_FILE_DEST="/tibero/tibero7/database/tibero_db"

MAX_SESSION_COUNT=20      # 디폴트가 20, 10 임. 다를경우만 작성해도 됨
MAX_BG_SESSION_COUNT=10

TOTAL_SHM_SIZE=2G
MEMORY_TARGET=3G

```

이에 따라서 `vi $TB_HOME/client/config/tbdsn.tbr` 도 수정 (DB_NAME)
![[Tibero 설치하기-20240719092741715.webp]]

- <font color="#0070c0">Database 실행</font>
```Shell
id # uid=1024(tibero) gid=1024(dba) groups=1024(dba)
tbboot nomount 
cd /tibero/tibero7/client/bin
tbsql sys/tibero
# 에러 날 경우 아래 trouble shooting 참고
```

### <span style="background:#fff88f">Database 생성</span>
- <font color="#0070c0">생성</font>
```SQL
-- DB name 확인하기!

create database "tibero_db"
user sys identified by tibero
maxinstances 8
maxdatafiles 100
character set MSWIN949
national character set UTF16
logfile
group 1 'log001.log' size 50M,
group 2 'log002.log' size 50M,
group 3 'log003.log' size 50M
maxloggroups 255
maxlogmembers 8
noarchivelog
datafile 'system001.dtf' size 100M autoextend on next 10M maxsize unlimited
default temporary tablespace TEMP
tempfile 'temp001.dtf' size 100M autoextend on next 10M maxsize unlimited
extent management local autoallocate
undo tablespace UNDO
datafile 'undo001.dtf' size 200M autoextend on next 10M maxsize unlimited
extent management local autoallocate
SYSSUB
datafile 'syssub001.dtf' size 10M autoextend on next 10M maxsize unlimited
default tablespace USR
datafile 'usr001.dtf' size 100M autoextend on next 10M maxsize unlimited
extent management local autoallocate;
```

*위 코드 설명*
```SQL
-- 데이터베이스 생성 및 사용자 정의
-- 데이터베이스 이름을 "tibero_db", 유저 : sys, 비번 : tibero
create database "tibero_db"
user sys identified by tibero

maxinstances 8 -- 최대 8개의 인스턴스를 허용
maxdatafiles 100 -- 최대 100개의 데이터 파일을 허용
character set MSWIN949 -- 문자 집합 설정
national character set UTF16 -- 국가 문자 집합 설정

-- 로그파일 설정 
-- 3개의 로그 파일 그룹을 각각 50MB 크기로 생성 
logfile
group 1 'log001.log' size 50M,
group 2 'log002.log' size 50M,
group 3 'log003.log' size 50M
maxloggroups 255   -- 최대 255개의 로그 그룹을 허용
maxlogmembers 8    -- 각 로그 그룹당 최대 8개의 로그 멤버를 허용
noarchivelog       -- 아카이브 로그 사용 x

-- 데이터 파일 설정 (데이터 파일을 100MB 크기로 생성하고, 필요시 10MB씩 자동으로 확장(크기 무제한))
datafile 'system001.dtf' size 100M autoextend on next 10M maxsize unlimited

-- 임시 테이블스페이스 설정
default temporary tablespace TEMP
tempfile 'temp001.dtf' size 100M autoextend on next 10M maxsize unlimited
extent management local autoallocate

-- UNDO 테이블스페이스 설정
undo tablespace UNDO
datafile 'undo001.dtf' size 200M autoextend on next 10M maxsize unlimited
extent management local autoallocate

-- 시스템 서브 테이블스페이스 설정
SYSSUB
datafile 'syssub001.dtf' size 10M autoextend on next 10M maxsize unlimited

-- 사용자 테이블스페이스 설정
default tablespace USR
datafile 'usr001.dtf' size 100M autoextend on next 10M maxsize unlimited
extent management local autoallocate;
```

다 만들면 나오기!
`exit`

- <font color="#0070c0">Data Dictionary 만들기</font>
```Shell
ps -ef | grep tbsvr
# tibero     34691   16223  0 16:59 pts/1    00:00:00 grep --color=auto tbsvr
tbboot

sh $TB_HOME/scripts/system.sh

# Enter SYS password:
tibero
# Enter SYSCAT password:
syscat
# 어쩌구
Y
```

### <span style="background:#fff88f">Tibero Studio3 연결</span>
- Host : 127.0.0.1
- Port : 1234
- Database : tibero_db
- username : sys
- password : tibero
  
[[DBeaver에 Tibero 연결하기]] 로 DBeaver에도 연결해보기~

# Trouble Shooting
- <font color="#0070c0">tibero 계정 환경 변수 설정 반영 하는 부분</font>
*<font color="#c0504d">에러 내용</font>*
```Shell
[tibero@antcho: ~]#~/.bash_profile
-bash: /home/tibero/.bash_profile: Permission denied
[tibero@antcho: ~]#.~/.bash_profile
-bash: .~/.bash_profile: No such file or directory
```
책이 잘못 된건가?
`source ~/.bash_profile` 임

- <font color="#0070c0">잘 되다가 vm, 컴 껐다가 키니까 네트워크 연결 안됨</font>
```Shell
IPADDR=10.0.2.15 
NETMASK=255.255.255.0 
GATEWAY=10.0.2.2 
DNS1=8.8.8.8
```
설정하고 연결 완료

- <font color="#0070c0">hostname</font>
  ![[Tibero 설치하기-20240719092826452.webp|612]]
license 파일 열어보니 hostname 이 tibero-svr임
`hostnamectl set-hostname tibero-svr` 로 변경

- <font color="#0070c0">tbsql 을 실행하기 위한 libncurses.so.5 라이브러리를 찾을 수 없음</font>
root 에서
```Shell
# RHEL에서는 패키지를 설치하기 위해 시스템이 Red Hat Subscription Management에 등록 필요
sudo subscription-manager register --username=antcho1024 --password=W5bst8gfDCDMaff
sudo subscription-manager attach --auto

# 라이브러리 설치
sudo yum install epel-release
sudo yum install ncurses-compat-libs -y 
sudo yum install ncurses

# 라이브러리가 설치된 위치에서 `libncurses.so.5`로 심볼릭 링크를 생성
# 라이브러리가 설치된 위치가 /usr/lib64 인지 /usr/lib 인지 확인 해야함
grep -rl 'libncurses.so.6' /usr

# 확인 후 둘 중 하나
# sudo ln -s /usr/lib64/libncurses.so.6 /usr/lib64/libncurses.so.5
# sudo ln -s /usr/lib/libncurses.so.6 /usr/lib/libncurses.so.5
```
root 계정에서 이 같은 과정을 거친후 다시 tibero 계정가서 해보면 성공
이과정해서도 안되면 환경변수 설정 해줘야할수도

- <font color="#0070c0">Network 에러 날 경우</font>
```
cat /etc/resolv.conf # 에 nameserver 확인 후 없으면
vi /etc/resolv.conf
nameserver 8.8.8.8 # 추가
```

- <font color="#0070c0">Studio 연결 안됨 문제.. </font>
결론부터 말하자면 포트 포워딩 문제 였음..
생각해보면 당연한건데 멍청이 같았다..

문제의 포트포워딩 
> 이름 : Rule 1 / 프로토콜 : TCP / 호스트 IP :  / 호스트 포트 : 1234 / 게스트 IP :  / 게스트 포트 : 22

이따구로 설정했다..
솔직히 찾아보고 고친게 아니라 이것저것 해보고 생각해보다 보니 어? 하게 된거라 맞는지는 잘 모르겟다. -> 틀림 ㅎ
![[Tibero 설치하기-20240718183430323.webp|485]]
위와 같은 상황인거 같은데
내가 127.0.0.1(localhost) 의 1234 번 포트로 접속했을때 내 VM의 22번 포트로 연결해주겠다! 가 
포트 포워딩이다.
근데 티베로의 리스너 포트를 8629로 열어뒀음
host에서 아무리 연결시도해도 안된듯 하다..
이를 테스트하고 알게된건 telnet 테스트이다.. 이거까지 디테일 하게 쓰긴 귀찮으니 생략
여튼 그래서 
포트 포워딩의 게스트 포트를 8629 로 바꾸고 
tool 에서 db 접속정보 입력시 127.0.0.1:1234 함 <font color="#ff0000">(-> 틀림)</font>
![[Tibero 설치하기-20240718182334939.webp|484]]

![[Tibero 설치하기-20240718182830616.webp]]
결국 성공!!

했지만! 이마저도 틀렸다.
이번엔 host의 1234 port를 8629(DB 연결을 위한 port)만 열어두니 SSH 접속이 안됨

<font color="#ff0000">해결방법</font>
![[Tibero 설치하기-20240719140031618.webp|510]]
이것도 해결하고 생각해보면 당연한건데 개멍청했음
포트 포워딩으로 내 host의 어떤 port와 VM의 어떤 포트를 연결해두면 거기로 이러케 저러케 해줌

<font color="#ff0000">그럼 기본적으로 ssh 접속할 포트를 22로 뚫어두고</font>
<font color="#ff0000">db 접속할 포트 뚫어두고 app 설치할때마다 걔는 포트가 생길테니 나의 host와 port를 연결해주면 되는것!</font>

결론적으로 내가 사용한 IP 흔적들을 기록하겟다..

(NAT 모드 일경우)
- 포트 포워딩 
![[Tibero 설치하기-20240719140355481.webp|510]]
- /etc/sysconfig/network-scripts/ifcfg-enp0s3
  BOOTPROTO=static 
  IPADDR=10.0.2.15 
  NETMASK=255.255.255.0 
  GATEWAY=10.0.2.2 
  DNS1=8.8.8.8
- ssh 접속
  IP : 127.0.0.1
  user : root
  port : 22
- FileZilla 접속
  호스트 : sftp://127.0.0.1  (기본 설정인 ftp 하면 에러남)
  사용자 : tibero tibero / root root
  port : 22
- studio
  host : 127.0.0.1
  1234
  db : tibero_db
  user : sys
  pwd : tibero
사실 이것(NAT 모드)도 VM 위의 guest OS끼리는 통신할 수 없음
-> 그럴 땐! [[호스트 전용 네트워크 + NAT 모드]]
