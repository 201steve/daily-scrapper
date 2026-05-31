# 일일 정보 브리핑 자동화 — Claude Code Routines 실습

매일 정해진 시간에 **지정 주제의 정보를 자동 수집 → 요약 → 메일 발송**까지 처리하는 워크플로우를 Claude Code Routines로 구축하는 실습 자료입니다.

코드 작성·터미널 명령은 **단 한 줄도 필요 없습니다**. GitHub 웹과 Claude Code 웹에서 클릭만으로 끝납니다.

---

## 1. 완성 시 동작 모습

```
[매일 오전 7시]
       ↓
Scrapper routine 자동 시작 (Claude Code 클라우드)
       ↓
지정 주제의 최근 24시간 정보 수집 → GitHub의 raw/2026-05-31.md 로 푸시
       ↓ (GitHub push 이벤트)
Briefing routine 자동 시작
       ↓
raw 파일 읽고 템플릿 형식으로 요약 → summary/2026-05-31.md 로 푸시
       ↓ (GitHub push 이벤트)
Relay routine 자동 시작
       ↓
summary 파일을 지정 이메일로 발송 → 완료
```

- 본인 PC가 꺼져 있어도 정상 동작합니다. (Anthropic 클라우드에서 실행)
- 모든 결과물은 GitHub 저장소에 날짜별로 영구 보관됩니다.

---

## 2. 사전 준비물

| 항목 | 비용 | 필요 시간 |
|---|---|---|
| GitHub 계정 | 무료 | 3분 |
| Claude Max 구독 | (이미 있음) | — |
| Gmail 계정 (발송용) | 무료 | (이미 있음) |
| 추적 주제 1개 | — | 결정만 |
| 수신 Gmail 주소 | — | 결정만 |

> **추가 API 요금 없음**: Routine 실행은 Max 구독 사용량에 차감되며, 별도 토큰 과금 없습니다. 일일 실행 횟수 캡이 있으나 본 워크플로우(하루 3회)는 한도 내입니다.

---

## 3. 폴더 구조 (이 저장소)

```
test-brief-agent/
├── README.md                       ← 이 파일
├── .claude/
│   └── skills/
│       ├── scrapper/
│       │   └── SKILL.md           ← 정보 수집 절차
│       ├── briefing/
│       │   ├── SKILL.md           ← 요약 절차
│       │   └── template.md       ← 브리핑 출력 형식 (분리)
│       └── relay/
│           └── SKILL.md           ← 메일 발송 절차
├── raw/
│   └── .gitkeep                    ← Scrapper가 매일 파일 추가
└── summary/
    └── .gitkeep                    ← Briefing이 매일 파일 추가
```

- **Routine** = 언제 시작할지(트리거) 정의 → Claude Code 웹에 등록
- **Skill** = 무엇을 어떻게 할지(절차) 정의 → 본 저장소 `.claude/skills/`에 저장
- Routine은 실행 시 본 저장소를 임시 작업 디렉토리로 사용하며 `.claude/skills/` 폴더를 자동 인식합니다.

---

## 4. 왜 이렇게 나누었는가 — 프롬프트 vs 스킬 vs 템플릿

> 이 실습의 핵심 학습 목표

**Q. 사실 잘 짠 프롬프트 하나면 수집·요약·발송이 모두 해결된다. 왜 굳이 3개의 스킬과 1개의 템플릿 파일로 쪼개 놓았나?**

**A. 에이전틱 워크플로우에서 "프롬프트 / 스킬 / 템플릿"이 각각 어떤 역할을 맡는지 몸으로 익히기 위해서다.** 실무 규모의 에이전트는 거의 항상 이 세 층을 분리해서 운영하기 때문에, 가장 단순한 예제에서부터 그 경계선을 그어 두는 것이 목적이다.

### 4.1 세 층의 역할

| 구성요소 | 한 줄 정의 | 본 프로젝트에서 | 변경 빈도 | 누가 만지나 |
|---|---|---|---|---|
| **Prompt** (Routine entry) | "지금 무엇을 시작할 것인가" 진입 명령 | `Use the scrapper skill for today.` | 거의 안 바뀜 | Routine 설정자 |
| **Skill** (`SKILL.md`) | "어떻게 수행할 것인가" 절차·역할 정의 | scrapper / briefing / relay | 가끔 (업무 절차 개선 시) | 워크플로우 설계자 |
| **Template** (`template.md`) | "결과물이 어떤 형태여야 하는가" 출력 명세 | briefing/template.md | 자주 (포맷 튜닝) | 콘텐츠 담당자 |

