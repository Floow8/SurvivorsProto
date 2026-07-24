# SurvivorsProto — Gameplay Design (agreed 2026-07-24)

The single source of truth for the **first 10 minutes** of the game. Agreed
step-by-step with the user; the build must match this file. Balance values live
here — change them here first, then in the editor.

Companion docs: `VS_DESIGN_ANALYSIS.md` (why these choices — the three pillars),
`vampire-survivors-reference.md` (reference systems data).

---

## A. Controls & camera

1. **Input = movement only** (ZQSD / arrows / stick). No aim, no attack button.
   The mouse is only used in menus and level-up picks. (Aim logic is preserved
   in docs for a future manually-aimed weapon — never on the starter weapon.)
2. **Fixed top-down camera**, boom arm **1700** (pulled back from 1200 to read
   the crowd), pitch −70, no rotation inheritance.

## B. The clock

3. Run = **10:00 (600 s)**. Survive to win, death = game over, timer on HUD.
4. `T` = minutes elapsed (float 0→10) drives all scaling below.

## C. Monsters — the ladder

One `BP_Enemy`, data-driven kinds via `InitEnemy(Kind, T)`. Kind = color +
size + stats; behaviour is shared (walk at player, stop at physical touch,
contact damage on overlap only — **no damage without physical touch**;
ranged/spell monsters are a later feature).

5. **10 tiers**, level 1 → 10:

| Kind | Name | HP base | Speed | Dmg | XP | Scale | Color |
|---|---|---|---|---|---|---|---|
| 0 | Bat | 12 | 130 | 5 | 2 | 0.30 | dark gray |
| 1 | Zombie | 25 | 90 | 8 | 3 | 0.45 | sickly green |
| 2 | Skeleton | 35 | 115 | 9 | 4 | 0.50 | bone white |
| 3 | Ghost | 28 | 160 | 8 | 5 | 0.40 | pale blue |
| 4 | Mudman | 65 | 70 | 12 | 6 | 0.60 | brown |
| 5 | Werewolf | 55 | 175 | 14 | 8 | 0.55 | slate |
| 6 | Giant Bat | 85 | 120 | 12 | 10 | 0.70 | purple |
| 7 | Mantichana | 130 | 95 | 16 | 14 | 0.80 | orange |
| 8 | Big Mummy | 210 | 65 | 20 | 20 | 0.95 | tan |
| 9 | Swamp Brute | 160 | 110 | 18 | 16 | 0.85 | dark green |

   Per-spawn scaling (baked at spawn): HP ×(1+0.3·T), Dmg ×(1+0.15·T),
   Speed +6·T, XP ×(1+0.25·T). Speed always stays far below the player's.

6. **Minute ladder:** minute 0 = **bats only**. Each new minute unlocks the
   next tier into the pool (minute 1 → Zombie, minute 2 → Skeleton, … minute
   9 → Swamp Brute). The pool is **cumulative** — bats never stop.
7. **Weighted to newest:** during a tier's debut minute it takes **~40%** of
   spawns; the rest is uniform across everything unlocked. Every minute
   visibly announces its new monster.

## D. Population model

8. **Alive-count target, not spawn rate:** keep `200 + 10×minute` monsters
   alive (200 at start → **300 at minute 10**), spawned **900–3500** units
   from the player (mostly off-screen). Deaths → replacements spawn far away.
   Refill capped at 40 spawns/tick to avoid hitches. No despawning.
9. Player contact damage gated by **0.5 s i-frames** (`TakeHit`).

## E. Weapons — all automatic

10. **Magic Wand** (starter): every 0.8 s (before upgrades) fires a visible
    bolt at the **nearest** enemy. Damage 8–12 (rolled per shot ×Might).
    Bolt spawns ~80 units in front of the character so point-blank enemies
    can't swallow it invisibly. Multi-projectile upgrades fan at ±12°.
11. **Garlic Aura** (unlock via level-up): pulse every 0.75 s around the
    player; radius 250+30/stack, damage (3+2/stack)×Might-factor.
12. More weapons (zones, orbits…) — later pass, out of this spec.

## F. XP loop

