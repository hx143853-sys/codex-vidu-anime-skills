---
name: vidu-multi-reference-video
description: Use when designing Vidu reference-to-video prompts from multiple saved subjects or raw reference images for any scripted video project. Covers inline Vidu subject tokens, raw image mapping, script/storyboard context checks, scene atmosphere continuity, multi-character anti-fusion, camera motion, dialogue mouth-control, and moderation-safe concise prompting. Pair with vidu-skills for CLI submission.
---

# Vidu Multi-Reference Video Prompt Skill

Use this skill to write stable Vidu `character2video` / reference-to-video prompts that use multiple saved subjects or raw images. This skill is generic and must adapt to the current script, genre, episode, and scene.

## Required Context

Before writing a final prompt, read:

- the target script passage,
- the target storyboard row,
- nearby storyboard rows in the same location or scene,
- the material/subject table or available Vidu subject list.

Do not blindly trust the storyboard `出场角色` column. If the script clearly puts another character, monster, prop, weapon, skill, vehicle, or important object on screen, include it and reference it correctly.

## Current Vidu Mechanism

Vidu reference-to-video has two practical routes:

- Saved subject route: pass `--material "name:id:version"` and use exact inline token `[@name]` in the prompt body.
- Raw image route: pass one or more `--image` references and map them as `参考图1/参考图2/...` in text.

The total reference count for normal reference generation is usually 1-7 across images and materials. Follow `vidu-skills` for the final CLI parameters.

## Saved Subject Inline Tokens

Use saved Vidu subjects by default when they exist.

- Put `[@name]` directly in the body sentence wherever the subject appears.
- Do not only place subject names in a final tag list.
- Do not hide all subjects inside a long `参考关系` paragraph when the same token can be used in `场景`, `人物`, `道具`, `特效`, or `动作时间轴`.
- The token must exactly match the `--material` name.
- Use tokens for scenes, characters, monsters, props, weapons, vehicles, skills, effects, UI panels, and important objects.
- With subject tokens, skip long repeated appearance descriptions. Add only short state words when useful.

Good:

```text
场景：[@场景主体]，黄昏斜光，空气紧张。
人物：[@角色A] 位于画面左侧中景，侧脸看向 [@角色B]；[@角色B] 在画面右侧前景背对镜头。
道具：[@道具主体] 被 [@角色A] 握在右手，靠近胸前。
动作时间轴：0-2秒，[@角色A] 握紧 [@道具主体]；2-5秒，[@角色B] 缓慢后退。
```

Bad:

```text
提示词正文完全不写主体，只在最后附加：角色A 角色B 场景主体 道具主体
```

## Raw Image Mapping

Use raw image mapping only when a reference is not available as a Vidu subject.

```text
参考关系：参考图1是<场景/道具/角色>；参考图2是<角色名>的形象参考图2（简短外貌和服装）；参考图3是<特效名>参考图。
```

Continue referring to `参考图N` when assigning roles. Do not write only `参考图片` or `人设图`.

## Atmosphere Continuity

Atmosphere must come from script context.

- Determine the stable mood for a scene or short sequence before generating individual shots.
- Keep time, weather, lighting, and emotional tone consistent across adjacent shots unless the story changes.
- Do not force rain, cold tone, warm tone, darkness, fog, ruins, blood, or any genre marker unless the script supports it.
- Avoid prompt terms that alter character colors globally, especially `高饱和度`, `低饱和度`, `降低饱和度`, `提高饱和度`.
- Prefer concrete story atmosphere: `阴天`, `晴朗午后`, `夜晚路灯`, `黄昏斜光`, `室内顶灯`, `窗外自然光`, `压抑`, `紧张`, `短暂安全`, `危机逼近`, `暧昧误会`.
- Interior scenes should not mention rain inside. Only mention outside weather if it is visible or important in the script.

The final prompt should contain the chosen atmosphere phrase only, not the decision process.

## Prompt Structure

