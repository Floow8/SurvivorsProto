# SurvivorsProto — Automated Smoke Test

The regression net. Run after **every** batch of Blueprint changes, before
handing the build to the player. First full pass: 2026-07-24 12:14.

## How it works

The gameplay Blueprints contain permanent **log markers** — `PrintString`
nodes with *Print to Log* only (invisible in-game, negligible cost):

| Marker | Where | Proves |
|---|---|---|
| `BOLT-HIT` | `BP_Projectile` trace-hit branch | wand fires AND connects |
| `ENEMY-HIT` | `BP_Enemy` AnyDamage entry | damage actually REACHES an enemy (BOLT-HIT without ENEMY-HIT = the 2026-07-27 null-target regression) |
| `ENEMY-DIE` | `BP_Enemy` AnyDamage death branch | damage → death chain |
| `GEM-XP` | `BP_PlayerStats.AddXP` entry | gem dropped, survived, got picked up |
| `LVL-UP` | `BP_PlayerStats.AddXP` level loop | XP curve → level-up loop |

## Procedure (via MCP, ~60s)

1. `EditorAppToolset.IsPIERunning` → must be **false** (never test a stale
   world; PIE actor names must restart at `_C_0`).
2. `EditorAppToolset.StartPIE` (`warmupSeconds` 35, in-viewport). A "timed
   out waiting" error usually means slow init — re-check `IsPIERunning`.
3. Wait ~30 s (player stands still; systems run on their own).
4. `LogsToolset.GetLogEntries` category `LogBlueprintUserMessages`,
   pattern `BOLT-HIT|ENEMY-DIE|GEM-XP|LVL-UP`.
5. `EditorAppToolset.StopPIE`.

## Pass criteria (30 s, fresh run, no input)

- `BOLT-HIT` ≥ 10 — weapon connects (silence = the wand is broken)
- `ENEMY-HIT` ≥ `BOLT-HIT` count — hits must actually damage someone; a
  stream of BOLT-HIT with no ENEMY-HIT means ApplyDamage lost its target
- `ENEMY-DIE` ≥ 8 — kill chain intact
- `GEM-XP` ≥ 3, first one **later than ~5 s** — gems have a ground phase
  (instant `GEM-XP` after every `ENEMY-DIE` = insta-vacuum regression)
- `LVL-UP` ≥ 1 — XP curve alive
- Also scan pattern `(?i)Accessed None|Infinite loop` → must be empty.

## Hard-won rules (violating these caused every regression so far)

- **Never trust a graph write** — `read_graph_dsl` after every
  `write_graph_dsl`, then still verify by *running* (this file).
- **Mid-PIE compiles don't reach the running world.** Restart PIE before
  trusting any test result. Continuing `_C_##` numbering = stale world.
- Full gotcha list: memory `unreal-mcp-api-gotchas` (implicit-self pin
  miswiring, pure-node re-evaluation, loop mangling, CDO resets…).

## Baseline result (2026-07-24)

22 s window: 20 BOLT-HIT, 20 ENEMY-DIE, 6 GEM-XP (first at ~17 s), 1 LVL-UP.
