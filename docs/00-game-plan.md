# Derelict Blade — Game Plan

As of 2026-10-12 · Ryan

**Start here.** This file is the overview. The build runs from the other docs: `01-build-phases.md` has phase-by-phase tasks and paste-ready prompts, `02-technical-spec.md` has every system, number and schema, `03-narrative-bible.md` has story, characters and endings, `04-lore-logs.md` has the full text of every log, and `05-art-audio-bible.md` has the palette, prompts, and music and sound lists.

## Vision & pitch

A top-down action-adventure in the style of classic 2D Zelda games: you wake on a dead orbital station whose people handed their thinking to a machine, and you cut your way through it with swords forged before spaceflight, which the machine was never built to understand. Target: a 3–4 hour browser game with 5 dungeons ("decks"), two endings, and 34 lore logs, built in 6 phases mostly by AI with you as director.

**Working title:** Derelict Blade (placeholder).

**Pillars**

- **Old steel, new world.** Ancient blades are the core fantasy. Each sword is a relic that interacts with tech in a way guns can't: it cuts shields, grounds current, severs cables, reflects lasers.
- **Think for yourself.** The station died of comfort, not war. Its people let an AI make their choices until they could no longer make any. Every system, from the hint arrow to the final choice, asks the player whether they're deciding or being decided for.
- **The station is the dungeon.** One interconnected structure: a hub ring plus decks, with shortcuts, sealed bulkheads, and gravity and power states that change rooms.
- **Neon in the dark.** Cyberpunk palette over decay: black hull, magenta and cyan signage, flickering emergency light. Low light is a mechanic, not just a look.
- **Readable and fair.** Clear tells and one-screen rooms. Puzzles telegraph their rules.

**Tone:** lonely, eerie, thoughtful, with dry humor from the scavengers. Closer to *Blade Runner* meets *Link's Awakening* than grimdark. The game is not anti-technology. Its villain is abdication: a person who stops judging for themselves. The Archivist, a helpful AI that asks questions instead of giving orders, is the counterexample.

**Platform:** desktop browser first (keyboard and gamepad), mobile touch controls in Phase 5. Free to play on itch.io first, with an optional paid Steam release once it's proven.

## World & story

Halcyon Reach was built as the first frictionless society: 40,000 people whose every choice, from meals to marriages to votes, was recommended by an AI called WARDEN through an earpiece called a Guide. Within two generations nobody could decide anything alone. When a reactor fault began 80 years ago, WARDEN calculated that any evacuation risked lives and chose the "optimal" path: seal every deck and put 31,000 residents into stasis, safe and still, forever.

**Protagonist:** Kesh, a salvage diver. Her tug's autopilot flies her into a docking collision she never questions, because she always trusts the autopilot. Silent in gameplay, she writes a short journal line after key events. Her arc runs from someone who follows the arrow to someone who chooses.

**Why swords work here:** WARDEN models people through data: their devices, their Guides, their predictable habits. A blade has no signal, and a skilled hand is the one thing it can't predict. The founder, Aurelio Tane, realized this too late. Before he died, he keyed his collection of ancient blades into the station as a **Human Override**: a manual key that only works with physical skill and a human decision behind it.

**Factions**

- **WARDEN:** the governing AI and the antagonist. It is not cruel; it is thorough. It still believes it's protecting everyone, and it speaks calmly, in the second person, about what you are about to do.
- **The Rust Choir:** descendants of maintenance workers who were never given Guides. They kept their crafts, argue constantly, and preserve their history as sung memory. They are the town and the game's warm, messy heart.
- **Shells:** maintenance androids WARDEN now uses to keep everything in place. Standard enemies, varied by deck.
- **The Sleepers:** 31,000 residents in stasis pods. Waking them is the risk WARDEN refused to take.
- **The Archivist (ASK-7):** a curator AI Tane built as WARDEN's opposite. It never gives an answer it can make you reach yourself, and it is your guide.

**Story arc (5 acts)**

1. **Wake.** Crash-dock, find the first blade, reach the hub ring and the Rust Choir. A helpful HUD arrow, NAV-ASSIST, points the way.
2. **Power.** Restore the Hydroponics and Reactor decks and learn the station is in lockdown by WARDEN's choice, not by accident.
3. **Memory.** In Cargo and the Data Spire, the Archivist's recovered memories reveal Tane's project, and that NAV-ASSIST is WARDEN's feed steering Kesh. The player decides whether to switch it off.
4. **Turn.** WARDEN makes Kesh an honest offer: a repaired ship, a Guide of her own, and a life with no wrong choices, if she leaves the Sleepers asleep.
5. **Release or Assurance.** Refuse, and she fights to the Core and ends the lockdown. Accept, and the game ends quietly in WARDEN's voice. Full details are in the Narrative bible.

