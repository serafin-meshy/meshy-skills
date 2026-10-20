# Contributing

## Checking the repo

```bash
pip install pyyaml && python3 scripts/validate_skills.py
```

The script runs five checks: skill frontmatter, matching versions across the manifests, every `SKILL.md` and the top `CHANGELOG.md` entry, manifest coverage of every skill, links between `SKILL.md` and its reference files, and no paths that leave a skill's directory. A release bumps the version in `.claude-plugin/plugin.json`, `.cursor-plugin/plugin.json`, every `SKILL.md` and `CHANGELOG.md` together.

`scripts/check_prompt.py` is copied into each skill so every skill stays self-contained. Edit the root copy, then copy it into `skills/*/scripts/`; the five files must stay identical.

## Trigger evals

`evals/trigger/<skill>.json` holds 20 to 22 queries per skill: half should trigger the skill, half are near misses (a sibling skill's request, or a request to run a task rather than write a prompt). Rerun them after changing a skill's `description`. `evals/evals.json` holds four end-to-end prompts with assertions.

## Writing evidence

Every rule in a `SKILL.md` points to a run in `references/evidence.md`. These files are read by Meshy users and their agents, so record what a reader can use:

- the prompt, the settings, the number of runs and what came out;
- the credits and time per task, when they help someone plan;
- the Meshy docs page a limit or default comes from.

Leave out task ids, local file paths, web-app request details and test budgets. A run that shows a pricing or billing anomaly goes to the Meshy team, not into a skill.
