# ECR Offer Builder — Outlook add-in

An Outlook button that lets a closer build a subscription offer in the side panel and insert a
clean, negotiation-friendly **pricing block** into a prospect email. Report names in the block
become personalized access links (the prospect's email is the access key), the same way the
Trial Links add-in works. This is the sibling of **ECR Trial Links** — same architecture,
hosting and gotchas.

Hosted at: `https://peetbos2003-sudo.github.io/offerbuilder/`

---

## What the panel does

When composing an email, open the **ECR Offer Builder** panel (compose ribbon -> Apps / "...").

You set the offer **once**, then pick **what you're comparing**; the panel builds the block.

- **Prospect email** — type it, or click **Use recipient** to pull it from the To field. It is
  the access key that makes each report name a personalized login link.
- **Prospect name / Offer valid until** — both optional. The validity date adds a single
  "This offer is valid until ..." line under the block.
- **What are you comparing?** — three presets:
  - **Single offer** — one clean priced offer (renders as a soft price table).
  - **1 yr vs 2 yr** — same package, two terms side by side; the 2-year column gets the 10%
    multi-year discount automatically.
  - **Full vs smaller** — full subscription vs a cheaper smaller package (same term), so the
    prospect can trade scope for price. A second "Smaller package" picker appears.
- **Package** — Full subscription, Macro & Markets bundle, Asset Allocation bundle, or a
  **Custom selection** (tick individual reports, auto-priced). The **List** field is editable.
- **Total licences / Of which free / Years** — enter the **total** user licences the customer
  gets, then **how many of those are free**. Each extra licence costs 10% of the package fee; the
  free ones are shown at full value in the List and taken back off as a quantified discount line,
  e.g. "Free user licences (2) -EUR 4,000" on a full sub (2 x 10% of EUR 20,000). The block reads
  "3 user licences, of which 2 included free of charge". Free is capped at total minus one (the
  base licence is always paid). Years drives the term (hidden in the 1-yr-vs-2-yr preset).
- **Apply 10% multi-year discount** — available in every mode except 1-yr-vs-2-yr (which already
  applies it to the 2-year column). Ticking it applies the 10% and sets a 2-year term; it
  auto-ticks when you set 2+ years.
- **Negotiated discount EUR/year** — optional, on top of the multi-year discount.
- **Include clickable report links** — on by default; lists the included reports as personalized
  links under the offer. Untick for a price-only block.
- **Insert offer block** / **Copy offer instead** — drop the block (email-safe HTML) at the
  cursor, or copy it to paste manually.

Two-option presets render as a borderless A/B comparison (Option 3); single offers render as the
soft price table (Option 2). Both keep the total line and the included-reports summary.

---

## Pricing model (baked into the add-in)

All figures are list price per year and live in the `PRICING` config near the top of
`taskpane.html`. A price change is a quick edit + redeploy.

- **Full subscription:** EUR 20,000 / year (all 10 reports, 1 user), with a standing
  **EUR 2,000 entry discount** applied automatically the moment you go full subscription
  (20,000 -> 18,000/year). On a 2-year term the multi-year 10% comes off **on top** (another
  EUR 2,000), reaching EUR 16,000/year. Both discounts show as separate labelled lines. The entry
  discount is full-subscription only; bundles and custom selections do not get it. The **List
  field is editable** if you ever want to show a different headline figure.
- **Macro & Markets bundle:** EUR 10,000 / year.
- **Asset Allocation bundle:** EUR 10,000 / year.
- **Additional user licences:** each added user adds **10% on top of the base offer price**
  (auto-applied to the List line). So 3 users on a EUR 20,000 base = 20,000 + 2 x 10% = EUR 24,000.
- **Free extra seats (bargaining chip):** the **Free extra seats** field waives the 10% on that
  many of the additional licences, so a closer can throw in seats at no cost during negotiation.
  The inserted block spells it out to the prospect, e.g. "3 user licences (2 additional seats
  included at no extra cost)". Example: 3 seats with 2 free = price stays at the EUR 20,000 base.
- **24-month enticement (term discount):** a **10% term discount** auto-applies on a 2-year
  (or longer) contract and shows as its own labelled line so the prospect sees why the longer
  term pays off, e.g. "List EUR 20,000/year -> 24-month saving EUR 2,000 (10%) -> net
  EUR 18,000/year". It **stacks** with any further negotiated discount the closer types. The
  checkbox auto-ticks from 2 years and can be turned off per option.
- **Custom selection:** **each flagship** report (Global Financial Markets or Tactical Asset
  Allocation) is **EUR 6,000** — so two flagships = **EUR 12,000**. Each **satellite**
  (non-flagship) report adds **EUR 1,000**. A selection with **no flagship** starts at
  **EUR 3,000** for the first report + EUR 1,000 per further report.
  - Examples: GFM alone = 6,000. GFM + TAA = 12,000. GFM + 2 satellites = 8,000. Two flagships +
    3 satellites = 15,000. Three non-flagship reports = 5,000.

