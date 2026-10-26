# 图像质量评价（IQA）商业竞品调研
---

## 0. 调研说明

### 0.1 方法
### 0.2 一个前置澄清：商业界的「IQA」其实是四件不同的事

调研中最先浮现的结论是：把这些产品放在一起看，「图像质量评价」在商业上并不是一个市场，而是**四个客户完全不同的市场**：

| 类型 | 问的问题 | 典型客户 | 代表产品 |
|---|---|---|---|
| **I. 器件评测型** | 这台相机/这条 ISP 管线好不好？ | 手机厂、Tier-1、模组厂 | DXOMARK Analyzer、Imatest、iQ-Analyzer-X |
| **II. 内容筛选型** | 这批图里哪些能用、哪些该丢？ | 内容平台、摄影师、电商、AI 数据团队 | Aftershoot、Sightengine、FiftyOne、腾讯云 TIIA |
| **III. 采集合规型** | 这张图达标没有？不达标怎么重拍？ | 身份核验、KYC、银行、政务 | OFIQ、NFIQ 2、Regula、Veriff |
| **IV. 传输保真型** | 压完之后画质掉了多少？ | 流媒体、CDN、视频云 | SSIMPLUS、Beamr、Cloudinary q_auto |

实验室现有的能力（整体质量 + 曝光/对比度/白平衡/色彩的属性级打分 + 文字解释）在坐标系上**跨在 I 和 II 之间**：属性维度像 I 型，单张图 + 文字解释像 II 型。这个定位判断贯穿后文所有洞察。

---

## 1. 分类总览

| # | 类别 | 代表厂商 | 部署形态 | 商业模式 | 属性级细度 | 文字解释 | 改进建议 |
|---|---|---|---|---|---|---|---|
| A | 影像实验室 / 画质测试 | DXOMARK、Imatest、Image Engineering | 本地软件 + 实验室硬件 | 卖软件许可 + 卖设备 + 卖服务 | ★★★★★ | 数值+图表为主 | 以咨询服务形式 |
| B | 云厂商画质评分 API | 阿里云、腾讯云、百度、火山、华为、AWS、Sightengine、Everypixel | 云 API | 按调用量 | ★★☆☆☆ | 几乎没有 | 没有 |
| C | 图片处理 / CDN | Cloudinary、imgix、Uploadcare、七牛、又拍云 | 云服务 / CDN | 按流量+存储+转换次数 | ★☆☆☆☆（质量是手段不是产品） | 没有 | 自动替你做了 |
| D | 摄影工作流（选片/修图） | Aftershoot、Imagen、Narrative、FilterPixel、Evoto、Lightroom | 桌面软件 + 云 | 订阅 / 按张计费 | ★★★☆☆ | 标签式 | 没有（直接给结果） |
| E | 视频质量 | IMAX(SSIMWAVE)、Beamr、Interra、Telestream、Mux、Conviva、腾讯/阿里/火山 | 本地/云/SaaS | 企业许可 + SaaS | ★★★☆☆ | 报告式 | 有（码率/档位建议） |
| F | 证件 / 身份影像质量 | OFIQ、NFIQ 2、Regula、Veriff、Jumio、Kofax | SDK / 本地库 / 开源 | 卖 SDK + 按核验次数 | ★★★★★ | 结构化枚举 | **有，且是核心卖点** |
| G | 电商 / UGC / 图库审核 | Shutterstock、Adobe Stock、Amazon、内容审核厂商 | 人审 + 机审混合 | 平台规则（非售卖） | ★★★★☆（规则维度全） | 有（拒稿理由） | 有（知识库文章） |
| H | 数据集质量治理 | Voxel51 FiftyOne、Cleanlab、Labelbox | 开源 + 企业版 | 开源引流 + 企业订阅 | ★★★☆☆ | 没有 | 给操作（打标签、筛除） |
| I | 手机 / ISP 产业链 | Qualcomm、MediaTek、ArcSoft、Morpho | 内部工具 / IP 授权 | 芯片绑定 / IP 授权 | 不公开 | — | — |
| J | AIGC 图像质量评估 | Artificial Analysis、LLM-as-judge 平台 | 榜单 / 评测 SaaS | 榜单免费 + 咨询/API | ★★☆☆☆ | 有（LLM 生成） | 有 |
| K | **消费级 AI 摄影点评**（额外发现） | Adobe Project Indigo、PhotoCritic、Evalens、PixMentor | App / Web | 免费 + 订阅 | ★★★☆☆ | **有，且是全部卖点** | **有** |

补充发现的相邻类别（本文只做简要说明）：文化遗产数字化质量（iQ-Analyzer-X CH）、汽车影像质量标准（IEEE 2020-2024）、显示/音频等跨域 benchmark（DXOMARK Display/Audio/Battery）。

---

## 2. A 类：影像实验室 / 画质测试

这一类是**属性级评价做得最细、最权威的一档**，也是与实验室能力最接近的一类。

### 2.1 DXOMARK

法国公司，从 DxO Labs 分拆。它同时经营三件生意：公开榜单（品牌）、测试软硬件（产品）、调优咨询（服务）。

**2.1.1 Analyzer（测试软件套件）**
- 链接：<https://corp.dxomark.com/analyzer/>
- 面向：手机厂、模组厂、汽车 Tier-1、监控/医疗/航天影像团队；用户是**影像工程师**。
- 覆盖维度（官网分组）：Resolution & Sharpness（MTF、模糊、纹理）、Color（保真/一致性/跨光源稳定性）、Noise & Tone、Dynamic Range & Contrast（含局部对比度）、**3A（自动曝光 / 自动白平衡 / 自动对焦）**、Flare、Motion（运动模糊/帧间一致性）、Timing（曝光时长、帧率、同步延迟）、Depth、Geometry（畸变、边缘锐度）、Real-life Scenes（**感知指标，对齐人眼**）、Flicker。
- 输入输出：单图 / 批量 / 视频都支持，需要配套**实体测试图卡 + 受控灯箱**（DXOMARK Chart、HDR Noise、Dead leaves、ColorChecker；灯光稳定系统、六足机器人、动态场景装置）。
- 输出形态：Python 字典、Excel、HTML、PDF、AZV（JSON 兼容）。**能导出、能跨测试对比**。
- 版本分层：AZ FREE（核心指标 GUI）、AZ GUI（基础+进阶）、**AZ API（Python 编程接口）**、AZ PREMIUM（GUI+API+视频指标）。
- 部署：本地软件 + 实验室硬件。官方称全球已装 **150+ 实验室**。
- 文字解释/建议：以**数值 + 图表 + 文档化的数值参考区间**为主，不生成自然语言解释。解释与建议是通过下面的咨询服务交付的。
- 定价：不公开，询价制（待核实具体价位）。

**2.1.2 公开榜单的属性级拆分（最值得参考的一张「属性树」）**
- 链接：<https://www.dxomark.com/what-we-test-camera/>
- 相机总分 = Photo（主摄 47%）+ Video（27%）+ Photo Bokeh（4%）+ Photo Telephoto（13%）+ Photo Ultra-Wide（3%）+ Video Telephoto（5%）+ Video Ultra-Wide（1%）。
- Photo 下的六个属性：**Exposure、Color、Autofocus、Texture、Noise、Artifacts**。Video 在这六项基础上多一项 **Stabilization**。
- 每台设备约拍摄分析 **4000 张照片 + 200 分钟视频**，并用**多文化焦点小组**确定「可接受的渲染」基准。
- 这意味着：DXOMARK 的属性级评价是**分数 + 权重 + 层级树**，且每个属性在评测文章里配有大段自然语言解读和样张对比。**面向读者的那一层是有文字解释的，面向工程师的软件那一层没有。**

**2.1.3 Camera ISP Optimization / ISP Tuning Services**
- 链接：<https://corp.dxomark.com/catalog/testing-tools/camera-isp-optimization/>、<https://corp.dxomark.com/catalog/services/testing-benchmarking/tuning-services/>
- 卖的是「实验室包 + 方法论 + 人」：受控测试环境、校准图卡、精确灯光、Analyzer 软件，支持 RAW 与处理后图像（以区分 sensor 行为和 ISP 影响）。
- 覆盖：降噪、纹理、锐度、色彩、HDR、时域稳定性。
- 交付：四阶段的 KPI 驱动迭代流程，DXOMARK 工程师参与制定测试协议。
- 这是**「解释 + 建议」在 A 类的真实商业形态：不做进软件，做成服务卖**。

**2.1.4 GEA（Golden Eye Assistant）与 AIOP（All-in-One Portrait）**
- 链接：<https://corp.dxomark.com/news/dxomark-at-mwc-2026-exploring-the-future-of-mobile-imaging/>；产品目录页 <https://corp.dxomark.com/catalog/testing-tools/product-type/charts-equipements/golden-eye-assistant-gea/>（本次抓取返回 502，**详细规格待核实**）。
- **GEA**：官方定义为「AI-powered image quality assessment tool that enhances perceptual evaluation, enabling faster and more scalable analysis」，定位是让**感知评价**更快、更一致、可规模化。目标用户是工程团队。
  - **这是本次调研里，权威影像实验室把 AI 用于「替代/加速人眼主观评价」最明确的公开信号。**
  - 但官方未公开：是否基于多模态大模型、是否输出文字、覆盖哪些属性、如何定价。**全部待核实。**
- **AIOP**：把人像质量的关键指标整合进一个框架——**肤色还原、细节、虚化（bokeh）、曝光、色彩**。是典型的「按属性拆解 + 打包成一个场景化评估」的产品思路。

### 2.2 Imatest

