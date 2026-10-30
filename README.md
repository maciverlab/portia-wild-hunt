# Can a Tiny Brain Plan? Reconstructing a Predatory Detour in the Jumping Spider *Portia labiata*

Interactive viewer for a 3D reconstruction of a hunt in the wild by the
white-moustached jumping spider, *Portia labiata*, on the orb-weaving spider
*Argiope versicolor*.

**View it: https://maciverlab.github.io/portia-wild-hunt/**

The model is about 94 MB, so the first load takes a little while; it is cached
afterwards. It needs a browser with WebGL, which every current desktop browser
has.

## Using the viewer

- Drag to orbit, scroll to zoom, right-drag to pan.
- The buttons at the top left jump to set views of the hunt, including the
  **Fig 3C** and **Fig 3D** cameras from the Blender scene, at their own focal
  lengths.
- **Toggle objects** shows or hides the prey's visual volumes, the foliage,
  *Portia*'s trajectory phases and the prey's-eye colouring of her route. The
  page opens on the foliage and the trajectory; everything else is off.
- The timeline plays the hunt at 1×, 10× or 50×. **◀ Step** and **Step ▶**
  (or the ← and → keys) move one video frame (1/30 s) and pause.
- **Prey's Visual Range Highlight** sets how strongly the foliage visible to
  the prey is coloured. The plant carries the photograph and the seen /
  not-seen coding as two colour layers on one mesh, and the slider fades
  between them.

## Contents

- `docs/index.html`: the viewer (three.js r128, loaded from jsDelivr).
- `docs/draco/`: the Draco decoder, about 1.1 MB. Served from here rather than
  from a CDN so that a page loading a 94 MB model off its own origin cannot
  then fail because jsDelivr is slow or blocked.
- `.github/workflows/deploy.yml`: publishes `docs/` to GitHub Pages.

The model is not stored in git. It is attached to the
[latest release](https://github.com/maciverlab/portia-wild-hunt/releases/latest),
and the deploy workflow copies it into the site each time a release is
published or a change to `docs/` is pushed to `main`. The workflow takes
whatever `.glb` the release carries, so the model can be renamed without
touching it — but `MODEL_URL` at the top of `docs/index.html` has to match the
asset's filename.

Code and data for the accompanying paper will be added here.

## Updating the model

Publish a new release with the model attached (tag names must start with `v`).
The site redeploys within a couple of minutes:

    gh release create v2.0 portia_blended_small.glb --title v2.0 --notes "What changed"

To replace the model in the current release instead, upload over it and
redeploy:

    gh release upload v2.0 portia_blended_small.glb --clobber
    gh workflow run deploy.yml

If the filename changes, change `MODEL_URL` in `docs/index.html` to match in
the same push, or the site will deploy and then fail to find its model.

## How the model is built

From the Blender scene, in order:

1. Export to `.glb`.
2. `draco_glb.py` compresses the geometry, using the Draco encoder that ships
   inside Blender's own glTF add-on. 481 MB → 117 MB, worst position error
   1 part in 33,000 of a mesh's own extent.
3. `slim_glb.py` drops the animation clips that drive only marker empties —
   seventeen of twenty here — and re-encodes the textures at the resolution
   they already have. 117 MB → 94 MB.
4. `merge_glb.py` is for changing one object afterwards without paying
   for the other 93 MB. Re-export just that object from Blender, run
   `draco_glb.py` on it alone, then splice it in: a node whose name
   matches one already in the file takes over its mesh and transform,
   and everything else — the animations, the skin, the textures, the
   other meshes' compressed geometry — is copied through byte for byte.
   Run Draco on the donor, never on the merged file: `draco_glb.py`
   reads each primitive's accessors to report what it saved, and an
   already-compressed primitive has no `bufferView` to read.
   It refuses a donor carrying a skin or an animation, since neither
   can be renumbered into an existing rig without judgement.

Anything that opens the result without a Draco decoder will show an empty
scene, because `KHR_draco_mesh_compression` is listed in `extensionsRequired`.

## Views and starting state

The view buttons in the top bar come from `VIEWS` in `docs/index.html`; the
first is where the page opens. To add one, go to the moment and angle you want
and press **V**. The view, including the timeline position and the field of
view if it is not the default, is copied to the clipboard ready to paste into
`VIEWS`. A view with a `time` moves the timeline there and pauses.

`START_HIDDEN` lists what is switched off when the page opens, by group id or
by object name. It currently holds the visual volumes, the prey's-eye route
colouring and the trajectory markers, leaving the foliage and *Portia*'s
trajectory phases on. `HIDE_ALWAYS` switches something off *and* keeps it out
of the panel, so it cannot be turned back on.

## Previewing locally

Put the `.glb` in `docs/` and serve the folder over http (opening
`index.html` directly won't load the model):

    python3 -m http.server -d docs
