# 相关工作精读：AgenticIQA 与 IQA-T1

> 目的：弄清两篇论文解决什么问题、怎么做的，以及对 IQAgent 来说哪些**能直接抄**、哪些**要改了再用**、哪些**不能抄**。
> 读法：两篇均从 arXiv HTML 原文逐段通读；IQA-T1 另外核对了开源代码（工具实现、prompt、奖励函数、训练配置、SFT 样本）。
> 日期：2026-09-14 · 配套：`docs/ARCHITECTURE.md`、`docs/PRD.md`

---

## 0. 先说结论

**一张定位图**——三个系统的差别在于「最终判定由谁给出」：

| | 谁规划 | 谁给出质量判定 | 工具的角色 | 要不要训练 LLM |
|---|---|---|---|---|
| **AgenticIQA** | 微调 VLM | **VLM**（自己判失真严重度，再把工具分数与 VLM 概率融合） | 给 VLM 当参谋的打分器 | 要（SFT 7B） |
| **IQA-T1** | 微调 VLM | **VLM**（直接输出分数） | 给 VLM 当"眼睛"的确定性证据图 | 要（SFT + GRPO 4B） |
| **IQAgent（你们）** | 通用 LLM | **标定过的属性模型** | 判定的唯一来源 | **不要** |

**三条净结论：**

1. **两篇都把最终判定交给了微调过的 VLM，所以都必须训练。** IQAgent 走的是免训练路线，可信度来自标定而不是微调——这是一条两篇都没走的路，不是落后，是差异化。
2. **两篇在打分精度上的增益都很小**，价值主要在可解释性。AgenticIQA（GPT-4o）在 TID2013 上只比单个工具 TopIQ 高 0.009 SRCC，在 BID 和 AGIQA-3K 上还输给单模型 Q-Align；IQA-T1 七个数据集平均 PLCC 只比不做推理的 DeQA 高 0.009。这从外部印证了 IQAgent 的分工：**别让 LLM 层去跟专用模型比打分，把它花在规划、解释和建议上。**
3. **两篇都没碰「可执行的调优建议」「建议有效性验证」「措辞强度校准」「能力缺口诚实应答」**——恰好是 IQAgent 顾问定位的核心。如果实验室想发论文，这是空位。

**最值得抄的三样东西：**

| 来源 | 抄什么 | 放进 IQAgent 的哪里 | 要改什么 |
|---|---|---|---|
| IQA-T1 | 15 个确定性 OpenCV 证据工具（MIT 许可） | `image_probe`，补失真类缺口 | 去掉逐图 min-max 归一化，改固定量纲；同时输出标量统计 |
| AgenticIQA | Planner 的 JSON schema 与标志位规则 | `iqa-plan` 诊断计划 | 局部查询要**开启**评估（它是关闭的） |
| AgenticIQA | 五参数 logistic 分数对齐 + 按失真类型实测工具擅长项 | `calibration.yaml` 第一级、`capabilities.yaml` 的 `best_at` / `validation` | 用你们自己的数据拟合；论文公式有笔误 |

---

## 1. AgenticIQA

**AgenticIQA: An Agentic Framework for Adaptive and Interpretable Image Quality Assessment**
Hanwei Zhu, Yu Tian, Keyan Ding, Baoliang Chen, Bolin Chen, Shiqi Wang, Weisi Lin（NTU / 浙大 / 华南师大 / 阿里达摩院 / 港城大 / 南信大）· arXiv 2509.26006v2 · 2025-10-01

### 1.1 解决什么问题

- 传统 IQA 模型（FR/NR）：分数准，但不可解释，而且是固定管线，不随用户问题变化。
- VLM 类 IQA：能解释，但分数粗糙（"好/差"），对 prompt 敏感。
- 两类方法都把**打分**和**解释**当成两件独立的事，而作者认为二者互相依赖：解释揭示退化是什么，打分把它压缩成一个数。

### 1.2 需求（作者提出的三条）

- **自适应**：按图像内容和用户意图动态生成评估策略，而不是固定管线。
- **模块化**：把传统打分模型和 VLM 统一编排。
- **可解释**：给分的同时给出结构化、忠实的理由。

### 1.3 技术路线与架构

Plan–Execute–Summarize，三个角色**共用同一个 VLM**：

