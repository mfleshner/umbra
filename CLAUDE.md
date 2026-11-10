# Umbra — handoff notes

2D side-view stealth game (Mark of the Ninja gameplay, Splinter Cell vibe) for phone browsers. Built for Matt to play himself on his Android phone (landscape), with a USB-C controller as the main input, touch as backup, and keyboard + mouse on PC. Lethal play. Short procedurally generated runs.

Everything lives in **one file: `index.html`** (HTML + CSS + JS, no build, no dependencies besides Google Fonts). Hosted on GitHub Pages from `main`.

## Keep it this way
- Single self-contained file. No frameworks, no bundler.
- Landscape only. Controller first, touch still fully playable, mouse aims manually.
- Original names only: no Sam Fisher / Third Echelon / Splinter Cell names, logos or the green trifocal look. Our operative has amber goggles.
- Keep sessions focused: one feature area per session to save usage.

## Code map (inside the `<script>`, in order)
| Section | What's there |
|---|---|
| constants & tuning | `T` tile size 32, rooms are `RWT`×`RHT` = 20×9 tiles. `TUNE` holds speeds, noise radii, shadow threshold, guard vision, cooldowns, ammo/HP. Tweak here first. |
| level generation | `generate(diff)` builds a grid of rooms (4×2 at facility 1, up to 6×4). DFS spanning tree + extra links. Horizontal links = door gaps, vertical links = 1-tile hatch (PLAT) + ladder. Room types: office, server, storage, catwalk, hall. Adds pipes, crates, decor, windows, lamps, objectives (`M.objs`), extraction (`M.ext`, farthest room by BFS), guards (`spawnGuard`). |
| tiles | `SOLID`, `PLAT` (one-way), `PIPE` (hangable, non-solid). Ladders are a separate layer `M.lad`. Crates are SOLID cells listed in `M.crates` (drawn differently). |
| audio | Tiny WebAudio synth: `sfx(name)`. No audio files. |
| input | Keyboard, mouse, touch (floating left joystick + DOM buttons + tap-world-to-shoot), Gamepad API (`readPad`, standard mapping). All merged into the `I` object each frame. `lastDevice` drives prompt labels and laser sight. |
| light & noise | `lightAt(x,y)` = sum of lamps/windows in the same room (no occlusion; rooms clip light). `noise(x,y,r,kind,src,tx)` spawns a visible ring and makes guards in range `hear()` it (radius ×0.55 across rooms). |
| guards | States: `patrol` → `suspicious` (meter ≥ 0.4, or heard something) → `alert` (meter ≥ 1, shoots). Vision = cone (`TUNE.guardHalfAngle`, `guardRange`) + tile LOS. Detection rate scales with player light, distance, crouch, hanging, sprint, and bodies found. Bodies seen in light (or close) raise `G.bodiesFound`, which speeds detection for the rest of the facility. Guards only walk horizontally (no ladders yet). |
| player | States: `normal`, `climb`, `hang` (pipes), `mantle` (auto ledge climb when pushing into a ledge while airborne or holding up), `takedown`. Act priority: takedown > objective > extract > drag body. Drop-takedown from a pipe onto a guard below. |
| shooting | Hitscan `rayShot`: head box = kill, body = 2 hits (first one alerts). Lamps are shootable (glass noise). Guard bullets are projectiles. |
| modes & screens | `G.mode`: title / play / pause / results / dead. Scoring in `completeFacility`. Run carries HP (+1 per clear) and ammo (+6 per clear). Best run saved in `localStorage` key `umbra.best`. |
| rendering | World → darkness overlay (half-res canvas, lamp/window cut-outs clipped to rooms) → glows, windows with rain, emissives (LEDs, screens) → guard cones, bodies, guards, player, laser, tracers, noise rings, markers → NV filter (amber `color` composite + scanlines) → vignette → HUD. Guards are only drawn when the player has LOS, or as amber outlines during sonar. |

## Controls
Controller: L-stick move (full push sprints), R-stick aim, RT/RB fire, A jump, B crouch, X act, Y night vision, LB sonar, D-up whistle, Start pause.
Keyboard: A/D move, W/S climb/grab/drop, Shift sprint, Space jump, C crouch, E act, mouse aim + click, Q NV, R sonar, F whistle, Esc pause.
Touch: left half = floating joystick, tap the world to shoot that spot, FIRE auto-aims nearest guard (else nearest lamp), JUMP, ACT, CROUCH, NV, SONAR, WHSTL.

## Roadmap (agreed for after v1)
- Thermal goggles (see guards through thin walls)
- Security cameras
- Gadgets: sticky cam, sticky shocker, noise maker, smoke (gadget wheel)
- Heavier enemy types (armored heavies that need headshots)
- Eliminate-target objective
- Sound pass, debug/tuning panel
- Possible: guards using ladders, breaker boxes that kill a room's lights, wall split-jump

## Testing tips
- Headless smoke test: Playwright + `/usr/bin/google-chrome`, viewport 800×360, click `[data-primary]`, mash keys, watch `pageerror`.
- Commit messages end with the co-author line Claude Code adds.
