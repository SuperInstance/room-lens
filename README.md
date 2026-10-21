# room-lens

**Phone room-map page (v1).** Open it on a phone, tap START CAMERA, and the
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
  frame counts. Nothing leaves the phone until you tap DOWNLOAD BUNDLE.

v1 limits: bundle is download-only (no upload path yet), no splat fitting
in-page, orientation-glyph election is a reduced 2×3-subpixel form of the
canon 4×6 descriptor, and poses depend on the phone granting motion
sensors (frames bank regardless).
