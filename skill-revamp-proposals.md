# Axie Classic -- Skill Stat Revamp Proposals

## Current State of Skill

Skill is one of four Axie stats (HP, Speed, Skill, Morale) and is widely considered the weakest and least impactful stat in Axie Classic. It currently governs three things:

1. **Combo damage multiplier** -- When an Axie plays 2+ cards in one turn, each card's damage is multiplied by `1.0 + (skill * 0.55 - 12.25) / 100.0 * 0.985`
2. **Chain shield bonus** -- When 2+ teammates play same-class cards in the same round: `(card_defense * skill) / 500.0` additive shield bonus
3. **Turn order tiebreaker** -- 3rd priority after speed and lower HP (almost never relevant in practice)

### Skill Values by Class

| Class | Base Skill | Per-Part Bonus | Pure (6 parts) | Combo Multiplier |
|-------|-----------|----------------|-----------------|------------------|
| Mech | 43 | +0 | 43 | +11.2% |
| Dawn | 39 | +0 | 39 | +9.1% |
| Bug | 35 | +0 | 35 | +6.9% |
| Bird | 35 | +0 | 35 | +6.9% |
| Aquatic | 35 | +0 | 35 | +6.9% |
| Beast | 31 | +0 | 31 | +4.7% |
| Plant | 31 | +0 | 31 | +4.7% |
| Reptile | 31 | +0 | 31 | +4.7% |
| Dusk | 27 | +0 | 27 | +2.6% |

### Why Skill is the Weakest Stat

1. **No part scaling.** Skill is the ONLY stat that receives zero bonus from body parts. HP, Speed, and Morale all gain +1 to +3 per part depending on part class. Skill gets nothing. A "pure" Mech (43 base) has the same 43 Skill as any Mech with any parts.
2. **No buff/debuff system.** Speed has Speed+/Speed-. Morale has Morale+/Morale-. Skill has no corresponding buff or debuff. It cannot be modified during battle at all (except by gear, which only reduces it).
3. **Tiny combo multiplier range.** The difference between the worst (Dusk, +2.6%) and best (Mech, +11.2%) is only 8.6 percentage points. On a 100 ATK card, that's 8.6 damage -- less than one body part's worth of stat difference in other stats.
4. **Combo multiplier doesn't scale with combo size.** Playing 2 cards or 4 cards gives the exact same skill-based bonus. There is no reward for bigger combos.
5. **Chain shield bonus is negligible.** At 43 Skill with a 40 DEF card, the chain bonus is `(40 * 43) / 500 = 3.44` shield. Rounding means ~3 extra shield.
6. **Gear system uses Skill as a dump stat.** The HP gear trades Skill for HP (-2 Skill per gear level, up to -20 at level 10), confirming even the designers treated Skill as the least valuable stat.

### The Core Problem

Every other stat has a clear, felt impact on gameplay:
- **HP** -- directly determines survivability. Every point matters.
- **Speed** -- determines turn order, which decides who gets to attack, who gets disabled, who dies first. Massive strategic implications.
- **Morale** -- governs crits and Last Stand. Even with the issues outlined in the morale revamp doc, Morale creates memorable moments.
- **Skill** -- gives a small percentage bonus that players rarely notice or plan around.

Skill needs to do something that players can see, feel, and build around.

---

## Proposal 1: Skill-Based Card Draw

### Core Idea
High-Skill Axies draw more cards. Skill represents tactical awareness -- the ability to find the right tool at the right time. This is the most intuitive interpretation and directly addresses the game's draw-dependency problems.

### Mechanics

**Bonus Draw System:**
Each Axie has a Skill accumulator that fills each round:
```
accumulator += skill
when accumulator >= 100: draw 1 extra card, accumulator -= 100
```

| Class | Skill | Extra Draw Every N Rounds | Draws Over 10 Rounds |
|-------|-------|--------------------------|----------------------|
| Dusk | 27 | ~3.7 rounds | 2-3 extra draws |
| Beast/Plant/Reptile | 31 | ~3.2 rounds | 3 extra draws |
| Bug/Bird/Aquatic | 35 | ~2.9 rounds | 3-4 extra draws |
| Dawn | 39 | ~2.6 rounds | 3-4 extra draws |
| Mech | 43 | ~2.3 rounds | 4 extra draws |

