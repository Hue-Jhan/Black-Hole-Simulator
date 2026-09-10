# Gargantua

A real-time black hole renderer in a single `index.html` — Three.js just
opens a WebGL canvas; everything you see is one GLSL fragment shader
raymarching curved light paths around a Schwarzschild black hole.

**Run it:** open `index.html` in any WebGL-capable browser. No install, no
build. **Controls:** drag to orbit, scroll to zoom.

## The physics

### Event horizon

A non-spinning black hole of mass `M` has an event horizon at the
**Schwarzschild radius**:

```
r_s = 2GM / c²
```

Nothing that crosses `r_s` — not even light — escapes. The simulation works
in natural units where `r_s = 1` (the uniform `uRs`); every other distance in
the scene (disk radii, camera range) is expressed as a multiple of it.

### Bending light: the geodesic equation

Light doesn't travel in straight lines near a strong gravity well — it
follows a **null geodesic** of the Schwarzschild metric. In the orbital
plane, using `u = 1/r`, the exact photon trajectory satisfies:

```
d²u/dφ² + u = (3/2) · r_s · u²
```

The right-hand side is the general-relativistic correction; without it
(`d²u/dφ² + u = 0`) you get plain straight-line optics. Solving this exactly
needs an RK4 integrator in `φ`. This shader instead uses the equivalent
Cartesian **force approximation**, integrated directly in 3D:

```
a(pos) = -1.5 · r_s · |h|² / r⁵ · pos,   h = pos₀ × dir₀
```

`h` is the photon's angular momentum, fixed at the ray's start. Because the
force always points along `±pos` (a *central* force), `h` is exactly
conserved — `d/dt(pos × v) = v×v + pos×a = 0` — so the photon provably stays
in a single plane, matching real Schwarzschild geodesics, even though the
force law is a leading-order stand-in for the exact equation above. Rays are
marched with adaptive step size (fine near the hole, coarse far away) until
they either cross `r_s` (absorbed) or escape to sample the starfield —
which is exactly why the **Einstein ring** (a lensed, ring-shaped image of
whatever's directly behind the hole) just falls out of the raymarch, with no
special-case code for it.

Real gravity has infinite range, but for a compact, cinematic silhouette the
shader windows the force with `atten = 1 / (1 + (r/uDiskOuter)^8)` — an
artistic (non-physical) choice. It's steep enough that distortion stays
tight around the disk, but stays C-infinity smooth (unlike a hard on/off
cutoff, tried and discarded: it left a visible ring-shaped seam in the
starfield, since bending stopped dead at one exact radius instead of fading
out naturally).

### The accretion disk

Matter spiraling into a black hole heats up and glows before crossing the
horizon. The innermost radius a stable circular orbit can exist at is the
**ISCO** (innermost stable circular orbit):

```
r_isco = 3 · r_s     (Schwarzschild, non-spinning)
```

Below that, orbits are unstable and matter plunges in — so the disk in this
scene starts at `uDiskInner ≈ 3·r_s`. It's rendered as a thin ring in the
horizontal plane; density comes from animated fractal (fBm) noise advected
by roughly Keplerian differential rotation, `ω(r) ∝ r^(-3/2)`, and color
follows a blackbody-ish ramp — white-hot near the ISCO, cooling to red at
the outer edge.

### Relativistic beaming

Since the disk spins, one side always moves toward the camera and the other
away — and special relativity says approaching sources look brighter and
bluer, receding ones dimmer and redder. The **relativistic Doppler factor**
for a source moving at velocity `v` is:

```
D = 1 / (1 - v·μ)         μ = cos(angle between velocity and line of sight)
```

Observed intensity scales as `I_obs = D³ · I_emit` (relativistic beaming).
The shader computes `v(r) ≈ sqrt(r_s / 2r)` (Keplerian speed, capped below
`c`) and uses `D` both to brighten/dim the disk and to push its color toward
blue (approaching) or red (receding) — the classic asymmetric-brightness look
of every serious black hole render since *Interstellar*.

## What's real vs. artistic license

| Real physics | Simplified / stylized |
|---|---|
| Schwarzschild horizon at `r_s`, ISCO at `3r_s` | Force-law lensing instead of exact geodesic RK4 |
| Central-force ⇒ planar, angular-momentum-conserving light paths | Lensing artificially windowed off past the disk (real gravity has no cutoff) |
| Relativistic Doppler beaming (`D³`, color shift) | No gravitational redshift/time-dilation dimming near the horizon |
| Einstein ring emergent from raymarching, not drawn | Background comets/marker stars are cinematic flourishes, not physical |

## More

Full uniform list, exact tuning constants, and the resume-from-scratch next
steps live in `SUMMARY.md`.
