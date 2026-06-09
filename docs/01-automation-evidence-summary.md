# B1-3 자동화 실행 증거 요약

## 1. 자동화 목적

Cloudflare D1에 저장된 Home-curator 수집 파이프라인의 실행 상태를 Make로 조회하고, 결과를 Google Sheets에 기록한 뒤 Telegram으로 일일 리포트를 발송하는 노코드 자동화를 구현했다.

## 2. 전체 흐름

Make Scheduler
→ HTTP GET `https://home-curator.hoseuk0917.workers.dev/health-report`
→ Google Sheets Add a Row
→ Telegram Bot Send a Text Message

## 3. HTTP API 응답 확인

Make HTTP 모듈에서 `/health-report`를 호출했고, 응답 상태 `200`을 확인했다.

응답에 포함된 주요 필드:

- status
- checked_at
- today_runs.count
- today_runs.error
- metrics.today_items
- table_counts.items
- table_counts.curations
- table_counts.runs
- message

## 4. Google Sheets 기록 확인

Make의 Google Sheets 모듈을 통해 실행 결과가 스프레드시트에 자동으로 추가되었다.

기록 컬럼:

- checked_at
- status
- today_runs_count
- today_runs_error
- today_items
- total_items
- curations
- runs
- message

## 5. Telegram 알림 확인

Make의 Router 조건 분기에서 status가 ok가 아닌 경우 Telegram Bot 모듈을 통해 점검 필요 알림이 전송되도록 구성했다. 과제 증거 확보를 위해 2nd 경로 필터를 일시적으로 테스트 조건으로 바꾸어 Telegram 경로가 실제 실행되는 것을 확인했다.

✅ Home-curator 일일 리포트

상태: ok  
오늘 실행: 5회  
오늘 실패 run: 0회  
오늘 수집 items: 442  
전체 items: 55945  
전체 curations: 1498  
전체 runs: 439  
메시지: harvest pipeline healthy

## 6. 제출 증거 파일

스크린샷:

- `screenshots/01-make-scenario-flow.png`
- `screenshots/02-http-success-output.png`
- `screenshots/03-google-sheets-log.png`
- `screenshots/04-telegram-report.png`
- `screenshots/05-make-router-flow.png`
- `screenshots/06-make-router-alert-path-test.png`
- `screenshots/07-make-router-final-flow.png`

Worker API 변경 패치:

- `outputs/b1_3_health_report_api_20260609.patch`
- `outputs/b1_3_health_report_api_v2_20260609.patch`

## 7. Make와 Zapier 비교

Make는 HTTP 응답 JSON을 시각적으로 펼쳐보고 각 필드를 Google Sheets와 Telegram 메시지에 매핑하기 쉬웠다. 또한 시나리오 화면에서 HTTP, Google Sheets, Telegram 모듈이 한 흐름으로 표시되어 자동화 구조를 설명하기 좋았다.

Zapier도 같은 자동화를 구현할 수 있지만, 단계형 구성 중심이라 이번처럼 중첩 JSON 값을 확인하고 여러 출력 채널에 매핑하는 흐름은 Make가 더 직관적이었다.

## 8. 보안 처리

- Telegram Bot Token은 GitHub에 저장하지 않았다.
- API Key와 비공개 인증 정보는 README와 증거 문서에 포함하지 않았다.
- `/health-report`는 운영 상태 요약과 집계 지표만 반환하며 원문 데이터나 토큰을 반환하지 않는다.
- 제출용 캡처에서는 가능한 범위에서 개인 식별 정보와 민감 정보를 노출하지 않는 방향으로 정리했다.

## 9. 최종 결과

B1-3 노코드 자동화는 실제 동작까지 확인되었다.

완료된 자동화:

Cloudflare Worker API
→ Make HTTP 호출
→ Router 조건 분기
→ status = ok이면 Google Sheets 자동 기록
→ status != ok이면 Telegram Bot 경고 알림


