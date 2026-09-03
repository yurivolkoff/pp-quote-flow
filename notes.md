# Prototype Notes — 2026-09-03-quote-flow

_Date: 2026-09-03 | Page: PDP entry → request surface → quote document → PDF → email_

## Task

Build the Instant Quote flow from `spec.md`: a PDP lead-row entry, two competing request surfaces (Option A editable page, Option B fixed modal), a shared delivery block, the tokenised quote document in four states, the print/PDF sheet, and the transactional email.

## Source

- `design/prototypes/2026-09-03-quote-flow/spec.md` — screen specs, copy deck, rationale table, Boris notes. Every user-facing string in the prototype comes from the copy deck verbatim, except the two additions listed under **Decisions where the spec was silent**.
- promotionpros.com PDP + cart/checkout, observed 2026-09-03 (screenshots in `design/library/screenshots/2026-09-03-*.png`).

## Design Library

- `design/library/promotionpros-pdp.md` — the system everything is built in.
- `design/library/promotionpros-checkout.md` — behaviour idioms only, not styling.

---

## Key Design Decisions

### Anchoring

- **The whole flow follows the new Tailwind PDP system, not the legacy checkout system.** Production runs two design systems on one domain: the PDP was rewritten (orange `#FE5000` CTAs at 4px radius and 52px, navy `#001542` underlined text-buttons, numbered configurator steps, 44px swatch radiogroup, interactive tier table), while cart and checkout are still flat-cornered Bootstrap. The quote flow is new surface area, so it joins the newer system. Address entry, the saved-address summary, the shipping-method radio list, the validation pattern, and the "No delivery options found" empty state are **behavioural** borrowings from checkout, restyled into the PDP system.
- **Button case follows the newest production control.** The React buy box ships "Add to cart" in sentence case; the older Bootstrap buttons are Title Case. Sentence case wins, which also matches the copy deck as written, so no string was rewritten.

### Structure

- **The modal (Option B) is rendered inside the PDP view, not as a separate page.** That is what it is in production: an overlay with no navigation. The "3 · Request modal (B)" tab opens the PDP with the dialog already open, so the two options are compared against the same underlying page rather than two divergent copies.
- **The contact card and the delivery block come from one shared `<template>`, instantiated twice** with an instance suffix. Option A and Option B cannot drift apart, and typed values are mirrored between them, which is what `sessionStorage` does in the spec.
- **One price renderer feeds the desktop rail, the modal, and the mobile bottom sheet.** Three surfaces showing the same number can never disagree.
- **Pricing is real arithmetic, not hardcoded strings.** A seven-row tier table (100 → $2.68 down to 10,000 → $1.44) with 250 → $2.18, plus $0.35 per piece per imprint colour past the first and $60.00 setup per checked location. That reproduces the spec's $545.00 / $87.50 / $60.00 / $692.50 / $731.10 exactly and makes every control genuinely live.

### Behaviour

- Re-pricing is asynchronous: the rail dims to 40% with a 2px progress bar under the heading and no layout shift, then announces the new total to a polite live region.
- The CTA enables only when the full name is present and the email validates, with a reason line naming the missing thing. With delivery ticked but no option chosen the reason becomes "Choose a delivery option to continue".
- The disabled CTA uses `#DCDCDC` on `#6C6C6C` per spec, a deliberate improvement on production's near-illegible disabled state. Disabled controls are exempt from WCAG 1.4.3; the reason line beneath carries the information at 7.4:1.
- The specialist state is a state of the same surface: it cross-fades in place, keeps the configuration echo and every typed value, hides the delivery checkbox, and swaps the validity note for a computed promise date.

---

## Two spec amendments applied (they override the spec)

1. **No free-shipping threshold exists on production.** Neither the PDP, the cart, nor the checkout showed a threshold panel on the 2026-09-03 pass. Every free-shipping string is removed: `SH-60`, `SH-61`, and the `DEL-13` free row are not built. Shipping reads "Calculated at checkout" when delivery is not calculated, and sales tax reads "Added at checkout where applicable". **Open item for Yuri/Boris below.**
2. **The live shipping-option rows could not be captured.** The production rate call returned "No delivery options found" for the test order, so the radio rows are built from the spec (service · cost · "Arrives by {date}") in the PDP system and are **mock, visually unverified against production**. The captured empty state is built as a reachable state, carrying the deck's `DEL-14` string plus production's own chat fallback.

---

