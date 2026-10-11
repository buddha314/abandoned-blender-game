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

## SETUP

### Logic Nodes runtime

Max's movement and Intro's reset rules use Logic Nodes. Their generated Python
components require the `uplogic` package in **UPBGE's bundled Python**. A copy
installed in your system Python or user site packages is not enough for the
standalone player. If Max does not move, check the player console for
`ModuleNotFoundError: No module named 'uplogic'`.

In UPBGE, use **Logic Nodes → Install or Update Uplogic Module** with Online
Access enabled, then restart UPBGE and reopen `blend/levels/Intro.blend`.
Confirm the console has no `KX_PythonComponent[MaxMovement]` startup error.
The current project was checked with Uplogic 5.1.1 and UPBGE 5.4 alpha.

### Asset library import choices

Keep the character source in `blend/characters/Max.blend`. Configure an asset
library in **Edit → Preferences → File Paths → Asset Libraries** that includes
the character source directory, then select that library in the Asset Browser.
Save source changes before importing or reloading them in a level.

| Choice | Behavior | When to use it |
| --- | --- | --- |
| Append | Creates a local, editable copy in the level. Later source edits do not propagate. | Independent copies and quick experiments. |
| Append (Reuse Data) | Appends locally while reusing previously imported dependencies where possible. It does not create a live source link. | Repeated asset imports without unnecessary duplicate data. |
| Link | References data in the source `.blend`. Linked data is read-only; create a library override for level-specific transforms and supported edits. Source changes become available when reloaded. | A shared character maintained across levels. |
| Pack | Embeds supported external resources; packing linked libraries is a separate operation. It is not an Asset Browser import method and does not make linked objects editable. | Preparing files for transfer or distribution. |

For Max, prefer a **collection asset** containing both `MaxBody` and its child
`Max`, with movement logic attached to `MaxBody`. Link the collection and create
a library override for its hierarchy to position it in each level. Append is
also valid when automatic source updates are not needed. Verify the imported
hierarchy rather than assuming that dragging a single mesh asset includes its
physics parent and logic.

After replacing Max in `Intro.blend`:

1. Confirm both `MaxBody` and the visible `Max` are present in the level, with
   `Max` parented to `MaxBody`.
2. Move `MaxBody` to the spawn point. Keep the visible mesh separate from the
   collision capsule: `MaxBody` uses Character physics and capsule bounds;
   `Max` uses No Collision. The body is invisible and the mesh is visible.
3. Confirm `MaxMovement` is attached and enabled on `MaxBody`, and its Walk
   node targets the imported body.
4. Reassign **IntroRules → Get World Position → Object** to the new `MaxBody`.
5. Force Compile, save the level with a 3D Viewport in the active workspace,
   then check visibility, A/D movement, and falling below Z = −5 to respawn.

The character source now marks `MaxCollection`, containing `Max` and `MaxBody`,
as the asset. Use this collection when importing the complete character.

### Library overrides: selecting and moving Max in a level

A linked collection initially appears in Intro as a **collection instance**.
The instance references the source hierarchy, but its child objects are not
direct members of the level's scene. This is why `MaxBody` can exist in the
linked library yet be unavailable in a logic node's Object picker.

1. Select the `MaxCollection` instance in the Outliner.
2. Hover over the 3D Viewport and use **F3 → Make Library Override**.
3. Confirm the resulting collection exposes `MaxBody` and `Max` in the level,
   with `Max` still parented to `MaxBody`.
4. Select `MaxBody` in **IntroRules → Get World Position → Object**. Creating
   an override does not automatically reconnect a node after deleting its
   previous target.

Overrides preserve references to the source while allowing supported local
changes. However, automatically created hierarchy members can be **system
overrides**: Blender maintains them to preserve the hierarchy, and they remain
read-only. An object can therefore have an override and no transform locks
checked, yet still refuse to move.

To make the body editable, select **MaxBody itself** in the Outliner and use
**right-click → Library Override → Make → Selected**. This promotes that
member to a user-editable override. Move `MaxBody` with **G → X** in the
viewport; the visible `Max` follows through parenting. Moving the mesh alone
would separate its appearance from the physics body.

Maintain shared appearance and character logic in `blend/characters/Max.blend`;
use overrides for each level's placement and supported local adjustments.
Reload linked data after saving source changes. Avoid making everything local
unless an independent copy is intended, since that breaks the source link.
After replacing or overriding the hierarchy, verify node targets, Force
Compile, and save in Object Mode before testing.

## GOTCHAS

### Standalone startup crashes with a mesh saved in Edit Mode

Intro's standalone player crashed during its first rendered frame when Ground
was saved in Edit Mode. A debugger backtrace pointed to `BM_mesh_bm_to_me`,
called through `scene_graph_update_tagged` and `KX_Scene::UpdateDepsgraph`.
A copy saved in Object Mode ran for an eight-second test without reproducing
that crash; the test was stopped by a timeout, not a normal game exit.

**Workaround:** Exit Edit Mode with **Tab** in the 3D Viewport, save the level,
then use **Standalone Start**. After reopening a repaired file, avoid saving an
older open copy over it. This diagnosis and test were performed with Codex.

### Restart Game crashes when the saved workspace has no 3D Viewport

With the UPBGE 5.4 alpha build at commit `5aa7b7693e69`, Intro's fall-reset
logic triggered a segmentation fault when **Restart Game** reloaded the level.
The level's saved active workspace contained a Logic Nodes editor and a File
Browser, but no 3D Viewport. The crash occurred in
`LA_BlenderLauncher::LA_BlenderLauncher` while accessing the viewport region
(`ar->winx` / `ar->winy`). Startup with a viewport succeeded; a forced fall
reproduced the crash during reload.

**Workaround:** Keep a 3D Viewport in the active workspace and save the level
before playing. In `blend/levels/Intro.blend`, the File Browser panel was changed
to a 3D Viewport, the Logic Nodes editor was retained, and the compiled logic was
saved. A forced-fall test then reloaded the level and exited successfully.

Reopen the repaired file from disk if an older copy is still open; saving that
older copy could restore the problematic layout. This is a project workaround;
UPBGE still needs to handle a missing viewport safely. Diagnosis and validation
were performed with Codex.

### Bullet physics scene crashes on shutdown or game switch

In the local UPBGE 5.4 alpha build at `5aa7b7693e69`, any scene using Bullet
physics crashed when it exited or switched to another `.blend`. This affected
Intro's old fall reset and its **Escape → ReturnToLobby** game actuator. The
crash occurs in `CcdPhysicsEnvironment::~CcdPhysicsEnvironment()`: a game engine
source file compiled with single precision overwrites a Bullet member laid
out for double precision. This is an UPBGE build issue, not a Logic Nodes error.

The level now moves `MaxBody` back to its spawn position when Z drops below −5,
so falling no longer reloads the game. Returning to the lobby still requires
the engine fix. For an UPBGE source checkout matching this build, apply the
project patch and rebuild both the player and editor:

```sh
patch -p1 -d /path/to/upbge -i patches/upbge-bullet-precision.patch
cmake --build /path/to/build_upbge --target blenderplayer -j 4
cmake --build /path/to/build_upbge --target blender -j 4
```

Restart the editor after rebuilding. A forced fall returned Max to spawn, and
an Intro → lobby transition exited cleanly with the rebuilt player.