The accumulator is deterministic -- no RNG. Players can predict exactly when their extra draw happens.

**Team Draw:** The extra card is drawn by the team (goes into the shared hand), not specifically by the Axie. This prevents high-Skill Axies from hoarding cards and keeps the system team-relevant.

### Why This Works
- Card draw is the most universally powerful mechanic in card games. Making Skill govern it instantly makes the stat matter.
- Deterministic accumulator means players can plan around it (high skill expression).
- Mech (43 Skill) draws ~1.5 more extra cards over 10 rounds than Dusk (27 Skill). Significant but not game-breaking.
- Solves the Beast draw-dependency problem indirectly -- pair a Beast with a high-Skill Mech to draw into Beast combos faster.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Card advantage snowballs too hard | High | Hand size cap of 10 cards; extra draws beyond cap are wasted |
| Makes Mech/Dawn mandatory in every team | Medium | The advantage is ~1 extra card over low-skill classes per 10 rounds; meaningful but not decisive |
| Reduces importance of draw-fixing card effects | Low | Card-based draw effects are still valuable for targeted draws; this is untargeted |

---

## Proposal 2: Combo Depth Scaling

### Core Idea
Overhaul the combo multiplier so it scales with the number of cards played. High-Skill Axies get dramatically more value from 3-4 card combos, while the 2-card combo bonus stays modest. This makes Skill the "combo mastery" stat.

### Current vs Proposed Formula

**Current:** `combo_multiplier = 1.0 + (skill * 0.55 - 12.25) / 100.0 * 0.985` (flat, regardless of combo size)

**Proposed:** `combo_multiplier = 1.0 + (skill * 0.01 * (cards_in_combo - 1))`

### Damage Comparison

| Skill | Cards | Current Multiplier | Proposed Multiplier | Change |
|-------|-------|-------------------|--------------------| -------|
| 27 (Dusk) | 2 | x1.026 | x1.27 | +24% |
| 27 (Dusk) | 3 | x1.026 | x1.54 | +51% |
| 27 (Dusk) | 4 | x1.026 | x1.81 | +78% |
| 35 (Aqua) | 2 | x1.069 | x1.35 | +28% |
| 35 (Aqua) | 3 | x1.069 | x1.70 | +63% |
| 35 (Aqua) | 4 | x1.069 | x2.05 | +98% |
| 43 (Mech) | 2 | x1.112 | x1.43 | +32% |
| 43 (Mech) | 3 | x1.112 | x1.86 | +75% |
| 43 (Mech) | 4 | x1.112 | x2.29 | +118% |

A Mech playing a 4-card combo gets x2.29 damage per card -- more than doubling each card's output. A Dusk doing the same gets x1.81. The gap is meaningful.

### Design Intent
This makes big combos a legitimate strategy tied to the Skill stat. Currently, playing 4 cards vs 2 cards gives the same Skill bonus, so there's no Skill-based reason to save for a big turn. With this change, a Mech saving 4 cards for one explosive turn is a viable playstyle.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| 4-card combos become too powerful | High | Energy cost naturally limits 4-card combos; also consider a cap at x2.0 |
| Reinforces combo dependency (bad for Beast) | Medium | Beast has 31 Skill so gets less scaling; Beast's identity should come from Morale, not Skill |
| Makes single-card plays feel worse | Medium | Single cards still deal base damage; this rewards saving, not punishes spending |

---

## Proposal 3: Skill as Card Efficiency (Energy Discount)

### Core Idea
High-Skill Axies occasionally play cards for free. Each round, an Axie's first card has a chance to cost 0 energy, representing technical mastery -- executing moves with minimal effort.

### Mechanics
```
free_card_chance = (skill - 25) * 2.5%
```

| Class | Skill | Free Card Chance |
|-------|-------|-----------------|
| Dusk | 27 | 5% |
| Beast/Plant/Reptile | 31 | 15% |
| Bug/Bird/Aquatic | 35 | 25% |
| Dawn | 39 | 35% |
| Mech | 43 | 45% |

