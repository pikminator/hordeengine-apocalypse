# HordeEngine Apocalypse

"28 Days Later" / "28 Years Later" survival horror addon for [All of Create — Aeronautics v1.7](https://www.curseforge.com/minecraft/modpacks/all-of-create-aeronautics).
NeoForge 1.21.1. Java 21. GeckoLib 4.8.4.

## Atmosphere

Not zombies. Infected. Living humans with the rage virus.

- **They sprint** — faster than you. Pure adrenaline, no shambling.
- **They smell you** — living flesh has a scent. Blood draws them faster.
- **They hear you** — noise, footsteps, machines, screams. Silence is survival.
- **They drown** — they're alive. Water 3+ blocks deep kills them. But they climb out of shallows.
- **They climb** — ladders, obstacles. Their bodies run at full capacity.
- **They smash** — doors, glass, fences. They don't turn handles. They break through.
- **They swarm** — one spots you → every infected nearby joins the chase.
- **They evolve** — zones grow more dangerous each day. The infection spreads.
- **They never despawn** — saw you an hour ago? Still coming.

4 infected types: **Runner** (84%, fast), **Fast** (8%, outruns sprint), **Alpha** (4%, smart, leads the horde, screams), **Crawler** (4%, low profile, crawls).

## What this does

- **Statistical horde** — infected migrate between zones, never despawn, don't burn in sunlight
- **Noise-driven spawning** — loud actions amplify spawns up to 3×. Stay quiet, stay safe
- **Ambient dread** — 7×7 zone ambient sound scan. Hear the horde before you see it
- **Block breaking** — doors, glass, fences, wood. Stone too at high density
- **Swarm cohesion** — FollowHordeGoal: idle infected adopt nearby ally's target (10-block radius)
- **Alpha coordination** — ScreamAlertGoal: 32-block scream + scent burst alerts the entire horde
- **Infection** — one bite → 35 seconds to die. No cure. Dead player becomes a RunnerEntity
- **Water physics** — shallow → chase at 1.3× speed. Deep → drown + seek shore
- **Scent trails** — ScentTargetGoal with path reachability verification. Rain washes scent away
- **Grudge system** — escaped a zone with live infected → twice as many next time
- **Day scaling** — day 3: 1.3×, day 7: 1.7×, day 30: 2.2× population multiplier
- **Create integration** — moving contraptions emit noise proportional to speed and size
- **Grace period** — first 20 minutes: eternal day, no spawns

## Requirements

- [All of Create — Aeronautics v1.7](https://www.curseforge.com/minecraft/modpacks/all-of-create-aeronautics)
- The mods listed in [modlist.md](modlist.md)
- **Remove these mods** (they conflict with HordeEngine AI):
  - `enhancedai` — AI goals clash
  - `zombieapocalypseaddon` — duplicate spawn/horde system
  - `smoothchunk` — thread scheduler conflict, micro-stalls
  - `chunkactivitytracker` — same as above
  - `e4mc` — network flooding

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

Entity tracking via events (join/leave/death) — zero world scans in the tick loop.
O(1) zone queries via HashMap. Scent goals verify path reachability before committing to expensive pathfinding.

## Development

See [DEVGUIDE.md](DEVGUIDE.md) for build instructions, architecture overview, and debugging.
