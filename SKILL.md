---
name: motion-reel
description: >
  Make a code-made motion-graphics reel for Demon Clips (9:16 Instagram Reel / YouTube Short), like the
  "Made in code" promo: kinetic captions wiped in by a moving object, a physics ball that bounces and
  stamps lines, light and night scenes, a magnifier over real code, a pill that morphs into a
  "Comment KEYWORD" card, sound effects and music synthesized in Python, a Hindi voice-over (AI or the
  user's own via the Voice-over Recorder app), and the platform outro. Use this whenever the user wants
  a new animated reel, promo, explainer animation, "make a video like this", "recreate this reel in our
  style", a kinetic-typography or motion-design short, a reel from a reference link, or changes to the
  motion, timing, sound, voice-over or outro of such a reel, even if they don't say "skill" or "Remotion".
  Not for cutting or reposting someone else's footage (that is ClipWrap) or for the recorder app itself.
---

# Motion reel (Demon Clips Studio)

Every pixel and every sound of these reels is made in code: Remotion for the picture,
`pipeline/synth.py` for the sound, Edge TTS or the user's own recording for the voice. The reference
build is **"Made in code"**: `remotion/src/promo/MadeInCode.tsx`, `pipeline/promo.py`,
`reels/261120-made-in-code/`. Read them before writing a new reel. Copy their patterns, not their copy.

Work inside `D:\Claude-Chrome\wearethedemon\studio` and follow `studio/CLAUDE.md` (environment, render
path, A: drive for scratch, ask before downloads, commit only when asked, no Claude attribution).

## What the user expects every time (don't make them ask)

The user is direct and gets frustrated when something has to be asked for twice. Deliver the whole
thing the first time:

- **Voice-over.** Hindi narration, timed line by line to the beats. Use the AI voice (Edge TTS
  `hi-IN-MadhurNeural`) unless they've recorded their own in the Voice-over Recorder. A reel with
  only music came back as "no voice over?".
- **The outro appended.** The postable files are `-ig.mp4` (Follow) and `-yt.mp4` (Subscribe), with
  the outro's own sound effects and one soundtrack mastered across reel and outro.
- **Both scripts.** Roman Hinglish and Devanagari, so 4 files: `{roman,deva}-{ig,yt}`.
- **Lively but smooth motion.** Things pop, jiggle, pulse and overshoot. Nothing jerky or rushed:
  give big moves time (a pendulum pass is 1.5 s; 1 s was "too fast and not smooth"), and give
  anything fast a motion-blur trail.
- **House look: Paper Glass Lab.** Pale grid, white glass, channel red #E5162D / #B80F22, ink and
  white. Night interludes (#0E0E12 plus a red glow) are allowed for rhythm, but the user hasn't
  formally approved them, so keep them optional. **No yellow, no emoji in the video, no fake numbers,
  no verified badges.**
- **Honest claims.** If the reel says "every sound is code", it is. Code on screen is this reel's
  real source, and a test checks it.
- **Build what they asked for.** If they ask for X, don't offer a lighter version and ask again.

## Workflow

### 1. Format and idea
If there's a reference reel, take only its **format**: structure, rhythm and kinds of moves. Never
its script, captions or designs. Write 5-8 beats, one line each: what moves, what text appears, what
sound plays. Verify any fact before it goes into the copy. Use a "Comment KEYWORD" card only with a
real deliverable in `reward.md`. Draft that file and mark it as needing the user's approval; they
send the DMs by hand.

### 2. Folder and copy
Create `reels/<YYMMDD>-<slug>/` with:
- `copy.yaml`:
  - `handle`, `keyword`, `brand`.
  - A `roman:` and a `deva:` block for the on-screen text.
  - A `voice:` block: `id`, `rate`, `lines: [{at, text, roman}]`. `text` is Devanagari for the TTS,
    with English words spelled the way a hi-IN voice says them (कोड, क्लॉड, एडिटर, डी एम). `roman` is
    what the user reads in the recorder.
- `reward.md`: DRAFT until the user approves it.
- `notes.md`:
  - the build command and the files to post;
  - a before-posting checklist and a cover-frame suggestion;
  - post copy: an IG caption with at most 5 hashtags, a YT title of at most 100 characters, and a
    description with 3-5 hashtags.

### 3. Beats sheet first
Put the beat times in one JSON file, `remotion/src/promo/<slug>.beats.json`, in seconds.
- **Shared:** the composition and the Python mix both read it, which keeps every sound on its frame.
- **Names:** name beats after what happens (`cut`, `wall`, `land[]`, `reveal`, `stretch`, `toCard`,
  `keyword`, `brand`, `fadeOut`).
- **Ending:** the last 0.4 s stays calm and silent, so the loop back to frame 0 is clean.
- **Length:** 25-30 s of reel, plus the 3.5 s outro.

### 4. Composition
Write `remotion/src/promo/<Name>.tsx` and register it in `Root.tsx`, with `calculateMetadata` taking
the length from the beats.
- **Schema:** a zod schema for every piece of copy (`script: "roman" | "deva"`, lines, card, brand,
  `audio`, `fps`). The roman copy is the default.
- **Time:** in seconds, through `../motion` (`seconds`, `springAt`, `ramp`, `wiggle`, `pulse`, `loop`,
  `stagger`) and the springs in `brand/tokens.ts` (`SPRING.ui` 18/160/0.8, `SPRING.stamp` 11/220/0.7).
  Never frame counts, so any fps works.
- **Kit:** `Backdrop`, `NightBackdrop`, `FitLine` (fits a line to a width), `Card`, `Chip`, `Stamp`,
  `Rich`, `Phone`, `CaptionBar`, `SafeOverlay`.
- **Text:**
  - Keep it inside the safe zone: x 60-950, y 250-1560. Nothing meant to be read goes under 34 px.
  - Roman captions are UPPERCASE Rajdhani; second lines are Poppins 600 italic in red.
  - Devanagari isn't uppercased. It needs bigger max sizes (about 148/120 against Roman 150/128, the
    pill 72 against 64), `lineHeight ≥ 1.25` and `lang="hi"`.
- **Variety:** mix 4-6 moves from the vocabulary below, in a different order each reel. YouTube treats
  repeated templates as inauthentic content.

### 5. Sound and voice
See "Sound" below. Music and sound effects go on a `Bus` at the beat times. The music ducks −9 dB and
the effects −5 dB under the voice. Then comes the limiter, then loudnorm to −14 LUFS. Each voice line
ends before the next one starts.

### 6. Pipeline module
`pipeline/promo.py` is wired to the one Made-in-code reel (`SLUG`, `COMPOSITION`, `BEATS_JSON`, its
`music`/`sfx`).

**The first time you build a second reel, generalise it** instead of copying the file:
- Move the shared parts (`load_copy`, `voiceover`, `soundtrack`, `_master`, `render`, `filmstrip`,
  `final`) into a module that takes a reel config: slug, composition id, beats path, and that reel's
  `music(b)` and `sfx(b)`.
- Keep `promo.py` as a thin config for Made in code, and give the CLI a `--reel <slug>`.
- Do the same for the recorder: `pipeline/recorder.py` `Project` takes the slug, and
  `pipeline record --reel <slug>` works, so the Voice-over Recorder app can voice any reel.

Keep the tests passing, and say in your report that you made this change.

### 7. Filmstrip review before the full render
Render key-beat stills (6 per row, `--filmstrip`) into
`A:\claude-workspace\screenshots\demon-studio\<slug>\` and look at them yourself. Check:
- that text is readable and inside the safe zone, and Devanagari isn't too small;
- for empty moments (a card sitting blank for 0.6 s looks broken, so start its text within about
  0.4 s);
- that long last lines don't touch the edges, and layered effects line up.

Fix, re-render the stills, then render the video.

### 8. Render, outro, final files
Build the mix, then render each script. `final()` then:
- appends the platform outro: `O.render_outro(platform, script, keyword=..., reward=...)`, with its
  third bullet "Comment KEYWORD: reward";
- joins the two videos and masters one soundtrack across both;
- encodes H.264 CRF 18, BT.709 limited range, AAC 320k 48 kHz, +faststart.

### 9. Verify, then report
- **Format** (`ffprobe`): 1080×1920, 30 fps, yuv420p, BT.709, AAC 48 kHz stereo, length = reel +
  outro.
- **Loudness:** −14 ±1 LUFS and true peak ≤ −1 dBTP on every final file.
- **Audio present:** check RMS per second with ffmpeg.
- **Outro frames:** headline, card, tap, Following/Subscribed, keyword bullet.
- **Tests:** `uv run python -m pytest -q` passes. Add tests for the new reel:
  - the on-screen code is real;
  - the beats are in order;
  - the props are valid for both scripts;
  - every voice line ends 0.05 s before the next;
  - the soundtrack runs full length with a silent last 0.4 s;
  - the keyword has a reward.

Report in plain words:
- the 4 files to post, as links;
- the reel beat by beat, as a short list;
- what you checked, and what you couldn't (you can't listen, so they should play it);
- open decisions: reward approval, cover frame.

Remind them that the VS Code preview can play these files silently ("why there is no audio?"). VLC,
Windows Media Player or Chrome play them properly.

## Motion vocabulary (code in MadeInCode.tsx)

One **hero object drives the reel** (the red ball). Everything else reacts to where it is, which gives
cause and effect: the ball passes, the text appears.

**Paths**
- **Pendulum.** `ballAt`: `x = 540 − SWING·cos(t·TURN)`, `y` on the string's circle (`stringY`),
  with `TURN = π / pass` and a 1.5 s pass. `PendulumString` has a soft shadow and snaps to the pivot
  after the cut.
- **Hop.** `hop(a, b, u, h)` is a straight line minus `4·h·u·(1−u)`: a real parabola with apex height
  `h`. Chain hops down a staircase of landing points. Ease the final settle with
  `Easing.out(Easing.back(1.5))`.
- **Flight into a wall.** Ease-in quad, plus a small `sin(u·π)` arc.
- **Glide.** `lensAt`: Catmull-Rom through keyframes, easing in on the first segment and out on the
  last.
- **Motion-blur trail.** `Trail`: sample the last 0.09 s of the path and draw `ceil(length / 5 px)`
  circles (at most 48) under an SVG `feGaussianBlur`, shrinking and fading toward the tail
  (`0.16·(1−k)^1.4`). The copies overlap, so it reads as one comet, never as beads.

**Text**
- **Wipe that follows the object.** `SceneSwing`/`SwingText`: each caption is a full-frame layer
  clipped with `clip-path: inset()` at the ball's x. The new caption shows behind the ball, the old
  one ahead of it, and the direction alternates each pass.
- **Stamp on landing.** `StackLine`: the line pops with `springAt(c, at, SPRING.stamp)` exactly on
  the landing beat.

**Impacts and life**
- **Squash and stretch.** `squashAt`: for 0.14 s after a landing, `sx += 0.3·sin` and
  `sy −= 0.26·sin`, with `transform-origin: 50% 100%`. A wall hit stretches the other way.
- **Ripple.** A flat ellipse ring growing from the contact point over 0.45 s.
- **Burst.** 14 sparkles and dots, seeded with `random()` so renders repeat exactly. Use it only on
  the one big moment (the keyword).
- **Jiggle and pulse.** `wiggle(c, at, dur, hz, amp)` shakes rotation; `pulse(c, at, dur, amount)`
  bumps scale. Multiply pulses together, and add a soft pulse every second at the end so the card
  keeps breathing.
- **Shine.** A skewed white gradient bar crosses the pill once.

**Scenes**
- **Light to night.** The ball drags a `NightBackdrop` panel across the paper. Its leading edge is a
  tilted line (`panelEdge`/`edgeX`, tilt easing 0 → 14° → −6°) used to clip the night layer as a
  polygon.
- **Back to paper.** Cut back on a strong beat.
- **Camera push-in.** 3-6 % per scene is fine, but never in a scene with a magnifier.

**Focus and reveal**
- **Ball to lens.** `lensRadius`/`Lens`: the ball inflates into a red ring (spring), fills with
  glass, grows a handle and glides over a grey code layer while the rest dims. The reveal words then
  pop inside the glass, and it closes back into the ball (`Easing.in(Easing.back(1.4))`).
- **Magnifying.** Inside the glass, draw the same layer scaled by `ZOOM` (2.1) with
  `transform-origin` at the lens centre. If the base layer moves (a camera push), the magnified copy
  drifts out of line.
- **Code on screen.** It comes from `code-lines.ts`: verbatim lines of this reel's own source, kept
  in sync by a test, colour-highlighted only inside the lens.

**CTA morph**
- **The shape.** `Morph` builds one rounded rectangle from three springs: width `sW`, height `sH`
  (looser, so it wobbles) and card `sC`. `w = 60 + 840·sW + 20·sC`, `h = hPill + 430·sC`, and the
  radius goes from pill to 52.
- **The sequence.** Ball → pill with the CTA text sliding up → jiggle and shine → card. The card
  holds MessageCircle + "comment karo", then a 190 px italic keyword stamped in from scale 1.7 to 1
  with a burst, then the post line.
- **The ending.** `Brand` lockup (emblem, Anton wordmark, mono tag), a fade to night, and a still
  last 0.4 s. A `@handle` watermark sits top-left throughout.

## Sound (`pipeline/synth.py`, `pipeline/audio.py`)

`S.Bus(seconds)` is a stereo timeline:
- `add(clip, at, gain_db, pan)` places a clip.
- `mix(other, db)` sums two buses.
- `duck_under(key, depth_db)` sidechain-ducks under another bus.
- `fade_out(start, end)` and `peak()`.

Read every time from the beats JSON. The rule: **every visual beat gets a sound**, and fast motion
gets air. Give each whoosh its own `seed`, and pan small sounds ±0.15-0.7.

| Visual | Sound | Gain dB |
|---|---|---|
| Fast pass or flight | `whoosh(dur, lo, hi, peak, seed)`, peaking where it moves fastest, panned with the direction | −11 to −16 |
| Pass extremes, UI ticks | `tick(freq, dur)` | −24 |
| Cut, string snap | `snap()` + `riser(dur)` into the next hit | −13 / −17 |
| Big hit (wall, keyword) | `impact()` | −9 / −10 |
| Landing | `thud(150, 50, 0.4)` + `pop(900+80·i, 300)`, rising with each landing | −8 / −13 |
| Lights up, completes | `ting(Hz)`: 1318.5, 1568, 1760, 2093, 2637 | −13 to −15 |
| Inflate, stretch, morph | `boing(f0, f1, dur)` | −10 to −14 |
| Reveal, sparkle, shine | `shimmer(base, gap)` | −13 to −22 |
| Shrink, close | `zip_down()` | −14 |
| Jiggle | `rattle()` | −13 |

**Music** at 120 BPM, A minor `Am F C G`, 2 s bars (a 1.5 s pass = 3 beats, so motion and music
lock).
- `pad` per bar, quieter and darker where the viewer reads or looks.
- A `pluck` Karplus-Strong arpeggio in eighths, two octaves up.
- `kick` on the motion accents and four-on-the-floor in the build.
- `hat` and `sub` only in the night scenes.
- A final `pad` under the wordmark.

**Voice**
- **AI voice:** Edge TTS 7.3.1, `hi-IN-MadhurNeural` at +8 % (`hi-IN-SwaraNeural` is the female
  option). It's cached in `build/voice/`, and only script text is ever sent.
- **Own voice:** `voiceover(mode="auto")` prefers the recorder take (`audio/voice-take.json`), then
  `audio/my-voice.*` (one take cut at the pauses by `pipeline/ownvoice.py`), then the AI voice.
  `--voice ai|own` forces one.
- **Lines:** each line is 2-6 words and starts on its beat. Measure every clip: it must end
  ≥ 0.05 s before the next line, and the last one before `fadeOut`. Shorten the words if not.
  High-pass 80 Hz, placed at −2 dB.

**Mix and master**
- **`soundtrack()`:**
  - ducks the music −9 dB and the effects −5 dB under the voice, and mixes the bed at −4 dB;
  - fades from `fadeOut − 0.2` to `seconds − 0.4`, keeping a peak guard of 0.98.
- **`_master()`:** a look-ahead limiter **first**, then two-pass linear loudnorm to −14 LUFS /
  −1.5 dBTP. Without the limiter, a peaky synth mix pushes loudnorm into dynamic mode, which pumps.
- **The outro:** `outro_sound(platform)` reads `outro.beats.json`. It puts a click exactly on the
  finger tap, a shimmer on the morph, and YT bells or IG pops. It's mastered **together with** the
  reel soundtrack in `final()`, so the loudness doesn't jump at the join.

## When the user wants their own voice

Point them to **Voice-over Recorder** in the Start menu (a Windows app built from
`recorder/windows/`).
- The reel plays without the AI voice, and each line turns red on its cue.
- Takes are checked line by line, and the Sync slider nudges a take by ±300 ms.
- **Make videos** rebuilds the four files with the take they picked.

For a new reel, the recorder first needs the `--reel` change from step 6. After any change to
`RecorderApp.cs`, run `recorder/windows/install.ps1`.
