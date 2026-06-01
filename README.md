# HordeEngine Apocalypse

"28 Days Later" survival horror addon for [All of Create — Aeronautics v1.7](https://www.curseforge.com/minecraft/modpacks/all-of-create-aeronautics).
NeoForge 1.21.1.

## What this does

- **Statistical horde system** — zombies migrate between zones, never despawn, don't burn in sunlight
- **Burst spawning** — enter an infested zone → 8 zombies/tick until counter depleted
- **Noise attraction** — breaking blocks, walking, sprinting, jumping, taking damage, opening chests, Create machines — everything attracts nearby zombies
- **Ambient dread** — 7×7 zone ambient sound scan. Hear the horde before you see it
- **Block breaking** — zombies smash doors, fences, trapdoors, glass, wood to reach you
- **Group consciousness** — one zombie spots you → entire zone (5×5 for Screamers) aggroes
- **Screamers** — no line-of-sight needed, play alert sound, give Speed III burst to all nearby
- **Create integration** — moving contraptions (trains, bearings, drills) emit noise proportional to speed and size
- **Tornado protection** — tornadoes destroy buildings but preserve chests, beds, and mechanisms
- **Performance-first** — zero world scans in tick loop. O(1) entity queries via custom tracker. Verified 0% server tick load via Spark profiler

## Requirements

- [All of Create — Aeronautics v1.7](https://www.curseforge.com/minecraft/modpacks/all-of-create-aeronautics)
- The mods listed in [modlist.md](modlist.md)
- Recommended: remove `enhancedai`, `zombieapocalypseaddon` (HordeEngine replaces them)

## Installation

1. Install All of Create — Aeronautics v1.7
2. Install additional mods from [modlist.md](modlist.md) into `mods/`
3. Merge `config/` and `datapacks/` from this repo into the instance folder
4. Copy `HordeEngine-1.1.0.jar` into `mods/`
5. Create a **new world** (biome changes need fresh worldgen)
6. Launch. First 20 minutes are grace period — eternal day, no spawns

## Commands

| Command | Output |
|---------|--------|
| `/horde zone` | Counter, horde loaded, ALL zombies in zone |
| `/horde nearby` | 3×3 zone grid, single message |
| `/horde total` | Global stats across all zones |
| `/horde debug` | Toggle spawn/noise messages in chat |
| `/horde grace end` | End grace period immediately |
| `/horde reload` | Reload config from `hordeengine-common.toml` |

## Modified configs

| Config | What changed | Why |
|--------|-------------|-----|
| `enhancedmobcap-common.toml` | Monster cap 0.4, water 0.2, ambient 0.1 | HordeEngine handles zombies |
| `zombieawareness/Features.toml` | All mechanics → false | Keep only sound assets |
| `zombieawareness/General.toml` | blockBreak=false, findSense=0 | Disable CPU-heavy logic |
| `Weather2/Tornado.toml` | Blacklist mode: protect chests, beds, mechanisms | Tornado destroys walls, not interiors |
| `tornadophysics-common.toml` | Expanded destroyableBlocks (wood+glass+wool) | Buildings break, mechanisms stay |
| `terrablender.toml` | overworld_region_size=2 | Smaller biomes |
| `tectonic.json` | temperature_offset=-0.18, scale=0.16 | Fewer jungles, more varied |
| `quark-common.toml` | Toretoise=false | Saves 3% TPS |
| `modernfix-mixins.properties` | Thread/classinfo tweaks | Less memory, less thread contention |
| `physicsmod/physics_server.toml` | Various | Physics performance |
| `presencefootsteps/userconfig.json` | Louder wind, stronger foliage | Immersion |
| `ComplementaryReimagined...txt` | Better underwater, less pitch-black | Visibility |

## Performance

HordeEngine uses **zero world scans** in the tick loop. Entity tracking via events (join/leave/death).
Verified 5× with Spark profiler: HordeEngine = 0% of server tick time.

**PC requirements:** 16 GB RAM minimum. Close Chrome before playing. The modpack has 234 mods + Sable physics. Swap file should be fixed (not auto) — 4-8 GB.

## Development

See [DEVGUIDE.md](DEVGUIDE.md) for build instructions, architecture overview, and debugging.
