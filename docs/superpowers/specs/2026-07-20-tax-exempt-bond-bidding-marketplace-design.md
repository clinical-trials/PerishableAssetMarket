# Tax-Exempt Bond Bidding Marketplace — Design (DRAFT)

- **Status:** DRAFT — brainstorming capture, not yet approved
- **Date:** 2026-07-20
- **Working name:** TBD (placeholder: *CapStack*)
- **Author of record:** (you) — captured collaboratively with Claude

> ⚠️ **Read me first.** Every statutory number, day-count, and compliance mechanic
> below is marked **⚠️ VERIFY** where I am not certain. These are the load-bearing
> facts of the whole product — none should be treated as true until confirmed against
> the IRC / Treasury regs / the relevant state QAP or agency rules. I have not
> verified them; I've captured them as *design intent + open questions*.

---

## 1. One-sentence thesis

A **compliant competitive-bidding marketplace for tax-exempt affordable-housing bond deals** — it runs the *regulated bid events* across a bond's life, matches **expiring tax-exempt capacity** to **shovel-ready projects**, and enforces the **compliance clocks** so nobody wins an illegal, expired, or non-competitive deal.

The moat is **encoded compliance + timing**, not matchmaking. A directory of public NOFAs/QAPs is nearly worthless (the PDFs are already public); the value is the software that (a) reads the 100-page rules, (b) runs a legally-defensible competitive bid, and (c) back-schedules everything from a fixed bond date so the humans hit the deadline.

---

## 2. Domain primer (why this is hard)

**The parties (deal triangle):**
- **Issuer** — governmental / conduit issuer that *holds the tax-exempt capacity*. The capacity is on a clock.
- **Borrower** — the developer who borrows bond proceeds to **start the next project**.
- **Lender / bank** — actually funds the bonds (typically a direct purchase / private placement).

**The perishable asset — tax-exempt bond capacity:**
- Private-Activity Bond (PAB) **volume cap** is allocated per-state, per-year (roughly per-capita). ⚠️ VERIFY current per-capita figure.
- Financing **≥50%** of a project's aggregate basis + land with tax-exempt PABs auto-unlocks the **4% LIHTC** — no competitive QAP scoring fight. (The "50% test.") ⚠️ VERIFY exact test wording.
- **Carryforward:** unused cap can be carried forward and "captured." User's number: **within 4 years of original issue.** My memory: statutory carryforward is **3 years** (IRC §146(f), Form 8328). ⚠️ VERIFY exact window — the product's clock depends on it.
- **Recycling:** when a multifamily bond is repaid, the issuer can re-issue **"recycled" bonds without new volume cap** if done within **~6 months** (IRC §146(i)(6)). Catch: **recycled bonds do NOT generate fresh 4% credits** — they're cheap tax-exempt debt + cap preservation only. ⚠️ VERIFY 6-month window + "issuer has interest / borrower does not" constraint on who may hold the recycled authority.

