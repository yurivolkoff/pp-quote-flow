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
6. **Where the captured production labels conflict with the analysis document, production wins.** Applied in a second pass once the capture landed: the price vocabulary, the location list, the method names, the step numbering and the location-reset behaviour all come from production now, not from the analysis. What changed is listed under "What the production capture changed".

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

The capture section in `promotionpros-pdp.md` landed after C and the quote-page modules were built. Reconciling against it, under the product-owner rule that captured production labels beat the analysis document, produced nine corrections. All are applied.

1. **The price vocabulary is production's, on every surface.** The flow had used the copy deck's `SH-51`–`SH-59` labels. It now uses the labels from production's own **Price breakdown** dialog, line for line: `Base Item Price` with `qty ×` and the unit price, the label-only rows `Decoration method: {Method}` and `Imprint 1 — {location}, {n}-color {Method}`, then `Imprint Method Setup` and `Imprint Method Run Charge`, a divider, `Total Price`, `Average price per unit`, and the footnote "Taxes and delivery costs are not included. They will be calculated at checkout." The quote page, the PDF and the email carry the same lines, with `Subtotal with Setup Fee` before the shipping row and `Total Price` after it.
2. **The disclosure is `View Price Breakdown`**, production's own link label, not the deck's `See price breakdown`. Used in C, in D and on the PDP.
3. **The collapsed summary uses `Subtotal with Setup Fee:` and `Price each:`.** These are production's short labels for the same figure the deck called Total. C's summary block and D's pinned footer both carry them, so the flow never invents a second word for a number production has already named.
4. **The subtotal figure is not goods-only.** The analysis treated production's box as excluding setup and run charges and built D's footer strategy around not redefining that word. The capture shows `Subtotal with Setup Fee: $330.25`, identical to the dialog's `Total Price`. There is therefore one money figure in D, not two, and no reuse risk to mitigate. The separate in-flow subtotal box was dropped: the pinned footer **is** production's collapsed block.
5. **The location list is production's**: `Back`, `Front (small)`, `Front (large)`, `Two Sided`, `No Imprint`, offered identically by every method. The mock product's own Barrel / Clip / Cap vocabulary is gone, and the deck strings that named "barrel" now name "Back".
6. **Method names are production's**, in its past-participle Title Case: `Pad Printed`, `Screen Printed`, `Laser Engraved`, `Full-Color Dye Sublimation`. Running prose keeps a gerund form ("pad printing on the back"), because "Pricing covers Pad Printed on the back" is not a sentence.
7. **Production resets the imprint location on every method change**, even when the new method offers the old one — confirmed by the URL's `l=` param changing. D now reproduces that reset rather than diverging from it, and announces it in the info band above the footer, which is the only part of the behaviour the flow adds.
8. **The step numeral badges are back.** D runs production's own sequence: `1 Colors` (read-only), `2 Quantity`, `3 Decoration Method`, `4 Imprint Details`. The analysis had dropped the badges on the argument that "3" and "4" without "1" and "2" is incoherent; restoring all four resolves that rather than trading it away.
9. **The controls themselves.** The method control is a `role="radiogroup"` of plain `<button aria-pressed>` cards, hand-authored BEM (`imprint-method-card`), not Tailwind — computed CSS reused verbatim at 8px radius, unselected `1px solid #EEEEEE` on white at `12px 16px`, selected `2px solid #FE5000` on `rgba(254,80,0,.05)` at `11px 15px`. The location control is a `react-select` combobox, reproduced as a native select. The `Imprint Colors` row is a read-only computed label, as built. Production's per-method `Decoration Charges` table with its `First Location` / `Extra Location` rows is reproduced, including the green-check "First imprint included" note for a method whose first location is free.

**Also added from the capture:** the Pantone/PMS control. Production exposes it on the PDP, not in a quote surface, so the prototype puts the checkbox in the PDP's decoration step and reports it read-only in C, in D, and on the quote, the PDF and the email. The capture found that ticking it did not move the live subtotal, so it is priced at proofing rather than at configurator time, and the flow says so.

---

## Two spec amendments still in force

1. **No free-shipping threshold exists on production.** Every free-shipping string is removed: `SH-60`, `SH-61` and the `DEL-13` free row are not built. Shipping reads "Calculated at checkout" until the delivery module is used.
2. **The live shipping-option rows could not be captured.** The production rate call returned "No delivery options found" for the test order, twice, so the radio rows are built from the deck and are **mock, visually unverified**. The captured empty state is a reachable state carrying `DEL-14` verbatim.

---

## Decisions where the analysis was silent

