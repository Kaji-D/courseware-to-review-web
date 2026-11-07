---
name: courseware-to-review-web
description: "Turn PPT/PDF/lecture notes/courseware into single-file interactive review webpages (Chinese study notes, step-by-step exam tutorials, clickable matrices, animations) and deploy them. Use when the user asks to 把课件/PPT/讲义整理成复习网页、交互式学习网页、考试型专题页、把知识点做成动画/逐步讲解，或继续整理新资料。"
---

# Courseware → Interactive Review Web

Build a **single-file, offline-capable HTML review page** from courseware. The goal is not to re-layout slides; it is to turn the material into a study path that supports **recall, worked examples, exam writing, and mobile review**.

## When to use

Use this skill when the user wants any of:

- 把 PPT / PDF / 讲义 / 教材截图整理成复习网页
- 把某章/某题型做成交互式学习页、动画讲解页、考试专项页
- 把已有复习网页继续扩充新章节/新资料
- 让知识点“一步步讲清楚为什么这样算、考试怎么写”
- 把单文件网页发布成公网可访问地址

If the user only wants a text summary, do not build a webpage. Ask/confirm the output form when ambiguous.

## Non-negotiables

1. **Single file**: all CSS/JS/SVG inline, zero external fonts/CDN/images. Must open offline.
2. **Content first**: chapter → section → knowledge point → example/exam point. Do not start with visual styling.
3. **Exam-oriented**: every key point answers *what / why / how / example / pitfall / how to write in exam*.
4. **No page-level horizontal scroll**: 320/360/390/768/1024/1440 must fit; wide tables/matrices scroll inside their own container.
5. **Animation must explain** sequence, causality, location, comparison, or state. No decorative animation.
6. **Accessible**: keyboard focus, `aria` on tabs/details, dark mode, reduced motion, no-JS fallback.
7. **Verify math**: dimensions, row/column convention, softmax direction, element-wise exp, rounding. Never trust slide numbers without recomputing.

## Workflow

### 0. Scope (2 min)

Write down:

- range: whole course / one chapter / one exam topic
- output: review notes / topic tutorial / question bank / one-page cheat sheet
- reader: self / classmates / teacher
- acceptance: can recall / can solve / can read on phone / can print
- convention: row vs column vectors; softmax by row vs column; any `1/√d`

If the user gave a previous review page, inspect its structure and reuse its design tokens/components rather than inventing a new style.

### 1. Extract courseware

Use `python-pptx` for text + images + positions; pure-image slides need visual reading.

```python
from pptx import Presentation
prs = Presentation(pptx)
for i, slide in enumerate(prs.slides, 1):
    for j, sh in enumerate(slide.shapes):
        text = getattr(sh, "text", "").strip()
        if text: print(i, j, text)
        if sh.shape_type == 13:
            open(f"slide{i:03d}_{j}.{sh.image.ext}", "wb").write(sh.image.blob)
```

Rules:

- Record slide numbers in the content map.
- Formulas/matrices/diagrams: redraw when possible; otherwise keep image + add text explanation.
- Watch for GBK/old-font garbling; read the image/original instead of guessing.
- One slide may contain 5 stacked images; order by position, not filename.
- Mark uncertain content `[待核对]`.

### 2. Restructure content

Build a table:

`chapter | section | knowledge point | level (掌握/理解/了解) | one-line conclusion | why | how/formula | example | pitfall | source slide`

Every KP must have:

- a one-sentence conclusion
- why (intuition/motivation/connection)
- how (steps/formula)
- at least one example or pitfall
- source page

Turn slide fragments into complete sentences. Use tables for comparisons and flows for processes. For exam calculations, model the full sequence: problem → conventions → steps → exam template → pitfalls → answer.

### 3. Information architecture

Whole-course review:

```text
fixed topbar + 3-level TOC (chapter→section→KP)
chapter header (goals + one-line intro)
  section
    KP cards
    figure/demo
    example/pitfall
  chapter exam-point summary
course summary (comparison tables + one-line thread)
```

Single exam topic:

```text
hero (what/how many points/how many steps)
problem card (conditions + tasks + scores)
conventions (3–5 cards)
4-step interactive walkthrough
exam template (scoring actions + 30s checklist)
pitfalls
full answer (matrices + copyable 7-line answer + print)
```

TOC rules:

- auto-generate from DOM, not hand-written 60 links
- default: open only current chapter/section
- scroll spy: include chapter headers + sections + KPs
- hash links must land below fixed topbar (`scroll-margin-top`)
- mobile: drawer, close after click
- recalc positions after demos/folds change height (`ResizeObserver`)

### 4. UI baseline

Copy the design tokens from `assets/page-template.html` or the method reference. Minimum:

```css
:root{
  --bg:#f3f6fa; --surface:#fff; --surface2:#eef3f8;
  --ink:#172431; --muted:#5c6b7c; --faint:#8795a5;
  --line:#dce5ee; --accent:#315b8a; --accent-soft:#e8f0fa;
  --gold:#a56b12; --gold-soft:#fdf2da;
  --green:#2a7255; --green-soft:#e5f4ec;
  --red:#a94535; --red-soft:#fdece8;
}
```

Rules:

- system Chinese font; monospace for numbers/matrices
- body 15–16px, line-height 1.75–1.8
- content max-width 1020–1180px
- cards: 1px border, 12–18px radius, subtle shadow
- color semantics: blue=structure, gold=exam, green=correct, red=pitfall
- responsive breakpoints: ~1200/1120/940/760/640/480/360
- wide tables: `.tbl-wrap{overflow-x:auto}`; fixed layout with % widths
- matrices: CSS Grid + brackets + `tabular-nums`; allow card-level horizontal scroll
- dark mode: override semantic variables; check contrast on accent buttons

