---
name: briefing
description: raw/ 폴더 최신 파일을 읽고 채용공고를 fit_score 기반으로 요약하여 summary/ 폴더에 저장한다.
---

## 작업 순서

1. `raw/` 폴더에서 가장 최근에 추가된 `YYYY-MM-DD.md` 파일을 찾아 읽어라.
2. 같은 스킬 폴더의 `template.md` 파일을 읽어 출력 형식을 파악하라.
3. 각 공고에 대해 **fit_score**를 산출하라 (가산 방식, 합산이 10을 초과하면 10으로 고정):
   - React 매칭: +2점 (기술스택에 React 포함 시)
   - TypeScript 매칭: +2점 (기술스택에 TypeScript 포함 시)
   - TanStack Query / React Query: +1점 (기술스택에 TanStack Query, React Query 포함 시)
   - SSE / 실시간 스트리밍: +1점 (기술스택 또는 직무 설명에 SSE, Server-Sent Events, 실시간 스트리밍, EventSource 포함 시)
   - AI 서비스/제품 경험: +2점 (LLM, AI 서비스, 생성형 AI, AI product, ML product 관련 직무 시)
   - 에디터 구축 경험: +1점 (Lexical, Draft.js, Tiptap, Slate, ProseMirror, 리치텍스트·웹 에디터 구축 관련 시)
   - 성능 최적화 경험: +1점 (Core Web Vitals, LCP, 번들 최적화, Lighthouse, 코드 스플리팅 관련 시)
4. **fit_reason**을 작성하라: 점수를 구성하는 매칭 항목을 한 문장으로 설명 (예: "React·TypeScript 스택 일치, AI 제품 경험 우대")
5. fit_score 내림차순으로 정렬하고, 동점 시 posted_date 최신순으로 정렬하라.
6. 상위 최대 10개 공고만 summary에 포함하라.
7. `template.md` 형식에 맞춰 한국어로 작성하라.
8. 결과를 `summary/YYYY-MM-DD.md` 경로로 새 파일을 생성하고 커밋·푸시하라.
   - 날짜는 raw 파일과 동일하게 사용
   - 커밋 메시지: `brief: YYYY-MM-DD`

## 요약 원칙

- raw에 없는 사실을 추가하지 말 것
- 출처 링크는 raw 파일에 있는 URL을 그대로 사용
- 템플릿에 정의된 섹션 외 다른 내용을 추가하지 말 것

## 금지 사항

- 템플릿 형식 임의 변경 금지
- summary 파일에 메타 정보(작성자, 모델명 등) 추가 금지
- raw 파일 수정·삭제 금지
- fit_score 10점 초과 금지 (가산 합계가 10을 넘어도 10으로 고정)