```
用户问题 + 图像（+ 可选参考图）
   │
   ▼
Planner ── 输出 JSON 计划：query_type / query_scope / distortion_source / reference_mode / required_tool / 四个开关
   │
   ▼
Executor（按开关顺序执行四个子任务）
   ├─ ① 失真检测   VLM 看图，从 7 类里挑候选失真：Blurs / Color distortions / Compression / Noise / Brightness change / Sharpness / Contrast
   ├─ ② 失真分析   VLM 给每类失真打严重度：none / slight / moderate / severe / extreme + 一句理由
   ├─ ③ 工具选择   VLM 读工具元数据，为每类失真选一个 IQA 模型
   └─ ④ 工具执行   调模型 → 五参数 logistic 映射到 [1,5]
   │
   ▼
Summarizer
   ├─ 先判断证据是否足够 → 不够就让 Planner 重新规划（自我修正回路）
   ├─ 解释：综合失真类型、严重度、工具分数
   └─ 打分：以工具分均值为中心构造高斯权重 α_c，乘以 VLM 在五个质量档上的概率 p_c，加权求和
```

- **工具库**：附录列了 28 个模型（14 FR + 14 NR），全部来自 IQA-PyTorch。每个工具的「擅长哪些失真」是在 KADID-10k 上按失真类型实测得出的。
- **训练**：Qwen2.5-VL-7B 全参 SFT，数据为 AgenticIQA-200K（GPT-4o 基于 Q-Pathway 与 DQ-495K 的已有标注生成：5 万规划 + 10 万执行 + 5 万总结）加 Q-Instruct-200K；4×A100，2 个 epoch。推理单卡 3090 可跑。
- **评测基准**：AgenticIQA-Eval，1000 张图的选择题，**按模块分轨评测**（Planner 250 / Executor 500 / Summarizer 250），每题至少两名专家交叉核验。

### 1.4 关键结果（原文核对后的数字）

**打分（SRCC）：**

| | TID2013（合成失真） | BID（真实模糊） | AGIQA-3K（AI 生成） |
|---|---|---|---|
| TopIQ（单个 FR 工具） | 0.9075 | — | — |
| Q-Align（单个 NR 模型） | 0.8313 | **0.8967** | **0.8013** |
| AgenticIQA（微调 Qwen2.5-VL-7B） | 0.7780 | 0.7771 | 0.7165 |
| AgenticIQA（GPT-4o） | **0.9165** | 0.8889 | 0.7937 |

**相对"同一 VLM 直接打分"的增益（SRCC）：**

| | TID2013 | BID | AGIQA-3K | LLVisionQA |
|---|---|---|---|---|
| GPT-4o 骨干 | +0.1598 | +0.0376 | +0.0062 | +0.07% |
| 微调 Qwen 骨干 | +0.0726 | +0.0097 | +0.0035 | +0.03% |

**模块级准确率（AgenticIQA-Eval）**：微调 Qwen 总体 75.40%，GPT-4o 76.30%，人类 81.80%。其中「失真识别」一项人类也只有 75.00%——这个子任务本身就很模糊。

**作者自述局限（附录 E.1，只有三条）**：规划阶段的错误会向下游传播；性能受骨干模型上限约束；顺序执行导致速度与扩展性受限。

### 1.5 批判性阅读