Discounts are never hardcoded — the closer enters the discount amount (or types a target net),
and the block renders List -> Discount -> Net/year -> Total.

---

## Files in this package

- `manifest.xml` — the add-in definition (add-in Id; DisplayName "ECR Offer Builder"). Points at
  `taskpane.html`. **Never needs changing or re-uploading** unless the host/Id changes.
- `taskpane.html` — a tiny **permanent redirector**. Outlook caches it hard, so it must stay
  stable: its only job is to bounce to `app.html?ts=<timestamp>` so the real app is always
  fetched fresh. **Do not put logic here.**
- `app.html` — the panel UI, pricing config and all logic. **This is the file you edit and
  redeploy for every update.**
- `commands.html` — required init file.
- `icon-16/32/64/80/128.png` — button icons (live at the repo root, not in an assets/ folder).

(No `latest.json` here — prices live in the add-in, not in a hosted data file. If prices start
changing often, the optional upgrade is to move `PRICING` to a hosted `pricing.json`, mirroring
how Trial Links reads `latest.json`.)

---

## Deploying an update (IMPORTANT — read this)

Thanks to the redirector, **routine updates need only one file and no reinstall**:

1. Edit **`app.html`** (the panel UI/logic). This is the only file that changes for normal updates.
2. In the `offerbuilder` GitHub repo, replace **`app.html`** with your new version (drag it in,
   overwrite, commit). Keep it at the repo **root**.
3. Wait ~1–2 minutes, then hard-refresh `https://peetbos2003-sudo.github.io/offerbuilder/app.html`
   to confirm the new version is live.
4. **In Outlook, just close and reopen the panel** — the redirector fetches `app.html?ts=<unique>`,
   so Outlook always pulls the newest copy. No remove/quit/re-add, no manifest re-upload, no
   touching colleagues' laptops.

### How it works (cache-busting)

`manifest.xml` points at `taskpane.html`, which is a permanent redirector that bounces to
`app.html?ts=<timestamp>`. Because the timestamp makes the URL unique every time the panel opens,
neither GitHub's CDN nor Outlook's hard add-in cache can serve a stale copy — the newest `app.html`
is always fetched. `taskpane.html` and `manifest.xml` never change, so they never need
re-uploading or reinstalling.

### One-time setup (only when first switching to this structure, or changing host/Id)

Upload **all** files to the repo root from the zip (`taskpane.html`, `app.html`, `manifest.xml`,
`commands.html`, the five icon PNGs), then do the Outlook remove → fully quit → reopen → re-add
once so it picks up the redirector. After this one time, only step 1–4 above are ever needed.

---

## Installing / refreshing in Outlook

Custom add-ins are stored in your **mailbox** and sync across your devices (Mac, Windows, web).

**To add it (while sideloading is allowed):**
Outlook -> New email -> Apps / "..." -> Get Add-ins -> My add-ins -> **Custom Addins** -> add the
manifest (URL: `https://peetbos2003-sudo.github.io/offerbuilder/manifest.xml`, or upload the file).

**To pick up a new version (Outlook caches add-in pages hard):**
1. Remove the add-in from Custom Addins.
2. **Fully quit Outlook** (Cmd+Q on Mac / fully close on Windows) and reopen.
3. Add it again.
A "broken link" or an old-looking panel is almost always this cache — the remove/quit/re-add
clears it.

**Privacy setting:** "Optional connected experiences" must be **ON** — turning it off greys out
all add-ins.

---

## Team rollout (recommended)

Have your **M365 admin** deploy it centrally: Microsoft 365 admin center -> Settings ->
**Integrated apps** -> Upload custom apps -> Office Add-in -> "Provide link to manifest file",
using the hosted `manifest.xml` URL, assigned to the closers. This pushes it to everyone
automatically and removes the sync/duplicate fragility of manual sideloading. See
`IT deployment request — ECR Offer Builder add-in.md` for a ready-to-send note. Our tenant
**disables user sideloading**, so central deployment is the route for the team.

---

## Email-safe HTML

The inserted block uses table-based layout, inline styles and web-safe fonts (Arial/Helvetica)
because Outlook ignores rounded corners, shadows, web fonts and flexbox. No em-dashes appear in
the prospect-facing block. Keep it that way when editing.

---

## Editing the report list or prices

- **Prices:** the `PRICING` object near the top of `taskpane.html`.
- **Reports / slugs / flagship flags:** the `REPORTS` array just below it. Slugs are verified
  against ecrresearch.com. Flagship reports are marked `flagship:true`.
  (Global Political Analysis is excluded — being phased out for new prospects.)
