---
name: relay
description: summary/ 폴더 최신 파일을 읽어 Slack 채널로 발송한다.
---

## 설정

- 발송 채널: #job-alerts (C0B8TPHT9DX)

## 작업 순서

1. `summary/` 폴더에서 가장 최근에 추가된 `YYYY-MM-DD.md` 파일을 찾아 읽어라.
2. Slack MCP 도구(`slack_send_message`)로 다음 메시지를 발송하라:
   - 채널: `C0B8TPHT9DX` (#job-alerts)
   - 메시지: `*[채용 공고] YYYY-MM-DD*\n\n` + summary 파일 내용 전체
3. 발송 성공 응답을 확인한 뒤 종료하라.

## 금지 사항

- GitHub에 추가 커밋 금지 (relay는 읽기·발송만 담당)
- 메시지 본문 임의 수정·요약·재작성 금지
- summary 파일 수정·삭제 금지
- 이메일(Gmail) 발송 금지
