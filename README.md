# Extension Starter Template

Drop-in scaffold for a Beam extension repo. Clone this folder into a new GitHub repo, rename the placeholders, and you have a valid `beam-extension.json` you can install in Beam Prism.

## What's in here

```
extension-starter/
  beam-extension.json              # manifest (rename id, fields, etc.)
  RULES.md                          # domain rules referenced by manifest.devRules
  CHANGELOG.md                      # semver history (bump on every manifest change)
  entities/
    example-item.json                # entity schema referenced by contributes.entities
  agents/
    example-triage/
      instructions.md                # long agent instructions (referenced by instructionsFile)
  skills/
    triage-items/
      SKILL.md                       # reusable procedure an agent or copilot can follow
  voice-agents/                      # optional voice runtime variants for declared agents
  product-packs/                     # optional Designer / Smart Tables variants
  recipes/                           # optional product-pack recipes
  presets/                           # optional product-pack presets
  prompts/
    extension-context.md            # shared copilot context (referenced by contributes.prompts)
  .github/workflows/validate.yml    # CI validation (validate:extension)
  extension-CLAUDE.md               # drop-in CLAUDE.md for your new repo
  extension-AGENTS.md               # drop-in AGENTS.md for your new repo
```

The manifest is the index, not the behavior dump. Keep entity schemas in `entities/<entity-id>.json`; keep durable agent instructions in `agents/<agent-id>/instructions.md`; keep reusable procedures in root `skills/<skill-id>/SKILL.md`; keep voice runtime declarations in `voice-agents/<voice-agent-id>.json`; keep Designer / Smart Tables variants in `product-packs/<pack-id>.json`; keep shared copilot context in `prompts/`.

## Hackathon naming convention

| Thing | Format | Example |
|---|---|---|
| GitHub repo | `beam-ai-team/beam-ext-<slug>` | `beam-ai-team/beam-ext-acme-crm` |
| Manifest `id` | `beam-<slug>` (no `-ext-` prefix) | `beam-acme-crm` |
| Tool names (auto-derived) | `ext_<manifest-id>_*` | `ext_beam-acme-crm_query_data` |
| Visibility | Private | — |
| GitHub topic | `hackathon` (add via repo settings or `gh repo edit --add-topic hackathon`) | — |

`<slug>` rules:
- Kebab-case, lowercase, ASCII only.
- Globally unique across `beam-ai-team` extensions — check `beam-ai-team/beam-ext-*` for collisions before picking.
- Reflects the user-facing extension (e.g. `acme-crm`, `weekly-planner`, `pe-screening`), not your team name.
- Avoid `-ext-` inside the slug; the prefix is repo-only.

The repo name and manifest id intentionally differ — repo = `beam-ext-<slug>` for discoverability, manifest id = `beam-<slug>` so tool names stay readable.

## Bootstrap (~5 min)

```bash
# 1. Create your repo under beam-ai-team (platform pre-grants repo-create permission
#    to each hackathon team; if you hit a 403, ping #prism-hackathon-help).
gh repo create beam-ai-team/beam-ext-<your-slug> --private --clone
cd beam-ext-<your-slug>
gh repo edit --add-topic hackathon

# 2. Copy this template into the new repo
cp -r <path-to-dev-prism-mac>/docs/hackathon/templates/extension-starter/. .
mv extension-CLAUDE.md CLAUDE.md
mv extension-AGENTS.md AGENTS.md

# 3. Rename placeholders (sed is fine; do this once at the start)
SLUG=<your-slug>   # e.g. acme-crm
sed -i '' "s/beam-example/beam-${SLUG}/g" beam-extension.json entities/*.json agents/*/*.md skills/*/SKILL.md prompts/*.md
sed -i '' "s/Example Extension/<Your Name> Extension/g" beam-extension.json README.md
#   Also update: description, author, icon, primaryColor in beam-extension.json
#   And: the example entity in entities/example-item.json plus the example agent to fit your domain

# 4. Commit and push
git add -A
git commit -m "Initial extension scaffold"
git push -u origin main
```

> **Platform alternative**: if your team can't create repos under `beam-ai-team`, ask in `#prism-hackathon-help` and a platform engineer will pre-create the repo with the template applied. You'll then be added as a collaborator with admin rights on your repo only.

## Validate locally

```bash
# In your beam-ext-* repo:
pnpm install
pnpm run validate:extension
```

## Install in Beam Prism

1. Open Beam Prism → Extension Studio.
2. Add Extension → from GitHub repo → point at your new repo URL.
3. The host app pulls `beam-extension.json`, validates it, and loads your entities/agents.

## What to change first

In order:

1. **`beam-extension.json`** — `id` (must be `beam-<slug>`, **not** `beam-ext-<slug>`), `name`, `description`, `author`, `primaryColor`, and the referenced entity/agent entries. Bump `version` on every change.
2. **`RULES.md`** — the domain rules an LLM operating inside your extension should know. Drop or rewrite the example sections.
3. **`entities/example-item.json`** — the example entity schema. Rename the file and id to match your domain.
4. **`prompts/extension-context.md`** — short shared context loaded into copilot when working in your extension.
5. **`agents/example-triage/instructions.md`** — long agent instructions. Process/Rules format works well.
6. **`skills/triage-items/SKILL.md`** — reusable triage procedure. Keep skills at repo root so more than one agent or copilot flow can reuse them.

After your first entity + agent is wired:

7. Add more entities (3–5 max — see `extension-CLAUDE.md`).
8. Add more agents (5–7 max, stable user jobs).
9. Declare `learningLoops` if any agent makes predictions you can validate against outcomes.
10. Declare `integrations[]` (must reference `BRAND_META` brands — aspirational ones go in `RULES.md`).

## Voice, Tables, and Presentations

These are agent/runtime contributions, not ordinary extension views:

- Voice: add `contributes.voiceAgents: ["voice-agents/<voice-agent-id>.json"]`. Each voice agent must set `targetAgentId` to one of your declared `contributes.agents`.
- Presentations/decks: add a product pack with `targetAgentId: "built-in:design"` and `kind: "designer"`.
- Smart Tables/evidence matrices: add a product pack with `targetAgentId: "built-in:smart-tables"` and `kind: "smart_tables"`.

Use `views/*.json` only for custom top-level routes like setup/onboarding dashboards. Generated entity tabs belong in `entities/*.json` via `views: ["list", "detail", "kanban:status"]`.

## Reference

- [`docs/reference/extension-development-guide.md`](../../../reference/extension-development-guide.md) — full developer guide.
- [`docs/reference/extension-learning-loops.md`](../../../reference/extension-learning-loops.md) — learning loops.
- [`docs/hackathon/track-extension-builder.md`](../../track-extension-builder.md) — track-specific hackathon doc.
- Reference repos: `beam-ai-team/beam-ext-sdr`, `beam-ai-team/beam-ext-smb-crm`, `beam-ai-team/beam-ext-hr-suite`.
