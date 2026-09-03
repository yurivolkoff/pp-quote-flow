# Prototype Notes — 2026-09-03-quote-flow

_Date: 2026-09-03 | Page: PDP entry → request modal → quote document → PDF → email_

## Task

Build the Instant Quote flow: a PDP lead-row entry, a lean request surface, the tokenised quote document in four states with two optional post-capture modules, the print/PDF sheet, and the transactional email.

**R3 (2026-09-03) replaced the request surface.** Options A (editable page) and B (fixed modal) were judged too heavy and are removed entirely — markup, CSS and JS. In their place: **Option C**, a 480px lean modal, and **Option D**, a 640px modal with three editable production controls. Company, phone, need-by and delivery moved off the request surface onto the quote page as two optional modules.

## Source

- `research/reports/2026-09-03-quote-request-lean-options-ux-analysis.md` — the R3 spec: element inventories, interaction spec, PDP-control reuse map, consistency matrix, risk register, copy deck delta, quote-page change list.
- `design/prototypes/2026-09-03-quote-flow/spec.md` — the original screen spec and copy deck. Still the source for every string the analysis marks "reused".
- promotionpros.com PDP, cart and checkout, observed 2026-09-03 (`design/library/screenshots/2026-09-03-*.png`).

## Design Library

- `design/library/promotionpros-pdp.md` — the system everything is built in, including the **"Configurator controls — captured 2026-09-03"** section, which Option D is built against.
- `design/library/promotionpros-checkout.md` — behaviour idioms only, not styling.

---

## Product-owner decisions applied this round

1. **Vocabulary follows the production PDP flow-wide.** "Decoration" names the process; "imprint" names the mark, its colors and its charges. The unit word is "pieces" on every surface. Applied to views 1, 2, 3, 4, 5 and 6, so no buyer sees two words for one thing on adjacent surfaces.
2. **Option C is a modal, not an inline panel.** The analysis recommended an inline panel replacing the buy box; the product owner chose a user-initiated 480px modal over the scrimmed PDP, becoming a full-screen sheet below 960px. Built as decided. The trade the analysis named still applies: the modal costs a scrim, a focus trap, a scroll lock and a mobile sheet, and buys one component that can also serve the cart and a future quick-view.
3. **Option D is a 640px modal with a pinned footer**, editable quantity, decoration method and imprint location on production's own controls, colour and imprint colors read-only.
4. **Company, phone, need-by and delivery are not on the request surface.** They are two optional modules on the quote page, open to any link holder, both keeping the quote number and adding a dated revision stamp.
5. **No free-shipping strings.** The specialist fallback stays. Cart entry is specced in one paragraph below, not built.

---

## Key Design Decisions

### Anchoring

- **The whole flow follows the new Tailwind PDP system, not the legacy checkout system.** Production runs two design systems on one domain: the PDP was rewritten (orange `#FE5000` CTAs at 4px radius and 52px, navy `#001542` underlined text-buttons, numbered configurator steps, 44px swatch radiogroup, interactive tier table), while cart and checkout are still flat-cornered Bootstrap. The quote flow is new surface area, so it joins the newer system. The address entry, the saved-address summary, the shipping-method radio list and the "no rates" empty state are **behavioural** borrowings from checkout, restyled into the PDP system.
- **Reused controls keep production case; flow-authored strings are sentence case.** `Decoration Method`, `Imprint Colors`, `Minimum Quantity`, `Price per unit`, `Subtotal with Setup Fee:`, `Unit Price:`, `Quantity:`, `Edit Color & Quantity`, `First Location`, `Extra Location`, `Imprint #1 cost:` are verbatim. Everything the flow writes itself is sentence case.

### Structure

