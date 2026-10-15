# Evidence: texture prompts

Live runs from 2026-10-07. Refines went to `POST /openapi/v2/text-to-3d` with `{"mode":"refine","preview_task_id":"<id>","texture_prompt":"<prompt>","enable_pbr":true}`; retextures went to `POST /openapi/v1/retexture` with `text_style_prompt` on a remeshed dwarf mesh. Results were judged from the 512-pixel thumbnails, so fine detail such as scratches could not be checked. One sample per arm unless a count is given. Output folders are under `api-testing/meshy_output/` in the author's workspace.

## Formula versus bare prompt (helmet refine, 10 credits each)

| Arm | Prompt | Refine task | Result |
|---|---|---|---|
| Bare | "iron helmet" | `01a11761-b982-7191-8bf3-2c232675b946` | Plain grey steel, light rust, rivets the same color as the shell |
| Formula | "Realistic medieval style. Dome and visor: dark hammered iron with fine scratches. Rivets: brass. Top ridge: polished steel. Cheek guards: dark iron with an engraved vine pattern along the edges. Moderate wear, light rust in the dents and around the rivets. Satin finish on the metal." | `01a11761-c27e-763b-84e2-2f1a5072a480` | Brass rivets, dark iron, rust patches, engraved vine pattern |

Folder `ab-test-20261007_111901/`. The skill's example replaces "Top ridge" with "The raised strip running over the top", which has not been tested.

## What the detailed refines followed and missed

| Prompt asked for | Refine produced | Task |
|---|---|---|
| Rivets: brass | Brass rivets | helmet `01a11761-c27e-…` |
| Top ridge: polished steel | Dark, same as the shell | helmet |
| Cheek guards: engraved vine pattern along the edges | Engraving beside the eye slit | helmet |
| Dome and visor: dark hammered iron | Dark blue-black metal; hammered texture not visible at thumbnail size | helmet |
| Face and hands: warm tan skin, rosy cheeks and nose | As asked | dwarf `01a117a0-d569-723d-a9bf-2cdb11b1abf1` |
| Beard and hair: grey with lighter streaks | As asked | dwarf |
| Tunic: faded red wool with a visible weave | As asked | dwarf |
| Apron, belt and boots: dark brown leather with scuffs and small burn marks on the apron | Belt and boots brown leather; apron charcoal grey | dwarf |
| Stylized hand-painted style | Smooth 3D-render look | dwarf |

Dwarf prompt: "Stylized hand-painted style. Face and hands: warm tan skin with rosy cheeks and nose. Beard and hair: grey with lighter streaks. Tunic: faded red wool with a visible weave. Apron, belt and boots: dark brown leather with scuffs and small burn marks on the apron. Moderate wear on the apron and boots. Matte skin and cloth, slight sheen on the leather." Folder `char-test-20261007_122822/`.

## The apron: color words and damage words (Retexture on remesh task `01a117b6-f4fc-75d4-a209-4074d0e946ce`, 10 credits each)

Only the apron slot changed between runs; the rest of the prompt was the dwarf prompt above with "Stylized game-art style" and the apron given its own slot.

| Apron slot | Apron color | Retexture task |
|---|---|---|
| Dark brown leather with scuffs and small burn marks | Charcoal grey | `01a117b7-bef3-7632-8e39-e76c0ef646f5` |
| Dark brown leather with scuffs | Dark grey-brown | `01a117bd-d9dd-71f7-8d52-85edaedc7a7e` |
| Brown leather | Muted grey-brown | `01a117bd-ddb3-77a8-b32f-e0de15f31412` |
| Saddle-brown leather (3 runs) | Warm brown in all 3 | `01a117bd-e0f4-7278-809e-4daa396949d3`, `01a117c1-1e2f-708e-92ec-0b96f8854426`, `01a117c1-1f58-73cb-9449-3bbc0a10ad16` |
| Saddle-brown leather with small burn marks | Warm brown, slightly redder | `01a117c1-2077-77a7-9e6a-b17fb4af0b1f` |

Generic color words drifted toward grey, and "small burn marks" pushed "dark brown" to charcoal. A specific hue held in 4 of 4 runs, including the run with burn marks. The apron came out charcoal whether grouped with the belt and boots or given its own slot, so the grouping was not the cause. Folders `apron-test-20261007_130119/` and `apron-followup-20261007_130454/`.

## Sources

- [Text to 3D API](https://docs.meshy.ai/api/text-to-3d): refine `texture_prompt` max 800 characters; `enable_pbr` default false; `texture_image_url` loses to `texture_prompt` when both are set.
- [Image to 3D API](https://docs.meshy.ai/api/image-to-3d): `texture_prompt` max 800 characters; `texture_prompt` wins over `texture_image_url`.
- [Retexture API](https://docs.meshy.ai/api/retexture): `text_style_prompt` max 800 characters; `image_style_url` wins over `text_style_prompt`; `multiview_image_urls` needs `ai_model: "meshy-7"`; `enable_pbr` default false; 8K costs 15 credits.
- [Prompting Best Practices](https://docs.meshy.ai/webapp/guides/prompting): "stone and wood and metal and glass" listed as too many materials.
- [AI Texturing guide](https://docs.meshy.ai/webapp/guides/3d-model/ai-texturing): troubleshooting advice to add material, era, wear and color descriptors.
- [UV Unwrap API](https://docs.meshy.ai/api/uv-unwrap): GLB input up to 40,000 faces, 5 credits.