1. **增益主要来自「选对工具」，而不是「agent 推理」。** TID2013 是合成失真，本来就有很强的专用工具（TopIQ 单独 0.9075），agent 只多拿了 0.009。在真实失真（BID）和 AI 生成图（AGIQA-3K）上，一个 Q-Align 就赢了它。增益表也显示：工具库对口的数据集增益大，不对口的几乎为零。
2. **「动态工具选择」没有看上去那么动态。** 28 个工具里只有 5 个（TOPIQ、DISTS、QAlign、UNIQUE、LIQE）写了明确的擅长项，其余大多写着 "no known strengths for any specific distortion"（VIF 的描述还自相矛盾）。无参考场景下，实际可选的基本就是两三个模型。
3. **VLM 在当裁判。** 失真检测和严重度分级都是 VLM 看图判的；Summarizer 的 prompt 明确写着"信息不足时直接分析图像"；「Other」类问题（美学、构图、色彩、情绪）完全由 VLM 目测回答；**局部区域的问题直接关闭工具执行**（`region but no tool → tool_execute = false`）。这些做法之所以可行，是因为它的 VLM 在 40 万条 IQA 指令上微调过。IQAgent 用的是没有经过 IQA 微调的通用 LLM，照搬就会直接违反 IRON RULE 1。
4. **分数融合公式不能直接用来展示分数。** `q = Σ α_c · p_c · c` 没有重新归一化（Σ α_c·p_c ≤ 1），所以 q 不落在 [1,5] 区间。对 SRCC/PLCC 这类相关性指标没影响，但如果要给用户看绝对分数就是错的。另外它把 VLM 的概率掺进了分数，又一次让 VLM 参与判定。
5. **附录参数表有明显错误。** CLIPIQA 那一行的五个 β 与 TopIQ_FR 完全相同，大概率是复制粘贴失误。正文的 logistic 公式把标准形式里的 `1/(1+exp(·))` 写成了 `1/exp(·)`，也是笔误。**这些数字不要直接拿来用。**
6. **没有官方代码。** 论文写的项目页 `agenticiqa.github.io` 目前返回 404；GitHub 上只有一个第三方的 `easy_agenticIQA`（14 星，非官方）。能直接复用的只有附录里的完整 prompt。
7. **许可证风险。** 它的工具全部来自 IQA-PyTorch，而 IQA-PyTorch 是 **PolyForm Noncommercial 1.0.0（禁止商用）**。IQAgent 的 demo 面向合作方，如果后续涉及商业合作，就不能沿用这套工具来源。

### 1.6 对 IQAgent：抄 / 改 / 不抄

**✅ 直接抄**

**(a) Planner 的 JSON schema 和确定性开关规则** → 落到 `iqa-plan`。它把"需求拆解"做成了一个带枚举字段的结构化输出，正好是 ARCHITECTURE §6.2 缺的那份具体 schema。按 IQAgent 的能力改写后：

```json
{
  "query_type": "quality | aesthetic | advice | compare | capability",
  "scope": "global | [{\"name\": \"天空\", \"roi\": null}]",
  "dimension_source": "explicit | inferred",
  "dimensions": { "global": ["exposure", "contrast"] },
  "required_model": null,
  "unavailable_requested": ["noise"],
  "plan": {
    "probe": true,
    "assess": true,
    "roi_locate": true,
    "patch": false,
    "advise": true
  }
}
```

三处关键改动：
- **局部查询要开启评估**：`scope` 是区域时，走 `roi_locate → roi_crop → assess`，而不是像原文那样关掉工具。原文这么设计是因为它的工具只能评整图；IQAgent 有原始分辨率通路和 ROI 裁切（§15）。
- **新增 `unavailable_requested`**：原文没有「用户问到系统没有的维度」这个概念。这一项是 IRON RULE 2 的入口。
- **开关规则由 `validate_plan` 程序校验**，而不是只写在 prompt 里（见 skill/tool 的区别）：`dimensions` 必须全部是 `capabilities.yaml` 里 `status: available` 的维度；`explicit` 时不得自行追加维度。

**(b) 五参数 logistic 分数对齐** → 作为 `calibration.yaml` 的第一级（原始分 → 统一 [1,5]），第二级再做分档。这是 VQEG / Sheikh 2006 的标准做法，务必用**标准形式**：

```python
import numpy as np
from scipy.optimize import curve_fit

def logistic5(x, b1, b2, b3, b4, b5):
    # 标准形式；AgenticIQA 正文漏写了分母里的 "1+"
    return b1 * (0.5 - 1.0 / (1.0 + np.exp(b2 * (x - b3)))) + b4 * x + b5

# 每个属性模型在你们自己的锚定集上单独拟合，参数写进 calibration.yaml
params, _ = curve_fit(logistic5, raw_scores, mos_1to5, p0=[np.ptp(mos_1to5), 1, np.mean(raw_scores), 0, np.mean(mos_1to5)], maxfev=20000)
```

**(c) 「按类别实测擅长项」的方法** → 填 `capabilities.yaml`。原文在 KADID-10k 上逐失真类型测每个工具的相关性，据此写元数据。IQAgent 应该做同样的事：每个属性模型在留出集上**按场景 / 属性分组**测 SROCC，结果同时填进 `best_at` 和 §13.3 的 `validation` / `confidence_ceiling`。这一步正是 IRON RULE 3 所依赖的数据来源。