1. **C has no pinned footer.** The analysis specced a pinned footer for D only. C runs under 500px tall, so its CTA sits in flow; at 375 it lands just at the fold rather than above it, which is the one place C is slightly worse than the inline panel the analysis preferred.
2. **One breakdown string across the flow.** Production ships both "See price breakdown" (apron) and "View Price Breakdown" (tumbler) for the same control. The flow uses `View Price Breakdown` everywhere, so C, D and the PDP agree. A Boris ask asks production to normalise.
3. **The method card carries a one-line constraint summary** ("Up to 4 colors · 5 locations"). Production's card is name-only, because the constraints live in a Decoration & Imprint accordion that D does not have. Without it, D's clamp notices arrive unexplained.
4. **The colour-count clamp above one colour needed a second string.** `D-21` is written for a method that prints in one colour. A method that clamps from four colours to three gets a parallel sentence in the same shape.
5. **`No Imprint` is a real priced state.** Choosing it drops the setup and run charges to zero, sets `Imprint Colors` to `None`, and removes those lines from the breakdown, rather than quoting a decoration the buyer just declined.
6. **The chosen delivery option can be changed.** The analysis gives the delivery module a confirmation and a re-send but no way back. "Change delivery" returns to the options list; without it, a mis-picked service is a dead end on a document that stays live for 30 days.
7. **The details revision stamp reads "Details added Sep 4, 2026"**, parallel to `QD-05`, which the deck defines only for delivery.
8. **The email carries the full breakdown**, not a short summary block, because the instruction was that the production breakdown lines appear on the quote page, the PDF and the email alike.
9. **The tier curve, the per-method charges, the rep contact details and the product image** are mock, carried from the R1 build. The location list and the method names are no longer mock: both are production's.
10. **The prototype's own chip rail floats above the scrim at 1280** so modal states stay switchable, and drops behind the full-screen sheet below 960px. Prototype scaffolding, not product.

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

**View 3 — Quote modal (D).** Production's step order, 1 Colors through 4 Imprint Details, with steps 2, 3 and 4 live:
- Quantity: click a tier column, or type. Under 100 → `SH-22`. Over 10,000 → specialist state mid-edit.
- Decoration method: four cards. **Change the location to Front (large), then pick Laser Engraved, and both dependency notices fire at once** — production discards the chosen location on any method change, so it returns to Back, and laser prints in one colour, so the count clamps from 2 to 1 and the run charge is removed (`D-21`, carrying `D-22` back to the PDP).
- Laser Engraved also shows production's green-check "First imprint included" note and its Free / Free first-location charges.
- Imprint location: the same five options for every method, as production offers them.
- `No Imprint` zeroes the setup and run charges and sets Imprint Colors to None.
- Full-Color Dye Sublimation is the call-for-pricing trigger.
- **Pantone/PMS** is ticked on the PDP, in the decoration step, exactly where production puts it. D then reports it read-only, and it flows to the quote, the PDF and the email. It is priced at proofing, not at configurator time.

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
| B-PDP019 | Colour swatches, not drop-downs | Pass with exception (re-run) | Product colour is a 44px swatch radiogroup. Imprint location is a drop-down in D — because production itself ships a `react-select` combobox for it, with the five options reused verbatim. Reusing production's control is the instruction; inventing a swatch picker for it would be the divergence the reuse map forbids. |
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
6. **D reproduces production's location reset on method change, and announces it.** Production performs the reset silently; the notice is the only addition.
7. **Production's step checkmarks are dropped**, the numeral badges and the dark step bar are kept.
8. **The "Add Additional Imprint Location" control is not built.** Multi-location quoting is out of scope for this round; production exposes it and it was not exercised in the capture either.
9. **The Pantone/PMS checkbox is exposed for every spot-colour method, not only for Screen Printed.** Production showed it under Screen Printed on the captured product and listed the same $50-per-colour charge in Laser Engraved's read-only accordion. Whether that is a per-product difference or a gap is unresolved; the prototype shows it wherever a colour count exists, which is the more consistent reading.
10. **The shipping-option rows are mock.** See amendment 2.
12. **The "Chat with us" fallback button is navy outline, not orange outline** as captured. Orange text at 14px is 3.3:1 on white; navy is 15.9:1.
13. **Product imagery is an inline SVG placeholder**, so the file stays self-contained.
14. **Annotation dots exist only on the PDP lead row.** Every other view carries a legend, and each modal carries its own legend beside the dialog at 1100px and wider.

---

## Open items for Yuri