**The economics that create urgency:**
- If tax-exempt capacity **expires unused**, the next deal must use **taxable** debt at ~**+50 bps** (user's figure; spread varies by market/credit). On a **$2.5M** issue that's ~**$12,500/yr** in extra interest — often the difference between a deal that pencils and one that dies.
- So the platform sells **saved basis points + capacity that would otherwise be wasted**, by assembling a closeable tax-exempt deal *before the clock runs*.

**The 50-state problem:**
- LIHTC is allocated by each state's Housing Finance Agency under its own **Qualified Allocation Plan (QAP)** — different scoring, set-asides, deadlines. **California's QAP (CTCAC) is ~100 pages**, rewritten annually; every state differs; county/city NOFAs (e.g., **LACDA**) stack on top.
- 50 states × ~100 pages × annual revisions + local NOFAs = the rules no human holds in their head. Structuring these into machine-readable rules is the defensible work.

---

## 3. Users & business model (who pays)

**Principle:** charge the side that captures the most *durable* value and transacts *repeatedly* — the **banks** — not the one-off small developer. Never tax the holder of the scarce asset (the issuer) — you want every dollar of expiring capacity *on* the platform.

| Party | What they pay | Rationale |
|---|---|---|
| **Issuer** (supply) | **Free** | They're the reason the marketplace exists; don't add friction to the scarce asset. |
| **Bank** (bidder) | **Subscription (annual seat) + success fee on won deals** | A single won $2.5M loan = 15–30 yrs of interest income; richest, most repeat participant. |
| **Borrower** (demand) | **Free / minimal early** | The liquidity you're courting; charge later via a small success fee, if at all. |

**Fee mechanics under discussion (user's raw notes):** "pay to have access, pay to bid, pay to win"; "$500 or $1000 to start the bid"; "banks pay subscription vs. pay-only-if-you-bid."

**Recommended stack:**
1. **Free supply** (issuers list capacity/deals).
2. **Bank subscription** (all-you-can-bid seat) — predictable revenue, the pros.
3. **Success fee** on the winning bank when a deal closes (flat or small bps of the issue; e.g., 25 bps of $2.5M ≈ $6,250) — aligns platform with *closed* deals.
4. **Refundable bid deposit ($500–$1000)** as an **anti-flake / seriousness gate** — returned on a genuine bid, forfeited if a winner walks. Preserves the user's "pay to bid" instinct **without** taxing the early liquidity a young marketplace is starving for.
5. **Pay-per-bid** deferred until liquidity is healthy (optional later layer).

> Open tension: charging to *bid* suppresses liquidity at cold-start. Resolution above = deposit-not-fee early, real fees once proven.

---

## 4. Core product — the two engines

Everything rides on two shared engines, reused by every auction type:

### 4a. Countdown / back-schedule engine (the heartbeat)
You never pick the deadline — **the bond does.** Given a fixed anchor date, the platform generates the working schedule and drives the parties to it.

**Worked example (user-provided):**
```
Apr 1  ── REDEMPTION (anchor) ── capacity freed / clock starts
Apr 8      platform posts the deal, notifies qualified bidders
Apr 22     bids due
May 6      award winner, generate compliance certificate
May 20     docs + counsel sign-off
Jun 1  ── TARGET: next deal closes / proceeds redeployed
```
- The engine sends nudges, tracks each party's task, and **blocks** an award that would miss a legal window.
- ⚠️ **Date-direction open question:** user said GIC bidding "complete 60 days *before* redemption," but the Apr 1 → Jun 1 example is 60 days *after* redemption. Likely **two different clocks** (a redemption-triggered recycle/deploy window vs. a bid-must-finish-before-X regulatory deadline). Must be pinned down before build.

### 4b. Compliance rulebook engine (the moat)
Ingests the 100-page rule sources → structured, queryable rules; validates every proposed pairing/award against them.
- **Sources:** state QAPs, agency NOFAs (LACDA/HCD/etc.), IRC/Treasury arbitrage + volume-cap rules.
- **Enforces:** eligibility (who may hold recycled authority), the 50% test, reserve/minimum, minimum # of bona-fide bidders, and every clock.
- **Never** lets the marketplace match an illegal or expired pairing.

---

## 5. Auction rules (the rulebook, so far)

- **Sealed competitive bids.** Bona-fide competition is mandatory (partly a compliance requirement of the arbitrage safe harbor).
- **Reserve / minimum bid must be met** or **no award** (no forced sale below reserve). Direction depends on auction type (see §6).
- **No "buy-it-now."** A fixed-price shortcut would blow the "bona fide competition" the safe harbor requires. Competition is the product.
- **Minimum number of bona-fide bidders** (⚠️ VERIFY — arbitrage GIC safe harbor typically requires ≥3 bids from providers with no material interest; Treas. Reg. §1.148-5(d)(6)).
- **Refundable seriousness deposit** to bid; forfeited on winner walk-away.
- **Compliance certificate** auto-generated on award (documents the bona-fide bid for the file).

---

## 6. The two auction types

### Type A — Loan-rate reverse auction  *(recommended v1 wedge)*
- **What's bid:** the interest **rate/terms** to fund a posted deal. **Banks bid; lowest rate wins.**
- **Value:** directly compresses the **+50 bps**; fits the "start the next project" story.
- **"Minimum bid" here** = a **reserve in the rate direction** (a maximum acceptable rate ceiling, or minimum bid-validity standards); unmet → no award.
- **Risk:** relationship-heavy; banks may resist disintermediation → harder cold-start.

### Type B — GIC / investment-agreement bidding  *(module 2)*
- **What's bid:** the **yield** on a Guaranteed Investment Contract for un-spent bond proceeds/reserves. **Providers bid; highest qualifying yield wins.**
- **"Minimum bid" here** = a **reserve/floor yield** the winner must at least meet (safe-harbor benchmark). Natural fit for the "minimum must be met" rule.
- **Why it's a tempting *alternative* wedge:** most standardized/rule-bound → easiest to make provably correct; recurs on **every** deal; there's already a **manual market (bidding agents)** that pays for it → clearer willingness-to-pay + a known incumbent to beat.
- **Downside:** it's the "plumbing," not the headline origination story.

Both auctions share §4a + §4b. Choosing the wedge decides which UI/flow ships first, not the architecture.

---

## 7. v1 scope (proposed) vs. roadmap

**v1 (the wedge) — assume Type A unless you redirect:**
- One auction type (loan-rate reverse auction).
- Countdown engine + compliance rulebook for **one jurisdiction first** (proposed: **California / CTCAC**, largest LIHTC market and the hardest 100-page QAP — proving it there proves it anywhere). ⚠️ Confirm CA vs. your home market (NM?).
- Bank subscription + refundable deposit + success fee.
- Manual/assisted rule ingestion for that one jurisdiction (LLM-assisted, human-reviewed) — not yet automated 50-state.

**Roadmap (later modules, not v1):**
- Type B (GIC bidding).
- Additional states (clone the ingestion pipeline; breadth after depth).
- Developer-side tooling / QAP fit-scoring.
- Capacity matching across a $100M issuer pool → many $2.5M deals.
- Full three-party deal-assembly workflow to close.

---

## 8. Data model sketch (first cut)

- **Issuer** — id, jurisdiction, capacity holdings.
- **Capacity** — type (fresh cap / carryforward / recycled), amount, jurisdiction, **expiry clock**, eligibility constraints.
- **Bond** — anchor dates (issue, redemption, maturity), status.
- **Project (Deal)** — borrower, location, unit count, AMI targets, size, readiness.
- **Auction** — type (A/B), reserve, min-bidders, open/close dates (derived by countdown engine), status.
- **Bid** — bidder, sealed value (rate or yield), deposit status, validity flags.
- **Party** — issuer / borrower / bank / GIC-provider, KYC/eligibility.
- **ComplianceRule** — source doc, machine-readable predicate, jurisdiction, effective dates.
- **Clock** — anchor date, rule, computed deadlines, notifications.
- **Award + ComplianceCertificate** — winner, generated evidence-of-competition.

---

## 9. Assumptions & OPEN QUESTIONS (please correct in review)

1. **Wedge:** defaulting to **Type A (loan-rate)**. Confirm — or switch to **Type B (GIC)** for a faster provable v1.
2. **Bidders in Type A** = **banks bidding rate**. Confirm it isn't developers bidding for capacity (different auction, different payer).
3. **First jurisdiction** = **California/CTCAC**. Confirm vs. NM or wherever your relationships are.
4. **Date direction:** are there two clocks (redemption-triggered deploy window *and* a bid-must-finish-before-redemption window)? Resolve the "60 days before" vs. Apr1→Jun1 "60 days after."
5. **Carryforward window:** 3 years (my memory) vs. 4 years (your note). ⚠️ VERIFY.
6. **Recycling:** 6-month window + the "issuer has interest / borrower does not" rule. ⚠️ VERIFY exact constraint.
7. **Min-bidders / safe harbor:** ≥3 bids? Which safe harbor governs each auction type? ⚠️ VERIFY.
8. **Who is customer #1** you can actually call next week — a specific issuer, bank, or developer? (Determines go-to-market, not architecture.)

---

## 10. Explicitly OUT of scope (YAGNI, for now)

- Automated 50-state rule ingestion on day one (depth-first on one state instead).
- Two-sided cold-start of the full capital marketplace.
- Developer QAP fit-scoring product.
- Anything that reproduces public PDFs as a "directory" (no moat).

---

## 11. Key risks

- **Regulatory correctness** — the whole value prop is compliance; a wrong clock or eligibility rule is a legal liability, not just a bug. → Human-in-the-loop review of every ingested rule; counsel sign-off gate.
- **Cold-start liquidity** — no bids = no marketplace. → Start where *one* issuer's pool creates its own liquidity; deposit-not-fee to bid.
- **Disintermediation resistance** (Type A) — banks/agents may resist. → Type B (GIC) has an existing paid manual market that's easier to convert.
- **Verification debt** — every ⚠️ above is a landmine if shipped unverified.

---

## 12. Next steps

1. You review this doc and correct §9 (the open questions) + any ⚠️ VERIFY facts you already know.
2. Lock the wedge (A vs B) and first jurisdiction.
3. Then: writing-plans skill → detailed implementation plan for v1.
