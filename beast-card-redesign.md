# Axie Classic -- Beast Card Redesign Proposal

## Problem Statement

Beasts in Axie Classic suffer from a fundamental timing/draw dependency problem that makes them inconsistent and frustrating to play:

1. **Beasts are not fast** (speed 35, average) and go after Birds (43) and Aquatics (39), meaning they often get killed or crippled before they can attack.
2. **Beast cards are combo-dependent** -- Ronin needs 3+ cards, Shiba needs 4 cards, Nut Cracker needs its pair, Night Steal needs a combo. If you don't draw the right combination, the cards are individually weak.
3. **Draw RNG is punishing** -- You draw 6 cards round 1, then 3/round from a 24-card deck. A Beast in the mid/back needs its specific cards at the right time, but there's no card selection or draw filtering.
4. **Burst-or-bust design** -- Beast cards deal huge damage on crits/combos but provide almost nothing if conditions aren't met. A Ronin without a combo is just 80 ATK. A Risky Beast outside Last Stand is just 120 ATK.
5. **No resilience** -- Beast has the lowest HP (31 base) tied with Mech, no healing, no shields. If they don't burst, they die.

## Beast Class Stats Reference

| Stat | Base | Per-Part Bonus | Pure Beast (6 parts) |
|------|------|----------------|----------------------|
| HP | 31 | +0 | 31 (336 combat HP) |
| Speed | 35 | +1 | 41 |
| Skill | 31 | +0 | 31 |
| Morale | 43 | +3 | 61 |

- **Class advantage:** Beast beats Plant/Reptile/Dusk (+15% damage)
- **Class disadvantage:** Beast loses to Aquatic/Bird/Dawn (-15% damage)
- **Class cycle:** Plant > Aquatic > Beast > Plant. Beast beats the tanks (Plant/Reptile/Dusk) and loses to the fast classes (Aquatic/Bird/Dawn).
- **Same-class bonus:** +10% damage when Beast uses Beast cards
- **Morale drives:** Critical hit chance, critical hit damage multiplier (capped at 200%), Last Stand entry threshold, Last Stand turns

## Current Beast Cards (Gen 1)

### Mouth
| Card Name | Energy | Type | ATK | DEF | Effect |
|-----------|--------|------|-----|-----|--------|
| Nut Crack | 1 | Melee | 105 | 30 | Deal 120% damage when comboed with another 'Nut Cracker' card |
| Piercing Sound | 1 | Ranged | 85 | 30 | Destroy 1 of your opponent's energy |
| Death Mark | 1 | Ranged | 110 | 30 | Apply Lethal to target if this Axie's HP is below 30% |
| Self Rally | 0 | Support | 0 | 20 | Apply Speed+ and Morale+ to team for 2 rounds |

### Horn
| Card Name | Energy | Type | ATK | DEF | Effect |
|-----------|--------|------|-----|-----|--------|
| Branch Charge | 1 | Melee | 100 | 0 | Guaranteed critical strike when attacking first |
| Ivory Stab | 1 | Melee | 90 | 30 | Gain 1 energy per critical strike dealt by your team this round |
| Merry Legion | 1 | Melee | 90 | 40 | Bonus 20% shield when comboed |
| Sugar Rush | 1 | Melee | 110 | 40 | Apply Aroma to self until next round; Apply Attack- to self |
| Sinister Strike | 1 | Melee | 130 | 20 | Deal 200% damage on critical strikes |
| Acrobatic | 1 | Melee | 110 | 30 | Apply Speed+ to this Axie for 2 rounds when comboed with another card |

### Back
| Card Name | Energy | Type | ATK | DEF | Effect |
|-----------|--------|------|-----|-----|--------|
| Single Combat | 1 | Melee | 80 | 20 | Guaranteed critical strike when comboed with at least 2 other cards |
| Heroic Reward | 0 | Melee | 55 | 0 | Draw a card when attacking an Aquatic, Bird, or Dawn target |
| Nitro Leap | 1 | Melee | 120 | 35 | Attack first if any Axie is in Last Stand |
| Revenge Arrow | 1 | Ranged | 120 | 25 | Deal 200% damage when in Last Stand |
| Woodman Power | 1 | Melee | 80 | 40 | Apply Stun when hit by a Plant or Reptile card. Triggers once per round |
| Juggling Balls | 1 | Ranged | 40 | 30 | Strike 3 times |

