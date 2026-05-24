# Report 1 — Cursed Coliseum Chest Rewards

**Date:** 2026-05-23
**Scope:** Three alternate reward-distribution approaches that replace artifacts (and any gear) inside Cursed Coliseum chests, using in-game gems as the primary substitute. Includes alternate non-gem reward ideas and the code surface required to ship each approach.

---

## 1. Baseline — how the Cursed Coliseum pays out today

These are the facts the proposals are built on. Every claim is sourced from the codebase.

### 1.1 What Cursed Coliseum chests actually contain right now

The chest reward table is hard-coded in the client remote-config file `RMConfig.Pro.cs`. Two `ChestReward` entries exist for `ChestType.coloseum_chest` / `ChestId.coloseum_chest`, and **both pay out a single artifact** with amount = 1:

- One entry: `ItemType.artifact, id = "bug", amount 1–1`
- One entry: `ItemType.artifact, id = "bonus-spin", amount 1–1`

There are **no gear entries, no currency entries, and no other item types in the current Cursed Coliseum chest table**. Other chest tiers (wood / copper / silver / gold / mystic) only pay `Currency.gold` in fixed quantities; they are unrelated to Cursed Coliseum.

### 1.2 How chests are gated

The session and chest economy is also pinned in `RMConfig.Pro.cs` under `ColiseumConfig`:

- `maxSession = 2` sessions per day
- `maxChest = 2` chests held at once
- An internal `rewards` list (15 integer slots scaling from 15 up to 10 000) drives the per-win reward ramp inside a session — see the same `ColiseumConfig` class.

`ColiseumData` in `ColiseumData.cs` describes the session state: `maxWin = 7`, `maxLose = 2`, `mode` is `normal` or `premium`, `sessionStatus` is `empty | selecting-parts | ready | finished-success | finished-fail | expired`. The view layer (`ColiseumView.cs`, around line 101) caps chest creation against `RMConfig.inst.coliseum.maxChest` and shows a "full chest" message when reached. There is also a partially built collectibles track (`CCCollectibleListView.cs`, `ColiseumExtraRewardsStateReducer.cs`) — its code is commented out today but the scaffolding is there if we want to revive a milestone track later.

### 1.3 Reward types the engine already knows how to deliver

`BattlePass/Models.cs` defines `ItemType` with five constants — `currency`, `artifact`, `bonus_spin`, `lottery`, `gear`. The reward popup (`RewardPopup.cs`) routes `currency` items through `CurrencyView`, artifacts through `ArtifactView`, and everything else through `RewardItemView`. **This means a new chest payout that uses only `currency`, `bonus_spin`, and `lottery` already has a working display pipeline; no UI work is required for any of the three proposals below.**

The full currency palette is defined in `Common.cs`:

- Web2 (off-chain): `gem`, `gold`, `dust`
- Web3 (on-chain): `slp` (+ `slp_in`, `slp_ex`), `maxs` (+ `maxs_in`, `maxs_ex`), `axs`, `ron`, `weth`, `usdc`

### 1.4 Today's role of `gem`

`gem` is defined in `Common.cs` line 93 as a Web2 soft currency. Its current sinks are concentrated in `RMConfig.Pro.cs`:

- Gear **crafting** costs 20–200 gems per craft (lines 167–175)
- Gear **upgrading** costs a flat 20 gems per attempt (lines 177–182)
- Sticker / cosmetic baseline price is 10 gems (Shop models)

No code path in the searched scope **awards** gems through Coliseum, Wheel, Quest, or Battle Pass today. Once gear is removed, the gem sink shrinks dramatically; new sinks (or a tighter faucet) will be needed to keep the currency meaningful. This shapes how aggressively the proposals below seed gems.

### 1.5 Where the source of truth lives

Loot rolls themselves are not visible inside the Unity client search — the client stores the result via `ChestStateReducer.cs` and `ArtifactsDataReducer.cs`. The actual roll most likely lives in the matchmaker / season backend (the `cp-state-*` repos, behind endpoints under `HttpConstants.MatchMakerPrefix`). For all three proposals, the **content** of a chest is decided server-side; the client only renders what arrives in the `RewardResponse` / `ChestData.rewards` payload. The client config in `RMConfig.Pro.cs` is the published "expected drop" preview shown to players, not the rolling logic.

