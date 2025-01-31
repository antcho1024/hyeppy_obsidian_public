**- host port, container port** 란?
    
    host port : Docker 호스트 머신(Docker를 실행하는 머신)의 포트
    container port : Docker 컨테이너 내부의 애플리케이션이 수신 대기하는 포트
    
    → kubernetes 세계관에서의 container port와 겹쳐도 상관 없음
    
     ![[Port-20240609123143095.webp]]
    
    
**- host port, container port 포드 포워딩**
    
    *ex) host port 가 80, container port : 5000 인 경우*
    
     ![[Port-20240609123246086.webp]]
    
    호스트 포드 80 과 컨테이너 포드 5000을 연결 시키는 것.
    
    **근데 만약!!!! port가 정해져 있는 컨테이너라면!** ex) MYSQL → 3306 포트 고정

    ![[Port-20240609124043851.webp]]
    
    다른 포트를 사용한다면 접속 불가능
    
		![[Port-20240609124212513.webp]]
		
    ```powershell
    docker run -e MYSQL_ROOT_PASSWORD=1 -p 80:3306 --name test.mysql mysql:latest
    ```
    
    정해진 포트가 있는건 그 포트로 해주는것 주의!
    
    출처 : [https://blog.naver.com/alice_k106/220278762795](https://blog.naver.com/alice_k106/220278762795)