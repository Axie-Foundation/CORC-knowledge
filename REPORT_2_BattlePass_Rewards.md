# Report 2 — Battle Pass Rewards

**Date:** 2026-05-23
**Scope:** Three alternate reward-distribution approaches for the Battle Pass (BP) that replace artifacts and gear with in-game gems as the primary substitute. Includes alternate non-gem reward ideas and the code surface required.

---

## 1. Baseline — how the Battle Pass pays out today

Every claim here is from the codebase.

### 1.1 Two tracks, server-driven tiers

The BP runs as two parallel tracks. `BattlePass.cs` (under `Home/BattlePass/Models.cs`, lines 63–75) defines `BattlePass.Type` as `normal` and `premium`. Each track is a list of `Milestone` objects with these fields (lines 77–83):

- `tier` — string label such as `"bronze-3"`, `"gold-1"`
- `index` — 0-based slot number
- `score` — BP-score threshold required to claim
- `rewards` — `List<RewardItem>`
- `claimed` — whether the player has already claimed it

The tier list itself, the score thresholds, and the rewards-per-tier are **all server data**, returned by `GET {MatchMakerPrefix}/season/user-battle-passes` (see `Actions.BattlePass.cs`). The client never hard-codes a tier-by-tier reward list — it just renders what the backend returns.

### 1.2 Premium track unlock

The premium track is purchased via `POST {MatchMakerPrefix}/season/user-battle-pass/buy`. The unlock cost is **whatever the server returns in `premium.cost`** — a `Currency` object that the client renders unmodified through `BuyBattlePassView.cs:26–28` and `CurrencyView`. The remote-config flag `RMConfig.inst.battlePass.premiumEnable` (in `RMConfig.Pro.cs:343`) gates whether the premium track is even visible. There is **no hard-coded "premium costs 1000 gems" anywhere in the client** — the price is a number on the server, today most likely paid in `gem` (the only currency the gear pipeline also priced in), but the engine doesn't care.

When the premium track is purchased, the reducer auto-marks every normal-track milestone as already-claimed (`BattlePassDataReducer.cs:53–59`); the player does not retroactively pick up normal-track rewards.

### 1.3 What rewards a tier can grant

`BattlePass/Models.cs` lines 86–92 define the full `ItemType` enum:

- `currency` — handled in `RewardPopup.cs` via `CurrencyView`
- `artifact` — added to inventory by `ArtifactsDataReducer.cs:76–99` on `BattlePass_Claim`
- `bonus_spin` — daily-wheel free spins, rendered by `RewardPopup.cs:56–58`
- `lottery` — lottery tickets, surfaced in `MilestoneView.cs:37–40`
- `gear` — **declared in `ItemType` but not handled by `GearsDataReducer.cs`**. If the server today returns gear in a BP milestone, the client silently drops it. Because gear is being removed anyway, this dead branch can simply be deleted; do not invest time wiring it up.

The currency sub-types available (`Common.cs:86–96`):

- Web2: `gem`, `gold`, `dust`
- Web3: `slp` (+ `slp_in`, `slp_ex`), `maxs` (+ `maxs_in`, `maxs_ex`), `axs`, `ron`, `weth`, `usdc`

### 1.4 How a player earns BP score

`Actions.BattlePass.cs:26–29` follows a `BattlePass_Buy` action with `Quest_GetList`, and `Actions.Quest.cs:34–45` shows quests as the proximal score source — the server increments BP score when a quest is claimed via `POST {MatchMakerPrefix}/user-quests/claim`. Quests are described in `Quest/Models.cs:16–45`:

- `category` (string, e.g. `"daily_quests"`)
- `timeCycle` — `"daily"`, `"weekly"`, `"monthly"`, `"seasonally"`, `"lifetime"`
- `targetProgress` / `currentProgress`
- `rewards` and `premiumRewards` — `List<FlatReward>`

The actual quest catalog (which quests exist, what their target counts are, how much BP score each grants) is **not in the client** — it's served by the matchmaker. There is no hard-coded per-day BP-score cap visible in the client code.

### 1.5 What lives where on the client (engineering-impact map)

