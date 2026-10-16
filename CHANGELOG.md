# Changelog

## [0.1.0] - 2026-10-14

First release: four prompt-writing skills distilled from the "Meshy Prompt Formulas" tests (live A/B runs on 2026-10-07, Meshy Edit runs on 2026-10-14).

### Added

- **`meshy-text-to-3d-prompt`** — the shape-only formula for the Text to 3D preview `prompt`; what to leave out (colors, materials, damage, pose, face count, scenes, thin parts, long lists) and the parameter that replaces each; the finding that Text to 3D gives no standard A-pose and rig-ready characters belong on the image route; remesh-at-generation to stay under the 300,000-face rigging limit; a symptom table; evidence file; code and plain meta prompts; `check_prompt.py`.
- **`meshy-image-prompt`** — the Text to Image formula for images that convert cleanly to 3D; `pose_mode` versus pose words (equally clean, never both, never conflicting); `aspect_ratio`, `remove_background`, `generate_multi_view` and model costs; the single-versus-multi-view finding; the 44-credit rig-ready pipeline with rig measurements; evidence; meta prompts; `check_prompt.py`.
- **`meshy-texture-prompt`** — the style-then-part-by-part formula for `texture_prompt` and `text_style_prompt`; specific hues over color families (saddle-brown 4 of 4, brown drifted grey); damage words that shift generic colors; ambiguous part names; no shape or lighting words; `enable_pbr`; a symptom table; evidence; meta prompts; `check_prompt.py`.
- **`meshy-edit-prompt`** — the Meshy Edit formula (verb, part with its side, new part, details, what remains); the untextured-model precondition; what the lasso controls versus the prompt; the internal prompt rewrite and why "keep everything else unchanged" is redundant; the 20-run results; `check_prompt.py`.
- `scripts/check_prompt.py` (one copy per skill): word-list lint for length, material and color words, pose conflicts, missing framing phrases, generic hues, shape words in texture prompts and unsided parts in Edit prompts.
- `scripts/validate_skills.py`, the five repo checks, as in meshy-3d-agent.
- `evals/trigger/<skill>.json`: trigger eval sets (20 queries each, 22 for text-to-3d; half should-trigger, half near-miss negatives such as sibling-skill requests and "run the task" requests). Each description was run through the skill-creator description-optimization loop (claude-fable-5-1, 3 runs per query, 60/40 train/test): all four passed every train and test query on the first iteration, so the hand-written descriptions were kept unchanged (2026-10-15).
