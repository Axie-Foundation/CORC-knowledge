# Axie Classic -- Class Passive Ability Proposals

**Date:** 2026-05-15
**Goal:** Introduce subtle, class-specific passive abilities that reinforce each archetype's identity without becoming the primary reason to play that class.

---

## Design Guidelines

1. **Subtle, not dominant.** A passive should account for roughly 3-8% of an Axie's value in a match. If a passive is the main reason you're picking a class, it's too strong.
2. **Reinforce, don't patch.** Passives should make the class better at what it already does, not compensate for weaknesses. Beast should feel more Beast-like, not become a tank.
3. **No raw stat increases.** Effects should be conditional, situational, or utility-based. No "+5 ATK" passives.
4. **Readable by opponents.** Players should be able to see what passives their opponent's Axies have. No hidden information.
5. **One passive per Axie.** Each Axie gets exactly one passive based on its body class. This is not a build choice -- it's an innate class trait.

---

## Class Reference

| Class | HP | Speed | Skill | Morale | Archetype | Beats | Loses To |
|-------|-----|-------|-------|--------|-----------|-------|----------|
| Beast | 31 | 35 | 31 | 43 | Crit burst, Last Stand | Plant/Reptile/Dusk | Aquatic/Bird/Dawn |
| Bug | 35 | 31 | 35 | 39 | Shields, disruption, durability | Plant/Reptile/Dusk | Aquatic/Bird/Dawn |
| Bird | 27 | 43 | 35 | 35 | Speed, burst, backdoor | Aquatic/Bird/Dawn target group | Plant/Reptile/Dusk |
| Plant | 43 | 31 | 31 | 35 | Tank, healing, energy control | Aquatic/Bird/Dawn | Beast/Bug/Mech |
| Aquatic | 39 | 39 | 35 | 27 | Speed + sustain, consistent damage | Beast/Bug/Mech | Plant/Reptile/Dusk |
| Reptile | 39 | 35 | 31 | 35 | Off-tank, disabling, sustain | Beast/Bug/Mech | Plant/Reptile/Dusk (neutral) |
| Mech | 31 | 39 | 43 | 27 | Combo mastery, precision | Plant/Reptile/Dusk | Aquatic/Bird/Dawn |
| Dawn | 35 | 35 | 39 | 31 | Balanced, adaptable | Plant/Reptile/Dusk | Beast/Bug/Mech |
| Dusk | 43 | 39 | 27 | 31 | Durable speedster, attrition | Aquatic/Bird/Dawn | Beast/Bug/Mech |

*Note: Class advantage cycle is Plant/Reptile/Dusk > Aquatic/Bird/Dawn > Beast/Bug/Mech > Plant/Reptile/Dusk.*

---

## Beast (High Morale -- Crit Burst / Last Stand)

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | Killer Instinct | After landing a critical hit, this Axie's next attack this round deals +8% damage. | Rewards the crit-heavy archetype with a small snowball effect. Only matters if crits land. |
| 2 | Cornered Fury | When this Axie drops below 30% HP, gain +1 energy for the team (once per battle). | Converts the "about to die" moment into a resource spike. Fits the desperate aggression theme. |
| 3 | Blood Scent | This Axie deals +5% damage to targets below 50% HP. | Finisher instinct. Helps Beast secure kills but doesn't help against full-HP targets. |
| 4 | Adrenaline | The first card this Axie plays each round has +10% shield value. | Small defensive bump on the opening play. Doesn't change Beast's fragility, just softens the first hit slightly. |
| 5 | Pack Awareness | If another Beast ally is alive, this Axie's cards draw 0.5 seconds faster during the planning phase. | Quality-of-life for multi-Beast teams. Extremely subtle -- planning speed, not combat power. |
| 6 | Defiant Roar | When this Axie enters Last Stand, all allies gain +5% damage for 1 round. | Converts the Last Stand moment into a team buff. Small enough to not be worth forcing. |
| 7 | Primal Reflexes | After being attacked, this Axie has a 15% chance to gain Speed+ for 1 round. | Occasional speed boost from getting hit. Fits the "reactive predator" archetype. |
| 8 | Tendon Tear | Critical hits from this Axie apply Speed- to the target for 1 round. | Crits gain utility beyond damage. Helps Beast keep up with faster targets after landing a crit. |
| 9 | Feral Focus | If this Axie plays exactly 1 card in a round, that card gains +10% attack. | Rewards restraint. When Beast can't combo, the solo card is slightly better. Anti-combo-dependency. |
| 10 | Thick Blood | This Axie takes 5% less damage from Poison and Bleed effects. | Minor DoT resistance. Beast often dies to chip damage -- this helps at the margins. |

