---
name: vidu-anime-drama
description: Use when generating Vidu anime-drama shots from storyboard sheets and material sheets. Handles character2video defaults, anime-style prompting, atmosphere continuity, explicit reference mapping, camera angle and gaze direction, multi-character spatial staging, context lookup across nearby shots, and the rule that every visible character must have a confirmed reference image.
---

# Vidu Anime Drama Shot Skill

Use this skill when the input is a script, storyboard table, material table, or exported character/scene library and the target output is Vidu reference-to-video shots.

Pair with an execution skill for actual Vidu CLI submission and downloads. This skill is for planning, validation, and prompt writing.

## Non-Negotiables

- Default task type: `character2video`.
- Default model: Q2 / `3.1`, unless the user requests another model.
- Minimum duration: `5s`.
- Default output: `16:9`, `1080p`, `h264`, no audio.
- Base style: `日系动漫风格`. Do not use `赛璐璐`.
- Every shot must state camera angle, camera position, character facing direction, gaze target, and spatial relationship.
- Characters should not look at camera unless the storyboard explicitly requires it.
- Multi-character shots should usually keep 1-2 primary characters and never exceed 4 named on-screen characters.
- Never write prompt text as if Vidu can see previous generated clips. Do not use `承接上一镜`, `延续上个视频`, or `和前面一样`.
- If any visible character lacks a confirmed reference image or saved Vidu subject, stop and ask the user before generation.

## Workflow

1. Read the target storyboard row.
2. Read nearby rows and the source script to resolve scene state, ambiguous group labels, and who is actually visible.
3. Match the storyboard scene to the material table scene reference.
4. Build the visible-subject list: characters, monsters, props, effects, UI, and scene.
5. Verify every visible character has a confirmed image or saved Vidu subject.
6. Determine whether the shot is interior or exterior before writing weather.
7. Write a concise, visual Vidu prompt with strong weight on scene content, camera, staging, and action.
8. Submit with the Vidu CLI skill, save task IDs, poll, and download using `<episode>_<shot>.mp4` naming.

## Reference Mapping

When using images directly, map each reference explicitly:

```text
参考关系：参考图1是室内场景图；参考图2是角色A的形象参考图（黑色短发、制服、冷静气质）；参考图3是角色B的形象参考图（长发、虚弱幸存者气质）。
```

When using saved Vidu subjects, identify both the subject token and the role:

```text
参考关系：@室内客厅是室内客厅场景参考；@角色A是角色A的形象参考主体（黑色短发、制服、冷静气质）；@角色B是角色B的形象参考主体（长发、虚弱幸存者气质）。
```

For character references, use this wording:

- `XX的形象参考图N（外貌描述）`
- `XX的形象参考主体（外貌描述）`

Do not write only `XX人设图`.

## Prompt Shape

Keep prompts concise and visual. The skill rules are mostly pre-submit checks, not text to paste into the prompt.

Recommended order:

1. `风格`
2. `审核安全表达` only when needed, one short phrase
3. `氛围`
4. `参考关系`
5. `场景与色调`
6. `镜头与机位`
7. `人物与空间关系`
8. `动作时间轴`
9. `台词/口型`
10. `约束`
11. `质量`

Keep a typical final prompt around 400-1200 Chinese characters. Avoid long policy paragraphs, repeated negative rules, and chapter-lock explanations in the final Vidu prompt.

## Indoor/Outdoor Weather Rule

This is an internal decision step.

- Interior shot (`内`): do not write that it is raining indoors. Do not add rain sound, window rain streaks, or outdoor rainy scenery unless the shot explicitly looks outside or the window/outdoor view is the subject.
- Interior shot during a cloudy/rainy story period: write only lighting and atmosphere, such as `阴天室内冷色调，冷灰漫射光，低对比度，压抑安静`.
- Character wetness is allowed only as character state, for example `发梢湿润，衣物湿润，地面少量水迹`.
- Exterior shot (`外`) or shot looking outside: use `下雨`, `雨幕`, `积水`, `雨声`, and `阴天` when supported by the script.
- Keep scene color consistent across shots; do not make only one character's shot suddenly much colder or bluer.

## Multi-Character Staging

- Place characters in separate screen zones: left/right, foreground/background, seated/standing, near/far.
- Avoid overlapping faces and bodies.
- If only one character is speaking, state that only that character opens their mouth.
- If a secondary character is only a reaction target, use side-back view, over-shoulder, silhouette, or defocus.

## Moderation-Safe Adaptation

Handle sensitive content internally and convert it into cinematic, non-explicit visuals.

- Zombie/action: use `惊悚`, `低吼`, `扑来`, `暗色污渍`, `丧尸化`; avoid graphic gore.
- Bathroom/romance/sexualized scenes: keep characters clothed or covered, use steam, silhouette, hands, face, props, and cutaways; avoid nudity or explicit contact.
- Do not include real public figures, living artists, private data, API keys, task IDs, or generated media URLs in prompts or public artifacts.

## Prompt Skeleton

```text
风格：日本动漫风格，日系动漫风格，日系题材动漫风格，二次元手绘动画质感，冷色调，动漫电影大片感。
审核安全表达：画面克制，不展示裸露和血腥细节。
氛围：阴天室内冷色调，压抑安静，末日生存感。
参考关系：@场景A是室内场景参考；@角色A是角色A的形象参考主体（外貌描述）；@角色B是角色B的形象参考主体（外貌描述）。
场景与色调：保持场景A的空间结构，冷灰光线，背景简洁虚化。
镜头与机位：中景，低机位从角色B身侧缓慢上摇到角色A方向。
人物与空间关系：角色B坐在画面右下方，抬头看向画面左侧；角色A站在画面左侧中景，二人相距一米以上，脸部不重叠，不看镜头。
动作时间轴：0-2秒，角色B低头喘息；2-5秒，角色B慢慢抬头，角色A静立观察。
台词/口型：无台词，两人都不张口。
约束：不要真人写实，不要欧美3D，不要赛璐璐，不要额外人物，不要身份融合。
质量：角色一致性强，空间关系清楚，构图稳定，冷色调统一，动作清晰。
```

