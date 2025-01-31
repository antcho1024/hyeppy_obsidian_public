### Privilege(권한)

- **정의**: Privilege는 데이터베이스 객체에 대한 특정 작업을 수행할 수 있는 권한을 의미합니다.
- **종류**:
    - **시스템 권한(System Privileges)**: 데이터베이스 전체에 영향을 미치는 작업에 대한 권한입니다. 예를 들어, 사용자 생성, 테이블 생성, 데이터베이스 시작/중지 등이 포함됩니다.
    - **객체 권한(Object Privileges)**: 특정 데이터베이스 객체(테이블, 뷰, 시퀀스 등)에 대한 작업을 수행할 수 있는 권한입니다. 예를 들어, SELECT, INSERT, UPDATE, DELETE 등이 포함됩니다.

role = privilege의 묶음
- 사용자나 다른 Role에 할당될 수 있습니다.

https://velog.io/@sezzzini/DB-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B6%8C%ED%95%9C-System-Privileges-%EB%A1%A4-Role