---

## Bug (Moderate Morale -- Shields, Disruption, Durability)

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | Exoskeleton | Shields on this Axie retain 10% of their value into the next round (up to 20 shield). | Bug is the shield class. Letting a fraction of shield persist rewards defensive play without stacking infinitely. |
| 2 | Hive Mind | When another Bug ally plays a card, this Axie gains +3 shield. | Small team synergy for multi-Bug comps. 3 shield per ally card is minor but accumulates. |
| 3 | Antenna Sense | At the start of each round, if this Axie has more shield than HP, gain 5% attack for this round. | Rewards the "shielded attacker" playstyle Bug already pursues. Conditional and situational. |
| 4 | Molting | Once per battle, when this Axie would be stunned, remove the stun and gain 10 shield instead. | Anti-CC for the disruption class. Once-per-battle keeps it subtle. |
| 5 | Parasitic Touch | When this Axie deals damage to a target with a debuff, heal for 3% of damage dealt. | Tiny sustain that requires setup (debuff on target first). Fits Bug's debuff-applying identity. |
| 6 | Swarm Pressure | If this Axie and an ally both attacked the same target this round, the target loses 5 shield. | Team coordination bonus. The shield strip is small but compounds Bug's anti-shield disruption. |
| 7 | Chitin Plating | The first time this Axie takes damage each round, reduce that damage by 8%. | One-time damage reduction per round. Rewards Bug for surviving to late-round where this small edge matters. |
| 8 | Compound Eyes | This Axie can see the top card of the draw pile at all times. | Information advantage that fits Bug's strategic identity. No combat power, just planning quality. |
| 9 | Sticky Web | When this Axie is attacked by a faster Axie, the attacker loses Speed+ (if they have it). | Anti-speed tech. Bug is slow (31 Speed) -- this strips speed buffs from attackers, leveling the field. |
| 10 | Regenerative Shell | At the start of each round, if this Axie has 0 shield, gain 5 shield. | Small passive shield generation when unshielded. Ensures Bug always has a sliver of defense. |

---

## Bird (High Speed -- Burst, Backdoor, Glass Cannon)

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | Tailwind | If this Axie attacks first in the round, gain +3% damage on that attack. | Tiny reward for being fastest. Bird already goes first -- this makes the first strike marginally sharper. |
| 2 | Evasive Maneuver | The first time this Axie would take damage each round, reduce it by 5%. | One-time damage mitigation per round. Helps Bird's terrible HP (27 base) without making it tanky. |
| 3 | Aerial Advantage | This Axie's ranged cards gain +5% shield value. | Bird has several ranged cards. Small defensive bonus on ranged plays reinforces hit-and-run. |
| 4 | Keen Eyes | This Axie can see how many cards each opponent Axie has in hand. | Information advantage for the scout/assassin archetype. Helps Bird choose targets but gives no combat power. |
| 5 | Slipstream | After this Axie attacks, if it was the first to attack this round, gain Stench for 1 turn (cannot be targeted by the next attacker). | Pseudo-evasion after striking first. Fits the "strike and fade" Bird fantasy. Only triggers if Bird attacks first. |
| 6 | Hollow Bones | This Axie takes 10% more damage from melee attacks but 10% less from ranged attacks. | Thematic trade-off. Makes Bird weaker to melee (which it already avoids by being fast) but resilient to ranged poke. |
| 7 | Migration | If this Axie is the last alive on its team, gain +1 Speed for the rest of the battle. | "Last bird standing" gets even faster. The +1 Speed is tiny but can matter for ties. |
| 8 | Thermal Riding | At the start of rounds 3, 6, and 9, this Axie draws 1 additional card. | Timed card draw at specific intervals. Fits the "arrives at the right moment" Bird fantasy. Predictable for opponents. |
| 9 | Dive Bomb | If this Axie targets an enemy in the back position, gain +5% damage. | Small backline targeting bonus. Rewards Bird's assassin role without making front-targeting feel bad. |
| 10 | Wind Shear | When this Axie plays 2+ cards in a combo, the last card gains +1 priority (attacks earlier in the resolution order). | Subtle card ordering advantage within combos. Helps Bird's burst land before the target can react. |

