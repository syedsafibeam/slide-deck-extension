# Design Deck Skill

Use this skill when generating an HTML slide deck from a user prompt and a `DeckDesignProfile`.

## Inputs

- **`DeckDesignProfile`** — resolved by the calling agent from the workspace profile file or a fallback preset. Shape:

  ```ts
  interface DeckDesignProfile {
    version: 1
    preset: string                    // e.g. 'beam-pitch-deck'
    brand: {
      name: string
      primary: string                 // hex, e.g. '#003D8F'
      accent: string                  // hex
      background: string              // hex (slide bg)
      foreground: string              // hex (body text)
      logoPath?: string               // workspace-relative
    }
    typography: {
      headingFont: string             // CSS font-family stack
      bodyFont: string                // CSS font-family stack
      minFontPx: number               // minimum body font size in px
    }
    defaults: {
      theme: 'light' | 'dark'
      includeAboutSlide: boolean
      includeLiveDemoSlide: boolean
      outputAspectRatio: '16:9'
    }
    agentGuidelinesPath: string       // e.g. '.beam/deck-guidelines.md'
    lockedLayouts?: string[]
    assets?: { logo?: string; logoDark?: string; wordmark?: string }
    decksDir?: string                 // e.g. 'decks' or 'decks/pitch'
  }
  ```

- **User prompt** — short brief: topic, optional audience, optional slide count, optional tone.
- **Optional context** — the file at `profile.agentGuidelinesPath` (tone, voice, do/don't), knowledge-base hits, attachments.

## Procedure

1. Take the `DeckDesignProfile` as input (the agent has already resolved it).
2. Parse the prompt for topic / audience / slide count / tone. Defaults when unstated: audience = "general business", slide count = 8, tone = "professional".
3. The calling agent must surface at least one clarifying question (and at most two) before generation — the questioning posture is mandatory for this skill, never skip it. The 2-question cap is enforced upstream.
4. Draft a numbered outline. Insert an "About" slide near the front if `profile.defaults.includeAboutSlide`. Append a "Live Demo" placeholder slide if `profile.defaults.includeLiveDemoSlide`.
5. Generate one standalone HTML file using the structure below.

## Output structure

A single `.html` file. No external CSS or JS dependencies — everything inline. Resolve all template values from the profile.

```html
<!DOCTYPE html>
<html lang="en" data-theme="{profile.defaults.theme}">
<head>
  <meta charset="utf-8">
  <title>{Deck title}</title>
  <style>
    :root {
      --primary: {profile.brand.primary};
      --accent: {profile.brand.accent};
      --bg: {profile.brand.background};
      --fg: {profile.brand.foreground};
      --heading-font: {profile.typography.headingFont};
      --body-font: {profile.typography.bodyFont};
      --min-font: {profile.typography.minFontPx}px;
    }
    body {
      margin: 0;
      background: var(--bg);
      color: var(--fg);
      font-family: var(--body-font);
      font-size: var(--min-font);
    }
    .slide {
      aspect-ratio: 16 / 9;
      min-height: 100vh;
      padding: 4rem;
      box-sizing: border-box;
      page-break-after: always;
      display: flex;
      flex-direction: column;
    }
    .slide h1, .slide h2 {
      color: var(--primary);
      font-family: var(--heading-font);
      margin: 0 0 1rem;
    }
    .slide h1 { font-size: 3rem; }
    .slide h2 { font-size: 2rem; }
    .slide .accent { color: var(--accent); }
    .slide ul { font-size: 1.5rem; line-height: 1.6; }
    .slide aside.notes { display: none; }
    .title-slide { justify-content: center; }
    .footer { position: fixed; bottom: 1rem; right: 2rem; font-size: 0.8rem; opacity: 0.6; }
  </style>
</head>
<body>
  <section class="slide title-slide">
    <h1>{Deck title}</h1>
    <p class="accent">{Audience / one-line subtitle}</p>
    <!-- if profile.brand.logoPath: <img src="{profile.brand.logoPath}" alt="{profile.brand.name} logo" style="height:60px;"> -->
  </section>

  <!-- if profile.defaults.includeAboutSlide -->
  <section class="slide">
    <h2>About {profile.brand.name}</h2>
    <ul><li>{one-line who-we-are}</li><li>{one-line what-we-do}</li></ul>
    <aside class="notes">{Speaker notes}</aside>
  </section>

  <section class="slide">
    <h2>{Slide title}</h2>
    <ul><li>{point}</li></ul>
    <aside class="notes">{Speaker notes}</aside>
  </section>

  <!-- ...content slides... -->

  <!-- if profile.defaults.includeLiveDemoSlide -->
  <section class="slide">
    <h2>Live Demo</h2>
    <p class="accent">{One-line demo cue for the presenter}</p>
    <aside class="notes">{What to demonstrate}</aside>
  </section>

  <div class="footer">{profile.brand.name} · {Deck title}</div>
</body>
</html>
```

## Rules

- **One topic per slide.** Don't pack two ideas into one section.
- **Never invent facts.** Use only the prompt, the conversation, the knowledge base, and the agent guidelines file.
- **The profile is non-negotiable.** Primary on headings, accent on highlights, background/foreground per theme. Don't introduce new colors.
- **Standalone HTML only.** No `<script src=...>`, no Google Fonts links, no CDN CSS. Everything inline so the file opens offline.
- **Include speaker notes** as `<aside class="notes">` per slide.
- **Default slide count is 8** when unstated. Hard cap at 20.
- **Omit the logo `<img>` entirely** if `profile.brand.logoPath` is unset — don't render a broken image.
- **Pick the right logo asset** for the theme: prefer `profile.assets.logoDark` when `profile.defaults.theme === 'dark'`; otherwise `profile.assets.logo` (fall back to `profile.brand.logoPath`).
- **Use `profile.decksDir` for output location** (default `decks`). The calling agent writes the file.

## Reference

- Profile shape: see "Inputs" above.
- Profile resolution + record-writing: `agents/slide-generator/instructions.md`.
- Starter presets: `presets/beam-default.json`, `presets/beam-pitch-deck.json`.
- Invoked by the product pack `product-packs/slide-deck-pack.json` (kind: `designer`, target: `built-in:design`).
