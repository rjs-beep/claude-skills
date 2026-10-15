# /scan-trend-signals — Beauty Trend Intelligence Signal Synthesis

볼트: `C:\Users\rjs01\CCFM-LLM-Wiki-local`

## 역할

이 커맨드는 **신규 웹 검색을 최소화하거나 하지 않는다.**
역할은 Wiki에 이미 저장된 Observation을 종합해 Signal을 생성·갱신하는 것이다.

## 사용법

```
/scan-trend-signals           # 미연결 Observation 전체 처리
/scan-trend-signals PDRN      # 특정 Theme 관련 Observation만 처리
```

## 실행 절차 (순서 고정)

### Step 0. Preflight

`validate_trend_intelligence.py` 실행. 이슈 있으면 먼저 해결 후 진행.

### Step 1. 미연결 Observation 탐색

`08_TREND_INTELLIGENCE/OBSERVATIONS/` 전체를 읽어
Signal에 아직 연결되지 않은 Observation 목록 작성.

기준: 해당 Observation basename이 어느 Signal의 `observation_refs`에도 없는 경우.

### Step 2. 기존 Active Signal과 비교

각 미연결 Observation에 대해:
1. `08_TREND_INTELLIGENCE/SIGNALS/` 전체 읽기
2. Observation의 `themes`, `categories`, `observation_type`과 기존 Signal 비교
3. **동일 현상이면 기존 Signal에 연결** (새 Signal 생성 금지)
   - 기존 Signal의 `observation_refs`에 추가
   - `last_seen` 갱신
   - `entity_count`, `organization_count` 재계산
   - `signal_level` 재평가 (아래 Step 5)

### Step 3. 신규 Signal 후보 생성

기존 Signal에 매핑되지 않는 Observation이 있으면:
1. `signal_key` 결정 — 시장 현상 자체가 identity (브랜드명/인물명 사용 금지)
   - 올바른 예: `injectable-inspired-pdrn-skincare`
   - 잘못된 예: `medicube-pdrn`, `emily-weiss-ingredient`
2. 기존 Signal에 동일 `signal_key`가 없는지 확인 (중복 생성 금지)
3. status=`candidate`로 Signal 파일 생성
4. `observation_refs`에 해당 Observation 연결

### Step 4. Source Independence 확인

Signal Level 계산 전 Source Independence 검증:
- `original_source_url`이 같은 Observation은 독립 Source로 계산하지 않는다
- 브랜드 공식 채널(brand_official, press_release_repost) 여러 개 = 독립 Source 1개
- 독립적인 source_types 목록을 Signal의 `source_types`에 기록

### Step 5. Signal Level 계산

```
L0  단순 언급
L1  단일 의미 있는 변화 (Observation 1개)
L2  독립 Entity/Org 2개 이상에서 같은 변화
    조건: entity_count >= 2 OR organization_count >= 2
L3  3개 이상 독립 Source Type OR 2개 이상 국가에서 반복
    조건: len(source_types) >= 3 OR len(markets) >= 2
L4  정량 신호(검색량/판매/Rank 등) 동반
    조건: metric_status = linked
```

애매한 경우 한 단계 낮게 평가. 과도한 승격 금지.
단일 Observation만으로 L2 이상 불가.

### Step 6. Observation backlink 갱신

각 Observation 파일의 `## Related → Signal` 섹션에 연결된 Signal WikiLink 추가.

### Step 7. Active Signal Index 갱신

`build_trend_indexes.py` 실행:
```
python 99_SYSTEM/scripts/build_trend_indexes.py
```

### Step 8. log.md append

```
## [YYYY-MM-DD] ingest | trend-scan-signals

Processed observations: {N}건
New signals: {signal_key 목록}
Updated signals: {signal_key 목록}
Level upgrades: {signal_key: L1→L2 등}
```

## 출력 형식

```
## Signal Synthesis 완료

### 처리 결과
- 미연결 Observation: {N}건 처리
- 기존 Signal 업데이트: {N}건
- 신규 Signal 생성: {N}건 (모두 candidate 상태)
- Level 변경: {signal_key: L1→L2 등}

### 주의 사항
- {single entity 등 Level 미충족 사항 있으면 기록}

### 다음 권장 작업
- /beauty-intelligence-weekly (Signal이 충분히 축적되면)
```
