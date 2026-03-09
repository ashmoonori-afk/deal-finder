---
name: shop-price
description: >
  This skill should be used when the user asks to "track price",
  "show price history", "가격 추이 보여줘", "가격 변동",
  "지금 사도 돼?", "가격 그래프", "매수 타이밍", "언제 사는 게 좋아?",
  or wants price trend analysis, buy/wait recommendations, or
  historical price charts for specific products.
version: 0.1.0
---

# Shop Price Graph Agent

특정 제품의 **실시간 가격 수집 + 히스토리컬 가격 추이 차트**를 생성하고, 매수/대기 판단과 다음 세일 예측까지 제공한다.

## 필수 레퍼런스 참조

- `references/platform-selectors.md` — 플랫폼별 가격 요소 CSS 셀렉터
- `references/chartjs-templates.md` — Chart.js 차트 렌더링 템플릿

## 입력 / 출력

- **입력**: `research_output.json` (특정 product_id) 또는 사용자 직접 URL
- **출력**: `price_output.json` → `price-graph.html` (워크스페이스 저장)

## 실행 파이프라인

### Step 1: 대상 제품 확정

**경로 A — research_output.json 연계:**
- `research_output.json`에서 사용자가 지정한 product_id 또는 Deal Score 1위 제품 선택
- 이미 수집된 가격/플랫폼 정보 활용

**경로 B — 사용자 직접 URL:**
- 사용자가 제품 URL을 직접 제공한 경우 (Research Agent 스킵)
- Claude in Chrome으로 해당 URL 접근 → 제품명, 현재 가격 수집

### Step 2: 현재 가격 실시간 수집

Claude in Chrome으로 제품 페이지 접근:
1. 현재 판매가 추출
2. 할인 전 정가 추출 (있을 경우)
3. 쿠폰/카드 혜택 표시 여부 확인
4. 재고/품절 상태 확인
5. 타임스탬프 기록 (`price_scraped_at`)

**크로스 플랫폼 가격 수집:**
동일 제품의 다른 플랫폼 가격도 수집하여 `platform_prices[]`에 저장:
- 다나와/에누리에서 최저가 비교
- 주요 플랫폼 3-5개의 현재가

### Step 3: 히스토리컬 가격 데이터 탐색

WebSearch + Claude in Chrome을 활용하여 과거 가격 데이터 수집:

**데이터 소스 우선순위:**
1. 다나와 가격변동 그래프 (가장 신뢰도 높음)
2. 에누리 가격 추이
3. 네이버 쇼핑 가격 비교 히스토리
4. 커뮤니티 게시글 (뽐뿌, 클리앙 핫딜 게시판) — 과거 세일가 참조
5. 해외 제품: CamelCamelCamel (Amazon), Keepa

**수집 기간**: 최근 6개월 (config.limits.price_history_months)

**최소 데이터 포인트**: 3개 이상. 2개 미만 시 "히스토리 데이터 부족" 경고 표시.

### Step 4: 통계 분석

수집된 데이터 포인트로 아래 통계 산출:

| 지표 | 계산 방법 |
|------|-----------|
| `highest` | 기간 내 최고가 |
| `lowest` | 기간 내 최저가 (역대 최저가) |
| `average` | 기간 내 평균가 |
| `current_vs_lowest` | (현재가 - 최저가) / 최저가 × 100 |
| `current_vs_average` | (현재가 - 평균가) / 평균가 × 100 |
| `trend` | 최근 3개 데이터 포인트 기울기 → "upward" / "downward" / "stable" |

### Step 5: 매수/대기 판단

아래 규칙에 따라 `recommendation.verdict`를 결정:

```
BUY (매수 적기):
  - 현재가 ≤ 역대 최저가 × 1.05 (최저가 근접 5% 이내)
  - 또는: 현재가 < 평균가 × 0.85 (평균 대비 15% 이상 저렴)
  - 또는: trend == "upward" AND 현재가 < 평균가 (오르기 전 진입)

WAIT (대기 추천):
  - 현재가 > 평균가 × 1.10 (평균 대비 10% 이상 비쌈)
  - 또는: trend == "downward" (하락 추세)
  - 또는: 30일 이내 예상 세일 이벤트 존재

NEUTRAL (판단 보류):
  - 위 조건에 해당하지 않는 경우
  - 또는: 데이터 포인트 부족 (3개 미만)
```

**다음 세일 예측:**
- 연간 주요 세일 캘린더 참조: 쿠팡 로켓세일(3,6,9,12월), 11번가 십일절(11월), 네이버 쇼핑라이브(수시), 블랙프라이데이(11월), 아마존 프라임데이(7월)
- 해당 플랫폼의 다음 예상 세일까지의 기간 계산

### Step 6: HTML 차트 생성

`price_output.json`을 기반으로 Chart.js 인터랙티브 차트 HTML 생성:

**차트 구성:**
- **메인 차트**: Line chart — 가격 추이 (X: 날짜, Y: 가격)
- **평균선**: 점선으로 평균가 표시
- **최저가선**: 초록 점선으로 역대 최저가 표시
- **현재가 포인트**: 강조 마커 (빨간 원, 크게)
- **이벤트 마커**: 세일/프로모션 시점에 라벨 표시 ("11.11 세일", "블프" 등)
- **툴팁**: 호버 시 날짜, 가격, 플랫폼, 이벤트 정보

**차트 하단 정보 패널:**
- 통계 요약: 최고가 / 최저가 / 평균가
- 매수/대기 판정: 큰 배지로 "✅ 매수 적기" 또는 "⏳ 대기 추천"
- 판정 근거: 1-2문장 설명
- 다음 세일 예측: "📅 다음 예상 세일: {이벤트명} ({날짜})"
- 크로스 플랫폼 가격 비교: 다른 플랫폼 현재가 표 (있을 경우)

**기술 스택:**
- Chart.js (CDN: `https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js`)
- 단일 HTML 파일, 인라인 CSS
- 반응형 (max-width: 800px)

### Step 7: Gate Check & 저장

1. Gate 2b 검증 (data_points ≥ 2, verdict 유효)
2. `price-graph.html`을 워크스페이스에 저장
3. 사용자에게 파일 링크 + 요약 제공

## 사용자 응답 형식

```
📈 {제품명} 가격 분석 완료

현재가: ₩{현재가} ({플랫폼})
역대 최저가: ₩{최저가} ({날짜})
6개월 평균: ₩{평균가}

{✅ 매수 적기 / ⏳ 대기 추천}: {판정 근거 1줄}
📅 다음 예상 세일: {이벤트명} ({예상 시기})

[가격 그래프 보기](computer:///path/to/price-graph.html)
```