**🔧 改了再用**

**(d) Summarizer 的「证据是否充分 → 重新规划」回路。** 思路要保留，但两点必须改：
- 充分性判断改成**确定性规则**：计划里的必需维度出现在 `degraded` 列表中 → 重试或重新规划；而不是让 LLM 判断"够不够"。
- 证据不足时的兜底**绝不能是**"直接看图"，而应该是"如实报告该维度未取到结果"。

**(e) 按模块分轨评测。** AgenticIQA-Eval 把规划、执行、总结分开打分，便于定位是哪一环出错。IQAgent 的 `eval/` 回归集应该加一条**规划轨**：给定用户问题，检查计划中的维度和范围是否正确。100–250 条就够，成本低，而且规划错误会向下游传播（作者自述局限第一条），最值得单独测。

**❌ 不要抄**

- VLM 做失真检测和严重度分级（违反 IRON RULE 1；它能这么做的前提是 40 万条 IQA 微调数据）。
- VLM 概率参与分数融合，以及未归一化的融合公式。
- "信息不足就直接看图"的兜底策略。
- 三个角色都用 LLM 调用：Executor 在 IQAgent 里应该是确定性网关（tool），而不是 LLM agent——这也顺带消除了作者自述的顺序执行慢的问题。
- 附录 Table 5 的拟合参数（有明显错误，而且是别人的模型、别人的数据）。

---

## 2. IQA-T1

**IQA-T1: Tool-based Visual Evidence Reasoning for Image Quality Assessment**
西北工业大学 + 香港科技大学（通讯作者 Wei Wei、Qifeng Chen）· arXiv 2607.12375v1 · 2026-07
代码：`github.com/zibuyu-02/IQA-T1`（**MIT 许可**，已开源工具、训练配置、奖励函数与数据）

### 2.1 解决什么问题

MLLM 做 IQA 存在**语义偏置**：视觉编码器的表征偏向高层语义，对噪声、模糊、压缩伪影这类低层退化不敏感。作者用一个式子概括——一张图和它加了噪声的版本，视觉表征几乎相同（‖z_v(I) − z_v(I_d)‖ ≈ 0），但人的主观分数差很多。

现有两种补救思路都不够：
- **纯文本推理**（Q-Insight、VisualQuality-R1）：推理链没有视觉锚点，依赖模型内部表征。
- **区域裁切推理**（Zoom-IQA、Q-Probe）：裁出来的是原始像素块，与具体的感知属性没有显式关联，本质上还是交给模型"语义地看"。

> 这个论证与 ARCHITECTURE §14.2 的「降采样后哪些属性存活」是同一件事的两面，可以作为外部引证。

### 2.2 需求

- 证据必须**显式、结构化、对应具体感知属性**（不是裸像素块）。
- 模型要学会**怎么调用**工具，也要学会**什么时候调用**。
- 工具必须**确定性**：数据构造、SFT、RL 三个阶段对同一张图输出完全一致，否则策略学习不稳定。

### 2.3 技术路线与架构

```
原图 ──► MLLM（Qwen3-VL-4B）开始推理
              │  输出 <NoiseResidualMap> 这样的标签
              ▼
        工具在原始分辨率图上计算 ──► 证据图长边缩到 384 px ──► 作为下一轮图像输入
              │                                         （最多 6 轮）
              ▼
        <think>……引用证据图……</think><answer_start>{"score": X.XX}<answer_end>
```

**15 个工具，覆盖 7 个属性**（按代码注释与语义归类）：

| 属性 | 工具 | 实现要点 |
|---|---|---|
| 结构 | GradientMagnitudeMap / GradientMagnitudeHistogram / GradientOrientationCoherenceMap | Sobel 梯度幅值（固定截断归一化）、梯度直方图、结构张量方向一致性 |
| 锐度 | HighFrequencyResidualMap / DoGSharpnessMap / LocalGradientDeviationMap | 原图减高斯模糊（σ=3）；高斯差分（σ=1, 2.5）；梯度与其局部均值之差 |
| 噪声 | NoiseResidualMap | 轻度高频残差 × 结构抑制掩膜（压掉边缘，只留平坦区的残差） |
| 伪影 | FourierMagnitudeSpectrum | 整图 FFT 对数幅度谱（周期性伪影表现为亮点 / 亮线） |
| 亮度 | LuminanceHistogram / ExtremeLuminanceMap | 亮度直方图；≤30 或 ≥225 的像素二值掩膜 |
| 色彩 | PerceptualColorfulnessMap / ChromaticDeviationMap / ColorSaturationClippingMap | Hasler 色彩度；Lab 空间色度幅值 √(a²+b²)；HSV 饱和度 ≥245 的二值掩膜 |
| 自然度 | MSCNNormalizedLuminanceMap / MSCNDistributionHistogram | BRISQUE 所用的 MSCN 系数图及其分布 |

