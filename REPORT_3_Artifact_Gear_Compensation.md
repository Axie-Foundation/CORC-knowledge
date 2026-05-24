# Report 3 — Compensating Players for Artifact & Gear Grind

**Date:** 2026-05-23
**Scope:** Five tiered compensation approaches for players who have invested time and resources into artifacts and gear before those systems are removed. Heavier investment → heavier payout. Every approach is anchored to data the server already stores or can compute from existing endpoints.

---

## 1. What players have actually been grinding (so we know what to refund)

### 1.1 Artifacts — sources, costs, sinks

Artifact data lives in `Home/Inventory/Artifact.cs`. Each `Artifact` instance carries:

- `id` — one of six types defined in the Rust engine `lunacia/src/artifact.rs:60–67` (`Plant`, `Aquatic`, `Beast`, `Reptile`, `Bug`, `Bird`), plus IDs seen in chest configs such as `"bug"` and `"bonus-spin"` (`RMConfig.Pro.cs:268–277`)
- `level` — upgrade tier; in-battle duration is `2^(level - 1)` turns (`Artifact.cs:53`)
- `amount` — count owned
- `lastUpgradeAmount` — total copies consumed reaching the current level (this is the most direct measure of grinding effort)
- `upgradeAmount` — copies still needed to reach the next level
- `upgradeFee` — `Currency` cost paid each time the player upgraded
- `rechargeable`, `rechargeFee` — daily-use cap (`todayUseCap = 10`, line 17) and a recharge price hard-coded at `Currency("slp", 100)` (line 22)

How a player obtained artifacts (every source observed):

- Battle Pass milestone claim — `ArtifactsDataReducer.cs:76–99`
- Daily Wheel spin reward — `ArtifactsDataReducer.cs:102–117`
- Quest claim — `ArtifactsDataReducer.cs:131–141`
- Generic `RewardResponse` (coliseum chest, ad-hoc events) — `ArtifactsDataReducer.cs:120–129`
- Cursed Coliseum chest — `RMConfig.Pro.cs:268–277`

How a player spent on artifacts:

- Upgrade — `POST {MatchMakerPrefix}/user-artifacts/upgrade` (`Actions.Artifact.cs:17–28`); cost is `upgradeFee`, server-defined.
- Recharge — `POST {MatchMakerPrefix}/user-artifacts/recharge` (`Actions.Artifact.cs:30–40`); cost is 100 SLP per recharge.

The current inventory is the GET endpoint `/user-artifacts` (`Actions.Artifact.cs:7–15`). Every field above is on the response. **Past upgrade and recharge transactions are not preserved in the client state** — only the current snapshot is. So compensation must either accept the snapshot as the truth (cheap, slightly unfair to players who spent and then lost what they bought through some other channel) or rely on backend audit/transaction logs (more accurate, more work).

### 1.2 Gear — sources, costs, sinks

Gear data lives in `Home/Gear/Models.cs`. Each `Gear` instance carries:

- `id` — unique instance id
- `type` — one of `hp`, `speed`, `skill`, `morale` (lines 21–24)
- `level` — upgrade tier, target max 10 per `GearUpgradeView.cs:143`
- `bonus` — "bonusUpgradeLevel"; 0–20, boosts upgrade success rate per `GearUpgradeView.cs:117–128`

Gear has **only one source: crafting**. Every piece of gear in a player's inventory was paid for. The craft cost table is hard-coded in `RMConfig.Pro.cs:166–175`:

| Type / Level | Dust | Gem |
|---|---|---|
| HP level 1 | 500 | 20 |
| HP level 2 | 1 200 | 42 |
| HP level 3 | 2 800 | 88 |
| HP level 4 | 8 000 | 200 |
| Speed level 1 | 750 | 20 |
| Speed level 2 | 1 800 | 42 |
| Speed level 3 | 4 200 | 88 |
| Speed level 4 | 12 000 | 200 |

Upgrade is a fusion: two same-type-same-level gears + 20 gems per attempt, succeeding with a probability boosted by the `bonus` field (`RMConfig.Pro.cs:177–182` and `GearUpgradeView.cs:117–128`). On success the two source gears are consumed and a new level+1 gear with bonus = 0 is created (`GearsDataReducer.cs:49–60`).

The inventory endpoint is `GET /user-gears?offset=X&limit=Y` (`Actions.Gear.cs:7–15`), returning `GearsData.items[]` with `type`, `level`, `bonus`, `id`.

### 1.3 What the snapshot can prove (no audit log)

