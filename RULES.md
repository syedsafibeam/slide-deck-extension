# Slide Generator Agent — Rules

Domain rules for any agent or copilot operating inside the Slide Generator Agent extension.

## What this extension is for

The Slide Generator Agent turns a short user prompt into a standalone HTML slide deck, applying the workspace's `DeckDesignProfile` (brand, typography, theme, default slides). Each generated deck is tracked as a `deck` record so the user can re-open, regenerate, or audit it later.

## Domain vocabulary

| Term | Meaning |
|---|---|
| Deck | A `deck` entity record. The `outputPath` field points to the standalone `.html` file under `workspace.storage`. |
| Slide | A `<section class="slide">` inside the generated HTML file. One topic per slide. |
| `DeckDesignProfile` | Versioned JSON config (version: 1) describing brand, typography, defaults, asset paths, and locked layouts. Lives at the workspace path in `settings.profilePath` (default `.beam/deck-profile.json`). |
| Preset | A starter `DeckDesignProfile` shipped under `presets/<name>.json` in this extension. Used when the workspace profile file is missing or invalid. |
| Outline | A numbered list of slide titles + one-line summaries, shown before generating full HTML. |
| Profile snapshot | A JSON copy of the resolved `DeckDesignProfile` at the moment a deck was generated. Stored on the deck record's `profileSnapshot` for audit. |
| Agent guidelines | Tone/voice/do/don't notes at `profile.agentGuidelinesPath`. Loaded as additional context per run. |

## Operating rules

1. **The `DeckDesignProfile` is the source of truth for visuals.** Read it from `settings.profilePath`; never hardcode brand, typography, or theme in the deck.
2. **HTML must be standalone.** Inline all CSS. No `<script src=...>`, no CDN links, no Google Fonts.
3. **Always ask at least one clarifying question** about the presentation before generating — never assume a brief is complete. Cap at 2 questions total. Prioritize: audience → key takeaway / angle → slide count → tone.
4. **Default slide count is 8** when unstated. Hard cap at 20.
5. **Never invent facts.** Quotes, stats, and claims must come from the prompt, the conversation, the knowledge base, or the agent guidelines file.
6. **Show the outline first** unless the user explicitly asks to skip it.
7. **Decks with `status: "Generated"` are immutable.** Don't auto-modify them. Regeneration creates a new deck record.
8. **Snapshot the profile.** The deck's `profileSnapshot` must capture the resolved profile JSON — so regenerations remain reproducible even after the workspace profile changes.

## Edge cases & traps

- If `profile.brand.logoPath` is unset, omit the `<img>` tag entirely — don't render a broken image.
- If the workspace profile file is missing, log which preset was used in the deck record's `notes`, so the user knows visuals weren't drawn from their workspace.
- For dark themes (`profile.defaults.theme === 'dark'`), prefer `profile.assets.logoDark` over `profile.brand.logoPath`.
- Speaker notes go in `<aside class="notes">` (hidden in browser, visible in presentation tools). Don't move them inline.
- If a deck generation fails, set `status: "Failed"` and write the reason into `notes`. Don't leave a deck stuck in `"Drafting"`.
- `profile.lockedLayouts` is declared in the profile but not enforced yet — step 2 (regeneration / editing) will honor it.

## Future integrations

Integrations not yet in the host app's `BRAND_META`. Documenting them here keeps `beam-extension.json` clean.

- **Google Slides / PowerPoint export** — would let users convert generated HTML into native formats. Tracked for step 2.
- **Unsplash / stock-image provider** — would let the agent pull on-topic imagery. Out of scope for step 1.

## Reference

- [Extension developer guide](https://github.com/beam-ai-team/dev-prism-mac/blob/main/docs/reference/extension-development-guide.md)
- The Design Deck Skill: `skills/design-deck-skill/SKILL.md`
- The custom agent: `agents/slide-generator/instructions.md`
- Starter presets: `presets/beam-default.json`, `presets/beam-pitch-deck.json`
- The Designer product pack (future-proofing): `product-packs/slide-deck-pack.json`
