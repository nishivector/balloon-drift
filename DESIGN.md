# Balloon Drift — Design Document
### Round 18

---

## IDENTITY

**Game name:** Balloon Drift
**Tagline:** *She just wanted to see what was up there.*

---

### Protagonist

**Name:** Dot
**Type:** A small red heart balloon — not a shape, not a metaphor, a balloon. She has two tiny dark eyes painted near her top, and a 28px white specular glint on her upper-left that makes her look alive. Her string hangs 40px below her, curling slightly to the right.

**Backstory:** At the birthday party on Maplewood Court, eight-year-old Maya was distracted by the cake and let go. Dot didn't know what to do, so she did the only thing she could: she went up. She doesn't have a plan. She just knows that up feels right and the world down there is full of things that want to pop her.

---

### World

**Setting:** Suburban America, the vertical slice. You see it from the side: wooden power poles, sagging lines, TV antennas on shingled rooftops, a water tower in the distance, clotheslines between apartment windows, pigeons arguing on ledges. It's a Sunday morning in late spring — golden hour still lingers at the horizon. Everything is ordinary and oblivious to Dot.

**World feel (2 sentences):** You can almost hear the distant lawnmower two streets over and smell cut grass rising on warm air. The world feels large and indifferent in the way suburbs always do — beautiful if you squint, cluttered if you look too closely.

**Emotional experience:** Wistful ascent — quiet longing, gentle dread, unexpected joy

---

### Reference Games

| Game | DNA |
|---|---|
| **Flappy Bird** | One-axis danger, constant upward pressure, instant death, retry compulsion |
| **Alto's Adventure** | Environmental beauty in motion, a world that exists beyond the camera, music that matches the emotional beat |
| **Doodle Jump** | Vertical scrolling, player is always rising, obstacles come from all sides, casual but punishing |

---

## VISUAL SPEC

| Element | Value | Notes |
|---|---|---|
| **Background sky** | `#87AECF` | Exact value from brief; the color of a clear Sunday morning |
| **Sky gradient bottom** | `#B0CEDE` | Lighter near horizon — 200px linear gradient at bottom |
| **Sky gradient top** | `#5A8FB8` | Deepens slightly at top of viewport (altitude feeling) |
| **Player (Dot)** | `#E8223A` | Warm, saturated red. Not crimson, not candy — this is a proper balloon red |
| **Player highlight** | `#FFFFFF` | Single ellipse, 14×10px, upper-left of balloon, alpha 0.65 |
| **Player string** | `#C8A87A` | Warm tan, rendered as a 1.5px bezier curve, oscillates with a sin wave |
| **Power line poles** | `#6B4F3A` | Weathered dark wood brown |
| **Power line wires** | `#3A3530` | Nearly-black dark grey, 1.5px stroke |
| **Pigeons** | `#8B8982` | Greige body, `#C44` red feet |
| **Near-miss sparks** | `#FFE566` | Pale warm yellow — 6 particles, 120ms lifetime |
| **Pop particles** | `#E8223A` + `#FF6B6B` | Mix of balloon red and lighter pink, 12 fragments, spin outward |
| **Clouds** | `#FFFFFF` alpha 0.72 | Simple rounded rect clusters, parallax at 0.3× scroll speed |
| **Rooftops / BG** | `#D4956A` (brick) + `#6E6B66` (shingle) | Far background layer, parallax at 0.15× scroll speed |

**Bloom:** Yes — strength `0.4`, threshold `0.7`. Applies to Dot only. Her red heart should glow very slightly, like she has a little warmth inside her.

**Vignette:** Yes — gentle, radial, `rgba(0, 20, 40, 0.28)` at edges, feathered to nothing at 60% of viewport. Frames the action without darkening the sky.

