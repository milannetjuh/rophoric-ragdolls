# Euphoria-style ragdoll for Roblox: the definitive goal

This file describes **what the user wants the finished system to look and feel like**, what part of it you build, and how to tell when it's done. It deliberately says little about *how* to build it: the design, the architecture and the code are yours. Where a section gives numbers, they describe the *look*. Treat them as targets you can argue with, not as implementation rules.

---

## 0. Your role and how this project works

### Who does what

- **You (this chat) design and build the core engine:** the Euphoria-style active ragdoll and its hit reactions. This is the hard, important part, and the user wants your best thinking on it.
- **A second Claude chat builds on top of your engine later**, on the user's own PC with Roblox Studio: kicks, grabs and shoves, the map, crowds of wandering dummies, dummies helping each other, personalities, the third-person camera. That chat also brings your code into Studio, tests it, analyses recordings frame by frame and writes review files back to you.
- **So build the core to be extended by someone else** (section 5): clean public interfaces, clear documentation, and no shortcuts that only work for one dummy and one attack.

### How you work

- **You work in the user's GitHub repository with Claude Code, with no access to Roblox at all.** Neither you nor the user can run Roblox Studio while you work. The user can answer questions in chat. Later, on their own PC, the second chat pulls the repo into Studio, tests it, and brings results back to you.
- **No Rojo, and no other sync tool.** The user doesn't use them. The repo holds plain `.luau` files, and the second chat copies them into Studio by script. Make that simple and mistake-proof:
  - **Mirror the Studio hierarchy in folders,** for example `src/ReplicatedStorage/Ragdoll/Balance.luau` or `src/StarterPlayer/StarterPlayerScripts/PlayerCombat.client.luau`.
  - **Mark the script type in the file name:** `.server.luau` for a Script, `.client.luau` for a LocalScript, plain `.luau` for a ModuleScript. A ModuleScript with children is a folder with an `init.luau` inside.
  - **Write `MANIFEST.md`:** a table of every file, its exact place in Studio (for example `ReplicatedStorage.Ragdoll.Balance`) and its script type. Note anything that must exist in Studio besides scripts.
  - **Build everything else from code:** the floor, lighting, spawn points, dummies and settings folders are created by scripts at start. No hand-built parts or models in Studio, and no `.rbxl` or `.rbxm` files in the repo. Then the scripts alone are the whole game.
  - **The `README` already exists and is written for the public,** because the user plans to make the repo public. Keep its style and structure: tick off features as they're done, fill in the controls and settings sections, and keep the Getting started steps accurate. Don't replace it with internal notes; those belong in `docs/`.
- **Start from scratch, on purpose.** An earlier engine exists, but the user is deliberately **not** giving you its code. They want a new design, not a cleaned-up copy of the old one. Design from first principles: what makes real people look real when they're hit, and what's the best way to get that in Roblox physics. Section 9 lists the old engine's mistakes, so you don't have to rediscover them.
- **Ask before you build.** Start with the questions in section 7, wait for the answers, then propose a design and plan in plain words.

### Before you start

- **Tools on your machine:** git, **ffmpeg** (to pull frames from the reference videos) and the **Luau CLI** (`luau`, `luau-analyze`). Install them if they're missing, or ask the user to.
- **The repo** may be empty except for `reference/videos/` (the reference clips) and this plan. Set up the structure yourself.

### Writing code you can't run in Roblox

