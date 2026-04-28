---
name: zombie-subject-extraction
description: Use when extracting reusable production subjects and dubbing dialogue tables from AI anime drama scripts, novels, outlines, storyboards, or material tables. Builds spreadsheet-ready subject sheets for characters, scenes, props, effects, and other assets, plus a dialogue sheet for voice actors, including first appearance, importance, relationship/faction notes, multi-form naming, derived-form consistency, direct image-generation prompts, and VO/OS/SO speaker merging.
---

# Zombie Subject Extraction

Use this skill as a script breakdown plus AI asset bible. The output must let later agents generate consistent character images, scene references, props, effects, Vidu videos, and voice-actor dialogue sheets without rereading the entire script from scratch.

## Inputs

Use all available context before extracting:

- script text or script document,
- existing storyboard table if provided,
- existing material/subject table if provided,
- project style and genre from `zombie-anime-workflow`,
- target chapter/episode range,
- output folder and filename if table creation is requested.

If called directly and the user wants a file, ask before writing. If called by `zombie-anime-workflow`, treat the workflow confirmation as permission for this step only.

Do not generate images in this skill. Only prepare subject rows and image prompts unless the user explicitly approves image generation with a model/API.

## Categories

Extract every reusable visible or story-important subject into exactly one category:

- `角色`: named people, recurring survivors, enemies, zombies, monsters, groups that need visual consistency.
- `场景`: reusable locations and location states.
- `道具`: objects, weapons, vehicles, clothes, keys, phones, food, medical items, UI items when treated as an object.
- `特效`: abilities, energy effects, UI animations, infection effects, explosions, smoke, holograms.
- `其它`: production-relevant items that do not fit above.

Do not create separate subjects or dialogue speakers for `VO`, `OS`, `SO`, narration labels, or speaking modes. Merge them into the real speaker. Example: `角色A VO` and `角色A OS` are still `角色A`.

Only create a `系统`, `旁白`, or `广播` subject if it has a visible recurring interface, object, or visual identity. Otherwise keep it out of the subject table and only use it in dialogue/storyboard tables.

## Required Columns

Create spreadsheet-ready rows with these columns:

- `主体类别`
- `初登场集数`
- `名称`
- `重要程度`
- `设定`
- `提示词`
- `角色类型`
- `形象图`
- `形态来源`
- `备注`

For non-character rows, leave `角色类型` blank.

In `设定`, include the subject's story function, relationship/faction, personality or state, and why it matters. Do not write only a biography. The user should be able to tell whether the subject is protagonist, ally, antagonist, victim, zombie, passerby, safe zone, threat location, or plot device.

In `提示词`, write a direct image-generation prompt, not a loose summary.

## Dialogue Sheet

This section fully replaces the planned standalone `zombie-dialogue-table` skill. Do not route dubbing dialogue extraction to a separate skill.

Add a separate worksheet named `台词表` when the user requests subject extraction for a full production workbook, or when dubbing/voice-over delivery may be needed later.

Use this layout:

- Column A: `出场角色`
- Column B: `角色音色描述`
- Column C onward: one column per chapter/episode, such as `第1章`, `第2章`, `第3章`, or the project's existing episode labels.

Each role row contains that character's complete dialogue for the chapter/episode. Preserve the original wording and punctuation as much as possible.

`角色音色描述` is for later AI dubbing and voice training. Write it as a practical voice-casting note, not a personality biography.

Voice description should include:

- apparent age range and gender presentation,
- vocal texture, such as clear, soft, husky, bright, hoarse, cold, magnetic, childish, mature, exhausted, broken,
- speaking pace and rhythm,
- emotional baseline,
- volume/energy level,
- breathiness or tension,
- accent or lack of accent if script context supports it,
- special treatment for zombies, monsters, systems, radios, masks, or phones.

Good voice description example:

```text
年轻成年女性声线，音色清冷柔和，声线偏细但不尖，语速中等偏慢，情绪基调克制疏离；紧张时尾音发颤、气息变浅，适合从日常角色到末日求生者的转变。
```

Zombie or monster voice example:

```text
丧尸化男性声线，保留少量原本年轻男性底色，但主体为低沉嘶哑的喉音、断裂喘息和短促低吼；台词极少时以破碎音节和非理性咆哮为主。
```

System/UI voice example:

```text
中性电子提示音，音色清晰冷静，语速稳定，带轻微机械混响和系统播报感；情绪几乎无波动，适合全息UI提示与等级结算。
```

Dialogue rules:

- `VO`, `OS`, `SO`, inner monologue labels, off-screen labels, and parenthetical speaking modes must be merged into the base speaker.
- Do not create rows like `角色A VO`, `角色A OS`, `角色B（OS）`; use `角色A`, `角色B`.
- Keep complete dialogue lines intact. Do not split one full sentence just to make the table shorter.
- If one chapter has many lines for a character, place them in the same cell separated by line breaks.
- Prefix each line with a simple sequence number when helpful for recording, such as `1. ...`, `2. ...`.
- If the speaker is ambiguous, use the most likely character only when the script context is clear; otherwise mark `待确认 speaker` in the cell or `备注`.
- Do not invent dialogue, rewrite lines, or add performance notes unless the user asks.
- If one visual character has materially different voice states, keep one base row unless voice training requires separate voices. When separate voices are needed, use the same multi-form naming pattern, such as `角色A（日常声线）` and `角色A（感染后声线）`.
- Keep `角色音色描述` aligned with the subject table's relationship and form notes.

Formatting rules when creating an actual spreadsheet:

- Assign each role a stable fill or font color in `台词表`.
- Use the same color for that character's row header and dialogue text.
- Keep colors readable on white background.
- Reuse the same role colors across later workbook updates.
- If spreadsheet coloring is not possible in the current environment, still create the table structure and note that color formatting remains pending.


## Material Mapping Handoff

When image references or Vidu saved subjects exist, include or create a worksheet named `素材映射`. This sheet bridges the subject bible to video prompting and generation.

Recommended columns:

- `主体名称`
- `主体类别`
- `形态来源`
- `形象图路径或URL`
- `Vidu元素名称`
- `Vidu元素ID`
- `Vidu元素版本`
- `可用状态`
- `备注`

Downstream skills must use `Vidu元素名称` as the exact inline token name when a saved subject exists, for example `[@Vidu元素名称]`.

If the subject exists in the story but no image or Vidu element exists yet, mark `可用状态` as `待制作` or `缺失`, not as ready.

## Handoff Requirements

End the extraction step with:

- subject row count by category,
- dialogue speaker count,
- created/updated workbook path,
- material mapping sheet status,
- missing required visual subjects,
- next recommended skill.

Do not silently change subject names after downstream storyboard, prompt, or generation tables already reference them.

## Importance

Assign star importance by production reuse and story impact:

- `⭐⭐⭐⭐⭐`: core protagonist, main heroine/hero, major antagonist, main recurring base, signature monster, signature ability.
- `⭐⭐⭐⭐`: major supporting character, repeated important location, repeated weapon/prop/effect.
- `⭐⭐⭐`: named supporting character, one-episode but plot-important asset, scene needed for several shots.
- `⭐⭐`: minor named character, reusable background subject, low-frequency prop.
- `⭐`: one-off passerby, crowd, non-recurring detail that still needs a row.

Sort every category by `初登场集数`, then by importance descending, then by story order.

## Character Types

Default character type values:

- `男主角1`
- `男主角2`
- `女主角1`
- `女主角2`
- `配角`
- `龙套`

If the project already uses finer labels such as `反派`, `怪物`, `普通感染者`, or `特殊感染者`, use them only when helpful. Otherwise keep role type simple and put faction/threat information in `设定`.

## Multi-Form Naming

Split a subject into multiple rows when the visual reference must change for later generation.

Use this naming pattern:

- `角色A（常服）`
- `角色A（战斗服装）`
- `角色B（感染前）`
- `角色B（感染后）`
- `场景A（白天）`
- `场景A（夜晚断电）`
- `商店A（正常营业）`
- `商店A（被洗劫后）`

Only split forms when one of these changes matters visually:

- outfit, body state, injury, zombie/infection state, age/time skip,
- location lighting/weather/destruction/furnishing state,
- prop condition or upgraded form,
- effect color, scale, or mechanics.

For derived forms, keep consistency:

- Extract the base form first.
- Set `形态来源` to `基于：原主体名称`.
- In `提示词`, explicitly say it is generated from the base subject's identity and should preserve face, body silhouette, key clothing structure, or material logic as appropriate.
- In `形象图`, use `待生成` or `待参考生成：基于 原主体名称`; never fabricate an image path.

Example: `角色B（感染后）` should be derived from `角色B（感染前）` if both are the same person.

## Extraction Workflow