**训练**：
- **SFT**：Q-Tool 数据集 1.1 万条推理链（KonIQ 图像），证据图的 token 不计入 loss。
- **GRPO**：每个 prompt 采样 8 条，奖励 = 格式 + 分数准确度 `exp(−α|s_pred − s_gt|)` + 工具次数（0 次得 0；1~M 次得 1；超过 M 按平方衰减）+ 重复调用惩罚（同一工具调两次得 −1）。6×4090。

### 2.4 关键结果（原文核对后的数字）

**主表（全部只在 KonIQ 上训练，PLCC / SRCC）：**

| | KonIQ | SPAQ | LIVE-W | KADID | PIPAL | CSIQ | AGIQA | 平均 |
|---|---|---|---|---|---|---|---|---|
| DeQA（无推理） | **0.953**/0.941 | 0.895/0.896 | 0.892/0.879 | 0.694/0.687 | 0.472/0.478 | 0.787/0.744 | 0.809/0.729 | 0.786/0.765 |
| Zoom-IQA（区域裁切） | 0.938/0.922 | 0.902/0.900 | 0.887/0.870 | 0.701/0.700 | 0.468/0.465 | 0.797/0.754 | 0.816/0.765 | 0.787/0.768 |
| VisualQuality-R1 | 0.910/0.896 | 0.889/0.892 | 0.856/0.827 | **0.703/0.712** | 0.451/0.441 | 0.768/0.707 | 0.817/0.760 | 0.771/0.748 |
| **IQA-T1** | 0.942/0.925 | 0.905/0.908 | 0.888/0.860 | 0.686/0.708 | **0.480/0.490** | **0.850/0.840** | 0.816/0.754 | **0.795/0.784** |

**消融（七数据集平均 PLCC）：**
- SFT 时工具只返回占位符、不给证据图：0.705 → 给证据图：**0.756**（+0.051，两篇论文里最干净的一个"证据有用"的实验）
- 加上 RL 的工具次数奖励：0.790 → 再加重复惩罚：0.795

**调用策略（仅 KonIQ）：**

| 策略 | PLCC | 平均工具数 | 耗时 |
|---|---|---|---|
| 不用工具（Qwen3-VL-4B 基座） | 0.785 | 0 | 4.62 s |
| 15 个全给 | 0.865 | 15 | 6.06 s |
| 固定 3 个 | 0.895 | 3 | 5.32 s |
| 随机 3 个 | 0.871 | 3 | 5.42 s |
| **动态选择** | **0.942** | 2.34 | 5.09 s |

**换骨干（按表 4 自行计算的七数据集平均 PLCC）**：Qwen3-VL-2B ≈ 0.772，InternVL3.5-4B ≈ 0.762，Qwen3-VL-4B 0.795。2B 与 4B 差距不大。

**论文没有局限性章节。**

### 2.5 批判性阅读

1. **"低层证据"的故事在最该成立的地方没有成立。** KADID 是纯合成的低层失真，理论上是证据图最能发挥作用的数据集，但 IQA-T1 的 PLCC（0.686）比 Q-Insight、VisualQuality-R1、Zoom-IQA、DeQA 都低。它的优势集中在 CSIQ 和 PIPAL。平均分最高，但领先幅度很小（比 DeQA 高 0.009）。
2. **给得越多效果越差。** 15 个工具全给（0.865）不如固定给 3 个（0.895）——多余的证据图会稀释注意力。这条对 IQAgent 同样适用。
3. **解释的忠实性有漏洞（从开源训练数据里可以直接看到）：**
   - SFT 样本的目标分数**恰好等于真值 MOS**，推理链是在已知答案的情况下写的"事后理由"，而不是真正的推理过程。
   - 一条训练样本写道 *"ChromaticDeviationMap showing deviations toward red tones"*，但这个工具算的是 √(a²+b²) 色度幅值，**完全不含色相方向信息**，从图上根本看不出"偏红"。
   - 大多数证据图用的是**逐图 min-max 归一化**（`cv2.NORM_MINMAX`）：干净图和噪声很大的图，残差图都被拉满到 0–255。图只能说明问题**在哪里**，说明不了**有多严重**；而训练数据里却大量出现"低色彩强度""噪声明显"这类程度判断。
