# Artifact & Gear Compensation — Plain-Language Summary

## What's changing
We're removing artifacts and gear from the game. Players who spent time and resources grinding them deserve compensation in proportion to how much they invested.

## How we figure out who gets what
Every player has an inventory of artifacts and gear that the backend can read directly. From that we can tell:
- How many artifacts they own, of which types, at which levels
- How many gear pieces they own, of which types, at which levels, with what bonus values
- For each artifact, how many copies the player consumed reaching its current level

We combine these into one "investment score" per player and sort players into five tiers:

| Tier | Who it covers |
|---|---|
| 1 — Newcomer | Just started collecting |
| 2 — Collector | Moderate daily player |
| 3 — Enthusiast | Regular grinder |
| 4 — Veteran | Sustained, heavy investment |
| 5 — Whale | Top of the curve for grinding |

The exact line between each tier is set by where most players land — roughly bottom 30% are Newcomers, top 3% are Whales, with the middle bands in between.

## Five ways to pay players back

### Approach 1 — Flat gems per tier
Each tier gets a fixed gem grant. Same gems for everyone in the same tier.

**Why pick this:** easiest to communicate, fastest to ship.
**Trade-off:** the player just below a tier boundary gets noticeably less than the player just above it.

---

### Approach 2 — Proportional refund (no tiers)
Skip tiers entirely. Refund a percentage (e.g. 80%) of what each player demonstrably spent on crafting and upgrades, based directly on their inventory.

**Why pick this:** fairest mathematically — no tier cliffs, the player who spent twice as much gets twice the refund.
**Trade-off:** requires us to publish gem-equivalent rates for dust and SLP — and we'll be held to whatever rates we publish.

---

### Approach 3 — Tiered bundle
Same 5 tiers as Approach 1, but each tier gets a bundle instead of just gems: gems + free wheel spins + lottery tickets + (for top tiers) a commemorative sticker set.

**Why pick this:** more variety, more "wow" factor at higher tiers, recognizes high-investment players with something unique.
**Trade-off:** harder to communicate ("Tier 4 = 20 spins, 10 tickets, X gems, 3 stickers"). Commemorative items would be a new feature build if pursued seriously.

---

### Approach 4 — Refund tokens (choose your own bundle)
Each tier earns a number of "refund tokens." Players spend tokens in a special, time-limited shop on whichever rewards they value — gems, free spins, lottery tickets, stickers.

**Why pick this:** every player picks what they actually want. No "I would have rather had X" complaints.
**Trade-off:** biggest engineering build of the five. Players who don't notice the shop walk away with nothing usable.

---

### Approach 5 — Auto-grant + choice (hybrid) ★
Every player gets an automatic gem grant based on their tier (no action needed). Tiers 3 and above also get a small number of choice tokens for a time-limited shop that sells **only things gems can't buy** — commemorative stickers, exclusive sticker sets, etc.

**Why pick this:** every player gets a tangible gem grant they can immediately use; high-investment players also get the variety and recognition they expect. Engineering cost is bounded because the shop is small, short-lived, and only ships items already supported by the game.

**Trade-off:** slightly more communication needed than Approach 1, but worth it.

## What we'd actually pay out
- **Gems** — the main vehicle in every approach.
- **Free wheel spins** — solid filler if the wheel is staying.
- **Lottery tickets** — if lottery stays.
- **Stickers / cosmetics** — already in the shop today.
- **Commemorative items** for top-tier players — these would be new features, optional.

What we will **not** pay out: dust. Dust's only use today was gear crafting, which is being removed. Awarding dust at the same time as removing its only sink would just mean handing out worthless inventory.

## Recommendation
**Approach 5 (Hybrid)** is the best balance for most player bases — every player gets something tangible immediately, and the most-invested players also get the variety and recognition they expect. Engineering cost stays bounded.

- Use **Approach 1** only if we need to ship in days, not weeks.
- Use **Approach 2** only if backend transaction logs are accessible and we're comfortable publishing dust and SLP exchange rates.

## One thing to watch for
Without backend transaction logs, we can only refund based on what players still own today. A player who bought an artifact, used it up, and now owns zero copies gets nothing in the snapshot-only version. If we do have transaction logs accessible, factor in historical spending and bump the refund proportionally for those whose visible inventory understates their real grind.

## Decisions needed before this can ship
1. Do we have access to backend transaction logs? This decides whether failed upgrade attempts and recharges enter the calculation.
2. What gem-equivalent rates do we want to publish for dust and SLP?
3. Will the daily wheel survive the artifact removal? If yes, free spins remain valuable in bundles. If no, drop them.
4. Are commemorative stickers / titles in scope? If not, top-tier rewards stay gem-and-currency only.
5. How much time do we have? Approach 5 needs roughly 2–4 weeks of engineering; Approach 1 needs days.
