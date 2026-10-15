---
name: meshy-text-to-3d-prompt
description: Write, review or fix the `prompt` for a Meshy Text to 3D preview task, the untextured mesh that the API (`POST /openapi/v2/text-to-3d`), the web app and the Meshy MCP all start from. Use this skill whenever the user wants a prompt for a 3D model from text, asks what to type into Meshy, pastes a Meshy prompt and asks why the mesh ignored details, came out ornate, has the arms fused to the body or has too many faces, or when another skill or tool (meshy-3d-generation, the Meshy MCP) is about to call Text to 3D and needs the prompt and parameters written well. It covers shape-only wording, which details move to parameters (pose_mode, target_polycount, should_remesh, smart-topology), and when to send a character down the image route instead. Not for texture or retexture prompts (meshy-texture-prompt), Text to Image prompts that feed Image to 3D (meshy-image-prompt), or Meshy Edit (meshy-edit-prompt); it writes the prompt and parameters, it does not call the API.
license: MIT
metadata:
  author: meshy-dev
  version: "0.1.0"
---

# Meshy Text to 3D preview prompts

The Text to 3D preview makes an untextured mesh, so its prompt describes shape and nothing else. Colors, materials and surface damage go into the refine step's `texture_prompt`; pose and face count go into parameters. The field is `prompt`, maximum 800 characters, and Meshy's prompting guide says the earliest words carry the most weight, so the subject comes first.

The rules below were tested live on 2026-10-07 against `ai_model: "latest"` (Meshy 7.1), one or a few samples per comparison. [references/evidence.md](references/evidence.md) has the runs, task ids and what each one showed.

## Workflow