---

## Plant (High HP -- Tank, Healing, Energy Control)

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | Deep Roots | This Axie cannot be knocked back or have its position changed by enemy effects. | Positional stability for the front-line tank. Prevents displacement skills from disrupting Plant's role. |
| 2 | Photosynthesis | At the start of each round, heal 2% of max HP. | Tiny regen. On a 61-HP pure Plant (combat HP ~660), this is ~13 HP/round. Noticeable over a long game, irrelevant in burst. |
| 3 | Bark Skin | Shields on this Axie are 5% more effective (multiplicative with card DEF). | Amplifies Plant's existing shield cards. On a 50 DEF card, this is +2.5 shield. Subtle. |
| 4 | Thorn Response | When this Axie's shield is broken by an attack, deal 5% of the shield's value as damage back to the attacker. | Retaliatory damage when shields pop. Discourages mindless attacks into Plant's shields. |
| 5 | Nutrient Cycling | When this Axie plays a 0-cost card, the team gains +2 shield on all Axies. | Small team-wide shield from Plant's free cards. Rewards Plant's energy-efficient playstyle. |
| 6 | Overgrowth | If this Axie took no damage last round, gain +10 shield at the start of the next round. | Rewards rounds where Plant successfully deterred attacks. Self-reinforcing defense. |
| 7 | Grounding | This Axie is immune to Speed- debuffs. | Plant is already slow (31 Speed) -- Speed- is nearly meaningless. This just removes a feel-bad interaction. |
| 8 | Compost | When an ally Axie dies, this Axie heals for 5% of its max HP. | Converts team losses into small sustain for the last-standing tank. Fits the "outlast everyone" Plant archetype. |
| 9 | Root Network | Allies adjacent to this Axie gain +3% shield value on their cards. | Positional team aura. Rewards keeping Axies near the Plant tank. Small enough to not warp positioning. |
| 10 | Seasonal Cycle | Every 3 rounds, this Axie gains 1 energy. | Slow, predictable energy generation that rewards long games -- Plant's preferred game state. |

---

## Aquatic (Speed + Sustain -- Consistent Damage, Reliable)

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | Flowing Current | If this Axie attacks before the target this round, gain +3% shield on the attack's card. | Small defensive bonus from speed advantage. Aquatic is fast and this subtly rewards going first. |
| 2 | Tidal Rhythm | If this Axie played a card last round, gain +5% attack on the first card this round. | Rewards consistent play across rounds. Aquatic's identity is steady pressure, not burst. |
| 3 | Water Veil | This Axie has a 10% chance to cleanse 1 debuff at the start of each round. | Minor self-cleanse. Aquatic's speed means it often gets debuffed by faster Birds -- this occasionally helps. |
| 4 | Pressure Adaptation | This Axie takes 3% less damage for each debuff currently on it (max 9%). | Converts debuffs into minor resilience. Unique defensive mechanic that scales with pressure. |
| 5 | Upstream | When this Axie is attacked by a slower enemy, gain 3 shield. | Passive shield against slower targets. Aquatic is fast -- most melee attackers are slower. Tiny but consistent. |
| 6 | School Formation | If another Aquatic ally is alive, both gain +3% attack. | Small multi-Aquatic synergy. 3% is barely noticeable in single instances but rewards double-Aquatic compositions. |
| 7 | Cold Blooded | This Axie is immune to Chill effects. | Thematic resistance. Chill prevents Last Stand, which barely matters for Aquatic (27 morale), making this a niche perk. |
| 8 | Riptide | After this Axie's last card in a combo resolves, the target loses 3 shield. | Small shield strip at combo end. Helps Aquatic's consistent damage chip through shields. |
| 9 | Depth Sense | This Axie gains +5% damage against targets that have not played any card this round. | Rewards attacking targets caught without defense. Fits the opportunistic predator archetype. |
| 10 | Regenerative Scales | When this Axie deals damage, heal for 2% of damage dealt. | Tiny lifesteal. On a 100-damage hit, this is 2 HP. Over a full game, it adds up to meaningful sustain. |

