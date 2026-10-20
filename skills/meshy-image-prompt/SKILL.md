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

The rules below come from live runs on 2026-10-07 with `nano-banana-pro` for images and `ai_model: "latest"` (Meshy 7.1) for 3D. The formula and `pose_mode` were rechecked on six more image models on 2026-10-16 (see the `ai_model` row). The formula was also run on six props on 2026-10-20 with `gpt-image-2-5-flare` (see "Props"). [references/evidence.md](references/evidence.md) has the runs; [references/rig-ready-characters.md](references/rig-ready-characters.md) has the full rigging pipeline with costs.

## Workflow

1. **Decide what the image is for.** Feeding Image to 3D (this skill), or a final 2D image (drop the view, background and single-object slots and write composition, camera and lighting mood instead). For a character the user will rig or animate, follow the rig-ready pipeline in step 5.
2. **Write the prompt with the formula.** Subject first, then pose only when it is not set by `pose_mode`, then framing, materials and colors per part, style, and the fixed closing phrase.
3. **Set the parameters** from the table below. Never set `pose_mode` and describe a different pose in the same request. Default to `gpt-image-2-5-flare`, which works on every Meshy plan; suggest `nano-banana-pro` only when the user says they have a Pro plan or above.
4. **Check it.** Run `python3 scripts/check_prompt.py --kind image [--pose-mode a-pose] "…"` (path relative to this skill's directory). It flags missing framing, lighting or background phrases, thin or dense parts, clear glass, pose words that conflict with `pose_mode`, and length.
5. **Say what happens next.** For a rig-ready character: Image to 3D with `should_texture: true`, `should_remesh: true`, `target_polycount: 30000`, then Rigging with that task id. For anything else: Image to 3D (textured or mesh-only as the user needs). Add `should_remesh: true` and a `target_polycount` when the model goes into a game engine, a web viewer or AR; otherwise leave them out.
6. **Deliver prompt, parameters, next step and assumptions** in the output format at the end.

## The formula

```
[Subject], [pose, only if not using pose_mode], [full object or full body], [front view or three-quarter front view], centered with space around it, [material and color per part], [style], soft even studio lighting, plain white background, single object, no props, no text
```

Why each slot is there:

- **Full object in view.** A part cropped out of the frame has to be guessed.
- **Front view for characters, three-quarter front view for props.** These angles show the most surface. A single image that showed a rear three-quarter view of a toy robot left Image to 3D to invent the front, and it put a blob inside the glass dome. The image model sometimes draws a prop almost straight on anyway (a lantern and a sword in the prop test); both still converted cleanly.
- **Centered with space around it, at an aspect ratio that fits the object.** At 1:1, a delivery van's cargo box ran off the right edge despite this phrase; at 4:3 the whole van fit.
- **Soft even lighting, plain white background.** Hard shadows and highlights in the image are painted into the model's texture; a cast shadow on the ground becomes part of the model.
- **Single object, no props, no text.** The bare prompt "a knight's helmet" produced a feather plume, a grey background, a cast shadow and a tilted pose. A plume is a dense thin structure that converts badly. Other bare prompts added a lit flame to a lantern, a cartoon face to a car, bushes and a potted plant to a house, and a grey background with a diagonal angle to a sword.
- **Materials and colors per part.** Unlike the Text to 3D preview, this image carries the colors: Image to 3D textures from it.

Tested examples:

- Prop: `A medieval knight's helmet, full object, three-quarter front view, centered with space around it, dark hammered iron dome with a narrow eye slit and a pointed visor, brass rivets, polished steel ridge along the top, light rust in the dents, realistic style, soft even studio lighting, plain white background, single object, no props, no text` (with `aspect_ratio: "1:1"`)
- Prop with thin parts: `A farmhouse dining chair, full object, three-quarter front view, centered with space around it, upright back with five vertical spindles under a curved top rail, square seat, four slightly splayed legs joined by stretcher bars, honey-colored oak with visible grain and a satin finish, realistic style, soft even studio lighting, plain white background, single object, no props, no text` (with `aspect_ratio: "1:1"`; every spindle and stretcher survived Image to 3D)
- Wide prop: `A cartoon delivery van, full object, three-quarter front view, centered with space around it, short boxy body with an oversized rounded cab, glossy mint-green paint, white cargo box at the back, large round headlights, chunky black tires, chrome front bumper, roof rack, stylized style, soft even studio lighting, plain white background, single object, no props, no text` (with `aspect_ratio: "4:3"`)
- Character for rigging, with `pose_mode: "a-pose"`: `A stocky dwarf blacksmith, braided grey beard reaching the belt, leather apron over a red tunic, heavy brown boots, full body, front view, centered with space around it, stylized proportions, soft even studio lighting, plain white background, single character, no props, no text`

## Parameters

| Parameter | Use | Notes |
|---|---|---|
| `ai_model` | `gpt-image-2-5-flare`, `gpt-image-2`, `gpt-image-2-5-sunburst` or `nano-banana-pro` (9 credits) for the final image; `nano-banana-2` (6), `nano-banana-2-lite` or `nano-banana` (3) for drafts | Tested on 2026-10-16 with the two example prompts above, one sample per model. The three GPT Image models followed the formula best: pure white background, little or no shadow, every requested material, nothing added. `nano-banana-pro` and `nano-banana-2` passed, but with light grey backgrounds and soft shadows. `nano-banana` missed two requested details, and `nano-banana-2-lite` added a chainmail neck guard draped onto the floor. Untested: `nano-banana-2-1`, `t-pose`, and Image to Image, multi-view or rigging on any model except `nano-banana-pro`. `nano-banana-pro` needs a Pro plan or above; the GPT Image models and the other nano-banana models do not. |
| `pose_mode: "a-pose"` or `"t-pose"` | Characters you will rig | Leave pose words out of the prompt. `"a-pose"` gave a clean A-pose with the hands clear for every model requested, and mesh-only Image to 3D kept the arms clear of the torso on every one tested (2026-10-16, one sample each). On the nano-banana models it fixes the aspect ratio to 3:4 and ignores `aspect_ratio`. A posed `nano-banana` request is generated by `nano-banana-pro`, so it needs a Pro plan; `nano-banana-2`, `nano-banana-2-lite` and the GPT Image models keep the requested model and work on any plan. GPT Image models with no `aspect_ratio` set returned square images, so send `"3:4"` for a standing character. |
| `aspect_ratio` | `"3:4"` for a standing character, `"4:3"` for a wide object such as a vehicle or a bench, `"1:1"` for anything else | At 1:1 a van was cut off at the right edge; 4:3 fit it (one sample each). Cannot be combined with `generate_multi_view`. |
| `remove_background: true` | Returns a transparent PNG | Only when the user asks; the plain white background in the prompt already converts cleanly. |
| `generate_multi_view: true` | Three cropped views for Multi-Image to 3D | Same 9 credits as one image, 66 to 97 s instead of 24. See the multi-view finding below before choosing it. |

For Image to Image (cleaning up a user's photo, or making multi-views from a single generated image), the same prompt rules apply and `reference_image_urls` or `input_task_id` carries the source. `gpt-image-2-5-sunburst` is the editing-precision model for "change X, keep the rest" edits.

## Pose

For A-pose or T-pose use `pose_mode` and leave pose words out; for any other pose write it in the prompt and leave `pose_mode` unset; never let the two disagree.

- The parameter and the prompt gave equally clean A-poses (arms about 45 degrees from the body, hands clear), and both rigged with no forearm or hand weighted to the torso. The parameter's advantage is length: it leaves the 800 characters for the character. The tested A-pose wording, for when `pose_mode` is not used: "standing in an A-pose, arms straight and angled down about 45 degrees away from the body, hands open and clear of the body, legs slightly apart, facing forward".
- Using both added nothing and drifted the style (photoreal, smaller in frame, a floor shadow).
- A conflict, `pose_mode: "a-pose"` with "sitting on a stool" in the prompt, produced a blend: A-pose arms, straight legs and a stool between the legs.
- Statues, figurines and 3D prints in a specific pose: describe it body part by body part, body position, then arms, then hands, then head. "Kneeling on one knee, left hand on the knee, right hand holding a staff upright, head turned slightly left" works; "heroic pose" does not.
- Any pose that will go through Image to 3D: keep limbs clear of the body, hands away from the face and torso, and the weight on both feet or on a visible base. Arms against the body are the usual cause of a forearm fused to the torso after rigging.

## Rig-ready characters in three calls

1. Text to Image with `pose_mode: "a-pose"`, `aspect_ratio: "3:4"` and `ai_model: "gpt-image-2-5-flare"` (any plan) or `"nano-banana-pro"` (Pro plan or above): 9 credits. The full three-call pipeline was tested with `nano-banana-pro`; the `gpt-image-2-5-flare` A-pose image converted with the arms clear of the torso but was not rigged in testing.
2. Image to 3D with `should_texture: true`, `should_remesh: true`, `target_polycount: 30000`: 30 credits. The tested run came out at 30,912 faces, textured from the image. Without remesh, Image to 3D meshes came out at 632,000 to 1.37 million faces, over the 300,000 rigging limit.
3. Rigging with that task id as `input_task_id`: 5 credits. Includes walking and running animations.

Total 44 credits. The tested rig had a standard A-pose (upper arms 56° below horizontal, elbows bent 26°), 24 joints and no forearm or hand weighted to the torso. Ask for clothing that ends above the knee or splits between the legs: a long apron or skirt is weighted to both legs and stretches when they separate. Details and the longer alternative route are in [references/rig-ready-characters.md](references/rig-ready-characters.md).

## Props

Six props were tested on 2026-10-20: a lantern, a chair, a longsword, a potted plant, a cottage and a cartoon van. Each formula image (`gpt-image-2-5-flare`) went through textured Image to 3D on `latest`, one sample each. The runs are in [references/evidence.md](references/evidence.md).

- **The 3D followed the image on all six props,** including thin parts that are part of the object's structure: six chair spindles and the stretcher bars, the lantern's loop handle, the van's roof rack and the monstera leaves. The same six ideas on Text to 3D lost the cartoon style, some proportions and some part counts (see `meshy-text-to-3d-prompt`). Prefer this route when a prop's look matters. The image costs 9 credits and shows the design before any 3D is paid for; with textured Image to 3D the total is 39 credits, against 30 for a Text to 3D preview and refine.
- **Leave out dense clusters, not every thin part.** Feathers, plumes, chains and netting are the parts Meshy's error documentation names, and a plume was the problem in the helmet test. A handle, spindles, a rack and leaves converted fine here.
- **Glass.** "Clear glass panes" came out as an open frame with no panes, and the burner seen through the glass was modeled inside. "Frosted glass panes" came out as solid panes with the burner's outline painted on them. A clear glass dome on an earlier toy robot came out opaque white. Image to 3D makes no transparent material, so ask for frosted or tinted glass when the panes should be there.
- **Face counts.** Raw textured meshes ran from 47,804 faces (sword) to 1.6 million (van), and the thatched cottage reached 7.7 million faces in a 303 MB GLB, taking 821 seconds against 47 to 152 for the others. The same cottage image with `should_remesh: true` and `target_polycount: 30000` gave 30,028 faces, a 6.6 MB file, in 195 seconds, at the same 30 credits, and looked the same at viewing size. For a game engine, a web viewer or AR, set both.

## Single image or multi-view

One image shows one side; everything on the far side is inferred. When the back matters (a backpack, a key, a logo), the tested order of results was:

1. **Best:** generate one image, then Image to Image with `generate_multi_view: true` from it, then Multi-Image to 3D. The views kept the design and the orientation of the back parts consistent.
2. Text to Image with `generate_multi_view: true` directly: the three views disagreed on where a wind-up key sat, and the 3D put it on the wrong side.
3. A single image from an angle that hides the front: Image to 3D invented the hidden side.

Multi-view output is three cropped transparent PNGs (about 340 to 450 by 610 to 690 pixels) rather than one 1024-square image, at the same credit cost. A glass dome came out opaque white in every arm; transparent materials do not survive the conversion.

## Output format

```
Prompt: <the image prompt, under 800 characters>
Parameters: {"ai_model":"gpt-image-2-5-flare","aspect_ratio":"1:1"}   plus pose_mode / remove_background / generate_multi_view when they apply
Next step: Image to 3D   or, for a game engine, web viewer or AR, Image to 3D with should_remesh and a target_polycount   or, for rigging, Image to 3D with should_texture, should_remesh and target_polycount 30000, then Rigging
Assumptions: <values you chose that the idea did not give>
```

Keep the prompt itself under 800 characters; two of five long ideas ran over in testing when no length rule was applied.

## Resources

| File | Use it when |
|---|---|
| [references/evidence.md](references/evidence.md) | You need the runs behind a rule: pose test arms, multi-view test, bare versus formula images, the six-prop test |
| [references/rig-ready-characters.md](references/rig-ready-characters.md) | The user wants a character to rig or animate: the pipeline, costs, rig measurements, clothing advice |
| [assets/meta-prompt-code.txt](assets/meta-prompt-code.txt) | The user is building an app that turns ideas into image prompts with a language model; returns one line of JSON |
| [assets/meta-prompt-plain.txt](assets/meta-prompt-plain.txt) | Same, for a person who will read and paste the result |
| `scripts/check_prompt.py` | Lint a drafted prompt: `python3 scripts/check_prompt.py --kind image --pose-mode a-pose "<prompt>"` |

Related skills: `meshy-text-to-3d-prompt` when the model should come straight from text, `meshy-texture-prompt` for retexturing the result, `meshy-edit-prompt` for changing one part afterwards, `meshy-3d-generation` to run the tasks.
