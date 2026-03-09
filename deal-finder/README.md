# Deal Finder

AI 쇼핑 에이전트 — 사용자가 생각하지 못한 더 나은 선택지를 찾아주고, 최저가를 넘어 최고 혜택까지 계산해주는 플러그인.

## Overview

Deal Finder는 단순 가격 비교를 넘어 3가지 핵심 가치를 제공합니다:

1. **Proactive Discovery** — 사용자가 요청한 제품 외에 동급 경쟁제품과 가성비 파괴자를 선제적으로 탐색
2. **Best Deal Optimizer** — 쿠폰, 카드할인, 캐시백, 포인트, 멤버십을 모두 합산한 실질 구매가 계산
3. **Visual Intelligence** — 무드보드, 가격 추이 차트, 비교표를 HTML로 렌더링

## Commands

| Command | Description | Example |
|---------|-------------|---------|
| `/shop-research` | 제품 검색 + 대안 탐색 + 혜택 계산 | `/shop-research 노이즈캔슬링 이어폰 30만원 이하` |
| `/shop-moodboard` | 비주얼 카드 보드 생성 | `/shop-moodboard 겨울 패딩 50만원 이하` |
| `/shop-price` | 가격 추이 차트 + 매수/대기 판단 | `/shop-price https://coupang.com/...` |
| `/shop-compare` | 제품 비교표 생성 | `/shop-compare 갤럭시 S25 vs 아이폰 16` |

## Skills

| Skill | Trigger Phrases | Output |
|-------|----------------|--------|
| `shop-research` | "찾아줘", "추천해줘", "최저가", "deal" | `research_output.json` + 텍스트 요약 |
| `shop-moodboard` | "무드보드", "한눈에 보여줘", "카드로 정리" | `moodboard.html` |
| `shop-price` | "가격 추이", "매수 타이밍", "지금 사도 돼?" | `price-graph.html` |
| `shop-compare` | "비교해줘", "A vs B", "뭐가 더 나아?" | `comparison.html` |

## Architecture

Pipeline-as-JSON 구조로 에이전트 간 JSON 파일을 통해 상태를 전달합니다:

```
Orchestrator → user_intent.json
  → Research Agent → research_output.json
    → Moodboard Agent → moodboard.html
    → Price Graph Agent → price-graph.html
    → Comparison Agent → comparison.html
```

각 단계에 Stage-gate Validation이 적용되어 불완전한 데이터가 다음 단계로 넘어가지 않습니다.

## Supported Platforms

**국내**: Coupang, Naver Shopping, Gmarket, 11번가, SSG, Danawa, Enuri 등
**글로벌**: Amazon, eBay, AliExpress, iHerb, Rakuten, Walmart, Best Buy 등

## Setup

별도 환경변수나 API 키 없이 동작합니다. Claude in Chrome과 WebSearch를 활용하여 실시간 데이터를 수집합니다.

## Usage

자연어로 요청하면 적절한 스킬이 자동으로 트리거됩니다:

- "아이패드 에어 최저가 찾아줘" → Research
- "무선 이어폰 무드보드 만들어줘" → Research → Moodboard
- "이 제품 가격 추이 보여줘 [URL]" → Price Graph
- "갤럭시 vs 아이폰 비교해줘" → Research → Comparison
- "추천해줘" → Full Pipeline (Research → Comparison → Moodboard)
