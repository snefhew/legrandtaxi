# Le Grand Taxi — Design Spec v2
### Translating the "Ease Health" layered-surface system to black + red + white

Client brief: current site rated 3/10 ("too ugly, not professional"), target 9/10. Only the `variant-rood` (black + white + oxblood red) direction survives — `variant-nacht` (blue/gold) is retired per client instruction. Gold assets (`icon-gold.png`, `wordmark-gold.png`) are dead weight now; only `icon-white.png` / `wordmark-white.png` are canonical.

This spec adapts the *system* behind the reference (one saturated ink color, layered tint surfaces instead of shadows, disciplined single-weight type, consistent radius, no colored borders) — not its literal cream/green palette.

---

## 1. Color

### 1.1 The single ink color
Ease Health's Forest Ink is one saturated color used confidently across headings, links, icons, and CTAs — but *only* as a foreground (text/icon/fill) color, never as a background wash. We do the same with red, but red needs **two tuned values** because it sits on near-black rather than near-white, where a single mid-red can't serve both "large solid button fill" and "small text on dark" duty without failing one of them:

| Token | Hex | Usage |
|---|---|---|
| `--ink-fill` | `#b3372c` | Oxblood — solid CTA button backgrounds (white text sits on top), icon fills at larger sizes. Kept exactly as in the current codebase; it's restrained and already reads as "expensive," not "warning-light red." |
| `--ink-text` | `#d9695c` | Lightened oxblood — headings-as-links, the eyebrow label, price amounts, active nav state, icon *strokes*. `#c9564a` (current `--accent-text`) measures ~3.6:1 on `#0a0908` — fine for large display numerals but not for anything smaller/thinner. `#d9695c` clears 4.6:1, so it's safe for the smaller UI-weight text this token gets applied to (labels, small links) while `#b3372c`/`#c9564a` remain fine for big bold numerals and buttons. |

**Rule carried over unchanged:** these two values are the *only* saturated color in the system. No second accent hue, ever — not for a "success" green on the quote confirmation, not for a map pin, nothing. If a second signal color is ever needed, use white/gray weight or size, not a new hue.

### 1.2 Canvas → layered dark surfaces
The current codebase has exactly three flat, unrelated dark tones (`--bg:#0a0a0a`, `--bg-2:#000`, `--card:#111`) with no relationship between them — that's why the site reads flat/cheap rather than premium. Ease Health's depth model is a *warming, lightening* ramp as elevation increases (cream → keylime → mint → sage). We mirror that logic in the dark register: each step up in elevation gets very slightly warmer (more red undertone) and lighter, never a shadow.

| Token | Hex | Elevation | Usage |
|---|---|---|---|
| `--canvas` | `#0a0908` | 0 — base page | `<body>`, hero background |
| `--surface-1` | `#141110` | 1 | Footer, alternating section bands, nav bar on scroll |
| `--surface-2` | `#1d1714` | 2 | Cards — the quote form, price-row group container |
| `--surface-3` | `#28201b` | 3 — raised | Input fields on focus, hover state for card rows, active/pressed states |
| `--surface-image` | `#241512` | exclusive | **Reserved only** for the logo-mark panel and any future driver/car photography — see §4. Never used for ordinary cards. This is our answer to Ease Health's Slate Hush: one deliberately distinct, slightly more saturated warm-dark panel that imagery gets and nothing else gets, so photography doesn't have to compete visually with card chrome. |

Old → new mapping for the existing CSS custom properties:
```
--bg      → --canvas     (#0a0a0a → #0a0908, negligible shift, keep as-is)
--bg-2    → --surface-1  (#000    → #141110, was pure black/dead)
--card    → --surface-2  (#111    → #1d1714, was barely distinguishable from --bg)
--input-bg→ --surface-1 at rest, --surface-3 on :focus
```
The important change isn't the exact hexes, it's that `--surface-1/2/3` now form a visible, intentional ramp instead of three arbitrary near-blacks. Squint-test: right now `#0a0a0a`, `#000`, `#111` are indistinguishable at a glance — that's the "flat/cheap" feeling the client is reacting to as much as anything else.

### 1.3 Text
| Token | Hex | Usage |
|---|---|---|
| `--text` | `#f3efe9` | Primary text, headings, button labels on dark. Unchanged — this is already the correct off-white "Cream Paper" analog. |
| `--text-soft` | `#a39d92` | Secondary/dek text, field labels. Warmed slightly from `#9c9992` to sit on the new warm surface ramp instead of a neutral gray that fights it. |
| `--line` | `rgba(243,239,233,.12)` | Hairline dividers — unchanged logic, works fine. |
| `--line-strong` | `rgba(243,239,233,.28)` | Ghost button borders — unchanged logic. |

