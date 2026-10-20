# A 3D reconstruction of a wild *Portia labiata* hunt

Interactive viewer for a 3D reconstruction of a hunt in the wild by the
white-moustached jumping spider, *Portia labiata*, on the orb-weaving spider
*Argiope versicolor*.

**View it: https://maciverlab.github.io/portia-wild-hunt/**

The model is about 39 MB, so it can take a little while to load. It needs a
browser with WebGL, which every current desktop browser has.

## Using the viewer

- Drag to orbit, scroll to zoom, right-drag to pan.
- The buttons at the top right jump to set views of the hunt.
- **Toggle objects** shows or hides the prey's visual volumes, the foliage,
  *Portia*'s trajectory and the phases of the hunt.
- The timeline plays the hunt at 1×, 10× or 50×. **◀ Step** and **Step ▶**
  (or the ← and → keys) move one video frame (1/30 s) and pause.
- **Prey's Visual Range Highlight** sets how strongly the foliage visible to
  the prey is coloured.

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

## Views and starting state

The view buttons in the top bar come from `VIEWS` in `docs/index.html`; the
first is where the page opens. To add one, go to the moment and angle you want
and press **V**. The view, including the timeline position, is copied to the
clipboard, ready to paste into `VIEWS`. A view with a `time` moves the
timeline there and pauses.

Groups listed in `START_HIDDEN` (the visual volumes, by default) start
switched off.

## Previewing locally

Put `Portia_web.glb` in `docs/` and serve the folder over
http (opening `index.html` directly won't load the model):

    python3 -m http.server -d docs
