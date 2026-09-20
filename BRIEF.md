# The Climb & the Loop — Master Brief v3.2

> **How to use this file.** This is the complete, self-contained prompt for regenerating or
> improving the artwork `the-climb-and-the-loop.html`. Hand this file — plus the current HTML,
> if it exists — to any AI agent or designer. If the HTML exists, treat this brief as its
> specification and only change what the new request asks for. If it does not exist, rebuild
> it from this brief alone. Do not drop any section silently.

---

## 1 · Seed idea (verbatim, never paraphrase away)

The page visualizes exactly these two thoughts, originally written by the commissioner:

```text
think positive -> even failed, take it as a process of being successful.
think negative -> forever remember the mistake and blaming self or others -> more far away from being successful.
```

**Standing rule, from the very first brief:** *keep the painting's palette, brushwork and mood.*
Any change must preserve (a) the warm-vs-cold polarity between the panels, (b) the procedural
oil-brush look, (c) the quiet, contemplative dusk/dawn temperament.

The commissioner's verbatim lines are always preserved on-page as the small epigraph under each
brass plaque ("as first sketched — …"), whatever artistic rewording sits above them.

---

## 2 · Concept

A single self-contained HTML page, staged as a **dim private-gallery wall** showing an
**oil diptych** in gilt frames with brass plaques:

- **Panel I — “The Process”** (positive thinking): **dawn**. Morning blue sky melting to gold at
  the horizon, a rising sun, a small figure slowly **climbing a path of stones upward** — every
  stone is a marked failure (`×`), every stone a step. Birds flap slowly toward the sun. Warm
  golden motes drift up. Metaphor: failures become the staircase.
- **Panel II — “The Loop”** (negative thinking): **night**. Indigo storm sky, a bowed figure
  kneeling on a cracked slab labelled *the mistake*, encircled by a slowly breathing **vortex of
  blame** labelled *remember → blame self → blame others*, while a small spark labelled *success*
  hangs far away, its dotted trail fading before it arrives. Cold ash falls.
- **The polarity is the meaning:** Panel I is morning, Panel II is midnight — *one mind, two
  hours of the same sky*. Never make both panels the same temperature. A full midday-blue sky on
  Panel I was considered and rejected for this reason (but see §12 backlog).

Title of the work: **“The Climb & the Loop”**. Footer shows paint dabs (the kept palette), the
rule of the brief, and two wax seals (a sun, a spiral).

---

## 3 · Hard technical constraints

1. **One single HTML file**, fully self-contained: no external fonts, images, CSS, JS, CDN
   requests. Must work offline over `file://`.
2. No frameworks, no build step, no `<canvas>` libraries. Vanilla JS + Canvas 2D only.
3. All painting happens in a **logical coordinate space of 600 × 760** per panel; each canvas is
   sized to `element size × min(2, devicePixelRatio)` and transformed accordingly. Repaint on
   debounced resize (180 ms) and on language change. Panel sizing uses the **`padding-top`
   percentage technique** (`126.6667%` = 760/600), never `aspect-ratio`, and overlays use
   explicit `top/left/width/height` instead of `inset` — old iOS Safari must lay out correctly.
4. Deterministic painting: every random element comes from seeded `mulberry32` PRNGs.
   Canonical seeds: Panel I `20131`, Panel II `97731`, vortex `55217`, walker stroke-jitter
   `8123`. Same seeds ⇒ same painting.
5. Animation runs in one **resilient** `requestAnimationFrame` loop drawing only the overlay
   canvases; static layers repaint only on resize/language/reveal. Frame bodies are try/caught
   (one bad frame never kills the loop); a 1.5 s watchdog re-kicks a stalled loop and, after
   three stalls (e.g. iOS Low Power Mode), switches to a plain 33 ms timer driver;
   `visibilitychange` re-kicks on return; rAF has a `setTimeout` fallback.
   **`prefers-reduced-motion` is honored by default with an explicit override**: a localized
   **Motion pill** in the header (key `motion`, persisted in `localStorage['diptych-motion']`,
   states `on`/`off`, default follows the system) can force animation on or off. Static frame =
   vortex angle 0, walker fixed at path progress 0.45, birds frozen. Resize handling must
   ignore iOS's no-op resizes (toolbar show/hide while scrolling).