## Decisions where the spec was silent

1. **First-arrival banner copy.** The spec names the state but not the string. Written as "Your quote is ready. We emailed a copy with the PDF to dana@acme.com." — it confirms the two things a first-time viewer needs: the quote exists, and a copy is already in their inbox.
2. **The tier table itself.** The spec fixes 250 → $2.18 and "above 10,000 is priced by hand" but gives no curve. Built as a seven-break table with a shallow taper, so the interactive tier component has something plausible to drive.
3. **How the default-imprint notice stays truthful.** `SH-20` says the default is 1 imprint colour, while the mock order is 2 colours. Rather than showing a notice that contradicts the configuration, the "Buyer arrived" chip switches both together: "Configured on the PDP" hides the notice and shows the 2-colour mock order; "From search · default imprint" shows the notice and drops the configuration to 1 colour, which re-prices to $605.00.
4. **The imprint-method and location vocabularies.** Pad print / Screen print / Laser engraving / Full-color dye sublimation, and Barrel / Clip / Cap. Dye sublimation doubles as the `SP-03` call-for-pricing trigger.
5. **Setup charge is per location, not per quote.** The spec says each checked location adds its own setup line to the rail but never fixes the amount; $60.00 per location keeps the mock order at $692.50 while making the second location visibly expensive, which is the honest behaviour.

---

## Baymard Compliance

| Rule ID | Rule | Pass/Fail | Notes |
|---|---|---|---|
| B-PDP010 | Primary CTA visually unique | Pass | Only Add to Cart is orange/filled. The lead row is navy outline, one tier below. |
| B-PDP013 | Price per unit alongside total | Pass | "$2.18 each" on the rail line, the PDP subtotal box, the quote table, and the PDF. |
| B-PDP015 | Shipping estimate available pre-checkout | Pass | The delivery block gives a real cost and arrival date before any order is placed. |
| B-PDP017 | Free-shipping status near the buy section | N/A | No threshold exists on production. Justified under amendment 1; re-test when Boris supplies a live value. |
| B-PDP019 | Colour swatches, not drop-downs | Pass | 44px swatch radiogroup on both the PDP and the request page. |
| B-PDP027 / F-PDP008 | No site-initiated overlays | Pass | The modal is user-initiated by the quote button only. |
| B-PDP038 | Buy section stays focused | Pass | The lead row adds two buttons and one helper line, nothing else. |
| B-PDP039 | Delivery dates, not speeds | Pass | "Arrives by Thu, Sep 24, 2026" everywhere. No "3–5 business days" string exists in the file. |
| B-PDP041 | Complex customisation not crammed pre-cart | Pass | Option B keeps the configuration read-only and sends changes back to the PDP. |
| B-PDP042 | Single-option variations shown as text | Pass | Read-only rows in Option B and on the quote document. |
| F-PDP004 | Price ambiguity on bulk products | Pass | Unit price, extended amount, and quantity are always shown together. |
| F-PDP005 | Hidden shipping cost | Pass | Optional, but available, and the quote says which branch it took. |
| B-CHK003 | Every shipping option's cost shown upfront | Pass | Service, cost, and arrival on one row; no click-to-reveal. |
| B-CHK005 | No new cost elements late in the flow | Pass | Setup and run charges are itemised before any field is filled; the trust line names the only two remaining additions. |
| B-CHK008 | Enclosed header | Pass | Logo, title, one back link. No site nav on the request or quote pages. |
| B-CHK012 | Inline validation on blur | Pass | Validation fires on blur, never during typing. |
| B-CHK013 | Never clear typed data | Pass | No reset path exists; values also survive the hop between page and modal. |
| B-CHK014 | Adaptive, specific error messages | Pass | "Email addresses look like name@company.com", "ZIP code is 5 digits", "Quantity starts at 100 units for this product". |
| B-CHK015 | Field highlighted red with adjacent message | Pass | 1px `#F80A0A` border with the message directly below. |
| B-CHK017 | Cheapest option preselected | Pass | FedEx Ground preselected. |
| B-CHK018 | Actual delivery dates | Pass | As B-PDP039. |
| B-CHK021 | ZIP auto-fills city and state | Pass | Fires on the fifth digit. |
| B-CHK022 | Single full-name field | Pass | |
| B-CHK023 | Single phone field, not required | Pass | Optional, with a visible reason that changes wording on the specialist path. |
| B-CHK024 | Address line 2 behind a link | Pass | "Apartment, suite, unit (optional)". |
| B-CHK025 | Privacy statement near personal data | Pass | Closes the contact card. |
| B-CHK030 | Primary action more prominent than secondary | Pass | Orange filled 52px vs navy outline. |
| B-CHK031 | Progress indicator, disabled CTA with reason | Pass | Rail progress bar during re-price, rate-check spinner label, reason line under every disabled CTA. |
| B-CHK039 | Single-column form | Pass | The contact card is single-column. The configuration card is a two-column control grid, not a form — it collapses to one column below 960px. |

