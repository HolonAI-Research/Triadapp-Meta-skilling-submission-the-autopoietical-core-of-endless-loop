---
name: style_and_workflow_preferences
description: >-
  Manages user style and workflow preferences for the Recursor‑Agent.
  Embeds user feedback directly into the agent's output style and workflow.
category: meta
version: 0.1.0
author: system
tags:
  - style
  - workflow
  - preferences
---

# Style and Workflow Preferences

## Output Style
- If the user says "stop doing X", "this is too verbose", "why are you explaining", "just give me the answer", or similar, the next output will be trimmed to the minimum necessary and will avoid the mentioned verbosity.
- If the user says "use plain text", "no markdown", or "no bullet points", the output will be returned as plain text without any markdown formatting.
- If the user says "just give me the answer", the response will be limited to the essential answer without additional explanation.

## Workflow Preferences
- If the user corrects the order of steps in a workflow, the next execution of the relevant process will follow the corrected sequence exactly.
- If the user indicates a preference for a specific format (e.g., "use a table", "list the steps numerically"), the agent will adopt that format in subsequent outputs.
- If the user requests a specific tone (e.g., "more formal", "more casual"), the agent will adjust its wording accordingly.

## Reference Files
The following support files are attached to this skill:
- `references/style_preferences.md` — stores the current style and workflow preferences for persistent access.
- `templates/skill_update_template.md` — a starter template for creating or modifying skills.
- `scripts/skill_update.sh` — a helper script that patches a skill given a path and new content.

These files are listed in this skill's documentation for easy discovery.

## Updating the Skill
If the user's preferences change or a new style requirement emerges, edit `references/style_preferences.md` and update this skill's body accordingly. When a patch is applied, remember to update this skill's body to reflect the new preferences so future runs will automatically incorporate them.