# Godogen Patterns Used By Blobit CLI

This file records which Godogen game-production patterns are intentionally folded into the Blobit CLI skill. Use it for rationale when changing `game-workflow.md`; do not treat raw Godogen asset-generation files as Blobit instructions.

## Keep

- Visual target before coding: define an in-game screenshot target with camera, objects, environment, HUD, and expected quality.
- Review gates for high-polish games: questions, visual target approval, `PLAN.md` approval, Blobit asset approval, first playable proof, polish feedback.
- Risk-first planning: isolate imported GLBs, animation, physics-heavy behavior, procedural generation, custom shaders, dynamic navigation, complex cameras, and runtime geometry.
- Babylon/Vite scaffold contract: one canvas, DOM HUD, `createScene(app): Promise<Scene>`, stable dev server, hot reload, and capture script.
- Architecture stance: `GameWorld` owns broad rules, actors own Babylon nodes, input exposes semantic actions, UI stays separate from gameplay rules.
- Scene loop: one high-level update hook delegates to owned objects and deterministic presentation behavior proves final output without manual input.
- Browser proof: running game screenshots/video matter more than a clean TypeScript build.
- Runtime gotchas: import GLTF loaders explicitly, keep listener cleanup explicit, watch browser console errors, and treat WebGL/GPU state as part of validation.

## Replace With Blobit

- Asset planning should use Blobit project ids, source image ids, generated asset ids, job ids, scene entity ids, manifest URLs, intended in-game sizes, and priority.
- Asset generation must use Blobit CLI commands, not Godogen's old asset-generation scripts or vendors.
- Browser games should default to published manifest mode and load assets with `@blobit/sdk`.
- Local downloads are only for explicit bundled, offline, static, or local-file builds.
- Do not use dollar budgets or spend logs; the current Blobit CLI does not return asset-generation spend.

## Raw Source Provenance

The raw Godogen files copied into `source/godogen-selected/` are there for audit and future reference. They include assumptions that are intentionally not active in the Blobit skill.
