# HordeEngine Apocalypse — Architecture & Config

## Modpack context

234 mods. NeoForge 1.21.1. Java 21. GeckoLib 4.8.4.
Key dependencies: Create 6.0.10, GeckoLib, NeoForge.

**Must remove from pack:** `enhancedai`, `zombieapocalypseaddon`, `smoothchunk`, `chunkactivitytracker`, `e4mc`.

## Architecture

```
Tick loop:
  EntityTracker.tick → NoiseAttraction.tickFootsteps → CreateIntegration.tick
  → (if grace over) ZoneManager.tickMigration → HordeSpawner.tick
  → AmbientHandler.tick → ZoneReconciler.tick
  → NoiseAttraction.tickLightAttraction → DepletionSystem.tick
```

### Entity hierarchy

```
Zombie
 └── HordeZombieEntity        ← water, drowning, swim speed, persistence
      ├── RunnerEntity        ← 84% spawn, 15 goals
      ├── AlphaEntity         ←  4% spawn, HP 40, scent trail, scream
      └── CrawlerEntity       ←  4% spawn, HP 18, speed 0.55, low profile
```

### AI Goals (16 goals, registered per entity type)

| Priority | Goal | Runner | Alpha | Crawler |
|----------|------|--------|-------|---------|
| 0 | SprintGoal | ✅ | ✅ | ✅ |
| 1 | TryExitWaterGoal | ✅ | ✅ | ✅ |
| 2 | HordeJumpGoal | ✅ | ✅ | — |
| 3 | StuckFixGoal | ✅ | ✅ | ✅ |
| 4 | BreakBlockGoal | ✅ | ✅ | ✅ |
| 5 | ClimbClimbableGoal | ✅ | ✅ | ✅ |
| 6 | ParkourGoal | ✅ | ✅ | — |
| 7 | AvoidExplosionGoal | ✅ | ✅ | ✅ |
| 8 | MineTowardsTargetGoal | — | ✅ | — |
| 9 | BreakVehicleGoal | ✅ | ✅ | ✅ |
| 10 | ScreamAlertGoal | — | ✅ | — |
| 11 | BreakGlassGoal | ✅ | ✅ | ✅ |
| 12 | OpenDoorsGoal | — | ✅ | — |
| 13 | MeleeAttackGoal | ✅ | ✅ | ✅ |
| 14 | ScentTargetGoal | ✅ | ✅ | ✅ |
| 15 | FollowHordeGoal | ✅ | ✅ | ✅ |
| 16 | RandomStrollGoal | ✅ | ✅ | ✅ |
| 17 | LookAtPlayer + LookAround | ✅ | ✅ | ✅ |
| Target | NearestAttackableTarget(Player) | ✅ | ✅ | ✅ |

### Water mechanics (28YL canon)

- **Shallow (1-2 blocks):** SWIM_SPEED 1.3×, `getWaterSlowDown()=1.0f` → chase target
- **Deep (3+ blocks):** accelerated drowning (air drain ×2), drowning damage 2.0
- **No target:** TryExitWaterGoal seeks shore (spiral scan, radius 12)
- **Stuck >4 sec:** breaks blocks in front, emergency jumps
- **Out of water:** instant air reset

Physiology in `HordeZombieEntity.aiStep()`. Navigation fully delegated to `TryExitWaterGoal`.

### 6 Mixins

| Mixin | Target | Purpose |
|-------|--------|---------|
| MixinPlaySound | Level.playSound() | Sound → scent (noise attracts) |
| MixinMob | Mob.setTarget() | Chain reaction: target acquired → scent burst (horde-only filter) |
| MixinRecomputePath | PathNavigation.shouldRecomputePath() | Skip path recompute if recently alerted |
| MixinPreventWandering | RandomStrollGoal.canUse() | Suppress wandering if recently alerted |
| MixinMobPersistence | Mob (tick, checkDespawn, NBT) | 1-hour persistence, prevents despawning |
| MixinPlayer | Player (tick, getDestroySpeed) | Passive scent, blood scent, waypoints |

Shared cooldown check via `ZAUtil.isRecentlyAlerted()`.

### Scent system

- `ScentEntity`: floating invisible marker. `noPhysics=true`. MAX_AGE=2400 (2 min)
- Rain: decay ×3 (washes scent away). Strength = peak × age/2400
- `ScentTargetGoal`: finds strongest scent within 64 blocks. Verifies reachability via `createPath().canReach()`. No-path cooldown 5 sec

### Noise → Spawn pipeline

```
Player action → NoiseAttraction event → NoiseAccumulator.add(noise)
→ every 5 sec: pull zombies from neighbor zones + spawn ScentEntity
→ every 100 ticks (5 sec): HordeSpawner checks zoneNoise
→ spawn count × (1 + noise/100), capped at 3×
```

