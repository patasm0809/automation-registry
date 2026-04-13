# Automation Registry v2

김수민(ptsm89) 운영 자동화 현황을 한눈에 보는 정적 대시보드입니다.

- **배포 URL**: https://patasm0809.github.io/automation-registry/
- **소스**: `automation-registry/`
- **자동 상태체크**: `~/bots/registry-monitor/check_status.py` (launchd, 5분 주기)

## 파일 구성

| 파일 | 역할 | 수동 편집 |
|------|------|-----------|
| `index.html` | 대시보드 UI | O |
| `assets/style.css` | 스타일 | O |
| `registry.json` | 자동화 카탈로그(봇/스크립트/API/캘린더) | O (수동) |
| `changelog.json` | 모든 변경 이력 타임라인 | O (수동) |
| `status.json` | launchd 서비스 실시간 상태 | **X (자동 갱신)** |

## 갱신 프로토콜 (Claude 실행 규칙)

### 1. 새 봇/스크립트/API 추가 시

1. `registry.json` → `automations` 배열에 항목 추가
   - 필수 필드: `id`, `name`, `category`, `icon`, `path`, `schedule`, `purpose`, `tech`, `created_at`, `last_modified`, `state`
   - 민감 정보(토큰/API 키 등) 절대 기재 금지 — "경로에 저장됨"만 기재
2. `changelog.json` → `entries` 배열 맨 앞에 엔트리 추가
   ```json
   {
     "timestamp": "<ISO 8601 KST>",
     "action": "added",
     "target": "<id>",
     "category": "<category>",
     "summary": "<한줄 요약>",
     "reason": "<왜 추가했는지>"
   }
   ```
3. `registry.json` 내 `updated_at` 필드 업데이트
4. git commit + push
   ```sh
   cd /Users/ptsm/Desktop/patasm_claudecode_001/temp-projects/automation-registry
   git -c user.email=patasm0809@gmail.com -c user.name=patasm0809 \
       commit -am "registry: add <id>"
   git push
   ```

### 2. 수정 시
- `registry.json` 해당 항목 갱신 + `last_modified` 업데이트
- `changelog.json`에 `action: "modified"` 엔트리 추가
- commit/push

### 3. 삭제 시
- `registry.json` 해당 항목 제거
- `changelog.json`에 `action: "deleted"` 엔트리 추가
- commit/push

### 4. status.json — 수동 편집 금지
`~/bots/registry-monitor/check_status.py`가 5분마다 자동 갱신하고 자동 커밋/푸시합니다.
수동 편집 시 다음 사이클에 덮어써집니다.

## 카테고리

- `launchd-bot` — launchd 영속 서비스 (com.patasm.*)
- `cron` — Claude Code 세션 기반 크론 (7일 자동만료 가능)
- `script` — 수동 실행 스크립트
- `api` — API/외부 연동 (credentials)
- `calendar` — Google Calendar

## 상태 값

- `running` — 프로세스 활성
- `scheduled` — 크론/launchd 예약 대기
- `active` — API/캘린더 등 상시 유효
- `manual` — 수동 실행 대상
- `stopped` — 정지됨
- `error` — 비정상 종료

## 보안 정책

- **민감 정보 절대 금지**: API 키, 봇 토큰, 인증키 등은 이 저장소에 커밋하지 않습니다.
- `registry.json`의 `path`/`credentials_location`은 저장 경로만 기재하고, 실제 값은 각 봇의 config에만 존재합니다.
- `.gitignore`로 추가 방어층 불필요 (애초에 민감 파일을 이 폴더에 두지 않음).

## 상태체크 스크립트 관리

```sh
# 상태 확인
launchctl list | grep com.patasm.registry-monitor

# 수동 실행
python3 ~/bots/registry-monitor/check_status.py

# 중단
launchctl unload ~/Library/LaunchAgents/com.patasm.registry-monitor.plist

# 재시작
launchctl load ~/Library/LaunchAgents/com.patasm.registry-monitor.plist
```

---

© 2026 · ptsm89
