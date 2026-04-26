---
name: vidu-multi-reference-video
description: Use when creating concise Vidu reference-to-video prompts from multiple reference images or saved Vidu subjects, especially anime/drama shots with scenes, characters, props, effects, UI, dialogue, camera movement, multi-character staging, and moderation-sensitive action or romance content.
---

# Vidu Multi-Reference Video Prompt Skill

Use this skill to write stable, concise prompts for Vidu reference-to-video / `character2video` tasks that use multiple reference images or saved subjects.

Pair with the official Vidu CLI skill for actual submission, polling, and downloads.

## Current Vidu Mechanism

Vidu reference-to-video has two practical routes:

- Saved subject route: pass saved Vidu materials/subjects and refer to them in prompt text with `@name` or `[@name]`.
- Images-only route: pass 1-7 reference images directly. The model does not know which image is which unless the prompt maps them explicitly.

Core constraints to check before submission:

- `character2video` requires a non-empty text prompt.
- Total images + materials should stay within the supported reference count.
- Final prompt should be concise and visual. Keep camera, staging, and action as the main weight.

## Final Prompt Structure

Use this order:

1. `风格`
2. `审核安全表达` only when needed
3. `氛围`
4. `参考关系`
5. `场景与色调`
6. `镜头与机位`
7. `人物与空间关系`
8. `动作时间轴`
9. `台词/口型`
10. `约束`
11. `质量`

Important:

- Do not paste long internal rules into the final Vidu prompt.
- `审核安全表达` should be one short phrase only when needed.
- `氛围` should be compact, cinematic, and scene-specific.
- `约束` should be short and model-facing.
- Avoid repeated negative wording.

## Reference Mapping

For images-only tasks:

```text
参考关系：参考图1是室内场景图；参考图2是角色A的形象参考图（外貌描述）；参考图3是角色B的形象参考图（外貌描述）。
```

For saved Vidu subjects:

```text
参考关系：@室内客厅是室内客厅场景参考；@角色A是角色A的形象参考主体（外貌描述）；@角色B是角色B的形象参考主体（外貌描述）。
```

Character wording:

- `XX的形象参考图N（外貌描述）`
- `XX的形象参考主体（外貌描述）`

Do not write only `XX人设图`.

## Anti-Fusion Rules

Before generation, decide who is actually visible.

- Prefer 1-2 primary visible characters.
- Never exceed 4 named on-screen characters.
- Give each character a separate zone: left/right, front/back, seated/standing, near/far.
- Do not let faces overlap.
- For dialogue, state exactly who opens their mouth.
- Use silhouette, side-back view, or defocus for secondary characters.

Example:

```text
人物与空间关系：角色B坐在画面右下方墙边，抬头看向画面左侧；角色A站在画面左侧中景，身体侧对角色B，只露半身或侧背轮廓，二人相距一米以上，脸部不重叠，不看镜头。
台词/口型：无台词，两人都不张口。
```

## Camera And Motion

Every prompt should describe motion, not just a still image.

Specify:

- shot size,
- angle,
- camera position,
- camera movement,
- subject facing direction,
- gaze target,
- beginning and ending frame state.

Use a short timeline:

```text
动作时间轴：0-2秒，角色B低头喘息；2-5秒，她慢慢抬头，虚弱视线对上角色A，角色A保持静立观察。
```

Avoid:

- `承接上一镜`
- `延续上个视频`
- `和前面一样`

Restate visible scene state in every prompt.

## Indoor And Outdoor Weather

This is mostly an internal decision rule.

- Interior shots: do not describe rain indoors. Do not add rain sound, rain streaks, or outdoor rain views unless the shot looks outside.
- Interior shots during cloudy/rainy story periods: write `阴天室内冷色调`, `冷灰漫射光`, `低对比度`, `压抑安静`.
- Character wetness is character state only: `发梢湿润`, `衣物湿润`, `地面少量水迹`.
- Exterior shots or shots looking outside: use rain language when supported by the script.

In the final prompt, write only the resulting visual phrase, not this rule.

## Anime Drama Style

Useful style phrases:

- `日本动漫风格`
- `日系动漫风格`
- `日系题材动漫风格`
- `二次元手绘动画质感`
- `轻小说改编动画感`
- `动漫电影大片感`

Avoid:

- `赛璐璐`
- `真人写实`
- `欧美3D`
- living artist names

## Moderation-Safe Adaptation

Rewrite sensitive content into non-explicit visuals.

- Action/zombie: use threatening motion and atmosphere, not graphic gore.
- Bathroom/romance: use clothing, towels, steam, shadows, hands, faces, cutaways, and UI/event transitions.
- Keep final prompts focused on what should appear visually, not on policy explanations.

## Compact Prompt Example

```text
风格：日本动漫风格，日系动漫风格，日系题材动漫风格，二次元手绘动画质感，冷色调，动漫电影大片感。审核安全表达：画面克制，不展示裸露和血腥细节。氛围：阴天室内冷色调，虚弱、戒备、压抑安静。参考关系：@室内客厅是室内场景参考；@角色A是角色A的形象参考主体（黑色短发、制服、冷静警惕气质）；@角色B是角色B的形象参考主体（黑色长发、虚弱幸存者气质）。场景与色调：保持室内客厅冷灰光线，墙边阴影深，地面少量水迹。镜头与机位：中景，低机位从角色B身侧缓慢上摇到角色A方向。人物与空间关系：角色B坐在画面右下方墙边，抬头看向画面左侧；角色A站在画面左侧中景，身体侧对角色B，只露半身或侧背轮廓，二人相距一米以上，脸部不重叠，不看镜头。动作时间轴：0-2秒，角色B低头喘息；2-5秒，她慢慢抬头，虚弱视线对上角色A，角色A保持静立观察。台词/口型：无台词，两人都不张口。约束：不要真人写实，不要欧美3D，不要赛璐璐，不要额外人物，不要身份融合。质量：角色一致性强，空间关系清楚，构图稳定，冷色调统一，动作清晰。
```

