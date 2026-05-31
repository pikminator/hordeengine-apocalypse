# HordeEngine Apocalypse

A modpack addon that transforms Create: All of Create — Aeronautics into a 28 Days Later survival horror experience. Fast zombies, horde migration, ambient dread, and noise-based attraction — but still fully compatible with Create engineering.

## What this does

- Replaces vanilla zombie spawning with a **statistical horde system** — zombies migrate between zones, react to noise, and never despawn
- Adds **ambient sound layers** — hear the horde before you see it
- Balances world generation for **smaller biomes, less ocean, less rain**
- Makes zombies **fast, persistent, and deadly** — but escapable with skill
- Keeps Create intact — works alongside all machines and contraptions

## Requirements

- [All of Create — Aeronautics v1.7](https://www.curseforge.com/minecraft/modpacks/all-of-create-aeronautics) (NeoForge 1.21.1)
- The additional mods listed in [modlist.md](modlist.md)

## Installation

1. Install **All of Create — Aeronautics v1.7** via PrismLauncher or your preferred launcher.
2. Download and install all mods listed in [modlist.md](modlist.md) into the `mods` folder.
3. Copy everything from this repo into the Minecraft instance folder (merge `config/`, `shaderpacks/`, `datapacks/`).
4. Copy `HordeEngine-1.0.0.jar` into the `mods` folder.
5. **Create a new world.** Old worlds won't get the new biome generation.
6. Launch and survive.

## What you get

| System | Description |
|--------|-------------|
| **Horde migration** | Zombies flow between zones. 20% move every 30 seconds. |
| **Burst spawning** | Enter an infested zone → all zombies materialize in seconds. |
| **Noise attraction** | Chop trees, mine stone, build contraptions → zombies hear it. Sit quietly → safe. |
| **Ambient sounds** | Zombie Awareness investigate sounds play from high-density zones. Day: quiet. Night: loud. |
| **Special zombies** | Tank (3%), Screamer (4%), Spitter (4%), Mutant (8%). Each has unique stats. |
| **Block breaking** | Doors, fences, glass, torches break when zone density > 80%. |
| **Day scaling** | Zombie caps increase over weeks. Day 1-3: base. Day 30+: 2.2x. |
| **40% infested zones** | Not everywhere is dangerous. Find safe corridors. |
| **Grace period** | First 20 minutes: eternal day, no spawns. Time to prepare. |

## Modified configs

| Config | Changes |
|--------|---------|
| `tectonic.json` | Smaller biomes, less ocean, shorter mountains |
| `eclipticseasons-common.toml` | Rain reduced to 30% of vanilla |
| `toughasnails/temperature.toml` | Cold penalties halved, respawn clemency enabled |
| `enhancedai/common.toml` | Zombie/drowned/villager AI disabled (vanilla targeting) |
| `enhancedai/.../custom_hostile.json` | Villager attack chance 50% → 100% |
| `presencefootsteps/userconfig.json` | Louder wind, stronger foliage sounds |
| `ComplementaryReimagined...txt` | Better underwater visibility, less pitch-black caves |

## HordeEngine commands

```
/horde zone     — Current zone stats
/horde nearby   — 3x3 grid of neighbor zombie counts
/horde total    — All zones on the server
```
