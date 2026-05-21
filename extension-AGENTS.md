# Beam Extension — AGENTS.md (Drop-in for an External Extension Repo)

Drop this file into your `beam-ext-<slug>` repo as `AGENTS.md`. It is the Codex-native operational contract for an extension repo. `CLAUDE.md` (drop-in alongside this file) is the cross-tool rule map; this file is the actionable Codex contract.

## 0) Scope and precedence

**Scope**: this file applies to the full extension repo. A deeper `AGENTS.md` in a subdirectory overrides it.

**Instruction order** (highest → lowest):
1. Active session instructions (system / developer / user).
2. Nearest in-scope `AGENTS.md`.
3. `CLAUDE.md` and host-app reference docs.

## 1) Manifest contract

- `id`: kebab-case, globally unique. Format: `beam-<slug>`. Hackathon convention: the GitHub repo is `beam-ai-team/beam-ext-<slug>` (with `-ext-` infix), but the manifest `id` is `beam-<slug>` (no `-ext-`). Tool names are auto-derived as `ext_<manifest-id>_*`.
- `version`: semver. Bump on every manifest change; record in `CHANGELOG.md`.
- `description`, `author`, `icon` (Lucide name), optional `primaryColor`, `accentColor`.
- `contributes`: `entities`, `agents`, `prompts`. Optional `views`, `voiceAgents`, `productPacks`, `onboardingAgent`, `learningLoops`, `settingsSchema`.
- `integrations[]`: only brands present in the host app's `BRAND_META`.
- `permissions`: declare the platform capabilities your extension needs (`copilot.tools`, `sidebar.items`, `workspace.storage`, etc.).
- `devRules`: usually `"RULES.md"`. Loaded as agent context for anything operating in this extension.
- Treat `beam-extension.json` as an index. Entity schemas live in `entities/<entity-id>.json`; optional top-level view schemas live in `views/<view-id>.json`; agent behavior lives in `agents/<agent-id>/instructions.md`; reusable procedures live in root `skills/<skill-id>/SKILL.md`; voice runtime declarations live in `voice-agents/<voice-agent-id>.json`; Designer / Smart Tables variants live in `product-packs/<pack-id>.json`; shared copilot context lives in `prompts/`.
- Use relative file references in `contributes.entities`, `contributes.views`, `contributes.voiceAgents`, and `contributes.productPacks` for durable schemas. Inline objects are okay only for tiny drafts.

## 2) Entity contract

- 3–5 entities maximum.
- Every entity declares: one `required: true` field, `views: ['list', 'detail']` minimum, a `pagePrompt`.
- Put durable entity schemas in `entities/<entity-id>.json` and reference them from `beam-extension.json`.
- Use the field types the platform recognizes (`text`, `email`, `select`, `relation`, `richtext`, `number`, `date`, `boolean`).
- `pageActions[]` go in the "…" overflow menu. The primary "+ New <entity>" slot is owned by the platform.

## 3) Agent contract

- One user job per agent. Visible-agent budget: 5–7 per extension.
- Put each durable agent in `agents/<agent-id>/`. Use `instructionsFile: "agents/<agent-id>/instructions.md"` for long instructions. `instructions` inline only for short (<10-line) prompts.
- `modelTier` required (`economy` / `fast` / `balanced` / `powerful`). Never a hardcoded provider model id.
- `tools[]` scoped to what the agent actually needs. Don't grant `ext_<id>_write_data` to a read-only role.
- `trigger`: `schedule` (cron), `event`, or `manual`. Default scheduled scanners to `economy`/`fast`.
- Hidden roles (workers, sync jobs, workflow stages) do not get visible cards. Keep them out of `contributes.agents` if they don't represent a user job.

## 3.1 Skill contract

- Skills are reusable procedures, not agent-private prompt fragments.
- Put skills at root: `skills/<skill-id>/SKILL.md`.
- Agents can reference or invoke root skills from their instructions; keep the actual method in the skill so it can be reused by another agent, copilot action, or workflow.
- If the procedure is one-off and not reusable, keep it inside the agent instructions instead of creating a thin skill.

## 3.2 Runtime contribution contract

- Voice agents are voice-channel variants of declared agents, not ordinary views. Put them in `voice-agents/<voice-agent-id>.json` and list them under `contributes.voiceAgents`.
- Every voice agent must set `targetAgentId` to a real `contributes.agents` id and should declare its transcript handoff (`knowledge_base`, `entity_note`, `smart_table`, or `workflow_state`).
- Presentation/deck variants target `built-in:design` with `kind: "designer"` in `contributes.productPacks`.
- Smart Tables/evidence matrix variants target `built-in:smart-tables` with `kind: "smart_tables"` in `contributes.productPacks`.
- `views/*.json` is only for custom top-level extension routes. Generated entity tabs belong in entity schemas via `views: ["list", "detail", "kanban:status"]`.

## 4) Learning loops (optional but valued)

If an agent makes a prediction you can later validate against an outcome, declare a `learningLoops` entry. The validator requires:
- Entity ids resolve.
- `outcomeField` is a `select` on the target entity.
- `rulesStore` + `seedKnowledge` titles align (the knowledge_base entry's title must match the rules-store key).
- No duplicate loop ids.

See `docs/reference/extension-learning-loops.md` in the host app.

## 5) Substrate-first

Before adding a custom crawler, worker, or adapter inside the extension, check whether a host-app substrate already owns the job. If yes, use it. If no, document the gap in `RULES.md` under `## Future Integrations` and proceed minimally.

Substrates to check first:
- **Knowledge Base** (files / repos / documents / OCR).
- **Agent Channels** (Slack / Teams / Telegram / email).
- **Smart Tables** (tabular evidence).
- **Live-call / voice pipeline** (transcripts).
- **Shared outreach / workflow runtime** (delivery).

## 6) CI and validation

- `pnpm run validate:extension` runs the manifest validator. CI must run it on every PR.
- `learningLoops` shape is validated by the same workflow when present.
- Version bump check: if `beam-extension.json` changed but `version` did not, fail.
- The starter `.github/workflows/validate.yml` covers all of the above.

## 7) Repo hygiene

- One logical change per commit.
- Stage explicitly (no `git add .`).
- Rebase for conflicts; never force-push to shared branches.
- Update `CHANGELOG.md` on every release.

## 8) When operating in this repo as Codex

Before coding:
- Read `CLAUDE.md` (cross-tool map) and this file.
- Read `RULES.md` (domain rules).
- Identify which platform substrate would own the job; only build custom glue if the substrate cannot.

Before submitting:
- Manifest validates.
- Version bumped if manifest changed.
- `CHANGELOG.md` updated.
- All `modelTier` declarations are tier strings, not provider ids.
- No `integrations[].brand` values outside the host app's `BRAND_META`.

## 9) Reference

- Host-app rules: [`dev-prism-mac/CLAUDE.md`](https://github.com/beam-ai-team/dev-prism-mac/blob/main/CLAUDE.md), [`dev-prism-mac/AGENTS.md`](https://github.com/beam-ai-team/dev-prism-mac/blob/main/AGENTS.md).
- Developer guide: [`docs/reference/extension-development-guide.md`](https://github.com/beam-ai-team/dev-prism-mac/blob/main/docs/reference/extension-development-guide.md).
- Learning loops: [`docs/reference/extension-learning-loops.md`](https://github.com/beam-ai-team/dev-prism-mac/blob/main/docs/reference/extension-learning-loops.md).
- Reference repos: `beam-ai-team/beam-ext-smb-crm`, `beam-ai-team/beam-ext-sdr`, `beam-ai-team/beam-ext-hr-suite`.
