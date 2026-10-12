# Technical spec

The numbers, schemas and rules the build agents implement. When this spec and code disagree, this spec wins; change it in the design doc first, then re-export.

## Repo layout and conventions

```text
derelict-blade/
  CLAUDE.md                 agent rules
  CHANGELOG.md
  docs/                     exported design docs + proposals/
  content/
    tuning.json             every gameplay number
    input.json              default bindings
    strings.json            UI text
    world.ldtk              all rooms (LDtk project)
    blades.json  tools.json  pickups.json  shop.json  upgrades.json
    enemies/<id>.json       one file per enemy
    bosses/<id>.json        one file per boss
    npcs/<id>.json          NPC definition + dialogue
    dialogue/<scene>.json   cutscenes and story scenes
    logs/<deck>.json        lore logs, verbatim from docs/04-lore-logs.md
    quests.json  flags.json  music.json  sfx.json
  src/
    main.ts  config.ts
    scenes/                 Boot Preload Title Game UI Pause Dialogue Terminal Map Ending Credits
    systems/                input camera rooms combat health flags save audio lighting telemetry power gravity beams
    entities/               Player Enemy Boss Npc Pickup Interactable
    entities/behaviors/     composable enemy states
    entities/bosses/        one controller per boss
    data/schemas/           zod schemas, one per content type
    data/loaders.ts
  assets/
    palette.hex  palette.gpl  manifest.json
    sprites/  tiles/  portraits/  ui/  fx/  fonts/
    audio/music/  audio/sfx/
  tools/
    validate.ts  solvability.ts  text-check.ts  palette-check.ts
    palette-snap.py  sprite-pack.ts  loop-trim.py
    bot/                    Playwright bot: smoke, route, full
  worker/                   Cloudflare Worker for telemetry (D1)
  tests/
  .github/workflows/ci.yml
```

**Conventions**

- IDs are `snake_case` and stable forever: `enemy.spark_wisp`, `room.rx_14`, `flag.met_archivist`, `log.cg_03`.
- Room ids are `<deck>_<nn>`: `db` Docking Bay, `hb` Hub ring, `hy` Hydroponics, `rx` Reactor Ring, `cg` Zero-G Cargo, `ds` Data Spire, `cr` Core.
- TypeScript strict mode, no `any`. Systems talk through a typed event bus (`events.ts`), never by reaching into each other.
- Commits use Conventional Commits (`feat:`, `fix:`, `content:`, `art:`, `audio:`, `tune:`).

## Rendering, camera and controls

- **Internal resolution:** 320×180, integer-scaled (×4 at 1280×720, ×6 at 1920×1080), letterboxed, nearest-neighbor filtering.
- **Screen layout:** 16-px HUD strip on top; the room view is 320×160 = 20×10 tiles of 16×16 px.
- **Rooms:** one screen by default. Large rooms (bosses, hub plazas) are multiples of a screen, and the camera follows the player within them.
- **Transitions:** walking off an edge slides the camera to the next screen over 0.4 s, freezing gameplay. Doors fade to black over 0.25 s.
- **Depth:** sort by the y of each sprite's feet. Tall props use a split top layer so the player can walk behind them.
- **Frame rate:** fixed 60 Hz update, rendering decoupled.

| Action | Keyboard | Gamepad |
| --- | --- | --- |
| Move | WASD / arrows | Left stick / D-pad |
| Attack (hold to charge) | J or Z | A |
| Dash | Space | B |
| Tool slot 1 | K or X | X |
| Tool slot 2 | L or C | Y |
| Interact / talk | E | A (when prompted) |
| Previous / next blade | Q / R, or 1–5 | LB / RB |
| Map | Tab | Back / View |
| Pause / inventory | Esc | Start / Menu |

When an interact prompt is showing, the attack button interacts instead of attacking. All bindings are remappable in Phase 5.

## Tuning values

Starting values for `content/tuning.json`. Phase 5 changes them only with telemetry behind each change. Distances are in px at internal resolution (1 tile = 16 px), times in seconds, damage in half-cells.

**Player**

