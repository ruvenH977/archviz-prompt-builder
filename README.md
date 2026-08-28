# Archviz Prompt Builder

A single-page tool that assembles consistent prompts for turning 3D architectural
renders into photorealistic images. Pick the materials and landscaping present in
the render, adjust any text block, and export the prompt as a `.txt` file.

**Live:** https://ruvenh977.github.io/archviz-prompt-builder/

## How it works

Three columns:

| Column | Purpose |
| --- | --- |
| **Setup** | Checkboxes for the optional sections, furniture brand references, project name, export buttons |
| **Prompt blocks** | One editable text block per active item — edits are live and per-session |
| **Final prompt** | The assembled prompt, with word/character count |

Six sections are always included so every image shares the same look: Overall Style,
Lighting & Atmosphere, Color Temperature, Sky, Outdoor Furniture, Indoor Furniture.
Materials (14 options) and Landscaping (3 options) are opt-in per render.

Output order is fixed:

```
Opening line
OVERALL STYLE
LIGHTING & ATMOSPHERE
COLOR TEMPERATURE
SKY
MATERIALS
LANDSCAPING
OUTDOOR FURNITURE
INDOOR FURNITURE
```

## Editing the default texts

All wording lives in one place — the `SECTIONS` array at the top of the `<script>`
block in `index.html`. Nothing else needs to change when you rewrite a description.

```js
{ id:"concrete", label:"Concrete", text:"fair-faced concrete with fine formwork texture, …" }
```

- `type: "fixed"` — always in the prompt, one text block.
- `type: "group"` — checkbox list; `checked: true` makes an item on by default.
- The Outdoor Furniture entry uses a `template` with a `{BRANDS}` placeholder that the
  brand chips fill in.

Edits made in the browser are session-only by design — the file on disk stays the
source of truth. Use **Reset** on a block, or **Reset all texts to defaults**, to go back.

## Development

No build step and no dependencies. Open `index.html` in a browser, or serve it:

```bash
python3 -m http.server 8000
```

Type is Volte (loaded from marty-designhaus.ch) with Archivo Expanded as fallback, and
Work Sans for body text — both fetched at runtime, so the page needs a network
connection to render in the intended typeface.

## Deployment

GitHub Pages serves `main` / root directly. `.nojekyll` is present so Jekyll does not
process the files. Pushing to `main` republishes the site.
