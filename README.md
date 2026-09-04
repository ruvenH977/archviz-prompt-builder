# Archviz Prompt Builder

A single-page tool that assembles consistent prompts for turning 3D architectural
renders into photorealistic images. Pick the materials and landscaping present in
the render, adjust any text block, and export the prompt as a `.txt` file.

**Live:** https://ruvenh977.github.io/archviz-prompt-builder/

## How it works

Three columns:

| Column | Purpose |
| --- | --- |
| **Setup** | Checkboxes for the optional sections, custom materials, furniture brand references, project name, export buttons |
| **Prompt blocks** | One editable text block per active item — edits are live and per-session |
| **Final prompt** | The assembled prompt, with word/character count |

Five sections are always in the prompt so every image shares the same look: Overall
Style, Lighting & Atmosphere, Color Temperature, Sky and Outdoor Furniture. The
middle three take their wording from the selected Scene (see below). Materials and
Landscaping are ticked per render, and Background and Indoor Furniture are optional
sections that start off.

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

## Scene

Four presets — Sunny, Overcast, Sunset, Night — sit under **Scene** in the sidebar,
one selectable at a time. Picking one rewrites the **Lighting & Atmosphere**,
**Color Temperature** and **Sky** blocks; all three keep their place in the prompt
as separate blocks, only the wording changes. Switching presets overwrites any hand
edits in those blocks, and **Reset** on them restores the current preset rather than
the sunny one.

Color Temperature moves with the scene because the daylight presets want unlit,
neutral interiors while Night wants warm interior light switched on — a single fixed
wording would contradict one or the other.

The presets live in the `SCENES` array in `index.html`. Each holds one text per
section id, so putting another section under scene control means adding `scene: true`
to that section and a matching key to every preset.

## Saving and reusing a setup

**Save setup** writes a `.json` file holding everything the page cannot otherwise
remember: which items are checked, every text as currently edited, the custom
materials, the brand chips, the project name and the output options. **Load setup**
reads one back and restores all of it.

Use it to keep a per-project or per-house-type starting point next to the render
files, and to share a setup with someone else. The file is readable and safe to
hand-edit:

```json
{
  "app": "archviz-prompt-builder",
  "version": 3,
  "projectName": "Haus Meier Nord",
  "options": { "headings": true, "bullets": true },
  "brands": ["Dedon", "Minotti"],
  "opening": "Transform this 3D architectural render into …",
  "scene":    "sunny",
  "fixed":    { "sky": "…", "lighting": "…" },
  "optional": { "indoor-furniture": { "on": false, "text": "…" } },
  "groups": { "materials": { "concrete": { "on": true, "text": "…" } } },
  "custom": [{ "name": "Corten Steel", "text": "…", "on": true }]
}
```

Loading starts from the defaults and applies whatever the file contains, so a
partial file works — a JSON holding only `custom` gives you the defaults plus those
materials. Unknown item ids and unknown brand names are ignored. A file without
`"app": "archviz-prompt-builder"` is refused rather than half-applied.

## Adding materials at runtime

**+ Add custom material** at the bottom of the Materials list adds a row with its own
name and description, as many as you like. They sort after the fixed materials in the
prompt, count toward the all/none toggle and the export, and survive *Reset all texts
to defaults*. A custom material with no name contributes its description alone.

Custom materials live in the page session — save a setup, or promote the ones you
keep reaching for into `SECTIONS` below.

## Editing the default texts

All wording lives in one place — the `SECTIONS` array at the top of the `<script>`
block in `index.html`. Nothing else needs to change when you rewrite a description.

```js
{ id:"concrete", label:"Concrete", text:"fair-faced concrete with fine formwork texture, …" }
```

- `type: "fixed"` — always in the prompt, one text block.
- `scene: true` on a fixed section — its text comes from the selected entry in
  `SCENES` instead of its own `text`.
- `type: "optional"` — one text block with its own checkbox; `checked: true` starts it on.
- `type: "group"` — checkbox list; `checked: true` makes an item on by default.
- The Outdoor Furniture entry uses a `template` with a `{BRANDS}` placeholder that the
  brand chips fill in.

Edits made in the browser are session-only by design — the file on disk stays the
source of truth, and **Save setup** captures anything worth keeping. Use **Reset** on a
block, or **Reset all texts to defaults**, to go back.

## Colour codes

Long-press (or right-click) any material whose text carries a hex code to open the
colour picker: wheel plus brightness, 16 presets, a hex field, and RAL / NCS / Keim
entry. Picking a colour rewrites the hex **and** the colour word, so
`in a white (#FCFCFA)` becomes `in a beige (#EADEBD)` — the checkbox in the picker
turns that off for a one-off.

| System | Source |
| --- | --- |
| RAL | ~70 RAL Classic codes across the architectural range, inline in `index.html` |
| NCS | parsed from the notation (`S 2005-Y20R`, `1050-Y`, `0500-N`), so any code works |
| Keim | `keim-colors.js` — 294 colours from the KEIMFARBEN EXCLUSIV and KEIM AVANTGARDE lists |

All three are physical colour systems defined by printed or painted samples, so every
sRGB value here is an approximation — fine for steering a render, not colour-managed.

Keim codes default to EXCLUSIV; prefix with `A` or `AV` for AVANTGARDE (both number
colours 9001–9021), and append `soldalit` for that binder's reading of an AVANTGARDE
monochrome tone. If `keim-colors.js` is missing the field says so rather than failing
silently.

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
