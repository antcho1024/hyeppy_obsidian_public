
###### 기본
- 파일 열기
    - vi  파일이름
- **편집하기**
    - i
- **수정완료 후 종료하기**
    - ESC
- 저장하기
    - :wq
    - : 커서맨아래로 이동
    - w 저장하기
    - q 나가기
    - wq 저장하고 종료하기
    - q! 저장하지 않고 나가기
- **되돌리기**
    - u
- 폴더 생성
	- mkdir
- OS 확인
	- cat /etc/os-release
- 
###### yum 관련
- docker 와 관련된 추가한 레포 list 보기
    yum list installed | grep docker
- yum-config-manager : yum 저장소 cli 도구 (yum-utils install 로 설치 가능?)
- yum repo 저장되어 있는 곳 : /etc/yum.repos.d/
- 설치할 수 있는 도커 버전들 보기
  yum list docker-ce --showduplicates | sort -r
  sort -r 은 버전 내림차순
  showduplicates 는 동일한 이름의 패키지를 모두 표시

- 레포에 내가 원하는 버전이 있는지 알아보기
    `yum list docker-ce --showduplicates | grep 20.10.4`
- repo 업데이트 했는데 인식? 안될 때
    ```bash
    yum clean all
    yum repolist
    ```
- yum으로 설치한 package list
    `yum list installed`
- yum history보기

```sehll
    yum history
    yum history info 3
```

###### 권한 관련
- 어떤 파일 아래 있는 스크립트 전체 실행권한 주기
  `find . -type f -name "*.sh" -exec chmod +x {} \\;`
###### Network
- restart
  systemctl restart network (centOS7)
  systemctl restart NetworkManager (centOS8)
###### 시스템
- sysctl : 런타임중에 커널 매개변수를 보고, 설정하고, 관리할 수 있는 명령어 (시스템을 재푸팅할 필요 없음)
  https://zidarn87.tistory.com/583
- hostname 변경
	- /etc/hosts → 얘는 쿠베를 위한 hostname 변경? 임
	- hostnamectl set-hostname {호스트네임} # 얘 적용하고 나면 cat /etc/hostname 여기에도 변경되어 있음 (얘는 변경하고 나서도 restart 안해도 됨)
- ~/.bashrc
	- 명령어 alias
		  - **임시** : `alias k=kubectl`
		  - **영구**
		  `vi ~/.bashrc` 
		  `alias k=kubectl` # 추가 
		  `source ~/.bashrc`
	- 터미널창 색 꾸미기
	  export PS1="[\[\e[1;31m\]\u\[\e[m\]@\[\e[1;32m\]\h\[\e[m\]: \[\e[1;36m\]\w\[\e[m\]]#"
- repo 인식 못할시
	1. 기존 레포 삭제 or 백업
```Shell
mkdir ~/repo-backup mv /etc/yum.repos.d/*.repo ~/repo-backup/
```
	1. 새 repo 작성
```Shell
vi /etc/yum.repos.d/CentOS-Base.repo

[base] name=CentOS-$releasever - Base baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/ gpgcheck=1 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates [updates] name=CentOS-$releasever - Updates baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/ gpgcheck=1 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful [extras] name=CentOS-$releasever - Extras baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/ gpgcheck=1 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```
	1. YUM 캐시 클리어 및 리포지토리 메타데이터 업데이트
```Shell
yum clean all 
yum makecache
```
	1. 확인
```Shell
yum repolist
```

###### Kubernetes
- containerd ctr 명령어
    nerdctl 파일 → bin 폴더에 넣기
    - image list 보기
        ```bash
        ctr -n=k8s.io images list
        ```
        
    - 어떤 네임스페이스에 있는 image 전체 삭
        - ctr -n [k8s.io](http://k8s.io/) i rm $(ctr -n [k8s.io](http://k8s.io/) i ls -q)
    - 실행중인 컨테이너 목록 보기
        - ctr c list
- pod
	- 재시작
		- k rollout restart
- nerdctl
  설치
```Shell
yum install wget wget https://github.com/containerd/nerdctl/releases/download/v0.12.0/nerdctl-0.12.0-linux-amd64.tar.gz tar -xvf nerdctl-0.12.0-linux-amd64.tar.gz sudo mv nerdctl /usr/local/bin/ sudo chmod +x /usr/local/bin/nerdctl
```
- tag명령어로 이미지 이름 바꾸기
  nerdctl tag <이전 이미지 이름>:<이전 태그> <새 이미지 이름>:<새 태그>
  nerdctl images
- image, tar
  **image → tar**
  nerdctl save -o <파일명.tar> <이미지 이름>
  **tar → image**
  nerdctl load -i <파일명.tar>
- crictl
    crictl config --set runtime-endpoint=unix:///var/run/containerd/containerd.sock
###### etc

###### Trouble Shooting
 - <span style="background:rgba(240, 200, 0, 0.2)">백그라운드로 보낸 파일 복구 및 처리</span>
<font color="#0070c0">상황</font> 
   A 파일을 vi로 편집하다가 Cntrl + Z 키를 잘 못 누름
```
E325: ATTENTION
Found a swap file by the name "~/.bash_profile.swp"
          owned by: tibero   dated: Mon Jul 15 09:55:56 2024
         file name: ~tibero/.bash_profile
          modified: YES
         user name: tibero   host name: choTest.myguest.virtualbox.org
        process ID: 2768 (still running)
While opening file "/home/tibero/.bash_profile"
             dated: Sat Feb 10 17:12:56 2024

(1) Another program may be editing the same file.  If this is the case,
    be careful not to end up with two different instances of the same
    file when making changes.  Quit, or continue with caution.
(2) An edit session for this file crashed.
    If this is the case, use ":recover" or "vim -r /home/tibero/.bash_profile"
    to recover the changes (see ":help recovery").
    If you did this already, delete the swap file "/home/tibero/.bash_profile.swp"
    to avoid this message.

Swap file "~/.bash_profile.swp" already exists!
[O]pen Read-Only, (E)dit anyway, (R)ecover, (Q)uit, (A)bort:
```
   다시 들어가니 아래와 같은 오류 뜸
	-  <font color="#0070c0">원인</font>
		프로세스를 백그라운드로 보냈기 때문
	 - <font color="#0070c0">해결</font>
	   1. fg 혹은 fg %1 -> 백그라운드로 보내진 첫번째 프로세스를 포어그라운드로 갖고옴.
	   2. 처리 (:q! 등등)
	   3. 다시 fg 해보면 -bash: fg: %1: no such job 라고 뜸
	   4. 혹시 이래도 