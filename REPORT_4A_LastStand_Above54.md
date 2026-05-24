# Report 4A — Last Stand: New Bars Table and Trigger Formula (Variant A: Boost Above Morale 54)

**Date:** 2026-05-23
**Scope:** Two related changes to the Rust battle engine:
1. Replace the morale → Last Stand bars mapping with the user's proposed table.
2. Change the Last Stand activation formula so triggering is biased upward at high morale, **with the boost beginning above morale 54 and scaling linearly to +30% at morale 64**.

Variant B (boost beginning above morale 50) is covered in a separate report.

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

It's a **sublinear** curve — bars grow slowly with morale. Working through the user's table-range:

| Morale | morale × 1.4 − 30 | sqrt | ÷ 2 | floor → bars |
|---|---|---|---|---|
| 30 | 12 | 3.46 | 1.73 | **1** |
| 34 | 17.6 | 4.20 | 2.10 | **2** |
| 35 | 19.0 | 4.36 | 2.18 | **2** |
| 39 | 24.6 | 4.96 | 2.48 | **2** |
| 40 | 26.0 | 5.10 | 2.55 | **2** |
| 44 | 31.6 | 5.62 | 2.81 | **2** |
| 45 | 33.0 | 5.74 | 2.87 | **2** |
| 49 | 38.6 | 6.21 | 3.11 | **3** |
| 50 | 40.0 | 6.32 | 3.16 | **3** |
| 54 | 45.6 | 6.75 | 3.38 | **3** |
| 55 | 47.0 | 6.86 | 3.43 | **3** |
| 59 | 52.6 | 7.25 | 3.63 | **3** |
| 60 | 54.0 | 7.35 | 3.67 | **3** |
| 64 | 59.6 | 7.72 | 3.86 | **3** |
| 70 | 68.0 | 8.25 | 4.12 | **4** |

Today, even at morale 64, an Axie only gets 3 bars. The user's proposed table tops out at 6 bars at morale 60–64 — double the current ceiling at that morale band.

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

There is no random roll. Whether Last Stand triggers is determined entirely by the size of the overkill relative to morale. "Chance" varies between battles only because incoming damage varies between battles. This matters for the formula change below: "20% more often" cannot literally mean "+20% to a probability" because there is no probability in the current code. The cleanest interpretation, kept below, is "the morale ceiling above which an Axie can no longer Last Stand is 20–30% higher" — which is what the existing engine treats morale as.

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

Bars are not refilled by healing — `battle.rs` lines 612–613:

```rust
if fighter_state.is_in_last_stand() && new_hp > 0 {
    return Ok(());
}
```

Last Stand ends when `last_stand_turns` reaches 0 (the Axie dies) or a card disables it.

### 1.4 Card abilities that override the formula

These are independent of the change in this report and must keep working:

- **Bird tail PostFightII** (`card/bird.rs:1149–1175`): drains all remaining HP and forces Last Stand with **exactly 2 bars** via `.guarantee_last_stand(Some(2))`.
- **Reptile back BoneSailII** (`card/reptile.rs:758–773`): on lethal incoming damage, forces Last Stand with **exactly 3 bars** via `.guarantee_last_stand(Some(3))`.
- **Aquatic horn ShoalStar** (`card/aquatic.rs:575–589`): sets `context.turn.cannot_last_stand = true`, preventing the next damage's Last Stand from triggering.
- **Beast wing Sting** (`card/beast.rs:632–636`): same `cannot_last_stand = true` mechanism.

**Design heads-up:** the Bird's "guaranteed 2 bars" and Reptile's "guaranteed 3 bars" were balanced against a world where Last Stand maxed out at ~3 bars naturally. Under the new table a morale-60 Axie naturally Last-Stands with **6** bars. These guarantees shrink in relative power. The design team should decide whether to bump these guaranteed amounts when the new table ships.

### 1.5 Morale stat range in the game

From `model.rs` and the body-part class deltas:

- Base morale starts at `35 + (delta.morale_delta.round() × 4)` (`model.rs:49–54`).
- Body-part classes contribute morale: Beast / Bug = +3 per part, Bird / Plant = +1 per part, Aquatic / Reptile = +0 per part (`model.rs:64–72`).
- Observed morale values in test data (`lua/plant_beast_aqua.lua`, `lua/plant_poison.lua`) are in the 27–68 range.

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

Verification across the table:

| Morale | (morale − 30) / 5 | floor | Matches table |
|---|---|---|---|
| 30 | 0.0 | 0 | ✓ (≤ 34 → 0) |
| 34 | 0.8 | 0 | ✓ |
| 35 | 1.0 | 1 | ✓ |
| 39 | 1.8 | 1 | ✓ |
| 40 | 2.0 | 2 | ✓ |
| 44 | 2.8 | 2 | ✓ |
| 45 | 3.0 | 3 | ✓ |
| 49 | 3.8 | 3 | ✓ |
| 50 | 4.0 | 4 | ✓ |
| 54 | 4.8 | 4 | ✓ |
| 55 | 5.0 | 5 | ✓ |
| 59 | 5.8 | 5 | ✓ |
| 60 | 6.0 | 6 | ✓ |
| 64 | 6.8 | 6 | ✓ |
| 65 | 7.0 | 7 | (above the table — see §2.4) |

### 2.3 Rust implementation — drop-in replacement for `state.rs:254–255`

```rust
pub fn get_last_stand_turns(morale: i32) -> i32 {
    if morale < 35 {
        return 0;
    }
    (morale - 30) / 5
}
```

Integer division in Rust truncates toward zero for non-negative input, which is exactly `floor` for positive arguments. The `morale < 35` guard makes the table's "≤ 34 → 0" explicit.

### 2.4 Two design knobs to decide before merging

1. **Cap above morale 64.** The formula above extends naturally: morale 65–69 → 7 bars, etc. The user's table stops at 60–64 → 6. To hold the ceiling at 6:

    ```rust
    pub fn get_last_stand_turns(morale: i32) -> i32 {
        if morale < 35 { return 0; }
        ((morale - 30) / 5).min(6)
    }
    ```

2. **Card-effect guarantees** — leave Bird tail at `+2` and Reptile back at `+3` (now relatively weaker), or scale them up to match the new bars ceiling? They bypass `get_last_stand_turns` entirely (`card/bird.rs:1161`, `card/reptile.rs:770`).

---

## 3. Change 2 — The trigger formula (Variant A: boost above morale 54)

The user wants Last Stand to "trigger 20% more often" above morale 54, scaling linearly to "+30% at morale 64". Because the existing trigger is deterministic, the only well-defined reading of that is: **the effective morale used in the inequality is scaled up by +20–30% in the chosen band**.

This preserves the existing engine semantics — Last Stand remains deterministic given a damage roll — and adds only a single per-call multiplier.

### 3.1 The math

Let the morale boost percentage at a given morale m be:

```
boost%(m) =
    0                                                     if m ≤ 54
    20 + (m − 55) × (30 − 20) / (64 − 55)                 if 54 < m ≤ 64
    30                                                    if m > 64
```

This is a linear interpolation that hits +20% at the first morale point above 54 (i.e. morale 55) and +30% at morale 64, holding at +30% above 64.

Apply the boost to morale before the deterministic check:

```
effective_morale(m) = m × (1 + boost%(m) / 100)
```

Then the new trigger condition is just the old one with `effective_morale` substituted in:

```
(damage − current_HP) × 100  ≤  current_HP × effective_morale
```

### 3.2 Verification — what this means at concrete morale values

| Morale | boost% | effective morale | Survives overkill up to | Δ vs current |
|---|---|---|---|---|
| 50 | 0% | 50 | 50% of HP | — |
| 54 | 0% | 54 | 54% of HP | — |
| 55 | 20.0% | 66.0 | 66% of HP | +12% HP equivalent |
| 56 | ~21.1% | 67.8 | 67.8% of HP | |
| 60 | ~25.6% | 75.3 | 75.3% of HP | |
| 64 | 30.0% | 83.2 | 83.2% of HP | +19.2% HP equivalent |
| 70 | 30.0% (capped) | 91.0 | 91% of HP | |

### 3.3 Worked example

Axie at 100 current HP, 60 morale, takes 175 damage (75 overkill).

- **Current formula:** `75 × 100 ≤ 100 × 60` → `7500 ≤ 6000` → false → Axie dies.
- **Variant A new:** boost%(60) ≈ 25.6%. effective_morale ≈ 75.3. Check: `75 × 100 ≤ 100 × 75.3` → `7500 ≤ 7530` → true → **Last Stand triggers**.

The morale-60 Axie now survives a hit it would have died to under the current formula. Combined with the new 6-bar grant from §2, this means a morale-60 Axie also gets 6 turns of Last Stand instead of dying outright — a large net buff at high morale, which matches the user's stated intent ("scale heavily starting at 54 morale").

### 3.4 Rust implementation — drop-in replacement for `state.rs:247–251`