### Tail
| Card Name | Energy | Type | ATK | DEF | Effect |
|-----------|--------|------|-----|-----|--------|
| Luna Absorb | 0 | Support | 0 | 30 | Gain 1 energy after attacking |
| Night Steal | 1 | Melee | 90 | 20 | Steal 1 energy from your opponent when comboed with another card |
| Rampant Howl | 1 | Melee | 110 | 30 | Guaranteed Last Stand if killed this round when comboed with another 3 cards |
| Hare Dagger | 1 | Ranged | 115 | 20 | Draw a card if this Axie attacks at the beginning of the round |
| Nut Throw | 1 | Ranged | 105 | 30 | Deal 120% damage when comboed with another 'Nut Cracker' card |
| Gerbil Jump | 1 | Melee | 50 | 25 | Skip the closest target if there are 2 or more enemies remaining |

---

## Design Pillars for New Cards

All new cards stay true to the beast archetype (high damage, high morale/crit, aggressive) while solving the timing/draw dependency problem through three pillars:

1. **Residual Value** -- cards that do something useful even when conditions aren't ideal
2. **Setup/Inevitability** -- cards that build toward power turns rather than requiring everything at once
3. **Survivability Windows** -- limited defensive tools that let beasts reach their power spike

---

## New Mouth Card Options (20)

