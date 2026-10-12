# Build phases

Every phase below is written so an AI coding agent can execute it end to end: tasks in order, the files each produces, a paste-ready Go prompt, and the checks that must pass before you review.

## How to run the build

The repo is the agent's source of truth, and the design doc is yours. Each phase runs as a loop: export the doc into the repo, paste the phase's Go prompt into a coding agent, let it work task by task, then review the preview build at the checkpoint.

**Setup (once)**

1. Create an empty GitHub repo named `derelict-blade`.
2. Export every tab of the design doc as Markdown into `docs/` in the repo: `docs/00-game-plan.md`, `docs/01-build-phases.md`, `docs/02-technical-spec.md`, `docs/03-narrative-bible.md`, `docs/04-lore-logs.md`, `docs/05-art-audio-bible.md`.
3. Connect the repo to a Cloudflare Pages project so every branch gets a preview URL.
4. Open a Claude Code session (terminal, web or desktop) on the repo.

**Running a phase**

1. Paste the phase's Go prompt, unchanged.
2. The agent works one task per pull request, in the listed order. Each PR must pass CI: schema validation, solvability, text and palette checks, unit tests, build, and the playtest bot.
3. Tasks marked **[human]** pause the agent. It posts what it needs (a choice, an approval, a playtest) and waits.
4. At the phase checkpoint, you play the preview build and leave notes as issues titled `Phase N review: ...`. The agent fixes those before the next phase starts.

**Changing the design mid-build**

Edit the design doc first, re-export the changed tab to `docs/`, then tell the agent "Design changed: re-read docs/<file> and update what it affects." Never let code become the only record of a design decision.

**Effort model**

| Phase | Weeks | Your time | Agent work |
| --- | --- | --- | --- |
| 0 Foundation | 1–3 | ~6 h | Repo, engine core, pipeline, validators, style bible candidates |
| 1 Vertical slice | 4–8 | ~15 h | 22 rooms, 2 blades, 5 enemies, 1 boss, systems, first art and music |
| 2 Hub + Reactor | 9–16 | ~20 h | 40 hub screens, 12 NPCs, 20 rooms, power system, quests |
| 3 Cargo + Spire | 17–24 | ~20 h | 44 rooms, gravity and beam systems, Act 3 reveal |
| 4 Core + endings | 25–30 | ~15 h | 18 rooms, final boss, both endings, full-game validation |
| 5 Polish + launch | 31–36 | ~20 h | Tuning, accessibility, mobile controls, store pages |

## Agent rules (CLAUDE.md)

Phase 0 writes this file to the repo root verbatim. Every agent session reads it first.

```markdown
# Derelict Blade: agent rules

## Source of truth
- docs/ holds the design. Read docs/02-technical-spec.md before writing code and docs/03-narrative-bible.md before writing any text.
- If a task needs a mechanic, number or story fact the docs don't define, do not invent it silently. Write a proposal in docs/proposals/NNN-title.md, pick the most conservative option, and flag it in the PR description.

## Architecture
- The engine in src/ is generic. The game lives in content/ as JSON validated by zod schemas in src/data/schemas/.
- Every gameplay number lives in content/tuning.json. No magic numbers in src/.
- Every string the player reads lives in content/ (dialogue, logs, items, UI strings in content/strings.json).
- Rooms are authored in content/world.ldtk. Room metadata uses the LDtk custom fields defined in the spec.

## Workflow
- One task per branch and PR, named phase-N/task-N.M-short-name.
- Before opening a PR, run: npm run validate && npm test && npm run build && npm run bot:smoke. All must pass.
- PR description: what changed, files touched, how to test it in the preview build, any proposals raised.
- Tasks marked [human] require stopping and asking. Post the question as a PR comment or issue and wait.
- Append one line per merged task to CHANGELOG.md.

## Assets
- All art uses the 32-color palette in assets/palette.hex. Run tools/palette-snap.py on every generated image. CI rejects off-palette pixels.
- Record every AI-generated asset in assets/manifest.json: file, tool, prompt, seed, date, license or plan.
- Never use names, characters, sprites, music or text from existing games, films or books. Never put another work's title or an artist's name in a generation prompt.

## Writing
- Follow the writing rules in docs/03-narrative-bible.md. Dialogue boxes: max 2 lines of 38 characters each.
- Lore logs come verbatim from docs/04-lore-logs.md. Do not rewrite them.

## Quality bar
- 60 fps on a mid-range 2019 laptop in Chrome, Firefox and Safari.
- No console errors in a full bot run.
- Keyboard and gamepad must both work for every feature.
```

## Phase 0: Foundation (weeks 1–3)

