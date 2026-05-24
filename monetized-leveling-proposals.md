# Axie Classic -- Monetized Leveling Proposals

**Date:** 2026-05-11
**Goal:** Allow players to spend money to reach competitive parity, eliminating the XP grind barrier while maintaining a level playing field.

---

## The Core Tension

Axie Classic currently requires extensive XP grinding across multiple games in the ecosystem to level up Axies. This creates two problems:

1. **New players can't compete.** A skilled player with level-5 Axies loses to a mediocre player with level-20 Axies because of raw stat differences and locked parts.
2. **The grind dependency falls on other games.** Axie Classic's progression is bottlenecked by time spent in Origins, Homeland, etc. -- games the player may not enjoy.

The solution is to let players pay to skip the grind. The critical distinction: this is **pay-to-skip**, not **pay-to-win**. If the endgame is a level playing field where all max-level Axies are equal, then paying to reach that field faster gives no ongoing advantage. The player who paid and the player who ground are identical once they arrive.

---

## Proposal 1: Instant Max-Level Axie Passes

The most direct approach. Sell a consumable item that immediately sets an Axie to max level with all parts unlocked.

### How It Works
- Purchase an "Ascension Crystal" from the shop ($2-5 per Axie, or ~500 SLP equivalent)
- Apply it to any Axie you own -- that Axie is instantly max level with all cards/parts available
- No XP grinding required. The Axie is immediately competitive
- The crystal is consumed on use (one per Axie)

### Pricing Tiers
- Single crystal: $3
- Team pack (3 crystals): $7 (enough for one full team)
- Collector pack (10 crystals): $20 (for players who rotate teams frequently)

### Why It Works
- Zero ambiguity -- you pay, you're competitive. No grinding, no waiting
- The price is low enough that it doesn't feel exploitative but high enough to generate revenue
- Players who prefer grinding can still earn XP the traditional way -- this is an alternative, not a replacement
- Preserves the level playing field: a paid max-level Axie is identical to a ground max-level Axie

### Risks
- May feel like the game is "charging admission" to the real game
- Could devalue the time investment of players who already ground to max level (needs a migration reward for existing max-level players)
- Very simple -- may not generate ongoing revenue after the initial purchase wave

---

## Proposal 2: Season Pass Max-Level Subscription

Bundle competitive readiness into the seasonal structure. Each season pass automatically keeps all your Axies at max level for the duration of the season.

### How It Works
- Free players level Axies through XP at the current (or slightly accelerated) rate
- Season Pass holders ($10-15/season, ~3 months) get:
  - All owned Axies automatically set to max level for the season
  - All cards/parts unlocked on all owned Axies
  - When the season ends, Axies revert to their "earned" XP level unless the pass is renewed
  - If you buy a new Axie mid-season, it's immediately max level while the pass is active

### Why It Works
- Recurring revenue model (predictable for the business)
- Naturally bundles with the Challenge Passport / Battle Pass from the progression redesign
- Frames the payment as "season access" rather than "buying power"
- Free players aren't locked out -- they grind to the same endpoint, just slower
- Creates urgency: "get the pass to be competitive this season from day one"

### Risks
- Players who let the pass lapse feel punished (Axies drop back to earned level)
- Subscription fatigue if combined with too many other premium features
- The revert mechanic could feel hostile -- consider making it a one-way ratchet instead (once an Axie hits max during a pass, it stays max permanently)

---

## Proposal 3: XP Market (Player-to-Player Level Boosting)

Create an in-game marketplace where players can buy and sell XP tokens, allowing veteran players to monetize their grinding and new players to skip it.

### How It Works
- Playing any Axie game (Classic, Origins, Homeland) earns "Essence" alongside normal XP
- Essence is a tradeable token that can be applied to any Axie for instant XP
- Players can list Essence on an in-game marketplace for SLP or AXS
- Buyers purchase Essence and apply it to their Axies to level up instantly
- The marketplace sets a natural price floor/ceiling based on supply (grinders) and demand (new players)

### XP Equivalence
- 1 Essence = 100 XP (example ratio)
- Max level requires ~10,000 XP = 100 Essence per Axie
- Market price self-regulates: if Essence is too cheap, fewer people grind; if too expensive, more people grind

### Why It Works
- Creates a real economy connecting grinders and competitive players
- Grinders get financial reward for their time investment (validating their effort)
- Competitive players get fast access to max level (paying for convenience)
- Sky Mavis takes a small marketplace fee (2-5%) on each transaction
- Uses existing blockchain infrastructure (SLP, AXS, Ronin marketplace)

### Risks
- Essence farming bots could flood supply and crash prices
- Adds economic complexity -- players need to understand another currency
- Could cannibalize direct purchase revenue (why buy crystals when Essence is cheaper?)
- Requires marketplace development and moderation