| # | Name | Energy | Type | ATK | DEF | Effect | Design Rationale |
|---|------|--------|------|-----|-----|--------|-----------------|
| 1 | Fang Rend | 1 | Melee | 95 | 20 | Apply Bleed (2 HP/turn, 3 turns) to target. If this Axie has Morale+, apply 2 stacks instead. | Provides persistent damage even without combo. Morale+ synergy rewards existing beast buffs. |
| 2 | Primal Roar | 0 | Support | 0 | 30 | Apply Fear to target for 1 turn. Draw 1 card. | Free cycle card that disrupts enemy tempo. Solves draw dependency directly. |
| 3 | Jaw Trap | 1 | Melee | 80 | 40 | If this Axie is attacked before its turn this round, deal 150% damage instead. | Rewards being slow -- turns the speed disadvantage into a strength. |
| 4 | Rabid Bite | 1 | Melee | 100 | 20 | Gain 1 energy if this attack crits. Apply Jinx to target for 2 turns. | Energy sustain on-crit (reliable with high morale). Jinx shuts down enemy crits. |
| 5 | Bone Crush | 1 | Melee | 110 | 10 | Reduce target's shield by 50% before dealing damage. | Anti-shield tech. Beasts already have class advantage vs Plants but still struggle against heavy shields -- this punches through. |
| 6 | Hunger Howl | 0 | Support | 0 | 20 | Apply Lethal to target for 2 turns. This Axie gains Morale+ for 2 turns. | Free Lethal applicator. Doesn't need low HP like Death Mark. Sets up future crits. |
| 7 | Iron Jaw | 1 | Melee | 90 | 30 | If this Axie has more than 50% HP, gain +1 Last Stand tick at end of round. | Proactive Last Stand investment. Rewards playing it early, not just as a desperation move. |
| 8 | Savage Snap | 1 | Melee | 105 | 20 | Deal 130% damage if target has any debuff. | Payoff card for Beasts that apply debuffs. Unconditional 105 ATK baseline is still solid. |
| 9 | Pack Call | 1 | Melee | 85 | 25 | If any ally Beast played a card this round, gain +20% damage. Draw 1 card if 2+ ally Beasts played cards. | Multi-Beast team synergy. The draw effect is team-based, not self-combo-based. |
| 10 | Venom Fang | 1 | Melee | 75 | 20 | Apply 3 stacks of Poison. If this Axie is in Last Stand, apply 5 stacks instead. | Gives Beast access to Poison, which is permanent and bypasses shields. Last Stand bonus fits archetype. |
| 11 | Gnaw | 0 | Melee | 45 | 0 | Heal this Axie for 50% of damage dealt. | Free card, small heal. Gives Beast a trickle of sustain without changing archetype. |
| 12 | Tearing Bite | 1 | Melee | 90 | 15 | Apply Fragile to target for 2 turns. If comboed, also apply Penetrate for 1 turn. | Anti-tank utility. Fragile is baseline useful; Penetrate on combo is devastating. |
| 13 | Echo Growl | 1 | Melee | 80 | 30 | After attacking, put a copy of the next card this Axie plays this round into your discard pile. | Card generation. Doesn't draw immediately but ensures future rounds have more Beast cards. |
| 14 | Predator's Mark | 1 | Ranged | 70 | 20 | Mark target for 2 turns. Marked targets take 15% more damage from all Beast cards. | Team-wide damage amplifier. Even if this Beast dies, the mark helps allied Beasts. |
| 15 | Feral Instinct | 0 | Support | 0 | 35 | Reveal the top 2 cards of your deck. If either is a Beast card, draw it. | Direct draw-fixing. Solves the "wrong cards at wrong time" problem. |
| 16 | Snarl | 1 | Melee | 100 | 25 | If this Axie attacked last this round, gain 1 energy and apply Attack+ to self for next round. | Rewards being slowest. Converts speed disadvantage into resource advantage. |
| 17 | Maw Clamp | 1 | Melee | 95 | 30 | Disable 1 random card in target's hand next round. | Hand disruption. Helps Beast survive by reducing enemy options. |
| 18 | Carrion Call | 1 | Melee | 110 | 15 | Deal +20% damage for each dead ally. | Scales with losing board state. Beast is often last alive -- this makes that position stronger. |
| 19 | Gnashing Teeth | 1 | Melee | 60 | 20 | Hit twice. Each hit can crit independently. | Multi-hit with double crit chance. High morale makes this statistically better than a single 120 ATK hit. |
| 20 | Desperate Lunge | 1 | Melee | 85 | 25 | If this Axie has no other cards in hand, deal 180% damage and draw 2 cards. | Solves the "only bad cards left" problem. Makes even a lone card powerful. |

---

## New Horn Card Options (20)