**Goal:** a repo where content is data, validators run in CI, the player can move and swing the Seax in one room that already looks like the final game.

**Tasks**

1. **0.1 Scaffold.** Vite + TypeScript (strict) + Phaser 3, ESLint, Prettier, Vitest, zod. Write `CLAUDE.md` from the Agent rules section. Add the folder layout from the Technical spec. Add `.github/workflows/ci.yml` running lint, test, validate and build on every PR. Output: a booting black screen on the Pages preview.
2. **0.2 Engine core.** Scenes `Boot`, `Preload`, `Game`, `UI`, `Pause`. Internal resolution 320×180 with integer scaling and letterboxing. Input layer with keyboard and gamepad mapped to actions (bindings in `content/input.json`). Room camera with screen-by-screen slide transitions.
3. **0.3 Content pipeline.** Zod schemas for every type in the Data schemas section. Loaders with clear error messages (file, path, problem). Hot reload of `content/` in dev. LDtk import: tile layers, collision layer, and the entity types listed in the spec.
4. **0.4 Validators.** `tools/validate.ts` (schemas + cross-references), `tools/solvability.ts` (lock-and-key graph search), `tools/text-check.ts` (length limits), `tools/palette-check.ts`. All wired into `npm run validate` and CI.
5. **0.5 Player controller.** Movement, dash, Seax swing with hitbox, hurt, death, and respawn at the last room entrance. All values from `tuning.json`.
6. **0.6 Debug tools.** F1 overlay (hitboxes, FPS, room id, flags), F2 room warp list, F3 invincibility, F4 grant all blades and tools, F5 reload tuning. Stripped from production builds.
7. **0.7 Style bible candidates.** Using the Art and audio bible prompts, generate 3 style boards for the Docking Bay (tiles, player, one crawler, a neon sign), each snapped to the palette. **[human]** You pick one, or ask for changes.
8. **0.8 Test room.** One Docking Bay room in the chosen style: tileset, lighting (dark ambient, cyan and magenta signs, amber emergency light), the player and 2 crawlers. Write `tools/palette-snap.py` and `assets/manifest.json`.
9. **0.9 Bot skeleton.** Headless run of the built game in Playwright: boots, loads the test room, walks a scripted path, exits with a report. Wired to `npm run bot:smoke`.

**Go prompt**

```text
You are the build agent for Derelict Blade. Read docs/01-build-phases.md, docs/02-technical-spec.md and docs/05-art-audio-bible.md in full. Execute Phase 0, tasks 0.1 through 0.9, in order. One task per branch and PR, following CLAUDE.md once task 0.1 has created it. Use the exact values, folder names and schemas in the technical spec. For task 0.7, stop after posting the three style boards as images in a PR comment and wait for my choice. When all tasks are merged, post a Phase 0 report: what was built, the preview URL, open proposals, and anything you could not finish.
```

**Acceptance**

- [ ] Preview URL loads the test room in Chrome, Firefox and Safari at 60 fps
- [ ] Changing a value in `tuning.json` changes the game on reload, with no code change
- [ ] A deliberately broken content file fails CI with a readable error
- [ ] Keyboard and a gamepad both move, dash and attack
- [ ] Every pixel in the test room is in the palette

**Checkpoint [human]:** approve the style bible and play the test room for 10 minutes. Does moving and swinging already feel crisp? If not, tune it now, before any content exists.

## Phase 1: Vertical slice (weeks 4–8)

**Goal:** 25–35 minutes of finished game, from the crash to beating the Gardener Mk IV, good enough to show strangers. This phase decides whether the project continues.

**Tasks**