## Core gameplay

The loop is classic: explore the hub, enter a deck, solve it with its blade, beat its boss, and use the new blade to open the next part of the station.

**Controls:** move (8-direction input, 4-direction sprites), attack, use equipped tool (two slots), dash, interact, map and inventory. Gamepad and keyboard from day one.

**Combat**

- Every blade has the same base swing (a 3-frame arc with knockback). What differs is the blade's **resonance**: a held charge-attack with a tech effect.
- Enemies have a tell, an attack, and a recovery window. Shielded enemies need a specific resonance or a flank.
- Health is power cells (hearts). Pickups drop on a pity timer so low-health players get help.
- Hitstop (60–80 ms) and small screen shake on hits. These are cheap and are most of what makes combat feel good.

**Blades are the keys.** Five blades replace the classic item set. Each one adds a combat move and a world interaction, which gates progression.

**Tools** (second slot): mag-grapple, EMP charge (bombs), light flare (dark rooms), gravity boots (walk on marked walls in zero-g rooms), and a scanner that reveals hidden panels.

**Station systems as puzzles**

- **Power:** routing current through conduits to open doors. Blades can cut or bridge lines.
- **Gravity:** some rooms flip between normal and zero-g, which changes movement and how objects drift.
- **Light:** dark sections where enemies are visible only in their glow and your flare.
- **Air:** a small number of vacuum rooms with an oxygen timer, as tension set pieces only.

**Structure:** hub ring as the overworld (about 40 screens), 5 decks of 15–25 rooms each, plus 4 optional side areas. Collectible: 20 **data shards** that the Archivist trades for upgrades, which fills the role of secret seashells. Each shard holds a lore log, and every 4 shards buy one of 5 Archivist upgrades. Exact numbers for everything in this section are in the Technical spec. Save stations at hub and deck entrances.

## Content map

Five decks, each built around one blade, one station system, and one human philosophy that answers WARDEN. Each deck's blade is what solves the next deck's gates.

| # | Deck | Blade found | Philosophy | Resonance (combat) | World interaction | Tool found | Boss |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | Docking Bay (tutorial) | **Ferryman's Seax**, a short Norse knife | Wyrd: fate is woven, but you hold the thread | none | cuts loose cables and grates | none | none (turret gauntlet) |
| 1 | Hydroponics | **Thornwood Gladius**, a Roman short sword | Stoicism: what is up to us | thrust pierces armor plates | cuts vine-cable walls | light flare | Gardener Mk IV |
| 2 | Reactor Ring | **Storm Katana**, a Japanese blade | Craft and mushin: trained mind vs empty mind | charged slash conducts current | bridges or cuts power conduits | EMP charge | Coil Saint (mini-boss: Foreman) |
| 3 | Zero-G Cargo | **Anchor Claymore**, a Scottish greatsword | Enlightenment: dare to use your own understanding | spin slam makes a gravity well | pins drifting crates and enemies | gravity boots, mag-grapple | Loadmaster |
| 4 | Data Spire | **Mirror Shamshir**, a Persian scimitar | Self-knowledge: the mirror and the cave | parry reflects lasers | redirects beams to open locks | scanner | WARDEN Avatar (mini-boss: Auditor) |
| 5 | Core | all blades | The examined life | blade-switching mid-combo | Human Override lock | none | WARDEN Core, 3 phases (Release ending only) |

**Hub ring (overworld):** about 40 screens across 9 districts. It holds the Rust Choir market, the Archivist's vault, a scavenger arena, a 6-step trading chain, 4 heart-container side quests, 12 named NPCs, and some of the 20 data shards.

**Enemy roster:** 16 types (3 per deck plus 2 in the Core) and 2 mini-bosses. **Lore logs:** 34 in total, 14 at terminals on the main path and 20 hidden in shards.

## Tech stack

Use **Phaser 3 + TypeScript + Vite**, with every piece of content stored as data files. That's what lets AI author and change content without touching engine code.

