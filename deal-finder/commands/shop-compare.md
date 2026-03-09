---
description: Compare 2-4 products side by side with deal scoring
argument-hint: [product A vs product B]
---

Read the CLAUDE.md at @${CLAUDE_PLUGIN_ROOT}/CLAUDE.md for the full agent architecture.
Then load the comparison skill at @${CLAUDE_PLUGIN_ROOT}/skills/comparison/SKILL.md.

The user wants to compare: $ARGUMENTS

Execute the Comparison Agent pipeline:

1. Identify the 2-4 products to compare from user input
2. If only 2 products specified, check if there's a compelling 3rd option via Proactive Discovery (Layer 3 value disruptor) and suggest adding it
3. Collect full specs, real-time prices, reviews, and benefits for each product
4. Run Best Deal Optimizer on each — calculate 실질 구매가 with all benefit stacks
5. Build a spec_matrix with winner marking per row
6. Calculate Deal Score for each product using best_net_price
7. Generate verdict with winner, runner_up, and situation-based recommendations
8. Render an HTML comparison table with winner highlights and color-coded scores
9. Show both 현재가 and 실질 구매가 in the price rows, with 실질가 as the primary comparison basis
10. Save `comparison.html` to workspace and provide the link