---

## Reptile (HP + Steady -- Off-Tank, Disabling, Attrition)

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | Cold Blood | When this Axie is below 50% HP, debuffs it applies last 1 extra turn. | Extended debuffs at low HP. Reptile's disabling identity gets stronger as the game goes late. |
| 2 | Scale Armor | This Axie takes 3% less damage from critical hits. | Anti-crit defense. Reptile faces Beast/Bug (crit-heavy classes) in the class advantage cycle. Subtle counter. |
| 3 | Tail Regeneration | Once per battle, when this Axie drops below 25% HP, heal 8% of max HP. | One-time small heal at critical HP. Fits the "hard to finish off" Reptile archetype. |
| 4 | Ambush Predator | If this Axie did not play a card last round, gain +8% damage on the first card this round. | Rewards patience and skipping rounds. Reptile can afford to wait with its high HP. |
| 5 | Venomous Skin | When this Axie is attacked by a melee card, the attacker has a 10% chance to receive Poison (1 stack). | Passive punishment for melee attackers. Small chance, small effect, fits the "dangerous to touch" Reptile. |
| 6 | Camouflage | At the start of the battle, this Axie gains Stench for 1 turn. | Forces opponents to target someone else on round 1. Reptile uses this to survive to its preferred late-game. |
| 7 | Stone Skin | When this Axie has 0 cards in hand, gain +5% damage reduction. | Defensive bonus when out of options. Reptile's identity is durability -- this helps when the hand is empty. |
| 8 | Constricting Grip | When this Axie's attack breaks an enemy's shield, apply Speed- to the target for 1 round. | Shield-break utility. Reptile often attacks into shielded targets -- this adds a small disruption reward. |
| 9 | Basking | If this Axie was not attacked last round, gain +5 shield at the start of the next round. | Rewards being ignored. Reptile in the back line slowly builds shield when opponents focus the front. |
| 10 | Territorial | This Axie deals +5% damage to targets in the front position. | Front-targeting bonus. Reptile often attacks the nearest (front) target -- this makes that slightly more effective. |

---

## Mech (High Skill -- Combo Mastery, Precision, Versatile)

