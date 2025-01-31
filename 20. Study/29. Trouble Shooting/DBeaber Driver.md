---
sticker: emoji//1f497
---
⛔ 상황
DBeaver 를 설치 후 SQLite Driver 설치 중 오류 (sample data load를 위해)

*<span style="color:red"> error 내용</span>*
```
Network unavailable due to a certificate issue.
Try changing the setting `Use Windows trust store` in Preferences->Connections and restart DBeaver. It might help if you haven't overridden the trust store.
javax.net.ssl.SSLHandshakeException:PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target

dbeaver can't create driver instance (class 'org.sqlite.jdbc').
```

🟡 시도 (인터넷 서칭 결과)
상단 탭 > 윈도우 > 설정 > 연결 > security > use windows trust store ✔️
하지만 이미 체크 되어 있는 상태였고 아무리 다시 apply, 재기동 해도 안됐음

✅ 해결
상단 탭 > Database > Driver 관리자 > SQLite > Edit > Libraries
여기에서 jar 파일에 마우스 오버 했을 때 N/A 가 나옴

[https://mvnrepository.com/artifact/org.xerial/sqlite-jdbc](https://mvnrepository.com/artifact/org.xerial/sqlite-jdbc)
위의 링크 가서 
Central 에서 원하는 version 클릭 > 상단 표에 Files 에서 jar 파일 다운로드 > 
DBeaver driver edit 창에서 기존(N/A) jar 파일 delete 후 >
다운로드 받은 파일 add > reset to defaults

*(Offline 상태에서 driver 버전을 Update 할 때도 이 방법 사용)*

성공!