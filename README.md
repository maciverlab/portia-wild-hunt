# A 3D reconstruction of a wild *Portia labiata* hunt

Interactive viewer for a 3D reconstruction of a hunt in the wild by the
white-moustached jumping spider, *Portia labiata*, on the orb-weaving spider
*Argiope versicolor*.

**View it: https://maciverlab.github.io/portia-wild-hunt/**

The model is about 39 MB, so it can take a little while to load. It needs a
browser with WebGL, which every current desktop browser has.

## Using the viewer

- Drag to orbit, scroll to zoom, right-drag to pan.
- **Toggle objects** shows or hides the prey's visual volumes, the foliage,
  *Portia*'s trajectory and the phases of the hunt.
- The timeline plays the hunt at 1×, 10× or 50×.
- ***Argiope versicolor* seen brightness** sets how strongly the foliage
  visible to the prey is coloured.

## Contents

- `docs/index.html`: the viewer (three.js r128, loaded from jsDelivr).
- `.github/workflows/deploy.yml`: publishes `docs/` to GitHub Pages.

The model, `Portia_web.glb`, is not stored in git. It is attached to the
[latest release](https://github.com/maciverlab/portia-wild-hunt/releases/latest),
and the deploy workflow copies it into the site each time a release is
published or a change to `docs/` is pushed to `main`.

Code and data for the accompanying paper will be added here.

## Updating the model

Publish a new release with the model attached (tag names must start with `v`).
The site redeploys within a couple of minutes:

    gh release create v1.1 Portia_web.glb --title v1.1 --notes "What changed"

To replace the model in the current release instead, upload over it and
redeploy:

    gh release upload v1.0 Portia_web.glb --clobber
    gh workflow run deploy.yml

## Changing the starting view

Orbit to the view you want and press **V**. The view is copied to the
clipboard; paste it as `INITIAL_VIEW` in `docs/index.html`. Groups listed in
`START_HIDDEN` there (the visual volumes, by default) start switched off.

## Previewing locally

Put `Portia_web.glb` in `docs/` and serve the folder over
http (opening `index.html` directly won't load the model):

    python3 -m http.server -d docs
