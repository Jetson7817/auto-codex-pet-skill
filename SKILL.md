---
name: auto-codex-pet
description: Automatically create, iterate, validate, package, and install custom Codex pets from a character idea, reference image, or style brief. Use when the user asks to make a Codex pet, auto-generate a pet, create pet.json and spritesheet.webp, turn a character into a Codex pet, or produce a custom animated pet package for ~/.codex/pets. This skill orchestrates the installed hatch-pet and imagegen skills and adds practical QA rules for identity, logos, props, side views, and final installation.
---

# Auto Codex Pet

## Overview

Use this skill to turn a user concept into a working Codex custom pet package:

```text
${CODEX_HOME:-$HOME/.codex}/pets/<pet-id>/
  pet.json
  spritesheet.webp
```

This skill is an orchestration layer. Prefer reusing the installed `hatch-pet` skill and its scripts for pet-specific manifests, animation rows, atlas composition, validation, QA media, and packaging. Use `$imagegen` for visual generation.

## Trigger Examples

Use this skill for requests like:

- "帮我自动生成一个 Codex pet"
- "把这个角色做成 Codex 宠物"
- "生成 pet.json 和 spritesheet.webp"
- "做一个会动的 pet，放到 ~/.codex/pets"
- "按这个图片/设定孵化一个 Codex pet"
- "修复这个 pet 的动作行并重新打包"

If the user explicitly names `hatch-pet`, use `hatch-pet` directly. If they ask for a higher-level "auto" workflow or want the whole thing handled end-to-end, use this skill.

## Required Dependencies

Before starting, check that the following exist:

```bash
test -f "${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet/SKILL.md"
test -f "${CODEX_HOME:-$HOME/.codex}/skills/.system/imagegen/SKILL.md"
```

If `hatch-pet` is missing, install it from the official OpenAI skills repository before continuing, or tell the user it must be installed.

## Workflow

### 1. Clarify Only What Matters

Avoid over-questioning. If the user gives a clear character idea, proceed.

Infer reasonable defaults:

- `pet-name`: short friendly name, ASCII-safe id where possible.
- `style`: Codex digital pet style, pixel-art-adjacent, chibi, thick outline.
- `sensitivity`: avoid secrets, private repo names, client names, credentials, code, UI screenshots, and unrequested logos.

Ask only if the missing choice would materially affect the output, such as:

- exact pet name
- whether a specific trademark/logo must be visible
- whether to use an attached reference image

### 2. Prepare the Hatch Run

Use `hatch-pet`'s `prepare_pet_run.py`:

```bash
SKILL_DIR="${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet"
python "$SKILL_DIR/scripts/prepare_pet_run.py" \
  --pet-name "<Name>" \
  --description "<one short sentence>" \
  --pet-notes "<stable character description and constraints>" \
  --style-notes "Codex built-in digital pet style: small pixel-art-adjacent chibi mascot, chunky silhouette, thick dark 1-2 px outline, limited palette, flat cel shading, transparent-friendly chroma-key background." \
  --output-dir "${CODEX_HOME:-$HOME/.codex}/pet-runs/<pet-id>" \
  --force
```

For reference images, pass each `--reference /absolute/path/to/image`.

### 3. Generate And Record Base

Read the generated base prompt and use `$imagegen` built-in mode.

After generation, record the selected source:

```bash
python "$SKILL_DIR/scripts/record_imagegen_result.py" \
  --run-dir "<run-dir>" \
  --job-id base \
  --source "/absolute/path/to/generated_images/.../ig_*.png"
```

Show the base image to the user if the concept is subjective or branded. If the user dislikes it, regenerate the base before generating animation rows.

### 4. Generate Rows

Default row order:

```text
idle
running-right
running-left
waving
jumping
failed
waiting
running
review
```

Generate `idle` and `running-right` first. If `running-right` is visually symmetric enough, derive `running-left`:

```bash
python "$SKILL_DIR/scripts/derive_running_left_from_running_right.py" \
  --run-dir "<run-dir>" \
  --confirm-appropriate-mirror \
  --decision-note "<why mirroring is safe>"
```

If side-specific logos, readable text, one-sided props, or asymmetrical markings would become wrong when mirrored, generate `running-left` separately.

Record every generated row with `record_imagegen_result.py`.

### 5. QA Before Finalizing

Inspect row strips or the final contact sheet for:

- same character identity across rows
- no floating logos, text, badges, props, or detached effects
- side views hide chest logos when realistic
- crossed arms or hand-on-chin poses occlude chest marks instead of moving them
- no speed lines, shadows, detached sparkle/dust, visible grid, or text outside intentional logos
- full-body poses, no clipping, one pose per slot

For detailed QA guidance, read `references/pet-quality-rules.md`.

### 6. Finalize And Install

When all jobs are complete:

```bash
python "$SKILL_DIR/scripts/finalize_pet_run.py" --run-dir "<run-dir>"
```

Expected final outputs:

```text
<run-dir>/final/spritesheet.webp
<run-dir>/final/validation.json
<run-dir>/qa/contact-sheet.png
<run-dir>/qa/videos/
${CODEX_HOME:-$HOME/.codex}/pets/<pet-id>/pet.json
${CODEX_HOME:-$HOME/.codex}/pets/<pet-id>/spritesheet.webp
```

Validation must show:

```text
format: WEBP
mode: RGBA
width: 1536
height: 1872
errors: []
warnings: []
```

### 7. Report Back

Return concise results:

- pet package path
- contact sheet path
- validation status
- whether any rows were regenerated or mirrored
- instruction to restart Codex or run Force Reload to load the new pet

## Subagents

If subagents are allowed by the user and current tool policy, you may delegate row-strip generation exactly as `hatch-pet` prescribes.

If subagents fail or are unavailable, do not silently abandon the task. Explain briefly and continue sequentially only when the user has authorized continuing without subagents.

## Safety

Do not include secrets, private code, API keys, `.env` values, private client names, or private UI screenshots in prompts or generated pet details.

For living or public figures, prefer stylized, non-photorealistic, clearly illustrative pet art. Avoid defamatory, sexual, or demeaning portrayals.

For copyrighted characters, do not directly copy exact protected designs. Offer an original inspired alternative unless the user has rights or the request is otherwise clearly allowed.