---

## 2. Approach A — "Flat Gem Chest" (simplest cutover)

### Concept
Replace both artifact slots in the chest with a single, fixed gem payout that scales linearly by the player's session-win count. No randomness inside the chest; the same number of wins always pays the same number of gems.

### Distribution shape
The session already runs from 0 wins up to `maxWin = 7` (with a 12-win "premium" extension visible in `ColiseumConfig.rewards`). The 15-slot ramp inside `ColiseumConfig.rewards` (15, 25, 120, 200, 300, 400, 500, 1000, 1500, 2000, 3000, 4500, 6000, 8000, 10 000) is already used to communicate value scaling to the player — re-use it. The two chests per day (`maxChest = 2`) lock against the per-session win count via `ColiseumView.cs`'s `chestCount >= maxChest` check.

Concrete chest contents (replace the two `ChestReward` entries in `RMConfig.Pro.cs` lines 268–277):

- `chestType = coloseum_chest, chestId = coloseum_chest, rewards = [{ itemType = currency, id = "gem", amountFrom = X, amountTo = X }]`

with X chosen against a target weekly gem economy that the design team picks. Because the existing `ColiseumConfig.rewards` ramp is published in code with a clean 15→10 000 progression, scaling X by win count keeps the existing in-game preview UI honest.

### Pros
- One server change, one config change, ships fast.
- No new ItemType, no new reducer wiring (the existing `currency` path in `RewardPopup.cs` and `CurrencyView` already renders it).
- Removes the entire "what artifact did I open" surprise mechanic cleanly — important because the artifact concept is being deleted system-wide.

### Cons
- Loses the "I got X!" pull of variable loot. The chest becomes a billed transaction, not a moment of discovery.
- All the gem-economy pressure of the new system lands on a single faucet. If gem-per-chest is too high, the gear-upgrade sink (the one remaining gem sink today, per `RMConfig.Pro.cs` lines 177–182) is being deleted alongside gear — so gems will inflate fast unless a new sink ships at the same time.

### Risk
Low. Code-mechanical change inside one config block.

### Suggested alternate non-gem items to mix in if desired
- **`bonus_spin`** — already a defined `ItemType`. Handed off cleanly through the `RewardPopup`. Adds a downstream pull into the daily wheel (`WheelView.cs`, `SpinWheel` action) without adding any new reward primitives.
- **`gold`** — already in the chest economy on the non-Coliseum side; gives a second currency the player feels is "the easy one".
- **`lottery`** — supported `ItemType`; useful if there is an active lottery feature (referenced in `MilestoneView.cs:37–40` via `lotteryView`).

---

## 3. Approach B — "Tiered Chest Quality" (replaces artifact rarity with chest rarity)

### Concept
Keep loot variety by introducing **chest tiers within Cursed Coliseum** — cursed-bronze, cursed-silver, cursed-gold — gated by the in-session win count and/or the "normal" vs "premium" mode flag (`ColiseumData.mode` already supports `"normal"` and `"premium"`). Each tier rolls from a small drop table heavily weighted toward gems but spiced with `bonus_spin` and `lottery` tickets.

### Distribution shape
Use the existing `ChestId` enum to add new IDs (`ChestId.coloseum_chest_bronze`, `..._silver`, `..._gold`) alongside the current `ChestId.coloseum_chest`. Each new `ChestId` gets one `ChestReward` block in `RMConfig.Pro.cs`. Tier assignment happens server-side at chest creation based on win count:

- 0–2 wins → bronze (small flat gem amount + small bonus-spin chance)
- 3–5 wins → silver (medium gem amount + guaranteed bonus-spin OR lottery)
- 6–7 wins (and 12-3 extension) → gold (large gem amount + bonus-spin + lottery)

Within a tier, the `MaybeReward` shape (`BattlePass/Models.cs`) already supports `amountFrom`/`amountTo` ranges, which gives the chest the "loot roll" feel for free.

