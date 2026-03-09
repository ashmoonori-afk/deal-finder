---
name: shop-moodboard
description: >
  This skill should be used when the user asks to "make a moodboard",
  "show me options visually", "무드보드 만들어줘", "한눈에 보여줘",
  "카드로 정리해줘", "비주얼로 보고 싶어", or wants a visual product
  board with images, prices, and comparison cards rendered as HTML.
  Also auto-suggested by Research Agent when 3+ products are found,
  alternatives exist, or the user is browsing a category (not searching
  for a single specific product).
version: 0.2.0
---

# Shop Moodboard Agent — Pinterest Style

Research Agent의 결과를 **핀터레스트 스타일 매이슨리 보드**로 변환한다.
메인 검색 결과뿐 아니라 **연관 상품, 유사 상품, 함께 사면 좋은 상품**까지 포함하여
사용자가 한 화면에서 탐색-비교-결정할 수 있는 **비주얼 쇼핑 허브**를 생성한다.

## 필수 레퍼런스 참조

- `references/html-templates.md` — 무드보드 HTML/CSS 베이스 템플릿
- `references/card-design-rules.md` — 카드 디자인 가이드라인
- `references/pinterest-layout.md` — 핀터레스트 매이슨리 레이아웃 CSS 패턴

## 입력 / 출력

- **입력**: `research_output.json` (Research Agent 출력)
- **출력**: `moodboard_output.json` → `moodboard.html` (워크스페이스 저장)

## 실행 파이프라인

### Step 1: 데이터 로드 및 검증

`research_output.json`을 읽고 아래를 확인:
- `products[]` 배열이 1개 이상 존재
- 각 product에 `id`, `name`, `pricing` 존재
- `discovery.alternatives[]` 존재 시 대안 카드도 포함 대상

### Step 2: 연관/유사 상품 탐색 (Pinterest Discovery)

**이 단계가 핀터레스트와의 핵심 차별점이다. 반드시 실행한다.**

메인 검색 결과를 기반으로 3가지 유형의 연관 상품을 추가 탐색:

**Type A — Similar (유사 상품):**
- 검색 결과와 같은 카테고리이지만 다른 브랜드/라인업의 제품
- 검색 쿼리: `"{카테고리} 추천" "{카테고리} 인기" "{카테고리} 베스트"`
- 메인 결과에 없는 제품 중 평점 4.0 이상, 리뷰 100개 이상
- 예: 소니 이어폰 검색 → 젠하이저, B&O, JBL 등 유사 라인

**Type B — Companion (함께 사면 좋은 상품):**
- 메인 제품과 함께 사용하면 좋은 액세서리/소모품
- 검색 쿼리: `"{제품명} 케이스" "{제품명} 액세서리" "{카테고리} 필수 액세서리"`
- 번들 할인이 있으면 우선 표시
- 예: 이어폰 검색 → 전용 케이스, 이어팁, 케이블

**Type C — Cross-category (카테고리 확장):**
- 같은 use_case를 다른 형태로 해결하는 제품
- 사용자의 `use_case`를 기반으로 추론
- 예: "출퇴근 음악감상" → 이어폰 검색했지만 오버이어 헤드폰도 표시
- 예: "홈오피스 모니터" → 모니터암, 웹캠도 함께 표시

**연관 상품 수량:**
- Similar: 2-4개
- Companion: 1-3개
- Cross-category: 1-2개
- 전체 연관 상품 합계: 최대 8개 (보드가 너무 길어지지 않도록)

### Step 3: 이미지 수집

각 제품(메인 + 연관)의 대표 이미지 URL 수집:
1. `research_output.json`에 이미지 URL이 이미 있으면 사용
2. 없으면 Claude in Chrome으로 제품 페이지 접근하여 메인 이미지 URL 추출
3. 이미지 접근 불가 시 CSS 그라데이션 플레이스홀더 (제품명 + 카테고리 아이콘)

**이미지 품질 기준:**
- 최소 300px 너비
- 정사각형 또는 세로형 우선 (핀터레스트 레이아웃에 적합)
- 흰 배경 제품 사진 선호

### Step 4: 카드 데이터 구성

#### 메인 카드 (검색 결과 + 대안)

