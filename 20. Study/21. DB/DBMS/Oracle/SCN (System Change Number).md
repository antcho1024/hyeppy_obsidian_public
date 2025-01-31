

### SCN (System Change Number)
DB의 변경이 발생한 시점(Commit 발생 시, 트랜잭션 발생 시)에 생성되는 타임스탬프 혹은 고유 번호

### SCN 종류
**current SCN**
시스템의 현재 SCN. 지속적으로 증가한다.

**system checkpoint SCN**
마지막 [[CheckPoint]]시점의 SCN


**datafile checkpoint SCN**
*data file : 실제 데이터가 저장되는 디스크 상의 물리적 파일*
데이터 파일에 있는 SCN

**controlfile checkpoint SCN**
*Control File: DB의 제어 정보를 가지고 있는 파일*
컨트롤 파일에 있는 데이터 파일의 SCN


**start SCN**
데이터파파일에 있는 SCN

**stop SCN**
DB가 운영 중이면 stop SCN 값은 null이다. DB를 정상 종료할 경우 .stop SCN은 start SCN과 같은 값을 가진다. 만약 abort 옵션으로 DB를 내리게 된다면 stop SCN 값은 무한대를 가진다.

쓰이는 곳:
- 트랜잭션 관리
- 데이터 복구
- 읽기 일관성 유지