| # | Name | Energy | Type | ATK | DEF | Effect | Design Rationale |
|---|------|--------|------|-----|-----|--------|-----------------|
| 1 | Reckless Gore | 1 | Melee | 120 | 0 | Deal +30% damage but take 15% of damage dealt as self-damage. | High floor, no conditions. Self-damage fits "risky beast" fantasy and is offset by Last Stand potential. |
| 2 | Hardened Antler | 1 | Melee | 80 | 50 | If this Axie is the last alive on its team, gain Immune for 1 turn. | Defensive horn with a strong "last man standing" trigger. Beast is often the last alive. |
| 3 | Momentum Horn | 1 | Melee | 90 | 25 | This card's attack increases by 15 for each card this Axie has played in previous rounds (max +60). | Scaling card. Gets stronger each round the Beast survives. Rewards playing any Beast card, not specific ones. |
| 4 | Serrated Edge | 1 | Melee | 100 | 20 | Crits from this card cannot be prevented by Jinx or cannot_crit effects. Crit damage from this card ignores the 200% cap (up to 250%). | Anti-Jinx tech. Many players counter Beast with Jinx; this punches through. |
| 5 | Battering Ram | 1 | Melee | 95 | 30 | If target has shield, deal full damage to both shield AND HP simultaneously. | Shield piercing. Even with class advantage, Plant/Reptile shields absorb Beast burst -- this punches through without ignoring shield entirely. |
| 6 | Thorn Crown | 1 | Melee | 85 | 35 | When this Axie takes damage this round, deal 30% of damage taken back to attacker. | Retaliatory damage. Creates value from getting hit, which slow Beasts always do. |
| 7 | Siege Breaker | 2 | Melee | 160 | 10 | Cannot be played in a combo. Apply Stun to target if their shield breaks from this attack. | Big single-card play. Removes combo dependency entirely for this card -- rewards solo play. |
| 8 | Spiral Horn | 1 | Melee | 95 | 25 | Guaranteed crit if this Axie took damage earlier this round. | Reliable crit trigger for slow Beasts. Being hit first (speed disadvantage) becomes the enabler. |
| 9 | Fury Horn | 1 | Melee | 70 | 20 | Gain a Fury counter (persists across rounds). At 3 Fury counters, your next Beast card deals 200% damage and consumes all counters. | Stacking mechanic across rounds. Solves timing -- you don't need cards NOW, you build toward a spike. |
| 10 | Shedding Antler | 0 | Support | 0 | 40 | Discard 1 random non-Beast card from your hand. Draw 1 card. | Hand filtering. Actively solves draw RNG by replacing bad cards with potential Beast cards. |
| 11 | Impale | 1 | Melee | 115 | 15 | If target is Aquatic or Bird, ignore class disadvantage. | Directly addresses Beast's worst matchup. 115 ATK at full neutral damage against the fast classes that counter Beast. |
| 12 | Whiplash Horn | 1 | Melee | 90 | 20 | After attacking, attack the fighter behind target for 40% damage. | Cleave damage. Reaches backline without needing special targeting, guaranteed secondary value. |
| 13 | Bull Rush | 1 | Melee | 100 | 30 | If this Axie did not play any card last round, deal 140% damage. | Rewards patience. If you don't have the right cards one round, you get a power spike next round. |
| 14 | Crescent Horn | 1 | Melee | 105 | 25 | Apply Chill to target for 2 turns. | Anti-Last Stand. Prevents enemies from entering Last Stand, which is the counter to Beast burst. |
| 15 | Primal Charge | 1 | Melee | 85 | 20 | Gain +1 energy if this Axie has Morale+. Apply Morale+ to this Axie for 2 turns. | Self-sustaining buff loop. First play gives Morale+; second play also gives energy. |
| 16 | Razor Horn | 1 | Melee | 110 | 15 | Deal 20% bonus damage for each buff on this Axie. | Payoff for buff-stacking. Works with Morale+, Speed+, Attack+, etc. |
| 17 | Tremor Strike | 1 | Melee | 95 | 25 | Apply Speed- to target for 2 turns. If target is faster than this Axie, also apply Attack- for 1 turn. | Speed equalization. Pulls fast enemies down to Beast's level for future rounds. |
| 18 | Stubborn Horn | 1 | Melee | 80 | 40 | If this Axie enters Last Stand this round, restore 1 energy to your team and draw 1 card. | Last Stand payoff. Doesn't need to be played at the right time -- triggers passively. |
| 19 | Wild Charge | 1 | Melee | 90 | 15 | Attack a random enemy instead of the nearest. Deal +40% damage if target is in the back position. | Backline snipe with randomness. Fits the wild/unpredictable Beast theme. |
| 20 | Ancestral Horn | 1 | Melee | 100 | 30 | When this card is drawn, apply Morale+ to this Axie for 1 turn. (Passive draw trigger.) | Value on draw, not play. Even sitting in hand, it already did something. Reduces "dead draw" feelings. |

---

## New Back Card Options (20)

