# Axie Grow Laboratory: Concept Refinement & Approach Options

**Version:** 0.2
**Date:** 2026-06-09
**Status:** Concept proposal, post-falsification. Numbers reference the companion doc `axie-grow-lab-falsification-model.md`.

---

## 1. The Concept

An in-game laboratory where players attempt to grow the exact Axie they want for their goals (competitive build, progression unlock, collection). Output is **soulbound**: lab axies can never be sold, traded, rented, or transferred. The lab is fired either by paying or by burning soulbound axies earned through gameplay, creating a recurring spend loop and a deep asset sink.

**Design pillars (non-negotiable):**

1. A player can always wish for an exact axie, but can never guarantee it.
2. Soulbound in, soulbound out. No cash-out surface, ever.
3. Free players get a real path; paid players get cadence, not certainty.
4. Every failed roll is partial progress, never a dead end.

**Why this helps rather than hurts the on-chain economy:** lab output competes with nothing on the marketplace directly, and every lucky exact roll of a meta genome is an advertisement for breeding that genome on-chain. The honest caveat: the lab does absorb player-side demand for mid-tier "good enough" tradable axies, so marketplace impact must be monitored, not assumed away (Section 6).

---

## 2. Approaches Considered

### A. Catalyst Genetics (recommended)

Burned axies are genetic material, not just fuel. The lab samples each of the 6 part slots from a gene pool weighted by what the player fed it, using classic inheritance ratios (D:R1:R2 = 12:3:1). Feed it axies sharing parts with your target and those genes get heavier, up to a hard ceiling.

Why it wins: the "burn more for better odds" instinct becomes mechanically meaningful instead of a flat percentage bump, it creates gameplay demand for specific soulbound axies (a second loop feeding the first), and it gives free players a genuine grind expression. It is also the only approach that survived falsification without major surgery.

Two repairs were required to make it safe:

- **Lab entropy floor.** Naive gene pools let players assemble purebred-gene catalysts and hit 100% exact, breaking pillar 1. Every slot therefore carries a fixed 50% wild-gene mass that catalysts can never displace. Result: ordinary catalysts give ~0.28% exact (1 in 360), perfect catalysts cap at 1.56% (1 in 64). The gap between those two numbers is the entire investment-skill expression of the system, and even a perfect player misses 63 of 64 activations.
- **Boost applies to gene mass, never per-slot probability,** with the 50% per-slot cap enforced after boosting.

### B. Virtual Breeding Sim (rejected as standalone)

Player fills D/R1/R2 slots like designing a phantom parent pair, classic probability tables apply. Instantly legible to Axie veterans and fully transparent, but it contains a guarantee exploit: author the target into all six gene slots and exact probability is 100%. Constraining slot authorship to fix it collapses the design into Approach A. Keep the breeding-sim *presentation* (players recognize it) on top of catalyst mechanics.

### C. Iterative Mutation Chamber (viable as a v2 layer)

Start from a burned base axie, pay to reroll individual parts and watch it converge. More spend events and more agency, but it converges to a de facto guarantee unless rerolls carry instability (a chance to scramble an adjacent locked part). With instability it becomes a strong push-your-luck layer. Recommend shipping the catalyst lab first and evaluating this as an expansion, since it doubles the tuning surface.

### D. Wish Banner with Near-Miss Bias (rejected)

Declare a full target, every roll has a tiny exact chance plus a distribution biased toward partial matches. Cleanest "wish" UX, but it is the most loot-box-legible design to regulators and offers the least player agency. Approach A delivers the same wish fantasy with stronger defenses.

---

## 3. Core Loop (Approach A)