| Key | Value | Note |
| --- | --- | --- |
| `player.walkSpeed` | 80 px/s | 5 tiles/s |
| `player.accel` / `player.decel` | 900 / 1200 px/s² | Snappy, a tiny ease |
| `player.dashSpeed` | 220 px/s | |
| `player.dashTime` | 0.16 s | Invulnerable for the first 0.10 s |
| `player.dashCooldown` | 0.55 s | |
| `player.hitbox` | 10×8 px at the feet | Sprite is 16×24 |
| `player.iFrames` | 0.9 s | Sprite blinks at 15 Hz |
| `player.knockback` | 150 px/s, decays over 0.15 s | |
| `player.startCells` / `player.maxCells` | 3 / 12 | 1 cell = 2 half-cells |
| `player.inputBuffer` | 0.12 s | Attack and dash presses buffered |
| `player.fallRespawnDamage` | 1 | Pits respawn at the room entrance |

**Combat**

| Key | Value |
| --- | --- |
| `combat.hitstop` | 0.05 s per hit, 0.09 s on a kill, 0.12 s on a boss hit |
| `combat.shake` | 2 px for 0.10 s on hit taken; 3 px for 0.20 s on boss hit |
| `combat.enemyKnockback` | 120 px/s over 0.12 s (heavy enemies: 0) |
| `combat.chargeTime` | 0.8 s to full charge |
| `combat.enemyContactDamage` | 1 unless the enemy file overrides it |

**Economy and drops**

| Key | Value |
| --- | --- |
| `drops.cellChance` | 0.12; 0.40 when the player is at 2 half-cells or below |
| `drops.scrapChance` | 0.35 (1 scrap, 10% chance of 5) |
| `drops.pityKills` | Guaranteed cell drop after 8 kills without one while below max |
| `shop.cellRefill` | 10 scrap per full refill |
| `shop.flare` / `shop.emp` | 5 / 8 scrap; carry 5 / 5 (upgrade to 10 / 10) |
| `shop.cellFragment` | 120 scrap each, 2 in stock; 2 fragments = 1 cell |
| `forge.resonanceUpgrade` | 60 / 120 / 200 scrap for levels 1–3 per blade |
| `scrap.cap` | 250 (upgrade to 500) |

**Difficulty targets (for tuning passes)**

| Measure | Target |
| --- | --- |
| First-attempt death rate, normal room | under 15% |
| First-attempt death rate, boss | 40–70% |
| Median boss attempts | 2–3 |
| Median time per normal room | 20–60 s |
| Quit rate in any single room | under 3% |

## Combat and blades

Every blade shares one swing system. Blades differ in damage, speed, arc, and a **resonance**: the charged attack, which also opens that blade's world interactions.

**Swing:** windup, active frames, recovery. The hitbox is an arc in the facing direction. Attack chains: tapping again during recovery starts a second swing (2-hit combo; the second swing does +1 damage). Holding attack charges; at full charge the blade glows in its color and releasing fires the resonance.

| Blade | Damage | Windup / active / recovery (s) | Arc and reach | Resonance (charged) | World interaction | Color |
| --- | --- | --- | --- | --- | --- | --- |
| Ferryman's Seax | 2 | 0.03 / 0.08 / 0.07 | 120°, 18 px | none (Seax can't charge) | Cuts cables, loose grates, rope bridges | steel white |
| Thornwood Gladius | 2 | 0.04 / 0.10 / 0.08 | 90°, 20 px | **Thrust:** dashes 32 px forward, 4 damage, pierces armor | Cuts vine-cable walls | green |
| Storm Katana | 3 | 0.04 / 0.10 / 0.08 | 150°, 22 px | **Arc:** slash that conducts current to all enemies within 48 px of the target, 3 damage each, stuns Shells 1.5 s | Bridges or cuts conduits; grounds Spark Wisps | cyan |
| Anchor Claymore | 4 | 0.10 / 0.12 / 0.14 | 200°, 26 px | **Anchor:** spin slam, 5 damage, creates a gravity well (64 px radius, 2.5 s) that pulls in drifting objects and enemies | Pins crates and platforms in zero-g | amber |
| Mirror Shamshir | 3 | 0.04 / 0.09 / 0.07 | 130°, 22 px | **Mirror parry:** 0.25 s parry window; reflects bolts and lasers back at the source for double damage | Redirects beams into receivers | magenta |

