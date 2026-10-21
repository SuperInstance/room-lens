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


## v1.2 (first half) — AGENT PROBE mode: stats banking

Per `docs/BRIEF-PROBE-MODE.md`: the ASCII feed's reader can be an AGENT
watching stats, so the renderer may SWITCH MODES FREQUENTLY — every mode
dwell is an experiment and the stats are the result; the optimal
operating point becomes something LEARNED per room, not the three
hand-set tiers (which remain starting values only).

- **AGENT PROBE toggle** (default OFF; the human view stays calm). While
  ON, the active dwell OWNS the renderer — cols, fps target, ramp
  contrast, edge layer on/off — stepping through a programme of 1.5 s
  dwells; the status line announces each dwell.
- **Programmes are data**: the embedded default sweeps 9 modes
  (48–128 cols × 8–30 fps targets, edges-off and high-contrast
  variants); `?programme=<url>` fetches a new sweep without redeploying
  (the evolve pattern).
- **Per-dwell measured stats bank into the bundle** as `probeDwells[]`:
  achieved fps, frame entropy (Shannon over the glyph histogram), edge
  density, inter-frame change, bytes/frame, motion score, pose
  availability — plus 1–2 sample ASCII frames per mode. Rides the same
  DOWNLOAD/UPLOAD bundle as everything else.
- **`?autoprobe=1`** drives the whole loop unattended (camera → probe →
  console-reported stats) — the E2E hook and the fleet's no-thumbs mode.
- Headless E2E (fake moving camera): dwell `c48-f8` measured 7.9 fps
  against its 8 fps target, entropy 2.78 bits/glyph, edge density 0.048,
  13.4 kB/frame, 39-row sample banked — the engine works end to end.
  Bugs the E2E caught: a missing button lookup killed the whole script;
  the sensor-permission ask pends forever without a user gesture in
  headless (autoprobe skips it; real runs keep it); and the autoprobe
  click fired before the button listeners registered (deferred one
  tick).
- The adaptive-policy half (learning the operating point from banked
  stats) is the next build, per the brief's order.
