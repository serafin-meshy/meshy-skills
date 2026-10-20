# Evidence: Text to 3D preview prompts

Live runs from 2026-10-07 on `POST https://api.meshy.ai/openapi/v2/text-to-3d` with `ai_model: "latest"` (Meshy 7.1) unless noted. Each comparison is one sample per arm unless a count is given, so the results show what a wording controls, not how often it succeeds.

## Formula versus bare prompt (helmet)

| Arm | Prompt | Result |
|---|---|---|
| Bare | "a knight's helmet" | Ornate fantasy helmet with carved scrollwork and a pointed crest, none of it requested; 1,255,344 faces |
| Formula | "A medieval knight's helmet, tall rounded dome with a narrow horizontal eye slit and a pointed hinged visor, riveted cheek guards, raised ridge along the top from front to back, small breathing holes on the visor, realistic proportions, single standalone object" | Dome, eye slit, pointed visor, ridge, rivets and breathing slots followed; dented surface and the hinge not produced; 368,016 faces |

20 credits each.

## What detailed previews dropped, misread or added

| Prompt asked for | Preview produced | Kind |
|---|---|---|
| Dented battle-worn surface (helmet) | Smooth surface | Dropped |
| Pointed hinged visor (helmet) | Pointed visor, no hinge | Dropped |
| Thick braided beard reaching the belt (dwarf) | Short beard plus a thick braid around the neck that reads as a rope collar | Misread |
| Same beard wording, second run | Long beard, no braid | Dropped |
| Knee-length apron split between the legs (dwarf) | Bib over separate trousers, nothing below the belt | Misread |
| Same apron wording, second run | Knee-length belted coat | Misread |
| Oversized head; rolled sleeves (dwarf) | Head in normal proportion; long flared sleeves | Dropped |
| Nothing | Belt pouches | Added |
| Nothing | Fur cuffs on the boots | Added |

The largest and most common shapes were followed; small surface detail and specific accessories were not. Dwarf prompt: "A stocky dwarf blacksmith, short broad body with an oversized head and large hands, thick braided beard reaching the belt, knee-length apron split between the legs over a belted tunic, rolled sleeves, heavy boots, stylized proportions, single standalone character". The first run used `pose_mode: "a-pose"`; the second wrote the A-pose in the prompt with no `pose_mode`.

## Pose: 18 character previews, no standard A-pose

A standard A-pose means arms straight, angled about 45 degrees down, clear of the body.

| Preview setup | Previews | Arms clear of the body | Standard A-pose | Pose seen |
|---|---|---|---|---|
| `pose_mode: "a-pose"`, `latest` | 6 | 6 | 0 | Arms spread, elbows bent, palms up |
| `pose_mode` and A-pose words in the prompt, `latest` | 5 | 5 | 0 | Arms spread, elbows bent, palms up |
| A-pose words only, `latest` | 6 | 3 | 0 | 3 arms spread with palms up; 2 hands on hips; 1 arms hanging at the sides |
| `pose_mode: "a-pose"`, `meshy-6` | 1 | 1 | 0 | Arms spread lower, palms up |
| Comparison: Text to Image with `pose_mode`, then Image to 3D | 3 | 3 | 3 | Standard A-pose |

`pose_mode` is applied (the same pose came back in 11 of 11 runs with it), but on Text to 3D it gives an arms-out, palms-up pose. The A-pose words used: "standing in an A-pose with arms straight and angled down about 45 degrees away from the body, hands open and clear of the body, legs slightly apart".

One arms-out preview was remeshed, retextured and rigged: 24 joints, clean weights, no forearm or hand vertex weighted to the torso. But the bind pose has the elbows bent 75° against 19 to 29° on image-route rigs, and in Meshy's walking animation the hands do not face naturally. So the Text to 3D pose rigs but is not usable for animated characters. Pose words alone are not usable either: hands on hips and arms at the sides fuse the arms to the torso in the mesh.

## Face counts of raw previews

| Preview | Faces (triangles) |
|---|---|
| Dwarf, A-pose in the prompt | 2,329,628 |
| Dwarf, `pose_mode: "a-pose"` | 1,325,402 |
| Helmet, two-word prompt | 1,255,344 |
| Helmet, detailed prompt | 368,016 |

All four exceed the 300,000-face limit the Rigging API sets for `input_task_id`. Counts were read from the GLB files; the task object does not report them.

## Remesh at generation

A preview with the dwarf prompt, `pose_mode: "a-pose"`, `should_remesh: true`, `target_polycount: 30000` came out at 31,105 faces. The refine kept 31,105 and rigging accepted the refine task directly: 24 joints, no forearm or hand vertex weighted to the torso. 35 credits (20 + 10 + 5) against 40 with a separate Remesh call. At 31,000 faces the mesh is smoother and the beard braids are softer, but the overall form holds.

## Task object gaps

The preview task object returns `consumed_credits`, `created_at`, `finished_at`, `id`, `mode`, `model_type`, `model_urls`, `name`, `progress`, `prompt`, `seed`, `started_at`, `status`, `task_error`, `texture_image_url`, `texture_prompt`, `texture_urls`, `thumbnail_url`, `type`. It does not return `ai_model`, `pose_mode`, `should_remesh` or a face count, so log request bodies on the client if you need to verify which model or pose ran.

## Sources

- [Text to 3D API](https://docs.meshy.ai/api/text-to-3d): `prompt` and `texture_prompt` max 800 characters; `target_polycount` 100 to 300,000 with `should_remesh` (default 30,000) and 100 to 15,000 with `smart-topology` (default 4,000); `pose_mode` values `a-pose` and `t-pose`; `lowpoly` retires 2026-10-30.
- [Prompting Best Practices](https://docs.meshy.ai/webapp/guides/prompting): "key information placed first carries the highest weight"; "stone and wood and metal and glass" listed as a bad prompt.
- [Rigging API](https://docs.meshy.ai/api/rigging): 300,000-face limit for `input_task_id`; textured humanoid models only.
- [Pricing](https://docs.meshy.ai/api/pricing).
