# Reservation flow — recommendation (v1)

**Context:** Le Grand Taxi is a one-driver, one-car fixed-price airport transfer business (Schiphol, NL). Owner/driver Michell (06 85 63 90 77) needs every booking to reach him personally so he can accept it and put it in his own agenda. Static site (GitHub Pages today, maybe Vercel/Netlify later), no dev budget, no backend today.

## Recommendation: WhatsApp deep link as PRIMARY, free form-to-email as SECONDARY fallback, Google Calendar template link included in both

Do not pick just one channel — combine two zero-cost channels so a booking can never silently disappear, and give the customer a confirmation either way.

### 1. WhatsApp deep link (`wa.me`) — PRIMARY "Reserveer" action

This is the best fit for this business and should be the main button.

- **Cost:** free. **Backend:** none — it's a plain link.
- **Mechanism:** `https://wa.me/31685639077?text=<url-encoded message>`. Tapping/clicking it opens WhatsApp (native app on mobile, WhatsApp Web/Desktop on desktop) with Michell's chat already open and the message pre-typed in the input box. The customer still has to press Send — WhatsApp's own API never sends a message automatically, which is a good thing (no accidental auto-bookings, no spam-blasting risk from the link itself).
- **Number format:** the `wa.me` path takes the full international number with **no leading `+`, no `00`, no spaces/dashes**. Michell's `06 85 63 90 77` → `+31 6 85 63 90 77` → **`31685639077`**. (Drop the trunk `0`, prepend `31`.)
- **URL encoding:** the `text` param must be run through `encodeURIComponent()` (not raw `escape` or manual replace) so line breaks (`\n`), accented characters, and symbols like `€` survive. Line breaks inside the pre-filled message work fine in WhatsApp — `%0A` renders as a real newline.
- **Reliability:** very high. `wa.me` is Meta's own official short-link product, it's been stable for years, works identically on iOS, Android, and desktop browsers. On desktop without the WhatsApp Desktop app installed it falls back to WhatsApp Web (still works, just needs the customer to have once scanned a QR code to link their phone — see caveat below).
- **What if the customer has no WhatsApp installed?** This is the one real gap.
  - **Mobile:** if the WhatsApp app isn't installed, `wa.me` redirects to the WhatsApp download page in the App/Play Store instead of opening a chat — a dead end for that customer.
  - **Desktop:** if the customer has no phone linked to WhatsApp Web at all, the desktop web flow can't work either.
  - Mitigation: WhatsApp is used by ~85%+ of the Dutch population and is the de facto default in NL, so this affects a small minority — but it's why WhatsApp should not be the *only* option (see Formspree/Web3Forms fallback below).
