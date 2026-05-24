# Axie Classic -- Morale Stat Revamp Proposals

## Current State of Morale

Morale is one of four Axie stats (HP, Speed, Skill, Morale) and currently governs:

1. **Critical hit chance** -- Pseudo-random distribution based on `morale^2 / (sqrt(46) * 80)`, with increasing chance on consecutive non-crits
2. **Critical hit damage** -- `min(100 + sqrt(morale) * 10 + morale * 0.4 - 18, 200)%`, hard capped at 200%
3. **Last Stand entry** -- Axie enters Last Stand if overkill satisfies `(damage - hp) * 100 <= hp * morale`
4. **Last Stand ticks** -- `floor(sqrt(morale * 1.4 - 30) / 2)`

### Morale Values by Class

| Class | Base Morale | Per-Part Bonus | Pure (6 parts) | Morale Role |
|-------|------------|----------------|-----------------|-------------|
| Beast | 43 | +3 | 61 | Highest morale class. Crit/Last Stand archetype. |
| Bug | 39 | +3 | 57 | Second highest. Defensive morale user (shields + Last Stand). |
| Bird | 35 | +1 | 41 | Moderate. Fast attacker with some crit upside. |
| Plant | 35 | +1 | 41 | Moderate. Tank that occasionally benefits from Last Stand. |
| Reptile | 35 | +0 | 35 | Moderate base, no part scaling. Off-tank. |
| Dawn | 31 | +0 | 31 | Low. Speed/Skill hybrid, morale is a dump stat. |
| Dusk | 31 | +0 | 31 | Low. HP/Speed hybrid, morale is a dump stat. |
| Aquatic | 27 | +0 | 27 | Lowest. Speed-focused, morale is irrelevant. |
| Mech | 27 | +0 | 27 | Lowest. Skill-focused, morale is irrelevant. |

*Note: Mech, Dawn, and Dusk are secret classes with no class-specific parts. Their parts come from parent classes (Beast+Bug for Mech, Bird+Aquatic for Dawn, Plant+Reptile for Dusk), so effective morale depends on part selection. A Mech with 6 Beast parts would have 27 + 18 = 45 morale.*

### Current Crit Chance (Expected Probability)

Using the formula `round(morale^2 / (sqrt(46) * 80)) / 100`:

| Morale | Expected Crit % |
|--------|----------------|
| 27 | ~1.3% |
| 31 | ~1.8% |
| 35 | ~2.3% |
| 39 | ~2.8% |
| 41 | ~3.1% |
| 43 | ~3.4% |
| 57 | ~6.0% |
| 61 | ~6.9% |

These numbers are extremely low. Even a pure Beast (61 morale) crits less than 7% of the time on average. The pseudo-random distribution increases the chance after consecutive non-crits, but the base rates are far too low for crits to be a reliable strategy.

### Current Crit Damage Multiplier

Using `min(100 + sqrt(morale) * 10 + morale * 0.4 - 18, 200)`:

| Morale | Crit Multiplier |
|--------|----------------|
| 27 | 144% |
| 31 | 150% |
| 35 | 155% |
| 41 | 164% |
| 43 | 167% |
| 57 | 181% |
| 61 | 186% |

The 200% cap is never reached within normal morale ranges. Even pure Beast (61 morale) only gets 186%.

### Current Last Stand Ticks

Using `floor(sqrt(morale * 1.4 - 30) / 2)`:

| Morale | Last Stand Ticks |
|--------|-----------------|
| 27 | 1 |
| 31 | 2 |
| 35 | 2 |
| 41 | 3 |
| 43 | 3 |
| 57 | 3 |
| 61 | 4 |

The range is incredibly compressed: 1 to 4 ticks across the entire morale spectrum. The difference between 43 morale (Beast base) and 61 morale (pure Beast with max investment) is just 1 tick.

### Why Morale Underperforms