**Camera:** Side-scrolling vertical. World scrolls downward; Dot appears to rise. Camera X is fixed. Camera Y tracks Dot with a 0.08 lerp factor (soft follow — she leads slightly, creates tension). Dot's resting screen position is at 65% of canvas height from top (she floats in the lower-upper portion, giving room to see what's coming).

**Player silhouette (5 words):** Round heart, dangling string below

---

## SOUND SPEC

### Music Identity

**Genre/vibe:** Wistful music-box lullaby that slowly blossoms into something orchestral and full of feeling — like a child's toy growing up.

**Character of the music:** It sounds like a memory of something that hasn't happened yet — innocent, a little melancholy, genuinely beautiful.

**The Hook:** A four-note ascending motif — G4, B4, D5, G5 (a major arpeggio climbing one octave). Played on the music-box layer, it opens every 2-bar phrase like a tiny declaration. By Level 4, when the orchestral layers swell in underneath, this same four-note hook is played by a string ensemble and it hits like a miniature sunrise. Players will hum it. The motif is simple enough that it sticks immediately but resolves with enough warmth that it doesn't become annoying.

---

### Arrangement

| Layer | Instrument | Tone.js Synth | Role | Enters | Exits |
|---|---|---|---|---|---|
| **1 — Music Box** | Celesta / music box | `PluckSynth` with `attackNoise: 0.8`, `dampening: 3400`, `resonance: 0.88` | Main melody — the hook motif plus 8-bar phrase | Immediately on game start | Never — always present |
| **2 — Pizzicato Strings** | Plucked strings | `PolySynth` → `Synth` with `oscillator.type: "triangle"`, `envelope: {attack:0.005, decay:0.35, sustain:0, release:0.1}` | Counter-melody, fills space between hook notes | Level 2 (score 150) | — |
| **3 — Warm Pad** | Soft orchestral strings | `AMSynth` with `harmonicity: 1.5`, `detune: -8`, routed through `Reverb(3.5)` | Harmonic glue — holds the chord underneath | Level 3 (score 400) | — |
| **4 — Glockenspiel** | High shimmer arpeggios | `PluckSynth` with `dampening: 4800`, `resonance: 0.95` | Arpeggios on the offbeats — adds altitude and brightness | Level 4 (score 700) | — |
| **5 — Orchestral Swell** | Full strings + woodwind | `FMSynth` with `harmonicity: 2`, `modulationIndex: 3.5`, routed through `Reverb(5)` + `Chorus` | The emotional peak — makes the hook motif feel enormous | Level 5 (score 1100) | — |
| **6 — Bass Breath** | Low sustained pad | `Synth` with `oscillator.type: "sine"`, very slow `LFO` on volume | Root note sustain — gives the whole thing a pulse | Level 3 (score 400) | — |

---

### BPM and Structure

- **BPM:** 72 (exact, from brief)
- **Bar length:** 4 beats
- **Loop length:** 16 bars (64 beats = ~53.3 seconds per full cycle)
- **The hook motif occupies bars 1–2** of every 4-bar phrase, giving it a clear "breath" pattern
- **All new layers enter on bar 1** of the next 16-bar loop after their score threshold is crossed — never mid-bar

---

### Dynamic Music (State Changes)

| State | Trigger | Music Response |
|---|---|---|
| **Near-miss** | Dot passes within 20px of an obstacle | A single glockenspiel note rings one beat after the near-miss — like a little gasp of relief |
| **Pigeon flock approaches** | Flock spawns off-screen | Bass Breath LFO speed doubles for 2 bars — creates subtle unease before visible threat |
| **Death (pop)** | Dot touches obstacle | All music cuts instantly. A single PluckSynth note (C4) drops in pitch via `detune: -1200` over 400ms — the sound of a balloon dying. Silence for 0.8 seconds, then music re-enters at reduced volume (−8dB) and rebuilds over 2 bars |
| **Level up** | Score crosses threshold | The hook motif plays one octave higher than usual for exactly one repetition — a celebration without breaking the mood |
| **Win / Above the Clouds** | Score reaches 1500 | All 6 layers play simultaneously for the first time. Volume ceiling lifts from −6dB to 0dB. The hook motif loops twice with the full orchestral swell beneath it |
| **Altitude warning** | Dot reaches 80% of win height | High strings layer adds a tremolo effect (±2 semitones, 8Hz) — not scary, just electric |

