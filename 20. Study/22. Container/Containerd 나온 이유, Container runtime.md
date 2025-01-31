# Containerd가 나오게 된 Docker 역사

도커가 2013년에 나온 이후 IT 업계가 많이 달라짐
- APP의 배포단위가 war, jar, zip → Docker Image
- Docker를 사용할 수 있는 환경이기만 하면 APP은 Windows에서든, Ubuntu에서든 동일하게 동작

도커의 기본 아이디어
프로세스와 CPU, memory, disk I/O, network 등의 리소스들을 <span style="background:rgba(240, 200, 0, 0.2)">컨테이너</span>라는 하나의 그룹으로 묶어서 관리 → namespace, cgroup 등 Linux의 여러 커널기능들을 조합하여 만들

이 무렵 구글에서도 <span style="background:rgba(240, 200, 0, 0.2)">컨테이너</span>를 활용해 대규모 서비스 구축 프로젝트 : Project 7 → <font color="#ff0000">Kubernetes</font>

이런 성공신화를 써 내려가던 와중!!

컨테이너를 빌드하고, 실행하고 거기에 네트워크, 스토리지, CLI까지 제공해주는 Docker Engine이라는 패키지가 있었는데, 이게 하나의 패키지로 묶여있다보니 여러 불편함들이 생겨났음
→ Docker에 의존하고 있던 Kubernetes에서는 Docker 버전이 새로 나올때마다 Kubernetes가 크게 영향

따라서 이런 불편함을 없애기 위해

기존의 도커엔진은 Monolithic한 구조를 나누는 작업 하게 됨 (하나로 되어있던 구조에서 container runtime을 분리)

OCI (Open Container Initiative) 라는 container runtime 표준(container 관리자 표준)을 만들었음
→ 이를 따르는 container runtime (container 관리자)가 생겨나기 시작
- <font color="#ff0000">Containerd</font> (docker에서 만듦) (d는 daemon의 d임)
- CRI-O (kubernetes 전용)

👉 containerd로의 전환을 통해 Pod은 더 빨리 시작되고, CPU와 메모리의 사용량은 줄음

# Containerd
containerd, container runtime : 컨테이너 관리자

### KUBERNETES 와 CONTAINERD 통합 아키텍처의 발전
일반적으로 Kubernetes로 구축하는 클러스터에서는 Docker를 이용하여 컨테이너를 실행
→ Kubernetes과 Docker 사이에서는 Kubernetes에서 표준화 된 API 인 <font color="#ff0000">CRI (Container Runtime Interface)</font> 에 의해 교환이 이루어짐
![[Containerd 나온 이유, Container runtime-20240609140008797.webp]]

- Docker : CRI를 기본적으로는 지원하지 않기 때문에 Kubernets과 Docker는 <span style="background:rgba(240, 200, 0, 0.2)">“dockershim”라는 다리를 통해 교환</span>
- containerd 1.0 : kubelet과 containerd사이에서 작동하려면 <span style="background:rgba(240, 200, 0, 0.2)">cri-containerd라는 데몬이 필요</span>
- containerd 1.1 : cri-containerd 데몬이 이제 <span style="background:rgba(240, 200, 0, 0.2)">containerd CRI 플러그인</span>으로 리팩터링
    - CRI 플러그인은 containerd1.1에 내장되어 있으며 기본적으로 활성화

출처
[containerd는 무엇이고 왜 중요할까?](https://www.linkedin.com/pulse/containerd%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B4%EA%B3%A0-%EC%99%9C-%EC%A4%91%EC%9A%94%ED%95%A0%EA%B9%8C-sean-lee/?originalSubdomain=kr)
[CONTAINERD 란](https://yooloo.tistory.com/202)
