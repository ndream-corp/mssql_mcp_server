# mssql_mcp_server
# Microsoft SQL Server MCP 서버

이 서버는 Microsoft SQL Server 데이터베이스와의 안전한 상호작용을 가능하게 하는 모델 컨텍스트 프로토콜(MCP) 서버입니다. 이 서버는 AI 어시스턴트가 테이블을 나열하고, 데이터를 읽고, SQL 쿼리를 실행할 수 있도록 제어된 인터페이스를 제공하여 데이터베이스 탐색 및 분석을 보다 안전하고 구조화된 방식으로 수행할 수 있게 합니다.

## 기능

- 사용 가능한 SQL Server 테이블을 리소스로 나열
- 테이블 내용 읽기
- 적절한 오류 처리를 통한 SQL 쿼리 실행
- 환경 변수를 통한 안전한 데이터베이스 액세스
- 포괄적인 로깅
- 자동 시스템 종속성 설치

## 설치

이 패키지는 MCP를 통해 설치할 때 필요한 시스템 종속성(예: FreeTDS)을 자동으로 설치합니다:

```bash
pip install mssql-mcp-server
```

## 구성

다음 환경 변수를 설정하세요:

```bash
MSSQL_SERVER=localhost
MSSQL_USER=your_username
MSSQL_PASSWORD=your_password
MSSQL_DATABASE=your_database
```

## 사용법

### Claude Desktop과 함께 사용

`claude_desktop_config.json`에 다음을 추가하세요:

```json
{
  "mcpServers": {
    "mssql": {
      "command": "uv",
      "args": [
        "--directory", 
        "path/to/mssql_mcp_server",
        "run",
        "mssql_mcp_server"
      ],
      "env": {
        "MSSQL_SERVER": "localhost",
        "MSSQL_USER": "your_username",
        "MSSQL_PASSWORD": "your_password",
        "MSSQL_DATABASE": "your_database"
      }
    }
  }
}
```

### 독립형 서버로 사용

```bash
# 종속성 설치
pip install -r requirements.txt

# 서버 실행
python -m mssql_mcp_server
```

## 개발

```bash
# 저장소 클론
git clone https://github.com/RichardHan/mssql_mcp_server.git
cd mssql_mcp_server

# 가상 환경 생성
python -m venv venv
source venv/bin/activate  # Windows의 경우 `venv\Scripts\activate`

# 개발 종속성 설치
pip install -r requirements-dev.txt

# 테스트 실행
pytest
```

## 보안 고려사항

- 환경 변수나 자격 증명을 커밋하지 마세요
- 최소한의 필요한 권한을 가진 데이터베이스 사용자 사용
- 프로덕션 사용을 위한 쿼리 화이트리스트 구현 고려
- 모든 데이터베이스 작업 모니터링 및 로깅

## 보안 모범 사례

이 MCP 서버는 기능을 위해 데이터베이스 액세스가 필요합니다. 보안을 위해:

1. 최소 권한을 가진 전용 SQL Server 로그인 생성
2. sa 자격 증명이나 관리자 계정 사용 금지
3. 필요한 작업에만 데이터베이스 액세스 제한
4. 감사 목적을 위한 로깅 활성화
5. 데이터베이스 액세스에 대한 정기적인 보안 검토

자세한 지침은 [SQL Server 보안 구성 가이드](SECURITY.md)를 참조하세요:
- 제한된 SQL Server 로그인 생성
- 적절한 권한 설정
- 데이터베이스 액세스 모니터링
- 보안 모범 사례

⚠️ 중요: 데이터베이스 액세스를 구성할 때 항상 최소 권한 원칙을 따르세요.

## 라이선스

MIT 라이선스 - 자세한 내용은 LICENSE 파일을 참조하세요.

## 기여

1. 저장소 포크
2. 기능 브랜치 생성 (`git checkout -b feature/amazing-feature`)
3. 변경 사항 커밋 (`git commit -m 'Add some amazing feature'`)
4. 브랜치 푸시 (`git push origin feature/amazing-feature`)
5. 풀 리퀘스트 열기