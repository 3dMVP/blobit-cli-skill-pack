# Blobit CLI Skill Pack

Installable Codex skills for using the Blobit CLI and SDK as the asset backend for browser games.

## Contents

- `skills/blobit-cli/` - primary skill to ship with the Blobit CLI.
- `skills/babylon-help/` - optional support skill for Babylon.js API, loader, Vite, WebGL, and browser-capture lookup.
- `source/godogen-selected/` - selected raw Godogen source files used as provenance while carving the Blobit game workflow. These are not installed directly.

## Install Locally

```bash
mkdir -p ~/.codex/skills
rm -rf ~/.codex/skills/blobit-cli ~/.codex/skills/babylon-help
ln -s /Users/choudhry347/Desktop/Projects/blobit-cli-skill-pack/skills/blobit-cli ~/.codex/skills/blobit-cli
ln -s /Users/choudhry347/Desktop/Projects/blobit-cli-skill-pack/skills/babylon-help ~/.codex/skills/babylon-help
```

## Release Notes

The active Blobit skill keeps Blobit responsible for generated runtime assets, scene validation, publishing, and SDK manifest loading. Game logic, input, camera, UI, physics, and verification remain in the game repo.

The selected Godogen influence is limited to game-production workflow: visual target, review gates, risk planning, Babylon scaffold, architecture ownership, scene update patterns, browser proof, and runtime gotchas.

Do not ship Godogen's old `asset-gen`, `asset-planner`, or `rembg` flow with Blobit. Blobit replaces that path.
