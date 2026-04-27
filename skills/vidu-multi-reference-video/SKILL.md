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

- Build one normalized visible-subject list from the script, storyboard, and nearby scene context before writing the prompt.
- Subjects discovered from script context must be referenced the same way as subjects listed in the storyboard. If a saved Vidu subject exists, pass it as `--material` and write it as `[@name]` in the prompt body.
- Do not add a script-derived visible character, prop, monster, weapon, skill, vehicle, UI, or scene as plain text when a matching Vidu subject exists.
- If a visible subject has no saved subject and no raw image mapping, stop and ask before submission.
- Put `[@name]` directly in the body sentence wherever the subject appears.
- Do not only place subject names in a final tag list.
- Do not hide all subjects inside a long `参考关系` paragraph when the same token can be used in `场景`, `人物`, `道具`, `特效`, or `动作时间轴`.
- For saved Vidu subjects, do not write identity-introduction sentences such as `[@角色A] 是角色A的形象参考主体` or `[@场景主体] 是室内场景参考`. The blue subject token is already the identity anchor.
- The token must exactly match the `--material` name.
- Use tokens for scenes, characters, monsters, props, weapons, vehicles, skills, effects, UI panels, and important objects.
- With subject tokens, skip long repeated appearance descriptions. Add only short state words when useful.
- Once a subject has a `[@name]` token, use that exact token for every mention. Do not mix `[@name]` with the same subject written as ordinary text.
- Do not replace visible subjects with third-person pronouns or vague references such as `他`, `她`, `它`, `其`, `对方`, `这个人`, `那个人`, `她身上`, `他手里`. Repeat the exact subject token instead.

Good:

```text
画面主体：双人镜头，出镜人物为 [@角色A] 与 [@角色B]。
场景：[@场景主体]，黄昏斜光，空气紧张。
人物：[@角色A] 位于画面左侧中景，侧脸看向 [@角色B]；[@角色B] 在画面右侧前景背对镜头。
道具：[@道具主体] 被 [@角色A] 握在右手，靠近胸前。
动作时间轴：0-2秒，[@角色A] 握紧 [@道具主体]；2-5秒，[@角色B] 缓慢后退。
```

Bad:

```text
提示词正文完全不写主体，只在最后附加：角色A 角色B 场景主体 道具主体
参考关系：[@角色A] 是角色A的形象参考主体；[@角色B] 是角色B的形象参考主体。
人物：角色B站在旁边，[@角色A] 盯着她。
动作时间轴：0-2秒，[@角色A] 看着对方；2-5秒，他慢慢后退。
```

Good rewrite:

```text
画面主体：双人镜头，出镜人物为 [@角色A] 与 [@角色B]。
人物：[@角色A] 位于画面左侧中景，侧脸看向画面右侧的 [@角色B]；[@角色B] 位于画面右侧前景，身体侧对 [@角色A]。
动作时间轴：0-2秒，[@角色A] 看向 [@角色B]；2-5秒，[@角色B] 缓慢后退。
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
3. `画面主体数量`
4. `场景`
5. `人物/主体`
6. `道具/特效`
7. `镜头与机位`
8. `人物与空间关系`
9. `动作时间轴`
10. `台词/口型`
11. `质量`

Keep prompts concise and visual. Do not paste internal checklists, long policy disclaimers, or explanations into the final prompt.

## Internal Checks Are Not Prompt Text

The agent must enforce quality and safety checks while writing, but must not paste them into the final Vidu prompt by default.

- Do not add a `约束` section unless the user explicitly requests it or one short safety phrase is necessary.
- Do not output negative weather commands such as `不要下雨`. If rain is not visible or not supported by the script, omit rain language.
- Do not output long negative lists like `不要真人写实，不要欧美3D，不要额外人物，不要身份融合`.
- Do not explain who a saved subject is. Use `[@name]` directly in scene, character, prop, action, and gaze sentences.

If a check matters, express the positive visible result instead of a negative instruction.

## Visible Cast Count

State the visible people/creatures count near the front of every final prompt:

- `画面主体：单人镜头，出镜人物为 [@角色A]。`
- `画面主体：双人镜头，出镜人物为 [@角色A] 与 [@角色B]。`
- `画面主体：三人镜头，出镜人物为 [@角色A]、[@角色B]、[@角色C]。`
- `画面主体：一人一怪镜头，出镜主体为 [@角色A] 与 [@怪物主体]。`

Do not turn this line into a negative constraint such as `不要其他人`.

## Multi-Subject Anti-Fusion

- Keep 1-2 primary visible characters whenever possible; avoid more than 4 named visible characters.
- Assign each character a separate screen zone and posture.
- State who faces whom, who looks where, and who does not look at camera.
- Use direct subject-token relationships, such as `[@角色A] 看向 [@角色B]`, `[@角色A] 侧身面对 [@角色B]`, or `[@角色A] 背对镜头看向画面右侧`.
- Avoid overlapping faces or bodies.
- If a character is only a reaction target, use back view, over-shoulder, silhouette, or partial body to reduce fusion.
- For dialogue, state exactly who opens their mouth; all other visible characters should stay silent unless needed.

## Gaze And Camera Staring

Characters should not stare into the camera by default.

- Use camera-facing or direct eye contact only when the script requires POV, direct address, selfie/live-stream framing, a character looking through a peephole/scope, or intentional fourth-wall framing.
- Avoid `盯着镜头`, `看着镜头`, `直视镜头`, `正对镜头凝视` unless explicitly required.
- Prefer `[@角色A] 看向 [@角色B]`, `[@角色A] 看向 [@道具主体]`, `[@角色A] 看向画面左侧门口`, or `[@角色A] 视线越过镜头看向远处`.
- Do not write pronoun-based gaze like `角色A盯着她`; write the actual token relationship, for example `[@角色A] 看向 [@角色B]`.

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
- Subjects added from script reasoning are not plain text; they are also `[@name]` tokens or mapped raw images.
- No visible subject is replaced by `他/她/它/对方/这个人/那个人` in action, gaze, spatial, or dialogue-mouth descriptions.
- Every raw image is mapped as `参考图N`.
- Camera angle, gaze, spatial relation, motion, and mouth-control are explicit.
- No accidental `看镜头/盯镜头/直视镜头` wording appears unless the script explicitly requires it.
- No `承接上一镜` or previous-video-memory wording appears.
- No unsupported weather/color preset was copied from another project.
- No `高饱和度` or `低饱和度` wording appears.
- No `参考关系：[@name] 是...参考主体` identity-introduction sentence appears for saved subjects.
- No default `约束` section or long negative instruction list appears in the final prompt.
- No negative weather command such as `不要下雨` appears in the final prompt.
- A visible cast count line such as `单人镜头`, `双人镜头`, or `一人一怪镜头` appears near the front.

## Output Format

```text
镜头：<episode>_<shot>
参考：subjects=[@主体1, @主体2]；images=[参考图1, 参考图2]（如有）
Vidu参数：type=character2video，model=<Q2/Q3/...>，duration=<seconds>，aspect_ratio=16:9，resolution=1080p，codec=h264
提示词：
风格：<项目风格>。
氛围：<由剧本和连续镜头确定的氛围>。
画面主体：<单人/双人/三人/一人一怪镜头>，出镜主体为 [@主体A]、[@主体B]。
场景：[@场景主体]，<当前可见场景状态>。
人物/主体：[@角色A] <位置/朝向/表情>；[@角色B] <位置/朝向/表情>。
道具/特效：[@道具或特效主体] <与人物或动作关系>。
镜头与机位：<景别、角度、运镜>。
人物与空间关系：<[@角色A] 与 [@角色B] 的前后左右远近、视线关系；默认不看镜头>。
动作时间轴：0-2秒，<动作一>；2-5秒，<动作二>。
台词/口型：只有 [@说话角色] 张口说话：“<台词>”；其他角色不张口。
质量：主体一致性强，空间关系清楚，构图稳定，动作清晰，细节干净。
```

## Pairing With vidu-skills

Use `vidu-skills` for actual CLI submission, polling, cost checks, and download. This skill only governs prompt and reference design.
