## Why

Intro has a door marker, a level-start marker, and a connected navigation mesh, but no entrance sequence. A project-specific walkthrough lets the learner build and test that sequence while understanding how Max's linked character logic and Intro's level logic interact.

## What Changes

- Add a step-by-step exercise for moving Max from `MaxSpawn` at the door to `LevelStart` when Intro loads.
- Explain how to obtain navmesh waypoints, move `MaxBody` with Character physics, and hand control to the existing A/D movement component.
- Give checks for arrival, keyboard handoff, canyon respawn, and Escape.
- Leave game scripts, Logic Nodes, and animation assets for the learner to implement.

## Capabilities

### New Capabilities

- `max-door-entrance-instructions`: Project-specific instructions and acceptance checks for Max's Intro entrance.

### Modified Capabilities

None.

## Impact

Documentation under this OpenSpec change only. The exercise refers to `blend/characters/Max.blend` and `blend/levels/Intro.blend`, UPBGE's navmesh and Character APIs, and the existing `MaxMovement` component. It does not change runtime behavior.
