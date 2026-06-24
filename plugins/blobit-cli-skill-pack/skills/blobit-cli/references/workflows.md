# Blobit CLI Workflows

## Create a project

```bash
blobit --json projects create --name "Forest Demo"
```

## Generate a model from an image

```bash
blobit --json assets upload-image ./tree.png --project-id <project_id>
blobit --json jobs model-from-image <image_id> --project-id <project_id> --wait
```

Use `assets inspect` after generation if you need placement metadata:

```bash
blobit --json assets inspect <asset_id>
```

## Generate a model from a prompt

Use this when a separate source-image review pass is not important:

```bash
blobit --json jobs model-from-prompt "low-poly collectible crystal cluster, clean silhouette" --project-id <project_id> --name crystal_cluster --wait
blobit --json assets inspect <asset_id>
```

Use image first when the asset is visually important:

```bash
blobit --json assets generate-image "stylized hover bike, side view, studio lighting" --project-id <project_id>
blobit --json jobs model-from-image <image_id> --project-id <project_id> --name hover_bike --wait
blobit --json assets inspect <asset_id>
```

## Generate a world asset

```bash
blobit --json jobs world-from-prompt "small floating island arena with ramps, cover, and a central platform" --name island_arena --wait
blobit --json worlds collider <world_asset_id>
```

Use the collider asset for physics, navigation, raycasts, or gameplay spatial checks when the game runtime supports it.

## Add a model and place it semantically

```bash
blobit --json scene add-model <project_id> <asset_id>
blobit --json scene drop-to-floor <project_id> --model-id <model_id>
blobit --json scene place-near <project_id> <model_id> --target <target_id> --relation right_of --distance 1.5
blobit --json scene analyze <project_id>
```

## Arrange multiple models

Grid:

```bash
blobit --json scene arrange <project_id> --model-id a --model-id b --model-id c --model-id d --pattern grid --columns 2 --spacing 2
```

Circle:

```bash
blobit --json scene arrange <project_id> --model-id a --model-id b --model-id c --pattern circle --radius 4
```

Line:

```bash
blobit --json scene arrange <project_id> --model-id a --model-id b --model-id c --pattern line --axis x --spacing 3
```

## Validate before publish

```bash
blobit --json scene validate <project_id>
```

If warnings are returned, fix layout first, then rerun validate.

## Publish for a web app

```bash
blobit --json publish create <project_id> --allowed-origin http://localhost:3000
```

Use the returned `manifest_url` in the web app or game client. Blobit handles the asset and scene backend; game code should be built with the repo's normal coding tools.

## Use the Blobit SDK in the game client

Install the local SDK package:

```bash
npm link @blobit/sdk
```

Minimal usage:

```ts
import { loadBlobitManifest } from "@blobit/sdk";

const publish = await loadBlobitManifest({ manifestUrl });
const entities = publish.listEntities();
const asset = publish.getEntityAsset(entities[0]);
const assetUrl = publish.getAssetUrl(asset.asset_id);
```

The game client should use the returned `manifest_url` and the SDK for published runtime loading.

## Use Blobit as the game asset backend

For browser games, read `game-workflow.md`. In short:

1. Plan assets in `ASSETS.md` with intended in-game sizes.
2. Generate images, models, and worlds only through Blobit.
3. Inspect each generated asset before using it.
4. Use published manifest mode by default: publish the scene, keep the `manifest_url`, and load runtime assets with `@blobit/sdk`.
5. Use `blobit assets download` only when the user explicitly asks for bundled, offline, static, or local-file assets.
6. Do not ask for a dollar asset budget or report spend; ask for asset scope, count, priority, or quality target.
7. For high-polish games, use review gates: visual target approval, `PLAN.md` approval, Blobit asset approval, first playable proof, then polish passes.
8. Record the collision/physics choice before implementation: custom arcade collision, Babylon/Havok physics, or hybrid.
9. Verify the running game in the browser with screenshot or video capture.

## End-to-end game workflow

1. Create or inspect the Blobit project.
2. Generate the environment or prop assets you need.
3. Place and validate the scene with semantic `blobit scene ...` commands.
4. Publish the scene and capture `manifest_url`.
5. In the game repo, install `@blobit/sdk`.
6. Load the manifest in the client and render the scene assets with the game engine.
7. Build gameplay logic separately from Blobit.