### Pros
- Preserves the dopamine of "rare chest dropped!" while replacing the rarity vehicle (artifact id → chest id). Cleaner long-term: the chest is the meta object, not the artifact.
- Maps naturally onto `ColiseumData.mode == "premium"` for any "premium chest" upsell down the line — no new flags needed.
- Uses only the existing `RewardPopup` rendering path.

### Cons
- More server logic: chest tier must be assigned at creation time (`ColiseumStateReducer.cs` already handles the response shape; the backend needs the assignment rule).
- Multiple `ChestId` values means anywhere that case-matches on chest id (asset look-ups in `ChestListData.cs`, sprite mapping in the inventory view) will need new entries.
- Players who farm only minimum-win sessions get visibly worse chests — fine for a competitive arena, but it's a behavior change worth playtesting.

### Risk
Medium. Touches the chest creation pipeline server-side, plus three or four client files.

### Suggested alternate non-gem items to mix in
- **`bonus_spin` tokens** — clearest pull-through to the daily wheel and the existing artifact-replacement narrative.
- **`gold`** — cheaper currency for unlocking the gold-chest set (`ChestId.wood_chest…gold_chest` family), keeping the rest of the chest economy alive.
- **A premium currency drop in the gold tier** — `slp` or `axs` is already in `Common.cs`. SLP at 100 per chest mirrors the existing artifact `rechargeFee = Currency("slp", 100)` baseline from `Artifact.cs` line 22, so the number doesn't feel pulled from nowhere. (Web3 settlement implications obviously apply.)

---

## 4. Approach C — "Mission Chest + Currency Bank" (separates content from payout)

### Concept
Stop using a chest as the loot container at all. Drop the artifact slots from `RMConfig.Pro.cs` and instead award **gems directly into `CurrencyData`** when the chest is "opened", plus drop **tracker tokens** into a re-activated collectibles / milestone track. The collectibles infrastructure already exists but is commented out in `CCCollectibleListView.cs` and `ColiseumExtraRewardsStateReducer.cs` — switching this back on lets each chest fill a milestone that grants gems / `bonus_spin` / `lottery` on tier completion.

### Distribution shape
- Per-chest payout: a flat 100–500 gem grant (immediate, no "open" animation).
- Per-chest also adds 1 token to a **Cursed Coliseum collectibles track** (the same data shape the dormant `CCCollectibleListView` was built around).
- Track tiers (e.g. every 10 tokens, 30 tokens, 60 tokens, season-end) deliver larger gem bundles, plus `bonus_spin` and `lottery` rewards using existing `ItemType`s.

### Pros
- The variable-reward dopamine moves to **milestone unlocks**, where it's easier to balance because the unlock cadence is deterministic.
- Resurrects already-written code (`CCCollectibleListView.cs`, `ColiseumExtraRewardsStateReducer.cs`). The previous team built the skeleton — finishing it costs less than building tiered chests from scratch.
- Long-term: a track is a much better vehicle for seasonal rotation than a chest table. Each season can swap the milestone bundle without changing chest config.

### Cons
- Most upfront work of the three. Requires re-enabling and finishing the collectibles UI, building or wiring the backend endpoint that the reducer was originally designed for, and writing the milestone reward config.
- Loses the "open the chest" moment unless we keep a thin animation on the immediate gem grant — design call.
- More moving parts means more places to mis-tune the per-day faucet.

### Risk
Medium-high — the dormant collectibles code has not been audited for whether it ever shipped or what state it expected. A code archeology pass is needed before committing.

### Suggested alternate non-gem items to mix in
- **`bonus_spin`** — natural fit for big milestone unlocks; daily wheel becomes the variety vehicle.
- **`lottery` tickets** — reserve for end-of-track grand-prize milestones.
- **Cosmetics (stickers)** — already exist in the shop (`Shop/Models.cs`, baseline 10 gems). A "Cursed Coliseum sticker" is a low-engineering, high-perceived-value milestone reward.

---

## 5. Alternate non-gem reward ideas (codebase-supported palette)

Anything below is callable today through existing `ItemType` / `Currency` handling — no new client wiring beyond the appropriate sprite or string-table entry. These are the building blocks for any of the three approaches.

