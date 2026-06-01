# HordeEngine Apocalypse — Developer Guide

## Project locations

| What | Path |
|------|------|
| Source code | `C:\Users\dsd\projects\HordeEngine` |
| Built JAR | `C:\Users\dsd\projects\HordeEngine\build\libs\HordeEngine-1.1.0.jar` |
| This dist folder | `C:\Users\dsd\Desktop\projects\hordeengine-apocalypse` |
| Modpack (PrismLauncher) | "All of Create - Aeronautics" v1.7 |
| Modpack files | `C:\Users\dsd\AppData\Roaming\PrismLauncher\instances\All of Create - Aeronautics\` |

## Build & Deploy

```powershell
cd C:\Users\dsd\projects\HordeEngine
$javaBin = "C:\Users\dsd\AppData\Roaming\PrismLauncher\java\java-runtime-delta\bin\java.exe"
& $javaBin -Xmx64m -Xms64m "-Dorg.gradle.appname=gradlew" -classpath "gradle/wrapper/gradle-wrapper.jar" org.gradle.wrapper.GradleWrapperMain clean build --no-daemon
Copy-Item "build\libs\HordeEngine-1.1.0.jar" "C:\Users\dsd\AppData\Roaming\PrismLauncher\instances\All of Create - Aeronautics\minecraft\mods\HordeEngine-1.1.0.jar" -Force
```

Java: `C:\Users\dsd\AppData\Roaming\PrismLauncher\java\java-runtime-delta\` (MS OpenJDK 21.0.7)

## Modpack context

234 mods. Key: Create 6.0.10, Sable (Valkyrien Skies physics), Terralith, Tectonic, Quark.

Removed from pack: `enhancedai`, `zombieapocalypseaddon`, `Smoothchunk`, `Chunk Activity Tracker`.

## Architecture (19 files, ~1200 lines)

Tick loop: `EntityTracker` → `NoiseAttraction.tickFootsteps` → `CreateIntegration` → (if grace over) `ZoneManager.migration` → `SpawnHandler` → `AmbientHandler` → `BlockBreaker` → `ZoneReconciler`

### EntityTracker — O(1) queries, NO world scans
Tracks FROM_HORDE zombies via EntityJoinLevel/EntityLeaveLevel/LivingDeath events. Rebalance every 40 ticks. `getZombieCount(ZonePos)` = HashMap.get.

### SpawnHandler
Cooldown=10 ticks. Max 8/cycle. Distance 24-56. Light 0-13. Types: Tank(Husk)=3%, Screamer=4%, Spitter=4%, Mutant=10%, Normal=79%. Backoff after 5 fails — waits 5 sec.

### ZoneManager
Forest/Jungle/DarkForest/Mangrove=90% infested. Others=55%. Caps: Dense=55, Forest=40, Open=24, Sparse=14, Minimal=6, Cave=35. First wave ≤24. Migration: 28%/400 ticks, always ≥1 stays. Drowned: 1/8→land zombie.

### HordeMind
LivingChangeTargetEvent: 3×3 zone (5×5 Screamer). Screamers: no LOS, play sound, Speed III (5s) to alerted. Static `propagating` flag prevents recursion.

### Block breaking — two systems
1. MixinZombie.customServerAiStep: every 5 ticks, wood/glass/fences/trapdoors. 33% plays break sound.
2. BlockBreaker.tick(): every 20 ticks, doors/fences/trapdoors/glass. 100% instant.

### NoiseAttraction
Events: break, place, jump(5), fall(dist×3), damage(8), container(3), footsteps(sprint=4, walk=2). Pull: `random(noise/2+1)` from each neighbor.

### SpawnBlocker
PositionCheck: blocks only NATURAL. All other types pass. EntityJoinLevel: speed-boosts non-horde, never cancels.

### CreateIntegration
Tracks AbstractContraptionEntity via events. Every 40 ticks: noise = 3 + speed×5 + blocks/20. Stalled→silent.

### Mixins
- MixinZombie: `isSunSensitive→false`, block breaking in `customServerAiStep`
- MixinZombieSpawn: `finalizeSpawn` — persistence, canBreakDoors, stepHeight=1.0, Speed II, per-type attrs

## Commands

`/horde zone` — counter + horde loaded + ALL zombies (one-time scan)
`/horde nearby` — 3×3 grid, single msg
`/horde total` — global stats
`/horde debug` — toggle chat spam
`/horde grace end` — force end grace
`/horde reload` — reload config

## Key config values

zoneSizeBlocks=48, migrationInterval=400, migrationFraction=0.28
capDense=55, capForest=40, capOpen=24, capSparse=14, capMinimal=6, capCave=35
maxSpawnPerZone=70, spawnCooldown=10, maxSpawnPerCycle=8
minSpawnDistance=24, maxSpawnDistance=56, maxSpawnLight=13
zoneInfestedChance=0.55, zombieSpeed=0.43, mutantSpeed=0.52, mutantChance=0.10

## Modpack configs we changed

| File | What | Why |
|------|------|-----|
| enhancedmobcap-common.toml | monsterCap=0.4, water=0.2 | HordeEngine handles zombies |
| zombieawareness/Features.toml | All→false | Keep sounds, disable CPU logic |
| Weather2/Tornado.toml | Blacklist mode | Tornado preserves chests/beds |
| tornadophysics-common.toml | Expanded destroyable | Wood+glass+wool |
| terrablender.toml | region_size=2 | Smaller biomes |
| tectonic.json | temp_offset=-0.18 | Fewer jungles |
| quark-common.toml | Toretoise=false | 3% TPS saved |
| modernfix-mixins.properties | Thread/classinfo tweaks | Less memory |

## Debug

Spark: `/spark profiler start` → play 60s → stop → link. Or `/spark heapsummary` for memory.
Logs: `latest.log` — search `[HordeEngine]`, `Can't keep up`, `Non-horde zombie joined`.

**Verified 5×: HordeEngine = 0% server tick time.** All world scans replaced by EntityTracker.