| 카드 요소 | 소스 | 표시 형식 |
|-----------|------|-----------|
| 제품 이미지 | Step 3 | 다양한 높이 허용 (매이슨리) |
| 제품명 | `product.name` | 최대 2줄, 말줄임 |
| 정가 | `pricing.original_price` | 취소선, 작은 텍스트 |
| 현재가 | `pricing.current_price` | 중간 크기 |
| 실질 구매가 | `pricing.best_net_price` | **가장 크게, 초록색** |
| 할인율 | `pricing.discount_pct` | 빨간 라운드 배지 "-28%" |
| 혜택 한줄 | `pricing.best_scenario_label` | "💳 삼성카드+쿠폰 적용" |
| 플랫폼 | `pricing.platform` | 작은 배지 (쿠팡/네이버 등) |
| 평점/리뷰 | `reviews` | ★ 4.6 (12,840) |
| Deal Score | `deal_score` | 얇은 프로그레스 바 |
| 태그 | `discovery.layer` | 🏆 추천 / 💡 대안 / 🔥 가성비 |
| CTA | `pricing.url` | "최저가 보기 →" 링크 (새 탭) |

#### 연관 카드 (Related Products)

| 카드 요소 | 소스 | 표시 형식 |
|-----------|------|-----------|
| 제품 이미지 | Step 3 | 매이슨리 높이 |
| 제품명 | `related.display_name` | 최대 2줄 |
| 연관 이유 | `related.relation_reason` | 이탤릭, 회색 텍스트 |
| 가격 | `related.best_net_price` | 실질가만 표시 |
| 연관 타입 배지 | `related.relation_type` | 🔗 유사 / 🧩 함께쓰면 / 🔄 다른형태 |
| 태그 | `related.tags` | 작은 pill 뱃지들 |
| CTA | `related.cta_url` | "상품 보기 →" 링크 (새 탭) |

### Step 5: Pinterest 매이슨리 HTML 렌더링

**레이아웃 — CSS Masonry (핀터레스트 스타일):**

```
┌─────────────────────────────────────────────────────┐
│  🛒 노이즈캔슬링 무선 이어폰 TOP 5                    │
│  30만원 이하 | 2026년 3월 기준                        │
│  [가격대 ▼] [플랫폼 ▼] [평점 ▼]                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌────────┐  ┌──────────┐  ┌────────┐              │
│  │🏆      │  │💡        │  │🔥      │              │
│  │ [img]  │  │ [img]    │  │ [img]  │              │
│  │        │  │          │  │        │              │
│  │ XM5    │  │ 경쟁제품  │  │ 가성비  │              │
│  │₩221K   │  │          │  │ 파괴자  │              │
│  │최저가→  │  │ ₩189K   │  │₩155K   │              │
│  └────────┘  │ 최저가→  │  │최저가→  │              │
│  ┌────────┐  └──────────┘  └────────┘              │
│  │🏆      │  ┌──────────┐  ┌────────┐              │
│  │ [img]  │  │🏆        │  │💡      │              │
│  │ 2위    │  │ [img]    │  │ [img]  │              │
│  │₩199K   │  │ 3위      │  │        │              │
│  │최저가→  │  │₩215K    │  │ ₩175K  │              │
│  └────────┘  │최저가→   │  │최저가→  │              │
│              └──────────┘  └────────┘              │
│                                                     │
├─────────────────────────────────────────────────────┤
│  💡 이런 상품은 어때요?                               │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌────────┐  ┌──────────┐  ┌────────┐              │
│  │🔗 유사 │  │🧩 함께   │  │🔄 다른  │              │
│  │ [img]  │  │ [img]    │  │ 형태   │              │
│  │헤드폰  │  │ 전용케이스│  │ [img]  │              │
│  │₩245K   │  │ ₩12K    │  │ 스피커  │              │
│  │상품보기 │  │ 상품보기  │  │₩189K   │              │
│  └────────┘  └──────────┘  │상품보기  │              │
│                            └────────┘              │
└─────────────────────────────────────────────────────┘
```

