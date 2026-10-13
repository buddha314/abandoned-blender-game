# Making the Desert materials feel natural

This guide describes how to reduce obvious texture repetition on the terrain in `blend/levels/Desert-1.blend`, while keeping the material setup practical for the game's UPBGE runtime.

## What is in the scene

- `Plane` uses **Rocky Desert Ground**.
- `Plane.001` uses **Desert Cliff Surface**.
- Both materials use tiled image maps for color, roughness, normal, and displacement. The ground plane is scaled about 9.5×, so repeated features can become conspicuous over its large area.
- The ground material's image paths point to `D:/Janhe/...`; the cliff material points into a BlenderKit data folder outside this project. Relink or pack the source images before moving the project to another workstation.

## Suggested workflow

Work on a copy of the scene or save a new version before changing the materials.

1. **Make the source materials portable.** Relink missing images in the Image Editor or pack them into the blend file with **File → External Data → Pack Resources**. For a shared project, storing texture files under the project and using relative paths makes dependencies easier to track.
2. **Fix the base mapping first.** Inspect the UV map and the Mapping node on each material. Set a sensible texel scale for the terrain; avoid stretching one small patch across the whole landscape. Keep the existing image maps together: color, roughness, and normal must use compatible coordinates and scale. Set color maps to sRGB and data maps (roughness, normal, displacement) to Non-Color.
3. **Add broad color variation.** Use a low-frequency Noise Texture driven by Object or world-space coordinates and a Color Ramp to make a restrained mask. Use it to mix small color shifts into the base color, or to blend the ground and cliff materials across broad irregular patches. Keep the transition soft and subdued; high-contrast noise tends to look like camouflage.
4. **Vary roughness separately.** Feed a subtle, broad mask into a mix of roughness values or multiply it gently with the existing roughness map. This keeps large areas from having identical highlights. Do not feed color data directly into the normal input.
5. **Use a deliberate blend mask.** For this scene, start with vertex paint or a painted black-and-white mask to choose where rocky outcrops and sandy ground belong. Let noise distort or soften that mask. Height and slope can help automate the first pass, but artist-painted placement usually gives better control over paths, ledges, and exposed rock.
6. **Keep fine detail and large variation at different scales.** The image textures provide small surface detail. Broad color and roughness changes should span many texture repeats. Add only a restrained fine bump layer if the surface needs it; excessive noise at every scale makes the terrain visually busy.
7. **Check the actual game renderer.** Test the result in UPBGE, at the normal camera distance and under game lighting. Shader nodes supported in Blender's offline render may not be supported the same way in UPBGE. If a procedural mix is too expensive or unsupported, bake the final color, roughness, and normal result to textures and use those in the game. Keep the bake resolution and memory budget appropriate for the target hardware.
8. **Add geometry and set dressing where needed.** Material variation cannot hide a completely flat silhouette. Gentle terrain shape changes, scattered rocks, and transitions around cliffs reinforce the material breakup.

## A good first pass for this level

Keep **Rocky Desert Ground** as the base. Add a broad, low-contrast noise color variation across `Plane`, then use a painted mask to bring in **Desert Cliff Surface** on exposed rock areas and slopes. Let low-frequency noise break up the painted boundary. Preview under the game's lighting, adjust until the texture tiles are less obvious without making the landscape mottled, and bake the result if the UPBGE runtime does not support the node setup reliably.

## YouTube tutorials

Ryan King Art is a good match for learning how material layers and procedural masks are built. These videos focus on related rocky and desert materials; use them to learn the node ideas, then simplify or bake the result for the game.

- [Procedural Rocky Ground Material (Blender Tutorial) — Ryan King Art](https://www.youtube.com/watch?v=luQv93hwQxQ): builds a rocky ground shader from procedural nodes.
- [Procedural Rocky Ground (Blender Tutorial) — Ryan King Art](https://www.youtube.com/watch?v=q30dmqDXzOo): an earlier rocky-ground walkthrough.
- [Procedural Desert Rock Materials — Ryan King Art](https://ryanking.artstation.com/projects/JvO6WZ): page linking to the two-part desert rock material tutorial.
- [Procedural Mars Rock & Sand Material — Ryan King Art](https://ryankingart.gumroad.com/p/procedural-mars-rock-sand-material-blender-tutorial): tutorial page for a rock-and-sand material workflow.
- [How to Tile a Texture Without Repetition — Blender Guru](https://www.youtube.com/watch?v=-VgtSL5ZpYc): covers tile rotation, color variation, mixing materials, and height variation for large surfaces.
- [Blender Texture Tiling Fix | No More Repetitive PBR Textures](https://www.youtube.com/watch?v=VKyWzj0mcbs): a node-based example using texture mixing, noise, and a Color Ramp to break up PBR repetition.

## Project constraints to keep in mind

- Blender and UPBGE keep separate profiles, but the material and its image dependencies live in the blend file or referenced files.
- Do not assume that a complex procedural material will run in the game just because it previews in Blender. Verify it in UPBGE; bake when needed.
- Avoid baking credentials, account data, or unrelated profile settings into project setup instructions.
