---
name: zombie-video-prompting
description: Use when converting AI anime drama storyboards, scripts, subject tables, material tables, first-frame images, grid storyboards, or shot lists into high-quality AI video dynamic prompts. Supports multi-image/reference-to-video, first-frame image-to-video, single-image-to-video, grid-image-to-video, and text-to-video workflows; builds a pre-generation subject call plan, then per-shot prompt tables with inline Vidu saved-subject/chip references, subject/material mapping, segmented time-coded motion, explicit spatial relationships, camera movement, atmosphere continuity, dialogue mouth-control, concise constraints, and quality checks. Does not submit Vidu tasks, upload materials, download videos, or generate images/videos.
---

# Zombie Video Prompting

Use this skill after `zombie-storyboard-director` for direct multi-reference video prompts. For `首帧图生视频` or `宫格图生视频`, use `zombie-frame-grid-planning` first to prepare or select the first-frame/grid image, then use this skill for the final video-motion prompt.

This skill writes prompts only. It does not call Vidu, Seedance, Kling, Runway, upload materials, generate first frames, download videos, or report task status.

The goal is not to repeat the storyboard. The goal is to turn storyboard information into clear video motion instructions that a video model can animate.

## Required Context

Read available context before writing prompts:

- storyboard table,
- script or revised script section,
- neighboring shots in the same scene,
- subject/material table,
- Vidu saved-subject list if available,
- raw reference images if available,
- first-frame image if available,
- grid storyboard image if available,
- project style and genre,
- target episode/chapter and shot range,
- target duration, aspect ratio, model, resolution, and encoding if already decided.

Do not rely only on the storyboard `出场角色` column. It may omit visible characters, props, monsters, UI, vehicles, weapons, scene elements, or effects that are clear from the script.

If the production route is truly unknown and affects the prompt structure, ask once:

- `多图参考生视频`
- `首帧图生视频`
- `单图转动态视频`
- `宫格图生视频`
- `纯文本生视频`

If the user asks for a simulation or draft prompt table, proceed without requiring real Vidu subject IDs and mark unverified materials in `备注`.

## Output Modes

If the user asks for a copy-paste prompt, output only the prompt text.

Before outputting final prompt text, produce a `主体调用计划` unless the user has already approved one for the exact shot range.

If the user asks for a table, use these columns unless the user specifies otherwise:

- `镜头号`
- `时长`
- `生成方式`
- `场景`
- `出场角色/主体`
- `参考主体/素材`
- `视频提示词`
- `负面约束`
- `参数`
- `状态`
- `备注`

Do not include Vidu task IDs, download paths, completion status, or generation logs. Those belong to the generation skill.


## Subject Call Plan Gate

Before writing final `视频提示词`, build a per-shot `主体调用计划` from the storyboard, script context, subject/material table, and available reference assets.

Use these fields for each shot:

- `镜头号`
- `确认调用主体`
- `疑似应调用主体`
- `排除主体`
- `判断原因`
- `是否需要用户确认`

Default behavior:

- Output the `主体调用计划` first and wait for user confirmation before finalizing `视频提示词`.
- Treat storyboard `出场角色` as evidence, not as the whole truth.
- Put subjects that clearly appear, speak, act, or are required as a visible scene/prop/effect reference in `确认调用主体`.
- Put subjects implied by gaze, dialogue target, framing, or action but not clearly confirmed in `疑似应调用主体`.
- Put mentioned-only, previous-shot-only, whole-cast, or subject-table-only entries in `排除主体`.
- Set `是否需要用户确认` to `是` whenever any required visual subject is ambiguous, missing material, or only implied by the description.
- Do not mark a row `可提交` until the subject call plan for that row is confirmed or the user explicitly accepts text-only simulation.

Example:

```text
输入：出场角色=小明；画面描述=小明看向小红说话。
主体调用计划：确认调用主体=小明；疑似应调用主体=小红；排除主体=无；判断原因=小明明确出场并说话，小红是视线/说话目标但未被明确写成可见画面；是否需要用户确认=是。
```

