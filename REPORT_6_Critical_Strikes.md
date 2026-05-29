# Critical Strikes — Design Audit & Proposals

## TL;DR

The crit system in the battle engine isn't one design — it's **eight overlapping mechanics** layered on top of each other over time. They span pure RNG, pseudo-random pity timers, deterministic guarantees from card setups, defensive denial, debuff-driven forced crits, and crit-as-trigger synergies. Each was likely added to patch a feel-problem with the layer before it.

Today the system sits in an awkward middle: too many cards push toward guaranteed crits to feel rare, but base crit rates are too low to feel like a recurring beat. The result is the user's stated problem — crits feel underwhelming when they happen and confusing when they don't.

Three intersection-point designs are proposed at the end. **Recommended: Split-Tier Crits (Approach C)** as the lowest-risk, biggest-emotional-range change.

---

## Part 1 — How crits work today

### The eight layers

#### Layer 1 — Stat-driven random roll (the base)

Crit chance is a function of an Axie's Morale stat, scaled quadratically. A 68-Morale Axie has roughly a 9% expected crit chance per attack; a 90-Morale Axie crits 2–3× as often.

> **Design read:** this is the original "Morale tells you when crits happen" design. It's pure variance — fun in isolation, brutal across a long ladder where the same matchup swings on a single roll.

#### Layer 2 — Pity timer (pseudo-random distribution)

Sitting on top of Layer 1 is a Dota-style PRD table. Instead of rolling the raw chance every attack, the chance *climbs* the longer an Axie goes without a crit, and *resets* to zero after one lands. The table is tuned so the long-run average still matches the Morale-driven expected rate.

> **Design read:** this is the team's first acknowledgement that pure RNG was too swingy. You can't crit twice in a row by luck, and you can't go a whole match without one. This smooths the experience but also makes crits feel less like surprises — they become inevitable on a clock.

#### Layer 3 — Morale-scaled crit damage

The crit *multiplier* (how hard the crit hits) is also driven by Morale. At Morale 36 a crit deals ~155%; at Morale 100 it caps at 200%.

> **Design read:** Morale controls both *how often* and *how hard* — one stat doing two jobs. This is the single biggest reason crits historically dominated matches: a player who built into Morale got both axes of upside, and crits became the de-facto win condition rather than one tool among many.

#### Layer 4 — Guaranteed crits from card conditions

About seven card parts can force a crit when a setup is met. Examples seen in the code:

- "First attack of the round" (Little Branch horn)
- "Solo card, no combo" (Ronin II back)
- "3-card combo" (Ronin back)
- "Last teammate alive" (PeaceMaker II mouth, Lam II mouth)
- "In Last Stand" (The Last One II tail)
- "Chained with another Merry card" (Merry horn)

> **Design read:** the team's response to "RNG isn't enough — players want agency." These are the most interesting crits in the game design-wise: they're earned, telegraphed, and create deck-building puzzles. They're also the source of today's "crits don't feel rare enough" complaint — multiple cards in one deck stack guaranteed crits across a round.

#### Layer 5 — Additive crit chance bonus

A hook in the engine that adds a flat percentage to crit chance after PRD math but before the dice roll.

> **Design read:** wired up but no live cards write to it. A vestigial dial — likely an experimental tuning lever that never shipped.

#### Layer 6 — Crit denial (defensive)

Two ways to shut crits off:

- **Critical Block** — a card (Aquatic back) that disables crits on the Axie playing it for the round
- **Jinx** — a debuff (from Ranchu II tail) that prevents the affected Axie from critting for a round

> **Design read:** explicit counter-play to crit-heavy builds. The fact that one is labeled "Critical Block" in the localization strings suggests it was designed to be a visible, satisfying "you blocked it!" moment — but in practice it's a silent stat-check.

#### Layer 7 — Lethal (forced crit on the target)

A debuff applied to a defender that converts the next incoming hit into a crit, regardless of attacker stats.

> **Design read:** the most interesting layer mechanically. It's a "setup → payoff" pattern — turn 1 you mark a target, turn 2 anyone hitting it crits. It turns crits into a *team* event rather than a personal stat-check. Underexplored: there's only one application path.

#### Layer 8 — Crit-as-trigger (echo effects)

Cards that don't *cause* crits but react to them when they happen:

- **Imp II horn** — "Steal 1 energy per critical strike dealt by your team this round"
- **Dual Blade horn** — when a crit happens on its attack, *doubles* the crit damage

> **Design read:** the most recent evolution. Instead of crits being a damage event, they become a *signal* other cards listen to. Dual Blade in particular reveals the design tension — it can stack on top of a crit and produce a 4×+ damage swing in one hit. This is the "memorable moment" the design wants, but it's currently gated behind a specific part.

### The arc the code tells

1. Started with pure stat-driven RNG (Layers 1, 3)
2. Realized streaks felt unfair → added pity timer (Layer 2)
3. Realized players wanted agency → added guaranteed-crit triggers (Layer 4) and Lethal (Layer 7)
4. Realized crit-heavy builds had no counter → added denial (Layer 6) and Jinx
5. Started using crits as a *trigger* for other effects rather than as a pure damage event (Layer 8)
6. Began experimenting with a tuning dial that never shipped (Layer 5)

The team has been quietly moving *away* from "crit = big random damage" and *toward* "crit = a designable moment." But the original Layer 1/3 RNG is still the dominant path, which is why the current feel is muddled.

---

## Part 2 — The current problem

The user's framing was correct on both sides:

- **Crits used to be too powerful** because Morale double-dipped (frequency *and* multiplier), guarantees stacked easily, and Dual Blade-style amplifiers existed.
- **Crits now feel underwhelming** because the damage cap is at 200%, the base rate is low, and most crits happen on small attacks where the visual "CRITICAL" text doesn't translate to a real swing.

