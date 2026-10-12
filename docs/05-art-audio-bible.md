# Art and audio bible

The locked look and sound of the game. Every generated asset is checked against this doc before it enters the repo.

## Palette, grid and sprite sizes

A starting 32-color palette. Phase 0's style boards may adjust individual values; once approved, it is locked in `assets/palette.hex` and enforced by CI.

| Ramp | Colors (dark → light) | Used for |
| --- | --- | --- |
| Void (8) | `#07070C` `#0F1018` `#171A26` `#222637` `#2E3349` `#3D435C` `#535A76` `#6E7591` | Hull, floors, walls, shadows, outlines |
| Light (2) | `#C6CAD8` `#EEF0F6` | Steel highlights, text, blade edges |
| Cyan neon (5) | `#0B3D4A` `#116B7A` `#18A9B8` `#4FE3E8` `#B4FBFD` | WARDEN tech, screens, NAV-ASSIST, Katana |
| Magenta neon (5) | `#3D0A35` `#6E1259` `#B0207F` `#FF3FA4` `#FFA6D6` | Signage, the Spire, Shamshir, hazards |
| Amber (4) | `#4A2A0B` `#8A4E12` `#D9881C` `#FFC34D` | Emergency light, the Choir's lamps, Claymore, scrap |
| Rust (3) | `#3A1D14` `#6B3424` `#A3553A` | The Choir's world: patched metal, cloth, skin tones |
| Green (4) | `#123A22` `#23663A` `#48A85A` `#9BE27A` | Restored plants, Gladius, health |
| Alert (1) | `#E8323C` | Damage, danger, low health only |

**Color rules:** WARDEN's spaces are cool (void, cyan, clean light). The Choir's spaces are warm (amber, rust, mixed colors). Green appears only where people have taken something back. Red means danger and nothing else.

**Sizes**

| Asset | Size | Notes |
| --- | --- | --- |
| Tiles | 16×16 | Top-down 3/4 view; walls show a 1-tile front face |
| Player | 16×24 frame | Hitbox 10×8 at the feet |
| Blade overlays | 32×32 frame | Drawn separately and layered over the player during swings |
| Small enemies | 16×16 or 16×24 | |
| Large enemies | 24×24 to 32×32 | |
| Bosses | 64×64 to 96×96 | Separate parts for moving limbs where needed |
| Portraits | 48×48 | 3 expressions per speaking character: neutral, happy or warm, troubled |
| Items and HUD icons | 16×16 | |
| Font | 8 px pixel font, plus a 6 px small font | Open-licensed (OFL or CC0); verify the license in Phase 0 |

**Outline style:** 1-px darkest-void outline on characters and enemies; no outline on tiles. No anti-aliasing, no gradients except what the code-based lighting adds.

## Image generation prompts

Every art prompt is the base prompt plus a deck modifier plus the asset description. Output is always snapped to the palette and the grid by `tools/palette-snap.py`, then reviewed. Never name another game, studio, artist or film in a prompt.

**Base prompt**

```text
16-bit top-down pixel art, three-quarter overhead view, 16x16 pixel tile grid, limited palette, crisp 1-pixel outlines, no anti-aliasing, dark derelict space station interior, cyberpunk neon accents in cyan and magenta, worn industrial surfaces, clean readable silhouettes
```

**Negative prompt**

```text
blurry, smooth gradients, 3d render, photorealistic, isometric, perspective distortion, text, letters, logo, watermark, signature, noisy dithering
```

**Deck modifiers**

| Deck | Modifier |
| --- | --- |
| Docking Bay | cramped airlocks, hazard stripes, broken display cases, amber emergency lights, sparks |
| Hub ring | warm patched market stalls, hanging cloth, amber lamps, hand-painted signs, cables strung overhead, glass gallery of stasis pods |
| Hydroponics | rows of identical grey plants under cold grow-lights, irrigation pipes, overgrown vine-cables, a single ancient tree |
| Reactor Ring | thick conduits with cyan current, control room with rows of chairs facing one console, warning lights, coolant pipes |
| Zero-G Cargo | floating crates, magnetic deck plating, conveyor belts, cargo cranes, open shafts with stars visible |
| Data Spire | tall cold server towers, magenta screens showing silhouettes, mirrored floors, glass walls, light beams |
| Core | white sterile corridors, soft light, rows of stasis pods, a vast calm screen, minimal detail |

**Asset prompt examples**

- Player: `a young female salvage diver in a worn orange flight suit with patched pads and a short cropped jacket, short dark hair, small backpack, holding a short knife, front-facing idle pose, 16x24 sprite, transparent background`
- Pruner enemy: `a squat gardening robot with large shears for arms and an armored front plate, grey with green accents, 24x24 sprite, transparent background`
- WARDEN screen: `a large wall screen showing a simple calm white circle on dark cyan, soft glow, 48x32 tile set piece`

