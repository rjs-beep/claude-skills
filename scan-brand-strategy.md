# /scan-brand-strategy — 브랜드전략 인사이트 스캔 & 위키 갱신

볼트: `C:\Users\rjs01\CCFM-LLM-Wiki-local`
대상: 김병훈(에이피알), Emily Weiss(Glossier), Rory Sutherland(Ogilvy)

## 목적

3인의 외부 발언·인터뷰·아티클에서 인사이트를 추출해 각 Entity 페이지를 갱신한다.
신제품 기획·벤치마킹에 즉시 인용 가능한 형태(결론→근거→출처 bullet)로 유지.

## 실행 절차

### 1. 소스 원문 확보

사용자가 제공한 소스(URL, 텍스트 붙여넣기, 파일 경로)를 확인한다.
소스가 없으면 먼저 요청한다: "스캔할 소스(기사/인터뷰/영상 스크립트)를 붙여넣어 주세요."

원문이 확보되면 `00_RAW/brand-strategy/` 아래 저장한다:
- 파일명: `YYYY-MM-DD_인물약칭_간단설명.md` (예: `2026-08-22_김병훈_포브스인터뷰.md`)
- 파일 내용: 원문 그대로 (수정 없이). 이 파일은 이후 절대 수정하지 않는다.

### 2. 대상 인물 식별

| 키워드 | 대상 파일 |
|---|---|
| 김병훈 / 에이피알 / APR / 메디큐브 / 에이프릴스킨 | `03_ENTITIES/PEOPLE/EXTERNAL/김병훈.md` |
| Emily Weiss / Glossier / Into The Gloss | `03_ENTITIES/PEOPLE/EXTERNAL/Emily_Weiss.md` |
| Rory Sutherland / Ogilvy / Alchemy / 행동경제학 | `03_ENTITIES/PEOPLE/EXTERNAL/Rory_Sutherland.md` |

### 3. 인사이트 추출 원칙

- **결론 → 근거 → 출처** 순서 bullet (3단 구조 고정)
- 수치·날짜·직접 인용구가 있으면 반드시 포함
- 의견·추측 제외. 발언·행동·결과·수치만 기록
- 신제품 기획·커뮤니티 마케팅·소비자 심리와 직결되는 정보 우선
- 한 소스에서 인사이트 3~7개가 적정. 무관한 내용은 추가하지 않는다

예시 형식:
```
- **[결론 한 줄]**
  - 근거: [발언/데이터/사건 구체 내용]
  - 출처: [소스 파일명 또는 URL, 날짜]
```

### 4. Entity 페이지 갱신

해당 인물의 `## 핵심 인사이트` 섹션 아래에 날짜 헤더로 추가:

```markdown
### YYYY-MM-DD — [소스 간단 설명]

- **[결론]**
  - 근거: ...
  - 출처: ...
```

규칙:
- 기존 bullet 삭제 없이 append
- "_미수집._" 플레이스홀더가 있으면 제거하고 실제 내용으로 교체
- frontmatter `sources` 배열에 소스 추가:
  ```yaml
  - type: article  # 또는 interview / video / podcast / book
    title: "소스 제목"
    date: "YYYY-MM-DD"
    raw_file: "00_RAW/brand-strategy/YYYY-MM-DD_인물약칭_설명.md"
  ```
- frontmatter `last_confirmed` 날짜 갱신
- frontmatter `my_context_status` → `populated` 로 변경 (첫 인사이트 추가 시)

### 5. INGEST 마무리

`index.md` Brand Strategy Insights 섹션의 해당 인물 줄에서 마지막 스캔 날짜 갱신.

`log.md` 맨 끝에 append:
```
## [YYYY-MM-DD] ingest | brand-strategy-scan | <인물명>

Updated:
- [[인물명]] (entity-external) — <추가된 인사이트 수>건 추가, 소스: <소스 제목>

Sources:
- 00_RAW/brand-strategy/<파일명>
```

## 출력

스캔 완료 후 다음을 출력한다:
1. 추가된 인사이트 수 및 요약 (bullet)
2. 갱신된 파일 목록
3. 다음 스캔 추천 소스 힌트 (있으면)
