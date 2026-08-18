# Le Grand Taxi — Design & Copy Standard

This is the binding standard for this site. Every agent working on this repo must read this file first and follow it. It exists because the client repeatedly caught generic/"AI-looking" patterns creeping back in across multiple build passes — this file is the fix: a persistent rule set instead of re-explaining it every time.

---

## Calibration (Taste dials)

- **DESIGN_VARIANCE: 5** — asymmetric editorial layouts are the house style (see the hero), not centered/safe grids. Lean into intentional imbalance.
- **MOTION_INTENSITY: 4** — considered hover/focus polish (lift, glow, reveal-on-hover), never scroll-jacking or heavy parallax. Motion should feel premium and restrained, not showy.
- **VISUAL_DENSITY: 3** — generous spacing, one clear focal point per section. Never dashboard-dense.

## The anti-AI-generic checklist (binding, check every pass)

**Never use:**
- Numbered lists (`1.` `2.` `3.`, or a big `01` `02` `03` index treatment) or bullet points, anywhere, for anything — including "steps," "reasons," or "features." This is the single most-repeated client complaint across every round. Use prose, distinct sections, or typographic hierarchy instead — never a visible list marker.
- Pill-shaped / rounded "tab" or "badge" containers as the default way to present short trust phrases, tags, or categories ("floater tabs"). If something needs to stand out, make **color and typography** do that work — a bold colored word, a colored heading, a colored rule — not a container shape.
- Generic checkmark-circle icon bullets (● ✓ in a colored circle next to each point) — this is exactly the staxi.nl-style pattern the client flagged as generic. Prose with a strong lead sentence does this job better.
- Filler/padding sentences added just to "sound more content-rich." Every sentence must carry either real information or a real search term. Short sentences, always.
- Em-dashes (—, U+2014) or ellipses (…, U+2026) anywhere in copy. A plain en-dash (–, U+2013) is fine in price displays ("€ 39,–") only.
- Superfluous headings — a heading must earn its place by adding real scannability value, or it gets cut.
- A second accent hue. Red/brass + the warm-dark surface ramp only, ever.
- Box-shadow, or a colored border on a card (depth comes from the layered surface ramp, never a shadow or stroke).

**Always:**
- Ground new copy in specific, concrete promises rather than vague reassurance — e.g. not "we keep you updated," but "je krijgt een bericht van Michell circa 15 minuten van tevoren." Specificity reads as trustworthy; vagueness reads as generic.
- Reuse the existing layered dark-surface system, Bodoni Moda display type, and red/brass palette already established in `index.html` and `info.html` — do not introduce a new visual language without being explicitly asked to.
- Test every change in an actual browser (desktop ~1280px and mobile 375px) before calling it done.

## Content standard drawn from competitor research (use the substance, never their visual style)

These are real, valuable pieces of information Michell asked to have reflected on the site — add the *information*, written in our own short-sentence, no-list, color-not-pills style, never copying a competitor's bullet/numbered/icon-circle presentation:

- **Delay handling, no extra cost**: if a flight is delayed, the flight is tracked and the pickup time is adjusted accordingly, at no extra charge. This should live near the existing flight-tracking content — consolidate, don't duplicate, if something similar already exists.
- **Route/safety**: the driver always takes the fastest safe route.
- **Driver quality**: an experienced, professional driver who knows the roads — tie this to Michell by name where it reads naturally, consistent with the site's personal-service positioning.
- **Cancellation policy**: a concrete "gratis annuleren tot X uur van tevoren" line (see the specific instruction already sent to the info.html work for the exact wording/number).

Do not add reviews/testimonials — Michell only has Uber/Bolt platform reviews right now, not ones specific to Le Grand Taxi. Revisit once real reviews exist.
