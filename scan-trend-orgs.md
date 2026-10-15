# /scan-trend-orgs — Beauty Trend Intelligence Orgs Scan

볼트: `C:\Users\rjs01\CCFM-LLM-Wiki-local`

## 사용법

```
/scan-trend-orgs              # Watchlist Priority A 전체
/scan-trend-orgs A
/scan-trend-orgs B
/scan-trend-orgs A 14d        # Priority A, 최근 14일
/scan-trend-orgs Glossier     # 특정 조직 지정
```

## 탐색 우선 변화 유형

조직 Scan에서 아래 변화를 우선 탐색한다:

- **launch**: 신규 제품 / 컬렉션 출시
- **ingredient**: 새로운 성분 채택 또는 강조
- **formulation**: 제형 변화 (텍스처, 제형 타입)
- **positioning**: 핵심 메시지 또는 소구 변화
- **campaign**: 신규 캠페인 / 크리에이티브 방향 변화
- **retailer_expansion**: 유통채널 확장 (신규 리테일러)
- **country_expansion**: 신규 국가 진출
- **ranking**: 플랫폼/리테일러 랭킹 또는 노출 변화
- **rd_direction**: R&D / 특허 / 연구 협업 변화
- **device_procedure_msg**: 디바이스 또는 시술 연결 메시지 채택

## 실행 절차

### Step 0. Preflight

1. `validate_trend_intelligence.py` 실행
2. `trend_scan_state.json` 읽기 — 마지막 scan 이후 기간 계산
3. `08_TREND_INTELLIGENCE/WATCHLISTS/ORGS.md` 읽기

### Step 1~10

`/scan-trend-people`과 동일 구조를 따른다.

대상 Entity 파일 위치: `03_ENTITIES/EXTERNAL_ORGS/`

차이점:
- Entity 파일의 `organization_type`에 따라 검색 방향 조정
  - `brand`: 제품 출시, 광고 캠페인, SNS 소구 변화 중심
  - `manufacturer_odm`: OEM 성분/제형 공급 트렌드 중심
  - `retailer`: 입점 브랜드 변화, 카테고리 큐레이션 중심
  - `platform`: 알고리즘/노출 변화, 급부상 카테고리 중심
  - `media_research`: 보고서 발행, 트렌드 리포트 중심

### Source Independence 주의

- 브랜드 공식 보도자료를 재게시한 기사 다수 = 독립 Source 1개로만 계산
- `original_source_url` 필드로 재게시 여부 추적
- 검색 결과에서 동일 원본 기사가 3곳에 있어도 독립 Source 카운트는 1

## 출력 형식

```
## Scan 완료 — {run_id}

### 결과 요약
- 대상 조직: {N}개
- Observation 생성: {N}건
- Coverage 누락: {목록 또는 "없음"}

### 생성된 Observations
- [[obs-...]] — {조직} — {변화 요약 한 줄}

### 다음 권장 작업
- /scan-trend-signals
```
