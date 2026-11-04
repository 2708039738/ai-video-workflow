用途：说明技能包目录、各平台放置与验证方式，并列出安装失败时的检查项。

# 平台与安装

## 目录结构要求

| 相对技能包根目录的路径 | 用途 |
| --- | --- |
| `SKILL.md` | 技能入口，必须直接位于 `character-reference-sheet/` 根层。 |
| `references/layout-and-identity.md` | 版式与身份标准。 |
| `references/qa-checklist.md` | 质量验收。 |
| `adapters/adapter-contract.md` | 适配合同。 |
| `references/platform-guide.md` | 平台与安装。 |
| `examples/test-cases.md` | 标准测试。 |

- [ ] 以技能包根层的 `SKILL.md` 为起点解析正文中的相对引用，不改成个人电脑的绝对路径。
- [ ] 保留原包已有目录；本清单只列入口及本次补齐的五个文件。
- [ ] 原文另引用 `prompts/generation-zh.md` 和 `prompts/generation-en.md`；应从原包保留，不能把本次五文件补齐视为这两个提示词文件已经提供。
- [ ] 公共分发不包含真人照片、私人母版或API密钥。

## 各平台的放置路径与验证方法

| 平台 | 放置路径或导入位置 | 验证方法 |
| --- | --- | --- |
| Claude Code，本地会话 | 用户级：`~/.claude/skills/character-reference-sheet/SKILL.md`；项目级：`.claude/skills/character-reference-sheet/SKILL.md`。 | 输入 `/character-reference-sheet`，确认载入本包，再要求读取五个相对路径；路径依据[Claude Code官方文档](https://code.claude.com/docs/en/skills)。 |
| WorkBuddy | 通过“专家·技能·连接器 → 技能 → 添加技能 → 上传技能”导入包，由客户端放置；代码开发项目可放在 `.codebuddy/skills/character-reference-sheet/SKILL.md`；待核实:当前客户端用户级本地技能目录。 | 在“已安装”中确认技能已启用，新建任务并明确调用 `character-reference-sheet`，要求读取五个相对路径；入口依据[官方技能文档](https://www.codebuddy.cn/docs/workbuddy/From-Beginner-to-Expert-Guide/Function-Description/Skills-Market)，项目路径依据[官方项目文档](https://www.codebuddy.cn/docs/workbuddy/From-Beginner-to-Expert-Guide/Function-Description/Project)。 |
| Codex，本地技能发现 | 用户级：`~/.agents/skills/character-reference-sheet/SKILL.md`；项目级：`.agents/skills/character-reference-sheet/SKILL.md`。 | 在CLI或IDE扩展中用 `/skills` 或 `$character-reference-sheet` 选择技能，确认能读取五个相对路径；路径与调用方式依据[OpenAI官方文档](https://learn.chatgpt.com/docs/build-skills)。 |
| 其他 agent | 放入该工具明确支持的技能目录；待核实:目标agent的技能根目录、导入入口及相对路径解析方式。 | 先确认工具能载入 `SKILL.md` 并打开五个文件；不支持技能安装时，手动读取正文并按适配合同交接。 |

- [ ] 安装可发现只说明技能能被读取；能否生成或编辑图片仍按[适配合同](../adapters/adapter-contract.md)核对。
- [ ] 本文路径依据官方文档核对，未在上述平台逐一实际安装测试。

## 安装失败的三种常见原因

| 原因 | 怎么确认 | 怎么修 |
| --- | --- | --- |
| 多套一层目录 | 平台预期的技能目录下先出现另一个同名目录，`SKILL.md` 被放到第二层。 | 将包含 `SKILL.md` 和资源目录的整层文件夹放到平台技能目录，不单独移动入口文件。 |
| frontmatter写坏 | 开头缺少成对的 `---`，YAML缩进或引号未闭合，或平台报告字段解析失败。 | 恢复原始frontmatter，对照目标平台要求检查字段；不要凭记忆增加市场专用元数据。 |
| 缓存未刷新 | 文件已在指定位置且可读，技能列表或当前会话仍显示旧内容。 | 按平台入口刷新技能列表、新建会话，必要时重启客户端后再次点名调用；不要把重启视为所有平台的固定安装步骤。 |

## 必须逐项核实的平台细节

- [ ] 待核实:Claude Code、WorkBuddy、Codex及其他agent支持本包的具体版本号。
- [ ] 待核实:目标平台的技能包大小、文件数量与参数上限。
- [ ] 待核实:实际图像生成器的参考图数量、输入尺寸、输出分辨率上限及图像编辑能力。
- [ ] 待核实:目标图像生成器的参考图语法、上传接口及图片与提示词的绑定方式。
- [ ] 待核实:WorkBuddy开放平台或技能市场提交所需字段与当前客户端本地导入要求的差异。
