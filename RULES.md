# Slide Deck — Rules

Domain rules for any agent or copilot operating inside the Slide Deck extension.

## What this extension is for

The Slide Deck extension turns a short user prompt into a standalone HTML slide deck, applying the workspace's saved brand (colors, font, logo). It targets users who need a quick, on-brand deck without opening a design tool.

## Domain vocabulary

| Term | Meaning |
|---|---|
| Deck | A single standalone `.html` file containing one or more slides. |
| Slide | A `<section class="slide">` inside the deck file. One topic per slide. |
| Brand | The user's saved configuration: `primaryColor`, `accentColor`, `fontFamily`, `logoPath`. |
| Outline | A numbered list of slide titles + one-line summaries, shown before generating full HTML. |

## Operating rules

1. **Brand config is the source of truth for visuals.** Read it from extension settings; never hardcode colors or fonts inside the deck.
2. **HTML must be standalone.** Inline all CSS. No `<script src=...>`, no CDN links, no Google Fonts.
3. **Cap follow-ups at 2.** If the user gave a one-liner, ask at most two clarifying questions (audience, slide count, or tone), then generate.
4. **Default to 8 slides** when slide count is unstated. Hard cap at 20.
5. **Never invent facts.** Quotes, stats, and claims must come from the prompt, the conversation, or the knowledge base.
6. **Show the outline first** unless the user explicitly asks to skip it.

## Edge cases & traps

- If `logoPath` is unset, omit the `<img>` tag entirely — don't render a broken image.
- If the user pastes a long brief, summarize the topic, audience, and slide count back in one line before generating, so they can correct it.
- Speaker notes go in `<aside class="notes">` (hidden in browser, visible in presentation tools). Don't move them inline.

## Future integrations

Integrations not yet in the host app's `BRAND_META`. Documenting them here keeps `beam-extension.json` clean.

- **Google Slides / PowerPoint export** — would let users convert generated HTML into native formats. Tracked for step 2.
- **Unsplash / stock-image provider** — would let the agent pull on-topic imagery. Out of scope for step 1.

## Reference

- [Extension developer guide](https://github.com/beam-ai-team/dev-prism-mac/blob/main/docs/reference/extension-development-guide.md)
- The Design Deck Skill: `skills/design-deck-skill/SKILL.md`
- The product pack: `product-packs/slide-deck-pack.json`