4. **最终分数仍然是 4B 的 MLLM 给的**，工具只是给它看的辅助输入，与 IQAgent"模型判定、LLM 转述"的分工正好相反。
5. **论文与代码的超参数对不上**：论文写奖励权重 1 / 5 / 1 / 2、工具上限 M=4；实际使用的配置 `compute_score_v3` 是 0.2 / 0.65 / 0.1 / 0.05，`max_tool_calls=3`，准确度衰减系数 2.0。**要复现以代码为准。**
6. 需要训练 VLM（SFT + RL），IQAgent 在 dsh 上调 DeepSeek API 的形态下走不了这条路。

### 2.6 对 IQAgent：抄 / 改 / 不抄

**✅ 直接抄（MIT 许可，保留版权声明即可）**

**(a) 15 个确定性证据工具 → `image_probe`。** 这是两篇论文里对 IQAgent 价值最大的单项。它直接对准你们现在的缺口：

| IQAgent 维度 | 状态 | 可用的 IQA-T1 工具 |
|---|---|---|
| 模糊 / 清晰度 | unavailable | DoGSharpnessMap、HighFrequencyResidualMap、GradientMagnitudeHistogram |
| 噪声 | unavailable | NoiseResidualMap |
| 压缩伪影 | unavailable | FourierMagnitudeSpectrum（JPEG 8×8 块效应在频谱上表现为规则的网格亮点；建议另加专门的块效应指标） |
| 曝光 | 有模型 | ExtremeLuminanceMap、LuminanceHistogram → 用作建议的辅助依据（例如"高光溢出占画面 x%"） |
| 色彩 | 有模型 | ColorSaturationClippingMap、PerceptualColorfulnessMap → 同上 |

两点定位不能忘：
- 它们仍然是**图像统计量，不是标定过的质量判定**，IRON RULE 2 照样适用；报告里要和模型判定分栏呈现。
- 但它们比"什么都不给"好得多，而且**证据图本身很适合 demo**：原图旁边放一张噪声残差热力图，比任何文字解释都直观。

**(b) 「原始分辨率计算 → 以低频证据图形式传给 LLM」这个模式。** 所有工具都先在原图上计算，再把证据图缩到长边 384 px。这把 ARCHITECTURE §15 的两条通路具体化了：
- 高频信息（噪声、锐度）在原始分辨率上被**先转换成空间分布图**，而空间分布在降采样后能够保留下来。
- 所以 LLM 的视觉通路**永远不需要看原始像素**，看"预先消化过的证据图"就够了。
- 这也是 §14.3 里 `vision_semantic` 角色做局部定位的好输入：例如 ExtremeLuminanceMap 直接标出过曝区域，可以据此生成 ROI。

**🔧 改了再用**

**(c) 去掉 min-max 归一化，改为固定量纲，并同时输出标量统计。** 这是抄工具时唯一必须改的地方，否则证据图无法跨图比较程度：

```python
def noise_residual(gray: np.ndarray, fixed_scale: float) -> tuple[np.ndarray, dict]:
    """IQA-T1 NoiseResidualMap 的改造版：固定量纲 + 标量统计。gray 为原始分辨率 float32 灰度图。"""
    low  = cv2.GaussianBlur(gray, (0, 0), 1.0)
    high = gray - low
    gx = cv2.Sobel(gray, cv2.CV_32F, 1, 0, ksize=3)
    gy = cv2.Sobel(gray, cv2.CV_32F, 0, 1, ksize=3)
    grad = cv2.GaussianBlur(np.sqrt(gx**2 + gy**2), (5, 5), 1.0)
    structure_mask = 1.0 - (grad / (grad.max() + 1e-6)) ** 0.6
    residual = np.abs(high) * structure_mask

    flat = structure_mask > 0.8                      # 只在平坦区统计，避开纹理
    stats = {
        "noise_p50":  float(np.percentile(residual[flat], 50)) if flat.any() else None,
        "noise_p95":  float(np.percentile(residual[flat], 95)) if flat.any() else None,
        "flat_ratio": float(flat.mean()),
    }
    # 固定量纲：fixed_scale 在参考图集上标定一次（例如全集 residual 的 99 分位），之后所有图共用
    vis = (np.clip(residual / fixed_scale, 0, 1) * 255).astype(np.uint8)
    return vis, stats                                 # stats 写入 Evidence Ledger；vis 缩到 384 px 后才给 LLM
```