- 链接：<https://www.imatest.com/>、<https://www.imatest.com/solutions/image-quality-factors/>
- 面向：相机/模组/传感器研发与产线测试工程师；比 DXOMARK 更偏工具、更平价、更工程师自助。
- 覆盖维度（官网 Image Quality Factors）：**Sharpness (MTF)、Noise、Dynamic Range、Color Accuracy、Distortion、Uniformity（暗角/shading）、Chromatic Aberration、Stray Light (Flare)、Color Moiré、Artifacts、ISO Sensitivity、Texture Detail、Blemishes**，另有 Flicker。支持 30+ 种标准图卡（SFRplus、eSFR ISO、SFRreg、Checkerboard 等）。
- 产品线：**Master**（分析套件）、**IS**（实时采集）、**IT**（自动化/产线集成）。
- 可扩展性：**IT 提供 API，含 C/C++ DLL、命令行 EXE、Python 模块、.NET DLL 四种接口**，功能与 Master/IS 模块对齐但无需人工干预，专为自动化产线设计。这是 A 类里「可编程接入」做得最直白的。
- 定价：订阅（2021-06 起）与永久许可并行，永久含 1 年支持；批量折扣（Master 订阅 4–9 套减 20%，10 套以上减 30%）。第三方经销商列出 **IT 单机 $5,300 / 浮动许可 $8,480**（来源为 findlight 非官方页，**待核实**）；官网本身为询价制。
- 文字解释/建议：无自然语言解释；靠文档和大量的技术知识库（Imatest 的 documentation 本身就是行业教材）。
- 权威背书：官方维护 **IEEE 2020（汽车影像质量）** 的对应说明页 <https://www.imatest.com/imaging/ieee-2020/>。

### 2.3 Image Engineering（iQ-Analyzer-X）

- 链接：<https://www.image-engineering.de/en/products/software/iq-analyzer-x/>
- 德国公司，同时做图卡、灯箱和分析软件。
- 覆盖维度：**resolution (SFR)、texture loss、dynamic range (OECF)、color accuracy、lens distortion、shading、flare、low-light、noise**。
- 技术形态：全 C++ 重写；**AI 驱动的图卡自动检测与 ROI 定位**（这是它的差异点——用 AI 来提速测量流程，而不是用 AI 来做质量判断）。
- 输入：图片与视频文件，支持外接 webcam、内置相机、live stream、capture server。
- 自动化：hot folder + 命令行。
- **结果与测试参数存入本地或服务器数据库，支持跨实验室、跨时间比对**——「可对比」在 A 类是硬需求。
- 版本：Free（无加密狗，功能受限）/ Pro（USB 加密狗，可装无限台设备，支持网络许可）/ **CH（Cultural Heritage 文化遗产版，功能裁剪、价格更低）**。
- 定价：不公开，询价制（**待核实**）。
- CH 版指向一个我原本没列的细分市场：**博物馆/档案馆数字化的扫描质量合规**（对应 FADGI、Metamorfoze、ISO 19264 等规范，**这些标准的细节本次未核实**）。

### 2.4 小结：A 类的共性

1. **属性级拆分最细，但几乎不产生自然语言。** 输出是数值、曲线、热图、PDF 报告。
2. **必须配硬件。** 图卡 + 灯箱是刚需，因为它们测的是「设备」不是「图片」——没有受控输入就没有可比的测量。
3. **解释和建议以「服务」交付**，不是软件功能。
4. **可对比、可归档、可跨实验室复现，是刚需而不是加分项。**
5. **商业模式是三层叠加**：卖软件许可 + 卖实验室设备 + 卖咨询服务；DXOMARK 还多一层「品牌榜单」。

---

## 3. B 类：云厂商与独立厂商的画质评分 API

这一类是**最「薄」的一类**：几乎都只给一到两个数，没有解释，没有建议。

### 3.1 中国云厂商

