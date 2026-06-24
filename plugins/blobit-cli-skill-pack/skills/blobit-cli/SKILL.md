---
name: blobit-cli
description: Use when working with Blobit to create projects, generate or reuse 3D assets, compose scenes, publish them, load published scenes in web apps with @blobit/sdk, or build browser games that use Blobit as the asset backend.
---

# Blobit CLI

Use `blobit` as the asset backend, and as the scene/publish backend when the game should consume a hosted Blobit scene.

- Use `blobit` to:
  - create projects
  - upload images
  - generate or reuse 3D assets
  - inspect placement metadata
  - compose and validate scenes
  - publish manifests
- For browser games, use normal code tools to plan, scaffold, implement, run, and visually verify the playable game.
- In game projects, Blobit owns runtime asset generation. Gameplay code, UI, input, camera, physics, and game state stay in the game codebase.
- For browser games, published manifest mode is the default asset delivery path: use the `manifest_url` returned by `blobit publish create` and load it with `@blobit/sdk`.

## When to use

Use this skill when the user asks to:
- create or manage Blobit projects and assets
- generate 3D models from images
- lay out a scene using semantic placement commands
- publish a Blobit scene for browser consumption
- integrate published Blobit content into a web game with the SDK
- create or update a browser game where Blobit should generate or supply the runtime assets

## Rules

- Prefer `blobit --json` when command output must be parsed.
- Prefer semantic scene commands over hand-editing raw transforms.
- For new assets, prefer Blobit image generation, image upload, or local-image upload plus model/world generation unless the user explicitly provides an existing model to reuse.
- For model and world generation commands, prefer the stored default provider unless the task needs a one-off override via `--provider`.
- Prefer `blobit config set-provider <provider>` over telling the user to edit Blobit config files by hand.
- Do not use the legacy provider alias `--endpoint-type` in new commands; use `--provider`.
- Inspect placement metadata before arranging or framing when an asset's origin, footprint, or floor contact is unclear.
- Run `scene analyze` before and after major layout changes.
- Run `scene validate` before publish.
- When a task changes visible scene geometry and a local preview or viewer is available, capture a screenshot and inspect for obvious visual issues.
- For browser games, avoid downloading assets and load the published Blobit manifest at runtime.
- Use `blobit assets download` only when the user explicitly asks for bundled, offline, static, or local-file assets.
- Do not ask for or report a dollar asset budget. The current Blobit CLI does not return asset-generation spend; ask for asset scope, asset count, and priority instead.
- For high-polish games, use review-gated mode by default: ask brief questions, generate a visual target, write `PLAN.md`, generate or assign Blobit assets, build a first playable, capture proof, then iterate on feedback.
- Before planning game implementation, ask or record the architecture choice for collision/physics: custom arcade collision, Babylon/Havok physics, or a hybrid.
- Do not use non-Blobit asset-generation vendors or scripts when the task asks for Blobit-made assets.
- Treat Blobit as the asset-and-scene backend only; use the repo's normal code tools to build game logic and rendering.
- For published runtime content, use the `manifest_url` returned from `blobit publish create` and load it with `@blobit/sdk`.

## What It Produces

- a Blobit `project_id`
- uploaded source image asset ids or reused source asset ids
- generated model asset ids and, when applicable, completed job ids
- a persisted scene with placed models, camera state, and semantic layout operations applied
- `scene analyze` and `scene validate` results
- a published `manifest_url` plus related publish metadata such as `publish_id` and `viewer_url`
- the runtime integration point in the web app where `@blobit/sdk` loads the published manifest
- for game tasks, `PLAN.md`, `STRUCTURE.md`, `ASSETS.md`, and visual proof from the running browser game when the project workflow supports them

## Pipeline

1. Inspect or create the Blobit project.
2. Upload a source image and generate a model, or reuse an explicit existing asset.
3. Inspect the generated or reused asset, especially placement metadata, before scene layout decisions.
4. Add the model to the scene.
5. Apply semantic placement operations such as `drop-to-floor`, `place-near`, `arrange`, `align`, `avoid-overlap`, `distribute`, `snap`, and `frame-camera`.
6. Run `scene analyze`, review overlaps, bounds, and warnings, and adjust the scene if needed.
7. Run `scene validate` before publish and treat validation warnings as a blocking issue unless the user explicitly accepts them.
8. Publish and capture the `manifest_url` and any related viewer or publish metadata.
9. In the game client, load that manifest through `@blobit/sdk`.
10. Build or update the actual game logic and rendering with the normal project stack.

For playable browser games, read `references/game-workflow.md` before planning or coding. It defines a planning and implementation loop adapted for Blobit-only assets.
For the Godogen-derived rationale behind that game workflow, read `references/godogen-patterns.md` only when updating the skill or reconciling workflow decisions.

## Runtime Validation

- After major layout or camera changes, run the local game, preview, or Blobit viewer when available.
- Capture a screenshot when rendered geometry matters and inspect for floating models, clipping, wrong scale, bad orientation, bad camera framing, and overlaps that semantic checks may miss.
- Treat screenshot review as a supplement to `scene analyze` and `scene validate`, not a replacement.
- If no runtime preview is available, rely on CLI validation and explicitly note that visual validation was skipped.

## Scene Notes

- Use `drop-to-floor` when an object should visibly rest on the stage or ground plane.
- Use `frame-camera` after adding, removing, or rearranging models, or when the current camera may no longer frame the subject.
- Inspect asset placement metadata before `arrange` or `place-near` when the object's origin, support surface, or footprint is unclear.
- Prefer semantic scene operations over direct transform edits; use direct transforms only for precise final adjustments or explicit user requests.
- Run `scene analyze` after bulk arrangement or overlap-resolution passes to confirm the scene matches expectations.

## Expected Outputs / Handoff

- the key ids and URLs: `project_id`, relevant asset ids, job ids, `publish_id`, `manifest_url`, and `viewer_url` when applicable
- a short summary of the scene operations applied and the resulting validation state
- the exact runtime integration point where the app loads the published manifest through `@blobit/sdk`
- a note about whether runtime screenshot validation was performed and what it found
- any unresolved layout, validation, or runtime limitations that still need follow-up

## Core commands

- `blobit --json projects list`
- `blobit --json projects create --name ...`
- `blobit --json assets generate-image ...`
- `blobit --json assets upload-image ...`
- `blobit --json jobs model-from-image ... --wait`
- `blobit --json jobs model-from-prompt ... --wait`
- `blobit --json jobs world-from-prompt ... --wait`
- `blobit --json assets inspect <asset_id>`
- `blobit --json assets download <asset_id> --out ...`
- `blobit --json scene add-model <project_id> <asset_id>`
- `blobit --json scene add-world <project_id> <asset_id>`
- `blobit --json scene analyze <project_id>`
- `blobit --json scene place-near ...`
- `blobit --json scene arrange ...`
- `blobit --json scene validate <project_id>`
- `blobit --json publish create <project_id> --allowed-origin ...`

Read `references/workflows.md` for end-to-end command recipes.
Read `references/game-workflow.md` for browser game planning, implementation, asset manifest, and visual proof guidance.
