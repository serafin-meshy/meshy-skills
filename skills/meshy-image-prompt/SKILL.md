---
name: meshy-image-prompt
description: Write the prompt and parameters for a Meshy Text to Image (or Image to Image) task whose output will feed Image to 3D, including A-pose characters for rigging and the three-call rig-ready pipeline. Use this skill whenever the user wants a 3D model to start from a generated image, asks for a Meshy image prompt, a concept or reference image "for 3D", a character they can rig or animate, an A-pose or T-pose, a multi-view image set, or asks why an Image to 3D result invented a front, fused the arms or baked shadows into the texture. It covers the subject-first image formula (full object, front or three-quarter view, even lighting, plain background), when to use pose_mode versus pose words, remove_background, generate_multi_view and aspect_ratio, and the Image to 3D settings that produced a riggable mesh. Not for Text to 3D preview prompts (meshy-text-to-3d-prompt), texture or retexture prompts (meshy-texture-prompt), or Meshy Edit (meshy-edit-prompt); it writes prompts and parameters, it does not call the API.
license: MIT
metadata:
  author: meshy-dev
  version: "0.1.0"
---

# Meshy image prompts for Image to 3D

Image to 3D reconstructs what the picture shows and invents the rest, so a good input image shows the whole object, from an angle that exposes the most surface, lit evenly, alone on a plain background. The Text to Image prompt describes that picture. This is also the only route that gives a standard A-pose for rigging: Text to 3D did not produce one in 18 tries, while Text to Image with `pose_mode` followed by Image to 3D did in 3 of 3.

The rules below come from live runs on 2026-10-07 with `nano-banana-pro` for images and `ai_model: "latest"` (Meshy 7.1) for 3D. [references/evidence.md](references/evidence.md) has the runs and task ids; [references/rig-ready-characters.md](references/rig-ready-characters.md) has the full rigging pipeline with costs.

## Workflow

