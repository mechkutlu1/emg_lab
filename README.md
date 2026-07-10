# BIO1 &middot; Exoskeleton Assistive Control Bench

An interactive, browser-based simulator for the elbow-exoskeleton laboratory (measurement of EMG signals and their use in assistive control). It lets students explore the lab's control schemes without the hardware, with a live arm/exoskeleton animation and instrument traces for joint angle, servo command, interaction force and EMG.

Everything runs client-side in a single `index.html`. No build step, no server, no dependencies (fonts load from Google Fonts with system fallbacks).

## What it teaches

Each task maps to a tab and shows the matching Arduino listing alongside the physics:

- **Task III &mdash; Position control** (Listing 5): the servo's inner loop tracks a demanded angle; students see tracking error and the effect of loop delay.
- **Task IV &mdash; Admittance control** (Listing 7): `positionDesired -= gain*(forceIs - forceDesired)`. The device becomes a virtual spring; students vary `gain`, `forceDesired` and `forceOffset`, and switch on travel limits (the impedance extension).
- **Task V &mdash; EMG control** (Listing 9): `motorPosition = map(emgSignal, 0, 600, 0, 150)`. Muscle activity drives the joint; a threshold deactivates the motor to remove jitter.
- **Optional &mdash; EMG + admittance**: EMG is mapped to a desired interaction force which the admittance loop then delivers.

Inputs students can manipulate: voluntary effort (or hold-to-flex / spacebar), an auto flex pattern, hand load, a master **assist ON/OFF** to demonstrate the need for assistance, and an **attentional-focus** selector (external / control / internal).

### Attentional focus

Based on Ay's 2021 PhD (Sakarya University of Applied Sciences), the focus condition scales the EMG the muscle emits for the *same* movement: internal focus (attend to the muscle) raises EMG activity, external focus (attend to the load) lowers it, per the constrained-action hypothesis. The physical effort and joint torque are unchanged, only the electrical signal the exoskeleton reads. A live EMG-activity (RMS) meter shows the effect. In the EMG and combined modes this inverts the usual coaching advice: internal focus gives a stronger, higher-SNR command that clears the deactivation threshold reliably, so it is the better myoelectric control input, while external focus can leave the signal too weak to drive the motor. The tabs also note that focus type is itself decodable from the EMG, so it can act as an extra input channel.

## The model

Planar elbow. State `[theta, omega, thetaServo]`, integrated with RK4 at 5 ms:

```
J*omega_dot = T_muscle + T_int + T_grav - b*omega
T_grav = -(m*g*lc + load*g*L)*cos(theta)
T_int  = kc*(thetaServo - theta) + bc*(omegaServo - omega)   # exo <-> arm coupling
omegaServo = (thetaCmd - thetaServo)/tau                     # servo inner loop
```

Controllers only ever set `thetaCmd`, exactly like `myservo.write(positionDesired)`. The interaction torque `T_int` is what the force sensor reports; EMG is generated from muscle activation plus noise. Parameters live at the top of the `<script>` block (`P`, `C`) and are easy to retune for a class.

## Deploy to GitHub Pages

1. Create a repository, e.g. `exoskeleton-control-bench`.
2. Add `index.html` (and this `README.md`) to the repository root and push:
   ```bash
   git init
   git add index.html README.md
   git commit -m "BIO1 exoskeleton assistive control bench"
   git branch -M main
   git remote add origin https://github.com/<you>/exoskeleton-control-bench.git
   git push -u origin main
   ```
3. On GitHub: **Settings &rarr; Pages &rarr; Build and deployment**, set **Source = Deploy from a branch**, **Branch = main**, folder **/ (root)**, then Save.
4. The site appears at `https://<you>.github.io/exoskeleton-control-bench/` within a minute or two.

To try it locally, just open `index.html` in a browser, or run `python3 -m http.server` in the folder and visit `http://localhost:8000`.

## Notes

- Works on desktop and mobile; keyboard focus and reduced-motion preferences are respected.
- The physics constants are illustrative, chosen so the control effects are clearly visible rather than to reproduce a specific rig.