| # | Name | Energy | Type | ATK | DEF | Effect | Design Rationale |
|---|------|--------|------|-----|-----|--------|-----------------|
| 1 | Berserk Stance | 1 | Melee | 75 | 35 | For each card this Axie plays this round (including this one), gain Attack+ for next round. Persists 1 round. | Converts any combo size into future value. Even playing 1 card gives Attack+ next round. |
| 2 | Thick Hide | 0 | Support | 0 | 50 | Gain Immune for 1 turn. Cannot be played in a combo. | Pure defensive option. Immune blocks debuffs (Stun, Poison, Chill). Solo-play requirement prevents abuse with high-damage combos. |
| 3 | Adrenaline Surge | 1 | Melee | 100 | 20 | If this Axie's HP dropped below 50% this round, deal 150% damage and draw 1 card. If HP is above 50%, deal base damage. | Reactive burst. Gets hit, then hits back harder. 100 ATK baseline is still acceptable when condition isn't met. |
| 4 | Blood Frenzy | 1 | Melee | 90 | 15 | Each time this Axie crits this round, this card's damage is increased by 30 (applies retroactively to future hits in combo). | Snowball mechanic. First crit powers up remaining cards. High morale makes the first crit likely. |
| 5 | Pack Hunter | 1 | Melee | 95 | 25 | Deal +15% damage for each ally that attacked the same target this round (before or after). | Team coordination without requiring same-class chain. Any ally hitting same target helps. |
| 6 | Primal Guard | 1 | Melee | 70 | 45 | Apply Regen (1 stack) to this Axie for 2 rounds. | Offensive card with sustain. 70 ATK isn't dead damage, and Regen (20% max HP/round) helps Beast survive to combo turns. |
| 7 | Ambush Pounce | 1 | Melee | 115 | 15 | If played as the only card this round (no combo), gain Speed+ for 2 rounds and attack before fighters with equal speed. | Anti-combo design. Rewards playing a single card per round, which is what happens when draws are bad. |
| 8 | Rage Swipe | 1 | Melee | 85 | 25 | Deal damage to target. Then deal 40 damage to each adjacent enemy. | AoE damage. Guaranteed value against multi-Axie boards without needing conditions. |
| 9 | Warchief's Banner | 0 | Support | 0 | 30 | Apply Attack+ and Morale+ to all Beast allies for 1 round. This Axie takes 10% of its max HP as self-damage. | Cheap team-wide buff. Self-damage pushes toward Last Stand which activates other Beast cards. |
| 10 | Undying Fury | 1 | Melee | 80 | 30 | This card cannot be discarded by opponent effects. When this Axie enters Last Stand, play this card automatically (free, targets nearest enemy). | Auto-cast on Last Stand. Guaranteed value -- the card finds its own timing. |
| 11 | Relentless Charge | 1 | Melee | 100 | 20 | After this card resolves, if target survived, put this card on top of your draw pile instead of discarding it. | Recursive card. If you don't kill, you get it back immediately. Reduces deck dilution. |
| 12 | Claw Flurry | 1 | Melee | 35 | 20 | Hit 3 times. Each hit that crits heals this Axie for 20 HP. | Multi-hit + crit healing. With high morale, expect 1-2 crits per use = 20-40 HP healed. Reliable sustain. |
| 13 | Intimidating Presence | 1 | Melee | 90 | 30 | Apply Speed- to all enemies for 1 turn. | AoE Speed debuff. Directly solves the "Beast goes last" problem for the entire team. |
| 14 | Counter Stance | 1 | Melee | 70 | 40 | If this Axie is attacked this round, the next card this Axie plays deals +50% damage (persists until used). | Stores value for future rounds. Getting hit charges up a future card regardless of what that card is. |
| 15 | Mauling Strike | 2 | Melee | 145 | 20 | Guaranteed critical hit. Apply Chill to target for 2 turns. | Expensive but unconditional crit. No combo needed. Chill prevents enemy Last Stand. 2-energy cost balances the power. |
| 16 | Savage Leap | 1 | Melee | 105 | 20 | Target the enemy with the lowest HP instead of the nearest. If target dies, gain 1 energy. | Execute card. Finds kills even on the backline. Energy refund on kill rewards aggression. |
| 17 | Survival Instinct | 0 | Support | 0 | 45 | If this Axie has no other Beast cards in hand, draw 2 cards. Otherwise, gain 25 shield. | Contextual utility. Draws when you need cards, shields when you have your combo. Always useful. |
| 18 | Stampede | 1 | Melee | 80 | 20 | Deal +10% damage for each card in your hand (max +60% at 6+ cards). | Rewards card accumulation. If you saved cards waiting for the right moment, this grows strong. |
| 19 | Feral Resilience | 1 | Melee | 90 | 30 | When this card is in your discard pile at the start of a round, gain +1 Last Stand tick (once per game). | Passive benefit from discard pile. Guarantees extra Last Stand survivability just by playing the card once. |
| 20 | Alpha Strike | 1 | Melee | 110 | 20 | If this Axie dealt damage last round, deal +25% damage. If this Axie did not play a card last round, draw 1 card instead. | Fork card -- two different benefits depending on situation. Active last round = damage bonus. Inactive last round = card draw. Never feels wasted. |