*Note: Mech is a secret class (Beast + Bug parents). It has the highest Skill (43) but lowest HP and Morale. Parts come from Beast and Bug, not Mech.*

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | Overclock | When this Axie plays 3+ cards in a combo, the combo multiplier is calculated as if Skill were 5 points higher. | Amplifies Mech's defining stat (Skill) during big combos. Only triggers on 3+ card combos -- no benefit to small plays. |
| 2 | Precision Targeting | This Axie's attacks ignore 3% of the target's shield. | Flat shield penetration. Mech's precision archetype expressed as slightly bypassing defenses. |
| 3 | System Diagnostic | At the start of each round, this Axie reveals whether the opponent's Axie in front has more ATK or DEF cards in hand. | Information advantage for the analytical class. No combat power, just strategic insight. |
| 4 | Efficient Design | 0-cost cards played by this Axie gain +5 shield. | Small defensive bonus on free cards. Mech's efficiency archetype benefits from maximizing every card. |
| 5 | Calculated Strike | If this Axie attacks a target that was already attacked this round, gain +5% damage. | Rewards focused fire and team coordination. Mech's precision means it strikes where the team has already weakened. |
| 6 | Modular Assembly | This Axie's off-class cards (non-Beast, non-Bug) lose only 50% of the normal off-class penalty. | Reduced penalty for hybrid part builds. Mech is the versatility class -- this lets it use diverse cards more effectively. |
| 7 | Power Surge | Once per battle, when this Axie plays a 4-card combo, gain 1 energy after the combo resolves. | Energy refund on maximum combo. Once-per-battle prevents abuse. Rewards Mech's combo identity. |
| 8 | Heat Sink | This Axie is immune to Attack- debuffs. | Prevents enemies from reducing Mech's damage output. Mech's strength is its damage -- this protects that identity. |
| 9 | Servo Motors | If this Axie played a combo last round, gain +1 Speed for this round (non-stacking). | Temporary speed boost from combo activity. Mech is moderately fast (39) -- this occasionally pushes it ahead in turn order. |
| 10 | Logic Core | When this Axie is the target of a debuff, there is a 10% chance the debuff is reflected to the attacker instead. | Small debuff reflection. Fits the "calculated" Mech archetype -- occasionally turning enemy disruption back on them. |

---

## Dawn (Balanced Skill -- Adaptable, Flexible, Jack-of-All-Trades)

*Note: Dawn is a secret class (Bird + Aquatic parents). It has moderate stats across the board with slightly high Skill (39). Parts come from Bird and Aquatic.*

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | First Light | At the start of the battle, this Axie and all allies gain +3 shield. | Small opening-round team defense. Dawn's "new beginning" theme expressed as an initial advantage. |
| 2 | Adaptive Stance | If this Axie took damage last round, gain +3% attack this round. If it didn't, gain +3% shield this round. | Contextual bonus that shifts based on what happened. Fits Dawn's adaptive identity -- always getting a small edge. |
| 3 | Horizon Sight | This Axie can see the top 1 card of its own draw pile. | Mild information advantage. Dawn's balanced nature means better planning is more valuable than raw power. |
| 4 | Twilight Transition | When this Axie switches from attacking to defending (or vice versa) between rounds, gain +5% on the new action. | Rewards alternating playstyles. Dawn's flexibility means it naturally shifts between attack and defense. |
| 5 | Aurora Shield | Once per battle, when this Axie would take lethal damage outside of Last Stand, survive with 1 HP instead. | One-time death prevention. Dawn's moderate morale (31) means it rarely gets Last Stand -- this compensates slightly. |
| 6 | Balanced Metabolism | Buffs on this Axie last 1 extra turn. | Extended buff duration rewards Dawn's moderate stat profile -- buffs are proportionally more valuable on balanced stats. |
| 7 | Mediator | When this Axie is between two allies (middle position), all allies gain +2% attack. | Positional team aura. Dawn as the team glue, benefiting from being in the center. |
| 8 | Dual Nature | This Axie's Bird-class cards gain +3% attack. This Axie's Aquatic-class cards gain +3% shield. | Small bonus that amplifies Dawn's parent-class cards differently. Bird cards hit harder, Aquatic cards defend better. |
| 9 | Equinox | At the halfway point of the battle (round 5 in a 10-round game), gain 1 energy and draw 1 card. | Timed power spike at mid-game. Predictable for both players, fits the "turning point" Dawn theme. |
| 10 | Gentle Glow | Allies adjacent to this Axie heal 1% of their max HP at the start of each round. | Tiny team heal aura. On a 600 HP ally, this is 6 HP/round. Barely noticeable but adds up in long games. |

---

## Dusk (High HP + Speed -- Durable Speedster, Attrition, Control)

*Note: Dusk is a secret class (Reptile + Plant parents). It has high HP (43) and decent Speed (39) but the lowest Skill (27). Parts come from Reptile and Plant.*

