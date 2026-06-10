# B1-3 노코드 자동화 과제

## 주제

Cloudflare D1 기반 자동 수집 결과 모니터링 및 노코드 일일 리포트 자동화

## 프로젝트 목적

기존 Home-curator 프로젝트는 Cloudflare Worker의 cron trigger를 통해 뉴스·기술 자료 메타데이터를 자동 수집하고, Cloudflare D1에 수집 결과와 실행 로그를 저장한다.

이번 B1-3 과제에서는 이 운영 데이터를 노코드 자동화 도구인 Make와 연결하여, 매일 수집 상태를 확인하고 정상 상태는 Google Sheets에 기록하며 비정상 상태는 Telegram으로 경고 알림을 받는 자동화 흐름을 구현했다.

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

Make의 Router 조건 분기에서 status가 ok가 아닌 경우 Telegram Bot 모듈을 통해 점검 필요 알림을 개인 Telegram 채팅으로 발송하도록 구성했다. 또한 과제 증거 확보를 위해 2nd 경로 필터를 일시적으로 테스트 조건으로 바꾸어 Telegram 경로가 실제 실행되는 것을 확인했다.

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
- `screenshots/05-make-router-flow.png`
- `screenshots/06-make-router-alert-path-test.png`
- `screenshots/07-make-router-final-flow.png`

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
→ Router 조건 분기
→ status = ok이면 Google Sheets 행 추가
→ status != ok이면 Telegram Bot 경고 알림 발송

현재 Make 시나리오 이름:

`Home Curator Health Report`

현재 시나리오 상태:

`Active`




## Zapier 비교 구현

동일한 자동화 목적을 Zapier에서도 별도로 구현했다.

Zapier 구성:

Schedule by Zapier
→ Webhooks by Zapier GET `/health-report`
→ Filter by Zapier `status = ok`
→ Google Sheets Create Spreadsheet Row

Zapier 실행 결과:

- `/health-report` Webhook 호출 성공
- Filter 조건 통과 확인
- `B1-3 Zapier Health Logs` 스프레드시트에 실행 결과 1행 추가 확인

Zapier 증거 파일:

- `screenshots/08-zapier-webhook-success.png`
- `screenshots/09-zapier-filter-success.png`
- `screenshots/10-zapier-sheets-success.png`
- `screenshots/11-zapier-sheets-output.png`

## Make와 Zapier 비교 결과 보강

Make는 Router를 이용한 시각적 분기와 Telegram 알림 구성이 직관적이었다. HTTP 응답 JSON을 펼쳐 보고 Google Sheets와 Telegram에 매핑하는 과정도 한 화면에서 확인하기 쉬웠다.

Zapier는 Schedule, Webhook, Filter, Google Sheets 단계가 순서대로 구성되어 초보자가 흐름을 따라가기 쉬웠다. 다만 중첩 JSON 필드 매핑과 Google Sheets 헤더 인식은 Make보다 까다로웠고, 시트/워크시트 선택을 다시 고르거나 새 스프레드시트를 만들어야 안정적으로 동작했다.

최종적으로 Make는 운영 자동화에 더 적합했고, Zapier는 동일 구조를 비교 구현하는 용도로 활용했다.

## 평가 보완: 설계 기준과 심층 분석

### 1. 두 도구의 동일 워크플로우 설계 기준

Make와 Zapier는 서로 다른 UI와 실행 방식이 있지만, 비교를 위해 다음 기준을 동일하게 맞췄다.

1. 입력 이벤트 동일화
- Make: Scheduler
- Zapier: Schedule by Zapier
- 두 도구 모두 시간 기반 Trigger를 사용하여 수동 개입 없이 실행되도록 구성했다.

2. 입력 데이터 동일화
- 두 도구 모두 Cloudflare Worker의 `/health-report` API를 호출한다.
- 동일한 D1 운영 상태 JSON을 입력값으로 사용한다.

3. 핵심 처리 단계 동일화
- HTTP GET으로 상태 리포트를 가져온다.
- status 값을 기준으로 정상 여부를 판단한다.
- 실행 결과를 Google Sheets에 기록한다.

4. 조건 기준 동일화
- status = ok이면 정상 상태로 본다.
- status != ok이면 warning/error로 보고 점검 필요 상태로 본다.

5. 출력 기준 동일화
- Make와 Zapier 모두 최종 결과를 외부에서 확인 가능한 형태로 남긴다.
- Make는 Google Sheets와 Telegram을 사용했고, Zapier는 Google Sheets 기록으로 비교 구현 증거를 남겼다.

### 2. Trigger와 Action의 차이

Trigger는 자동화의 시작 조건이고, Action은 Trigger 이후 실제로 수행되는 작업이다.

