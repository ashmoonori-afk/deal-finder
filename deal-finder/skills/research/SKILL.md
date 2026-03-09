---
name: shop-research
description: >
  This skill should be used when the user asks to "find a product",
  "search for deals", "recommend something to buy", "최저가 찾아줘",
  "추천해줘", "어디서 사는 게 제일 싸?", "좋은 거 있어?", or needs
  product research, deal hunting, or purchase recommendations across
  shopping platforms. Also triggers on any product name + "찾아줘/알아봐줘".
version: 0.2.0
---

# Shop Research Agent (Token-Efficient v2)

사용자 의도를 파싱하고, 멀티 플랫폼 제품 검색을 수행하며, **사용자가 생각하지 못한 대안까지 선제적으로 탐색**하는 핵심 에이전트. 모든 다운스트림 에이전트(Moodboard, Price Graph, Comparison)의 데이터 소스가 된다.

> **⚡ 토큰 효율 원칙: 브라우저 스크린샷 없이 텍스트 기반으로 데이터를 수집한다.**
> CLAUDE.md의 Token Efficiency Engine 섹션을 반드시 숙지하고 따른다.

## 필수 레퍼런스 참조

작업 시작 전 반드시 아래 파일을 읽고 내용을 반영한다.

- `references/platform-selectors.md` — 플랫폼별 CSS 셀렉터 및 스크래핑 패턴
- `references/deal-score-formula.md` — Deal Score 계산 상세 로직
- `references/benefit-search-patterns.md` — 쿠폰/카드/캐시백 탐색 패턴
- `references/discovery-heuristics.md` — 대안 탐색 전략 및 카테고리별 비교 기준

## 실행 파이프라인 (Token-Efficient v2)

> **핵심 원칙: 브라우저 스크린샷 없이 텍스트만으로 데이터를 수집한다.**

### Step 0: Intent Parsing

사용자 요청에서 아래 필드를 추출하여 `user_intent.json`을 생성한다.

| 필드 | 추출 방법 | 예시 |
|------|-----------|------|
| product_category | 직접 언급 또는 추론 | "남성 로퍼" |
| keywords | 핵심 스펙/기능/스타일 키워드 | ["개더링", "주름", "소프트 레더"] |
| reference_brands | 레퍼런스로 언급된 브랜드 | ["Saint Laurent", "The Row"] |
| budget | 금액 언급 시 추출 | { max: 300000, currency: "KRW" } |
| requirements | 필수 조건 | ["남성용", "가죽"] |
| use_case | 사용 목적/맥락 | "데일리 캐주얼" |

**budget 미언급 시**: 해당 카테고리의 일반적 가격대를 3개 티어(budget/mid/premium)로 나눠 전체 탐색.

### Step 1: Parallel WebSearch Blast (Layer 1 + 2 + 3 동시)

**단일 메시지에서 WebSearch 3~5개를 동시 호출한다.**
각 쿼리는 서로 다른 레이어의 정보를 수집한다.

```
[동시 호출 — 하나의 tool call 블록에 모두 포함]

Query A (Direct Match — Layer 1):
  "{레퍼런스 브랜드} {제품} 가격 구매 사이트 {현재연도}"
  → 사용자가 언급한 정확한 브랜드/제품 가격 + 구매처

Query B (Same-Tier Alternatives — Layer 2):
  "{카테고리} {스타일 키워드} 브랜드 추천 대안 {현재연도}"
  → 같은 가격대/스타일 경쟁 브랜드 발굴

Query C (Value Disruptors — Layer 3):
  "{카테고리} {스타일 키워드} 가성비 10만원대 20만원대"
  → 저렴하지만 스타일 재현도가 높은 대안

Query D (Benefits — Best Deal Optimizer):
  "{주요 플랫폼} 명품 신발 쿠폰 카드할인 캐시백 {현재월}"
  → 혜택 정보 수집

Query E (Community/Review — 선택):
  "{카테고리} {스타일} 후기 추천 Reddit OR 뽐뿌 OR 클리앙"
  → 커뮤니티 평가 높은 숨은 강자 발굴
```

**🚫 절대 하지 않는 것:**
- 이 단계에서 브라우저를 열지 않는다
- WebSearch를 하나씩 순차 호출하지 않는다
- 검색 결과에서 개별 페이지를 방문하지 않는다

### Step 2: Data Enrichment (선택적 보강)

WebSearch 결과만으로 **가격, 브랜드, 구매처 URL**이 충분히 확보되면 이 단계를 **스킵**한다.

**부족할 때만** 아래를 최소한으로 실행:

```
보강 수단 (우선순위 순):

1. WebFetch (최대 2회)
   → 핵심 리스팅 페이지의 텍스트 파싱
   → "이 URL에서 상품명, 가격, 이미지 URL을 추출해줘"
   → 스크린샷 없이 텍스트만 수신

2. JavaScript DOM Scraping (WebFetch 실패 시)
   → 브라우저에서 페이지 1회 로드 → JS로 전체 상품 JSON 추출
   → document.querySelectorAll('.product-card') 패턴 활용
   → 스크린샷 사용 금지

3. Browser Screenshot (위 모든 방법 실패 시 — 최후 수단)
   → 최대 2장까지만 허용
   → "[TOKEN LOG] 스크린샷 사용: {이유}" 로그 기록
```

### Step 3: Image URL Extraction

**이미지 파일을 다운로드하거나 스크린샷으로 캡처하지 않는다.**

```
이미지 URL 확보 방법 (우선순위):

1. WebSearch 결과에 포함된 product image URL 활용
2. WebFetch로 상품 페이지 → og:image 메타태그 추출
3. 쇼핑몰 CDN 패턴으로 URL 구성
   → Farfetch: cdn-images.farfetch-contents.com/XX_XX_1000.jpg
   → Coupang: thumbnail URL (search snippet에서 추출)
   → Naver: shopping.phinf.naver.net/...
4. placeholder 이미지 URL (확보 실패 시)
   → https://placehold.co/400x400/f5f5f5/999?text={brand}

무드보드 HTML에서는 <img src="외부URL">로 직접 참조.
→ 로컬 이미지 저장 = 0
```

### Step 4: Best Deal Optimizer

**각 제품(Direct Match + 대안 포함)에 대해 혜택 스택을 구성한다.**
Step 1의 Query D 결과를 기반으로 혜택을 계산.

**필요 시 추가 WebSearch (최대 2회):**
```
- "{플랫폼} 카드할인 {현재월}" — 5대 카드사 즉시할인
- "명품 직구 할인 코드 {현재월}" — 해외 직구 할인
```

**실질 구매가 시나리오 계산:**
- `최대 혜택`: 모든 쿠폰 + 최적 카드 + 멤버십 적용
- `일반 구매`: 자동 적용 쿠폰만
- `best_net_price`: 최대 혜택 시나리오의 실질 가격

### Step 5: Scoring & Ranking

1. Deal Score 공식에 따라 각 제품 점수 산출 (실질 구매가 기준)
2. Direct Match와 대안을 통합하여 `best_net_price` 기준으로 재정렬
3. `discovery_summary` 작성: "요청하신 X 외에 N개 대안을 찾았습니다. 특히..."

### Step 6: Output

`research_output.json`에 저장 (스키마는 CLAUDE.md 참조). 필수 포함:
- `products[]` — 전체 제품 목록 (Direct Match + 대안)
- `products[].pricing.benefits` — 혜택 스택
- `products[].pricing.net_price_scenarios` — 실질 구매가 시나리오
- `products[].image_url` — 외부 이미지 URL (다운로드 아님)
- `discovery` — 대안 탐색 결과 및 요약
- `search_meta` — 검색 메타데이터

### Step 7: Gate Check

Gate 1 검증 실행. 실패 시 최대 2회 재시도.

### Token Budget Self-Check (자기 검증)

```
파이프라인 완료 후 확인:
✓ WebSearch 호출 횟수: ___회 (목표: 4~6회, 대부분 병렬)
✓ WebFetch 호출 횟수: ___회 (목표: 0~2회)
✓ Browser Screenshot 횟수: ___회 (목표: 0회, 최대 2회)
✓ 수집된 제품 수: ___개
✓ 이미지 URL 확보율: ___%

⚠️ Screenshot > 2회이면 파이프라인 비효율 경고
⚠️ WebSearch를 순차 호출했으면 병렬화 개선 필요
```

## 사용자 응답 형식

리서치 결과를 사용자에게 보고할 때:

1. **한줄 요약**: "N개 제품을 찾았고, 놓치면 아까운 대안 M개도 있습니다"
2. **TOP 추천** (Deal Score 순): 각 제품별 실질 구매가 포맷 표시
3. **대안 섹션**: "💡 이런 선택지도 있습니다" — Layer 2, 3 결과
4. **구매 링크**: 모든 제품에 직접 구매 URL 포함

## Additional Resources

- **`references/platform-selectors.md`** — 플랫폼별 스크래핑 패턴
- **`references/deal-score-formula.md`** — 점수 산출 로직
- **`references/benefit-search-patterns.md`** — 혜택 탐색 전략
- **`references/discovery-heuristics.md`** — 대안 탐색 휴리스틱
- **`examples/research-output-sample.json`** — 출력 JSON 샘플
