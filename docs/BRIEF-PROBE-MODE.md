# BRIEF - Probe mode: the page as an active sensor (Casey, 2026-10-21)

## The inversion
The ASCII feed does not owe a human a stable, legible view. Its
reader is an agent watching stats. So the renderer - on-device, as
close to the camera as possible - may SWITCH MODES FREQUENTLY:
resolution sweeps, ramp changes, edge layer on/off, fps targets,
exposure behaviour. Confusing for a human; pure information for
an agent. Every mode dwell is an experiment, and the stats are
the result.

## What a probe run banks
Per mode dwell (1-2 s each, tagged by mode id in the bundle):
- the mode parameters (cols, fps target, ramp, edge on/off),
- measured stats: achieved fps, frame entropy, edge density,
  inter-frame change rate, bytes per frame, pose/motion state,
- 1-2 sample ASCII frames per mode.
Plus the audio analogue already shipped: the MEASURE ROOM sweep
is the same philosophy in sound - a probe, not a performance.

## What it teaches
The optimal operating point is per room, per light, per task -
learned, not hand-set. Probe stats feed adaptive policy v2: which
resolution actually resolves THIS rooms edges, where the edge
layer earns its bytes, what the splat lane prefers downstream.
The three hand tiers of v1.1 become starting values only.

## Shape
- An AGENT PROBE toggle on the page (default off; the human view
  stays calm when off).
- Probe programmes are data (a list of mode dwells), so the fleet
  can ship new sweeps without redeploying the page - the evolve
  pattern.
- Safety rails as in the evolve shell: honest status always,
  probe runs announce themselves, one tap stops.