1. **Crit chance is negligible.** Sub-7% even at maximum morale. Crits are a pleasant surprise, never a strategy.
2. **Crit damage cap is unreachable.** The 200% cap exists but no morale value comes close. The formula peaks at 186% for pure Beast.
3. **Last Stand ticks are compressed.** 1-4 ticks across the full range. Investing 18 morale (pure Beast parts) buys you 1 extra tick over base Beast.
4. **Morale has no impact until the Axie is nearly dead** (Last Stand) or gets lucky (crit). Speed matters every round. HP matters every hit. Morale might matter once per game.
5. **No morale-focused class fantasy is viable.** Beast and Bug are the morale classes, but neither can build around morale because the payoffs are too small and unreliable.
6. **Low-morale classes don't miss it.** Aquatic (27) and Mech (27) function perfectly well without meaningful morale. The stat could be removed from their sheet and they'd barely notice.

---

## Proposal 1: Exponential Last Stand Scaling

### Core Idea
Make Morale scale non-linearly for Last Stand ticks so high-morale Axies have dramatically more Last Stand bars. Low-morale Axies keep their 1-2 ticks; high-morale Axies get a true "second phase" of combat.

### Current vs Proposed Last Stand Ticks

| Morale | Class Examples | Current Ticks | Proposed Ticks | Change |
|--------|---------------|--------------|----------------|--------|
| 27 | Aquatic, Mech | 1 | 1 | -- |
| 31 | Dawn, Dusk | 2 | 1 | -1 |
| 35 | Reptile | 2 | 2 | -- |
| 39 | Bug (base) | 2 | 3 | +1 |
| 41 | Bird (pure), Plant (pure) | 3 | 3 | -- |
| 43 | Beast (base) | 3 | 4 | +1 |
| 45 | Mech (Beast parts) | 3 | 4 | +1 |
| 50 | Beast (mixed parts) | 3 | 5 | +2 |
| 57 | Bug (pure) | 3 | 7 | +4 |
| 61 | Beast (pure) | 4 | 8 | +4 |

### Formula
```
base_ticks = floor((morale - 25) / 5)
bonus_ticks = floor(max(0, morale - 50) / 3)
total_ticks = base_ticks + bonus_ticks
```

### Impact Across Classes

**High-morale classes (primary beneficiaries):**
- **Pure Beast (61):** 8 ticks. An entire second phase of combat. Enough actions to draw into a combo, play 2-3 cards, and threaten real damage from Last Stand.
- **Pure Bug (57):** 7 ticks. Bug becomes a Last Stand specialist -- its shield-heavy cards can stall even longer while in Last Stand.
- **Beast base (43):** 4 ticks. A meaningful upgrade from 3. Even without maxing morale through parts, Beast gets a useful Last Stand.

**Mid-morale classes (moderate impact):**
- **Bird/Plant pure (41):** 3 ticks. Same as current. These classes aren't morale-focused and don't gain or lose from this change.
- **Bug base (39):** 3 ticks. Slight gain. Bug benefits more from part investment toward 57.
- **Reptile (35):** 2 ticks. Same as current. Reptile's identity is HP/shields, not Last Stand.

**Low-morale classes (minimal change):**
- **Dawn/Dusk (31):** 1 tick. Actually loses 1 tick vs current. These classes have no morale investment and shouldn't rely on Last Stand.
- **Aquatic/Mech (27):** 1 tick. Same as current. Last Stand is not their game.

### Last Stand Entry Threshold
Scale the entry threshold with morale so high-morale Axies are more likely to enter Last Stand:
- Current formula: enter if `(damage - hp) * 100 <= hp * morale`
- This already scales with morale. At 61 morale, overkill can be up to 61% of remaining HP. At 27 morale, only 27%. No change needed here -- the existing formula already rewards morale investment.

### Interaction with Existing Mechanics
- **Chill** prevents Last Stand entry entirely -- remains the primary counter regardless of tick count
- **Lethal** kills through Last Stand -- hard counter to any Last Stand strategy
- **Last Stand tick reduction:** Defenders hit in Last Stand lose 2 ticks per attack. At 8 ticks, a pure Beast survives ~4 hits in Last Stand -- meaningful but not infinite
- **High burst** past the entry threshold still bypasses Last Stand entirely

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Beast/Bug Last Stand stalling becomes oppressive | High | Chill and Lethal remain hard counters; tick reduction on hit (-2) limits survival |
| Pure Bug (7 ticks) + shield cards = unkillable | High | Last Stand HP per tick is very low; shields in Last Stand are a known balance lever |
| Low-morale classes losing ticks feels punishing | Medium | Dawn/Dusk/Aquatic/Mech don't build around Last Stand; 1 tick is sufficient for their playstyle |
| Games drag with long Last Stand sequences | Medium | Each tick is ~1 action; 8 ticks is 8 actions max, not 8 full rounds |
| Makes morale parts mandatory for Beast/Bug | Low | Acceptable -- morale IS their class identity |