## Route Input Requirements

Before finalizing a prompt table, verify route-specific inputs:

- `多图参考生视频`: saved subjects or raw images exist for every required visible subject.
- `首帧图生视频`: approved first-frame image path exists for each shot, plus a motion prompt.
- `单图转动态视频`: one approved source image path exists for each shot.
- `宫格图生视频`: approved grid image path and known grid count/panel order exist.
- `纯文本生视频`: user has accepted text-only generation.

If inputs are missing, mark the row as `不可提交` in `状态` or `备注` and list missing items.

## Status Column

When producing a table, include `状态` unless the user asks for a minimal table.

Recommended values:

- `草稿`
- `待素材`
- `待首帧或宫格图`
- `待参数确认`
- `可提交`
- `不可提交`

The generation skill should submit only rows marked `可提交` or explicitly confirmed by the user.

## Parseable Reference Format

In `参考主体/素材`, use a parseable list, not prose.

Preferred formats:

- saved subject: `<Vidu主体引用>=真实元素名:element_id:version`
- local image: `角色A=/absolute/path/to/character-a.png`
- missing: `角色A=缺失`
- text-only: `角色A=文本描述`

Every Vidu subject reference used in `视频提示词` must appear in `参考主体/素材`. If a subject reference appears in the prompt but no matching material mapping exists, mark the row `不可提交`.

Use the exact subject reference syntax required by the current Vidu tool. In the Vidu web UI this may be a blue `@真实元素名` chip; in current `vidu-cli` prompt text the documented plain-text form is `[@真实元素名]`, paired with `--material "真实元素名:id:version"`. Do not output placeholder names such as `@主体名`, `[@name]`, or `[@角色A]` in a real prompt unless the saved element is literally named that.

## Handoff To Generation

End prompt-table work with:

- prompt table path,
- selected shot range,
- row counts by `状态`,
- production route per row,
- subject call plan confirmation status,
- parameter assumptions,
- missing materials or source images,
- next recommended skill.

Do not include task IDs, download paths, or generation status in this skill.

## Dynamic Prompt Principle

Every final video prompt should answer:

1. What is the dramatic purpose of this shot?
2. What is visible at the first frame?
3. What changes during the shot?
4. What should the last frame look like?
5. What must stay consistent?

A good video prompt contains motion, not just a beautiful still image. Before finalizing, make sure the prompt includes:

- subject movement,
- expression or detail movement,
- environmental/effect movement when appropriate,
- camera movement,
- beginning and ending state,
- concise quality words.

## Prompt Structure

Write every final `视频提示词` as a segmented time-coded dynamic prompt, not as a labeled analysis block.

The required structure is:

```text
<style + genre + scene mood + visible environment with inline saved scene token when available>. 保持同一<saved subject tokens / subject names and key identity anchors>，主体一致性强。
0-X秒：<concrete visible first beat with exact framing, subject position, gaze, action, camera movement, environment/effect motion, and beat ending state>.
X-Y秒：<concrete visible second beat with motion change, camera path, subject action/reaction, and beat ending state>.
<final quality words>.
```

Do not output analysis labels inside the final prompt unless the user explicitly asks for an analysis format. The model-ready prompt should contain only the opening style/scene sentence, the consistency sentence, time-coded dynamic beats, and concise quality words.

The final prompt must be directly usable as a video-model prompt. It should read like the user's example:

```text
3D动漫电影风格，科幻机甲大战怪兽，黄昏末日[@城市废墟]里烟尘压低，压迫感强。保持同一[@银黑色巨型未来机甲]、同一[@白发少女驾驶员]、同一[@黑色红光巨型怪兽]，主体一致性强。
0-2秒：超远景从废墟街道低机位缓慢推进，[@银黑色巨型未来机甲]站在画面左侧前景，[@黑色红光巨型怪兽]压在右侧楼体残骸后方，双方隔着碎裂街道对峙，烟尘向两侧翻滚，结束时两个主体都停在同一条街道轴线上。
2-4秒：镜头切入驾驶舱特写，[@白发少女驾驶员]坐在驾驶座中央，双手握紧操纵杆，抬眼看向前方红色警报光和远处怪兽投影，镜头轻微震动，结束时[@白发少女驾驶员]的眼神收紧。
电影级构图，动态镜头，主体一致性强，动作连贯，光影层次清晰。
```

