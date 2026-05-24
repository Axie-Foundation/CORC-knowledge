# Last Stand — Variant A: Plain-Language Summary

(Boost begins above morale 54)

## What's changing
Two things about Last Stand are getting reworked:

1. **How many bars an Axie gets when Last Stand triggers.** Today, even an Axie at high morale only gets about 3 bars — Last Stand is short. We're changing this so high-morale Axies get a lot more bars.

2. **When Last Stand triggers in the first place.** Today, Last Stand only saves an Axie if the killing blow didn't overshoot the Axie's current HP by too much — and the higher the morale, the more overshoot it can absorb. We're making high-morale Axies survive even bigger overshoots than they can today.

## The new bars table (same in both Variants A and B)

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

## The trigger change (Variant A specifics)

In Variant A, the trigger boost starts at morale **55** and ramps up to its maximum at morale 64:

| Morale | Boost to "how much overshoot is survivable" |
|---|---|
| 54 or below | No change from today |
| 55 | About +20% more overshoot survivable |
| 60 | About +26% |
| 64 | About +30% — the cap |
| 65+ | Stays at +30% |

**What this means in practice:** an Axie that would die today at morale 60 from a big hit may now survive that hit and enter Last Stand instead. Combined with the new bars table, high-morale Axies become a lot tougher — they trigger Last Stand more often AND last longer once they do.

## The math behind it

For anyone who wants the actual formula, here it is in three pieces.

**Step 1 — Calculate the boost percentage for the Axie's morale (m):**

```
If m ≤ 54:        boost = 0%       (no change from today)
If 54 < m ≤ 64:   boost = 20 + (m − 55) × (10 / 9)   percent
If m > 64:        boost = 30%      (capped)
```

So at morale 55 the boost is +20%, at morale 64 it's +30%, and it scales linearly in between.

**Step 2 — Apply the boost to morale to get the "effective morale":**

```
effective_morale = m × (1 + boost / 100)
```

**Step 3 — Last Stand triggers when:**

```
(damage − current_HP) × 100  ≤  current_HP × effective_morale
```

In plain terms: the Axie can survive an overshoot up to *effective_morale* percent of its current HP. Today that's just *morale* percent; under Variant A, high-morale Axies get a 20–30% bigger survivable window.

For comparison, today's formula is just the third step with `effective_morale` replaced by plain `morale`:

```
Today:        (damage − current_HP) × 100  ≤  current_HP × morale
Variant A:    (damage − current_HP) × 100  ≤  current_HP × effective_morale
```

The trigger remains deterministic — there is no random roll. "More often" means "the condition is easier to satisfy" for high-morale Axies.

## A concrete example
An Axie with 100 HP and 60 morale takes a 175-damage hit.

- **Today:** the overshoot is too much for its morale to absorb. The Axie dies.
- **After Variant A:** the boost makes the morale check more forgiving. The Axie survives the hit and enters Last Stand with 6 bars (from the new table).

That's the headline buff — what used to be a guaranteed death becomes ~5 turns of fighting in Last Stand.

## Why pick Variant A
- **More conservative.** It only buffs Axies at morale 55 and up. Axies at morale 50–54 (already 4 bars under the new table) behave the same way they do today on the trigger check.
- **Closer match to the original wording.** The phrase "scale heavily starting at 54 morale" lines up cleanly with "boost begins above morale 54."
- **The morale band where bars jump from 4 to 5 is also the band where the trigger boost kicks in** — easier to communicate as one unified "high-morale buff" zone.

If we want a wider buff that pulls in morale 51–54 as well, see Variant B.

## Things to decide before shipping
- **Above morale 64**, the new bars formula keeps going (morale 65–69 → 7 bars, etc.). Do we cap at 6, or let the trend continue for buff-stacked Axies?
- Two cards today (a **Bird tail** and a **Reptile back**) guarantee Last Stand with 2 and 3 bars respectively, bypassing the normal morale formula. Under the new table those guaranteed amounts feel weak compared to a high-morale Axie's natural 6. Do we bump them up, or leave them as a "Last Stand floor" for low-morale Axies?
- Bars deplete at **-2 the first turn and -1 each turn after**. So 6 bars in the new table = 5 turns of Last Stand uptime, not 6. If we want more uptime, depletion would need to change too.
- The change is server-side (the Rust battle engine). **Old replays** may need a "pre-change" version flag so they replay the way they were originally played.

## One thing to watch for
Axies whose morale gets buffed mid-battle (some cards push morale up) will cross the morale-54 threshold during a fight. The moment they do, they jump from no boost to +20% boost — it's a single step rather than a gradual ramp. Players will feel this.