```rust
pub fn can_last_stand(&self, morale: i32, decrement: i32, guarantee_last_stand: bool) -> bool {
    if self.is_chimera || decrement < self.hp {
        return false;
    }
    if guarantee_last_stand && self.hp > 0 {
        return true;
    }
    let effective_morale = apply_morale_boost_a(morale);
    (decrement - self.hp) as i64 * 100 <= self.hp as i64 * effective_morale as i64
}

/// Variant A: +20% at morale 55, scaling linearly to +30% at morale 64.
fn apply_morale_boost_a(morale: i32) -> i32 {
    if morale <= 54 {
        return morale;
    }
    let m = morale.min(64);
    // boost_x100 ranges from 20 (at m=55) to 30 (at m=64). Integer math: ((m-55) * 10) / 9.
    let boost_x100: i32 = 20 + ((m - 55) * 10) / 9;
    morale + (morale * boost_x100) / 100
}
```

Two implementation notes:

- The integer-math version is exact at the endpoints (`boost_x100 = 20` at morale 55, `boost_x100 = 30` at morale 64) and is monotone everywhere in between, with at most 1-percentage-point rounding error at intermediate morales. If exact floating-point boost is needed, use `f64` arithmetic and round at the end.
- The cast to `i64` in `can_last_stand` prevents the rare overflow case where `decrement × 100` or `current_HP × effective_morale` could exceed `i32::MAX`.

### 3.5 What is NOT changed by this

- The Chimera exclusion still applies.
- The "damage must be lethal" prerequisite still applies.
- The `guarantee_last_stand` card-override path still bypasses the morale check.
- `cannot_last_stand` (set by Aquatic ShoalStar and Beast Sting) still blocks Last Stand entirely.
- Bar depletion in `round_runner.rs:616–625` is unchanged: −2 bars on entry, −1 per subsequent turn.
- The "no healing out of Last Stand" rule in `battle.rs:612–613` is unchanged.

---

## 4. Why Variant A over Variant B?

Variant A is the **more conservative** choice. It leaves the mid-morale band (50–54) at current behaviour and only buffs the genuinely-high-morale Axies. Pairs cleanly with the new bars table: morale 55–59 (which now grants 5 bars) is also the band where the trigger boost begins. The two changes describe the same band.

The user's stated language — "scale heavily starting at 54 morale" — matches Variant A by name.

For a more aggressive change that pulls in morale 51–54 as well, see Report 4B.

---

## 5. Code surface — files to touch

Engine (`game-server-main/lunacia/src/`):

1. `state.rs:254–255` — replace `get_last_stand_turns` with the new formula from §2.3.
2. `state.rs:247–251` — replace `can_last_stand` with the Variant A body from §3.4, and add the `apply_morale_boost_a` helper alongside.
3. Optional: bump `Some(2)` in `card/bird.rs:1161` and `Some(3)` in `card/reptile.rs:770` if the design team decides those guarantees should scale.

Tests:

4. Add unit tests for `get_last_stand_turns` covering boundary morales (34/35, 39/40, 44/45, 49/50, 54/55, 59/60, 64/65) and for `can_last_stand` covering the boost boundaries (54/55) and the morale-64 ceiling.

Client mirror:

5. `axie-classic/Assets/Scripts/Models/Rust/FighterState.cs` — the `LastStandTurns` field at line 31 receives the value from the engine; no formula change is needed client-side. Confirm any UI that previews "expected bars" reflects the new linear table.

Documentation:

6. `.codemap/card-battle-core.yaml`, `.codemap/card-battle-data.yaml`, `.codemap/models-rust.yaml` — refresh.
7. `game-server-main/ARCHITECTURE.md` (Last Stand trigger phase, around line 259) — refresh wording.

---

## 6. Open questions for the design team

1. **Cap above morale 64?** Let the new bars formula extend naturally (morale 70 → 8 bars) or clamp at 6.
2. **Card-effect guarantees** — leave Bird tail at +2 bars and Reptile back at +3 bars, or scale them with the new table?
3. **Bar-depletion rate** — under the new table a morale-60 Axie gets 6 bars; with the current `-2 on entry, -1 per turn` rule that's still only 5 turns of Last Stand uptime. Acceptable, or should depletion also scale?
4. **Buff-driven morale spikes** — `buff.rs:114–122` lets cards push morale above the natural cap. A buff that bumps morale from 54 → 55 jumps the Axie straight from no boost to +20% boost. Is that the desired feel, or should the curve cross 0% more gradually?
5. **Rust-engine versioning** — battle replays serialize old state. Confirm replay determinism survives the formula change, or version replays so old ones resolve with the old formula.