Rules for this segmented prompt:

1. Start with style, genre, time, mood, location, and visible environment in one sentence.
2. Add a consistency lock sentence: `保持同一...` using inline saved-subject tokens when available.
3. Use time-coded beats as separate lines: `0-1秒：`, `1-3秒：`, `3-5秒：`, etc.
4. Every time beat must describe a concrete visible image, subject, camera move, action change, and ending state. Do not write abstract placeholders like `建立起始构图`, `推动主要动作`, `停在结果反应`, `画面准备切入`, or `当前信息压力`.
5. Include exact framing and camera language inside the beat itself: `低机位近景`, `中景缓慢后拉`, `浅景深特写推近`, `快速横移`, `门缝视角`.
6. Include subject position and gaze target inside the beat: left/right/front/back/foreground/background only when useful, and exact targets like `[@角色B]看向[@角色C（感染后）]`.
7. If the storyboard uses sound-first or black-screen narration, convert it into a visible silent-video equivalent. Do not rely on hearing sound. Use visual suspense such as dark frame fade-in, trembling hand, mouth opening, door crack, shadow movement, subtitle-free panic expression, or a silent scream.
8. Do not include full dialogue unless the user needs lip-sync. If dialogue exists, describe visible mouth movement and speaker only: `[@角色B]嘴唇颤抖地说话，非说话角色不张口`.
9. Put negative constraints in the `负面约束` column, not inside the final prompt, unless visually necessary.

Do not append saved subjects as a loose list at the end of the prompt. A Vidu saved subject must appear inline exactly where that subject is visible or referenced in the action. For example, write `[@角色A]半蹲在[@卧室场景]窗边，透过窗户看向远处[@便利店门口]` instead of writing a normal prompt and then adding `参考主体：[@角色A]、[@卧室场景]、[@便利店门口]`.

Keep prompt length practical:

- simple 5-second shot: about 120-220 Chinese characters,
- complex action shot: about 250-450 Chinese characters,
- 12-15 second grid/sequence shot: about 450-800 Chinese characters.

If too long, remove repeated style words and still-image clothing details that are already anchored by references. Keep concrete visible beats, camera movement, ending state, subject tokens, and continuity.

## Timing Rules

Use a timeline only when it improves clarity.

For a 5-second shot:

```text
0-1秒：<concrete first visible frame with framing and subject position>.
1-3秒：<specific movement, expression, gaze, camera, environment motion>.
3-5秒：<specific ending image and motion result>.
```

For an 8-second shot:

```text
0-2秒：<visible starting frame>.
2-5秒：<main visible action>.
5-8秒：<ending composition and emotional/action result>.
```

For a 12-second shot:

```text
0-2秒：<visible starting frame>.
2-5秒：<first action>.
5-8秒：<action escalation or reaction>.
8-10秒：<climax>.
10-12秒：<aftermath or final frame>.
```

Do not mechanically split simple shots into many tiny beats.

## Production Routes

### 多图参考生视频

Use when the shot should be generated from multiple reference images or Vidu saved subjects.

Prompt must include:

- style and script-supported atmosphere,
- inline saved-subject tokens when available,
- scene, character, prop, monster, weapon, UI, and effect subjects as needed,
- action and expression,
- spatial relation only where it prevents confusion,
- gaze target,
- camera movement,
- start/end state,
- concise quality words.

### 首帧图生视频 / 单图转动态视频

Use when the first frame or single reference image already exists.

Do not redesign the image. Animate it.

Prompt must include:

- `保持首帧构图、角色外观、服装、道具、场景、光影一致`,
- first visible movement,
- subject expression or pose change,
- environmental/effect movement,
- camera movement,
- final frame state.