1. **Pick the route first.** If the user wants to rig or animate a character, do not write a preview prompt: send them to the image route (`meshy-image-prompt`). In 18 character previews Text to 3D never produced a standard A-pose. `pose_mode: "a-pose"` gives an arms-out, palms-up pose in 11 of 11 runs; it rigs, but the hands face the wrong way in the walk animation. Props, statues, printable figures and anything that will not be animated stay on Text to 3D.
2. **Sort the idea into three piles.** Shape words go in the prompt. Colors, materials and damage go in a "texture ideas" note for the refine (`meshy-texture-prompt`). Pose, face budget and topology go in parameters (table below).
3. **Write the prompt with the formula**, then read it once as the model will: subject first, largest forms before small ones, at most about eight shape details.
4. **Check it.** Run `python3 scripts/check_prompt.py --kind preview "…"` (path relative to this skill's directory). It flags length, material and color words, damage words, pose words, "low-poly", and a missing closing phrase. Fix what it reports.
5. **Deliver prompt, parameters, texture ideas and assumptions** in the output format at the end. State which values you chose that the idea did not give.

## The formula

```
[Subject], [overall shape and proportions], [main parts, largest to smallest], [large surface relief], [shape style], single standalone object
```

| Slot | What goes in it | Example words |
|---|---|---|
| Subject | A specific noun | "medieval knight's helmet", not "helmet" |
| Shape and proportions | Silhouette words | tall, squat, tapered, rounded, oversized head, short broad body |
| Main parts | Large forms first, then smaller ones | dome, then visor, then cheek guards, then rivets |
| Large surface relief | Detail big enough to change the geometry | rivets, raised ridges, carved patterns, breathing holes |
| Shape style | How the forms are drawn | realistic proportions, stylized, chibi, faceted low-poly look |
| Closing phrase | Tells the model it is one object | "single standalone object" or "single standalone character" |

Tested examples:

- Prop: `A medieval knight's helmet, tall rounded dome with a narrow horizontal eye slit and a pointed hinged visor, riveted cheek guards, raised ridge along the top from front to back, small breathing holes on the visor, realistic proportions, single standalone object`
- Character (for a statue or print, not for rigging): `A stocky dwarf blacksmith, short broad body with an oversized head and large hands, thick braided beard reaching the belt, knee-length apron split between the legs over a belted tunic, rolled sleeves, heavy boots, stylized proportions, single standalone character`

The bare prompt "a knight's helmet" produced an ornate fantasy helmet with scrollwork and a crest that nobody asked for. The formula prompt produced the dome, eye slit, pointed visor, ridge, rivets and breathing slots. A short prompt is an invitation for the model to design; a formula prompt is a specification.

## Leave these out of the preview prompt

- **Colors and material words.** The preview has no texture, so "dark iron", "leather apron" and "brass rivets" add nothing to the mesh and crowd out shape words. Material words name the surface, not the shape: write "treasure chest", not "wooden treasure chest"; "apron", not "leather apron". Keep them for the texture ideas.
- **Small surface damage.** "Dented battle-worn surface" produced a smooth mesh. Dents, scratches and rust are texture work.
- **Pose.** See step 1. Writing "standing in an A-pose, arms 45 degrees from the body" without `pose_mode` gave hands on hips in one run and arms against the body in 3 of 6; with `pose_mode` the words changed nothing.
- **Face count and the word "low-poly".** "Low-poly" changes the look (faceted), not the count. Set the count with parameters.
- **Scenes, groups, dense repeats and thin parts.** Describe one object. Lattices, scaffolding, piles of small items, wires, strings, feathers and plumes are the common causes of failed generations in Meshy's error documentation. Pick the main object of a scene and say so in the assumptions.
- **Lighting, background, text and logos.** They mean nothing to a mesh generator.
- **Long detail lists.** The preview dropped details from longer lists (the hinge on the visor, the oversized head, the rolled sleeves) and invented others (belt pouches, fur boot cuffs). About eight shape details is the practical ceiling; put the ones that matter most earliest.

Two wording findings worth knowing: "thick braided beard reaching the belt" became a rope-like braid around the neck in one run and a plain long beard in another, so a detail that depends on reading two words together ("braided" + "beard") is at risk; and "knee-length apron split between the legs" became a bib over trousers, which happens to be good for rigging, so for characters prefer clothing that ends above the knee or splits between the legs.

## Set these with parameters, not words

| Want | Parameter | Notes |
|---|---|---|
| A-pose or T-pose | `pose_mode: "a-pose"` or `"t-pose"` | Applied consistently, but on Text to 3D the result is arms-out, palms-up, not a standard A-pose. Rig-ready characters take the image route. |
| A face budget on a standard model | `should_remesh: true`, `target_polycount: N` (100 to 300,000, default 30,000) | Default `should_remesh` is false on Meshy 6 and 7, which leaves previews at 368,000 to 2.3 million faces, above the rigging limit of 300,000. A preview at 30,000 kept its count through refine and rigged directly. |
| Clean game-ready topology at a set count | `model_type: "smart-topology"`, `ai_model: "meshy-t2"`, `target_polycount: N` (100 to 15,000, default 4,000) | A different generator, not a cleanup pass: softer detail, separate open parts. 5 credits instead of 20. Only when the user asks for low-poly, game-ready topology or gives a face budget; "for a game" alone is not a request for it. |
| Quad topology | `topology: "quad"` with `should_remesh: true` | Not available with `meshy-t2`. |
| Metal that reads as metal | `enable_pbr: true` on the refine | A refine setting, but decide it now so the texture prompt can rely on it. |

Credits: a preview is 20 on `latest` (Meshy 7.1) and `meshy-6`, 5 on `meshy-6-lite` and `meshy-t2`; the refine is 10 more. The task object does not echo `ai_model`, `pose_mode` or a face count, so record the request body yourself if you need to verify a run later.

## Output format

Give the user all four parts; an application can take the JSON, a person can read the lines.

```
Prompt: <the preview prompt, under 800 characters>
Parameters: {"mode":"preview","ai_model":"latest"}   plus pose_mode / should_remesh / target_polycount / model_type when they apply
Texture ideas: <colors, materials and damage moved out of the prompt, for the refine>
Assumptions: <values you chose that the idea did not give, such as style or proportions>
Note: <only when needed: "this character should go through Text to Image with pose_mode and Image to 3D so it can be rigged">
```

When the user's idea is a scene or group, say which object you kept. When the user asked for something the preview cannot do (a pose, a color, a specific face count by words), say where it went instead of silently dropping it.

## When the result comes back wrong

| Symptom | Likely cause | Fix |
|---|---|---|
| Ornate details nobody asked for | Prompt too short, model filled the gaps | Add the shape slots; name the parts you want |
| A named detail is missing | Too many details, or the detail is surface-level (dents, hinge) | Cut the list to the eight that matter, move surface detail to the texture prompt, regenerate (same 20 credits) |
| Part misread (braid around the neck, apron as a coat) | Two-word detail read as two things | Reword as one unambiguous noun phrase, or accept and fix at Edit |
| Arms against the body, hands on hips | Pose words without `pose_mode` | Rig-ready characters: image route. Otherwise set `pose_mode` and accept the arms-out pose |
| Over 300,000 faces, rigging refuses it | Default `should_remesh: false` | Regenerate with `should_remesh: true, target_polycount: 30000`, or Remesh (5 credits) |

## Resources

| File | Use it when |
|---|---|
| [references/evidence.md](references/evidence.md) | You need the test runs behind a rule: prompts, task ids, counts, what came out |
| [assets/meta-prompt-code.txt](assets/meta-prompt-code.txt) | The user is building an app that turns ideas into preview prompts with a language model; returns one line of JSON |
| [assets/meta-prompt-plain.txt](assets/meta-prompt-plain.txt) | Same, for a person who will read and paste the result |
| `scripts/check_prompt.py` | Lint a drafted prompt before handing it over: `python3 scripts/check_prompt.py --kind preview "<prompt>"` |

Related skills: `meshy-texture-prompt` for the refine's `texture_prompt`, `meshy-image-prompt` for the image route and rig-ready characters, `meshy-edit-prompt` for changing one part of a finished model, `meshy-3d-generation` to run the task.
