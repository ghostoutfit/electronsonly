# electronsonly — Electrons Digital Model (v1)

Single-file sim. No build step — edit `index.html` directly.

To run locally: `python3 -m http.server 8099 --directory /Users/jkremer/Projects/electronsonly`, then open `localhost:8099/`.

## Files

```
index.html          — the entire sim
images/
  favicon.png / favicon.svg
  logo-placeholder.png
  Stopwatch.png
  Turtle.png
```

## Deployment

GitHub Pages from `main` branch root. Push to `origin` to deploy.
Remote: `https://github.com/ghostoutfit/electronsonly.git`

## CRITICAL: Tab naming

The UI button labels do NOT match `data-tab` values or `currentTab`:

| Button label | `data-tab` / `currentTab` |
|---|---|
| **Shells** | `'shells'` |
| **Trails** | `'orbit3d'` |
| *(hidden "iTunes Visualizer")* | `'trails'` |

`draw()` returns early for orbit3d: `if (currentTab === 'orbit3d') { drawOrbit3D(ctx); return; }` — the shells `_drawGrid` is never reached in orbit3d mode; the 3D grid lives inside `drawOrbit3D`.

## Key JS architecture

- **`idleParticleLoop()`** — main animation loop; any exception before it starts (at init) freezes the page
- **`syncTabButtons()`** — called at init; references `forceCtrlBox` (not `forceModeRow`)
- **`syncForceVizVisibility()`** — gates force panel and Force Values checkbox to shells tab; hides Force Values when `arrowMode === 0`
- **`drawForceVizShell()`** — force panel, shell mode; repulsion arrows always draw radially outward (`cosA, sinA`) regardless of inner-electron clamped positions
- **`getShellRadius(s)`** — shell 0 clamped to `Math.max(px, animParent.sphereR * 1.4)` for heavy atoms
- **`arrowMode`** — 0 = forces off, 2 = forces on; resets to 0 when switching to shells tab

## Rendering order in `draw()` (shells tab)

1. Clear canvas
2. Compute layout vars (`_W`, `_H`, `_tbH`, `_camX`, `_centerY`)
3. `_drawGrid(...)` — screen space, **behind** all atom content
4. `ctx.save()` + world transform (`translate` → `scale(viewZoom)` → `translate`)
5. Draw cloud / particles / electrons
6. `ctx.restore()`
7. Atomic radius line + label — screen space, above atom
8. "Not to scale" notice — screen space, bottom center

## Grid system

### Shells grid — `_drawGrid(ctx, nucSX, nucSY, W, H, tbH, zoom)`
- Screen-space Cartesian or radial grid (toggled by `gridType`)
- Spacing: `20 * shellBaseR * zoom` px per major cell — scales with zoom
- TRON neon style; color follows `_themeGridRgb`

### Trails 3D grid — inside `drawOrbit3D`
- Floor (XZ) plane + 3 axes, orthographic projection via `proj3D(nx, ny, nz, r3d)`
- `proj3D`: `screenX = cx + (x·cosAZ + z·sinAZ)`, `screenY = cy - (y·cosEL - rz·sinEL)`
- Scale: `_gScale = shellBaseR * orb3dRadScale * viewZoom`
- Extent: `Math.ceil(Math.hypot(W, H) / 2 / _gScale / _step) * _step` — always fills screen
- Step: `20` pm; all lines batched into a single `stroke()` call
- Pulse: `0.75 + 0.25 * Math.sin(t / 1100)`

### Theme color
- `_themeGridRgb = [r, g, b]` global, updated in `applyTheme(key)`

## Force visualization

- `_fvDefaultPosition()` — vertically centered between toolbar and bottom, right-justified
- Forces default off on load (`arrowMode = 0`)

### `fmtNNLabel(nN, peer)`
- No decimals normally; **1 decimal when attraction and repulsion round to the same integer** (e.g. K, Na, Ca) — pass the other force value as `peer`

## Shell colors

```js
SHELL_COLORS = ['#ff4488', '#00ddff', '#ff8800', '#88ff00']
// index:          0 (pink)   1 (blue)   2 (orange)  3 (lime)
```