**Forge upgrades (per blade, 3 levels):** level 1 shortens charge time to 0.6 s, level 2 adds +1 resonance damage, level 3 adds a blade-specific bonus (Seax: +1 swing damage; Gladius: thrust pierces 2 enemies; Katana: chain range 72 px; Claymore: well lasts 4 s; Shamshir: parry window 0.35 s).

**Tools** (equip in either slot)

| Tool | Effect | Ammo |
| --- | --- | --- |
| Light flare | Thrown 64 px; lights a 112-px radius for 8 s; stuns light-sensitive enemies 2 s | 5 (upgrade 10) |
| EMP charge | Placed; detonates after 1.5 s; breaks EMP walls; disables Shells and shields in 40 px for 3 s; 4 damage | 5 (upgrade 10) |
| Mag-grapple | 96-px line; pulls light objects to you or you to heavy ones | unlimited |
| Gravity boots | Toggle; walk on marked plating in zero-g; can't dash while on | unlimited |
| Scanner | 3-s pulse; reveals hidden panels, in-room shards and weak points; 2-s cooldown | unlimited |

**Juice list (Phase 1, task 1.14):** hitstop and shake per tuning; hit flash (1 frame white) on enemies; 6–10 spark particles per hit in the blade's color; scrap and cells magnetize to the player within 24 px; the charge glow pulses faster as it nears full; a 1-frame freeze and flash on a perfect parry; death slowdown to 0.3× for 0.4 s; a low-health heartbeat SFX and red vignette at 2 half-cells or below; footstep dust on dash; a camera nudge of 2 px toward the attack direction.

## Enemy roster

16 enemies, built from shared behavior states. Every attack has a visible tell of at least 0.3 s (0.25 s in the Core). HP is in damage points; Seax and Gladius hit for 2.

| ID | Name | Deck | HP | Dmg | Speed (px/s) | Behavior | Tell | Weakness / counter | Size |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `crawler` | Crawler | Docking | 2 | 1 | 30 / lunge 140 | Wanders; lunges 48 px when the player is within 56 px | Crouches 0.4 s | Any blade | 16×16 |
| `turret` | Wall Turret | Docking | 4 | 1 | static | Fires a bolt (110 px/s) every 1.6 s when it has line of sight | Barrel glows 0.5 s | Hit from the side or behind; reflected bolts later | 16×16 |
| `vine_drone` | Vine Drone | Hydroponics | 3 | 1 | 50 | Sine-wave flight toward the player, retreats after contact | Wings flare 0.3 s before diving | Any blade; flare stuns | 16×16 |
| `pruner` | Pruner | Hydroponics | 6 | 2 | 40 / charge 160 | Armored front; charges in a line, stuns itself 1.2 s on hitting a wall | Shears open 0.5 s | Gladius thrust pierces the front; otherwise hit from behind | 24×24 |
| `spore_pod` | Spore Pod | Hydroponics | 1 | 1 | static | Bursts when hit, leaving a 32-px poison cloud for 1.5 s | Swells when the player is near | Hit from range (thrust) or dash through after the burst | 16×16 |
| `spark_wisp` | Spark Wisp | Reactor | 2 | 1 | 70 | Follows conduits, electrifying them; leaves conduits to chase within 64 px | Crackles 0.3 s | Katana charged slash grounds it (instant kill) | 16×16 |
| `coil_guard` | Coil Guard | Reactor | 8 | 2 | 35 | Shield up; drops shield for 1 s after its baton swing | Raises baton 0.5 s | Katana resonance passes through the shield; EMP drops it 3 s | 16×24 |
| `arc_mine` | Arc Mine | Reactor | 1 | 2 | static | Explodes 0.8 s after the player comes within 24 px, chaining to mines within 48 px | Blinks faster | Trigger the chain from a safe distance | 16×16 |
| `drift_shell` | Drift Shell | Cargo | 4 | 1 | 60 drift | In zero-g, moves in straight lines and bounces off walls | Thruster flash on each bounce | Claymore well catches it; easy to hit when pinned | 16×16 |
| `loader_arm` | Loader Arm | Cargo | 10 | 2 | static base | Grabs crates within 80 px and throws them at the player | Claw opens 0.6 s | Hit the base during the throw recovery (1.0 s) | 32×32 |
| `magnet_bot` | Magnet Bot | Cargo | 5 | 1 | 40 | Pulls the player toward it at 60 px/s within 72 px | Ring glows 0.4 s | Gravity boots resist the pull; Claymore slam stuns | 16×16 |
| `lens_sentry` | Lens Sentry | Spire | 4 | 2 | rotates 45°/s | Sweeping laser beam | Beam aims as a thin line for 0.6 s | Shamshir parry reflects the beam back (instant kill) | 16×16 |
| `mimic` | Mimic Hologram | Spire | 3 | 1 | player speed | Copies the player's inputs with a 0.5-s delay | Flickers when about to swing | Stand still to make it idle, then strike; can press plates | 16×24 |
| `censor` | Censor | Spire | 6 | 1 | teleports | Teleports every 2.5 s; its pulse disables the player's tools 3 s | Static ring 0.4 s before the pulse | Scanner reveals its next position | 16×24 |
| `warden_knight` | Warden Knight | Core | 12 | 2 | 55 | Cycles through Pruner armor, Coil shield and Sentry beam stances | Visor color shows the stance | Use the blade that counters the current stance | 24×32 |
| `guide_swarm` | Guide Swarm | Core | 1 each, groups of 6 | 1 | 90 | Surrounds the player; while 3+ are alive the screen distorts and the controls drift slightly | Whisper SFX | Katana chain or Claymore well clears them in one hit | 8×8 each |