Use this order for production prompts:

1. `风格`
2. `氛围`
3. `场景`
4. `人物/主体`
5. `道具/特效`
6. `镜头与机位`
7. `人物与空间关系`
8. `动作时间轴`
9. `台词/口型`
10. `约束`
11. `质量`

Keep prompts concise and visual. Do not paste internal checklists, long policy disclaimers, or explanations into the final prompt.

## Multi-Subject Anti-Fusion

- Keep 1-2 primary visible characters whenever possible; avoid more than 4 named visible characters.
- Assign each character a separate screen zone and posture.
- State who faces whom, who looks where, and who does not look at camera.
- Avoid overlapping faces or bodies.
- If a character is only a reaction target, use back view, over-shoulder, silhouette, or partial body to reduce fusion.
- For dialogue, state exactly who opens their mouth; all other visible characters should stay silent unless needed.

## Camera And Motion

Every prompt must describe video movement, not just a still image.

Specify:

- shot size,
- camera angle,
- camera position,
- camera movement,
- subject facing direction,
- gaze target,
- start and end frame states.

Use a short time axis for dynamic shots:

```text
动作时间轴：0-2秒，<主体A动作>；2-5秒，镜头<推进/后撤/上摇/横移>，<主体B反应>。
```

## Dialogue Handling

- One shot should normally have one speaking character.
- Do not split a complete line awkwardly.
- If dialogue is long, increase duration or create same-scene alternate angles with natural sentence boundaries.
- Always write mouth-control: `只有 [@角色] 张口说话` or `无台词，所有角色不张口`.

## Moderation-Safe Language

Use safety rewrites only when needed and keep them short.

- Prefer concise phrases such as `画面克制，不展示血腥细节`.
- Avoid long policy-style text that weakens the visual prompt.
- Replace graphic gore with non-graphic action and reaction language.
- Replace explicit sexual wording with safe story framing such as `暧昧误会`, `脸红`, `眼神躲闪`, `镜头错位`.
- Do not use living artists or public figures as style references.

## Pre-Submit Checklist

Before calling `vidu-cli`:

- Script and storyboard were both read.
- Nearby scene context was used to determine atmosphere continuity.
- Visible subjects were resolved from script, not only from storyboard columns.
- Every saved subject is passed as `--material` and appears as exact `[@name]` in the prompt body.
- Every raw image is mapped as `参考图N`.
- Camera angle, gaze, spatial relation, motion, and mouth-control are explicit.
- No `承接上一镜` or previous-video-memory wording appears.
- No unsupported weather/color preset was copied from another project.
- No `高饱和度` or `低饱和度` wording appears.

## Output Format

```text
镜头：<episode>_<shot>
参考：subjects=[@主体1, @主体2]；images=[参考图1, 参考图2]（如有）
Vidu参数：type=character2video，model=<Q2/Q3/...>，duration=<seconds>，aspect_ratio=16:9，resolution=1080p，codec=h264
提示词：
风格：<项目风格>。
氛围：<由剧本和连续镜头确定的氛围>。
场景：[@场景主体]，<当前可见场景状态>。
人物/主体：[@角色A] <位置/朝向/表情>；[@角色B] <位置/朝向/表情>。
道具/特效：[@道具或特效主体] <与人物或动作关系>。
镜头与机位：<景别、角度、运镜>。
人物与空间关系：<前后左右远近、视线、是否看镜头>。
动作时间轴：0-2秒，<动作一>；2-5秒，<动作二>。
台词/口型：只有 [@说话角色] 张口说话：“<台词>”；其他角色不张口。
约束：不要真人写实，不要欧美3D，不要赛璐璐，不要额外人物，不要身份融合。
质量：主体一致性强，空间关系清楚，构图稳定，动作清晰，细节干净。
```

## Pairing With vidu-skills

Use `vidu-skills` for actual CLI submission, polling, cost checks, and download. This skill only governs prompt and reference design.
