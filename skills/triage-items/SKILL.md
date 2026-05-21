# Triage Items Skill

Use this skill when an agent or copilot action needs to classify extension items by priority and update their status consistently.

## Inputs

- A batch of extension records.
- The priority options available on the entity.
- Any domain-specific urgency rules from `RULES.md`.

## Procedure

1. Read the item title, notes, due date, and current priority.
2. Classify urgency using the domain rules.
3. Preserve existing values when the evidence is ambiguous.
4. Write only the fields needed for the classification.
5. Summarize the batch outcome with counts by priority.

## Rules

- Do not invent missing facts.
- Do not mark more than three items high priority in one run unless the domain rules explicitly allow it.
- Keep the user-facing summary short and auditable.