---

## Proposal 2: Morale as a Resource (Rage System)

### Core Idea
Convert Morale from a passive stat into an active resource called "Rage" that all Axies generate based on their Morale stat. High-morale Axies generate more Rage and can spend it on powerful combat effects. Low-morale Axies generate minimal Rage and interact with the system only occasionally.

### Rage Generation
```
rage_per_round = floor(morale / 10)
max_rage_pool  = morale
rage_decay     = 50% of unspent rage at end of round (floor)
```

| Morale | Class Examples | Rage/Round | Max Pool | Effective Accumulation |
|--------|---------------|-----------|----------|----------------------|
| 27 | Aquatic, Mech | 2 | 27 | Decays too fast to accumulate; can trigger 8-cost ability every ~4 rounds |
| 31 | Dawn, Dusk | 3 | 31 | Slow; can trigger 8-cost ability every ~3 rounds |
| 35 | Reptile, Bird (base), Plant (base) | 3 | 35 | Moderate; occasional 12-cost ability access |
| 39 | Bug (base) | 3 | 39 | Moderate; 15-cost ability reachable in ~4 rounds |
| 41 | Bird (pure), Plant (pure) | 4 | 41 | Steady; 15-cost ability in ~3 rounds |
| 43 | Beast (base) | 4 | 43 | Good; 20-cost ability reachable in ~4 rounds |
| 57 | Bug (pure) | 5 | 57 | Strong; 25-cost ability reachable in ~4 rounds |
| 61 | Beast (pure) | 6 | 61 | Maximum; 25-cost ability in ~3 rounds |

### Rage Abilities (Spend Options)

| Ability | Rage Cost | Effect | Who Can Realistically Use It |
|---------|-----------|--------|------------------------------|
| Focused Strike | 8 | Next card this Axie plays is a guaranteed critical hit | All classes (every 3-4 rounds) |
| Ferocity | 12 | Next card this Axie plays deals +50% damage | 35+ morale classes (every 3-4 rounds) |
| Defiant Stand | 15 | Guaranteed Last Stand entry this round (ignores overkill threshold) | 39+ morale classes (every 3-4 rounds) |
| Undying Fury | 20 | Gain +2 Last Stand ticks | 43+ morale classes (every 4 rounds) |
| Berserker's Roar | 25 | All cards this Axie plays this round deal +30% damage and guaranteed crit | 57+ morale (Bug/Beast pure) only realistically |

### Usage Rules
- Rage is spent during the planning phase, before cards are committed
- Multiple Rage abilities can be activated per round if the Axie has enough Rage
- Rage is visible to the opponent (informed counterplay)
- Rage resets to 0 on Last Stand entry
- Rage generation continues in Last Stand (high-morale Axies can use Rage abilities during their Last Stand ticks)

### Class-Specific Implications

**Beast (43-61 morale):** Primary Rage user. At 61 morale, generates 6 Rage/round and can afford Berserker's Roar (25 cost) regularly. Rage gives Beast a reliable way to guarantee crits without depending on the base crit formula.

**Bug (39-57 morale):** Secondary Rage user. At 57 morale, generates 5 Rage/round. Bug's defensive playstyle means it can save Rage for Defiant Stand (guaranteed Last Stand) or Undying Fury (+2 ticks), complementing its shield-heavy cards.

**Bird/Plant (35-41 morale):** Moderate Rage users. At 41 morale (pure), generates 4 Rage/round. Occasional Focused Strike (8 cost, guaranteed crit) gives Birds a burst tool and Plants an unexpected offensive option.

**Reptile (35 morale):** Minimal Rage. Generates 3/round. Can trigger Focused Strike occasionally but rarely reaches higher-tier abilities. Reptile's identity remains HP/shields, not Rage.