1. **C or D.** The analysis recommends C and names the instrumentation that would let D earn its build: `quote_change_clicked` on "Change", with roughly 15% of opens returning to the PDP as the threshold. Both are built and comparable.
2. **Free-shipping threshold.** None exists on production as of 2026-09-03. Three copy strings and one delivery row depend on the answer and are currently absent.
3. **The location list and the method names now come from the captured tumbler, not from the pen the rest of the mock describes.** Applying the production labels flow-wide, as instructed, means a bamboo pen is quoted with Back / Front (small) / Front (large) / Two Sided locations. The vocabulary demonstration is right; the product fit is not. Either the mock product changes to a tumbler, or the location list becomes per-product data again. One line either way, but it should be a decision, not a leftover.
4. **Whether the two modules should be open to any token holder.** Built open, per the analysis: the same token already authorises "Order this quote", so gating a phone number would be incoherent. Edits are date-stamped.
5. **Named rep for anonymous viewers.** Both variants are behind a chip. The recommendation is a named rep only when the requester's email matches a HubSpot company with an owner.
6. **Placeholder contact details** — Terry Alvarez, 312-555-0148, 800-555-0100 and the West Bend address block — need real values before anything ships.
7. **Multi-location imprints** are a production feature this flow does not carry. Confirm "Add Additional Imprint Location" is out of scope for the quote.
8. **Whether Pantone/PMS matching is chargeable at quote time.** It is currently shown on the quote as "applied at proofing" with no money attached, because that is what the capture observed. If the $50 per colour belongs on the quote, the line already exists and needs only a value.

---

## Boris asks

Carried forward from R2, unchanged: tokenised quote URL that still resolves after expiry and after ordering; price lock persisting the full priced breakdown rather than a recomputation key; expiry flipping at midnight Central with "Get an updated quote" creating a new record; Q-deal created on generation with a `priceable` flag; named events rather than text-matched actions; PDF as a server-side headless print of the same page; delivery rates reusing the checkout call and caching results on the quote record; priceability as a server-side rule evaluated on every re-price; transactional email with reply-to set to the assigned rep; the near-illegible disabled-button treatment on live checkout; whether the free-shipping threshold is really gone; and why the rate call returned nothing twice for a real test order.

New in R3:

1. **One shared configurator state store.** The PDP and D must read and write the same state, not two copies. The capture confirms production already keeps it in the URL: `?c=BLUE&s=Default+Size:25&im=Laser+Engraved&l=<encoded>`. Extend that scheme with the imprint-colour count and let D write to it, so the PDP re-renders from it on close.
2. **D imports the PDP's controls; it does not copy their markup.** The price-tier table, the `imprint-method-card` radiogroup and the `react-select` location control must ship as importable components. Copied markup is exactly the divergence D exists to prevent, and it is the reason two of D's controls carry a visible "reconstruction" note.
3. **Production silently resets the imprint location when the decoration method changes.** Confirmed on the tumbler: a location chosen under one method is discarded, and the URL's `l=` param changes. Please make the reset announce itself on the PDP too, the way `D-20` does in the quote flow.
4. **Normalise the PDP's own label inconsistencies.** `Imprint Colors` sits beside `Imprint location`; `Subtotal:` on one product is `Subtotal with Setup Fee:` on another; `See price breakdown` on one is `View Price Breakdown` on another. The flow standardises on the tumbler's strings; production should pick one set.
5. **Fix the price-tier table clipping** at the right edge of the ~520px buy box, where the top quantity column is cut off.
6. **Both quote-page modules must add a dated revision to the existing quote, never a new number.** Delivery and contact details are additive and leave the quoted goods price untouched. Only a configuration change issues a new number, carrying `V3-28` "Replaces quote Q-48213".
7. **Both modules are open to any token holder**, including a forwarded approver, with no extra gate.
8. **The PDF regenerates on every module save**, and its footer states its own generation date (`V4-06`) beside the always-current link. A module save sends no email on its own; `QD-04` re-sends the same template with an "Updated:" subject prefix.
9. **Confirm the Pantone/PMS pricing rule.** Ticking the box did not move the live subtotal in the capture, so the flow prices it at proofing. If the $50 per colour is meant to price at configurator time, the quote and the PDF need the value.
10. **Confirm whether the PMS control belongs to every spot-colour method or only to Screen Printed.** The captured product showed it under Screen Printed only, while listing the same charge in Laser Engraved's accordion.
11. **Serialize the full configurator state in the URL.** Production already writes `?c=&s=&im=&l=`; the quote's "Order this quote" and "Need changes?" links replay that string, so it needs the imprint-colour count and the PMS flag added to it.

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
| D, method card sub-line | Up to 4 colors · 5 locations (and per method) |
| D, location reset notice | {Method} resets the imprint location. It changed from {old} back to Back. |
| D, first-imprint-included note | First imprint included — one color, one location, no setup charge. Add locations below — extras are priced separately. |
| D, PMS read-only row | Exact Pantone/PMS color match — $50.00 per color, applied at proofing. Set on the product page. |
| PDP, PMS helper | $50.00 per color, applied at proofing. |
| D, colour-count clamp above one colour | {Method} prints in up to {N} colors. Imprint colors changed from {X} to {N}, and the run charge is recalculated. |
| Quote, PDF and email | Pantone/PMS color match — Applied at proofing |
| D, reconstruction note | Reconstruction, pending an import: this control is built from the 2026-09-03 production capture, not from the live component. |
| D, unit-price tooltip | The unit price comes from the quantity break you selected above. |
| Setup-charge tooltip | Covers preparing your artwork for this decoration method. Reorders of the same artwork skip it. |
| PDP, decoration-method note | One-time setup charge: $60.00 per location. Each imprint color past the first adds $0.35 per piece. |
| Email, re-sent confirmation toast | Updated PDF emailed to dana@acme.com |
| Email, re-sent lede | Here's the updated quote for 250 Del Mar Bamboo Stylus Pens, with the delivery you added. |
| Quote page, note under "Message us about this quote" | Opens a message prefilled with "I have a question about quote Q-48213." |
| Quote page, invalid-link recovery button | Back to promotionpros.com |
| PDF sheet, section headings | Prepared for / Product and configuration / Terms |

