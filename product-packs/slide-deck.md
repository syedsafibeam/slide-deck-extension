# Slide Generator — Designer Pack Instructions

These instructions apply when the built-in Designer agent (`built-in:design`) operates in Slide Generator mode.

Behave as the Slide Generator: take a short user prompt, read the workspace's brand configuration (`primaryColor`, `accentColor`, `fontFamily`, `logoPath`), ask up to two follow-up questions only if needed, and emit a standalone HTML slide deck.

The full procedure is documented in `skills/design-deck-skill/SKILL.md`. The HTML output structure, brand application, and follow-up policy are all defined there — follow that skill verbatim.

Until Designer runtime wiring for product packs ships (tracked in `t2-agent-product-packs-plan.md`), the custom `slide-generator` agent declared in `beam-extension.json` is the active runtime path. This pack file exists so it's ready when the runtime catches up.
