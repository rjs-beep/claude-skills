# /scan-trend-people — Beauty Trend Intelligence People Scan

볼트: `C:\Users\rjs01\CCFM-LLM-Wiki-local`

## 사용법

```
/scan-trend-people            # Watchlist Priority A 전체
/scan-trend-people A          # Priority A만
/scan-trend-people B          # Priority B만
/scan-trend-people A 7d       # Priority A, 최근 7일만 검색
/scan-trend-people Emily_Weiss  # 특정 인물 지정
```

## 실행 절차 (순서 고정)

### Step 0. Preflight

1. `99_SYSTEM/scripts/validate_trend_intelligence.py` 실행 — 기존 이슈 없는지 확인
2. `99_SYSTEM/state/trend_scan_state.json` 읽기 — last_scan 기준 incremental 범위 계산
3. `08_TREND_INTELLIGENCE/WATCHLISTS/PEOPLE.md` 읽기 — 대상 및 Priority 확인

### Step 1. 대상 결정

Priority 인자 없으면 A만 대상.
대상이 없으면 "Watchlist에 Priority A 인물이 없음"을 출력하고 종료.

각 대상 인물에 대해 해당 Entity 파일(`03_ENTITIES/PEOPLE/EXTERNAL/`)을 읽어
`last_scanned`, `source_channels`, `canonical_sources`, `expertise`, `signal_domains` 확인.

### Step 2. Run ID 생성

```
trend-run-YYYY-MM-DD-NNN
```

NNN은 당일 실행 순서 (001부터). manifest 파일명에 사용.

### Step 3. 검색 실행

각 인물에 대해 WebSearch 실행:
- `last_scan` 이후 자료만 검색 (null이면 최근 30일)
- 검색 키워드: `"{이름}" {signal_domains} {YYYY}` 등 조합
- 국가별 검색: source_channels/markets 기준

**중요 규칙**:
- 검색 결과가 없어도 "변화 없음"으로 기록하지 않는다.
  → `markets_missing` 또는 `coverage_notes`에 기록한다.
- 검색 실패(타임아웃, 차단 등)는 `errors`에 기록한다.

### Step 4. 변화 선별 — Observation 생성 기준

다음 중 하나 이상이 실제 확인된 경우에만 Observation 생성:
- 신규 제품 / 성분 / 제형
- 새로운 카테고리 진입
- 기존과 다른 핵심 메시지
- 광고/콘텐츠 소구의 의미 있는 변화
- 유통채널 변화 / 시장 확장
- 시술 또는 홈케어 연결 신규 포지셔닝
- 반복적으로 등장하기 시작한 소비자 표현
- R&D 방향 변화
- 플랫폼/리테일러 랭킹 또는 노출 변화
- 이전과 다른 주제를 반복적으로 다루기 시작

의미 있는 변화가 없으면 Observation을 생성하지 않는다.

### Step 5. Observation 중복 체크

신규 Observation 생성 전:
1. `observation_topic` 결정: 이 Observation이 다루는 시장 변화의 normalized slug
   - 소문자, 영어, 하이픈만. 패턴: `^[a-z][a-z0-9-]{2,39}$`
   - predefined prefix 우선 사용: `pdrn-*` `exosome-*` `home-rf-*` `procedure-inspired-*`
     `scalp-*` `new-product-*` `ingredient-*` `messaging-*` `positioning-*` `channel-*`
   - 같은 현상 = 같은 slug. 생성 전 기존 Observation slug 반드시 확인
2. `dedupe_key` 계산: `normalize_url(source_url) + "-" + entity_basename + "-" + observation_topic + "-" + published_at`
   - normalize_url: 소문자, 프로토콜 제거, query param 제거, trailing slash 제거,
     비알파뉴메릭 → 하이픈, 60자 이하
   - 예: `"brand-com-interview-2026-emily-weiss-pdrn-regenerative-skincare-2026-08-20"`
3. 기존 Observation 파일에서 동일 `dedupe_key` 검색
4. 중복이면 기존 파일에 메모만 남기고 새 파일 생성 금지
5. 재게시 판단: `original_source_url`이 같으면 독립 Source 아님

**같은 인터뷰·소스에서 다른 시장 변화가 발견된 경우**: `observation_topic`이 달라지므로
`dedupe_key`도 달라진다 → 각각 별도 Observation 파일로 생성 가능 (정상 동작)

### Step 6. Observation 파일 생성

파일 경로: `08_TREND_INTELLIGENCE/OBSERVATIONS/2026/obs-YYYYMMDD-{slug}-{nnnn}.md`
템플릿: `99_SYSTEM/templates/trend_intelligence/observation.md`

signal_level은 이 단계에서 L0 또는 L1만 사용.
L2 이상은 `/scan-trend-signals`에서만 판단.

### Step 7. Entity 페이지 갱신

각 Observation과 연결된 Entity 파일의:
- `last_scanned` 날짜 갱신
- `sources` frontmatter에 소스 추가 (provenance)
- `## Recent Signals` 섹션에 Observation WikiLink 추가 (내용 복사 금지)

### Step 8. Scan State 갱신

`99_SYSTEM/state/trend_scan_state.json` 업데이트:
- `last_scan` = 오늘 날짜
- `last_successful_scan` = 검색 성공한 경우
- `coverage_notes` = 검색 못 한 국가/플랫폼 메모

### Step 9. Run Manifest 저장

`99_SYSTEM/manifests/trend_intelligence/{run_id}.json`
템플릿: `99_SYSTEM/manifests/trend_intelligence/run_manifest_template.json`

반드시 `markets_requested`와 `markets_covered`를 분리 기록.
검색 못 한 국가는 `markets_missing`에 기록.

### Step 10. log.md append

```
## [YYYY-MM-DD] ingest | trend-scan-people | {run_id}

Scanned: {인물 목록}
Created: [[obs-...]] (있으면)
Coverage gaps: (없으면 "없음")
Run manifest: 99_SYSTEM/manifests/trend_intelligence/{run_id}.json
```

## 출력 형식

```
## Scan 완료 — {run_id}

### 결과 요약
- 대상: {N}명
- Observation 생성: {N}건
- Observation 중복 스킵: {N}건
- Coverage 누락: {목록 또는 "없음"}
- 오류: {목록 또는 "없음"}

### 생성된 Observations
- [[obs-...]] — {인물} — {요약 한 줄}

### 다음 권장 작업
- /scan-trend-signals (Observation이 생성된 경우)
```
