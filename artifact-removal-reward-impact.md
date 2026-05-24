# Artifact Removal -- Reward Economy Impact & Replacement Plan

**Date:** 2026-05-11
**Purpose:** Map every place artifacts appear as rewards and propose replacements.

---

## Current Artifact Reward Sources (Complete Inventory)

### 1. Gold Chests (Battle Drops)

Each chest gives gold + a % chance at 1 artifact (class-weighted random type):

| Chest Tier | Gold Range | Artifact Drop Rate | Artifact Qty |
|------------|-----------|-------------------|-------------|
| Wood | ~4 | 5% | 1 |
| Copper | ~12 | 10% | 1 |
| Silver | ~30 | 20% | 1 |
| Gold | ~72 | 40% | 1 |
| Mystic | ~192 | 100% (guaranteed) | 1 |

**Config:** `artifact_drop_rate` column in `gold_chests` DB table. Code: `gold_chest.go:64-71`.

**Replacement options:**
- Replace artifact slot with additional gold (e.g., +50-200 gold per tier)
- Replace with dust (gear crafting material) to funnel players into the gear system
- Replace with gem (premium currency, lower quantities: 2-10 per tier)
- Replace with bonus wheel spin (matches Colosseum chest precedent)
- Set `artifact_drop_rate = 0` in DB -- **zero code changes needed**, chest just gives gold only

---

### 2. Colosseum Chests

Each Colosseum chest gives exactly one reward -- either an artifact OR a bonus wheel spin:

| Outcome | Rate | Reward |
|---------|------|--------|
| Artifact | `(wins+1)/(maxWins+1)` | 1 artifact (class-weighted) |
| Bonus Spin | remainder | 1 free wheel spin |

More wins = higher artifact chance. Lock duration: 12 hours. Max 2 pending.

**Config:** Code-driven in `coloseum_chest_manager.go:37-82`.

**Replacement options:**
- Always give bonus spin (simple, already the fallback)
- Replace artifact outcome with gold (e.g., 100-300 scaling with wins)
- Replace with dust + gold combo
- Requires code change in `coloseum_chest_manager.go`

---

### 3. Wheel of Fortune (Golden Wheel)

10 slots (web3) / 8 slots (web2). Slot 1 is always overwritten with an artifact:

| Slot | Type | Amount |
|------|------|--------|
| **1** | **artifact** | **1** |
| 2 | gold | 80 |
| 3 | gold | 80 |
| 4 | gold | 40 |
| 5 | gold | 40 |
| 6 | gold | 40 |
| 7 | gold | 20 |
| 8 | gold | 20 |
| 9 | maxs_in | 100 |
| 10 | slp_in | 100 |

5 base daily spins. Costs: free (1st, requires 1 win), then 2/4/6/8 gems.

**Config:** DB table `default_wheel_items` + code override at `wheel.go:248-257`.

**Replacement options:**
- Replace slot 1 with higher gold (200+ to feel premium)
- Replace with dust (250-500, makes gear crafting more accessible)
- Replace with gem (10-20, high perceived value)
- Replace with sticker fragment (if sticker system is revived)
- Requires code change in `wheel.go` to stop overwriting slot 1 with artifact, plus DB update

---

### 4. Battle Pass (Season Milestones)

Artifacts are heavily woven into both tracks. Season 1 data:

**Free Track (13 milestones):**

| Milestone | Score | Reward |
|-----------|-------|--------|
| 0 | 200 | 100 gold |
| 1 | 400 | 1 beast artifact |
| 2 | 600 | 1 aquatic artifact |
| 3 | 800 | 200 gold |
| 4 | 1000 | 1 plant artifact |
| 5 | 1200 | 3 bird artifacts |
| 6 | 1400 | 300 gold |
| 7 | 1600 | 3 bug artifacts |
| 8 | 1800 | 3 reptile artifacts |
| 9 | 2000 | 500 gold |
| 10 | 2200 | 3 beast artifacts |
| 11 | 2400 | 3 aquatic artifacts |
| 12 | 2600 | 3 plant artifacts |

9 of 13 milestones give artifacts (~22 total). Premium track is even more artifact-heavy (~30+ artifacts across all milestones, bundled with SLP and MAXS).

Premium pass also grants **+10 bonus artifact uses/day** for the season.

**Config:** JSON in `battle_pass_configs.milestones` column, fully data-driven per season.

**Replacement options:**
- Replace artifact milestones with gold (scale: 200-500 gold per milestone to match perceived value)
- Replace with dust + gem mix (e.g., 100 dust + 5 gems per artifact milestone)
- Replace with gear boxes (random gear of appropriate level)
- This is the biggest perceived-value hit -- artifacts were the "exciting" reward in the pass
- **No code changes needed** -- milestone rewards are DB/CMS configured. Just update the JSON.