1. Read the full requested script range first. Do not rely only on a storyboard row, because storyboard cast columns can omit characters.
2. Identify chapter/episode boundaries and scene transitions.
3. List candidate subjects by category.
4. Merge aliases, nicknames, VO/OS/SO variants, and role labels into the real subject.
5. Decide multi-form rows only when visual continuity requires it.
6. Assign first appearance from the earliest chapter/episode where that form appears.
7. Write relationship-aware settings.
8. Write direct image prompts using the project style and the category suffix rules below.
9. Extract the `台词表` sheet from the same script pass.
10. Sort subject rows by first appearance.
11. Run the quality checklist before finalizing.

If a subject is mentioned but never visually appears and is not needed for materials, put it in `备注` instead of making a full row.

## Prompt Rules

Use the project's declared visual style. If no style is declared, default to `日系动漫风格，二次元手绘动画质感`.

Do not inject fixed weather, rain, cold color, destruction, blood, or apocalypse decay unless the script state says so. Especially for early zombie-outbreak scenes, normal everyday anime environments may be correct.

Avoid prompt words that directly distort all colors, such as `高饱和度`, `低饱和度`, or extreme filter language, unless the user asks for that look. Prefer story atmosphere words such as `阴天`, `压抑`, `清晨`, `黄昏`, `室内冷光`, `紧张`, `末日爆发初期`.

### Character Prompt Structure

Write detailed character prompts in this structure when possible:

```text
生成一个角色设定图。
角色身份与关系：...
发型：...
发饰：...
装饰：...
面部表情：...
上衣：...
内搭：...
外套：...
裙子/裤子：...
鞋子：...
配饰：...
动作与姿态：...
风格：...
构图：角色设计参考图，保持同一角色身份、五官、发型、轮廓、服装结构和配饰一致。
背景：白色纯色背景。
限制：禁止文字、符号、标签、水印、无关人物。
同一角色多视图设定参考图，包含脸部正面特写、脸部侧面特写、人物正面站姿、人物背面站姿；主视觉区以正面、侧面、背面三个核心视角为主体，补充信息区拆分面部特写。
```

If script details are limited, infer visual design from role, age, genre, and tone, but mark uncertain details as production assumptions in `备注` if they may affect continuity.

### Scene Prompt Suffix

Scene prompts must describe the actual script state:

```text
完整场景全景参考图，展示前景、中景、远景和整体空间结构，地面、墙面、建筑、陈设与主要环境元素清楚可见，透视准确，层次完整，适合作为后续镜头统一场景参考。场景中不要出现人物或动物，除非用户明确要求。
```

Do not call scenes `破败不堪` by default. Use `干净`, `日常`, `刚刚混乱`, `断电`, `雨夜`, `被洗劫后`, or other state words only when supported by the script.

### Prop Prompt Suffix

```text
单一道具参考图，道具主体居中完整展示，白色纯色背景，无人物无杂物干扰，轮廓清晰，材质和结构细节明确，边缘干净，便于后续抠图和复用。
```

### Effect Prompt Suffix

```text
单一特效参考图，背景简洁干净，特效主体完整突出，发光范围、能量轨迹、粒子层次和边缘形态清晰，视觉冲击力强，便于后续叠加与复用。
```

### Other Prompt Suffix

No fixed suffix. Explain why it exists and how it should be reused.

## Output

Preferred workbook structure:

- `角色`
- `场景`
- `道具`
- `特效`
- `其它`
- `台词表`

If the user asks for one sheet only, use `主体表`.

Recommended filename:

```text
<小说名>_主体表.xlsx
```

When updating an existing workbook, create a versioned copy unless the user explicitly says to overwrite.

## Quality Checklist

Before finalizing, verify:

- No `VO`, `OS`, `SO`, narrator markers, or speech-mode labels are separate characters.
- Every recurring visual subject is captured.
- Multi-form subjects use consistent parentheses naming.
- Derived forms identify their base subject in `形态来源`.
- `设定` explains story relationship/faction, not only appearance.
- `提示词` is detailed enough for image generation.
- Scene prompts do not add people, animals, rain, decay, or zombie destruction unless supported by script.
- Rows are sorted by first appearance.
- `形象图` is blank, `待生成`, or a real known file path/URL; never invent paths.
- `台词表` uses real speakers only; no `VO`, `OS`, or `SO` pseudo-speaker rows.
- `台词表` includes `角色音色描述` for every speaking character.
- `台词表` preserves complete original dialogue and does not invent lines.

## Stop And Ask

Stop and ask the user before proceeding if:

- the script or requested chapter range is missing,
- the output path is unknown and the user asked for a file,
- the user asks to generate images but has not chosen a model/API,
- a derived form needs a base image but the base subject is missing,
- the script is ambiguous about whether two names refer to the same person and merging would risk continuity.
