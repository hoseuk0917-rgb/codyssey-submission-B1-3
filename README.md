# B1-3 노코드 자동화 과제

## 주제

Cloudflare D1 기반 자동 수집 결과 모니터링 및 노코드 일일 리포트 자동화

## 프로젝트 목적

기존 Home-curator 프로젝트는 Cloudflare Worker의 cron trigger를 통해 뉴스·기술 자료 메타데이터를 자동 수집하고, Cloudflare D1에 수집 결과와 실행 로그를 저장한다.

이번 B1-3 과제에서는 이 운영 데이터를 노코드 자동화 도구인 Make와 연결하여, 매일 수집 상태를 확인하고 Google Sheets에 기록하며 Telegram으로 리포트를 받는 자동화 흐름을 구현했다.

## 최종 자동화 흐름

Make Scenario 구성:

Make Scheduler
→ HTTP GET `/health-report`
→ Google Sheets Add a Row
→ Telegram Bot Send a Text Message

## 사용 도구

- Cloudflare Worker
- Cloudflare D1
- Make
- Google Sheets
- Telegram Bot
- GitHub

## 자동화 API

Cloudflare Worker에 노코드 도구가 읽을 수 있는 상태 리포트 API를 추가했다.

API endpoint:

`https://home-curator.hoseuk0917.workers.dev/health-report`

이 API는 D1에 저장된 실행 로그와 수집 메타데이터를 조회하여 JSON으로 반환한다.

주요 반환 항목:

- `status`
- `checked_at`
- `latest_run`
- `latest_curation`
- `today_runs`
- `today_run_sources`
- `recent_runs`
- `table_counts`
- `metrics`
- `d1_capacity`
- `message`

## 수집 상태 리포트 예시

실제 실행 시 다음과 같은 지표가 Make로 전달되었다.

- 상태: `ok`
- 오늘 실행 횟수: `5`
- 오늘 실패 run: `0`
- 오늘 수집 items: `442`
- 전체 items: `55945`
- 전체 curations: `1498`
- 전체 runs: `439`
- 메시지: `harvest pipeline healthy`

## Google Sheets 기록

Make의 Google Sheets 모듈을 사용하여 실행 결과를 한 행씩 누적 기록했다.

저장 컬럼:

- checked_at
- status
- today_runs_count
- today_runs_error
- today_items
- total_items
- curations
- runs
- message

## Telegram 알림

Make의 Telegram Bot 모듈을 사용하여 실행 결과 요약을 개인 Telegram 채팅으로 발송했다.

전송 메시지 형식:

✅ Home-curator 일일 리포트

상태: ok  
오늘 실행: 5회  
오늘 실패 run: 0회  
오늘 수집 items: 442  
전체 items: 55945  
전체 curations: 1498  
전체 runs: 439  
메시지: harvest pipeline healthy

## 실행 증거

다음 캡처 파일을 `screenshots/` 폴더에 저장했다.

- `screenshots/01-make-scenario-flow.png`
- `screenshots/02-http-success-output.png`
- `screenshots/03-google-sheets-log.png`
- `screenshots/04-telegram-report.png`

## 구현 산출물

Cloudflare Worker에 추가한 API 변경 내역은 `outputs/` 폴더에 patch 파일로 보관했다.

- `outputs/b1_3_health_report_api_20260609.patch`
- `outputs/b1_3_health_report_api_v2_20260609.patch`

## 보안 처리

- Telegram Bot Token은 저장소와 제출물에 포함하지 않았다.
- API Key, 비공개 토큰, 계정 인증 정보는 README와 문서에 기록하지 않았다.
- 제출 캡처에서는 가능한 범위에서 개인 계정 정보와 민감 정보를 노출하지 않는 방향으로 정리했다.
- `/health-report` API는 운영 상태 요약과 메타데이터 집계만 반환하며, 원문 데이터나 인증 토큰은 반환하지 않는다.

## Make와 Zapier 비교

이번 구현에서는 실제 동작 자동화는 Make로 완료했다. Zapier는 동일 구조로 구현 가능하지만, 무료 플랜 기준으로 Make가 시각적 흐름 구성, HTTP 응답 매핑, Google Sheets와 Telegram 연결을 한 화면에서 확인하기 쉬워 이번 과제 구현 도구로 선택했다.

비교 요약:

- Make: 시각적 플로우 확인이 쉽고 HTTP JSON 응답 매핑이 직관적이다.
- Zapier: 단계형 자동화는 쉽지만, 복잡한 분기와 데이터 매핑은 Make보다 덜 직관적으로 느껴졌다.
- 이번 과제 목적은 Cloudflare D1 운영 데이터를 외부 알림과 기록 시스템으로 연결하는 것이므로 Make가 더 적합했다.

## 최종 상태

B1-3 제출용 핵심 자동화는 실제 동작까지 확인했다.

완료된 흐름:

Cloudflare Worker API
→ Make HTTP 모듈 호출
→ Google Sheets 행 추가
→ Telegram Bot 리포트 발송

현재 Make 시나리오 이름:

`Home Curator Health Report`

현재 시나리오 상태:

`Active`
