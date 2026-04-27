---
name: vidu-anime-drama
description: Use when generating Vidu anime/comic drama shots from any script, storyboard, and material/subject table. Enforces script-first context reading, scene/sequence atmosphere continuity, Vidu subject inline-token prompting, reference validation, camera/staging clarity, dialogue mouth-control, and safe concise prompts for reference-to-video generation.
---

# Vidu Anime Drama Shot Skill

Use this skill when turning a script plus storyboard/materials into Vidu reference-to-video shots. It must work for any series or genre, not one fixed project.

## Core Rules

- Default task type is `character2video`.
- Default model is `Q2 / 3.1` unless the user chooses another model.
- Minimum duration is `5s`; increase duration for long dialogue or complex movement.
- Default output is `16:9`, `1080p`, `h264`, no audio unless the user says otherwise.
- Use the style requested by the user or project. For anime projects, prefer `日系动漫风格`, `二次元手绘动画质感`. Do not use `赛璐璐`.
- Read the script and storyboard before generating. Never rely only on the storyboard row.
- Never write prompts as if Vidu has seen the previous generated clip. Do not use `承接上一镜`, `延续上个视频`, or `和前面一样`.
- If any visible character/monster/prop/weapon/skill/effect lacks a confirmed reference or accepted text-only design, stop and ask the user before submission.

## Script-First Context

Before writing any prompt:

1. Read the target storyboard row.
2. Read the matching script passage.
3. Read neighboring rows in the same scene or short sequence.
4. Resolve missing or vague storyboard fields from the script, especially `出场角色`, `众人`, `怪物`, `道具`, `技能`, and off-screen speaker references.
5. Decide what is actually visible in this shot and what is only implied or off-screen.

The storyboard is a guide, not the only source of truth. If the storyboard omits a character, prop, monster, or action that the script clearly requires on screen, include it and reference the correct subject.

## Sequence Atmosphere Lock

Build atmosphere from the script, not from fixed defaults.

- Determine the stable atmosphere for a continuous scene or short sequence of shots: time of day, weather, location state, emotional pressure, and lighting source.
- Keep that atmosphere consistent across adjacent shots in the same scene unless the script clearly changes it.
- Do not make one shot suddenly cool-toned and the next warm-toned without a story reason.
- Do not force rain, cold tone, blue tone, mist, darkness, blood, or ruins unless the script or scene state supports it.
- Avoid broad color-control words that distort subject colors, especially `高饱和度`, `低饱和度`, `降低饱和度`, `提高饱和度`.
- Prefer story-grounded atmosphere words: `阴天`, `晴朗午后`, `黄昏斜光`, `清晨薄雾`, `室内顶灯`, `窗外自然光`, `压抑`, `紧张`, `暧昧误会`, `肃杀`, `安全区短暂平静`, `危机逼近`.
- If the scene is indoors, do not describe weather occurring indoors. Only mention outdoor weather when the shot sees outside or the script makes it relevant.

Final prompts should include only the resulting visual atmosphere phrase, not the reasoning rule.

## Vidu Subject Inline Token Rule

Prefer saved Vidu subjects over raw images whenever they exist.

- Submit saved subjects with `--material "name:id:version"`.
- In the prompt body, use the exact inline token `[@name]` directly inside the sentence.
- Do not only place subject chips at the end or only list them in `参考关系`.
- Use subject tokens for scenes, characters, monsters, props, weapons, vehicles, skills, and effects.
- The token must exactly match the Vidu subject name passed in `--material`, including Chinese characters, spaces, punctuation, and full-width parentheses.
- For saved subjects, do not waste prompt weight on long appearance descriptions. The token carries identity. Add only short role/state clarifiers when needed.

Good body-embedded token patterns:

- `场景：[@场景主体]，黄昏斜光从窗外进入，房间安静压抑。`
- `人物：[@角色A] 位于画面左侧前景，侧脸看向画面右侧的 [@角色B]。`
- `道具：[@武器主体] 被 [@角色A] 双手握住，动作克制但紧张。`
- `特效：[@技能主体] 从地面升起，挡在 [@角色A] 与 [@怪物主体] 之间。`

