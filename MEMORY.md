# HordeEngine Apocalypse — Session Memory

Key facts from development sessions. Load at session start to restore context after `/clear`.

## Performance (verified 5× with Spark)

- HordeEngine = **0% server tick time**. All `getEntitiesOfClass` replaced by EntityTracker (event-driven, O(1) HashMap queries)
- TPS issues were NEVER from HordeEngine — always system/other mods
- Main TPS killer: swap file (was 17 GB auto → fixed to 4-8 GB). 15.8 GB RAM, Minecraft uses ~7.5 GB total (5 GB heap + 2.5 GB native)

## Mods to remove from All of Create — Aeronautics v1.7

- `enhancedai` — HordeEngine handles block breaking + sprint via own mixins
- `zombieapocalypseaddon` — duplicates spawn/horde/blood moon
- `smoothchunk` — thread scheduler conflicts
- `chunkactivitytracker` — same
- `e4mc` — network flood

## Mods modified but kept

- `zombieawareness` — ALL mechanics disabled, kept ONLY sound assets (used in our sounds.json)
- `enhancedmobcap` — monster cap reduced to 0.4 (HordeEngine handles zombies)
- `modernfix` — thread_priorities=false, fix_loop_spin_waiting=false, clear_mixin_classinfo=true
- `quark` — Toretoise=false (3% TPS saved)

## Known crashes (not HordeEngine)

- Waves mod — `JsonHelpers.compareVersions`, NoSuchElementException. Remove it.
- Sable — `VoxelNeighborhoodState` ArrayIndexOutOfBoundsException (leaf decay). Sable is needed for Create Aeronautics.
- watut — missing shader `particle.json`, triggers Create config load crash. Update or remove watut.

## Architecture decisions

- `spawnCooldown=10` (was 0, then 20). Current: 0.5s. Balance between film-feel burst and TPS.
- `maxSpawnLight=13` — canopy OK (12-13), direct sun blocked (15). Critical for daytime forest danger.
- Forest infestation 90% vs 55% elsewhere. Key for "28 Days Later" daytime pressure.
- Removed EnhancedAI dependency entirely. Brought block breaking back into MixinZombie.customServerAiStep. Simpler, faster, no external dep.
- `zoneSizeBlocks=48` (3 chunks). Smaller zones = more granular migration.
- `STEP_HEIGHT=1.0` on all horde zombies — parkour over 1-block obstacles.
- Speed II effect (infinite) on all horde zombies. Base speed 0.43.

## Build system

- Gradle wrapper. Java: MS OpenJDK 21 from `C:\Users\dsd\AppData\Roaming\PrismLauncher\java\java-runtime-delta\`
- `compileOnly` dependency on Create 6.0.10 via flatDir from mods folder
- Binary: `gradle/wrapper/gradle-wrapper.jar` — no gradle installation needed
- Deprecation warning on SpawnHandler/MixinZombie/BlockBreaker — `builtInRegistryHolder().key().location()` is deprecated in 1.21.1 but works. Use `BuiltInRegistries` in future MC versions.

## Debug tools

- Spark profiler: `/spark profiler start` → play 60s → `/spark profiler stop` → link
- `/spark heapsummary` for memory
- Logs: grep `[HordeEngine]` in `latest.log`
- `/horde zone` shows ALL zombies (horde + other mods) via one-time AABB scan
- Shtreimel: `/shtreimel rank` for contraption TPS diagnosis (install separately)

## Configs changed outside our mod

| Config | Key change | Why |
|--------|-----------|-----|
| enhancedmobcap-common.toml | monster=0.4, water=0.2 | HordeEngine handles zombies |
| zombieawareness/Features.toml | ALL→false | Keep sounds, kill CPU |
| Weather2/Tornado.toml | Blacklist mode | Protect chests/beds |
| tornadophysics-common.toml | Expanded destroyable | Wood+glass+wool |
| terrablender.toml | region_size=2 | Smaller biomes |
| tectonic.json | temp_offset=-0.18 | Fewer jungles |
| quark-common.toml | Toretoise=false | 3% TPS |
| modernfix-mixins.properties | thread tweaks off | Unblock tick loop |
