부하테스트 스크립트를 받았다.

### Oracle
*파일 구성*
- **SQL 스크립트:**
    - `/mnt/data/db-load/db-load/oracle/load_temp_tablespace.sql`
    - `/mnt/data/db-load/db-load/oracle/load_undo_tablespace.sql`
    - `/mnt/data/db-load/db-load/oracle/load_lock_test.sql`
    - `/mnt/data/db-load/db-load/oracle/load_insert_delete.sql`
    - `/mnt/data/db-load/db-load/oracle/init_test_table.sql`
- **테스트 실행 스크립트:**
    - `/mnt/data/db-load/db-load/oracle/run_load_test.sh`
- **로그 파일들:**
    - `/mnt/data/db-load/db-load/oracle/logs/session_*_load_*.log`

init_test_table.sql