---

### Start Screen Music

**Different mood-setter.** On the start screen, only Layer 1 (Music Box) plays, at −10dB, with a longer reverb tail (`Reverb(6.0)`). It loops the 8-bar phrase quietly. It sounds like the toy box in a child's room at dusk — beckoning, not urgent. When the player taps START, the reverb tail fades over 1.5 seconds as the in-game music begins from bar 1, Layer 1, full volume.

---

### Sound Effects

| Effect | Trigger | Tone.js Implementation | Why It Fits |
|---|---|---|---|
| **String tug (left drift)** | Tap left | `PluckSynth`, note A3, `attackNoise: 1.2`, 80ms. Slightly lower pitch. | The string pulls — you feel the physical object |
| **String tug (right drift)** | Tap right | `PluckSynth`, note C4, `attackNoise: 1.2`, 80ms. Slightly higher pitch. | Same as left but brighter — directional audio cue |
| **Drift coast** | No input, balloon gliding | Subtle `NoiseSynth`, `type:"pink"`, very quiet (`−28dB`), 200ms fade — wind on latex | The silence isn't empty; air has texture |
| **Near-miss sparkle** | Within 20px of obstacle | `PluckSynth` (glockenspiel register), note G5, `resonance:0.97`, single ping + `Reverb(1.5)` | That little gasp of relief made musical |
| **Pop / Death** | Contact with obstacle | `NoiseSynth` with `type:"white"`, `envelope:{attack:0.001, decay:0.12, sustain:0}`, + PluckSynth C4 pitched down via `detune: -1200` over 400ms | The sharp burst of air, then the sad falling tone |
| **Pigeon coo** | Pigeon spawns | `FMSynth`, `harmonicity:0.5`, `modulationIndex:2`, notes C3→B♭2 over 200ms, at `−18dB` | Recognizable bird sound — establishes world and warns player |
| **Level up chime** | Score threshold crossed | `PolySynth` plays G4-B4-D5 staccato (the first three notes of the hook), `envelope:{attack:0.01, decay:0.4, sustain:0}` | The hook motif as a reward — cohesion between music and effects |
| **Win sting** | Score 1500 reached | All layers surge + `PluckSynth` plays the full four-note hook motif one final time, high octave (G5-B5-D6-G6), long reverb tail `Reverb(8)`, fades over 4 seconds | The hook motif is the win. You heard it all along; now it's saying goodbye |
| **Cloud ping** | Entering cloud layer (Level 5) | `PluckSynth`, note D6, `resonance:0.99`, very soft `−22dB`, with `Reverb(4)` — ephemeral, crystalline | Clouds should sound different from air — lighter, finer |

---

## MECHANIC SPEC

### Core Loop

Dot rises automatically and continuously. The player steers her left and right to thread through obstacles. Everything kills her in one hit. The longer she stays alive, the higher she climbs and the richer the music becomes.

### Player Spawn

**Spawn coordinates:** `x = canvas.width / 2`, `y = canvas.height - 90px`
The balloon begins fully visible, centered horizontally, with her string just above the bottom edge of the screen. The world starts moving upward (i.e., obstacles scroll down) 1.5 seconds after game start — giving the player exactly 1.5 seconds to understand the visual and prepare.

### Primary Input

