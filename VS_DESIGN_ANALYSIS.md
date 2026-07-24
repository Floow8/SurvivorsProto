# Vampire-Survivors Design Analysis (reference brief)

> **See also:** `vampire-survivors-reference.md` — the complete systems & data
> reference (stats, PowerUps, weapons/evolutions, Arcanas, enemy/wave mechanics,
> stages, characters). This file is the *why*; that file is the *what*.

The reference this prototype is chasing. First section is the user's own
write-up (from a YouTube deep-dive they made); second section is the deeper
"why it's good" that matters most for our build; third is what it means for us.

Reference screenshots live in the user's `Pictures/Screenshots` (2026-07-24):
sparse slow opening (0:12–0:30, a handful of bats), Magic Wand auto-firing at the
nearest enemy, flat 2D white damage numbers, flat 2D red HP bars under enemies,
blue XP gems, a Level Up! panel (Magic Wand "Fires at the nearest enemy", Santa
Water "Generates damaging zones", Laurel "Shields from damage when active"),
enemy types swapping over time (bats → zombies/ogres by ~1:00), GAME OVER screen.

---

## 1. What the game is / does well (user's write-up)

**What it is.** The striking thing on first launch is the *lack* of controls:
you can only move up/down/left/right. No attack button, no dash. Every ability
is on an automatic timer, triggered by your cooldown stat. The game spawns
endless hordes of mostly 2–3 monster types; your automatic abilities kill them;
they drop gems that give XP and level you up. Each level-up offers up to 4
choices (how many depends on your Luck stat and unlocked items). Items unlock via
achievements (survive X minutes, etc.), as do the survivors you can pick.
You also collect coins kept across death, spent on unlocking characters and
permanent power-ups. The game periodically swaps monster types and spawns
bosses/elites; occasional events (bat fly-bys, plant waves that circle you in),
but enemies mostly don't vary much.

**What makes it great (TL;DR list):**

- Abilities are limited, timed, and mostly weak on their own — this avoids
  handing the player too much power right away.
- Mixing/matching ability types to cover weaknesses, or stacking to chase an
  ultimate power spike, is fun across many runs.
- Great direct visual+audio feedback for hits and pickups — a constant dopamine
  stream.
- Achievements, unlocks, and rogue-lite permanent power-ups pull you back in.
- A well-balanced power curve that never gives too much nor gets too
  frustrating — winning and losing both feel just in reach at once. Hard to pull
  off, and this is only early access.
- Good fundamentals: surprising elements (e.g. an exploding enemy) are
  introduced early when there's little to lose, minimizing unexpected
  frustration without feeling patronizing.
- Super simple controls: no held attack button. Very accessible, including to
  players with disabilities.
- Length and pacing sit in a nice spot. Could be a little faster at the start,
  but overall length is good.

---

## 2. The deeper "why it's good" (matters most for us)

These are the drivers under the juice — the things to protect in our build:

1. **Agency through movement/positioning.** There *is* skill: how you move and
   position (the closest thing to "aiming" here) always makes you feel you have
   impact on the run, even though the single biggest variable is which items you
   chose. Never remove that felt agency.

2. **Progression is learning a flowchart, not balance.** The sense of "getting
   better" is really you learning which items are good vs garbage and in what
   order to take them. Within a couple hours you can predict good/bad picks
   fairly accurately. **The game is not balanced — and it does not need to be.**
   It's fun this way; that's not a flaw. Don't chase symmetric balance.

3. **Hoarding + crowd control is intrinsically satisfying.** Amassing a swarm and
   then mowing it down triggers something primal in the player's brain. This —
   not just numbers and juice — is a core reason people buy in after seeing a
   single minute. Protect the fantasy of being surrounded and cutting through it.

---

## 3. Implications for THIS prototype

- **Player never aims. Auto-target is correct.** The first weapon (Magic Wand)
  auto-fires at the nearest enemy. Input is movement only. (An earlier detour to
  manual mouse-aim was a mistake and was reverted.)
- **Skill lives in positioning** — kiting, funnelling, and not getting boxed in.
  Keep enemy speed well under the player's so positioning stays the lever.
- **Slow, sparse opening on purpose**, ramping up. Movement feels gentle early.
- **Feedback first**: flat 2D screen-facing damage numbers and HP bars (never
  3D/tilted), plus pickup pops. This is the dopamine layer.
- **Multiple monster types swapped in over time**, VS-style, plus later
  elites/bosses.
- **Upgrades need not be balanced.** A spread of clearly-different power items
  (some strong, some niche) beats four symmetric stat bumps — the fun is the
  flowchart.
- **Crowd control is the point** — spawn enough that being surrounded and
  clearing it feels good; lean on this rather than raw stat inflation.

### Current demo goal

A **5-minute** run: auto-weapons, multiple monster types spawning and swapping
over time, XP → level-up → power-up choices, slow sparse start that ramps into
crowds. See `GAME_DESIGN.md` for the concrete numbers/formulas.
