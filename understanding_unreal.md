# Understanding Unreal — the mental model behind this project

A companion to `survivors_game_blueprint_guide.md`. That guide tells you *which
buttons to click*. This one explains *why the engine is shaped this way*, so the
next phases stop feeling like magic incantations.

---

## 1. The ownership chain: who runs the game?

Unreal splits "the game" across several objects that each own one job. When
something doesn't work, the first question is always *which of these should own
this logic?*

```
GameMode          rules of the match — win/lose, score, what to spawn, respawn
  └── PlayerController   the human's intent — input, camera target, UI ownership
        └── Pawn / Character   the body in the world — mesh, collision, movement
              └── Components   the parts bolted on (camera, mesh, movement)
GameState / PlayerState   replicated shared facts (score, timer) if multiplayer
```

Rules of thumb for a survivors game:

- **Run timer, difficulty curve, "you died" → GameMode.** It exists once and
  outlives your character.
- **XP total, level, chosen upgrades → PlayerState** (or the Character if you
  never go multiplayer; PlayerState is the tidier home).
- **Health, movement, weapon firing → Character.**
- **Which enemies exist right now → a spawner Actor** (what we built) or the
  GameMode.

Your project currently uses `BP_ThirdPersonGameMode` and
`BP_ThirdPersonPlayerController` from the template. Phase 5 will put the run
timer and death handling in the GameMode.

---

## 2. Actor lifecycle — when your code runs

An Actor is anything that can be placed in a level. The events fire in this
order:

1. **Construction Script** — runs in the *editor* every time you move or edit the
   actor, and once at spawn. Use for procedural setup (e.g. building a mesh from
   a variable). It runs before the game starts, so it cannot see other actors
   reliably.
2. **BeginPlay** — the game has started, the world is populated, other actors
   exist. This is where you start timers, find the player, bind events. Our
   `BP_Spawner` starts its spawn timer here.
3. **Tick** — every frame, with `DeltaSeconds` telling you how long the last
   frame took. Our `BP_Enemy` chases the player here.
4. **EndPlay / Destroyed** — cleanup.

### Why `DeltaSeconds` matters

`Tick` fires as fast as the machine allows — maybe 60×/sec, maybe 144×/sec. If
you wrote `location = location + 5` per tick, the enemy would move nearly three
times faster on a fast PC. Multiplying by `DeltaSeconds` converts
"units per frame" into **units per second**, which is framerate-independent.
That is why `Speed = 200` means "200 units/second" and the enemy behaves the
same everywhere.

### Tick vs Timers — the single most useful optimisation

`Tick` is the expensive default. A timer is a scheduled callback that costs
nothing between firings.

- Needs to happen *smoothly, every frame*? → Tick. (Enemy movement.)
- Needs to happen *periodically*? → Timer. (Spawning, auto-firing a weapon,
  regenerating health.)

We used `Set Timer by Function Name` on the spawner precisely so it isn't
checking a stopwatch 60 times a second. For Phase 3's auto-firing weapon, use a
timer too — not a Tick with an accumulator.

**When you have 300 enemies, Tick becomes your bottleneck.** See §6.

---

## 3. Actors vs Components — "is-a" vs "has-a"

- An **Actor** is a thing in the world with a transform (location/rotation/scale).
- A **Component** is a capability bolted onto an Actor. It usually cannot exist
  on its own.

Your `BP_Enemy` *is an* Actor and *has a* sphere mesh component. Your character
*is a* Character and *has a* SpringArm, a Camera, a SkeletalMesh, and a
CharacterMovementComponent.

The **root component** matters: moving the Actor moves the root, and everything
attached follows. Scaling the root scales children too — a common source of
"why is my collision the wrong size?"

### The SpringArm lesson from Phase 1

The camera sits on a SpringArm ("CameraBoom") — an invisible pole that also
pulls the camera in when a wall gets between it and the player. Its behaviour is
governed by flags that interact in a way the guide glossed over:

- `bUsePawnControlRotation` — should the arm follow where the *controller* is
  looking (i.e. the mouse)? For third person: yes. For fixed top-down: **no**.
- `bInheritPitch / bInheritYaw / bInheritRoll` — should the arm follow the
  *actor's* rotation? These still apply when `bUsePawnControlRotation` is off.

Turning off only the first flag is not enough. The character rotates to face its
movement direction, the arm inherits that yaw, and the camera spins as you walk
in circles. **All four must be off** for a truly fixed overhead camera. That is
why the implementation set more than the guide listed.

---

## 4. Collision — the three questions

Every collision problem is one of these three, and they are configured
separately per component:

1. **Is collision even on?** (`Collision Enabled`: No Collision / Query Only /
   Physics Only / Query and Physics)
2. **What am I?** (`Object Type`: Pawn, WorldStatic, WorldDynamic…)
3. **What do I do about each other type?** (`Ignore` / `Overlap` / `Block`)

Two objects **block** only if *both* say Block. They generate an **overlap**
event only if both say at least Overlap and both have
`Generate Overlap Events` ticked. This mutual-consent rule is behind almost
every "my overlap event never fires" bug.