**Aquatic/Mech (27 morale):** Near-zero Rage impact. Generates 2/round, most of which decays. Can afford Focused Strike every ~4 rounds. The system exists for them but doesn't define their play.

**Dawn/Dusk (31 morale):** Low Rage. Generates 3/round. Similar to Reptile -- occasional access to cheap abilities, but not a core mechanic.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Complexity increase for all players | High | Rage is optional; low-morale classes can ignore it with minimal loss |
| UI/UX burden (new resource bar, spend interface) | High | Requires significant client work; Rage bar visible on each Axie |
| Beast guaranteed crits via Rage removes crit counterplay | Medium | Rage is visible; opponent can play around known Rage spending |
| Bug with Defiant Stand + shields becomes unkillable | Medium | Defiant Stand costs 15 Rage; Bug can afford it every ~3 rounds, not every round |
| Low-morale classes feel excluded from a major system | Medium | They aren't excluded -- Focused Strike at 8 cost is accessible to all. They just don't define their strategy around it. |

---

## Proposal 3: Morale-Driven Crit Overhaul

### Core Idea
Dramatically overhaul the crit system so that Morale creates a clear, felt difference in crit behavior across all classes. High-morale Axies crit reliably with meaningful damage bonuses. Mid-morale Axies crit occasionally. Low-morale Axies almost never crit. Make crits a core mechanic, not a background probability.

### New Crit Chance Formula
Replace the current `morale^2 / (sqrt(46) * 80)` formula (which produces sub-7% even at max morale) with a linear, aggressive scaling:
```
crit_chance = max(0, (morale - 25) * 2.5%)
```

| Morale | Class Examples | Current Crit % | New Crit % | Change |
|--------|---------------|---------------|------------|--------|
| 27 | Aquatic, Mech | ~1.3% | 5% | +3.7% |
| 31 | Dawn, Dusk | ~1.8% | 15% | +13.2% |
| 35 | Reptile, Bird/Plant base | ~2.3% | 25% | +22.7% |
| 39 | Bug (base) | ~2.8% | 35% | +32.2% |
| 41 | Bird (pure), Plant (pure) | ~3.1% | 40% | +36.9% |
| 43 | Beast (base) | ~3.4% | 45% | +41.6% |
| 57 | Bug (pure) | ~6.0% | 80% | +74.0% |
| 61 | Beast (pure) | ~6.9% | 90% | +83.1% |

### New Crit Damage Formula
Remove the 200% cap. Replace with morale-scaled crit damage:
```
crit_damage_multiplier = 1.0 + (morale * 0.8 / 100)
```

| Morale | Class Examples | Current Crit Damage | New Crit Damage |
|--------|---------------|--------------------| ----------------|
| 27 | Aquatic, Mech | 144% | 121.6% |
| 31 | Dawn, Dusk | 150% | 124.8% |
| 35 | Reptile | 155% | 128.0% |
| 39 | Bug (base) | 159% | 131.2% |
| 41 | Bird (pure), Plant (pure) | 164% | 132.8% |
| 43 | Beast (base) | 167% | 134.4% |
| 57 | Bug (pure) | 181% | 145.6% |
| 61 | Beast (pure) | 186% | 148.8% |

Per-hit crit damage is lower, but crits happen 10-15x more often. The expected damage over many attacks is far higher and far more consistent.

### Expected Damage per 100 ATK Card (Accounting for Crit Rate)

| Morale | Class Examples | Current Expected | New Expected | Change |
|--------|---------------|-----------------|--------------|--------|
| 27 | Aquatic, Mech | 100.6 | 101.1 | +0.5% |
| 31 | Dawn, Dusk | 100.9 | 103.7 | +2.8% |
| 35 | Reptile | 101.3 | 107.0 | +5.6% |
| 39 | Bug (base) | 101.7 | 110.9 | +9.1% |
| 41 | Bird (pure), Plant (pure) | 102.0 | 113.1 | +10.9% |
| 43 | Beast (base) | 102.3 | 115.5 | +12.9% |
| 57 | Bug (pure) | 104.9 | 136.5 | +30.1% |
| 61 | Beast (pure) | 106.0 | 143.9 | +35.8% |