| Input | Behavior |
|---|---|
| **Tap / click left half of screen** | Apply leftward horizontal impulse: `vx -= 2.8px per frame for 6 frames` |
| **Tap / click right half of screen** | Apply rightward horizontal impulse: `vx += 2.8px per frame for 6 frames` |
| **Hold left** | Continuous impulse while held, same value per frame |
| **Hold right** | Same |
| **No input** | Horizontal velocity decays by `vx *= 0.92` each frame (gentle float resistance — feels like air) |
| **Vertical** | Dot always rises at the current `riseSpeed` — player has zero vertical control |

**Max horizontal speed:** ±5.5px/frame — enough to feel responsive, not so much that she zips across the screen.

**Canvas bounds:** Dot cannot leave the left or right edge. At either wall, `vx` is set to 0 and she bounces back at `vx * -0.35` (soft wall, slight rebound).

### Key Timing Values

| Value | Amount |
|---|---|
| Initial rise speed | 1.4px/frame |
| Rise speed at Level 5 | 2.6px/frame |
| Obstacle scroll speed (matches rise speed) | Same as rise speed |
| Obstacle spawn interval (Level 1) | Every 3.2 seconds |
| Obstacle spawn interval (Level 5) | Every 1.5 seconds |
| Near-miss window | 20px from Dot's center to obstacle edge |
| Death state duration | 0.8s freeze + 1.2s respawn fade-in |
| Balloon bob amplitude (idle) | ±4px vertical sine, period 2.8s |
| String sway offset | 6px horizontal sine, period 2.1s, slightly out of phase with bob |

### Dead State

Dot touches any obstacle → immediate pop. The balloon body is replaced by 12 red/pink particle fragments that spin outward at random angles (`velocity: 1.5–4px/frame`, fade over 600ms). The string falls limply. The pop sound fires. Music cuts instantly with the death tone. After 0.8 seconds of silence, the screen shows "Gone..." in small text (`#87AECF + 40% dark`, centered). After 1.2 more seconds, the player respawns at the beginning of the current level (obstacles reset for that level, score does not reset). Music re-enters quietly and rebuilds.

### Score

- **+1 point per second** of survival (continuous)
- **+15 points** for each near-miss (within 20px, not a hit)
- **+50 points** for clearing each level (reaching the altitude threshold)
- Score displayed top-right in the same round sans-serif as the title, small, `#FFFFFF` at 70% opacity

### Win Condition

Dot reaches an altitude of **1500 score-units** (which corresponds to approximately 4.5 minutes of clean play at Level 5 speed). Above this threshold, the music win sting fires and a final cinematic plays: the camera pulls back slightly (zoom out 15%), the sky deepens to a rich twilight `#4A7BA8`, and Dot floats into a single small white cloud and disappears. Text: *"She made it."*

### Lose Condition

Dot is popped. She respawns. There is no "game over" — only "try again from this level." There are no lives. The emotional cost of death is the silence that follows.

---

## DIFFICULTY CURVE

| Level | Name | Rise Speed | Spawn Interval | Obstacle Types | New Element | Duration/Goal |
|---|---|---|---|---|---|---|
| **1** | Morning Calm | 1.4px/frame | 3.2s | Horizontal power lines only | Establishing the world — single wire spans, clear gaps (min 160px) | Score 0–150 |
| **2** | The Telephone Exchange | 1.7px/frame | 2.6s | Power lines + angled spans | Lines appear at 15°–30° angles, forcing diagonal navigation | Score 150–400 |
| **3** | Pigeon Parade | 2.0px/frame | 2.2s | Lines + flying pigeons | Pigeons fly in from screen edges, horizontal movement at 1.2–2.0px/frame, 2–3 per wave | Score 400–700 |
| **4** | Chimney Row | 2.3px/frame | 1.8s | Lines + pigeons + chimney stacks | Chimney stacks emerge from bottom (static), creating vertical obstacle columns with narrow pass-throughs (min 90px) | Score 700–1100 |
| **5** | Above the Clouds | 2.6px/frame | 1.5s | All of the above + wind gusts | Wind gusts: every 8–12 seconds, a horizontal force of `vx ±1.8px` pushes Dot for 1.5 seconds (small UI wind-direction indicator appears 1 second before gust) | Score 1100–1500 |

