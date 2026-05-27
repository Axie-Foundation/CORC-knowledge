# Last Stand — Variant D: Per-Morale-Point Trigger Boost (Plain-Language Summary)

(Boost starts at +1% at morale 40 and grows by +1% per morale point above 39)

## What's changing
The earlier proposal (Variant C) scaled the trigger boost in 5-morale bands — every morale point inside a band got the same boost. The new proposal makes **each morale point matter independently**: every morale point above 39 adds another +1% to the trigger boost.

- Morale 40 → +1%
- Morale 41 → +2%
- Morale 42 → +3%
- … +1% per point …
- Morale 64 → +25%

Bars still come in 5-morale bands (same as Variants A/B/C). Only the trigger boost is now continuous per point.

---

## The new chart in full

| Morale | Bars | Trigger boost |
|---|---|---|
| 34 or below | 0 | +0% |
| 35–39 | 1 | +0% |
| 40 | 2 | +1% |
| 41 | 2 | +2% |
| 42 | 2 | +3% |
| 43 | 2 | +4% |
| 44 | 2 | +5% |
| 45 | 3 | +6% |
| 46 | 3 | +7% |
| 47 | 3 | +8% |
| 48 | 3 | +9% |
| 49 | 3 | +10% |
| 50 | 4 | +11% |
| 55 | 5 | +16% |
| 60 | 6 | +21% |
| 64 | 6 | +25% |

---

## The formula that matches the new chart exactly

In three steps (same shape as the complex formula you have):

```
Step 1 — Boost percentage for an Axie of morale m:
  boost% = max(0, m − 39)

Step 2 — Apply the boost to morale:
  effective_morale = m × (1 + boost% / 100)

Step 3 — Last Stand triggers when:
  (damage − HP_before) × 100  ≤  HP_before × effective_morale
```

Or as a single inequality with the boost inlined:

```
Last Stand triggers when:
  (damage − HP_before) × 100  ≤  HP_before × m × (100 + max(0, m − 39)) / 100
```

This is essentially as compact as the simple formula you have, and produces your chart exactly.

---

## Comparison to your two candidate formulas

The headline finding: **neither of your two formulas actually produces the +1%/point chart.** They both produce something nearby, which is why they "feel similar" at the high end, but they're not the same shape and they don't match the chart.

### Your simple formula

```
(damage − HP_before) < HP_before × (morale − 13) × 1.6 / 100
```

This formula gives every morale point a slightly different effective morale — so it's smooth, like the +1%/point chart wants. But the **amount** of boost it gives is consistently larger than the chart at every relevant morale, with the biggest gap at mid-morale (40–55) and the smallest at the top end (60–64).

| Morale | Your chart wants | Simple formula gives | Gap |
|---|---|---|---|
| 30 | +0% | **−9%** (worse than today) | ▼ 9 pts |
| 35 | +0% | +1% | + 1 pt |
| 39 | +0% | +7% | + 7 pts |
| 40 | +1% | +8% | + 7 pts |
| 44 | +5% | +13% | + 8 pts |
| 49 | +10% | +18% | + 8 pts |
| 50 | +11% | +18% | + 7 pts |
| 55 | +16% | +22% | + 6 pts |
| 60 | +21% | +25% | + 4 pts |
| 64 | +25% | +28% | + 3 pts |

Two things to notice:

1. The simple formula gives Axies a **negative effective morale boost** below morale 35 — they're worse off than they are today. Probably unintended.
2. The simple formula already boosts in the 35–39 band where your chart says +0%, and then gives roughly +7–8 percentage points *more* than your chart wants through the mid-morale band.

**If you ship the simple formula, the chart you drew no longer describes the game.** That's a designer choice, not a bug — but you'd need to either redraw the chart or accept the mismatch.

### Your complex formula

```
boost% = max(0, floor((m − 35) / 5) × 5)
effective_morale = m × (1 + boost / 100)
trigger when: (damage − HP_before) × 100 ≤ HP_before × effective_morale
```

This is the formula that produces the **old +5%-per-band chart** — the one we documented as Variant C. It does **not** produce the new +1%/point chart.

Inside a band, the complex formula gives every morale point the same boost (morale 40 = morale 44 = +5%; morale 45 = morale 49 = +10%). The boost only changes when you cross a band boundary. That's the opposite of what your new chart asks for.

It does have one curious property: at the **top of each band** (morale 44, 49, 54, 59, 64), the complex formula matches the +1%/point chart exactly. But for every morale point *inside* a band, it gives more boost than the +1%/point chart wants.

| Morale | Your chart wants | Complex formula gives | Gap |
|---|---|---|---|
| 39 | +0% | +0% | 0 |
| 40 | +1% | +5% | + 4 pts |
| 41 | +2% | +5% | + 3 pts |
| 42 | +3% | +5% | + 2 pts |
| 43 | +4% | +5% | + 1 pt |
| 44 | +5% | +5% | **0 (matches at band end)** |
| 45 | +6% | +10% | + 4 pts |
| 49 | +10% | +10% | **0 (band end)** |
| 54 | +15% | +15% | **0 (band end)** |
| 59 | +20% | +20% | **0 (band end)** |
| 64 | +25% | +25% | **0 (band end)** |

### Why they "feel similar"

Both formulas converge at the high end (morale 60–64) on roughly +25% boost, and they're both smooth-or-stepwise upward functions of morale. At a glance — especially if you're looking at the top morale ranges — they look interchangeable.

