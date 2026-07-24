# Vampire Survivors — Complete Systems & Data Reference

**Scope note:** compiled July 2026 against the official wiki (vampire.survivors.wiki, run by Weirdgloop) at roughly patch v1.15 "The Lycaeum". Content counts move with almost every patch — treat every "there are N of X" figure as a snapshot, not a constant. Poncle ships free content updates frequently, so anything numeric here should be re-verified against the wiki before you build tooling on it.

---

## 1. The core loop

A run is a single stage attempt. You move; weapons fire automatically. Killing enemies drops experience gems; gems level you up; each level-up offers a choice of new weapons, weapon upgrades, or passive items. Survive to the stage time limit (30:00, 20:00, or 15:00 depending on stage) and the stage is "complete" — after which a Reaper spawns every minute and kills you (65,535 damage, ignores almost everything) unless Endless mode is on.

Between runs, gold buys permanent **PowerUps**, and achievements unlock characters, weapons, stages, and modes.

Three layers of progression stack:

| Layer | Persists | Examples |
|---|---|---|
| In-run | No | Weapon levels, passive items, Arcanas, stage pickups |
| Per-character permanent | Yes | Golden Eggs, morph unlocks |
| Account-wide permanent | Yes | PowerUps, unlocked characters/weapons/stages/Arcanas, Relics |

---

## 2. Player stats

There are 21 player stats. All apply to your character except **Seal**, which is a meta-toggle that removes content from the drop pool.

Most stats are multipliers layered on a base value. Each has an associated passive item, most have a PowerUp rank track, and Golden Eggs can raise any of them permanently for a single character. Characters carry their own base modifiers (e.g. one starts at 100 Max Health and 30 Magnet).

### Offensive

| Stat | What it does | Notes |
|---|---|---|
| **Might** | Multiplies damage dealt | The universal damage multiplier |
| **Area** | Scales weapon hitbox/AoE size | Some weapons ignore it entirely |
| **Speed** (Projectile Speed) | Projectile travel speed | At −100% projectiles are frozen in place; below −100% they fire *backwards* |
| **Duration** | How long timed weapon effects persist | Auras, orbits, lingering pools |
| **Cooldown** | Reduces interval between weapon activations | Hard-floored; Tragic Princess (III) can push past for its listed weapons |
| **Amount** | Extra projectiles per activation | The single most build-defining stat; PowerUp caps at rank 1 |

### Defensive / sustain

| Stat | What it does |
|---|---|
| **Max Health** | Health pool |
| **Armor** | Flat damage reduction per hit; also feeds Divine Bloodline (IX) damage |
| **Recovery** | HP regenerated per second |
| **Revival** | Extra lives. On death, consumes one and revives at half health |
| **Move Speed** | Character movement speed |

### Economy / meta

| Stat | What it does |
|---|---|
| **Magnet** | Pickup radius for gems and items |
| **Luck** | Raises good-RNG odds (chest tiers, coin bags, Orologion spawns) and *reduces* the chance of hostile wave events and trap frequency |
| **Growth** | XP multiplier |
| **Greed** | Gold multiplier. Also feeds base damage of Night Sword and Muramasa |
| **Curse** | Raises enemy Max Health, Move Speed, and spawn rate |

### Level-up control (PowerUp-only, no in-run item)

| Stat | What it does |
|---|---|
| **Reroll** | Re-draw the level-up / chest offer set |
| **Skip** | Decline the level-up entirely in exchange for XP |
| **Banish** | Permanently remove one option from the run's pool |
| **Seal** | Remove specific weapons/items from the pool account-wide (three tiers plus Seal All) |
| **Charm** | Bias the offer pool toward chosen items |
| **Defang** | Reduce enemy damage output |
| **Omni** | Applies effects across a wider set of weapons |

### Key stat mechanics

- **Curse spawn formula:** `effectiveSpawnInterval = spawnInterval ÷ totalCurse`. More Curse = tighter spawn cadence = more XP and gold, at the cost of tankier, faster enemies. It's the classic risk/reward lever, and several characters (Dracula, Bahamut) convert Curse into Might.
- **Luck applied to wave events:** `chanceWithLuck = eventChance ÷ totalLuck`. Higher Luck genuinely suppresses hostile map events.
- **Luck applied to traps:** `effectiveCooldown = trapCooldown × totalLuck`. Higher Luck spaces trap triggers further apart.
- **Additive cap:** the additive pools for Might, Area, Growth, and Greed are hard-capped at +1000%. Passives and Golden Eggs past that threshold do nothing. Arcana multipliers are applied as a separate external multiplier and therefore bypass the ceiling — which is why late-game "infinite scaling" builds are Arcana-driven, not item-driven.
- **Stage modifiers** are applied as a multiplier on top of item/PowerUp bonuses. Luck is the exception: stage Luck bonus is *added* to the character's Luck stat instead.
- **Weapon-level stats vs player stats:** a leveled weapon may gain "Area" or "Amount" that reads identically to the player stat but only affects that weapon. Some level-granted values are also immune to the corresponding player-stat multiplier.

---

## 3. PowerUps (permanent meta-progression)

PowerUps are bought with gold from the main menu and apply to every character.

