---
description: Create a visual product moodboard with prices and deal badges
argument-hint: [product category or query]
---

Read the CLAUDE.md at @${CLAUDE_PLUGIN_ROOT}/CLAUDE.md for the full agent architecture.
Then load the moodboard skill at @${CLAUDE_PLUGIN_ROOT}/skills/moodboard/SKILL.md.

The user wants a visual moodboard for: $ARGUMENTS

Execute the full pipeline:

1. First run Research Agent to collect products + alternatives + benefits
2. Then run Moodboard Agent to render an HTML card board
3. Each card must show: product image, name, 정가(strikethrough), 현재가, **실질 구매가(largest/highlighted)**, discount badge, platform badge, deal score bar, benefit summary label, and CTA button with purchase link
4. Include Proactive Discovery products with 💡 or 🔥 badges
5. Save the HTML file to the workspace folder
6. Provide the file link to the user