---

## New Tail Card Options (20)

| # | Name | Energy | Type | ATK | DEF | Effect | Design Rationale |
|---|------|--------|------|-----|-----|--------|-----------------|
| 1 | Tenacious Tail | 0 | Support | 0 | 35 | Gain 1 energy. If this Axie has 50% HP or less, gain 2 energy instead. | Upgraded Luna Absorb. Same free energy baseline, but scales when damaged -- which Beasts always are. |
| 2 | Whip Lash | 1 | Melee | 100 | 20 | If this Axie attacked last in the round, deal 140% damage. | Direct "slowness payoff." Transforms the speed disadvantage into a 140 effective ATK card. |
| 3 | Tail Sweep | 1 | Melee | 70 | 25 | Hit all enemies for 70 damage. Apply Morale+ to self for 1 turn. | AoE damage. Guaranteed value against all 3 enemies, doesn't need combo or conditions. |
| 4 | Bristle Tail | 1 | Melee | 85 | 35 | When this Axie takes damage this round, gain +10 shield (max 3 triggers, 30 shield total). | Reactive defense. Slow Beast gets hit often -- this converts incoming attacks into shields. |
| 5 | Scavenger's Instinct | 0 | Support | 0 | 20 | Look at the top 3 cards of your deck. Put 1 into your hand and the rest back in any order. | Scry + draw. Directly solves the draw RNG problem. Free to play. |
| 6 | Fury Tail | 1 | Melee | 95 | 20 | Deal +5% damage for each point of Morale this Axie has above 40 (e.g., +105% at morale 61). | Morale-scaling damage. A pure Beast (61 morale) gets +105% = 195 damage. Rewards the archetype directly. |
| 7 | Dying Wish | 0 | Support | 0 | 30 | When this Axie dies or enters Last Stand, deal 80 damage to the enemy that killed it and apply Lethal for 2 turns. | Death trigger. Creates value even when the Beast dies. No timing dependency at all. |
| 8 | Lashing Frenzy | 1 | Melee | 50 | 15 | Hit 2 times. If comboed with any card, hit 3 times instead. | Multi-hit with easy condition. 2 hits baseline (100 ATK equivalent), 3 on any combo (150 ATK). |
| 9 | Persistence | 1 | Melee | 90 | 25 | This Axie gains +1 Last Stand tick. If already in Last Stand, deal 150% damage. | Dual-purpose. Before Last Stand: invest in survivability. During Last Stand: burst. |
| 10 | Tail Feint | 1 | Melee | 95 | 20 | Apply Stench to this Axie for 1 turn after attacking. | Self-Stench. After dealing damage, enemies are forced to target someone else. Pseudo-evasion for Beast. |
| 11 | Rallying Cry | 0 | Support | 0 | 25 | All allies gain Morale+ for 2 turns. If this Axie is the only Beast, also gain Attack+ for 2 turns. | Team Morale buff. Works in mixed teams (solo Beast gets a bonus). Improves crit chance for everyone. |
| 12 | Stored Power | 1 | Melee | 80 | 20 | Deal +20% damage for each unspent energy your team has (max +80% at 4 energy). | Energy hoarding payoff. If you save energy waiting for the right cards, this grows. 80 ATK + 80% = 144 at 4 energy. |
| 13 | Chain Breaker | 1 | Melee | 105 | 25 | Remove all debuffs from this Axie before dealing damage. Deal +10% damage for each debuff removed. | Self-cleanse + damage. Beasts often get debuffed by faster enemies -- this turns debuffs into damage. |
| 14 | Shadow Pounce | 1 | Melee | 90 | 15 | Target the same enemy your previous card targeted this round. If not in combo, target the enemy that last attacked this Axie. | Smart targeting. In combo: focused fire for kills. Solo: retaliation. Never targets randomly. |
| 15 | Endurance | 0 | Support | 0 | 40 | Gain Regen (1 stack) for 2 rounds. If this Axie has Morale+, gain 2 stacks instead. | Free sustain. Regen at 20% max HP per stack per round is significant. Morale+ synergy. |
| 16 | Tail Spin | 1 | Ranged | 85 | 20 | Steal 1 random buff from target. If target has no buffs, gain 1 energy instead. | Buff steal with a fallback. Always does something -- steal or energy, plus 85 ATK. |
| 17 | Cornered Beast | 1 | Melee | 100 | 30 | If this is your only Axie alive, gain Speed+ and Attack+ for the rest of the game (non-stacking). | "Last man standing" permanent buff. Beast is often the last Axie. Permanent Speed+ solves the speed problem. |
| 18 | Ferocious Swipe | 1 | Melee | 110 | 15 | If this Axie has Attack+, guaranteed critical strike. | Reliable crit with simple condition. Pair with Self Rally, Warchief's Banner, or any Attack+ source. |
| 19 | Primal Memory | 1 | Melee | 85 | 25 | After playing, add a random Beast card that was previously discarded this game back to your hand. | Card recycling. Reduces the "I used my good cards and now I have nothing" problem. |
| 20 | Last Gasp | 1 | Melee | 75 | 30 | If this Axie would die this turn, instead survive with 1 HP and deal 200% damage. Once per game. | Cheat death. Guarantees at least one big hit even if the Beast gets bursted. Once-per-game balances it. |