13. Every kill drops a **visible gem** (value = the monster's baked XP).
    Gems persist 60 s, magnet-pull at **120** base range, collected at ~60–90.
14. XP curve: `XPRequired(L) = 10 + 8·(L−1) + 2·(L−1)²`. Target **15–20
    levels** over the 10 minutes, fast early. Extra level-ups queue.
15. **Level-up:** pause, offer **4 distinct random picks from 7** (distinctness
    guaranteed per panel; repeats across levels allowed — stacking is the
    build system):

| # | Perk | Effect |
|---|---|---|
| 0 | Damage | ×1.25 |
| 1 | Attack Speed | interval ×0.85 (floor 0.25 s) |
| 2 | Move Speed | ×1.08 |
| 3 | Max Health | +25, heal 25 |
| 4 | Garlic Aura | +1 stack (grants on first pick) |
| 5 | Projectile | +1 (cap 5) |
| 6 | Magnet | ×1.4 |

## G. Set pieces

16. **Elites** at **3:00, 6:00, 9:00** — huge red variant (scale 1.5), never
    despawns (teleports back if outrun). Escalating: HP `400×(1+T)` (≈1600 /
    2800 / 4000), Dmg `25+2·T`, XP `100+40·T` (a level-up-sized burst).
17. Minutes 8–10 = barely-survivable saturation; winning is positioning, not
    stats (pillar #1/#3).

## H. Feedback layer (all flat 2D, always)

18. Damage numbers and HP bars are **screen-space** — never rotated by enemy,
    camera, or facing. Player bar sits under the character (~character
    width); enemy bars appear on first hit (60×8), hidden on the killing blow
    (no one-frame flash). Player base speed **340**.

## Player base stats

| Stat | Value |
|---|---|
| MaxHealth | 100 |
| MoveSpeed | 340 |
| Damage | 10 (±20% per shot) |
| FireInterval | 0.8 s |
| ProjectileCount | 1 |
| MagnetRadius | 120 |
| IFrames | 0.5 s |

## Performance notes (when fps drops, in order)

1. Enemy Tick Interval → 0.05 (20 Hz)
2. Object pooling for enemies/gems/projectiles
3. Instanced meshes for rendering

## Agreed additions (backlog, in priority order)

1. **Wand fire range** — the wand only engages when the nearest enemy is
   within **900 units**; no shots before first contact. ✅ built
2. **Garlic aura visual** — colored disc under the player; each stack grows it
   and shifts the color green→red. ✅ built
3. **Chests** — rare drop (~2–3%) from normal monsters: pickup triggers a
   **casino-roulette (777) spin** that lands on a random perk or new weapon.
   Elites always drop a **high-tier chest = 3 random perks** and give a
   much larger XP burst. (v1: instant grant + popup; roulette animation after.)
4. **Weapon system** — VS-style **base weapons** in slots (6 eventually,
   4 in v1), each with per-rank upgrades; **HUD shows weapon slots + rank
   pips**. Build order: slot/rank refactor first, then weapons one by one,
   smoke-tested each. Specs (user-defined, VS-derived):

| Weapon | Trigger | Pattern | Rank-up | Status |
|---|---|---|---|---|
| Magic Wand | 0.8 s | bolt at nearest enemy within 900 | +1 bolt (fan ±12°), cap 5 | ✅ built — visual TODO: should read as a **fireball/spell** (glowing, trailing), not a bullet |
| Garlic | 0.75 s pulse | aura around player, radius 250+30/rank | +radius +damage, disc grows & shifts color | ✅ built |
| Whip | 1.2 s | instant slash zone at short reach, **always East** (screen-right, fixed axis — not facing-relative); hits all enemies in it | **+damage and +range per rank**; rank 2 adds the **West** slash; cap 8 | ✅ built |
| Knife | fast (0.6 s) | **no targeting** — flies straight **North** (screen-up; fixed world axis, user-confirmed). Fast, low damage | rank 1→3: +1 knife North (parallel, ±45 lateral offsets); ranks 4–5: +1 knife **South** each — cap 5 = 3 North + 2 South | ✅ built |
| King Bible | continuous | book(s) **orbiting** the character slowly at a fixed radius; damage on contact with an enemy (with per-target hit cooldown so one pass ≠ multi-hit) | +1 book per rank, **evenly spaced** around the circle (books share the orbit, offset by 360°/count); orbit radius grows a little per rank | ⬜ spec'd |
| *(more base weapons as user defines them)* | | | | |

**Passives** (occupy perk pool, not weapon slots):

| Passive | Effect | Per-rank | Status |
|---|---|---|---|
| Bracelet | **Armor** — flat reduction on every hit taken: `damage taken = max(1, damage − 2×ArmorRank)` | +1 Armor rank (−2 dmg/hit), no cap for now | ⬜ spec'd — needs `Armor` stat added to `TakeHit` |

## Status

- Built & working: ladder kinds w/ colors, population spawner, wand, garlic,
  gems+magnet, 4-distinct level-ups, 2D bars/numbers, elite #1, game over.
- This spec's deltas being applied: 10:00 timer (was 5:00), population ramp
  200→300, weighted-newest mix, elites ×3, camera 1700, wand forward-offset.
