## Context

This is a learning exercise for the saved Intro scene, not a game change. In `blend/levels/Intro.blend`, `MaxSpawn` sits by the door, `LevelStart` sits near Max's current play position, and `Walkable` is a connected Navigation Mesh. `MaxBody` has Character physics and contains the existing `MaxMovement` component. The visible `Max` mesh is its child. `MaxMovement` is authored in `blend/characters/Max.blend`; its `ExecutionCondition` already names the existing Boolean game property `NL__MaxMovement`.

The linked `MaxBody` in Intro is an override. Blender does not allow the learner to add or edit its source game properties there. The exercise must use runtime property changes for Intro-specific state and keep reusable character settings in `Max.blend`.

## Goals / Non-Goals

**Goals:**

- Teach the learner to make Max appear at the door on Intro load, follow `Walkable` to `LevelStart`, and then accept A/D input.
- Explain which object owns route planning, physical movement, and visible animation.
- Give observable checks for successful arrival and for failure cases.

**Non-Goals:**

- This OpenSpec change does not add an entrance component, alter Logic Nodes, or create animation assets.
- The exercise does not change canyon respawn or Escape's return to the lobby.

## Decisions

### Use the saved level markers and navigation mesh

The guide uses `MaxSpawn` as the entrance point, `LevelStart` as the destination, and `Walkable` to calculate route points. They are level-specific objects. A straight-line movement would be simpler, but it would not teach the navmesh workflow or respect later changes to the walkable route.

### Move the Character body and gate the existing controls

The guide has the learner move `MaxBody` through its Character controller's `walkDirection` toward successive navmesh waypoints. `MaxMovement` also writes `walkDirection`; the level entrance therefore sets `MaxBody["NL__MaxMovement"]` false at start and true at arrival. The source component's `ExecutionCondition` checks that property every update. Moving the visible child alone would leave its collider behind. Driving `MaxBody` with a transform Action would compete with Character physics.

### Keep the scene sequence in Intro

The learner creates an Intro-only Python component on `LevelController`. Its `start()` obtains the four named scene objects, places `MaxBody` at `MaxSpawn`, clears previous Character velocity, computes a route with `Walkable.findPath(start, goal)`, and records the current waypoint. Its `update()` compares the body's horizontal position with the next waypoint, advances through nearby points, and sets horizontal `walkDirection` toward the current target. At the final point it stops the Character and re-enables `NL__MaxMovement`. This is an algorithm outline for the learner, not implementation code.

Use a small arrival radius so physics does not oscillate around a point. If `findPath` returns no points, log the failure and return Max to `LevelStart` with normal controls enabled so the level remains playable. Do not treat a failed route as a successful entrance.

### Animate the visible child separately

`Max.blend` currently has no armature or Action. The learner can first prove the route and control handoff, then create a simple looping walk cycle for the visible `Max` in the source character file. Play it while the body follows the route and stop it on arrival. A body bob is an initial visual exercise; moving limbs requires a rig or separate mesh parts. Rotation toward travel direction can be added after the route works.

## Guided exercise

1. Open `blend/characters/Max.blend`. Verify that `MaxBody` has `NL__MaxMovement = True` and that `MaxMovement` uses `NL__MaxMovement` as its Execution Condition. Save, then reopen Intro so the linked component refreshes. Do not add the property on Intro's linked override.
2. Open `blend/levels/Intro.blend`. Confirm `MaxSpawn` is at the door, `LevelStart` is at the desired play position, and `Walkable` has Navigation Mesh physics. Confirm `MaxBody` is a Character and `Max` is its visible child.
3. In Intro, create a Python component on the local `LevelController`. Give it a level-specific name such as `IntroEntry`. In `start()`, set `NL__MaxMovement` false before the route begins; place and reset `MaxBody` at `MaxSpawn`; request the route from `Walkable` to `LevelStart`.
4. In `update()`, ignore vertical position when deciding whether a waypoint has been reached. Apply a normalized horizontal direction through the Character controller, using a speed near the existing A/D speed of `0.1` as a starting point. Advance to the next waypoint inside the arrival radius. On the last waypoint, stop the Character and set `NL__MaxMovement` true.
5. Test the physical entrance before adding a walk cycle. Once it succeeds, create the visual Action in `Max.blend` and arrange for the Intro component to start and stop it with the route. Keep animation on the visible `Max`, while `MaxBody` owns position and collision.
6. Launch Intro from the lobby and confirm Max appears at the door, follows the route, ignores A/D during entry, and responds to A/D after arrival. Then force a canyon fall and press Escape to confirm existing recovery and lobby behavior still work.

## Risks / Trade-offs

- **Linked override edit error** → Edit source component settings in `Max.blend`; only change the game property on the runtime `MaxBody` in Intro.
- **Two movement writers** → Ensure the Execution Condition is set and the level component switches `NL__MaxMovement` at start and arrival.
- **No route or unreachable marker** → Inspect marker placement and `Walkable`, log the failure, and restore playability at `LevelStart`.
- **Sliding appearance** → Add and play a visual walk Action after the route works; navigation alone does not animate Max's mesh.
- **Startup order** → Both components may evaluate on the first frame. Verify that Max does not visibly respond to held A/D input before the entrance component gates it.

## Migration Plan

This change is documentation-only, so there is no migration or rollback. The learner can save work in the two `.blend` files after completing each exercise stage and restore an earlier saved copy if an experiment fails.

## References

- [UPBGE Navigation Mesh API](https://upbge.org/docs/latest/api/bge.types.KX_NavMeshObject.html)
- [UPBGE Character controller API](https://upbge.org/docs/latest/api/bge.types.KX_CharacterWrapper.html)
- [UPBGE Python components](https://upbge.org/docs/latest/manual/manual/python_components/introduction.html)