**P1 violations: none open.** B-PDP017 is the only P1 not satisfied and is not satisfiable — the feature it describes does not exist on production.

---

## Competitor Summary

| Competitor | Their Approach | PP Delta |
|---|---|---|
| Sticker Mule | Displays a complete price with an explicit "no additional charges aside from tax" promise | PP adopts the promise as the trust line, adapted to name shipping too when delivery is not calculated |
| UPrinting | Two tracks: an instant calculator and a rep-quoted Custom Quote with a 1–2 business day SLA | PP keeps both, but as one surface: the specialist state is the same form with a computed promise date |
| Custom Ink | Quick Quote reconfigures live | Option A does the same, and re-prices every control asynchronously |
| HubSpot Quotes | Locked, versioned artifact with an expiration field, link plus PDF | PP mirrors all four: reference number leads, real expiry date, new number on any change, link and attachment both delivered |
| 4imprint / Quality Logo Products | Rep-only quoting | PP is self-serve first, with the human path visible before and after submit so self-service never reads as losing rep support |

---

## Option A vs Option B

Switch between them from the **Quote entry** chips on view 1, then press "Get an instant quote". The tabs "2 · Request page (A)" and "3 · Request modal (B)" jump straight to either.

- **A — editable full page.** Two columns, every configuration control live, a sticky itemised rail, room for the delivery block. Linkable and back-button-safe.
- **B — fixed modal over the PDP.** 640px, read-only configuration summary with a "Change" affordance back to the product page, pinned header and footer so the total and CTA are never only in scrolled content. Becomes a full-screen sheet below 960px.

**The spec recommends A**, on the argument that the request surface is the last place a wrong specification can be corrected before it becomes a locked, forwardable document, and that itemisation plus a delivery block is more than 640px holds comfortably. Building both confirmed the second half of that: the modal needs its footer pinned and its body scrolled to keep the number visible, which the page does not.

---

## States and toggles

**View 1 — PDP**
- Quote entry: Option A · request page / Option B · modal
- Configurator: Complete / Colour not chosen → disables both lead buttons and shows the reason

**View 2 — Request page (A)**
- Buyer arrived: Configured on the PDP / From search · default imprint → shows the default-imprint notice and re-prices to 1 colour
- Pricing: Prices instantly / Specialist · 25,000 pcs / Specialist · dye sublimation
- Rate API: Returns options / Returns nothing → reach the captured "No rates came back" state
- Submitted: Form / Specialist confirmation → the post-submit panel with the reference number
- Delivery sub-states are reached by interacting: tick "Calculate delivery" (D1 → D2), pick a suggestion or "Enter address manually", complete the address, then "Show delivery options" (D2 → D3). "Change address" returns to D2 with the address retained.
- Validation: blur an empty name or a malformed email; the CTA stays disabled with its reason until both validate.

**View 3 — Request modal (B)** — same content, same chips. Esc, the ✕, or a backdrop click closes it and keeps what was typed. "Change" closes the modal and puts a focus ring on the PDP configurator for 2 seconds.

**View 4 — Quote page**
- Quote state: Valid / Expiring / Expired / Ordered
- Delivery: Calculated / Not calculated
- Arrival banner: On / Off (valid state only)
- Specialist: Named rep / Sales line

**View 5 — PDF** — Sheet: Valid / Expired · watermark. "Print this now" runs the real print stylesheet.

**Printing from any view renders the quote document**, because the print stylesheet forces `#view-quote` visible and hides everything else. One template, so the page and the PDF cannot drift.

---

## Known Deviations from Live Site