1. **Decide what the image is for.** Feeding Image to 3D (this skill), or a final 2D image (drop the view, background and single-object slots and write composition, camera and lighting mood instead). For a character the user will rig or animate, follow the rig-ready pipeline in step 5.
2. **Write the prompt with the formula.** Subject first, then pose only when it is not set by `pose_mode`, then framing, materials and colors per part, style, and the fixed closing phrase.
3. **Set the parameters** from the table below. Never set `pose_mode` and describe a different pose in the same request.
4. **Check it.** Run `python3 scripts/check_prompt.py --kind image [--pose-mode a-pose] "…"` (path relative to this skill's directory). It flags missing framing, lighting or background phrases, thin or dense parts, pose words that conflict with `pose_mode`, and length.
5. **Say what happens next.** For a rig-ready character: Image to 3D with `should_texture: true`, `should_remesh: true`, `target_polycount: 30000`, then Rigging with that task id. For anything else: Image to 3D (textured or mesh-only as the user needs), no remesh settings unless asked.
6. **Deliver prompt, parameters, next step and assumptions** in the output format at the end.

## The formula

```
[Subject], [pose, only if not using pose_mode], [full object or full body], [front view or three-quarter front view], centered with space around it, [material and color per part], [style], soft even studio lighting, plain white background, single object, no props, no text
```

Why each slot is there:

- **Full object in view.** A part cropped out of the frame has to be guessed.
- **Front view for characters, three-quarter front view for props.** These angles show the most surface. A single image that showed a rear three-quarter view of a toy robot left Image to 3D to invent the front, and it put a blob inside the glass dome.
- **Soft even lighting, plain white background.** Hard shadows and highlights in the image are painted into the model's texture; a cast shadow on the ground becomes part of the model.
- **Single object, no props, no text.** The bare prompt "a knight's helmet" produced a feather plume, a grey background, a cast shadow and a tilted pose. A plume is a dense thin structure that converts badly.
- **Materials and colors per part.** Unlike the Text to 3D preview, this image carries the colors: Image to 3D textures from it.

Tested examples:

- Prop: `A medieval knight's helmet, full object, three-quarter front view, centered with space around it, dark hammered iron dome with a narrow eye slit and a pointed visor, brass rivets, polished steel ridge along the top, light rust in the dents, realistic style, soft even studio lighting, plain white background, single object, no props, no text` (with `aspect_ratio: "1:1"`)
- Character for rigging, with `pose_mode: "a-pose"`: `A stocky dwarf blacksmith, braided grey beard reaching the belt, leather apron over a red tunic, heavy brown boots, full body, front view, centered with space around it, stylized proportions, soft even studio lighting, plain white background, single character, no props, no text`

## Parameters

| Parameter | Use | Notes |
|---|---|---|
| `ai_model` | `nano-banana` (3 credits) for drafts; `nano-banana-pro`, `gpt-image-2`, `gpt-image-2-5-flare` or `gpt-image-2-5-sunburst` (9 credits) for the final image | `nano-banana-2` is 6, `nano-banana-2-lite` 3. `nano-banana-pro` and `pose_mode` need a Pro plan or above. |
| `pose_mode: "a-pose"` or `"t-pose"` | Characters you will rig | Leave pose words out of the prompt. On the nano-banana models it fixes the aspect ratio to 3:4 and ignores `aspect_ratio`; a posed `nano-banana` request is generated by `nano-banana-pro`. GPT Image models keep the requested model and ratio. |
| `aspect_ratio` | `"3:4"` for a standing character, `"1:1"` for anything else | Cannot be combined with `generate_multi_view`. |
| `remove_background: true` | Returns a transparent PNG | Only when the user asks; the plain white background in the prompt already converts cleanly. |
| `generate_multi_view: true` | Three cropped views for Multi-Image to 3D | Same 9 credits as one image, 66 to 97 s instead of 24. See the multi-view finding below before choosing it. |

For Image to Image (cleaning up a user's photo, or making multi-views from a single generated image), the same prompt rules apply and `reference_image_urls` or `input_task_id` carries the source. `gpt-image-2-5-sunburst` is the editing-precision model for "change X, keep the rest" edits.

## Pose

For A-pose or T-pose use `pose_mode` and leave pose words out; for any other pose write it in the prompt and leave `pose_mode` unset; never let the two disagree.

- The parameter and the prompt gave equally clean A-poses (arms about 45 degrees from the body, hands clear). The parameter's advantage is length: it leaves the 800 characters for the character.
- Using both added nothing and drifted the style (photoreal, smaller in frame, a floor shadow).
- A conflict, `pose_mode: "a-pose"` with "sitting on a stool" in the prompt, produced a blend: A-pose arms, straight legs and a stool between the legs.
- Statues, figurines and 3D prints in a specific pose: describe it body part by body part, body position, then arms, then hands, then head. "Kneeling on one knee, left hand on the knee, right hand holding a staff upright, head turned slightly left" works; "heroic pose" does not.
- Any pose that will go through Image to 3D: keep limbs clear of the body, hands away from the face and torso, and the weight on both feet or on a visible base. Arms against the body are the usual cause of a forearm fused to the torso after rigging.

## Rig-ready characters in three calls

1. Text to Image with `pose_mode: "a-pose"`, `ai_model: "nano-banana-pro"`: 9 credits.
2. Image to 3D with `should_texture: true`, `should_remesh: true`, `target_polycount: 30000`: 30 credits. The tested run came out at 30,912 faces, textured from the image. Without remesh, Image to 3D meshes came out at 632,000 to 1.37 million faces, over the 300,000 rigging limit.
3. Rigging with that task id as `input_task_id`: 5 credits. Includes walking and running animations.

Total 44 credits. The tested rig had a standard A-pose (upper arms 56° below horizontal, elbows bent 26°), 24 joints and no forearm or hand weighted to the torso. Ask for clothing that ends above the knee or splits between the legs: a long apron or skirt is weighted to both legs and stretches when they separate. Details and the longer alternative route are in [references/rig-ready-characters.md](references/rig-ready-characters.md).

## Single image or multi-view

One image shows one side; everything on the far side is inferred. When the back matters (a backpack, a key, a logo), the tested order of results was:

1. **Best:** generate one image, then Image to Image with `generate_multi_view: true` from it, then Multi-Image to 3D. The views kept the design and the orientation of the back parts consistent.
2. Text to Image with `generate_multi_view: true` directly: the three views disagreed on where a wind-up key sat, and the 3D put it on the wrong side.
3. A single image from an angle that hides the front: Image to 3D invented the hidden side.

Multi-view output is three cropped transparent PNGs (about 340 to 450 by 610 to 690 pixels) rather than one 1024-square image, at the same credit cost. A glass dome came out opaque white in every arm; transparent materials do not survive the conversion.

## Output format

```
Prompt: <the image prompt, under 800 characters>
Parameters: {"ai_model":"nano-banana-pro","aspect_ratio":"1:1"}   plus pose_mode / remove_background / generate_multi_view when they apply
Next step: Image to 3D   or   Image to 3D with should_texture, should_remesh and target_polycount 30000, then Rigging
Assumptions: <values you chose that the idea did not give>
```

Keep the prompt itself under 800 characters; two of five long ideas ran over in testing when no length rule was applied.

## Resources

| File | Use it when |
|---|---|
| [references/evidence.md](references/evidence.md) | You need the runs behind a rule: pose test arms, multi-view test, bare versus formula images |
| [references/rig-ready-characters.md](references/rig-ready-characters.md) | The user wants a character to rig or animate: the pipeline, costs, rig measurements, clothing advice |
| [assets/meta-prompt-code.txt](assets/meta-prompt-code.txt) | The user is building an app that turns ideas into image prompts with a language model; returns one line of JSON |
| [assets/meta-prompt-plain.txt](assets/meta-prompt-plain.txt) | Same, for a person who will read and paste the result |
| `scripts/check_prompt.py` | Lint a drafted prompt: `python3 scripts/check_prompt.py --kind image --pose-mode a-pose "<prompt>"` |

Related skills: `meshy-text-to-3d-prompt` when the model should come straight from text, `meshy-texture-prompt` for retexturing the result, `meshy-edit-prompt` for changing one part afterwards, `meshy-3d-generation` to run the tasks.
