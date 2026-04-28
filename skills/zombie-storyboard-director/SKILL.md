---
name: zombie-storyboard-director
description: Use when converting AI anime drama scripts, novels, outlines, or revised chapters into detailed professional storyboard tables. Produces shot-level sheets with shot number, duration, shot size, scene, independently judged visible/speaking characters, director-grade picture descriptions, dialogue, sound, and notes while preserving plot and dialogue. Picture descriptions should include current screen content, composition, shot size, character action, emotion/reaction, scene focus, camera angle/movement, and visual emphasis, but the skill must not create Vidu prompts, dynamic prompt columns, visible-subject mapping columns, saved-subject tokens, or reference-image mappings.
---

# Zombie Storyboard Director

Use this skill to convert script text into a detailed professional anime storyboard table. This is the step after subject/dialogue extraction and before video prompting.

The output is a director storyboard, not a Vidu prompt plan. It may describe composition, staging, camera angle, simple camera movement, emotion, and visual exaggeration, but it must not use Vidu saved-subject tokens or video-generation parameter language.

## Inputs

Use all available context before making the storyboard:

- script text or script document,
- target chapter/episode range,
- existing subject workbook if provided,
- existing rough storyboard if provided,
- user-provided edits or revised script fragments,
- output folder and filename if table creation is requested.

If the user asks for a file and no output location is known, ask before writing. If called by `zombie-anime-workflow`, treat the workflow confirmation as permission for this step only.

## Required Columns

Use exactly these columns unless the user explicitly asks otherwise:

- `镜头号`
- `时长`
- `景别`
- `场景`
- `出场角色`
- `画面描述`
- `台词`
- `音效`
- `备注`

Do not add these columns in this skill:

- `动态提示词`
- `可见主体`
- `人物站位与空间关系`
- `视线关系`
- `Vidu提示词`
- `主体token`
- `参考图`

Those belong to the later video-prompting skill. Do not split staging or gaze into separate columns; include only necessary director information inside `画面描述`.


## Shot ID Contract

Use stable `镜头号` values that can be joined with later frame/grid planning, video prompt, generation log, and redo tables.

Recommended formats:

- `001`, `002`, `003` inside a single episode/chapter,
- `E01-S001` or `第1集-001` for multi-episode projects.

Do not renumber existing shots after downstream artifacts exist. If a shot is later split, use suffixes like `E01-S021A` and `E01-S021B` and keep the original shot number in `备注`.

## Handoff To Video Prompting

The storyboard does not include Vidu tokens or reference mappings. Before video prompting, pass or report:

- storyboard workbook path,
- subject workbook path,
- target shot range,
- production route,
- style/genre,
- scene continuity notes,
- shots that may need first-frame/grid planning.

If a shot is visually complex, mark in `备注`: `建议首帧` or `建议宫格`. Do not create video prompts in this skill.

## Field Rules

### 镜头号

Number shots in order within each chapter/episode.

Use `1`, `2`, `3` or `1-1`, `1-2`, `1-3` depending on the user's existing format. If no format exists, use plain sequential numbers inside each chapter sheet.

### 时长

Set duration for future editing. The target rhythm is detailed but not chaotic.

Default rules:

- No dialogue or simple action: `5s`.
- Short dialogue under about 20 Chinese characters: `5s`.
- Medium action/reaction beat: `6-8s`.
- Strong action, reveal, system UI, or emotional beat: `7-10s`.
- Very long dialogue: split into multiple shots at natural pauses, keeping each shot's dialogue preferably within about 20 Chinese characters.

If a shot contains strong action, reveal, or reaction, add time as needed.

### 景别

Use professional but concise storyboard terms:

- `特写`
- `近景`
- `中景`
- `全景`
- `远景`
- `大特写`
- `过肩`
- `群像`

Use `景别` as the main visual scale. More detailed camera or composition information belongs in `画面描述`, not in extra columns.

### 场景

Use the script's scene heading when available.

Keep the scene name stable across the chapter. If the script has repeated or messy scene headings, normalize gently without changing meaning.

Example:

```text
1-2 日 内 住宅客厅
```

### 出场角色

List only characters that appear or speak in the shot.

Judge `出场角色` independently for every shot from the current script beat, dialogue, and actual screen content. Do not copy the previous shot's characters, the whole chapter's cast, or every subject-table entry into every row.

Use real character names. Merge `VO`, `OS`, `SO`, and inner-monologue labels into the same speaker name. Do not create `角色A VO` or `角色B OS` as separate roles.

Do not force subject-table multi-form names unless the script needs that clarity. The storyboard can use normal names like `角色A` and `角色B`.

If a character is only mentioned by dialogue and is not visible or speaking in the current shot, do not list that character in `出场角色`. If the shot description implies a second character is visible, include that character explicitly rather than leaving them hidden in the prose.

### 画面描述

This is the most important field. It must read like a professional director's storyboard description, not a plot summary.

Write what the current screen should show according to the script. Every `画面描述` must include the current visible content, main subject action, emotion or reaction, scene focus, and basic composition/camera information. Include:

- what event is happening,
- who is doing what,
- what object, monster, threat, UI, or result appears,
- what emotional state or story beat is visible,
- what changes from the previous shot.
- basic composition and visual focus,
- character action and reaction,
- camera angle or simple camera movement when useful,
- anime/film/3D/CG-style exaggeration when it strengthens the original beat.

Do not write `画面描述` as a rewritten plot sentence. It should be concrete enough that a later video-prompting step can infer the real visible subjects, opening frame, motion, camera path, and ending state without guessing.

