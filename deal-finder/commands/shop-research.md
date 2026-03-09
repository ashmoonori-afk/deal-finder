---
description: Search products, find best deals, and discover better alternatives
argument-hint: [product or query]
---

Read the CLAUDE.md at @${CLAUDE_PLUGIN_ROOT}/CLAUDE.md for the full agent architecture.
Then load the research skill at @${CLAUDE_PLUGIN_ROOT}/skills/research/SKILL.md.

The user wants to find the best deal for: $ARGUMENTS

Execute the Research Agent pipeline:

1. Parse user intent into `user_intent.json` fields (product_category, keywords, budget, requirements, use_case)
2. If budget is not mentioned, search across all price tiers (budget/mid/premium)
3. Run all 3 Discovery Layers — never skip Layer 2 and Layer 3
4. For each product found, run Best Deal Optimizer (coupons, card discounts, cashback, points, bundles)
5. Calculate net_price_scenarios for each product
6. Score and rank by Deal Score using best_net_price
7. Present results with the 실질 구매가 format from CLAUDE.md
8. Include direct purchase links for every product
9. Always end with a discovery_summary highlighting unexpected alternatives