- `BattlePassHubView.cs` — track UI, progress bar (lines 159–188 do the `gainedScore / deltaScore` calc), "Claim All" plumbing
- `MilestoneView.cs` — per-tier reward icons and amounts
- `MilestonePair.cs` — groups the free + premium tier; line 16 enforces the premium lock state via `!premium.owned && RMConfig.premiumEnable`
- `BuyBattlePassView.cs` — purchase flow; cost rendered from server response
- `BattlePassDataReducer.cs` — state slice; refresh window is 60s (`BattlePassData.outdated`)
- Per-reward-type reducers — `ArtifactsDataReducer.cs` (currently handles BP claim), `GearsDataReducer.cs` (does not handle BP claim — confirmed by inspecting the full file)
- `RewardPopup.cs` — generic reward display; already routes `currency`, `artifact`, `bonus_spin`, `lottery`

### 1.6 Adjacent progression systems (so they don't get confused with BP)

`home-progression` (the codemap module) groups four distinct systems:

- **Battle Pass** — this report
- **Tower of Solo** — separate climb event (`Home/Tower/TowerHubView.cs`, `FloorView.cs`)
- **Daily Wheel** — separate spin reward (`Home/Wheel/WheelView.cs`), today can drop artifacts (`ArtifactsDataReducer.cs:102–117`)
- **Lunacia PvE Campaign** — story progression (`Home/HomePvELobby/`)

None of those three rolls into BP score directly per the searched code; they are independent loops.

### 1.7 Critical: today the BP code path can deliver artifacts but not gear

`ArtifactsDataReducer.cs` lines 76–99 explicitly handles the `BattlePass_Claim` flux action and appends artifact rewards to inventory. `GearsDataReducer.cs` has no equivalent case. So in practice **today's BP awards artifacts but never gear**, regardless of what tier configs the backend ships. Removing artifacts therefore removes the only meaningful tier-prize "item" the BP currently delivers — everything else is currency, bonus_spin, or lottery. The proposals below all account for this.

---

## 2. Approach A — "Currency Track" (uniform gem ramp)

### Concept
Replace every artifact tier in both the normal and premium tracks with `currency / gem` rewards. The ramp is monotone: each tier pays more gems than the last, with premium-track tiers paying about 2× the same-index normal tier.

### Distribution shape
Two design knobs:

- **Per-tier gem amount.** Use the existing `Currency.gem` `RewardItem` shape (`{ itemType: "currency", itemInfo: { id: "gem", amount: N } }`). The client renders this through `CurrencyView` already; zero client work.
- **Tier count.** Server-driven via `BattlePassData.items[].milestones.Count`. The exact season length stays whatever ops currently runs.

A worked illustrative ramp (the design team picks the absolute numbers; the shape is what matters):

| Tier index | Normal-track gems | Premium-track gems |
|---|---|---|
| Early (first ~10 tiers) | Small (e.g. 10–20) | ~2× normal |
| Mid (middle ~20 tiers) | Medium (e.g. 40–80) | ~2× normal |
| Late (final ~10 tiers) | Large (e.g. 150–500) | ~2× normal + a finale bundle |

This ramp lives entirely on the matchmaker side; the only client change is **deleting the `ItemType.artifact` branch from `ArtifactsDataReducer.cs:76–99`** along with the artifact system overall.

### Pros
- Zero new client behaviour. `RewardPopup.cs` and `CurrencyView` already render this exactly.
- Players see a clean "BP = X gems / season" expectation. Easy to communicate, easy to monetize: the premium upgrade price becomes a direct gem-for-gem trade where the premium-track payout has to clear the unlock fee for the BP to feel "worth it".
- Lowest risk path. No new flux action, no new data type, no UI work.

### Cons
- Variety disappears. Today a tier opens and shows a coloured artifact icon; tomorrow every tier is the same gem icon at a different amount. This may hurt perceived value, especially mid-track.
- Concentrates the gem faucet. With the gear-crafting sink (`RMConfig.Pro.cs:166–175`) and gear-upgrade sink (`RMConfig.Pro.cs:177–182`) being deleted, the gem economy needs a new sink before the BP faucet opens or gems will inflate immediately.

### Risk
Low.

### Suggested alternate non-gem items to mix in if desired
Sprinkle one of these onto every 5th tier to break the visual monotony — all already supported:

- **`bonus_spin`** tokens (1–5 per milestone) — pulls players back into the daily wheel.
- **`lottery`** tickets — only if the lottery feature is being retained.
- **Stickers** (shop cosmetic, baseline 10 gems in `Shop/Models.cs`) — would require sending stickers through the BP reward pipeline; `RewardItem` currently has no sticker `ItemType`, so this requires a new `ItemType` (engineering work in `Models.cs:86–92`, `RewardPopup.cs`, and a sticker reducer).