**Rule carried over unchanged:** no colored (red) borders anywhere. Depth and separation come from the surface ramp and from `--line`, never from an oxblood outline around a card. The current codebase already respects this — keep it that way as new components get added.

---

## 2. Type

### 2.1 The discipline, adapted deliberately
Ease Health's rule is "one display face, one weight (300, light), never mixed." Their reasoning is that a single restrained weight reads as calm/clinical/trustworthy for a wellness brand. **We deliberately invert the weight, not the discipline**: a one-driver premium airport transfer wants to read as confident/precise/automotive, not spa-like. So our rule is **one display face, one weight (900, heavy), never mixed with a second display weight** — same non-negotiable consistency, opposite character, chosen because it fits a black-car-service brand better than a light serif would. This is the one place we intentionally diverge from the reference's literal weight while keeping its *discipline*.

- Display: `-apple-system, "Segoe UI", Arial, sans-serif` stack, rendered at weight **900 only**. (Client's existing `"Arial Black"` first-choice is fine to keep as an explicit fallback for browsers that don't have a 900-weight system font, but lead with the system stack so it renders crisper on most machines.)
- Display sizes — fixed 3-step scale, not open-ended clamp() guessing: **40px / 56px / 74px** (mobile / tablet / desktop). This maps almost exactly onto the site's current `clamp(2.6rem,6.4vw,4.6rem)` (41.6–73.6px) — the existing scale was already close to right, just formalize it as named steps so every future headline uses one of the three, not an arbitrary size.
- Tracking: `-0.02em` at the two larger sizes (was `-0.015em` — push slightly tighter, heavy weight + tight tracking is what makes bold display type read "premium" instead of "shouty").
- Line-height: `0.96` — unchanged, already correct for a heavy display face.

### 2.2 Body / UI
- Sans stack unchanged: `-apple-system, "Segoe UI", Roboto, sans-serif`.
- Weights: 400 (body), 600 (labels, buttons, price amounts) — never anything between. Currently the codebase already mostly does this; formalize it as a rule so it doesn't drift.
- 8-step scale (rem): `0.68 / 0.75 / 0.8 / 0.85 / 0.9 / 1 / 1.05 / 1.5` — covers eyebrow label through hero dek through price-row numerals without inventing a new size per component, which is what's happening today (`.85rem`, `.88rem`, `.9rem`, `1.05rem`, `1.3rem` in the current stylesheet are five near-duplicate sizes that should collapse into the shared scale).

### 2.3 Eyebrow
Unchanged and correct: `ui-monospace,"SF Mono",Consolas,monospace`, uppercase, `.12em` tracking, `--ink-text` color. Ease Health reserves this treatment exclusively for eyebrow labels — the current codebase already does this correctly (only `.hp-eyebrow` and `.hp-section-label` use the mono stack), keep that exclusivity as new sections get added; don't reach for the mono font anywhere else.

---

## 3. Spacing / Shape

| Rule | Current | New |
|---|---|---|
| Card radius | `2px` (`--radius`) | `12px` — the single biggest "looks cheap" signal. 2px reads like an unstyled `<input>` default; Ease Health's 14px (we go 12px to keep a touch more precision/automotive edge) reads as deliberately designed. |
| Badge/pill radius | n/a today | `999px` — for any future status pill/badge, kept categorically separate from the 12px card radius per the reference's rule (two radius values total, system-wide, never a third). |
| Card padding | `1.2rem` (~19px) on `.hp-quote` | `32px` (`2rem`) minimum, `40px` on larger breakpoints — under Ease Health's own ~21px floor today, which is why the quote card feels cramped. |
| Box-shadow | none used | **stays none, everywhere, forever.** Already correct — the current CSS has zero box-shadow rules. Depth must keep coming only from the `--surface-1/2/3` ramp in §1.2, never from a drop-shadow, as new components (testimonial cards, driver-photo panel, etc.) get built. |
| Colored borders | none used | **stays none.** Cards separate from the canvas via fill (surface ramp) and hairline `--line`, never a red or white stroke. |

---

## 4. The hero watermark fix

### Diagnosis (from the actual asset, not a guess)
`assets/icon-white.png` is **337 × 353px**. Viewed directly, it's an ornate calligraphic monogram (a looped "L"/infinity flourish) with a *thin compass-and-star pendant* hanging below it — the pendant is fine-line detail surrounded by a lot of transparent padding at the bottom of the canvas. Two consequences follow directly from that geometry:

1. **The visual weight of the mark sits in the upper ~60% of its own bounding box**, not centered in it — the lower third is mostly empty space around the thin pendant tail. Current CSS vertically centers the *box* (`top:50%; transform:translateY(-50%)`), which means the *ink* actually ends up sitting above true center. That's a structural reason it "doesn't look great" regardless of size/opacity tuning — the positioning math is centering the wrong thing.
2. **Fine calligraphic linework does not survive being blown up to `min(60vw,620px)` at `opacity:0.1`.** At that scale the delicate curls and the compass ticks lose their edges against the near-black canvas and read as a smudge rather than a mark — the current treatment is simultaneously too big (dominates the entire hero, competing directly with the H1 which sits at the same vertical center) and too indistinct (the detail that would justify that size is illegible at 10% opacity).

### Fix — concrete values
```css
.hp-hero-mark{
  position:absolute; z-index:1;
  opacity:0.18;                 /* was 0.1 — smaller + corner-anchored can afford more presence */
  width:min(22vw, 260px);       /* was min(60vw,620px) — ~42% of the old max width */
  right:-3%;
  bottom:-9%;                   /* was top:50% + translateY(-50%) */
  top:auto; transform:none;
  -webkit-mask-image:radial-gradient(circle at 65% 40%, #000 55%, transparent 85%);
          mask-image:radial-gradient(circle at 65% 40%, #000 55%, transparent 85%);
  pointer-events:none;
}
```
Reasoning for each change:
- **Size (620px → 260px max):** at 260px the flourish's main loop stays legible as a shape instead of dissolving into noise; it no longer occupies the full height of the hero.
- **Position (vertical-center → bottom-right corner):** moves the mark out of the H1's reading zone entirely — text lives top-left-to-center, mark lives in the bottom-right corner behind the CTA row, which is exactly the kind of low-stakes real estate a watermark should occupy. This also sidesteps problem (1) above: anchoring from the bottom edge means the empty pendant-tail padding just extends further off the visible edge instead of pulling the ink's visual center off true-center.
- **Opacity (0.1 → 0.18):** now that it's smaller and tucked in a corner rather than spanning the whole hero, it can carry slightly more presence without competing with foreground content — 0.1 was compensating for the mark being oversized; once sized correctly it needs less compensation, but it was actually *under*-opacity relative to its new modest footprint, so nudge up not down.
- **Mask (`radial-gradient`):** feathers the crop so the compass-pendant's empty lower area fades rather than presenting a hard rectangular alpha edge — softer, more "watermark," less "sticker."
- If Michell ever commissions a re-export of the mark: ask for a **tight-cropped version with the pendant trimmed or the padding removed**, which would let a future pass drop the mask entirely and simplify the CSS. Not required for this pass — the mask above compensates for the existing asset.

This same corner-anchor + mask + `--surface-image` background (§1.2) pattern is also what should be used for any future driver/car photography panel, per the reference's "one exclusive imagery treatment" rule.

---

## 5. Component-level notes (see `demo.html` for rendered examples)

- **Nav bar:** keep current flex layout; swap `--radius` on `.hp-btn` to 12px, add `background:var(--surface-1)` + `backdrop-filter:blur(10px)` on scroll (currently only the dev-only switcher bar has this treatment — the real nav should too, it's a good pattern, just not applied yet to `.hp-nav`).
- **Price-row card:** wrap `.hp-prices` in a `--surface-2` panel with 12px radius and 32-40px padding instead of the current borderline-unstyled top-border list — turns four bare rows into an actual card component, consistent with the quote-form card.
- **Primary button:** `--ink-fill` background, `--text` label, 12px radius, `600` weight, add a `filter:brightness(1.08)` hover state (no shadow, per §3).
- **Footer:** promote to `--surface-1` explicitly (already does this via `--bg-2`, just relabel to the new token), keep the hairline top border.

---

## 6. Summary of what NOT to do (carried from the reference, binding)
- Never introduce a second saturated hue. Red + the near-black ramp + off-white only.
- Never use `--ink-fill` / `--ink-text` as a background fill/wash — foreground only.
- Never mix display-type weights — 900 only, everywhere the display face appears.
- Never add box-shadow.
- Never add a colored border to a card.
- Never use a radius outside the two system values (12px components, 999px pills).
- Never reuse the `--surface-image` tone for an ordinary card — it's reserved for the mark/photography.
