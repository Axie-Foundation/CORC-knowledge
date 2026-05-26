# Report 4C — Last Stand: New Bars Table and Trigger Formula (Variant C: Banded 5-Morale Step Boost)

**Date:** 2026-05-23
**Scope:** Two related changes to the Rust battle engine:
1. Replace the morale → Last Stand bars mapping with the user's proposed table.
2. Change the Last Stand activation formula so triggering is biased upward in **discrete 5-morale bands**, with each band adding +5% boost, starting at morale 40 and capping at +25% at morale 60+.

Variants A (boost begins above morale 54, linear ramp to +30%) and B (boost begins above morale 50, linear ramp to +30%) are covered in separate reports.

Every value below comes from the Rust engine source under `game-server-main/lunacia/src/`.

---

## 1. Baseline — current Last Stand implementation

### 1.1 The bars are computed by a formula, not a lookup table

`state.rs`, lines 254–255:

```rust
pub fn get_last_stand_turns(morale: i32) -> i32 {
    ((morale as f64 * 1.4 - 30.0).sqrt() / 2.0).floor() as i32
}
```

In plain math:

```
bars = floor( sqrt(morale × 1.4 − 30) / 2 )
```

It's a sublinear curve — bars grow slowly with morale. Even at morale 64, an Axie only gets 3 bars under the current formula. The user's proposed table tops out at 6 bars at morale 60–64 — double the current ceiling.

### 1.2 The trigger is deterministic, not random

`state.rs`, lines 247–251:

```rust
pub fn can_last_stand(&self, morale: i32, decrement: i32, guarantee_last_stand: bool) -> bool {
    !self.is_chimera
        && decrement >= self.hp
        && ((decrement - self.hp) * 100 <= self.hp * morale
            || (guarantee_last_stand && self.hp > 0))
}
```

In plain math:

```
Axie is not a Chimera
AND damage >= current_HP                              // this hit would kill
AND (
      (damage − current_HP) × 100  ≤  current_HP × morale
   OR  (card-effect guarantee AND current_HP > 0)
)
```

The morale-dependent inequality rearranges to:

```
overkill_damage / current_HP  ≤  morale / 100
```

i.e. **an Axie can survive an overkill hit up to morale% of its current HP**.

There is no random roll. Whether Last Stand triggers is determined entirely by the size of the overkill relative to morale.

### 1.3 What happens once Last Stand triggers

`round_runner.rs`, lines 616–625:

```rust
fn get_reduced_last_stand_turns(
    last_stand_turns: i32,
    enter_last_stand: bool,
) -> i32 {
    if enter_last_stand {
        2     // first turn after entering Last Stand: lose 2 bars
    } else {
        1     // every subsequent turn: lose 1 bar
    }
}
```

Bars are not refilled by healing (`battle.rs:612–613`). Last Stand ends when `last_stand_turns` reaches 0 or a card disables it.

### 1.4 Card abilities that override the formula

- **Bird tail PostFightII** (`card/bird.rs:1149–1175`): forces Last Stand with **exactly 2 bars** via `.guarantee_last_stand(Some(2))`.
- **Reptile back BoneSailII** (`card/reptile.rs:758–773`): on lethal incoming damage, forces Last Stand with **exactly 3 bars** via `.guarantee_last_stand(Some(3))`.
- **Aquatic horn ShoalStar** (`card/aquatic.rs:575–589`): sets `context.turn.cannot_last_stand = true`.
- **Beast wing Sting** (`card/beast.rs:632–636`): same `cannot_last_stand = true` mechanism.

**Design heads-up:** these guarantees were balanced against a world where Last Stand maxed at ~3 bars naturally. Under the new table a morale-60 Axie naturally gets 6 bars. The guarantees shrink in relative power — the team should decide whether to bump them.

### 1.5 Morale stat range in the game

From `model.rs:49–54`: base morale starts at `35 + (delta.morale_delta.round() × 4)`. Body-part class deltas (`model.rs:64–72`): Beast / Bug = +3, Bird / Plant = +1, Aquatic / Reptile = +0. Observed test-data range is 27–68 (`lua/plant_beast_aqua.lua`, `lua/plant_poison.lua`).

---

## 2. Change 1 — The new morale → bars table

### 2.1 The user's proposed table

| Morale band | Bars |
|---|---|
| ≤ 34 | 0 |
| 35–39 | 1 |
| 40–44 | 2 |
| 45–49 | 3 |
| 50–54 | 4 |
| 55–59 | 5 |
| 60–64 | 6 |

### 2.2 The formula that produces it exactly

```
bars = max(0, floor((morale − 30) / 5))
```

This is the same formula used in Variants A and B — the bars table is independent of the trigger-formula variant.

### 2.3 Rust implementation — drop-in replacement for `state.rs:254–255`

