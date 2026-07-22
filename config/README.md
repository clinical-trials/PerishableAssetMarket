# Regulatory Assumptions — how this works

`assumptions.yaml` is the **single source of truth** for every uncertain regulatory
constant this product depends on (expiration windows, clock directions, minimum
bidder counts, transfer-eligibility rules, and the two load-bearing legal positions).

**Rule:** no day-count, window, or bidder-count is hard-coded anywhere else. The
Rulebook engine and Countdown engine read these values from this file. Change a
value here → the whole system updates.

## These are assumptions, not law
Every entry is a working assumption we build on **until it is verified**. Nothing
here is legal advice. Each value must be confirmed against the cited authority (and
with bond counsel) before any production or real-money use.

## Status of each entry
| status | meaning |
|---|---|
| `ASSUMED_UNVERIFIED` | working guess — needs confirmation (the default) |
| `USER_ASSERTED` | stated true by the domain owner; still needs counsel sign-off |
| `DESIGN_OPEN` | a product choice not yet made |
| `VERIFIED` | confirmed against authority — safe to rely on |

## To update an assumption (safe for a non-engineer)
1. Edit the `value*` field.
2. Change `status` to `VERIFIED` once confirmed.
3. Add `verified_by`, `verified_on`, and `source_note` with the citation.
4. Leave `verify_against` in place as the pointer to where it was checked.

## Current verification debt (as of 2026-07-20)
Still `ASSUMED_UNVERIFIED` and load-bearing:
- `windows.carryforward` — **3 vs 4 years** discrepancy to reconcile (IRC §146(f))
- `windows.recycle` — ~6-month window (IRC §146(i)(6))
- `transfer_eligibility_rules.recycled_authority_issuer_interest` — the "issuer-has-interest" rule
- `clocks.deploy_window` + `clocks.gic_bid_completion` — the **two clock directions** to reconcile
- `auction_rules.gic_min_bidders` — safe-harbor minimum (Treas. Reg. §1.148-5(d)(6))

Plus the two `USER_ASSERTED` legal foundations (cross-issuer transfer; software operates the auction)
that need counsel confirmation before the Series A.
