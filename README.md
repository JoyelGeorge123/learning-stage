# Madathil Marketing — 3D Solar Website Template

A single-file, production-styled marketing website where the **entire page is a live 3D scene**.
A WebGL world (solar farm, terrain, house, wind turbine, animated sky) is rendered behind the content,
and **GSAP ScrollTrigger** flies the camera through it as the visitor scrolls.

Built as a design template — no build step, no bundler, no external 3D models or textures.

---

## Quick start

Double-click `design.html`, or serve it locally:

```bash
cd d
python -m http.server 8000
# then open http://localhost:8000/design.html
```

**Requirements:** a browser with WebGL (Chrome, Edge, Firefox, Safari 15+) and an internet
connection on first load, because Three.js and GSAP are fetched from CDN.

---

## Tech stack

| Layer | Choice | Loaded from |
| --- | --- | --- |
| 3D engine | Three.js `r160` (ES module + import map) | `unpkg.com` |
| Animation | GSAP `3.12.5` + ScrollTrigger | `cdnjs.cloudflare.com` |
| Styling | Hand-written CSS, CSS custom properties | inline in `design.html` |
| 3D assets | 100% procedural (geometry, canvas textures, shaders) | — |

---

## The 3D world

Everything is generated in code — there are no `.glb`, `.obj` or image files.

- **Terrain** — 520×520 plane, 150×150 segments, displaced by fractal value noise, vertex-coloured
  by height and slope, flat-shaded for a stylised low-poly look.
- **Sky** — custom `ShaderMaterial` on a back-side sphere. Blends horizon → zenith, adds a sun disc
  glow and a wide warm halo around the real sun vector.
- **Sun** — emissive ball plus an additive glow sprite, driving a shadow-casting `DirectionalLight`
  (2048px shadow map) and the scene's fog colour.
- **Solar array** — 42 panels (6 rows × 7 columns) on tracker posts. Each panel uses a
  canvas-drawn texture: cell grid, busbars and a metal frame. Every pivot rotates to face the
  current sun position.
- **Property** — house with pyramid roof and roof-mounted panels, battery container with a pulsing
  emissive strip, 3-blade wind turbine, power poles with catenary wires.
- **Landscape** — 34 trees, scattered rocks, 7 drifting clouds, 420 floating dust particles.
- **Energy flow** — 9 additive orbs travelling a `CatmullRomCurve3` from the array to the house,
  fading in and out along the path, plus 3 expanding ground rings.

---

## GSAP behaviour

- **Camera timeline** — one scroll-scrubbed timeline (`scrub: 0.8`) spans the whole 550vh page and
  tweens two objects: `cam` (camera position) and `look` (look-at target). Eight keyframes take the
  camera from a wide dawn shot, down between the panel rows, across to the house and battery, up to
  the turbine, and finally to an aerial pull-back.
- **Section copy** — each section runs its own scrubbed timeline: text rises and fades in, then out.
- **Ambient loops** — turbine rotation, cloud drift, ring pulses, battery glow breathing, dust
  rotation, loading-bar sweep.
- **Reveal** — `IntersectionObserver`-style ScrollTriggers fire the stat counters once.

The render loop is registered on `gsap.ticker`, so 3D rendering and GSAP animations stay in sync.

---

## Live solar intelligence

The HUD, the sun's position, the sky, the shadows and every panel angle are driven by the
**visitor's real local time** (`new Date()`), not hard-coded values:

| Value | Source |
| --- | --- |
| Sun elevation / azimuth | Sine curve across the 06:00–18:00 window |
| Panel tracker angle | Derived from elevation |
| Array output (kW) | `sin(elevation) × 11.4` |
| Sun position & light colour | Recomputed vector, drives the shader and shadows |
| Fog colour | Interpolates warm day → cool dusk |

Because it tracks real time, the page genuinely looks different in the morning and at night.

---

## Page structure

| Section | Scroll height | Content |
| --- | --- | --- |
| Hero | 100vh | Headline, CTA, trust markers |
| Stats | 60vh | 4 animated counters |
| Technology | 110vh | 6 cards with hover spotlight |
| Process | 100vh | 4 steps |
| Savings model | 100vh | Interactive calculator |
| CTA | 80vh | Conversion block |

Plus a fixed header, a telemetry HUD, a scroll-progress bar, and a footer.

Section heights are declared per section via `data-h="100"` on each `<section class="sec">`. The
camera timeline reads those values to distribute its keyframes, so **changing a `data-h` value
re-times the camera automatically**.

---

## Savings calculator

Three sliders — monthly bill, usable roof area, monthly consumption — drive:

```
panels = clamp(4, 120, floor(area / 2.4))
system = panels × 0.55 kW
generation = panels × 0.29 kWh/day × 30
saving = min(generation, consumption) × (bill / consumption)
```

Currency is formatted in INR with `toLocaleString('en-IN')`.

---

## Customising

**Colours** — every colour is a CSS variable at the top of the file:

```css
:root{ --gold:#ffc24b; --gold-2:#ff8a3d; --blue:#4da6ff; --cyan:#3ee6d6; }
```

**Camera path** — edit the `keys` array in the camera timeline:

```js
const keys = [
  { p:[0, 17, 60], l:[0, 6, -2] },   // p = camera position, l = look-at target
  ...
];
```

**World layout** — `ROWS`, `COLS`, `SX`, `SZ`, `Z0` control the array; `groundY(x, z)` is the single
source of truth for terrain height and is reused to seat every object on the ground.

**Scene lighting** — `sunLight`, the hemisphere light and the ambient light sit together in one
block right after the sun.

---

## Degradation & performance

- If WebGL or the CDN is unavailable, the loader reports **"3D engine unavailable"** instead of
  leaving a blank page.
- `prefers-reduced-motion: reduce` skips the scroll-scrubbed camera, reveals all copy immediately and
  leaves the camera at the hero position.
- Pixel ratio is capped at 2; shadows and antialiasing are the only expensive features, and
  `prefers-reduced-motion` users get a static scene.

---

## Git

The folder is an initialised repository (branch `master`) with `design.html` untracked.