At current values, morale contributes almost nothing to expected damage (100.6 to 106.0 across the full range -- a 5.4% spread). With the new formula, the spread is 101.1 to 143.9 -- a 42.3% gap between Aquatic and pure Beast. Morale now matters enormously.

### Crit Side Effect: "Shaken" Debuff
When a critical hit lands, apply **Shaken** to the target for 1 turn:
- Target's shield effectiveness is reduced by 20%
- Does not stack (refreshes duration)

**Class implications:**
- Pure Beast (90% crit): Shaken is effectively permanent on its target. Plant/Reptile shields are 20% weaker against Beast at all times.
- Pure Bug (80% crit): Near-permanent Shaken. Bug's own shields aren't affected (Shaken is on the target, not self).
- Bird/Plant pure (40% crit): Shaken applied about half the time. Noticeable but not reliable.
- Aquatic/Mech (5% crit): Almost never applies Shaken. Irrelevant.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| 90% crit rate makes Beast damage output too high | High | Per-crit multiplier is lower (148.8% vs 186%); total expected is +35.8%, which is significant but within balance range |
| Pure Bug at 80% crit with shield-heavy cards is oppressive | High | Bug crits add damage but Bug's ATK values are generally moderate; monitor Bug burst combos |
| Mid-morale classes (Bird/Plant at 40%) have high variance | Medium | 40% is high enough to plan around; pseudo-random distribution smooths streaks |
| Low-morale classes (Aquatic/Mech) lose what little crit they had | Low | They had 1-3% crit; 5% is actually higher. They never built around crits anyway. |
| Shaken + high crit rate invalidates shield strategies | Medium | 20% reduction, not removal. Shields still absorb 80% effectiveness. Test at 15% if too strong. |

---

## Proposal 4: Morale as Tenacity (Scaling Damage Reduction)

### Core Idea
Redefine Morale as "fighting spirit" -- a stat that makes high-morale Axies progressively harder to kill as they lose HP. Instead of being a crit/Last Stand stat, Morale provides scaling damage reduction based on missing HP. Every class benefits at low health, but high-morale classes benefit dramatically more.

### Damage Reduction Formula
```
hp_percent = current_hp / max_hp
dr_scaling = (1.0 - hp_percent) * (morale / 100)
damage_reduction = min(dr_scaling, 0.60)  // hard cap at 60%
```

### Damage Reduction by HP Threshold (All Classes)

| HP % | Aqua/Mech (27) | Dawn/Dusk (31) | Reptile (35) | Bug base (39) | Bird/Plant pure (41) | Beast base (43) | Bug pure (57) | Beast pure (61) |
|------|---------------|----------------|--------------|---------------|---------------------|----------------|---------------|----------------|
| 100% | 0% | 0% | 0% | 0% | 0% | 0% | 0% | 0% |
| 75% | 6.8% | 7.8% | 8.8% | 9.8% | 10.3% | 10.8% | 14.3% | 15.3% |
| 50% | 13.5% | 15.5% | 17.5% | 19.5% | 20.5% | 21.5% | 28.5% | 30.5% |
| 25% | 20.3% | 23.3% | 26.3% | 29.3% | 30.8% | 32.3% | 42.8% | 45.8% |
| 10% | 24.3% | 27.9% | 31.5% | 35.1% | 36.9% | 38.7% | 51.3% | 54.9% |
| Last Stand | 16.2% | 18.6% | 21.0% | 23.4% | 24.6% | 25.8% | 34.2% | 36.6% |

Last Stand DR is reduced to 2/3 of the 10% HP value to prevent infinite stalling.

### Class-Specific Implications

**Beast (43-61 morale):** The primary beneficiary. A pure Beast at 25% HP takes 45.8% less damage -- a 100-damage card deals only 54. This gives Beast the time window it desperately needs to draw into combos. Beast's low HP (31 base) means it reaches low HP% faster, so the DR kicks in sooner in absolute terms but later in relative terms.