```rust
pub fn get_last_stand_turns(morale: i32) -> i32 {
    if morale < 35 {
        return 0;
    }
    (morale - 30) / 5
}
```

Optionally clamp at 6 bars for morale > 64 with `.min(6)`.

---

## 3. Change 2 — The trigger formula (Variant C: banded 5-morale step boost)

The user proposed a trigger-boost table:

| Morale band | Boost |
|---|---|
| ≤ 34 | +0% |
| 35–39 | +0% |
| 40–44 | +5% |
| 45–49 | +10% |
| 50–54 | +15% |
| 55–59 | +20% |
| 60–64 | +25% |

Unlike Variants A and B, this is a **step function** — every morale point within a band receives the same boost. The boost only changes at band boundaries (40, 45, 50, 55, 60).

### 3.1 The math

**Step 1 — Calculate the boost percentage for morale m:**

```
boost%(m) = max(0, floor((m − 35) / 5) × 5)
```

This delivers the user's table exactly, with no special-case branching.

**Step 2 — Apply the boost to morale:**

```
effective_morale(m) = m × (1 + boost%(m) / 100)
```

**Step 3 — Trigger condition (same as Variants A and B):**

```
(damage − current_HP) × 100  ≤  current_HP × effective_morale
```

### 3.2 Verification — every row of the user's table

| Morale | (m−35) / 5 | floor | × 5 | max(0, …) | Expected | Match |
|---|---|---|---|---|---|---|
| 30 | −1.0 | −1 | −5 | **0** | +0% | ✓ |
| 34 | −0.2 | −1 | −5 | **0** | +0% | ✓ |
| 35 | 0.0 | 0 | 0 | **0** | +0% | ✓ |
| 39 | 0.8 | 0 | 0 | **0** | +0% | ✓ |
| 40 | 1.0 | 1 | 5 | **5** | +5% | ✓ |
| 44 | 1.8 | 1 | 5 | **5** | +5% | ✓ |
| 45 | 2.0 | 2 | 10 | **10** | +10% | ✓ |
| 49 | 2.8 | 2 | 10 | **10** | +10% | ✓ |
| 50 | 3.0 | 3 | 15 | **15** | +15% | ✓ |
| 54 | 3.8 | 3 | 15 | **15** | +15% | ✓ |
| 55 | 4.0 | 4 | 20 | **20** | +20% | ✓ |
| 59 | 4.8 | 4 | 20 | **20** | +20% | ✓ |
| 60 | 5.0 | 5 | 25 | **25** | +25% | ✓ |
| 64 | 5.8 | 5 | 25 | **25** | +25% | ✓ |
| 65 | 6.0 | 6 | 30 | **30** | (above table) | uncapped extends naturally |

### 3.3 Above-morale-64 behaviour

The raw formula keeps climbing: morale 65–69 → +30%, morale 70–74 → +35%, etc. Since the user's table stops at 60–64 → +25%, the design team must choose one of three options:

| Option | Formula | Behaviour at morale 70 |
|---|---|---|
| Keep climbing | `max(0, floor((m − 35) / 5) × 5)` | +35% |
| Cap at +25% | `clamp(floor((m − 35) / 5) × 5, 0, 25)` | +25% |
| Cap at a different ceiling | `min(X, max(0, floor((m − 35) / 5) × 5))` | +X% |

**Recommendation:** cap at +25% to match the user-supplied table. If buff cards (`buff.rs:114–122`) can push morale above 64, the cap prevents runaway boost scaling.

### 3.4 Effective-morale lookup (worked through every band)

For an Axie with 100 current HP, the effective morale and survivable overkill at each band:

| Morale | Boost | effective_morale | Survivable overkill | Versus today |
|---|---|---|---|---|
| 30 | +0% | 30 | 30% of HP | unchanged |
| 39 | +0% | 39 | 39% of HP | unchanged |
| 40 | +5% | 42 | 42% of HP | +2 HP equivalent |
| 44 | +5% | 46.2 | 46.2% of HP | +2.2 HP equivalent |
| 45 | +10% | 49.5 | 49.5% of HP | +4.5 HP equivalent |
| 49 | +10% | 53.9 | 53.9% of HP | +4.9 HP equivalent |
| 50 | +15% | 57.5 | 57.5% of HP | +7.5 HP equivalent |
| 54 | +15% | 62.1 | 62.1% of HP | +8.1 HP equivalent |
| 55 | +20% | 66.0 | 66.0% of HP | +11 HP equivalent |
| 59 | +20% | 70.8 | 70.8% of HP | +11.8 HP equivalent |
| 60 | +25% | 75.0 | 75.0% of HP | +15 HP equivalent |
| 64 | +25% | 80.0 | 80.0% of HP | +16 HP equivalent |