**CSS 구현 핵심:**
```css
/* Pinterest Masonry with CSS columns */
.masonry-grid {
  column-count: 3;
  column-gap: 16px;
}
.masonry-grid .card {
  break-inside: avoid;
  margin-bottom: 16px;
  border-radius: 16px;          /* 핀터레스트 둥근 모서리 */
  overflow: hidden;
  box-shadow: 0 1px 6px rgba(0,0,0,0.08);
  transition: transform 0.2s, box-shadow 0.2s;
}
.masonry-grid .card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 16px rgba(0,0,0,0.15);
}
@media (max-width: 768px) { .masonry-grid { column-count: 2; } }
@media (max-width: 480px) { .masonry-grid { column-count: 1; } }
```

**색상 팔레트:**
- 배경: `#FAFAFA` (연한 회색, 핀터레스트 느낌)
- 카드 배경: `#FFFFFF`
- 실질가 텍스트: `#16A34A` (초록)
- 할인 배지: `#EF4444` 배경 + 흰 텍스트
- 플랫폼 배지: 각 플랫폼 브랜드 컬러
- 연관 상품 섹션 배경: `#F0F9FF` (연한 파랑)
- CTA 링크: `#2563EB` (파랑, 호버 시 밑줄)

**카드 내부 구조:**
1. 이미지 (전체 너비, 비율 유지, hover 시 살짝 확대)
2. 태그 배지 (이미지 좌상단 오버레이)
3. 제품명 (bold)
4. 가격 블록: 정가(취소선) → 현재가 → **실질가(크게, 초록)**
5. 혜택 한줄 요약 (작은 회색 텍스트)
6. 평점 + 리뷰수 (한 줄)
7. Deal Score 바
8. "최저가 보기 →" 또는 "상품 보기 →" 링크

**필수 포함 요소:**
- 상단 헤더: 제목 + 날짜 + 필터 영역
- 메인 섹션: 검색 결과 카드들 (매이슨리)
- 구분선 + "💡 이런 상품은 어때요?" 헤더
- 연관 섹션: 연관 상품 카드들 (매이슨리, 약간 작은 사이즈)
- 모든 링크: `target="_blank"` (새 탭에서 열기)
- 각 카드 하단: 해당 쇼핑몰 직접 링크

**HTML 기술 스택:**
- 순수 HTML + CSS (프레임워크 없음, 단일 파일)
- CSS columns (매이슨리) + Flexbox (카드 내부)
- 이미지: `<img>`, object-fit: cover, lazy loading
- 호버 애니메이션: CSS transition

### Step 6: Gate Check & 저장

1. Gate 2a 검증 실행 (cards ≥ 1, 각 card에 product_id/cta_url 존재)
2. `moodboard.html`을 워크스페이스 폴더에 저장
3. 사용자에게 파일 링크 제공

## 사용자 응답 형식

```
📌 핀터레스트 스타일 무드보드를 생성했습니다!

🛒 메인 결과: {N}개 제품 + {M}개 대안
💡 연관 상품: {K}개 (유사 상품, 함께 쓰면 좋은 상품, 다른 형태)

[무드보드 보기](computer:///path/to/moodboard.html)

🏆 1위: {제품명} — 실질 ₩{가격} (정가 대비 -{할인}%)
💡 주목: {대안 제품} — {대안 이유}
🔗 연관 추천: {연관 상품} — {연관 이유}
```

## Related Products 탐색 JSON Schema

`moodboard_output.json`의 `related_products` 필드:

```json
{
  "related_products": {
    "items": [
      {
        "product_id": "rel_001",
        "display_name": "제품명",
        "relation_type": "similar | companion | cross_category",
        "relation_reason": "연관 이유 한줄",
        "image_url": "https://...",
        "current_price": "₩289,000",
        "best_net_price": "₩245,000",
        "platform": "Coupang",
        "cta_url": "https://...",
        "tags": ["태그1", "태그2"]
      }
    ]
  }
}
```

| relation_type | 배지 | 탐색 전략 |
|---------------|------|-----------|
| `similar` | 🔗 유사 | 같은 카테고리, 다른 브랜드/라인 |
| `companion` | 🧩 함께쓰면 | 액세서리, 소모품, 번들 |
| `cross_category` | 🔄 다른형태 | 같은 use_case, 다른 제품군 |