6. Keep total paint time per full repaint roughly < 150 ms on a mid laptop.

---

## 4 · Brush engine (the “brushwork” of the brief)

- `brush(ctx, x0,y0,x1,y1, w, color, opts)` — tapered stroke stamped as overlapping discs along a
  quadratic curve (control point offset perpendicular by `bend`). Three passes by default:
  shadow (`shade −34`, offset +11% width, α×0.5), body, highlight (`shade +32`, −13%, α×0.38,
  radius ×0.55). `passes:false` for background scumbling. Taper via `sin(πt)^0.45`.
- `dab`, `grain` (thousands of ±5% tone specks), clipped contour-following `band` scumbles for
  hills/field, painted-in vignettes and “varnish” gradients.
- DOM overlays complete the oil effect: canvas-weave `repeating-linear-gradient` + varnish sheen
  (`::before`/`::after` on `.canvas-wrap`), plus a full-page SVG `feTurbulence` grain overlay.
- Frame: wood-gradient `.frame` + gilt `.fillet`; brass `.plaque` with screws; palette-knife
  divider between panels (hidden < 880 px, single column).

**Key geometry (logical 600×760):**
sun `(408,190)` r=62; hill bump `170·exp(−((x−430)/210)²)` subtracted from field top ≈512;
stones (x, y, r): `(112,652,36) (164,600,31) (222,545,27) (284,486,24) (340,424,21) (392,360,18)`;
vortex center `(300,530)`, ellipse squash `0.78`, radius 158→18, 3 arms, 2.35 turns;
success spark `(86,104)`.

**Sky (Panel I, dawn):** linear gradient stops `0 #9cb8d2 → .38 #d3dccd → .68 #f2dda6 → 1 #e9b264`;
sky strokes split into blue set (upper) and warm set (lower). Vignette neutral
`rgba(44,36,32,.3)`, top varnish cool `rgba(38,50,72,.2)`.

---

## 5 · Animation spec

- **Climber**: walks `WALK_PTS = [(112,630) (164,581) (222,528) (284,471) (340,411) (392,349)]`
  (stone tops) in `WALK_MS = 36000`, linear by arc length, then fades out (last 10%) and
  reappears (first 7% fade-in) at the first stone — an endless ascent. Leg swing `sin(phase)`,
  stride phase `= progress·pathLength/26·π`, body bob `|sin(phase)|·1.3`, red sash flutters.
  Drawn each frame with fixed-seed jitter (`8123`) so texture never shimmers.
- **Birds** (3): parameters `{v:11, y0:148, amp:14, ph:0.45, s:1.0, fl:5.2}`,
  `{v:8.5, y0:174, amp:18, ph:0.62, s:0.85, fl:4.4}`, `{v:13, y0:128, amp:11, ph:0.80, s:1.12,
  fl:6.1}`; drift left→right across span 750 px wrapping at −75, sinusoidal altitude, wing flap
  `0.45+0.55·(0.5+0.5·sin(t·fl+…))`.
- **Vortex**: pre-rendered offscreen layer, composited each frame rotated by
  `0.085·sin(t·0.00045)+0.045·sin(t·0.00013+2.1)` around `(300,530)` — a breath, not a wheel.
  Loop labels stay on the static layer (readable).
- **Particles**: 24 warm rising motes on Panel I (some with cross-glints), 34 cold falling ash on
  Panel II — drawn on per-panel fx canvases, cleared and redrawn each frame.
- All animated canvases sit *below* the weave/varnish pseudo-elements so moving figures remain
  “painted into” the canvas.

---

## 6 · Copy (artistic rewording) & i18n

Five languages: `en`, `zhHant` (Traditional, HK), `zhHans`, `ja`, `ko`.
Switcher = brass pills in the header. Persist in `localStorage['diptych-lang']`; **first visit
defaults to Traditional Hong Kong Chinese (`zhHant`)** — the house default, regardless of the
visitor's browser language; a visitor's switched choice persists and wins on later visits.
`<html lang>` is updated. **Any new string must be added in all five languages** — no exceptions.