### 宫格图生视频

Use when the user provides a grid storyboard image.

Connect grid panels into a continuous sequence. Do not describe a static collage.

Rules:

- Treat each panel as a key beat.
- Preserve the same characters, scene, lighting, clothing, props, and visual logic across beats.
- Convert panel order into time order.
- Avoid saying `六宫格画面` unless the target tool specifically needs it.
- Ask for grid count only if it is not visible or not provided.

For a 6-panel, 12-second shot:

```text
0-2秒：第1格建立场景；2-4秒：第2格角色/主体特写；4-6秒：第3格动作启动；6-8秒：第4格对手/反应；8-10秒：第5格冲突高潮；10-12秒：第6格余波定格。
```

### 纯文本生视频

Use when no visual reference exists.

Add enough visual design for the model to understand the subject, but avoid overloading. Focus on character/subject design, environment, action, camera, mood, and quality.

## Storyboard Conversion Rules

For each shot, derive the video prompt from the script and storyboard:

- `镜头目的`: environment setup, suspense, reaction, action escalation, emotional turn, reveal, aftermath.
- `主体优先级`: main visual subject versus background/reaction subjects.
- `空间关系`: camera position, subject position, target position, foreground/midground/background, distance, occlusion, and line of sight.
- `起始构图`: subject placement, framing, pose, and gaze target.
- `动作路径`: where the subject moves from/to, speed, force, and emotional change.
- `镜头路径`: camera follows, pushes, pulls, pans, tilts, tracks, or holds.
- `结束构图`: close-up, back view, standoff, object reveal, door, UI result, monster threat, or emotional freeze.
- `连续性`: adjacent shots must keep time, weather, scene state, character state, and atmosphere consistent.

Do not invent new plot, new dialogue, or new visible subjects unless the user asks for creative rewriting.

## Vidu Saved-Subject Rules

If a subject is a Vidu saved subject, both conditions must be true in the later generation step:

- CLI/API material includes the subject, such as `--material "name:id:version"`.
- Prompt text uses the exact Vidu inline subject reference in the body.

In this skill, write the prompt body with Vidu subject references whenever saved subjects are known or assumed. Use the exact reference format exposed by the current Vidu tool: Vidu UI may render a blue `@真实元素名` chip, while current `vidu-cli` documentation uses `[@真实元素名]` in plain text. The skill examples use placeholders only; real outputs must use actual saved element names from the bound Vidu account.

Use inline subject tokens every time a saved subject is materially involved in the shot: the location/scene token, observing character, observed target, monster, prop, vehicle, weapon, or important environment anchor. Do not put tokens only in the consistency sentence or at the end of the prompt.

Correct:

```text
[@角色B]站在[@场景A]门口，双手抓紧外套边缘；[@角色A]站在室内阴影中，侧身看向[@角色B]。
```

Correct:

```text
0-2秒：门后视角从[@卧室场景]门缝缓慢推进，[@角色A]半蹲在窗户右侧阴影里，身体压低，侧脸贴近窗帘边缘，视线穿过窗户看向楼下远处的[@便利店门口]。
```

Wrong:

```text
角色B站在场景A门口，她看着角色A。
```

Wrong:

```text
角色A半蹲在窗边看向远处便利店门口。参考主体：[@角色A]、[@卧室场景]、[@便利店门口]。
```

Do not write identity explanation sentences:

```text
[@角色B] 是角色B的形象参考主体，[@场景A] 是场景参考。
```

The token itself is the identity anchor.

## Subject And Pronoun Rules

Build the confirmed visible subject list from the approved `主体调用计划`, storyboard, script context, and material table before writing prompts.

Once a subject has a saved-subject token, use that token consistently in the prompt.

Avoid replacing visible subjects with vague words:

- `他`
- `她`
- `它`
- `对方`
- `这个人`
- `那个女人`
- `男主`
- `女主`
- `关键角色`
- `主角`
- `怪物`
- `那个东西`

Use exact names or tokens:

- `[@角色A]`
- `[@角色B]`
- `[@角色C（感染后）]`
- `[@场景A]`
- `[@道具A]`

