# /beauty-intelligence-weekly — Beauty Intelligence Weekly Report

볼트: `C:\Users\rjs01\CCFM-LLM-Wiki-local`

## 역할

이미 축적된 Observation / Signal / Theme를 종합한다.
**새로운 웹 검색을 하지 않는다.**
Wiki에 없는 새로운 사실을 Weekly Report에서 만들어내지 않는다.

## 사용법

```
/beauty-intelligence-weekly           # 이번 주 (자동 계산)
/beauty-intelligence-weekly 2026-W34  # 특정 주 지정
```

## 실행 절차

### Step 0. Preflight

1. `validate_trend_intelligence.py` 실행 — 이슈 있으면 경고 후 계속
2. 주차 범위 계산 (ISO 8601 주차 기준)

### Step 1. 데이터 수집

해당 주 범위(`observed_at` 기준)에 해당하는:
- 신규 Observation 목록
- 갱신된 Signal 목록 (`last_seen`이 해당 주인 Signal)
- Level 변경된 Signal

**이 주에 Observation/Signal이 없으면 Report를 생성하지 않는다.**
빈 Report는 만들지 않는다.

### Step 2. 핵심 변화 선별

**Executive Summary**용 핵심 변화 최대 3~5개 선별 기준:
- Signal Level이 가장 높은 것 우선
- 이번 주 신규로 L2 이상 달성한 Signal 우선
- 복수 시장에서 동시 관찰된 것 우선

### Step 3. Report 작성

**Fact와 Inference를 반드시 구분한다:**
- `> **Fact**: ...` — Observation에서 직접 확인된 사실
- `> **Inference**: ...` — 패턴에서 유추한 해석 (불확실성 명시)

Observation/Signal에 없는 내용을 추가하지 않는다.

### Step 4. 파일 저장

`08_TREND_INTELLIGENCE/REPORTS/WEEKLY/YYYY-Www.md`

### Step 5. log.md append

```
## [YYYY-MM-DD] ingest | beauty-intelligence-weekly | {week}

Report: 08_TREND_INTELLIGENCE/REPORTS/WEEKLY/{week}.md
Observations referenced: {N}건
Signals referenced: {N}건
New signals in report: {목록}
Accelerating signals: {목록}
```

## Report 구조

```markdown
---
type: trend_weekly_report
week: "YYYY-Www"
generated_at: "YYYY-MM-DD"
observation_count: 0
signal_count: 0
---

# Beauty Intelligence — YYYY Wxx

## Executive Summary

가장 중요한 변화 최대 3~5개. 각 항목은 Signal 또는 Observation WikiLink 포함.

## New Signals

이번 주 신규 L2 이상 Signal.
_없으면 섹션 생략._

## Accelerating Signals

기존보다 Observation이 추가된 Signal (Level 변경 여부 무관).
_없으면 섹션 생략._

## Emerging Cross-Market Patterns

복수 국가에서 나타난 패턴.
_없으면 섹션 생략._

## People to Watch

이번 주 의미 있는 Observation을 만든 사람.
| 이름 | Observation | 요약 |
|---|---|---|

## Organizations to Watch

이번 주 의미 있는 변화를 보인 조직.
| 조직 | Observation | 요약 |
|---|---|---|

## BM Implications

상품기획 관점의 해석.

> **Fact**: (Observation/Signal에서 직접 확인된 내용)
> **Inference (불확실)**: (패턴 기반 해석)

## Watch Next Week

아직 확정되지 않은 Candidate Signal 또는 추가 확인이 필요한 Observation.

## Sources

이번 리포트에 참조된 Observation/Signal 전체 목록.
- [[obs-...]]
- [[signal-key]]
```

## 안전 원칙

1. Observation/Signal에 없는 사실을 Report 본문에 추가하지 않는다.
2. "이번 주 트렌드로 보인다"류 추측성 종합은 Inference 태그를 반드시 단다.
3. 검색 기반 내용이 필요하면 `/scan-trend-people` 또는 `/scan-trend-orgs`를 먼저 실행한다.
4. 빈 주(Observation 0건)에는 Report 파일을 생성하지 않는다.
