# Beam Extension — CLAUDE.md (Drop-in for an External Extension Repo)

Drop this file into your `beam-ext-<slug>` repo as `CLAUDE.md`. It is a hackathon-shaped slice of the host-app rules — the bits that apply when you're building an extension in your own repo. The canonical rules live in `dev-prism-mac/CLAUDE.md`; if anything here conflicts, the canonical doc wins.

## What you are building

A Beam extension: a `beam-extension.json` manifest, entities, agents, prompts, optional voice agents, optional Designer / Smart Tables product packs, optional learning loops, optional integrations. The host app installs your repo via the catalog; you don't touch `dev-prism-mac`.

Recommended repo shape: `beam-extension.json` is the index; entity schemas live in `entities/<entity-id>.json`; optional top-level view schemas live in `views/<view-id>.json`; durable agent instructions live in `agents/<agent-id>/instructions.md`; reusable procedures live in root `skills/<skill-id>/SKILL.md`; voice runtime declarations live in `voice-agents/<voice-agent-id>.json`; Designer / Smart Tables variants live in `product-packs/<pack-id>.json`; shared copilot context lives in `prompts/`.

## Non-negotiables

1. **Manifest discipline.** `id` is kebab-case and globally unique. For the hackathon, repos live under `beam-ai-team/beam-ext-<slug>` (private, tagged `hackathon`) and the manifest `id` is `beam-<slug>` — **the repo has a `-ext-` infix, the manifest id does not**. `version` follows semver. Every manifest change ⇒ version bump + changelog entry.
2. **Substrate-first.** Before adding a custom crawler, worker, or adapter inside the extension, check whether a host-app substrate already owns the job:
   - **Knowledge Base** — files, repos, documents, OCR.
   - **Agent Channels** — Slack, Teams, Telegram, email.
   - **Smart Tables** — tabular evidence.
   - **Live-call / voice pipeline** — transcripts.
   - **Shared outreach / workflow runtime** — delivery.
   If the substrate can't do it, document the gap in `RULES.md` under `## Future Integrations` rather than building a sidecar.
3. **`modelTier` only.** Declare `economy` / `fast` / `balanced` / `powerful`. Never a hardcoded provider model id.
4. **Visible agent budget.** 5–7 stable user jobs is the sweet spot. If you're at 10+, consolidate via modes, entity `pageActions`, workflows, or shared skill prompts.
5. **`integrations[].brand`** must exist in the host app's `BRAND_META`. Aspirational brands go in `RULES.md` under `## Future Integrations`.
6. **`instructionsFile`** for long agent prompts. Use `agents/<agent-id>/instructions.md`; don't inline 50-line prompts in the manifest.
7. **No secrets in the repo.** API keys live in the host workspace, not in your extension.

## Entity design

- 3–5 entities maximum.
- Every entity has at minimum: one `required: true` field, `views: ['list', 'detail']`, and a `pagePrompt`.
- Put durable entity schemas in `entities/<entity-id>.json` and reference them from `contributes.entities`.
- Declare `subtitle`, `defaultView`, `defaultSort`, `pageActions` where useful.
- Status options are localized strings; do not rephrase them across releases (downstream rules may match on them).

## Agent design

- One user job per agent. Process/Rules structure works well.
- Give each durable agent its own folder under `agents/`.
- Scope tools per agent. Don't grant `ext_*_write_data` to a read-only summarizer.
- For scheduled scanners, default to `economy` or `fast` tier. Reserve `balanced`/`powerful` for genuine reasoning work.
- If the agent predicts something we can later validate against an outcome, declare a `learningLoops` entry (see `extension-learning-loops.md` in the host app).

## Skill design

- Skills are reusable procedures. Put them at root `skills/<skill-id>/SKILL.md`, not under one agent.
- Agents can reference skills from their instructions, but the reusable method belongs in the skill file.
- If a procedure is not reusable, keep it in the agent instructions instead of creating a thin skill.

## Runtime variants

- Voice agents are not views. Declare them under `contributes.voiceAgents`, store them in `voice-agents/*.json`, and point `targetAgentId` at one of your declared extension agents.
- Presentation/deck variants are product packs for the Design Agent: `targetAgentId: "built-in:design"`, `kind: "designer"`.
- Smart Tables variants are product packs for Smart Tables: `targetAgentId: "built-in:smart-tables"`, `kind: "smart_tables"`.
- Use `views/*.json` only for custom top-level routes. Entity list/detail/card/kanban tabs belong on `entities/*.json`.

## UI / copy quality

- Use product language, not implementation language. "Item", not "row"; "Send", not "dispatch payload".
- Empty states must answer "what should the user do next?"
- Status colors are owned by the host app — don't ship custom color tokens.

## Validation

Your repo CI must:
- Validate the manifest (`pnpm run validate:extension`).
- Validate `learningLoops` shape if declared (entity ids resolve, `outcomeField` exists as a `select`, `rulesStore` + `seedKnowledge` titles align).
- Fail PRs that change the manifest without bumping `version`.

The starter template includes `.github/workflows/validate.yml` — keep it green.

## Commit hygiene

- Stage explicitly (no `git add .`).
- One logical change per commit.
- Use rebase for conflicts; never force-push to shared branches.

## When in doubt

- Read the canonical guide: [`docs/reference/extension-development-guide.md`](https://github.com/beam-ai-team/dev-prism-mac/blob/main/docs/reference/extension-development-guide.md).
- Read a reference repo: `beam-ai-team/beam-ext-smb-crm`, `beam-ai-team/beam-ext-sdr`, `beam-ai-team/beam-ext-hr-suite`.
- Ask in `#prism-hackathon-help` (the Slack expert bot lives there).
