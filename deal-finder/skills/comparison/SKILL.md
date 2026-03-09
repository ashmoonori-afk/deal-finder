---
name: shop-compare
description: >
  This skill should be used when the user asks to "compare products",
  "which one is better", "비교해줘", "뭐가 더 나아?", "A vs B",
  "차이가 뭐야?", "어떤 걸 사야 해?", or needs a side-by-side
  comparison of 2-4 products with specs, pricing, and verdict.
version: 0.1.0
---

# Shop Comparison Agent

2~4개 제품의 스펙, 가격, 리뷰를 나란히 비교하고, **실질 구매가 기준**으로 최종 추천 결론을 도출한다. Proactive Discovery로 사용자가 미처 생각하지 못한 제품도 비교 대상에 추가한다.

## 필수 레퍼런스 참조

- `references/comparison-table-template.md` — 비교표 HTML 템플릿
- `references/deal-score-formula.md` — Deal Score 계산 상세 로직

## 입력 / 출력

- **입력**: `research_output.json` 또는 사용자 지정 제품 2~4개
- **출력**: `comparison_output.json` → `comparison.html` (워크스페이스 저장)

## 실행 파이프라인

### Step 1: 비교 대상 확정

**경로 A — research_output.json 연계:**
- `research_output.json`의 상위 2-4개 제품을 자동 선택
- `discovery.alternatives`에 Deal Score가 높은 대안이 있으면 비교 대상에 포함
- "사용자가 요청한 제품 vs 우리가 찾은 더 나은 대안" 구도 권장

**경로 B — 사용자 직접 지정:**
- 사용자가 "A vs B" 형태로 제품을 명시한 경우
- 해당 제품들의 정보가 research_output에 없으면 추가 리서치 수행

**경로 C — Proactive 추가:**
- 사용자가 2개만 비교 요청했더라도, 가성비 관점에서 주목할 제품이 있으면:
  - "비교 요청하신 2개 외에, 이 제품도 함께 비교해봤습니다" 형태로 제안
  - 최대 4개까지만 (그 이상은 가독성 저하)

### Step 2: 상세 데이터 수집

각 비교 대상 제품에 대해 아래 정보 수집:

**필수 수집 항목:**
- 전체 스펙 (카테고리에 따라 동적 결정)
- 현재 가격 + 실질 구매가 (Best Deal Optimizer 적용)
- 평점 + 리뷰수 + 리뷰 요약 (장단점 각 3개)
- 보증기간, 반품정책
- 배송 조건 (비용, 소요일)

**카테고리별 핵심 비교 스펙 예시:**

| 카테고리 | 핵심 스펙 |
|----------|-----------|
| 이어폰/헤드폰 | 드라이버, ANC, 코덱, 배터리, 무게, 방수 |
| 스마트폰 | 디스플레이, AP, RAM/저장, 카메라, 배터리, OS |
| 노트북 | CPU, GPU, RAM, SSD, 디스플레이, 무게, 배터리 |
| 가전 | 용량/크기, 에너지등급, 소음, 핵심 기능 |
| 의류/패션 | 소재, 사이즈, 컬러, 세탁법, 원산지 |

부족한 데이터는 Claude in Chrome으로 제품 페이지 추가 스크래핑.

### Step 3: 비교 매트릭스 구성

`spec_matrix` 구조:
- **columns**: ["스펙", 제품1 이름, 제품2 이름, ...]
- **rows**: 각 스펙 항목별 값 + `winner` 인덱스

**행 구성 순서:**
1. 💰 가격 (현재가)
2. 💰 실질 구매가 (best_net_price) ← **이것이 진짜 비교 기준**
3. 📊 Deal Score
4. 카테고리별 핵심 스펙들 (중요도 순)
5. ⭐ 평점 / 리뷰수
6. 🚚 배송 / 반품

**winner 판정 기준:**
- 숫자 비교: 높을수록 좋은 항목 (배터리, 평점 등) → max가 winner
- 숫자 비교: 낮을수록 좋은 항목 (무게, 가격 등) → min이 winner
- 정성 비교: 등급/텍스트 → 우열 판단하여 winner 설정
- 동점: `winner: -1` (무승부)

### Step 4: Deal Score 산출 및 종합 판정

각 제품에 Deal Score 산출 (실질 구매가 기준):

```
Deal Score = (Quality × 0.30) + (Price Value × 0.25) + (Reviews × 0.20)
           + (Shipping × 0.10) + (Return Policy × 0.10) + (Brand Trust × 0.05)

Price Value는 best_net_price 기준으로 계산
```

**verdict 구성:**
- `winner`: Deal Score 최고 제품
- `reason`: 1-2문장 근거 ("가성비와 노캔 성능 모두 우수")
- `runner_up`: 2위 제품
- `runner_up_reason`: 2위가 더 나은 상황 설명 ("애플 생태계 사용자에게는...")
- `proactive_pick`: Proactive Discovery로 추가된 제품이 있다면 별도 언급

### Step 5: HTML 비교표 생성

`comparison_output.json`을 기반으로 비교표 HTML 생성:

**레이아웃:**
- 고정 헤더: 제품 이미지 + 이름 + 실질가 (스크롤 시 고정)
- 스펙 행: 교대 배경색 (가독성)
- Winner 셀: 배경 `#FDE047` (노랑 하이라이트)
- Deal Score 행: 프로그레스 바 + 숫자
- 가격 행: 정가(취소선) + 현재가 + **실질 구매가(강조)**
- 혜택 요약: 각 제품 열 아래 "💳 {최적 카드} 시 ₩{절약액} 추가 할인"

**하단 Verdict 배너:**
- 큰 폰트로 "🏆 종합 추천: {제품명}"
- 추천 이유 1-2문장
- 각 제품별 "이런 사람에게 추천" 태그
- 전체 제품 구매 링크 버튼

**기술 스택:**
- 순수 HTML + CSS (단일 파일)
- 반응형 테이블 (좁은 화면에서 가로 스크롤)
- 색상 코딩: best_deal(초록), good_deal(파랑), neutral(회색), overpriced(빨강)

### Step 6: Gate Check & 저장

1. Gate 2c 검증 (products ≥ 2, rows ≥ 3, deal_scores 길이 == products 길이, winner 유효)
2. `comparison.html`을 워크스페이스에 저장
3. 사용자에게 파일 링크 + 결론 제공

## 사용자 응답 형식

```
📊 비교 분석 완료: {제품1} vs {제품2} {vs 제품3}

🏆 종합 추천: {winner}
  - Deal Score: {점수}/100
  - 실질 구매가: ₩{가격} (💳 {최적 카드} 적용)
  - 핵심 장점: {1-2줄}

🥈 차선: {runner_up}
  - {이런 상황이면 이 제품이 더 나은 이유}

💡 {proactive_pick이 있으면}
  - "비교 중 발견: {제품명}도 주목할 만합니다 — {이유}"

[비교표 보기](computer:///path/to/comparison.html)
```