| # | Name | Effect | Rationale |
|---|------|--------|-----------|
| 1 | Enduring Shadow | This Axie takes 3% less damage in rounds 5+. | Late-game damage reduction. Dusk's attrition identity means it wants long games -- this rewards surviving to late rounds. |
| 2 | Dusk Veil | At the start of each round, if this Axie has more HP than any enemy Axie, gain +3 shield. | Conditional passive shield from HP advantage. Dusk has high HP, so this triggers often in the early-mid game. |
| 3 | Twilight Drain | When this Axie deals damage to an enemy with a buff, steal 1 random buff (once per round). | Buff theft that fits the "shadow predator" Dusk archetype. Once-per-round keeps it subtle. |
| 4 | Lingering Darkness | Debuffs applied by this Axie have a 15% chance to last 1 extra turn. | Occasional extended debuffs. Dusk's Reptile/Plant cards include disabling effects -- this makes them slightly stickier. |
| 5 | Umbral Fortitude | When this Axie is the last alive on its team, gain +5% damage reduction. | "Last one standing" defense. Dusk's durability means it often outlives allies -- this makes the final stand slightly better. |
| 6 | Shadow Step | After this Axie is attacked, gain +2 Speed until end of round (max 1 trigger per round). | Reactive speed boost. Dusk is moderately fast (39) and this occasionally pushes it ahead of mid-speed Axies. |
| 7 | Gloaming | If this Axie played no cards last round, gain +10 shield at the start of the next round. | Rewards skipping rounds. Dusk's high HP means it can afford to wait -- this adds defense for patience. |
| 8 | Penumbra | This Axie's Plant-class cards gain +3% shield. This Axie's Reptile-class cards gain +3% attack. | Small bonus that amplifies Dusk's parent-class cards. Plant cards defend better, Reptile cards hit harder. |
| 9 | Nightfall | In rounds 7+, this Axie gains +5% damage. | Late-game damage scaling. Dusk wants long games -- this creates a slow-building threat in the final rounds. |
| 10 | Eclipse | Once per battle, when this Axie receives a critical hit, negate the bonus crit damage (take normal damage instead). | One-time crit negation. Dusk faces Beast/Bug (high morale, crit-focused) -- this occasionally denies a big crit. |

---

## Summary: Passive Design Patterns

Each class's 10 passives fall into recurring categories:

| Pattern | Example | Classes That Use It |
|---------|---------|--------------------|
| **Conditional damage bonus** | Blood Scent (+5% vs low HP targets) | Beast, Bird, Aquatic, Reptile, Mech, Dawn, Dusk |
| **One-time-per-battle trigger** | Molting (stun cleanse), Aurora Shield (survive lethal) | Bug, Reptile, Dawn, Dusk, Mech |
| **Passive shield/defense** | Bark Skin (+5% shield), Exoskeleton (shield retention) | Plant, Bug, Aquatic, Dusk |
| **Information advantage** | Keen Eyes (see opponent hand size), Compound Eyes (see draw pile) | Bird, Bug, Mech, Dawn |
| **Late-game scaling** | Enduring Shadow (-3% damage in rounds 5+), Nightfall (+5% in rounds 7+) | Dusk, Plant, Reptile |
| **Team synergy aura** | Root Network (+3% ally shield), Hive Mind (+3 shield per Bug ally card) | Plant, Bug, Aquatic, Dawn |
| **Debuff interaction** | Cold Blood (extended debuffs at low HP), Tendon Tear (crits apply Speed-) | Reptile, Beast, Dusk |
| **Speed/turn-order manipulation** | Primal Reflexes (chance Speed+), Shadow Step (reactive Speed+) | Beast, Bird, Dusk, Mech |

### Selection Guidance

When choosing which 1-2 passives to implement per class, prefer passives that:
1. Are easy to understand in one sentence
2. Don't require UI changes beyond a passive icon tooltip
3. Don't interact complexly with existing card effects
4. Are easy to implement server-side (simple conditional checks in the damage/shield formulas)