From `/user-artifacts` and `/user-gears` alone, we can directly read:

- Every artifact a player currently holds, its level, and `lastUpgradeAmount` (which is the cumulative copies consumed to reach this level)
- Every gear piece a player currently holds, its type, level, and bonus

From these we can **derive** (no transaction log needed):

- For each gear piece at level L: at minimum the player crafted 2^(L−1) level-1 pieces and paid 2^(L−1) × craft-cost-L1 in dust+gems. Plus 20 gems × (number of upgrade attempts), of which we can only count the **successful** attempts (= L − 1 fusions on the surviving path).
- For each artifact at level L: the player accumulated `lastUpgradeAmount` × {ever-increasing cost path} copies and paid the corresponding `upgradeFee`s.

What the snapshot **cannot** prove without audit logs:

- Failed gear upgrade attempts (each costs 20 gems and burns 2 gears).
- Recharge spending on artifacts that were later expended or replaced.
- Resources spent on items the player no longer holds.

This gap is the central design choice of this report: **how much "unobservable" past investment do we approximate**, and how do we keep it fair?

### 1.4 No existing refund / dismantle / salvage pattern in code

Grep across the Unity client for `refund`, `dismantle`, `salvage`, `recycle`, `compensation`, `convert` returned no existing pipeline. This is a first-time event, with no precedent code path to reuse. Every approach below adds a one-shot ingestion script on the backend plus a one-shot reward delivery to the client (using the existing `RewardResponse` shape that `ArtifactsDataReducer.cs:120–129` already consumes).

---

## 2. The "Investment Score" — single calculation used by all five approaches

To tier compensation fairly we need one number per player. Define:

```
ArtifactScore = Σ (per artifact owned)
                  rarity_weight(artifact.id)
                × level_weight(artifact.level)
                × max(1, artifact.amount)

GearScore     = Σ (per gear piece owned)
                  type_weight(gear.type)
                × craft_cost_dust(gear.type, gear.level)    // from RMConfig.Pro.cs:166–175
                × (1 + 0.05 × gear.bonus)                   // bonus reflects extra gem spent boosting success rate

InvestmentScore = ArtifactScore + GearScore × W_gear
```

`W_gear` is a single dial that lets the design team rebalance the artifact↔gear weight without rewriting tier thresholds.

`level_weight(level)` should not be linear because artifact and gear costs grow superlinearly with level:

- Artifact: in-battle duration scales as `2^(level - 1)` (`Artifact.cs:53`) → use `2^(level - 1)` as the level weight.
- Gear: per-level craft cost in `RMConfig.Pro.cs:166–175` roughly doubles per level for HP and more than doubles for Speed → use the actual craft cost as the weight directly (this is what's in the `GearScore` formula above).

`rarity_weight(artifact.id)` — there are no canonical rarities in code, but artifacts paid the SLP recharge cost equally; treat all six types as equal weight unless the design team wants to bias toward harder-to-obtain IDs.

All five approaches below sort players by `InvestmentScore` and assign each player to a tier. Only the **reward shape** differs.

---

## 3. Approach 1 — Flat Gem Refund (5 tiers, percentile-based)

### Concept
Compute `InvestmentScore` for every player. Sort. Tier into 5 buckets by percentile. Each tier receives a fixed gem grant.

### Tier shape
| Tier | Percentile band (of players who own ≥1 artifact or gear) | Gem grant (illustrative shape; absolute values are a design call) |
|---|---|---|
| 1 — Newcomer | Bottom 30% | Small (e.g. baseline B) |
| 2 — Collector | 30–60% | ~2.5 × B |
| 3 — Enthusiast | 60–85% | ~5 × B |
| 4 — Veteran | 85–97% | ~10 × B |
| 5 — Whale | Top 3% | ~25 × B |

Pick `B` against (a) the per-craft gem cost at level 1 (20 gems, `RMConfig.Pro.cs:167`) and (b) the average upgrade gem spend at level 2–3 to keep the bottom tier above "did I really get something?" and the top tier below "this breaks the gem economy".

### Pros
- One number to communicate per tier. Easiest player-side message.
- Backend work is minimal: a single batch script reading `/user-artifacts` and `/user-gears`, computing the score, and pushing one `RewardResponse` per player. The client renders it through the existing `CurrencyView` path.

### Cons
- Percentile-banding produces unfair cliffs. The player at the 84th percentile gets less than the player at the 86th percentile even if their actual investment is nearly identical.
- A whale who's been farming for 18 months and a mid-tier player who just hit Veteran two days ago get the same payout inside that band.
- Whales tend to be vocal — capping their reward at "Tier 5 fixed amount" risks Reddit-grade pushback if the cap feels arbitrary.

### Risk
Low. Pure batch job.

---

## 4. Approach 2 — Linear Refund (proportional gem grant, no tiers)

### Concept
Skip tiers entirely. Refund a percentage of every gem actually spent on craft + upgrade + recharge, calculable directly from the snapshot.

- For each gear piece: refund `R_craft × dust_cost_in_gem_equivalent(level) + R_upgrade × 20 × (level − 1)`. The "dust cost in gem equivalent" is a conversion rate the team picks (since dust will be orphaned post-removal anyway).
- For each artifact: refund `R_artifact_upgrade × Σ upgrade_fees_implied_by_lastUpgradeAmount + R_recharge × 100 SLP_in_gem_equivalent × (estimated recharge count)`.

`R_*` are refund coefficients between 0 and 1. A common shape: 80% for directly-observable spend (the gear/artifact still in inventory), 30–50% for recharge spend (which is harder to count without audit logs).

### Pros
- Fairest by raw "you spent X, here is X × Y". No cliffs.
- Scales naturally — a player with 10× the gear gets ~10× the refund.
- Communication: "we refunded 80% of what we can prove you spent" is a strong, defensible message.

### Cons
- Requires an explicit gem-equivalent rate for dust and SLP. Pinning those rates is a public statement the team will be held to.
- Without audit logs, the recharge term is a guess — players whose history was largely recharges (not upgrades) feel under-paid.
- The whales' refund is unbounded — could be very large for the top of the top.

### Risk
Low-medium. The dust→gem and SLP→gem conversion rates are a real decision with downstream economy implications.

---

## 5. Approach 3 — Tiered Bundle (5 tiers, multi-item per tier)

### Concept
Same 5-tier banding as Approach 1, but each tier delivers a **bundle** rather than a single gem amount. Higher tiers get more bundle slots (gems + bonus spins + lottery tickets + cosmetic).

### Tier shape (qualitative — design team picks magnitudes)

| Tier | Gems | Bonus spins (`ItemType.bonus_spin`) | Lottery tickets (`ItemType.lottery`) | Cosmetic / one-of-a-kind |
|---|---|---|---|---|
| 1 — Newcomer | Small bundle | 3 | 1 | — |
| 2 — Collector | Medium bundle | 6 | 3 | — |
| 3 — Enthusiast | Large bundle | 10 | 5 | One sticker (`Shop/Models.cs`) |
| 4 — Veteran | XL bundle | 20 | 10 | "Founding Hero" sticker set (3 stickers) |
| 5 — Whale | Top bundle | 50 | 25 | "Founding Hero" sticker set + unique commemorative title or frame **(new feature — see risk)** |

Bonus spins and lottery tickets are zero-engineering using existing `ItemType`s and the `RewardPopup` path. Stickers are already in the shop catalog. **Commemorative title / frame / banner is not in the codebase** and is a real feature build if pursued; treat it as optional.

### Pros
- More variety per tier — softens the "your tier is just a number of gems" feel from Approach 1.
- Lets us honor the most-invested players with something unique (the commemorative item) — strong retention signal.
- All low-tier items are codebase-supported with no new client wiring.

### Cons
- Communication is harder ("Tier 4 = 20 spins, 10 tickets, 3 stickers, X gems" doesn't fit on a card cleanly).
- Commemorative items are a feature build if pursued; cost should be scoped before promising them.
- Risk of "I would have rather had more gems" — give players an explicit choice if possible (see Approach 4 below).

### Risk
Low if commemoratives are dropped; medium if they're kept.

---

## 6. Approach 4 — Player-Choice Refund Tokens (5 tiers, choose-your-own bundle)

### Concept
Compute tier as in Approach 1, but instead of giving a fixed bundle, grant each player a number of "refund tokens" proportional to their `InvestmentScore`, plus a one-time **redemption shop** that converts tokens into gems / bonus spins / lottery tickets / stickers at published rates.

This is the same pattern as Battle-Pass Approach C in Report 2, but applied as a one-shot artifact-removal event rather than a seasonal ramp.

### Tier shape
- Tier 1 (Newcomer): N tokens
- Tier 2 (Collector): 2N tokens
- Tier 3 (Enthusiast): 5N tokens
- Tier 4 (Veteran): 10N tokens
- Tier 5 (Whale): 25N tokens

The shop offers, e.g., 1 token → 10 gems, 5 tokens → 1 bonus spin, 10 tokens → 1 lottery ticket, 50 tokens → 1 sticker. (Exact rates are a design call.) Tokens expire 30–60 days after the event so the inventory clears.

### Pros
- Maximizes perceived fairness — every player picks what they value.
- Whales who only care about gems get gems; collectors who want spins or stickers get those.
- A "loud minority" can't easily complain that "I would have preferred X" because everyone got X-and-Y as choices.

### Cons
- The most engineering of any approach: a new `Currency` entry in `Common.cs:86–96`, a new redemption shop view, a new endpoint to convert tokens, an expiry policy. None of these exist in the searched code.
- Players have to actively redeem — those who don't notice / don't engage walk away with nothing usable, which is a worse player experience than auto-granting gems.
- The redemption shop becomes a balance-tuning surface forever ("why is the gem rate 1:10 and not 1:12?").

### Risk
Medium-high. Real feature build for a one-shot event.

---

## 7. Approach 5 — Hybrid: Auto-Grant + Optional Top-Up via Choice (5 tiers, best of both)

### Concept
Every player is auto-granted a baseline gem refund based on `InvestmentScore` (Approach 1 or 2 mechanism). Additionally, players in Tier 3 and above are granted refund tokens that can be spent in the redemption shop for cosmetics, bonus spins, and lottery tickets that **gem cannot otherwise buy** (e.g., commemorative stickers, "Founding Hero" sticker set).

This is the recommended shape because it strictly dominates the others on player experience while limiting the engineering blast radius.

### Tier shape

| Tier | Auto-gem grant | Choice tokens |
|---|---|---|
| 1 — Newcomer | Small | 0 |
| 2 — Collector | Medium | 0 |
| 3 — Enthusiast | Large | Few (e.g. 1 sticker's worth) |
| 4 — Veteran | XL | Several (e.g. 1 sticker set + spins) |
| 5 — Whale | Top | Many + 1 commemorative item |

The auto-grant uses the existing `currency` payout shape (zero new client wiring); the choice tokens use a redemption shop scoped only to non-gem items (so the shop catalog is small, fixed, and lifetime-limited — no new tuning surface forever).

### Pros
- Every player gets a usable thing immediately (gems) — no "I have tokens but never redeemed them" footgun.
- High-investment players still get the variety, commemorative weight, and choice.
- Engineering cost is bounded — the redemption shop only exists for ~2 months and only ships items already supported by the codebase (`ItemType.bonus_spin`, `ItemType.lottery`, stickers).
- The "loudest" whale gets a commemorative AND a big gem grant — silences the "you didn't honor my investment" complaint.

### Cons
- Slightly more communication: "you got X gems + Y tokens to spend in the shop until [date]".
- Token shop expiry must be communicated clearly (no precedent for this in the codebase).
- Two code paths (auto-grant + shop) is more than one, but each is small.

### Risk
Medium. The bounded scope is the win.

---

## 8. Cross-approach mechanics — calculating each player's tier

### 8.1 Concrete formula the backend should run once

For every player with at least one artifact or one gear piece:

```
ArtifactScore  = Σ artifacts:    2^(level - 1) × max(1, amount)
GearScore_raw  = Σ gear pieces:  craft_cost_dust(type, level) × (1 + 0.05 × bonus)
InvestmentScore = ArtifactScore + W_gear × GearScore_raw
```

where `craft_cost_dust(type, level)` is read from the `RMConfig.Pro.cs:166–175` table.

Tier banding by percentile, after the score is computed for everyone:

| Tier | Band |
|---|---|
| 1 | Bottom 30% |
| 2 | 30–60% |
| 3 | 60–85% |
| 4 | 85–97% |
| 5 | Top 3% |

Or, if absolute thresholds are preferred to percentile bands, derive them from the percentile distribution on the existing player set and pin them so they survive subsequent gem-economy changes.

### 8.2 Delivery — uses an existing client path

The granted rewards arrive in a `RewardResponse` payload that `ArtifactsDataReducer.cs:120–129` and the generic `CurrencyData` reducer already consume. The client requires **no new code** to render a one-shot batch grant — only the backend script and the player-facing message.

### 8.3 What's NOT in the formula and why

- **Failed gear upgrade attempts.** Each failure costs 20 gems and burns 2 gears (`GearUpgradeView.cs` + `RMConfig.Pro.cs:177–182`). Counting failures requires the backend transaction log — assume it exists at the matchmaker layer, but it's not in the client. If transaction logs are available and easy to query, add a "failed-upgrade penalty refund" term: `R_failed × 20 gems × failure_count`. If not, the snapshot formula underpays grinders who failed a lot; consider bumping `W_gear` 10–20% to compensate.
- **Recharges.** `Artifact.cs:22` shows recharges at 100 SLP each, capped at one per day per artifact when `todayUsed >= todayUseCap` (10 uses/day). The snapshot doesn't say how many times a player recharged historically. If the backend logs it, add `R_recharge × 100 SLP_in_gem × total_recharges`; otherwise omit.
- **Past inventory.** A player who bought an artifact, used it up, and now owns zero copies gets nothing. This is unavoidable without a backend audit log.

---

## 9. Codebase-supported reward items (no client wiring required)

| Item | In-code handle | Notes |
|---|---|---|
| Gems | `Currency.gem` (`Common.cs:93`) | Primary refund vehicle. |
| Gold | `Currency.gold` | Filler. |
| Bonus spins | `ItemType.bonus_spin` (`BattlePass/Models.cs:88`) | Renders through existing reward popup. |
| Lottery tickets | `ItemType.lottery` (`BattlePass/Models.cs:89`) | Only if lottery feature kept. |
| Stickers | Shop item (`Shop/Models.cs`, base price 10 gems) | Commemorative use case. |

Items a designer may want that **do not exist in code** today (so they need real feature work):

- Profile frames, banners, name tags, emotes
- Unique commemorative titles
- Axie cosmetic skins
- Time-limited stat boosts

**Dust** is in the codebase (`Currency.dust`) but its only sink is gear crafting (`RMConfig.Pro.cs:166–175`) — which is being removed. Do not refund into dust unless a new sink ships at the same time; otherwise dust becomes worthless inventory.

---

## 10. Code surface — what every approach touches

1. **Backend matchmaker / state service** (out of this repo tree; reachable via `HttpConstants.MatchMakerPrefix`) — owns the batch script that reads `/user-artifacts` and `/user-gears`, computes `InvestmentScore`, and writes the `RewardResponse`. This is where 90% of the work lives.
2. `ArtifactsDataReducer.cs:120–129` — already handles generic `RewardResponse`s, no change needed.
3. `RewardPopup.cs` — renders the grant; already supports every `ItemType` used above.
4. `Common.cs:86–96` — only touched if Approach 4 or 5 adds a refund-token currency.
5. `Shop/Models.cs` — touched only if commemorative stickers or new shop items ship.
6. `RMConfig.Pro.cs:166–175` — read-only reference for the craft-cost table used by the scoring formula. After artifacts and gear are deleted, these blocks themselves should be removed.
7. A new player-facing "Artifact & Gear Removal" splash / FAQ view — recommended but not strictly required; not in the codebase today.

---

## 11. Recommendation

**Approach 5 (Hybrid)** is the recommended path for most player bases: every player gets a tangible gem grant they can immediately use; high-investment players additionally get the variety and commemorative weight they expect; the engineering blast radius is bounded by a small, short-lived shop and a single batch script.

**Approach 2 (Linear)** is recommended only if the design team has access to the backend transaction logs and is willing to publish gem-equivalent rates for dust and SLP. In that scenario it's the most defensible approach from a "fairness" angle and avoids tier cliffs entirely.

**Approach 1 (Flat Gem Refund, 5 tiers)** is the fastest possible solution. Run it if the artifact removal must ship in days, not weeks.

In every case, the auto-grant payload goes through the existing `RewardResponse` shape — there is no client-side feature work for the baseline grant.

---

## 12. Open questions for the design team

1. Is there a backend transaction log accessible to the batch script? This decides whether failed gear upgrades and recharges can enter the formula.
2. What's the desired gem-equivalent rate for dust and SLP? Required by Approach 2; useful messaging for all approaches.
3. Will the daily wheel survive the artifact removal? If yes, `bonus_spin` is a strong filler in Approach 3/4/5. If no, drop it.
4. Will lottery remain? Same logic — drop `lottery` from all bundles if it's being removed.
5. Are commemorative stickers / titles in scope? If not, drop them from Approach 3 and 5; the auto-grant + token shop still works without them.
6. Time pressure: how many weeks until artifact removal ships? Approach 5 needs ~2–4 weeks of engineering; Approach 1 needs days.
7. Do we want to publish the formula? Players will reverse-engineer the tier boundaries either way; pre-empting that with a clear "how we calculated your tier" page reduces backlash.