They diverge in the mid-morale band (40–55), where the simple formula is the most generous, the complex formula is medium, and your new chart is the most conservative.

---

## My recommendation

**Use the new formula I gave at the top of this report, not either of the two you already have.**

Reasoning:

1. **Your two candidate formulas don't match your new chart.** The simple formula is too generous everywhere; the complex formula is the old 5-band design. If matching the chart matters (and it should, because the chart is what the design intent looks like to a designer or a player), neither is the answer.

2. **The new formula is just as short as the simple one.** Step 1 is `boost = max(0, morale − 39)` — one operation. Step 2 multiplies. Step 3 is identical to today's trigger check. There's no piecewise math, no floor function, no magic constants.

3. **The new formula is "closer to the original" in a meaningful sense.** Today's trigger is `(damage − HP_before) × 100 ≤ HP_before × morale`. The new formula is `(damage − HP_before) × 100 ≤ HP_before × morale × (100 + boost) / 100`. The only change is multiplying the right side by `(100 + boost) / 100`, where `boost` is 0 below morale 40 and 1, 2, 3, … above it. At morale ≤ 39 the formula is mathematically identical to today.

If you have to pick between **only** the simple and complex formulas as written:

- **Pick the simple formula** if you can accept that the chart no longer matches and that mid-morale Axies will be noticeably more durable than the chart suggests.
- **Avoid the complex formula** for this design intent. It's the old +5%-per-band design — if you've decided you want per-morale-point granularity, the complex formula throws that granularity away the moment it's applied.

---

## The "scaling all the way to the top" question

You said the boost should scale "all the way to the top." With the new formula, the boost keeps growing past morale 64 — there's no implicit cap:

- Morale 65 → +26%
- Morale 70 → +31%
- Morale 80 → +41%
- Morale 100 → +61%

If buff cards can push morale meaningfully above 64, this gets large quickly. Three options:

1. **Let it keep climbing** (your stated intent). Simple, but a buff-stacked morale-90 Axie ends up with a +51% boost, which is substantially more than anything in the chart.
2. **Cap at +25% above morale 64.** Morale 65+ all get the same +25% as morale 64. Safest for balance.
3. **Cap at a higher tier** — e.g. +40% above morale 79, so buff cards still feel rewarding but the runaway scaling is contained.

Worth a heads-up on this comparison: your simple formula naturally tames itself at very high morale because it's linear (morale × 1.6, roughly), so it grows more slowly than the new formula past morale 70 or so. If you choose to leave the new formula uncapped, expect aggressive scaling at the extreme end of the morale range that the simple formula wouldn't produce.

---

## Side-by-side at every morale in the chart range

| Morale | New formula (matches chart) | Simple formula | Complex (5-band) |
|---|---|---|---|
| 30 | +0% | −9% | +0% |
| 35 | +0% | +1% | +0% |
| 39 | +0% | +7% | +0% |
| 40 | **+1%** | +8% | +5% |
| 41 | **+2%** | +9% | +5% |
| 42 | **+3%** | +10% | +5% |
| 43 | **+4%** | +12% | +5% |
| 44 | **+5%** | +13% | +5% |
| 45 | **+6%** | +14% | +10% |
| 46 | **+7%** | +15% | +10% |
| 47 | **+8%** | +16% | +10% |
| 48 | **+9%** | +17% | +10% |
| 49 | **+10%** | +18% | +10% |
| 50 | **+11%** | +18% | +15% |
| 55 | **+16%** | +22% | +20% |
| 60 | **+21%** | +25% | +25% |
| 64 | **+25%** | +28% | +25% |

The new formula (bold column) is the only one that matches your chart at every row.

---

## Things to decide before shipping

1. **Pick a formula.** New (matches chart), simple (more generous, chart needs updating), or complex (old +5%/band design).
2. **Decide the above-morale-64 behaviour** — let the boost keep climbing, cap at +25%, or cap at a higher tier.
3. **Bars cap above morale 64** — same question for the bars table (clamp at 6 or let it grow).
4. **Card-effect guarantees** — Bird tail's +2 bars and Reptile back's +3 bars get weaker relative to the new ceiling of 6. Bump them?
5. **Bar depletion rate** — 6 bars at morale 60 still only buys ~5 turns of Last Stand uptime under today's `−2 entry, −1/turn` depletion. Acceptable, or should depletion scale too?
6. **Replay versioning** — old battles need to replay the same way they were played. Confirm replay determinism survives the change, or version replays so old ones resolve with the original formula.

---

## One-page recap of all four variants

| | Variant A | Variant B | Variant C | **Variant D** |
|---|---|---|---|---|
| Boost starts at morale | 55 | 51 | 40 | **40** |
| Maximum boost (at morale 64) | +30% | +30% | +25% | **+25%** |
| Shape | linear ramp | linear ramp | 5-band step | **per-morale-point ramp** |
| Cliffs | 1 | 1 | 5 | **0** |
| Aligns with bars table | no | no | yes (bands) | yes (boost grows per morale point) |
| Mid-morale (40–49) gets boost | no | no | yes (capped at +10% in band) | **yes (smooth +1% to +10%)** |
| Best mental model | "high morale gets a buff" | "high-ish morale" | "every 5 morale = one tier" | **"each point of morale matters"** |