If no saved-subject token exists, use the exact subject name consistently as plain text.

## Gaze And Camera Rules

Do not make characters look at the camera unless the script clearly asks for POV, direct address, livestream framing, or intentional camera-facing misunderstanding.

Avoid by default:

- `看向镜头`
- `直视镜头`
- `盯着观众`
- `正对镜头凝视`

Use exact gaze targets:

- `[@角色B] 看向 [@角色C（感染后）]`
- `[@角色A] 看向 [@角色B]`
- `[@角色C（感染后）] 抬头看向靠在墙边的 [@角色B]`
- `[@角色A] 看向画面右侧的门口`

For multi-character shots, state foreground/background or left/right only when it prevents identity fusion or action confusion.

## Spatial Relationship Rules

Every prompt with observation, hiding, pursuit, standoff, or multi-subject action must make the spatial logic explicit enough for generation:

- camera viewpoint: door gap, window side, over-the-shoulder, high angle, low angle, hallway looking inward, room looking outward,
- observing subject location and posture: crouching by the window, standing behind a sofa, leaning against the door, hiding in foreground shadow,
- target location: downstairs street, far background entrance, left side doorway, behind the counter, outside the window,
- line of sight: through window glass, past the door crack, over a shoulder, across the room, around an obstacle,
- depth relation: foreground / midground / background, near / far, upstairs / downstairs, inside / outside,
- occlusion or frame anchors: door frame, curtain edge, window frame, shelf, car body, broken wall.

Avoid vague wording such as `看向远处`, `观察外面`, or `空间压迫感强` unless the same sentence states exactly where the subject is, what the subject looks through or past, and what visible target is in which part of the frame.

## Camera Language

Choose 1-2 camera motions per shot. Do not stack too many.

Useful options:

- environment setup: `低机位广角缓慢推进`, `高空俯拍缓慢下降`, `横移扫过场景`.
- emotion close-up: `镜头缓慢推近到侧脸特写`, `从手部动作上摇到眼神`, `浅景深背景虚化`.
- action conflict: `低机位跟拍冲刺`, `快速横移跟随主体移动`, `碰撞瞬间轻微震动`, `冲击后镜头短暂后拉`.
- suspense: `镜头从角色背后缓慢推进`, `门缝视角缓慢拉近`, `焦点从前景道具转移到后景人物`.
- aftermath: `烟尘中镜头缓慢后拉`, `从地面碎片上摇到主体背影`, `镜头停在角色坚定侧脸`.

## Action Language

Use concrete verbs instead of vague adjectives.

Character verbs:

- `握紧`, `后退半步`, `抬眼`, `侧身`, `转头`, `屏住呼吸`, `伸手`, `扶住墙`, `踉跄`, `停住`, `低头`, `抬头`, `皱眉`, `眼神收紧`, `嘴唇微动`.

Zombie/monster verbs:

- `低吼`, `扑出`, `撞开障碍`, `抬头嗅探`, `挥爪`, `踉跄逼近`, `张口嘶吼`, `身体抽动`, `撞碎门框`, `从阴影中爬出`.

Environment/effect verbs:

- `烟尘翻滚`, `火星飘散`, `水花飞溅`, `玻璃碎片坠落`, `灯光闪烁`, `阴影掠过`, `雾气扩散`, `雨水沿边缘滑落`, `能量电弧跳动`, `冲击波扩散`.

## Atmosphere Rules

Atmosphere must come from the script context or reference image, not fixed defaults.

Do not carry weather, rain, cold color, blue filter, ruin, blood, or extreme darkness from previous chapters or projects unless supported by the current scene.

Avoid global color words that distort character colors:

- `高饱和度`
- `低饱和度`
- `降低饱和度`
- `提高饱和度`
- extreme color filters unless requested.

Prefer story-state words:

- `阴天`
- `白天室内自然光`
- `夜晚室外压迫`
- `末日爆发初期`
- `刚刚脱险`
- `食物危机`
- `暧昧误会反转`
- `安全区秩序建立`
- `危机逼近`
- `短暂安全后的紧张`

