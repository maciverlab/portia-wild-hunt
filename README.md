# Can a Tiny Brain Plan? Reconstructing a Predatory Detour in the Jumping Spider *Portia labiata*

Interactive viewer for a 3D reconstruction of a hunt in the wild by the
white-moustached jumping spider, *Portia labiata*, on the orb-weaving spider
*Argiope versicolor*.

**View it: https://maciverlab.github.io/portia-wild-hunt/**

The model is about 85 MB, with a 10 MB point cloud of the prey beside it, so
the first load takes a little while. After the download the geometry is
decompressed, which takes a few seconds more. It needs a browser with WebGL,
which every current desktop browser has.

## Using the viewer

- Drag to orbit, scroll to zoom, right-drag to pan.
- The buttons at the top left set the view:
  - **Overview** is the Fig 1 camera, looking at the web plane. The page opens
    here.
  - **Attack** jumps to one second before the strike and looks at *Portia*
    from just in front of the web, beside the prey.
  - **Follow Portia** keeps her in the middle of the view as she moves.
  - **From the prey** looks at her from the prey's eyes.
  - **Fig 3C** and **Fig 3D back** are the cameras from the Blender scene, at
    their own focal lengths and roll.

  Dragging or scrolling from a fixed view hands it back to the orbit camera.
- **Toggle objects** shows or hides the foliage, the web, *Portia*'s
  trajectory (coloured by phase) and the prey's visual volume. Switching the
  visual volume on also colours red the foliage the prey can see.
- The timeline is coloured by behavioural phase and turns red at the strike.
  It plays the hunt at 1×, 10×, 50× or 100×. **◀ Step** and **Step ▶** (or
  the ← and → keys) move one video frame (1/30 s) and pause; Shift+← and
  Shift+→ move 10 s. Space plays and pauses.

## Contents

- `docs/index.html`: the viewer (three.js r128, loaded from jsDelivr).
- `docs/draco/`: the Draco decoder, about 1.1 MB. Served from here rather than
  from a CDN so that a page loading a large model off its own origin cannot
  then fail because jsDelivr is slow or blocked.
- `.github/workflows/deploy.yml`: publishes `docs/` to GitHub Pages.

The model and the point cloud are not stored in git. They are attached to the
[latest release](https://github.com/maciverlab/portia-wild-hunt/releases/latest),
and the deploy workflow copies every `.glb` the release carries into the site
each time a release is published or a change to `docs/` is pushed to `main`.
The page loads them by name, so the assets must be called what `MODEL_URL`
and `POINTS_URL` at the top of `docs/index.html` say:

- `animation_final_foliage.glb`: the scene and the animation
- `argiope_points.glb`: the prey's point cloud

Code and data for the accompanying paper will be added here.

## Updating the model

Publish a new release with both files attached (tag names must start with
`v`). The site redeploys within a couple of minutes:

    gh release create v3.0 animation_final_foliage.glb argiope_points.glb --title v3.0 --notes "What changed"

Without the GitHub CLI, do the same from the repository's **Releases** page:
draft a new release, attach both files and publish it.

To replace a file in the current release instead, upload over it and
redeploy:

    gh release upload v3.0 animation_final_foliage.glb --clobber
    gh workflow run deploy.yml

If a filename changes, change `MODEL_URL` or `POINTS_URL` in
`docs/index.html` to match in the same push, or the site will deploy and then
fail to find its model.

## How the model is built

From the Blender scene, in order:

1. Export the animated scene to `.glb`.
2. Weld the spider mesh. The export writes three corners for every triangle
   (4.28 M vertices for 1.43 M triangles), and every one of them is skinned
   on the GPU each frame. The corners differ only in their normals, because
   the mesh is flat-shaded, so the normals are dropped and the duplicates
   merged, leaving 760 k vertices. Without normals glTF draws a mesh
   flat-shaded, so it looks the same.
3. `draco_glb.py` compresses the geometry, using the Draco encoder that ships
   inside Blender's own glTF add-on.
4. `slim_glb.py` keeps only the four clips that move anything
   (`Portia_merged_track`, `Portia_Walk`, `ArmatureAction.001`,
   `Spider_bodyAction`) and re-encodes the textures at the resolution they
   already have.
5. The foliage is exported on its own, with the prey's view baked into it,
   and spliced in with `merge_glb.py` in place of the old scan. The prey's
   view is reduced to a single byte per vertex (`_SEEN`, 255 where the prey
   can see it) before compression: two four-channel colour layers across
   5.5 M vertices are more than the browser's Draco decoder can hold, and it
   fails without reporting anything.

`argiope_points.glb` is the prey's point cloud, clipped to the Discoid swept
2 mm along its normal (`clip_points_to_discoid.py`). Blender's glTF exporter
drops loose vertices, so it is written separately as a points primitive.

Anything that opens the result without a Draco decoder will show an empty
scene, because `KHR_draco_mesh_compression` is listed in `extensionsRequired`.

## Views, timing and starting state

All of these are named constants in the script in `docs/index.html`:

- `OVERVIEW` and `FIGURES` are the fixed views, with each camera's position,
  roll (`up`) and field of view, taken from the Blender scene.
- `ATTACK_VIEW` sets the Attack view: how long before the strike, and how far
  in front of the web, to the right and up from the prey the camera sits.
- `PHASES` holds the behavioural phases and their timeline colours.
- `LAYERS` lists the rows in **Toggle objects** and which start on;
  `HIDE_ALWAYS` switches objects off and keeps them out of the panel.
- `FILE_PER_S`, `HANDOFF` and `ATTACK_RATE` map the exported clips onto hunt
  time. The scene was exported at 10 fps with its keys authored at 30, and the
  attack clip is authored against 60 fps.

## Previewing locally

Put both `.glb` files in `docs/` and serve the folder over http (opening
`index.html` directly won't load the model):

    python3 -m http.server -d docs