The gap: crits are too quiet to feel like a beat in every match, but too consequential to make more frequent. That's the intersection we need to find.

---

## Part 3 — Three proposed designs

Each proposal picks a different axis from the existing system to lean into. They are alternatives, not stages.

---

### Approach A — Charged Crits

**Core idea:** delete the random roll. Crits become a resource the player visibly builds.

**How it plays:**
- Every clean hit (non-crit, non-blocked) adds 1 *charge* to that Axie's crit meter.
- At 4 charges, the *next* attack is a guaranteed crit and empties the meter.
- Morale shifts roles: instead of a % chance per attempt, it determines charges-per-hit. High-Morale Axies gain 2 charges per hit; low-Morale 1.
- Existing guaranteed-crit cards (Little Branch, PeaceMaker II, etc.) become "instantly fill 2 charges" — still strong, no longer a free outcome.
- Jinx becomes "drain a charge"; Lethal becomes "your target's next hit on this Axie deals as if at max charge."

**What this fixes:**
- Every match has crits, on a predictable cadence — roughly one per Axie every 4 hits.
- Crits are *telegraphed* — both players see the bar fill, both can play around it.
- Eliminates "I lost because they crit turn 1" without eliminating "I lost because they set up a crit chain."

**What it costs:**
- Kills the surprise factor entirely. Some players love unpredictable crits — this design says that emotion is the price.
- Mitigation: give a small (~5%) "lucky overflow" chance at 3 charges to crit one hit early, preserving a sliver of surprise.

**Risk:** medium. The PRD table goes away, but most card hooks (`guarantee_crit`) translate cleanly into "fill the meter."

---

### Approach B — Lethal-First (rare but devastating)

**Core idea:** crits only happen when *set up* by a card or debuff. Remove the random roll entirely. To compensate, make every crit much harder.

**How it plays:**
- No more Morale-based random crits. The only way to crit is to trigger a guaranteed condition (Layer 4) or apply a Lethal-style debuff (Layer 7).
- Crit damage cap raised to 250–300% (from today's 200%).
- Client adds a screen-shake and brief time-pause on every crit — they're now genuinely rare.
- Expand Lethal: today there's one application path. Add 2–3 more (Aroma-style debuff that converts on break, "next attack on a stunned target crits," "low-HP target crits").

**What this fixes:**
- Every crit becomes a *story moment*. Roughly 1 per 2–3 rounds instead of multiple per round.
- The swings are *legible* — your opponent saw Lethal apply last round and chose not to cleanse it.
- Memorability goes way up; frequency goes way down.

**What it costs:**
- The biggest balance lift of the three. Every Axie tuned around Morale's frequent small crits now never crits at all — full re-tune required.
- Low-Morale builds lose their identity unless they have access to Lethal application.

**Risk:** high. Best fit if the design vision is "crits should be the headline moment of every match, never the background hum."

---

### Approach C — Split-Tier Crits *(recommended)*

**Core idea:** keep the existing system, but split crit outcomes into two tiers — a frequent "Glancing" crit and a rare "Heavy" crit. Smallest change, biggest emotional range.

**How it plays:**
- The existing PRD roll still decides whether *a crit* happens.
- A second roll then decides which tier:
  - **Glancing crit (~85% of crits):** +30–40% damage. Mild visual flourish — a color change on the damage number, light SFX. Not a "moment."
  - **Heavy crit (~15% of crits):** +150–200% damage. Full SFX, "CRITICAL!!" text, brief shake.
- Any `guarantee_crit` trigger or Lethal hit *always* upgrades to Heavy. So earned crits remain the big crits.
- Dual Blade's hard-coded 200% multiplier becomes the *definition* of a Heavy crit — it stops being an exception and becomes the canonical "the big one."

**What this fixes:**
- Crit cadence stops being binary. Combat at every Morale level has a constant low hum of small crits, keeping things textured.
- Heavy crits become rare enough (~1–2 per match) to feel earned, without disappearing.
- The "earned = big" principle is now baked into the system: setup-based crits *always* land Heavy, while random ones almost always land Glancing.

**What it costs:**
- Slightly more complex for spectators reading combat logs. Mitigation: the client already prints "CRITICAL" text — just give it two visual treatments.
- Average match damage shifts by ~5–10%; minor Morale curve retune.

**Risk:** low. Layered on top of the existing system without replacing any of it. Most synergy cards keep working, with small extensions (Imp II steals 1 energy on Glancing, 2 on Heavy).

---

## Part 4 — Comparison table

| | Crit cadence | Average per-crit impact | Predictability | Re-balance scope |
|---|---|---|---|---|
| **A — Charged** | Predictable, ~1 per 4 hits | Medium | High | Medium |
| **B — Lethal-First** | Rare, ~1 per 2–3 rounds | Very high | Very high | Large |
| **C — Split-Tier** | Same as today; most small | Glancing low, Heavy very high | Same as today | Small |

---

## Part 5 — Recommendation

**Ship Approach C first.**

It preserves the team's existing balance work, keeps every layer of the current system functional, and produces the emotional range the design needs: a constant low-grade "something is happening" texture, with rare punctuation moments that are unambiguously big. The "earned crits always go Heavy" rule cleanly resolves the current contradiction where setup-based cards (Lethal, Ronin, PeaceMaker II) feel underwhelming despite being designed as payoff moments.

If telemetry after C still shows crits feel under-frequent or over-frequent, Approach A becomes the natural next step — it's a structural change that benefits from having C's tiered framing already in place.

Approach B is best reserved as a "we want to fundamentally redefine the role of crits in the game" decision, which would belong in a larger combat overhaul, not an iteration.
