# Blobit Game Workflow

Use this workflow when the user wants a playable browser game and Blobit should be the source of generated runtime assets.

## Role Split

- Blobit owns generated assets: images, 3D models, world assets, placement metadata, scene validation, and optional publishing.
- The game repo owns gameplay: project scaffold, TypeScript code, Babylon.js scene logic, input, camera, physics, UI, game state, and browser verification.
- Use Babylon.js + TypeScript + Vite as the default stack when no existing browser game stack is present.
- Do not use non-Blobit asset generators or external asset-generation scripts. Procedural code is acceptable for abstract gameplay geometry, collision helpers, debug visuals, particles, and UI.
- Published manifest mode is the default runtime asset delivery path for browser games: `blobit publish create` -> `manifest_url` -> `@blobit/sdk`.
- Local asset downloads are explicit opt-in only for bundled, offline, static, or local-file game builds.

## State Files

Keep long-running game work resumable through files in the game repo:

- `PLAN.md` - risk tasks, main build scope, and verification criteria
- `STRUCTURE.md` - runtime architecture, entrypoints, module ownership, asset paths, and verification commands
- `ASSETS.md` - Blobit project ids, source image asset ids, generated asset ids, job ids, local paths or manifest URLs, intended in-game sizes, and usage notes
- `MEMORY.md` - discoveries, workarounds, failed attempts, and decisions that should survive context loss

## Review-Gated Mode

Use this as the default flow for high-polish, ship-ready, or subjective visual game requests.

1. Ask 3-5 concise questions about game style, core mechanic, target quality, asset scope, and collision/physics architecture.
2. Generate or define the visual target and stop for approval.
3. Write `PLAN.md` with risk tasks, architecture choices, and verification criteria, then stop for approval.
4. Generate or assign Blobit assets in `ASSETS.md`, inspect them, validate the scene, publish when runtime loading is needed, then stop for approval.
5. Build the first playable version.
6. Capture screenshot or video proof from the running browser game and ask for feedback.
7. Continue polish passes until the user approves final quality.

Do not ask for a dollar budget or report spend. The current Blobit CLI does not return asset-generation spend. Ask for asset scope, asset count, asset priority, or quality target instead.

## Architecture Decision

Before writing `PLAN.md`, ask or record how collision and physics should work:

- Custom arcade collision: best for deterministic combat, pickups, triggers, lane hazards, and simple arena bounds.
- Babylon/Havok physics: best for physical props, debris, rigid bodies, ragdoll-like behavior, and simulation-heavy spectacle.
- Hybrid: default recommendation for polished action games; keep core combat custom and use physics for props and environmental reactions.

Record the decision in `PLAN.md` and `MEMORY.md` so later agents do not silently choose a different approach.

## Pipeline

1. Read the game brief and existing project shape.
2. For high-polish game requests, follow review-gated mode; otherwise write or update `PLAN.md` directly.
3. Record the collision/physics architecture choice before implementation.
4. Isolate risky features before the main build: imported GLB animation, procedural generation, custom shaders, dynamic navigation, complex camera controls, runtime geometry, and physics-heavy behavior.
5. Scaffold or reuse the Babylon/Vite project. Preserve existing dependency versions unless the task is a migration.
6. Plan assets in `ASSETS.md` before generating them. Each runtime asset needs an intended in-game size and priority.
7. Generate, inspect, and record assets with Blobit.
8. For browser runtime assets, publish the Blobit scene and load it through `@blobit/sdk` unless the user explicitly chose local asset mode.
9. Implement playable slices. Keep the dev server running and use hot reload as the inner loop.
10. Run `npm run check` and `npm run build` before final proof.
11. Verify visible work from browser screenshots or video, not only from a clean build.

## Blobit Asset Flow

Prefer reviewed source images before expensive 3D generation when visual fidelity matters.

Prompt to image:

```bash
blobit --json assets generate-image "stylized sci-fi crate, game prop, clean studio lighting" --project-id <project_id>
```

