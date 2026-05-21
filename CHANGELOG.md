# Changelog

All notable changes to this extension are documented in this file. Follow [semver](https://semver.org/): bump the manifest `version` and add an entry here on every manifest change.

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