핵심은 **변경 빈도와 변경 주체가 다르다는 것**이다. 한 덩어리의 거대한 프롬프트는 셋 중 무엇 하나를 바꾸려고 해도 전체를 다시 검토해야 한다. 분리해 두면 각자의 사이클대로 따로 움직일 수 있다.

### 4.2 분리하지 않으면 잃는 것

**1) 재사용성 (Reusability)**
- 통합 프롬프트: "주간 브리핑"을 만들려면 처음부터 다시 작성
- 분리 구조: 같은 `briefing` 스킬을 주간/월간 Routine에서 그대로 호출, `template.md`만 교체

**2) 버전 관리 (Auditability)**
- 통합 프롬프트: "왜 지난주부터 메일 형식이 바뀌었지?" → Routine 설정 화면의 변경 이력으로 추적
- 분리 구조: `template.md`의 git history에 누가 언제 무엇을 왜 바꿨는지 그대로 남음 → PR 리뷰·롤백 가능

**3) 관심사 분리 (Separation of Concerns)**
- 통합 프롬프트: 콘텐츠 담당자가 메일 형식만 바꾸려 해도 수집·발송 로직까지 읽고 건드릴 위험
- 분리 구조: `template.md`만 열어 수정 → 다른 단계는 0% 영향

**4) 디버깅 용이성 (Debuggability)**
- 통합 프롬프트: 실패하면 어디서 실패했는지 한 세션 안에서 뒤져야 함
- 분리 구조: scrapper / briefing / relay 각자의 세션 로그로 분리 → 어느 단계에서 깨졌는지 즉시 식별

### 4.3 실제 시나리오로 비교해 보기

**시나리오: "메일 본문에 '한줄평' 섹션을 하나 추가하고 싶다."**

- **통합 프롬프트 방식**: Claude Code 웹 → Routine 편집 → 거대한 단일 프롬프트 안에서 출력 형식 부분 찾기 → 신중히 수정 → 다른 곳 깨질 위험 검증
- **본 프로젝트 방식**: GitHub 웹에서 `template.md` 열기 → 한 줄 추가 → Commit. **끝.** scrapper와 relay는 손도 안 댐.

**시나리오: "수집 출처에 네이버 뉴스도 추가하고 싶다."**

- 통합 프롬프트 방식: 거대 프롬프트 전체 재검토
- 본 프로젝트 방식: `scrapper/SKILL.md`만 수정. briefing·relay·template은 영향 없음.

### 4.4 에이전트 개발에서 일반화하기

이 패턴은 본 실습에 국한된 트릭이 아니라 **에이전트 시스템 설계의 일반 원칙**이다.

- **Prompt = 트리거 어댑터**: 외부 신호(스케줄, 이벤트, 사용자 입력)를 받아 "어떤 스킬을 어떤 컨텍스트로 호출할지" 결정만 함. 비즈니스 로직 없음.
- **Skill = 역할/능력 모듈**: 한 가지 일을 잘 하도록 설계된 재사용 단위. 다른 에이전트가 재활용 가능.
- **Template/Data = 산출물 명세와 입력 데이터**: 코드와 분리된 콘텐츠 자산. 비기술자가 편집 가능.

규모가 커질수록 이 경계가 흐려진 시스템은 유지보수가 빠르게 무너진다. **가장 단순한 예제에서부터 경계를 명확히 두는 훈련**이 이 실습의 진짜 목적이다.

> 💡 **부가 학습 과제**: 위 시나리오 두 개를 실제로 본인 repo에서 수행해 보라. "어느 파일만 만지면 되는가"를 손가락으로 짚어가며 확인하면 세 층의 역할이 체감된다.

---

## 5. 실습 절차

### Step 1. 빈칸 2곳 채우기

#### 1-1. 추적 주제 입력
- 파일: `.claude/skills/scrapper/SKILL.md`
- 수정 위치: `## 주제` 섹션 아래 `※ 여기에 추적할 주제를 한 줄로 적으세요 ※`
- 예시:
  ```
  - 국내 호텔 M&A 및 투자 동향
  ```

#### 1-2. 수신 이메일 입력
- 파일: `.claude/skills/relay/SKILL.md`
- 수정 위치: `## 설정` 섹션의 `수신 이메일: ※ 여기에 받는 사람 Gmail 주소를 적으세요 ※`
- 예시:
  ```
  - 수신 이메일: yourname@gmail.com
  ```