Keep sequence atmosphere consistent across adjacent shots in the same scene. A new character appearing should not change the whole color mood unless the story state changes.

## Constraint Rules

Use concise shot-specific constraints only.

Good:

```text
主体一致，动作连贯，不新增角色。
```

Acceptable when needed:

```text
约束：不要让角色看镜头，不要身份融合，不要额外人物。
```

Bad:

```text
审核安全表达：所有角色均为成年人，衣着完整；表现末日惊悚、雨天逃生后的潮湿状态，不表现裸露、色情接触、血腥细节。全章氛围锁……
```

Safety constraints are thinking rules first. Put them in the prompt only when visually necessary and keep them short.

## Dialogue And Mouth Movement

If a shot has dialogue:

- identify the speaker exactly,
- describe the speaker's visible mouth movement inside the relevant time-coded beat,
- keep mouth movement tied to that speaker only.

If no dialogue:

- write `无台词，角色不张口` only when mouth mistakes are likely.

Do not include full long dialogue unless needed for lip-sync. For a video prompt, summarize who speaks and the emotional beat of the line.

## Quality Words

Always end with concise quality words, but do not bury motion under generic tags. Use a short style-aware ending such as `电影级构图，动态镜头，主体一致性强，角色面部稳定，服装与道具稳定，动作连贯，镜头运动流畅，光影层次清晰，细节干净，高清锐利。`

Avoid long tag piles such as `masterpiece, best quality, ultra detailed, cinematic, 8k`.

## Missing Materials

Before finalizing prompts, compare visible subjects against available materials.

If a required character, monster, prop, scene, weapon, or effect has no material/reference and the production route requires references:

- stop before generation,
- report missing subjects,
- do not silently replace them with imagination unless the user allows text-only description.

For simulation drafts, mark missing materials in `备注` instead of stopping.

## Weak Prompt Example

Avoid weak prompts such as `角色在房间里很紧张，怪物出现，电影感，高质量。` because they lack subject position, visible action, camera movement, and ending state.

## Quality Checklist

Before finalizing, verify:

- the `主体调用计划` has been produced and confirmed for the target shot range, unless the output is clearly marked as simulation,
- production route is clear or reasonably inferred,
- prompt is dynamic, not just a still-image description,
- prompt has a beginning, middle, and ending state,
- prompt uses separate time-coded segments such as `0-2秒：` and `2-5秒：`,
- every time-coded segment includes visible image, subject, camera movement, action change, and ending state,
- prompt includes camera movement,
- prompt includes environmental/effect movement when appropriate,
- prompt uses exact Vidu inline subject references for saved subjects when available,
- saved-subject tokens appear inline at the visible action/location, not only in a suffix list or consistency sentence,
- prompt explains spatial relationships clearly: where the camera is, where the main subject is, what the subject sees or faces, and where the target is in frame depth,
- prompt does not mix token subject names with pronouns,
- no identity explanation sentence like `[@角色] 是...参考主体`,
- characters do not unintentionally look at camera,
- multi-character shots separate actions and gaze targets clearly,
- atmosphere matches the script and scene, not fixed defaults,
- constraints are concise,
- final prompt does not contain unresolved decision words such as `或者`, `可选择`, `如果需要`, `生成/选择`, `建议使用`, or `看情况`,
- quality words are present but not bloated,
- no Vidu task submission language, task ID, download path, or completion status appears,
- parameters are recorded separately from prompt text.

## Stop And Ask

Stop and ask the user before proceeding if:

- production route is unknown and the user wants final prompts, not simulation,
- storyboard/script path is missing and cannot be inferred,
- material table is required but missing,
- saved-subject token names conflict,
- a required visible subject has no reference and text-only generation is risky,
- the `主体调用计划` has ambiguous or suspected subjects that need user confirmation,
- the user asks to submit generation tasks.

Do not ask if the user provided enough context for a useful prompt. Make a reasonable default and proceed.
