# Research 15: The simplest v0 stack for Iron Foundry (Three.js)

Researched 2026-11-24. Each version was read from the npm registry or GitHub releases on that date. Each claim cites its source and the date that source carries.

## Recommendation

**Use Vite 8.3.8, three 0.187.0 with `WebGLRenderer`, and TypeScript. Write the kinematic AABB controller by hand, with no physics library. Hand-write the follow camera. Test with Vitest 5.1.1 on the pure game-logic modules plus one Playwright 1.65.1 smoke test. Deploy with the official three-action GitHub Pages workflow, using `working-directory: games/iron-foundry` and a relative `base: './'`.**

| Piece | Choice | Exact version (published) |
|---|---|---|
| Bundler / dev server | Vite | `vite@8.3.8` (2026-11-11) |
| Renderer | three.js, `WebGLRenderer` (not WebGPU for v0) | `three@0.187.0` / r187 (2026-10-30), `@types/three@0.187.0` (2026-10-30) |
| Language | TypeScript. Vite strips the types, and `tsc --noEmit` checks them | `typescript@7.0.2` (2026-07-08); fall back to `6.0.3` if TS 7 trips anything |
| Physics / controller | **Hand-rolled** kinematic controller: swept AABB, resolved one axis at a time | none |
| Camera | Hand-rolled damped follow camera, about 30 lines | none |
| Unit tests | Vitest, `environment: 'node'` | `vitest@5.1.1` (2026-11-18) |
| Smoke test | Playwright, Chromium only, against `vite preview` | `@playwright/test@1.65.1` (2026-11-13) |
| Deploy | `actions/checkout@v7`, `setup-node@v7`, `configure-pages@v6`, `upload-pages-artifact@v5`, `deploy-pages@v5` | v7.0.1 (2026-07-20), v7.1.0 (2026-10-08), v6.0.0 (2026-03-25), v5.0.0 (2026-04-10), v5.0.1 (2026-09-01) |
| Node | 24 LTS (`.nvmrc` → `24`) | Vite needs `^20.19 \|\| >=22.12`, Vitest needs `^22.12 \|\| ^24 \|\| >=26` |

### Why the hand-rolled controller beats Rapier and cannon-es here

- **The level is boxes.** Catwalks, conveyor belts, furnace blocks, gear pickups and lava pools are all axis-aligned boxes. v0 has no rolling barrels, ragdolls or stacks of crates, so nothing needs rigid-body dynamics.
- **Rapier's character controller would not save much work.** The Rapier docs (fetched 2026-11-24) say the KCC "does not apply gravity... it is up to you to emulate gravity", so you still hand-write gravity, jumping, coyote time and conveyor push. What Rapier adds is autostep, slope limits and snap-to-ground, and a box level needs none of those. It also brings a WASM module that must be initialised asynchronously before the first frame and in every test, plus a package with an unpacked size of about 15 MB (`@dimforge/rapier3d-compat@0.21.0`, published 2026-09-25).
- **cannon-es is effectively unmaintained.** Its last release was `0.20.0` on 2022-08-12, and its last push to the default branch was 2024-01-06 (`pmndrs/cannon-es`, read via `gh` on 2026-11-24). It is a dynamics engine with no character controller, so it would be the most work of the three.
- **A hand-rolled controller is the best migration seam.** It is pure TypeScript with no WASM, no async init and no renderer. Vitest can step it deterministically at a fixed `dt`, and it ports unchanged to whatever engine comes next. A 9-year-old can also read it: "move, then push out of the wall".
- **When to switch to Rapier:** add `@dimforge/rapier3d-compat` behind the same `physics` interface once a level needs dynamic objects, slopes or rotated colliders. The interface below is designed so that swap stays inside one module.

## The renderer-independence seam

```
games/iron-foundry/
  src/game/            PURE: no `three`, no DOM, no Date/Math.random
    types.ts           Vec3, Box, InputFrame, GameState, GameEvent
    level.ts           level 1 as DATA: boxes {min,max,kind:'solid'|'conveyor'|'lava'|'checkpoint'|'machine'}, gear spawns
    physics.ts         moveAndCollide(pos, vel, halfExtents, boxes, dt) → {pos, vel, grounded, groundKind}
    player.ts          gravity, jump (+coyote time, jump buffer), conveyor push
    rules.ts           collect gears, lava → respawn at last checkpoint, 10 gears + touch machine → won
    step.ts            step(state, input, dt): GameEvent[]   ← the one entry point
  src/input/keyboard.ts  DOM keys → InputFrame {moveX, moveZ, jump, cameraYaw}
  src/render/            the ONLY place that imports `three`
    scene.ts           builds meshes FROM level.ts data; syncs them from GameState each frame
    axel.ts            Axel from primitives (Box/Capsule/Cylinder geometry)
    followCamera.ts
  src/main.ts            fixed-timestep loop: accumulator, step() at 1/60 s, render() once per frame
```