1. **1.1 Combat system.** Hitboxes and hurtboxes, damage in half-cells, knockback, i-frames, hitstop, screen shake, the charge-attack framework with a per-blade `resonance` hook, and enemy damage reactions. Per the Combat spec.
2. **1.2 HUD.** Power cells, scrap counter, active blade icon, two tool slots, a minimap of the current deck, and the NAV-ASSIST arrow (a small cyan chevron at the screen edge pointing to the next objective).
3. **1.3 Enemy framework + 5 enemies.** Behaviors as composable state machines (`idle`, `patrol`, `chase`, `telegraph`, `attack`, `recover`, `stunned`). Build Crawler, Turret, Vine Drone, Pruner and Spore Pod from the roster.
4. **1.4 Docking Bay.** 6 rooms from its deck brief, including the turret gauntlet and the Seax room.
5. **1.5 Hydroponics deck.** 16 rooms from its deck brief. Must pass solvability. Includes the Gladius room, the flare, 2 small keys, the boss key, the map terminal, 1 secret room and 1 shortcut.
6. **1.6 Blades and tools.** Seax, Gladius (thrust resonance, vine-wall cutting) and the light flare, with dark-room lighting.
7. **1.7 Gardener Mk IV.** Full boss per the Bosses section: 3 phases, telegraphs, arena, intro and defeat sequence, blade-room reward.
8. **1.8 Dialogue and terminals.** Dialogue box with typewriter text, portraits, choices and flag effects. Terminal reader for logs, with paging and a log archive in the pause menu. Archivist first meeting. Logs DB-01 to DB-03 and HY-01 to HY-05, placed per the Lore logs doc.
9. **1.9 Hub stub.** 6 hub screens: Dock Spoke, part of the Choir Market (Mother Brass and Vend), the save station and the Garden Spoke door.
10. **1.10 Save system.** 3 slots, save stations, continue on death at the last save or deck entrance, and the save format in the spec.
11. **1.11 Production art.** Player (all animations, Seax and Gladius overlays), the 5 enemies, Gardener Mk IV, Docking and Hydroponics tilesets, portraits of Kesh, the Archivist, Mother Brass and Vend. **[human]** approve the player sprite before the rest is made.
12. **1.12 Audio v1.** SFX set from the Sound effects list. Music: Title, Docking Bay, Hub, Hydroponics, Boss. **[human]** pick 1 of 3 takes per track.
13. **1.13 Telemetry + bot v1.** Event logger to a Cloudflare Worker with D1 per the spec. Bot v1 completes the slice using the solvability route and reports deaths and stuck rooms.
14. **1.14 Feel pass.** Implement the full juice list in the Combat spec, then record a 60-second capture of the slice for review.

**Go prompt**

```text
You are the build agent for Derelict Blade. Phase 0 is merged. Re-read CLAUDE.md and all of docs/. Execute Phase 1, tasks 1.1 through 1.14, in order. Build levels from the room grammar and the Docking Bay and Hydroponics deck briefs in docs/02-technical-spec.md, composing rooms only from the listed puzzle primitives. Use lore log text verbatim from docs/04-lore-logs.md. Stop at tasks marked [human] and wait. When everything is merged, post the Phase 1 report with the preview URL, the bot's completion time and death heatmap, and a list of every proposal raised.
```

**Acceptance**

- [ ] The bot completes the slice from a new save with no soft-locks
- [ ] Solvability passes for Docking Bay and Hydroponics
- [ ] All 8 slice logs readable and archived
- [ ] A first-time player can finish in 25–35 minutes (bot time × about 2.5 as a first estimate)
- [ ] No room has more than 25% of all bot deaths

**Go/no-go [human]:** 5 outside testers play the slice without help while you watch or record. Continue to Phase 2 if at least 4 finish, at least 3 say they'd play more, and nobody gets stuck for more than 5 minutes. Otherwise spend one or two weeks on what they hit, then retest.

## Phase 2: Hub ring + Reactor deck (weeks 9–16)

**Goal:** the station opens up. The full hub ring is alive with the Rust Choir, and the Reactor deck adds the power system and Act 2's turn.

**Tasks**

1. **2.1 Full hub ring.** About 40 screens across the 9 districts in the Hub ring deck brief, with gated spokes to every deck, 2 save stations, and shortcuts that open as decks are cleared.
2. **2.2 NPCs and dialogue.** All 12 Rust Choir NPCs from the Narrative bible, each with dialogue for every act (driven by `act` and story flags), portraits with 3 expressions, and idle animations.
3. **2.3 Shops and economy.** Vend's shop (cells refills, flares, EMP charges, the 2 buyable heart pieces), Mother Brass's blade forge (resonance upgrades), the haggling mini-interaction, and the scrap economy values in the spec.
4. **2.4 Side quests.** The 4 heart-container quests and steps 1–3 of the 6-step trading chain, as defined in the Narrative bible's quest list.
5. **2.5 Map and shards.** Full-station map screen (visited rooms only, plus map terminals per deck), the 20 data-shard placements available so far, the Archivist upgrade menu (4 shards per upgrade).
6. **2.6 Power system.** Conduits, junctions, powered doors, lifts and hazards per the Station systems section.
7. **2.7 Reactor Ring deck.** 20 rooms from its deck brief, including the Foreman mini-boss and the Katana room.
8. **2.8 Storm Katana and EMP charge.** Charged-slash conduction, conduit bridging and cutting, EMP walls and EMP stuns.
9. **2.9 Reactor enemies.** Spark Wisp, Coil Guard, Arc Mine.
10. **2.10 Coil Saint.** Full boss per the Bosses section.
11. **2.11 Act 2 story.** The beats and flags for Act 2, the first WARDEN broadcasts, and the HB and RX logs.
12. **2.12 Art and audio.** Hub and Reactor tilesets, NPC sprites and portraits, Reactor enemies and bosses. Music: Choir Market, Vault, Shop, Reactor, Mini-boss. **[human]** approve NPC portraits and pick music takes.