## Raw Image Reference Rule

Use raw image mapping only when the reference is not a saved Vidu subject.

- Map every raw image near the front of the prompt: `参考图1是...；参考图2是...`.
- For character images, write: `XX的形象参考图N（简短外貌/服装/身份描述）`.
- Continue using `参考图N` later in the prompt when assigning scene, character, prop, or effect roles.
- Never write vague phrases like `参考图片` or `人设图` without mapping.

## Prompt Order

Use this order, but keep the final prompt concise:

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

Do not paste long internal rules, chapter explanations, or repeated warnings into the final Vidu prompt. Keep prompt weight on visible content, subjects, camera, spatial relation, and action.

## Camera And Staging

Every shot must state:

- shot size,
- camera angle,
- camera position,
- camera movement,
- facing direction,
- gaze target,
- left/right/foreground/background relation,
- beginning and ending action state.

Characters should not look at camera unless the story explicitly requires it. Multi-character shots should keep visible named subjects to 1-4, with 1-2 primary characters whenever possible.

## Multi-Subject Anti-Fusion

- Give each visible character a separate screen zone: left/right, foreground/background, seated/standing/kneeling, near/far.
- Avoid overlapping faces, merged bodies, or vague `两人站在一起`.
- For dialogue, state exactly who opens their mouth and who stays silent.
- For reaction shots, use over-shoulder, back view, silhouette, or partial body when that reduces identity fusion.

## Dialogue And Duration

- Each shot should normally have only one speaker.
- Do not split a complete line of dialogue in a way that breaks performance unless the user asks for micro-shots.
- If a line is long, extend duration or divide into same-scene alternate angles while keeping the full sentence segments natural.
- Match duration to action and dialogue length; do not force every shot to the minimum when the line needs more time.

## Moderation-Safe Prompting

Keep safety wording short and scene-specific.

- Use one compact phrase only when needed, such as `画面克制，不展示血腥细节`.
- Avoid long policy disclaimers that bury the main visual content.
- Rewrite graphic or sexualized wording into story-safe visual language without changing plot meaning.
- Do not mention living artists or real public figures as style references.

## Workflow

1. Read script, storyboard, material table, and available Vidu subjects.
2. Identify the current scene/sequence atmosphere from script context.
3. Resolve all visible subjects from script plus storyboard.
4. Verify references or subject tokens for every visible subject.
5. Prefer `--material` + inline `[@name]` for saved subjects; use `参考图N` only for raw images.
6. Write a concise prompt using the order above.
7. Check camera, gaze, spatial relation, motion, dialogue mouth-control, and atmosphere continuity.
8. Submit through `vidu-skills`, poll, download, and save with the requested naming format.

## Generic Prompt Skeleton

```text
风格：<项目指定风格>，日系动漫风格，二次元手绘动画质感。
氛围：<根据剧本和连续镜头确定的时间/天气/情绪/光线>。
场景：[@场景主体]，<只描述当前镜头真实可见的场景状态>。
人物/主体：[@角色A] 位于<位置>，<朝向/视线/状态>；[@角色B] 位于<位置>，<朝向/视线/状态>。
道具/特效：[@道具或特效主体] <与角色或场景的关系>。
镜头与机位：<景别>，<平视/俯视/仰视/过肩/主观视角>，<推拉摇移或静止>。
人物与空间关系：<谁在前景/后景/左/右/近/远，谁看向谁，不看镜头>。
动作时间轴：0-2秒，<动作一>；2-5秒，<动作二>。
台词/口型：只有 [@说话角色] 张口说话：“<台词>”；其他角色不张口。
约束：不要真人写实，不要欧美3D，不要赛璐璐，不要额外人物，不要身份融合。
质量：主体一致性强，空间关系清楚，构图稳定，动作清晰，细节干净。
```

## Stop And Ask

Stop before generating when:

- visible subjects cannot be matched to references,
- the storyboard and script conflict in a way that changes who appears,
- the scene atmosphere cannot be determined from script context,
- multiple candidate subject forms exist and the correct one is ambiguous,
- the request risks moderation failure and cannot be safely rewritten without changing meaning.
