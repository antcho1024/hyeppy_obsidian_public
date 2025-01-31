Linux (Red Hat)에 PostrgreSQL 설치하기
- 환경: [[Tibero 설치하기]] 여기 서버에 설치 해보겠삼..


**설치**
```shell
yum install postgresql-server
```

**폴더 생성**
```shell
mkdir -p /data/pgsql
```


```shell
cat /etc/passwd | grep postgres
...
chown postgres.postgres /data/pgsql
...
ll /data
```
![[PostgreSQL 설치하기-20250120103639351.webp]]


```shell
# 계정 전환
su - postgres

# $ 빼야함
# DB 생성, 기동
$ initdb --pgdata=/data/pgsql
```
![[PostgreSQL 설치하기-20250120103901962.webp]]

**postgresql start**
```shell
pg_ctl -D /data/pgsql start
```
![[PostgreSQL 설치하기-20250120103935982.webp]]

**DB 상태 확인**
```shell
# Root 계정으로 다시 가서!

netstat -nlp | grep postgres
ps -ef | grep postgres
psql -h /tmp -p 5432 -U postgres -l
```
![[PostgreSQL 설치하기-20250120104336325.webp]]








참고 블로그: [링크](https://justdaily.tistory.com/13)