**Go prompt**

```text
You are the build agent for Derelict Blade. Phases 0 and 1 are merged and the Phase 1 review issues are closed. Re-read CLAUDE.md and all of docs/. Execute Phase 2, tasks 2.1 through 2.12, in order. Write NPC dialogue from the character voice guides and act beats in docs/03-narrative-bible.md, and run the dialogue flag check on every PR. Build the Reactor Ring from its deck brief. Stop at [human] tasks. Finish with the Phase 2 report: preview URL, bot route time from new game to Coil Saint, death heatmap, and every proposal raised.
```

**Acceptance**

- [ ] Bot completes new game → Coil Saint with no soft-locks; solvability passes for the hub and 3 decks
- [ ] Every NPC has a line for every act they're present in; the flag check finds no unset or unused flags
- [ ] Power puzzles cannot be left in an unwinnable state (reset switch in every power room)
- [ ] Scrap income and prices let a player who explores buy both heart pieces by the end of Reactor

**Checkpoint [human]:** play from a new save to the Coil Saint (about 75 minutes). Read every NPC's lines in one sitting. Check that the Choir feels like people, and that the WARDEN reveal lands.

## Phase 3: Zero-G Cargo + Data Spire (weeks 17–24)

**Goal:** the two most mechanically complex decks, and Act 3's reveal, where the player learns the helpful arrow was WARDEN all along.

**Tasks**

1. **3.1 Gravity system.** Zone-based zero-g with inertia and drag, gravity-flip switches, gravity-boot wall walking on marked plating, drifting objects, and the Claymore's gravity well, per the Station systems section.
2. **3.2 Mag-grapple.** Grapple posts, pulling small objects to you, and pulling yourself to large ones. 6-tile range.
3. **3.3 Zero-G Cargo deck.** 22 rooms from its deck brief, including the Claymore room.
4. **3.4 Cargo enemies and Loadmaster.** Drift Shell, Loader Arm, Magnet Bot, then the Loadmaster boss.
5. **3.5 Beam system.** Lasers, mirrors, splitters, beam receivers, and the Shamshir's parry reflection, per the Station systems section.
6. **3.6 Scanner.** Reveals hidden panels, shard locations within the room, and enemy weak points. 3-second pulse, 2-second cooldown.
7. **3.7 Data Spire deck.** 22 rooms from its deck brief, including the Auditor mini-boss and the Shamshir room.
8. **3.8 Spire enemies and WARDEN Avatar.** Lens Sentry, Mimic Hologram, Censor, then the WARDEN Avatar boss.
9. **3.9 Act 3 reveal.** The Archivist memory sequence and the NAV-ASSIST reveal scene. The player chooses whether to disable NAV-ASSIST, sets the flag `nav_assist_off`, and the map switches to the Archivist's map (only rooms you've walked).
10. **3.10 Remaining quests.** Trading chain steps 4–6 and the remaining heart quests.
11. **3.11 Logs.** CG and DS logs, placed per the Lore logs doc.
12. **3.12 Art and audio.** Cargo and Spire tilesets, enemies, bosses, WARDEN Avatar hologram FX. Music: Cargo, Spire. **[human]** pick takes.

**Go prompt**

```text
You are the build agent for Derelict Blade. Phases 0–2 are merged and review issues are closed. Re-read CLAUDE.md and all of docs/. Execute Phase 3, tasks 3.1 through 3.12, in order. The gravity and beam systems need unit tests for every rule in the Station systems section before any room uses them. Build both decks from their deck briefs. Implement the Act 3 reveal exactly as written in the Narrative bible. Stop at [human] tasks. Finish with the Phase 3 report: preview URL, bot route time from new game to WARDEN Avatar, death heatmap, proposals.
```

**Acceptance**

- [ ] Gravity and beam systems have passing unit tests for every rule in the spec
- [ ] No zero-g room can strand the player (every zero-g room has a gravity anchor or exit reachable by drifting)
- [ ] Bot completes new game → WARDEN Avatar with no soft-locks
- [ ] The reveal scene plays correctly with NAV-ASSIST on, and the game stays completable after either choice

**Checkpoint [human]:** play Cargo and the Spire. Zero-g is the riskiest mechanic for feeling bad. If it frustrates you, cut its room count before polishing.

