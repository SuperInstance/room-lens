# AUDIO-FIX-REPORT — the dual-1 ambient-audio defect, fixed and proven

**Status: FIXED. Re-take can go FOR REAL.** (2026-10-22, zcode; this is
the signal document.)

## 1. Root cause, named from the source (not inferred)

`index.html`, before this fix:

- `startMicRecorder()` created the walkabout's ambient recorder as
  `new MediaRecorder(new MediaStream(stream.getAudioTracks()))`.
- `btnRir`'s MEASURE ROOM handler created a **second**
  `new MediaRecorder(new MediaStream(stream.getAudioTracks()))` — on the
  **same underlying audio track**.
- On the phone's browser, the second recorder **evicts** the first (one
  MediaRecorder per track): the ambient recorder silently went
  `inactive` mid-walkabout, with no `onstop` wired at that moment — its
  chunks stayed in `micChunks`, orphaned.
- `stopMicRecorder()` then hit its early-return —
  `if (!micRecorder || micRecorder.state === 'inactive') resolve(null)`
  — and **discarded the orphaned chunks**. `audioData` stayed `null`,
  so `buildBundle()` wrote `meta.audio.present = false`.
- The five sweeps banked fine because `measureRec` was the surviving
  recorder. Exactly the observed defect: RIR measures present, ambient
  audio absent.

## 2. The fix (SuperInstance/room-lens, commits `2a0564c` + `fd903e7`)

1. **Serialize, never stack**: MEASURE ROOM now stops the ambient
   recorder FIRST (banking what it has as a part), runs its sweep alone,
   then restarts the ambient recorder. No moment ever has two
   MediaRecorders on one track.
2. **Salvage**: `stopMicRecorder()` no longer discards an evicted
   recorder's chunks — if chunks exist, they bank as a part. (Belt for
   the suspenders above.)
3. **Parts, honestly**: a measure-interrupted take banks `audioParts[]`
   (each with `startedAt`/`offsetMs`/`interrupted`); `meta.audio` says
   how many parts and where the gaps are (the gaps are the measures).
4. **The stop path banked nothing** (found by the self-test): `btnStop`
   called a raw `micRecorder.stop()` that bypassed `stopMicRecorder`, so
   the final ambient part died unbanked. `btnStop` now banks first,
   then releases the mic.
5. **`?autoprobe=audioselftest`**: the regression hook — camera →
   WALKABOUT (ambient on) → MEASURE ROOM mid-walk (the eviction
   trigger) → stop → console-reports the bundle's audio fields. Fake
   devices; Casey does not perform for tests.

## 3. The self-test numbers (the proof)

Local headless Chrome, fake media devices, against the served page
(liaison route `/lens`, which serves this repo's `index.html` live):

- **Before the stop-path fix**: `audioPresent: true, audioParts: 1,
  rirMeasures: 1, ambientBytes: 16,437` — the criterion passed, but the
  post-measure ambient was still being lost (an interrupted take owes
  TWO parts; one banked).
- **After (commit `fd903e7`)**: **`audioPresent: true, audioParts: 2,
  rirMeasures: 1, ambientBytes: 107,886, frames: 72`** — ambient audio
  present, in both parts (pre-measure + post-measure), with a swept-sine
  measure coexisting mid-walk. The regression is gone at both failure
  points.

## 4. Where the fixed page is served

**Live at `http://<bench-LAN-IP>:8765/lens`** — the liaison server now
serves the room-lens checkout at `/lens` (communicator commit follows
this report; the route reads `../room-lens/index.html` from disk per
request, so every `git push` to room-lens is live on reload — the
evolve pattern, no deploy step). The phone opens it over the LAN, same
as the bench pages. (The Cloudflare static host carries only the
chiaroscuro doors today; wiring a token-free Pages deploy for room-lens
is a separate nice-to-have — the LAN route is the phone-reachable,
push-to-deploy path now.)

## 5. What the dual-1 take still gives us

- The 5 RIR sweeps + 54 keyframes + 270 ASCII frames are good data for
  the splat lane.
- Cross-device sync from the sweeps is **weak on this take**: the
  witness chain's energy map shows the expected sweep arrivals
  (02:19:52 → t≈75s, 02:22:33 → t≈236s, ...) landing in the chain's
  QUIET buckets (rms 3-5 m-units) — the phone-speaker sweeps at gain
  0.4 across the room are at or below the witness's ambient floor.
  Envelope correlation found no honest match (score ~0.004 raw; the
  300-2000 Hz band envelope r≈1 at a constant witness moment = template
  self-match artifact, not signal). **Reported as unmeasurable on this
  take, not guessed.** The re-take protocol wants either a louder sweep
  (gain rail allows 0.4→0.3... no, UP needs a rail decision — flag to
  Casey) or a clap-based sync as originally briefed, or both devices
  closer for at least one sync event.
