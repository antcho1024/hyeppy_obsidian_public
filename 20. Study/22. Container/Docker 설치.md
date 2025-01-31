
1. 도커 설치
    
    ```powershell
    yum install -y docker-ce docker-ce-cli
    systemctl start docker
    systemctl enable docker
    systemctl status docker
    ```
    
2. image pull
    
    ```powershell
    docker pull nginx
    ```
    
3. container 실행
    
    ```powershell
    docker run -d --name my-nginx -p 8080:80 nginx
    ```
			
	참고 :[[도커 컨테이너 생명주기]]
	    - 분리모드 (-d) : Background 로 실행
		 - name : 이름 설정
		- p : port 설정 ( {host port} : {container port} )   
			
	---
	
```

```
``