- **Both modals live inside the PDP view, not as separate pages.** That is what they are in production: overlays with no navigation. Tabs 2 and 3 open the PDP with the corresponding dialog already open, so C and D are compared against one underlying page rather than two divergent copies.
- **One contact template, two instances.** Full name and work email come from a single `<template>` instantiated with a suffix into C and D, with typed values mirrored between them — what `sessionStorage` does in the spec. The two options cannot drift apart.
- **One price renderer** feeds C's disclosure, D's disclosure and the quote document. Three surfaces showing one number cannot disagree.
- **The quote holds a snapshot, not a live read.** Submitting freezes the configuration into the quote record; changing the PDP afterwards does not silently re-write an issued document. A configuration change is what produces a new number.
- **Pricing is real arithmetic.** A seven-row tier table (100 → $2.68 down to 10,000 → $1.44) with 250 → $2.18, plus a per-method run charge per imprint color past the first and a per-method setup charge. Pad print reproduces the spec's $545.00 / $87.50 / $60.00 / $692.50 exactly.

### Behaviour

- Re-pricing is synchronous in C (it has no controls) and live in D. D announces every new total to a polite live region.
- The CTA enables only when the full name is present and the email validates; in D it also requires a quantity at or above the minimum. The reason line names the missing thing.
- Every dependency clamp in D renders a notice, is announced, and persists until the next change or submit. Nothing is ever clamped silently.
- The specialist state is a state of the same modal: it replaces the itemisation in place, keeps every typed value, and swaps the promise line for a computed date. In D the controls stay live, so editing back restores the priced state.

---

## Option C vs Option D

Switch between them with the **Quote entry** chips on view 1, then press "Get an instant quote". Tabs "2 · Quote modal (C)" and "3 · Quote modal (D)" open either directly.

- **C — lean modal, 480px.** Thumbnail, one configuration line in words, a "Change" affordance back to the PDP, the total with its unit price, a "See price breakdown" disclosure, two required fields, one CTA, one promise line. Zero editable controls, six interactive elements. Amber notice when the buyer never set decoration on the PDP. Full-screen sheet below 960px, no sticky footer needed.
- **D — editable modal, 640px, pinned footer.** Everything C shows plus production's price-tier table with the paired `Quantity:` / `Unit Price:` fields, the decoration-method card radiogroup with its `Decoration Charges` table, and the imprint-location control. Colour and imprint colors are read-only rows with a helper naming where they change. Total and CTA live in the pinned footer, so the number is never only in scrolled content.

**The analysis recommends C**, on the argument that it is a full effort tier lighter, needs no new controls, and works on mobile without a sheet — with the instruction to instrument `quote_change_clicked` and let D earn its build if more than roughly 15% of opens end in a return to the PDP. Building both confirms the second half: D needs a pinned footer and a scrolling body to keep the number visible, which C does not.

---

## What the production capture changed

The capture section in `promotionpros-pdp.md` landed after C and the quote-page modules were built. Reading it against the analysis produced four corrections, all applied.

1. **The subtotal box is not goods-only.** The analysis treated production's `Subtotal:` box as excluding setup and run charges, and built D's whole footer strategy around not redefining that word. The capture shows the multi-method product renders **`Subtotal with Setup Fee:`** carrying base + setup + run — `$330.25`, the same figure the breakdown dialog calls `Total Price`. D now uses the production label and the production number, which equals the flow's Total. The analysis' highest-risk reuse, two competing money figures in one modal, does not exist. The breakdown disclosure moved back into the box, which is where production puts it, and the footer carries the Total and the CTA only.
2. **The imprint-location control is a `react-select` combobox**, not the swatch-style radiogroup the analysis assumed, and not a native `<select>` either. The prototype uses a native select as the closest static stand-in; the real build imports the component. Logged as a deviation and a Boris ask.
3. **The method control is a `role="radiogroup"` of plain `<button aria-pressed>` cards**, hand-authored BEM (`imprint-method-card`), not Tailwind. Computed CSS reused verbatim: 8px radius, unselected `1px solid #EEEEEE` on white at `12px 16px`, selected `2px solid #FE5000` on `rgba(254,80,0,.05)` at `11px 15px`. The prototype had used `role="radio"` / `aria-checked` at a 4px radius; both are corrected.
4. **Production silently resets the imprint location whenever the method changes**, even when the new method offers the old location — confirmed by the URL's `l=` param changing. D deliberately does not copy that: it keeps the location where the new method offers it, and where it cannot, it falls back and says so (`D-20`). That silent reset is precisely the failure `D-20` exists to prevent. Logged for Boris as an intended divergence.

