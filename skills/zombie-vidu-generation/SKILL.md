---
name: zombie-vidu-generation
description: Use when submitting, querying, downloading, or managing Vidu video-generation tasks for AI anime drama production from prompt/storyboard tables. Handles Vidu reference-to-video, image-to-video, text-to-video, first-frame/video workflows, saved-subject material mapping, pre-submit confirmation, prompt provenance logging, batch task logs, status polling, downloads, failed-shot reporting, codec/resolution/model parameters, and safe retry planning. Requires vidu-skills/vidu-cli for actual API execution.
---

# Zombie Vidu Generation

Use this skill after `zombie-video-prompting`, or whenever the user asks to actually generate, query, download, retry, or manage Vidu video tasks for an AI drama.

This skill is the execution layer. For prompt writing, use `zombie-video-prompting`. For the actual Vidu CLI contract, defer to `vidu-skills` and its bundled references.

## Hard Gate

Before submitting any generation task, show the user a concise submit plan and wait for explicit confirmation.

The submit plan must include:

- shot range and count,
- generation route,
- model version mapping, such as Q2 -> `3.1`, Q3 -> `3.2`,
- aspect ratio,
- duration range,
- resolution,
- codec,
- transition mode if used,
- schedule mode if used,
- output/log path,
- visible subject/material names that will be sent,
- Vidu account subject check results: existing elements, newly created elements, missing subjects, and image requests,
- actual prompt text summary or full prompt text for the selected rows.

Do not submit if:

- the user has not explicitly confirmed,
- a required reference subject/material is missing,
- a required saved subject does not exist in the bound Vidu account and has not been created or replaced by a user-approved raw image,
- prompt contains a Vidu subject reference but no matching Vidu material mapping exists for final generation,
- combined `--image` + `--material` count is outside 1-7 for `character2video`,
- duration is invalid for the selected model,
- the user asks for a route but required inputs are missing.

For simulation, dry-run, or planning, do not submit. Produce commands, checks, or tables only.


## Pre-Submit Validation Report

Before asking for final submission confirmation, produce a validation summary:

- total rows selected,
- rows ready to submit,
- rows blocked,
- blocked reasons:
  - missing prompt,
  - saved subject not found in the bound Vidu account,
  - user image needed for subject creation,
  - missing material mapping,
  - token/material mismatch,
  - invalid duration for model,
  - invalid route inputs,
  - image/material count outside limits,
  - missing source image for `img2video`,
  - missing grid image for grid route.

Do not ask for submission confirmation until blocked rows are either removed from scope or fixed.

## Batch Blocking Policy

If some rows are blocked and others are valid:

- do not silently skip blocked rows,
- report blocked rows and reasons,
- ask whether to submit only valid rows or fix blocked rows first,
- never expand submission scope beyond confirmed valid rows.

## Task Log Provenance

Task logs should preserve enough information to reproduce each generation. Include these columns when possible:

- `source_prompt_table`
- `source_row_hash`
- `prompt_text`
- `prompt_file`
- `cli_prompt_arg`
- `cli_command_or_args`
- `vidu_cli_version`
- `material_mapping_source`
- `submitted_at`

`prompt_text` is the canonical prompt that was sent or intended to be sent. `prompt_file` records a temporary prompt-file path only if one was used. `cli_prompt_arg` records the exact CLI prompt argument form. Do not treat a file-indirection argument as user-visible prompt text.

If a redo task is submitted, preserve original task lineage fields from the redo table.

## Required Inputs

Read as much as available:

- prompt table from `zombie-video-prompting`,
- storyboard table,
- script context when scene/subject ambiguity exists,
- material/subject table or Vidu element list,
- bound Vidu account element list or `vidu-cli element list` access,
- project output folder,
- requested episode/chapter and shot range,
- requested naming pattern,
- requested model, aspect ratio, duration, resolution, codec, transition, schedule mode.

Expected prompt table columns:

- `镜头号`
- `时长`
- `生成方式`
- `场景`
- `出场角色/主体`
- `参考主体/素材`
- `视频提示词`
- `负面约束`
- `参数`
- `备注`

If the table has different column names, infer carefully and report the mapping before submission.

## Route Mapping

Use these Vidu task types:

- `多图参考生视频` -> `vidu-cli task submit --type character2video`
- `参考生视频` -> `character2video`
- `文生视频` -> `text2video`
- `首帧图生视频` / `单图转动态视频` -> `img2video`
- `首尾帧图生视频` -> `headtailimg2video`
- `宫格图生视频` -> usually `img2video` if the grid is a single image, or `character2video` if combined with saved subjects

For actual Vidu command syntax and parameter limits, read `vidu-skills` and `vidu-skills/references/parameters.md` as needed. Do not guess flags.