## Phase 4: Core + endings (weeks 25–30)

**Goal:** the game is content complete. Both endings play, every flag resolves, and the bot finishes the whole game.

**Tasks**

1. **4.1 Blade switching.** Mid-combo blade switching (LB/RB or 1–5), with each switch keeping the combo counter. Core rooms require 2–3 blades in sequence.
2. **4.2 WARDEN's offer.** The Act 4 scene at the Core Gate per the Narrative bible: WARDEN's full offer, the Accept/Refuse choice, and its flags.
3. **4.3 Assurance ending.** Accept path: the ship repair sequence, the Guide fitting, and the epilogue where Kesh's journal lines are replaced by WARDEN's phrasing. Credits.
4. **4.4 Core deck.** 18 rooms from its deck brief (Refuse path only), with Warden Knights and Guide Swarms.
5. **4.5 WARDEN Core.** The 3-phase final boss and the Human Override lock sequence.
6. **4.6 Release ending.** The waking of the Sleepers, the Choir scene, the final Archivist exchange, Kesh's last journal line, and credits.
7. **4.7 Core logs.** CR logs, including the founder's confession.
8. **4.8 Full-game validation.** Solvability for the whole station, full flag audit, both endings reachable, 100% completion reachable (all shards, cells and quests), and a full bot run of each ending.
9. **4.9 Art and audio.** Core tileset, WARDEN Core, ending scenes. Music: Core, Final Boss, Release, Assurance, Title (final). **[human]** pick takes.

**Go prompt**

```text
You are the build agent for Derelict Blade. Phases 0–3 are merged and review issues are closed. Re-read CLAUDE.md and all of docs/. Execute Phase 4, tasks 4.1 through 4.9, in order. Implement both endings exactly as written in docs/03-narrative-bible.md, word for word. After task 4.8, freeze features: from here on, only fixes and tuning. Finish with the Phase 4 report: preview URL, full-run bot times for both endings, a 100% completion route, death heatmap, and every flag with where it is set and read.
```

**Acceptance**

- [ ] Bot finishes both endings from a new save
- [ ] 100% completion is reachable, and the pause menu shows completion percentage
- [ ] Full flag audit is clean
- [ ] First-time playthrough estimate is 3–4 hours

**Checkpoint [human]:** play the whole game once, start to finish, Release path. Then load a save before the offer and take Assurance. The endings carry the theme, so judge them on whether they make you feel it, not explain it.

## Phase 5: Polish + launch (weeks 31–36)

**Goal:** tuned, accessible, playable on phones, and published.

**Tasks**

1. **5.1 Playtest round A.** **[human]** recruit 10 testers. The agent builds a tester build with telemetry on and a feedback form link on the pause screen.
2. **5.2 Tuning pass A.** The agent reads telemetry and form results, writes a tuning report (top 10 problem rooms with deaths, time and quit rates), and changes only `tuning.json` and room layouts. Targets are in the Tuning section of the spec.
3. **5.3 Playtest round B + tuning pass B.** Repeat with 10 new testers.
4. **5.4 Art cleanup.** **[human]** hire a pixel artist for 20–40 hours on the player, 6 bosses and portraits. The agent packages a brief and the source files.
5. **5.5 Accessibility.** Remappable controls, damage scale (0.5×, 1×, 1.5×), screen shake toggle, flash reduction, text speed, high-contrast hazards, and NAV-ASSIST default setting.
6. **5.6 Touch controls.** Virtual stick and 4 buttons, auto-shown on touch devices, with layout editing.
7. **5.7 Performance.** Per-deck asset loading, total download under 40 MB, 60 fps on a 2019 mid-range laptop and a 2021 mid-range phone.
8. **5.8 Store assets.** Trailer captured from a scripted bot route (90 seconds), 6 screenshots, itch.io page, Steam page draft with the AI content disclosure.
9. **5.9 Launch.** **[human]** publish on itch.io. Steam after 2–4 weeks of itch feedback if you choose.

**Go prompt**

```text
You are the build agent for Derelict Blade. The game is content complete. Re-read CLAUDE.md and all of docs/. Execute Phase 5, tasks 5.1 through 5.9, in order. During tuning, change only content/tuning.json and room layouts, and justify every change with telemetry in the PR. Stop at [human] tasks. Do not add features. Finish with a launch checklist and the final build URL.
```

**Acceptance**

- [ ] No room has a first-attempt death rate above 50% outside boss arenas
- [ ] Median first playthrough is 3–4 hours in round B
- [ ] Every accessibility option works on keyboard, gamepad and touch
- [ ] Lighthouse performance score 90+ on the landing page
