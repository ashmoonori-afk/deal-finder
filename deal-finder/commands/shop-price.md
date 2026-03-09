---
description: Track price history and get buy/wait recommendation
argument-hint: [product name or URL]
---

Read the CLAUDE.md at @${CLAUDE_PLUGIN_ROOT}/CLAUDE.md for the full agent architecture.
Then load the price-graph skill at @${CLAUDE_PLUGIN_ROOT}/skills/price-graph/SKILL.md.

The user wants price tracking for: $ARGUMENTS

Execute the Price Graph Agent pipeline:

1. If a URL is provided, scrape the product page directly via Claude in Chrome
2. If a product name is provided, first run a quick research to identify the product, then proceed
3. Collect current price from the product page in real-time
4. Search for historical price data (Danawa price history, Enuri, community posts, CamelCamelCamel for Amazon)
5. Calculate stats: highest, lowest, average, trend direction
6. Apply buy/wait/neutral verdict logic from the skill
7. Predict the next expected sale event
8. Generate an interactive Chart.js price chart as HTML
9. Save `price-graph.html` to workspace and provide the link
10. Present the summary with verdict, stats, and next sale prediction