1. **Register a wish.** Player picks the 6 target parts. Class is selected deterministically as part of the activation (this shrinks the RNG surface and strengthens the "not pure chance" argument).
2. **Load the chamber.** Burn 5 soulbound axies minimum. Catalysts sharing target genes raise odds via the weighted pool.
3. **Overload (optional).** Extra burns beyond 5 add a boost with diminishing returns (1.35x at +1, ~1.97x at +5, hard cap 2.0x, saturating at about 6 extras). The saturation must be visible in the UI; hidden caps read as a scam once spreadsheeted.
4. **Roll.** Lab produces a soulbound axie. Exact match is the jackpot; partial matches are the product.
5. **Recycle.** Any lab output can be burned as a catalyst for the next attempt. The entropy floor makes this recursion provably safe (gene concentration converges to the cap and stops), so every failure is inventory for the next wish.

**Free path:** soulbound axie earn rates tuned to roughly 1 free activation per week, gated behind progression milestones rather than raw playtime (anti-bot). A free player has an expected shot at a 4-of-6 match within about 7 rolls and a real weekly lottery moment, while exact-chasing remains functionally paid.

**Paid path:** direct activation purchase (~$5 equivalent as a starting point) and/or purchasable synthetic catalysts. Money buys attempts and gene weight, never gene locks.

---

## 4. The "Never Guaranteed" Pillar, Made Survivable

Zero mercy fails its own stress test: at baseline odds the 95th percentile player needs ~1,076 rolls for an exact hit, a five-figure zero-payout story that invites regulatory and community blowback. Full pity violates pillar 1. The surviving middle:

**Partial pity.** After every 25 activations against the same registered target, the lab permanently guarantees one additional matching part on every subsequent roll for that target, capped at 3 locked parts (resets if the target changes). The floor rises, the ceiling never closes: even at full pity with perfect catalysts, exact probability tops out at 12.5% per roll. The wish stays a wish forever, but the worst case is bounded and the long grind visibly compounds.

---

## 5. Pricing Anchor

The intuitive anchor (expected cost to the exact axie should undercut the tradable equivalent's price) fails: it forces activation prices below $0.50 and guts the sink. The correct anchor is the **viable axie**. A 5-of-6 match is competitively usable in most metas and arrives in an expected 9 to 33 rolls depending on catalyst quality. Tune activation price so expected cost to a 5/6 match lands at roughly 0.6 to 0.9x the marketplace price of the comparable tradable axie. At a $5 activation, that means viability for roughly $46 to $164 expected, with the exact axie remaining a fat-tail jackpot ($320+ expected, uncapped tail softened by pity). Most delivered value flows through near-misses; the exact roll is the dopamine event and the marketing moment.

---

## 6. Risks & Pre-Launch Kill Metrics

- **Marketplace cannibalization.** Falsifiable prediction: mid-tier tradable floors (60th-90th value percentile) soften because the lab serves exactly that buyer, while purebred and breeding-stock prices hold or rise. Define before launch: if mid-tier median sale prices drop more than 25% versus pre-launch baseline (token-price adjusted), activation pricing is too cheap and must rise. Do not launch without this dashboard.
- **Regulatory.** Paid RNG with published odds is loot-box shaped. Defenses in order of strength: airtight soulbound output with zero transfer/rental surface, a productive free path, deterministic class plus partial pity, full in-client odds disclosure (mandatory on Apple/Google regardless). One "lend your lab axie" feature converts this into gambling in Belgium and the Netherlands overnight. Counsel review required before paid launch.
- **Multi-account farming.** The lab adds a reason to bot parallel accounts for free rolls. Mitigated by progression-gated earn rates and soulbound output, but flag to the anti-bot team.
- **Catalyst supply drift.** If specific soulbound axies become reliably farmable, everyone's effective odds drift from baseline toward the cap, compressing the investment expression. Needs a supply-side simulation once earn tables exist.

---

## 7. Open Questions

1. Real part-pool sizes per slot per class (shifts the wild-gene math slightly).
2. Whether body/eyes/ears join the RNG surface or stay deterministic with class.
3. Synthetic catalyst pricing and whether they carry genes or only mass.
4. Mutation Chamber (Approach C) as a v2 layer once the base loop has live data.