---

## Proposal 4: Graduated Level Unlock Pricing (Pay-Per-Tier)

Instead of one flat price, create a tiered system where players can buy exactly the amount of progression they want, with diminishing grind requirements at each tier.

### How It Works
- Axie leveling is divided into 4 tiers:
  - **Tier 1 (Levels 1-5):** Free, earned in ~2-3 hours of play. Basic cards unlocked.
  - **Tier 2 (Levels 6-10):** Earnable through ~10 hours of play, OR purchasable for $1. Full card set unlocked.
  - **Tier 3 (Levels 11-15):** Earnable through ~30 hours of play, OR purchasable for $2. Stat optimization unlocked.
  - **Tier 4 (Levels 16-20, max):** Earnable through ~80 hours of play, OR purchasable for $3. Competitive max stats.
- Each tier is purchased independently -- buy only what you need
- Purchasing a tier also retroactively grants all lower tiers
- Bundle: "Max Level Pass" ($5) unlocks all 4 tiers instantly

### Why It Works
- Players choose their own tradeoff between time and money
- Tier 1 is free and fast -- everyone can play within hours
- The crucial "full card set" unlock (Tier 2) is only $1 -- low barrier to competitive viability
- Total cost to max is only $5 per Axie -- extremely accessible
- Grind hours per tier increase exponentially, making the paid shortcut more valuable at higher tiers

### Risks
- Players may feel nickeled-and-dimed by 4 separate tiers (mitigated by the bundle)
- Tier 2 at $1 may be too cheap to generate meaningful revenue (could adjust upward)
- Still requires some grind for free players -- doesn't completely eliminate the problem

---

## Proposal 5: Competitive License Model (Flat Fee for Full Access)

The most radical approach. Separate the game into a free casual mode and a licensed competitive mode where all Axies are automatically max level.

### How It Works
- **Casual Mode (free):** Play with Axies at their earned XP level. Unranked matches, Training Grounds, Collection Atlas. No level restrictions, but matchmaking considers Axie level.
- **Competitive License ($15-20, one-time purchase):** Unlocks Competitive Mode where:
  - All Axies you own are treated as max level with all parts unlocked
  - Ranked ladder, tournaments, seasonal challenges, Strategist Rank
  - No level-based matchmaking adjustment -- pure skill-based matching
  - License is permanent (not seasonal)
- In Competitive Mode, the game normalizes all stats -- every player is on equal footing regardless of actual Axie level

### Why It Works
- Completely eliminates the grind-for-competitive problem
- One-time purchase means no subscription fatigue, no recurring cost anxiety
- Casual mode remains fully free -- no paywall for the game itself
- Competitive Mode is explicitly the "level playing field" the design goals call for
- $15-20 is a standard game price and signals quality (not free-to-play exploitation)
- Creates a clear dividing line: casual play is for fun, competitive play is for skill

### Risks
- Splits the playerbase into two pools (casual and competitive)
- Players may resent paying for "the real game" -- perception of a paywall
- Revenue is one-time, not recurring (could be offset by seasonal cosmetic sales)
- Existing max-level players may feel cheated if they ground for free and now others can pay $15

---

## Comparison Matrix

| Approach | Price Point | Revenue Model | Grind Eliminated? | Pay-to-Win Risk | Implementation Effort |
|----------|-----------|--------------|-------------------|----------------|----------------------|
| 1. Instant Max-Level Pass | $3-7 per team | One-time per Axie | Yes (per Axie) | None (same endpoint) | Low |
| 2. Season Pass Subscription | $10-15/season | Recurring | Yes (while active) | None (same endpoint) | Medium |
| 3. XP Market | Player-set | Transaction fees | Yes (via purchase) | None (same endpoint) | High |
| 4. Graduated Tier Pricing | $1-5 per Axie | One-time per tier | Partially (per tier) | None (same endpoint) | Medium |
| 5. Competitive License | $15-20 once | One-time | Yes (in comp mode) | None (normalized stats) | Medium-High |

---

## Recommended Approach

**Option 1 (Instant Max-Level Pass)** is the simplest and most direct. It does exactly what the design goals ask for: eliminates the grind, creates a level playing field, and generates revenue without creating power gaps. The low price ($3-7 per team) makes it accessible, and players who prefer grinding still can.

**Option 5 (Competitive License)** is the boldest and most philosophically clean. It creates an explicit separation between "play for fun" and "play to compete," which aligns with the broader vision of Axie Classic as a high-skill competitive game. The one-time $15-20 price is a proven model (chess.com, competitive card games).

**Option 1 + Option 5 combined** is the strongest package: sell Ascension Crystals for Axie leveling in casual mode, and sell a Competitive License that normalizes everything in ranked. This gives players three paths to competitive play -- grind, pay per Axie, or buy the license.
