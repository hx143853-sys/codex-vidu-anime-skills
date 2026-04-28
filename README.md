# AI Anime Drama Production Skills

一套可公开部署的通用 agent 技能包，用于把小说、剧本、分镜表和物料表转成 AI 动漫剧生产流程。重点支持日系动漫、丧尸末日、生存爽剧、系统流、动作戏等项目，也可以按项目风格扩展到其它类型。

这些技能不是某个 agent 专属。只要你的 agent 支持读取 `SKILL.md` 或类似的本地技能说明文件，就可以按自己的技能目录结构导入使用。

## 技能列表

完整流程技能：

- `zombie-anime-workflow`: 总控流程，负责询问风格、类型、制作方式，并按步骤调用其它功能技能。
- `zombie-subject-extraction`: 主体提取和配音台词表，覆盖角色、场景、道具、特效、其它，多形态命名、关系设定、形象提示词、角色音色描述。
- `zombie-storyboard-director`: 剧本转分镜表，生成镜头号、时长、景别、场景、出场角色、画面描述、台词、音效、备注。
- `zombie-frame-grid-planning`: 首帧图和宫格图规划，用于首帧图生视频、单图转动态视频、宫格图生视频路线。
- `zombie-video-prompting`: 分镜转视频提示词，输出可直接用于视频模型的连续动态提示词，支持 Vidu 主体引用 `[@主体名]`。
- `zombie-vidu-generation`: Vidu 生成执行层，负责提交前检查、参数映射、任务日志、查询、下载和失败处理。
- `zombie-shot-review-redo`: 成片审查和返工规划，定位角色融合、主体错误、场景天气错误、动作错误、口型错误等问题。

附加 Vidu 提示词技能：

- `vidu-anime-drama`: 动漫剧镜头提示词经验规则。
- `vidu-multi-reference-video`: Vidu 多参考主体视频提示词规则。

## 推荐工作流

1. 使用 `zombie-anime-workflow` 作为总控入口。
2. 先确认项目风格、类型、制作路线：`多图参考生视频`、`首帧图生视频` 或 `宫格图生视频`。
3. 调用 `zombie-subject-extraction` 生成主体表和配音台词表。
4. 调用 `zombie-storyboard-director` 生成分镜表。
5. 根据制作路线，必要时调用 `zombie-frame-grid-planning` 规划首帧或宫格图。
6. 调用 `zombie-video-prompting` 生成视频提示词。
7. 用户确认后，再调用 `zombie-vidu-generation` 提交 Vidu 任务。
8. 成片后调用 `zombie-shot-review-redo` 做返工表和重做提示词。

## 关键规则

- 每一步生成、上传、提交 Vidu、下载替换文件前，都需要用户确认。
- 不要只看分镜的 `出场角色` 列，要结合剧本上下文判断真实出场主体。
- Vidu 已保存主体必须在提示词正文中使用 `[@主体名]`，并在素材映射中有对应元素。
- 不要写 `[@角色] 是角色参考图` 这种身份解释句，主体 token 本身就是身份锚点。
- 最终视频提示词应是连续电影化段落，不要输出 `风格：`、`镜头与机位：`、`动作时间轴：` 这种分段标签，除非用户明确要求分析格式。
- 不要把安全审核话术、全章氛围锁、过多负面约束塞进主提示词。约束应简短，主要作为写提示词时的思考规则。
- 不要继承旧项目的固定天气、雨天、冷色调、血腥、破败等默认设定。氛围必须来自当前剧本、当前章节和当前场景。
- 多人镜头要明确每个主体的动作和视线，避免角色融合；角色不要无故直视镜头。
- `VO`、`OS`、`SO` 是说话方式，不是新角色，应合并到真实说话人。

## 部署方式

把需要的技能文件夹复制到你的 agent 技能目录中。每个技能文件夹至少需要包含：

```text
skill-name/
  SKILL.md
```

如果你的 agent 支持额外 UI 元数据，也可以保留：

```text
skill-name/
  agents/
    openai.yaml
```

最小部署方式：

```powershell
Copy-Item -Recurse .\skills\zombie-anime-workflow <你的-agent-技能目录>
Copy-Item -Recurse .\skills\zombie-subject-extraction <你的-agent-技能目录>
Copy-Item -Recurse .\skills\zombie-storyboard-director <你的-agent-技能目录>
Copy-Item -Recurse .\skills\zombie-frame-grid-planning <你的-agent-技能目录>
Copy-Item -Recurse .\skills\zombie-video-prompting <你的-agent-技能目录>
Copy-Item -Recurse .\skills\zombie-vidu-generation <你的-agent-技能目录>
Copy-Item -Recurse .\skills\zombie-shot-review-redo <你的-agent-技能目录>
```

## Vidu 依赖

本仓库不内置 Vidu API 密钥，也不保存任何账号凭据。实际提交、查询和下载 Vidu 任务时，需要你的本地环境已经安装并配置好 Vidu CLI 或对应官方 Vidu 技能。

使用前请在本地配置自己的 Vidu API token，且不要把 token 写入 `SKILL.md`、任务日志、表格或 GitHub 仓库。

## 隐私与公开说明

这个仓库是公开版技能包，已清理：

- 私有剧本内容
- 客户项目名称
- 本地文件路径
- 飞书链接
- 微信临时文件路径
- API key、Bearer token、环境变量
- Vidu 任务 ID 和生成媒体链接

如果你把自己的项目规则继续写入这些技能，请在推送前重新检查是否包含私有剧情、客户信息、素材链接或密钥。