**Rules:**
- Only the first card played by each Axie per round can be free
- 0-cost cards are not eligible (they're already free)
- The free card still counts toward combo calculations
- The energy is refunded after the card is played (visible to both players)

### Alternative: Deterministic Version
Instead of RNG, use an accumulator like Proposal 1:
```
efficiency_accumulator += skill
when accumulator >= 120: next card played is free, accumulator -= 120
```
Mech gets a free card every ~3 rounds. Dusk gets one every ~4.5 rounds.

### Design Intent
Energy is the primary constraint in Axie Classic. Making Skill reduce energy costs -- even occasionally -- creates enormous value. A Mech that gets a free card 45% of rounds effectively has ~0.45 extra energy per round, which is a massive economic advantage.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| RNG version creates feel-bad moments | High | Use deterministic accumulator instead |
| Free 2-cost cards are too powerful | Medium | Only applies to 1-cost cards, or discount is -1 energy rather than free |
| Stacks with energy generation cards | Medium | Monitor; may need to cap total energy advantage per round |

---

## Proposal 4: Skill as Precision (Shield Penetration and Accuracy)

### Core Idea
Redefine Skill as combat precision. High-Skill Axies strike more accurately -- partially bypassing shields, ignoring evasion effects, and dealing more consistent damage. This makes Skill the anti-tank stat.

### Mechanics

**Shield Penetration:**
```
penetration_percent = (skill - 25) * 1.5%
```
A percentage of the Axie's damage bypasses the target's shield and hits HP directly.

| Class | Skill | Shield Penetration |
|-------|-------|--------------------|
| Dusk | 27 | 3% |
| Beast/Plant/Reptile | 31 | 9% |
| Bug/Bird/Aquatic | 35 | 15% |
| Dawn | 39 | 21% |
| Mech | 43 | 27% |

Example: Mech plays a 100 ATK card against a target with 80 shield. Currently, 80 damage goes to shield, 20 to HP. With 27% penetration: 27 damage goes directly to HP, 73 damage hits shield first. Result: shield absorbs 73 (7 remaining), HP takes 27. Total HP damage = 27 vs current 20.

**Accuracy Bonus:**
- At 35+ Skill: immune to "Miss" effects
- At 40+ Skill: counter-attacks retain 100% damage instead of the current reduction
- At 43+ Skill: attacks ignore the target's evasion buffs (Stench targeting change is unaffected)

### Design Intent
Shield penetration directly addresses the Plant/Reptile wall problem. High-Skill classes (Mech, Dawn) can chip through shields consistently, while low-Skill tanks (Plant at 31 = 9% penetration) barely benefit when attacking. This creates a natural counter-relationship: Skill-heavy Axies are anti-tank.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Shield-based strategies become unviable | High | 27% penetration still means 73% of damage is absorbed by shield; shields remain strong |
| Plants/Reptiles lose their identity | Medium | Their identity is high HP + shields; penetration reduces but doesn't eliminate shield value |
| Stacks with existing shield-bypass cards | Medium | Penetration is percentage-based and additive with flat bypass effects; cap total penetration at 50% |

---

## Proposal 5: Skill as Adaptability (Card Transformation)

### Core Idea
High-Skill Axies can transform cards in hand, representing tactical flexibility. Once per round, an Axie with sufficient Skill can replace a card in hand with a different option from its deck. The Axie is "skilled" enough to adapt its approach mid-fight.

### Mechanics

**Transform Action (during planning phase):**
- At 35+ Skill: Once per round, may discard 1 card from hand and draw 1 card from deck
- At 39+ Skill: When transforming, look at the top 2 cards and choose 1 (put the other back)
- At 43+ Skill: When transforming, look at the top 3 cards and choose 1

**Rules:**
- Transform happens during the planning phase, before cards are committed
- The discarded card goes to the bottom of the draw pile (not lost)
- Transform costs no energy
- Each Axie can only transform once per round
- Below 35 Skill: no transform ability at all

### Transform Quality by Class

| Class | Skill | Transform Available | Cards Seen |
|-------|-------|--------------------| -----------|
| Dusk | 27 | No | -- |
| Beast/Plant/Reptile | 31 | No | -- |
| Bug/Bird/Aquatic | 35 | Yes | 1 (blind swap) |
| Dawn | 39 | Yes | 2 (choose 1) |
| Mech | 43 | Yes | 3 (choose 1) |

### Design Intent
This is hand-fixing as a stat-based mechanic. Low-Skill classes are stuck with what they draw. High-Skill classes can sculpt their hand. A Mech seeing 3 cards and choosing the best one is a powerful tool for finding combos, answers, or specific tech cards.

The 35 Skill threshold means Beast (31), Plant (31), and Reptile (31) don't get access -- but they can be paired with a Mech/Dawn/Bug/Bird/Aquatic teammate who transforms to find cards that benefit the whole team.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Creates a hard divide between classes with/without transform | High | The threshold could be lowered to 30 (include Beast/Plant/Reptile at 1-card blind swap) |
| UI complexity for the transform action | Medium | Simple drag-to-discard interface; shows replacement options |
| Slows down the planning phase | Medium | Transform is optional and fast; 5-second timer for the decision |
| Makes Mech too versatile | Medium | Mech already has the highest Skill by design; this reinforces its identity |

---

## Proposal 6: Skill-Scaled Ability Effects

### Core Idea
Make card effect magnitudes scale with the Skill stat. Debuff durations, buff durations, heal amounts, and special effect percentages are all modified by a Skill multiplier. Skill becomes the "quality of execution" stat -- the same card played by a Mech is slightly more potent than when played by a Plant.

### Mechanics
```
effect_multiplier = 1.0 + (skill - 27) * 0.025
```

| Class | Skill | Effect Multiplier |
|-------|-------|-------------------|
| Dusk | 27 | x1.00 (baseline) |
| Beast/Plant/Reptile | 31 | x1.10 (+10%) |
| Bug/Bird/Aquatic | 35 | x1.20 (+20%) |
| Dawn | 39 | x1.30 (+30%) |
| Mech | 43 | x1.40 (+40%) |

### What Scales with Skill

| Effect Type | Base | At Skill 35 | At Skill 43 |
|-------------|------|-------------|-------------|
| Debuff duration (2 turns) | 2 turns | 2.4 -> 2 turns | 2.8 -> 3 turns |
| Buff duration (2 turns) | 2 turns | 2.4 -> 2 turns | 2.8 -> 3 turns |
| Poison stacks (3) | 3 | 3.6 -> 4 | 4.2 -> 4 |
| Heal amount (50 HP) | 50 | 60 | 70 |
| Shield bonus (20%) | 20% | 24% | 28% |
| Energy steal (1) | 1 | 1 | 1 (integers don't round up easily) |

**Rounding rule:** All scaled values are floored. This means duration increases only kick in at high Skill values (need +0.5 to round up to next integer turn), making the breakpoints feel meaningful.

### Design Intent
This is the subtlest approach -- it doesn't add new mechanics, it amplifies existing ones. Every card in the game becomes slightly better with higher Skill. The effect is most noticeable on duration-based effects: a Mech applying a 2-turn Stun gets 3 turns instead. That's a 50% increase in stun duration, which is huge.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Hard to communicate to players (hidden multiplier) | High | Show Skill multiplier on card tooltips; highlight extended durations |
| Integer rounding makes many effects not scale | Medium | Adjust base values or multiplier to hit meaningful breakpoints |
| Balancing every card effect for Skill scaling is complex | High | Apply globally with the multiplier; individual cards can be flagged as "Skill-immune" if needed |
| Energy/card count effects can't scale meaningfully | Medium | Exempt integer resource effects (energy, card draw counts) from Skill scaling |

---

## Proposal 7: Skill as Combo Memory (Persistent Combo Tracker)

### Core Idea
Skill enables a persistent combo counter that tracks cards played across rounds. At certain thresholds, bonus effects trigger automatically. High-Skill Axies maintain their counter; low-Skill Axies lose progress to decay. This turns Skill into a "momentum" stat.

### Mechanics

**Combo Counter:**
- Each card an Axie plays adds +1 to its combo counter
- Counter persists across rounds
- At end of each round, counter decays by `floor((50 - skill) / 10)`

| Class | Skill | Decay/Round | Net Gain (1 card/round) | Net Gain (2 cards/round) |
|-------|-------|------------|------------------------|-------------------------|
| Dusk | 27 | -2 | -1 (counter drops) | 0 (breaks even) |
| Beast/Plant/Reptile | 31 | -1 | 0 (breaks even) | +1 (slow build) |
| Bug/Bird/Aquatic | 35 | -1 | 0 (breaks even) | +1 (slow build) |
| Dawn | 39 | -1 | 0 (breaks even) | +1 (slow build) |
| Mech | 43 | 0 | +1 (always grows) | +2 (fast build) |

**Combo Thresholds:**

| Counter | Effect |
|---------|--------|
| 3 | Next card deals +15% damage |
| 5 | Draw 1 card |
| 8 | Next card is a guaranteed critical hit |
| 12 | All cards this Axie plays next round deal +30% damage |

Thresholds are consumed when triggered (counter resets to 0 after hitting a threshold, or drops by the threshold amount -- design choice).

### Design Intent
Mech is the only class with 0 decay, meaning every card it plays permanently builds the counter. It reliably hits the 8-counter crit threshold in 4 rounds of playing 2 cards/round. Dusk (decay 2) can barely maintain a counter and essentially never reaches high thresholds organically.

This creates a "momentum" archetype where high-Skill Axies that survive and keep playing cards become increasingly dangerous. It rewards sustained aggression over time.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Counter tracking adds complexity | Medium | Simple incrementing number; display as a small counter on the Axie |
| Mech with 0 decay might be too strong | Medium | Adjust decay formula; could use `floor((45 - skill) / 10)` so Mech decays by 0 only after rounding |
| Counter-reset vs counter-subtract is a major balance lever | Medium | Test both; subtract creates faster re-triggering for high-Skill Axies |
| Doesn't help classes that can't play many cards | Low | Those classes aren't Skill-focused anyway; Skill should reward card-heavy play |

---

## Proposal 8: Skill as Foresight (Information Advantage)

### Core Idea
Skill provides strategic information. High-Skill Axies can see upcoming cards and opponent actions, representing tactical awareness and battlefield intelligence. Information advantage is one of the most powerful tools in competitive games.

### Mechanics

**Deck Vision (passive, always active):**

| Skill Threshold | Ability |
|-----------------|--------|
| 30+ | See the top card of your own draw pile at all times |
| 35+ | See the top 2 cards of your draw pile |
| 39+ | See the top 3 cards of your draw pile |
| 43+ | See the top 3 cards AND see how many cards each opponent Axie has in hand |

**Enhanced Scry:**
When any card effect lets you "look at" or "reveal" cards, Skill determines how many extra cards you see:
```
bonus_cards_revealed = floor(skill / 15)
```
At 43 Skill: +2 cards revealed. At 27 Skill: +1 card.

**Planning Phase Intel (at 40+ Skill):**
During the planning phase, see which enemy Axies have cards committed (but not which cards). This tells you who is going to attack and who is passing, without revealing specific plays.

### Design Intent
Information is power. A Mech player who can always see their next 3 draws can plan 2-3 rounds ahead, saving energy for the right moment and playing the right cards at the right time. A Dusk player is playing blind.

This doesn't add damage or resources -- it adds *decision quality*. Skilled players (both the stat and the player) benefit more from information than casual players, creating a healthy skill-expression loop.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Information advantage is hard to balance numerically | High | Doesn't directly affect damage/HP, so it's self-balancing; better info = better decisions, but the ceiling depends on the player |
| UI clutter showing future draws | Medium | Small card previews at the edge of the screen; toggle-able in settings |
| May slow down planning phase (analysis paralysis) | Medium | Information is always visible, not an additional action; planning timer stays the same |
| Opponent hand-size info could be too powerful | Low | Only shows count, not specific cards; already partially inferrable from gameplay |

---

## Proposal 9: Skill as Technique (Card Sequencing Bonus)

### Core Idea
Reward card play order within a combo. When an Axie plays multiple cards, each subsequent card gets a Skill-based bonus to both ATK and DEF. This replaces the current flat combo multiplier with a system where the sequence of cards matters and high-Skill Axies get progressively stronger hits within a combo.

### Mechanics
```
sequence_bonus_atk = skill * 0.3 * (card_position_in_combo - 1)
sequence_bonus_def = skill * 0.2 * (card_position_in_combo - 1)
```

| Position | Bonus at Skill 27 | Bonus at Skill 35 | Bonus at Skill 43 |
|----------|-------------------|--------------------|--------------------|
| 1st card | +0 ATK / +0 DEF | +0 ATK / +0 DEF | +0 ATK / +0 DEF |
| 2nd card | +8 ATK / +5 DEF | +11 ATK / +7 DEF | +13 ATK / +9 DEF |
| 3rd card | +16 ATK / +11 DEF | +21 ATK / +14 DEF | +26 ATK / +17 DEF |
| 4th card | +24 ATK / +16 DEF | +32 ATK / +21 DEF | +39 ATK / +26 DEF |

### Design Intent
This creates strategic depth in card ordering. You want your strongest card last in the combo to get the biggest sequence bonus. But some cards have effects that need to trigger first (debuffs, shield breaks). High-Skill Axies get more reward for sequencing correctly.

A Mech's 4th card getting +39 ATK is substantial -- it turns a 100 ATK card into a 139 ATK card before other multipliers. Combined with same-class bonus and class advantage, that's a major damage increase. The DEF bonus also means later cards in a combo provide meaningful shield.

### Comparison to Current System

For a 4-card combo with 100 ATK cards:

| Skill | Current Total Damage | Proposed Total Damage | Change |
|-------|---------------------|----------------------|--------|
| 27 | 4 * 102.6 = 410 | 100 + 108 + 116 + 124 = 448 | +9.3% |
| 35 | 4 * 106.9 = 428 | 100 + 111 + 121 + 132 = 464 | +8.4% |
| 43 | 4 * 111.2 = 445 | 100 + 113 + 126 + 139 = 478 | +7.4% |

Total damage increase is modest (~8%), but the distribution is back-loaded, making the last card feel powerful.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Optimal card ordering becomes a "solved" problem | Medium | Card effects create ordering tension (debuffs first vs damage last); not always clear-cut |
| Flat ATK/DEF bonus favors low-ATK multi-hit cards | Low | Bonus is additive, so percentage impact is higher on weak cards; this is intentional (weak cards benefit from technique) |
| 2-card combos get minimal benefit | Low | Intentional; the system rewards bigger combos, which require more planning |

---

## Proposal 10: Skill as Versatility (Cross-Class Mastery)

### Core Idea
Skill determines how well an Axie uses cards from other classes. Currently, same-class cards get +10% damage. This proposal adds a penalty for off-class cards that is reduced by Skill, and grants high-Skill Axies the ability to treat off-class cards as if they were native.

### Mechanics

**Off-Class Penalty (new):**
```
off_class_penalty = max(0, (40 - skill) * 0.5%)
```

| Class | Skill | Off-Class Penalty | Net with Same-Class Bonus |
|-------|-------|--------------------|--------------------------|
| Dusk | 27 | -6.5% damage | Same-class: +10%, Off-class: -6.5% (16.5% gap) |
| Beast/Plant/Reptile | 31 | -4.5% damage | Same-class: +10%, Off-class: -4.5% (14.5% gap) |
| Bug/Bird/Aquatic | 35 | -2.5% damage | Same-class: +10%, Off-class: -2.5% (12.5% gap) |
| Dawn | 39 | -0.5% damage | Same-class: +10%, Off-class: -0.5% (10.5% gap) |
| Mech | 43 | 0% (no penalty) | Same-class: +10%, Off-class: 0% (10% gap) |

**Cross-Class Chain (at 40+ Skill):**
Off-class cards count as the Axie's native class for chain bonus calculations. A Mech playing a Bird card triggers chain bonuses as if it were a Mech card.

**Cross-Class Combo (at 43+ Skill):**
Off-class cards trigger same-class combo effects. If a card says "when comboed with another Beast card," a Mech (43 Skill) can trigger it with any card.

### Design Intent
This opens up hybrid builds. Currently, using off-class parts is suboptimal because you lose the same-class bonus. With Skill-based versatility, high-Skill classes can mix and match parts from any class without penalty. A Mech with Bird horn and Aquatic tail takes no damage penalty on those cards.

This makes team building more creative. Instead of "pure builds only," players can experiment with cross-class combinations on high-Skill Axies while keeping pure builds optimal on low-Skill classes.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Off-class penalty hurts Dusk/Beast/Plant/Reptile builds | Medium | Penalty is small (-4.5% to -6.5%); these classes were already using same-class parts anyway |
| Mech becomes the best shell for any card | High | Mech pays for 43 Skill with low HP (27 base) and low Morale (27 base); it's glass cannon versatility |
| Cross-class combo triggering may break specific card synergies | Medium | Audit cards with same-class combo requirements; flag any that become too powerful with cross-class triggering |
| Reduces incentive for "pure" builds on high-Skill classes | Low | Same-class bonus still exists (+10%); pure builds are still optimal for damage, but hybrids become viable |

---

## Comparison Matrix

| Criteria | 1. Card Draw | 2. Combo Depth | 3. Energy Discount | 4. Precision | 5. Adaptability | 6. Effect Scaling | 7. Combo Memory | 8. Foresight | 9. Sequencing | 10. Versatility |
|----------|-------------|---------------|--------------------| -------------|-----------------|-------------------|-----------------|-------------|---------------|-----------------|
| **Core fantasy** | Tactical awareness | Combo mastery | Technical efficiency | Combat precision | Battlefield adaptation | Quality of execution | Building momentum | Intelligence gathering | Martial technique | Cross-class mastery |
| **When it matters** | Every round | During combos | Every round | Every attack | Planning phase | Every card effect | Across rounds | Planning phase | During combos | Team building + combat |
| **Felt impact** | High (extra cards) | High (big combos) | High (free cards) | Medium (chip damage) | High (hand sculpting) | Low-Medium (subtle) | Medium (delayed payoff) | Medium (information) | Medium (ordering) | Medium (build flexibility) |
| **Implementation complexity** | Low | Low | Low-Medium | Medium | Medium-High | Medium | Medium | Medium-High | Low | Medium |
| **Server changes** | Small (draw logic) | Small (formula) | Medium (energy logic) | Medium (damage calc) | Medium (new action) | High (every card effect) | Medium (new tracker) | Medium (visibility system) | Small (formula) | Medium (class system) |
| **Client changes** | Small (draw anim) | None | Small (energy VFX) | Small (penetration VFX) | High (transform UI) | Medium (tooltip updates) | Small (counter display) | High (deck vision UI) | None | Small (class indicator) |
| **Risk of breaking balance** | Medium | High | Medium-High | Medium | Medium | High | Medium | Low | Low | Medium-High |
| **Skill expression added** | Medium | Medium | Low | Low | High | Low | Medium | High | High | Medium |
| **Benefits all classes** | Yes (scaled) | Yes (scaled) | Yes (scaled) | Yes (scaled) | No (threshold) | Yes (scaled) | Yes (scaled) | Partially (threshold) | Yes (scaled) | Yes (scaled) |
| **Mech identity** | Draw engine | Combo king | Efficient fighter | Precise striker | Adaptive tactician | Effect amplifier | Momentum builder | Battlefield analyst | Technique master | Versatile hybrid |

---

## Recommended Groupings

### Tier 1: Highest Impact, Reasonable Implementation
- **Proposal 1 (Card Draw)** -- Universally powerful, simple to implement, immediately felt. Best single-proposal option.
- **Proposal 2 (Combo Depth)** -- Minimal code change (one formula), dramatically increases Skill's impact on combos.

### Tier 2: Strong Thematic Fit, Moderate Implementation
- **Proposal 5 (Adaptability)** -- Most interesting for competitive play; highest skill expression. Requires UI work.
- **Proposal 9 (Sequencing)** -- Clean replacement for current combo system; adds strategic depth with minimal complexity.
- **Proposal 7 (Combo Memory)** -- Unique cross-round tracking mechanic; rewards sustained play.

### Tier 3: Niche or Complex, Consider for Layering
- **Proposal 4 (Precision)** -- Good anti-tank tool but narrow in scope; works well layered with Proposal 1 or 2.
- **Proposal 8 (Foresight)** -- Powerful in competitive play but hard to balance; information advantages are invisible to spectators.
- **Proposal 10 (Versatility)** -- Opens build diversity but may undermine class identity; best suited as a secondary effect.

### Not Recommended as Primary
- **Proposal 3 (Energy Discount)** -- Energy manipulation is dangerous for game balance; free cards can spiral.
- **Proposal 6 (Effect Scaling)** -- Too subtle and creates a balancing nightmare (every card effect needs audit).

### Best Combinations
1. **Card Draw (1) + Combo Depth (2):** Skill gives you more cards AND makes combos stronger. High-Skill Axies draw into big combos and deal massive damage from them.
2. **Card Draw (1) + Sequencing (9):** Extra draws + card ordering rewards. Draw more, sequence better.
3. **Adaptability (5) + Combo Memory (7):** Transform to find the right cards + build momentum over time. High planning depth.
4. **Precision (4) + Versatility (10):** Anti-tank penetration + cross-class flexibility. Mech becomes the ultimate hybrid attacker.