Rules that keep the seam honest:

1. **Game logic sees only data.** `step()` takes plain state plus an `InputFrame` and returns `GameEvent`s (`gearCollected`, `respawned`, `won`). The renderer reads `GameState` and reacts to the events with sparkles and sounds. Nothing flows from render back into game, apart from the camera yaw already folded into `InputFrame` so that "W" means away from the camera.
2. **The level exists once, as data.** The renderer builds meshes from `level.ts`, and physics collides against the same array. The next engine reads the same file.
3. **Use a fixed timestep.** Run a 1/60 s step inside an accumulator, and clamp frame `dt` to about 0.1 s so a backgrounded tab does not tunnel Axel through a catwalk. Keep max speed × step smaller than the thinnest box. This also makes the logic deterministic, which is what lets the tests run with zero flakes.
4. **Enforce it with a test, not a convention.** One Vitest spec reads every file under `src/game/` and fails if it imports `three` or touches `window`/`document`. That costs no extra lint configuration and runs alongside the other tests.

**Physics core (one axis at a time):** apply `vel.x*dt`, push out of any overlapping solid on X, then repeat for Z and then Y. If the Y push-out came from below, set `grounded = true`, `vel.y = 0` and record `groundKind`. If the ground is a conveyor, add the belt's push vector to the position. Lava, checkpoints, gears and the machine are overlap tests only and never push the player.

**Follow camera:** the desired position is `player + rotateY(offset(0, 4, 8), yaw)`. Move toward it each frame with `cam.position.lerp(desired, 1 - Math.exp(-6 * dt))`, then call `lookAt(player + (0, 1.2, 0))`, which is frame-rate independent. Arrow keys or the mouse change `yaw`. Do not use `OrbitControls`. Occlusion handling (the camera clipping into walls) can wait until after v0.

## Test setup that makes "zero skipped or failing" realistic

**Vitest (the bulk of the tests; fast and deterministic):** use `environment: 'node'`, with tests in `specs/` or `src/**/*.test.ts`.
- **physics:** landing on a catwalk sets `grounded`; Axel cannot walk through a wall; one-axis resolution slides along walls; a large `dt` does not tunnel.
- **player:** a jump reaches the expected apex; coyote time allows a jump for N ms after leaving a ledge; a conveyor moves Axel when he is idle.
- **rules:** touching a gear collects it once; lava respawns Axel at the last checkpoint and keeps his gears; the machine does nothing at 9 gears and sets `won` at 10.
- **Golden-path replay:** a scripted list of `InputFrame`s, stepped at 1/60 s from spawn, collects all 10 gears and wins. This is the integration test. It breaks whenever someone edits the level so that it can no longer be completed, which is exactly the regression a kid's level edits cause.
- **Purity guard:** the import-scan test from rule 4.

**Playwright (exactly one or two smoke tests, Chromium only):**
- `webServer` runs `vite build && vite preview`, and the test loads `/?debug`. In debug mode `main.ts` exposes `window.__ironFoundry = { state }`.
- Assert three things: the canvas exists, there are no console errors, and after `keyboard.down('KeyW')` for about 500 ms, `expect.poll(() => state.player.pos.z)` has changed. Assert on game state, never on pixels or screenshots, because headless GPU output varies.
- **Required launch args for WebGL in headless CI:** `--use-angle=swiftshader --enable-unsafe-swiftshader`. Chromium removed the automatic SwiftShader WebGL fallback. The deprecation began in Chrome 130, and from Chrome 139 the fallback is opt-in, so without the flag WebGL context creation fails (Chromium `docs/gpu/swiftshader.md` and the blink-dev "Intent to Remove: SwiftShader Fallback" thread, both 2025). This is the most likely cause of a first red run.
- Set `retries: 0` so a flaky test is fixed rather than masked, and use `expect.poll` rather than `waitForTimeout`.

## GitHub Pages deploy from a subfolder

- **Branch publishing cannot serve `games/iron-foundry/`.** It can only use `/` or `/docs` (GitHub Docs, "Configuring a publishing source", fetched 2026-11-24). Use **Settings → Pages → Source: GitHub Actions**. As of 2026-11-24 Pages is not yet enabled on `stoudtio/co-op-mode` (the `GET /repos/stoudtio/co-op-mode/pages` endpoint returns 404), and the repo holds only `CONTEXT.md` and `LICENSE`.
- **The workflow** follows the Vite static-deploy guide (vite.dev/guide/static-deploy, fetched 2026-11-24). Its template uses checkout v7, setup-node v7, configure-pages v6, upload-pages-artifact v5 and deploy-pages v5, with `permissions: {contents: read, pages: write, id-token: write}`, a `concurrency: pages` group and `environment: github-pages`. Add:
  - `defaults.run.working-directory: games/iron-foundry`, plus `cache-dependency-path: games/iron-foundry/package-lock.json` on setup-node.
  - A step that copies `games/iron-foundry/dist` into `_site/games/iron-foundry/` before `upload-pages-artifact` with `path: _site`. A repo has **one** Pages site and each deploy replaces it whole, so the artifact must be assembled at the site level from the start. A second game then becomes one more copy line rather than an overwrite.
  - Tests in a separate `pull_request` job (`npm ci && npm test && npx playwright install --with-deps chromium && npx playwright test`), required for auto-merge. The deploy job runs only on push to `main`.