Later decks reuse earlier enemies in new mixes (for example, Coil Guards in Cargo, Crawlers in zero-g). The deck briefs specify each deck's mix.

## Bosses and mini-bosses

Every boss teaches its deck's blade under pressure. Health bars show phase breaks. Each boss arena has 2 cell drops the first time and no drops on retry, plus a checkpoint at the arena door.

**Gardener Mk IV (Hydroponics)**: 36 HP, 3 phases, 64×64.

- Phase 1 (36–24): sweeps shears in a 180° arc (0.6-s tell, shears raised). Its armored front blocks swings; Gladius thrust pierces it.
- Phase 2 (24–12): plants 4 vine pods that grow into walls over 3 s, shrinking the arena. Cutting a pod before it grows stuns the Gardener for 2 s.
- Phase 3 (12–0): regrows a severed limb every 8 s; thrust the glowing joint before it regrows. Adds 2 Vine Drones.

**Foreman (Reactor mini-boss)**: 20 HP, 32×32. Directs 3 Coil Guards by whistle. Kill the guards with Katana chains, then the Foreman is defenseless for 4 s.

**Coil Saint (Reactor)**: 48 HP, 3 phases, 64×80. An android preacher wired to 4 Tesla coils.

- Phase 1: invulnerable while any coil is powered. Cut the 4 conduits to its coils (Katana) while dodging arc bolts (0.5-s tell).
- Phase 2: re-powers 2 coils at random every 10 s; Katana charged slash on the Saint while it channels deals double damage.
- Phase 3: floods the floor with current in a checkerboard (1-s tell); stand on unpowered tiles and use the EMP to stop the flood for 4 s.

**Loadmaster (Cargo)**: 54 HP, 3 phases, in zero-g. A crane core with 3 detachable arms.

- Phase 1: arms throw crates. Claymore well catches crates; bat them back into the core for 6 damage.
- Phase 2: arms detach and drift as separate enemies (10 HP each); grapple to the core between them.
- Phase 3: the arena's gravity flips every 6 s with a 1-s klaxon; use gravity boots to stay on the core.

**Auditor (Spire mini-boss)**: 24 HP, 32×32. Fires a slow "verdict" laser that tracks the player; parry it back with the Shamshir 3 times.

**WARDEN Avatar (Spire)**: 60 HP, 3 phases, a hologram in Kesh's shape.

- Phase 1: mirrors the player's movement exactly. Mirrors in the arena let the player attack its reflection's position.
- Phase 2: predicts the player. Its attacks aim where the player will be in 0.5 s, so changing direction late beats it. A Censor appears every 15 s.
- Phase 3: fires 4 rotating lasers; reflect each back with the Shamshir to break the projector.

**WARDEN Core (Release path only)**: 90 HP, 3 phases, uses every blade.

