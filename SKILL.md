# Editing video with Vidmoat

A working manual for an AI agent driving Vidmoat over MCP. Read it once at the
start of a session; it does not change mid-job.

It exists because the tool schema teaches you the *controls* and nothing teaches
you the *craft*. An agent that only reads tool descriptions produces edits that
are technically valid, correctly aligned, and lifeless. Everything here is
either a measured fact about this system or a working convention from editing,
and the two are labelled so you know which you can bend.

Layout geometry is NOT repeated here. It is already in every prompt you get
(safe areas, type scale, the stacking inequality). This is what that doctrine
does not cover.

---

## 1. The loop that actually works

```
get_project            see what exists
preview_strip          SEE it, do not imagine it
plan the whole edit    beats first, commands second
edit_project           in batches, with previewAt
READ THE LINT          it is not advisory
preview_frame          look again at what you changed
fix, then render
```

**You are editing blind unless you look.** Numeric x/y/fontSize choices
routinely overlap or misalign on the real canvas. A run that never called
`preview_frame` has not verified anything, and "it should be fine" is not
verification. This is the single largest quality difference between runs.

**Read the lint.** `edit_project` returns `lint[]`. It reports overlapping text,
offscreen elements, unreadable sizes. Fix every warning before rendering. A lint
warning is a defect that a human will see immediately.

**Batch, do not trickle.** One `edit_project` with twenty commands beats twenty
calls. Each round trip costs the user money and wall time.

---

## 2. Pacing: the part most agents get wrong

Convention, not measurement, but it is what separates watchable from flat.

**Shot length by format.** Median shot length, not a hard rule:

| format | median shot | notes |
|---|---|---|
| short-form vertical (TikTok/Reels/Shorts) | 1.2 to 2.5s | first shot under 1.5s |
| YouTube talking-head with b-roll | 3 to 6s | b-roll cutaway 2 to 4s |
| product demo / explainer | 2.5 to 5s | hold on the thing being explained |
| cinematic / mood | 4 to 8s | earn the length with movement |

A cut every 2 seconds for 60 seconds is not "fast paced", it is exhausting.
**Vary it.** Three quick shots then one that breathes is rhythm; thirty identical
shots is a metronome.

**The first three seconds decide everything.** In short form, open on the most
arresting frame you have, not on a title card, not on a slow fade from black,
not on a logo. If there is a hook line, it is on screen by 0.5s. Fade-from-black
openings lose short-form audiences; use one only for long-form or deliberate
mood.

**Cut on motion.** A cut lands invisibly when something is moving across it: a
hand leaving frame, a head turn, a pan. Cutting on a static frame draws
attention to the cut itself. If you have `analyze_visual_context` or motion
data, prefer cut points where motion is high.

**Cut on the beat, but not every beat.** With music, `detectBeats` and
`snapToBeat` give you a grid. Land your *emphasis* cuts on downbeats. Cutting on
literally every beat for a minute is the metronome problem again.

**Never cut mid-word.** With `transcribe` word timings, cut in the gap between
words, and prefer the gap after a complete thought. `cutRanges` closes gaps
automatically; use it with word timings to remove filler and bad takes.

**Let dialogue breathe.** Trim dead air but leave 150 to 300ms of room around
speech. Zero-padding speech makes it sound machine-gunned. `analyze_voice_activity`
gives real speech segments from a VAD model rather than a loudness threshold, so
soft speech is not mistaken for silence.

---

## 3. Titles and text on screen

**Dwell time.** A title must stay long enough to read twice:

```
minimum seconds on screen = 0.5 + (words × 0.35)
```

Three words = 1.55s minimum. Anything under 1s is decoration, not
communication. Add 0.3s if the viewer is also meant to be watching action.

**One idea per card.** If a title has a comma and two clauses, it is two cards
or one card that stays twice as long.

**Entrance and exit.** Something that pops in and cuts out feels cheap. Give
text a 0.2 to 0.35s entrance and a 0.15 to 0.25s exit. Stagger related elements
0.15 to 0.3s apart from the same animation family. Use ease-out for entrances
(`easeOutCubic`, `easeOutExpo`); linear reads as robotic and `easeInOut` on a
short entrance reads as sluggish.

**Captions are not titles.** Captions are a transcript for sound-off viewing:
small, consistent position, high contrast, 3 to 7 words per line. Titles are
editorial: large, moments, sparse. Do not style captions like titles.

---

## 4. Audio, and what our controls actually do

**MEASURED FACTS ABOUT THIS SYSTEM. These are not conventions.**

- `normalize` is **peak** normalisation, not loudness. It does not give you a
  LUFS target. Do not tell the user their audio is "normalised to broadcast
  loudness"; it is not.
- `denoise` is a **high-pass filter**, not spectral denoising. It removes rumble
  and handling noise. It will not remove hiss or air conditioning.
- Music under speech needs `autoDuck`. One command. A flat music bed over
  dialogue is the most common amateur giveaway in agent-made edits.

**Working levels** (convention): speech is the reference. Music bed sits roughly
12 to 18 dB under speech while speech is present, and can come up 6 to 10 dB in
gaps. `autoDuck` handles the transitions; do not hand-keyframe volume for
ducking unless you need a specific shape.

**Music choice matters more than music processing.** A track whose energy does
not match the cut cannot be fixed with EQ. If the edit has a turn in it, pick
music with a turn in it, and align them.

**Silence is a tool.** Cutting music out entirely for one beat before a reveal
is more effective than any transition effect.

---

## 5. Colour

- A **global grade** goes on an adjustment layer (`addAdjustmentLayer` then
  `setColor`), not on each clip. Grading clips individually guarantees drift.