## Standard Video Parameters

Default only when the user has not specified:

- aspect ratio: `16:9`
- resolution: `1080p`
- codec: `h264_pro` for user-facing `H264_PRO`
- movement amplitude: `auto`
- sample count: `1`
- schedule mode: omit unless the user specifies `claw_pass` or `normal`; Vidu CLI can auto-detect

Model mapping:

- Q1 -> `3.0`
- Q2 -> `3.1`
- Q2 Pro -> `3.1_pro`
- Q3 -> `3.2`

Important limits:

- `character2video` with `3.1`: duration 2-8s.
- `character2video` with `3.2`: duration 1-16s and requires `--transition pro` or `--transition speed`.
- video resolution is `1080p`; 2K/4K is image-only for current CLI matrix.
- `character2video` requires total `--image` + `--material` count between 1 and 7.

Project phrase mapping:

- User says `H264_PRO`, `H264 Pro`, or `H264PRO`: use user-facing `H264_PRO` and CLI/API value `--codec h264_pro`. Do not downgrade this to `h264`.
- User says plain `H264`: use `--codec h264`.
- User says `电影大片`: prefer `--transition pro` for Q3/video routes where transition is allowed or required.
- User says `Q2`: use model `3.1` and do not pass `--transition` for `character2video`.
- User says `Q3`: use model `3.2` and pass `--transition pro` unless the user asks for speed.

## Material Mapping

For saved Vidu subjects:

1. Extract every required saved subject from `参考主体/素材` and every Vidu subject reference in the prompt.
2. Check the bound Vidu account for each subject before submission, using existing mappings or `vidu-cli element list --keyword "<name>"`.
3. Match each prompt subject reference to exactly one material mapping in the form `name:id:version`.
4. Pass the matching mapping via repeated `--material "name:id:version"`.
5. Keep the Vidu subject reference inline in the prompt body at the point where the subject appears.

Use the exact subject reference syntax required by the current Vidu tool. In the Vidu web UI this may be a blue `@真实元素名` chip; in current `vidu-cli` prompt text the documented plain-text form is `[@真实元素名]`, paired with `--material "真实元素名:id:version"`. Do not submit placeholder references such as `@主体名`, `[@name]`, or `[@角色A]` unless the saved element is literally named that.

If material names in the table are display names only, run:

```bash
vidu-cli element list --keyword "<name>"
```

If multiple elements match, stop and ask which one to use.

If no element matches, stop and report missing subjects. Ask the user to provide 1-3 reference images for each missing saved subject. After the user provides images and confirms creation, create the subject with:

```bash
vidu-cli element create --name "<name>" --image "<image1>" [--image "<image2>" --image "<image3>"]
```

Record the returned `id` and `version`, update the material mapping to `name:id:version`, and keep the real Vidu subject reference inline in the prompt. Do not silently remove the reference, replace it with plain text, or submit as text-only generation unless the user explicitly approves that fallback.

For raw images:

- pass each local path with repeated `--image "<path>"`,
- total `--image` + `--material` must stay within Vidu limits,
- for this drama workflow, prefer creating a reusable saved subject when the same character/scene/prop will recur across shots,
- do not upload or create persistent Vidu elements unless the user confirms the subject creation scope.

## Prompt Handling

Use the `视频提示词` exactly as the final prompt unless it violates execution rules.

Before submission, check:

- the prompt is a final segmented time-coded video prompt, not an unresolved analysis or planning note,
- no empty prompt,
- no unsupported audio-only instructions if video has no audio,
- no Vidu subject reference without material mapping,
- no saved subject appears only in a trailing reference list; required saved subjects should appear inline where they are visible or referenced,
- no obvious identity mismatch between `参考主体/素材` and prompt tokens.

Append negative constraints to the prompt only when the user explicitly wants them embedded. Otherwise leave `负面约束` in the log table and rely on the prompt itself.

Pass the actual prompt text directly to the Vidu CLI by default. Use a temporary prompt file only when command length or encoding makes direct text unsafe, and only after confirming the CLI supports prompt-file indirection. When a prompt file is used, logs must still preserve the full `prompt_text`, plus `prompt_file` and `cli_prompt_arg`.

User-facing submit plans, confirmation prompts, and final summaries must show the actual prompt text or a concise real-text summary. Do not display a local prompt-file argument as a substitute for the prompt.

## Submit Workflow

