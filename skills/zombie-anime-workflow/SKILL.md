---
name: zombie-anime-workflow
description: Use as the master workflow when the user wants to produce an AI anime drama, especially Japanese-anime zombie/survival projects. Orchestrates the full pipeline by asking style/genre/production-route questions, confirming each next step, and routing work to subject/dialogue extraction, storyboard, video prompting, and Vidu generation skills without skipping approvals.
---

# Zombie Anime Workflow

Use this master skill whenever the user wants to make an AI drama from a script, novel, outline, storyboard, or material table.

These instructions are agent-portable. Do not assume a specific assistant runtime, local directory layout, or proprietary agent brand unless the user's environment explicitly provides it.

This skill does not replace the functional skills. It controls workflow order, asks the right questions, and prevents the agent from jumping directly into generation.

## Functional Skills

Route work to these skills when available:

- `zombie-subject-extraction`: subject extraction, subject table, and dubbing dialogue sheet.
- `zombie-storyboard-director`: script-to-storyboard table.
- `zombie-frame-grid-planning`: first-frame image prompt and grid storyboard image prompt planning.
- `zombie-video-prompting`: subject call planning and video prompt planning for multi-reference video, first-frame image-to-video, and grid-image video workflows.
- `zombie-vidu-generation`: Vidu task submission, polling, download, naming, and redo management.
- `zombie-shot-review-redo`: generated-shot review, issue classification, redo prompt rewriting, and redo batch planning.

If one of these skills is missing, continue with the closest available local skill and state the fallback briefly.

## Core Principle

Work step by step. Do not skip ahead.

Ask for confirmation before any step that creates/modifies files, submits paid/API generation, uploads materials, downloads/replaces outputs, or changes project state.

For read-only inspection, planning, schema validation, or draft recommendations, proceed without confirmation. If the user has already explicitly authorized a named step and scope, do not ask again inside the delegated functional skill; treat that confirmation as permission for that step only.

Before any real generation or file-producing batch, ask the user for confirmation. This includes:

- creating or modifying tables,
- generating images,
- generating first frames,
- generating grid images,
- submitting Vidu tasks,
- downloading/replacing videos,
- batch redo operations.

Before final video prompts and before any Vidu submission, require a confirmed `主体调用计划` from `zombie-video-prompting`. The plan must identify confirmed subjects, suspected subjects, excluded subjects, reasoning, and whether user confirmation is needed for each target shot.

For low-risk reading, inspecting, or planning, proceed without asking unless the user told you to pause.


## Shared Handoff Contract

Every functional skill must end with a compact handoff block so another agent can resume the project without guessing from chat history.

Use this handoff block whenever a step creates, updates, or validates production artifacts:

```text
交接摘要：
- project_name:
- project_folder:
- episode_or_chapter_range:
- production_route:
- created_or_updated_files:
- active_subject_workbook:
- active_storyboard_workbook:
- active_frame_grid_table:
- active_video_prompt_table:
- active_task_log:
- approved_parameters:
- missing_inputs_or_materials:
- next_recommended_skill:
- requires_user_confirmation_before_next_step: yes/no
```

Do not rely only on conversation memory when a project has multiple files or may resume later.

## Project Manifest

For any multi-step project, maintain a project manifest when a project folder is known. Prefer:

```text
<project_folder>/00_production_manifest.json
```

If JSON is inconvenient, use `00_production_manifest.md`. Minimum fields:

- project name,
- style,
- genre,
- production route,
- active chapter/episode range,
- active subject workbook path,
- active storyboard workbook path,
- active frame/grid planning path,
- active video prompt table path,
- active Vidu task log path,
- output video folder,
- model/API parameters,
- saved-subject/material mapping source,
- last completed step,
- pending user approvals,
- known missing materials.

Update the manifest after each completed production step. Do not overwrite user production files silently; create versioned files unless the user explicitly approves overwrite.

## Standard Artifact Names

Recommended project artifact structure:

```text
00_production_manifest.json
01_subjects/<project>_主体表_v001.xlsx
02_storyboard/<project>_分镜表_v001.xlsx
03_frame_grid/<project>_首帧宫格规划_v001.xlsx
04_video_prompts/<project>_视频提示词_v001.xlsx
05_vidu_logs/<project>_vidu任务日志_v001.xlsx
06_outputs/第N集/
07_review_redo/<project>_返工表_v001.xlsx
```

When updating an artifact, increment the version suffix unless the user explicitly asks to edit in place.

## Shot ID Contract

Use one stable `镜头号` value across storyboard, frame/grid planning, video prompt table, Vidu task log, and redo table.

Recommended formats:

- single episode/chapter sheet: `001`, `002`, `003`,
- multi-episode/chapter project: `E01-S001` or `第1集-001`.

Do not renumber existing shots after downstream files exist. If a shot is split, use suffixes such as `E01-S021A` and `E01-S021B`, and keep the original source shot in `备注`.

## Upstream Artifact Integrity

Do not silently mutate upstream artifacts. If a downstream step discovers an upstream issue, record it in `备注`, `missing_inputs_or_materials`, or a separate issue list, then ask before editing the upstream workbook.

Examples:

- missing subject in subject table,
- inconsistent character name,
- storyboard shot needs splitting,
- material token mismatch,
- scene continuity conflict.

## Initial Intake

When the workflow starts, ask concise guided questions before doing production work.

Ask question 1: AI drama style.

Offer options first, and allow the user to add custom details:

- `日系动漫风格`
- `国漫风格`
- `韩漫风格`
- `美漫风格`
- `写实电影感`
- `二次元手绘动画质感`
- `其它，我补充`

