## Microsoft SQL Server 보안 구성

### 제한된 SQL Server 로그인 생성

MCP 서버를 위해 최소 권한을 가진 전용 SQL Server 로그인을 생성하는 것이 중요합니다. 'sa' 계정이나 전체 관리자 권한을 가진 로그인을 사용하지 마십시오.

#### 1. 새로운 SQL Server 로그인 생성

```sql
-- sysadmin 또는 관리자 권한으로 연결
CREATE LOGIN mcp_login WITH PASSWORD = 'your_secure_password';

-- 로그인에 대한 데이터베이스 사용자 생성
USE your_database;
CREATE USER mcp_user FOR LOGIN mcp_login;
```

#### 2. 최소 필요 권한 부여

기본 읽기 전용 액세스(탐색 및 분석에 권장):
```sql
-- SELECT 권한만 부여
USE your_database;
GRANT SELECT TO mcp_user;
```

표준 액세스(데이터 수정 가능하지만 구조적 변경 불가):
```sql
-- 데이터 조작 권한 부여
USE your_database;
GRANT SELECT, INSERT, UPDATE, DELETE TO mcp_user;
```

고급 액세스(복잡한 쿼리를 위한 임시 객체 생성 가능):
```sql
-- 고급 작업을 위한 추가 권한 부여
USE your_database;
GRANT SELECT, INSERT, UPDATE, DELETE TO mcp_user;
GRANT CREATE TABLE TO mcp_user;  -- 임시 테이블용
```

### 추가 보안 조치

1. **네트워크 액세스**
   - 특정 IP 주소에서만 SQL Server 연결 허용
   - Windows 방화벽을 사용하여 포트 액세스 제한
   - 민감한 데이터에 대해 Always Encrypted 사용 고려

2. **쿼리 제한**
   - 데이터베이스 역할을 사용하여 권한 관리
   - 세분화된 액세스 제어를 위한 행 수준 보안(RLS) 구현:
   ```sql
   -- RLS 구현 예시
   CREATE SCHEMA Security;
   GO
   
   CREATE FUNCTION Security.fn_securitypredicate(@user VARCHAR(50))
   RETURNS TABLE
   WITH SCHEMABINDING
   AS
   RETURN SELECT 1 AS fn_securitypredicate_result
   WHERE @user = USER_NAME();
   GO
   
   CREATE SECURITY POLICY Security.TablePolicy
   ADD FILTER PREDICATE Security.fn_securitypredicate(UserColumn)
   ON dbo.YourTable;
   ```

3. **데이터 액세스 제어**
   - 열 액세스를 제한하기 위해 뷰 사용
   - 필요한 경우 열 수준 암호화 구현
   - 가능한 경우 스키마 수준 권한 사용:
   ```sql
   GRANT SELECT ON SCHEMA::YourSchema TO mcp_user;
   ```

4. **정기 감사**
   - SQL Server 감사 활성화
   - 모니터링을 위한 확장 이벤트 사용
   - 감사 로그를 정기적으로 검토

### 환경 구성

MCP 서버를 설정할 때, 다음과 같은 제한된 자격 증명을 환경에 사용하십시오:

```bash
MSSQL_SERVER=your_server
MSSQL_USER=mcp_login
MSSQL_PASSWORD=your_secure_password
MSSQL_DATABASE=your_database
```

### 사용 모니터링

MCP 로그인 데이터베이스 사용을 모니터링하려면:

```sql
-- 현재 연결 확인
SELECT * FROM sys.dm_exec_sessions
WHERE login_name = 'mcp_login';

-- 사용자 권한 보기
SELECT * FROM sys.database_permissions
WHERE grantee_principal_id = USER_ID('mcp_user');

-- 쿼리 실행 모니터링
SELECT * FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle)
WHERE session_id IN (
    SELECT session_id FROM sys.dm_exec_sessions
    WHERE login_name = 'mcp_login'
);
```

### 모범 사례

1. **인증 보안**
   - 가능한 경우 Windows 인증 사용
   - 비밀번호 정책 구현
   - TLS 1.2 이상 연결 활성화
   - 자격 증명 정기적으로 회전

2. **권한 관리**
   - 최소 권한 원칙 준수
   - 권한 정기적으로 검토 및 감사
   - 적절한 경우 포함된 데이터베이스 사용
   - 적절한 직무 분리 구현

3. **모니터링 및 감사**
   - 민감한 작업에 대한 SQL Server 감사 설정
   - 실패한 로그인 시도 모니터링
   - 스키마 변경 추적
   - 자세한 모니터링을 위한 확장 이벤트 사용

4. **데이터 보호**
   - 민감한 열에 대해 Always Encrypted 사용
   - 투명한 데이터 암호화(TDE) 구현
   - 백업 암호화 사용
   - 동적 데이터 마스킹 사용 고려:
   ```sql
   ALTER TABLE YourTable
   ALTER COLUMN SensitiveColumn ADD MASKED WITH (FUNCTION = 'default()');
   ```

5. **네트워크 보안**
   - 모든 연결에 대해 암호화 사용
   - 적절한 방화벽 규칙 구성
   - Azure SQL을 사용하는 경우 Azure Private Link 사용 고려
   - 적절한 네트워크 분할 구현

### 정기 보안 검토

1. **주간 작업**
   - 실패한 로그인 시도 검토
   - 무단 스키마 변경 확인
   - 리소스 사용 패턴 모니터링

2. **월간 작업**
   - 사용자 권한 감사
   - 보안 정책 검토
   - 새로운 보안 패치 확인
   - 백업 암호화 검증

3. **분기별 작업**
   - 종합적인 보안 평가
   - 보안 정책 검토 및 업데이트
   - 필요 시 침투 테스트
   - 문서 업데이트

항상 조직의 보안 정책 및 준수 요구 사항을 이 지침과 함께 따르십시오.