| 厂商/产品 | 接口 | 输出维度 | 链接 |
|---|---|---|---|
| 阿里云 视觉智能开放平台（VIAPI） | **AssessSharpness** | 单一清晰度评分（越高越清晰） | [文档](https://help.aliyun.com/zh/viapi/developer-reference/api-image-definition-score) |
| 阿里云 VIAPI | **EvaluateVideoQuality** | **本次调研中维度最全的国产 API**：主观画质评分；客观画质（清晰度、点状噪声、压缩噪声）；颜色质量（饱和度、丰富度、对比度、亮度）；瑕疵（场纹、划痕、卡帧、闪烁、黑屏花屏） | [文档](https://help.aliyun.com/zh/viapi/developer-reference/api-video-quality-assessment) |
| 阿里云 智能媒体管理 IMM / OSS | 图片质量评分 | **只返回 `OverallQualityScore`（0–1）**，文档称内部综合了清晰度、噪声失真、色彩饱和度、曝光度等，但**不分维度返回** | [IMM](https://help.aliyun.com/zh/imm/user-guide/image-quality-evaluation-1)、[OSS](https://help.aliyun.com/zh/oss/user-guide/image-quality-score) |
| 腾讯云 TIIA 图像分析 | **AssessQuality** | `ClarityScore`(0–100，综合噪声/曝光/模糊/压缩)、`AestheticScore`(0–100，构图/色彩)、`LongImage`、`BlackAndWhite`、`SmallImage`、`BigImage`、`PureImage` | [文档](https://cloud.tencent.com/document/api/865/36899) |
| 腾讯云 数据万象 CI | 图片质量评分 | 同上，绑定 COS 存储桶使用 | [文档](https://cloud.tencent.com/document/product/460/84019) |
| 百度智能云 BOS | 图像质量识别 `quality` | `aesthetic`、`clarity` 两个 double（**取值范围文档未明示，待核实**） | [文档](https://cloud.baidu.com/doc/BOS/s/qkce7ewdi) |
| 火山引擎 veImageX | 画质评估组件 | **大众美学评分、噪声强度评分、纹理丰富度评分、色调均衡程度评分**（本次抓取文档页为空，指标名来自官方文档标题与官方技术文章，**具体取值范围待核实**） | [文档](https://docs.volcengine.com/docs/508/111896) |
| 火山引擎 视频点播 | **VQScore** | 无参考视频/图像质量评分 0–100，对标 VMAF 但不需参考；用于端到端优化、质量监控、低质筛查 | [文档](https://www.volcengine.com/docs/4/337732) |
| 华为云 内容审核 Moderation | `clarity-detect` | **清晰/模糊二分类**（面向表单、票据等原始图像），另有构图质量相关能力（**待核实**） | [产品页](https://www.huaweicloud.com/product/moderation.html) |

**观察：**
- 国产云的画质 API 基本挂在**内容审核 / 对象存储 / 媒体处理**之下，是附属能力，不是独立产品。
- **「清晰度 + 美观度」是国产云的事实标准双指标组合**（腾讯、百度都是这个形态）。
- **曝光、白平衡、对比度、色彩基本不单独暴露**——它们被折叠进了「清晰度」或「综合评分」里。阿里云的 EvaluateVideoQuality 是唯一把颜色相关拆成四项（饱和度/丰富度/对比度/亮度）的。
- 计费：阿里云支持按量付费、预付费资源包、预付费 QPS 三种；腾讯云「图片质量评估费用」是内容识别下的一个计费项，按次计（**具体单价待核实**）。

### 3.2 国际云厂商

- **AWS Rekognition**：两处
  - `DetectFaces` 返回人脸的 `Quality`：**Sharpness + Brightness**，各 0–1；`QualityFilter` 可过滤过小/过糊/过暗的人脸。
  - `DetectLabels` 加 `IMAGE_PROPERTIES` 参数，返回 **Brightness / Sharpness / Contrast，各 0–100**，以及主色调（前景/背景分开）。
  - 链接：[Image Properties](https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectLabelsImageQuality.html)、[Detecting faces](https://docs.aws.amazon.com/rekognition/latest/dg/faces-detect-images.html)
  - **AWS 是国际大厂里唯一把 brightness/sharpness/contrast 三个属性分开返回的。**
- **Google Cloud Vision**：**没有画质评分或模糊检测**。Image Properties 只返回主色调与裁剪建议。[文档](https://cloud.google.com/vision/docs/detecting-properties)
- **Azure AI Vision**：**没有模糊/曝光/质量分**。官方与社区的建议是用 Custom Vision 自己训一个 Pass/Fail 分类器。[Image Analysis 概览](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-image-analysis)
- **这是一个很强的信号：Google 和 Azure 判断「通用画质评分」不值得做成平台能力。**

### 3.3 独立 API 厂商

| 厂商 | 输出 | 解释/建议 | 定价 | 链接 |
|---|---|---|---|---|
| **Sightengine** | `quality.score` **单一 0–1 分**；另有 Image Properties 单独返回 **sharpness / contrast / brightness**；还有 color detection（主色/点缀色/其他色，RGB+HEX） | **无任何文字解释**，文档只给分段释义表（<0.25 很差、0.25–0.45 差到尚可、0.45–0.60 尚可到好、0.60–0.85 好到很好、>0.85 很好到完美），并建议用户自己设 0.5 阈值 | 免费 2000 次/月；Starter $29/月含 1 万次；Pro $99/月含 4 万次；超出 $0.002/次；**Enterprise 支持 custom models** | [质量检测](https://sightengine.com/docs/image-quality-detection)、[图像属性](https://sightengine.com/docs/image-properties)、[定价](https://sightengine.com/pricing) |
| **Everypixel** | **0–100 分 + 文字等级标签**（Poor / Good / Excellent）；**区分两个模型**：`/ugc_photo_quality`（UGC）与 `/stock_photo_quality`（图库） | 只有等级标签，无解释 | 免费 500 次；$0.6 / 1000 次，按量付费无订阅 | [Photo Quality](https://labs.everypixel.com/photo-quality)、[API](https://labs.everypixel.com/api) |
| **APILayer Image Quality** | 直接跑 **BRISQUE** 算法给一个分 | 无 | 有免费档 | [Marketplace](https://apilayer.com/marketplace/image_quality-api) |
| **Imagga** | **未发现专门的画质评分端点**（只有 tagging、color extraction、cropping、moderation）。**待核实是否有未公开能力** | — | — | [文档](https://docs.imagga.com/) |
| **Clarifai** | 无现成画质模型，但**平台本身的卖点就是「接入你自己的模型」**：支持从 HuggingFace 或本地上传自定义模型，自动部署，可与其他模型和 **agent operators** 组成 workflow | — | 企业版才开放本地模型上传 | [Model Upload](https://docs.clarifai.com/compute/upload/)、[Workflows](https://docs.clarifai.com/create/workflows/create/) |

**关键发现：**
- **Everypixel 的「按场景分模型」（UGC vs Stock）是这一类里少有的产品化思考**——它承认「好照片」的标准依赖使用场景。
- **Sightengine 的分段释义表是「最低成本的解释」**：不生成文字，但告诉你这个数字该怎么读。这是一个值得注意的折中形态。
- **B 类没有任何一家提供改进建议。**

---

## 4. C 类：图片处理与 CDN

这一类的本质是：**画质评价不是卖给客户的产品，而是内部用来自动决定压缩参数的手段。** 客户买的是「小文件 + 看不出差别」，不是分数。

- **Cloudinary**
  - `q_auto` 分四档 `:best / :good（默认）/ :eco / :low`。机制是：启发式 + 试编码一两次 + 用自研 SSIM 变体快速校验，为**每张图**定制编码参数。
  - 自研感知指标 **SSIMULACRA / SSIMULACRA 2**（Structural SIMilarity Unveiling Local And Compression Related Artifacts），基于多尺度 SSIM，在 CIE Lab* 空间计算，**专门针对压缩伪影而非相机标定类失真**调过。**已开源**：<https://github.com/cloudinary/ssimulacra2>
  - 链接：[图像优化文档](https://cloudinary.com/documentation/image_optimization)、[SSIMULACRA 博客](https://cloudinary.com/blog/detecting_the_psychovisual_impact_of_compression_related_artifacts_using_ssimulacra)
  - **Cloudinary 是本次调研中 agent 化走得最远的图片厂商**：发布了 **5 个 MCP Server**（Asset Management、Environment Config、Structured Metadata、**Analysis**、MediaFlows），可被 Cursor / Claude Desktop / Claude Code / VS Code 调用；2026 年 5 月推出 **Cloudinary Agents**，面向企业视觉媒体团队做分类、搜索、审核、流程自动化的多步操作。链接：[AI Agent Tools & MCP](https://cloudinary.com/documentation/cloudinary_llm_mcp)
- **imgix**：`auto=compress` 把质量降到 q=45、剥离元数据、按透明度选 PNG/JPEG；可与 `auto=format`、`auto=enhance` 组合。没有质量评分对外。[文档](https://docs.imgix.com/en-US/apis/rendering/automatic)
- **Uploadcare**：**Adaptive Quality**——「分析图像，选择刚好在可见伪影出现之前的压缩档位」；质量预设 normal / better / best。默认在 Dashboard 的 Delivery 中开启。没有对外的质量分。[文档](https://uploadcare.com/docs/optimization/)
- **七牛云**：`imageView2` 有 quality 参数；有内容审核（鉴黄/暴恐/敏感人物）和图像超分（去压缩模糊、增强纹理）。**未发现对外的图片质量评分 API（待核实）**。[dora 文档](https://developer.qiniu.com/dora/api/3683/img)
- **又拍云**：图片处理 quality 默认 75；有内容审核鉴黄。**未发现质量评分 API（待核实）**。[文档](http://docs.upyun.com/cloud/image/)

**观察：** C 类唯一暴露给外部的「质量语言」是**档位名**（best/good/eco/low、normal/better/best）。这是一个很实用的产品提示：**普通用户要的不是分数，是档位。**

---

## 5. D 类：摄影工作流（选片与修图）

这一类是**属性级评价最贴近终端用户、且已经完全跑通商业闭环的一类**。客户是职业摄影师，付费动机极其明确：一场婚礼 3000–5000 张，人工选片 6–10 小时。

| 产品 | 覆盖维度 | 输出形态 | 解释 | 建议 | 定价 | 链接 |
|---|---|---|---|---|---|---|
| **Aftershoot** | 闭眼、模糊、重复；以及「主观项」：光线、背景干净度、构图 | **每张 1–100 分**；分桶：Selected / Highlights / Closed Eyes / Blur / Duplicates；**模糊敏感度和重复相似度都有滑块可调** | 分桶名即解释 | 无 | AI Culling $10/月（年付），**照片数不限、不按张计费** | [Culling FAQ](https://aftershoot.com/culling-faq/)、[Selects](https://aftershoot.com/selects/) |
| **Narrative Select** | **对焦评分 0–100**；眼睛状态（闭眼/眨眼/低头/接吻）；场景内按锐度排序 | Close-Ups 面板自动放大到人脸；无人脸时用 **Key Element Detection** 找主体；Scenes View 按锐度排序自动顶出最佳帧；一键 Ship to Lightroom | 标签式 | 无 | Lite $10/月（年付）到 Ultra $60/月，**按功能分层** | [Select](https://narrative.so/select)、[定价](https://narrative.so/pricing) |
| **Imagen AI** | 选片：合焦、闭眼、表情、重复；修图：**Personal AI Profile 用你自己的 3000+ 张历史修片训练**，复刻你的风格 | 分组 + 高亮最佳 | 无 | 无 | 修图 $0.05/张，$7/月起；年付 $59/月(1.8万张)–$199/月(7.2万张)；选片 $12/月起；AI Crop、Talent AI 各 +$0.01/张 | [官网](https://imagen-ai.com/)、[定价](https://imagen-ai.com/pricing/) |
| **FilterPixel** | **单次通过检查全部**：锐度与对焦精度、运动模糊、手抖、眨眼、表情质量、**曝光质量（高光溢出/暗部死黑）**、重复相似度、构图取景；**DeepCull** 模式做题材特化（婚礼等）的「高潮瞬间/情绪/故事」判断 | 分数 + 分组；**敏感度阈值可按拍摄场景调整** | 无 | 无 | $19/月（选片）、$35/月（选片+基础修图） | [DeepCull](https://filterpixel.com/deepcull)、[AI Culling](https://filterpixel.com/ai-photo-culling-software) |
| **Evoto** | AI Culling（2025 秋上线）标记模糊、曝光不佳等；核心是批量人像精修 | 标记 + 分组 | 无 | 无 | **按导出张数扣 credit**，约 4.6–6.9 美分/张，套餐越大越便宜 | [官网](https://www.evoto.ai/)、[Credits 说明](https://support.evoto.ai/understanding-evoto-credits-and-pricing/) |
| **Adobe Lightroom Classic — Assisted Culling** | 选择条件：主体对焦、眼部对焦、眼睛睁开、图像质量；剔除条件：曝光问题、误拍（misfires）、文档/票据 | 左右面板显示筛选项与**质量指标（主体锐度、眼部锐度、睁眼检测）**；**可展开看每张的详细评分，理解 AI 为什么这么判** | 分数明细 | 无 | 含在 Creative Cloud 订阅中（2025-10 起 Early Access） | [帮助文档](https://helpx.adobe.com/lightroom-classic/help/assisted-culling.html) |
| **Photo Mechanic (Plus)** | **无 AI 质量判断**，纯靠读嵌入式 JPEG 预览带来的浏览速度 | — | — | — | $24.99/月 或 $249/年；永久 $299 / Plus $399 | [Capterra 页](https://www.capterra.com/p/210470/Photo-Mechanic/) |

**关键洞察：**
1. **这类产品全部不给改进建议。** 原因很实在：照片已经拍完了，建议没有用。它们的价值是**替你做决策**，不是教你。
2. **「敏感度滑块」是这一类的标配。** Aftershoot、FilterPixel 都让用户调阈值——**厂商承认质量标准因人而异，把最终判断权还给用户**。这是一个非常重要的产品设计共识。
3. **Imagen 的 Personal AI Profile 是「可扩展/个性化」的最强形态**：不是接入你的模型，而是**用你的数据训一个属于你的模型**。
4. **Adobe 的「可展开看详细评分」是可解释性的最低成本实现**：不写文字，但把分数摊开给你看。
5. **Photo Mechanic 在 2026 年仍然没有 AI，却还活着且不便宜**——说明这个市场里「速度」本身就是可以独立售卖的价值。

---

## 6. E 类：视频质量

### 6.1 感知质量度量（卖「一个可信的数」）

- **IMAX / SSIMWAVE — SSIMPLUS**（2022 年 SSIMWAVE 被 IMAX 收购，现为 IMAX VisionScience）
  - **Viewer Score 0–100**，每 20 分一档分为 bad / poor / fair / good / excellent。
  - **设备感知**：同一段视频在电视和手机上给出不同分数——厂商声称 PSNR 和 VMAF 都做不到这点。
  - 厂商声称与人眼主观评分相关性 **>90%**，速度比 VMAF 快 **10 倍**（**均为厂商声称，未独立验证**）。
  - 品牌营销手法与 DXOMARK 同构：连续多年**公开评测超级碗直播/OTT 的画质成绩单**，把度量变成品牌。
  - 链接：[产品文档](https://docs.ssimwave.com/)、[SSIMPLUS 介绍](https://blog.ssimwave.com/ssimplus-video-quality-measure)、[IMAX ViewerScore 博客](https://medium.com/imaxtechnology/the-ssimplus-score-2fd52bbd9578)
- **Beamr — CABR**：用自研感知质量度量在**帧级**挑「码率最低但感知不劣化」的候选编码；53 项专利，经大规模主观测试验证；宣称码率降低 30–50%。质量度量本身不单独售卖，是编码器的一部分。链接：<https://beamr.com/cabr>
- **火山引擎 VQScore**：无参考，0–100，用于端到端优化、质量监控、推荐排序、低质筛查。

### 6.2 文件级 QC（卖「合规报告」）

- **Interra Systems BATON**：行业标准级的自动化音视频 QC 平台，AI/ML 增强。检查内容包括格式合规、黑场/静帧、响度、以及**基于神经网络的音视频质量检查、语言识别、伪影检测**；支持**自动 + 人工混合 QC 在同一 workflow 里**。链接：<https://www.interrasystems.com/file-based-qc.php>
- **Telestream**：提供从贡献到分发的监测与深度诊断产品线（Qualify 的具体指标清单**待核实**）。链接：<https://www.telestream.com/measurement-overview/>

### 6.3 播放端 QoE（卖「运营可见性」）

- **Mux Data**：**Overall Viewer Experience 0–100**，由四个子分构成：**Playback Success、Startup Time、Playback Smoothness、Video Quality**。另有 **Negative Impact Score** 直接把「哪个问题对整体 QoE 伤害最大」排出来。**这是「属性级 + 影响归因」在商业产品里最清晰的一个例子。** 链接：[Overall Viewer Experience](https://www.mux.com/docs/guides/data-overall-viewer-experience-metric)
- **Conviva**：实时 QoE 分析，默认采集 50+ 指标，支持自定义元数据扩展；宣称日处理 5 万亿事件。链接：<https://www.conviva.com/>
- （Bitmovin Analytics、NPAW 属同类，本次未逐项核实，**待核实**。）

### 6.4 中国视频云的「修复增强」路线

- **腾讯云 音画质重生**（2023-04 上线）：流程是 **智能分析 → 画质修复 → 画质增强 → 智能转码**。
  - 智能分析提供**帧级 / 场景级 / 视频级**三种粒度。
  - 画质修复：去压缩失真、视频降噪、人脸修复、去划痕。
  - 画质增强：超分辨率重建、细节增强、色彩增强、SDR 转 HDR。
  - 链接：[云点播文档](https://cloud.tencent.com/document/product/266/89538)
  - **注意这里的产品逻辑：分析不是为了给用户看分数，是为了决定修复策略。**
- **阿里云 窄带高清 2.0**：把编码器的优化目标从「最大保真」改为「主观体验最优」，包含基于主观质量的码控与自适应量化、帧间一致性、编码伪影抑制、纹理增强。上线前用 30 人主观打分验证。链接：<https://promotion.aliyun.com/ntms/act/mtsnarrowband.html>

---

## 7. F 类：证件 / 身份 / 票据影像质量

**这是本次调研最重要的发现所在。** 这一类把「属性级评价 + 文字解释 + 改进建议」做到了商业上最成熟的程度，因为它有一个别的类别都没有的东西：**明确的、可仲裁的行业标准，以及一个必须当场纠正的用户。**

### 7.1 OFIQ + ISO/IEC 29794-5（人脸图像质量）

- **ISO/IEC 29794-5:2025** 于 2025 年 4 月发布，定义了人脸图像的**统一质量分 + 大量质量分量（quality components）**。链接：<https://www.iso.org/standard/81005.html>
- **OFIQ（Open Source Face Image Quality）** 由德国联邦信息安全办公室 **BSI** 开发，是该标准的**参考实现，开源，可商用可政用**。链接：[BSI 页面](https://www.bsi.bund.de/EN/Themen/Unternehmen-und-Organisationen/Informationen-und-Empfehlungen/Freie-Software/OFIQ/OFIQ_node.html)、[GitHub](https://github.com/BSI-OFIQ/OFIQ-Project)
- **输出形态：一个向量** = 统一质量分（整数 0–100，越高越好，且**该分数被设计为可预测识别性能**）+ **25~27 个独立质量分量**（不同资料说法为 25+ / 27，**确切数量待核实**）。
- **分量按来源分成两组，这个分法极具参考价值：**
  - **采集相关（capture-related）**：光照均匀性、锐度/清晰度、曝光等——**由拍摄条件决定，可以通过重拍改善**。
  - **主体相关（subject-related）**：姿态、表情、遮挡、睁眼等——**由被拍者决定**。
- **NIST** 通过 **FATE SIDD** 评测活动，**按每个质量分量分别测试各家算法**——也就是说，属性级评价在这个领域有**独立第三方的、分维度的公开评测**。链接：<https://pages.nist.gov/ifpc/>

### 7.2 NFIQ 2（指纹图像质量）

- NIST 出品，开源，**ISO/IEC 29794-4 的参考实现**。质量分 **0–100**（老版 NFIQ 只有 1–5 五档）。
- 核心理念被写进标准：**质量分必须与「操作中的识别性能」数值校准过**，这样全球不同厂商的传感器给出的质量值才可互相解读。
- 由 NIST + 德国 BSI + BKA + MITRE + Fraunhofer IGD + h_da + Secunet 联合开发。
- 链接：[NIST 页面](https://www.nist.gov/services-resources/software/nfiq-2)、[GitHub](https://github.com/usnistgov/NFIQ2)

> **这两个是整个调研里「属性级质量评价」权威性最高的样本：开源 + 国际标准 + 政府机构背书 + 独立第三方分维度评测 + 分数与下游任务性能校准。**

### 7.3 商业 SDK 与 KYC 厂商

- **Regula Document Reader SDK**
  - 三项图像质量检查：**resolution（分辨率/PPI）、focus（对焦/模糊）、glares（反光眩光）**。
  - **输出不是分数，是三值枚举**：`resolution enough` / `resolution not enough` / `check not performed`（focus 同构）。
  - 有硬门槛：**至少 200 PPI**、证件不能被边缘裁切（必须留背景）、每页单独成图。
  - 链接：[Image Quality Assessment](https://docs.regulaforensics.com/develop/doc-reader-sdk/overview/image-quality/)、[Image Quality Requirements](https://docs.regulaforensics.com/develop/doc-reader-sdk/overview/image-quality-requirements/)
  - **这是「面向决策的输出」而非「面向分析的输出」的典型：用户要的是过/不过，不是 0.73。**
- **Veriff — Assisted Image Capture**：**在拍摄过程中实时给出反馈和指引**，检测出「模糊、眩光、人脸被遮挡、证件破损」等问题后**当场告诉用户怎么修**，避免提交后被拒再重来。链接：<https://www.veriff.com/blog/veriff-launches-assisted-image-capture-solution>
- **Jumio — Course Correction**：实时评估照片质量，用清晰的指引提示用户当场纠正（模糊、糊片、眩光过多）。链接：<https://www.jumio.com/course-correction/>
- **Inlite ClearImage / WaBR**：REST API，读条码之外提供**图像清理、旋转、纠偏**以便后续 OCR。链接：<https://www.inliteresearch.com/web-api-barcode-reader.php>
- **Kofax / Tungsten VRS Elite**：扫描端的「图像完美化」软件——自动纠偏、自动裁切、亮度/对比度调整、边缘清理、去噪点、Gamma 校正、自动方向、自动彩色检测、空白页检测。**逻辑是：先评估每页，再自动套用正确的扫描设置**，避免重扫、提升下游 OCR/ICR 准确率。链接：[产品说明](https://www.ecmconnection.com/doc/vrs-virtualrescan-0001)

### 7.4 小结：F 类为什么能做到解释和建议

三个前提缺一不可：
1. **有标准。** ISO 29794 系列把「什么算达标」写死了，解释才有依据。
2. **有下游任务做锚。** 质量分是按「会不会影响识别率」校准的，不是审美判断。
3. **用户还在现场。** 人还举着手机，建议立刻可执行，所以建议值钱。

---

## 8. G 类：电商 / UGC / 图库的质量把关

这一类不卖产品，**卖的是「准入规则」**。但它们的规则文档是**本次调研中最接近「属性级评价 + 自然语言解释 + 改进建议」的公开材料**。

### 8.1 Shutterstock

- 质量标准覆盖（照片与视频通用）：**Focus（主体必须合理清晰）、Exposure、Noise、Film Grain、Compression Artifacts、Color Banding / Posterization、Pixelation、Composition（地平线歪、干扰元素）、Post-Production（后期手法生硬）、Sensor Spots / Lens Dust、Reflections、Digital Conversion Quality、White Balance（非故意的偏色）、Styling**。
- 视频另加：**Aliasing / Moiré、Stabilization、Distortion、Video Editing、Frame Rate & Shutter Speed、Interlacing**。
- **审核方式：审稿人在 100% 原图比例下逐张检查**，并明确建议投稿人用同样方式自检。
- **反馈方式：拒稿时给出具体的质量问题类别**（如「Noise / Artifacts：图像含过多噪声、颗粒、伪影或栅格化不良」），并为每个类别配一篇知识库文章讲**为什么被拒、怎么避免**（例：用最低 ISO 拍摄）。
- 链接：[质量要求总表](https://submit.shutterstock.com/help/en/articles/10594682-photo-and-video-quality-requirements)、[Noise/Artifacts](https://support.submit.shutterstock.com/s/article/Why-was-my-content-rejected-for-Noise-Artifacts)、[Exposure](https://submit.shutterstock.com/help/en/articles/10615874-content-quality-standards-exposure)

### 8.2 Adobe Stock

- 审核维度：**锐度、曝光、分辨率、噪声、压缩、伪影、整体清晰度**；要求「主体锐利清晰、噪声与伪影最小化、光线/对比/色彩平衡」。
- 明确指出噪声与伪影的成因：高 ISO、过度后期、传感器灰尘、卷帘快门（skew / jello / flash banding）。
- 链接：[质量与技术标准](https://helpx.adobe.com/stock/contributor/content-moderation/quality-technical-standards-reasons-content-refusal.html)、[拒稿原因](https://helpx.adobe.com/stock/contributor/help/reasons-for-content-rejection.html)

### 8.3 电商平台（以 Amazon 为例）

- 规则更偏**形式合规**而非感知质量：主图最长边 ≥1000px（才能开放放大）、纯白底 RGB(255,255,255)、商品占画面 ≥85%、无文字/logo/水印、支持 JPEG/PNG/TIFF/GIF。
- 不合规会被拒绝、移除、修改，甚至导致 listing 被搜索抑制。
- 链接：[Seller Central 图片要求讨论](https://sellercentral.amazon.com/seller-forums/discussions/t/8bc27788-53c9-4cf8-ad43-c5eaa9400692)（**具体条款以 Seller Central 官方帮助页为准，部分细节待核实**）

**观察：** G 类证明了一件事——**当「质量」需要被仲裁时，人们最终依赖的是一份写清楚的、分维度的、带例子的自然语言规则**，而不是一个分数。

---

## 9. H 类：训练数据集质量治理

- **Voxel51 FiftyOne**
  - **Data Quality 面板**检测六类问题：**Brightness（过亮/过暗）、Blurriness（过糊/异常锐）、Aspect Ratio（极端长宽比）、Entropy（信息量异常）、Near Duplicates（基于 embedding）、Exact Duplicates（基于文件哈希）**。
  - 每类结果写入专门的样本字段（`brightness`、`blurriness`、`aspect_ratio`、`entropy`、`nearest_neighbor`、`filehash`）。
  - **呈现方式：交互式直方图 + 可拖动阈值滑块**，阈值可保存/重置；命中的样本自动在网格中呈现；用户可一键打标签、筛选、批量处理。
  - 链接：[Data Quality 文档](https://docs.voxel51.com/enterprise/data_quality.html)
  - **FiftyOne 有一套 agent「skills」生态**（<https://docs.voxel51.com/agents/skills_ecosystem/skill_cards.html>）：用户用自然语言提要求，agent 匹配意图并依次调用合适的 skill。skill 清单里包含 `fiftyone-data-quality`（找模糊图等）和 `fiftyone-dataset-curation`（检查 schema 与媒体质量：模糊、亮度、损坏、分辨率）。
  - **这是本次调研中，「属性级图像质量评估」被封装成 agent skill 的唯一明确商业案例。**
- **Cleanlab（Datalab / CleanVision）**
  - 检测的低质图类型：**dark、light、blurry、low-information、grayscale、odd-size、odd-aspect-ratio**。
  - 输出形态很值得注意：**每类问题同时给 `is_<issue>` 布尔判定和 `<issue>_score`（0–1，越接近 1 越严重）**——既给结论又给程度。
  - 链接：[Datalab 图像教程](https://docs.cleanlab.ai/master/tutorials/datalab/image.html)、[CleanVision](https://github.com/cleanlab/cleanvision)
- **Labelbox / Scale AI**：以标注与数据引擎为主，图像质量筛查多通过与 FiftyOne 等工具集成或自定义规则实现（**Scale AI 是否有独立的画质筛查功能，本次未找到官方说明，待核实**）。

**观察：** H 类是 B 类之外**唯一把曝光（brightness）单列为一个可调阈值维度的主流商业形态**，而且它的交互范式（直方图 + 阈值滑块 + 批量打标签）非常值得注意：**它不回答「这张图好不好」，它回答「这批图里哪些是异常值」。**

---

## 10. I 类：手机 / ISP 产业链

- **Qualcomm Chromatix / Chromatix Light**：官方的 IQ（Image Quality）调优工具。Chromatix 给专家开放 ISP 全部模块；**Chromatix Light 简化为一组可调项：roll-off、AWB、色调（tint）、色彩校正矩阵、饱和度、对比度、降噪、锐化、AEC**。还带 sensor 驱动生成工具。链接：[ISP tuning process](https://docs.qualcomm.com/bundle/publicresource/topics/80-88500-4/125_ISP_tuning_process.html)
- **MediaTek Imagiq**：18-bit HDR-native ISP + APU 上的 AI 后处理（AI-NR 降噪等），并向 OEM 开放 ISP raw 数据流控制。**未发现公开的画质评估工具**。链接：<https://www.mediatek.com/technology/imagiq-ai-camera>
- **ArcSoft 虹软**：算法与解决方案供应商（暗光、HDR、相机标定方案），按不同 sensor 特性和平台 ISP 特点调优降噪算法与参数。**未发现独立的画质评估/评测软件产品（待核实）**。链接：<https://www.arcsoft.com.cn/technology/index.html>
- **Morpho**：嵌入式影像算法 IP 供应商（Super-Resolution、Semantic Filtering、Deep Deblur——**按模糊成因选择四种不同的去模糊 AI 模型**）。历史上以「产品拿到 DxOMark 高分」作为营销背书。**无独立画质评估产品**。链接：<https://www.morphoinc.com/en/technology>
- **第三方 ISP 调优服务商**：eInfochips、Image Quality Labs、InnoWave、CK Vision 等提供 MTK/高通平台的 IQ Tuning 服务。

**观察：** 产业链上游的逻辑是——**调优工具（暴露旋钮）是芯片厂给的，评测工具（判断好坏）是第三方（DXOMARK/Imatest）给的，两者刻意分离。** 这个分离本身就是一种行业共识：**裁判不该由选手来当。**

---

## 11. J 类：AIGC 图像质量评估

- **Artificial Analysis Image Arena**
  - 方法：同一 prompt 生成两张图并排盲测，人工投票，更新 **ELO**。评判维度提示为**视觉连贯性、细节渲染、prompt 遵从度、艺术质量**。
  - 差异化：**同时公布推理成本（每 1000 张生成图的美元数）**，直接画出「质量–成本」帕累托前沿。
  - 链接：[排行榜](https://artificialanalysis.ai/image/leaderboard/text-to-image)、[方法论](https://artificialanalysis.ai/image/methodology)
- **LLM-as-a-judge（多模态）**：评测平台类厂商（如 Future AGI、Galileo 等）把多模态 LLM 当评审，**输入 rubric + 生成结果 + 原始媒体，输出分数 + 理由**。这在 2026 年已经是评测 SaaS 的标准形态之一。链接：<https://futureagi.com/blog/multimodal-llm-as-a-judge/>
- **观察**：AIGC 领域的质量评价**重心不在传统失真维度**（生成图一般没有噪声和压缩伪影问题），而在语义合理性、结构连贯性、prompt 对齐、审美。**同时，这个领域是「输出理由」被当成默认要求的领域**——因为单一分数无法区分一批都很好看的图。

---

## 12. K 类（额外发现）：消费级 AI 摄影点评

> **这是我在规定范围之外发现的、对我们最有参考价值的一类。** 它是本次调研中**唯一把「文字解释 + 改进建议」当作全部产品价值**的类别。

- **Adobe Project Indigo**（实验性 iOS 相机 App，2026-07 更新）
  - **AI Photo Critique**：拍完后分析照片，给出「专业意见」，覆盖**取景（framing）、光线（lighting）、色彩（colors）、情绪冲击（emotional impact）**。
  - **Capture & Edit Suggestions**：给出**重拍建议**——换取景、改曝光、改构图，并能**在取景器里直接指出具体的干扰物**。
  - 技术实现：初期使用 **Google 的 Nano Banana 云端模型**，结果约 2K 边长，需联网。
  - 链接：[Adobe Research AI Playground](https://research.adobe.com/articles/playground/playground.html)、[App Store](https://apps.apple.com/us/app/project-indigo/id6742591546)、[TechCrunch 报道](https://techcrunch.com/2026/07/20/adobe-camera-apps-new-feature-will-critique-your-photos-using-ai/)
  - **这是一家顶级影像厂商公开押注「大模型给图像质量做自然语言点评 + 可执行建议」的案例。**
- **第三方点评类 App/Web**（生态已经相当密集）：
  - **PhotoCritic**（Google Play）：构图与取景、光线与曝光、色彩和谐与平衡、技术质量、艺术表达、情绪冲击六个维度，**给出具体可执行的分步改进建议**。
  - **Photo Critique**（<https://photocritique.app/>）：宣称用一位职业摄影师的 **4000+ 条真实点评**训练，30 秒内给出构图、光线、曝光、色彩、氛围、叙事的反馈。
  - **Evalens**（<https://evalens.app/photo-critique>）：六维——构图规则、光线与曝光技法、色彩理论与影调、对焦与锐度、创意叙事、整体技术水准；带 **AI Coach 追踪长期进步**。
  - **PixMentor**（<https://pixmentor.com/>）：分项打分 + 改进提示 + **针对具体器材的建议**。
- **共同形态**：都是「**属性级分数 + 每项一段文字 + 具体改进动作**」，都免费试用 + 订阅变现，都没有任何权威背书或准确率依据。

---

## 13. 产品 × 功能矩阵

图例：● 有且是核心 / ◐ 有但较弱或部分 / ○ 无 / ? 待核实

| 产品 | 属性级分维度 | 单一总分 | 文字解释 | 改进建议 | 批量 | 视频 | 导出报告 | 跨次对比 | 本地部署 | 接入自定义模型 | LLM / Agent | 权威背书 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| DXOMARK Analyzer | ● | ◐ | ○ | ○（走服务） | ● | ● | ● | ● | ● | ◐(Python API) | ○ | ● |
| DXOMARK 公开榜单 | ● | ● | ● | ○ | — | ● | ● | ● | — | ○ | ○ | ● |
| DXOMARK GEA | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ?（官方称 AI-powered） | ● |
| Imatest | ● | ○ | ○ | ○ | ● | ● | ● | ● | ● | ●(IT API 四种接口) | ○ | ●(IEEE 2020) |
| iQ-Analyzer-X | ● | ○ | ○ | ○ | ● | ● | ● | ●(数据库) | ● | ◐ | ◐(AI 图卡检测) | ● |
| 阿里云 EvaluateVideoQuality | ● | ● | ○ | ○ | ● | ● | ○ | ○ | ○ | ○ | ○ | ○ |
| 阿里云 IMM 图片质量评分 | ○ | ● | ○ | ○ | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| 腾讯云 AssessQuality | ◐(清晰+美观+5 布尔) | ○ | ○ | ○ | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| 火山 veImageX 画质评估 | ●(4 项) | ○ | ○ | ○ | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| AWS Rekognition | ◐(亮度/锐度/对比度) | ○ | ○ | ○ | ● | ● | ○ | ○ | ○ | ○ | ○ | ○ |
| Google Vision / Azure Vision | ○ | ○ | ○ | ○ | — | — | — | — | — | ●(Custom Vision) | ○ | ○ |
| Sightengine | ◐(properties 分三项) | ● | ◐(分段释义表) | ○ | ● | ● | ○ | ○ | ○ | ◐(Enterprise) | ○ | ○ |
| Everypixel | ○ | ●+标签 | ◐(等级标签) | ○ | ● | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| Clarifai | — | — | — | — | ● | ● | ○ | ○ | ◐ | **●（核心卖点）** | ●(agent operators) | ○ |
| Cloudinary | ○（内部用） | ○ | ○ | ●(自动优化) | ● | ● | ○ | ○ | ○ | ○ | **●(5 个 MCP + Agents)** | ◐(SSIMULACRA 开源) |
| Aftershoot | ◐(分桶) | ●(1–100) | ◐(桶名) | ○ | ● | ○ | ◐ | ○ | ●(桌面) | ○ | ○ | ○ |
| Narrative Select | ●(对焦 0–100 + 眼部) | ○ | ◐(标签) | ○ | ● | ○ | ◐ | ●(场景内排序) | ● | ○ | ○ | ○ |
| FilterPixel | ●(8 项信号) | ● | ◐ | ○ | ● | ○ | ◐ | ○ | ◐ | ○ | ○ | ◐(厂商自测 94%) |
| Lightroom Assisted Culling | ●(主体/眼部锐度、睁眼) | ● | ◐(可展开评分明细) | ○ | ● | ○ | ○ | ○ | ● | ○ | ○ | ○ |
| Imagen AI | ◐ | ○ | ○ | ○ | ● | ○ | ○ | ○ | ◐ | **●(Personal AI Profile)** | ○ | ○ |
| IMAX SSIMPLUS | ◐ | ●(0–100，设备感知) | ◐(报告) | ●(档位/码率) | ● | ● | ● | ● | ● | ○ | ○ | ◐(厂商声称) |
| Interra BATON | ● | ○ | ●(QC 报告) | ◐ | ● | ● | ● | ● | ● | ◐ | ◐(ML 检查) | ● |
| Mux Data | ●(4 个子分) | ●(0–100) | ◐ | ●(Negative Impact Score) | ● | ● | ● | ● | ○ | ○ | ○ | ○ |
| **OFIQ / ISO 29794-5** | **●(25+ 分量，分采集/主体)** | ●(0–100) | ◐(结构化) | ●(可推导) | ● | ○ | ◐ | ● | ●(开源) | ◐ | ○ | **●(ISO+BSI+NIST)** |
| **NFIQ 2** | ● | ●(0–100) | ○ | ○ | ● | ○ | ○ | ● | ●(开源) | ○ | ○ | **●(ISO+NIST)** |
| Regula Doc Reader | ●(3 项枚举) | ○ | ●(枚举即结论) | ●(隐含) | ● | ○ | ◐ | ○ | ●(SDK) | ○ | ○ | ● |
| Veriff / Jumio | ●(失败原因分类) | ○ | ● | **●(实时指引)** | ● | ● | ◐ | ○ | ○ | ○ | ○ | ◐ |
| Shutterstock / Adobe Stock | **●(14+ 维规则)** | ○ | **●(拒稿理由+知识库)** | **●(怎么避免)** | ● | ● | ○ | ○ | — | — | ◐ | ● |
| FiftyOne Data Quality | ●(6 类) | ○ | ○ | ◐(给操作) | ● | ● | ◐ | ● | ●(开源+企业) | ●(插件/自定义模型) | **●(agent skills)** | ○ |
| Cleanlab CleanVision | ●(7 类，判定+严重度) | ○ | ○ | ◐ | ● | ○ | ◐ | ○ | ●(开源) | ◐ | ○ | ○ |
| **Adobe Project Indigo** | ●(4 维) | ○ | **●(核心)** | **●(核心，含重拍指引)** | ○ | ○ | ○ | ○ | ○(云端模型) | ○ | **●(Nano Banana)** | ◐(Adobe 品牌) |
| PhotoCritic / Evalens / PixMentor | ●(6 维) | ● | **●(核心)** | **●(核心)** | ◐ | ○ | ◐ | ◐ | ○ | ○ | ● | ○ |

---

## 14. 需求洞察：正面回答八个问题

### Q1. 商业产品普遍都提供哪些功能？哪些是「人人都有」的基础功能？

**几乎人人都有的五项（基础盘）：**

1. **一个可排序的总分。** 无论叫 quality score、ClarityScore、Viewer Score 还是 1–100 分，**所有产品都提供一个可以用来排序和卡阈值的单一数字**。这是最低共同分母。
2. **批量处理。** 没有一个产品只支持单图。所有 B/D/H 类产品的存在理由就是规模化。
3. **一个可调的阈值或档位。** Sightengine 让你设 0.5，Aftershoot/FilterPixel 给敏感度滑块，FiftyOne 给直方图滑块，Cloudinary 给 best/good/eco/low。**质量标准可调是共识，不是特色。**
4. **结果可导出/可编程接入。** JSON、CSV、Excel、PDF、SDK、REST——至少有一种。
5. **清晰度/锐度维度。** 这是唯一一个**所有类别、所有产品都覆盖**的属性维度。（顺带说明：**模糊是商业界的「第一属性」**，而这恰好是实验室目前的空白。）

**很常见但不算人人都有的：**
- 重复/近似图检测（D、H 类标配，B 类基本没有）
- 主体/人脸感知（D、F 类标配）
- 美学分（中国云厂商标配，欧美厂商少见）

### Q2. 哪些功能只有少数产品才有，构成差异点？

| 差异点 | 谁有 | 为什么稀缺 |
|---|---|---|
| **自然语言解释「为什么」** | Adobe Project Indigo、消费点评 App、图库审核规则、DXOMARK 评测文章 | 需要多模态模型或人工，成本高；且大多数 B2B 场景不需要 |
| **具体的改进建议 / 重拍指引** | Veriff、Jumio、Project Indigo、点评 App、Shutterstock 知识库 | **只在「用户还能重拍」时才有价值** |
| **设备/场景感知的分数** | IMAX SSIMPLUS（手机 vs 电视不同分）、Everypixel（UGC vs Stock 两个模型） | 需要针对场景分别标注和校准 |
| **分数与下游任务性能校准** | NFIQ 2、OFIQ | 需要大规模识别性能数据；这是「质量分可信」的真正门槛 |
| **可扩展、接入自定义模型** | Imatest IT API、Clarifai、FiftyOne 插件、Sightengine Enterprise | 见 Q6 |
| **跨时间/跨实验室的结果数据库与对比** | iQ-Analyzer-X、DXOMARK、Interra BATON | 是 A/E 类的刚需，其他类几乎没有 |
| **影响归因（哪个维度拖累了总分）** | **Mux 的 Negative Impact Score** 是最清晰的一个 | 需要有权重模型，且要能反推 |
| **属性来源归类（采集相关 vs 主体相关）** | **只有 OFIQ / ISO 29794-5** | **本次调研中最被低估的一个设计** |
| **agent / MCP 接入** | Cloudinary（5 个 MCP + Agents）、FiftyOne（agent skills）、Clarifai（agent operators） | 2025–2026 才出现，仍是早期 |

### Q3. 客户真正付费买的是什么？（说需求，不是功能）

把八类产品的付费动机抽出来，只有**五种真实需求**：

1. **「省掉人工逐张看的时间。」**（D 类、H 类、G 类机审）
   - 婚礼摄影师 3000 张选片 6–10 小时 → Aftershoot 几十分钟。Imagen 官方宣称后期总时长最多减少 90%。
   - **这是整个市场里最大、最刚性、变现最顺畅的需求。** 客户买的是**时间**，不是准确度——所以敏感度滑块比高精度更重要，因为返工成本要低。
2. **「给我一个所有人都认的数，好让我们别吵了。」**（A 类、F 类、E 类的度量）
   - 手机厂和模组供应商要有共同的验收语言；KYC 系统要有可仲裁的达标线；流媒体采购要有可比的画质基线。
   - 客户买的是**仲裁权与可比性**。这也是为什么 ISO/NIST 背书在 F 类那么值钱，为什么 DXOMARK 的公开榜单能反过来支撑它的软件生意。
3. **「别让不合格的东西流到下游。」**（F 类、G 类、H 类、E 类 QC）
   - 模糊证件导致 OCR 失败和人工复核；低质训练数据毁掉模型；不合规素材导致下架。
   - 客户买的是**避免下游损失**。所以这类客户对「误杀」的容忍度低于「漏放」——他们要的是可解释的拒绝理由。
4. **「在不让用户察觉的前提下少花钱。」**（C 类、E 类编码）
   - Cloudinary q_auto、Beamr CABR、阿里窄带高清。客户买的是**带宽/存储成本**，质量评价只是约束条件。
   - **这类客户永远不会为「质量报告」付费，他们只为省下的账单付费。**
5. **「让我变得更好。」**（K 类）
   - 摄影爱好者想进步。客户买的是**教学与反馈**。这是唯一一个「解释和建议」本身就是全部商品的需求。

**一个重要推论：需求 1–4 都不需要自然语言解释，只有需求 5 需要。** 商业界解释做得少，不是做不到，是**大部分客户不为解释付钱**。

### Q4. 结果怎么呈现？「解释」和「建议」商业上到底做不做、怎么做、什么颗粒度？

**呈现形态从轻到重有六档，商业产品各据一档：**

| 档位 | 形态 | 代表 | 颗粒度 |
|---|---|---|---|
| 1 | 一个数 | Sightengine、阿里云 IMM、APILayer | 无解释 |
| 2 | 一个数 + 分段释义表 | Sightengine 文档、IMAX SSIMPLUS 的五档 | **解释「这个数怎么读」，不解释「这张图为什么这样」** |
| 3 | 一个数 + 等级标签 | Everypixel（Poor/Good/Excellent）、Cloudinary（best/good/eco/low） | 标签即解释 |
| 4 | 分维度的数 / 分桶 | DXOMARK 属性树、Mux 四个子分、Aftershoot 分桶、Lightroom 可展开评分 | **靠「哪一项低」隐式解释，不写句子** |
| 5 | 结构化枚举结论 | Regula（enough / not enough / not performed）、OFIQ 质量分量向量、Shutterstock 拒稿类别 | 面向决策，可直接驱动动作 |
| 6 | 自然语言点评 + 具体动作 | Project Indigo、点评 App、Shutterstock 知识库文章、DXOMARK 评测文章 | **完整解释 + 可执行建议** |

**关于「解释」的结论：**
- **B2B 工具类（A/B/C/E/H）基本停在第 1–4 档。** 解释以文档、图表、样张对比的形式存在于**软件之外**。
- **第 4 档「分维度」是商业界解释的主流做法**：不写句子，但把总分拆开，让用户自己看出问题在哪。Lightroom「可展开看详细评分」、Mux 四子分 + Negative Impact Score 都是这个思路。**这是性价比最高的一档。**
- **第 6 档只在两种场景出现**：(a) 用户还能重拍（F 类、Project Indigo）；(b) 产品本身卖的就是教学（K 类）。

**关于「建议」的结论：**
- **建议的价值完全取决于「还来不来得及」。**
  - **来得及** → 建议是核心卖点：Veriff 实时指引、Jumio course correction、Project Indigo 重拍建议、Shutterstock「下次用低 ISO」。
  - **来不及**（照片已拍完、视频已编码完） → **商业产品一律不给建议，直接替你做决定或直接替你修好**（Aftershoot 直接分桶、Cloudinary 直接优化、腾讯画质重生直接修复）。
- **颗粒度上最成熟的是 F 类**：不但指出问题，还**归类了问题的来源**（OFIQ 的 capture-related vs subject-related），因为这直接决定了「你该重拍，还是该让对方摘下眼镜」。

### Q5. 有没有产品已经在用大模型或 agent？形态是什么样的？

**有，但比想象的少，且集中在四种形态：**

1. **用大模型做「点评 + 建议」的生成**（形态最成熟）
   - **Adobe Project Indigo**：调用 Google Nano Banana 云端模型，对取景/光线/色彩/情绪出具「专业意见」，并给重拍与编辑建议、在取景器中指认干扰物。
   - 一批消费级 App（PhotoCritic、Evalens、PixMentor、photocritique.app）：六维打分 + 每维一段文字 + 具体改进动作。
2. **用 AI 加速专业主观评价**
   - **DXOMARK GEA（Golden Eye Assistant）**：官方定义为「AI-powered image quality assessment tool that enhances perceptual evaluation」，目标是让感知评价更快、更一致、可规模化。**是否用大模型、是否出文字，官方未公开，待核实。**
   - **Image Engineering iQ-Analyzer-X**：用 AI 做图卡检测与 ROI 定位——**AI 用来提速测量流程，不用来做质量判断**。
3. **把图像能力封装成 agent 可调用的工具**
   - **Cloudinary**：5 个 MCP Server（含 Analysis）+ Cloudinary Agents（2026-05），支持 Cursor / Claude Desktop / Claude Code / VS Code 自然语言操作媒体资产。
   - **Voxel51 FiftyOne**：agent skills 生态，`fiftyone-data-quality`、`fiftyone-dataset-curation` 等 skill 可被 agent 按意图调用；**这是「图像质量评估作为 agent skill」最直接的商业先例。**
   - **Clarifai**：自定义模型可与 agent operators 组成 workflow。
4. **多模态 LLM-as-a-judge**
   - 评测 SaaS（Future AGI、Galileo 等）把多模态模型当评审：输入 rubric + 输出 + 原始媒体，返回分数**和理由**。这是 AIGC 评测领域的默认形态。

**没有发现的形态：** 目前**没有找到任何一个成熟商业产品，是「对话式的、多轮的、能接入多个专业 IQA 模型并整合成报告」的 agent**。Cloudinary 和 FiftyOne 最接近，但它们的 agent 调的是资产管理和数据集操作，不是一组属性级画质模型。**这是一个真实的空白。**

### Q6. 有没有产品把「可扩展、能接入自己的模型」当作卖点？

**有，而且分成四种截然不同的「可扩展」：**

| 类型 | 含义 | 代表 |
|---|---|---|
| **接口可扩展** | 给 API，让你把它嵌进你的系统 | **Imatest IT**（C/C++ DLL、CLI、Python、.NET 四种接口，专为无人值守产线设计）、DXOMARK AZ API（Python）、iQ-Analyzer-X（hot folder + CLI） |
| **模型可托管** | 你把自己的模型传上来，平台负责部署和编排 | **Clarifai**（核心卖点：HuggingFace 或本地模型上传 → 自动部署 → 可与其他模型和 agent operators 组 workflow；本地上传限企业版）、Sightengine Enterprise（custom models） |
| **测量可定制** | 你能定制图卡和指标本身 | **DXOMARK Custom Development of Charts and Metrics**（<https://corp.dxomark.com/catalog/testing-tools/custom-camera-charts/>）、iQ-Analyzer-X 自定义测试模板与参考数据 |
| **标准可个性化** | 不是接入你的模型，是**用你的数据训一个你的模型** | **Imagen AI Personal AI Profile**（3000+ 张历史修片训练出复刻你风格的模型）——这是 D 类最强的护城河 |

**观察：**
- **「可扩展」在 B2B 工具类是硬性要求**（Imatest 把 API 做成独立产品线 IT 并单独定价），在 B 类云 API 里几乎不存在（你只能用它给的那个分）。
- **Clarifai 是唯一一家把「接入你自己的模型」当成公司定位的**，但它不是 IQA 厂商——它是通用 AI 平台。**「专门承载多个属性级 IQA 模型的平台」这个位置，商业上是空的。**

### Q7. 对属性级（分维度）评价的支持程度如何？谁做得最细？细到什么程度？

**排名（从最细到最粗）：**

1. **OFIQ / ISO 29794-5（人脸）—— 最细。**
   - **统一质量分（0–100）+ 25~27 个独立质量分量**，且**分量被分成「采集相关」和「主体相关」两组**。
   - 每个分量有国际标准定义，有 NIST FATE SIDD 按分量分别做第三方评测。
   - **这是全世界唯一一个「属性级图像质量评价」被写成 ISO 标准、有开源参考实现、有政府机构背书、有独立第三方分维度评测的领域。**
2. **DXOMARK —— 商业产品里最细。**
   - 测试软件侧：12 个测量类目（含 3A 分别拆成 AE/AWB/AF）。
   - 公开榜单侧：**带权重的属性树**——Photo 六属性（Exposure/Color/Autofocus/Texture/Noise/Artifacts）+ Video 七属性（多 Stabilization），上面再套镜头维度（主摄/长焦/超广/Bokeh）的加权。
   - **AIOP 进一步把「人像」这个场景拆成肤色/细节/虚化/曝光/色彩五维**——这是「按场景重新组织属性」的范例。
3. **Imatest / iQ-Analyzer-X —— 维度全但不聚合。**
   - Imatest 13+ 个质量因子，iQ-Analyzer-X 9 个核心维度。它们**刻意不给总分**——工程测量不做主观加权。
4. **NFIQ 2、IEEE 2020-2024** —— 标准级的维度定义（IEEE 2020 覆盖 flare、dynamic range、contrast、noise、flicker、geometry、resolution、color）。
5. **Mux Data** —— 四个子分 + 影响归因，是 QoE 领域最清晰的属性级设计。
6. **火山 veImageX（4 项）、阿里云 EvaluateVideoQuality（客观 3 项 + 颜色 4 项 + 瑕疵 5 项）** —— 国产云里最细的两个。
7. **FiftyOne（6 类）、Cleanlab（7 类）** —— 属性级但定位是「异常检测」而非「质量评分」。
8. **AWS Rekognition（亮度/锐度/对比度 3 项）** —— 国际大厂唯一分维度的。
9. **腾讯/百度（清晰度 + 美观度 2 项）、Sightengine、Everypixel、阿里 IMM（1 项）** —— 最粗。

**对实验室维度的正面对照：**

| 实验室维度 | 商业界覆盖情况 |
|---|---|
| **整体质量分** | **人人都有**，是基础功能，不是差异点 |
| **曝光** | A 类（DXOMARK Exposure、Imatest ISO Sensitivity/Dynamic Range）、F 类（OFIQ 光照）、D 类（FilterPixel 曝光质量、Lightroom 曝光剔除）、H 类（FiftyOne/Cleanlab brightness）、G 类（Shutterstock Exposure）、AWS brightness。**覆盖广，但在云 API 里基本不单独暴露** |
| **对比度** | A 类（DXOMARK Dynamic Range & Contrast）、IEEE 2020、AWS contrast、阿里 EvaluateVideoQuality。**是最少被单列的一项** |
| **白平衡** | **A 类（DXOMARK AWB、Qualcomm Chromatix AWB 调优）和 G 类（Shutterstock White Balance 是独立拒稿理由）覆盖，云 API 全线缺失。** 这是一个**有真实行业需求、但公开 API 几乎没人提供**的维度 |
| **色彩** | A 类（Color Accuracy/Fidelity）、G 类、阿里颜色质量四项、火山色调均衡。覆盖较好 |
| 美学/构图（实验室不做） | **中国云厂商全线提供（腾讯 AestheticScore、百度 aesthetic、火山大众美学）**，D 类和 K 类也都做。**不做这块，等于主动放弃了与国产云 API 的可比性** |
| 失真类：模糊 | **全类别覆盖率最高的维度，实验室的空白正好落在商业界的第一属性上** |
| 失真类：噪声 | A 类、G 类（Shutterstock/Adobe Stock 的头号拒稿理由）、阿里、火山、腾讯 ClarityScore 内部 |
| 失真类：压缩伪影 | A 类、C 类（Cloudinary SSIMULACRA 专门为此调过）、E 类、阿里 EvaluateVideoQuality |

### Q8. 商业模式有哪些形态？

本次调研见到的全部六种，以及各自的适配条件：

| 模式 | 代表 | 适配什么 |
|---|---|---|
| **按调用量** | Sightengine（$0.002/次，$29/$99 月档）、Everypixel（$0.6/1000）、阿里云/腾讯云按量付费 + 预付费资源包 + 预付费 QPS | 云 API；**客户量大、单价极低，必须走量** |
| **按处理张数/credit** | Imagen（$0.05/张）、Evoto（4.6–6.9 美分/张，导出才扣） | 结果有明确产出物；**「导出才扣费」是降低试用门槛的关键设计** |
| **固定订阅（不限量）** | Aftershoot（$10/月不限张数）、Narrative（$10–$60 分层）、FilterPixel（$19/$35） | 摄影师无法预测月产量；**「不限量」本身就是卖点，直接对打按张计费** |
| **永久买断 + 年度维护** | Imatest（永久含 1 年支持）、Photo Mechanic（$299/$399） | 工程工具、产线工具；**客户要求版本可冻结、结果可复现** |
| **卖设备/实验室** | DXOMARK（图卡、灯箱、六足机器人、整套实验室包）、Image Engineering | 测「设备」必须控制输入；**硬件是这门生意真正的护城河** |
| **卖服务/咨询** | DXOMARK ISP Tuning（四阶段 KPI 驱动）、eInfochips/Image Quality Labs 的 IQ Tuning | 「解释和建议」的商业化出口 |
| **（第七种）开源 + 标准背书** | OFIQ、NFIQ 2、SSIMULACRA 2 | 不直接变现，**通过成为事实标准来获取话语权**；OFIQ/NFIQ2 靠政府预算，SSIMULACRA 靠带动 Cloudinary 主业 |

**一个跨模式的规律：** **免费档在这个市场几乎是标配**——Sightengine 2000 次/月、Everypixel 500 次、Imatest/iQ-Analyzer-X 的 Free 版、Aftershoot 30 天全功能试用、Narrative 全功能 Ultra 试用。原因是**质量判断的标准高度主观，客户必须先用自己的图试过才敢买**。

---

## 15. 市场空白与机会（基于调研事实的观察，不是产品方案）

1. **「属性级 + 自然语言解释」之间存在一道清晰的裂缝。**
   属性拆得最细的（DXOMARK Analyzer、Imatest、OFIQ）**不说人话**；说人话的（Project Indigo、点评 App）**属性很粗且没有权威依据**。两者之间是空的。
2. **白平衡和对比度是「有行业需求、无公开 API」的维度。**
   Shutterstock 把 White Balance 列为独立拒稿理由，Qualcomm 把 AWB 列为核心调优项，但**没有任何一个公开的云 API 单独返回白平衡评价**。
3. **没有「多个专业 IQA 模型的统一承载平台」。**
   Clarifai 做通用模型托管但不懂画质；DXOMARK/Imatest 懂画质但只跑自家算法；FiftyOne 懂数据集但质量维度只有 6 个。**「把一批属性级 IQA 模型挂上来、用自然语言调度、整合成一份报告」这件事，商业上没有成品。**
4. **对话式 IQA agent 是空白。** 见 Q5。
5. **「质量分与下游任务性能校准」这个方法论，尚未扩散出生物识别领域。**
   NFIQ 2 / OFIQ 把「质量分必须能预测识别性能」写进标准，是这个领域公信力的根本来源。通用 IQA 的分数至今没有等价的锚。
6. **「属性来源归类」几乎无人借鉴。** OFIQ 的 capture-related / subject-related 二分法，是把「诊断」转化为「可执行建议」的关键一步，但只存在于人脸领域。
7. **AIGC 图像的传统失真维度评价需求正在消失，但语义/结构维度的需求在上升。** 这会长期改变「图像质量」这个词的所指。

---

## 16. 相邻但不同的领域：工业视觉缺陷检测（简要说明）

Landing AI、Instrumental、Cognex 等做的是**产品外观缺陷检测**（划痕、错料、装配偏差）。与 IQA 的根本区别：

| | IQA | 工业缺陷检测 |
|---|---|---|
| 判断对象 | **图像本身**的成像质量 | 图像中**被拍物体**的物理质量 |
| 成像条件 | 通常不可控（UGC、手机） | 严格受控（固定光源、固定工位、固定视角） |
| 「差」的定义 | 感知质量下降 | 与规格不符 |
| 图像模糊时 | **是待评价的目标** | **是要排除的干扰，需要重拍** |

对 IQA 而言，工业视觉只在一个点上相关：**它们也需要「这张图能不能用」的前置判断**，但那通常是用固定阈值的简单检查解决的，不构成独立产品。**不建议展开。**

---

## 17. 待核实信息清单

| # | 待核实内容 | 原因 | 建议核实途径 |
|---|---|---|---|
| 1 | **DXOMARK GEA 的完整规格**：覆盖属性、是否基于多模态大模型、是否输出文字、部署形态、定价 | 官方产品目录页抓取返回 502，仅有新闻稿一段描述 | 直接联系 DXOMARK 销售；或核实 corp.dxomark.com/catalog 恢复后的页面 |
| 2 | DXOMARK AIOP 的完整指标定义与商业形态 | 仅有 MWC 新闻稿一句话 | 同上 |
| 3 | Imatest IT 的官方价格（$5,300 / $8,480 来自第三方经销商页面） | 官网为询价制 | Imatest 官方报价 |
| 4 | iQ-Analyzer-X 各版本价格与模块清单 | 官方 web shop 下线，指引联系销售 | Image Engineering 销售 |
| 5 | 百度智能云 BOS `aesthetic` / `clarity` 的取值范围 | 文档未明示，示例值为 0.26 / 0.28 | 百度云官方文档或工单 |
| 6 | 火山引擎 veImageX 画质评估四项指标的定义与取值范围 | 文档页抓取为空 | docs.volcengine.com/docs/508/111896 重试或工单 |
| 7 | 火山引擎 VQScore 是否提供分维度输出、计费单价 | 文档页抓取为空 | 同上 |
| 8 | 腾讯云数据万象「图片质量评估」的具体单价（元/千次） | 文档只确认它是内容识别下的一个计费项 | 腾讯云价格计算器 |
| 9 | 华为云是否有构图质量相关的独立接口 | 仅在营销页看到提法 | support.huaweicloud.com API 参考 |
| 10 | Imagga 是否存在未公开的画质评分端点 | 官方文档未见 | docs.imagga.com 全量检索或联系销售 |
| 11 | 七牛云、又拍云是否有未公开的画质评分 API | 官方文档未见 | 各自开发者中心全量检索 |
| 12 | OFIQ 质量分量的**确切数量与完整名称列表**（资料中 25+ / 27 说法不一） | 需查 OFIQ Public Report 或 ISO/IEC 29794-5 正文 | [OFIQ GitHub](https://github.com/BSI-OFIQ/OFIQ-Project) 的 Public Report PDF |
| 13 | Telestream Qualify 的具体质量指标清单 | 搜索未命中产品页 | telestream.com 产品页 |
| 14 | Bitmovin Analytics、NPAW 的质量指标设计 | 本次未逐项核实 | 各自官方文档 |
| 15 | Scale AI 是否有独立的图像质量筛查功能 | 未找到官方说明 | scale.com 文档 |
| 16 | FADGI / Metamorfoze / ISO 19264 的具体质量指标 | 本次未核实，仅由 iQ-Analyzer-X CH 推断该市场存在 | 各标准官方文档 |
| 17 | FilterPixel「94% 一致率」「92–97% 一致率」的测试方法 | 厂商自述，无第三方验证 | 厂商白皮书（若有） |
| 18 | IMAX SSIMPLUS「>90% 相关性」「比 VMAF 快 10 倍」的验证条件 | 厂商自述 | SSIMWAVE 技术白皮书 |
| 19 | Amazon 图片要求的官方条款原文（本文多处依赖第三方整理） | 部分细节来自第三方博客 | Seller Central 官方帮助页 |
| 20 | Onfido 的图像质量检查项（本次未单独核实） | 时间所限 | onfido.com 文档 |

---

## 18. 对需求整理的启发（仅此一节，简短）

> 按要求，这里**不给架构、不给方案**，只列出调研中对「需求怎么写」有直接参考价值的几点观察。

1. **「解释」和「建议」要分开当两件事讨论。** 商业界的事实是：解释的主流做法是**分维度摊开分数**（第 4 档），只有在「用户还能重拍」或「产品卖教学」时才升级到自然语言（第 6 档）。我们要先确认演示场景属于哪一种。
2. **「分维度 + 影响归因」可能比「每维一段话」性价比更高。** Mux 的 Negative Impact Score 和 Lightroom 的「可展开看评分明细」都说明：告诉用户「是哪一项拖累了总分」，往往比写一段文字更有效。
3. **阈值/敏感度可调是商业共识，不是高级功能。** 几乎所有 D 类和 H 类产品都给滑块。这暗示需求文档里可能需要讨论「谁来定好坏的线」。
4. **OFIQ 的「采集相关 vs 主体相关」二分法值得作为属性组织方式的参考素材。** 它是把诊断转成建议的关键一步。
5. **模糊/噪声/压缩伪影是商业界覆盖率最高的一组维度，而这正是实验室的空白。** 需求讨论时这个落差需要被明确记录下来（是暂不覆盖，还是通过 skill 补）。
6. **白平衡是一个「有行业需求、公开 API 无人提供」的维度**，实验室已经有这个能力，这在对外展示时是可讲的差异点。
7. **「可扩展、能接入自己的模型」在 B2B 工具类是硬需求（Imatest 单独做成 IT 产品线并单独定价）**，可作为「模型作者接入」这条需求线的外部佐证。
8. **对话式、多轮、调度多个属性级 IQA 模型并整合成报告的 agent，商业上目前是空白。** 这既是机会，也意味着没有可直接对标的参照物。

---

*本文档所有事实均标注来源链接。厂商自述的性能数据已标注为「厂商声称」。无法从官方渠道确认的均已列入第 17 节待核实清单。*
