<p align="center">
  <img src="https://img.shields.io/badge/version-0.2.0-blue" alt="version">
  <img src="https://img.shields.io/badge/platform-Claude%20Cowork-purple" alt="platform">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="license">
  <img src="https://img.shields.io/badge/lang-한국어-red" alt="language">
</p>

# 🛒 Deal Finder — AI Shopping Agent Plugin

> **"사용자가 생각하지 못한 더 나은 선택지를 찾아주고, 최저가를 넘어 최고 혜택까지 계산해주는 AI 쇼핑 에이전트"**

Deal Finder는 [Claude](https://claude.ai) 기반의 쇼핑 에이전트 플러그인입니다.
단순히 최저가를 비교하는 것을 넘어, **사용자가 모르는 대안을 선제적으로 탐색**하고 **쿠폰·카드할인·캐시백·포인트를 모두 합산한 실질 구매가**를 계산합니다.

---

## ✨ 핵심 기능

### 🔍 Core Engine 1: Proactive Discovery (선제적 대안 탐색)

사용자가 "A를 찾아줘"라고 하면, A만 찾는 것은 검색엔진이 하는 일입니다.
Deal Finder는 **A + 사용자가 모르는 B, C까지 함께 제시**합니다.

```
Layer 1: Direct Match (사용자 요청 그대로)
    "소니 WF-1000XM5 찾아줘" → 해당 제품 검색

Layer 2: Same-Tier Alternatives (동급 경쟁제품)
    → 같은 가격대 + 같은 스펙 티어의 경쟁 제품 자동 탐색
    → "이 제품을 찾으셨다면, 같은 가격대에 이런 선택지도 있습니다"

Layer 3: Value Disruptors (가성비 파괴자)
    → 한 단계 아래 가격대에서 90% 이상의 성능을 내는 제품
    → "예산의 60%로 90%의 만족을 얻을 수 있는 제품이 있습니다"
```

### 💰 Core Engine 2: Best Deal Optimizer (최고 혜택 탐색기)

단순 판매가 비교는 절반의 정보입니다. 실제 소비자가 최종 지불하는 **실질 구매가**를 계산합니다.

```
표시 가격 (Listed Price)
  - 🎫 쿠폰 할인 (Coupon)
  - 💳 카드사 즉시할인 (Card Discount)
  - 🔄 캐시백 (Cashback)
  - 🏷️ 포인트 적립 (Points)
  + 🚚 배송비 (Shipping)
  ────────────────────────────
  = ✅ 실질 구매가 (Net Purchase Price)
```

**실제 출력 예시:**

```
💰 소니 WF-1000XM5

정가          ₩359,900
현재가        ₩259,000  (-28%)
━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎫 쿠폰       -₩15,000  (플랫폼 + 판매자)
💳 카드할인    -₩15,000  (삼성카드)
🔄 캐시백     -₩5,180   (로켓와우 2%)
🏷️ 적립금     -₩2,590   (쿠팡캐시 1%)
🚚 배송비     ₩0        (로켓배송)
━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 실질 구매가  ₩221,230  (정가 대비 -38.5%)

📌 조건: 삼성카드 + 첫 구매쿠폰 + 로켓와우
📎 구매링크: [쿠팡에서 보기](https://...)
```

### ⚡ Token Efficiency Engine v2

브라우저 스크린샷 의존도를 극단적으로 줄이는 데이터 수집 전략입니다.

| 수집 수단 | 토큰 비용 | 용도 |
|-----------|-----------|------|
| WebSearch (병렬) | ★☆☆☆☆ | 가격, 브랜드, 리뷰 일괄 수집 |
| WebFetch | ★★☆☆☆ | 리스팅 페이지 텍스트 파싱 |
| JS DOM Scraping | ★★★☆☆ | 구조화 데이터 JSON 추출 |
| Browser Screenshot | ★★★★★ | 최후 수단 (최대 2장) |

**결과:** 기존 대비 **토큰 약 4배 절감** (~80K → ~20K)

---

## 📦 모듈 구성

| 모듈 | 커맨드 | 설명 | 출력 |
|------|--------|------|------|
| **Research Agent** | `/shop-research` | 제품 검색 + 3-Layer 대안 탐색 + 혜택 계산 | `research_output.json` |
| **Moodboard Agent** | `/shop-moodboard` | Pinterest 스타일 비주얼 카드 보드 | `moodboard.html` |
| **Price Graph Agent** | `/shop-price` | 가격 추이 차트 + 매수/대기 판단 | `price-graph.html` |
| **Comparison Agent** | `/shop-compare` | 스펙/가격/리뷰 나란히 비교표 | `comparison.html` |

---

## 🏗️ 아키텍처

### Pipeline-as-JSON

에이전트 간 JSON 파일을 통해 상태를 전달하는 파이프라인 구조입니다.

```
User Request
    │
    ▼
┌─────────────┐    user_intent.json    ┌──────────────────────────────────┐
│  Orchestrator│ ────────────────────▶  │  Research Agent                  │
│  (Intent     │                        │  ┌────────────────────────────┐  │
│   Parser)    │                        │  │ 1. Direct Match            │  │
└─────────────┘                        │  │ 2. Proactive Discovery     │  │
                                        │  │    (Layer 1→2→3)           │  │
                                        │  │ 3. Best Deal Optimizer     │  │
                                        │  └────────────────────────────┘  │
                                        └──────────────┬───────────────────┘
                                                       │
                                             research_output.json
                                                       │
                          ┌────────────────────────────┼───────────────────────┐
                          ▼                            ▼                       ▼
                   ┌────────────┐             ┌──────────────┐       ┌──────────────┐
                   │ Moodboard  │             │ Price Graph   │       │ Comparison   │
                   │ Agent      │             │ Agent         │       │ Agent        │
                   └─────┬──────┘             └──────┬───────┘       └──────┬───────┘
                         │                           │                      │
               moodboard.html              price-graph.html         comparison.html
```

### Stage-gate Validation

각 단계의 JSON 출력은 다음 단계로 넘어가기 전 반드시 검증을 통과해야 합니다.

```
Gate 0 (user_intent.json):  ✓ action 유효성  ✓ query 비어있지 않은지
Gate 1 (research_output):   ✓ products ≥ 1   ✓ pricing.url 유효성   ✓ deal_score 범위
Gate 2a (moodboard):        ✓ cards ≥ 1      ✓ cta_url 존재
Gate 2b (price_output):     ✓ data_points ≥ 2 ✓ verdict 유효성
Gate 2c (comparison):       ✓ products ≥ 2   ✓ spec_matrix rows ≥ 3
```

불합격 시 해당 단계를 재실행합니다 (최대 2회).

---

## 🎨 Moodboard — Pinterest 스타일

Research Agent 실행 후 **3개 이상 제품이 발견되면 무드보드를 자동 제안**합니다.

**특징:**
- CSS Masonry 레이아웃 (3열 → 2열 → 1열 반응형)
- 각 카드: 이미지 → 태그 → 브랜드/제품명 → 가격 블록 (정가/현재가/실질가) → Deal Score → 구매 링크
- 연관 상품 섹션: Similar(🔗), Companion(🧩), Cross-category(🔄) 3가지 타입
- 호버 애니메이션, 컬러 코딩된 태그

**연관 상품 유형:**

| Type | 아이콘 | 설명 | 예시 |
|------|--------|------|------|
| Similar | 🔗 | 같은 카테고리, 다른 브랜드 | YSL 로퍼 → Lemaire 로퍼 |
| Companion | 🧩 | 액세서리, 번들 | 이어폰 → 전용 케이스 |
| Cross-category | 🔄 | 같은 용도, 다른 제품군 | 이어폰 → 오버이어 헤드폰 |

---

## 📊 Deal Score 공식

모든 제품에 0~100점의 Deal Score가 부여됩니다.

```
Deal Score = (Quality × 0.30)
           + (Price Value × 0.25)
           + (Reviews × 0.20)
           + (Shipping × 0.10)
           + (Return Policy × 0.10)
           + (Brand Trust × 0.05)
```

| 항목 | 가중치 | 산출 방법 |
|------|--------|-----------|
| Quality | 30% | 스펙 대비 가격 비율 정규화 |
| Price Value | 25% | 최저가 ÷ 현재가 × 100 |
| Reviews | 20% | 평점 × log(리뷰 수) |
| Shipping | 10% | 무료배송 100점, 유료 시 비례 차감 |
| Return Policy | 10% | 30일 무료반품 100점, 14일 70점, 불가 20점 |
| Brand Trust | 5% | 플랫폼 신뢰도 + 브랜드 히스토리 |

---

## 🌍 지원 플랫폼

### 국내 (Korea)

| 플랫폼 | 용도 |
|--------|------|
| Coupang (쿠팡) | 종합 쇼핑, 로켓배송 |
| Naver Shopping | 최저가 비교, 네이버페이 |
| Gmarket / Auction | 종합 쇼핑, 이벤트 |
| 11번가 | 종합 쇼핑, SK 혜택 |
| SSG (신세계몰) | 백화점 브랜드 |
| Danawa (다나와) | 전자제품 가격비교 |
| Enuri (에누리) | 전자제품 가격비교 |
| TMON / WeMakePrice | 타임세일 |

### 글로벌

| 플랫폼 | 용도 |
|--------|------|
| Amazon (US/JP/DE) | 종합 쇼핑, 프라임 |
| eBay | 중고/리퍼, 글로벌 셀러 |
| AliExpress | 초저가 직구 |
| Farfetch / SSENSE | 명품/디자이너 브랜드 |
| MR PORTER / Mytheresa | 프리미엄 패션 |
| iHerb | 건강식품 |

---

## 🚀 사용법

### 설치

1. `deal-finder.plugin` 파일을 다운로드합니다
2. Claude Cowork에서 플러그인을 설치합니다
3. 별도 API 키나 환경변수 설정이 필요 없습니다

### 자연어 트리거

자연어로 요청하면 적절한 스킬이 자동으로 트리거됩니다:

| 입력 예시 | 실행되는 모듈 |
|-----------|---------------|
| "아이패드 에어 최저가 찾아줘" | Research |
| "30만원 이하 무선 이어폰 추천해줘" | Research → (자동 제안) Moodboard |
| "무선 이어폰 무드보드 만들어줘" | Research → Moodboard |
| "이 제품 가격 추이 보여줘 [URL]" | Price Graph |
| "갤럭시 vs 아이폰 비교해줘" | Research → Comparison |
| "추천해줘" | Full Pipeline (모든 모듈) |

### 커맨드 직접 호출

```
/shop-research 노이즈캔슬링 이어폰 30만원 이하
/shop-moodboard 겨울 패딩 50만원 이하
/shop-price https://coupang.com/vp/products/123456
/shop-compare 갤럭시 S25 vs 아이폰 16 vs 픽셀 9
```

---

## 📁 프로젝트 구조

```
deal-finder/
├── .claude-plugin/
│   └── plugin.json          # 플러그인 매니페스트
├── CLAUDE.md                # 마스터 에이전트 아키텍처 (LLM 실행 문서)
├── README.md                # 이 파일
├── commands/
│   ├── shop-research.md     # /shop-research 커맨드 정의
│   ├── shop-moodboard.md    # /shop-moodboard 커맨드 정의
│   ├── shop-price.md        # /shop-price 커맨드 정의
│   └── shop-compare.md      # /shop-compare 커맨드 정의
└── skills/
    ├── research/
    │   ├── SKILL.md          # Research Agent 스킬 정의
    │   ├── references/       # 플랫폼 셀렉터, 점수 공식 등
    │   └── examples/         # 출력 JSON 샘플
    ├── moodboard/
    │   ├── SKILL.md          # Moodboard Agent 스킬 정의
    │   └── references/       # HTML 템플릿, 카드 디자인 룰
    ├── price-graph/
    │   ├── SKILL.md          # Price Graph Agent 스킬 정의
    │   └── references/       # Chart.js 템플릿
    └── comparison/
        ├── SKILL.md          # Comparison Agent 스킬 정의
        └── references/       # 비교표 HTML 템플릿
```

---

## 🔧 기술 스택

| 기술 | 용도 |
|------|------|
| Claude (Anthropic) | LLM 에이전트 기반 |
| WebSearch API | 실시간 가격/리뷰 수집 |
| WebFetch | 상품 페이지 텍스트 파싱 |
| Chart.js | 가격 추이 인터랙티브 차트 |
| CSS Masonry | Pinterest 무드보드 레이아웃 |
| Pretendard 폰트 | 한국어 최적화 타이포그래피 |

---

## 🧭 로드맵

- [x] 4-Module Architecture (Research, Moodboard, Price, Comparison)
- [x] Proactive Discovery 3-Layer
- [x] Best Deal Optimizer (실질 구매가)
- [x] Token Efficiency Engine v2
- [x] Pinterest Masonry Moodboard
- [ ] `references/` 폴더 실제 데이터 채우기 (platform-selectors, deal-score-formula 등)
- [ ] 가격 알림 기능 (Price Alert)
- [ ] 위시리스트 기능
- [ ] 사용자 카드 보유 정보 연동 (개인화된 카드할인 추천)
- [ ] 해외직구 관부가세 자동 계산
- [ ] 커뮤니티 리뷰 분석 (뽐뿌, 클리앙, Reddit)

---

## 📜 라이선스

MIT License

---

## 🙏 기여

이슈와 PR을 환영합니다. 특히 아래 영역에 도움이 필요합니다:

- 플랫폼별 CSS 셀렉터 패턴 (`references/platform-selectors.md`)
- 카테고리별 비교 기준 추가 (`references/discovery-heuristics.md`)
- 새로운 쇼핑 플랫폼 지원 확장

---

<p align="center">
  <b>Deal Finder</b> — 쇼핑은 검색이 아니라 발견이다. 🛍️
  <br>
  Built with ❤️ by <a href="https://github.com/ashmoonori-afk">Gwanghoon</a>
</p>