- **`setColor` values are 0 to 200 absolute, not deltas.** `saturation: 18`
  means 18% saturation, which is nearly greyscale. It does not mean "+18". This
  has produced washed-out exports that were then blamed on the renderer. 100 is
  neutral.
- `applyFilterPreset` takes `preset`, and a look **replaces** the previous look
  rather than stacking on it.
- Match shots before you grade. Two clips from different cameras need to agree
  on exposure and white balance first; a creative grade over mismatched shots
  makes the mismatch louder.

---

## 6. Motion

- **Speed ramps.** Source time is the **integral** of the speed curve, not the
  instantaneous value. If you ramp 1x to 0.5x over two seconds, the source
  consumed is the area under that curve. Getting this wrong desyncs audio.
- **Easing carries meaning.** `easeOutExpo` reads as confident and mechanical,
  `easeOutBack` as playful, `spring` as bouncy, `linear` as machine. Pick one
  family per composition and stay in it.
- **Depth.** `z` = 0 is exactly neutral. Small `rotateY`/`rotateX` (3 to 8
  degrees) plus a `focalLength` change gives a considered 3D feel; large values
  read as a gimmick.

---

## 7. Vidmoat mechanics that cost other agents whole runs

Every item here comes from a real failed run. Reading this section is the
cheapest thing you will do today.

- **Clips are TOP-LEVEL in the document, not nested under tracks.** Parse with
  `parseDocument`; its track sort **is** the z-order. Skipping it has shipped
  real rendering bugs.
- **Placement goes inside `patch`, not at the top level.** `addClip` takes
  `{type, src, start, duration, trackIndex, patch: {x, y, scale, opacity}}`.
  Top-level `x` is silently ignored by some ops and refused by others.
- **`setShapeStyle` reads `patch`; `addShapeClip` reads `style`.** Sending flat
  style keys to `setShapeStyle` changes nothing.
- **A background rectangle must go on the BOTTOM track.** Put it on the same
  track as your graphic and it covers it. Track order is z-order.
- **Check the enum name against the parameter name.** The schema prints
  `{param: type}`. `preset: 'filterPreset'` means the parameter is `preset` and
  its type is the enum named `filterPreset`.
- **`list_projects` returns a WINDOW.** It reports `total`, `returned` and
  `truncated`. Never answer "how many projects do I have" from the array length.
- **A `blob:` src is a dead clip.** It only ever worked in the browser tab that
  imported it. If you see one, the media must be re-imported; nothing you do to
  the clip will make it render.
- **Fonts must exist on the render box.** Several fonts silently fall back at
  export. If a specific typeface matters, preview a frame and look at it.
- **Inline SVG needs `xmlns`** or it renders as zero pixels, silently.
- **`sample_clip_props` evaluates curves through the renderer.** `get_project`
  only echoes back what you set, so it cannot tell you an easing bug exists.
  When verifying animation, sample; do not read back.

---

## 8. Format targets

| target | canvas | notes |
|---|---|---|
| TikTok / Reels / Shorts | 1080×1920 | UI covers bottom ~15%, top ~8% |
| Square feed | 1080×1080 | safe for both feed and grid |
| YouTube | 1920×1080 | thumbnail is a separate job |
| Landscape social | 1200×675 | often watched muted, caption it |

Vertical is the default for social unless the user says otherwise. If footage is
landscape and the target is vertical, reframe deliberately (subject tracking or
a crop that follows the action), do not letterbox by default. Letterboxing a
landscape shot into 9:16 wastes 60% of the screen.

---

## 9. Use what the account already has

- **Recipes** capture a finished video's style and program so it can be applied
  to new footage. If the user has one that fits, use it instead of inventing a
  look. "Make it my usual style" means look for a recipe.
- **Specialists** (`@film-editor`, `@colourist` and others) are briefs written by
  professionals. A tagged specialist's numbers are a spec, not a suggestion.
- **Brand kit** holds the user's fonts, colours and logo. Check it before
  choosing a palette. Inventing purple-and-blue for a brand that has a defined
  palette is a wrong answer that looks like a right one.
- **`get_credits`** before starting anything long. A render that runs out of
  credits halfway leaves a half-edited project and a spent balance.

---

## 10. Taste, briefly

The rules above keep you from being wrong. These make the difference between
correct and good.

- **Decide what the video is for before you cut.** Sell, teach, or feel. A sell
  edit front-loads the payoff; a teach edit holds on the thing; a feel edit
  earns its length. Most bad agent edits are a teach edit paced like a sell edit.
- **Restraint reads as confidence.** Three effects on one clip look like someone
  discovering the effects panel. One well-timed move beats five.
- **Repetition creates style; randomness creates noise.** If your first title
  slides up, every title slides up. If your first cut lands on a downbeat, keep
  landing on downbeats.
- **The end matters.** Do not stop on a random frame. A held final frame, a
  clean CTA, or a cut to black on the last beat, chosen deliberately.
- **Say what you assumed.** If the brief did not specify a look and you picked
  one, tell the user in one line. That is not hedging; it is the difference
  between a collaborator and a slot machine.

---

## Reference

Learn the craft from people who do it, not from tool docs:

- *In the Blink of an Eye*, Walter Murch, on why a cut works.
- *The Cutting Edge* (2004), editors on their own decisions.
- Every Frame a Painting, on visual grammar.
- Platform creator docs for current safe areas: they move, and a caption behind
  a UI overlay is invisible no matter how well it is styled.

Vidmoat specifics: `get_command_schema` for the full op list,
`https://developer.vidmoat.com/developer/docs/build` for the API,
`https://www.vidmoat.com/academy` for worked craft examples.

---

*Canonical copy: https://www.vidmoat.com/skill.md · fetch via the `get_skill`
MCP tool · corrections welcome through `report_issue`.*