- **Separate the maths from Roblox.** Keep everything that doesn't need the engine in plain Luau modules with no Roblox calls: springs and smoothers, balance and step planning, the reaction state machine, pose maths, the test metrics. Test them outside Roblox with the open-source **Luau CLI** (`luau`, `luau-analyze`) and a small test runner, with simple stand-ins for `Vector3` and `CFrame` where needed. Run the type checker (`--!strict`) and a linter (for example selene) on all code.
- **Keep the Roblox layer thin and careful:** creating parts and constraints, reading and writing physics, input. Double-check every Roblox API you use. Where you're not sure an API behaves as you expect, add a comment `-- VERIFY IN STUDIO:` and list it in `INTEGRATION.md`.
- **Fail loudly and safely.** Every system checks its setup at start and prints a clear message (which file, what's wrong) instead of failing silently. Each major system (reactions, steps, arms, falls, behaviour) can be switched off with a flag, so one broken part doesn't block testing the rest.
- **Every tuning value in settings** (see "Built to be tuned by settings" below).
- **Built-in tests:** one key or command runs the checks from section 8 and prints a short **PASS/FAIL report** to Studio's Output window, short enough to paste into a chat.
- **Aim for "tuned on the first run", not just "working".** Nobody can tune by eye until it's in Studio, so get as close as possible on paper:
  - Start from **physically plausible values**: realistic body proportions and mass distribution (the head about 8% of body mass, the torso about half), human joint ranges, and joint strengths and reaction times taken from biomechanics, not guessed.
  - Check each system offline: run the pure-Luau simulation pieces (springs, balance, step planning) through scripted scenarios and confirm the curves look right before shipping them.
  - Write **`TUNING.md`**: every config value, what it does on screen, its safe range, and which way to turn it for common complaints, for example "steps look choppy → raise X". The second chat will use this to tune in Studio quickly.
- **Tools for judging motion in Studio:**
  - **Debug drawing**, toggled by a key: the guiding pose as a ghost, the centre of mass, planned foot placements, which behaviour is active, and current strengths.
  - **Record and replay:** record the last few seconds of a dummy and replay them in slow motion, frame by frame, from any angle.
  - **A fixed random seed** for tests, so a failing reaction can be repeated exactly.
- **`INTEGRATION.md`:** a step-by-step checklist for the first time the code meets Studio. It covers how to bring the scripts into Studio (following `MANIFEST.md`), what should happen when you press Play, how to run the tests, every `VERIFY IN STUDIO` item, and what to paste back to you.
- **Small, traceable steps.** Commit often with clear messages. Tag each milestone, and keep a short `CHANGELOG.md` saying what changed and what to look at when testing it. Problems found during integration can then be traced to the milestone that caused them.

### Built to be tuned by settings

The plan is that **your first version gets tuned by someone else**: the second chat, working in Studio, adjusts the feel by changing settings, not by rewriting your code. Make that easy:

- **No magic numbers in the code.** Every value that affects how something looks or feels lives in settings: strengths, stiffness, damping, timings, distances, angles, thresholds, chances, blend speeds, step sizes, gravity, health and damage. If you're unsure whether a value counts, make it a setting.
- **Grouped and named in plain words,** by what you see on screen: for example `Impact`, `HeadSnap`, `TorsoTilt`, `Stagger`, `Steps`, `ArmsProtect`, `Hunch`, `Recovery`, `Falls`, `GetUp`, `Idle`, `Walk`, `Behaviour`, `Damage`. Each setting has a comment with its unit (studs, seconds, degrees, per second) and a safe range.
- **Live changes.** Changing a setting while the game is running takes effect immediately, with no restart and no respawn. Make settings changeable both in Studio's Properties panel (for example as Attributes on a settings folder) and from a script or the command bar, with one simple call such as `Settings.set("Steps.Duration", 0.35)` and `Settings.get(...)`. The second chat tunes through scripts, so the script route matters most.
- **Changes blend in** where it matters, so changing a setting mid-reaction doesn't itself cause a jerk.
- **Export.** One command prints all settings that differ from the defaults, as Luau that can be pasted straight into the settings file. Tuning done in a play session can then be saved into the repo.
- **Presets on top of settings.** Dummy personalities (toughness, courage, fighter and so on) are presets that override a subset of the settings, plus per-dummy randomness. Presets are tunable the same way.
- **Feature switches** for every major system (see "Fail loudly and safely" above), also in settings.
- **`TUNING.md`** (see above) explains every setting: what it changes on screen, its safe range, and which way to turn it for common complaints.

If a part of the feel can only be changed by editing code, for example the shape of a curve, consider exposing it anyway: as a few control points, an easing choice or a blend weight.

### Platform

- **This is a Roblox game,** written entirely in **Luau** and using Roblox's own physics. No other engine, and no external libraries or packages: everything is your own code.
- **The characters are standard R15 rigs** (15 body parts) with a Humanoid. You may use the Humanoid, replace it or turn parts of it off, but the dummies must look like normal Roblox R15 characters.
- **Single-player.** It will never be multiplayer. That means you can choose where the dummies' physics runs purely for smoothness and performance. For example, the player's client can own or simulate the dummies, which avoids replication stutter. Decide deliberately, and explain the choice. Note that Studio's Play mode runs a client and a server.
- **Realistic gravity (the user's choice).** Roblox's default gravity (196.2 studs/s²) is about 5× real life at human scale (1 stud is about 0.28 m, so real gravity is about 35 studs/s²). That makes falls look fast and light. Set gravity close to real life and adjust everything that depends on it, including the player's jump height and walk feel. Note that the speeds and distances in section 2 were measured in the reference game, under whatever gravity it uses. Match how they **look**, not the raw numbers.
- **Frame-rate independent.** The user's PC runs Studio at about 170 fps, and players may run at 60 or below. Everything must look the same at any frame rate: use the frame time everywhere. Remember that physics steps at its own fixed rate, separate from rendering frames.
- **Check current APIs:** the Roblox Creator documentation is at `create.roblox.com/docs`. Roblox adds and deprecates physics and animation APIs regularly. Prefer current ones: constraints such as `AlignPosition` and `AlignOrientation` over legacy `BodyMovers`, and `RunService.PreSimulation` / `PostSimulation` over deprecated events.

---

## 1. The goal

In the user's words: a physics engine that **"looks rotoscoped… like it's made to feel like a human getting hit"**, with motion that is **"smooth, without stuttering"**, the way the dummies move in the reference game, **"like they're mocapped"**.

That means three things at once:
- **Smooth and human:** the motion has weight, natural ease-in and ease-out, overlapping body parts and follow-through. It never jitters, snaps, teleports or flickers. Watching it should feel like watching mocap of a stunt performer.
- **Reactive:** the hit decides what happens: where it landed, which hand, how hard, which way, and the body's balance at that moment. No two hits look exactly the same, and a hit can always interrupt whatever the body is doing.
- **Intent:** a hurt person protects their face, backs off, steadies themselves, decides to fight back or run, and gets up when knocked down.

**The approach the user wants: a physics body with human motion.** The body is **always fully physical** and reacts to everything, while animation and control only *guide* it, so that its movement looks like mocap. Physics comes first, made to look human. That's not the same as "animation with some physics blended in at hits".

The models are **Half Sword, Blade & Sorcery and Boneworks / Bonelab** (fully physics-driven bodies) for how physical it should be, and **Red Dead Redemption 2** (Euphoria with mocap-quality motion) for how human it should look. Also Euphoria in GTA IV and the PuppetMaster asset for Unity: active ragdolls that try to stay upright and act, and that physics can overpower. The reference is the Roblox game **"Bloody Ragdoll Physics"**. On top of its motion, this engine adds **full gore**: blood, bruises and wounds, dismemberment and slicing (section 5b).

---

## 1b. Smoothness: exactly what the user means

This is the user's top priority, so here it is in their own terms. These answers come from direct questions to the user.

### The three faults the user hates

1. **Sudden jerks:** the body or a limb suddenly speeds up, stops or changes direction within a frame, like a hiccup in the motion.
2. **Robotic moves:** motion that starts and stops at full speed, with no ease-in or ease-out, like a machine.
3. **Choppy steps:** feet moving in quick start-stop steps instead of a flowing stumble or walk.

A fine constant jitter wasn't one of the complaints, but don't introduce one.

### What it should feel like

- **The pace is a sharp hit, then slow and human afterwards.** The moment of impact is instant and violent: the head snaps immediately, with no wind-up and no delay. Everything after that flows at a calm human pace. A whole reaction to one normal punch takes **about 3s** before the dummy is back to normal, like the reference.
- **The body is heavy and loose,** like a real, slightly dazed person. Weight carries through: when the torso is shoved, the hips, legs and arms are dragged along with real momentum. Limbs swing and settle naturally. It isn't a trained fighter absorbing the hit, and it isn't a floppy exaggerated ragdoll either.
- **Slight follow-through:** arms and head swing a little past where they're going and settle softly, the way real flesh and momentum do. That means "a little past and settle", not "wobble back and forth".
- **Hits flow from wherever the body is.** If a punch lands while the dummy is still reacting to the last one, the new reaction starts from the current motion and adds to it. Nothing restarts, resets or snaps to a starting pose.
- **Falls are realistic first.** A fall is fully physical and can be messy. The get-up can be more guided, as long as it **starts exactly from how the body landed** and blends out of it continuously.
- **Idle and walking can use Roblox's default animations** as the guide for the physical body. Starting and stopping must be smooth, though: no flicker, no instant switch between idle and walk, and the feet must not stutter.

### The trade-off rule

The user's answer to "which is worse if you can't avoid both": **a dull reaction is worse than a visible imperfection.**

- **Never** soften, delay or dampen the impact to hide a jerk. The hit must read instantly and strongly.
- Smoothness work belongs **after** the impact: in the recovery, the steps, transitions between behaviours, and how control comes back.
- Both are still faults. The goal is a violent, instant impact **and** a perfectly continuous motion afterwards.

### What this means for the tests

The smoothness tests in section 8 deliberately ignore the first 0.05s after a hit. That's the one moment that's allowed to be sudden. Everything after it must pass.

---

## 2. What the reference looks like

### Reference videos in the repo

The user will put recordings of the reference game in the repo, under `reference/videos/`. You can't watch video, but you can pull still frames out of a clip with **ffmpeg** (for example every 0.05s over a short window) and look at them in sequence. Do this, and study them closely:

| File | What it shows |
|---|---|
| `2026-11-22 01-17-29.mp4` | **Main reference.** Spaced-out normal punches on several dummies, about 33s. |
| `2026-11-22 01-11-18.mp4` | Four quick normal punches on one dummy. |
| `2026-11-21 23-02-33.mp4` | A dummy walks up and is hit, in slow motion. |
| `2026-11-21 23-06-07.mp4` | A stumble after a hit from the side. |
| `heavy hits.mp4` | Heavy punches. |
| `several hits.mp4` | Several hits in a row. |
| `idle walk bump.mp4` | Walking dummies bumping into the player. |

### What frame-by-frame analysis found

- **One continuous performance.** From the punch until the dummy stands again, **nothing changes speed in a single frame except at the moment the fist lands**. The head snaps; then shoulders, hips, steps, a hand to the face and straightening up all follow in one flowing movement. It never looks like separate phases handing over.
- **A typical normal head punch:**
  - the head snaps back;
  - the body is knocked back about **2–4 studs** with 1–2 stumbling steps (peak pelvis speed about 8–12 studs/s);
  - it folds into a **deep forward hunch (about 45–90°) with a hand on its face** and backs away for 1–2s;
  - it slowly straightens, the hand leaving the face last;
  - it stands for a moment, then comes back in, about **3–3.5s** after the hit.
- **Turning:** a hit turns a dummy at most about 90° (side-on), never a spin.
- **Toughness:** knocked down after about **3–4 normal punches**. It sits down, crumples, or stumbles back and falls, always **away from** the attacker. It dies only after a lot more (several knockdowns' worth).
- **Behaviour:** after 2–3 hits, a dummy either fights back or flees. Some approach with a guard up.
- **Getting up** is a human sequence (roll, hands and knees, a knee up, stand) and takes a moment; afterwards the dummy is groggy.

### The user's specific request for a hard punch

> "right now the dummy leans back after a right punch, same goes for the left. i want it to be more realistic in the way that if i punch someone in real life with a lot of force they stumble backwards and their torso tilts to the left and their head goes back"

The user has since clarified that the torso should tilt **away from the fist**, in whatever direction the punch actually pushes it, based on where and how the fist lands. So:
- the head goes back first;
- the torso tilts sideways in the direction of the push;
- the body stumbles back;
- a left punch naturally mirrors a right one, because the fist comes from the other side.

A plain "lean back" for every hit is exactly what the user doesn't want.

---

## 3. Hard requirements (non-negotiable)

These are the things the user has repeatedly rejected. How you avoid them is up to you.

1. **No stutter.** The only instant event is the hit itself. Nothing visible jumps, snaps or teleports while the dummy stands, falls or gets up. In particular:
   - no single-frame changes in speed;
   - no snapping between states, for example from lying to standing;
   - no flicker between walking and idle.
2. **Nothing a human body couldn't or wouldn't do.** Joints stay in human range. Arms always do something with a purpose (hang, balance, protect the face, guard, brace a fall). The user has specifically rejected:
   - a hand held up in the air that doesn't touch the face (it reads as a wave, a "stop" or a "hello");
   - arms thrown up wide with the palms forward (surrender);
   - an arm sticking straight out (T-pose);
   - a hand on top of or behind the head ("salute");
   - palms up (a shrug);
   - arms reaching forward like holding a tray;
   - an arm pointing at the sky while lying down;
   - instant spins;
   - falling forward onto the attacker;
   - sliding along the ground after landing.
3. **Every hit reacts to that hit.** Where it landed, which side the fist came from, how hard and which way all visibly change the reaction. A right and a left punch must not look the same.
4. **It must scale.** Eventually **10+ dummies** will walk around a map at once. The core must be designed so that dummies nobody is touching are cheap (for example a light animated mode), and full active-ragdoll simulation only runs where it's needed. You don't have to build the crowd, but your design must not make it impossible. It should run at 60 fps.

---

## 4. Everything else is your call

Architecture, state machine, how strength is lost and regained, whether to use animation clips or procedural poses, how balance and stepping work: all yours. If you think a different approach gives a better result than anything suggested here, take it, and explain it to the user in plain words.

One suggestion that's likely to matter for the mocap look: hand-authored poses made of a few joint angles tend to look procedural. Motion based on **real human movement** looks human by nature.

**The user won't record themselves** acting out motion (so no "Animation from Video" from their own footage), and new animations can't easily be uploaded to Roblox from your side. Some options that work without either:
- **Free motion-capture data, converted offline.** Public mocap libraries, such as the CMU Graphics Lab Motion Capture Database, include falls, stumbles, pushes, getting up and fighting moves. You can convert the data (for example from BVH files) to R15 joint rotations with a script on your machine, and store it in the repo as Luau data tables. The engine then plays those tables itself to guide the physical body: no Studio and no asset upload needed. **Check each library's license** before using it, and only use data the license allows in a game. List what you used in `CREDITS.md`.
- **Procedural motion done well:** physics plus good control (balance, stepping, joint springs with human timing) and carefully shaped guide poses, using what's known about how people react to being hit.
- **Roblox's default animations** for idle and walk (already allowed in section 1b).

Which mix to use is your call. Real mocap data, where a good match exists, is the most direct route to the "mocap" look.

### Research worth drawing on

These are published sources on the techniques behind Euphoria-style characters. Use the **ideas**, adapted to Roblox's physics; none of them is a recipe to copy. Read the ones you can reach, and rely on what you already know about the others.

**Overview**
- Geijtenbeek & Pronost (2012), *"Interactive Character Animation Using Simulated Physics: A State-of-the-Art Review"*, Computer Graphics Forum. A map of the whole field: joint control, balance, and tracking motion data.

**A physical body that follows motion data and reacts to hits** (the core of the "physics body, human motion" approach)
- Zordan & Hodgins (2002), *"Motion Capture-Driven Simulations that Hit and React"*, Symposium on Computer Animation. A simulated character tracks mocap with joint controllers, and the controllers' strength changes when it's hit.
- Zordan, Majkowska, Chiu & Fast (2005), *"Dynamic Response for Motion Capture Animation"*, SIGGRAPH. After a hit, the character is simulated, then blended back into a matching motion clip.
- Mandel (2004), *"Versatile and Interactive Virtual Humans: Hybrid Use of Data-Driven and Dynamics-Based Motion Synthesis"*, master's thesis, Carnegie Mellon University. Switching between animation and simulation for hits and falls.
- RootMotion's **PuppetMaster** documentation (Unity asset): a practical take on a muscle-driven ragdoll that follows an animated target, with pin weights, muscle weights, and losing and regaining balance.

**Balance and stepping**
- Pratt, Carff, Drakunov & Goswami (2006), *"Capture Point: A Step toward Humanoid Push Recovery"*. Where to step to stop falling after a push.
- Yin, Loken & van de Panne (2007), *"SIMBICON: Simple Biped Locomotion Control"*, SIGGRAPH. Simple, robust balance feedback for a physical walking character.
- Coros, Beaudoin & van de Panne (2010), *"Generalized Biped Walking Control"*, SIGGRAPH. Robust physical walking, stepping and push recovery.

**Smooth transitions and springs** (directly relevant to section 1b)
- David Bollo (2018), *"Inertialization: High-Performance Animation Transitions in Gears of War"*, GDC talk. Switching to a new target without a visible jump, by decaying the difference smoothly.
- Daniel Holden's articles at theorangeduck.com, especially *"Spring-It-On: The Game Developer's Spring-Roll-Call"*. Critically damped springs, smoothing and inertialization, with code-level explanations.

---

## 5. Scope: what you build, and what you leave room for

### You build (the core)

- **The active ragdoll:** an R15 dummy that stands, idles, walks and turns smoothly while staying fully physical. Roblox's default idle and walk animations may guide it.
- **Hit reactions** to a generic hit (section 2): impact, stagger with real steps, protect and hunch, recovery, back to normal. Interruptible by further hits.
- **Balance and falls:** knockdowns (sit-down, crumple, stumble-back), lying on the ground, protecting while down, getting up, groggy afterwards, failing to get up when badly hurt.
- **Damage:** knockdown after 3–4 normal punches, death after much more. **A dead dummy becomes a limp ragdoll and the body stays on the map** (no respawn by default; provide a debug command to clear or spawn dummies).
- **Simple per-dummy behaviour, enough to test the reactions:**
  - back off after a hit and come back after about 3s;
  - a guard up for fighters;
  - fight back or flee after 2–3 hits;
  - stay away while covering.
- **Personality parameters per dummy:** toughness, courage (fight or flee), fighter or not, reaction speed. Use a few presets with some randomness. The crowd layer will use these later.
- **Bumps:** when the player walks into a dummy, or dummies bump into each other, the dummy reacts physically with a small stagger or a step, as in `idle walk bump.mp4`.
- **The player can be hit (the user's choice):** when a dummy fights back, the player has **health, can be knocked down, can die, and respawns**. Keep it simple but solid. If it fits your design, the player's own body may use the same engine.
- **A minimal player and test scene:** a floor, a first-person player with visible arms, a **normal left and right punch** and a **heavy punch** (slower and stronger; it can knock a fresh dummy down), dummy spawning, debug commands and the test report.

### Leave clean extension points for (built later by the second chat)

- **Other attacks: kicks, grabs, shoves, throws.** Reactions should take a **generic hit description**, not "a punch": the part hit, the contact point, the direction, the impulse or strength, the attacker's position, the source side, and an attack type for flavour. A kick to the leg or a shove to the chest must then work through the same path. For grabs, the body must be able to be **held**: a part attached to something external, which the reaction and balance systems respect.
- **Crowds:** many dummies wandering a map, cheap when idle, waking up into full simulation when hit, bumped or near the player.
- **Group behaviour:** dummies that help each other, or flee, when one nearby is attacked. Expose events such as `Hit`, `KnockedDown`, `Died` and `GotUp`, and a way to give a dummy a goal (go here, attack this target, flee from this point).
- **A third-person camera,** switchable with first person. Nothing in the engine may assume first person.

Document the public interface in `docs/ENGINE.md`: how to spawn a dummy, apply a hit, read its state, listen to its events, set its goals and personality, and add a new attack.

### Milestones (suggested; reorder if you have a better plan)

1. **Repo and tools:** the folder layout and `MANIFEST.md`, the test scene built in code, punch controls, debug commands, the test report, the offline Luau tests, `INTEGRATION.md`.
2. **Standing:** idle, walk and turn, smooth and physical.
3. **Hit reactions:** normal and heavy punches to the head, chest and gut, including the side tilt, with real stumble steps.
4. **Protect and arms:** a hand to the face, hunching, balance with the arms, guard.
5. **Falls:** knockdowns, the ground, getting up, death.
6. **Behaviour and personality:** back off, return, fight back or flee, presets.
7. **Scale-readiness:** a cheap idle mode, waking up when hit, a quick performance check with 10–15 dummies.
8. **Gore:** blood, bruises and wounds, dismemberment, slicing, and the body reacting to its injuries (section 5b).
9. **Polish:** variety, documentation (`docs/ENGINE.md`).

You'll likely finish several milestones before the first test in Studio. If you're unsure how something will feel, don't guess silently: make it a config value, note it in `INTEGRATION.md`, and move on. The most important thing for the first Studio run is that pressing Play gives a dummy you can punch, even if some systems are switched off.

---

## 5b. Gore and injuries

The user wants **full gore**, and **you build all of it** as part of the core: the effects as well as the engine support. It's used by normal punches and, later, by abilities (blades, powers and so on) that the second chat will add. The same rule applies as for motion: describe the look, and leave the method to you.

### What the user wants

- **Blood:** splatter on hits that matches the hit's force and direction, drips, stains on the body and the ground, and pools under bodies that are bleeding or dead.
- **Bruises and wounds:** marks that appear **where** the body was hit and **build up** with repeated hits: bruises from blunt hits, cuts and gashes from blades, wounds that bleed. A dummy that has taken a beating should look like it.
- **Dismemberment:** limbs and heads can come off, cut cleanly or torn. The severed part is a physical object that falls, rolls and bleeds. The stump shows a wound and bleeds.
- **Slicing:** bodies and body parts can be cut **along the line of a blade or ability**, not only at the joints. Both pieces are physical, with visible cut faces.
- **Normal punches use gore too:** blood and bruises from punches, more from heavy punches. Dismemberment and slicing need enough force or the right attack (abilities). A punch never takes a limb off.

### As realistic as possible (the user's emphasis)

The gore must look **as realistic as Roblox can possibly make it.** Cheap, blocky "Roblox gore" is exactly what the user doesn't want. Specifically:

- **No cylinder blood puddles,** and no flat discs, spheres or plain red parts standing in for blood. Pools must look like real liquid on the ground: irregular, organic edges, spreading out over time from where the blood is coming from, following the ground, slightly glossy and wet, darker and thicker in the middle. They must not stick out above the floor, clip through it, or float.
- **No uniform red cubes or balls** for spatter. Blood droplets fly in arcs with gravity, stretch with their speed, and leave **splatter marks shaped by how they landed:** round drops when falling straight down, elongated streaks with tails at an angle, spray patterns from a hard hit.
- **Real blood colour, and it changes over time:** dark crimson, not bright red. Fresh blood is wet and glossy; older blood darkens and dries. Blood on bodies smears and runs downward.
- **Bruises look like real bruises:** soft-edged and blotchy. They darken and change colour as they build up (red, then purple and blue), sit exactly where the hits landed, and follow the body's surface. They aren't flat stickers.
- **Wounds and cut surfaces look like flesh,** not a flat colour: layered skin, tissue and, where it makes sense, bone at severed limbs. They are sized and angled to match the cut that made them.
- **Severed parts and stumps** bleed in a believable way (a spurt that slows down, then drips) and leave trails as they move.

Use every tool Roblox offers to get there: textures and decals with proper transparency, `SurfaceAppearance` (with PBR materials, normal maps and roughness for wetness), particles with good textures and physics, beams and trails, and runtime images or meshes where they're available. Where the best-looking method needs textures or meshes uploaded to Roblox, which you can't do from your machine, **create the image files in the repo** (for example generated or painted textures as PNG), list them in `INTEGRATION.md` with exactly how they're used, and use a decent fallback until they're uploaded.

The bar is games like those in section 1, not a typical Roblox game. If a choice is between "fast but obviously fake" and "realistic but harder", choose realistic, then make it fast.

### The body reacts to its injuries

Injuries aren't only visual. They change how the body moves, through the same physics:
- **A missing or badly hurt leg:** the dummy can't keep its balance on it. It falls, kneels or hops, and it crawls or stays down instead of standing up normally.
- **A missing or hurt arm:** it can't protect or brace with that arm. The other arm takes over, and the hurt arm may be clutched or hang.
- **Head off, or a fatal cut:** instant death; the body goes limp.
- **Wounds:** a hurt dummy may clutch a wound, limp or move more slowly, depending on where it was hit.
- **Missing parts never break the engine:** no errors, no exploding physics, no limb snapping back into place. Every system (balance, steps, arms, get-up, behaviour) must cope with any part missing at any moment, including in the middle of a reaction.

### For abilities (the second chat builds these later)

Make gore part of the **generic hit description** from section 5: a **damage type** (for example blunt, cut, pierce), and for cuts a **cutting line or plane**. Expose simple calls for abilities, along the lines of: apply a wound here, sever this part, slice along this plane, add blood here. Document them in `docs/ENGINE.md`.

### Roblox realities

- Roblox has no built-in way to cut a body in two. Choose a method deliberately (for example precut pieces, runtime geometry, or dynamic meshes), and explain the choice to the user. Any heavy work must never cause a visible hitch: a slice that freezes the game for a frame breaks the smoothness rule.
- Mark anything you can't confirm offline (for example limits on runtime meshes or images) as `VERIFY IN STUDIO`.
- **Performance:** blood, decals, severed parts and pools must have caps and cleanup, so 10+ dummies on a map stay at 60 fps.
- **A gore setting** with at least off, light and full, plus a switch per kind (blood, wounds, dismemberment, slicing), all in settings. That's for tuning and performance, and for Roblox's content rules: Roblox restricts realistic blood and dismemberment to experiences with a suitable maturity rating. Note this in the README so the user can set the experience's rating correctly before publishing.

---

## 6. What comes back to you after integration

- errors and warnings from Studio's Output window;
- the test report from section 8;
- review files from the second chat, which records the game and analyses it frame by frame. Those reviews describe **what is visibly wrong**, with timestamps; their code suggestions are optional;
- the user's own impressions.

---

## 7. Questions: what's settled and what's open

**Already answered by the user (don't ask again):**

| Topic | Answer |
|---|---|
| Platform | Claude Code in the GitHub repo, no Roblox Studio while you work |
| Players | Single-player only |
| Approach | A fully physical body, guided to move like mocap (section 1b) |
| Smoothness | See section 1b |
| Torso tilt direction | Away from the fist, in the direction the punch pushes |
| Torso tilt amount | **Clearly visible, about 15–25°** for a normal punch (more for a heavy one) |
| Head | **Depends on the hit:** a hit to the side of the jaw turns the head away along the fist's path; a straight hit to the face sends it straight back |
| Stumble | **Every normal punch causes 1–2 steps back**, like the reference |
| Reaction length | About 3s per normal punch |
| Violence | **Full gore**: blood, bruises and wounds, dismemberment and slicing. You build it all (section 5b). Punches cause blood and bruises; dismemberment and slicing need enough force or an ability |
| Punch feel on the player's side | **Nothing extra:** no camera kick, no hit stop, no sounds for now. The dummy's reaction alone must sell the hit |
| Controls | Your choice; document them in the README |
| Self-recorded motion | **No:** the user won't record themselves acting out motion (see section 4 for alternatives) |
| Dummy looks | Doesn't matter yet; plain R15 dummies are fine |
| Toughness | Like the reference: down after 3–4 normal punches, dead after several knockdowns' worth |
| Death | A limp body that stays on the map |
| Gravity | Realistic (about 35 studs/s² at R15 scale), not Roblox's default |
| The player being hit | Health, can be knocked down and die, respawns |
| Dummies | 10+ later, wandering a map; some personality variety |
| Camera | Switchable first and third person (the second chat adds third person) |
| Division of work | You build the core; another chat extends it (section 5) |

**How to work with the user:** ask your questions first and show the user your design in plain words. Once they agree, **work through as many milestones as you can without stopping**: don't stop after each one to report. Keep `CHANGELOG.md` up to date instead, and summarise everything at the end.

**Still open (ask if it matters to your design):**
1. Which moment in the reference looks most "right" to you?
2. Anything else you'd like to add, now that you've seen the plan?

Ask anything else you need, too.

---

## 8. How "done" is measured

Build these into the in-game test report. Exact thresholds can be tuned with the user.

**Smoothness** (logged every frame, over 10 normal head punches, and across falls and get-ups):
- **Pops:** a body part changing speed by more than about 4 studs/s in one frame. **Exclude** the first 0.05s after a hit, and real contacts: a body landing on the ground, a foot planting, bodies bumping into each other. Those are allowed to be sudden. Measure speed changes per second of time, not per frame, so the threshold means the same at any frame rate. Target: **0**.
- **Teleports:** any visible part, or anything the body follows, moving more than about 0.2 studs in one frame, outside spawning. Target: **0**.
- **Walk flicker:** switches between walking and not walking during a single reaction. Target: **at most 1**.

**Reaction shape:**
- Right- and left-hand head punches tilt the torso clearly to opposite sides, with the head moving back before the torso tilts.
- Knock-back of a normal punch: about 2–4 studs.
- After head punches, a hand on the face most of the time between about 0.5s and 2s.
- Hands are never above the head, and never above the shoulder unless on the face.

**Behaviour:**
- Knockdown on the 3rd or 4th normal punch in most runs; death only after several knockdowns' worth of damage.
- Every fall lands away from the attacker.
- No walking back in while covering, or within about 3s of a hit.
- No turning faster than about 200°/s while upright; no hit turns the dummy more than about 90°.

**On the ground:** no sliding while down; no hand raised in the air while lying; gets up within about 1.5–3s.

**Scale:** 10–15 idle or walking dummies plus one being hit, at 60 fps in Studio.

**Gore:**
- Wounds and bruises appear where the hit landed and build up.
- No primitive blood shapes (cylinders, discs, spheres, plain red parts). Pools have organic edges and lie flat on the ground without clipping.
- Severing or slicing any part at any moment, including mid-reaction, causes no errors, no physics explosion and no frame hitch.
- A one-legged dummy can't stand normally; a one-armed dummy protects with the other arm.
- 10+ bloodied, dismembered bodies on the map still run at 60 fps.

**Frame rate:** the smoothness and reaction-shape results are the same at 60 fps and uncapped (150+ fps).

**The real test:** the user records normal right- and left-hand punches, spaced out, through to a knockdown and get-up. Side by side with the reference, **it flows like the reference and looks like a real person being hit.**

---

## 9. Pitfalls the earlier engine ran into

These are problems that showed up in the earlier engine and how they looked. They're not a description of its design: don't rebuild it, just avoid repeating these.

- **Holding the body too firmly cancels the hit.** If whatever keeps the body upright stays strong during the impact, a jab is undone within a few frames, and punches look like the head just "bops". The body has to be nearly free at the moment of impact and regain control gradually.
- **Most of the stutter came from switches, not from the physics.** Examples that were measured:
  - strengths jumping from 0 to full in one frame when one phase ended and the next began;
  - the "intent" being teleported onto the body every frame while stumbling (jumps of 2–3 studs per frame);
  - quick catch steps (0.18s) that yanked the foot onto its path and then stopped it dead;
  - arm behaviours switching in one frame;
  - the walk animation flickering on and off around a small dead zone.

  Smoothing these cut the jolts by about 5×, but they were patched on afterwards. Design for continuity from the start.
- **Every punch made the dummy lean back** because the punch direction was almost straight forward with a small upward lift, and the reaction never knew where the fist came from.
- **Hand-keyed reaction poses (a few joint angles per key) looked procedural,** and put hands in places a person wouldn't, such as beside the head instead of on the face.
- **Rest poses defined relative to the head made an arm point at the sky** when the body lay on its back.
- **Downed bodies colliding with the player** caused problems; so did internal helper parts colliding with the visible body.
- **A normal punch once launched a dummy about 15 studs.** Keep impulses in proportion.
- **When judging recordings,** an apparent "slide" turned out to be the camera moving.
