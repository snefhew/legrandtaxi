# UX Findings — Price Calculator Popup Redesign

Le Grand Taxi · booking modal research
Scope: destination-input pattern, pickup-address friction, mobile modal mechanics, trust UX. Companion prototype: `modal-prototype.html`.

---

## 0. What exists today (from `index.html`)

The "price calculator" (`.hp-quote`) is a static card permanently inline in the hero section, duplicated identically across both design variants (`#reserveren-1` / `#reserveren-2`). It has:

- 4 plain `<input type="text">` fields: Ophaaladres, Bestemming, Datum (`placeholder="dd-mm-jjjj"`), Tijd (`placeholder="uu:mm"`) — none of these have real date/time semantics, so mobile keyboards don't offer the native date/time picker and nothing validates format.
- A hardcoded result row: `€ 39,–`, always, regardless of input.
- A "Reserveer" button that is a bare `<a href="#">` — no click handler, does nothing.
- Both the nav "Reserveer" link and the hero "Bereken je prijs" link just anchor-scroll down to this same static card (`#reserveren-1`).

This confirms the client's complaint directly: it looks like a form but behaves like a picture of a form. There is no actual calculation, no popup, and destination is free text despite the business being exactly 4 flat rates + 1 fallback formula.

---

## 1. Modal mechanics for mobile (arrivals-hall use case)

The person filling this in is very often standing in Schiphol arrivals, one-handed, possibly with a suitcase, on patchy terminal wifi. Design for that specifically, not for a desktop dialog shrunk down.

**Recommendations, in priority order:**

1. **Use the native `<dialog>` element with `.showModal()`**, not a hand-rolled `position:fixed` div. It gives you, for free and correctly: a real top-layer stacking context, an `::backdrop` pseudo-element, native `Escape`-to-close, and — critically — a built-in focus trap so Tab/Shift+Tab can't leak focus into the page behind it. Hand-rolled modals get focus-trapping wrong constantly; `<dialog>` has excellent support in every modern engine (Chrome/Edge/Safari/Firefox) and needs no library.
2. **Bottom sheet on mobile, centered card on desktop** — one component, two presentations via `@media`. A centered dialog on a phone either forces the user to reach up to the top half of the screen (bad one-handed) or crams content into a tiny box. Anchoring the sheet to the bottom edge, sliding up to ~90vh, keeps the close affordance and primary actions in thumb range. This is the single highest-leverage mobile change relative to the current design.
3. **Sticky action bar inside the sheet.** Put the live price and the "Reserveer" button in a footer bar that stays pinned to the bottom of the modal (`position: sticky; bottom: 0`) rather than at the end of a scrolling form. On a phone with the keyboard open (date/time fields, "overig" destination text), the actionable summary must stay visible without scrolling past the keyboard.
4. **44px minimum touch targets, but target ~48px.** Apple HIG's floor is 44×44pt; Google's Material floor is 48dp. Given the "holding luggage, one thumb" context, aim for 48px on the city chips and the primary CTA — the floor is a minimum, not a target.
5. **16px minimum font-size on all inputs.** iOS Safari auto-zooms the viewport when a focused input's computed font-size is under 16px. On a form meant to be filled in under 15 seconds at an airport, an unexpected zoom-and-recenter is exactly the kind of friction that tanks completion. This applies to the date/time native inputs too.
6. **Don't destroy state on close.** Backdrop click and Escape must close the modal, but the form's values should survive being reopened — someone dismissing the modal to re-read the flat-rate table, then reopening it, should not have to retype. Simplest implementation: never remove the form from the DOM, just toggle the dialog open/closed; the browser retains input values automatically as long as the elements aren't destroyed.
7. **Respect `prefers-reduced-motion`.** `index.html` already has a global rule for this (lines 16–18 of the current file) — reuse that exact pattern for the sheet's open/close transition (fall back to an instant show/hide, no slide/fade).
8. **Keyboard accessibility for the chips.** The destination quick-select must be real, focusable, `Enter`/`Space`-activatable controls — implemented as visually-hidden native `<input type="radio">` + styled `<label>`, not `<div onclick>`. This gives correct screen-reader semantics ("radio button, 1 of 5, Amsterdam, 39 euro") and native keyboard support for free.
9. **Visible focus ring in the brand's own terms.** The style direction forbids colored borders and heavy shadows — so the focus indicator should be a clean off-white outline (e.g. `outline: 2px solid #f3efe9; outline-offset: 2px`), not the browser default blue glow, and not the red accent (red is reserved for CTAs/key numbers per the brief, not focus state).
10. **Trigger consistency.** Both existing CTAs ("Reserveer" in the nav, "Bereken je prijs" in the hero) should open the *same* modal via `showModal()` instead of anchor-scrolling to an inline card. That removes an entire class of "did the page just jump?" disorientation on mobile.

