# Your first survivors-style game — Blueprint build guide

A complete beginner's walkthrough. By the end you'll have a top-down character you drive with WASD, and enemies that spawn and chase you. That's the skeleton of the whole game — everything else (weapons, XP, upgrades) attaches to this.

Keep this open on a second screen while Unreal is open on your main one.

---

## First: the words you'll keep hearing

You don't need to memorise these — just glance back when one confuses you.

- **Blueprint** — Unreal's visual scripting. You wire up boxes (*nodes*) instead of typing code. Each Blueprint is usually one "thing" in your game (the player, an enemy, a spawner).
- **Node** — a single box that does one job (e.g. "get the player's location"). You drag them onto a canvas called the *Event Graph*.
- **Wire** — the line connecting nodes. **White wires** = order of operations (do this, then this). **Colored wires** = data being passed around (a number, a location, an object).
- **Event** — a node that *starts* a chain of logic when something happens. `Event BeginPlay` fires once when the game starts. `Event Tick` fires every single frame (~60 times a second).
- **Actor** — anything that can exist in your level (a wall, an enemy, a pickup). The most general "thing."
- **Character / Pawn** — a special Actor that can be controlled and can move. Your player is a Character.
- **Component** — a part bolted onto an Actor (a mesh so you can see it, a collision shape, a camera).
- **Variable** — a labelled container that stores a value (a number like `Speed = 200`, or an object like "the player").
- **Compile** — Unreal checks your Blueprint for errors and makes it runnable. You click the **Compile** button (top-left of the Blueprint editor) after editing. Then **Save**.
- **PIE (Play In Editor)** — the green **Play** button at the top. Runs your game instantly inside the editor so you can test.

**How you add a node:** right-click on empty canvas → a search box appears → type the node name → click it. Or drag off the end of an existing wire and the same search appears (this auto-filters to compatible nodes, which is easier).

---

## Phase 0 — Create the project (~5 min)

We are **not** building movement from scratch. Unreal's Third Person template already includes a fully working character with WASD movement and input set up. We'll start from that and just point the camera straight down. This saves you from the single most fiddly beginner task (input setup), so you can focus on learning the parts that matter.

1. Open the **Epic Games Launcher** → Unreal Engine → **Launch 5.8**. (Or open UE 5.8 directly.)
2. In the project browser: **Games** category → **Third Person** template.
3. On the right: set **Blueprint** (not C++), **Starter Content** on is fine, quality/target defaults are fine.
4. Name it something like `SurvivorsProto`, pick a folder, click **Create**.
5. Wait for the editor to open. Press the green **Play** button to confirm you can run around in third person with WASD + mouse. Press **Esc** or **Stop** to end. Good — that's your starting point.

---

## Phase 1 — Make it top-down (~15 min)

Right now the camera sits behind the character. We want it high above, looking down. All the changes happen inside one Blueprint.

### 1.1 Open the player Blueprint
- In the **Content Drawer** (button at the bottom of the screen, or press **Ctrl+Space**), navigate to a folder like `Content/ThirdPerson/Blueprints`.
- Double-click **BP_ThirdPersonCharacter** to open it.
- You'll see the **Viewport** (a 3D preview) and a list of **Components** on the left. You should spot a `CameraBoom` (also called a Spring Arm — an invisible pole the camera sits on) and a `FollowCamera`.

### 1.2 Aim the camera down
- Click **CameraBoom** in the Components list.
- On the right is the **Details** panel. Find these fields and set them:
  - **Transform → Rotation**: set **Y (Pitch)** to `-70`. (Negative pitch tilts it downward. Try `-60` for a slight angle or `-90` for a pure top-down look — pick what feels good later.)
  - **Camera → Target Arm Length**: set to `1200`. (This is how far the camera pulls back — higher = you see more of the map.)
  - **Camera → Use Pawn Control Rotation**: **uncheck** it. This stops the camera from spinning when you move the mouse, so it stays locked overhead.
  - Also uncheck **Enable Camera Lag** if it's on (optional; keeps the camera perfectly rigid).
- Click **Compile**, then **Save**.

### 1.3 Stop the mouse from steering
There's a catch: in the template, *movement direction* is tied to where the mouse is aimed. With a fixed overhead camera we don't want that — we want W to always mean "up on screen." The fix is to remove the mouse-look logic.

