# abandoned-blender-game

UPBGE project scaffold for a multi-`.blend` workflow.

## Directory layout

- `blend/`
  - `levels/` scene/level `.blend` files
  - `characters/` character rigs/animations
  - `props/` reusable prop files
  - `shared/` linked libraries shared across blends
- `assets/`
  - `textures/`, `materials/`, `audio/`, `models/`
- `logic/`
  - `python/` gameplay Python scripts
  - `configs/` game/config data
- `build/` exported runtime builds

## Suggested workflow

1. Keep reusable data in `blend/shared/` and link/append into level files.
2. Keep gameplay scripts in `logic/python/` and attach them from UPBGE.
3. Export distributable builds into `build/`.