> 텍스트 에디터로 직접 수정하거나, GitHub에 업로드한 뒤 GitHub 웹 UI에서 연필 아이콘 클릭해 편집해도 됩니다.

---

### Step 2. GitHub 저장소 만들기

1. [github.com](https://github.com) 로그인
2. 우측 상단 **`+`** 클릭 → **New repository**
3. 입력:
   - Repository name: `daily-briefing` (원하는 이름으로 변경 가능)
   - **Private** 선택 (Public도 무방하나 정보 노출에 주의)
4. 하단 **Create repository** 클릭

---

### Step 3. 파일 업로드 (git 명령 불필요)

1. 방금 만든 빈 저장소 화면 중앙의 **"uploading an existing file"** 링크 클릭
2. Mac Finder에서 본 폴더(`test-brief-agent`) 열기
   - **`.claude` 폴더가 안 보이면**: `Cmd + Shift + .` 눌러 숨김 파일/폴더 표시
3. **폴더 안의 항목 전부**를 GitHub 업로드 창으로 드래그
   - `.claude/`, `raw/`, `summary/`, `README.md` 모두 포함
   - **주의**: `test-brief-agent` 폴더 자체가 아니라 **그 안의 항목들**을 드래그
4. 페이지 하단 Commit 메시지에 `Initial setup` 입력 후 **Commit changes** 클릭

업로드 완료 후 저장소에서 위 [폴더 구조](#3-폴더-구조-이-저장소) 그대로 보이면 성공.

---

### Step 4. Claude Code 웹에 커넥터 2개 연결

1. [claude.com/code](https://claude.com/code) 로그인 (Max 계정)
2. 좌측 메뉴 **Settings** → **Connectors**
3. 두 커넥터 연결:

| 커넥터 | 용도 | 권한 |
|---|---|---|
| **GitHub** | 저장소 읽기·쓰기 | `daily-briefing` repo만 선택 허용 |
| **Gmail** | 메일 발송 | 발송 주체가 될 본인 Gmail 계정 연결 |

> 💡 **참고**: Gmail 커넥터에 연결한 계정 = "보내는 사람". 받는 사람 주소는 Step 1-2에서 입력한 값.

---

### Step 5. Routine 3개 생성

Claude Code 웹 좌측 메뉴 **Routines** → **Create routine** 을 3번 반복.

#### Routine ① — scrapper

| 항목 | 값 |
|---|---|
| Name | `scrapper` |
| Repository | `daily-briefing` 선택 |
| Trigger | **Schedule** → `매일 07:00`, Timezone: **Asia/Seoul** |
| Prompt | `Use the scrapper skill for today.` |

#### Routine ② — briefing

| 항목 | 값 |
|---|---|
| Name | `briefing` |
| Repository | `daily-briefing` 선택 |
| Trigger | **GitHub event** → Event: `push`, Path filter: `raw/**` |
| Prompt | `Use the briefing skill.` |

#### Routine ③ — relay

| 항목 | 값 |
|---|---|
| Name | `relay` |
| Repository | `daily-briefing` 선택 |
| Trigger | **GitHub event** → Event: `push`, Path filter: `summary/**` |
| Prompt | `Use the relay skill.` |

---

### Step 6. 첫 실행 테스트

자동 실행을 기다리지 말고 즉시 검증합니다.

1. Routines 페이지 → **scrapper** 클릭 → 우측 상단 **Run now** 클릭
2. 표시되는 세션 URL을 새 탭에서 열어 실시간 진행 관찰
3. 약 2~5분 후 다음 순서로 확인:
   - [ ] GitHub `daily-briefing` repo에 `raw/YYYY-MM-DD.md` 생성됨
   - [ ] **자동으로** `briefing` routine 시작 → `summary/YYYY-MM-DD.md` 생성됨
   - [ ] **자동으로** `relay` routine 시작 → 지정 Gmail 수신함에 메일 도착

세 단계 모두 통과하면 설정 완료. 다음 날 오전 7시부터 자동으로 매일 실행됩니다.

---

## 6. 결과물 확인 방법

| 무엇 | 어디서 |
|---|---|
| 일일 수집 원본 | `daily-briefing` repo의 `raw/YYYY-MM-DD.md` |
| 일일 요약 (브리핑) | `daily-briefing` repo의 `summary/YYYY-MM-DD.md` |
| 일일 브리핑 메일 | 지정 Gmail 수신함, 제목 `[일일 브리핑] YYYY-MM-DD` |
| 실행 이력·로그 | Claude Code 웹 → Routines → 각 routine → Runs 탭 |

---

## 7. 운영 중 변경하는 법

### 추적 주제 바꾸기
- GitHub 웹에서 `.claude/skills/scrapper/SKILL.md` 열기 → 연필 아이콘 → 주제 수정 → Commit
- 다음 실행부터 새 주제 반영

### 요약 템플릿 바꾸기
- GitHub 웹에서 `.claude/skills/briefing/template.md` 열기 → 형식 수정 → Commit
- 다음 실행부터 새 형식 반영

### 수신 이메일 바꾸기
- GitHub 웹에서 `.claude/skills/relay/SKILL.md` 열기 → 이메일 수정 → Commit

### 실행 시간 바꾸기
- Claude Code 웹 → Routines → scrapper → Edit → Schedule 변경

### 일시 중지
- Claude Code 웹 → Routines → 각 routine → **Disable** 토글

---

## 8. 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| Briefing이 자동 시작 안 함 | Path filter 오설정 또는 GitHub 권한 누락 | Briefing routine trigger의 path가 `raw/**` 인지 확인 / Settings → Connectors → GitHub 권한 재승인 |
| Relay 메일 발송 실패 | Gmail 커넥터 미연결 또는 권한 부족 | Settings → Connectors → Gmail 재연결 |
| 매일 실행 안 됨 | Schedule timezone 오설정 | scrapper routine의 Timezone을 `Asia/Seoul`로 다시 설정 |
| "daily cap" 에러 | 하루 routine 실행 횟수 한도 초과 | 다음 날 자동 리셋. 빈번하면 routine 수 줄이기 |
| 빈 raw 파일 생성됨 | 해당 주제의 24h 내 자료 없음 | 정상 동작. 주제를 더 넓히거나 빈도 조정 |
| Skill을 인식 못 함 | `.claude/skills/` 경로 오류 | repo 루트에 `.claude/skills/<name>/SKILL.md` 형태로 있는지 확인 |

---

## 9. 작동 원리 (이해하고 싶은 사람만)

### Routine vs Skill
- **Routine**: 트리거(when) + 진입 지점(entry prompt). Claude Code 웹에 등록.
- **Skill**: 재사용 가능한 절차(how). 본 저장소 `.claude/skills/` 에 저장.
- Routine 프롬프트는 한 줄("use the X skill")만 두고, 실제 로직은 모두 Skill 파일에 둠 → 수정·버전 관리가 쉬움.

### 자동 체이닝
3개 routine은 **GitHub의 push 이벤트**로 자동 연결됩니다. API 토큰이나 `/fire` 호출이 필요 없는 이유:
- Scrapper가 `raw/` 에 커밋·푸시 → GitHub가 push 이벤트 발생 → Briefing routine의 트리거 매칭 → 자동 시작
- 동일 패턴이 Briefing → Relay에도 적용

### Skill 파일이 저장되는 위치
- 본 저장소(`daily-briefing`)의 `.claude/skills/` 폴더
- Routine 실행 시 Anthropic 클라우드 인프라가 본 저장소를 임시 작업 디렉토리로 clone → `.claude/skills/` 자동 인식
- 별도 업로드·배포 절차 없음. **GitHub에 커밋·푸시하면 즉시 다음 실행에 반영.**

### 비용
- Routine 실행은 Max 구독 사용량(5시간/7일 윈도우)에서 차감
- API 토큰 과금 별도 없음
- 일일 routine 실행 횟수 캡 존재 (계정·플랜별)
- 한도 초과 시 자동 정지 (메터드 오버리지 옵션을 켜지 않은 한 추가 청구 없음)

---

## 10. 빠른 점검 체크리스트

설정 완료 후 1주일간 매일 아침 확인:

- [ ] 오전 7시~7시 10분 사이 Gmail에 `[일일 브리핑]` 메일 수신
- [ ] 메일 본문 형식이 `briefing/template.md`와 일치
- [ ] GitHub repo에 그날 날짜의 `raw/` 와 `summary/` 파일 둘 다 존재
- [ ] Routines 페이지에서 3개 routine 모두 마지막 실행 상태 "Success"

문제 발생 시 [8. 트러블슈팅](#8-트러블슈팅) 참고.
