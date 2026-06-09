# B1-3 노코드 자동화 과제

## 주제
Cloudflare D1 기반 자동 수집 결과 모니터링 및 노코드 일일 리포트 자동화

## 프로젝트 구성
1. Make와 Zapier로 동일한 자동화 워크플로우 비교 구현
2. Home-curator 수집 파이프라인의 D1 메타데이터를 기반으로 운영 상태 리포트 자동화

## 기본 워크플로우
매일 정해진 시간 실행
→ Cloudflare Worker API `/health-report` 호출
→ 상태값에 따라 정상/주의/실패 분기
→ Google Sheets에 실행 로그 저장
→ Telegram 또는 이메일로 일일 리포트 발송

## 사용 도구
- Cloudflare Worker
- Cloudflare D1
- Make
- Zapier
- Google Sheets
- Telegram 또는 Gmail

## 보안 원칙
API 키, 토큰, 계정 이메일은 제출 문서와 스크린샷에서 마스킹한다.
