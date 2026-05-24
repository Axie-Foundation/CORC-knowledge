# Battle Pass Rewards — Plain-Language Summary

## What's changing
We're removing artifacts and gear from the game. The Battle Pass currently rewards artifacts at many tiers; those need to be replaced, mostly with gems.

## How the Battle Pass works today (quick recap)
- Two tracks run side by side: a **free track** everyone uses, and a **premium track** that players pay gems to unlock.
- Each track has a series of tiers. Players progress by completing quests.
- The rewards at each tier are set by the server — we can change them without shipping a new client build.
- Artifacts are currently the main "item" players collect from tiers. Gear is technically defined as a reward type, but the code never actually delivers it (a half-built feature). Removing artifacts therefore strips the main collectible from every tier.

## Three ways to handle it

### Option A — Gem ramp (simplest)
Replace every artifact tier with a gem reward. Higher tiers pay more gems. Premium-track tiers pay roughly 2× the same-position free-track tier.

**Why pick this:**
- Zero new client code — the game already knows how to display gem rewards.
- Fastest cutover.
- Players know exactly what they're working toward.

**The trade-off:**
- Loses variety. Every tier becomes the same gem icon with a different number.
- The premium unlock fee is paid in gems and the rewards come back in gems — the numbers have to work out so buying premium clearly comes out ahead.

---

### Option B — Free track is light, premium is heavy
Free track stays useful but small: small gem amounts mixed with free wheel spins and lottery tickets. Premium track pays substantially more gems plus a few "milestone bundles" at standout tiers (think tier 25, 50, 75). Optionally, 3–4 premium-only tiers reward unique cosmetics like a special sticker set.

**Why pick this:**
- Strongest monetization signal — premium has clear added value at every tier, not just at unlock.
- Free players still get a regular feel-good (those wheel spins keep the daily wheel alive after artifacts are gone).

**The trade-off:**
- More careful tuning — if the free track gets too thin, players resent it; too thick, premium is hard to sell.
- Two distinct reward ramps to maintain.

---

### Option C — Pass Marks → choose your own reward
Each tier pays a "Pass Mark" token. Players spend Marks in a season-long shop on whatever they want — gems, free spins, lottery tickets, stickers. Marks expire at season end.

**Why pick this:**
- Maximum flexibility — change what Marks buy every season without touching tier configs.
- Brings back the "I'm collecting toward something" feel that artifacts used to deliver, with the player choosing what to collect toward.

**The trade-off:**
- Biggest engineering build of the three (new currency, new shop, new endpoint, expiry rules).
- Adds an indirection some players resent ("why didn't I just get gems?").

## Other items we could pay out alongside gems
- **Free wheel spins** — already supported by the game; natural fit.
- **Lottery tickets** — only if lottery stays.
- **Gold** — filler currency for lower-tier rewards.
- **Stickers / cosmetics** — they exist in the shop today, but wiring them as Battle Pass rewards is a small new feature.

## One thing to watch for
The premium track unlock is paid in gems today. If we then turn the Battle Pass into a heavy gem faucet, players are essentially being paid back in the currency they just spent. The premium-track gem total has to clearly exceed the unlock cost for premium to feel worth buying.

Also: don't pay dust through the Battle Pass — gear crafting was its only use, and we're removing that.

## Recommendation
- Need to ship fast? **Option A.**
- Want premium to feel distinctly valuable at every tier? **Option B.**
- Want maximum room to rotate seasonal rewards? **Option C.**