- **Should it be the only flow?** No — make it the prominent, default "Reserveer" button (this matches the site's "Michell replies personally" positioning perfectly), but keep one small secondary link/option for people who don't use WhatsApp.

### 2. Free form-to-email service — SECONDARY fallback ("Liever geen WhatsApp?")

For the minority without WhatsApp, and as a durable written record that doesn't depend on Michell checking a chat app.

- **Candidates:** Formspree and Web3Forms were evaluated; both work from a **plain static `<form>`** with `method="POST"` and no JavaScript, no build step, no server — exactly what a GitHub Pages/static site needs.
  - **Formspree free tier:** roughly 50 submissions/month, form posts to `https://formspree.io/f/{formId}`, requires creating a free account and one form to get that ID. First submission from a new domain requires a one-time confirmation click by the account owner (Michell or whoever sets it up).
  - **Web3Forms free tier:** roughly 250 submissions/month, form posts to `https://api.web3forms.com/submit`, needs only a free "access key" (obtained via email, no full account signup required), also works from static HTML with a hidden `access_key` field.
  - Both send the form contents straight to a real inbox (Michell's email) — no server-side code needed at all, which fits the "no backend" constraint exactly.
  - Neither is instant-push the way WhatsApp is — it's email, so it depends on Michell checking his inbox (or having phone email notifications on).
- **Customer confirmation:** both services can show an on-page "thank you" / redirect after submit; both support an autoresponder-style confirmation email back to the customer's address (configurable in the account dashboard). This should be turned on so the customer isn't left wondering if the booking went through — important since Michell might not reply within minutes.
- **Spam/abuse:** free tiers have monthly caps, which double as a natural throttle. Add a honeypot hidden field (a text input hidden via CSS, real users never fill it, bots do — the service can auto-reject) and consider Web3Forms' free reCAPTCHA/spam-filter option. A single person spamming 100 submissions would burn through the free tier fast — worth knowing, not a launch blocker for a low-traffic local taxi site.

### 3. Calendar integration — Google Calendar "add event" URL, included in the message/email

- **v1 recommendation:** use Google Calendar's URL-based event creation — `https://calendar.google.com/calendar/render?action=TEMPLATE&text=...&dates=...&details=...&location=...` — **no OAuth, no API key, no Google Cloud project.** It's just a specially-formatted link that opens Google Calendar's "add this event" screen pre-filled, which Michell taps to confirm and save. This is included as a real, tappable URL inside both the WhatsApp message text and the fallback email, so after Michell accepts a booking he's one tap away from having it on his agenda.
- **True calendar API integration** (Google Calendar API with OAuth, service accounts, refresh tokens, auto-inserting events without human confirmation) was considered and is **overkill for v1**: it needs a backend to hold credentials securely, ongoing OAuth token maintenance, and Google API quota/verification review for a public-facing app. For a one-driver business this is disproportionate complexity for the benefit — the URL-template approach gets 90% of the value (one tap to save) with zero infrastructure. Worth revisiting only if Michell wants bookings to auto-appear without him tapping anything.

### 4. Other zero/near-zero-cost option worth flagging

- **If/when the site moves to Vercel or Netlify:** a single small serverless function (e.g. `/api/reserve`) could receive the form POST and fan it out to *both* a WhatsApp-adjacent channel and email/Telegram in one request, removing the need for two separate buttons. This is genuinely simple (one file, ~30 lines, using a free transactional-email API like Resend's free tier, or the free Telegram Bot API to push an instant phone notification to Michell — arguably even better than email since it's push, instant, and free with no monthly cap). This is **not needed for v1** given the static-site constraint, but is the natural next step if the client outgrows the two-button approach and wants one unified "Reserveer" button with server-side fan-out. Flagging it, not building it now.

### Fallback if Michell misses/ignores a notification entirely
Neither WhatsApp nor email guarantees Michell sees it in time. Mitigations already in place / recommended:
- The phone number and `tel:` "Bel Michell" call button stay prominent on the page — a customer with a time-sensitive booking can always just call.
- Turning on the form service's customer-facing autoresponder ("We hebben je aanvraag ontvangen, Michell neemt snel contact op") sets expectations and reduces "did this even work?" anxiety, partially compensating for no guaranteed instant read-receipt equivalent on the email path.
- This is a one-driver business — there is no dispatcher fallback possible without paid infrastructure; WhatsApp read receipts (blue ticks) already give Michell (and indirectly the customer, if he's WhatsApp-savvy) some signal of whether it's been seen.

## Summary table

| Option | Cost | Backend needed | Reliability | Customer confirmation | Setup effort |
|---|---|---|---|---|---|
| WhatsApp `wa.me` link (primary) | Free | None | High (fails only if customer has no WhatsApp) | Implicit (WhatsApp read receipts) | None — just build the URL |
| Web3Forms / Formspree (fallback) | Free tier | None (pure static form) | High, but not instant (email-based) | Yes, via autoresponder | ~10 min: create free account/key |
| Google Calendar template link | Free | None | High | N/A (for Michell, not customer) | None — just build the URL |
| Vercel serverless fan-out | Free tier | Yes, minimal | High | Yes | Only if/when site moves off GitHub Pages |

## What a human still needs to do to go live
See the bottom of `reservation-prototype.html` for the exact checklist — in short: (1) nothing needed for the WhatsApp button, it works as-is; (2) sign up for a free Web3Forms access key (or Formspree form) and paste the key/ID into the fallback form's hidden field.
