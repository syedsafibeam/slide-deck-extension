# Design Deck Skill

Use this skill when generating an HTML slide deck from a user prompt and the extension's brand configuration.

## Inputs

- **Brand config** (from extension settings): `primaryColor`, `accentColor`, `fontFamily`, `logoPath`.
- **User prompt**: a short brief describing the deck topic and goals.
- **Optional context**: knowledge-base documents, attachments, or follow-up answers provided in the conversation.

## Procedure

1. Read the brand config from extension settings. If a field is missing, fall back to: `#003D8F` primary, `#FFB300` accent, `Inter, system-ui, sans-serif` font, no logo.
2. Parse the user prompt for: **topic** (always present), **audience**, **slide count**, **tone**.
3. Ask follow-up questions only for missing fields that materially change the output. Cap at **2 questions**. Defaults when unstated: audience = general business, slide count = 8, tone = professional.
4. Draft an outline first — a numbered list of slide titles with a one-line summary each. Show it to the user before generating the full HTML, unless they explicitly asked to skip the outline.
5. Generate a single standalone HTML file using the structure below.

## Output structure

A single `.html` file. No external CSS or JS dependencies — everything inline.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>{Deck title}</title>
  <style>
    :root {
      --primary: {brand.primaryColor};
      --accent: {brand.accentColor};
      --font: {brand.fontFamily};
    }
    body { margin: 0; font-family: var(--font); color: #111; }
    .slide { min-height: 100vh; padding: 4rem; box-sizing: border-box; page-break-after: always; }
    .slide h1 { color: var(--primary); font-size: 3rem; margin: 0 0 1rem; }
    .slide h2 { color: var(--primary); font-size: 2rem; margin: 0 0 1.5rem; }
    .slide .accent { color: var(--accent); }
    .slide ul { font-size: 1.5rem; line-height: 1.6; }
    .slide aside.notes { display: none; }
    .title-slide { display: flex; flex-direction: column; justify-content: center; }
    .footer { position: fixed; bottom: 1rem; right: 2rem; font-size: 0.8rem; color: #666; }
  </style>
</head>
<body>
  <section class="slide title-slide">
    <h1>{Deck title}</h1>
    <p class="accent">{Subtitle / audience}</p>
    <!-- if logoPath: <img src="{logoPath}" alt="logo" style="height:60px;"> -->
  </section>

  <section class="slide">
    <h2>{Slide title}</h2>
    <ul>
      <li>{point}</li>
    </ul>
    <aside class="notes">{Speaker notes}</aside>
  </section>

  <!-- ...more slides... -->

  <div class="footer">{Deck title}</div>
</body>
</html>
```

## Rules

- **One topic per slide.** Don't pack two ideas into one section.
- **Never invent facts.** Use only what's in the prompt, the conversation, or the linked knowledge base. If a number or quote isn't sourced, omit it.
- **Brand colors are non-negotiable.** `primaryColor` on headings, `accentColor` on highlights only. Don't introduce new colors.
- **Cap follow-ups at 2.** If the user gave a one-liner, ask at most two clarifying questions, then generate.
- **Standalone HTML only.** No `<script src=...>`, no Google Fonts links, no CDN CSS. Everything inline so the file opens offline.
- **Include speaker notes** as `<aside class="notes">` per slide — they're hidden in the browser but visible when the file is opened in a presentation tool.
- **Default slide count is 8** if unstated. Hard cap at 20 unless the user explicitly asks for more.
- **Omit the logo `<img>`** entirely if `logoPath` is empty — don't render a broken image.

## Reference

- Brand config schema: `beam-extension.json` → `contributes.settingsSchema`.
- Invoked by the product pack `product-packs/slide-deck-pack.json` (kind: `designer`, target: `built-in:design`).