Canvas-painted labels switch language too: each language carries its own canvas font stack
(en: Georgia italic; zhHant: Songti TC / Noto Serif TC / PMingLiU; zhHans: Songti SC / SimSun;
ja: Hiragino Mincho / Yu Mincho; ko: Nanum Myeongjo / Batang — upright, never italic, for CJK)
and calls `paintAll()` on switch. Current keys per language: `over, title, sub, plate1, plate2,
p1verb, p1main, p2verb, p2a, p2b, epiLabel, epi1, epi2, foot, motion, ariaPos, ariaNeg, cFall, cSuccess,
cFarther, cMistake, cRemember, cSelf, cOthers`. `epi1`/`epi2` are the commissioner's verbatim
English lines in **every** language. Legal lines (the two epigraphs and the footer licence
notice) intentionally stay in English across all languages — like museum labels.

Current English master copy (keep the spirit when revising; restore nothing “lost”):

- Panel I: **Think positive** → *even in falling, you are still walking — every failure a stone,
  every stone a step of becoming successful.*
- Panel II: **Think negative** → *the mistake kept forever, blame circling self and others, →
  success drifting farther, ever farther away.*
- Subtitle: *one mind, two hours of the same sky — the same brush, two directions of the heart.*
- Footer: *as every brief commands — keep the palette · keep the brushwork · keep the mood.*
- In-canvas (en): `each fall, a stone … each stone, a step` · `success` · `farther, ever farther …`
  · `the mistake` · `remember` / `blame self` / `blame others`.
  (CJK variants currently: 「一跌一石,一石一級」「銘記」「自責」「怨人」「愈來愈遠……」 /
  「転びは石に、石は階に」「己を責め」「人を責め」 / 「넘어짐은 돌이, 돌은 디딤돌이」「기억하며」…)

Authoritative source of all five translation sets: the `I18N` object in the existing HTML file.
When regenerating from scratch without it, translate faithfully in register — aphoristic,
museum-plate tone; never machine-literal.

---

## 7 · Provenance & invisible watermark (by CKH) — do not remove

Two independent layers, re-applied on **every** repaint:

1. **Painted stamps.** `wmStamp()` writes `WM_TEXT = 'by CKH'` 7× per static canvas
   (spots `[[.15,.22,−.26],[.79,.15,.24],[.31,.5,.4],[.7,.44,−.18],[.14,.8,.28],[.85,.87,−.32],
   [.52,.66,.12]]` = x, y, rotation), italic 600 24 px Georgia, alternating
   `rgba(255,255,255,.014)` / `rgba(16,12,6,.014)` (≈±3 tone levels: invisible at 100%, pops
   under any levels/curves stretch). `wmStampCenter()` adds one at `(300,532)` on the vortex
   layer at α 0.012. In reveal mode: α 0.85 + soft shadow.
2. **Pixel steganography.** `wmEmbedLSB(canvas)` writes
   `WM_MSG = 'by CKH · MMXXV · 4351'` bit-by-bit into the **LSB of the red channel** at positions
   `px = (Math.imul(i+1, 2654435761) >>> 8) % (w·h)` (fixed scatter key), **five repetitions**,
   decoded by majority vote ≥3-of-5 (`wmExtractLSB`). ±1 tone per pixel — imperceptible; clean
   copies (PNG/screenshot) decode 100%; heavy corruption degrades gracefully. Called from
   `paintAll()` for the two static panels.

**Reveal switches** (keep discreet): triple-click the red sun seal in the footer within 700 ms →
signatures glow for 5 s; or load the page with `#ckh` in the URL (persistent reveal).

**Attribution scaffolding:** `<meta name="author" content="CKH">`, copyright + description metas,
signed source comment (plate id **4351**), `body[data-author="CKH"]`, seal `title="by CKH"`, and
the `.attrib` footer line — rendered in the wall color `#1a130d` (invisible, selectable, turns
black in `@media print`).

**License:** the repository carries a plain-language **View-Only License** (`LICENSE.md`):
viewing, linking, and short attributed quotes are free; copying/republishing, modification,
derivative works, commercial use (incl. model training, NFTs), and watermark-stripping require
prior written permission from CKH. Standard alternates were evaluated and **rejected** —
CC BY-NC-ND 4.0 permits verbatim redistribution, PolyForm Strict 1.0.0 permits private
modification; neither matches “no copy, no modify without permission”. The footer shows a
visible licence line linking to `LICENSE.md`; the hidden `.attrib` line carries it too.