---

### 5. Quests

**Artifact as reward (web2 lifetime quests):**

| Quest | Target | Reward |
|-------|--------|--------|
| Finish 1 PvP battle | 1 | 1 plant artifact |
| Finish 5 PvP battles | 5 | 1 aquatic artifact |
| Finish 10 PvP battles | 10 | 1 beast artifact |

**Artifact as objective (daily/weekly quests):**

| Quest | Target | Reward |
|-------|--------|--------|
| Use artifact 1 time | 1 | 20 gold |
| Use artifact 5 times | 5 | 20 gold |
| Use artifact 10 times | 10 | 50 gold |

**Config:** Quest templates in DB, managed via CMS.

**Replacement options:**
- Replace artifact rewards with gold or dust
- **Delete "use artifact" quests entirely** -- these become impossible without artifacts
- **No code changes needed** for reward replacement (DB-driven)
- May need code change to remove `use_artifact` quest template type, or just remove those quests from DB

---

### 6. Tower Mode

Tower floor rewards are fully CMS/DB-configured. Specific floors award artifacts alongside gold/AXP. No hardcoded artifact rewards in code.

**Replacement options:**
- Replace with gold, dust, gems, or gear boxes per floor
- **No code changes needed** -- update floor reward JSON in CMS

---

### 7. Starter & Compensation Packs

| Pack | Contents |
|------|----------|
| Starter | 1 of each type not yet owned (up to 6 total) |
| Compensation | 8 of each type (48 total) |

**Config:** Hardcoded in `user_artifacts.go:62-104`, triggered via CMS admin.

**Replacement options:**
- Starter: replace with gold (600 gold = 100 per artifact)
- Compensation: replace with gold (4,800 gold = 100 per artifact) or mixed gold/dust/gems
- Requires code change in `user_artifacts.go` and `cms.go`

---

### 8. Over-Cap Conversion

Players at 71 copies auto-convert excess artifacts to 100 gold each. This disappears entirely if artifacts are removed. Minor gold source -- most players never hit the cap.

---

## Summary: Replacement Effort by Source

| Source | Code Change Needed? | Config-Only Fix? | Impact |
|--------|-------------------|-----------------|--------|
| Gold Chests | No | Yes (DB: set `artifact_drop_rate = 0`) | Low |
| Colosseum Chests | Yes (`coloseum_chest_manager.go`) | No | Low |
| Wheel of Fortune | Yes (`wheel.go` + DB) | No | Medium |
| Battle Pass | No | Yes (CMS: update milestone JSON) | **High** (perceived value) |
| Quests (rewards) | No | Yes (DB: update quest templates) | Low |
| Quests (objectives) | Maybe | Yes (DB: delete "use artifact" quests) | Low |
| Tower Mode | No | Yes (CMS: update floor rewards) | Low |
| Starter Pack | Yes (`user_artifacts.go`, `cms.go`) | No | Low |
| Compensation Pack | Yes (`user_artifacts.go`, `cms.go`) | No | Low |

---

## Recommended Replacement Currency: Dust

If a single replacement currency is chosen, **dust** (gear crafting material) is the best candidate:

1. Artifacts and gears serve similar purposes (team power progression)
2. Funneling artifact rewards into dust strengthens the gear system
3. Dust has clear value to players (gear crafting costs 500-12,000 dust per gear)
4. Dust is not a premium currency (unlike gems), so it doesn't inflate the economy
5. Gold is already abundant -- more gold has diminishing excitement

**Suggested conversion rates (artifact → dust):**

| Context | Per Artifact | Rationale |
|---------|-------------|-----------|
| Chest drops | 150 dust | Low-frequency, small bonus |
| Wheel slot | 250 dust | Mid-value slot, comparable to 80 gold |
| Battle Pass (free) | 200 dust per artifact | Milestone progression reward |
| Battle Pass (premium) | 300 dust per artifact | Premium track should feel better |
| Quest rewards | 150 dust per artifact | Early onboarding reward |
| Tower floors | 200-400 dust per artifact | Scales with difficulty |

---

## Total Dev Effort for Reward Rebalancing

| Task | Effort |
|------|--------|
| DB updates (chest rates, quest templates) | 1 hour |
| CMS updates (battle pass milestones, tower floors) | 2-3 hours |
| Code: Colosseum chest manager | 1 hour |
| Code: Wheel slot 1 replacement | 1 hour |
| Code: Starter/compensation pack | 1 hour |
| Testing all reward flows | 3-4 hours |
| **Total** | **~1 day** |

This is additive to the server-side disable effort (0.5 day) and optional client UI removal (1-5 days depending on scope).