---

## Two spec amendments still in force

1. **No free-shipping threshold exists on production.** Every free-shipping string is removed: `SH-60`, `SH-61` and the `DEL-13` free row are not built. Shipping reads "Calculated at checkout" until the delivery module is used.
2. **The live shipping-option rows could not be captured.** The production rate call returned "No delivery options found" for the test order, twice, so the radio rows are built from the deck and are **mock, visually unverified**. The captured empty state is a reachable state carrying `DEL-14` verbatim.

---

## Decisions where the analysis was silent

1. **C has no pinned footer.** The analysis specced a pinned footer for D only. C runs under 500px tall, so its CTA sits in flow; at 375 it lands just at the fold rather than above it, which is the one place C is slightly worse than the inline panel the analysis preferred.
2. **One breakdown string across the flow.** Production ships both "See price breakdown" (apron) and "View Price Breakdown" (tumbler) for the same control. The flow uses `C-06` "See price breakdown" everywhere, so C, D and the PDP agree. A Boris ask asks production to normalise.
3. **The itemisation keeps the deck's labels, not production's breakdown-dialog labels.** Production's dialog says "Base Item Price", "Imprint Method Setup", "Imprint Method Run Charge", "Total Price", "Average price per unit". The flow uses `SH-51`–`SH-59`, which are also the quote page's, the PDF's and the email's labels. One of the two vocabularies should win before development; logged for Boris.
4. **Method option names stay the flow's mock vocabulary.** Production's names are per-product data ("Laser Engraved", "Screen Printed"), not UI labels. The prototype keeps Pad print / Screen print / Laser engraving / Full-color dye sublimation, because six copy-deck strings and the terms line render from them.
5. **The method card carries a one-line constraint summary** ("Up to 4 colors · 3 locations"). Production's card is name-only, because the constraints live in a Decoration & Imprint accordion that D does not have. Without it, D's clamp notices arrive unexplained.
6. **Step numeral badges are dropped, the dark step bar is kept.** "3" and "4" without "1" and "2" is incoherent once D is not the four-step wizard, but the bar itself is a strong production idiom and holds the reused labels.
7. **The chosen delivery option can be changed.** The analysis gives the delivery module a confirmation and a re-send but no way back. "Change delivery" returns to the options list; without it, a mis-picked service is a dead end on a document that stays live for 30 days.
8. **The details revision stamp reads "Details added Sep 4, 2026"**, parallel to `QD-05`, which the deck defines only for delivery.
9. **The email splits one row into two.** The initial email's single "Imprint" row becomes "Decoration method: Pad print" and "Imprint: Barrel, 2 colors", so the email obeys the same decoration/imprint rule as every other surface.
10. **The tier curve, the method and location vocabularies, the rep contact details and the product image** are mock, carried from the R1 build.
11. **The prototype's own chip rail floats above the scrim at 1280** so modal states stay switchable, and drops behind the full-screen sheet below 960px. Prototype scaffolding, not product.

---

## Cart entry (specced, not built)

`V1-05` "Quote this cart" + `V1-06` sits below Proceed to Checkout as a full-width outline button and opens **Option C only**. Per-line quantity editing inside a modal is a second cart, so D is not offered there. C renders one configuration summary line per cart line up to three, then "+N more items", with the combined total below; setup charges stay per line in the breakdown rather than merging. The quote page renders one line-item block per product under a single quote number. Fields, modules, validity and actions are identical.

---

## States and toggles

**View 1 — PDP**
- Quote entry: Option C · lean modal / Option D · editable modal
- Configurator: Complete / Color not chosen → disables both lead buttons and names the missing thing
- Decoration: Set on the PDP / Never set · default → drives `SH-20` and drops the configuration to 1 imprint color so the notice is true
- Pricing: Prices instantly / Specialist · 25,000 pieces / Specialist · dye sublimation
- Submitted: Form / Specialist confirmation