For Phase 5 (enemy touches you → lose health), you want the enemy set to
**Overlap** the Pawn channel, not Block — otherwise enemies shove you around
instead of damaging you.

### `Sweep` on Set Actor Location

Our enemy teleports itself a small step each frame via `Set Actor Location`.
With `Sweep` off, it would pass straight through walls. With `Sweep` **on**, the
engine traces the path and stops it at the first blocking hit. That checkbox is
the difference between "moves" and "moves *through the world*".

Note this is deliberately *not* real pathfinding — no NavMesh, no AI Controller.
For a survivors game where enemies beeline at you, straight-line steering is both
correct and vastly cheaper.

---

## 5. Spawning and class references

`Spawn Actor from Class` needs a **class**, not an instance — the blueprint
*recipe*, not a thing in the world. In asset paths this is why you'll see the
`_C` suffix: `/Game/Survivors/BP_Enemy.BP_Enemy_C` is the generated class,
`BP_Enemy.BP_Enemy` is the blueprint asset that produced it.

**Collision Handling Override** decides what happens if the spawn point is
already occupied:

- `Always Spawn, Ignore Collisions` — never fails; things may interpenetrate.
- `Try To Adjust Location, But Always Spawn` — nudges out of the way, still
  guaranteed. Usually the right choice for enemy spawns.
- `Do Not Spawn` — silently returns null. This is why spawns "mysteriously stop
  working" when the player stands in the spawn ring.

Our spawner picks a random direction, normalises it to length 1, and multiplies
by `SpawnRadius` — placing enemies on a **ring** around the player rather than a
random box. A box can roll a point right on top of you; a ring guarantees a
minimum distance. `SpawnInterval` and `SpawnRadius` are exposed as
instance-editable variables, so you can select the spawner in the level and tune
them in the Details panel without touching the graph.

---

## 6. Performance — what actually breaks a survivors game

The genre's defining feature is *hundreds of enemies*, which collides head-on
with the engine's defaults. In rough order of impact:

1. **Blueprint Tick on every enemy.** 300 actors each running a Blueprint Tick is
   the classic wall. Mitigations, cheapest first: tick less often
   (`Tick Interval` 0.05 instead of every frame — enemies don't need 60Hz
   steering), then move the logic to a single manager actor that loops over all
   enemies, then C++.
2. **Draw calls.** 300 separate meshes = 300 draw calls. **Instanced Static
   Meshes** (or Niagara-rendered enemies) collapse those into one. This is the
   single biggest visual-scale unlock.
3. **Spawning cost.** Creating and destroying actors constantly causes hitching.
   **Object pooling** — keep a pool of dead enemies hidden and reuse them
   instead of `Destroy` / `Spawn` — is the standard fix.
4. **Collision queries.** Every overlap check between every projectile and every
   enemy is O(n×m). Keep collision shapes as simple spheres and disable any
   collision channel you don't need.

You do not need any of this yet. Build Phases 3–5 the simple way, then optimise
when the framerate actually drops — but know these exist so you don't design
yourself into a corner (e.g. don't give each enemy a complex skeletal mesh with
an AnimBlueprint if you plan on 300 of them).

---

## 7. Assets, levels, and saving

- `Content/` on disk = `/Game/` in path references. `/Game/Survivors/BP_Enemy`
  is `Content/Survivors/BP_Enemy.uasset`.
- **Blueprints must be Compiled, then Saved.** Compiling makes it runnable;
  saving writes the `.uasset`. Unsaved work lives only in the editor's memory
  and dies with the process.
- **Levels save separately from the assets they contain.** Placing an actor and
  saving the blueprint is not enough — `Ctrl+S` saves the *level*.
- This project uses **World Partition** (note the `WorldPartitionMiniMap` and
  `WorldDataLayers` actors), UE5's streaming system that loads only the parts of
  a big world near the camera. For a small arena it's harmless; just don't be
  surprised that the level is a folder of many small files rather than one big
  `.umap`.

---

## 8. Where the remaining phases land

| Phase | New objects | Key concept it teaches |
|---|---|---|
| 3 — Auto-firing weapon | `BP_Projectile`, timer on Character | Timers, `Get All Actors of Class` + nearest-target search, `ProjectileMovementComponent`, hit events |
| 4 — XP and level-ups | `BP_XPGem`, a UMG widget | Overlap pickups, UI (UMG), pausing the game, applying upgrades as variable changes |
| 5 — Health, death, timer | Health on Character, GameMode changes | Damage flow (`Apply Damage` / `Any Damage`), game-over state, escalating spawn difficulty |

The through-line: **Phase 3 makes it fun, Phase 4 makes it addictive, Phase 5
makes it a complete loop.** A 10-minute run with one weapon, upgrades and a
death screen is a genuine playable alpha.

### A design note on the spawner

Right now the spawner fires on a fixed interval forever. Phase 5 wants
difficulty to escalate. The cleanest change is to keep the timer but recompute
the interval as the run progresses (e.g. `interval = max(0.2, 2.0 - elapsed/60)`)
rather than adding more spawner actors. Keep one spawner; make it smarter.