- Click the **Event Graph** tab (top of the Blueprint editor).
- Scroll/pan around (hold right-mouse-button and drag to pan; mouse-wheel to zoom) until you find a group of nodes starting with **`IA_Look`** or **`InputAction Look`** — it connects to nodes named **Add Controller Yaw Input** and **Add Controller Pitch Input**.
- Drag a selection box around that whole `Look` chain to select those nodes, then press **Delete**.
- Leave the `Move` chain (with `IA_Move`) alone — that's your WASD.
- **Compile**, **Save**.

### 1.4 Test
- Press **Play**. You should now be looking down from above, driving the character with WASD, and the mouse no longer rotates anything.

**That's a real, playable top-down game loop.** It's empty, but the foundation is done. If you want to stop here for session one, this is a natural break point.

> **Common snags:** If the character walks off the edge of the small template floor, that's fine for now — Phase 2 keeps enemies near you. If W feels like it goes a weird direction, double-check you set **Use Pawn Control Rotation** to *unchecked* and deleted the Look nodes.

---

## Phase 2 — Enemies that spawn and chase you (~30–40 min)

Two new Blueprints: an **Enemy** (walks toward you) and a **Spawner** (creates enemies on a timer). We'll keep the "chase" logic deliberately simple — no AI navigation systems, just "each frame, step toward the player." For a survivors game where enemies beeline at you, that's exactly right.

### 2.1 Create the Enemy Blueprint
1. In the Content Drawer, pick a folder (e.g. `Content/ThirdPerson/Blueprints`). Right-click empty space → **Blueprint Class**.
2. In the pop-up, choose **Actor** as the parent class. Name it **BP_Enemy**. Double-click to open it.
3. We need to see it and give it a body. In the **Components** panel (top-left), click **+ Add** → search **Static Mesh** → add it. With that Static Mesh component selected, in **Details** find **Static Mesh** and pick a simple shape — type `Sphere` and choose the engine Sphere (or `Cube`). 
4. Scale it down: with the mesh selected, set **Transform → Scale** to about `0.5, 0.5, 0.5`.
5. **Compile**, **Save**.

### 2.2 Give the Enemy a speed variable
1. In the **My Blueprint** panel (lower-left), next to **Variables**, click the **+**.
2. Name it `Speed`. In its **Details**, set **Variable Type** to **Float**.
3. **Compile** (needed before you can set a default). Then set its **Default Value** to `200`.
4. Save.

### 2.3 Make it chase the player
Go to the **Event Graph** tab. You'll build one chain hanging off **Event Tick**. Tick fires every frame and hands you **Delta Seconds** (how long the last frame took) — multiplying by this keeps movement smooth regardless of framerate.