### 3.5 The cliff problem — what makes this different from Variants A and B

A step function has **discontinuities at every band boundary**:

| Boundary | Boost jump | Effective-morale jump (at 100 HP) |
|---|---|---|
| Morale 39 → 40 | +0% → +5% | 39 → 42 (+3) |
| Morale 44 → 45 | +5% → +10% | 46.2 → 49.5 (+3.3) |
| Morale 49 → 50 | +10% → +15% | 53.9 → 57.5 (+3.6) |
| Morale 54 → 55 | +15% → +20% | 62.1 → 66.0 (+3.9) |
| Morale 59 → 60 | +20% → +25% | 70.8 → 75.0 (+4.2) |

Five cliffs in the 40–64 morale range. By comparison:
- Variant A has one cliff (morale 54 → 55) and is linear afterward.
- Variant B has one cliff (morale 50 → 51) and is linear afterward.

This matters because **buff cards** (`buff.rs:114–122`) frequently push morale up by 1 mid-battle. Under Variant C, whether that +1 morale crosses a band boundary determines whether it's a no-op or a meaningful buff. Under Variants A and B, every +1 morale within the boosted range produces a smooth increment.

### 3.6 Worked example — the same Axie, three variants

Axie at 100 HP, 50 morale, takes 165 damage (65 overkill).

