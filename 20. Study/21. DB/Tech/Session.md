---
sticker: emoji//1f445
---
### Session
Database에는 여러 [[Instance]] 가 존재.
⭐ **이 instance에 사용자가 연결할 때 생기는 논리적 연결 : *<font color="#c0504d">Session</font>***

### SID, SERIAL # 
| SID (Session Identifier) | SERIAL# (Serial Number) |
| ------------------------ | ----------------------- |
| 세션을 고유하게 식별하는 식별자        | 세션에 고유하게 부여되는 번호        |
세션이 종료된 후에도 동일한 SID를 가진 새로운 세션이 시작될 때 정확한 세션 객체를 구분할 수 있도록 해줌.
ex) 세션이 종료된 후 동일한 SID가 다른 세션에 할당될 수 있습니다. 이 경우 SERIAL#는 새로 생성된 세션과 이전 세션을 구분하는 데 사용

참고 : [[동시성 제어]]