**Level transition:** Smooth, no interruption. New obstacle types phase in over 5 seconds — existing obstacles clear, new ones begin spawning. No level card, no cutscene. The world just... intensifies.

---

## LEVEL DESIGN DETAIL

### Level 1 — Morning Calm
- Single horizontal wire spans across the full width, alternating left-to-right and right-to-left attachment points so gaps appear on different sides
- Gap width: 160px minimum
- One wire visible on screen at a time
- Purpose: Teach the player that wires kill and that drifting slowly is survivable. Let them feel the floatiness.

### Level 2 — The Telephone Exchange
- Wires spawn in pairs now (two parallel wires 80px apart), and some cross diagonally (15°–30° angle)
- Gap width: 140px minimum
- Two wires may be on screen simultaneously
- Purpose: Introduce spatial reading. The player must plan 2 seconds ahead.

### Level 3 — Pigeon Parade
- Pigeons introduced. Each pigeon: 32×24px sprite (grey body, red feet, minimal detail). Flies in from left or right edge, travels at 1.2–2.0px/frame horizontally. Pigeons don't track Dot — they fly straight.
- 1–3 pigeons per wave, waves every 6–9 seconds
- Power lines continue from Level 2
- Gap width: 130px minimum
- Purpose: Introduce moving threats. The player must manage two attention zones.

### Level 4 — Chimney Row
- Chimney stacks: rectangular obstacles, 40px wide, emerge from the bottom of the screen. They don't move vertically — they're fixed structures Dot must fly around. Spawn in groups of 2–3 with 100–120px gaps between them.
- All prior obstacles continue
- Gap width (lines): 120px minimum
- Purpose: The level feels dense and urban. For the first time, the player may feel genuinely trapped.

### Level 5 — Above the Clouds
- Wind gusts activate. UI indicator: a small animated arrow (`→` or `←`) appears top-center of screen, 1 second before the gust. Arrow is `#FFE566`, 24px, fades out when gust begins.
- Gust applies `vx` force each frame for 1.5 seconds: `vx += (direction) * 1.8` — strong enough to feel like weather, weak enough that immediate correction is possible
- All prior obstacles continue at maximum density
- Sky gradient shifts: bottom is now `#87AECF`, top is `#4A7BA8` — visually marking altitude
- Cloud wisps begin drifting through the screen — purely visual, not obstacles
- Purpose: The home stretch. The music is full. The sky is beautiful. Dot is almost free.

---

## THE MOMENT

When Dot's music-box melody plays its four-note hook for the first time with the full orchestral swell underneath it — the exact moment Layer 5 enters in Level 5 — and the player realizes this gentle little tune has been building toward *this* all along.

---

## EMOTIONAL ARC

**First 30 seconds:** Confusion becomes delight. The player understands the input within 3 seconds (the first wire gives them a clear choice). The music-box is charming and quiet. They feel the floatiness of the balloon and immediately want to protect Dot. She's small and the world is big.

**After 2 minutes:** Concentration. Pigeons and wires coexist now. The pizzicato strings have joined the music-box and the track feels like it has warmth and intention. The player has probably died once or twice and restarted. They want to see what's higher. They're invested.

**Near the win:** Something close to elation. The sky is deep blue, clouds drift past, the full arrangement is playing, and Dot's little red form against that blue feels genuinely moving. The player isn't thinking about mechanics anymore — they're just watching her rise and hoping she makes it.

---

## THIS GAME'S IDENTITY IN ONE LINE

*This is the game where you guide a lost heart balloon through a world that doesn't know it's alive, and you root for it more than you expected to.*

---

## START SCREEN

### Idle Animation (Three.js canvas)