**Anti-lift friction (keep gentle — no devtools traps, no global right-click ban):**
`contextmenu` prevented on `.canvas-wrap` only; canvases `pointer-events:none`,
`user-select:none`, `-webkit-touch-callout:none`, `-webkit-user-drag:none`; a `copy` listener
appends `— “The Climb & the Loop” · artwork & words by CKH` to the clipboard when >40 non-space
characters are copied.

Honesty clause for future editors: client-side protection is friction + provenance, not a vault.
Never replace it with hostile UX (blur-on-devtools, keyloggers, debugger loops).

---

## 8 · Accessibility & behavior notes

- `.canvas-wrap` elements carry `role="img"` and localized `aria-label`s (translated!).
- Language buttons use `aria-pressed`; focus-visible outlines; `prefers-reduced-motion` honored.
- Layout: two-column grid ≥ 880 px, stacked below; panels lift slightly on hover.
- No console noise, no errors when opened via `file://`.

---

## 9 · Verification checklist (run after any change)

```bash
# 1. inline JS parses
awk '/<script>/{f=1;next} /<\/script>/{f=0} f' the-climb-and-the-loop.html > /tmp/check.js \
  && node --check /tmp/check.js

# 2. all i18n target ids exist exactly once
for id in t-over t-title t-sub t-plate1 t-plate2 t-p1verb t-p1main t-p2verb t-p2a t-p2b \
          t-epiLabel t-epi1 t-epiLabel2 t-epi2 t-foot wrapPos wrapNeg; do
  grep -c "id=\"$id\"" the-climb-and-the-loop.html   # expect 1 each
done

# 3. watermark hooks intact
grep -c "wmStamp\|wmEmbedLSB\|wmExtractLSB" the-climb-and-the-loop.html   # expect ≥ 8

# 4. manual: open the file, switch all 5 languages, confirm canvas labels change;
#    triple-click the red seal → signatures appear 5 s; append #ckh → persistent reveal;
#    copy a caption → clipboard ends with the CKH attribution line;
#    footer licence line links to /LICENSE.md and it serves HTTP 200;
#    first visit opens in 繁體中文（香港）by default; a chosen language persists;
#    tap the Motion pill → animation stops/starts and survives a reload;
#    on iPhone Safari (real Safari, not Files/QuickLook preview): painting fills its frame
#    and the climber/birds/vortex/particles move — also verify with Reduce Motion ON,
#    then force motion via the pill.
```

---

## 10 · Change protocol

1. State the request as a delta against §2/§5/§6 — never re-litigate settled decisions (dawn vs
   blue sky, warm/cold polarity, diptych structure) unless the commissioner explicitly reopens them.
2. Keep the single-file, zero-dependency constraint and the 600×760 logical space.
3. New visible string → new key in all five `I18N` sets + canvas font suitability check.
4. Never weaken §7 (watermark/attribution) as a side effect; if pixels get restructured, re-wire
   `wmEmbedLSB`/`wmStamp` call sites.
5. Re-run §9. Ship one file. Say what changed and why, briefly.

---

## 11 · Provenance of this brief

- Work: *“The Climb & the Loop” — a diptych on thinking*
- Commissioner & author: **CKH** · MMXXV · plate id 4351
- Lineage: seed words → oil-diptych visualization → multilingual (en/繁HK/简/日/한) →
  living panel (climber & birds animated) → dawn sky revision → embedded invisible watermark
  “by CKH” + anti-lift friction → view-only license (`LICENSE.md`) → default language 繁體（香港）,
  「正面思維／負面思維」 → this brief (v3.2).

---

## 12 · Backlog — candidate improvements (all optional, none committed)

- **Sky mode toggle** (`?sky=noon|dawn`): full clear-blue variant for comparison; default stays dawn.
- More languages (fr / es / de / pt / ru / th / vi) — follow §6 parity rules.
- Optional ambient audio (soft gallery room tone), off by default, never autoplays.
- Poster/print stylesheet: single-column, attribution visible, frames simplified.
- Zoom-on-hover lens over each panel (non-scaling overlay, respects reduced motion).
- Stronger steganography options (spread into G/B channels, larger repetition) if needed.
- A visible painted “CKH” monogram in a corner of each canvas — *only if the commissioner asks*;
  current design keeps all marks invisible.