Ask question 2: story type / genre.

Offer options first, and allow the user to add custom details:

- `丧尸末日`
- `都市异能`
- `系统流`
- `校园`
- `玄幻/魔法`
- `悬疑惊悚`
- `恋爱/暧昧误会`
- `动作爽剧`
- `其它，我补充`

Ask question 3: video production route.

Offer exactly these primary routes:

- `多图参考生视频`
- `首帧图生视频`
- `宫格图生视频`

Then ask for known project inputs:

- script path or text,
- storyboard path if already available,
- material table path if already available,
- project output folder,
- target episode/chapter range,
- whether only planning is needed or generation is allowed later.

Do not ask all questions at once if the user is already mid-project. Ask only what is missing.

## Route Definitions

### 多图参考生视频

Use when the user wants Vidu reference-to-video using multiple saved subjects or raw reference images.

Required downstream steps:

1. Confirm subject/material availability.
2. Use `zombie-video-prompting` to build per-shot prompts with explicit subject references.
3. Ask before submitting Vidu.
4. Use `zombie-vidu-generation` only after confirmation.

### 首帧图生视频

Use when each shot should first get a still first-frame image, then become image-to-video.

Required downstream steps:

1. Confirm target image model/API before first-frame generation.
2. Use `zombie-frame-grid-planning` to write first-frame image prompts.
3. Ask before generating first-frame images.
4. After first frames are approved or selected, use `zombie-video-prompting` to write image-to-video dynamic prompts.
5. Ask before submitting video tasks with `zombie-vidu-generation`.

### 宫格图生视频

Use when one shot uses a grid image as a visual plan/reference.

Required downstream steps:

1. Ask the grid count: `4`, `6`, `9`, `12`, `16`, `24`, or `48`.
2. Use `zombie-frame-grid-planning` to write grid image prompts from the single-shot picture description and script context.
3. Ask before generating grid images.
4. After grid images are approved or selected, use `zombie-video-prompting` to write image-to-video or reference-to-video dynamic prompts.
5. Ask before submitting video tasks with `zombie-vidu-generation`.

## Workflow Order

Default order for a new project:

1. Intake: style, genre, production route, files, output folder.
2. Subject and dialogue extraction: ask before calling `zombie-subject-extraction`.
3. Storyboard: ask before calling `zombie-storyboard-director`.
4. First-frame/grid planning: ask before calling `zombie-frame-grid-planning` when the route requires first-frame or grid images.
5. Video prompting: ask before calling `zombie-video-prompting`; first produce and confirm the `主体调用计划`, then write final segmented time-coded prompts.
6. Vidu generation: ask before calling `zombie-vidu-generation`; proceed only after the subject call plan and final prompt rows are confirmed.
7. Review and redo: ask which shots need review, then use `zombie-shot-review-redo`; never redo a batch without confirmation.

If the user already has some artifacts, skip completed steps only after confirming they should be reused.

## Confirmation Pattern

Use direct, concrete confirmations. Avoid vague questions.

Good:

```text
下一步我准备调用 `zombie-storyboard-director`，把第 1-3 章转成分镜表格。确认后我会开始生成表格。是否继续？
```

Good:

```text
我已经整理好目标集数前 5 个镜头的 Vidu 提示词。下一步会提交 Vidu 生成任务，参数为 Q2、1080P、H264 Pro、16:9。是否开始提交？
```

Good:

```text
我已经整理好目标镜头的主体调用计划：确认调用主体、疑似应调用主体、排除主体和判断原因都列好了。确认后我再生成分段时间轴视频提示词。是否确认这份主体调用计划？
```

Bad:

```text
接下来怎么做？
```

Bad:

```text
我直接开始生成。
```

## Generation Gates

Never generate without explicit confirmation in these cases:

- first subject/dialogue workbook for a project,
- updated subject/dialogue workbook after major script changes,
- storyboard table export,
- confirming the subject call plan before final video prompt rows,
- image generation,
- Vidu task submission,
- video download/overwrite,
- redo batches,
- uploading materials or creating Vidu subjects,
- pushing skill or project updates to GitHub.

If the user says “开始”, “继续”, “确认”, or clearly tells you to generate the named batch, that counts as confirmation for that batch only.

Do not treat old confirmation as approval for later unrelated batches.

## Production State Tracking

Keep track of these decisions in the conversation and, when useful, in a local manifest:

- project name,
- project folder,
- style,
- genre,
- production route,
- target chapters/episodes,
- active subject table,
- active storyboard,
- active material table,
- video parameters,
- approved model/API,
- current step,
- pending questions,
- generated task IDs and output files.

When resuming a project, inspect existing files first, then summarize what you found before asking for the next confirmation.

## Safety And Quality Guardrails

- Preserve the script. Do not invent plot or dialogue.
- If the user requests a seductive or misunderstanding opening, keep it story-safe and avoid explicit sexual framing.
- Avoid gore details unless the user explicitly wants them and generation policy allows it; use impact, shadows, reaction, and non-graphic motion when possible.
- For video prompts, do not paste long policy disclaimers into the final prompt. Keep safety wording short and visual.
- Keep scene atmosphere based on script context, not fixed defaults. Do not carry weather, color, or mood from a previous project unless the script supports it.
- Keep characters from staring into camera unless the shot is POV, direct address, or intentionally camera-facing.

## Output Discipline

At the end of each step, report:

- what was created or updated,
- where the files are,
- what assumptions were used,
- what the next recommended step is,
- whether user confirmation is needed before continuing.

Keep final answers concise. Do not dump entire prompts or tables unless the user asks.