**Bug (39-57 morale):** Second-highest benefit. Pure Bug at 25% HP: 42.8% DR. Combined with Bug's shield-heavy cards, this creates a very durable late-game Axie. Bug's identity shifts slightly toward "unkillable cockroach" which fits thematically.

**Plant (35-41 morale):** Moderate benefit. Pure Plant at 25% HP: 30.8% DR. Combined with Plant's high HP (47 base, pure 65), Plant reaches 25% HP late in the fight but becomes noticeably harder to finish off. This layers on top of Plant's shield cards.

**Bird (35-41 morale):** Moderate benefit but less relevant. Bird has low HP (27 base) and is often one-shot or killed before DR matters. Pure Bird at 25% HP: 30.8% DR, but 25% of Bird's HP pool is very few hit points.

**Reptile (35 morale):** Moderate. Similar to Plant but with slightly less DR (26.3% at 25% HP). Reptile's high HP means it benefits from DR in the mid-game.

**Aquatic (27 morale):** Minimal. 20.3% DR at 25% HP. Aquatic relies on speed and burst, not tenacity. The DR is noticeable but doesn't change Aquatic's playstyle.

**Mech (27 morale):** Minimal. Same as Aquatic. Mech is a glass cannon; 20.3% DR at 25% HP doesn't compensate for its low HP (27 base).

**Dawn/Dusk (31 morale):** Low. 23.3% at 25% HP. Minor survivability boost but not build-defining.

### Interaction with Existing Mechanics
- **Shield** applies before DR. Damage hits shield first; remaining damage is reduced by DR, then applied to HP.
- **True damage** (Poison ticks, percentage-based effects) ignores DR entirely.
- **Healing** interacts interestingly: healing an Axie increases HP%, which *reduces* DR. This creates a tension -- do you heal the Axie (restoring raw HP but losing DR) or let it stay low (keeping DR but risking death)?
- **Last Stand** has reduced DR (2/3) to prevent stalling.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Bug/Beast + shields at low HP becomes unkillable | High | 60% DR hard cap; Last Stand DR reduced to 2/3; true damage bypasses DR entirely |
| Plant (high HP + moderate DR + shields) stalls forever | High | Plant's DR is moderate (30.8% at 25% pure); reaching 25% HP on a high-HP Axie takes significant damage |
| Poison/true damage becomes the only viable kill method | Medium | DR only reaches extreme values (50%+) at very low HP on very high morale; burst past the DR threshold still works |
| Bird/Aquatic/Mech barely benefit, widening class gap | Medium | These classes have other strengths (speed, burst, skill). Not every class needs to benefit equally from every stat. |
| Hard to communicate DR percentage to players | Medium | Show "Tenacity" icon with percentage on health bar when DR > 10% |

---

## Proposal 5: Morale Aura (Team-Wide Morale Influence)

### Core Idea
Make Morale a partially team-shared stat. Every Axie projects a "Morale Aura" based on its Morale value that increases allies' effective Morale for crit and Last Stand calculations. When a high-morale Axie dies, the team suffers a "Morale Break" debuff. This turns Morale into a strategic team-building and positioning stat that affects all three Axies, not just the high-morale one.

### Aura Mechanics
```
aura_value = floor(morale / 5)
effective_morale = base_morale + sum(ally_auras)
// Aura does NOT count toward the projector's own morale
// Aura does NOT feed back into aura calculations (no recursion)
aura_received_cap = 15  // max morale received from allies
```

### Aura Values by Class

| Class | Base Morale | Pure Morale | Aura Projected (base) | Aura Projected (pure) |
|-------|------------|-------------|----------------------|----------------------|
| Beast | 43 | 61 | 8 | 12 |
| Bug | 39 | 57 | 7 | 11 |
| Bird | 35 | 41 | 7 | 8 |
| Plant | 35 | 41 | 7 | 8 |
| Reptile | 35 | 35 | 7 | 7 |
| Dawn | 31 | 31 | 6 | 6 |
| Dusk | 31 | 31 | 6 | 6 |
| Aquatic | 27 | 27 | 5 | 5 |
| Mech | 27 | 27 | 5 | 5 |

### Team Composition Examples

**Example 1: Beast / Plant / Aquatic (standard comp)**