| Reward idea | In-code handle | Where it already works | Notes |
|---|---|---|---|
| Daily-wheel free spins | `ItemType.bonus_spin` | `RewardPopup.cs:56–58`, `ArtifactsDataReducer.cs` (`SpinWheel`) | Strong pull-through to an existing system; no new sinks needed. |
| Lottery tickets | `ItemType.lottery` | `MilestoneView.cs:37–40` | Only useful if the lottery feature is being kept. |
| Gold | `Currency.gold` | All other chest tiers | Easy low-tier filler. |
| SLP | `Currency.slp` | Artifact recharge (`Artifact.cs:22`) | Web3 settlement implications; only viable if the SLP faucet design allows it. |
| AXS | `Currency.axs` | Already in `Common.cs` | Premium-tier only; meaningful Web3 payout. |
| Stickers / cosmetics | Shop item via `gem` | `Shop/Models.cs` | A reward item, not a stack of currency — good for milestone uniqueness. |
| **Dust** (with caution) | `Currency.dust` | Only existing sink is gear crafting (`RMConfig.Pro.cs:166–175`) | Once gear is removed, dust is orphaned. **Do not award dust unless a new sink ships first** — otherwise it becomes worthless overnight. |

Two reward concepts the user might ask about that **do not** exist in the codebase and would require new `ItemType` / `Currency` definitions (touching `Common.cs`, `BattlePass/Models.cs`, every reducer that switches on `ItemType`):

- Axie eggs (not present)
- Profile frames / emotes / avatar borders (not present)
- "Charms" (not present — search returned nothing)
- "Moonshards" / "Moonstones" (not present)

Adding any of these is a real feature build, not a config change — flag this when scoping.

---

## 6. Code surface — what changes regardless of approach

For all three approaches, the following files are the minimum touchpoints; this is the list a designer would hand to engineering.

1. `RMConfig.Pro.cs` — the chest reward table at lines 268–277, plus `ColiseumConfig` if the daily/session caps change.
2. `Common.cs` — only if a new currency or reward primitive is introduced (Approach C if we add a tracker token type).
3. `ChestListData.cs` / `ChestStateReducer.cs` — only if `ChestId` enum or chest state shape changes (Approach B).
4. `ColiseumStateReducer.cs` — for any change to how chest creation is gated.
5. `ColiseumView.cs` — the "preview" of chest contents in the arena UI.
6. `ColiseumChestInfoView.cs` — the chest-info popup; verify it renders the new reward types.
7. `RewardPopup.cs` — already supports all five `ItemType`s; only touched if a new `ItemType` is introduced.
8. The backend matchmaker service (out-of-tree but inferred from `HttpConstants.MatchMakerPrefix`) — owns the actual reward roll and is the only place where the artifact entries can be deleted with finality.
9. `CCCollectibleListView.cs` / `ColiseumExtraRewardsStateReducer.cs` — only for Approach C; both are currently commented out and need a finishing pass.
10. Documentation: `.codemap/home-competitive.yaml` — update after the change.

---

## 7. Recommendation

If the goal is to ship the artifact removal **with the least risk and fastest cutover**, run Approach A.

If the goal is to **preserve the rarity feel** of Cursed Coliseum after artifacts are gone, run Approach B.

If the goal is to **set up a seasonal Coliseum loot loop** that survives multiple content cycles, run Approach C — but only after auditing the dormant collectibles code.

In all three cases, do not award `dust` from Coliseum chests until a non-gear gem-substitute sink for dust is designed; today the sink is being removed and dust would become dead inventory.

---

## 8. Open questions for the design team

1. Does removing artifacts and gear simultaneously kill the `dust` sink entirely? If so, decide whether `dust` is repurposed or deprecated before designing any chest table that could ever include it.
2. The dormant `CCCollectibleListView` / `ColiseumExtraRewardsStateReducer` code — was it shipped previously, or rolled back? An eng owner should answer before Approach C is greenlit.
3. Is the `"premium"` Coliseum session mode (the 12-3 extension implied by `ColiseumConfig.rewards`) still meant to ship? If yes, premium-tier chests are a natural fit for Approach B.
4. Should `bonus_spin` rewards be deemphasised in chests if the daily wheel itself is being redesigned for the artifact removal? Today the wheel can drop artifacts (`ArtifactsDataReducer.cs` lines 102–117); that pipeline is being deleted too.
