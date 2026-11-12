# IELTSGo Female Tutor Avatar — Integration Guide

Demo file: `female-tutor-demo.html` (single self-contained HTML file, no
external dependencies). Female avatar only — the male avatar comes later,
after this one is validated.

## What this is

A web-native animated AI tutor: an original SVG character (professional
woman, navy blazer, brown side-ponytail) driven by a JavaScript
lip-sync/expression engine. It was built web-native because this machine
has **no Adobe tools** — there are no Illustrator (`.ai`), Character
Animator (`.chproj`), or After Effects files, and none are claimed.

## How it connects to IELTSGo

IELTSGo's Speaking screen already has everything the avatar needs:

1. **Coach speech (TTS audio → mouth).** IELTSGo's AI coach replies are
   spoken through the proxy's TTS endpoint, which returns audio. The
   avatar's **audio-file path** is built for exactly this: feed the TTS
   `<audio>` element (or an `AudioBufferSourceNode`) through a
   `MediaElementSource` → `AnalyserNode`, and the engine drives mouth
   opening from the real amplitude (louder = more open) and switches
   visemes by amplitude band. No new server work is needed — the audio
   already exists; only the analyser hookup is added.
   - In the demo, use the "Audio-file lip-sync" section with any MP3/WAV
     to see this path work.
2. **Exact sync (optional, later).** If IELTSGo ever gets word/phoneme
   timings for its TTS audio, the engine exposes
   `speakWithTimeline(text, [{t, viseme, open}])`, which plays precise
   visemes at millisecond offsets. The demo's "Sample viseme timeline"
   button shows this working with a hand-made timeline. This is the
   hook for frame-accurate sync; it is not required for the amplitude
   path above.
3. **Speaking screen states.** The engine already mirrors IELTSGo's
   conversation states: `Speaking` (coach talking), `Listening`
   (learner talking — head tilt + slow nod), `Idle`. Wire IELTSGo's
   mic-start/stop and TTS play/end events to `setExpression()` /
   `speak()` / `stop()` via the exposed `window.IELTSGoTutor` API:
   `speak(text)`, `speakWithTimeline(text, timeline)`,
   `setExpression(name)`, `setViseme(name)`, `stop()`.
4. **Expressions.** Nine states are built in: Neutral, Happy,
   Encouraging, Thinking, Listening, Surprised, Explaining, Concerned,
   Celebrating. IELTSGo can map them to feedback moments (e.g.
   Celebrating on a high band, Encouraging after a correction,
   Thinking while the AI reply is being generated).
5. **PNG export.** The demo's "Download PNG" button renders the current
   SVG pose to a transparent PNG (800×1040) via canvas — useful for
   thumbnails, the Play-Store-style listing image, or social posts.

## What to copy into IELTSGo

- The `<svg id="avatarSvg">` block (character art) and the engine
  `<script>` (or the `window.IELTSGoTutor` API surface) into the
  Speaking screen of `index.html`.
- Keep the SVG fills as presentation attributes (as written) so the
  PNG export keeps its colours.
- The avatar is transparent-background by design, so it sits on any
  IELTSGo screen background.

## Why no Character Animator puppet?

Adobe Character Animator puppets (`.chproj`) run only inside Character
Animator / After Effects — they cannot run in a browser. To use one on
the web you must pre-render it to video, which cannot lip-sync live to
arbitrary AI speech. That is why this avatar is SVG + JS: the mouth is
driven in real time by the actual audio the learner hears.

## Future alternative: Rive

If IELTSGo ever needs a compiled animation runtime (e.g. richer skeletal
motion, shared art across web + native), **Rive** is the natural next
step: small runtime, state machines, and web playback. It would mean
rebuilding the character in Rive's editor and driving mouth shapes from
the same amplitude/viseme signals. Not needed for v1 — the SVG engine
already covers lip-sync, expressions, and idle life.

## Known limits (honest)

- The `speak()` path uses the browser's `speechSynthesis` voices and a
  timed viseme cycle (~8 changes/sec, pulsed on word boundaries). It
  looks like talking, but it is not phoneme-accurate — true accuracy
  needs the amplitude path (audio file) or the viseme timeline.
- The avatar is front-facing only (no turnarounds in the demo).
- The demo has not been visually verified in a browser by the builder —
  visual QA (face quality, blink, lip-sync feel) is the parent's step.