Image to model:

```bash
blobit --json jobs model-from-image <image_asset_id> --project-id <project_id> --name sci_fi_crate --wait
blobit --json assets inspect <model_asset_id>
```

Local image to model:

```bash
blobit --json jobs model-from-file ./concept.png --project-id <project_id> --name hero_ship --wait
blobit --json assets inspect <model_asset_id>
```

Prompt directly to model when review is less important:

```bash
blobit --json jobs model-from-prompt "low-poly asteroid mining drone, readable silhouette" --project-id <project_id> --name mining_drone --wait
blobit --json assets inspect <model_asset_id>
```

World asset:

```bash
blobit --json jobs world-from-prompt "compact alien canyon arena with paths, cover, and a central objective" --name canyon_arena --wait
blobit --json worlds collider <world_asset_id>
```

## Local Asset Mode

Use this mode only when the user explicitly asks for bundled, offline, static, or local-file assets.

1. Download finalized model assets into `src/assets/models/` or another runtime asset folder.
2. Import them through Vite URLs.
3. Load them with Babylon's glTF loader.

```bash
blobit --json assets download <model_asset_id> --out src/assets/models
```

```ts
import "@babylonjs/loaders/glTF";
import { SceneLoader } from "../app/babylon";
import crateUrl from "../assets/models/sci_fi_crate.glb?url";

await SceneLoader.ImportMeshAsync("", "", crateUrl, scene);
```

## Published Manifest Mode

Use this mode by default when a browser game should consume Blobit-generated runtime assets.

1. Add generated assets or worlds to the Blobit scene.
2. Use semantic scene commands such as `drop-to-floor`, `arrange`, `avoid-overlap`, and `frame-camera`.
3. Run `blobit --json scene analyze <project_id>` and fix warnings.
4. Run `blobit --json scene validate <project_id>`.
5. Publish with allowed origins for the game dev server and deployed host.

```bash
blobit --json publish create <project_id> --allowed-origin http://127.0.0.1:5173
```

In the game:

```ts
import { loadBlobitManifest } from "@blobit/sdk";

const publish = await loadBlobitManifest({ manifestUrl });
const entities = publish.listEntities();
```

Use `publish.getEntityAsset(entity)` and `publish.getAssetUrl(asset.asset_id)` to resolve runtime files.

## Babylon Build Loop

- Keep `npm run dev` running at `http://127.0.0.1:5173`.
- Put asset URL imports in `src/game/assets.ts`.
- Keep scene creation behind `createScene(app): Promise<Scene>`.
- Use one high-level update loop that delegates to gameplay objects.
- Use DOM HUD for normal browser UI unless UI must exist inside the 3D world.
- Use the repo's existing browser capture tool. If none exists in a fresh scaffold, add a small Playwright/Chrome capture script before claiming visual verification.
- Add deterministic camera motion or scripted gameplay for final proof so the result can be verified without manual input.

## Verification Standard

For each visible slice:

```bash
npm run check
npm run build
node scripts/capture.mjs still screenshots/<task>/still.png  # or the repo's equivalent browser-capture command
```

For final proof:

```bash
node scripts/capture.mjs video screenshots/result/<N> 15
ffmpeg -y -i screenshots/result/<N>/video.webm -c:v libx264 -pix_fmt yuv420p -preset medium -crf 22 -movflags +faststart screenshots/result/<N>/video.mp4
```

The final browser media should prove the actual task: movement responds to input or scripted playback, collisions and camera framing work, imported assets are visible at the intended scale, UI is readable, and there are no obvious missing textures, clipping, overlaps, or browser console errors.

## Handoff

Report:

- Blobit `project_id`
- generated image/model/world asset ids
- job ids for generated assets
- local runtime paths or `manifest_url`
- relevant scene operations and validation state
- browser proof path and whether it was inspected
- unresolved limits such as missing Chrome/WebGL2, failed asset generation, or accepted validation warnings