**View 2 — Quote modal (C).** Reads all five chips. "Change", the ✕, Escape and a backdrop click all close it; focus returns to the quote button and typed values survive. "See price breakdown" opens the itemisation.

**View 3 — Quote modal (D).** Same chips plus live controls:
- Quantity: click a tier column, or type. Under 100 → `SH-22`. Over 10,000 → specialist state mid-edit.
- Decoration method: four cards. **Laser engraving from the Cap location fires both dependency notices at once** — location falls back to Barrel (`D-20`) and the imprint-color count clamps from 2 to 1 with the run charge removed (`D-21`, carrying `D-22` back to the PDP).
- Imprint location: the options list reloads per method.
- Full-color dye sublimation is the call-for-pricing trigger.

**View 4 — Quote page**
- Quote state: Valid / Expiring / Expired / Ordered / Invalid link
- Arrival banner: On / Off (valid state only) · Specialist: Named rep / Sales line
- Calculate delivery: Not calculated / Calculating / Options / Chosen / Rates empty. Reachable by interacting too: press "Calculate delivery", pick a suggestion or enter the address manually, then "Show delivery options".
- Add details: Not added / Added. Added fills company, phone and a Sep 15 need-by, which is earlier than the Sep 24 arrival and therefore fires `QD-14`.

**View 5 — PDF** — Sheet: Valid / Expired · watermark. The sheet mirrors whatever module states view 4 is in. "Print this now" runs the real print stylesheet.

**View 6 — Email** — Message: Initial / Re-sent after a module save, which prefixes the subject "Updated:" and fills the delivery line.

**Printing from any view renders the quote document.** Module triggers and forms are hidden in print; module results, the revision stamp, the shipping row and the company line are not. Verified by computed style, not by eye.

---

## Baymard Compliance

Rows re-run after the R3 changes are marked **(re-run)**.