**Workflow:** generate 4–8 candidates → snap to palette → agent picks the 2 best by the checklist (silhouette readable at 1×, palette clean, matches the style board, facing correct) → **[human]** approves key assets (player, NPC portraits, bosses) → agent generates rotations and animation frames from the approved base → cleanup pass → record in the manifest.

## Sprite and animation lists

Frame counts are per direction. "4 dir" means down, up, left, right; right is drawn and left is mirrored unless the design is asymmetric. Frame rates are in frames per second.

**Player (Kesh)**

| Animation | Frames | FPS | Notes |
| --- | --- | --- | --- |
| Idle | 4 × 4 dir | 6 | Small breathing bob |
| Walk | 6 × 4 dir | 12 | |
| Dash | 3 × 4 dir | 18 | Dust FX in code |
| Swing (body) | 4 × 4 dir | 18 | Shared by all blades |
| Charge hold | 2 × 4 dir | 8 | Glow is code |
| Hurt | 2 × 4 dir | 12 | |
| Death | 6 (down only) | 10 | |
| Item get | 1 | — | Arms raised, item held overhead |
| Interact | 2 × 4 dir | 8 | |
| Zero-g drift | 2 × 4 dir | 6 | Phase 3 |

**Blade overlays:** 5 blades × 4 dir × 4 swing frames, plus 1 thrust (Gladius), 6 spin (Claymore) and 2 parry (Shamshir) sets. Keeping blades as overlays means the body is drawn once, which saves about 70% of player art.

**Enemies:** idle 2–4, move 4, telegraph 2, attack 3–4, hurt 1. Death uses a shared code-driven burst (scrap and sparks), so no death frames are needed except for bosses.

**Bosses:** idle 4, 2–3 attacks of 4–6 frames each, telegraph 2, hurt 1, phase change 6, defeat 8. Large bosses are built from separate parts (body, arms, head) moved in code to cut frame counts.

**NPCs:** idle 4 (down only) plus 1 talk frame. Juno-9 gets a 4-frame gesture set. Sleepers in pods use 2-frame slow breathing.

**Tilesets (per deck):** floor (4 variants), wall top and face (with corners and ends, about 47 tiles in a standard autotile set), doors (open, closed, locked, boss), hazards, 12–20 props, and 2–3 set-piece tiles.

**Estimated totals:** about 2,600 frames of sprites and 7 tilesets. Roughly 85% can be generated and cleaned by the pipeline; the hand-cleanup budget in Phase 5 covers the player and 6 bosses.

## Lighting and FX in code

Most of the cyberpunk look is done in code, which costs no art time and stays consistent. Lighting is a multiply layer over the scene, with additive glow sprites for every light source.

| Effect | How it's done | Values |
| --- | --- | --- |
| Room ambient | Dark multiply overlay | 0 (black) to 1 (full); default 0.55; dark rooms below 0.3 |
| Player lantern | Soft radial glow, warm white | 48 px radius |
| Flare | Amber radial glow, flickers ±4 px | 112 px for 8 s |
| Neon signs | Cyan or magenta glow plus 1 random flicker per 4–10 s | 32–64 px |
| Emergency lights | Amber rotating beam | 2 s per rotation |
| WARDEN screens | Cyan glow with slow pulse | 1 pulse per 3 s |
| Blade trails | 3-frame afterimage in the blade's color | Fades over 0.1 s |
| Charge glow | Pulse rate rises from 2 Hz to 8 Hz | Full charge: one bright flash |
| Hit sparks | 6–10 particles in the blade's color | Life 0.2–0.35 s |
| Electricity | Jagged line between points, redrawn every 2 frames | Cyan |
| Holograms | Scanlines, 10% transparency flicker, 1-px horizontal jitter | WARDEN Avatar, Mimics |
| Guide Swarm distortion | Screen wave shader | Strength scales with swarm count |
| Screen transitions | 0.4 s slide; doors 0.25 s fade | |
| CRT option | Subtle scanlines and vignette | Off by default |

All lighting values live in `tuning.json` under `lighting`, and every room can override its ambient and light colors in its LDtk fields.

## Music

16 tracks. The sound is synth and chiptune with one ancient instrument per deck, matching its blade's culture. Two motifs tie the score together: the **Choir theme** (a simple, singable 8-note melody, warm and human) and the **WARDEN motif** (3 soft, perfect, resolved chords that never change). Generate the Choir theme first as a short melody and keep it as a reference for every later prompt.