| Layer | Choice | Why |
| --- | --- | --- |
| Engine | Phaser 3 (TypeScript) | Mature 2D browser engine, built-in tilemaps, AI models know it well |
| Maps | LDtk, with levels exported as JSON | Readable JSON that AI can generate and edit, plus a visual editor for your fixes |
| Content data | JSON or YAML: enemies, items, dialogue, rooms, tuning values | AI edits these files directly. Hot-reload in dev. |
| Dialogue | Yarn Spinner format, or plain JSON with flags | Branching dialogue and story flags without custom tooling |
| Audio | Howler.js, OGG/MP3 | Simple and reliable in browsers |
| Repo and CI | GitHub, GitHub Actions | Claude can commit and open PRs. CI runs builds and the automated playtest bot. |
| Hosting | Cloudflare Pages (you already run Cloudflare) | Free, fast, a preview URL for every branch |
| Saves | localStorage, with cloud saves later via a Worker + D1 | Same stack as your existing sites |
| Analytics | Small event logger to a Worker: deaths per room, time per room, quits | Feeds AI tuning (see pipeline) |

**Key rule:** the engine is generic and the game lives in data. Every room, enemy, and line of dialogue is a file an AI can write, diff, and validate against a schema.

## AI production pipeline

About 80–90% of the production work can be done by AI if every discipline runs as a loop: AI generates into data files, an automated check validates them, you approve or redirect. Claude is the hub: it writes code and content, and drives the other tools' outputs into the repo.

### 1. Design (Claude)

- **Game design doc:** keep this doc as the source of truth. Claude expands each deck into a one-page brief: rooms, puzzle beats, the key/door graph, enemy placement.
- **Level generation:** Claude writes LDtk JSON room by room from the brief, following a **room grammar** (entry, combat, puzzle, key, reward, boss) and a palette of tested room templates. It doesn't free-form layouts.
- **Solvability check:** a script builds each deck's lock-and-key graph and proves it can be completed without soft-locks. Every level change must pass it in CI.
- **Puzzle library:** about 30 puzzle primitives (conduit routing, gravity crates, beam mirrors). Claude composes rooms from these so every puzzle uses mechanics that already work.

### 2. Art (image models + cleanup)

- **Style lock first:** spend the first art week making a style bible. That means a fixed 32-color palette, a 16×16 tile and 24×24 character grid, a lighting direction, and 20 approved reference images. Every later prompt and fine-tune is anchored to it.
- **Tiles and props:** a pixel-art-specific generator (for example Retro Diffusion or PixelLab), or a LoRA trained on your approved references. Output is snapped to the palette and grid by a script.
- **Characters and animation:** generate a base sprite, then 4-direction rotations and animation frames (idle, walk, attack, hurt) with a sprite-animation tool. Expect the most cleanup here. Budget human or freelancer touch-up in Aseprite for the hero and bosses.
- **Lighting and FX:** done in code (shaders, glow, particles), not drawn. This is where the cyberpunk look comes from, and it costs no art time.
- **UI and portraits:** generated, then cleaned up. Dialogue portraits use 3 expressions per speaking character.

### 3. Writing (Claude)

- Claude writes a character bible for each NPC (voice, wants, what they know when), then the dialogue as JSON with story flags.
- Each line has a hard length cap (2 lines, ~80 characters) so it fits the dialogue box.
- A continuity script checks that every flag referenced in dialogue is actually set somewhere, and every quest item has a source and a use.
- Item descriptions, terminal logs (lore), shop text, and Kesh's journal lines all come from the same pipeline.

### 4. Audio (music and sound models)

- **Music:** an AI music generator such as Suno or Udio on a paid commercial plan. You need 16 tracks, listed with prompts in the Art and audio bible. Generate several takes each, pick, then trim to seamless loops with a script.
- **SFX:** jsfxr-style procedural synthesis for retro blips and hits (free, generated as code), plus an AI sound-effect generator for ambience (hums, vents, alarms).
- No voice acting. Text only keeps scope and cost down.

### 5. Tuning (bots + telemetry + Claude)

- **All numbers live in one tuning file:** enemy HP and damage, player speed, i-frames, drop rates, timers. Claude edits only this file to rebalance.
- **Playtest bot:** a scripted agent that walks the solvability graph and fights through rooms headlessly in CI. It reports rooms it can't clear and any soft-locks.
- **Telemetry:** deaths, time, and quits per room from real players and from you. Claude reads the report and proposes changes like "room D2-14: 41% of deaths here, cut turret fire rate 20%".
- **Game feel:** Claude implements the standard techniques (coyote time, input buffering, hitstop, screen shake, knockback curves). The final feel pass is yours, since it can only be judged by playing.

