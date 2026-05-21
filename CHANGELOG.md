# Changelog

All notable changes to this extension are documented in this file. Follow [semver](https://semver.org/): bump the manifest `version` and add an entry here on every manifest change.

## [0.2.0] - 2026-05-21

### Added
- Manifest renamed to `beam-slide-deck`; description, icon, and accent updated for the slide-deck domain.
- `contributes.settingsSchema` declaring brand config: `primaryColor`, `accentColor`, `fontFamily`, `logoPath`.
- Designer product pack `product-packs/slide-deck-pack.json` (target: `built-in:design`, kind: `designer`).
- Design Deck Skill `skills/design-deck-skill/SKILL.md` covering brand intake, follow-up policy (max 2 questions), and standalone HTML output structure.
- Updated `RULES.md` and `prompts/extension-context.md` for the slide-deck domain.

### Removed (from manifest)
- Example entity (`example-item`) and example agent (`example-triage`) references. The example files still exist on disk; delete when no longer needed.

## [0.1.0] - YYYY-MM-DD

### Added
- Initial scaffold from `dev-prism-mac` extension-starter template.
- One example entity (`example-item`).
- One example agent (`example-triage`).
