# Evidence: image prompts for Image to 3D

Live runs from 2026-10-07 with `nano-banana-pro` on `POST /openapi/v1/text-to-image` and `ai_model: "latest"` (Meshy 7.1) on Image to 3D, plus a comparison of six other image models on 2026-10-16. One sample per arm unless a count is given. Output folders are under `api-testing/meshy_output/` in the author's workspace.

## Formula versus bare prompt (helmet image, 9 credits each)

| Arm | Prompt | Task | Result |
|---|---|---|---|
| Bare | "a knight's helmet" | `01a11760-4349-701e-94bb-3ddaec45c525` | Feather plume, grey background, cast shadow, tilted pose |
| Formula | "A medieval knight's helmet, full object, three-quarter front view, centered with space around it, dark hammered iron dome with a narrow eye slit and a pointed visor, brass rivets, polished steel ridge along the top, light rust in the dents, realistic style, soft even studio lighting, plain white background, single object, no props, no text" | `01a11760-4436-7120-befa-266b6286bbd9` | Three-quarter view, near-white background, no props, the requested materials |

Folder `ab-test-20261007_111901/`.

## Pose test (dwarf blacksmith, one image per arm, then mesh-only Image to 3D, 96 credits)

Base prompt: "A stocky dwarf blacksmith, braided grey beard reaching the belt, leather apron over a red tunic, heavy brown boots, full body, front view, centered with space around it, stylized proportions, soft even studio lighting, plain white background, single character, no props, no text". Arms B and C appended "standing in an A-pose, arms straight and angled down about 45 degrees away from the body, hands open and clear of the body, legs slightly apart, facing forward". Arm D appended "sitting on a wooden stool, hands resting on his knees".

| Arm | Setup | Image | Image to 3D mesh |
|---|---|---|---|
| A | `pose_mode: "a-pose"`, no pose words | Clean A-pose, arms about 45 degrees from the body | Arms and hands separate from the torso |
| B | A-pose described in the prompt, no `pose_mode` | Clean A-pose | Arms separate, widest gap of the three |
| C | Both, matching | A-pose with arms higher; style and framing drifted (photoreal, smaller in frame, floor shadow) | Arms separate |
| D | `pose_mode: "a-pose"` with "sitting on a stool" | A mix: A-pose arms, straight legs, a stool between the legs | Not run |

Image tasks A to D: `01a11769-ab82-737e-9287-10dc069ff5f8`, `01a11769-b090-761c-8fc4-442ba005724d`, `01a11769-b17e-760e-9e57-e590194e1b21`, `01a11769-b545-748c-bcb2-86868058a226`. Mesh tasks A to C: `01a1176b-7e89-73c7-916d-e0978d63bbef`, `01a1176b-82e1-7732-aa29-e6c832f5d08d`, `01a1176b-8701-712f-ba90-bd5c93373a89`. Folder `pose-test-20261007_112923/`.

The three meshes came out at 0.6 to 1.4 million faces and were remeshed to 30,000, retextured with the source image as `image_style_url`, and rigged (60 credits, subfolder `rig/`). Rig results are in [rig-ready-characters.md](rig-ready-characters.md).

## Single image versus multi-view (toy robot with a battery pack and wind-up key on its back, 117 credits)

Prompt: "A stylized toy robot standing upright, round glass dome head, boxy teal body, short stubby legs, a large red battery pack with yellow-and-black warning stripes strapped to its back, a brass wind-up key sticking out of the back of the battery pack, single centered object, plain light-grey studio background, even soft lighting, full object visible". All 3D runs on `latest` with `multi_view_thumbnails: true`.

| Arm | Images | 3D result |
|---|---|---|
| A: single Text to Image, then Image to 3D | One 1024-square opaque image; the model drew a rear three-quarter view | The front was invented: a blob inside the dome |
| B: Image to Image with `generate_multi_view: true` from image A, then Multi-Image to 3D | Three cropped transparent PNGs, design and key orientation consistent with A | Best of the three |
| C: Text to Image with `generate_multi_view: true`, then Multi-Image to 3D | Three views that disagreed on the key's orientation | Key ended up on the side of the battery |

Multi-view Text to Image and Image to Image cost 9 credits, the same as one single image, and took 66 to 97 s against 24 s. The three views were about 340 to 450 by 610 to 690 pixels. The glass dome came out opaque white in all three arms. Folder `multiview-test-20261007_145613/`.

## Other image models (helmet and posed dwarf, 178 credits, 2026-10-16)

The helmet formula prompt above with `aspect_ratio: "1:1"`, and the pose-test base prompt with `pose_mode: "a-pose"` and no pose words, run on six more models. Each posed image then went through mesh-only Image to 3D (`latest`, `should_texture: false`, `multi_view_thumbnails: true`), and the mesh was judged from its front, left, back and right thumbnails. One sample per arm. nano-banana-pro is the 2026-10-07 baseline above, not rerun. nano-banana got only the helmet plus one posed image to check which model generated it, with no mesh. "Background" is the mean corner brightness, where 255 is pure white.

