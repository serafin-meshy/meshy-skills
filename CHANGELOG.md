# Changelog

## [Unreleased]

### Changed

- **`meshy-image-prompt`**: the model advice now comes from live tests of six more Text to Image models (2026-10-16, one sample per model): nano-banana, nano-banana-2, nano-banana-2-lite, gpt-image-2, gpt-image-2-5-flare and gpt-image-2-5-sunburst. `pose_mode: "a-pose"` gave a rig-ready A-pose on every model, and the GPT Image models followed the image formula most closely, so they now lead the list of models for final images. The 3-credit models drifted on the helmet prompt.
- **`meshy-image-prompt`**: prompts, the rig-ready pipeline and both meta prompts now default to `gpt-image-2-5-flare`, which works on every Meshy plan, instead of `nano-banana-pro`, which needs a Pro plan or above. The plan rules for `pose_mode` now match the docs: only a posed `nano-banana` request needs a Pro plan. The tested A-pose wording is included for anyone who prefers to write the pose in the prompt.
- **`meshy-edit-prompt`**: "How Edit works inside" is now "How Meshy reads the prompt" and explains what matters when you write a prompt: Meshy rewrites it based on the camera view and already keeps the rest of the model the same.
- Evidence files list the prompts, settings, run counts and results only; task ids and local file paths are gone.
- README: what you need before you start, where each prompt goes in the web app and the API, example requests, and a corrected Meshy Edit finding (naming the left shoulder put the raven on the left shoulder). Repo checks moved to `CONTRIBUTING.md`.
- **`meshy-text-to-3d-prompt`**: new "Props" section, based on six props run live on 2026-10-20 (a lantern, a chair, a longsword, a potted plant, a cottage and a cartoon van; one sample per arm). Before, the only prop tested was the knight's helmet. The formula held for each object's type and main parts, but the previews dropped a style that sets the whole look ("cartoon" gave a realistic van in 3 of 3), proportions, part counts and the size of a thin handle. Props whose look depends on these words are now sent to the image route; before, all props stayed on Text to 3D. Also new: leave out ground-level details on buildings (they came with an added ring of terrain), call holes "openings", and set `should_remesh` for real-time use (raw prop previews ran 47,000 to 2.2 million faces). Symptom table, meta prompts and evidence updated.
- **`meshy-image-prompt`**: new "Props" section from the same test. Image to 3D followed all six formula images, including spindles, a handle, a roof rack and leaves, so the thin-parts rule now targets dense clusters only. `aspect_ratio: "4:3"` for wide objects (at 1:1 a van was cut off). Clear glass came out as an open frame and frosted glass as solid panes. A thatched cottage came out at 7.7 million faces in a 303 MB GLB; with `should_remesh` and 30,000 faces it was 6.6 MB, at the same credits. Remesh is now recommended for game, web and AR use. The meta prompts pick the main object of a scene and leave out the ground around a building.
- `check_prompt.py`: preview prompts now flag ground-level details, cartoon or toy style words and "window panels"; image prompts flag clear glass.

## [0.1.0] - 2026-10-14

First release: four prompt-writing skills built from live A/B tests (Text to 3D, Text to Image, texturing and rigging on 2026-10-07, Meshy Edit on 2026-10-14).

### Added

- **`meshy-text-to-3d-prompt`**: the shape-only formula for the Text to 3D preview prompt; what to leave out (colors, materials, damage, pose, face count, scenes, thin parts, long lists) and the setting that replaces each; why rig-ready characters belong on the image route; remeshing at generation to stay under the 300,000-face rigging limit; symptoms and fixes; evidence; code and plain meta prompts; `check_prompt.py`.
- **`meshy-image-prompt`**: the Text to Image formula for images that convert cleanly to 3D; `pose_mode` versus pose words (equally clean, never both, never conflicting); `aspect_ratio`, `remove_background`, `generate_multi_view` and model costs; single image versus multi-view; the 44-credit rig-ready pipeline with rig measurements; evidence; meta prompts; `check_prompt.py`.
- **`meshy-texture-prompt`**: the style-then-part-by-part formula for texture and retexture prompts; specific hues over color families (saddle-brown 4 of 4, brown drifted grey); damage words that shift generic colors; ambiguous part names; no shape or lighting words; `enable_pbr`; symptoms and fixes; evidence; meta prompts; `check_prompt.py`.
- **`meshy-edit-prompt`**: the Meshy Edit formula (verb, part with its side, new part, details, what remains); the untextured-model requirement; what the lasso controls versus the prompt; why "keep everything else unchanged" is redundant; the 20-run results; `check_prompt.py`.
- `scripts/check_prompt.py` (one copy per skill): a prompt checker for length, material and color words, pose conflicts, missing framing phrases, generic hues, shape words in texture prompts and unsided parts in Edit prompts.
- Repo checks (`scripts/validate_skills.py`) and trigger evals (`evals/trigger/`), described in `CONTRIBUTING.md`.
