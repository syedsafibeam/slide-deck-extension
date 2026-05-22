# Changelog

All notable changes to this extension are documented in this file. Follow [semver](https://semver.org/): bump the manifest `version` and add an entry here on every manifest change.

## [0.7.0] - 2026-05-22

### Added
- `entities/deck.json` gains `openAction: { target: 'agent-deck-panel', pathField: 'outputPath', agentId: 'ext:beam-slide-deck:slide-generator' }`. Clicking any deck row anywhere (list, card, kanban, dashboard) now opens its generated HTML in the Slide Generator agent's right-rail decks panel instead of the entity edit form. Rows without a generated `outputPath` (status: Drafting / Outline Ready / Failed) still fall back to the form so partial records remain editable.
- `slide-generator` agent declares `capabilities: ['decks']`, which surfaces the right-rail Decks tab on its workspace page (same panel previously hardwired to the Designer Agent).
- A dedicated **DeckDashboardView** gallery is registered against `beam-slide-deck:deck`. The Dashboard tab now shows every deck as a card with title, status badge, audience, slide count, and tone — with a New deck button wired to the same chat prompt as `createAction`.

### Requires
- Host app `dev-prism-mac` with (a) `EntityOpenAction` support on `EntitySchema`, (b) `ExtensionAgentDeclaration.capabilities` plumbed through `ExtensionAgentExecutor` into `UnifiedAgent.capabilities`, (c) `openDeckInSidebar` / `openDesignArtifactPathInSidebar` generalized to accept an `agentId`, and (d) `DeckDashboardView` registered in `entity-dashboard-registry`. Older host builds ignore `openAction` (rows open the form) and `capabilities` (the Decks tab won't appear on the Slide Generator agent), and won't have a registered dashboard panel.

## [0.6.0] - 2026-05-22

### Changed
- Deck creation now routes through chat instead of the entity form. The primary `+ New deck` button (and ⌘N) sends the slide-generator prompt to the copilot in a new thread — no form modal in between.
- Replaced the old `pageActions[0]` "Create with AI" overflow item with a top-level `createAction: { mode: 'chat', label: 'New deck', prompt: ... }` on the entity. The prompt is unchanged; only the entry point moved (overflow → primary button).

### Requires
- Host app `dev-prism-mac` with `EntityCreateAction` support on `EntitySchema` (added in the same change). Older host builds will ignore `createAction` and fall back to the form-based flow.

## [0.5.0] - 2026-05-22

### Added
- `dashboard` view to the `deck` entity (counts-by-status overview, rendered by the host app's built-in dashboard view).

### Changed
- `deck.defaultView` flipped from `kanban:status` to `card` so the gallery is what opens first — title, status/tone badges, and audience/slideCount meta are surfaced by `EntityCardView`.
- The existing `pageActions[0]` ("Create with AI") stays as the chat-driven create affordance; it already routes to the `slide-generator` agent and is reused by every view tab.

### Notes
- The host app's entity view kinds are a closed enum (`list | detail | kanban:* | card | dashboard`) — a custom `gallery` kind that embeds `prism-decks-runtime` to render real slide previews would require host-app changes (a card-renderer hook + structured deck data on the entity), not just manifest edits.

## [0.4.1] - 2026-05-21

### Changed
- Agent specialization made explicit: Slide Generator now self-identifies as a specialized presentation-design agent in the manifest description, agent instructions, and copilot context.
- Follow-up policy flipped from "ask if missing" to **"always ask at least one"** — questioning posture is mandatory, cap stays at 2. Priority order: audience → key takeaway / angle → slide count → tone.
- Added "stay in lane" rule: non-deck requests get a polite redirect, not a drift into general-purpose chat.

## [0.4.0] - 2026-05-21

### Adopted `DeckDesignProfile` as the config shape

Config moved from flat `settingsSchema` fields to a versioned JSON file (`DeckDesignProfile`, `version: 1`) stored in the workspace. The form-level `settingsSchema` can't model nested objects (brand, typography, defaults, assets) or file paths beyond `folder`, so the rich config lives on disk and `settingsSchema` collapses to just the path and a fallback preset selector.

### Added
- `presets/beam-default.json` — starter `DeckDesignProfile` (light, 16:9, includes About slide).
- `presets/beam-pitch-deck.json` — pitch-deck preset (dark, 16:9, includes About + Live Demo, locked title/about/live-demo layouts).
- `deck.presetUsed` field (text) — captures which preset/profile was active per deck.

### Changed
- `settingsSchema` collapsed to two fields: `profilePath` (default `.beam/deck-profile.json`) and `defaultPreset` (select, default `beam-default`).
- `deck.brandSnapshot` renamed to `deck.profileSnapshot` (richtext) — now snapshots the full `DeckDesignProfile`, not just brand fields.
- `agents/slide-generator/instructions.md` — profile-resolution process: read `profilePath` → fall back to `defaultPreset` → optionally load `profile.agentGuidelinesPath`.
- `skills/design-deck-skill/SKILL.md` — inputs now reference the `DeckDesignProfile` interface; HTML output structure uses `brand.background`, `brand.foreground`, `typography.headingFont`/`bodyFont`/`minFontPx`, theme-aware logo selection, and `defaults.includeAboutSlide` / `defaults.includeLiveDemoSlide` / `defaults.outputAspectRatio`.
- `RULES.md`, `prompts/extension-context.md` updated for the profile-driven model.

### Notes
- `profile.lockedLayouts` is declared but not enforced yet — comes into play in step 2 (regeneration / editing).
- `profile.decksDir` (default `decks`) determines the output location under `workspace.storage`.

## [0.3.0] - 2026-05-21

### Architecture correction

Re-introduced the canonical entity + agent shape after reviewing the upstream `extension-development-guide.md` and `track-extension-builder.md`:

- Product packs validate at manifest level today, but Designer runtime registration is still pending (per `t2-agent-product-packs-plan.md`, 2026-05-20). A pack-only extension won't function at runtime yet.
- Every shippable extension needs at least one entity AND at least one agent per the track's "definition of ship".

### Added
- `entities/deck.json` — `deck` entity (title, prompt, status, audience, slideCount, tone, outputPath, brandSnapshot, notes). Views: list, detail, kanban:status, card.
- `agents/slide-generator/instructions.md` — custom agent (manual trigger, `modelTier: balanced`, tools scoped to deck read/write + KB search + file read). Reads brand, asks up to 2 follow-ups, drafts an outline, generates standalone HTML, writes a `deck` record with the output path under `workspace.storage`.
- `agentBindings` for `deck` (read + write).
- `product-packs/slide-deck.md` — Designer-mode instructions for the pack (used when product-pack runtime wiring ships).

### Changed
- `settingsSchema` field shape corrected to canonical: `key` (not `name`), top-level `title`/`description`.
- `product-packs/slide-deck-pack.json` shape corrected to canonical: `instructionsFile` (not made-up `skill`/`output`/`brand`/`input` fields).
- `RULES.md`, `prompts/extension-context.md` updated for the entity + agent shape.

## [0.2.0] - 2026-05-21

### Added
- Manifest renamed to `beam-slide-deck`; description, icon, and accent updated for the slide-deck domain.
- `contributes.settingsSchema` declaring brand config: `primaryColor`, `accentColor`, `fontFamily`, `logoPath`.
- Designer product pack `product-packs/slide-deck-pack.json` (target: `built-in:design`, kind: `designer`).
- Design Deck Skill `skills/design-deck-skill/SKILL.md` covering brand intake, follow-up policy (max 2 questions), and standalone HTML output structure.
- Updated `RULES.md` and `prompts/extension-context.md` for the slide-deck domain.

### Removed
- Example entity (`entities/example-item.json`), example agent (`agents/example-triage/`), and example skill (`skills/triage-items/`) — scaffold leftovers, not used by the slide-generator product pack.

## [0.1.0] - YYYY-MM-DD

### Added
- Initial scaffold from `dev-prism-mac` extension-starter template.
- One example entity (`example-item`).
- One example agent (`example-triage`).