- 标量统计进 Evidence Ledger，可被 `verify_claims` 引用；证据图只用于展示和定位。
- `fixed_scale` 按工具分别标定一次，写进配置。

**(d) 奖励函数的结构 → 改作 agent 轨迹的确定性检查项。** 你们不训练 LLM，用不上 GRPO，但它的四个奖励项可以原样变成对 dsh 会话日志的自动检查：输出格式是否合规、工具调用次数是否在预算内、是否重复调用同一工具、结论是否给出。直接加进 `eval/`。如果实验室以后要微调一个本地小模型来当规划器或解说员，这套奖励设计可以直接复用。

**(e) 「少而相关 > 多而全」。** 固定给 3 个工具胜过全部 15 个，这条直接支持 ARCHITECTURE §5.3 和 PRD 的受约束拆解：每轮给 LLM 的证据图设上限（建议 ≤3 张），由诊断计划决定给哪几张。

**❌ 不要抄**

- 让 LLM 输出最终质量分数。
- 用真值 MOS 反推理由的数据构造方式——得到的解释不忠实，上面已经给出了具体反例。
- 逐图 min-max 归一化的证据图（前提是你要用它表达程度）。

---

## 3. 落地清单（按里程碑）

| 里程碑 | 动作 | 来源 | 工作量 |
|---|---|---|---|
| **M0** | `iqa-plan` 采用改写后的 Planner schema；写 `validate_plan` | AgenticIQA §A.2 | 小 |
| **M0** | 核对许可证：IQAgent 不引入 IQA-PyTorch（非商用）；IQA-T1 工具代码保留 MIT 声明 | 两篇 | 极小 |
| **M1** | 移植 IQA-T1 的 15 个工具到 `image_probe`，改固定量纲 + 标量统计 | IQA-T1 `scripts/tools.py` | 中 |
| **M1** | `calibration.yaml` 第一级改为逐模型拟合的五参数 logistic | AgenticIQA §A.4（用标准公式） | 小 |
| **M1** | 每个属性模型按场景 / 属性分组测 SROCC，填 `best_at` 与 `validation` | AgenticIQA §A.3 的方法 | 中（需要留出集） |
| **M1** | `eval/` 增加规划轨，以及基于轨迹的格式 / 次数 / 重复检查 | AgenticIQA-Eval + IQA-T1 奖励项 | 小 |
| **M2** | demo 中展示证据图（原图 + 噪声残差 / 过曝掩膜并排） | IQA-T1 | 小 |
| **M2** | 证据图作为 `vision_semantic` 的 ROI 定位输入，每轮 ≤3 张 | IQA-T1 表 3 的结论 | 中 |

---

## 4. 附：本次核对中发现的摘要错误

阅读时先用网页摘要工具抓取过一轮，逐段对照原文后发现以下错误（记录下来，避免后续引用出错）：

- AgenticIQA「在所有数据集上达到 SOTA」——**不实**，BID 和 AGIQA-3K 输给 Q-Align。
- AgenticIQA 局限性「6 条」——**不实**，原文只有 3 条。
- IQA-T1 换骨干的平均 PLCC「2B 为 0.741、InternVL 为 0.699」——**不实**，按原表计算分别约为 0.772 和 0.762。
- IQA-T1 奖励权重与工具上限——论文数值与开源代码实际使用的配置不一致（见 §2.5 第 5 条）。

**来源**
- AgenticIQA：https://arxiv.org/html/2509.26006v2
- IQA-T1：https://arxiv.org/html/2607.12375v1 · 代码 https://github.com/zibuyu-02/IQA-T1（MIT）
- IQA-PyTorch 许可证：https://github.com/chaofengc/IQA-PyTorch/blob/main/LICENSE（PolyForm Noncommercial 1.0.0）