The description can include professional language such as `低角度`, `俯拍`, `推近`, `拉远`, `摇镜`, `快速切入`, `压迫感构图`, `反应特写`, `背景虚化`, and `动作夸张线`.

Still keep this as a storyboard, not a final AI video prompt. Do not include:

- Vidu saved-subject tokens,
- quality words,
- model parameters,
- reference image mappings,
- saved-subject identity explanation,
- long negative constraints,
- generic generation words such as `主体一致性强`, `高清质量`, `不要崩坏`.

Those are handled later by `zombie-video-prompting`.

Good `画面描述`:

```text
低角度近景，角色B靠在狭窄室内角落，脸颊泛红又惊慌，身体紧张后缩；画面先营造误会感，再用她慌乱阻止角色C靠近危险物的台词埋下反转。
```

Good:

```text
冰箱门被角色A拉开，冷白灯光照出空荡荡的隔层，只剩少量食物孤零零摆在角落；镜头从角色A皱眉的侧脸切到冰箱内部特写，物资危机被直接揭开。
```

Bad:

```text
日系动漫风格，冷色调，主体一致性强，不要崩坏，不要真人，不要错误，不要多角色融合。
```

The bad example is video-prompt constraint language, not storyboard.

### 台词

Preserve original dialogue and speaker.

Rules:

- Do not invent dialogue.
- Do not rewrite dialogue.
- Keep each shot to one speaker whenever possible.
- Keep each shot's dialogue preferably within about 20 Chinese characters.
- If two characters speak in rapid exchange, split into separate shots unless the lines are extremely short and naturally belong in one reaction beat.
- Long dialogue should be split across multiple shots at sentence boundaries or natural punctuation, and the same speaker/scene should continue with different shot size or composition.
- Do not split inside a word, idiom, or awkward phrase that breaks voice acting.

Format:

```text
角色A：看清楚，他现在还是人吗？
```

For system narration:

```text
系统：叮！检测到末日降临，【末日伙伴系统】已绑定！
```

### 音效

Write only useful production sound cues:

- environment sound,
- zombie growl,
- impact,
- UI notification,
- door lock,
- footsteps,
- crowd noise,
- weapon sound,
- silence or tension pause if important.

Do not overload every shot with sound if none is needed.

### 备注

Use for concise production notes:

- sensitive content should be handled implicitly,
- long dialogue was split at sentence boundary,
- scene heading normalized,
- original script ambiguity,
- continuity reminder.

Do not use `备注` for video prompt constraints.

## Splitting Rules

Split shots by story beats, dialogue rhythm, and editing usability.

Create a new shot when:

- scene changes,
- time/location changes,
- a new character enters or exits,
- a threat appears,
- an action reaches a result,
- a character reaction is important,
- the speaker changes and the dialogue needs clean editing,
- a system/UI message creates a new beat,
- the emotional direction changes.
- a dialogue line is too long for one 5-10 second shot.

Keep the average rhythm around `5-10s` per shot. Avoid extreme fragmentation, but the storyboard should be detailed enough for editing and later video prompt work.

Do not add extra plot events, extra reactions, or extra dialogue.

## Dialogue Length Handling

The user prefers edit-friendly dialogue length for AI video generation.

When a line is long:

1. Split at natural punctuation such as full stop, comma, question mark, exclamation mark, semicolon, dash, or clear breath pause.
2. Keep each shot's dialogue preferably within about 20 Chinese characters.
3. Continue the same scene and same speaker in the next shot.
4. Change shot size, composition, or reaction focus so the later edit does not feel static.
5. Preserve original wording and order.

Never cut a phrase in a way that makes voice acting or editing awkward.

## Visual Strengthening

Use anime, film, 3D, or CG-style visual exaggeration only to strengthen the existing plot beat:

- exaggerate facial reactions,
- emphasize threat scale,
- sharpen action impact,
- use dramatic reveal timing,
- use object or UI closeups,
- use reaction shots after important lines,
- use environmental details already implied by the script.

Do not add new events, new characters, new props, or new dialogue.

## Workbook Structure

For a multi-chapter script, prefer one sheet per chapter/episode:

- `第1章分镜`
- `第2章分镜`
- `第3章分镜`

If the user asks for a single sheet, use one `分镜` sheet and include chapter labels in the `镜头号` or `备注`.

Recommended filename:

```text
<小说名>_分镜表.xlsx
```

When updating an existing storyboard, create a versioned copy unless the user explicitly asks to overwrite.

## Quality Checklist

Before finalizing, verify:

- The table uses only the required columns.
- There is no `动态提示词` column.
- There is no `可见主体` column.
- There are no Vidu tokens such as `[@角色名]`.
- Dialogue preserves original wording and is split only at natural pauses.
- `VO`, `OS`, and `SO` are merged into the base speaker.
- One shot usually has one speaker and about 20 Chinese characters or less of dialogue.
- `出场角色` is judged per shot and does not mechanically repeat previous rows or the whole cast.
- The script's plot order is preserved.
- `画面描述` is detailed and director-grade, including current image content, subject action, emotion/reaction, scene focus, composition, and camera information when useful.
- `画面描述` does not contain Vidu prompt constraints, saved-subject mappings, or quality-word boilerplate.
- The storyboard is detailed enough for later prompt generation but not padded with invented story content.

## Stop And Ask

Stop and ask the user before proceeding if:

- the script or requested chapter range is missing,
- the user asks for a file but output path is unknown,
- the script has conflicting versions and it is unclear which one to use,
- the user requests a detailed Vidu prompt instead of a basic storyboard.
