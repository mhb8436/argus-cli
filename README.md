# Argus CLI

API 보안 취약점 점검 도구. Burp Suite와 유사한 기능을 명령줄(CLI)에서 제공합니다.

## 주요 기능

- **Proxy** - 브라우저 트래픽 캡처 (HTTP/HTTPS MITM)
- **Scanner** - 253개 YAML 기반 보안 룰로 자동 취약점 스캔 (Passive / Active)
- **Intruder** - Payload 자동 주입 (Sniper, Battering Ram, Pitchfork, Cluster Bomb)
- **Repeater** - HTTP 요청 수정 및 재전송
- **Sequencer** - 토큰 엔트로피 분석
- **Crawler** - 웹사이트 자동 크롤링
- **Report** - JSON / HTML / Excel 보고서 생성
- **JWT Analyzer** - JWT 토큰 보안 분석
- **Fingerprinter** - 서버/프레임워크 식별
- **Directory Bruter** - 디렉토리/파일 탐색
- **Decoder** - Base64, URL, HTML 인코딩/디코딩 및 해시 계산

## 설치

### 다운로드

[Releases](https://github.com/mhb8436/argus-cli/releases) 페이지에서 OS에 맞는 바이너리를 다운로드합니다.

| OS | Architecture | 파일 |
|----|-------------|------|
| Windows | x86_64 | `argus-cli-windows-amd64.exe` |
| macOS | Apple Silicon (M1/M2/M3/M4) | `argus-cli-darwin-arm64` |
| macOS | Intel | `argus-cli-darwin-amd64` |

### macOS / Linux 설치

```bash
# 다운로드 후 실행 권한 부여
chmod +x argus-cli-darwin-arm64
sudo mv argus-cli-darwin-arm64 /usr/local/bin/argus-cli

# 확인
argus-cli --version
```

### Windows 설치

1. `argus-cli-windows-amd64.exe` 다운로드
2. 원하는 디렉토리에 저장 (예: `C:\Tools\`)
3. 시스템 PATH에 해당 디렉토리 추가
4. PowerShell에서 확인:
```powershell
argus-cli-windows-amd64.exe --version
```

---

## 빠른 시작

### 1. 프로젝트 초기화

```bash
argus-cli init my-project
```

### 2. URL 직접 스캔

```bash
# Passive 스캔 (응답 분석만, 안전)
argus-cli scan passive http://target.com

# Active 스캔 (공격 payload 전송, 주의)
argus-cli scan active http://target.com

# 크롤링 + 전체 스캔
argus-cli scan full http://target.com
```

### 3. 프록시 기반 스캔 (권장)

브라우저로 직접 탐색하며 트래픽을 캡처하고, 캡처된 데이터를 기반으로 스캔합니다.

```bash
# Step 1: 프록시 시작
argus-cli proxy start --port 8082

# Step 2: 브라우저를 프록시로 연결
#   Chrome 예시 (모든 프로세스 종료 후 실행):
#   macOS:
open -na "Google Chrome" --args \
  --proxy-server="http://127.0.0.1:8082" \
  --proxy-bypass-list="" \
  --ignore-certificate-errors
#   Windows:
#   chrome.exe --proxy-server="http://127.0.0.1:8082" --proxy-bypass-list=""

# Step 3: 브라우저에서 대상 사이트 탐색 (로그인, 기능 사용 등)

# Step 4: 캡처 확인
argus-cli proxy history

# Step 5: 캡처된 트래픽으로 스캔
argus-cli scan passive --from-proxy --host target.com
argus-cli scan active --from-proxy --host target.com

# Step 6: 결과 확인 및 보고서
argus-cli scan results
argus-cli report generate -f xlsx -o report.xlsx
```

> **Chrome localhost 우회 문제**: Chrome은 `localhost`에 대해 프록시를 자동 우회합니다.
> 로컬 서버를 테스트할 경우 `/etc/hosts`에 `127.0.0.1 myapp.local` 추가 후 해당 호스트명을 사용하세요.

### 4. HTTPS 대상 프록시 (MITM)

```bash
# CA 인증서 생성
argus-cli proxy ca generate

# MITM 모드로 프록시 시작
argus-cli proxy start --port 8082 --mitm

# 브라우저에 CA 인증서 설치 필요
argus-cli proxy ca export
```

---

## 주요 명령어

### Scanner

```bash
argus-cli scan passive <url>                  # Passive 스캔
argus-cli scan active <url>                   # Active 스캔
argus-cli scan full <url>                     # 크롤링 + 전체 스캔
argus-cli scan crawl <url> --depth 3          # 크롤링만

argus-cli scan passive --from-proxy           # 프록시 트래픽 Passive 스캔
argus-cli scan active --from-proxy --host x   # 프록시 트래픽 Active 스캔

argus-cli scan results                        # 결과 조회
argus-cli scan results --severity HIGH        # 심각도 필터
argus-cli scan rules list                     # 룰 목록
```

### Proxy

```bash
argus-cli proxy start --port 8082             # 프록시 시작
argus-cli proxy start --port 8082 --mitm      # HTTPS MITM 프록시
argus-cli proxy history                       # 캡처 히스토리
argus-cli proxy ca generate                   # CA 인증서 생성
```

### Intruder

```bash
argus-cli intruder run \
  --type sniper \
  --request req.txt \
  --payloads payloads.txt \
  --concurrency 10
```

**공격 타입:**
- `sniper` - 한 위치씩 순차 주입
- `battering-ram` - 모든 위치에 동일 값
- `pitchfork` - 여러 리스트 병렬 주입
- `cluster-bomb` - 모든 조합

### Repeater

```bash
argus-cli repeater send req.txt               # 요청 전송
argus-cli repeater edit                       # TUI 편집기
argus-cli repeater history                    # 히스토리
```

### Report

```bash
argus-cli report generate -f xlsx -o report.xlsx    # Excel
argus-cli report generate -f html -o report.html    # HTML
argus-cli report generate -f json -o report.json    # JSON
```

### 기타

```bash
argus-cli scan dirbrute <url>                 # 디렉토리 브루트포스
argus-cli scan fingerprint <url>              # 서버 핑거프린팅
argus-cli scan jwt <token>                    # JWT 분석
argus-cli scan import-spec <file>             # OpenAPI/Swagger 임포트

argus-cli decoder encode "text" --type base64 # 인코딩
argus-cli decoder decode "dGV4dA==" --type base64  # 디코딩
argus-cli decoder hash "text" --type sha256   # 해시

argus-cli sequencer collect --url <url> --extract "Set-Cookie" --count 100
argus-cli sequencer analyze
```

---

## 보안 룰

253개 YAML 기반 보안 룰이 내장되어 있습니다.

| 카테고리 | 룰 수 | 예시 |
|---------|-------|------|
| SQL Injection | 15+ | Error-based, Union-based, Blind time-based |
| Cross-Site Scripting | 10+ | Reflected, DOM-based, Encoding bypass |
| NoSQL Injection | 4 | MongoDB operator, Auth bypass, Regex |
| CORS Misconfiguration | 3 | Wildcard origin, Null origin, Credentials |
| Response Header Security | 8 | CORS, Server version, JWT exposure, Stack trace |
| Authentication | 10+ | JWT expiration, Token in response, Auth bypass |
| Server-Side Template Injection | 1 | Generic SSTI detection |
| Local File Inclusion | 2+ | Path traversal, Header-based LFI |
| GraphQL | 4 | Introspection, Batch query, Depth limit |
| Rate Limiting | 2 | Login, API rate limit |
| ... | | |

---

## 프로젝트 관리

```bash
argus-cli init <name>                         # 프로젝트 생성
argus-cli init <name> --db custom.db          # 커스텀 DB 경로
argus-cli project export -o backup.zip        # 프로젝트 내보내기
argus-cli project import backup.zip           # 프로젝트 가져오기
```

데이터는 로컬 SQLite 파일에 저장되며, 외부 서버 연결이 필요 없습니다.

---

## License

Proprietary - All rights reserved.