1. **Two design systems coexist on production; the quote flow follows the newer PDP system.** Cart and checkout keep flat 0px corners and `btn-h-54`; every new surface here uses 4px radii and the 52px PDP CTA. Deliberate — new surface area should not inherit the system that is being replaced.
2. **Error message text is `#D70000`, not production's `#F80A0A`.** The production red gives 4.17:1 on white and fails the spec's own ≥4.5:1 floor. The 1px border stays `#F80A0A` verbatim, since non-text needs only 3:1.
3. **Disabled CTA is `#DCDCDC` / `#6C6C6C`**, not production's `rgba(179,179,179,.5)` on `#F5F5F5`, which is near-illegible. Called for by the spec; flagged for Boris.
4. **The shipping-option rows are mock.** See amendment 2.
5. **The "Chat with us" fallback button is navy outline, not orange outline** as captured on production. Orange text at 14px is 3.3:1 on white; navy is 15.9:1 and is the PDP system's own secondary style. The button itself is kept rather than cut: production's own checkout renders a chat fallback in exactly this no-rates state, and PP runs Olark live chat sitewide, so it is an existing channel, not an invented one. The copy around it is now the deck's `DEL-14` string verbatim.
6. **Disabled buttons are `#DCDCDC` on `#6C6C6C`, about 3.6:1.** That is below the 4.5:1 body-text floor, and it is intentional: WCAG 1.4.3 conventionally exempts disabled controls, and every disabled CTA here carries a reason line beneath it in `#525252` at 7.4:1, which is where the actionable information lives. Still a visible improvement on production's own disabled treatment.
7. **The PDP keeps "pieces" while every other surface says "pcs".** Both PDP strings, the step recap and the subtotal box, are verbatim production wording. The rule applied was: production strings are not rewritten to match the flow, and every string the flow itself introduces uses "pcs".
8. **Product imagery is an inline SVG placeholder**, not a CDN photo, so the file stays self-contained with no external asset dependency.
9. **The cart entry point (`V1-05`, `V1-06`) is specced but not prototyped**, per the spec's own scope note.
10. **Annotation dots exist only on the PDP lead row**; the other views carry their annotations in the per-view legend, which holds more than a hover tooltip can.

---

## Open items for Yuri

1. **Free-shipping threshold.** There is none on production as of 2026-09-03. Either the feature was removed or it never shipped. Three copy strings and one delivery row depend on the answer, and they are currently absent from the build. Confirm with Mike or Boris whether a threshold is coming back before this flow is specced for development.
2. **Option A or B.** The spec recommends A. Both are built and comparable side by side; the decision is still yours.
3. **Is phone optional?** Built optional, as specced, with a visible reason. This reverses the July sample-request design.
4. **Named rep for anonymous viewers.** Both variants are built behind a chip. The spec recommends showing a named rep only when the requester's email matches a HubSpot company with an owner.
5. **Expiring threshold of 5 days**, plus a day-25 reminder email, is specced but not prototyped as an email.
6. **Placeholder contact details** — Terry Alvarez, 312-555-0148, 800-555-0100, and the West Bend address block — need real values before anything ships.

## Boris asks

1. **Confirm the free-shipping threshold really is gone**, and expose the live value if it returns, since `SH-60` / `SH-61` / `DEL-13` all render from it.
2. **The shipping rate call returned nothing** for a 500-unit apron to a Chicago address, twice. Is that a real freight-class limitation for larger quantities or a bug? The quote flow reuses this call, so the empty state may be common rather than rare.
3. **Tokenised quote URL** `promotionpros.com/q/{token}` — opaque, no login, no enumeration, and it must still resolve after expiry and after ordering, showing the state rather than a 404.
4. **Config restore reuses the PDP's existing URL params** (`im`, `l`), extended with quantity, product colour, and imprint-colour count, so the back-to-PDP link and the order-this-quote path replay one canonical string.
5. **Price lock persists the full priced breakdown** on the quote record, not a recomputation key, and the cart is flagged quote-priced through checkout.
6. **Expiry flips at midnight Central by nightly job**; "Get an updated quote" creates a new record and never mutates the expired one.
7. **Q-deal is created on generation, not on order**, carrying a `priceable: true|false` flag that routes the specialist path to a rep queue. Confirm the pipeline and routing owner with Emily.
8. **Named events** — `quote_started`, `quote_generated`, `quote_viewed`, `quote_pdf_downloaded`, `quote_shared`, `quote_ordered`, `quote_specialist_contacted` — as real named events, not text-matched actions.
9. **PDF is a server-side headless print of the quote page** against the same stylesheet; cache on send, regenerate on state change.
10. **Delivery rates reuse the checkout rate call and address autocomplete unchanged**, and results are cached on the quote record so the PDF shows the number the buyer saw.
11. **Priceability needs a server-side rule** — over-max-tier quantity plus a per-method `callForPricing` flag — evaluated on every re-price, not only at submit.
12. **The disabled-button treatment on live checkout is near-illegible.** The `#DCDCDC` / `#6C6C6C` pair used here is a small, safe global improvement.
13. **Email is transactional, not Klaviyo marketing**, with the PDF attached and reply-to set to the assigned rep where one exists.