- **`base`:** Vite's guide says a project site needs `base: '/<REPO>/'`. Here that would be `'/co-op-mode/games/iron-foundry/'`. **`base: './'` (relative) is simpler and survives moves**, because a single-page game with no router needs no absolute paths. The game is then served at `https://stoudtio.github.io/co-op-mode/games/iron-foundry/`.

## Evidence (all dated)

| Claim | Source | Date |
|---|---|---|
| three latest is 0.187.0 (r187): removed code deprecated in r177; WebGL and WebGPU fixes | npm registry `three`; `gh` latestRelease `mrdoob/three.js` r187 | published 2026-10-30, read 2026-11-24 |
| three exports `three/addons/*` and `three/webgpu` | npm `three@0.187.0` `exports` field | 2026-11-24 |
| Vite latest stable 8.3.8 (8.4.0-beta.1 exists); bundles with Rolldown; Node `^20.19 \|\| >=22.12` | npm `vite`; GitHub release vitejs/vite v8.3.8 | 2026-11-11 |
| Vitest 5.1.1; 5.0 announced 2026-09-03; Node `^22.12 \|\| ^24 \|\| >=26` | npm `vitest`; vitest.dev/blog | 2026-11-18 / 2026-09-03 |
| Playwright 1.65.1 | npm `@playwright/test`; GitHub release microsoft/playwright v1.65.1 | 2026-11-13 |
| TypeScript stable 7.0.2 (6.0.3 is the previous line) | npm `typescript` | 2026-07-08 |
| Rapier JS latest `@dimforge/rapier3d-compat@0.21.0`, about 15 MB unpacked | npm registry | 2026-09-25 |
| The Rapier KCC does not apply gravity; it offers autostep, snap-to-ground and slope limits | rapier.rs/docs/user_guides/javascript/character_controller | fetched 2026-11-24 |
| cannon-es last release 0.20.0; last push 2024-01-06 | npm `cannon-es`; `gh api graphql` pmndrs/cannon-es | 2022-08-12 / 2024-01-06 |
| The Pages actions versions listed above | `gh` latestRelease for each action repo | see table |
| The Vite GitHub Pages template (actions, permissions, `base`) | vite.dev/guide/static-deploy | fetched 2026-11-24 |
| Branch publishing is limited to `/` or `/docs` | docs.github.com, "Configuring a publishing source for your GitHub Pages site" | fetched 2026-11-24 |
| Headless Chromium WebGL needs `--enable-unsafe-swiftshader` | chromium.googlesource.com `docs/gpu/swiftshader.md`; groups.google.com/a/chromium.org/g/blink-dev/c/yhFguWS_3pM | 2025 (deprecation began in Chrome 130, opt-in from Chrome 139) |

## Open risks

1. **SwiftShader may be removed entirely on Linux.** Chrome said it would experiment with removing SwiftShader on macOS and Linux. If the Playwright-bundled Chromium loses it, the smoke test needs a fallback. Option (a): use `channel: 'chromium'` with the flag. Option (b): run the smoke test against a `?headless` mode that skips `WebGLRenderer` and still asserts the game loop. Option (b) is cheap because the seam already makes the renderer optional. Check this on the first CI run.
2. **Tunnelling.** One-axis AABB resolution without sweeping tunnels if speed × step exceeds the thinnest box. Clamp `dt`, cap speeds and keep catwalks at least 0.3 units thick, and pin this with a unit test.
3. **TypeScript 7 is the new native compiler.** If an editor or tool chokes on it, pin `typescript@6.0.3`. Vite and Vitest strip types themselves, so this only affects `tsc --noEmit`.
4. **One Pages site per repo.** Any future workflow that deploys Pages separately will overwrite the game. Keep a single site-assembling workflow, and remember that the World's Ticks may also want Pages.
5. **Package manager:** the Vite template uses npm. The owner's other projects use Yarn. Pick one in the scaffold PR. This research does not change with the choice.
6. **Camera occlusion and wall clipping** are deferred. They become a visible problem in tight furnace corridors and may need a raycast against `level.ts` boxes, which can live in pure game logic.
7. **Assumption: the "migrate later" target reads data.** The seam assumes the next engine can consume `level.ts` and `step()`. Godot or Unity would mean a port of the logic, not a reuse, but the tests and the level data would still serve as its specification.
