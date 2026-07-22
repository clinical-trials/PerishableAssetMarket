# PerishableAssetMarket

**Working name: CapStack.** A compliant competitive-bidding marketplace for **expiring
tax-exempt bond capacity** — matched into shovel-ready affordable-housing deals *before the
clock runs out*. "eBay for the perishable asset."

The moat is **encoded compliance + timing**, not a directory of public PDFs: the software reads
each jurisdiction's rules, runs a legally defensible competitive bid, and back-schedules every
deadline from the bond's own dates.

## What's in this repo

| Path | What it is |
|---|---|
| `prototype/index.html` | Self-contained interactive prototype — open in any browser, no build step. |
| `config/assumptions.yaml` | **Single source of truth** for every uncertain regulatory constant (status-tracked, citation-linked). |
| `config/README.md` | How the assumptions registry works and how to update it. |
| `Tax-Exempt-Bond-Bidding-Marketplace-Brief.pdf` | The full 6-page written brief. |
| `Tax-Exempt-Bond-Bidding-Marketplace-OnePager.pdf` | Email-ready one-pager. |

## The prototype

Open `prototype/index.html`. It demonstrates:

- **Wedge toggle** — Type A (banks bid loan rate) ↔ Type B (providers bid GIC yield); the whole UI vocabulary swaps.
- **Jurisdiction** — California / CTCAC live, plus the 50-states-+-territories expansion grid.
- **Countdown engine** — back-schedules from the bond's redemption anchor to the target close.
- **Rulebook engine** — a compliance checklist that reads the assumptions and blocks illegal/expired pairings.
- **Two clocks** — the forward deploy window and backward GIC-bid deadline, with the unresolved direction flagged honestly.
- **Assumptions** — every regulatory value, editable, with live recompute across the app.

Design patterns borrow from an open-source perishable-asset auction platform
([iragm/fishauctions](https://github.com/iragm/fishauctions)): proxy/max bidding, reserve prices,
timed auto-close, role-based fees, automatic invoicing, watchlists — with the deliberate
divergences of **no buy-it-now** and an added **compliance layer**.

## Regulatory status — read this

Every regulatory number here is a **working assumption until verified** (see
`config/assumptions.yaml`). **Nothing in this repo is legal, tax, or investment advice.**

**Source review (2026-07-21):** a full-text search of the 108-page **CTCAC regulations** confirms
that document governs the LIHTC *tax credit* and **defers bond/volume-cap timing to CDLAC**. The
federal windows (IRC §146(f) carryforward, §146(i)(6) recycling, Treas. Reg. §1.148-5 GIC
bidding) are **not** in the QAP. Remaining verification: the **CDLAC Regulations** + the three
federal cites.

**Locked positions:** cross-issuer capacity transfer → true marketplace; the auction is operated
under an in-house **SEC compliance expert who maintains an active license**.

The prototype uses **fictional demo data** (illustrative issuers, amounts, and dates).
