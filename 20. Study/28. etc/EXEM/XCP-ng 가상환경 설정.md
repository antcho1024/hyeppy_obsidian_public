## 개인 서버 만들기

- XCP-ng
    
    물리 서버를 쪼개서 가상화해서 개인 서버로 사용
    
    XenServer(가상 서버 관리자)의 무료 버전 오픈소스 하이퍼바이저
    
- Xshell
    
- **과정 (CentOS 7 기준)**
    

1. XCP-ng 에서 회사에서 할당받은 서버 연결
    
    10.10.52.99 root exem1234
    
2. VM 추가 (CPU, Memory, Disk 설정)
    
3. Console 창 로그인
    
4. IP 설정 (BOOTPROTO, IPADDR, NETMASK, GATEWAY, DNS)
    
    ```powershell
    **vi /etc/sysconfig/network-scripts/ifcfg-eth0
    -------------
    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    
    #BOOTPROTO=dhcp
    BOOTPROTO=static
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_FAILURE_FATAL=no
    IPV6_ADDR_GEN_MODE=stable-privacy
    NAME=eth0
    UUID=4f548152-a670-4011-808b-613dc343648a
    DEVICE=eth0
    
    #ONBOOT=no
    ONBOOT=yes
    NETMASK=255.255.255.0
    GATEWAY=10.10.52.254 ( 본인 ip 대역대 맞게 입력해주세요 
    DNS1=8.8.8.8
    IPADDR=ip주소 입력하세요
    -----------------------
    systemctl restart network**
    ```
    
5. host name 설정
    
    [[Ubuntu/Linux] /etc/hosts의 모든 것](https://storycompiler.tistory.com/118)
    
    ```powershell
    vi /etc/hosts
    ---------
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    본인 ip ( xx.xx.xx.xx ) hostname ( 예 : moa-worker1 )
    ```
    
    - network test
        
        ping 8.8.8.8
        
        - Ping Test에 Ping이 가지 않을 경우
            - Network 재시작
            - Bootproto와 onboot 상태 재확인
        - Ping이 정상적으로 갔지만 사용 불가할 경우
            - IPADDR / GATEWAY / NETMASK 값 재확인
            - 해당 단어의 철자가 틀릴 경우도 사용불가 상태이니 재확인 필요
6. Xshell 연결
    
    - 프로토콜 : SSH
    - 호스트 : XCP VM OS에서 지정한 IP
    
    <aside> 💡 만약 연결이 안된다면
    
    - cent OS 는 오타
    - ubuntu 는 22 포트가 안열려 있는 경우가 많음
    
    </aside>
    

## ISO 추가

- ISO란

<aside> 💡 NFS ISO Network File System : 네트워크에 파일을 저장하는 메커니즘

원래 개별서버에 뒀는데 스토리지 낭비, 중구난방해서 하나 storag에 ISO 를 저장해두고 다 같이 씀

10.10.54.103:/mnt/Disk/ISO

<내 폴더에서 편하게 추가하기>

내pc > 오른쪽 클릭 > 네트워크 드라이브 연결 > 폴더에 \\10.10.54.103

</aside>

### Cent OS 8

- password
    
- Installation Destination → Done
    
- Network $ Host Name
    
    Configuration → IPv4 Settings
    
    - Method : Manual
    - add
    - Address : IP
    - NetMask : 32
    - GateWay : 10.10.55.254
    - DNS : 8.8.8.8
    
    Connect On 키고
    
- Software Selection
    
    - Base Environment > Minimal Install