| Formula | Effective morale | Survivable overkill | Trigger? |
|---|---|---|---|
| Today | 50 | 50% (50 HP) | ❌ 65 > 50 — dies |
| Variant A (boost > 54) | 50 (no boost) | 50% | ❌ dies |
| Variant B (boost > 50) | 50 (no boost; morale = 50 doesn't qualify as ">50") | 50% | ❌ dies |
| **Variant C** | 57.5 (+15%) | 57.5% | ❌ 65 > 57.5 — dies (closer but still no) |

Same Axie, takes 155 damage (55 overkill) instead:

| Formula | Effective morale | Survivable overkill | Trigger? |
|---|---|---|---|
| Today | 50 | 50% | ❌ 55 > 50 — dies |
| Variant A | 50 | 50% | ❌ dies |
| Variant B | 50 | 50% | ❌ dies (threshold is strictly greater than 50) |
| **Variant C** | 57.5 | 57.5% | ✅ 55 ≤ 57.5 — **survives** |

This is the Variant-C-only save: an Axie at exactly morale 50, taking a moderate overkill hit, survives in Variant C but dies in Variants A and B. The reach is wider, the maximum boost is smaller.

### 3.7 Rust implementation — drop-in replacement for `state.rs:247–251`

```rust
pub fn can_last_stand(&self, morale: i32, decrement: i32, guarantee_last_stand: bool) -> bool {
    if self.is_chimera || decrement < self.hp {
        return false;
    }
    if guarantee_last_stand && self.hp > 0 {
        return true;
    }
    let effective_morale = apply_morale_boost_c(morale);
    (decrement - self.hp) as i64 * 100 <= self.hp as i64 * effective_morale as i64
}

/// Variant C: banded step function. +5% per 5-morale band starting at morale 40.
/// Optionally cap at +25% with the .min(25) call.
fn apply_morale_boost_c(morale: i32) -> i32 {
    if morale < 40 {
        return morale;
    }
    let boost_pct: i32 = ((morale - 35) / 5) * 5;
    let boost_pct = boost_pct.min(25);   // <-- comment this line to let the formula keep climbing above morale 64
    morale + (morale * boost_pct) / 100
}
```

Notes:

- Rust integer division truncates toward zero. The `morale < 40` guard handles the negative-numerator edge case cleanly; without it, morale 30 would compute `(30 − 35) / 5 = −1` which is correct here but easier to reason about with the explicit early-return.
- Integer math on the multiplier means morale 44 gets `44 + (44 × 5) / 100 = 44 + 2 = 46` rather than 46.2 (small precision loss). For exact arithmetic use `f64`.

### 3.8 What is NOT changed

- The Chimera exclusion still applies.
- The "damage must be lethal" prerequisite still applies.
- The `guarantee_last_stand` card-override path still bypasses the morale check.
- `cannot_last_stand` (set by Aquatic ShoalStar and Beast Sting) still blocks Last Stand entirely.
- Bar depletion in `round_runner.rs:616–625` is unchanged: −2 on entry, −1 per turn.
- The "no healing out of Last Stand" rule in `battle.rs:612–613` is unchanged.

---

## 4. Why Variant C over Variants A and B?

### 4.1 The clean alignment with the bars table

Both the bars table (Change 1) and the trigger boost (Variant C) use the same 5-morale bands:

| Morale | Bars (Change 1) | Trigger boost (Variant C) |
|---|---|---|
| 35–39 | 1 | +0% |
| 40–44 | 2 | +5% |
| 45–49 | 3 | +10% |
| 50–54 | 4 | +15% |
| 55–59 | 5 | +20% |
| 60–64 | 6 | +25% |

Crossing each band boundary delivers **+1 bar AND +5% trigger boost simultaneously**. This is a much simpler mental model for players than Variants A/B, where the bars and the boost run on different curves.

In Variants A and B, "high morale" means two separate things (different bars curve vs. different trigger curve). In Variant C, "+5 morale tier" is the only thing that ever matters.

### 4.2 Variant comparison at a glance

| Property | Variant A | Variant B | Variant C |
|---|---|---|---|
| Shape | Linear ramp | Linear ramp | Banded step function |
| Boost begins at morale | 55 | 51 | 40 |
| Maximum boost | +30% (at morale 64) | +30% (at morale 64) | +25% (at morale 60+) |
| Smooth within band | yes (each morale point matters) | yes | no (morale 40 = morale 44) |
| Number of cliffs | 1 | 1 | 5 |
| Alignment with bars table | no | no | yes (1:1 band match) |
| Reach (lowest morale that benefits) | 55 | 51 | 40 |
| Conservatism (max boost) | most aggressive (+30%) | most aggressive (+30%) | most conservative (+25%) |

### 4.3 When to pick which

- **Pick A** if you want a narrow buff that only rewards extreme high-morale builds, smooth within the boosted band.
- **Pick B** if you want a wider linear buff that extends into mid-morale (51+).
- **Pick C** if you want the simplest player-facing mental model (every 5 morale = one tier up) and you're comfortable with cliffs at band boundaries.

The user's stated language in the original Question 4 — "scale heavily starting at 54 morale" — is closest to Variant A in name. But the user's later Question 4C table reveals a preference for the banded alignment, which suggests the design intent has evolved toward "make morale a tiered ladder with both bars and trigger scaling together."

---

## 5. Code surface — files to touch

Engine (`game-server-main/lunacia/src/`):

1. `state.rs:254–255` — replace `get_last_stand_turns` with the new formula from §2.3.
2. `state.rs:247–251` — replace `can_last_stand` with the Variant C body from §3.7, and add the `apply_morale_boost_c` helper alongside.
3. Optional: bump `Some(2)` in `card/bird.rs:1161` and `Some(3)` in `card/reptile.rs:770` if the design team decides those guarantees should scale.

Tests:

4. Add unit tests for `get_last_stand_turns` covering boundary morales (34/35, 39/40, 44/45, 49/50, 54/55, 59/60, 64/65) and for `can_last_stand` covering all five band boundaries plus the morale-64 ceiling. Variant C has more boundary cases than A/B; ensure the test matrix grows accordingly.

Client mirror:

5. `axie-classic/Assets/Scripts/Models/Rust/FighterState.cs` — the `LastStandTurns` field at line 31 receives the value from the engine; no formula change is needed client-side. Confirm any UI that previews "expected bars" or "expected trigger chance" reflects both new tables.

Documentation:

6. `.codemap/card-battle-core.yaml`, `.codemap/card-battle-data.yaml`, `.codemap/models-rust.yaml` — refresh to describe the banded boost.
7. `game-server-main/ARCHITECTURE.md` (Last Stand trigger phase, around line 259) — refresh wording.

---

## 6. Open questions for the design team

1. **Cap at +25% above morale 64, or let it keep climbing?** The user's table stops at +25%. Capping is safer for buff-card scaling; not capping is simpler code but lets a buff-stacked morale-80 Axie reach +45% boost.
2. **Cap bars at 6 above morale 64?** Same question as Variants A/B.
3. **Card-effect guarantees** — leave Bird tail at +2 bars and Reptile back at +3 bars, or scale to match the new bars ceiling?
4. **The 5 band-boundary cliffs.** Variant C jumps the boost by +5% at morale 40, 45, 50, 55, 60. Buff cards that cross those boundaries are now non-linearly more valuable than buff cards that move within a band. Is that intended (encouraging band-targeted team building) or a balance concern (introducing 5 sharp thresholds players will optimize toward)?
5. **Bar depletion** — under the new table a morale-60 Axie gets 6 bars; with `-2 on entry, -1 per turn` that's still only 5 turns of Last Stand uptime. Acceptable, or should depletion also scale per band?
6. **The morale-39 → morale-40 cliff specifically.** This is the cliff where buffing matters most (the Axie goes from no boost to +5% and the table is also adding the first bar after the entry tier). If buff cards commonly target Axies in the 35–39 range, designers may want to verify the cliff feels good rather than artificially binary.
7. **Rust-engine versioning** — confirm replay determinism survives the formula change, or version replays so old ones resolve with the old formula.
