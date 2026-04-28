---
name: zombie-shot-review-redo
description: Use when reviewing generated AI anime drama videos or images, diagnosing bad shots, classifying Vidu/video-model failures, rewriting redo prompts, and preparing batch redo tables. Handles character fusion, wrong subject identity, missing subjects, camera-gaze errors, scene/weather mismatch, action mismatch, mouth/dialogue errors, style inconsistency, unsafe content, material-reference problems, and redo submission planning. Does not submit Vidu tasks unless paired with a generation skill and confirmed by the user.
---

# Zombie Shot Review Redo

Use this skill after videos/images have been generated, or whenever the user lists bad shots and wants them diagnosed, rewritten, or prepared for batch redo.

This skill reviews and plans. It does not submit generation tasks by itself. Use `zombie-vidu-generation` only after the user confirms a redo batch.

## Inputs To Read

Read available context:

- generated video/image paths, screenshots, or user feedback,
- original script section,
- storyboard table,
- prompt table,
- Vidu task log,
- material/subject table,
- scene continuity table if available,
- neighboring shots before and after the bad shot,
- current production parameters.

If a video/image is available locally, inspect it visually when possible. If only user feedback is provided, treat that as the primary evidence and do not contradict it without visual proof.

## Output Modes

For a quick diagnosis, output:

- shot number,
- issue category,
- likely cause,
- redo action,
- whether the prompt/material/route should change.

For batch redo, produce a table with:

- `镜头号`
- `原问题`
- `问题分类`
- `严重程度`
- `根因判断`
- `需要保留`
- `需要改变`
- `素材/主体检查`
- `重做策略`
- `重写提示词`
- `推荐参数`
- `是否建议拆镜`
- `备注`

Do not overwrite the original prompt table. Create a new redo table or new sheet.


## Redo Lineage Columns

For batch redo tables, include lineage fields when available:

- `原task_id`
- `原视频路径`
- `原提示词摘要`
- `返工轮次`
- `新生成方式`
- `新task_id`
- `返工状态`
- `诊断依据`
- `诊断置信度`

Recommended `返工状态` values:

- `待确认`
- `待提交`
- `已提交`
- `成功`
- `失败`
- `放弃`

Use `诊断置信度`: `高` / `中` / `低`.

## Redo Handoff To Generation

When handing redo rows to `zombie-vidu-generation`, provide:

- redo table path,
- selected redo shot IDs,
- generation route per shot,
- rewritten final prompt,
- negative constraints,
- required materials/images,
- parameter changes,
- original task IDs and video paths,
- output/log target,
- rows blocked by missing material.

The generation skill must treat redo rows like a new prompt table while preserving original task lineage.

## Visual Inspection Fallback

If local visual inspection is not available:

1. Use user feedback as authoritative evidence.
2. Compare feedback against prompt/storyboard/materials.
3. Mark diagnosis confidence as `高`, `中`, or `低`.
4. Do not claim unseen visual details.

## Issue Categories

Use these categories consistently:

- `角色身份错误`: wrong person, unfamiliar face, wrong costume, wrong form.
- `角色融合`: two characters merge, faces/bodies mix, one person becomes another.
- `主体缺失`: required character/monster/prop/UI/scene is missing.
- `主体数量错误`: too many or too few people/subjects.
- `空间关系错误`: wrong left/right, distance, foreground/background, blocking, or scale.
- `视线错误`: character looks at camera unintentionally, wrong gaze target.
- `动作错误`: action not performed, wrong action, walking when not needed, gesture unclear.
- `镜头错误`: wrong framing, no close-up, wrong camera movement, all-wide/all-static.
- `场景错误`: wrong location, wrong room, wrong weather, wrong time of day.
- `氛围/色调错误`: chapter/sequence mood inconsistent, unwanted cold/warm/rain/darkness.
- `道具/特效错误`: missing UI, weapon, prop, skill effect, blood bar, energy effect.
- `台词/口型错误`: wrong speaker, multiple mouths moving, mouth moving when no dialogue.
- `风格错误`: too realistic, 3D when anime wanted, wrong visual genre.
- `审核/安全风险`: explicit sexual/gore content, unsafe framing, policy-sensitive detail.
- `参数/技术错误`: wrong duration, resolution, codec, model, transition, aspect ratio.

## Severity

Use:

- `S1 必须重做`: wrong identity, missing required subject, severe fusion, core action wrong, scene totally wrong.
- `S2 建议重做`: action/camera/atmosphere undermines story but usable as temporary cut.
- `S3 可接受微调`: minor color, motion, or detail issue.

When user says “必须返工”, mark at least `S1` unless evidence suggests it is a parameter-only issue.

## Diagnosis Rules

Always compare:

1. script intent,
2. storyboard picture description,
3. original prompt,
4. material/reference mapping,
5. generated result or user feedback,
6. neighboring shot continuity.

Common root causes:

- missing saved subject token in prompt,
- token appears in prompt but material not passed,
- too many visible characters for the model,
- prompt used pronouns or vague roles,
- prompt described an abstract intent instead of visible action,
- scene/weather default carried from another chapter,
- prompt overloaded with constraints and buried main action,
- style/quality words changed color or subject design,
- duration too short for action,
- route choice was wrong: direct reference video instead of first-frame/grid, or vice versa.

Do not blame the model vaguely if the prompt/materials can be improved.

## Redo Strategy

Choose one or more:

- `提示词重写`: sharpen subject, action, gaze, camera, and ending frame.
- `素材补齐`: create/add missing character/prop/scene/effect reference.
- `主体映射修复`: ensure `[@name]` matches `--material "name:id:version"`.
- `减少主体`: cap important visible people at 1-4; split crowd shots.
- `拆镜`: split one complex shot into two or more simpler shots.
- `改路线`: use first-frame image, grid image, or raw image reference instead of direct reference-to-video.
- `改参数`: duration/model/transition/movement/codec/aspect ratio.
- `改构图`: close-up instead of wide shot, near-to-wide instead of full wide, over-shoulder, POV, low angle.

For multi-character fusion:

- restate the exact number of people,
- use saved subject tokens for every visible person,
- place each person in separate screen areas,
- avoid face overlap,
- give each person a distinct action and gaze,
- reduce to 2 people when possible,
- if still unstable, use first-frame or grid route.

For wrong gaze/camera staring:

- remove any direct-camera wording,
- specify exact gaze target,
- use side view, over-shoulder, back view, or profile,
- only keep camera-facing gaze for POV/direct-address/misunderstanding shots.

For walking-by-default issues:

- explicitly state stillness or non-walking action,
- write `双脚站定`, `身体保持原地`, `只转头/抬眼/举手`,
- make camera move instead of character walking.

For missing action:

- put the action in the first or second time beat,
- use concrete verbs,
- avoid abstract phrases such as `压迫感`, `情绪爆发` unless paired with visible motion.

## Redo Prompt Rules

Rewrite final video prompts in the same final-prompt format required by `zombie-video-prompting`:

```text
<style + scene + atmosphere>. 保持同一[@subject]、同一[@scene]，主体一致性强。0-X秒，<concrete visible beat>. X-Y秒，<concrete visible beat>. ... <quality words>.
```

Do not write labeled analysis sections in the final prompt unless the user asks.

Avoid:

- `承接上一镜头`,
- `继续上一镜头`,
- `建立起始构图`,
- `推动主要动作`,
- `看向镜头` unless intended,
- pronouns like `他/她/对方/主角/女主`,
- long policy disclaimers,
- irrelevant negative prompts inside the main prompt.

Put negative constraints in `备注` or a separate `负面约束` column unless the generation tool requires inline constraints.

## Material Check

For each redo shot:

- list required visible saved subjects,
- verify each has an image/material ID or local reference path,
- verify prompt tokens exactly match material names,
- mark missing/conflicting subjects before generation.

If a subject appears in the shot but has no usable material and reference generation is required, stop and ask the user for the material or permission to use text-only description.

## Continuity Check

Before final redo prompt:

- check scene time/weather/lighting from script, not previous failed video,
- check character costume/form for that episode,
- check prop state and location,
- check previous and next shot for spatial direction,
- keep atmosphere consistent within the same sequence.

Do not copy the failed video's wrong weather, color, character costume, or scene layout.

## Review Table Example

```text
镜头号: 21
原问题: 两个角色混合，只出现一个人
问题分类: 角色融合 / 主体数量错误
严重程度: S1 必须重做
根因判断: 双人同框提示中主体位置和动作区分不足，可能缺少一个角色的 material token
需要保留: 室内客厅、当前剧情情绪、原台词口型
需要改变: 改成双人分离构图，左/右站位明确，减少肢体重叠
重做策略: 提示词重写 + 主体映射修复；必要时改首帧图生视频
```

## User Feedback Handling

If the user lists bad shots, treat the list as authoritative. Example:

```text
21镜头两个角色混合，49背影不是赵宇，57离去背影不是赵宇。
```

For each shot:

- locate original prompt/log if available,
- classify issue,
- rewrite only the relevant shot prompt,
- preserve correct style/scene continuity,
- do not change unrelated shots.

If the user says “画面内容重新构思”, rewrite the shot from script/storyboard intent, not by tweaking the failed prompt.

## Pre-Redo Confirmation

Before submitting redo tasks:

1. Show redo shot list and count.
2. Show parameter plan.
3. Show missing material check.
4. Show output/log target.
5. Ask for explicit confirmation.

Then hand off to `zombie-vidu-generation`.

## Final Response

Summarize:

- shots reviewed,
- must-redo shots,
- prompt/material fixes,
- redo table path if created,
- whether user confirmation is needed before submission.

Keep the response concise; do not paste all rewritten prompts unless requested.
