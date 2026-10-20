# Rig-ready characters

A rig-ready character takes three API calls: an A-pose image, one textured Image to 3D task that also remeshes, and Rigging. Rigging accepts at most 300,000 faces through `input_task_id` and needs a textured humanoid model; Image to 3D meshes came out at 632,000 to 1.37 million faces without remesh, so the remesh setting is required.

## The pipeline (44 credits, tested 2026-10-07)

| Step | Request | Credits | Tested result |
|---|---|---|---|
| 1. Text to Image | `{"ai_model":"nano-banana-pro","pose_mode":"a-pose","prompt":"<formula prompt, no pose words>"}` | 9 | Clean A-pose image (pose-test arm A) |
| 2. Image to 3D | `{"image_url":"<step 1 image>","should_texture":true,"should_remesh":true,"target_polycount":30000}` | 30 | 30,912 faces, textured from the image |
| 3. Rigging | `{"input_task_id":"<step 2 task>"}` | 5 | Standard A-pose, 24 joints, no forearm or hand weighted to the torso; walking and running animations included |

`input_task_id` on Image to 3D can replace `image_url` when the image came from a Meshy Text to Image task.

`nano-banana-pro` needs a Pro plan or above. On any plan, use `"ai_model":"gpt-image-2-5-flare"` with `"aspect_ratio":"3:4"` for step 1 at the same 9 credits: in the 2026-10-16 model comparison its `pose_mode` image was a clean A-pose and mesh-only Image to 3D kept the arms clear of the torso. That image was not taken through steps 2 and 3, which do not depend on the image model.

## The longer route (49 credits) also works

Mesh-only Image to 3D (20), Remesh to 30,000 faces (5), Retexture with the source image as `image_style_url` (10), then Rigging (5), plus the image (9). This is how the first rig test was run; Rigging accepts a Retexture task id.

Text to 3D can also remesh at generation (`should_remesh: true` on the preview gave 31,105 faces, kept through refine, rigged directly), but its `pose_mode` result is an arms-out, palms-up pose whose hands face unnaturally in the walk animation, so it is not usable for animated characters.

## Rig measurements

Angles measured from the joint positions in each rigged GLB, about 20,000 vertices each after remesh, 24 joints.

| Rig | Upper arm below horizontal | Elbow bend | Forearm or hand vertices weighted to torso | Vertices shared by arm and torso | Vertices weighted to both legs |
|---|---|---|---|---|---|
| Image route A, `pose_mode` image | 55 to 56° | 23 to 24° | 0 | 2.5% | 10.5% |
| Image route B, A-pose words in the image prompt | 52 to 53° | 28 to 29° | 0 | 1.0% | 7.3% |
| Image route C, both | 53 to 55° | 19 to 23° | 0 | 2.0% | 12.6% |
| One-task route (textured Image to 3D with remesh) | 56° | 26° | 0 | clean | — |
| Text to 3D with `pose_mode` (for comparison) | 46 to 47° | 75° | 0 | 0.37% | — |

No image-route rig has a forearm or hand fused to the body, which is the usual failure when arms touch the body in the source image. Some overlap at the shoulders is normal. The leg overlap comes from the long apron: a long apron or skirt is weighted to both legs and stretches when the legs separate, so ask for clothing that ends above the knee or splits between the legs.

## Not yet tested

Rigging a multi-part Smart Topology mesh; T-pose through this pipeline; characters with capes or held props.
