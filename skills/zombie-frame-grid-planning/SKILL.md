---
name: zombie-frame-grid-planning
description: Use when preparing first-frame image prompts or grid storyboard image prompts for AI anime drama video workflows. Converts scripts, storyboard rows, subject/material tables, and scene context into still-image prompt tables for 首帧图生视频, 单图转动态视频, or 宫格图生视频 before dynamic video prompting or Vidu generation. Does not submit image/video tasks.
---

# Zombie Frame Grid Planning

Use this skill when the production route is `首帧图生视频`, `单图转动态视频`, or `宫格图生视频`.

This skill plans image prompts only. It does not call image APIs, Vidu, upload materials, or generate videos. Ask for confirmation before any real image generation.

## Inputs To Read

Read available context before writing image prompts:

- script or revised script section,
- storyboard table,
- neighboring shots in the same scene,
- subject/material table,
- Vidu saved-subject list if available,
- raw reference images if available,
- target style, genre, aspect ratio, and image model,
- target episode/chapter and shot range.

Do not rely only on the storyboard `出场角色` column. Infer visible characters, props, UI, monsters, vehicles, weapons, and effects from the script and画面描述.

If a required visible subject has no material/reference and the route requires references, stop and report the missing subject.

## Output Tables

For first-frame planning, use columns:

- `镜头号`
- `时长`
- `场景`
- `出场主体`
- `参考主体/素材`
- `首帧图提示词`
- `图像参数`
- `后续视频衔接要点`
- `缺失素材`
- `状态`
- `备注`

For grid planning, use columns:

- `镜头号`
- `时长`
- `宫格数`
- `场景`
- `出场主体`
- `参考主体/素材`
- `宫格图提示词`
- `分格说明`
- `后续视频衔接要点`
- `图像参数`
- `缺失素材`
- `状态`
- `备注`

Do not include task IDs, download paths, or completion status. Those belong to generation skills.

## Route Decision

Use `首帧图生视频` when:

- the shot needs strong character/scene consistency,
- one decisive frame can anchor the whole video,
- the action is simple enough to animate from a single image,
- the user wants image approval before video generation.

Use `宫格图生视频` when:

- the shot has several distinct action beats,
- the user wants a visual storyboard image for a long or complex shot,
- the shot has important pose, camera, or staging changes that one first frame cannot control,
- the user explicitly asks for 4/6/9/12/16/24/48 grids.

Use `多图参考生视频` instead when the user already has reliable Vidu saved subjects and does not need a first-frame/grid image.


## Status Column

Add `状态` to planning tables when the output will feed later production. Recommended values:

- `草稿`
- `待素材`
- `待用户确认`
- `可生成图片`
- `图片已生成`
- `可进入视频提示词`
- `不可用`

Rows marked `待素材` or `不可用` must list the missing material or reason in `缺失素材` / `备注`.

## Parseable Reference Format

In `参考主体/素材`, use a parseable list, not prose.

Preferred formats:

- saved subject: `<Vidu主体引用>=真实元素名:element_id:version`
- local image: `角色A=/absolute/path/to/character-a.png`
- missing: `角色A=缺失`
- text-only: `角色A=文本描述`

The later video-prompting and generation skills must be able to verify every visible subject from this cell.

Use the exact subject reference syntax required by the current Vidu tool. In the Vidu web UI this may be a blue `@真实元素名` chip; in current `vidu-cli` prompt text the documented plain-text form is `[@真实元素名]`, paired with `--material "真实元素名:id:version"`. Do not output placeholder names such as `@主体名`, `[@name]`, or `[@角色A]` in a real prompt unless the saved element is literally named that.

## Handoff To Video Prompting

For each approved/generated image, hand off:

- `镜头号`,
- `生成方式`,
- `首帧图路径` or `宫格图路径`,
- `宫格数` if applicable,
- source prompt row,
- approved image version/selection,
- visible subjects/materials,
- unresolved missing materials,
- recommended video prompt notes.

Do not assume that a planning prompt means the image was generated. Only mark `图片已生成` or `可进入视频提示词` when the selected image path is known or the user explicitly approved it.

## First-Frame Prompt Rules

A first-frame prompt describes one still image only. Do not include video timeline language such as `0-2秒`.

Do not design a first frame from one storyboard row alone when script/storyboard context is available. First understand the episode/scene geography, adjacent shots, character positions, and the real object or threat being watched. If a sparse row says a character watches outside, but the script establishes the target is downstairs at a store entrance, the first-frame prompt must show that line of sight and target position instead of inventing an interior threat.

The prompt must include:

- style and genre,
- exact scene and atmosphere from script context,
- visible subjects using exact Vidu inline subject references when saved subjects are available,
- framing and camera angle,
- subject position and gaze target,
- explicit spatial relationship: camera viewpoint, subject position, target position, foreground/midground/background, and line of sight,
- opening-frame logic: what the viewer sees before motion begins and why this frame supports the next video prompt,
- pose and key expression,
- important props/effects/UI,
- lighting and environment details,
- consistency lock,
- quality words,
- no text/subtitles/watermarks unless the shot is specifically UI text.

Choose the most useful first frame:

- For dialogue/reaction shots: start with the speaker or listener's emotionally strongest still frame.
- For reveal shots: start before the reveal if suspense matters; start at the revealed composition if consistency matters more.
- For action shots: choose the frame just before the main action, with clear body orientation and spatial relationship.
- For UI/system shots: choose the UI fully readable only if text display is the visual purpose; otherwise make the UI symbolic and readable enough without overloading.

Avoid:

- vague composition such as `一个人在房间里`,
- pure mood with no subject positions,
- saved subject tokens placed only at the end instead of inline where the subject appears,
- characters staring at camera unless POV/direct address is intended,
- audio-only wording such as `听到`, `声音先行`,
- future-motion wording such as `即将移动`, unless it is visible in pose.