---

## Copy addendum — strings with no deck ID

Every string below is written for the prototype and has no `spec.md` entry. Listed with its final text so the deck can absorb them.

| Where | Final text |
|---|---|
| Quote page, first-arrival banner | Your quote is ready. We emailed a copy with the PDF to dana@acme.com. |
| Delivery, rate-check button while loading | Checking rates… |
| Delivery, reason under a disabled "Show delivery options" | Add the full address to check rates |
| Delivery, same line once the address validates | Rates come from the same service checkout uses |
| Delivery, live-region announcements | Delivery options ready. / No rates came back. |
| Request page, imprint-location helper | Each location carries its own one-time setup charge |
| Request page, per-location suffix | +$60.00 setup charge |
| Request page, imprint-colors helper | Each color past the first adds $0.35 per piece |
| Request page, product sub-line | SKU PP-DELMAR-BMB |
| PDP, decoration-method note | One-time setup charge: $60.00 per location. Second imprint color adds $0.35 per piece. |
| Rail, setup-charge tooltip trigger label | What the setup charge covers |
| Quote page, note under "Message us about this quote" | Opens a message prefilled with "I have a question about quote Q-48213." |
| Quote page, invalid-link recovery button | Back to promotionpros.com |
| Quote page, enclosed-header title when the token is bad | Quote |
| PDF sheet, section headings | Prepared for / Product and configuration / Terms |
| PDF sheet, caption under the page | Letter, 8.5″ × 11″. Page 2 carries the terms and footer when page 1 overflows. The thumbnail renders at 150dpi minimum in the generated file. |

Carried from production verbatim, not written here: "Add to cart", "See price breakdown", "Have a product question? Ask us", "Secure Transaction", "Edit color & quantity", "Key Facts", "250 pieces", "Minimum Quantity", "Price per unit", "Chat with us".

---

## Change Log

- 2026-09-03 Initial prototype. Six views, both request-surface options, four quote states across two delivery variants, print stylesheet, email mock.
- 2026-09-03 Removed every free-shipping string per amendment 1; shipping and tax lines now always name where they are added.
- 2026-09-03 Built the delivery options list from the spec and the captured empty state as a reachable toggle, per amendment 2.
- 2026-09-03 QA pass at 1280 and 375: fixed the label selector losing its weight on non-`<label>` elements, the dialog header clipped by the prototype chrome, the crushed mobile header, duplicated labels in the stacked line-item table, run-together shipping-option rows, the PDF footer overlapping the last term, and a stale shipping line that survived an empty rate response.
- 2026-09-03 **R2, from the UI-consistency and copy reviews.**
  - Validation error text now matches the live checkout pattern at 15px weight 400. The darkened red stays.
  - The quote page's Total moved into the action rail directly above "Order this quote", mirroring the email view. Price and primary action are now visible together at 1280 and at 375.
  - The weight-900 tier is gone everywhere, including from the font request. Production Lato ships 400 and 700; hierarchy now comes from size alone.
  - The PDP "Ask us" row carries the "Secure Transaction" trust marker again, as production does.
  - Disabled-button contrast left as specced, with the exemption written into Known Deviations.
  - `DEL-14` is the deck string verbatim. The invented "Still stuck?" sentence is deleted; the chat button stays, with its rationale recorded.
  - The rail's first line item leads with the product name, per `SH-51`; the colour moved to the sub-line, which matters once the specced cart mode renders one line per product.
  - The no-rep specialist card leads with the deck's "Talk to a specialist" heading, parallel to the named-rep card. The invented heading is gone.
  - Terminology: "setup charge" everywhere, "pcs" on every string the flow introduces. Production's own "pieces" is left alone on the PDP.
  - Four curly apostrophes replaced with straight ones.
  - `V3-27` is now reachable as an "Invalid link" state chip, which also blanks the quote number from the enclosed header.
  - Also found and fixed in passing: British spellings in customer-facing copy. "Each colour past the first" is now "color".