이번 프로젝트의 Trigger 예시는 다음과 같다.

- Make Scheduler
- Schedule by Zapier

이 Trigger들은 "언제 자동화를 시작할지"를 결정한다.

이번 프로젝트의 Action 예시는 다음과 같다.

- HTTP GET `/health-report`
- Google Sheets Add/Create Row
- Telegram Bot Send Message

Action은 "시작된 뒤 어떤 처리를 할지"를 담당한다.

예를 들어 매일 정해진 시간에 Scheduler가 실행되면, 그 다음 Action들이 순서대로 실행되어 API를 조회하고, 결과를 기록하고, 필요하면 알림을 보낸다.

### 3. 프로젝트 2 워크플로우의 단계별 입력/출력

1. Trigger: Make Scheduler
- 입력: 설정된 실행 시간
- 출력: 자동화 시작 이벤트

2. Action 1: HTTP GET `/health-report`
- 입력: Cloudflare Worker API URL
- 출력: D1 운영 상태 JSON

3. Router 조건 분기
- 입력: HTTP 응답의 status 값
- 출력: status = ok 경로 또는 status != ok 경로

4. Action 2: Google Sheets Add a Row
- 입력: checked_at, status, today_items, table_counts, message
- 출력: 스프레드시트에 운영 로그 1행 추가

5. Action 3: Telegram Bot Send Message
- 입력: status, warnings, latest_run, error message
- 출력: Telegram 점검 필요 알림

### 4. 조건 분기 설계 기준

조건 분기는 status 값을 기준으로 설계했다.

- status = ok: 수집 파이프라인이 정상으로 판단되어 Google Sheets에 기록한다.
- status != ok: warning 또는 error 상태로 판단하여 Telegram 경고 알림을 보낸다.

이 기준을 선택한 이유는 다음과 같다.

1. 운영자는 정상 상태보다 비정상 상태를 빠르게 확인해야 한다.
2. 정상 실행까지 모두 Telegram으로 보내면 알림 피로도가 커진다.
3. status 값은 `/health-report` API에서 이미 수집 결과, run 상태, 큐레이션 개수 등을 종합해 만든 운영 판단 값이다.
4. 따라서 자동화 도구에서는 복잡한 조건을 다시 계산하지 않고 status를 기준으로 단순하고 안정적으로 분기하는 것이 적합하다.

### 5. Make와 Zapier 비교 항목

| 비교 항목 | Make | Zapier |
|---|---|---|
| UI/UX | 노드 기반 시각적 흐름이 명확하다. | 단계형 리스트라 순서 이해가 쉽다. |
| 설정 난이도 | Router와 JSON 매핑은 익숙해지면 편하지만 초기 학습이 필요하다. | 기본 단계 구성은 쉽지만 세부 필드 매핑에서 막히는 경우가 있었다. |
| JSON 매핑 | nested JSON을 펼쳐 보고 매핑하기 비교적 쉽다. | 중첩 JSON 필드와 Google Sheets 헤더 인식이 까다로웠다. |
| 조건 분기 | Router로 여러 경로를 시각적으로 관리하기 좋다. | Filter는 단순 조건에는 적합하지만 복수 분기 확장에는 불리하다. |
| 실행 로그 확인 | 각 모듈별 실행 결과와 경로를 시각적으로 확인하기 쉽다. | 단계별 테스트 결과는 확인 가능하지만 전체 흐름 가시성은 상대적으로 약하다. |
| 연동 서비스 | Google Sheets, Telegram, HTTP 연동이 직관적이었다. | Webhook, Filter, Sheets 구성은 가능했지만 설정 재인식 문제가 있었다. |
| 운영 적합성 | 최종 운영 자동화 도구로 적합하다고 판단했다. | 비교 구현과 단순 자동화에는 적합하지만 복잡한 운영에는 불편했다. |
| 비용/실행 효율 | 잦은 주기 실행은 operations를 소모하므로 경고 중심 운영이 적합하다. | tasks 기반 과금이므로 반복 polling에는 부담이 있을 수 있다. |

### 6. 업무 요구사항 관점 비교

1. 협업
- Make는 시각적 노드가 보여 팀원이 흐름을 이해하기 쉽다.
- Zapier는 단계형 설명에는 좋지만 전체 분기 구조를 한눈에 보기 어렵다.

2. 운영
- Make는 Router와 History를 통해 어느 경로가 실행됐는지 확인하기 쉽다.
- Zapier는 테스트는 쉽지만 운영 중 복잡한 분기 추적은 상대적으로 어렵다.

3. 디버깅
- Make는 HTTP 응답, Sheets 입력, Telegram 출력이 모듈별로 확인된다.
- Zapier는 Webhook 테스트는 편하지만 Sheets 필드 인식 문제를 해결하는 데 시간이 걸렸다.