Carried from production verbatim, not written here: "Add to cart", "View Price Breakdown", "Base Item Price", "Imprint Method Setup", "Imprint Method Run Charge", "Total Price", "Average price per unit", "Taxes and delivery costs are not included. They will be calculated at checkout.", "I need an exact Pantone/PMS color match", "Back", "Front (small)", "Front (large)", "Two Sided", "No Imprint", "Pad Printed", "Screen Printed", "Laser Engraved", "Have a product question? Ask us", "Secure Transaction", "Edit Color & Quantity", "Key Facts", "250 pieces", "Minimum Quantity", "Price per unit", "Quantity:", "Unit Price:", "1 Colors", "2 Quantity", "3 Decoration Method", "4 Imprint Details", "Decoration Charges", "Setup", "Run Charge", "First Location", "Extra Location", "Imprint location", "Imprint Colors", "Imprint #1 cost:", "Subtotal with Setup Fee:", "Price each:", "Chat with us".

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
- 2026-09-03 **R3b — second reconciliation pass against the production capture**, under the product-owner rule that captured production labels beat the analysis document.
  - **Price vocabulary replaced flow-wide** with production's Price-breakdown lines: `Base Item Price`, `Decoration method: {Method}`, `Imprint 1 — {location}, {n}-color {Method}`, `Imprint Method Setup`, `Imprint Method Run Charge`, `Total Price`, `Average price per unit`, and production's footnote. Applied in C's disclosure, D's disclosure, the quote table, the PDF sheet and the email summary. The deck's `SH-51`–`SH-58` labels are retired.
  - **`View Price Breakdown` replaces `See price breakdown`** in C, D and the PDP. **`Subtotal with Setup Fee:` and `Price each:`** replace the flow's own Total wording in C's summary block, D's pinned footer and the PDP box. The quote page's rail and mobile bar now read `Total Price`.
  - **D restructured into production's four numbered steps** — `1 Colors` (read-only), `2 Quantity`, `3 Decoration Method`, `4 Imprint Details` — with the `Imprint Colors` read-only row moved under step 4 beside `Imprint #1 cost:`, where production puts it. The separate in-flow subtotal box was removed; the pinned footer is production's collapsed block.
  - **Location list replaced** with production's `Back` / `Front (small)` / `Front (large)` / `Two Sided` / `No Imprint`, offered identically by every method. `No Imprint` zeroes the decoration charges. Every "barrel" string in the flow became "Back".
  - **Method names replaced** with production's `Pad Printed` / `Screen Printed` / `Laser Engraved` / `Full-Color Dye Sublimation`, with a separate gerund form kept for running prose.
  - **Location reset now matches production**: any method change discards the chosen location and returns to Back, with a notice. The earlier keep-where-possible divergence is reversed.
  - **Per-method `Decoration Charges` reworked** to production's shape, including the green-check "First imprint included" note and Free / Free first-location charges for Laser Engraved against `$0.00` / `$1.25 per item` for an extra location.
  - **Pantone/PMS added** as a PDP control, reported read-only in C, D, the quote, the PDF and the email, priced at proofing per the capture.
  - Fixed a real bug found while testing the PMS path: submitting a quote never re-rendered the document, so the quote page, the PDF and the email kept showing the boot snapshot instead of the configuration just submitted.
  - Mobile fix: the long production label made the pinned footer wrap mid-phrase at 375, so the footer row stacks below 960px.
  - Print behaviour, console cleanliness and the full string audit re-verified after the rework.
