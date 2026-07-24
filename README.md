# SurvivorsProto

A Vampire-Survivors-style prototype built in **Unreal Engine 5.8, Blueprint-only** —
and built almost entirely by an AI agent (Claude Code) driving the Unreal Editor
through its built-in MCP server, iterating live against playtest feedback.

## The game

10-minute survival run in a swamp forest. You only move — every weapon fires
itself. Kill the swarm, collect gems, level up, pick perks and weapons, survive
the ramp-up and the elites at 3:00 / 6:00 / 9:00.

- **10 monster tiers** unlock one per minute (bats → zombies → skeletons →
  ghosts → mudmen → werewolves → giant bats → mantichanas → mummies → brutes),
  ~200–300 alive across the map at all times, stats baked at spawn
- **Weapons** (all automatic): Magic Wand (homing fireballs, multi-shot fan),
  Garlic aura (growing damage ring), Whip (East/West slash zones), Knife
  (North/South dart volleys) — ranked up through level-up picks
- **Perk draft**: every level-up offers 4 distinct picks from a growing pool

## Documents

| File | What |
|---|---|
| `GAME_DESIGN.md` | The agreed design spec — the build must match this file |
| `VS_DESIGN_ANALYSIS.md` | Why: the design pillars distilled from Vampire Survivors |
| `vampire-survivors-reference.md` | Reference systems data (wiki snapshot) |
| `TESTPLAN.md` | The automated smoke test — log-marker assertions run headless after every change batch |
| `survivors_game_blueprint_guide.md`, `understanding_unreal.md` | Early learning notes |

## Project layout

```
Content/
  Survivors/          Game code (Blueprint)
    BP_Enemy          One enemy class; 11 data-driven kinds (InitEnemy)
    BP_Spawner        Population-model spawner (alive-count target, no waves)
    BP_PlayerStats    Run state: health/XP/level/perks (component on the player)
    BP_WeaponComponent  All weapon logic (wand/garlic/whip/knife)
    BP_Projectile     Wand fireball (swept-trace hits)
    BP_XPGem, BP_DamageText, BP_PlayerHPBar
    Enemies/          Per-kind materials
    FX/               Weapon FX + knife projectile
    UI/               HUD, level-up panel, bars (screen-space 2D)
  ThirdPerson/, Characters/  Engine template content
```

## Notes

- Requires Unreal Engine 5.8 (uses the experimental in-editor MCP server for
  the AI-driven workflow, but the project itself is plain Blueprint).
- Test discipline: see `TESTPLAN.md` — permanent log markers
  (`BOLT-HIT`, `ENEMY-DIE`, `GEM-XP`, `LVL-UP`) + a 30-second headless PIE run
  gate every change.