### Spawn types (SpawnProcess.createZombie)

| Roll | Type | Entity | HP | Speed | Special |
|------|------|--------|-----|-------|---------|
| ≤4% | Alpha | AlphaEntity | 40 | 0.50 | MineTowardsTarget, ScreamAlert, OpenDoors, scent trail |
| ≤12% | Fast | RunnerEntity | 20 | 0.65 | Pure speed variant |
| ≤16% | Crawler | CrawlerEntity | 18 | 0.55 | Low profile, no jump/parkour |
| 84% | Default | RunnerEntity | 20 | Config.zombieSpeed | Standard infected |

### Infection system (single source of truth: InfectionMechanic)

- Trigger: direct melee hit from horde zombie on player
- 35 seconds to death. Phases: Blindness+Confusion (0-35s) → Slowness II+Weakness (10-35s) → Death
- Death: `HordeDamageTypes.infection()` → player becomes `RunnerEntity` with `FROM_HORDE` tag
- Contagion: direct hit only. Not airborne.
- Cleanup: respawn, logout, server stopping

### Block breaking — three systems

1. **BreakBlockGoal** (per-entity): doors, blocks in front. Tier scales with world days:
   - Day 1-3: tier 1 (doors, glass, dirt)
   - Day 4-7: tier 2 (+wood, fences)
   - Day 8-14: tier 3 (+stone, iron bars)
   - Day 15+: tier 4 (anything except unbreakable, hardness ≤75)
2. **BreakGlassGoal** (per-entity): sees player through glass → breaks it (10 ticks progress). Per-instance progress map.
3. **BlockBreaker** (zone-level): runs every `breakIntervalTicks` when density > `breakDensityThreshold`. Siege mode: at 40%+ density, breaks stone/cobble.

### EntityTracker — O(1) queries

Tracks `FROM_HORDE` zombies via `EntityJoinLevelEvent` / `EntityLeaveLevelEvent` / `LivingDeathEvent`.
Rebalance every 40 ticks. `getZombieCount(ZonePos)` = `HashMap.get()`. Zero world scans.

### Zone system

- `ZonePos`: immutable record (x, z). `fromBlock()` uses `Math.floorDiv` for correct negative coords
- `ZoneData`: zombieCount, drownedCount, lightLevel, grudge, timestamps. NBT persistence
- `ZoneManager`: biome-aware caps (Dense=55, Forest=40, Open=24, Sparse=14, Minimal=6, Cave=35)
- `ZoneSavedData`: world-save persistence across server restarts

### Key config values

```
zoneSizeBlocks=48, zoneSizeChunks=3
migrationInterval=400 (20s), migrationFraction=0.28
capDense=55, capForest=40, capOpen=24, capSparse=14, capMinimal=6, capCave=35
maxSpawnPerZone=70, spawnCooldown=10 (0.5s), maxSpawnPerCycle=8
minSpawnDistance=24, maxSpawnDistance=56, minSpawnLight=0, maxSpawnLight=15
zoneInfestedChance=0.55, zombieSpeed=0.50
gracePeriodTicks=24000 (20 min), ambientTickInterval=60 (3s), ambientScanRadius=3
dayScaling: scaleDay7=1.3, scaleDay14=1.7, scaleDay30=2.2
evolution (disabled by default): evolutionHealthFactor=0.3, evolutionSpeedFactor=0.1
```

## Build

```powershell
cd C:\Users\dsd\Desktop\HordeEngine_v2
$javaBin = "C:\Users\dsd\AppData\Roaming\PrismLauncher\java\java-runtime-delta\bin\java.exe"
& $javaBin -Xmx64m -Xms64m "-Dorg.gradle.appname=gradlew" -classpath "gradle/wrapper/gradle-wrapper.jar" org.gradle.wrapper.GradleWrapperMain clean build --no-daemon
```

## Deploy

```powershell
Copy-Item "C:\Users\dsd\Desktop\HordeEngine_v2\build\libs\HordeEngine-1.1.0.jar" "C:\Users\dsd\AppData\Roaming\PrismLauncher\instances\All of Create - Aeronautics\minecraft\mods\HordeEngine-1.1.0.jar" -Force
```

## Compiled .class paths

```
build/classes/java/main/hordeengine/          ← javap decompilation
build/libs/HordeEngine-1.1.0.jar              ← deploy artifact
```

## Debug

Spark: `/spark profiler start` → play 60s → stop → link. `/spark heapsummary` for memory.
Logs: `latest.log` — search `[HordeEngine]`, `Can't keep up`.
HordeCommand: `/horde zone`, `/horde nearby`, `/horde total`, `/horde debug`, `/horde grace end`, `/horde reload`.