## First-Frame Prompt Shape

Write as one continuous still-image prompt:

```text
日本动漫风格，日系末日惊悚题材动漫风格，白天室内狭窄房间，冷白灯光、潮湿地面、洗手台、半开的门清楚可见。保持同一[@场景A]、同一[@角色B]，主体一致性强。低机位近景，[@角色B]靠在画面右侧墙边，脸颊泛红又惊恐，双手撑住身侧，视线向下看向画面左下方危险位置，不看镜头，空间狭窄压迫。电影级构图，角色面部稳定，细节干净，高清锐利，无字幕，无水印。
```

## Grid Count Guidance

If the user does not provide grid count, recommend one and state the reason:

- `4宫格`: simple 5s reaction, reveal, or small action.
- `6宫格`: standard 8-12s shot with setup, reaction, action, result.
- `9宫格`: complex fight, chase, multi-character staging, or UI/action mixed shot.
- `12宫格`: long action sequence with several turns.
- `16宫格`: complicated montage or very detailed choreography.
- `24/48宫格`: rare; use for long sequences, music-video style montage, or when the user explicitly needs many keyframes.

Do not use large grid counts just to look impressive. More panels can make identities less stable.

## Grid Prompt Rules

A grid prompt describes one single composite storyboard image with multiple panels.

It must include:

- grid count and layout, such as `6宫格，2行3列`,
- reading order, usually left-to-right, top-to-bottom,
- same style, character identity, costume, scene, lighting, and color mood across panels,
- panel-by-panel visible action beats,
- clear spatial continuity,
- no text labels, no panel numbers, no subtitles, unless the user explicitly wants labels.

Panel prompts must be visual, not audio-based.

Bad:

```text
第1格听到声音，第2格准备切入，第3格推动动作。
```

Good:

```text
第1格，低机位看到房门缝隙和潮湿地面；第2格，[@角色B]靠在墙边，双手撑住身侧；第3格，镜头拉开揭示[@角色C（感染后）]蹲在前景危险位置；第4格，[@角色C（感染后）]抬头看向[@角色B]；第5格，[@角色A]从门口冲入；第6格，[@角色A]挡在[@角色B]前方。
```

## Grid Prompt Shape

Write as one continuous image prompt:

```text
6宫格动漫分镜图，2行3列，日本动漫风格，日系末日惊悚题材动漫风格，白天室内狭窄房间，保持同一[@场景A]、同一[@角色B]、同一[@角色C（感染后）]，角色服装、五官、场景光线一致。左到右、上到下阅读：第1格，低机位看到房门缝隙和潮湿地面；第2格，[@角色B]靠在墙边脸红惊恐；第3格，镜头拉开揭示[@角色C（感染后）]蹲在前景危险位置；第4格，[@角色C（感染后）]浑浊眼睛抬起看向[@角色B]；第5格，[@角色B]身体后缩双手撑地；第6格，危险距离压近。每格构图清楚，动作连续，无文字，无编号，无字幕，无水印，角色一致性强。
```

## Subject Token Rules

If a subject is a saved Vidu element, use the exact Vidu inline subject reference in the prompt and include it in `参考主体/素材`.

Place the subject reference where the subject is visually described. Do not write a normal prompt and append `参考主体：[@角色A]、[@场景A]` at the end. If the scene, character, prop, monster, or location is a saved subject, use its real Vidu reference directly in the sentence where it appears.

Do not write identity explanation sentences:

```text
[@角色B] 是角色B的参考主体。
```

Do not mix tokens with pronouns:

- avoid `他`, `她`, `对方`, `这个人`, `女主`, `主角`.
- use exact token/name every time ambiguity is possible.

If no saved subject exists, use the exact subject name as plain text and mark `缺失素材`.

## Continuity Rules

For adjacent shots in the same scene:

- keep scene time, lighting, weather, and emotional atmosphere consistent,
- do not introduce rain, cold tone, darkness, gore, or ruin unless script/reference supports it,
- do not change character costume/state unless the script says so,
- keep character positions logical from the previous/next shot.

For multi-character panels:

- keep at most 4 important characters per panel unless the user asks for a crowd,
- separate bodies and faces clearly,
- state left/right/foreground/background only where it prevents fusion,
- state exact gaze targets.

## Parameters

Record image parameters separately from the prompt:

- aspect ratio, usually `16:9`.
- image model/API requested by the user.
- resolution, if known.
- generation route, such as `首帧图`, `宫格图`.
- reference mode, such as saved subjects, raw images, or text-only.

If the user has not specified the image model/API, ask before real image generation. For planning-only tables, mark `待用户确认模型/API`.

## Pre-Generation Confirmation

Before generating any first-frame or grid image, show:

- shot range and count,
- route: first-frame or grid,
- image model/API,
- aspect ratio/resolution,
- subjects/materials to be sent,
- output folder,
- whether generation will cost credits/API calls.

Wait for explicit confirmation.

## Quality Checklist

Before finalizing each prompt, verify:

- it is a still-image prompt, not a video prompt,
- it has concrete visible composition,
- it includes style, scene, subject, pose, gaze, camera angle, atmosphere, and quality,
- tokens in prompt match `参考主体/素材`,
- no audio-only wording,
- no unwanted camera gaze,
- no identity explanation text,
- no fixed weather/color defaults from earlier scenes,
- no overlong prompt that buries the main image.

## Handoff

After first-frame/grid prompts are approved or generated:

- pass approved images and shot context to `zombie-video-prompting`,
- for video execution, pass final prompts and image paths to `zombie-vidu-generation`,
- never skip user confirmation before actual Vidu submission.