| Rule ID | Rule | Pass/Fail | Notes |
|---|---|---|---|
| B-PDP010 | Primary CTA visually unique | Pass | Only Add to cart is orange/filled on the PDP; the lead row is navy outline one tier below. While a modal is open, Add to cart is behind the scrim and inert. |
| B-PDP013 | Price per unit alongside total | Pass (re-run) | "$2.18 each" beside every total: C's total line, D's footer and subtotal box, the breakdown, the quote table, the PDF. |
| B-PDP015 | Shipping estimate available pre-checkout | Pass (re-run) | Delivery moved to the quote page but is still available before any order, with a real cost and arrival date. |
| B-PDP017 | Free-shipping status near the buy section | N/A | No threshold exists on production. Re-test when Boris supplies a live value. |
| B-PDP019 | Colour swatches, not drop-downs | Pass with exception (re-run) | Product colour is a 44px swatch radiogroup. Imprint location is a drop-down in D — because production itself ships a `react-select` combobox for it. Reusing production's control is the instruction; inventing a swatch picker for it would be the divergence the reuse map forbids. |
| B-PDP027 / F-PDP008 | No site-initiated overlays | Pass (re-run) | Both modals open only from the quote button. The ban covers site-initiated overlays. |
| B-PDP038 | Buy section stays focused | Pass | The lead row adds two buttons and one helper line. |
| B-PDP039 | Delivery dates, not speeds | Pass | "Arrives by Thu, Sep 24, 2026" everywhere. No "3–5 business days" string exists in the file. |
| B-PDP041 | Complex customisation not crammed pre-cart | Pass (re-run) | C is read-only. D exposes three controls, all production's own, keeps colour and imprint colors read-only with a helper naming where they change, and is user-initiated at 640px with a scrolling body — not a compressed second configurator. |
| B-PDP042 | Single-option variations shown as text | Pass | Read-only rows in both modals and on the quote document. |
| F-PDP004 | Price ambiguity on bulk products | Pass | Unit price, extended amount and quantity always shown together. |
| F-PDP005 | Hidden shipping cost | Pass (re-run) | Optional but available, and the quote states which branch it took in both the trust line and the shipping row. |
| B-CHK003 | Every shipping option's cost shown upfront | Pass | Service, cost and arrival on one row; no click-to-reveal. |
| B-CHK005 | No new cost elements late in the flow | Pass (re-run) | Setup and run charges are itemised before any field is filled. Delivery is the only figure added later, and it is added by the buyer, announced, and stamped on the document. |
| B-CHK008 | Enclosed header | Pass (re-run) | The quote page keeps the enclosed header. The request surface is a modal over the PDP, so the rule does not apply to it. |
| B-CHK012 | Inline validation on blur | Pass | Validation fires on blur, never during typing. |
| B-CHK013 | Never clear typed data | Pass (re-run) | No reset path exists; values survive closing a modal and hopping between C and D. |
| B-CHK014 | Adaptive, specific error messages | Pass (re-run) | "Email addresses look like name@company.com", "ZIP code is 5 digits", "Quantity starts at 100 pieces for this product". |
| B-CHK015 | Field highlighted red with adjacent message | Pass | 1px `#F80A0A` border, message directly below at 15px/400 in `#D70000`. |
| B-CHK017 | Cheapest option preselected | Pass | FedEx Ground preselected in the delivery module. |
| B-CHK018 | Actual delivery dates | Pass | As B-PDP039. |
| B-CHK021 | ZIP auto-fills city and state | Pass | Fires on the fifth digit. |
| B-CHK022 | Single full-name field | Pass | |
| B-CHK023 | Single phone field, not required | Pass (re-run) | Phone is optional and now lives in "Add details" on the quote page, still carrying its visible reason. |
| B-CHK024 | Address line 2 behind a link | Pass | "Apartment, suite, unit (optional)". |
| B-CHK025 | Privacy statement near personal data | Pass (re-run) | `SH-45` closes the two-field block in both C and D. It had been dropped when the contact card was reduced to two fields; restored in QA. |
| B-CHK030 | Primary action more prominent than secondary | Pass | Orange filled vs navy outline. |
| B-CHK031 | Progress indicator, disabled CTA with reason | Pass (re-run) | Rate-check spinner label, and a reason line under every disabled CTA naming the next missing thing. |
| B-CHK039 | Single-column form | Pass (re-run) | Both request surfaces ask two fields in one column. D's controls are a configurator, not a form, and stack below 960px. |

**P1 violations: none open.** B-PDP017 is the only P1 not satisfied and is not satisfiable — the feature it describes does not exist on production. B-PDP019 carries a documented exception, sourced to production's own control.

---

## Competitor Summary

| Competitor | Their Approach | PP Delta |
|---|---|---|
| Sticker Mule | Complete price with an explicit "no additional charges aside from tax" promise | PP adopts the promise as the trust line, adapted to name shipping too until the delivery module is used |
| ePromos | Quantity is the only control; price on load | Option C goes further: no controls at all, price carried in from the PDP |
| UPrinting | Two tracks: instant calculator and rep-quoted Custom Quote with a 1–2 day SLA | PP keeps both as one surface; the specialist state is the same modal with a computed promise date |
| Custom Ink | Quick Quote reconfigures live behind four fields plus a reCAPTCHA | Option D reconfigures live behind two fields and no captcha |
| HubSpot Quotes | Locked, versioned artifact with an expiration field, link plus PDF | PP mirrors all four, and adds additive modules that revise without renumbering |
| 4imprint / Quality Logo Products | Rep-only quoting | PP is self-serve first, with the human path visible before and after submit |

---

## Known Deviations from Live Site