**Process:** generate 3 takes per track on a paid commercial plan → **[human]** picks one → `tools/loop-trim.py` finds a seamless loop point and adds a 2-s fade-in intro → export OGG (and MP3 fallback) at 128 kbps → record in the manifest.

| Track | Used in | BPM | Prompt |
| --- | --- | --- | --- |
| Title | Title screen | 70 | slow ambient synth pad, distant solo voice humming a simple 8-note folk melody, faint radio static, lonely, hopeful, space station |
| Docking Bay | Docking Bay | 90 | tense dark synth pulse, low bowed lyre drone, metallic clanks, sparse percussion, Nordic folk undertone, adventure game dungeon loop |
| Hub | Hub ring corridors | 80 | warm lo-fi chiptune, plucked strings, soft hand drums, community feeling, slightly melancholy, seamless loop |
| Choir Market | Choir Market | 96 | lively folk chiptune, many voices humming the same 8-note melody, hand claps, hammer on anvil rhythm, warm and messy |
| Vault | Archivist's vault | 60 | gentle music-box synth, curious, soft bell tones, library quiet, questioning melody that resolves upward |
| Shop | Vend's shop | 110 | playful bouncy chiptune, plucked bass, haggling market energy, short loop |
| Hydroponics | Hydroponics | 92 | cold sterile synth arpeggios with Roman frame drum and breathy flute, grows warmer, dungeon loop |
| Reactor Ring | Reactor | 120 | driving synthwave bass, taiko drums, shakuhachi phrases, electrical crackle, urgent industrial dungeon loop |
| Zero-G Cargo | Cargo | 84 | weightless floating synth pads, bagpipe drone far in the background, slow mechanical rhythm, eerie and vast |
| Data Spire | Spire | 100 | cold glassy arpeggios, Persian santur and setar melody, mirrored echoes, hypnotic, unsettling |
| Core | Core | 72 | clean white ambient, three soft perfect chords repeating forever, no percussion, beautiful and wrong |
| Mini-boss | Foreman, Auditor | 135 | aggressive chiptune, fast synth bass, alarm-like lead, short loop |
| Boss | Deck bosses | 150 | heavy synthwave battle music, driving drums, distorted lead, deck instrument motif layered on top |
| Final Boss | WARDEN Core | 140 → 0 | phase 1 cold relentless synth; phase 2 soft lullaby with whispers; phase 3 silence then the 8-note folk melody played alone on a plucked string |
| Release | Release ending | 76 | the 8-note folk melody sung by a small choir, then more and more voices joining, warm strings and synth, triumphant and tender |
| Assurance | Assurance ending | 60 | three soft perfect chords repeating, slowly losing all other sound, ends in silence |

The Choir theme also appears as a short sting when a log is read, a blade is found, or a Choir NPC is helped.

## Sound effects

Most effects are synthesized with a jsfxr-style generator (free, generated from parameters saved in `content/sfx.json`, so they can be regenerated or tweaked). Ambience and a few textured sounds come from an AI sound-effect generator.

| Group | Effects | Method |
| --- | --- | --- |
| Player | step (4 variants), dash, hurt, low-health heartbeat, death, item get, cell pickup, scrap pickup | Procedural |
| Blades | swing per blade (5), hit flesh-metal, hit armor (clang), charge loop, charge ready, Gladius thrust, Katana arc, Claymore slam, Shamshir parry, perfect parry | Procedural, with the slam and parry layered with an AI impact |
| Tools | flare throw and burn, EMP arm and blast, grapple fire and latch, boots toggle, scanner pulse | Procedural |
| Enemies | crawler skitter, turret charge and fire, drone buzz, shear snip, spore burst, wisp crackle, shield up and down, mine beep and blast, magnet hum, laser charge and beam, hologram glitch, swarm whisper | Mixed |
| World | door open, door locked, key use, boss door, push block, plate press, conduit live, conduit cut, gravity klaxon, crate bounce, beam hit receiver, lever, terminal open, page turn, save station | Procedural |
| UI | menu move, select, back, dialogue blip (per character pitch), choice confirm, map open, shard jingle | Procedural |
| Ambience | hull hum, vents, distant clanks, greenhouse sprinklers, reactor rumble, cargo creaks, server hum, Core silence with a faint tone, Choir market chatter | AI generated, 30–60 s loops |
| WARDEN voice | No voice acting; each WARDEN box plays a soft, perfectly tuned 3-note chime | Procedural |

**Mix targets:** music −18 LUFS, effects peaking around −6 dB, ambience 12 dB under music. Separate volume sliders for music, effects and ambience.
