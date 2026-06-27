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

- **Prospect email** — type it, or click **Use recipient** to pull it from the To field. It is
  the access key that makes each report name a personalized login link.
- **Prospect name / Offer valid until** — both optional. The validity date adds a single
  "This offer is valid until ..." line under the block.
- **One option / Two options (A / B)** — present a single offer or a comparison, like Vladimir's
  live two-option layout.
- **Per option you set:**
  - **Contract label** (e.g. "2-year contract").
  - **Package** — Full subscription, Macro & Markets bundle, Asset Allocation bundle, or a
    **Custom selection** (tick individual reports, auto-priced).
  - **Years** and **User licences**.
  - **Discount EUR/year** or **Net EUR/year** — type either; the other recalculates. The block
    renders List -> Discount -> Net/year -> Total. Discounts are negotiated per deal, so nothing
    is hardcoded.
- **Include clickable report links** — on by default; lists the included reports as personalized
  links under each option. Untick for a pure price-only block.
- **Insert offer block** — drops the block at the cursor as email-safe HTML.
- **Copy offer instead** — copies it if you'd rather paste manually.

---

## Pricing model (baked into the add-in)

All figures are list price per year and live in the `PRICING` config near the top of
`taskpane.html`. A price change is a quick edit + redeploy.

- **Full subscription:** EUR 20,000 / year (all 10 reports, 1 user). The **List field is
  editable**, so a closer can show EUR 22,000 on a full sub to make the discount look larger.
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

- `manifest.xml` — the add-in definition (new add-in Id; DisplayName "ECR Offer Builder"). URLs
  point at the host above.
- `taskpane.html` — the panel UI, pricing config and all logic.
- `commands.html` — required init file.
- `icon-16/32/64/80/128.png` — button icons (live at the repo root, not in an assets/ folder).

(No `latest.json` here — prices live in the add-in, not in a hosted data file. If prices start
changing often, the optional upgrade is to move `PRICING` to a hosted `pricing.json`, mirroring
how Trial Links reads `latest.json`.)

---

## Deploying an update (IMPORTANT — read this)

Updates only go live when the files on GitHub are replaced. The reliable way:

1. Always start from the **latest zip** (`ECR-Offer-Builder-Outlook-Addin.zip`). Unzip it into a
   fresh folder. Do **not** upload from an old desktop copy — that's what causes "it reverted to
   an old version."
2. In the `offerbuilder` GitHub repo, upload the unzipped files, overwriting the old ones.
   You can **drag all files at once** (taskpane.html, manifest.xml, commands.html, and the five
   icon PNGs). Keep everything at the repo **root** — the manifest expects the icons at the root.
3. Wait ~2 minutes, then **hard-refresh** to confirm:
   `https://peetbos2003-sudo.github.io/offerbuilder/taskpane.html` (Ctrl+F5 / Cmd+Shift+R).
4. In Outlook, refresh the add-in (next section).

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
