# room-lens

**Phone room-map page (v1.1).** Open it on a phone, tap START CAMERA, and the
room renders live as ASCII. Tap START WALKABOUT and walk the room slowly:
the page banks timestamped ASCII frames + device-orientation pose stamps,
plus a JPEG keyframe every 5th frame, into one downloadable JSON session
bundle for the splat lane (SuperInstance/splat-room).

- Renderer lifted from [SuperInstance/chiaroscuro](https://github.com/SuperInstance/chiaroscuro):
  the Mirror door's capture core (buffer-canvas downscale → luminance →
  contrast stretch → density-ramp glyphs in source colour) and the
  `sobel_shape.js` shape layer (per-cell gradient descriptor; strong
  edges elect an orientation glyph `- | / \` — "characters are shapes,
  not pixels").
- **Adaptive fidelity** (per `ADDENDUM-ASCII-PHONE.md`): still phone →
  finer grid, lower fps (detail harvest); moving phone → coarser grid,
  higher fps (temporal resolution). Motion is read from
  deviceorientation/devicemotion when available, frame-difference
  otherwise.
- Honest status only: the page continuously reports measured fps, the
  active tier, grid size, motion score, pose availability and banked
  frame counts. Nothing leaves the phone until you tap DOWNLOAD BUNDLE or UPLOAD BUNDLE.

## v1.1 — audio, room measure, upload

- **Audio banking on the shared clock.** START CAMERA requests camera and
  microphone in the same gesture (raw chain: echo cancellation, noise
  suppression and auto-gain all off, so the room is captured, not
  processed). START WALKABOUT starts a MediaRecorder (opus/webm where
  offered) alongside frame banking; the bundle carries the audio as
  base64 plus `meta.audio` (start timestamp, offset vs session start,
  drift note). If the mic is refused, the page falls back to video-only
  and the status line says audio is OFF — never silently.
- **MEASURE ROOM (30s, experimental).** Plays a 20 s log swept-sine
  (20 Hz → 16 kHz, gain 0.4) through the phone speaker while a dedicated
  recorder listens, plus 4 s of decay tail. The recording banks into the
  bundle as a named `rirMeasures[]` asset — the raw material the splat
  lane deconvolves into an impulse response / RT60 room metadata.
- **UPLOAD BUNDLE.** Posts the same JSON the download saves to the
  `room-lens-upload` Cloudflare Worker, which stores it in the R2 bucket
  `room-lens-bundles` keyed by session id and returns the fetch URL:
  `GET https://room-lens-upload.casey-digennaro.workers.dev/bundle/<sessionId>`
  (`GET /list` shows stored sessions). Typical 5-minute walkabout bundle
  is ~6–9 MB (ASCII frames + JPEG keyframes + opus audio, base64); the
  endpoint takes it in one POST.
- Known v1.1 limits: the upload endpoint has no auth (unguessable
  session ids only — it is a single-user tool, not a public drop box);
  no splat fitting in-page; orientation-glyph election is a reduced
  2×3-subpixel form of the canon 4×6 descriptor; poses depend on the
  phone granting motion sensors (frames and audio bank regardless).