1. **Two design systems coexist on production; the quote flow follows the newer PDP system.** New surface area should not inherit the system being replaced.
2. **Error message text is `#D70000`, not production's `#F80A0A`.** The production red gives 4.17:1 on white and fails the spec's own 4.5:1 floor. The 1px border stays `#F80A0A` verbatim, since non-text needs only 3:1.
3. **Disabled CTA is `#DCDCDC` / `#6C6C6C`**, about 3.6:1 — below the body-text floor and intentional: WCAG 1.4.3 exempts disabled controls, and the reason line beneath carries the actionable information at 7.4:1.
4. **D's imprint-location control is a native `<select>`.** Production ships a `react-select` combobox with hashed class names; a static prototype cannot import it. The visible label, the option list and the behaviour match.
5. **D's decoration-method cards are a reconstruction** of production's hand-authored `imprint-method-card` BEM component. Computed colours, borders, radius and padding are verbatim from the capture; the class names are not. Both controls carry a visible "reconstruction, pending an import" note in the prototype.
6. **D does not preserve production's silent location reset on method change.** Deliberate — see "What the production capture changed", item 4.
7. **Production's step numeral badges and step checkmarks are dropped**, the dark step bar and its labels are kept.
8. **The "Add Additional Imprint Location" control is not built.** Multi-location quoting is out of scope for this round; production exposes it and it was not exercised in the capture either.
9. **The Pantone/PMS checkbox that production shows for screen print is not built.** The capture could not confirm whether it changes the price at configurator time.
10. **The flow's itemisation labels are the copy deck's, not production's breakdown-dialog labels.** See "Decisions where the analysis was silent", item 3.
11. **The shipping-option rows are mock.** See amendment 2.
12. **The "Chat with us" fallback button is navy outline, not orange outline** as captured. Orange text at 14px is 3.3:1 on white; navy is 15.9:1.
13. **Product imagery is an inline SVG placeholder**, so the file stays self-contained.
14. **Annotation dots exist only on the PDP lead row.** Every other view carries a legend, and each modal carries its own legend beside the dialog at 1100px and wider.

---

## Open items for Yuri

1. **C or D.** The analysis recommends C and names the instrumentation that would let D earn its build: `quote_change_clicked` on "Change", with roughly 15% of opens returning to the PDP as the threshold. Both are built and comparable.
2. **Free-shipping threshold.** None exists on production as of 2026-09-03. Three copy strings and one delivery row depend on the answer and are currently absent.
3. **Which itemisation vocabulary wins** — the copy deck's `SH-51`–`SH-59`, used across the modal, the quote page, the PDF and the email, or production's breakdown-dialog labels. They cannot both be right.
4. **Whether the two modules should be open to any token holder.** Built open, per the analysis: the same token already authorises "Order this quote", so gating a phone number would be incoherent. Edits are date-stamped.
5. **Named rep for anonymous viewers.** Both variants are behind a chip. The recommendation is a named rep only when the requester's email matches a HubSpot company with an owner.
6. **Placeholder contact details** — Terry Alvarez, 312-555-0148, 800-555-0100 and the West Bend address block — need real values before anything ships.
7. **The Pantone/PMS control and multi-location imprints** are production features this flow does not carry. Confirm they are out of scope for the quote.

---

## Boris asks

Carried forward from R2, unchanged: tokenised quote URL that still resolves after expiry and after ordering; price lock persisting the full priced breakdown rather than a recomputation key; expiry flipping at midnight Central with "Get an updated quote" creating a new record; Q-deal created on generation with a `priceable` flag; named events rather than text-matched actions; PDF as a server-side headless print of the same page; delivery rates reusing the checkout call and caching results on the quote record; priceability as a server-side rule evaluated on every re-price; transactional email with reply-to set to the assigned rep; the near-illegible disabled-button treatment on live checkout; whether the free-shipping threshold is really gone; and why the rate call returned nothing twice for a real test order.

New in R3:

1. **One shared configurator state store.** The PDP and D must read and write the same state, not two copies. The capture confirms production already keeps it in the URL: `?c=BLUE&s=Default+Size:25&im=Laser+Engraved&l=<encoded>`. Extend that scheme with the imprint-colour count and let D write to it, so the PDP re-renders from it on close.
2. **D imports the PDP's controls; it does not copy their markup.** The price-tier table, the `imprint-method-card` radiogroup and the `react-select` location control must ship as importable components. Copied markup is exactly the divergence D exists to prevent, and it is the reason two of D's controls carry a visible "reconstruction" note.
3. **Production silently resets the imprint location when the decoration method changes.** Confirmed on the tumbler: a location chosen under one method is discarded, and the URL's `l=` param changes. Please make the reset announce itself on the PDP too, the way `D-20` does in the quote flow.
4. **Normalise the PDP's own label inconsistencies.** `Imprint Colors` sits beside `Imprint location`; `Subtotal:` on one product is `Subtotal with Setup Fee:` on another; `See price breakdown` on one is `View Price Breakdown` on another. The flow inherits whichever it reuses.
5. **Fix the price-tier table clipping** at the right edge of the ~520px buy box, where the top quantity column is cut off.
6. **Both quote-page modules must add a dated revision to the existing quote, never a new number.** Delivery and contact details are additive and leave the quoted goods price untouched. Only a configuration change issues a new number, carrying `V3-28` "Replaces quote Q-48213".
7. **Both modules are open to any token holder**, including a forwarded approver, with no extra gate.
8. **The PDF regenerates on every module save**, and its footer states its own generation date (`V4-06`) beside the always-current link. A module save sends no email on its own; `QD-04` re-sends the same template with an "Updated:" subject prefix.
9. **Decide which itemisation vocabulary is canonical** before the breakdown is built twice — see Open items 3.

---

## Copy addendum — strings with no deck ID

| Where | Final text |
|---|---|
| Quote page, first-arrival banner | Your quote is ready. We emailed a copy with the PDF to dana@acme.com. |
| Quote page, revision stamp | Revised Sep 4, 2026 · Delivery added Sep 4, 2026 · Details added Sep 4, 2026. The quote number and the quoted goods price are unchanged. |
| Quote page, details stamp | Details added Sep 4, 2026 |
| Quote page, change a chosen delivery | Change delivery |
| Delivery, rate-check button while loading | Checking rates… |
| Delivery, reason under a disabled "Show delivery options" | Add the full address to check rates |
| Delivery, same line once the address validates | Rates come from the same service checkout uses |
| Delivery, live-region announcements | Delivery options ready. / No rates came back. |
| D, method card sub-line | Up to 4 colors · 3 locations (and per method) |
| D, colour-count clamp above one colour | {Method} prints in up to {N} colors. Imprint colors changed from {X} to {N}, and the run charge is recalculated. |
| D, reconstruction note | Reconstruction, pending an import: this control is built from the 2026-09-03 production capture, not from the live component. |
| D, unit-price tooltip | The unit price comes from the quantity break you selected above. |
| Setup-charge tooltip | Covers preparing your artwork for this decoration method. Reorders of the same artwork skip it. |
| PDP, decoration-method note | One-time setup charge: $60.00 per location. Each imprint color past the first adds $0.35 per piece. |
| Email, re-sent confirmation toast | Updated PDF emailed to dana@acme.com |
| Email, re-sent lede | Here's the updated quote for 250 Del Mar Bamboo Stylus Pens, with the delivery you added. |
| Quote page, note under "Message us about this quote" | Opens a message prefilled with "I have a question about quote Q-48213." |
| Quote page, invalid-link recovery button | Back to promotionpros.com |
| PDF sheet, section headings | Prepared for / Product and configuration / Terms |

Carried from production verbatim, not written here: "Add to cart", "See price breakdown", "Have a product question? Ask us", "Secure Transaction", "Edit Color & Quantity", "Key Facts", "250 pieces", "Minimum Quantity", "Price per unit", "Quantity:", "Unit Price:", "Decoration Method", "Imprint Details", "Decoration Charges", "Setup", "Run Charge", "First Location", "Extra Location", "Imprint location", "Imprint Colors", "Imprint #1 cost:", "Subtotal with Setup Fee:", "Price each:", "Chat with us".

---

## Change Log