**Rank caps** (from the wiki's own cost calculator, which is the most reliable public source for these):

| PowerUp | Max rank | Per-rank effect | Initial price |
|---|---|---|---|
| Might | 5 | +5% damage (→ +25%) | 200 |
| Armor | 3 | −1 damage taken (→ −3) | 600 |
| Max Health | 3 | +10% HP | 200 |
| Recovery | 5 | +0.1 HP/s (→ +0.5) | 200 |
| Cooldown | 2 | −2.5% (→ −5%) | 900 |
| Area | 2 | +5% (→ +10%) | 300 |
| Speed | 2 | +10% (→ +20%) | 300 |
| Duration | 2 | +15% (→ +30%) | 300 |
| **Amount** | **1** | +1 projectile | — |
| Move Speed | 2 | +5% (→ +10%) | 300 |
| Magnet | 2 | +25% (→ +50%) | 300 |
| Luck | 3 | +10% (→ +30%) | 600 |
| Growth | 5 | +3% XP (→ +15%) | 900 |
| Greed | 5 | +10% gold (→ +50%) | 200 |
| Curse | 5 | Raises enemy stats/spawns | — |
| Revival | 1 | +1 life | — |
| Omni | 5 | — | — |
| Charm | 5 | — | — |
| Defang | 5 | — | — |
| Reroll | 5 | — | — |
| Skip | 5 | — | — |
| Banish | 5 | — | — |
| Seal I / II / III | 10 each | — | — |
| Seal All | — | — | — |

### Cost formula

```
Price = InitialPrice                                          if TotalBought == 0
Price = InitialPrice × (1 + Bought) + floor(20 × 1.1^TotalBought)   if TotalBought >= 1
```

- `Bought` = ranks already purchased of *that* PowerUp. So base cost climbs linearly per PowerUp (200 + 400 + 600 + …).
- `TotalBought` = ranks purchased across *all* PowerUps. This is the "fee", and it compounds at 1.1× per rank bought — which is why the last few ranks cost orders of magnitude more than the first.
- **Buy order stopped mattering in v0.7.2 (June 2022)** when fees became additive. Older guides recommending a specific purchase sequence are obsolete.
- Full clear cost has been quoted at 27,148,513 gold (≈2.47M base + ≈24.68M fees) — but that figure moves every time ranks are added.
- **"Refund PowerUps"** returns everything at full value (minus a rounding artifact). This is a legitimate and frequently-recommended strategy: refund, then re-buy, to fix inflated post-DLC prices or to re-optimize.

### Golden Eggs

Golden Eggs permanently raise one stat for the character that collected them: +1% Might, +0.1 Armor, +1% Greed, etc. They're per-character, not account-wide, and they stack indefinitely — the primary long-term power source and the main reason veterans' characters wildly outscale a fresh save.

### Limit Break

Unlocked via the **Great Gospel** relic. Once a weapon is fully leveled, further level-ups on it can allocate stats to that weapon (instead of the usual gold or Floor Chicken filler). Limit Break scales the *weapon*, not the character's stat sheet.

---

## 4. Weapons

### Categories

| Type | Behavior |
|---|---|
| **Base** | Chosen from level-ups. Most are locked behind achievements initially |
| **Evolution** | Base weapon + catalyst passive + qualifying chest → replaces the base weapon |
| **Union** | Two fully-leveled base weapons merge into one, **freeing a weapon slot** |
| **Gift** | Grants an *additional* weapon or passive; the base weapon is not consumed |
| **Morph** | Character-specific transformation at high level with a specific Relic; starting weapon auto-evolves, no chest needed |

### Slot rules

- Six weapon slots selectable from level-ups. You can exceed six by picking up weapons that spawn on the stage floor.
- Most weapons are unique per run. Exceptions are single-use items like Candybox and Arma Dio.
- **Rarity** determines a weapon's weight in the level-up and chest pool.
- Already-owned weapons have a defined chance of being re-offered as an upgrade.

### Evolution mechanics

To evolve, you need:

1. The base weapon at **max level**, plus its catalyst passive in inventory.
   - For a **Union**, *both* base weapons must be maxed (and sometimes a passive too).
   - For Infinite Corridor, Crimson Shroud, Sole Solution, Ashes of Muspell, most post-1.0 weapons, and essentially all DLC weapons, **the passive must also be fully leveled**.
   - Bracelet, Bi-Bracelet, Super Candybox II Turbo, and Penshin Fatcha choices need **no passive at all**.
2. A **Treasure Chest containing an evolution reward**.

**Which chests can evolve:**

| Condition | Stages |
|---|---|
| Standard: bosses at/after 10:00 | Most stages |
| First chest evolves (early) | Mad Forest (Glowing Bat, 1:00), Lake Foscari (Foscaritrice, 2:00) |
| **All** chests evolve, any time | Dairy Plant, Il Molise, Cappella Magna, Boss Rash, Laborratory, Abyss Foscari, Hectic Highway |
| All chests except 3:00 and 6:00 bosses | Tiny Bridge |
| All chests except 3:00 and 5:00 bosses | Mt. Moonspell |
| Specific boss chest | Neo Galuga — Metal Alien at 9:00 |
| Inherits source wave's rules | Green Acres |
| Arcana chests (11:00 / 21:00) | Only evolve if you're already at max Arcana count |
| Any chest, unconditionally | Playing as **Gyorunton** |

**Multi-evolution chests** — higher-tier chests with enough rewards can evolve several weapons at once:

- Up to **five**: Il Molise, Bat Country, Tiny Bridge, and the Giant Enemy Crab chest
- Up to **three**: minute-10 chests in Dairy Plant / Gallo Tower / Cappella Magna; minute-9/11/13 non-Arcana chests in Il Molise; minute-20 chests in Inlaid Library / Dairy Plant / Gallo Tower / Cappella Magna / The Bone Zone; chests from The Drowner and Stalker

In **Co-op**, evolution works regardless of which player holds the weapon vs. the passive, as long as "share passives" is enabled in the level select menu.

### Base game evolution recipes

| Base weapon | Catalyst | Result |
|---|---|---|
| Whip | Hollow Heart | Bloody Tear |
| Magic Wand | Empty Tome | Holy Wand |
| Knife | Bracer | Thousand Edge |
| Axe | Candelabrador | Death Spiral |
| Cross | Clover | Heaven Sword |
| King Bible | Spellbinder | Unholy Vespers |
| Fire Wand | Spinach | Hellfire |
| Garlic | Pummarola | Soul Eater |
| Santa Water | Attractorb | La Borra |
| Runetracer | Armor | NO FUTURE |
| Lightning Ring | Duplicator | Thunder Loop |
| Pentagram | Crown | Gorgeous Moon |
| Gatti Amari | Stone Mask | Vicious Hunger |
| Song of Mana | Skull O'Maniac | Mannajja |
| Shadow Pinion | Wings | Valkyrie Turner |
| Clock Lancet | Silver Ring + Gold Ring (both max) | Infinite Corridor |
| Laurel | Metaglio Left + Metaglio Right (both max) | Crimson Shroud |
| Bracelet | — | Bi-Bracelet |
| Bi-Bracelet | — | Tri-Bracelet |
| Pako Battiliar | Hollow Heart (max) | Mazo Familiar |
| Ammo Appalate | Bracer (max) | Gunastrophe |
| Unearthly Bolt | Tirajisú (max) | Spirit Disturbance |
| Flames of Misspell | Torrona's Box (max) | Ashes of Muspell |
| Glass Fandango | Wings (max) | Celestial Voulge |
| Santa Javelin | Clover (max) | Seraphic Cry |
| Gaze of Gaea | Parm Aegis | Embrace of Gaea |
| Magi-Stone | Karoma's Mana | Kyra-Stones |
| Phas3r | Empty Tome (max) | Photonstorm |
| Chaos Rune | Spellbinder (max) | Wicked Ruler |
| Penshin Fatcha (Tonno Subito, Tonnado, Tonn'omoto, Tonn'oddeeo, Tonne, Unsurpassed) | — | Cycles through the set; 6+ evolutions → Miracle of Multiplication |

**Unions:**

| Bases | Catalyst | Union |
|---|---|---|
| Peachone + Ebony Wings | — | Vandalier |
| Phiera Der Tuphello + Eight The Sparrow | Tirajisú | Phieraggi |
| Vento Sacro + Bloody Tear | — | Fuwalafuwaloo |

**Gifts:**

| Base | Condition | Gift |
|---|---|---|
| Victory Sword | Torrona's Box (max) | Sole Solution |
| Candybox | Max-level passives + evolutions | Super Candybox II Turbo |

**Morphs** (all require reaching level 80):

| Character | Weapon | Relic | Becomes |
|---|---|---|---|
| Mortaccio | Bone | Chaos Malachite | Anima of Mortaccio |
| Yatta Cavallo | Cherry Bomb | Chaos Rosalia | Yatta Daikarin |
| Bianca Ramba | Carréllo | Chaos Lazulia | Carrozza! |
| O'Sole Meeo | Celestial Dusting | Chaos Altemanna | Profusione D'Amore |

### DLC evolution recipes

**Legacy of the Moonspell** (all catalysts must be max level):

| Base | Catalyst | Result |
|---|---|---|
| Silver Wind | Pummarola | Festive Winds |
| Four Seasons | Candelabrador | Godai Shuffle |
| Summon Night | Duplicator | Echo Night |
| Mirage Robe | Attractorb | J'Odore |
| Night Sword | Stone Mask | Muramasa |
| Mille Bolle Blu | Spellbinder | Boo Roo Boolle |

**Tides of the Foscari:**

| Base | Catalyst | Result |
|---|---|---|
| SpellString + SpellStream + SpellStrike | — | SpellStrom (union) |
| Eskizzibur | Armor (max) | Legionnaire |
| Flash Arrow | Bracer (max) | Millionaire |
| Prismatic Missile | Crown (max) | Luminaire |
| Shadow Servant | Skull O'Maniac (max) | Ophion |

**Emergency Meeting** — note: the catalyst passive is **consumed** on evolution here, and can be re-acquired if you have a free passive slot.

| Base | Catalyst (max) | Result |
|---|---|---|
| Report! | Mini Crewmate | Emergency Meeting |
| Lucky Swipe | Mini Engineer | Crossed Wires |
| Lifesign Scan | Mini Ghost | Paranormal Scan |
| Just Vent | Mini Shapeshifter | Unjust Ejection |
| Clear Debris | Mini Guardian | Clear Asteroids |
| Sharp Tongue | Mini Impostor | Impostongue |
| Science Rocks | Mini Scientist | Rocket Science |

**Operation Guns** — every recipe requires **Weapon Power-Up**, plus a second passive for most:

| Base | Second catalyst | Result |
|---|---|---|
| Long Gun | — | Prototype A |
| Short Gun | Bracer | Prototype B |
| Spread Shot | Empty Tome | Prototype C |
| C-U-Laser | Tirajisú | Pronto Beam |
| Firearm | Candelabrador | Fire-L3GS |
| Sonic Bloom | Armor | Wave Beam |
| Homing Miss | Duplicator | Multistage Missiles |
| Diver Mines | Attractorb | Atmo-Torpedo |
| Blade Crossbow | Clover | BFC2000-AD |
| Prism Lass | Wings | Time Warp |
| Metal Claw | Hollow Heart | Big Fuzzy Fist |

**Ode to Castlevania** (whip line — sample; the DLC also adds a large union/gift set):

| Base | Catalyst | Result |
|---|---|---|
| Alchemy Whip | Tirajisú | Vampire Killer |
| Wind Whip | Crown | Spirit Tornado Tip |
| Platinum Whip | Clover | Cross Crasher Tip |
| Dragon Water Whip | Attractorb | Hydrostormer Tip |
| Sonic Whip | Skull O'Maniac | Crissaegrim Tip |
| Vibhuti Whip | — | Daybreaker Tip |
| Vanitas Whip | — | Aurablaster Tip |
| Jet Black Whip | — | Mormegil Tip |

Emerald Diorama and Ante Chamber add further sets — check the wiki's Evolution page for those, as they're the newest and most volatile.

---

## 5. Arcanas

Arcanas are run modifiers presented as tarot-styled cards. Unlocked by finding the **Randomazzo** relic in Gallo Tower, which also grants the first card.

### Mechanics

- Toggle Arcanas on/off in the stage selection screen.
- You get up to **three** per run by default:
  1. Free pick at run start from *any* unlocked Arcana (or roll random).
  2. Boss at **11:00** drops an Arcana Chest → pick from 4 random uncollected Arcanas.
  3. Boss at **21:00** — same.
- Rerolls work on Arcana chests, and unlike level-ups, **previously shown options are never re-offered**.
- If you already hold 3+ Arcanas, the Arcana bosses drop ordinary chests instead. If Arcanas are disabled, those bosses don't spawn at all.
- **Once you've unlocked 23+ total Arcanas/Darkanas**, chests offer **6** options instead of 4 and grant **one free reroll each**.
- Timing exception: in **Boss Rash, Bat Country, and Tiny Bridge**, Arcana bosses spawn at 5:00 and 10:00 instead.
- Some characters start with a baked-in Arcana effect (e.g. Avatar Infernas has Heart of Fire XIX). These **don't consume a slot** and persist even with Arcanas disabled.

### Getting more than three

There's no hard cap on how many you can hold:

- **Queen Sigma** gets a bonus pick at levels 1, 2, 3, 77, and 108 — five extras.
- **Inverse mode**: the Merchant sells any unlocked Arcana for 20,000 gold, regardless of how many you hold. With **Endless mode** on, the Merchant respawns each 30-minute cycle, so this can be repeated until you own every unlocked card.
- **Moonlight Bolero (VI)** spawns extra treasure bosses whose chests can contain Arcanas, and the chance persists until all unlocked cards are collected.

The community's "collect everything in one run" setup stacks Queen Sigma + Inverse + Moonlight Bolero on Endless Cappella Magna.

### The 22 Arcanas

| # | Name | Effect | Unlock |
|---|---|---|---|
| 0 | Game Killer | Halts XP gain. Gems become exploding projectiles. All chests contain ≥3 items | Defeat The Ender in Cappella Magna |
| I | Gemini | Listed weapons come with a mirrored counterpart | Lv50 Pugnala |
| II | Twilight Requiem | Listed projectiles explode on expiry; explosion damage scales with Curse | Lv50 Dommario |
| III | Tragic Princess | Listed weapons' cooldown drops while moving | Lv50 Porta |
| IV | Awake | +3 Revivals. Each consumed Revival grants +10% Max HP, +1 Armor, +5% Might/Area/Duration/Speed | Lv50 Krochi |
| V | Chaos in the Dark Night | Projectile Speed oscillates −50%↔+50% over 10s; +1% Speed per level | Lv50 Giovanna |
| VI | Sarabande of Healing | Healing doubled; healing damages nearby enemies for the same amount | Find the Randomazzo |
| VII | Iron Blue Will | Listed projectiles gain up to 3 bounces, may pass through enemies and walls | Lv50 Gennaro |
| VIII | Mad Groove | Every 2 min, pulls all stage items, pickups, and light sources to you | Reach 31:00 in Mad Forest |
| IX | Divine Bloodline | Armor also boosts listed weapon damage and reflects enemy damage; bonus damage from missing HP; retaliation kills give +0.5 Max HP | Lv50 Clerici |
| X | Beginning | Listed weapons +1 Amount; your starting weapon and its evolution get +3 instead | Lv50 Antonio |
| XI | Waltz of Pearls | Listed projectiles gain up to 3 bounces | Lv50 Imelda |
| XII | Out of Bounds | Freezing enemies causes explosions; Orologions easier to find; Sorbettos can appear | Reach 31:00 in Gallo Tower |
| XIII | Wicked Season | Growth, Luck, Greed, Curse doubled at fixed intervals; +1% of each per 2 levels | Lv50 Christine |
| XIV | Jail of Crystal | Listed projectiles have a chance to freeze | Lv50 Pasqualina |
| XV | Disco of Gold | Coin bags trigger Gold Fever; gold gained restores equal HP | Reach 31:00 in Inlaid Library |
| XVI | Slash | Enables crits for listed weapons; doubles overall crit damage | Lv50 Lama |
| XVII | Lost & Found Painting | Duration oscillates −50%↔+50% over 10s; +1% Duration per level | Lv50 Poppea |
| XVIII | Boogaloo of Illusions | Area oscillates −25%↔+25% over 10s; +1% Area per level | Lv50 Concetta |
| XIX | Heart of Fire | Listed projectiles explode on impact; light sources explode; you explode when damaged | Lv50 Arca |
| XX | Silent Old Sanctuary | +3 Reroll/Skip/Banish; +20% Might and −8% Cooldown per **empty** weapon slot | Reach 31:00 in Dairy Plant |
| XXI | Blood Astronomia | Listed weapons emit damage zones scaled by Amount and Magnet; enemies in Magnet range take damage | Lv50 Poe |

**Important interaction rule:** an Arcana affects a weapon's evolution *only if the evolution is explicitly listed in the card's description*. Blood Astronomia lists both base and evolved forms; many cards don't.

Note also that V, XIII, XVII, and XVIII stack **multiplicatively** with other sources of their stat — which is why they're so strong in stacked builds.

### Darkanas

Introduced in patch 1.11, unlocked by the **Darkasso** relic found due south in Room 1665. There are 12. They share Roman numerals with regular Arcanas and reuse the same artwork.

The patch notes were explicit: *the only difference between Arcanas and Darkanas is theme; functionally they are identical.* The widespread community belief that Darkanas are "inverse" versions is not supported by the developer's own statement.

Example — **Stake to Your Heart (0)**: halts XP gain, enemies drop Gold Coins, damage is dealt to your gold instead of your HP, and a special merchant spawns every minute. Unlocked by defeating a Bat Dragon with Big Troubler.

---

## 6. Enemies

### Spawning

On stage entry, an initial batch spawns based on stage modifiers. Thereafter the game attempts periodic spawns. Enemies generally appear just off-screen and **despawn if you move far enough away**.

**Waves:** one wave per minute. Each wave defines a *minimum enemy count* and a *spawn interval*.

- If the live count is below the minimum, the game spawns until the quota fills.
- If the live count is above the minimum, it spawns one of each enemy type in the wave.
- **Hard ceiling: at 300+ live enemies, periodic spawning stops.** Only bosses and map-event enemies can spawn past that point. This is the mechanic behind screen-clearing builds — it's why killing fast actually *increases* incoming density, and why some setups deliberately let enemies live.

**Bosses:** spawn in specific waves. Higher health, higher damage, frequently resistant to several effects. They have a chance to drop a Treasure Chest. Crucially, **bosses don't despawn** when you run away — they teleport back onto your screen.

### Map events

Short, scripted spawns outside the regular cycle — a sweeping swarm, an encircling ring of high-HP enemies, etc.

| Type | Behavior |
|---|---|
| **Wave events** | Tied to a wave, trigger at the same second marks each time — fully predictable. Each defines spawn count, repeat count, and repeat interval. Most have a trigger chance reduced by Luck (`eventChance ÷ totalLuck`). Events with no listed chance, or chance 0, are guaranteed |
| **Traps** | Circular pressure plates in **Dairy Plant** and **Gallo Tower**. Stepping on one triggers an unavoidable random event from a stage-specific pool. Global cooldown = `trapCooldown × totalLuck` |
| **Special events** | One-time, on a global timer or unique trigger. Not part of stage waves — which means their enemies **cannot appear in Green Acres** |

### Enemy base stats

| Stat | Meaning |
|---|---|
| **Health** | Damage required to kill |
| **Power** | Contact damage before your Armor is applied |
| **Speed** (MoveSpeed) | Movement rate |
| **Knockback** | Multiplier on how far your weapons push them |
| **XP** | Experience granted on death |

### Enemy resistances

| Resistance | Effect |
|---|---|
| **Freeze resistance** | A numeric value. If it exceeds the freeze chance of the weapon that hit them, they don't freeze. **Orologion's freeze bypasses this entirely** |
| **Instant kill resistance** | Immunity to effects dealing damage equal to max health. Instant kill comes from **Pentagram, Gorgeous Moon, and Rosary** |
| **Debuff resistance** | Immunity to weakening effects. The classified debuffs are: knockback and freeze-resistance reduction from **Garlic** and **Soul Eater**, and the slow from **Mannajja** |

### Enemy skills (passive traits)

| Skill | Effect |
|---|---|
| **HP x Level** | Health multiplied by *your* level. Locked in at spawn time — leveling up afterward doesn't retroactively buff a live enemy. This is the mechanic that makes over-leveling dangerous |
| **Fixed Direction** | Moves in a straight line instead of continuously homing on you |
| **Floaty** (internally "Medusa") | Moves in a wavy, sinusoidal pattern |

Beyond these, individual enemies carry bespoke behaviors documented per-entry: **self-destruct** (Sig.ra Rossi, Poltergeist), **cannot move + fires projectiles on an interval** (Twin Snakes 2s, Twin Demons 1.5s, Lost Twin 1s, Twin Skulls 1s, the Molisano family), **multiple lives** (Scarleton has three), and **absorption** (Sketamari absorbs other enemies and inherits their health).

### Effects that modify enemies

- **Curse** — raises enemy Max Health and Move Speed and tightens spawn interval by the formula above.
- **Hyper mode** — raises the minimum spawn count and enemy movement speed per stage, and may raise their max health.

### Bestiary

The Bestiary (in-game: **Ars Gouda**) currently holds **360 entries** (372 in beta). Entries with **yellow names** are required for an unlock — e.g. Dragon Shrimp is yellow because killing 3,000 of them unlocks O'Sole Meeo.

Bestiary numbering differs by platform: mobile uses a different DLC ordering than Steam/Epic/Xbox/PlayStation/Switch. The wiki maintains a "Bestiary Locator" calculator that takes your missing entry number, your platform, and which DLCs you own, and tells you where to find that enemy.

### Enemy families by stage (representative — not exhaustive)

| Stage | Characteristic enemies |
|---|---|
| **Mad Forest** | Pipeestrello (Glowing Bat), Skeleton, Zombie, Mudman, Flower Wall (HP×Lv), Ghost, Werewolf, Giant Bat, Mantichana, Big Mummy, Venus |
| **Inlaid Library** | Dust Elemental, Lionhead (HP×Lv), Sig.ra Rossi (self-destruct), Hag (resists freeze/Rosary/debuff/knockback), Nesufritto, Mummy, Sneaky Head, Undead Witch, Undead Sassy Witch, Merdusa, Musc Musc, Testa di Mano |
| **Dairy Plant** | Milk Elemental, Merman, Lizard Pawn/Rook, Twin Snakes, Twin Demons, Jellyfish, Skeleton Ninja, Lost Twin, Melone, Minotaur, Mignotaur, Archon Lancia/Ascia, Skelewing, Tritont, Big Golem, Sword Guardian |
| **Gallo Tower** | Bloodbath, Skullino, Skulorosso, Scarleton (3 lives), Dragon Shrimp (HP×Lv), Poltergeist, Harzia, Impefinger, Ghiavolo, Undead Mage, Archon Spada/Disco, Manticore, Meat Golem, **Giant Enemy Crab** (unique boss, 25:00) |
| **The Bone Zone** | Twin Skulls, Skullone, Skeleton Panther, Giant Skeleton, Skeletone (HP×Lv), **Sketamari** (unique boss — absorbs enemies, fixed direction, resists nearly everything) |
| **Il Molise** | The Molisano family — Sad, Happy, Cute, Old, Dead. All immobile |
| **Whiteout** | Bambaman, Miragellos, Menta Elemental, Madd-Onna, Kizzune (heavily resistant, HP×Lv) |
| **Laborratory** | Holy Circuit Creations (RetroBot, T-DCCC, ConstableBot, ER-2000), Bounty Hunter, Space Hunter, Tri-Blunder |
| **The Coop** | Chickenfantry, Cockreliutennant, Chik, Egge, Pol.lo Rosso, Abraxas / Abraxas Phronesis / Abraxas Dynamis |
| **Space 54** | Gala Invader, Moon Rabbit, Moon Duck, Space Ant Onion, Space Pickle, ECMASlime, Sinistronz |
| **Moongolow** | Merman, Jellyfish, **the Atlantean bosses** — Sun, Moon, City, Volcano, and Moongolow Atlanteans (all resist freeze/Rosary/debuff/knockback; the four elemental Atlanteans also invade Mad Forest, Inlaid Library, and others) |
| **The Lycaeum** | Moon Anforas, Moongolow Atlanteans, Itchiocentaurs, **The Drowner**, Giant Enemy Crabs (button-summoned), Bat Dragons |

Notable specials: **The Reaper** (stage-completion executioner), **Stage Killer**, **The Ender** (Cappella Magna final enemy — unlocks Game Killer), **Megalo** variants (Megalo Impostor Rina, Megalo Elizabeth, etc. — heavily buffed boss forms), **The Directer**, **Je-Ne-Viv**.

---

## 7. Stages

**80 official stages** currently: 33 primary + 47 Adventure-mode-exclusive.

Primary breakdown:
- **Base game (20):** 5 normal, 7 bonus, 9 challenge, 2 special, 1 hidden
- **DLC (9 or more):** Mt. Moonspell, Lake Foscari, Abyss Foscari, Polus Replica, Neo Galuga, Hectic Highway, plus Ode to Castlevania, Emerald Diorama, and Ante Chamber stages

Adventure mode adds 13 base-game and 34 DLC stages (excluding repeats of main-game maps).

**Time limits:** 30:00, 20:00, or 15:00 depending on the stage.

**Unlocking:** every stage past Mad Forest is gated on an achievement, not gold. Mad Forest is unlocked by default; Inlaid Library opens at level 20, and the standard 5-stage chain then unlocks at level thresholds 40, 60, 80 on the previous map.

**Map generation:** most stages generate terrain procedurally as you move, so the map is effectively infinite. Items other than chests stay where they spawn — if you skip an item and keep moving, it's gone for good. Chests are the exception.

### Stage modifiers

Each stage applies its own modifiers to player and enemy stats: movement speed, projectile speed, gold gain, Luck, and enemy speed/health/quantity. Stage modifiers and enabled-mode bonuses stack **additively**, then the combined total is applied as a **multiplier** to your item and PowerUp bonuses. Luck is the exception — it's added directly to your Luck stat.

Known gold multipliers: Dairy Plant ×1.2; Gallo Tower, Bat Country, Tiny Bridge ×1.3; Cappella Magna ×1.4.

Stages also govern which passives and weapons spawn on the floor — which is why Inlaid Library is the classic evolution-farming map (Empty Tome and Stone Mask spawn on the ground, removing chest RNG from those recipes).

### Modes

Four unlockable modes, stackable with each other and with the modifiers:

| Mode | Effect |
|---|---|
| **Hyper** | Character and enemy move speed +65–75%, projectile speed +15–25%, gold ×1.5. Also raises the minimum enemy spawn count and may raise enemy max health |
| **Hurry** | Accelerates the clock |
| **Inverse** | Mirrors the stage; enables Merchant Arcana sales at 20,000 gold |
| **Endless** | No Reaper at the time limit — the run continues, cycling the wave table |

### Gameplay modifiers

| Modifier | Effect |
|---|---|
| **Arcanas** | Enables the Arcana system for the run |
| **Limit Break** | Allows stat allocation into maxed weapons |
| **Random Events** | Randomizes map events |
| **Random LevelUp** | Randomizes the level-up offer pool |

### The Lycaeum (v1.15) — a worked example of modern stage design

Unlocked by collecting **50 Vacuums** across all runs. A *finite* underwater stage rather than a procedural one:

- **Bottom section:** bookshelf obstacles. Enemies spawn only from the east and west screen edges. Below it is an open area continuously spawning Moon Anforas and occasionally Moongolow Atlanteans. Blue braziers serve as light sources. Three red buttons (center and both edges) each summon a Giant Enemy Crab.
- **Vertical section:** entering it spawns **The Drowner**, which can only be killed by normal damage or escaped by reaching the top. Every other floor holds a coffin dropping either a Treasure Chest or a Yellow Sign stage item. Those chests can only be collected with **Mad Groove (VIII)**, **Edge of the Earth (VIII)**, or a **Lavatrix Machina**. Layout: 10 coffins on the left, then 5 Itchiocentaur buttons, then 10 coffins on the right.
- **Top:** **Big Troubler** sells the Green Eyes Vizard (60,000), Survarocchi (120,000), and — after the relevant secret — the Enemahs (180,000). Buying the Green Eyes Vizard makes **Bat Dragons** visible, and they then invade Il Molise and Mad Forest.

---

## 8. Characters

**207 official playable characters** currently: 54 base game (22 of which are secrets) and 153 across the DLCs (56 secrets).

### Purchase economy

- All non-secret characters cost gold. **Each character purchased raises the cost of subsequent purchases by 10%, additively.**
- Exceptions: **Antonio** (free, the default starter), **Queen Sigma** (achievement-gated but 0 gold), **Dracula** (unlocked immediately on obtaining the Black Disk, no purchase required).

### Selection screen states

| State | Appearance |
|---|---|
| Unlocked & bought | Gray icon, character and starting weapon visible, name in white |
| Unlocked, not bought | Gray icon, weapon visible, character is a black silhouette, name in gray |
| Locked | Entire icon blue-gray, character/name/weapon all translucent |

A brand-new save shows only Antonio, Imelda, Pasqualina, and Gennaro. Once you unlock any new character, silhouettes of locked characters begin appearing.

### Multi-weapon characters

Most characters start with one weapon. Exceptions worth knowing:

- **Soleil Belmont** (Ode to Castlevania secret) — three whips: Jet Black, Vibhuti, Vanitas
- **Celia Fortner** — starts with both weapons involved in her unlock
- **Wood Rod** (Innocent Devil variant) — Magic Wand, Fire Wand, Lightning Ring
- **Pumpkin** (Innocent Devil variant) — Curved Knife, Troll Bomb

This interacts sharply with **Beginning (X)**, which grants +3 Amount to the starting weapon — multi-weapon characters get it on *every* starting weapon.

### Ability archetypes

Character passives fall into recognizable patterns:

| Archetype | Examples |
|---|---|
| **Per-level scaling, capped** | Antonio (+10% damage per 10 levels, max +50%), Imelda (+10% XP per 5 levels, max +30%), Pasqualina (+10% projectile speed per 5 levels, max +30%), Arca (−5% cooldown per 10 levels, max −15%) |
| **Per-level scaling, uncapped** | Soma, Shanoa, Blue Crescent Moon Cornell (+1% Might/level); Dracula (+1% Might/level *and* +1% per +1% Curse); Trouser (+1% Greed/level) |
| **Flat starting bonus** | Gennaro (+1 projectile), Porta (+30% Area), Brad/Stanley (+20% Might), Galamoth (+100% Might), Megalo variants (+250% Might) |
| **Stat conversion** | Probotector (+10% Might per +1 Armor), Newt (+1% Might per +1% Speed), Stanley (+1 Armor per +10% Might), Bahamut (+1% Might per +1% Greed *or* Curse), Christopher (+1 Armor per +23% Greed), Simon (+1 Armor per +23% Growth) |
| **Death-scaling** | Lucia (+4% Might, +0.2 Armor, +1% Greed per revival, all uncapped) |
| **Deliberate drawbacks** | Gallo and Divano (−50% Greed), Leda and Marrabbio (−80% Greed) |
| **Built-in Arcana** | Avatar Infernas (Heart of Fire XIX, doesn't use a slot) |
| **Rule-breaking** | Gyorunton (any chest can evolve), Queen Sigma (5 bonus Arcana picks), Sammy (converts gold to XP, ignoring Game Killer's XP halt), Master Librarian (+1000% Greed), Keremet (+20 Armor) |
| **Morphing** | Mortaccio, Yatta Cavallo, Bianca Ramba, O'Sole Meeo — see the Morph table above |

### Unlock chains

Ode to Castlevania in particular uses deep dependency chains. A representative one: unlocking Julia requires defeating Dracula; unlocking Richter requires Maria; Maria requires completing a stage with both Shanoa *and* Jules; Shanoa requires evolving Iron Ball and Javelin; Iron Ball requires John, who requires Julia's starting weapon. The **Unlocks menu in-game** documents these chains and is more reliable than any third-party list, because it reflects your actual DLC ownership.

Sample base-game unlock conditions:

| Character | Starting weapon | Unlock |
|---|---|---|
| Antonio | Whip | Default |
| Imelda | Magic Wand | Survive 5 minutes |
| Pasqualina | Runetracer | Survive 10 minutes |
| Gennaro | Knife | Purchase (600) |
| Arca | Fire Wand | Fire Wand to Lv4 in one run |
| Porta | Lightning Ring | Lightning Ring to Lv4 in one run / 3,000 kills in one run |
| Lama | Axe | Survive 20 minutes |
| Poe | Garlic | Collect 50 Floor Chickens across runs |
| Mortaccio | Bone | Kill 3,000 skeletons |
| O'Sole Meeo | Celestial Dusting | Kill 3,000 Dragon Shrimp |

---

## 9. Relics

Relics are permanent account-wide unlocks found on stages, not purchased.

| Relic | Grants |
|---|---|
| **Randomazzo** (Gallo Tower, north of start) | The Arcana system + Sarabande of Healing (VI) |
| **Darkasso** (Room 1665, due south) | The Darkana set |
| **Great Gospel** | Limit Break |
| **Black Disk** | Unlocks Dracula outright |
| **Chaos Malachite / Rosalia / Lazulia / Altemanna** | Morph catalysts for Mortaccio / Yatta Cavallo / Bianca Ramba / O'Sole Meeo |
| **Green Eyes Vizard** (Lycaeum, 60,000) | Makes Bat Dragons visible |
| **Survarocchi** (Lycaeum, 120,000) | — |
| **Enemahs** (Lycaeum, 180,000, secret-gated) | — |
| **Lavatrix Machina** | Attracts treasure chests |
| **Yellow Sign** | Reveals hidden stage items |
| **Milky Green Ball**, **Sorceress' Tears**, **Ars Gouda** (Bestiary), **Gracia's Mirror**, **Mindbender**, **Glass Vizard**, **Forbidden Scrolls of Morbane**, **Seventh Trumpet** | Various mode/UI/secret unlocks |

---

## 10. DLCs

| DLC | Adds |
|---|---|
| **Legacy of the Moonspell** | Mt. Moonspell, Eastern-themed weapon set (Silver Wind, Four Seasons, Summon Night, Mirage Robe, Night Sword, Mille Bolle Blu) |
| **Tides of the Foscari** | Lake Foscari, Abyss Foscari, the SpellString trio, Eskizzibur, Flash Arrow, Prismatic Missile, Shadow Servant |
| **Emergency Meeting** | Polus Replica, Among Us crossover; Mini-crewmate passives that are consumed on evolution |
| **Operation Guns** | Neo Galuga, Hectic Highway, Contra crossover; the Weapon Power-Up catalyst system |
| **Ode to Castlevania** | Castlevania crossover; the largest character roster addition, deep unlock chains, whip-tip evolution system |
| **Emerald Diorama** | Free expansion; own weapon/enemy/evolution set |
| **Ante Chamber** | Free expansion; own weapon/enemy/evolution set |
| **The Lycaeum (v1.15)** | Free update: underwater stage, Para Kooleo skins, new weapons, buffs to Magi-Stone / Mille Bolle Blu / Chula-Reh, a new Darkana for mid-run weapon purchases, new bosses, stage-item banishing, character selection filter, multiple save slots, character setup menu, Survarot power creep for all characters |

There's also a spin-off, **Vampire Crawlers: The Turbo Wildcard** — turn-based, card/deck-driven, grid maps, Evolution Statues that consume Evolution Gems to sacrifice cards into holographic versions with higher Mana costs. It uses a *different* stat model (Might capped +1000%, Amount capped +50, Growth and Greed +1000%), and splits stats into temporary (Amount, Might, Recovery, Hand, Duration, Area — shown left of screen, expire at end of battle) versus permanent (Growth, Greed, Luck, Revivals — shown right, persist for the run). Don't apply main-game stat assumptions to it.

---

## 11. Where to get raw data

If you want the actual numbers — per-enemy HP/Power/Speed/XP tables, per-weapon per-level stat curves, wave tables with exact second marks — these are the places to go:

1. **vampire.survivors.wiki** — the official wiki (Weirdgloop-hosted, CC BY-NC-SA 3.0). Key pages: `Enemies`, `Weapons/Overview Stats`, `Weapons/Combos`, `Player stats`, `Arcanas`, `Evolution`, `Stages`, `PowerUps`, `Characters`. It also hosts live calculators (`Calculators/PowerUp Cost`, `Calculators/BestiaryLocator`).
2. **The game's own data files.** Vampire Survivors is an HTML5/Electron game — the balance data ships as readable JSON in the app's resources. This is the authoritative source and the only way to get a genuinely complete, machine-readable dataset. If you're building a tool, extract from there rather than scraping.
3. **vampire-survivors.fandom.com** — the older Fandom wiki. Still maintained and often has different per-stat breakdowns (particularly the "which characters modify this stat" lists), though it occasionally lags the official wiki on new content.
4. **In-game Unlocks menu and Ars Gouda (Bestiary)** — the definitive source for unlock chains and enemy entries *for your specific DLC ownership and platform*, which no external list can match.

### Caveats worth carrying

- Enemy stat blocks are per-*variant*, not per-name. "Skeleton" covers six sprite variants across four stages with different stats; "Big Golem" has a version with HP×Level and freeze resistance and a version without.
- Bestiary numbering diverges between mobile and every other platform.
- Character counts, stage counts, and Arcana counts have all changed multiple times in the past year alone.
- Several older third-party guides still reference the pre-v0.7.2 PowerUp purchase-order optimization, which no longer applies.
