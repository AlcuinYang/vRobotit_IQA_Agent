# IQAgent

IQAgent 是一个承载实验室图像质量评价（IQA）能力的框架。它把实验室过往项目和已发表论文中的模型集中到一个平台上，用于在对外交流时现场展示这些模型的能力。

当前处于需求讨论阶段，仓库中只有文档，没有代码。

## 定位

- **框架为主**：agent 本身不追求强大的能力，够用即可。价值在于把实验室的模型接进来、把结果呈现清楚。
- **实验室模型 + 闭源 API**：闭源 API 作为 agent 的大脑，负责理解请求、分工、调用工具、整合结果和撰写报告；各项属性的评估由实验室模型完成。实验室暂时没有模型的属性，以后可通过安装 skill、由闭源 API 按 skill 评估来补充。
- **易于扩展**：新项目、新论文产出的属性模型或 skill，应能由模型作者自行接入。

## 运行平台

- 目标平台为 DeepSeek Harness（命令行 `dsh`），LLM 后端为 DeepSeek 系模型。
- skill 格式与 Anthropic 的 SKILL.md 相同。
- 实验室的属性级模型已部署为 HTTP API，由 agent 通过网络调用。

## 本阶段覆盖的能力

| 能力 | 输出 |
|---|---|
| 整体质量 | 分数 + 文字解释 |
| 曝光 | 分数 + 偏向 |
| 对比度 | 分数 |
| 白平衡 | 分数 + 偏向 |
| 色彩 | 分数 |

模糊、噪声、压缩伪影等失真类属性，以及美学、构图、人像、视频，本阶段不做，后续以接入模型或安装 skill 的方式补充。

## 文档

| 文件 | 内容 |
|---|---|
| [docs/PRD.md](docs/PRD.md) | 产品需求文档 v2（当前版本） |
| [docs/DEMAND-RESEARCH.md](docs/DEMAND-RESEARCH.md) | 画质评估的需求侧调研：谁需要、什么场景、需要什么样的评估 |
| [docs/COMPETITIVE-RESEARCH.md](docs/COMPETITIVE-RESEARCH.md) | 商业竞品调研（11 个类别、60 余个产品） |
| [docs/BRIEFING-MARKET-2026-09.md](docs/BRIEFING-MARKET-2026-09.md) | 竞品调研的汇报材料 |
| [docs/RELATED-WORK.md](docs/RELATED-WORK.md) | 相关工作精读：AgenticIQA 与 IQA-T1 |
| [docs/archive/PRD-v1.md](docs/archive/PRD-v1.md) | PRD v1，已被 v2 替代，仅存档 |
| docs/assets/ | 架构示意图 |