- Phase 1 "Consensus": 6 armored nodes, each needing a different blade. Its voice narrates what it predicts you will do next, and it is often right.
- Phase 2 "Assurance": Guide Swarms and soft whispered prompts on screen ("rest now"). The controls drift while 3+ swarms live.
- Phase 3 "Override": WARDEN stops attacking and simply asks the player to stop. To win, the player walks to the Human Override lock and performs a 5-blade sequence by hand (each blade's charged attack in a set order). No damage is dealt in this phase.

Each boss file in `content/bosses/` holds its HP, phase thresholds, attack list (name, tell time, active frames, cooldown, damage) and arena id.

## Station systems

Each system is a self-contained module in `src/systems/` with unit tests for every rule below. Rooms declare which systems they use in their LDtk fields.

**Power (from the Reactor deck on)**

- Power is a graph: sources, conduit tiles, junctions, and consumers (doors, lifts, lights, hazards, turrets). Power flows instantly through connected conduit.
- A Katana charged slash on a **broken** conduit bridges it for 10 s (amber glow). On a **live** conduit it cuts it permanently until the room resets.
- Rotatable junctions (struck with any blade) turn 90° clockwise.
- Live conduits damage the player on contact (1). Spark Wisps travel on live conduit only.
- Every power room has a reset switch by the entrance that restores the initial state, so no power room can become unwinnable.

**Gravity (Cargo deck on)**

- Rooms or zones are `normal` or `zero_g`. In zero-g: acceleration 300 px/s², drag 0.92 per frame, max speed 120 px/s, and the dash becomes a push that continues until you hit something.
- Gravity-flip switches toggle a zone between states with a 1-s klaxon tell.
- Gravity boots let you walk on marked plating; while they're on, movement uses normal rules on plating.
- Objects (crates, debris, enemies without thrusters) keep momentum and bounce with 0.6 restitution.
- Every zero-g room has an anchor point or exit reachable by drifting from any position, checked by the validator.

**Light (Hydroponics on)**

- Rooms set `ambient` from 0 (pitch black) to 1. Below 0.3 the room counts as dark: enemies are visible only by their glow, and hidden hazards stay unlit.
- The player carries a 48-px lantern glow. Flares light 112 px for 8 s. Neon signs and powered lights are light sources with radius and color.

**Beams (Data Spire)**

- Beams travel in straight lines from emitters until they hit a wall, mirror (45° deflect, rotatable by striking it), splitter (one beam into two at 90°), or receiver (opens linked doors while lit).
- The Shamshir parry reflects a beam back along its path for 0.5 s; a parried beam that hits its emitter destroys it.
- Beams damage the player (2) on contact, except during the parry window.

**Air (rare set pieces only)**

- Vacuum rooms show an O₂ meter: 20 s of air, refilled instantly at O₂ stations or in any pressurized room. At 0 the player takes 1 damage per second.
- Used in at most 3 rooms in the whole game.

## Room grammar and deck briefs

Agents build decks by composing typed rooms on a graph, then filling each room from the puzzle primitives. They never free-form a layout.

**Room types:** `entry`, `corridor`, `combat` (doors seal until enemies are cleared), `puzzle`, `key` (holds a small key), `lock` (small-key door), `map` (map terminal), `blade` (the deck's blade, behind a mini-challenge), `miniboss`, `boss_key`, `ante` (save point before the boss), `boss`, `secret` (hidden, holds a shard or scrap cache), `shortcut` (opens a one-way route back to the entry), `log` (terminal on the main path), `rest` (no threats, a breather).

**Deck rules (all decks)**

1. The entry connects to the hub ring. The map room is within 3 rooms of the entry.
2. 2–4 small keys; each small key is reachable before the lock it opens.
3. The blade room sits 45–60% of the way along the main path. At least 3 rooms after it require the new blade, and at least 1 room before it shows a blade gate the player can't pass yet.
4. One shortcut back to the entry opens after the blade room.
5. No more than 2 combat rooms in a row; a rest or puzzle room follows every 3 threat rooms.
6. One secret room per deck, found with a hint (a cracked wall, an odd tile, or a log that mentions it).
7. The boss key is in the deck's hardest puzzle room. The ante room has a save station.
8. Exactly 2 terminal logs on the main path; shards as listed in docs/04-lore-logs.md.

**Deck briefs**

| Deck | Rooms | Shape | Keys | Enemy mix | Primitives | Set pieces |
| --- | --- | --- | --- | --- | --- | --- |
| Docking Bay `db` | 6 | Linear, one side branch | 0 | Crawler, Turret | P01, P02, P13, P29 | The crash in your tug's cockpit; the Seax in a display case smashed by the impact; a turret gauntlet corridor; the first NAV-ASSIST arrow; the airlock to the hub |
| Hub ring `hb` | ~40 screens, 9 districts | Ring with 5 spokes | spokes open by story | Crawlers outside the market only | P02, P13, P20, P27 | Choir Market (shops, forge, NPCs), the Vault (Archivist), the Arena, the Sleeper Gallery (pods behind glass), Dock Spoke, Garden, Reactor, Cargo and Spire spokes, Core Gate |
| Hydroponics `hy` | 16 | Two wings joined by a central greenhouse | 2 small, 1 boss | Vine Drone, Pruner, Spore Pod, Crawler | P01, P02, P05, P10, P14, P15, P26 | A monoculture field of identical grey plants; a dark root tunnel (flare); a greenhouse where WARDEN's sprinklers water nothing; the Gladius growing inside a tree |
| Reactor Ring `rx` | 20 | Ring around a central core with a sealed middle | 3 small, 1 boss | Spark Wisp, Coil Guard, Arc Mine, Crawler | P03, P04, P09, P11, P16, P24, P25 | A control room of workers' chairs facing a single button; the Foreman's shift floor; the reactor heart reachable only after the Katana; a room-sized power routing grid |
| Zero-G Cargo `cg` | 22 | Vertical stack of holds linked by drifting shafts | 3 small, 1 boss | Drift Shell, Loader Arm, Magnet Bot, Coil Guard | P06, P07, P12, P17, P22, P23, P28 | A pin-factory assembly line still running with no workers; a floating graveyard of crates; a vacuum crossing (O₂); the Claymore anchoring a broken hull plate |
| Data Spire `ds` | 22 | Tall tower, 5 floors, each a ring of rooms | 3 small, 1 boss | Lens Sentry, Mimic, Censor, Spark Wisp | P08, P18, P19, P20, P21, P26, P30 | The Mirror Hall of citizens' predicted futures; a cave room where screens cast shadows on a wall; the Auditor's courtroom; the Shamshir in a vault of mirrors; the NAV-ASSIST reveal |
| Core `cr` | 18 | Descending spiral toward WARDEN | 2 small, no boss key (override lock) | Warden Knight, Guide Swarm, all earlier types | Combines 2–3 primitives per room, every blade required | The Sleeper vault; the room where WARDEN plays your own past choices back to you; the Human Override lock |

## Puzzle primitives

30 tested building blocks. Each is an LDtk entity (or entity group) with a unit test proving it can always be solved and reset. Rooms combine 1–3 primitives; Core rooms may use more.

| ID | Name | Rule | Needs | First deck |
| --- | --- | --- | --- | --- |
| P01 | Pressure plate | Opens linked doors while weighted by the player, a block, a crate or a mimic | none | Docking |
| P02 | Push block | Moves 1 tile per push in the facing direction; resets on room re-entry | none | Docking |
| P03 | Conduit bridge | Katana charge on a broken conduit powers it for 10 s | Katana | Reactor |
| P04 | Conduit cut | Katana charge on a live conduit cuts it, de-powering everything downstream | Katana | Reactor |
| P05 | Dark room | Ambient below 0.3; hazards and paths visible only by light | Flare | Hydroponics |
| P06 | Crate drift | Zero-g crates keep momentum; land one on a plate or in a socket | Zero-g | Cargo |
| P07 | Gravity flip | A switch toggles a zone's gravity with a 1-s tell | none | Cargo |
| P08 | Beam mirror | Rotatable 45° mirror; strike to turn 90° | Beams | Spire |
| P09 | Timed door | A switch opens a door for 4–8 s | Dash helps | Reactor |
| P10 | Clear-room door | Doors seal until all enemies are defeated | none | Hydroponics |
| P11 | EMP wall | A cracked, glowing wall that only an EMP breaks | EMP | Reactor |
| P12 | Grapple post | Pull yourself across a gap or pull a crate to you | Mag-grapple | Cargo |
| P13 | Cable cut | Cut a hanging cable to drop a bridge, a platform or a weight | Seax+ | Docking |
| P14 | Vine wall | Vine-cable growth that only a Gladius thrust clears | Gladius | Hydroponics |
| P15 | Lever sequence | 3–4 levers must be set in an order hinted in the room | none | Hydroponics |
| P16 | Floor hazard cycle | Electrified floor sections toggle on a 2-s rhythm | Timing | Reactor |
| P17 | Conveyors | Belts move the player and objects at 60 px/s | none | Cargo |
| P18 | Laser grid | Beams cross a corridor; parry to pass or reflect into a receiver | Shamshir | Spire |
| P19 | Mimic plate | A Mimic copies your moves; route it onto a plate you can't reach | Spire | Spire |
| P20 | Hidden panel | A wall panel or floor hatch visible only to the scanner | Scanner | Spire (hub revisit) |
| P21 | Beam splitter | Splits one beam into two at 90° to light 2 receivers | Beams | Spire |
| P22 | Fan current | Fans push drifting objects and the player in zero-g | Zero-g | Cargo |
| P23 | Magnet floor | Pulls metal objects and the player; gravity boots cancel it | Boots | Cargo |
| P24 | Power routing grid | Rotate junction tiles to route power from source to consumer | Power | Reactor |
| P25 | Twin plates | 2 plates must be held at once (block, crate, mimic or a pinned enemy) | none | Reactor |
| P26 | Lights-out | Striking a panel toggles it and its neighbors; make all lit | none | Hydroponics |
| P27 | Log password | A door asks a word or number found in a nearby log | Reading | Hub |
| P28 | Airlock timer | Cross a vacuum room on 20 s of air | O₂ | Cargo |
| P29 | One-way ledge | Drop down only; used for shortcuts and loops | none | Docking |
| P30 | Bolt redirect | Parry turret bolts into targets to hit switches | Shamshir | Spire |

## Data schemas

Zod schemas in `src/data/schemas/` match these examples exactly. Fields shown are required unless marked optional in a comment.

**Enemy** (`content/enemies/pruner.json`)

```json
{
  "id": "pruner",
  "name": "Pruner",
  "sprite": "enemies/pruner",
  "size": [24, 24],
  "hitbox": [16, 12],
  "hp": 6,
  "contactDamage": 2,
  "speed": 40,
  "knockbackScale": 0.5,
  "states": ["patrol", "telegraph", "charge", "stunned"],
  "attacks": [
    { "id": "charge", "tell": 0.5, "speed": 160, "damage": 2, "cooldown": 2.0, "onWall": "stun:1.2" }
  ],
  "armor": { "front": true, "piercedBy": ["gladius_thrust"] },
  "drops": { "table": "default" },
  "sfx": { "hurt": "enemy_hurt_metal", "die": "enemy_die_large" }
}
```

**Room fields** (LDtk level custom fields)

```json
{
  "roomId": "hy_07",
  "deck": "hy",
  "roomType": "lock",
  "requires": ["key:hy"],
  "grants": [],
  "systems": ["light"],
  "ambient": 0.2,
  "music": "hydroponics",
  "primitives": ["P05", "P10"],
  "notes": "Dark root tunnel; 2 Vine Drones; flare pickup if out of flares"
}
```

`requires` and `grants` use these tokens: `blade:<id>`, `tool:<id>`, `key:<deck>`, `bosskey:<deck>`, `flag:<id>`, `power:<roomId>`. The solvability tool reads only these fields plus door connections.

**Dialogue** (`content/npcs/mother_brass.json`, excerpt)

```json
{
  "id": "mother_brass",
  "name": "Mother Brass",
  "portrait": "portraits/mother_brass",
  "lines": [
    {
      "id": "mb_first",
      "when": { "act": 1, "notFlags": ["met_mother_brass"] },
      "expr": "neutral",
      "text": ["You came in through the wall.", "Nobody's done that in forty years."],
      "set": ["met_mother_brass"]
    }
  ]
}
```

Lines are chosen by the first match from the top. Each `text` entry is one box, at most 2 lines of 38 characters.

**Log** (`content/logs/hy.json`, one entry)

```json
{
  "id": "hy_02",
  "title": "Yield Report 4471",
  "author": "WARDEN, Agricultural Advisory",
  "concept": "Stoicism: control",
  "source": "terminal",
  "room": "hy_05",
  "pages": ["...", "..."]
}
```

`source` is `terminal` or `shard`. Pages are split at about 300 characters on sentence boundaries by the loader, never by hand.

**Flags** (`content/flags.json`): every flag with `id`, `description`, `setBy` and `readBy` (room, NPC or scene ids). The flag check fails CI if any flag is set and never read, read and never set, or missing from this file.

## Validators, bot, save and telemetry

**Validators (all run in CI on every PR)**

| Tool | Checks | Fails when |
| --- | --- | --- |
| `validate.ts` | Every content file against its zod schema; every id reference resolves | A schema error or a dangling reference |
| `solvability.ts` | Builds a state graph (room × inventory × flags) from `requires`/`grants` and door links; breadth-first search from new game | A deck boss, an ending or 100% completion is unreachable, or any reachable state can't reach the next goal (soft-lock) |
| `flags` check | Every flag in `flags.json` is both set and read | A flag is unused, unset, or undeclared |
| `text-check.ts` | Dialogue boxes (2×38 chars), item names (18), log titles (32), UI strings | Any string is over its limit |
| `palette-check.ts` | Every pixel of every PNG in `assets/sprites`, `tiles`, `portraits`, `ui` | A pixel is off-palette (alpha 0 is allowed) |
| `logs` check | Every log in `content/logs` matches `docs/04-lore-logs.md` verbatim and is placed in exactly one room | Text drift or a missing or duplicate placement |
| `manifest` check | Every asset file has a manifest entry | An unrecorded asset |

**Playtest bot (`tools/bot/`)**

- `bot:smoke`: boots the build, loads 5 sample rooms, checks for console errors. Under 2 minutes. Every PR.
- `bot:route`: follows the solvability route through the game using pathfinding plus scripted combat (attack the nearest enemy, dodge on tell, use the required blade). Reports deaths, stuck time and time per room. Nightly.
- `bot:full`: route mode for both endings plus a 100% completion run. Before each phase checkpoint.
- The bot plays with the same inputs a player has, through the input layer. It never calls game internals.

**Save format** (`localStorage` key `db_save_v1_slot<N>`, 3 slots)

```json
{
  "version": 1,
  "savedAt": "2026-10-12T14:00:00Z",
  "playtimeSec": 5420,
  "act": 2,
  "room": "rx_ante",
  "cells": 10, "maxCells": 14,
  "scrap": 87,
  "blades": ["seax", "gladius", "katana"],
  "activeBlade": "katana",
  "resonanceLevels": { "gladius": 1, "katana": 0 },
  "tools": { "flare": 4, "emp": 5 },
  "equipped": ["emp", "flare"],
  "keys": { "rx": 1 },
  "bossKeys": ["hy", "rx"],
  "flags": ["met_archivist", "met_mother_brass", "warden_broadcast_1"],
  "shards": ["hb_s1", "hy_s2"],
  "logsRead": ["db_01", "db_02", "hy_01"],
  "upgrades": ["carry_more"],
  "visited": ["db_01", "db_02"],
  "settings": { "navAssist": true }
}
```

`maxCells` counts half-cells. Saves migrate forward by `version`; never break old saves after launch.

**Telemetry** (Cloudflare Worker `POST /t`, D1 table `events`, opt-out in settings, anonymous session ids only)

| Event | Fields |
| --- | --- |
| `session_start` | build, platform, input type |
| `room_enter` / `room_exit` | room, time in room |
| `death` | room, cause (enemy id, hazard, boss attack), cells max |
| `boss_phase` | boss, phase, attempt number, time |
| `item_get` | item, room, playtime |
| `log_read` | log, read to the end (true/false) |
| `choice` | scene, option (NAV-ASSIST off, WARDEN offer) |
| `quit` | room, playtime |

The tuning report script (`npm run report`) turns this into per-room death rates, times and quit rates against the difficulty targets.