Build this left to right. (Reminder: drag off a node's output pin and start typing to find the next node.)

1. Find the existing **Event Tick** node (add it if it's missing: right-click → search "Event Tick").
2. Add **Get Player Character** → then drag off its blue output → **Get Actor Location**. This gives the player's position. Call it *PlayerLoc*.
3. Add **Get Actor Location** again but off of a **Self** reference (right-click → "Get Actor Location" with target self) — this is *the enemy's* position. Call it *MyLoc*.
4. Add a **subtract (vector)** node: right-click → search `vector - vector`. Plug **PlayerLoc** into the top pin and **MyLoc** into the bottom. Output = the direction+distance from enemy to player.
5. Drag off that result → **Normalize** (search `Normalize`). This shrinks it to a pure direction of length 1. Call it *Dir*.
6. Add a **multiply (vector * float)** node. Plug **Dir** into the vector pin. For the float, plug in `Speed` (drag your `Speed` variable in from My Blueprint → choose **Get**) multiplied by **Delta Seconds** — so add a **float * float** node first: `Speed × Delta Seconds`, feed its result into the vector multiply. Result = how far to move this frame, in the right direction.
7. Add a **vector + vector** node: **MyLoc** + (the movement step from step 6) = the new position.
8. Add **Set Actor Location**: plug the step-7 result into **New Location**. Check the **Sweep** box (so the enemy bumps into things instead of passing through). Connect the white execution wire from **Event Tick** into **Set Actor Location**.
9. **Compile**, **Save**.

> If that node chain feels like a lot: the whole thing says, in plain English, *"every frame, look at where the player is, figure out the direction toward them, and scoot a little bit that way."* That's it.

### 2.4 Create the Spawner
1. Content Drawer → right-click → **Blueprint Class** → **Actor** → name it **BP_Spawner**. Open it.
2. Go to the **Event Graph**.
3. Off **Event BeginPlay**, add **Set Timer by Event**:
   - Set **Time** to `2.0` (spawn every 2 seconds).
   - Check **Looping**.
   - Drag off the red **Event** pin on the left of the timer node → **Add Custom Event** → name it `SpawnEnemy`. (This creates a new event node that the timer will call repeatedly.)
4. Now build the spawn logic off your new **SpawnEnemy** event:
   - **Get Player Character** → **Get Actor Location** = the player's position.
   - We want to spawn a bit away from the player, at a random spot. Add two **Random Float in Range** nodes, each set **Min = -1000**, **Max = 1000**. One is your X offset, one is your Y offset.
   - **Break** the player location (drag off it → search **Break Vector**) to get its X, Y, Z. 
   - Add a **Make Vector** node. Set **X** = (player X + randomX), **Y** = (player Y + randomY), **Z** = player Z. (Use two **float + float** nodes for the X and Y additions.)
   - Add **Spawn Actor from Class**:
     - **Class** = `BP_Enemy` (pick it from the dropdown).
     - For **Spawn Transform**: drag off the transform pin → **Make Transform**, and plug your Make Vector into its **Location**.
     - Set **Collision Handling Override** to **Try To Adjust Location, But Always Spawn** (avoids spawns silently failing).
   - Connect the white execution wire: **SpawnEnemy → Spawn Actor from Class**.
5. **Compile**, **Save**.

### 2.5 Put the Spawner in the level
The Spawner only runs if it exists in your level.
- From the Content Drawer, **drag BP_Spawner into the viewport** (drop it anywhere; its position doesn't matter since it spawns relative to the player).
- **Save** the level (Ctrl+S).

### 2.6 Test
- Press **Play**. Every 2 seconds a sphere should appear near you and start rolling toward you. Drive around with WASD and watch them track you.

**Now it feels like a game.** You move, things hunt you. This is the beating heart of a survivors title.

> **Common snags:**
> - *No enemies appear:* Make sure BP_Spawner is actually placed in the level (2.5), and that the Class in Spawn Actor is set to BP_Enemy.
> - *Enemies spawn but don't move:* Re-check the Set Actor Location's execution wire is connected to Event Tick, and that Speed isn't 0.
> - *Enemies spawn on top of you:* Increase the random range (e.g. Min -1500 / Max 1500) so they appear further out.

---

## The road from here to a Steam alpha

You've built the foundation. Here's the rest of the loop, in the order I'd tackle it. Each phase is its own short session.

- **Phase 3 — Auto-firing weapon.** On a timer on your Character, spawn a `BP_Projectile` that flies toward the nearest enemy. Give the projectile an "on hit → destroy the enemy" rule. This is where the game becomes *fun* — you stop running helplessly and start clearing hordes.
- **Phase 4 — XP and level-ups.** When an enemy dies, spawn an XP gem. Player picks it up → XP goes up → at a threshold, pause and show 3 upgrade choices (faster fire, more damage, extra projectile). This is the addictive core.
- **Phase 5 — Health, death, and a run timer.** Enemy touches you → lose health. Health hits 0 → game over screen. Add a 10-minute survival timer with spawns escalating over time. **This completes a shippable alpha loop.**
- **Then: content + polish.** More enemy types, a few weapons, one boss, a title screen. That's your "content for review" build.

A single 10-minute run with one weapon, XP upgrades, and a death screen genuinely counts as a playable alpha — the loop *is* the game.

## A note on using Claude + the Unreal MCP here

For these early phases, build the node graphs by hand. Wiring them yourself is how the concepts actually stick, and Phase 1–2 are small. Later — once you understand what a Blueprint *is* — lean on the MCP for the repetitive scaffolding: creating the projectile Blueprint, building the upgrade-choice UI widget, placing Niagara hit effects, generating enemy variants. That split gets you speed on the boring parts without skipping the learning on the parts that matter. The MCP is also still experimental, so expect the occasional hiccup and keep saving often.