---

## Summary of Design Categories

### Cards that solve "wrong cards at wrong time" (draw fixing)
- Primal Roar (Mouth #2), Feral Instinct (Mouth #15), Desperate Lunge (Mouth #20)
- Shedding Antler (Horn #10)
- Survival Instinct (Back #17)
- Scavenger's Instinct (Tail #5), Primal Memory (Tail #19)

### Cards that turn slowness into a strength
- Jaw Trap (Mouth #3), Snarl (Mouth #16)
- Spiral Horn (Horn #8)
- Counter Stance (Back #14)
- Whip Lash (Tail #2)

### Cards that reduce combo dependency
- Siege Breaker (Horn #7), Bull Rush (Horn #13)
- Ambush Pounce (Back #7), Alpha Strike (Back #20)

### Cards that build value over time (reduce burst-or-bust)
- Momentum Horn (Horn #3), Fury Horn (Horn #9), Primal Charge (Horn #15), Ancestral Horn (Horn #20)
- Berserk Stance (Back #1), Feral Resilience (Back #19)

### Cards that create value even when Beast dies
- Dying Wish (Tail #7), Last Gasp (Tail #20)
- Undying Fury (Back #10)

### Cards that address the Aquatic/Bird speed + class disadvantage problem
- Impale (Horn #11) -- ignores class disadvantage vs Aquatic/Bird

### Cards that punch through shields (relevant vs Plant/Reptile despite class advantage)
- Bone Crush (Mouth #5), Tearing Bite (Mouth #12)
- Battering Ram (Horn #5)

### Balance Notes
All stat values are calibrated against existing Beast card ranges (0-130 ATK, 0-50 DEF, 0-2 energy) and the damage formula (card_attack * same_class_bonus * class_advantage * combo_bonus * crit_multiplier). No card exceeds the power level of existing top-end combinations like Sinister Strike crit (130 * 1.1 * 2.0 = 286 effective) or Risky Beast in Last Stand (120 * 1.1 * 2.0 = 264 effective).
