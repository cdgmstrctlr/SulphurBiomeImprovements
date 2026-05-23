# Sulphur Biome Improvements

# Sulfur Biome Enhancements (Minecraft Bedrock Add‑On)

This add‑on improves several aspects of the new sulfur biome in Minecraft Bedrock Edition, enhancing both visuals and gameplay while maintaining full compatibility with existing worlds.

## Features

- **Improved Potent Sulfur Placement**  
  Potent sulfur now almost always generates in water with air above it, allowing noxious gases to be visible.

- **Optimized Sulfur Pool Generation**  
  Reduces sulfur pool generation retries to improve performance with minimal impact on pool count.

- **Restored Glow Lichen Lighting**  
  Reintroduces glow lichen on walls and ceilings at a moderated density — between the original Chaos Cubed version and the current shipping version.

- **Surface Sulfur Springs**  
  Adds rare sulfur springs with geysers to surface sulfur biomes, making these biomes feel more alive and unique.

---

# Technical Details

## Sulfur Pool Potent Sulfur

This add‑on uses no resource packs, so it is safe to remove from worlds where it has already modified generation. Removing it (and deleting it from the world’s behavior pack folder) restores the original Chaos Cubed generation for all newly generated chunks.

### Why the Original Placement Was Inconsistent

The original potent sulfur placement relied on the `minecraft:search_feature` component to scan an 8×8×8 volume for a sulfur block with water above it. However, the C++ `minecraft:sulfur_pool` runtime feature generates from an origin point often placed at the pool’s edge. Because the side the origin is on cannot be predicted, the search frequently selected suboptimal locations — typically water columns without air above them.

### New Two‑Stage Search Method

This add‑on replaces the original search with a more reliable two‑stage process:

1. **XZ‑Area Search (5×5×5)**  
   Looks for an air block with air on all four sides and water below.

2. **Vertical Column Search Follows**  
   - Allowing magma introduces a rare chance for a geyser to appear in the caves.

The vertical search is performed twice:
- Once at the original sulfur pool anchor point  
- Once at the new XZ search location  

This increases the likelihood that each pool will contain at least one potent sulfur block, improving gas coverage across larger pools.

### Performance Adjustments

- Sulfur pool tries reduced: **256 → 131**  
- Cave‑floor search depth reduced: **32 → 16**  
  - Empirically, 16 performs better — possibly due to an internal watchdog timer.

These changes are injected via an override of `minecraft:potent_sulfur_search_feature`. Because some Bedrock runtime features cannot be called directly from creator add‑ons (hmm), this override is implemented as an aggregate feature that runs both search methods after the orginal aggregate feature runs.

---

## Glow Lichen

Glow lichen has been re‑added to sulfur caves to restore the atmospheric lighting originally intended for the biome.

- Based on a different underground feature rule (the original was removed in 26.30.29).
- Uses **919 tries** (I didn't save the original feature rule, but I believe it was approximately 1200).
- Produces good illumination while still allowing some natural shadows.
- Uses the existing multiface glow lichen feature shipped with the game.

---

## Surface Biome Sulfur Springs

The default sulfur spring features do not allow placement on sulfur or cinnabar blocks. Even when modified to allow these blocks, springs still fail to generate—likely due to how the “root” generator handles sulfur columns from caves.

To compensate, this add‑on introduces a new feature rule that:

- Places springs directly on the surface using the world heightmap.
- Runs with either 0 tries or 1 try per chunk.
- Still results in several springs appearing in sulfur surface biomes, making them feel special and distinct.

---

## Surface Sulfur Cubes

Attempts to enable sulfur cube spawning in surface sulfur biomes were unsuccessful. Overriding their spawn rule and entity does not appear to activate the changes. This may require an in-house fix.

---

## Compatibility

- **Safe to add to existing worlds**  (Requires Chaos Cubed)
- **Safe to remove** (restores original generation for new chunks)  
- No resource pack required
