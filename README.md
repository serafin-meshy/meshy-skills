# Meshy Prompting

Agent skills that write good prompts for [Meshy](https://www.meshy.ai): one skill per prompt field, each built from live A/B tests rather than guesswork. They pair with the [meshy-3d-agent](https://github.com/meshy-dev/meshy-3d-agent) skills and the Meshy MCP server, which run the tasks; these skills decide what to type.

| Skill | Prompt field | Describes | Leaves out |
|---|---|---|---|
| [`meshy-text-to-3d-prompt`](skills/meshy-text-to-3d-prompt/) | Text to 3D preview `prompt` (800 chars) | Shape: silhouette, parts, proportions, large relief | Colors, materials, damage, pose, face count |
| [`meshy-image-prompt`](skills/meshy-image-prompt/) | Text to Image / Image to Image `prompt` feeding Image to 3D | The whole object as it should be converted: view, materials, lighting, background; A-pose characters for rigging | Props, text, scenes, a pose that conflicts with `pose_mode` |
| [`meshy-texture-prompt`](skills/meshy-texture-prompt/) | `texture_prompt` (refine, Image to 3D) and `text_style_prompt` (Retexture), 800 chars | Surfaces: material, color, pattern and wear per part, finish | Shape changes, lighting, shadows |
| [`meshy-edit-prompt`](skills/meshy-edit-prompt/) | Meshy Edit prompt box (web app, after a lasso) | The selected part by name and side, its new shape, what is left after a removal | "it", "keep everything else unchanged", colors and materials |

Two rules apply everywhere: put the subject first (Meshy's prompting guide says the earliest words carry the most weight), and set anything a parameter controls with the parameter (`pose_mode`, `target_polycount`, `remove_background`, `enable_pbr`).

## What each skill contains

- `SKILL.md`: the workflow, the formula with a slot table, tested example prompts, the rules with the run behind each one, an output format and a symptom table.
- `references/evidence.md`: the live runs with task ids, counts and what came out, so a claim can be checked or cited without spending credits.
- `assets/meta-prompt-code.txt` and `assets/meta-prompt-plain.txt` (three of the four skills): system prompts that turn a rough idea into a formula prompt with a language model, for people building their own prompt helpers. The code version returns one line of JSON under 800 characters; the plain version returns labeled lines. Both were tested on Claude Sonnet 5.5 and Claude Haiku 4.5.
- `scripts/check_prompt.py`: a standard-library lint, `python3 scripts/check_prompt.py --kind preview|image|texture|edit "<prompt>"`, that flags the mistakes the tests found most often.

## Findings the skills are built on

- A short prompt is an invitation to design: "a knight's helmet" came back with scrollwork and a crest; the formula prompt came back with the dome, eye slit, visor, ridge, rivets and breathing holes that were asked for.
- Text to 3D produced no standard A-pose in 18 character previews, with or without `pose_mode`. Text to Image with `pose_mode: "a-pose"` then Image to 3D produced one in 3 of 3, and those rigs were clean. A rig-ready character takes three calls and 44 credits.
- Texture colors want a hue, not a family: "saddle-brown leather" held in 4 of 4 runs; "brown" and "dark brown" drifted to grey, and "small burn marks" turned "dark brown" charcoal.
- Meshy Edit rewrites your sentence against a render from your camera angle, so "raven on shoulder" went to the unselected shoulder twice and "the left shoulder" went to the right one twice; the lasso, not the prompt, sets how much of the model changes.

## Install

Claude Code, as a plugin marketplace:

```bash
claude plugin marketplace add serafin-meshy/meshy-skills
```

Or copy any `skills/<name>` directory into `.claude/skills/` (Claude Code), `.cursor/skills/` (Cursor) or `.agents/skills/` (Codex); every skill is self-contained.

## Checking the repo

```bash
pip install pyyaml && python3 scripts/validate_skills.py
```

## Sources

The formulas and evidence come from the "Meshy Prompt Formulas" test series (2026-10-07) and the Meshy Edit test (2026-10-14), checked against the Meshy API docs ([Text to 3D](https://docs.meshy.ai/api/text-to-3d), [Text to Image](https://docs.meshy.ai/api/text-to-image), [Image to 3D](https://docs.meshy.ai/api/image-to-3d), [Retexture](https://docs.meshy.ai/api/retexture), [Rigging](https://docs.meshy.ai/api/rigging), [Pricing](https://docs.meshy.ai/api/pricing)) and the [Prompting Best Practices](https://docs.meshy.ai/webapp/guides/prompting) guide on 2026-10-14. Each comparison in the evidence files is one to six samples per arm; the files say which.