| Axie | Base Morale | Aura Received | Effective Morale | Impact |
|------|------------|---------------|------------------|--------|
| Pure Beast (61) | 61 | Plant 8 + Aqua 5 = 13 | 74 | Beast crits even harder; more Last Stand ticks |
| Pure Plant (41) | 41 | Beast 12 + Aqua 5 = 15 (capped) | 56 | Plant gains meaningful crit chance and extra Last Stand |
| Pure Aquatic (27) | 27 | Beast 12 + Plant 8 = 15 (capped) | 42 | Aquatic gains noticeable crit chance (was nearly zero) |

**Example 2: Bug / Bug / Beast (high-morale comp)**

| Axie | Base Morale | Aura Received | Effective Morale | Impact |
|------|------------|---------------|------------------|--------|
| Pure Beast (61) | 61 | Bug 11 + Bug 11 = 15 (capped) | 76 | Maximum morale in the game; extreme crit/Last Stand |
| Pure Bug #1 (57) | 57 | Beast 12 + Bug 11 = 15 (capped) | 72 | Near-Beast level effective morale |
| Pure Bug #2 (57) | 57 | Beast 12 + Bug 11 = 15 (capped) | 72 | Same; entire team operates at 72+ effective morale |

**Example 3: Aquatic / Bird / Mech (low-morale comp)**

| Axie | Base Morale | Aura Received | Effective Morale | Impact |
|------|------------|---------------|------------------|--------|
| Pure Aquatic (27) | 27 | Bird 8 + Mech 5 = 13 | 40 | Meaningful upgrade; crits become possible |
| Pure Bird (41) | 41 | Aqua 5 + Mech 5 = 10 | 51 | Moderate boost; Bird gains crit consistency |
| Mech (27) | 27 | Aqua 5 + Bird 8 = 13 | 40 | Mech gains incidental crit chance |

### Morale Break (Death Penalty)
When an Axie with 40+ base Morale dies (true death, not Last Stand entry):
- All surviving allies receive **Demoralize** for 1 round
- Demoralize: reduce effective Morale by 10 (reduces crit chance and Last Stand threshold)
- Demoralize does not stack; refreshes duration if another high-morale ally dies
- Demoralize does NOT apply if the dying Axie enters Last Stand (Break triggers on final death only)

**Who triggers Morale Break:**

| Class | Base Morale | Triggers Break? |
|-------|------------|----------------|
| Beast | 43 | Yes |
| Bug | 39 | No (base < 40, but pure Bug at 57 would if threshold is base morale) |
| Bird/Plant | 35 | No |
| Reptile | 35 | No |
| Dawn/Dusk | 31 | No |
| Aquatic/Mech | 27 | No |

*Design note: The 40+ threshold means only Beast (43 base) naturally triggers Morale Break. Bug base (39) doesn't qualify. This could be adjusted to 38+ to include Bug, or the threshold could use effective morale (including parts) instead of base morale.*

### Strategic Implications by Class

**Beast:** The premier aura battery. A pure Beast projects 12 aura to each ally -- the highest in the game. Protecting Beast becomes a team priority. Losing Beast means losing 12 effective morale from each surviving ally AND triggering Morale Break (-10 for 1 round). The round after Beast dies, allies are at -22 effective morale from where they were.

**Bug:** Strong aura contributor (11 at pure). Bug's defensive tools (shields, stun) help it stay alive longer, maintaining its aura. Bug/Beast comps project massive combined aura.

**Plant:** Moderate aura (8 at pure). Plant's high HP means it tends to be the last to die -- its aura stays active longest. This is a subtle but consistent team benefit.

**Bird:** Moderate aura (8 at pure). Bird tends to die first (low HP, front-line aggression), so its aura contribution is often brief. This creates a trade-off: Bird provides burst damage but its aura dies early.

