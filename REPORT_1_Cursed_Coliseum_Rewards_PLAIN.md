# Cursed Coliseum Chests — Plain-Language Summary

## What's changing
We're removing artifacts and gear from the game. Today, Cursed Coliseum chests drop one of two artifacts — that's all they drop. We need to replace what's inside those chests with something else, mainly gems.

## Today's situation, quickly
- Players play up to **2 Cursed Coliseum sessions per day** and earn up to **2 chests per day**.
- Each chest currently contains **one artifact**, drawn from a pool of two options.
- That's the entire chest economy that's being replaced.

## Three ways to handle it

### Option A — Just give gems (simplest)
Replace the artifact inside each chest with a gem reward. The more wins a player earned in the session, the bigger the gem payout from that chest.

**Why pick this:**
- Ships in days.
- No new UI work needed — the game already knows how to show gem rewards.
- Easy for players to understand.

**The trade-off:**
- Every chest looks identical. The "what did I get?" thrill is gone.
- All of the gem-economy pressure lands on this one source.

---

### Option B — Bronze / Silver / Gold Cursed chests
Instead of one chest type, players earn a Bronze, Silver, or Gold Cursed chest depending on how many wins they had in the session. Each tier mostly drops gems but also throws in a free wheel spin, a lottery ticket, or a cosmetic.

**Why pick this:**
- Keeps the "rare drop" feel. The chest itself becomes the variety, instead of the artifact inside it.
- Maps naturally onto the existing "premium session" idea if we want to add a premium upsell later.
- Players who push for more wins feel rewarded with visibly better chests.

**The trade-off:**
- More design and backend work than Option A.
- Players who farm only minimum-win sessions get visibly worse chests — fine for a competitive arena, but worth playtesting.

---

### Option C — Chests fill a milestone track
Each chest pays a small gem amount immediately and also fills a milestone track. Hit a milestone, get a big bundle. Track resets each season.

**Why pick this:**
- Best long-term shape — rewards can rotate every season without rewriting the chest table itself.
- Revives a half-finished feature ("collectibles track") that was already started by the previous team.

**The trade-off:**
- Most engineering work of the three.
- Needs an audit of the old half-built code before committing.
- Loses the "open the chest" moment unless we keep a thin animation on the immediate gem grant.

## Other items we could pay out alongside gems
- **Free wheel spins** — pulls players back to the daily wheel.
- **Lottery tickets** — only if we keep the lottery feature.
- **Gold** — a softer currency, good as filler in mid-tier chests.
- **Stickers / cosmetics** — already exist in the shop today.
- **SLP** — if the broader economy allows it; meaningful for top-tier chests in Option B.

## One thing to watch for
"Dust" is a currency that only exists today for gear crafting. Once gear is gone, dust has nothing to spend it on. **Don't put dust in chests until we give it a new purpose** — otherwise it becomes worthless inventory the day this ships.

## Recommendation
- Need to ship fast? **Option A.**
- Want to keep the variety players expect from a chest? **Option B.**
- Want this to be the foundation of a rotating seasonal loop? **Option C.**