- 2026-09-03 Initial prototype. Six views, both request-surface options, four quote states across two delivery variants, print stylesheet, email mock.
- 2026-09-03 Removed every free-shipping string per amendment 1.
- 2026-09-03 Built the delivery options list from the spec and the captured empty state as a reachable toggle, per amendment 2.
- 2026-09-03 QA pass at 1280 and 375; fixed label weight, dialog header clipping, mobile header, stacked-table labels, shipping-option rows, PDF footer overlap, stale shipping line.
- 2026-09-03 **R2**, from the UI-consistency and copy reviews: error text at 15px/400, quote Total moved into the action rail, weight-900 tier removed, "Secure Transaction" restored, disabled contrast documented, `DEL-14` verbatim, `SH-51` leads with the product name, deck heading on the no-rep card, straight apostrophes, `V3-27` reachable, British spellings fixed.
- 2026-09-03 **R3 — request surface replaced, delivery and contact details moved to the quote page.**
  - **Removed Options A and B entirely** — the editable request page, the fixed 640px summary modal, the shared contact card with five fields, the in-surface delivery card, the request-page price rail and its mobile bottom bar, and every selector, template and handler belonging to them. No dead code: the file contains no "pcs", no "Request page (A)" and no "Imprint method".
  - **Added Option C**, a 480px user-initiated modal: thumbnail, one-line configuration summary in words (`C-02`), "Change" plus helper (`SH-11`, `SH-12`), amber default-decoration notice (`SH-20`), total with unit price (`C-05`), "See price breakdown" (`C-06`) holding `SH-51`–`SH-59a`, full name and work email (`SH-31`–`SH-33`), privacy line (`SH-45`), one CTA (`SH-62`, `SH-63`), promise line (`C-08`), close (`C-11`). Full-screen sheet below 960px.
  - **Added Option D**, a 640px modal with a pinned footer: production's price-tier table with paired `Quantity:` / `Unit Price:` fields and the MOQ note, the decoration-method card radiogroup with its `Decoration Charges` table, the imprint-location control in the dashed imprint box with the read-only `Imprint Colors` row and `Imprint #1 cost:`, read-only colour and imprint-colours rows with helper `D-23`, the production `Subtotal with Setup Fee:` box holding the breakdown disclosure, and Total plus CTA in the footer. Live recompute, `SH-66` announcements, dependency notices `D-20`, `D-21`, `D-22` on a live region, `SH-22` below the minimum, specialist state above the top tier.
  - **Reconciled D against the production capture** once it landed: subtotal box label and figure, method-card computed CSS and `aria-pressed` semantics, `Full Color` title case, `Edit Color & Quantity` recap label, breakdown disclosure moved into the box. Four corrections, listed above.
  - **Added the two quote-page modules.** Calculate delivery (`QD-01`–`QD-05`, wrapping `DEL-03`–`DEL-17` unchanged, including the captured empty state) and Add details (`QD-10`–`QD-14`, holding `SH-34`–`SH-38`, `SH-43`, `SH-44`). Both add a dated revision stamp, keep the quote number, update the total, the trust line, the "Prepared for" block, the PDF and the re-send email.
  - **Vocabulary pass across all six views**: "imprint method" became "decoration method" on the quote page, the PDF and the email; "pcs" became "pieces" in `SH-22`, `SH-51`, `V5-01` and every generated string; the email's single "Imprint" row split into "Decoration method" and "Imprint".
  - **Quote page now arrives with delivery not calculated**, at $692.50 with trust line `SH-59a`. It reaches $731.10 only when the buyer uses the module. The PDF footer gained `V4-06`.
  - QA at 1280 and 375 fixed four breaks found by screenshot: the modal annotation panels both displayed at once because `body.show-ann .ann-list` out-specified `.modal-ann{display:none}`; the annotation panels had no `top` and rendered off-screen; the prototype chip rail painted over the full-screen sheet at 375; and the sheet started under the sticky prototype bar. Print behaviour was verified by computed style against an always-on copy of the print stylesheet, not by eye.
  - Restored `SH-45`, dropped when the contact card was reduced to two fields (B-CHK025).
  - Added "Change delivery" so a chosen shipping option is not a dead end.