---

## 2. Destination input: quick-select, not freeform — clear recommendation

**Recommendation: replace the "Bestemming" text input with a 5-option segmented/chip control — Amsterdam, Rotterdam, Utrecht, Almere, Overig — each showing its price inline, selectable in one tap.**

Freeform text is the wrong pattern here, for reasons specific to this business model, not generic UX dogma:

- **The destination space isn't open — it's a closed set of 4 flat rates plus one fallback formula.** Free text implies the system needs to interpret arbitrary input (geocode it, guess intent, handle "Adam" vs "Amsterdam" vs "A'dam" vs a typo). But the price for those 4 cities is *already a known constant* — there is nothing to calculate. Making the user type when the business already knows the answer is friction with no corresponding benefit.
- **Chips let the price show before a decision is even made.** Render each chip as `Amsterdam · €39`, `Rotterdam · €69`, etc. The user sees all four fixed prices simultaneously, at a glance, before typing anything — which directly reinforces the hero's own promise ("vaste prijs, geen verrassingen") *inside the booking flow itself*, not just in marketing copy above it.
- **One tap vs. type-and-hope.** A freeform field requires: tap field → keyboard appears → type N characters → (maybe) autocomplete suggestion appears → tap it → keyboard dismisses. A chip requires: tap chip. On a touchscreen at an airport, that's the difference between a 15-second and a 45-second interaction.
- **Chips beat a `<select>` dropdown too.** A native select is two taps (open, then choose) and hides all 4 prices behind a closed control until opened — chips show everything in one visual sweep with one tap to commit. Given there are only 5 options total (4 cities + "overig"), a dropdown's main advantage (compact for long lists) doesn't apply.
- **"Overig" (other) must stay honest, not fake a number.** When "Overig" is selected, reveal a small freeform text field *and* switch the price display to the real fallback rate as a formula — "€2,15/km + €0,40/min — Michell bevestigt de exacte prijs" — rather than computing a fake distance-based number the prototype can't actually verify. Never show a confident-looking € amount the business can't stand behind.
- **Implementation note:** style the chips as real `<input type="radio" name="destination">` + `<label>` pairs (see §1.8) so they're keyboard/screen-reader operable, not just tap-only `<div>`s.

## 2a. Bonus structural finding: a direction toggle removes a whole field

Because nearly every trip is either *Schiphol → city* (arrivals hall, the stated primary use case) or *city → Schiphol* (departure), consider a two-way toggle at the very top of the modal: **"Naar Schiphol" / "Vanaf Schiphol"**. Selecting a direction auto-fills whichever side of the trip is Schiphol ("Schiphol Airport") and relabels the chip group and the remaining freeform field accordingly (chip group becomes "waar kom je vandaan" or "waar ga je heen"; the other field becomes the freeform pickup detail only when needed). This collapses "type Schiphol into a field" entirely for the majority case and is demonstrated in the prototype.

---

## 3. Ophaaladres (pickup address): keep freeform, reduce friction around it

Pickup can legitimately be anywhere, so freeform text remains correct here — but it can still be made much easier:

- **`autocomplete="street-address"` and `autocapitalize="words"` on the input.** This lets iOS/Android offer saved-address autofill (from Contacts/past forms) and keeps the mobile keyboard's auto-capitalization sane for Dutch street names, at zero implementation cost and no third-party API dependency.
- **Deliberately do *not* reach for a Google Places Autocomplete widget.** It adds an external network dependency, an API key/cost, and a loading delay — exactly the wrong trade-off for someone on patchy airport wifi trying to finish a booking in under a minute. The native `autocomplete` attribute is free, offline-capable, and good enough for this use case.
- **Single-line field, not split into street/number/city.** Splitting into multiple fields (like a checkout form) adds taps and validation surface for a field that a human dispatcher (Michell) reads and interprets personally anyway — there's no downstream system parsing it into structured address components. Uber, Bolt, and Dutch taxi apps all use one freeform pickup line; that's the right precedent, not an e-commerce shipping form.
- **Keep the existing placeholder pattern** (`"Straat, huisnummer, plaats"`) — it's already a good, concrete format hint and should carry over unchanged.
- **When the direction toggle (§2a) is set to "Vanaf Schiphol," this field can be replaced by a one-tap Schiphol quick-value** ("Schiphol Airport, Aankomsthal") instead of asking the user to type their own current location, since it's already known context. This is the single biggest friction removal for the specific "standing in arrivals" scenario called out in the brief.

---

## 4. Trust-building UX inside the booking flow (not just marketing copy)

The brand has no stock photography and no testimonials to lean on — the actual trust asset is "one real person, Michell, personally confirms every booking, there is no call center." That fact should surface *procedurally*, inside the form itself, not just as a sentence in the hero:

- **Confirmation-state microcopy should name Michell and describe what happens next in concrete, human terms**, not generic "Booking received!" SaaS copy. E.g.: *"Michell heeft je aanvraag ontvangen en bevestigt 'm persoonlijk — meestal binnen een paar minuten, via WhatsApp of telefoon."* This converts an abstract promise into an expected, checkable next action (watch your phone for a message from a named person), which is more reassuring than a generic checkmark.
- **Show what was actually submitted, verbatim, before confirming.** A one-line summary ("Schiphol → Amsterdam, vandaag 14:30, €39") right before the "Reserveer" tap acts as a lightweight trust checkpoint — the user isn't sending a booking into a void, they're confirming specifics a real person will read.
- **No fake urgency or fake social proof in this flow** ("3 people are viewing this route!", star ratings) — it would directly contradict the one-real-person positioning and read as exactly the generic, bolted-on pattern the client is trying to get away from.
- **The fixed price shown in the modal should visually match the fixed price shown in the on-page rate table** (§Tarieven section already on the homepage) — same numbers, same styling for the amount — so the modal doesn't feel like a separate, less-trustworthy system bolted onto the page. Consistency between the static rate table and the interactive quote is itself a trust signal.
- **Because there's no dispatcher, don't imply real-time automated confirmation.** Avoid copy like "Instant confirmation" — it's not literally true (Michell confirms personally) and overpromising immediacy sets up a disappointment if he's mid-drive. "Meestal binnen enkele minuten" is honest and still fast.

---

## 5. Summary of concrete recommendations

| Area | Current | Recommended |
|---|---|---|
| Container | Static inline card | `<dialog>` modal, bottom-sheet on mobile / centered on desktop |
| Trigger | 2 anchor links scrolling to inline card | 1 shared `showModal()` handler on both CTAs |
| Bestemming | `<input type="text">`, fake €39 always | 5-option radio-chip group with price shown per chip, live-updating total |
| Ophaaladres | `<input type="text">`, no autocomplete | Same field, add `autocomplete="street-address"`, optional 1-tap "Schiphol" quick-fill via direction toggle |
| Datum / Tijd | `<input type="text">` with placeholder only | `<input type="date">` / `<input type="time">` for native pickers + real validation |
| Reserveer button | Dead `<a href="#">` | Real submit → in-modal confirmation state with Michell-specific microcopy |
| Close behavior | N/A (not a modal) | Escape + backdrop click, state preserved, focus returns to trigger |
| Motion | N/A | Respects existing global `prefers-reduced-motion` rule |

See `modal-prototype.html` for a working implementation of all of the above.