### 5. Animation & interaction

Only add interaction that explains the content.

Minimum useful patterns:

| Pattern | Implementation | Use |
|---|---|---|
| step tabs | `.step-tab/.step-panel` | 4-step tutorial |
| player | `step/total/static/timer` | SVG process |
| cell click | event delegation + highlight | matrix derivation |
| fold | `grid-template-rows:0fr/1fr` | long details |
| scroll reveal | IntersectionObserver | long reading |
| static overview | render final state once | revision/print |
| auto play | interval + stop at end | overview |

State rules:

- stop timer before switching step/tab
- reset step on tab change
- clear timers when leaving a step
- static mode disables timers
- render clears old SVG/DOM first
- provide prev/next/reset/static for every animated demo
- default do not autoplay on load
- honor `prefers-reduced-motion`

### 6. Exam-topic recipe

For a calculation question:

1. Restate problem, points, required outputs.
2. State conventions explicitly: column vs row, `KᵀQ` vs `QKᵀ`, softmax direction, scaling.
3. Show dimension chain.
4. Step 1: build matrices; step 2: score matrix; step 3: exp + column sums + normalize; step 4: output.
5. For each matrix cell, make it clickable and show its source arithmetic.
6. Give exam template, 30-second checklist, common traps.
7. Give complete answer + copyable text + print.
8. Recompute all values in Python; show 4 decimals; note rounding differences.

If the slide uses a convention different from the common `softmax(QKᵀ)V`, say so explicitly. Do not mix conventions.

### 7. QA gates

Run these before delivering:

1. **Structure**: parse HTML; no duplicate IDs; no dangling anchors; no `.fig` inside `.case-h/.box-h/.tbl-title/<p>`; no `<sub>/<sup>` inside SVG `<text>`.
2. **Console/network**: no pageerror, no console.error, no 404, no external requests.
3. **Widths**: 320/360/390/768/1024/1440; `scrollWidth == clientWidth`; wide elements scroll locally.
4. **Visual**: screenshot each figure/matrix; check clipping/overlap; check final column/row; check dark mode.
5. **Interaction**: all tabs/steps/prev/next/play/pause/reset/static/fold/copy/print/links.
6. **Accessibility**: Tab order, focus visible, aria-selected, aria-live, contrast.
7. **No-JS/reduced-motion**: content visible; animations off.
8. **Deployment**: HTTP 200; mobile open; cross-page links; refresh.

Useful checks:

```python
# bad nesting
for c in ['case-h','kp-head','fold-head','box-h','tbl-title']:
    for el in root.xpath(f'//*[contains(@class,"{c}")]'):
        for f in el.xpath('.//*[contains(@class,"fig")]'):
            print("BAD", c, f.get('id'))
```

```js
const d = await page.evaluate(() => ({
  sw: document.documentElement.scrollWidth,
  vw: document.documentElement.clientWidth,
  h: document.documentElement.scrollHeight
}));
```

### 8. Deployment

Local preview:

```powershell
python -m http.server 8787 --bind 127.0.0.1
```

Public temporary URL:

```powershell
winget install --id Cloudflare.cloudflared -e
cloudflared tunnel --url http://127.0.0.1:8787 --no-autoupdate
```

Quick Tunnel is temporary; use Cloudflare Pages/Vercel/GitHub Pages/own server for permanent hosting. Build a `_site` directory with `index.html`, keep relative links working, add favicon/meta.

## Top pitfalls (learned from real builds)

- **Figure inside a flex header**: `.fig` inside `.case-h/.box-h/.tbl-title` gets squeezed. Move it to `.kp-body`.
- **Wrong SVG selected**: `root.querySelector('svg')` may grab the 15×15 header icon. Use `.demo-stage svg`.
- **HTML `<sub>` in SVG `<text>`**: text escapes/scatters. Use Unicode subscripts (`vₜ`) or `<tspan baseline-shift="sub">`.
- **TOC empty**: KPs stored as `.card.kp[id]`, not `h4[id]`.
- **Stale scroll spy**: demos/folds change height; use `ResizeObserver`.
- **TOC grows forever**: only add `.open`; prune to current path.
- **Hash + smooth scroll conflict**: `html{scroll-behavior:auto}`; click smooth; hash instant correction.
- **Final step dimmed**: `done = static || final`, don't dim final.
- **Matrix clipped on mobile**: reduce `--cw` at ≤360; `overflow-x:auto`; answer grid min 290 and single column at ≤360.
- **CSS overrides SVG attributes**: class fill wins over `fill="none"`; use a dedicated class or inline style.
- **Convention mix-up**: `A=KᵀQ` + column softmax vs `A=QKᵀ` + row softmax; state it, never mix.
- **Rounding**: 0.8251 vs 0.8252; compute unrounded, display 4 decimals, explain.
- **External deps**: CDN fonts/KaTeX/images break offline; inline everything.

## Assets in this skill

- `assets/page-template.html` — working single-file skeleton: topbar, KP card, table, formula, 3-step tabs, clickable matrices, exam template, print CSS.
- `assets/content-model.json` — structured content model for exam-topic pages.
- `references/method.md` — full methodology, UI/animation rules, bug patterns, deployment.
- `references/checklist.md` — one-page pre-flight checklist.
- `references/prompts.md` — copy-paste prompts for extraction, IA, UI, animation, exam page, QA.

## Definition of done

- A reader who has not seen the PPT can follow the page and recall the key points.
- Every formula/matrix is correct and dimensioned.
- Every animation can be paused/replayed/statically viewed.
- 320px phone works; no page-level horizontal scroll.
- Dark mode, reduced motion, no-JS fallback work.
- TOC/steps/folds/copy/print/cross-links all work.
- If published: a real URL returns 200 and opens on a phone.
