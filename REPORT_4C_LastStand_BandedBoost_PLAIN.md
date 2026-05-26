# Last Stand — Variant C: Plain-Language Summary

(Banded 5-morale step boost — +5% per band, capped at +25%)

## What's changing
Two things about Last Stand are getting reworked, same as in Variants A and B:

1. **How many bars an Axie gets when Last Stand triggers.** Today even a high-morale Axie only gets about 3 bars. After the change, high-morale Axies get up to 6 bars.

2. **When Last Stand triggers in the first place.** Today, Last Stand only saves an Axie if the killing blow didn't overshoot the Axie's current HP by too much. The higher the morale, the more overshoot it can absorb. We're making this more forgiving for mid-and-high-morale Axies.

What's different in Variant C is **how** the trigger gets more forgiving: instead of a smooth ramp like in Variants A and B, the buff steps up in clean 5-morale tiers.

## The new bars table (same as Variants A and B)

| Morale | Bars |
|---|---|
| 34 or below | 0 |
| 35–39 | 1 |
| 40–44 | 2 |
| 45–49 | 3 |
| 50–54 | 4 |
| 55–59 | 5 |
| 60–64 | 6 |

## The trigger boost (Variant C specifics)

| Morale | Trigger boost |
|---|---|
| 34 or below | +0% |
| 35–39 | +0% |
| 40–44 | +5% |
| 45–49 | +10% |
| 50–54 | +15% |
| 55–59 | +20% |
| 60–64 | +25% |
| 65+ | +25% (recommended cap), or keeps climbing |

**What this means in practice:** the buff turns on at morale 40 — much earlier than Variants A (morale 55) or B (morale 51). The cap is smaller (+25% vs +30% in A/B), so the buff is more spread out across the player base rather than concentrated at the top end.

## The math behind it

For anyone who wants the actual formula, here it is in three pieces.

**Step 1 — Calculate the boost percentage for the Axie's morale (m):**

```
boost% = max(0, floor((m − 35) / 5) × 5)
```

With an optional cap at +25% above morale 64:

```
boost% = clamp( floor((m − 35) / 5) × 5,  0,  25 )
```

So at morale 40 the boost is +5%, at morale 45 it's +10%, at morale 60 it's +25%, and it doesn't change within a band.

**Step 2 — Apply the boost to morale:**

```
effective_morale = m × (1 + boost / 100)
```

**Step 3 — Last Stand triggers when** (same as Variants A and B):

```
(damage − current_HP) × 100  ≤  current_HP × effective_morale
```

In plain terms: the Axie can survive an overshoot up to *effective_morale* percent of its current HP.

For comparison with today's formula and the other two variants:

```
Today:        (damage − current_HP) × 100  ≤  current_HP × morale
Variant A:    (damage − current_HP) × 100  ≤  current_HP × effective_morale       (smooth ramp above morale 54)
Variant B:    (damage − current_HP) × 100  ≤  current_HP × effective_morale       (smooth ramp above morale 50)
Variant C:    (damage − current_HP) × 100  ≤  current_HP × effective_morale       (5-morale steps starting at morale 40)
```

Step 2 and Step 3 are identical across all three variants. Only Step 1 (how the boost is calculated) differs.

The trigger remains deterministic — there is no random roll. "More often" means "the condition is easier to satisfy" at higher morale.

## A concrete example

An Axie at 100 HP and morale 50 takes a 155-damage hit (55 overkill).

- **Today:** Axie dies (overkill is too much for morale 50 to absorb).
- **Variant A:** still dies — morale 50 is below A's threshold (55).
- **Variant B:** still dies — morale 50 is not above B's threshold (strictly above 50, so 51+).
- **Variant C:** Axie **survives** with +15% boost (effective morale becomes 57.5).

This is the Variant-C-only save: a morale-50 Axie taking a moderate overkill hit lives in C but dies in A and B. The buff reaches further down the morale ladder, even though its ceiling is lower.

## Why pick Variant C
- **Cleanest mental model.** The bars table and the trigger boost both use the same 5-morale bands. Every band crossed gives **+1 bar AND +5% trigger boost at the same time.** Easy for players to internalise: "every 5 morale is one tier up." Variants A and B run the bars and the trigger on different curves.
- **Reaches more players.** The buff starts at morale 40, not 51 or 55. Mid-morale Axies benefit too, not just high-end builds.
- **More conservative cap.** Maximum boost is +25% vs +30% in A/B. Less risk of runaway scaling from buff cards stacking morale.
- **Tier-based design rewards tier-based investment.** Players can target a specific band ("I'm trying to push my Aquatic to morale 50") and feel the payoff immediately.

## The trade-off
Variant C is a **step function**. Within a band, morale doesn't matter — a morale-40 Axie and a morale-44 Axie both get +5%. The buff only changes when you cross a band boundary.

This creates **five cliffs** at morale 40, 45, 50, 55, 60. A card that buffs an Axie's morale from 44 to 45 is now meaningfully more valuable than a card that buffs from 41 to 42, even though both are +1 morale.

Variants A and B each have only one cliff (at morale 55 or 51), and are smooth everywhere else.

## Things to decide before shipping
- **Above morale 64**, do we cap at +25% or let the formula keep climbing (morale 70 → +35%, etc.)?
- **The bars table cap** — same question. Cap at 6 bars or let it grow?
- **Card guarantees** — Bird tail's +2 bars and Reptile back's +3 bars become weaker relative to a high-morale Axie's natural 6 bars. Do we bump them?
- **Bar depletion** — under the new table a morale-60 Axie gets 6 bars, but with `−2 on entry, −1 per turn` that's only 5 turns of Last Stand uptime. Should depletion scale too?
- **Replay versioning** — old battles need to replay the same way they were played. Either confirm determinism survives the change, or version replays so old ones resolve with the old formula.

## One thing to watch for
The morale-39 → morale-40 cliff is special. That's where the Axie goes from "no buff at all" to "+5% buff" in a single morale point, and it's also where the bars table starts giving 2 bars instead of 1.

If buff cards commonly push Axies through that boundary mid-battle, designers should make sure the moment feels like a satisfying threshold rather than an arbitrary line in the sand. The same concern applies (less acutely) at each of the other four boundaries.

## How Variant C compares — one-page recap

| | Variant A | Variant B | Variant C |
|---|---|---|---|
| Buff starts at | morale 55 | morale 51 | morale 40 |
| Maximum buff | +30% | +30% | +25% |
| Shape | smooth ramp | smooth ramp | 5-morale steps |
| Cliffs | one | one | five |
| Aligns with bars table | no | no | **yes (1:1)** |
| Reaches mid-morale (40–49) | no | no | **yes** |
| Best mental model for players | "high morale gets a buff" | "high-ish morale gets a buff" | **"every 5 morale = one tier up"** |