**Aquatic/Mech:** Low aura (5). Their contribution is minor, but they *receive* aura from allies, which is more impactful. An Aquatic on a team with Beast goes from 27 to 42 effective morale -- a meaningful jump.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Triple-Beast/Bug comps with 70+ effective morale on everyone | High | Aura cap of 15 prevents unlimited stacking; these comps sacrifice HP/Speed diversity |
| Morale Break feels punishing when losing is already bad | Medium | 1-round duration limits impact; only triggers on final death, not Last Stand |
| Hard to communicate effective morale across the team | Medium | Show aura icon with +value next to each Axie's morale stat |
| Always-include-Beast team building becomes solved | Medium | Aura competes with other team-building needs (type coverage, speed tiers, card synergy) |
| Low-morale comps feel excluded | Low | Even low-morale teams share 5-8 aura each; the system is universal, just more impactful for high-morale teams |

---

## Comparison Matrix

| Criteria | 1. Exponential LS | 2. Rage Resource | 3. Crit Overhaul | 4. Tenacity DR | 5. Morale Aura |
|----------|-------------------|------------------|-------------------|----------------|----------------|
| **Solves "morale doesn't feel impactful"** | Yes (at death) | Yes (every round) | Yes (every attack) | Yes (at low HP) | Yes (always) |
| **When morale matters** | At death | Every round | Every attack | Below 50% HP | While alive |
| **Benefits high-morale classes** | Dramatically | Dramatically | Dramatically | Dramatically | Dramatically |
| **Benefits mid-morale classes** | Moderately | Moderately | Moderately | Moderately | Moderately |
| **Benefits low-morale classes** | Minimally | Minimally | Minimally | Minimally | Moderately (as receivers) |
| **Implementation complexity** | Low | High | Medium | Medium | Medium-High |
| **Client (Unity) changes** | Minor (LS bar count) | Major (Rage UI) | Minor (crit VFX, Shaken) | Medium (DR indicator) | Medium (aura display) |
| **Server (Rust) changes** | Low (formula tweak) | High (new system) | Medium (formulas + Shaken) | Medium (DR calculations) | Medium (aura calculations) |
| **Risk of breaking balance** | Medium | High | Medium-High | Medium-High | Medium |
| **Class identity preserved** | Yes | Yes | Yes | Partially (shifts high-morale to tank) | Yes (enhances team role) |
| **Skill expression added** | Low | High | Low-Medium | Medium | Medium |
| **Counterplay clarity** | High (Chill/Lethal) | Medium (visible Rage) | Medium (anti-crit cards) | Medium (true damage/burst) | High (kill the aura source) |
| **Combinable with others** | Yes (with 3, 4, or 5) | Standalone | Yes (with 1 or 5) | Yes (with 1 or 3) | Yes (with 1 or 3) |

---

## Recommended Combinations

### Pair 1: Exponential LS (1) + Crit Overhaul (3)
High-morale Axies crit reliably AND have a massive Last Stand phase. Two different payoffs at two different game moments. Beast/Bug crit consistently throughout the fight, then enter a long Last Stand where they can continue dealing crit-boosted damage. Low-morale classes (Aquatic, Mech) barely interact with either system.

### Pair 2: Tenacity DR (4) + Morale Aura (5)
High-morale Axies become team anchors -- harder to kill at low HP and buffing allies while alive. Losing the Beast is devastating (lost aura + Morale Break + no more DR benefit). Creates deep strategic tension around protecting vs sacrificing high-morale Axies. Bug benefits strongly from both (DR makes it durable, aura helps the team).

### Pair 3: Exponential LS (1) + Morale Aura (5)
High-morale Axies have massive Last Stand AND empower the team. The aura means even if Beast enters Last Stand, the team already benefited from its presence. If Beast survives Last Stand (unlikely but possible with 8 ticks), the aura continues. Creates a "rally around the Beast" team dynamic. Plant and Aquatic allies gain meaningful crit/LS improvements from Beast's aura.

### Not Recommended Together
- **Rage (2) + anything:** Rage is a standalone system redesign. Layering other morale changes on top of an active resource system creates excessive complexity and balance risk.
- **Tenacity DR (4) + Exponential LS (1):** Both extend survival. Stacking them makes high-morale Axies take too long to kill -- DR at low HP delays death, then 8 Last Stand ticks delay it further. Pick one survival mechanism.
- **Crit Overhaul (3) + Tenacity DR (4):** High-morale Axies would deal massive damage via crits AND be hard to kill at low HP. Too much power concentration in one stat.