| Model | Helmet (credits, s) | Helmet result | Posed dwarf (credits, s) | Pose and image | Mesh |
|---|---|---|---|---|---|
| nano-banana-pro (baseline) | 9, 47 | Pass: three-quarter view, background 246, requested materials | 9, 32 | A-pose, hands clear; background 225, cast shadow | Arms clear |
| nano-banana | 3, 128 | Partial: soft cast shadow; brass-edged ridge instead of polished steel, rounded visor | 3, 21 | Task reported `nano-banana-pro`; A-pose, hands clear; photoreal, not stylized | Not run |
| nano-banana-2 | 6, 83 | Pass with notes: every material; background 243, soft shadow, short mail fringe added | 6, 132 | A-pose, hands clear; outlined illustration style, leather cap added | Arms and hands clear, 991,726 faces |
| nano-banana-2-lite | 3, 10 | Partial: chainmail neck guard added, draped onto the floor; background 241, shadow | 3, 11 | A-pose, hands clear; photoreal; background 224, floor shadow | Arms clear, widest gap, 1,077,158 faces |
| gpt-image-2 | 9, 31 | Pass: background 254, faint contact shadow, every material | 9, 36 | A-pose with lower arms, hands clear; background 254, no shadow | Arms clear, hands nearest the hips, 1,353,064 faces |
| gpt-image-2-5-flare | 9, 19 | Pass: background 254, faint contact shadow, every material | 9, 19 | A-pose, hands clear; background 254, no shadow | Arms clear, 1,137,416 faces |
| gpt-image-2-5-sunburst | 9, 35 | Pass: background 254, no shadow, every material; rust heavier than asked | 9, 36 | A-pose, hands clear; background 254; tight crop, hair 9 px from the top | Arms clear, 1,227,994 faces |

Findings:

- `pose_mode: "a-pose"` gave a rig-ready A-pose for every model requested: 6 of 6 posed images and 5 of 5 meshes, one sample each. The GPT Image models apply it too.
- The GPT Image models followed the formula most closely: pure white backgrounds, little or no shadow, every requested material and nothing added, on both prompts. The nano-banana family, pro included, gave light grey backgrounds and soft shadows on most images.
- The 3-credit models drifted on the helmet. nano-banana dropped two requested details; nano-banana-2-lite added a dense, thin chainmail part.
- "Stylized proportions" was followed loosely: the dwarfs from nano-banana (via pro) and nano-banana-2-lite came out photoreal, and the one from nano-banana-2 as an outlined illustration.
- All four nano-banana requests with `pose_mode` returned 896×1200 (3:4). The GPT Image models returned 1024×1024 with no `aspect_ratio` set; `pose_mode` with `aspect_ratio: "3:4"` on a GPT model was not tried.
- The posed nano-banana request reported `nano-banana-pro`, as the docs say, but was billed 3 credits, against 9 for nano-banana-pro requested directly.
- Without remesh every mesh was 0.99 to 1.35 million faces, over the 300,000-face rigging limit.

Image tasks:

| Model | Helmet | Posed dwarf |
|---|---|---|
| nano-banana | `01a14575-acf0-778c-bcdf-73d97ea0f8e3` | `01a14575-b4b9-741c-8736-f67cee5aeaec` |
| nano-banana-2 | `01a14575-bb5d-707f-ad07-05eb3d9e48d9` | `01a14575-c0be-771c-90de-bef676d0c675` |
| nano-banana-2-lite | `01a14575-c585-764a-a8da-c775882927a1` | `01a14575-ca9c-7261-9135-4fd2aab80a90` |
| gpt-image-2 | `01a14575-cf81-706e-8a64-df4129ae7adc` | `01a14575-d453-774f-9b10-9a4d3b466fee` |
| gpt-image-2-5-flare | `01a14575-d97d-76b8-b8ac-d09bce32db4f` | `01a14575-de3e-7309-a67c-a7a7976d0f35` |
| gpt-image-2-5-sunburst | `01a14575-e36e-7690-b4f4-77992a025100` | `01a14575-e827-7061-b2a2-402564184345` |

Mesh tasks (20 credits each, 37 to 42 s): nano-banana-2 `01a14578-4f25-7121-bb9f-e34d4b6c6d29`, nano-banana-2-lite `01a14578-5682-775e-b4b1-c6b9f9180b53`, gpt-image-2 `01a14578-5d58-77cc-807f-1b0ce54d6fb3`, gpt-image-2-5-flare `01a14578-6573-7027-8e38-3a329890a022`, gpt-image-2-5-sunburst `01a14578-6d7b-7283-a814-879be548928c`. Folder `model-compare-20261016_100501/`, with `RESULTS.md`.

## Sources

- [Text to Image API](https://docs.meshy.ai/api/text-to-image): `ai_model` required; credits nano-banana 3, nano-banana-2 6, nano-banana-2-lite 3, nano-banana-pro 9, GPT Image models 9; `pose_mode` on the nano-banana models fixes the aspect ratio to 3:4 and a posed `nano-banana` request is generated by `nano-banana-pro` (Pro plan or above); `generate_multi_view` and `aspect_ratio` cannot be combined; `image_urls` holds three URLs with multi-view, otherwise one.
- [Image to Image API](https://docs.meshy.ai/api/image-to-image): same `generate_multi_view`, `aspect_ratio` and `remove_background` rules; GPT Image models cost 12.
- [Image to 3D API](https://docs.meshy.ai/api/image-to-3d): `should_remesh` default false on Meshy 6 and 7; `target_polycount` 100 to 300,000 (default 30,000); `pose_mode`; `texture_prompt` max 800 characters.
- [Rigging API](https://docs.meshy.ai/api/rigging): 300,000-face limit for `input_task_id`; textured humanoid models only.
