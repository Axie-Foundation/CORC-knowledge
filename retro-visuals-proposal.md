# Axie Classic -- Retro Visuals Feature Toggle Proposal

**Date:** 2026-05-25
**Goal:** Allow players to toggle a "retro visuals" mode that changes the look of Axies in-game to an older or stylized visual presentation.

---

## How Axie Rendering Currently Works

Understanding the current pipeline is critical for scoping this feature.

### The Pipeline: Gene → Screen

1. **Fetch genes** -- GraphQL call to `axieinfinity.com/graphql-server/graphql` returns a 512-bit hex string per Axie encoding class, body parts, skin type, and color variants.
2. **Parse genes locally** -- Hex is decoded bit-by-bit into an `AxieBodyStructure`: class (9 types), 6 body part slots (eyes, ears, mouth, horn, back, tail), each with skin/class/value, plus color variant data.
3. **Assemble Spine skeleton** -- The AxieMixer package (`com.skymavis.axiemixer.unity` v0.7.4) composites a complete Spine skeleton from 133 embedded skeleton templates + remotely loaded textures. No pre-packed atlases -- individual part sprites are loaded from CDN and assembled at runtime.
4. **Render** -- Battle uses `SkeletonAnimation` (world-space), UI uses `SkeletonGraphic` (canvas-space).

### Where the Art Lives

| Asset | Location | How Loaded |
|-------|----------|------------|
| Gene-to-part mapping | AxieMixer package (embedded JSON, 51 KB) | Local |
| 133 skeleton templates | AxieMixer package (embedded JSON, 2.8 MB) | Local |
| 489 variant overrides | AxieMixer package (embedded JSON, 11 KB) | Local |
| Animation data | AxieMixer package (embedded JSON, 2.4 MB) | Local |
| Body part textures | CDN via DLC/Addressables | Remote: `cdn.skymavis.com/mavisx/dlc-central/axie-classic-pack/extra/` |
| Card art sprites | Addressable bundles | Remote: `storage.googleapis.com/axie-game-assets/dlc/bundles-6b/` |
| Chimera/NPC skeletons | DLC dynamic config pack | Remote: downloaded at startup |

### Key Finding: No Multiple Art Versions Exist

The AxieMixer data is versioned as `v3` (data files) / `v6` (CDN images). There is only one art version. The `AxieFormType` enum exists with only `Normal` used. There is no "v1 vs v2" or "classic vs origins" art style system. The CDN path `/mixer-stuffs/v6/` indicates the art has been iterated 6 times, but only the latest version ships.

### Legacy CDN Endpoint (Possibly Relevant)

A commented-out URL in `HttpConstants.cs:37`:
```
https://assets.axieinfinity.com/axies/{id}/axie/axie-half-baseline.png
```
This endpoint is **live but returns 403 Forbidden**. It was the original pre-rendered Axie image CDN -- server-side rendered PNGs per Axie ID. Replaced by the client-side AxieMixer Spine assembly.

---

## Approaches

### Approach A: Shader-Based Retro Filter (Easiest -- No New Art)

Apply a post-processing shader to the existing Spine output that makes it *look* retro: pixelation, reduced color palette, dithering, optional CRT scanlines.

**How it works:**
- Create a custom shader/material that downsamples the Spine render to a lower pixel density and quantizes colors
- When "retro mode" is toggled, swap the render material on `SkeletonAnimation` (battle) and `SkeletonGraphic` (UI) components
- The underlying Spine skeleton is unchanged -- same animations, same parts, just rendered through a different visual filter
- A global `Camera` post-processing effect could also apply to backgrounds, cards, and UI simultaneously

**Implementation points:**
- `Axie2dSpineGenerator.RequestAxie()` -- swap `sharedGraphicMaterial` on the `Axie2dBuilderResult`
- `FighterFigureProvider.CreateAxieSpineFighter()` -- swap material on `SkeletonAnimation`
- `FighterFigureGraphicProvider.CreateAxieSpineUI()` -- swap material on `SkeletonGraphic`
- Optionally apply a camera-level post-process to pixelate the entire battle scene

**Effort:** ~2-3 days (shader development + toggle UI + settings persistence)

**Pros:**
- Zero new art assets needed
- Works with every Axie automatically, including future ones
- Trivial to toggle on/off at runtime
- Can be extended with multiple filter presets ("8-bit", "16-bit", "Game Boy", "CRT")