The start screen renders the same world as the game, in a static high-altitude slice — the camera is elevated: rooftops visible at the bottom third of screen, a broad expanse of `#87AECF` sky above. Dot floats in the center-upper area, performing her idle animation:

- **Vertical bob:** Sine wave, amplitude ±8px, period 2.8 seconds
- **String sway:** Bezier curve control point shifts ±6px horizontally, period 2.1 seconds, slightly out of phase with bob
- **Subtle rotation:** ±2° tilt, in sync with horizontal sway (she leans slightly as the string swings)
- **Background:** Two distant telephone poles visible at lower-left and lower-right, wires sag between them. A single distant pigeon sits on the right wire, still. A slow parallax cloud drifts left to right at 0.2px/frame.
- **Atmosphere:** No obstacles animate. This is the world at rest. Dot is the only thing moving.

### SVG Overlay Spec

**Option A — Title treatment:**
- Text: `Balloon Drift`
- Font: Rounded, hand-friendly sans-serif (CSS `font-family: 'Nunito', 'Quicksand', system-ui`; weight 700)
- Size: `72px` desktop / `48px` mobile
- Color: `#FFFFFF`
- SVG filter: `feGaussianBlur stdDeviation="4"` + `feComposite` — gives the title a soft warm glow
- Glow color: `#FFD580` (warm gold) at 60% opacity layered behind the white text
- Position: Centered horizontally, 38% from top of screen

**Option B — Iconic silhouette:**
- A simplified SVG of Dot — heart shape + string — appears to the right of the title text
- Size: 64×80px (including string)
- Fill: `#E8223A` with white highlight ellipse
- The silhouette gently bobs in sync with the Three.js Dot below (CSS animation mirror, same period)
- This creates visual unity: the SVG Dot and the 3D Dot are clearly the same character

**START button:**
- Small, centered below title, 20% from bottom
- Text: `Tap to fly`
- Style: No border, no box — just the text in `#FFFFFF` at 80% opacity, 22px, letter-spacing 0.12em
- Pulses opacity 80%→100%→80% on a 1.6 second cycle — a breathing, not a blinking

---

## IMPLEMENTATION NOTES FOR PROGRAMMER

1. **Balloon hitbox:** Circle, radius 22px from Dot's center. The visual is 48px tall (heart) but the hitbox is smaller — this is intentional. It feels fair. Players will never feel cheated.
2. **Power line hitbox:** The wire itself is 1.5px wide visually, but hitbox is 6px tall centered on the wire. The pole structure is not a hitbox (you can brush past the pole).
3. **Pigeon hitbox:** Circle, radius 12px from pigeon center.
4. **Chimney hitbox:** Exact rectangle, no padding.
5. **Wind gust:** Do not apply wind during the first 0.5 seconds of Level 5 — let the player settle. First gust comes at `8 + random(0, 4)` seconds after Level 5 begins.
6. **Near-miss detection:** Check every frame. If Dot's center is within 20px of any obstacle's nearest edge, AND she did not collide: +15 points, spawn 6 `#FFE566` particles from Dot's position, play glockenspiel ping. This can only trigger once per obstacle (mark each obstacle as "near-miss credited" when triggered).
7. **Parallax layers (scroll speed multipliers, all relative to rise speed):**
   - Far background (rooftops, sky gradient): 0.15×
   - Mid background (poles, water tower): 0.45×
   - Gameplay layer (obstacles): 1.0×
   - Clouds: 0.30×
8. **String physics:** The string is a quadratic bezier curve. Start point: bottom-center of Dot. End point: 40px directly below. Control point: offset by `vx * 6` horizontally (string lags behind motion). Update every frame.
9. **Level transition:** When score crosses a level threshold, mark `transitioning = true` for 5 seconds. During transition, no new obstacles spawn. Existing obstacles finish their path and exit. After 5 seconds, new obstacle type begins spawning. This creates a brief "breath" between levels without ever stopping.

---

*End of Design Document — Balloon Drift, Round 18*