---

## 3. Approach B — "Free-Light / Premium-Heavy" (asymmetric tracks)

### Concept
Keep the gem ramp idea from Approach A but break the symmetry between tracks. The free track pays a small gem stream plus heavy `bonus_spin` and `lottery` rewards; the premium track pays a much larger gem stream and reserves the "big bundles" for premium-only tiers. The free→premium gap stays meaningful even though both tracks pay gems.

### Distribution shape
- **Free track**: per-tier rewards alternate between 5–20 gems and 1–3 bonus spins / 1 lottery ticket. Cumulative free-track gems across the season target a tight figure (something like "10–15% of the premium-track total") so premium feels distinctly rewarding.
- **Premium track**: per-tier gem rewards 3–10× the same-index free reward, plus 2–3 "milestone bundles" at fixed indexes (e.g. tier 25, tier 50, tier 75) that drop a large gem bundle + a bonus-spin or lottery bonus.
- **Premium-only "vanity" tiers** (3–4 of them) can be reserved for unique items — stickers are the only such item currently in the codebase (`Shop/Models.cs`).

Like Approach A, the actual values live server-side in `BattlePassData.items[].milestones[].rewards`.

### Pros
- Premium track has a clear, sellable advantage at every milestone, not just on track unlock.
- Free players still get a regular feel-good (bonus spins keep the wheel alive after gear removal pulls value out of it).
- No new client `ItemType` or reducer wiring required — same rendering pipeline as Approach A.

### Cons
- Asymmetry needs to be tuned carefully. If the free track gem stream is too thin, players grow resentful (Reddit-grade risk). If it's too thick, premium is hard to sell.
- More server data to maintain. The matchmaker config grows two distinct ramps instead of one.
- Free players who don't engage with the daily wheel get a worse share of the BP value than they do today.

### Risk
Low-medium. Server data only, but more dials to turn.

### Suggested alternate non-gem items to mix in
- **`bonus_spin`** — heavy emphasis on the free track, especially in the early/mid tiers.
- **`lottery`** — only if the feature stays.
- **Stickers / cosmetics** — premium-only, reserved for the 3–4 vanity tiers above. Adds a sticker `ItemType` is a real (but small) feature build.
- **`slp`** — already supported (`Currency.slp` in `Common.cs`, and `Artifact.cs:22` uses an `slp` cost of 100 as a baseline). Use sparingly at premium late-tier bundles. Web3 settlement implications apply.

---

## 4. Approach C — "Quest-Currency Conversion" (BP pays a soft currency, not gems directly)

### Concept
Stop putting gems directly in BP tiers. Instead, every milestone awards a new **soft tracker token** (call it Coliseum Pass Mark / Season Mark — pick one) that the player can spend in a small in-BP shop on the artifact-removal-era reward palette: gem bundles, bonus spins, lottery tickets, stickers. The mark itself has no other use and decays at season end.

### Distribution shape
- **Per tier**: 1–3 Marks (a flat amount, or a small ramp).
- **In-BP Shop** (new view alongside `BattlePassHubView`): converts X Marks → Y gems, or X Marks → 1 bonus spin, etc., at fixed published rates.
- **Premium track** could either pay 2–3× as many Marks as free, OR add a Premium Shop with exclusive conversion items (e.g. sticker exclusives, larger gem bundles, axie cosmetics if those ever ship).

The Mark is a new `Currency` enum entry in `Common.cs:86–96` and a new `CurrencyData` slice; the shop is a new view.

### Pros
- Decouples reward pacing from gem-economy tuning. The design team can change the Mark→gem rate any season without re-issuing BP tier configs.
- Brings back the "I'm collecting toward something" feel that artifacts used to deliver. Players choose what their grind converts into.
- Easiest seasonal rotation — swap the Mark shop's catalog each season without touching tier configs.

### Cons
- Most engineering of the three: new currency, new reducer (or extension of `CurrencyData`), new shop view, new conversion endpoint.
- Adds a layer of indirection ("why didn't I just get gems?") that some players resent. Communication / tutorial overhead.
- The Mark itself needs an expiry policy at season end; nothing in the current codebase has a precedent for currency expiry that I could find.

### Risk
Medium-high. Net-new feature surface area.

### Suggested alternate non-gem items to mix in
The Mark shop is the variety vehicle, so it can carry whatever the team wants without further BP changes:

- Gem bundles of various sizes
- `bonus_spin` packs
- `lottery` tickets
- Stickers (would need a sticker `ItemType` to deliver them)
- Rotating "weekly featured" item — engineering hook already exists in shop-style views

---

## 5. Codebase-supported reward palette (excluding the items being removed)

These are the building blocks all three approaches can mix at no UI cost:

| Reward | In-code handle | Status |
|---|---|---|
| Gems | `Currency.gem` | Works through `CurrencyView`, baseline of all three approaches. |
| Gold | `Currency.gold` | Works. Lower-rank filler. |
| **Dust** | `Currency.dust` | **Only sink is gear crafting, which is being deleted.** Do not award until a new sink ships. |
| SLP | `Currency.slp` | Works. Web3 settlement implications apply. |
| AXS | `Currency.axs` | Works. Web3. |
| Bonus spins | `ItemType.bonus_spin` | Works; pulls into the daily wheel. |
| Lottery tickets | `ItemType.lottery` | Works only if lottery feature kept. |
| Stickers / cosmetics | Shop item, no current BP `ItemType` | **Requires a new `ItemType`** plus reducer wiring — small feature build. |

Reward concepts a designer might suggest that **do not exist in code** today and would be new features, not config changes:

- Axie eggs
- Profile frames, emotes, avatar borders
- Charms, moonshards, moonstones, runes (note: `home-economy` uses "gear" not "runes" — "rune" is a UI-tooltip-only term)
- Card backs, banner art, name colours
- Time-limited buffs / energy refills

---

## 6. Code surface — what changes regardless of approach

Touchpoints for any of the three:

1. **Backend matchmaker / season service** (out of this tree, but reachable via `HttpConstants.MatchMakerPrefix`) — owns the milestone reward arrays returned by `GET /season/user-battle-passes`. This is where artifact entries get deleted permanently.
2. `ArtifactsDataReducer.cs` — the `BattlePass_Claim` handler at lines 76–99 should be removed when artifacts are deleted system-wide.
3. `GearsDataReducer.cs` — no BP-claim case today; verify there is no in-flight work to wire gear into BP that needs cancelling. Then delete the file alongside gear removal.
4. `BattlePass/Models.cs:86–92` — remove `artifact` and `gear` from `ItemType` when those systems are gone.
5. `BattlePassHubView.cs`, `MilestoneView.cs`, `MilestonePair.cs` — no functional changes needed for Approaches A and B; minor display tweaks (icon spacing, label sizing) may be wanted now that the variety mix shifts.
6. `RewardPopup.cs` — already supports the kept `ItemType`s. No changes for A/B; for C, add the new Mark currency to the currency-render path (which auto-handles new `Currency.*` values).
7. `Common.cs:86–96` — only for Approach C, to register the new Mark currency.
8. `RMConfig.Pro.cs:342–346` — the client `BattlePassConfig`; only touched if a new client-side toggle is needed (e.g. for the Mark shop in Approach C).
9. Documentation: `.codemap/home-progression.yaml` — update with the new reward shape.

---

## 7. Recommendation

If shipping the artifact and gear removal as a single content release with as little new code as possible, **Approach A** is the right baseline.

If preserving "premium feels distinctly worth it" is the top priority — especially because the premium track is the BP's monetization handle — **Approach B**.

If the design team wants room to rotate seasonal rewards aggressively without re-touching tier configs, **Approach C** is the highest-ceiling choice but pays for it with real feature build.

For all three: do not pay `dust` as a BP reward until a non-gear sink for dust exists; otherwise dust becomes inventory clutter the moment gear is removed.

---

## 8. Open questions for the design team

1. Is the premium track unlock price (`premium.cost` returned by the matchmaker) going to stay denominated in gems? If yes, this caps how many gems can be paid out before the BP becomes net-negative for the player.
2. What is the desired total BP-score earnable per season for an engaged player? The client doesn't reveal this — it's a server config. The proposals above don't depend on the number, but tier reward sizing does.
3. Should the lottery feature survive the artifact removal? It shares the seasonal-event design space with BP; deciding now avoids designing tiers around `ItemType.lottery` that may be removed shortly after.
4. Will the daily wheel survive in some form after artifacts are deleted? If yes, `bonus_spin` rewards remain valuable. If no, they're worthless and should be removed from BP entirely.
5. Are stickers / cosmetics on the roadmap as a real reward channel? If yes, paying engineering cost on a new sticker `ItemType` now is worth it; if no, BP rewards stay currency-only.
