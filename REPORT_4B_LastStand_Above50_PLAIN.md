# Last Stand — Variant B: Plain-Language Summary

(Boost begins above morale 50)

## What's changing
Same two changes as Variant A:

1. **More bars at high morale** — the bars table changes so high-morale Axies last longer in Last Stand.

2. **More frequent Last Stand triggers at high morale** — Axies above the threshold survive larger overshoots than they can today.

The single difference between Variants A and B is **where the trigger boost starts**:
- **Variant A** starts at morale **55**.
- **Variant B** starts at morale **51** — a wider band.

## The new bars table (same as Variant A)

| Morale | Bars |
|---|---|
| 34 or below | 0 |
| 35–39 | 1 |
| 40–44 | 2 |
| 45–49 | 3 |
| 50–54 | 4 |
| 55–59 | 5 |
| 60–64 | 6 |

Today, an Axie at morale 60–64 only gets 3 bars. After this change, the same Axie gets 6 bars — double what it had before.

## The trigger change (Variant B specifics)

In Variant B, the trigger boost starts at morale **51** and ramps up to its maximum at morale 64:

| Morale | Boost to "how much overshoot is survivable" |
|---|---|
| 50 or below | No change from today |
| 51 | About +20% more overshoot survivable |
| 55 | About +23% |
| 60 | About +27% |
| 64 | About +30% — the cap |
| 65+ | Stays at +30% |

**What this means in practice:** more Axies benefit than in Variant A. Anything at morale 51 or above gets some boost, not just the high-end builds. A mid-morale Axie at morale 52 that would die today may now survive into Last Stand.

## The math behind it

For anyone who wants the actual formula, here it is in three pieces.

**Step 1 — Calculate the boost percentage for the Axie's morale (m):**

```
If m ≤ 50:        boost = 0%       (no change from today)
If 50 < m ≤ 64:   boost = 20 + (m − 51) × (10 / 13)   percent
If m > 64:        boost = 30%      (capped)
```

So at morale 51 the boost is +20%, at morale 64 it's +30%, and it scales linearly in between.

**Step 2 — Apply the boost to morale to get the "effective morale":**

```
effective_morale = m × (1 + boost / 100)
```

**Step 3 — Last Stand triggers when:**

```
(damage − current_HP) × 100  ≤  current_HP × effective_morale
```

In plain terms: the Axie can survive an overshoot up to *effective_morale* percent of its current HP. Today that's just *morale* percent; under Variant B, any Axie above morale 50 gets a 20–30% bigger survivable window.

For comparison, today's formula is just the third step with `effective_morale` replaced by plain `morale`:

```
Today:        (damage − current_HP) × 100  ≤  current_HP × morale
Variant B:    (damage − current_HP) × 100  ≤  current_HP × effective_morale
```

The only difference between Variant A's formula and Variant B's formula is in Step 1: A starts boosting above morale 54 (slope 10/9), B starts boosting above morale 50 (slope 10/13). Steps 2 and 3 are identical.

The trigger remains deterministic — there is no random roll. "More often" means "the condition is easier to satisfy" for high-morale Axies.

## Two concrete examples

**1. A high-morale Axie (where Variants A and B behave nearly the same)**
100 HP, 60 morale, takes a 175-damage hit.
- **Today:** Axie dies.
- **After Variant B:** survives. Enters Last Stand with 6 bars.

**2. A mid-morale Axie (where Variant B differs from Variant A)**
100 HP, 52 morale, takes a 158-damage hit.
- **Today:** Axie dies.
- **After Variant A:** still dies (morale 52 is below Variant A's threshold of 54).
- **After Variant B:** survives. Enters Last Stand with 4 bars.

This second example is the key difference. Variant B reaches further down the morale curve.

## Why pick Variant B
- **More aggressive.** Buffs a wider band of Axies, not just the high-end builds.
- **Pushes the meta toward "morale stacking" across more team archetypes.** Beast and Bug body parts (which give the most morale per part) become attractive choices for a broader range of team designs.
- **If we want the change to feel impactful for the broader player base** — not just whales who built max-morale teams — Variant B reaches more players.

If we want a more conservative buff that only rewards extreme high-morale builds, see Variant A.

## Things to decide before shipping
Same considerations as Variant A:
- **Cap bars at 6 above morale 64**, or let them keep growing?
- **Adjust the hard-coded Bird tail (+2 bars) and Reptile back (+3 bars) guarantees** to match the new bars table, or leave them alone?
- **Adjust bar depletion rate** to give the longer bars more uptime?
- **Version old replays** so they don't change behaviour retroactively.

## One thing to watch for unique to Variant B
Variant B's threshold (morale 50) sits right at the boundary where the new bars table already grants 4 bars. So an Axie crossing from morale 50 to 51 sees a double effect at once: bars don't change (still 4), but the trigger boost jumps from 0% to +20% in a single step. Players will notice this cliff.

If we want a gentler curve, we could ramp the boost in starting from 0% at morale 50 instead of jumping to +20% at morale 51 — but that would be a different formula than what was originally requested.
