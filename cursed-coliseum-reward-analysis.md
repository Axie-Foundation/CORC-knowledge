# Axie Classic -- Cursed Coliseum Reward Scaling Analysis

**Date:** 2026-05-19
**Source:** `cp-matchmaker/domain/colosseum/`, `cp-matchmaker/domain/chest/`, `cp-matchmaker/app/api/v1/service/reward/serviceimpl/colosseum.go`, `cp-matchmaker/infra/options/colosseum.go`

---

## Session Structure

- **Max 7 wins** to complete, **2 losses** and you're eliminated
- Entry fee escalates per session: `100 gold * 1.1^(sessions_today)` (100, 110, 121, 133...)
- Premium mode (web3 only): flat 100 SLP entry fee
- Max 8 sessions per day
- Session expires after 24 hours (with 30-min grace period)
- Before each battle, one Axie gets "cursed" -- 2 of its parts are randomly replaced (the core roguelike mechanic)

---

## Per-Win Rewards (During Session)

AXP scales with consecutive win streak. Formula: `200 * currentWinStreak`.

| Win # | Streak | AXP Earned |
|-------|--------|------------|
| 1st win | 1 | 200 |
| 2nd win | 2 | 400 |
| 3rd win | 3 | 600 |
| 4th win | 4 | 800 |
| 5th win | 5 | 1,000 |
| 6th win | 6 | 1,200 |
| 7th win | 7 | 1,400 |

A loss resets the streak to 0. Example: a player who goes W-W-W-L-W-W-W-W earns 200+400+600+0+200+400+600+800 = 3,200 AXP. A perfect 7-0 run earns 200+400+600+800+1000+1200+1400 = **5,600 AXP**.

Each win also gives **1 Guild XP**. Web3 players additionally roll for probabilistic **SLP chest drops** per win (weighted by win count and reputation score).

A player who loses both games without any wins receives **100 AXP** as consolation.

---

## End-of-Session Rewards (Gold/SLP)

Only awarded when the session ends (either 7 wins or 2 losses):

| Wins | Gold (Normal) / SLP (Premium) |
|------|------|
| 0 wins (0W-2L) | 0 (+ 100 AXP consolation) |
| 1 win | 15 |
| 2 wins | 25 |
| 3 wins | 120 |
| 4 wins | 200 |
| 5 wins | 300 |
| 6 wins | 400 |
| 7 wins (with losses) | 500 |
| 7 wins (perfect, 0 losses) | **1,000** (2x bonus) |

The jump from 2 to 3 wins (25 to 120 gold) is the biggest step in the reward curve -- that's the breakpoint where "good enough to profit" kicks in, since the entry fee is 100 gold. The perfect run bonus (1,000 vs 500) is a strong incentive for flawless play.

---

## Guild Points (End-of-Session)

Awarded at session end if the Coliseum is in a guild season:

| Wins | GP |
|------|-----|
| 0 | 0 |
| 1 | 2 |
| 2 | 4 |
| 3 | 6 |
| 4 | 8 |
| 5 | 10 |
| 6 | 13 |
| 7 | 16 |

---

## Colosseum Chest (End-of-Session)

One chest drops per session (max 2 held at once, 12-hour lock timer). Contents are either an artifact OR a bonus wheel spin, with artifact chance scaling by wins:

| Wins | Artifact Chance | Spin Chance |
|------|----------------|-------------|
| 0 | 12.5% | 87.5% |
| 1 | 25% | 75% |
| 2 | 37.5% | 62.5% |
| 3 | 50% | 50% |
| 4 | 62.5% | 37.5% |
| 5 | 75% | 25% |
| 6 | 87.5% | 12.5% |
| 7 | 100% | 0% |

Formula: `artifact_chance = (wins + 1) / 8`

---

## Collectible MAXS Bonus (7 wins only, rank-gated)

If your team includes Axies with special collectible tags and you complete the session (7 wins), you compete for a limited leaderboard spot:

| Tag | Top N Slots | MAXS Reward |
|-----|------------|-------------|
| Mystic | 200 | 2,000 |
| Origin | 200 | 1,000 |
| Xmas | 200 | 1,000 |
| Japan | 400 | 500 |

These are additive -- a team with both Mystic and Origin tags can earn 3,000 MAXS.

---

## Player Achievements

On completing 7 wins:
- **0 losses:** "Perfect" achievement
- **Any losses:** "Master" achievement

---

## The Key Design Tension

The reward curve heavily front-loads AXP (streak-based, per win) and back-loads gold/SLP (milestone-based, end of session). This creates different incentive structures depending on what the player is optimizing for:

- **Grinding AXP** is best served by winning early and often, even if you lose the session. A 3W-2L session with a 3-win streak still earns 1,200 AXP from wins alone.
- **Earning gold/SLP** requires completing sessions, ideally with 3+ wins. The 2-to-3-win jump (25 to 120 gold) is where sessions become profitable relative to the 100 gold entry fee.
- **The perfect run** is disproportionately rewarded: 5,600 AXP + 1,000 gold + guaranteed artifact chest + 16 GP + achievement. This is roughly 3x the value of a 7W-1L run.
- **Win streak risk:** A loss mid-session doesn't just cost one game -- it resets the AXP multiplier. Going W-W-W-L-W-W-W-W earns 3,200 AXP. Going W-W-W-W-L-W-W-W earns 3,600 AXP. The later the loss occurs, the less AXP is lost (because the post-loss streak has fewer remaining games to rebuild).

### Profitability Breakpoints

| Wins | Gold Earned | Entry Fee | Net Profit | Profitable? |
|------|------------|-----------|------------|-------------|
| 0 | 0 | 100 | -100 | No |
| 1 | 15 | 100 | -85 | No |
| 2 | 25 | 100 | -75 | No |
| 3 | 120 | 100 | +20 | **Yes** |
| 4 | 200 | 100 | +100 | Yes |
| 5 | 300 | 100 | +200 | Yes |
| 6 | 400 | 100 | +300 | Yes |
| 7 | 500 | 100 | +400 | Yes |
| 7 (perfect) | 1,000 | 100 | +900 | Yes |

*Entry fee shown is for the first session of the day (100 gold). Later sessions cost more.*