1. Read prompt rows for the requested shot range.
2. Normalize shot IDs and file names, such as `6_1`, `6_2`, or the user's naming convention.
3. Check the bound Vidu account for every required saved subject.
4. If a subject is missing, ask for 1-3 images, create the saved subject only after confirmation, and update the material mapping.
5. Build material mapping for each row.
6. Validate route, model, duration, resolution, aspect ratio, codec, transition, material count.
7. Decide direct prompt text versus supported prompt-file indirection, preserving `prompt_text` either way.
8. Present submit plan and wait for user confirmation.
9. Submit one row at a time. Do not hide partial failures.
10. Write a task log workbook/CSV after each successful submit.
11. Report task IDs and trace IDs.

Recommended task log columns:

- `镜头号`
- `文件名`
- `生成方式`
- `task_id`
- `trace_id`
- `state`
- `model`
- `duration`
- `aspect_ratio`
- `resolution`
- `codec`
- `transition`
- `schedule_mode`
- `materials`
- `vidu_account_subject_check`
- `created_elements`
- `prompt_text`
- `prompt_file`
- `cli_prompt_arg`
- `prompt`
- `submit_time`
- `last_query_time`
- `downloaded_files`
- `error_code`
- `error_message`
- `备注`

## Command Patterns

Reference-to-video with saved subjects:

```bash
vidu-cli task submit `
  --type character2video `
  --prompt "<final prompt with exact Vidu subject references>" `
  --material "name:id:version" `
  --duration 5 `
  --model-version 3.2 `
  --aspect-ratio 16:9 `
  --transition pro `
  --resolution 1080p `
  --codec h264_pro `
  --movement-amplitude auto
```

Q2 reference-to-video:

```bash
vidu-cli task submit `
  --type character2video `
  --prompt "<final prompt>" `
  --material "name:id:version" `
  --duration 5 `
  --model-version 3.1 `
  --aspect-ratio 16:9 `
  --resolution 1080p `
  --codec h264_pro `
  --movement-amplitude auto
```

Query:

```bash
vidu-cli task get <task_id>
```

Download when complete:

```bash
vidu-cli task get <task_id> --output "<download_dir>"
```

Always parse the single JSON stdout line. On CLI failure, copy `error.type`, `http_status`, `code`, and `message` exactly.

## Query And Download Workflow

When the user says `查询`, `查一下`, or asks why tasks are not complete:

1. Read the latest task log if available.
2. Query every non-terminal task, or the user-specified shot/task IDs.
3. Update `state`, `last_query_time`, errors, and any returned metadata.
4. Summarize counts: success / processing / failed / unknown.
5. If the user asks to download, download only `success` tasks unless they explicitly ask otherwise.

When downloading:

- create a stable folder such as `<project>/视频/第N章` or the user's requested folder,
- name files by shot/file name if possible,
- keep original downloaded file extension,
- write downloaded local paths into the task log,
- report failed or not-ready tasks separately.

## Failure And Retry Rules

For task submit CLI failures:

- do not infer causes,
- report exact JSON error fields,
- fix only local issues such as missing file path or invalid duration.

For Vidu task `state=failed`:

- report exact `err_code` and `err_msg`,
- do not automatically retry with changed prompts unless the user asks,
- suggest a retry plan only after identifying the likely prompt/material issue from observable evidence.

Common retry categories:

- subject fusion: reduce visible characters, make positions/gaze explicit, split into two shots.
- missing subject: create/list material first.
- overlong prompt: simplify non-motion details, keep time beats and tokens.
- duration/model mismatch: adjust duration or model.
- policy/review issue: rewrite visually, avoid explicit sexual/gore content, keep adult-safe and non-graphic.

## User Confirmation Patterns

Before submit, say something like:

```text
准备提交目标章节 1-5 镜头到 Vidu：
模型 Q2 -> 3.1，16:9，1080p，H264_PRO，character2video，多图参考生视频。
编码 H264_PRO -> CLI codec h264_pro。
已检查 Vidu 账号主体：<真实场景主体引用>、<真实角色主体引用> 均有对应 element。
预计提交 5 个任务，任务日志写入 ...
请确认“开始提交”后我再执行。
```

If the user already explicitly says `开始生成`, `开始提交`, or `我同意提交`, that counts as confirmation for the described scope. Still do not expand beyond the stated scope.

## Safety And Privacy

Content sent to Vidu includes prompts, images, and saved subject references. Do not upload or submit private materials unless the user has authorized that scope.

Do not expose API tokens, secrets, credentials, or environment variables in logs or final messages. This does not apply to safe Vidu saved-subject prompt tokens such as `[@角色A]`, which must stay visible when they are part of the prompt.

Do not include API keys in generated task logs, spreadsheets, GitHub commits, or skill files.

## Final Response

After submit, report:

- submitted shot count,
- task IDs,
- task log path,
- any newly created Vidu elements,
- any failures with exact error fields.

After query/download, report:

- success / processing / failed counts,
- downloaded file paths,
- task log path,
- any tasks needing user decision.