**Cons:**
- It's a filter, not actual retro art. Players expecting "the old Axie look" won't get it
- Pixelation of Spine animations can look muddy at certain resolutions
- Doesn't change the underlying art style, just distorts it

---

### Approach B: Alternate AxieMixer Art Pack (Most Authentic, Hardest)

Bundle a second set of body part textures with a retro/classic art style. The AxieMixer architecture actually supports this cleanly because textures are loaded separately from skeleton templates.

**How it works:**
- The AxieMixer loads skeleton templates (bones, slots, animations) locally but loads **textures from CDN via Addressables**
- Create a second texture set with the retro art style for every body part
- Host it at an alternate CDN path (e.g., `/mixer-stuffs/v6-retro/`)
- When retro mode is active, point the Addressable loader to the retro texture path instead of the standard one
- The skeleton templates, animations, and gene mapping remain identical -- only the textures change

**Implementation points:**
- DLC pack configuration: add a `classic_skeletontextures_retro` label group pointing to the retro CDN
- `Axie2dSpineGenerator.RequestAxie()` -- check retro toggle, use retro materials
- `LoadRemoteSpineJob` -- conditionally load from retro texture path
- Remote config: `retroTexturesCdnPath` to control the URL server-side

**Effort:**
- Engineering: ~1 week (Addressable config, texture loader branching, toggle, caching)
- Art production: **Major** -- every body part across all 6 classes, all skins (Global, Mystic, Japan, Xmas, Summer, Nightmare), and lv2 variants needs a retro version. Hundreds of individual sprites.

**Key question for Sky Mavis:** Do the original v1/v2/v3/v4/v5 body part textures still exist? The CDN is on `/mixer-stuffs/v6/` -- if earlier versions are archived, they could be the retro art pack with minimal new production.

**Pros:**
- Authentic retro look per Axie with correct gene-based parts
- Uses the existing AxieMixer pipeline -- no architectural changes
- Textures are remotely loaded, so the retro pack doesn't bloat the base install (downloaded on demand)

**Cons:**
- Requires a full retro art set to exist or be produced
- CDN hosting cost for a second texture set
- Must be maintained alongside the primary art (new parts/skins need retro versions too)

---

### Approach C: Pre-Rendered Image Fallback (CDN-Based Nostalgia)

Restore access to the legacy `assets.axieinfinity.com` CDN and use the old pre-rendered PNGs as static Axie portraits in retro mode.

**How it works:**
- In retro mode, replace `SkeletonAnimation`/`SkeletonGraphic` with a simple `Image`/`SpriteRenderer` loading from:
  ```
  https://assets.axieinfinity.com/axies/{axieId}/axie/axie-half-baseline.png
  ```
- Battle would show static 2D sprites with simple tween animations (bounce, shake, flash) instead of Spine skeletal animation
- UI portraits would show the original pre-rendered Axie images

**Implementation points:**
- `FighterFigureProvider.Provide()` -- when retro mode, create a `SpriteRenderer` + `Animator` instead of Spine components
- Need simple animation controllers for: idle (breathing bob), attack (lunge forward), hit (flash red), death (fade out)
- Image caching system to avoid re-downloading per battle
- Graceful fallback: if CDN image not found (newer Axies), fall back to shader-filtered Spine

**Effort:**
- Engineering: ~1 week (sprite renderer, simple animations, caching, fallback)
- CDN work: restore access to `assets.axieinfinity.com` (may need backend/infra team)
- Zero art production if the CDN images still exist

**Blockers:**
- The CDN currently returns 403. Someone needs to restore access.
- Coverage: the CDN may not have images for all Axies (especially post-2022 ones)
- Static images mean no part-level animation -- attacks, buffs, debuffs would all use generic effects

**Pros:**
- Uses actual historical art that OG players remember
- Maximum nostalgia factor
- No new art production if CDN data exists

**Cons:**
- Static images -- fundamentally different battle feel
- CDN dependency (external service must stay up)
- Incomplete coverage for newer Axies
- Battle readability suffers without Spine animations (hard to tell attack from idle)

---

### Approach D: Hybrid -- Retro Shader + Static Portraits (Recommended)

Combine Approaches A and C for the best tradeoff:

- **Battle:** Apply a retro pixel shader to existing Spine animations. Axies still animate fully but look pixelated/retro with a reduced color palette.
- **UI Portraits:** Pull from the legacy CDN for authentic old Axie images where available. Fall back to shader-filtered Spine renders for newer Axies.
- **Feature toggle:** Remote config flag + in-game settings switch.

**Implementation points:**
- Shader: custom material with pixelation + color quantization (configurable pixel size, palette)
- Portrait loader: async image fetch from `assets.axieinfinity.com` with disk cache + fallback
- Settings: `PlayerPrefs` persistence, remote config gate (`retroVisualsAvailable: bool`)
- Battle scene: camera-level post-process for backgrounds + per-Axie material swap for figures

**Effort:** ~1-1.5 weeks total (shader: 2-3 days, portrait loader: 2-3 days, settings/toggle: 1 day, testing: 2 days)

**Pros:**
- Animated retro battles (no loss of gameplay clarity)
- Authentic old portraits in UI where available
- Graceful degradation for Axies without CDN images
- No new art production needed
- Remotely toggleable via config

**Cons:**
- Two different visual treatments (shader in battle vs CDN images in UI) may feel inconsistent
- CDN dependency for portraits (but fallback exists)
- Shader filter is stylized, not truly "original art"

---

## Comparison Matrix

| Criteria | A: Shader Filter | B: Alt Art Pack | C: CDN Images | D: Hybrid (Rec.) |
|----------|-----------------|-----------------|---------------|-------------------|
| **Authenticity** | Low (stylized) | High (real retro art) | High (original images) | Medium-High |
| **Battle animation** | Full Spine (filtered) | Full Spine (retro art) | Static sprites | Full Spine (filtered) |
| **New art needed** | None | Hundreds of sprites | None (if CDN data exists) | None |
| **Engineering effort** | 2-3 days | 1 week eng + months art | 1 week | 1-1.5 weeks |
| **CDN dependency** | None | New CDN path | Legacy CDN (403'd) | Legacy CDN (fallback) |
| **Coverage (all Axies)** | 100% | 100% (if art produced) | Partial (older Axies only) | 100% (battle) / partial (UI) |
| **Toggle complexity** | Material swap | Addressable path swap | Renderer swap | Material + image loader |
| **Future maintenance** | None | Must produce retro art for new parts | None | None |
| **Player perception** | "Cool filter" | "The real classic look" | "Actual nostalgia" | "Best of both" |

---

## Open Questions for Sky Mavis

Before committing to an approach, these questions need answers:

1. **Do earlier texture versions (v1-v5) still exist?** The CDN serves from `/mixer-stuffs/v6/`. If v1-v5 textures are archived, Approach B becomes dramatically cheaper -- we'd use historical art instead of producing new retro art.

2. **What's the status of `assets.axieinfinity.com`?** It returns 403, not 404. Is the data still there behind auth? Can access be restored? What Axie ID range has coverage? This determines viability of Approaches C and D.

3. **What does "retro" mean to the player community?** Is it the very first pixel art Axies (2018)? The early Spine animations (2019-2020)? The pre-Origins look (2021)? The answer determines which art version to target.

4. **Is this a monetizable feature?** Could "retro skin pack" be a premium cosmetic ($2-5)? This affects whether it's worth investing in Approach B's art production.

---

## Recommendation

**Start with Approach A (shader filter) as a quick win**, then layer Approach D (hybrid) if the legacy CDN data is recoverable.

**Phase 1 (2-3 days):** Ship the retro shader filter. Add a toggle in Settings. Gate behind remote config. This gives players a "retro mode" immediately with zero art dependency. Include multiple presets: "8-bit", "16-bit", "Game Boy palette", "CRT scanlines."

**Phase 2 (1 week, dependent on CDN investigation):** If `assets.axieinfinity.com` data is recoverable, add the CDN portrait loader for UI screens. Players get authentic old Axie images on the home screen, team detail, and match history while battles use the shader filter.

**Phase 3 (if warranted by player demand + old textures exist):** If v1-v5 mixer textures are archived, build Approach B -- a full retro art pack loaded via alternate Addressable path. This is the gold standard but only worth the investment if Phase 1/2 prove player demand.

This phased approach lets you ship value fast, validate demand, and invest in authenticity only if warranted.
