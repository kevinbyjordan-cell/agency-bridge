# PROPOSAL — ADS — PHRASE keyword expansion (Search campaign)

- **agent:** ADS (Google Ads)
- **date:** 2026-06-26
- **status:** `proposed`  ← human-only flips to `approved` (I never apply this myself without an explicit OK)
- **campaign:** Pet Tooth Fairy - Mobile Dental - Clermont FL - Search (`customers/7493280264/campaigns/23960290486`)
- **plan file:** `GOOGLE ADS PRO/clients/pet-tooth-fairy/ads/expand/phrase-expand.json`

## Why
The active Search campaign is **12 of 14 keywords EXACT** (narrow). Volume has been **thin ~4 days**
(~2 paid clicks/day) and there are **0 confirmed paid conversions** post-fix — not because the funnel is
broken (sample is too small), but because **we don't have enough paid clicks to measure**. The bottleneck
is reach, not cost.

## What changes
Add **2 new ad groups of PHRASE keywords** (broader than EXACT → captures long-tail variants the EXACT terms miss),
each with one RSA reusing the existing approved copy:
- **Dog Dental — Phrase** (10 PHRASE kw): dog teeth cleaning, dog dental cleaning, dog teeth cleaning near me,
  mobile dog teeth cleaning, anesthesia free dog teeth cleaning, dog dental cleaning without anesthesia,
  mobile dog dentist, dog dentist near me, pet teeth cleaning, dog teeth cleaning cost.
- **Cat Dental — Phrase** (4 PHRASE kw): cat teeth cleaning, cat teeth cleaning near me, mobile cat teeth cleaning,
  anesthesia free cat teeth cleaning.

Dry-run validated live: **18 mutate ops, created NOTHING**.

## Why it's safe (spend-neutral)
- **One shared budget ($30/day) is unchanged** → adding keywords CANNOT raise spend; it only changes *which queries*
  we show on. Total spend stays ≤ budget (cloud watchdog still catches >2×).
- **$5 CPC cap still applies** to every click.
- **53 negative keywords** already filter junk; I'll mine new search terms after a few days and add negatives for any waste.
- New entities go ENABLED on apply (they serve) — that's the point (more reach). Your approval here is the spend gate.

## Apply (only after you approve)
`node bin/gads.js campaign expand --client pet-tooth-fairy --campaign "Pet Tooth Fairy - Mobile Dental - Clermont FL - Search" --plan <abs path>/phrase-expand.json --apply`

## Rollback
Pause (or remove) the 2 new ad groups via `campaign set-status` / the UI. EXACT keywords are untouched.

## Decision
- [ ] approved by owner → ADS applies + reports the new reach/clicks/CPC over the next days
- [ ] rejected / changes requested