4. 확장
- Make는 status 외에도 warning/error/deep_fetch_fail/curate_fail 등으로 Router를 확장하기 쉽다.
- Zapier는 Filter가 늘어나면 Zap이 길어지고 관리가 어려워질 수 있다.

5. 비용
- 두 도구 모두 짧은 주기 polling은 비용과 실행량 부담이 생긴다.
- 실제 운영에서는 하루 1회 리포트와 실패 시 알림만 보내는 방식이 효율적이다.

### 7. 프로젝트 2에서 Make를 선택한 이유

최종 운영 도구로 Make를 선택한 이유는 다음과 같다.

1. 조건 분기 관리
- Make의 Router는 status = ok와 status != ok 경로를 시각적으로 구분하기 쉬웠다.
- 향후 warning, error, curate_fail, cluster_fail 등으로 분기가 늘어날 때도 확장성이 좋다.

2. JSON 응답 매핑
- `/health-report`는 nested JSON 구조를 가진다.
- Make는 HTTP 응답의 하위 값을 펼쳐 Google Sheets와 Telegram에 매핑하기 쉬웠다.

3. 운영 확인성
- Make는 각 모듈의 실행 여부와 경로를 화면에서 확인할 수 있어 운영 모니터링에 적합했다.

### 8. 실패 모니터링, 재시도, 대체 경로 전략

외부 연동이 실패할 경우 다음 전략을 적용할 수 있다.

1. 실패 모니터링
- `/health-report`의 latest_run.status, today_runs.error, warnings 값을 기준으로 감지한다.
- status != ok이면 Telegram 경고 알림을 보낸다.

2. 재시도 전략
- HTTP 호출 실패 시 Make의 error handler 또는 재실행 기능을 사용한다.
- 장기적으로는 Cloudflare Worker 내부에서 실패 단계별 재시도 로직을 구현한다.

3. 대체 경로
- Google Sheets 기록 실패 시 Telegram으로 "로그 저장 실패" 알림을 보낸다.
- Telegram 실패 시 Email 또는 다른 메신저로 대체 알림을 보낼 수 있다.
- D1 자체에는 runs 테이블이 있으므로 외부 시트 저장이 실패해도 원본 실행 로그는 남는다.

### 9. 분기 규칙 증가 시 구조화 전략

현재는 status = ok / status != ok의 2개 분기지만, 운영이 복잡해지면 다음처럼 구조화한다.

1. 1차 분기: ok / warning / error
2. 2차 분기: harvest / cluster / curate / deep_fetch
3. 출력 분리: 일일 리포트 / 즉시 장애 알림 / 누적 로그
4. 공통 메시지 템플릿 사용
5. 복잡한 판단 로직은 노코드가 아니라 Cloudflare Worker 코드 내부로 이동

예시 확장:

- harvest_fail → Telegram 즉시 알림
- cluster_fail → Telegram 즉시 알림
- curate_zero → warning 알림
- daily_ok → 하루 1회 요약 리포트
- db_capacity_warning → 저장 용량 점검 알림

### 10. 노코드 자동화의 한계와 코드 기반 확장

노코드 자동화는 빠르게 연결하고 검증하기 좋지만 다음 한계가 있었다.

1. 중첩 JSON 처리와 필드 매핑이 도구별로 불안정하다.
2. Google Sheets 헤더 인식처럼 UI/캐시 문제에 영향을 받는다.
3. 복잡한 재시도, 중복 방지, 단계별 장애 복구를 구현하기 어렵다.
4. 실행 주기가 짧으면 비용과 task/operation 사용량이 증가한다.

따라서 장기 운영에서는 다음처럼 코드 기반 자동화로 확장하는 것이 적합하다.

1. Cloudflare Worker 내부에서 Telegram API 직접 호출
2. harvest, cluster, curate, deep_fetch 단계별 실패 감지
3. 하루 1회 daily report 자동 생성
4. 실패 단계별 즉시 alert
5. D1 runs/run_sources/curations 기반 운영 대시보드 구축

### 11. 최종 운영 방향

과제 제출용으로는 Make와 Zapier를 모두 사용하여 노코드 자동화 도구 비교를 완료했다.

실제 운영 방향은 다음과 같다.

- Make/Zapier는 학습 및 비교 구현 용도로 사용한다.
- 최종 운영은 Cloudflare Worker cron과 Telegram API 직접 호출로 단순화한다.
- 하루 1회 전체 리포트와 실패 시 즉시 알림을 병행한다.
- Google Sheets는 과제 증거와 간단 로그 용도로만 사용하고, 장기 로그는 D1 runs 테이블을 기준으로 관리한다.