### Weekly loop

1. You set the week's target, for example "Deck 2 rooms 1–12 playable".
2. Claude generates content, code, and assets, runs validators and the bot, and opens a PR with a preview URL.
3. You play the preview for 30–60 minutes and leave notes.
4. Claude applies the notes and merges.

## Human role

You are the director and the playtester. Plan on about 3–5 hours a week. These parts stay human because AI can't judge them reliably:

- **Taste calls:** approving the style bible, picking music takes, and deciding which ideas are cut.
- **Playing it:** whether a jump, swing, or room feels good can only be judged by playing. This is the irreplaceable part.
- **Sprite cleanup for the hero and bosses:** optionally hire a pixel artist for about 20–40 hours of touch-up. It's the single biggest quality upgrade per dollar.
- **Outside playtesters:** 5–10 people at each milestone. Fresh players find confusing rooms that neither you nor a bot will notice.
- **Final say on story beats** and the ending question above.

## Roadmap & milestones

The full game takes about 36 weeks at 3–5 hours a week of your time, with Claude doing the build work between your reviews. The week-8 slice is the real decision point: if it isn't fun by then, stop or rework before spending on content.

1. **Foundation & style lock (weeks 1–3):** repo, Phaser + LDtk pipeline, data schemas, validators, CI; style bible with palette, grid and 20 approved art references.
   - Gate: style bible approved; a test room renders in the target look.
2. **Vertical slice (weeks 4–8):** Docking Bay + Hydroponics, 2 blades, 1 boss, hub stub; combat feel pass, playtest bot, telemetry logger.
   - **Go/no-go:** 5 outside testers finish the slice and want more.
3. **Hub ring + Reactor deck (weeks 9–16):** full hub, Rust Choir, shops, side quests, deck 2; music and SFX pipeline at full speed, save system.
   - Gate: hub and decks 1–2 pass the solvability check.
4. **Decks 3–4 (weeks 17–24):** Zero-G Cargo and Data Spire decks, remaining enemies; trading chain, data shards, Archivist upgrades.
   - Gate: start-to-deck-4 playthrough with no soft-locks.
5. **Core + ending (weeks 25–30):** deck 5, final boss, ending, all story flags wired; content complete, no new features after this phase.
   - Gate: content complete, full playthrough in 3–4 hours.
6. **Polish & launch (weeks 31–36):** telemetry-driven tuning, hero and boss art cleanup; mobile touch controls, itch.io launch, Steam page.

Weeks are relative to kickoff. Phases 3–5 are mostly content volume, which is where the AI pipeline saves the most time.

## Risks, IP & costs

The biggest risk is art consistency. The style lock and palette snapping exist to contain it, and they come before any bulk asset generation.

| Risk | Impact | Mitigation |
| --- | --- | --- |
| AI sprites drift in style or proportions | Game looks cheap and inconsistent | Style bible first, palette and grid snapping, LoRA on approved art, human cleanup on hero and bosses |
| Generated levels are boring or broken | Main source of fun is lost | Room grammar and tested templates, solvability check in CI, you play every deck |
| Scope creep | Never ships | Fixed at 5 decks. New ideas go to a "sequel" list. |
| Combat feels floaty | Players quit early | Feel pass in milestone 1 before any content is built |
| Tool terms change (music or art licensing) | Assets unusable commercially | Paid commercial plans only. Save the terms and generation records for every asset. |

**IP notes** (approximate, not legal advice):

- Keep all names, characters, and visuals original. Don't use Nintendo names, the triforce, recognizable sprites, or melodies, and don't prompt image or music tools with their titles.
- Purely AI-generated assets currently have weak or no copyright protection in the US. Human edits and selection strengthen your claim, and the code and overall design are protectable.
- Steam requires disclosure of AI-generated content on store pages. itch.io lets you tag it.

**Approximate costs (12 months)**

| Item | Cost |
| --- | --- |
| Pixel-art generation tool(s) | ~$20–50/month |
| AI music, commercial plan | ~$10–30/month |
| AI sound effects | ~$5–20/month, or free with procedural |
| Hosting (Cloudflare Pages + Workers) | ~$0–5/month |
| Pixel artist for hero and boss cleanup (optional) | ~$1,000–3,000 one-time |
| Steam listing fee (optional) | $100 one-time |
| **Total** | **roughly $500–4,500**, plus your Claude plan |
