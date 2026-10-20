# Changelog

## [Unreleased]

### Changed

- **`meshy-image-prompt`**: the model advice now comes from live tests of six more Text to Image models (2026-10-16, one sample per model): nano-banana, nano-banana-2, nano-banana-2-lite, gpt-image-2, gpt-image-2-5-flare and gpt-image-2-5-sunburst. `pose_mode: "a-pose"` gave a rig-ready A-pose on every model, and the GPT Image models followed the image formula most closely, so they now lead the list of models for final images. The 3-credit models drifted on the helmet prompt.
- **`meshy-image-prompt`**: prompts, the rig-ready pipeline and both meta prompts now default to `gpt-image-2-5-flare`, which works on every Meshy plan, instead of `nano-banana-pro`, which needs a Pro plan or above. The plan rules for `pose_mode` now match the docs: only a posed `nano-banana` request needs a Pro plan. The tested A-pose wording is included for anyone who prefers to write the pose in the prompt.
- **`meshy-edit-prompt`**: "How Edit works inside" is now "How Meshy reads the prompt" and explains what matters when you write a prompt: Meshy rewrites it based on the camera view and already keeps the rest of the model the same.
- Evidence files list the prompts, settings, run counts and results only; task ids and local file paths are gone.
- README: what you need before you start, where each prompt goes in the web app and the API, example requests, and a corrected Meshy Edit finding (naming the left shoulder put the raven on the left shoulder). Repo checks moved to `CONTRIBUTING.md`.

## [0.1.0] - 2026-10-14

First release: four prompt-writing skills built from live A/B tests (Text to 3D, Text to Image, texturing and rigging on 2026-10-07, Meshy Edit on 2026-10-14).

### Added

- **`meshy-text-to-3d-prompt`**: the shape-only formula for the Text to 3D preview prompt; what to leave out (colors, materials, damage, pose, face count, scenes, thin parts, long lists) and the setting that replaces each; why rig-ready characters belong on the image route; remeshing at generation to stay under the 300,000-face rigging limit; symptoms and fixes; evidence; code and plain meta prompts; `check_prompt.py`.
- **`meshy-image-prompt`**: the Text to Image formula for images that convert cleanly to 3D; `pose_mode` versus pose words (equally clean, never both, never conflicting); `aspect_ratio`, `remove_background`, `generate_multi_view` and model costs; single image versus multi-view; the 44-credit rig-ready pipeline with rig measurements; evidence; meta prompts; `check_prompt.py`.
- **`meshy-texture-prompt`**: the style-then-part-by-part formula for texture and retexture prompts; specific hues over color families (saddle-brown 4 of 4, brown drifted grey); damage words that shift generic colors; ambiguous part names; no shape or lighting words; `enable_pbr`; symptoms and fixes; evidence; meta prompts; `check_prompt.py`.
- **`meshy-edit-prompt`**: the Meshy Edit formula (verb, part with its side, new part, details, what remains); the untextured-model requirement; what the lasso controls versus the prompt; why "keep everything else unchanged" is redundant; the 20-run results; `check_prompt.py`.
- `scripts/check_prompt.py` (one copy per skill): a prompt checker for length, material and color words, pose conflicts, missing framing phrases, generic hues, shape words in texture prompts and unsided parts in Edit prompts.
- Repo checks (`scripts/validate_skills.py`) and trigger evals (`evals/trigger/`), described in `CONTRIBUTING.md`.
