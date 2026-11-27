---
title: "GEN3C: 3D-Informed World-Consistent Video Generation with Precise Camera Control（完整精读与研究笔记）"
date: 2026-11-27 04:52:07
index_img: "/GEN3C/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/GEN3C/)

# 第一部分：论文精读报告

# GEN3C 精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2503.03751v1（2025-03-05）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2503.03751) | [PDF](https://arxiv.org/pdf/2503.03751v1) | [Project Page](https://research.nvidia.com/labs/toronto-ai/GEN3C/) | [Code](https://github.com/nv-tlabs/GEN3C) | [Model](https://huggingface.co/nvidia/GEN3C-Cosmos-7B) | [Dataset](https://huggingface.co/datasets/nvidia/GEN3C-Testing-Example)

<!-- PAPER-METADATA END -->


**权威依据：** 用户提供的同版本 PDF；页码均指 PDF 物理页。

<!-- PTH:BEGIN INIT -->
## 0. 自动初始化：论文与材料

**论文结构**

【原文事实】Xuanchi Ren、Tianchang Shen（共同第一作者）、Jiahui Huang、Huan Ling、Yifan Lu、Merlin Nimier-David、Thomas Müller、Alexander Keller、Sanja Fidler、Jun Gao，机构 NVIDIA、University of Toronto、Vector Institute（PDF p.1）。arXiv v1 日期 2025-03-05。本报告绑定该版本，未将后续仓库或会议版本的内容混入。

| 部分 | PDF 页码 | 内容 |
|---|---|---|
| Abstract、§1 Introduction | 1–2 | 问题、显式几何条件及应用 |
| §2 Related Work | 2–3 | NVS、可控生成、视频一致性 |
| §3 Background | 3 | 视频扩散、VAE、训练目标 Eq.1 |
| §4 Method | 3–6 | 时空缓存、渲染、融合、训练、推理 |
| §5 Experiments and Applications | 6–9 | 训练、单/双视角、驾驶、动态、消融、Cosmos |
| §6 Conclusion | 9 | 总结与动态内容限制 |
| References | 9–13 | [1]–[76] |
| Appendix A | 14 | 自回归更新，Eq.4–5 |
| Appendix B | 14–15 | 训练、推理及基线设置 |
| Appendix C | 15–16 | 掩码比较及补充定性结果 |

**资源与证据**

PDF 16 页，全文 Figure 1–16（正文 12，附录 4），Table 1–6（全部正文），无独立编号 Algorithm。B1 目录 `figure-catalog.json` 保留原文题注；B2 `figure-map.json` 保存 PDF 物理页、区域和展示副本的映射。16 幅图均有 PDF 保真区域和 JPEG 展示副本，11 PASS_EXACT、5 PASS_WITH_NOTES；展示副本不声称无损，客户端渲染未验证。源 PDF SHA-256 为 `5f2462a60f2bed4f097ea79680de022eec4e5ea2f2a30660af9c2a2e80a067d7`。Source 为辅助载体，内部无独立版本印记；HTML 有外部装饰资源缺失，不影响完整 PDF 阅读。原始论文资源保持不变。

后文以【原文事实】【背景知识】【合理推断】【独立评价】【尚未证实】标识来源。原文事实包括作者明确声称但未独立证明的观点；定性截图只能证明相应可见现象，不能替代指标。所有页码指上传 PDF 物理页。

本单元只做资源和目录登记，不涉及需要解释的技术机制或数学推导。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1.1 标题：从相机参数条件到几何条件

**标题译文**

GEN3C: 3D-Informed World-Consistent Video Generation with Precise Camera Control

GEN3C：由三维信息引导、具有精确相机控制的世界一致性视频生成。

**术语辨析**

- **GEN3C**：作者在 Abstract 和 Introduction 将名称与 Camera Control 和 3D Consistency 相联系。它首先是生成式视频模型，而非旨在直接输出高精度三维网格的重建网络。
- **3D-Informed**：三维信息提供显式引导。§4.1–4.3 通过深度反投影建立点云，在目标相机下渲染为二维条件视频，再以视频扩散生成输出。三维缓存是条件，而非要求最终图像完全等于点云渲染；因此允许修正噪声、补全遮挡，也存在偏离真实几何的可能。
- **World-Consistent**：关注相机运动和时间变化中同一世界的内容保持一致。Figure 2 的往返运动示例将“相机转回来后是否仍看见同一个场景”作为直观动机。世界一致性不等于静止：动态内容中同一物体可以随时间移动。它也不是仅相邻帧平滑，短时平滑无法保证离开视野的区域再次出现时仍正确。
- **Precise Camera Control**：用户指定相机轨迹；相机位姿和内参用于几何投影，而不仅是送给网络的数值条件。位姿决定观察中心与方向，内参（如焦距）决定成像关系；Figure 1 的 Dolly Zoom 同时改变两者。投影给出像素级空间锚点，但网络输出并未被硬性要求逐像素遵循几何，因此“precise”是设计目标和实验主张，不是严格误差为零的数学承诺。

【合理推断】标题指向一个条件生成问题：在稀疏或动态观测不完备时，以部分三维信息和目标相机共同约束视频生成，兼顾观测一致性与不可见区域的合理补全。仅凭标题不能确定点云是否是最终重建产物、是否支持任意相机轨迹、是否需要输入位姿和精确深度、是否有长程一致性保证。这些须由 §4 和 §5 验证。

本单元是标题解释，不涉及独立的公式推导。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 摘要：为什么要把三维几何放在生成模型之外？

**摘要全文翻译**（PDF p.1）

我们提出 GEN3C，一种具有精确相机控制和时间三维一致性的生成式视频模型。先前的视频模型已经能够生成真实感视频，但它们往往很少利用三维信息，导致诸如物体突然出现或消失之类的不一致现象。相机控制即使得到实现，也并不精确，因为相机参数仅仅是神经网络的输入，网络仍必须推断视频如何依赖于相机。相比之下，GEN3C 由一个三维缓存引导：该缓存是通过预测种子图像或此前生成帧的逐像素深度而得到的点云。在生成接下来的帧时，GEN3C 以根据用户提供的新相机轨迹对三维缓存进行二维渲染的结果为条件。关键在于，这意味着 GEN3C 不需要记忆自己此前生成了什么，也不需要根据相机位姿推断图像结构。相反，该模型可以将其生成能力集中于此前未观测的区域，以及将场景状态推进至下一帧。我们的结果表明，相较于先前工作，GEN3C 具有更精确的相机控制，并在稀疏视角新视角合成中取得最先进的结果，即使是在驾驶场景和单目动态视频等具有挑战性的设置中也是如此。结果最好以视频形式观看。欢迎查看我们的项目网页：https://research.nvidia.com/labs/toronto-ai/GEN3C/ 。

**逻辑拆解**

| 原文逻辑环节 | 含义与后续证据 |
|---|---|
| 背景 | 已有大规模视频生成具有强外观先验，但缺少显式 3D 历史。§1–2。 |
| 问题 1 | 离开视野又重新看到的物体可能消失或改变。Figure 2 用往返相机展示历史遗忘，而非单纯画质不足。 |
| 问题 2 | 相机位姿数值本身并未直接给出像素在目标图像上的位置；网络须同时学习几何映射和内容生成。 |
| 方法 | 深度预测 → 反投影点云 → 缓存 → 新相机投影 → 条件扩散。由外部几何确定部分结构。§4.1–4.4、Figure 3。 |
| 长序列 | 此前生成帧的深度可进入更新后的缓存；缓存成为跨视频片段的显式历史。§4.5、Appendix A。 |
| 结果 | 单/双视角、驾驶、动态等实验分别提供定量及定性证据。§5、Tables 1–4。 |

【背景知识】由已知三维点和目标相机可以确定其在图像平面上的投影，但点云不包含所有遮挡后表面；扩散模型的任务可理解为带部分观测条件的生成。缓存不能替代所有神经时序建模，因为运动、光照变化、未观测区域和噪声修复仍由网络处理。

【独立评价】作者所谓“不需要记忆”指显式空间历史减轻网络隐式记忆负担，而非证明网络没有时序状态或无需关注跨帧信息；“集中全部生成能力”是动机性表述，不是对计算资源的形式化分解。不可见区域本质上多解，补全结果逼真不等于恢复唯一真实场景。Table 4 还显示动态任务的 SSIM 并非最优；摘要中的 SOTA 主张应限定在报告的比较协议与指标。

一句话理解：GEN3C 将几何可确定的像素引导交由三维缓存的投影提供，再利用视频扩散模型处理不完整、不精确的条件与未知内容，以改善稀疏观测下的相机可控性及跨时间一致性。

摘要本身未给出公式，相关数学机制在 §4 和公式精读中展开。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 核心贡献：原文主张及其论证位置

**贡献归纳**

论文 Introduction 未采用编号贡献列表。下列为依据 p.1–2、§4 与 §5 的主张重组，并非原文逐条列出的 Contributions。

1. **问题刻画（问题层）**：缺少显式几何的视频生成，既难以将用户相机参数转换成准确画面，又难以持久记忆离开视野的内容。现象并非作者首次发现；本文将两个问题联系到 3D 表示及渲染这一共同结构。Figure 2 负责提供动机实例，§2 比较相机控制与历史表示路线。
2. **时空 3D 缓存条件生成（核心方法）**：从一个或多个视角、静态或动态输入，以及历史生成帧预测深度并反投影成点云，按时间/视角存储；在目标相机下投影，使用二维条件视频指导扩散生成。相机变换由几何负责，生成模型负责补全与修正。§4.1–4.4、Figure 3。点云、投影、单目深度、扩散本身是继承的工具；新意主张主要在这一组合和训练/推理机制。
3. **多视角条件融合（具体方法）**：保留每个视角的独立点云，分别渲染，再编码、掩码乘法、共享首层及跨视角最大聚合，避免显式合并时深度不一致直接形成混杂的像素条件。§4.3、Eq.2–3、Figure 4、10、Table 6、Figure 16。该结构旨在兼容不同视角数；并不代表网络能无限制地处理任意规模输入或严格校正所有相机误差。
4. **自回归历史与可编辑性（机制与应用）**：将新生成观测通过深度对齐再反投影写回缓存，后续片段从扩展的三维历史中取得条件。显式空间表示还能通过移除或移动点云来修改车辆轨迹，改变目标内外参可制作 Dolly Zoom。§4.5、Appendix A、Figure 1、8。
5. **广泛实验和底座迁移（实验贡献）**：静态单/双视角、驾驶与单目动态 NVS 四类设置，深度扰动与融合消融，以及 SVD 到 Cosmos 的扩展。§5.1–5.7、Tables 1–6、Figures 5–16。作者主张准确相机控制、广泛泛化及高质量生成。

**证据对应**

| 贡献 | 直接依据 | 后续需要审查 |
|---|---|---|
| 几何条件 | Fig.3, §4.1–4.4；Fig.14 | 投影误差修复与新区域真实性不可混为一谈 |
| 多视角融合 | Fig.4/10、Table 6、Fig.16 | 不同视角数、冲突量、遮挡类型及光照的独立控制 |
| 长程记忆 | Fig.1/2, §4.5, Appendix A | 长程累计漂移、错误缓存污染；未给出严格一致性证明 |
| 扩展应用 | Fig.7–9、Table 3–4 | FID 不等于几何精度；动态 SSIM 比 GCD 低 |
| 迁移 | Fig.11–12, §5.7 | Cosmos 对比仅定性、分辨率差异、训练成本 |

【原文事实】论文没有给出关于三维一致性的普遍数学保证。Eq.1 为一般扩散目标，Eq.2–3 为融合设计，Eq.4–5 为对齐优化；这些属于数学化算法，而非独立的新物理理论或可识别性定理。这里只归纳作者贡献主张，独立创新性与充分性留待综合评价。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 结论及作者承认的限制

**结论全文翻译**（§6，PDF p.9）

本文介绍了 GEN3C，一种具有精确相机控制的一致性视频生成模型。我们通过从种子图像或此前生成的视频构建三维缓存来实现这一目标。随后，我们依据用户提供的相机轨迹将缓存渲染成二维视频，以强条件引导视频生成，从而获得比此前方法更准确的相机控制。我们的结果在稀疏视角新视角合成中也是最先进的，即使在驾驶场景和单目动态新视角合成等具有挑战性的设置中亦是如此。

**局限（Limitations）**：当生成具有动态内容的视频时，GEN3C 依赖预先生成的视频提供物体运动。生成这样的视频本身就是一个挑战。一项有前景的扩展是在训练视频生成模型时引入文本条件，以通过提示指定运动。

**结论再核对**

【原文事实】作者总结两条主要成果：几何缓存使相机控制更准确；稀疏视角与动态场景中的 NVS 质量提高。该结论覆盖单视角、双视角、驾驶及单目动态输入，但动态输入本身承担运动条件，不能把该工作解读成已经解决了从文本自由生成可控物体运动。

【独立评价】结论中的 SOTA 需要按指标和任务分别读。Table 1/2 中所报告的 GEN3C 指标优于列出的基线；Table 3 的 FID 为分布相似度，不直接测量目标相机几何；Table 4 的 LPIPS、FID 改善明显而 SSIM 更低。精确相机控制的支持来自 TSED（Table 1）和定性目标姿态对照，未证明任意轨迹具有统一的几何误差上界。自回归缓存扩展可帮助保留历史，但之前生成的错误也可能被固化。后两条为独立评价，非作者在 Limitations 中逐字列出的限制。

后续文本提示运动是作者提出的未来工作方向，而非本文已实现模块。结论段没有新的公式，数学条件与失效机制将在后续分析。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2.1 图表功能总览

**图表功能目录**

| 编号 | PDF 页 | 章节 | 主要功能 |
|---|---:|---|---|
| Figure 1 | 1 | teaser | 总览应用：长视频、静态/动态 NVS、驾驶、Dolly Zoom、编辑 |
| Figure 2 | 2 | Introduction | 动机：往返相机与历史遗忘 |
| Figure 3 | 3 | Method overview | 全方法：时空缓存、投影、扩散 |
| Figure 4 | 4 | 4.3 | 融合：显式、拼接、共享首层与 max-pool |
| Figure 5 | 5 | 5.2 | 单视角 NVS 定性对照 |
| Figure 6 | 6 | 5.3 | 双视角 NVS 定性对照 |
| Figure 7 | 7 | 5.4 | 驾驶偏离轨迹与补全 |
| Figure 8 | 8 | 5.4 | 驾驶车辆移除和轨迹编辑 |
| Figure 9 | 8 | 5.5 | 单目动态视频重观察 |
| Figure 10 | 8 | 5.6 | 多视角几何与光照冲突的融合消融 |
| Figure 11 | 9 | 5.7 | SVD/Cosmos 底座对比 |
| Figure 12 | 9 | 5.7 | Cosmos 极端旋转外推 |
| Figure 13 | 15 | C.2 | 单视角补充基线对照 |
| Figure 14 | 15 | C.2 | 投影修正和缺失区域生成 |
| Figure 15 | 16 | C.3 | Kubric4D 动态时序细节 |
| Figure 16 | 16 | C.1 | 掩码拼接/乘法的 OOD 对比 |
| Table 1 | 6 | 5.2 | 单视角定量与域内/域外差异 |
| Table 2 | 7 | 5.3 | 双视角插值/外推定量 |
| Table 3 | 7 | 5.4 | 驾驶 FID 随偏移变化 |
| Table 4 | 8 | 5.5 | 单目动态定量 |
| Table 5 | 8 | 5.6 | 深度噪声敏感性 |
| Table 6 | 8 | 5.6 | 融合策略定量消融 |

**阅读顺序**

先读 Fig.1–2 确定问题与应用，再以 Fig.3–4 理解缓存、投影及多视角聚合如何连接扩散模型。随后 Fig.5–9 与 Table 1–4 分别回答输入稀疏性、域外泛化、驾驶视角偏移、动态运动保持及编辑。Fig.10 与 Table 6 是同一融合设计的定性/定量互补；Table 5 单独改变深度噪声比；Fig.16 进一步对掩码选择给出定性消融。Fig.11–12 说明更换预训练视频底座的潜力，Fig.13–15 是相应任务的补充结果。

注意区分示意图、可见视觉现象、数据表和完整视频证据。截图无法独立验证所有帧的三维一致性；定量性能也不直接证明所有不可见区域的真实几何。各 Figure 的保真 PDF 与展示副本以 B2 映射记录为准，本报告只复用经核验的 JPEG。图表总览仅建立证据阅读顺序，数学推导与逐模块技术机制留在相应单元。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
### 2.2.1 Figure 1 · 应用总览与世界一致性

![Figure 1](GEN3C/Figures/Figure_01.jpg)

来源：原论文 PDF p.1，teaser；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 1. GEN3C can generate long and temporally consistent videos with precise camera control. We apply it to various applications, including single-view and sparse-views novel view synthesis, monocular dynamic video novel view synthesis, and driving simulation. With an explicit 3D cache, GEN3C further supports generating videos with cinematic effects, such as Dolly Zoom which simultaneously changes poses and intrinsics, and 3D editing.

**Caption 中文**

GEN3C 能够通过精确的相机控制生成长且具有时间一致性的视频。我们将它应用于多种任务，包括单视角和稀疏视角新视角合成、单目动态视频的新视角合成和驾驶模拟。借助显式三维缓存，GEN3C 进一步支持带有电影效果的视频生成，例如同时改变相机位姿和内参的 Dolly Zoom（滑动变焦），以及三维编辑。

**布局与子图**

图由上方多种输入/生成序列及底部应用示例组成。左侧 Inputs 区分 single view（小猫图像）、sparse views（城市场景的少量不同视角）和 dynamic video（河马视频）；右侧 Generated video 给出相机轨迹示意及多个采样时刻的输出。小猫一行出现帧号 26、39、106、118，说明作者展示的是跨长序列的不同时间片，而非四张独立图片。动态河马一行展示在相同输入运动之上改变观看轨迹。底部 Cinematic effect 为雪景的 Dolly Zoom；Driving simulation 显示 Original / Lane change (4m) 和 Original / Editing 两组，区分相机移动与场景内容编辑。

**技术逻辑**

静态输入反投影后，同一时刻点云复制至各目标帧；动态输入则为每个时刻提供几何。相机轨迹改变时，同一部分三维内容在新视角下重新投影，遮挡后内容需由生成模型补全。Dolly Zoom 不只是数字放大：位姿改变观察点，焦距调节视角，两者配合可大致保持前景尺寸而改变背景透视。车辆编辑修改缓存中的对象而非直接在最终帧涂抹。对应 §4.1–4.5、§5.4。

**证据边界**

【原文事实】图展示作者选定的多种应用。它不是同一测试集的统一性能表；仅凭抽帧不能评估 118 帧间是否每一帧连续、真实三维结构是否一致、编辑对象是否在全部视角正确。Dolly Zoom 说明可使用不同内参，不意味着任意内参或极端轨迹均通过定量验证。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
### 2.2.2 Figure 2 · 往返运动中历史遗忘

![Figure 2](GEN3C/Figures/Figure_02.jpg)

来源：原论文 PDF p.2，Introduction；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 2. Motivation: Our model can generate consistent videos when the camera covers the same region multiple times, while previous work produces severe artifacts due to the lack of explicit modeling of the history.

**Caption 中文**

动机：当相机多次覆盖同一区域时，我们的模型能够生成一致的视频；此前方法由于缺乏对历史的显式建模而产生严重伪影。

**布局与子图**

两行分别为 CameraCtrl 与 Ours；列按前进/返回两个运动阶段展示帧 6、29、66、79（以图上的运动阶段和帧号辨认）。室内走廊、门口、楼梯等是用来比较内容身份与空间位置的锚点。CameraCtrl 返回画面出现与早期观测不相符的场景结构；Ours 则更接近此前可见的楼梯与门口布局。重点不是逐帧锐度，而是时间相隔较远后重新出现的区域。

**技术逻辑**

【背景知识】静态世界的一致性要求相机返回旧视角时观察到同一几何与外观，而不只是相邻帧连续。仅输入相机数值和有限时序潜特征，网络需隐式记忆场景；GEN3C 将已有 RGB-D 观测转为显式空间缓存，返回视角能直接重新渲染旧点。§4.5 和 Appendix A 的更新使新生成区域也能进入历史。

**证据边界**

该图是动机与选定实例，不是循环一致性误差实验。图未给出两次相机位姿完全相同的误差阈值、跨全部序列的回访统计或逐帧位姿估计，因此不能推出严格回环闭合或完全消除遗忘。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
### 2.2.3 Figure 3 · 从时空缓存到条件视频扩散

![Figure 3](GEN3C/Figures/Figure_03.jpg)

来源：原论文 PDF p.3，Method overview；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 3. Overview of GEN3C. With the user input, which can be a single-view image, multi-view images, or dynamic video(s), we first build a spatiotemporal 3D cache (Sec. 4.1) by predicting the depth for each image and unprojecting it into 3D. With the camera poses from the user, we then render the cache into video(s) (Sec. 4.2), which are fed into the video diffusion model to generate a photorealistic video that aligns with the desired camera poses (Sec. 4.3 & 4.4).

**Caption 中文**

GEN3C 概览。对于用户输入（可以是单视角图像、多视角图像或动态视频），我们首先通过预测各图像的深度并将其反投影到三维空间来构建时空三维缓存（§4.1）。随后，利用用户提供的相机位姿，将缓存渲染为视频（§4.2），再将其输入视频扩散模型，生成与所需相机位姿一致的真实感视频（§4.3、§4.4）。

**布局与子图**

从左至右四部分：Inputs（single-view V=1,L=1；multi-view V>1,L=1；dynamic video(s) V≥1,L>1）；Spatial-temporal 3D cache（Estimate 深度、Unproject 反投影，静态分支 Duplicate t）；Render 3D cache（按用户相机序列分别投影各个视角的点云）；3D-informed video generation（Encoder 将渲染帧变潜变量，汇聚后与视频扩散/噪声 latent 交互，Decoder 输出 RGB）。图中同一列不同时间的点云来自时间索引，视角维度保留独立来源。黑色缺失区域不是已观测到的黑色物体，正文另有可见性 mask。图内“Spatial maxpool”标签不能独立解释成空间降采样；Eq.3 明确在各视角的首层特征上取最大。

**技术逻辑**

输入不是简单的 V 张图全部融合为单一 3DGS。对于 t 时刻 v 来源，深度和相机参数产生有色点云 P^{t,v}；目标 C^t 下渲染得到 RGB I^{t,v} 与覆盖掩码 M^{t,v}，沿 t 堆叠成 I^v,M^v。E 冻结，编码 z^v=E(I^v)，掩码 min-pooling 至潜空间后相乘，和共同的带噪目标 z_tau 沿通道拼接。共享首层逐视角执行，跨 V 维逐元素 max-pool 后送给余下的扩散模型。训练目标是目标真实视频的干净 latent，而推理从高斯噪声开始迭代去噪并解码。输出仍是 RGB 视频，不是缓存中缺失点的唯一三维恢复。静态复制和动态逐时刻缓存使同一网络能处理两种场景。

**证据边界**

【原文事实】Figure 3 是架构总览，不包含全部通道维度或训练细节；它省略了 CLIP 首帧条件、长视频写回及深度对齐。正文 Eq.2–3 和 Appendix A 优先于简略图示。模型受几何条件引导，但不具有形式化硬性投影一致性约束。

Eq.2–3 的聚合轴和掩码位置按正文明确；完整张量和推导见数学单元。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
### 2.2.4 Figure 4 · 三种多视角融合方式

![Figure 4](GEN3C/Figures/Figure_04.jpg)

来源：原论文 PDF p.4，4.3；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 4. Three approaches to fuse the point cloud from two views.

**Caption 中文**

来自两个视角的点云的三种融合方法。

**布局与子图**

(a) Explicit fusion：P_A、P_B 先合并为 P_{A∪B}，再 Render、Encode 成一个条件潜变量；(b) Latent concat：P_A、P_B 各自 Render 成独立投影，再 Encode 为 z_A、z_B，沿通道 concat；(c) Ours：各自 Render、Encode，并分别进行首层计算，随后 Maxpool 得到 z′。三幅小图的分叉与合流位置是关键：前两种分别在三维表示和潜变量通道合流，本文是在共享网络首层后融合特征。

**技术逻辑**

显式合并要求各深度坐标可靠对齐；来自不同帧的颜色或光照若冲突，三维合并会把不一致投影成重影或相互遮挡。直接 concat 保留不同来源但通道数随 V 变化，网络首层权重输入宽度固定，需事先指定最大 V、填充和排序；交换视角可能导致不同输出。共享首层 f 对每个视角的 masked latent 和同一个 noisy latent 独立处理，跨视角逐元素 max 保持输出形状固定且对排列不变：交换输入顺序不会改变各位置候选值的最大值。它不显式计算置信度或保证选中正确几何；特征可由不同来源在不同通道获胜。§4.3、Eq.2–3。

**证据边界**

Figure 10 和 Table 6 对比的是显式融合与本文方案；正文称 concat 经验上可工作，但图 4 本身没有给出三者完整数值。排列不变是聚合形式的性质，不是跨视角冲突必然解决或视角数任意扩展仍能泛化的证明。

最大聚合维度 V 而非通道/空间维；输出尺寸固定，置换不变性来自 max 的交换性。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
### 2.2.5 Figure 5 · 单视角 NVS 的可见细节

![Figure 5](GEN3C/Figures/Figure_05.jpg)

来源：原论文 PDF p.5，5.2；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 5. Qualitative results for single-view novel view synthesis. Compared to the baselines, our model generates photorealistic novel view images that precisely align with the camera poses. The green zoom-in boxes highlight the fine-grained details that our model can preserve.

**Caption 中文**

单视角新视角合成的定性结果。与基线相比，我们的模型生成了与相机位姿精确对齐的真实感新视角图像。绿色放大框突出显示模型能够保留的细粒度细节。

**布局与子图**

列从 Input、CameraCtrl、NVS-Solver、Ours 到 GT；三行展示不同场景，绿色框为局部对比。第一行观察室内椅腿等细结构与相对位置；第二行比较装饰性室内结构与纹理；第三行以列车车身文字/边缘等为视觉锚点。输入与 GT 并非同一视角，因此不能以直接拷贝输入的局部纹理作为成功。对照 GT，Ours 的局部形状和位置在所示实例中更接近目标。

**技术逻辑**

相机参数条件模型需学习位姿到像素的映射，几何 warp 约束法则可能受深度误差或去噪强度影响。GEN3C 投影缓存提供新视角的结构位置，模型学习修复点云孔洞及错误投影；细结构能否保留与输入深度和条件生成共同相关。对应 §5.2 和 Table 1。GT=Ground Truth，为真实目标视角。

**证据边界**

图只支持展示样本中的可见细节，不能从绿色框单独确认更准确的深度或全部序列的控制精度。Table 1 补充全测试集合指标；未给出逐类别细结构统计。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
### 2.2.6 Figure 6 · 双视角插值和外推

![Figure 6](GEN3C/Figures/Figure_06.jpg)

来源：原论文 PDF p.6，5.3；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 6. Qualitative results on two-views novel view synthesis. Compared to the baselines, our model generates much more plausible and realistic novel views with a smooth transition between two input views even if the overlap is small (such as the first row).

**Caption 中文**

双视角新视角合成的定性结果。与基线相比，即使重叠较小（例如第一行），我们的模型也能生成更合理、更真实的新视角，并在两个输入视角之间实现平滑过渡。

**布局与子图**

左边两列是两个输入视角，后面依次为 PixelSplat、MVSplat、Ours 和 GT；各行不同场景包括室内空间及室外屋顶/场景结构。第一行两个输入重叠有限，难点在中间相机下同时协调两个来源；后续场景能观察结构边界、纹理连续及缺失区域。重建式结果在输入覆盖之外的区域容易出现破碎或模糊，Ours 的画面在示例中更完整。

**技术逻辑**

PixelSplat/MVSplat 输出可渲染 Gaussian splats，依赖输入对三维几何的约束；GEN3C 保留两个独立点云来源，在潜特征层融合，利用视频先验对两视角没有直接覆盖的区域做生成式补全。插值是在两输入观测之间合成；外推是超出其覆盖轨迹范围。两种设定不等价，外推通常未知区域更大。§5.3、Table 2。

**证据边界**

所示“平滑过渡”来自代表性画面与作者主张，截图没有直接量化时间一致性；更完整的视觉外观不证明不可见表面真实。重建基线输出显式场景，本文输出生成视频，任务输出性质有差别。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
### 2.2.7 Figure 7 · 驾驶场景偏离原轨迹

![Figure 7](GEN3C/Figures/Figure_07.jpg)

来源：原论文 PDF p.7，5.4；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 7. Qualitative results on novel view synthesis for driving scene. Our model can fill in the missing regions in the original video even when the deviation is large, while reconstruction-based baselines produce severe artifacts.

**Caption 中文**

驾驶场景新视角合成的定性结果。即使偏移很大，我们的模型也能补全原始视频中缺失的区域，而基于重建的基线产生严重伪影。

**布局与子图**

行分别为 Nerfacto、3D-GS 与 Ours；列为 Original、Left 2m、Left 4m、Up 1m、Up 2m。随着相机从原始轨迹横向左移或上移，路面、车道、路侧物体和背景的遮挡关系改变。重建基线在较大偏离时暴露缺失、拉伸和破碎区域；Ours 生成的道路边界和场景外观更连贯。左右偏移与上移是不同方向的测试。

**技术逻辑**

Waymo Open Dataset 输入包含驾驶视频；附录 B.5 使用 LiDAR 点云提供深度尺度，对逐帧预测深度对齐、反投影并保留时间维度。新轨迹下缓存投影无法覆盖全部目标像素，扩散先验填补这些区域。驾驶基线 Nerfacto 和 3DGS 使用三相机所有训练帧；生成方法不是依靠每个新轨迹的真实监督图像。Table 3 只定量报告横向偏移不同幅度的 FID，而纵向变化为此处定性。

**证据边界**

偏离原轨迹缺少逐像素 GT。视觉合理及 FID 较低不等于交通环境的准确性、物体位置或道路标线完全正确；不能直接将生成结果认定为安全验证所需的真实驾驶数据。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
### 2.2.8 Figure 8 · 在显式缓存中进行车辆编辑

![Figure 8](GEN3C/Figures/Figure_08.jpg)

来源：原论文 PDF p.8，5.4；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 8. Qualitative results on 3D editing for driving scene. We remove and modify the trajectory of cars from the original scene.

**Caption 中文**

驾驶场景三维编辑的定性结果。我们从原始场景中移除车辆，并修改车辆轨迹。

**布局与子图**

图按 Original、Removal、Editing 等展示编辑前后的驾驶画面，两组场景中的绿色框标记被移除或改变位置的车辆及相邻区域。第一种编辑要求车辆消失且原车辆所在区域被合理填补；第二种改变对象空间位置/轨迹，画面中新位置及周围场景需协调。应将编辑区域与未编辑背景分开观察。

**技术逻辑**

缓存是带颜色和空间坐标的点云，允许针对物体区域删除点或对点施加空间变换。之后按目标相机重新投影；移除点后露出的背景未必在原缓存中，生成模型利用上下文补全。因此这不是只在二维像素掩码内编辑，也不是神经模型从任意文字自主决定三维动作。正文 §4.1、5.4 说明这种能力，具体对象分割与编辑交互细节并未完整算法化。

**证据边界**

图为定性编辑示例。没有可核对的真实编辑后视频、编辑后碰撞约束或语义保持指标；不能证明所有对象可以被正确分割或运动轨迹满足动力学。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
### 2.2.9 Figure 9 · 单目动态视频的新轨迹

![Figure 9](GEN3C/Figures/Figure_09.jpg)

来源：原论文 PDF p.8，5.5；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 9. Qualitative results on monocular dynamic NVS. Our model generates plausible novel camera trajectories for the given dynamic video. Note that, when zooming out from the original video, our model successfully reasons depth of field out.

**Caption 中文**

单目动态 NVS 的定性结果。我们的模型为给定动态视频生成合理的新相机轨迹。注意，当相机相较原视频拉远时，我们的模型能够成功推断景深效果。

**布局与子图**

左侧为动态河马输入视频的抽帧；另两组分别表现环绕和拉远的观看方式，每组以多个时刻展示同一动态对象及水面、背景。观察相机改变时的河马姿态与场景边界：模型必须既改变观看方向和距离，又保持对应时间的运动状态。作者特别指出拉远时背景与景深外观的变化。

**技术逻辑**

动态视频的各帧对应不同世界时刻，因此不能将其当作静态场景的多视角观测直接融合。§4.1 定义 P^{t,v}，单目动态 V=1，每个 t 的点云来自相应输入帧；用 C^t 投影该时刻的几何，再由视频扩散同时处理相邻帧与未知区域。这里的“生成轨迹”是沿给定轨迹生成视频，不是模型预测用户应使用哪条轨迹。

**证据边界**

河马例子无新视角 GT，作者关于景深的陈述为定性观察，没有光学焦距、光圈或景深物理模型的测量。输入视频已提供动态对象运动；不能把结果解读为模型独立生成真实运动学。§5.5 Table 4 使用另一有 GT 的合成动态数据集量化。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN FIG-10 -->
### 2.2.10 Figure 10 · 深度和光照冲突下的融合

![Figure 10](GEN3C/Figures/Figure_10.jpg)

来源：原论文 PDF p.8，5.6；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 10. Qualitative results on ablating different fusion strategies. GEN3C can generate a realistic novel view with misaligned depth and different lighting in the input views, while the explicit fusion strategy fails.

**Caption 中文**

不同融合策略的消融定性结果。当输入视角深度未对齐且光照不同时，GEN3C 能生成真实感新视角，而显式融合策略失败。

**布局与子图**

两组示例各以两个小输入图示来源，对比 Explicit Fusion、Ours 和 GT。一个场景可见桌面及其边缘等结构，另一个为室内光照/亮度不同的视角。比较关键是来自两个来源的结构是否重叠错位、亮暗是否产生不协调接缝。显式融合在错位和外观冲突区域有可见伪影，本文输出更连贯。

**技术逻辑**

P_A 和 P_B 直接求并集时，一个真实表面可能因预测深度偏差落在不同三维位置；投影后相互遮挡或形成双重边缘。合并时还丢失来源身份，颜色差异难以归因。独立渲染与共享首层融合保留每个来源的候选图像，在网络特征空间协调冲突。关联 Fig.4 与 Table 6。

**证据边界**

图展示两种不一致因素共同出现时的实例，未分别控制深度误差和光照误差，也未证明网络恢复了真实三维对齐。Table 6 给出总体性能差异，不单独测量颜色归一化或几何注册误差。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-10 -->

<!-- PTH:BEGIN FIG-11 -->
### 2.2.11 Figure 11 · 更换视频扩散底座

![Figure 11](GEN3C/Figures/Figure_11.jpg)

来源：原论文 PDF p.9，5.7；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 11. Qualitative comparison on using different base models: Stable Video Diffusion (SVD) [4] v.s. Cosmos [1]. When having a more powerful video generation model, GEN3C is able to generate more realistic output with less artifacts. Note that the slight misalignment between the two results is due to the models using different video resolutions.

**Caption 中文**

不同基础模型的定性比较：Stable Video Diffusion（SVD）[4] 与 Cosmos [1]。采用更强的视频生成模型时，GEN3C 能生成更真实、伪影更少的输出。注意，两组结果的轻微未对齐是由于模型使用了不同的视频分辨率。

**布局与子图**

两行分别为 SVD 与 Cosmos；各列为 Input 和 New camera 1–4。输入是室内厨房场景，多个目标相机暴露不同的台面、墙体和物体区域。对比时看跨视角物体身份及结构边缘，Cosmos 的部分外观和细节在展示样本中更自然。不能将两行轻微空间位置偏差全部归为相机控制错误，题注明确说明分辨率不同。

**技术逻辑**

§5.7 用 Cosmos1.0Diffusion-7BVideo2World 替换 SVD；渲染帧由 Cosmos tokenizer 编码，其嵌入与 noisy latent 拼接，继续利用相同的几何条件思想。RE10K、DL3DV 上 fine-tune 10,000 steps、batch size 64。本文方法利用视频底座所学生成先验，不要求将点云表示本身替换为 Cosmos 内部几何。

**证据边界**

对比仅定性，底座参数规模、训练数据、分辨率和计算量未完全控制；不能由该图证明某一结构单独带来数值提升，或 Cosmos 在所有表格和动态任务上复现 SVD 的结论。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-11 -->

<!-- PTH:BEGIN FIG-12 -->
### 2.2.12 Figure 12 · Cosmos 极端视角变化

![Figure 12](GEN3C/Figures/Figure_12.jpg)

来源：原论文 PDF p.9，5.7；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 12. Example of extreme NVS using Cosmos as the base model: the input view is the middle one, and our model is capable of rotating significantly to the left and right.

**Caption 中文**

以 Cosmos 为基础模型的极端 NVS 示例：中间图是输入视角，我们的模型能够向左和向右进行大幅度旋转。

**布局与子图**

三幅街道/有轨电车场景图，中间为输入，两侧为相机旋转后的生成图。目标视角新增视野区域并改变结构投影，原视角已可见的街景信息应保持可识别。图中未列出具体角度或相机轨迹误差。

**技术逻辑**

对单视角点云而言，大幅旋转使投影覆盖范围收缩，遮挡后以及原图视野之外的内容不能从单张图唯一恢复。更强的生成底座可提高外观合理性，几何渲染仍只提供已知区域的锚点。与 Fig.11 的底座替换共同解释极端外推潜力。

**证据边界**

作者没有在此给出目标视角真实图或极端角度分组的定量结果。可见画面完整不代表重建未知世界真实几何，也不代表任意旋转都可成功。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-12 -->

<!-- PTH:BEGIN FIG-13 -->
### 2.2.13 Figure 13 · 补充单视角定性对照

![Figure 13](GEN3C/Figures/Figure_13.jpg)

来源：原论文 PDF p.15，C.2；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 13. Additional qualitative results for single-view novel view synthesis.

**Caption 中文**

单视角新视角合成的额外定性结果。

**布局与子图**

列为 Input、GenWarp、MotionCtrl、Ours、GT，展示三组不同场景。应分别比较生成目标与 GT 的布局、视角相关物体相对位置、细节及伪影；不同输入与输出本来就存在目标视角差异。它补充 Fig.5 中未同时展示的两种基线，并保留相同的定性阅读逻辑。

**技术逻辑**

GenWarp 利用几何重投影引导生成，MotionCtrl 属于相机参数条件路线；GEN3C 进一步使用时空 3D cache 和经过训练的条件融合。附录 B.3 说明 GenWarp、MotionCtrl 使用基于 SVD 的官方 checkpoint，测试数据与本文相同。Table 1 是这些方法的定量对照。

**证据边界**

附录 C.2 正文称提供“quantitative”结果，但 Figure 13 题注明确为 qualitative，实际内容为图像而非数值表。这里按实际呈现视作补充定性证据，不捏造数值或把基线论文内容独立验证。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-13 -->

<!-- PTH:BEGIN FIG-14 -->
### 2.2.14 Figure 14 · 显式投影缺陷如何被生成模型修正

![Figure 14](GEN3C/Figures/Figure_14.jpg)

来源：原论文 PDF p.15，C.2；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 14. Illustration of rendered depth images and model outputs. Our model can fix the error in the depth projection (such as the orange handrail in the first image and the light in the second one), and generate realistic content in the missing regions (such as the inpainted railway).

**Caption 中文**

渲染深度图像与模型输出的示意。我们的模型能够修复深度投影的错误（如第一幅图中的橙色扶手和第二幅图中的灯），并在缺失区域生成真实感内容（例如补全后的铁轨）。

**布局与子图**

Input 与 Ours 并列两组。输入是通过深度估计反投影后再投影得到的彩色条件图，黑色孔洞/缺失区清晰可见；并非只显示以灰度编码的标量深度图。第一组列车/铁路场景重点看橙色扶手以及轨道缺失区域，第二组室内图重点看灯和其投影变形。输出相对条件图修复部分破碎形状并补全空白。

**技术逻辑**

该图具体解释生成网络为何不能只做传统点云渲染：估计深度错误引起像素在新相机下的错误移动，稀疏点云或遮挡变化形成 disocclusion。条件扩散并非在缺失像素中直接复制输入，而是根据投影条件和学习先验生成整段视频。mask 告知已覆盖区域；潜空间乘法抑制未覆盖特征。

**证据边界**

图没有对修复后的真实深度进行验证；生成图像中橙色扶手更合理并不意味着其坐标可反推出正确三维表面。题注称 rendered depth images，应根据实际图像与 §4.2 理解为深度引导的彩色投影条件。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-14 -->

<!-- PTH:BEGIN FIG-15 -->
### 2.2.15 Figure 15 · Kubric4D 动态对象细节

![Figure 15](GEN3C/Figures/Figure_15.jpg)

来源：原论文 PDF p.16，C.3；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 15. Qualitative results on Kubric4D [14]. Compared to GCD [54], our method demonstrates superior performance in preserving object details and dynamics in input videos.

**Caption 中文**

Kubric4D [14] 上的定性结果。与 GCD [54] 相比，我们的方法在保留输入视频中的物体细节与动态方面表现更好。

**布局与子图**

两组不同合成动态场景，每组含 Frame 7、Frame 14；对照 Input、Ours、GCD 和 GT。图中可观察投影条件的边缘缺失，以及不同时间的物体位置、外观和细节。阅读时同一时刻横向比较 GT，时间上纵向比较物体运动是否延续；不能拿不同帧的位置变化当作错误。

**技术逻辑**

Kubric4D 具有受控合成的三维点云及动态序列，输入为给定物体运动的视频，目标是换相机观察同一时刻。作者与 GCD 在 20 个保留测试场景进行比较（§5.5），GCD 256×384 输出放大至相同评估分辨率。GEN3C 逐时刻建立点云、投影并由视频模型生成目标序列。

**证据边界**

Table 4 的 LPIPS、FID 支持感知质量改善，PSNR 接近，SSIM 反而低于 GCD；不能以图中物体细节优势声称所有指标均优。合成场景中观察的时序运动不证明真实动态场景中物理一致性。

本图不包含需独立推导的数学公式；涉及的数学定义在相应公式单元展开。

<!-- PTH:END FIG-15 -->

<!-- PTH:BEGIN FIG-16 -->
### 2.2.16 Figure 16 · 掩码进入网络的两种方式

![Figure 16](GEN3C/Figures/Figure_16.jpg)

来源：原论文 PDF p.16，C.1；B2 核验的 JPEG 展示副本，区域及保真 PDF 见 figure-map.json。

**Caption 原文**

Figure 16. Comparison of different strategies for incorporating masking information into the model. (Left) the mask channel is concatenated to the latent as an additional channel. (Right) the mask values are applied directly to the latent through element-wise multiplication.

**Caption 中文**

将掩码信息纳入模型的不同策略比较。（左）将掩码通道作为额外通道与潜变量拼接。（右）通过逐元素相乘将掩码值直接应用于潜变量。

**布局与子图**

左右两列比较驾驶场景的 Latent concat mask 与 element-wise multiplication 两种条件注入。可比较路面、车辆和边界的变形或伪影：图中右列车辆和街景结构相对清晰连贯。该图并非多视角 latent concat 的消融，而是单独改变 mask 输入形式。

**技术逻辑**

附录 C.1 与 §4.3：将 M′ 作为附加通道 concat 会增大首层输入维度，新增训练参数未经过大规模预训练；驾驶横移的掩码分布超出训练条件且无目标 GT 时，新增权重可能泛化差。直接 z^v ⊙ M′ 保持原潜空间通道数，用广播将未覆盖 latent 置零。min-pool 对二值覆盖掩码保守地保留全部覆盖块。零值不是专门的“未知”类别，网络需从条件上下文理解含义。

**证据边界**

只有定性图及作者解释，未提供掩码策略的数值表、不同随机种子或掩码分布的统计。新增参数与 OOD 泛化是作者给出的机制解释，不是已排除其他因素的因果证明。

Eq.2 中 M′ 的单通道空间 mask 广播到所有 latent 通道，乘法不扩充 latent 通道数。

<!-- PTH:END FIG-16 -->

<!-- PTH:BEGIN TABLE-01 -->
### 2.3.1 Table 1 · 单视角与域外泛化

来源：PDF p.6，§5.2。

**Caption 原文**

Table 1. Quantitative results for single view to video generation. RE10K is the in-domain dataset and Tanks and Temples is the out-of-domain dataset.

**Caption 中文**

单视角到视频生成的定量结果。RE10K 为域内数据集，Tanks and Temples 为域外数据集。

**数据复录**

| 方法 | T&T PSNR↑ (dB) | T&T SSIM↑ | T&T LPIPS↓ | RE10K PSNR↑ (dB) | RE10K SSIM↑ | RE10K LPIPS↓ | RE10K TSED↑ |
|---|---:|---:|---:|---:|---:|---:|---:|
| MotionCtrl | 13.46 | 0.46 | 0.42 | 13.60 | 0.59 | 0.46 | 0.1363 |
| CameraCtrl | 15.88 | 0.55 | 0.29 | 18.40 | 0.72 | 0.25 | 0.8033 |
| GenWarp | 16.04 | 0.50 | 0.39 | 15.50 | 0.61 | 0.40 | 0.0330 |
| NVS-Solver | 16.95 | 0.59 | 0.27 | 16.90 | 0.69 | 0.30 | 0.7286 |
| GEN3C | 18.66 | 0.67 | 0.20 | 19.88 | 0.78 | 0.20 | 0.9143 |

**实验解释**

【原文事实】§5.2 两数据集各 100 个测试序列，T&T（Tanks and Temples）域外，RE10K 域内。CameraCtrl 用同样数据、训练协议和视频底座复现，唯一描述的主要条件改变是以 Plücker 嵌入替代缓存渲染。GenWarp、NVS-Solver 使用相同的深度估计和全局尺度/偏移对齐；附录 B.3 补充官方 checkpoint/代码。比较既考察目标图像重合，也考察感知质量及三维一致性。

【背景知识】PSNR 是对像素均方误差的对数度量（dB，越高越好），SSIM 衡量局部结构/亮度/对比度相似性（越高越好），LPIPS 衡量深层特征感知距离（越低越好）；TSED 为作者引用 [69] 的 3D consistency score，本论文未展开其具体计算阈值，不能自行假设它就是相机位姿误差。

【独立评价】在所列基线中，T&T 最强 PSNR NVS-Solver 16.95，GEN3C 18.66（+1.71 dB）；RE10K 最强 CameraCtrl 18.40，GEN3C 19.88（+1.48 dB）。TSED 0.9143 比 CameraCtrl 0.8033 高 0.1110。CameraCtrl PSNR 从域内到域外降 2.52 dB，GEN3C 降 1.22 dB，支持作者对相机几何条件域外适应性的主张，但两个测试集合并非相同内容与轨迹的受控变量，仅能解释为所报告数据集差异。Table 1 中 TSED 只列 RE10K，不能声称域外 TSED 也被量化改善。无置信区间或显著性检验。

指标差值按原表计算；PSNR 差值是 dB，不是线性百分比。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
### 2.3.2 Table 2 · 双视角插值/外推

来源：PDF p.7，§5.3。

**Caption 原文**

Table 2. Quantitative results for two-views NVS. The two values in each table cell represent the interpolation and extrapolation results, respectively.

**Caption 中文**

双视角 NVS 的定量结果。每个表格单元的两个数值分别代表插值和外推结果。

**数据复录**

| 数据集 | 方法 | PSNR↑ (dB) | SSIM↑ | LPIPS↓ |
|---|---|---|---|---|
| T&T | PixelSplat | 21.34 / 17.45 | 0.70 / 0.65 | 0.42 / 0.46 |
| T&T | MVSplat | 20.90 / 16.08 | 0.70 / 0.63 | 0.39 / 0.44 |
| T&T | GEN3C | 22.22 / 20.51 | 0.76 / 0.72 | 0.14 / 0.16 |
| RE10K | PixelSplat | 19.74 / 16.95 | 0.75 / 0.70 | 0.40 / 0.44 |
| RE10K | MVSplat | 21.40 / 15.51 | 0.78 / 0.69 | 0.30 / 0.37 |
| RE10K | GEN3C | 24.08 / 21.56 | 0.86 / 0.83 | 0.11 / 0.15 |

**实验解释**

§5.3：两数据集各 40 个测试序列，随机选两输入帧；插值目标在两帧之间，外推在范围外。基线是前馈稀疏视角重建 PixelSplat 和 MVSplat，接收带相机参数的两张图像并输出 3D Gaussian splats，使用官方 checkpoint/代码（附录 B.4）。GEN3C 不求解统一的稠密显式场景，而以两来源的独立深度点云投影指导目标视频生成。

所有所列插值/外推指标 GEN3C 均最佳。T&T 外推 PSNR 比最佳基线 PixelSplat 高 3.06 dB，RE10K 高 4.61 dB；相应插值优势是 0.88 和 2.68 dB。GEN3C 自身也有明显外推损失（T&T 1.71 dB、RE10K 2.52 dB），因此生成先验减轻而非消除欠约束。视觉实例见 Fig.6。LPIPS 改善尤其明显，与生成模型学习的外观先验相符，但不能据此断言最终输出可提取高精度 3DGS 或网格。所比较的是 NVS 输出，非三维重建精度。

插值/外推差值均按同数据集同指标的第一/第二列数值计算。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
### 2.3.3 Table 3 · 驾驶场景 FID 与轨迹偏移

来源：PDF p.7，§5.4。

**Caption 原文**

Table 3. Quantitative results of FID [18] for NVS on driving scene. GEN3C significantly outperforms the baselines, especially when generating novel views that are far away from the original trajectory.

**Caption 中文**

驾驶场景 NVS 的 FID [18] 定量结果。GEN3C 显著优于基线，尤其在生成远离原轨迹的新视角时。

**数据复录**

| 方法 | y ± 0.0 m | y ± 1.0 m | y ± 2.0 m | y ± 4.0 m |
|---|---:|---:|---:|---:|
| Nerfacto | 48.34 | 67.77 | 80.41 | 112.40 |
| 3D-GS | 34.81 | 53.85 | 61.78 | 81.26 |
| GEN3C | 7.93 | 18.19 | 25.11 | 35.33 |

**实验解释**

§5.4 从验证集筛选 18 个静态场景，用前相机原轨迹的横向位移构造目标轨迹；Nerfacto 与 3DGS 以三个相机全部训练帧优化场景（B.5）。目标新轨迹无 GT，因此评估 FID（Fréchet Inception Distance，越低越好；无量纲分布距离），而不是有参考图的 PSNR。表中 y 表示横移（单位 m）；Fig.7 的上移例不在表格列中。

GEN3C 在零偏移即有较低 FID，偏移 4m 时 35.33，3DGS 为 81.26、Nerfacto 为 112.40。GEN3C 从 7.93 增至 35.33，仍随偏移变大退化。重建方法对新可见区域缺少支撑，而生成先验可改善这些区域的外观。由于 FID 衡量集合统计而非逐帧位姿、对象坐标、驾驶安全或三维一致性，结论应是所报告设置下分布视觉质量优势；作者未报告独立显著性检验，也未给出目标轨迹真实像素。

本表不包含需要独立推导的公式；数据比较使用表内原始数值和指标方向。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
### 2.3.4 Table 4 · 单目动态 NVS

来源：PDF p.8，§5.5。

**Caption 原文**

Table 4. Quantitative results on monocular dynamic NVS.

**Caption 中文**

单目动态 NVS 的定量结果。

**数据复录**

| 方法 | PSNR↑ (dB) | SSIM↑ | LPIPS↓ | FID↓ |
|---|---:|---:|---:|---:|
| GCD | 19.37 | 0.67 | 0.48 | 150.64 |
| GEN3C | 19.41 | 0.63 | 0.29 | 98.58 |

**实验解释**

§5.5：GCD 发布的 Kubric 20 个留出测试场景，使用 GCD 公开训练 checkpoint，原输出 256×384，上采样到本文输出分辨率。输入动态视频已提供运动，目标为沿新轨迹观察同一动态场景。LPIPS 与 FID 较低（改善 0.19 和 52.06），PSNR 仅高 0.04 dB，SSIM 低 0.04。因此优势主要反映感知和集合分布质量，并非四项指标全面领先，也不足以说明几何位置误差更低。分辨率和上采样本身是比较的潜在混杂因素。真实域外 Sora/MovieGen 视频的结果在 Fig.9 只作定性展示。

本表不包含需要独立推导的公式；数据比较使用表内原始数值和指标方向。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
### 2.3.5 Table 5 · 深度噪声鲁棒性

来源：PDF p.8，§5.6。

**Caption 原文**

Table 5. Quantitative results on the robustness analysis on noisy depth estimation. The two values in each table cell represent the interpolation and extrapolation results, respectively.

**Caption 中文**

含噪深度估计鲁棒性分析的定量结果。每个单元的两个值分别表示插值和外推结果。

**数据复录**

| 噪声比 | PSNR↑ (dB) | SSIM↑ | LPIPS↓ |
|---|---|---|---|
| 0% | 24.08 / 21.56 | 0.86 / 0.83 | 0.12 / 0.15 |
| 3% | 22.39 / 21.00 | 0.83 / 0.81 | 0.16 / 0.18 |
| 10% | 20.85 / 19.64 | 0.79 / 0.76 | 0.19 / 0.22 |
| 30% | 18.52 / 17.91 | 0.72 / 0.70 | 0.29 / 0.31 |

**实验解释**

§5.6 沿用 §5.3 双视角设置，对估计深度增加高斯噪声，比例以 d_0.95−d_0.05 的分位数跨度为尺度。0% 表示没有额外扰动而非真值深度。随噪声增加，PSNR、SSIM 降低、LPIPS 增大，表明几何条件质量实质影响输出。3% 时插值 PSNR 下降 1.69 dB、外推 0.56 dB，作者称小噪声下降可忽略，但该表无统计检验且 1.69 dB 不能一概视为可忽略。30% 时对应下降 5.56、3.65 dB；模型仍能输出图像，不能称深度误差不重要。30% 也不是所有像素均相对深度偏差 30%。

【原文差异】0% 的插值 LPIPS 是 0.12，而 Table 2 和 Table 6 的同类值为 0.11；其他基准值一致。保留原表数值，不擅自修正，也不推测究竟是重复实验差异还是笔误。噪声公式第二参数的方差/标准差惯例及采样相关性未详细说明。

原文的高斯噪声写法和分位数尺度见 MATH-05；差值使用表内原始数值。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
### 2.3.6 Table 6 · 独立投影与显式融合

来源：PDF p.8，§5.6。

**Caption 原文**

Table 6. Ablation of different fusion strategies on RE10K dataset. The two values in each table cell represent the interpolation and extrapolation results, respectively.

**Caption 中文**

RE10K 数据集上的不同融合策略消融。每个单元的两个值分别表示插值和外推结果。

**数据复录**

| 方法 | PSNR↑ (dB) | SSIM↑ | LPIPS↓ |
|---|---|---|---|
| Explicit Fusion | 21.81 / 19.87 | 0.79 / 0.75 | 0.21 / 0.28 |
| Ours | 24.08 / 21.56 | 0.86 / 0.83 | 0.11 / 0.15 |

**实验解释**

原表表头未印箭头，方向据 §5.2 同指标补充。§5.6 与 §5.3 相同实验设置，显式融合将多个估计点云直接合并再投影，本文独立投影后共享首层及 max-pool。两种设定下 Ours 的 PSNR 分别高 2.27/1.69 dB，SSIM 高 0.07/0.08，LPIPS 低 0.10/0.13。Fig.10 说明光照和深度错位可能导致显式融合伪影。虽然正文一处只说“predict the interpolation”，题注及数据单元明确同时报告插值与外推，故按实际表格两列解释。该消融并非 Fig.4 全部三种策略的定量对照；latent concat 没有独立行。还没有将融合改动与额外参数规模、不同训练时长分别控制的细粒度消融。

本表不包含需要独立推导的公式；数据比较使用表内原始数值和指标方向。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 2.4 图表之间的逻辑关系

**证据链**

| 问题/主张 | 结构设计证据 | 实验依据 | 读者应能回答 |
|---|---|---|---|
| 相机控制不精确、返回场景不一致 | Fig.2 动机、Fig.3 缓存投影、Appendix A 写回 | Fig.5、13、14；Table 1 | 相机如何从数值变为像素空间约束？ |
| 稀疏视角融合存在几何/外观冲突 | Fig.4 的三个合流位置，Eq.2–3 | Fig.6、10；Tables 2、6 | 为什么先分别渲染再聚合？ |
| 深度不准确，部分区域缺失 | Fig.3、14；§4.2–4.3 | Table 5、Fig.16 | 缺失与错误投影分别怎样处理？ |
| 极端外推下重建覆盖不足 | Fig.1、7 | Tables 2、3 | 为什么生成先验可以改善外观？ |
| 动态物体保持运动状态 | Fig.3 时空缓存；Fig.9、15 | Table 4 | 时间轴与多视角轴为何不能混用？ |
| 可编辑性、基础模型升级 | Fig.8、11–12 | §5.4、5.7 | 哪些能力来自显式条件，哪些来自底座？ |

**互补与缺口**

Fig.4 是设计空间，Fig.10 和 Table 6 是实际比较；Fig.16 为另一独立 mask 设计的定性结果，不能合并为同一变量。Fig.14 解释深度投影为何需要修复，Table 5 测试人工深度扰动对质量的影响，但未分离遮挡缺失、几何错位、深度尺度和不同空间频率噪声。Fig.5/13 的细结构与 Table 1 的总体 PSNR/SSIM/LPIPS、TSED 互补；Fig.6 与 Table 2 在双视角设置互补。Fig.7 的纵向偏移与横移例均为定性，Table 3 只量化横移。Fig.9 和 Fig.15 分别是域外动态定性和合成动态有 GT 的案例；Table 4 只支持相应测试集的数值结论。

【独立评价】架构图与消融表不能代替严格因果实验；作者未量化长程重复访问时的真实几何误差、生成缓存漂移、编辑准确性、遮挡区真实性或所有动态物理规律。完整视频、不同轨迹与更强几何 GT 将提供补充证据。本单元的关系图谱不新增数学公式，具体公式在后文单独解释。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 3.1 研究问题与任务定义

**任务定义**（§1、§4.1、§5.2–5.5）

【原文事实】GEN3C 研究由相机轨迹控制的世界一致性视频生成，具体应用包含稀疏视角新视角合成（Novel View Synthesis, NVS）、驾驶新轨迹模拟和动态 NVS。它不直接学习一个用于每条相机射线确定性积分的统一隐式场，而是提供可投影的近似 3D 缓存作为视频生成条件。任务需同时面对两个目标：真实已观测内容不随目标相机或历史长度任意改变；未知区域能被生成成合理的连续视频。

【背景知识】稀疏观测反问题欠定：可见表面与其投影对目标相机提供一部分约束，被遮挡区域可有多个与输入完全相容的几何/外观解。如果采用确定性重建，仅有一张/两张图时未知几何往往难以估计；生成模型通过训练分布提供先验，但不能从无证据恢复唯一真相。GEN3C 的设计是把已知信息和先验分别交给几何缓存与扩散模型处理，而非声称解决不可识别性。

**输入输出**

| 情况 | 输入 | 缓存 | 输出 |
|---|---|---|---|
| 单视角静态 | 一张 RGB，源相机（缺失时估计） | V=1，复制到 L 个目标时刻 | 沿 C^1…C^L 的 RGB 视频 |
| 稀疏/多视角静态 | V 张 RGB 及源相机 | V 个独立有色点云，时间复制 | 同一场景目标视角序列 |
| 单/多目动态 | V 个同步、同长 L 的视频及源相机序列 | L×V 个逐时刻点云 | 同一运动在新相机下的视频 |
| 自回归长视频 | 种子/缓存和更长目标相机序列 | 随生成片段更新的缓存 | 多片段相接的长视频 |
| 编辑 | 上述缓存经空间操作 | 移除/修改/添加点的缓存 | 重新生成的编辑视频 |

源相机未知时，§4.1 提出 DROID-SLAM 估计，但该系统仍依赖估计是否可靠；论文未说明仅一张无标定图片如何稳定求得所有内外参。目标相机由用户给定，位置/朝向/成像参数控制投影，网络不负责预测应选哪条轨迹。逐像素深度一般来自预训练估计器，真实驾驶数据额外借助 LiDAR 尺度。动态任务假定运动已由输入视频提供（§6）。

【合理推断】可将条件生成抽象为从 p(x_target | seed observations, target cameras) 取样，缓存是中间条件表示；这是辅助理解的概率表达，不是论文单列的公式。因输出是 RGB 序列而非可独立测量的场景模型，NVS 感知指标和几何恢复指标必须区分。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 3.2 研究背景、动机及根本困难

**根因分析**

【原文事实】§1 将应用背景定位于电影制作、VR/AR、机器人和社交平台的可交互内容生成。传统图形学对精细三维资产依赖高；NeRF、3DGS 等 NVS 减少手工建模，但通常需要较密的多视角观测，远离训练轨迹时容易产生伪影。大规模预训练视频生成具有外观先验，但相机可控与跨时间一致性不足。

根因可以拆成三层（【合理推断】，结合 §1–2）：

1. **相机控制缺少显式像素锚点**：仅将相机参数或 Plücker ray embedding 作为网络输入时，网络还需学习场景几何和位姿如何决定目标图像。不同训练分布的轨迹/布局要求泛化；几何反投影再投影可直接把部分源内容放到新图上的合理位置，降低学习这一映射的负担。
2. **历史记忆缺少持久世界坐标**：相邻帧时序模块可保证局部平滑，但物体离开视野后没有稳定的历史地址，返回时可能重采样不同内容。点云缓存为已有表面保存位置与 RGB；再投影可访问与相机运动方向无关的历史。缓存并不存储所有材质、光照或动态状态。
3. **稀疏观测及错误估计造成几何条件不完整**：视场外、遮挡后区域和无纹理/细结构的深度不可靠；直接点云渲染只能显示已估计表面，多来源显式合并可使错位和外观冲突变成伪影。扩散模型提供修复/补全先验；独立视角渲染与融合留给网络协调。这同时引入生成歧义和误差传播风险。

三层困难相互联系，却不等价。相机控制改善不保证内容真实，稳定历史也可能稳定地保留错误；填补未知区域改善外观不能消除观测歧义。本文选择“显式几何提供部分约束 + 生成先验补全”而非要求模型从少量观测建立完备三维世界。

**证据与边界**

Figure 2 是往返相机的直观反例；Table 1 对同底座和同训练协议 CameraCtrl 的比较，提供域内/域外相机控制和一致性相关证据。Fig.14 展示投影错误和空洞，Table 5 测试深度噪声，Fig.10/Table 6 说明融合冲突，Fig.7/Table 3 展示远离原轨迹的重建覆盖问题。长程缓存写回的机制见 Appendix A，但未对比相同长度下无缓存/有缓存的专门回环误差；问题真实存在的证据不等于完整因果拆解或数学证明。本文并非首次发现这些困难，主要贡献是以时空点云缓存和生成式渲染统一处理。

本单元的几何关系在后续公式单元详细推导，不另造原文未提供的公式。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 3.3 相关工作：从显式重建到几何引导的生成 NVS

**研究路线**（§2，PDF pp.2–3；参考文献编号沿用论文）

本节只依据本文对相关工作的描述，不检索引用论文全文；对基线技术细节不作独立事实核验。

| 路线 | 本文引用的代表工作 | 核心思想 | 作者指出的局限 |
|---|---|---|---|
| 优化式 NVS/重建 | 3DGS [25]、NeRF [36]、Nerfacto [51]；大规模 [3,28,63,72]、质量 [2,21,59]、速度 [38,59]、动态 [11,33] | 从多视角重建可渲染场景 | 需要较密输入，极端视角伪影 |
| 几何正则辅助稀疏重建 | [10,39,44,49,55,64,71,76] | 以深度/法线先验补充欠约束优化 | 对先验估计误差敏感 |
| 前馈稀疏视角重建 | PixelSplat [6]、MVSplat [8] 等 [7,22,43,52,56,68,74] | 学习输入视图到可渲染表示或新视图的映射 | 作者认为训练数据有限、域外/极端视角泛化困难 |
| 生成先验辅助重建 | ReconFusion [62]、CAT3D [12] | 利用图像/视频生成先验增强稀疏重建 | 逐场景优化耗时 |
| 接近的并行工作 | ReconX [30]、ViewCrafter [70] | 以几何/生成机制引导新视角 | 作者称依赖 DUSt3R [57] 多视角对齐，薄结构及错位可能产生伪影 |
| 单视图生成引导 | MultiDiff [37] | 深度 warp 作为视频扩散条件 | 本文称仅考虑单视角 |
| 相机数值条件 | [15,31,48,54,60,61]；实验中 MotionCtrl [60]、CameraCtrl [16] | 微调视频模型，以相机参数/嵌入为条件 | 需学习相机到视频映射，训练外相机运动困难 |
| 无训练几何引导 | [19,67]（含 NVS-Solver） | 以深度 warp 调整预训练扩散去噪 | 一致性引导强度需权衡，可能产生伪影或相机控制不足 |
| 持久世界/历史 | [34]、Streetscapes [9]、CVD [27]、StreamingT2V [17] | [34] 从历史 SfM 点云渲染条件并用 GAN；[9] 依赖高度图；[27] 同步生成多个视频；[17] 用 latent feature maps 记忆 | 高度图未必可用、同步帧不能保留视野外历史、潜历史不直接提供 3D 相机控制 |

**区别与继承**

最相近的技术不应笼统归为“完全没有三维”。MultiDiff、ReconX、ViewCrafter、[34] 和训练自由几何 warp 路线已经使用几何信息。本文重点区别是：(i) 以每张图的深度点云作为独立时空缓存元素，不要求所有来源先成功配准为同一个点云；(ii) 在给定目标相机下为每个来源单独渲染；(iii) 在预训练视频扩散首层后做 permutation-invariant 聚合，通过训练学习错误修复和空洞补全；(iv) 用深度仿射对齐把生成片段写回历史。与 CameraCtrl 的受控比较更接近验证数值条件 vs 几何渲染条件的差异。与 PixelSplat/MVSplat 比较的是最终图像，不证明缓存比显式高斯更准确地重建三维表面。

【背景知识】SfM（Structure from Motion）依赖多个观测推断相机和稀疏三维点；从单张图深度反投影不需要同一表面在多个视角有足够对应，但依赖单目先验和相机尺度。GAN（Generative Adversarial Network）与 diffusion 是不同生成训练范式；本文相较 [34] 主要采用深度预测和扩散式协调，而不是首次提出点云条件。Plücker embedding 编码空间射线几何，可提供相机方向信息，不能直接告诉网络场景表面的颜色/深度。

【独立评价】引言中的前人局限属于作者叙述，单篇论文的图表仅支持测试基线上的相对表现，未证明所有前馈重建、数值条件或显式融合必然失败。§2 未独立评估所有所引工作；新颖性应限定为具体机制组合，而非“首个三维可控视频生成”。本单元比较技术路线，不需要新增数学推导。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-01 -->
## 3.4.1 方法：时空点云缓存的构建

**核心内容**（§4.1，p.4；Fig.3；§5.1，p.6）

【原文事实】本文选择从每个 RGB 图像的预测深度反投影得到的带色点云作为缓存基本元素，因为跨域深度估计已有较强能力，且点云可以快速渲染及编辑。以 v 表示来源视角，t 表示时间，缓存是 $L\times V$ 个独立的 $\mathcal P^{t,v}$（不是一个固定大小的稠密三维数组，也不是学习得到的高斯基元）。每个点的几何位置由对应像素深度、相机内参和外参确定，颜色来自源像素。

| 应用 | 源数据 | 构建操作 |
|---|---|---|
| 单图到视频 | 一张 RGB | 创建一个点云，沿时间复制 L 份 |
| 静态 NVS | V 张不同视角 RGB | 每张各自深度预测/反投影，沿时间复制 |
| 动态 NVS | V 个同步且同长 L 的视频 | 每个时间、每个来源各自反投影，不跨时间强制合并 |

源相机参数假设已知，否则用 DROID-SLAM [53] 估计。§5.1 训练中静态真实数据用 DAV2（Depth Anything V2 [66]）逐帧深度估计并对齐 SLAM 场景尺度；驾驶数据使用 LiDAR 尺度；合成 Kubric4D 来自已有点云序列，可以直接渲染 RGB-D。缓存允许选择性移除、添加或移动物体（Fig.8），但这些编辑并非通过一个学习的 4D 流场自动预测。

**机制与证据**

【背景知识】t 表示场景状态变化，v 表示同一时刻的不同空间观测。若两个时间 t、t+1 的动态对象位置不同，将它们简单合并到同一静态点云会同时出现旧、新位置；时空索引保留对应状态，目标第 t 帧仅从该时间的候选点云获得条件。若场景静止，则同一世界几何对任意 t 适用，复制只是索引层面的复用，并不要求重估深度 L 次。多视角独立保存允许来源不完全对齐，不应将其理解成已经获得一致的全局几何。缓存所保存的是有色可见表面采样：源图遮挡后、视野外区域没有直接观测，深度预测也可能产生错误；不能仅凭缓存表示完整物体。

【数学连接】针孔相机下深度图反投影可写为 $X_c=d(u,v)K^{-1}[u,v,1]^T$，再通过源相机的 camera-to-world 变换获得 $X_w$。这里像素坐标 v 与来源视角 v 容易混淆，后文用 $(u_p,v_p)$；补充推导详见 MATH-03。原文 §4.1 未固定深度为 z-depth 或 ray distance 的具体约定，该表达仅在 z-depth 假设成立时适用。

【证据边界】深度与相机估计误差会传递到缓存；没有可靠多视角约束时不存在真实几何保证。单张无标定图的全部内外参估计细节和动态图像同步误差未详细说明。

<!-- PTH:END METHOD-01 -->

<!-- PTH:BEGIN METHOD-02 -->
## 3.4.2 方法：将缓存投影为二维条件视频

**核心内容**（§4.2，p.4；Fig.3、14）

【原文事实】设用户提供目标相机轨迹 $\mathcal C=(C^1,\ldots,C^L)$。§4.2 定义渲染器

$$ (I^{t,v},M^{t,v}):=R(\mathcal P^{t,v},C^t). $$

$I^{t,v}$ 为从目标第 t 个相机观察来源 v 的 RGB 投影；$M^{t,v}$ 标记未覆盖（disocclusion）及需补全区域。结合 §4.3 的乘法与附录 A.1 的 coverage 语义，在乘法中的有效覆盖取 1、未覆盖取 0。沿时间堆叠：$I^v\in\mathbb R^{L\times3\times H\times W}$，$M^v\in\mathbb R^{L\times1\times H\times W}$。共得到 V 个彼此独立的条件视频及其 mask。注意 R 的输入是目标相机 C^t，所有来源在同一个目标相机平面投影；源相机用于创建点云，不是这里所改变的轨迹。

流程：从缓存选取时刻 t 的每个来源点云 → 依据 C^t 投影到同一目标平面 → 处理可见点并记录覆盖 → 收集每个来源的 L 帧 → 各自进入编码器。

**机制与证据**

【背景知识】点云投影只对存在的点建立位置约束。一个源像素对应的三维点，在目标视角下可能移出视野、被更近点遮挡、落到离散像素之间；原来被遮挡的区域在新视角暴露（disocclusion），但没有源点可提供真实 RGB。因此遮挡/缺失不同于已知像素为黑色；使用 mask 可以告诉模型条件可用性。§4.2 没有展开具体 z-buffer、splat 半径、插值实现，不能将某个常见点渲染器的代码细节当作原文算法。Fig.14 的彩色重投影输入显示黑色孔洞及错误投影，正是扩散需要处理的两类问题。显式投影相较纯 pose embedding 的不同在于目标图上的候选结构已经由几何给出，而不是只给出相机坐标。

【数学连接】$R$ 不是最终生成网络：投影对输入点的位置是可计算的确定性几何变换，而 $\hat x=D(z)$ 来自条件去噪。掩码的维度为 1 个通道，后续 min-pool 到 $h,w$ 后在潜变量通道广播；详见 Eq.2–3。

【证据边界】“precise”仅描述引导关系和实验结果，不是生成图像与投影逐像素硬一致。深度、标定、离散采样和可见性误差都可能影响 R，且生成过程可能重写已覆盖区域。

<!-- PTH:END METHOD-02 -->

<!-- PTH:BEGIN METHOD-03 -->
## 3.4.3 方法：多视角条件的注入与融合

**核心内容**（§4.3，pp.4–5；Fig.4、10、16；Table 6）

【原文事实】设计原则是尽可能少引入新可训练参数，使微调网络仍能利用大规模预训练分布。每个 $I^v$ 经冻结 VAE encoder $E$ 得到 $z^v=E(I^v)$；掩码 $M^{v,\prime}$ 从 $M^v$ 经 min-pooling 对齐潜空间网格。训练时从目标真实视频 $x$ 获得 clean latent $E(x)$，构造共同的 noisy latent $z_\tau=\alpha_\tau E(x)+\sigma_\tau\epsilon$。

$$z^{v,\prime}=\text{In-Layer}(\text{Concat}(z^v\odot M^{v,\prime},z_\tau)),\tag{2}$$

$$z'=\text{Max-Pool}\{z^{1,\prime},\ldots,z^{V,\prime}\}.\tag{3}$$

其中 Concat 沿 channel 维，In-Layer 是视频扩散模型共享的第一层；所有来源使用同一个 $z_\tau$，并非 V 个独立的去噪样本。max-pool 在 view 轴按元素聚合，后续剩余层只接受一个形状固定的特征。SVD 原 latent 空间 $L'=L,C=4,h=H/8,w=W/8$；掩码为 $L\times1\times h\times w$，通过广播相乘；拼接得到每个来源的 $L\times8\times h\times w$（在采用上述 SVD 输入通道假设下，不含实现可能另有的独立条件）。

**机制与证据**

以 V=2 的场景为例，分别渲染的两个候选在目标相机下可能一个提供椅背、一个提供遮挡后桌面；若深度相差使椅腿错位，直接点云合并会产生错误叠加。Eq.2 分别在共享网络中提取候选条件特征，Eq.3 在每个特征位置取最大而不规定来源顺序。因此可以让不同来源的有效信息进入融合结果，网络随后通过上下文修复冲突。它没有计算显式配准矩阵或每像素几何置信度，也不是对 RGB 或点坐标直接 max-pool。

另一选择 latent concat 需要固定 V 或预设最大容量和顺序；增加来源会改变通道数及输入权重。本文按 view 共享首层参数，不随 V 增加首层参数数量，但首层计算成本仍随 V 增加。显式融合对坐标及外观一致性要求更高。掩码若作为额外 channel 拼接（Fig.16 左），会引入未预训练的新通道参数；直接乘法（右）保持 latent channel 数，在训练外掩码上作者观察更少伪影。min-pool 意味着一个潜空间单元内只要有未覆盖像素即设为无效；可能损失小的真实细节但避免混入未知区域。

【数学连接】Eq.3 的置换不变性由每位置最大运算交换律保证。$\max(a,b)=\max(b,a)$；推广任意有限 V，不要求先固定视角顺序。$z^{v,\prime}$ 的首层权重共享是必要前提之一；不应将聚合写成对空间分辨率下采样。Eq.2–3 张量及公式细节见 MATH-02。

【证据边界】Table 6 只量化显式融合与本文，Fig.16 是掩码策略的定性对照。正文最大训练 V=4，形式上的任意 V 输入不证明对更大 V 的效果或计算成本不变。max 特征不保证自动筛除错误深度、冲突颜色或遮挡。

<!-- PTH:END METHOD-03 -->

<!-- PTH:BEGIN METHOD-04 -->
## 3.4.4 方法：条件扩散训练与目标数据

**核心内容**（§3，p.3；§4.4，p.5；§5.1，p.6；Appendix B.1，p.14）

【原文事实】在 §3 的 latent diffusion 基础上，本文选择预训练 SVD，将原 image-to-video 模型的前向改为 $f'_\theta$。训练需要“由缓存渲染的有缺陷视频”与“同目标相机轨迹下的真实 RGB 视频 x”的配对。静态真实数据通过同一原视频选取不同时间的视图构建静态缓存和目标连续帧；同步驾驶视频使用 3 相机缓存、FRONT 相机 GT；Kubric4D 用动态点云序列渲染多个来源轨迹及不同目标轨迹。

对于每个配对：①计算 $I^v,M^v$；②冻结编码器得到目标 $z_0=E(x)$ 和各条件 $z^v$；③随机采样扩散时间 $\tau$ 与高斯噪声 $\epsilon$，构造 $z_\tau$；④按 Eq.2–3 共享首层和聚合；⑤以第一帧 CLIP embedding 作为额外条件；⑥通过 Eq.1 的 denoising objective 优化修改后的模型，以干净 $z_0$ 作为回归 target（不是把噪声 $\epsilon$ 当成本文的实际 y）。作者使用 AdamW，lr $3\times10^{-5}$，条件 dropout 15%（rendered videos 与首帧 CLIP），逐阶段提高分辨率、再微调时间层；详细数据设置见 EXP-01。

**机制与证据**

为什么有缺陷的投影能够训练补全？训练中输入点云来自不同视角的估计深度，目标图来自同相机真实观测（或合成 GT），在同一空间坐标下构成从“部分、不准确条件”到“完整目标”的监督。扩散网络通过不同噪声等级的重建目标学习条件分布。目标中的真实遮挡后内容与上下文建立统计联系，模型并非通过确定性的点云插值生成所有未知区域。首帧 CLIP 提供整体语义/外观条件，cache render 提供逐时刻空间锚点，二者不能等同；CLIP 不直接编码全部精确像素位置。训练 dropout 支持推理 classifier-free guidance，但原文没有详述条件是否联合/独立 dropout 的具体代码。

【数学连接】一般扩散 Eq.1 的目标可取 $\epsilon$、$\alpha_\tau\epsilon-\sigma_\tau x_0$ 或 $x_0$；本文的 $y=z_0$ 来自 §4.4 明确说明。对条件模型的期望平方误差在固定输入的理想最优预测为目标条件均值，此为背景知识，不等于最终生成序列简单输出均值。推理通过采样轨迹和多步去噪生成。

【证据边界】配对监督训练能够鼓励与目标相机一致，但原文没有额外显式三维几何一致性损失或外观物理约束；不能推断微调对所有不可见区域给出唯一真实解。

<!-- PTH:END METHOD-04 -->

<!-- PTH:BEGIN METHOD-05 -->
## 3.4.5 方法：推理、自回归长视频与三维历史更新

**核心内容**（§4.5，pp.5–6；Appendix A.1/B.2，p.14；Fig.1–2）

【原文事实】短视频推理：从 Gaussian noise 初始化 latent，依据用户相机序列渲染 cache 并通过修改的 $f'_\theta$ 迭代去噪；用预训练 VAE decoder $D$ 得 RGB。附录 B.2：classifier-free guidance weight 3，25 diffusion steps；正文报告 1 张 NVIDIA A100 生成 14 帧约 30 s。生成长序列时，将视频分为长度 L 的 chunks，相邻片段重叠 1 帧，依次预测。

缓存更新步骤（Appendix A.1）：

1. 第一段按短视频管线生成 RGB。
2. 对片段末帧用 DAV2 预测像素深度 $d$（正文 §4.5 的描述为“for each generated frame”，附录 A.1 明确举末帧实现；此处保留差异，不假定两者完全一致）。
3. 在该生成帧已知的用户相机位姿处渲染旧 cache，取得深度 $d^{\mathrm{tgt}}$ 及覆盖 mask $M$。
4. 优化全局尺度 $s$ 和平移/偏移 $t$，最小化已覆盖处的深度残差：

$$s,t=\arg\min_{s,t}\|(sd+t-d^{\mathrm{tgt}})\cdot M\|_2^2.\tag{4}$$

5. 应用 $d'=sd+t$（Eq.5），结合该帧 RGB 和已知相机反投影，再追加到原有三维缓存。
6. 以扩展 cache 渲染下一段目标相机轨迹，循环直到完成。

这里的 $t$ 是 Eq.4–5 中的深度平移参数，不是时间索引。若所有 chunks 长度相同且相邻 1 帧 overlap，k 段共有 $k(L-1)+1$ 个不同帧（【补充推导】）；实际末段可能较短。

**机制与证据**

为何不能直接追加预测深度？独立深度预测可能处于不同尺度或零点，新旧点云即使来自同一表面也会形成错位。旧 cache 在同一相机下给出部分可比较深度，通过重叠覆盖区最小二乘确定仿射对齐，再将同一变换应用于新增观测。这样既保留旧区域尺度，又能把新视野中生成的区域写入持久历史。返回旧视角时，可重渲染旧空间点，而无需仅靠最近片段的时序特征记忆。这里的“reprojection error”在 Eq.4 实际上是同目标平面上的深度残差，不是优化三维相机 pose 的二维特征重投影距离；相机位姿已知。

【数学连接】Eq.4 可视为在 mask 选中的有效像素上对 $d^{\mathrm{tgt}}$ 关于 $d$ 做一元线性回归；若有效重叠深度方差为零，尺度不可唯一求解；无有效像素时对齐无约束。MATH-04 给出闭式推导和条件。

【独立评价】缓存增长不保证纠错：如果新生成区域是幻觉，其 RGB 和深度可能被当作后续条件永久保存；尺度/偏移只能纠正全局仿射误差，不能修复局部非线性错位。论文未给出缓存冲突清理、回环闭合、增长上限及长期累计误差的系统实验。正文与附录写回帧选择不一致，不能从 PDF 确认最终代码实际写回多少帧。

<!-- PTH:END METHOD-05 -->

<!-- PTH:BEGIN METHOD-06 -->
## 3.4.6 方法：Cosmos 基础模型迁移

**核心内容**（§5.7，p.9；Fig.11–12）

【原文事实】§5.7 将 SVD [4] 替换为 Cosmos1.0Diffusion-7BVideo2World [1]。作者沿用此前微调协议，使用 Cosmos tokenizer 编码缓存渲染帧，与 noisy latent 拼接，利用 RE10K 和 DL3DV 两真实数据集 fine-tune 10,000 步，batch size 64。Fig.11 比较同输入和目标相机下 SVD/Cosmos 输出，Fig.12 展示单图大角度左右旋转。论文称更强预训练底座能够改善图像真实感，减少伪影。

这一扩展不是重新学习缓存几何，也没有把 Cosmos 定义为 3D 重建网络。核心接口是给定目标相机下的缓存渲染与视频模型的 token/latent 条件输入。

**机制与证据**

框架之所以具有迁移性，在于近似几何来自外部缓存，主要结构约束已经投影成二维视频，与 SVD 某一特定时序层没有本质绑定。更强的预训练模型可在欠观测区域提供更丰富的外观先验，缓解极端视角的生成困难。可是两个模型的 token 结构、分辨率、参数与训练分布可能不同，迁移实现需要与目标 tokenizer 和网络输入形状匹配。正文仅说明拼接 noisy latent 与 rendered embeddings，不能把 SVD 的 4 通道、L′=L、H/8、W/8 或所有多视角聚合具体结构无条件套给 Cosmos。

【数学连接】通用接口可理解为 $c_{\mathrm{render}}=E_{\mathrm{base}}(I)$ 与 noisy latent 的条件组合；这是补充抽象，不是 §5.7 新公式。原文未提供 Cosmos 的具体 token shape 或全部网络改动，无法推导相应参数量。

【证据边界】Fig.11–12 是定性，题注明示分辨率差异。不能外推 Cosmos 全部 Table 1–6 指标更好，或由 10K steps 推断训练耗时和可复现成本。

<!-- PTH:END METHOD-06 -->

<!-- PTH:BEGIN MATH-01 -->
## 3.5.1 数学：条件视频扩散训练目标 Eq. (1)

**核心内容**（§3，p.3；§4.4，p.5）

【原文事实】§3：从 $x_0\sim p_{\rm data}(x)$ 采样干净数据，从 $p_\tau$ 采样扩散时间 $\tau$，从 $\mathcal N(\mathbf 0,\mathbf I)$ 采样噪声 $\epsilon$，构造

$$x_\tau=\alpha_\tau x_0+\sigma_\tau\epsilon.$$

训练损失 Eq.1：

$$\mathbb E_{x_0\sim p_{\rm data}(x),\,\tau\sim p_\tau,\,\epsilon\sim\mathcal N(\mathbf 0,\mathbf I)}\left[\|f_\theta(x_\tau;c,\tau)-y\|_2^2\right].\tag{1}$$

| 符号 | 定义 |
|---|---|
| $x_0$ | 干净训练视频 |
| $x_\tau$ | 噪声等级为 $\tau$ 的样本 |
| $\alpha_\tau,\sigma_\tau$ | 噪声 schedule 系数，控制保留信号及噪声量 |
| $\epsilon$ | 标准高斯噪声 |
| $f_\theta$ | 可训练去噪模型 |
| $c$ | 可选条件 |
| $y$ | 预测目标，可为 noise、velocity 或 clean sample |

论文列出的三种 target 是 $\epsilon$、$\alpha_\tau\epsilon-\sigma_\tau x_0$、$x_0$。实际训练 §4.4 在 VAE 潜空间进行：$x\in\mathbb R^{L\times3\times H\times W}$，$z_0=E(x)\in\mathbb R^{L'\times C\times h\times w}$；$z_\tau=\alpha_\tau z_0+\sigma_\tau\epsilon$；$y=z_0$。SVD 中 $L'=L,C=4,h=H/8,w=W/8$，时间维未压缩。条件 c 包含渲染经 Eq.2–3 处理后的条件及首帧 CLIP embedding。最终 RGB 为 $\hat x=D(z)$。

**机制与证据**

【背景知识与补充推导】固定 $x_\tau,c,\tau$ 后，最小化关于预测向量 a 的条件风险 $R(a)=\mathbb E[\|a-y\|^2\mid x_\tau,c,\tau]$。将平方展开为 $a^Ta-2a^T\mathbb E[y\mid\cdot]+\mathbb E[y^Ty\mid\cdot]$，梯度为 $2a-2\mathbb E[y\mid\cdot]$，Hessian 为 $2I$，故理想最优 $a^*=\mathbb E[y\mid x_\tau,c,\tau]$。单次 denoising regression 学习条件统计规律，不是从有限观测直接解析求出真场景。扩散采样从随机初值逐步更新潜变量，不等于一个步骤直接输出均值。对于 clean prediction，在 $\sigma_\tau\ne0$ 时可重排 $\epsilon=(x_\tau-\alpha_\tau x_0)/\sigma_\tau$；这不是原文指定的采样器实现。

训练时有目标视频，推理没有 E(x)，使用采样过程中当前的 $z_\tau$ 与渲染条件。不能将 GT 编码作为推理输入。

【数学条件】条件均值推导要求目标二阶矩有限和足够表达能力；换算噪声需 $\sigma_\tau\ne0$。Eq.1 未在本文推导 score matching 与每种 schedule 的等价性，不能假设 $\alpha_\tau^2+\sigma_\tau^2=1$ 必然对本实现成立。

【证据边界】式 (1) 是继承的扩散目标，创新在条件构造与融合。该损失没有明确深度、位姿或跨帧三维重投影硬约束。

<!-- PTH:END MATH-01 -->

<!-- PTH:BEGIN MATH-02 -->
## 3.5.2 数学：潜空间遮罩与跨视角特征聚合 Eq. (2)–(3)

**核心内容**（§4.3，pp.4–5；Fig.3–4、16）

【原文事实】对来源 $v\in\{1,\ldots,V\}$，$I^v\in\mathbb R^{L\times3\times H\times W}$，$M^v\in\mathbb R^{L\times1\times H\times W}$。冻结 E 得到 $z^v=E(I^v)\in\mathbb R^{L\times4\times h\times w}$（SVD），$M^{v,\prime}$ 为 min-pool $M^v$（pool size $H/h\times W/w$），形状 $L\times1\times h\times w$。原文：

$$z^{v,\prime}=\text{In-Layer}(\text{Concat}(z^v\odot M^{v,\prime},z_\tau)),\tag{2}$$

$$z'=\text{Max-Pool}\{z^{1,\prime},\ldots,z^{V,\prime}\}.\tag{3}$$

$\odot$ 逐元素乘法，mask 单通道广播；$z_\tau=\alpha_\tau E(x)+\sigma_\tau\epsilon$；Concat 沿通道，In-Layer 共享首层，Max-Pool 在 V 个来源之间取最大。

**机制与证据**

【补充推导 1：mask】覆盖 mask 取 0/1，对低分辨率位置 $(i,j)$ 对应的高分辨率块 $B_{ij}$，$M^{v,\prime}_{t,1,i,j}=\min_{(p,q)\in B_{ij}}M^v_{t,1,p,q}$。全覆盖才 min=1；任一未覆盖则 min=0。故 $\tilde z^v_{t,k,i,j}=z^v_{t,k,i,j}M^{v,\prime}_{t,1,i,j}$ 在未覆盖块对全部 k 置零。SVD 下 H/h=W/w=8，整除时每块 8×8，64 像素都覆盖才保留 latent 单元。VAE 感受野可能跨块，min-pool 并不保证严格隔离全部不可靠信息，也可能损失一部分有效细结构。

【补充推导 2：维度与置换】每来源 masked latent 与 noisy latent 均为 L×4×h×w，拼接为 L×8×h×w，首层输出记为 L×F×h_f×w_f（不假定具体 F 或 stride）。对位置 $q=(t,k,i,j)$，$g_v(q)$ 为首层输出，则 $z'(q)=\max_{v=1}^V g_v(q)$。任意来源排列 $\pi$ 有

$$\max_{v=1}^V g_{\pi(v)}(q)=\max_{v=1}^V g_v(q).$$

最大值只依赖候选集合而与顺序无关，且首层共享，故排列不变。V=1 时等于该来源；V=2 时两来源同位置特征 $(1,4)$ 和 $(3,2)$ 得 $(3,4)$，不同通道可来自不同来源，并非选择完整某一个视角或平均。重复完全相同的来源对 max 值不变；新增来源若更大则改变结果。最大值是特征数值选择，不等价于置信度或物理深度最小值。

【补充推导 3：参数/计算】简单拼接 V 个 C 通道条件及 C 通道噪声，首层宽度 $(V+1)C$。对固定输出 F 的二维卷积、核 k×k，参数量与 $(V+1)CFk^2$ 成正比（示意，非本文精确参数量）。共享首层每次输入 $2C$，参数量不随 V 改变；计算量仍含 V 次首层。

【成立条件】所有来源需渲染/编码到相同目标时间与空间尺寸；同一 noisy latent、共享首层和同一条件规则是置换不变前提。若增加来源顺序相关 embedding，需重新分析。最大运算在并列最大处不可微但有次梯度，训练梯度集中于获胜候选，是潜在取舍。

【证据边界】Fig.3 “Spatial maxpool” 不能覆盖 Eq.3 的明确定义；Table 6、Fig.16 支持特定设置的选择，不证明任意 V 和冲突条件的最优性。

<!-- PTH:END MATH-02 -->

<!-- PTH:BEGIN MATH-03 -->
## 3.5.3 数学：相机反投影与再投影（补充几何推导）

**核心内容**（§4.1–4.2，p.4；Fig.1、3、10、14）

【原文事实】§4.1 由 RGB 和估计深度反投影得到 $\mathcal P^{t,v}$，§4.2 用目标相机 $C^t$ 渲染得到 $I^{t,v},M^{t,v}$。论文未给出针孔模型逐步公式。以下为【背景知识及补充推导】，不冒称作者公式或实现细节。

设源像素 $(u_p,v_p)$ 的齐次坐标 $\tilde p=(u_p,v_p,1)^T$，内参 $K_s=\begin{bmatrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}$（无 skew 的针孔相机例）；深度 $d_s$ 假定为源相机 z 坐标。源 camera-to-world 为 $(R_s,t_s)$，目标 world-to-camera 为 $(R_t,t_t)$，目标内参 $K_t$。下标 s/t 指 source/target，与时间 t、Eq.4 的尺度 s 和偏移 t 区分。

**机制与证据**

1. **反投影到源相机系**。由透视模型 $\tilde p\propto K_sX_s$，$X_s=(X,Y,Z)^T$，$Z=d_s$，得

$$X_s=d_sK_s^{-1}\tilde p=\begin{bmatrix}(u_p-c_x)d_s/f_x\\(v_p-c_y)d_s/f_y\\d_s\end{bmatrix}.$$

若深度是沿单位射线的距离而非 z-depth，则改为 $X_s=d_sK_s^{-1}\tilde p/\|K_s^{-1}\tilde p\|$。不能混用。

2. **变换到世界坐标并保存颜色**：$X_w=R_sX_s+t_s$，附带源 RGB。来源相机需在相容尺度和世界坐标系，否则对应物理表面发生冲突。

3. **目标相机坐标**：$X_t=R_tX_w+t_t$。若目标为 camera-to-world $(R_{cw},t_{cw})$，则 $R_t=R_{cw}^T,t_t=-R_{cw}^Tt_{cw}$（旋转正交）。

4. **目标像素投影**：$q=K_tX_t$，$u'_p=q_1/q_3,v'_p=q_2/q_3$。有效点需 $Z_t>0$、投影在图像边界内，并按可见性处理遮挡。离散投影不覆盖每个目标像素，mask 记录覆盖。论文未指定具体 z-buffer 实现。

【补充误差传播】两相机平行、相同焦距 f、水平平移 b，得目标 $u'_p=u_p-fb/d_s$（相机向右移动 b>0）。若估计深度 $d_s+\delta d$，

$$\delta u'=fb\left(\frac1{d_s}-\frac1{d_s+\delta d}\right)=\frac{fb\delta d}{d_s(d_s+\delta d)}\approx\frac{fb}{d_s^2}\delta d.$$

近似需 $|\delta d|\ll d_s$。大基线、近表面、深度误差放大像素偏差；解释显式融合的重影及外推难点。一般相机旋转、内参差异和遮挡不在此特例中。

【补充关系】Dolly Zoom 的近似成像尺寸 $h_{img}\approx fH_{obj}/Z$；改变前景距离 Z 并按比例改变 f 可保持其像高，背景不同深度不保持同样比例，产生透视变化。仅针孔和物体尺寸相对深度较小的近似。

【证据边界】这些公式说明几何引导来源，不是原文 renderer 或深度估计器实现；未知表面、姿态误差和生成修复不能由针孔公式消除。

<!-- PTH:END MATH-03 -->

<!-- PTH:BEGIN MATH-04 -->
## 3.5.4 数学：深度尺度/偏移对齐 Eq. (4)–(5)

**核心内容**（Appendix A.1，p.14；§4.5，pp.5–6）

【原文事实】Appendix A.1 对自回归片段末帧估计深度 $d$，在该帧相机处渲染既有点云深度 $d^{\text{tgt}}$，掩码 $M$ 标明覆盖。原文公式：

$$s,t=\arg\min_{s,t}\left\|(s\cdot d+t-d^{\text{tgt}})\cdot M\right\|_2^2.\tag{4}$$

$$d'=s\cdot d+t.\tag{5}$$

$s$ 是全局 scale，$t$ 是全局 translation/offset（非时间索引）；$d'$ 是对齐深度。$d^{\text{tgt}}$ 为旧缓存渲染深度而非真实传感器 GT。目标相机已知，不在本优化调整。

**机制与证据**

【补充推导：闭式解】展开像素 i，令 $a_i=d_i,b_i=d_i^{\text{tgt}},w_i=M_i^2$。二值 mask 下 $w_i=M_i$，仅覆盖像素计误差。目标 $J(s,t)=\sum_iw_i(sa_i+t-b_i)^2$。对 s,t 求导置零：

$$\sum_iw_i a_i(sa_i+t-b_i)=0,\quad\sum_iw_i(sa_i+t-b_i)=0.$$

令 $W=\sum_iw_i$，$\bar a=\sum_iw_ia_i/W$，$\bar b=\sum_iw_ib_i/W$。第二式给出 $t=\bar b-s\bar a$。代入第一式，均值项消去：

$$s=\frac{\sum_iw_i(a_i-\bar a)(b_i-\bar b)}{\sum_iw_i(a_i-\bar a)^2},\qquad t=\bar b-s\bar a.$$

正规方程：

$$\begin{bmatrix}\sum_iw_i a_i^2&\sum_iw_i a_i\\\sum_iw_i a_i&W\end{bmatrix}\begin{bmatrix}s\\t\end{bmatrix}=\begin{bmatrix}\sum_iw_i a_i b_i\\\sum_iw_i b_i\end{bmatrix}.$$

系数矩阵半正定；$W>0$ 且加权深度方差非零时，行列式 $W\sum_iw_i(a_i-\bar a)^2>0$，唯一无约束最小二乘解存在。$W=0$ 时任意 s,t 给出零损失；若所有覆盖深度相同，s 和 t 不可区分。小重叠或近常深度会病态。例：有效像素 $d=(1,2,3)$，缓存深度 $(3,5,7)$，则 $\bar a=2,\bar b=5$，分子4、分母2，$s=2,t=1$；未覆盖像素同样应用 $d'=2d+1$，但无直接目标深度约束。

几何意义：同一目标相机下新深度需与旧缓存处在相容尺度。过程不改变 RGB，也没有优化相机、非线性局部误差或 3D 刚体旋转。

【成立条件】假设重叠区对应同一表面、mask 可靠、两深度可由全局仿射变换近似关联。原文未给出 s>0、对齐后正值、鲁棒残差或异常值处理约束；补充闭式解仅针对 Eq.4 的无约束问题，不能断言代码使用该闭式求解。

【独立评价】旧缓存错误会传播；非重叠新区域只共享全局变换，未得到真深度验证。写回生成 RGB-D 既可增强历史一致性，也可能放大错误。

<!-- PTH:END MATH-04 -->

<!-- PTH:BEGIN MATH-05 -->
## 3.5.5 数学：深度扰动的相对尺度及鲁棒性

**核心内容**（§5.6，p.9；Table 5，p.8）

【原文事实】§5.6 在预测深度上添加高斯噪声，原文记法

$$\mathcal N(0,s*(d_{0.95}-d_{0.05})).$$

$d_{0.95},d_{0.05}$ 是估计深度的95%和5%分位数，$s$ 为 noise ratio；Table 5 取0%、3%、10%、30%。此处 s 不是 Appendix A.1 的尺度参数。正文未明确正态分布第二参数是方差还是标准差，也未说明跨像素相关性，保留原记法，不擅自修改。

**机制与证据**

【背景知识】分位数跨度 $\Delta d=d_{0.95}-d_{0.05}$ 比 max-min 对极少量极端异常值稳健，提供整个深度分布的场景尺度，而不是每个像素相对自身深度。辅助理解：若第二参数意指标准差，$\Delta d=10$m、s=0.03 时标准差为0.3m；若意指方差则不能如此解释。非论文实测场景数据。由 MATH-03 的平行相机特例，像素误差近似 $\delta u'\approx fb\delta d/d^2$；同样绝对深度扰动在近处/远处、不同目标基线产生不同像素偏差，因此噪声比例与最终指标不存在直接线性关系。

【数字关联】Table 5 插值 PSNR 从24.08到22.39/20.85/18.52；外推从21.56到21.00/19.64/17.91。SSIM 下降、LPIPS 上升。0% 仍依赖原估计深度；人工高斯扰动不代表系统性尺度错、边缘断裂、视图不一致或水下散射等复杂误差。

【证据边界】不从未定义的噪声参数推导精确方差或统计显著性；模型对噪声有一定容忍但随误差退化，不证明任意几何误差下鲁棒。

<!-- PTH:END MATH-05 -->

<!-- PTH:BEGIN EXP-01 -->
## 3.6.1 实验：训练数据与优化设置

**核心内容**（§5.1，p.6；Appendix B.1–B.2，p.14）

【原文事实】§5.1 的难点是缺少真实、同步、多视角动态视频配对，作者以真实静态序列提供空间一致性监督、以合成多目动态序列提供时间一致性监督，并加入真实驾驶数据。

| 数据集 | 规模（论文陈述） | 缓存生成与目标配对 |
|---|---|---|
| RE10K [75] | 74,766 clips，室内外房地产场景 | DROID-SLAM 相机、DAV2 每帧深度、场景尺度对齐；随机等间距 V∈[1,4] 帧，缓存当作静态，L 个连续目标帧包含其中一帧 |
| DL3DV [29] | 10k 视频 | 相同标注和采样流程 |
| Waymo Open Dataset [50] | 1000 scenes，每场景200帧 | DAV2 深度与 LiDAR 尺度对齐；三相机同步 L 帧构成 V=3 缓存，GT 只取 FRONT 相机 |
| Kubric4D [14,54] | GCD 生成的3000场景 | 已有动态多对象点云序列；渲染 V∈[1,4] 条轨迹形成缓存，另一轨迹为 GT |

作者强调 WOD 多相机深度、相机 ISP（Image Signal Processor，图像信号处理）差异使模型学会协调不同来源。真实静态视频可通过同一段序列过去/未来图像模拟目标视角变化，不需要原始采集有多台同步相机，但此法假设静态场景。最大训练 V=4，方法形式允许推理可变 V。数据集规模不等于作者实际训练样本数，原文没有完整训练/验证划分清单。

附录 B.1 训练设置：AdamW、learning rate 3e-5，条件 dropout 15%（cache renders 与首帧 CLIP）；渐进训练三阶段：

| 阶段 | 数据 | 分辨率 | 序列长度 | iterations |
|---|---|---|---|---:|
| 1 | RE10K、DL3DV | 320×576 |14|100K|
| 2 | 四数据集 |576×1024|14|100K|
| 3 | 四数据集，仅微调 temporal layers | 从576×1024 resize 后 center crop 320×576 |随机14–56|10K|

全训练约32张 A100、4天。附录 B.2 推理 classifier-free guidance weight 3、25 steps；正文短序列 14 帧约30秒/单 A100。Cosmos 训练单独见 §5.7。

**机制与证据**

训练配对机制决定模型实际学习何种泛化：静态真实数据提供新相机下的目标 GT；驾驶多相机让条件之间具有现实的不一致；合成动态数据提供同步目标与真实几何，使视频模型学会按时刻保留运动。不能把静态缓存复制当作新增监督视角；复制只是重复同一场景。训练中 GT 是当前目标轨迹，目标与来源不同，监督学习修复不完整投影。

【补充量级】总 SVD 训练迭代为210K；若32张 GPU 约4天持续占用，则约128 GPU-days，仅是按论文时间换算，不是云账单、精确 FLOPs 或单次训练复现成本。推理14/30≈0.47帧/秒，仅表示报告设置，不能推及其他分辨率或 Cosmos。

【证据边界】真实动态多视角新轨迹监督不足仍是核心限制。最大训练 V=4 不证明任意数量和任意相机轨迹的泛化。未报告多数据集独立贡献、精确随机种子和完整训练代码细节。

<!-- PTH:END EXP-01 -->

<!-- PTH:BEGIN EXP-02 -->
## 3.6.2 实验：单视角 NVS、域外与相机控制

**核心内容**（§5.2，pp.6–7；Appendix B.3，p.14；Table 1；Fig.5/13/14）

【原文事实】从一个输入 RGB 预测深度创建缓存，给定目标轨迹生成视频。RE10K 与 T&T 各100测试序列，指标为 PSNR、SSIM、LPIPS，另在 RE10K 报 TSED。基线 MotionCtrl、CameraCtrl、GenWarp、NVS-Solver；CameraCtrl 以同训练数据、协议和视频底座复现，缓存渲染替换为 Plücker embedding。GenWarp 与 NVS-Solver 共用 DAV2 及全局 shift/scale；附录 B.3 说明 GenWarp、MotionCtrl 官方 SVD checkpoint，NVS-Solver 官方无训练代码。

**机制与证据**

Table 1（完整数据已在2.3.1）：GEN3C 在 T&T 达18.66/.67/.20，在 RE10K 达19.88/.78/.20，TSED .9143。相比同底座 CameraCtrl，PSNR 分别提高2.78、1.48 dB；TSED .8033→.9143。最强其他 T&T PSNR 为 NVS-Solver 16.95。Fig.5 和13 提供细结构与布局的定性对照，Fig.14 显示输入投影缺陷及修复；Table 1 是系统质量证据。作者将域外优势归因于显式几何：相机到像素位置映射由投影提供，网络不必只依赖训练轨迹先验。

【背景知识】PSNR $=10\log_{10}(MAX^2/MSE)$，对相同动态范围下的单一 MSE，dB 差 $\Delta$ 对应 MSE 比 $10^{-\Delta/10}$；若表中为逐图 PSNR 的均值，不应直接把均值差换算为全数据集 MSE 改善。SSIM 局部结构、LPIPS 深层特征；TSED 由 [69] 引用，本文未展开精确公式与阈值，不将其改称真实相机轨迹误差。

【证据边界】比较支持指定设置下条件表示优势，但不证明精确位姿误差为零或所有域外相机轨迹有效；T&T 无 TSED，且域内/域外差异还包含场景分布。无误差条、重复训练或显著性报告。

<!-- PTH:END EXP-02 -->

<!-- PTH:BEGIN EXP-03 -->
## 3.6.3 实验：双视角稀疏 NVS 插值与外推

**核心内容**（§5.3，p.7；Appendix B.4，pp.14–15；Table 2；Fig.6）

【原文事实】从视频随机选两帧输入，分别估深并建立两个缓存来源。插值目标位于两输入之间，外推在两输入范围外。RE10K、T&T 各40测试序列。PixelSplat [6]、MVSplat [8] 官方代码及 checkpoint（PixelSplat 为 RE10K 训练版），从带姿态图像预测 3D Gaussian splats，再沿目标轨迹渲染。GEN3C 从两个独立点云渲染视频、融合生成。

**机制与证据**

Table 2：T&T GEN3C PSNR 22.22/20.51，RE10K 24.08/21.56；LPIPS 分别 .14/.16、.11/.15。外推相对 PixelSplat PSNR +3.06/+4.61dB（T&T/RE10K），显示在该测试设置中生成先验优势更明显。Fig.6 第一行重叠较小，重建式方法缺少可匹配观测约束，独立投影与扩散可以补全新露区域。这里并非证明更精确的点云深度，而是显示从相同稀疏输入合成的新视角图像质量改善。

【补充计算】两输入约束的外推损失：GEN3C T&T PSNR 22.22−20.51=1.71dB，RE10K 24.08−21.56=2.52dB，LPIPS 亦上升。说明极端观察仍增加未知区域；与 Table 5 噪声敏感性互补，区分未覆盖与错误覆盖。

【证据边界】本文称任意 V 推理，但定量 Table 2 为双视角，不可推出所有视角数。重建方法的目标可包括显式几何，而本文比较的是输出 RGB；未知区域视觉合理性与真实几何恢复不是同一证据。

<!-- PTH:END EXP-03 -->

<!-- PTH:BEGIN EXP-04 -->
## 3.6.4 实验：驾驶模拟及三维编辑

**核心内容**（§5.4，pp.7–8；Appendix B.5，p.15；Table 3；Fig.7–8）

【原文事实】Waymo 验证集筛选18个静态场景；以原 FRONT 相机轨迹横移构造目标轨迹，Nerfacto 与 3DGS 用三相机全部训练帧优化，官方代码（B.5）。推理驾驶视频对每帧 DAV2 深度和 LiDAR 尺度对齐后反投影，沿时间维形成 3D cache（可存在动态对象）；使用新轨迹渲染和生成。注意“18个静态场景”的定量设置不代表动态编辑示例均被量化。

**机制与证据**

Table 3：横移 y±0/1/2/4m，GEN3C FID 7.93/18.19/25.11/35.33；3DGS 34.81/53.85/61.78/81.26；Nerfacto 48.34/67.77/80.41/112.40。Fig.7 另展示左移及上移视角，Fig.8 通过修改 cache 移除/改变车辆轨迹。道路重建沿原轨迹观察足够，但远离轨迹暴露此前不可见区域，生成先验补全有助于改善外观。编辑后缓存投影缺失的背景由网络补全。

【背景知识】FID 比较真实与生成图像深层特征的均值及协方差：常见形式 $\|\mu_r-\mu_g\|^2+\mathrm{Tr}(\Sigma_r+\Sigma_g-2(\Sigma_r^{1/2}\Sigma_g\Sigma_r^{1/2})^{1/2})$（一般定义，论文仅引用 [18]）。低 FID 不意味着某一目标帧的像素、姿态或对象位置正确。

【证据边界】目标新轨迹无 GT，因此表格只支持分布相似度；不具备安全驾驶验证所需真实几何、交通行为、碰撞等指标。Fig.8 编辑定性，缺对象跟踪和动态物理评估。

<!-- PTH:END EXP-04 -->

<!-- PTH:BEGIN EXP-05 -->
## 3.6.5 实验：单目动态新视角生成

**核心内容**（§5.5，p.8；Appendix B.6，p.15；Table 4；Fig.9、15）

【原文事实】输入单目动态视频，目标是按用户新相机轨迹重新观察同一场景。每帧估深、反投影并沿时间维拼接，逐时刻渲染。Kubric 由 GCD 发布的20个 held-out 测试场景；GCD 公开 checkpoint 在256×384训练，其预测上采样至相同评估分辨率。Sora 与 MovieGen 的动态视频仅域外定性展示（Fig.9），补充 Fig.15 为 Kubric 细节。

**机制与证据**

Table 4：GCD 19.37/.67/.48/150.64，GEN3C 19.41/.63/.29/98.58（PSNR/SSIM/LPIPS/FID）。作者称对象细节和动态保留更好，感知指标支持这一主张的部分内容；PSNR 近似、SSIM 反而差。动态时空 cache 的关键是源视频中运动状态随 t 已知；改变的是观察相机，而非学习完整的运动生成机制。Fig.9 河马环绕/远离，Fig.15 同一帧号对比 GCD、GT；没有将不同时间的状态直接融合。

【补充量化】LPIPS 降0.19，FID 降52.06，PSNR 增0.04dB，SSIM 减0.04；因无重复运行方差，不能说微小 PSNR 差具有统计显著性。

【证据边界】合成动态测试与真实动态泛化不同；Sora/MovieGen 视频无真实目标相机帧。分辨率差异和上采样可能影响指标。作者 §6 明确依赖预先生成动态视频提供对象运动，文本运动条件属未来方向。

<!-- PTH:END EXP-05 -->

<!-- PTH:BEGIN EXP-06 -->
## 3.6.6 实验：融合、深度噪声和掩码消融

**核心内容**（§5.6，pp.8–9；Appendix C.1，p.15；Tables 5–6；Fig.10、16）

【原文事实】§5.6 在 §5.3 双视角设定中分析融合和深度噪声。Table 6 比较显式点云融合 vs Ours；Table 5 对估计深度添加分位数尺度高斯扰动；附录 C.1 Fig.16 比较 mask concat 与 element-wise multiply。必须区分 Fig.4(b) 的多视角 latent concat 和 Fig.16 左的 mask channel concat。

**机制与证据**

融合：Table 6 PSNR 21.81/19.87→24.08/21.56，SSIM .79/.75→.86/.83，LPIPS .21/.28→.11/.15，Fig.10 展示未对齐深度和不一致照明导致显式融合伪影。独立渲染保存来源信息、共享首层和 max-pool 增强协调能力。深度噪声：0%、3%、10%、30% 对三项指标单调退化，说明几何是有效条件而非可忽略输入。0% 并非真实深度；3% 插值下降1.69dB，不宜一概称可忽略。Fig.16 mask 乘法相对新增 mask 通道定性伪影少，作者解释是新增参数对 OOD mask 泛化不足。

【数学关联】Eq.2 的 mask 乘法保持 C 通道，Eq.3 跨 V 聚合；噪声 $\mathcal N(0,s(d_{.95}-d_{.05}))$ 原文未定义第二参数约定，详见 MATH-05。Table 5 0% 插值 LPIPS .12 与 Table 2/6 .11 不一致，按表保留。

【证据边界】只有两种融合策略定量；没有 latent concat 单独行、无真实深度 oracle、没有独立控制深度/光照冲突或长序列缓存消融。mask 策略仅定性，不能将机制解释当因果隔离。

<!-- PTH:END EXP-06 -->

<!-- PTH:BEGIN EXP-07 -->
## 3.6.7 实验：更强基础模型 Cosmos

**核心内容**（§5.7，p.9；Fig.11–12）

【原文事实】Cosmos1.0Diffusion-7BVideo2World 底座，rendered frames 经 tokenizer，与 noisy latent concat；RE10K 与 DL3DV 10,000 steps、batch size64。Fig.11 定性比较 SVD vs Cosmos，四个新相机；Fig.12 单输入左右大旋转。作者说更强底座提供更真实结果和更少伪影，题注承认分辨率造成轻微未对齐。

**机制与证据**

这一实验验证几何渲染条件接口并非只能使用 SVD；在部分观测下，生成底座能力影响未知区域外观。它与缓存的作用不同：cache 提供已有空间结构，底座先验负责未知内容和不完整投影修正。大旋转图显示潜力，但没有 GT 或具体角度。

【数值范围】10K steps×batch64=640,000 个批内样本位次（若步数与 batch size 按常规全局定义；不代表独立训练样本数）。不能用这个数字推定 GPU 时间或与 SVD 的算力公平比较。

【证据边界】未报告 Cosmos 在 Tables 1–6 的同协议数值，未报告分辨率统一、随机种子、模型大小或训练计算控制。不存在可直接据此验证的总体 SOTA 数值。

<!-- PTH:END EXP-07 -->

<!-- PTH:BEGIN SYNTHESIS-01 -->
## 4.1 贡献再评价：创新究竟在哪里？

**核心内容**（§2–6、Appendix A–C 与全部图表）

【独立评价】论文最重要的技术贡献不是新型三维基元、深度模型或扩散方程，而是将显式可投影的三维历史作为视频生成条件，并协调多源信息、跨片段记忆和动态场景。逐项复核如下。

| 内容 | 性质 | 证据强度与限制 |
|---|---|---|
| 历史遗忘与相机控制困难 | 已知问题的统一刻画 | Fig.2、§2；未首次证明困难存在 |
| 时空点云缓存引导视频生成 | 核心方法设计 | Fig.3、§4.1–4.4、Table1–4；继承深度反投影与点云渲染，非新三维理论 |
| 独立渲染+共享首层+max-pool | 具体融合创新 | Eq.2–3、Fig.4/10、Table6；相较显式融合有定量证据，未完整比较 concat |
| 掩码乘法与 min-pool | 条件注入设计 | §4.3、Fig.16；定性支持 OOD 泛化，未量化不同掩码条件 |
| 自回归缓存更新 | 长视频机制 | §4.5、Appendix A、Fig.1/2；深度仿射对齐合理，缺长序列专门消融 |
| 3D 编辑与动态新视角 | 应用能力 | Fig.8/9/15、Table4；运动由输入给定，编辑仅定性 |
| SVD→Cosmos | 框架可迁移证据 | §5.7、Fig.11/12；没有同指标比较 |

理论上，Eq.3 的置换不变性可从 max 操作直接得到，Eq.4–5 属于仿射深度最小二乘对齐。没有新的可识别性定理、生成一致性上界或全局最优保证；作者没有声称推导新的物理渲染理论。

**机制与证据**

把三维缓存视为可直接寻址的几何条件，区别于纯相机数值条件；把生成模型视为能够处理不完整观测的概率先验，区别于仅对已建模表面进行确定性渲染。这个混合设计的意义在于让两类能力互补，而非把生成结果当作完整三维重建。独立渲染的关键不仅是减少配准前处理，也是把多个不一致候选保留到网络内部，让学习过程有机会协调。

【数学关联】Eq.1 是继承的潜空间去噪目标，Eq.2–3 规定注入/聚合，Eq.4–5 规定记忆写回前深度对齐。它们之间形成“几何条件→扩散训练/采样→历史扩展”的因果信息流，但并不合起来构成世界一致性证明。

【尚未证实】与完整引用文献的优先权/新颖性未外部核验；本报告仅依照本篇作者叙述与其自身对比进行评价。

<!-- PTH:END SYNTHESIS-01 -->

<!-- PTH:BEGIN SYNTHESIS-02 -->
## 4.2 结论再评价：哪些成立，哪些尚待验证？

**核心内容**（§4.5–6、Appendix A–C、Table1–6）

**支持较强的结论**：在报告的单视角及双视角协议中，GEN3C 的图像重合度和感知指标全面优于所列基线；与 CameraCtrl 同底座同训练设置比较支持几何条件对相机控制和域外表现的价值。Table 6 对多视角显式融合替代方案提供定量支持。Table 5 随深度扰动退化说明几何条件并非可忽略。

**有条件成立的结论**：驾驶偏移下 FID 较低说明分布外观更接近参考，但没有真实目标轨迹图像；动态 Kubric 的 LPIPS、FID 改善，SSIM 下降，不支持全部指标更优。Cosmos 迁移和编辑是定性能力展示。Fig.2 说明往返场景一致性改善的实例，不是严格回环闭合实验。

**作者明示限制**：动态视频生成依赖已有视频提供运动，文本提示运动属于未来工作（§6）。

**独立技术边界**：

- 输入深度和相机错误：大基线/近距离误差可放大到目标像素，显式独立渲染不能保证修复。
- 稀疏视角歧义：可见表面约束不唯一决定不可见面，生成先验可能产生可信但错误内容。
- 缓存的长程风险：新生成错误可能成为历史；仿射对齐不能解决局部错位，重叠不足时退化。未说明缓存冲突管理或可扩展上限。
- 动态限制：逐帧时空缓存保存给定运动，不是显式场景流、材质/物理模型或可自由推断新运动。
- 实验可比性：输出图像的比较不等于统一几何重建指标；分辨率和预训练底座会影响感知指标。

**机制与证据**

潜在替代解释：强视频底座本身可能改善 LPIPS/FID；独立融合的增益可能部分来自训练和保留来源信息；深度对齐准确程度、输入覆盖大小、相机轨迹难度可能共同影响域外结果。因此进一步实验应在相同底座/分辨率/训练预算下分别关闭 cache、写回、融合、掩码，并用不同覆盖率、真实深度与估计深度、长程回访对照。作者已有的同底座 CameraCtrl 比较能减少部分混杂，但不能隔离所有机制。

【数学关联】MATH-03 的投影误差特例与 MATH-04 对齐可解条件解释何时几何会不可靠；Eq.1 只优化条件去噪目标，不含强制真实几何的约束。因此质量指标和真实场景一致性不应画等号。

【证据边界】上面的补充实验均为独立评价与建议，未实际开展。原文差异（Table 5 LPIPS、正文/附录写回帧数、Fig.13 定性/定量措辞）已保留，不由猜测修正。

<!-- PTH:END SYNTHESIS-02 -->

<!-- PTH:BEGIN SYNTHESIS-03 -->
## 4.3 知识内化与开放问题

**核心内容**（全文综合，§3–6、Appendix A–C）

读完后应能用以下技术链条独立重述 GEN3C：输入带相机的图像/视频 → 估计深度/对齐尺度 → 反投影成 $\mathcal P^{t,v}$ → 按目标 $C^t$ 独立渲染 RGB 与 coverage mask → 冻结编码 → latent mask 乘法 → 与同一 noisy latent 拼接 → 共享首层/跨视角 max-pool → 条件去噪 → 解码 RGB → 长视频时按重叠 chunk 写回对齐深度的历史。

关键辨析：

| 概念 | GEN3C 中的含义 | 不应混淆为 |
|---|---|---|
| 3D cache | 独立有色点云的时空数组及增量历史 | 精确全局 3DGS/mesh、完整 4D 场 |
| Pose control | 目标相机用于几何投影 | 网络预测目标轨迹，或数学硬约束 |
| V / L | 来源空间视角数 / 时刻数 | 单一序列全部帧无条件静态融合 |
| max-pool | 共享首层后跨来源逐元素最大 | 深度最小遮挡、空间下采样、置信度平均 |
| coverage mask | 投影覆盖，min-pool 后抑制未知 latent | 真值物体分割或精确置信度 |
| conditional generation | 学习不完整条件下的目标视频分布 | 唯一真实三维世界恢复 |
| scale/offset | 旧缓存覆盖区域深度仿射对齐 | 相机 pose 优化或全局回环闭合 |
| world consistency | 希望相机回访与动态状态相容 | 已证明所有帧和几何都严格一致 |

开放问题：如何显式区分已观测区域的可信保留与未知区域的可控生成？如何建立多视角/时间的几何与外观置信度？如何防止缓存污染并处理冲突、尺度漂移和长程回访？如何将动态运动状态从观察轨迹中进一步解耦？在无 GT 的真实极端视角下如何评价真实性而非只评价生成画面质量？

**机制与证据**

自检问题：如果把多视角点云先求并集会失去哪种信息？答：来源身份及独立候选，错位已发生在投影前。若只有一个视角，max-pool 是否仍有效？答：跨 V 聚合退化为恒等于该视角首层特征，其余条件生成仍成立。若目标相机大幅移开，缓存全未覆盖会怎样？答：mask 抑制条件，输出更依赖其他条件和预训练生成先验，不能验证真实世界；自回归对齐也可能无约束。为什么动态视频需要不同 t 的 cache？答：时间表示不同场景状态，不能将运动物体不同位置当成同一静态物体。为什么 FID 低不能说明几何正确？答：其比较特征分布而非逐点三维对应。

数学自检：Eq.1 target 是 $E(x)$，不是必然噪声；Eq.2 拼接沿 channel，Eq.3 沿 view；Eq.4 若 $\sum_iw_i(a_i-\bar a)^2=0$，尺度不唯一；Eq.5 将全局仿射应用到整个新深度，包括无真实目标值区域。由这些关系可解释模型的实际机制及失效条件。

【证据边界】以上开放问题与自检属于阅读后独立知识内化，非作者列出的新实验结论。跨论文继承关系和水下迁移尚未验证，留待后续研究笔记；不将本文生成结果外推为真实水下视觉恢复。

<!-- PTH:END SYNTHESIS-03 -->



---

# 第二部分：学习与研究笔记

# GEN3C 学习与研究笔记

> 精读报告 SHA-256：cdb2b2330787909ce7d5e666cd44c954dec074def3c4f97d65c62da256e2aff6
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：关键概念与反事实理解

### 三维缓存不是三维真值

**锚点**：精读 §3.4.1–2、§3.5.3、Fig.3、14。原文将深度反投影的有色点云作为缓存，而非要求训练一个高精度场景函数。把缓存看作“能按目标相机检索的带坐标历史”比称为“重建结果”更准确。

【补充推理】同一源像素可以由多个真实世界解释。例如针孔相机中心相同、沿该像素射线上的两个不同深度点具有同一颜色时，单幅输入无法区分两点。平移相机 b 后，二者目标投影相差 $fb(1/d_1-1/d_2)$ 的绝对值；因此观测歧义在新视角显现。假设 f=500 px、b=0.2 m，d=1 m 或2 m，两种深度的视差分别100 px与50 px。两个世界在源视角同样正确，却在目标图差50 px。这是教学特例，并非本文数据。DAV2 选出其中某个预测，GEN3C 再条件生成；先验选择不能增加真实观测。

### 几何条件与生成先验分工

**反事实 A**：没有缓存，直接使用相机参数。模型仍可能从训练视频学会视差，但目标几何位置不是确定的条件；这与 Table 1 的 CameraCtrl 比较相关。**反事实 B**：没有扩散，只渲染缓存。已知表面可以重投影，却存在空洞和错误投影（Fig.14）。**反事实 C**：将估计点云强行合并再渲染。深度冲突会在网络看到输入之前产生混杂，Fig.10/Table 6 说明实际代价。GEN3C 的设计并非让几何或扩散单独解决问题，而是将几何可确定部分和数据驱动补全分开。A/B/C 中只有部分有专门实验，不能宣称所有反事实已经被同条件验证。

【补充形式化】记观测为 O，世界潜变量为 W，目标相机 C，渲染函数 g，则 $p(X_C|O,C)=\int p(X_C|W,C)p(W|O)\,dW$。观测稀疏时后验 $p(W|O)$ 可能多模态；GEN3C 并未显式表示该后验，而以估计缓存及条件扩散近似目标视频分布。这一抽象帮助分清真实世界歧义与网络随机性：同一模型多次采样差异大可能反映未知信息，也可能只反映扩散模型不稳定，不能直接当作可靠校准的不确定性。

### 最大聚合、遮挡与可信度

**锚点**：精读 §3.4.3、§3.5.2、Eq.2–3。max-pool 是特征聚合，不是 z-buffer。例：两个来源对同一 latent 单元分别产生 $(1,4)$ 与 $(3,2)$，max 为 $(3,4)$，可组合来源，但没有将深度正确性作为显式变量。若错误来源产生强激活，max 仍可能选择它。替代设计包括置信度加权、跨视角注意力或显式几何一致性过滤；这些可能更稳健，也会引入更多可训练参数、依赖校准及训练分布。应在相同底座与预算下对比，不能仅因某设计更复杂就断言优越。

### 记忆写回与反馈风险

**锚点**：精读 §3.4.5、§3.5.4。缓存可视为状态 $S_k$，段生成 $Y_k$（以下记 $Y_k$）与更新 $S_{k+1}=U(S_k,Y_k)$。在已有覆盖区域对齐深度能够减少尺度不一致，但新未覆盖区域的几何和颜色来自模型预测。若错误写回并在后续目标中被重投影，原先一次生成偏差可变成持久条件。一个自检思路是保留不同来源的 provenance（真实观测、生成观测及其置信度），分别衡量回访时真实内容保持和生成内容自洽。它们分别回答“是否真实”和“是否内部一致”，二者可分离。

**边界**：上述概率、误差例子与反馈抽象是教学拓展；论文并没有显式后验、置信度校准或可靠性分层模块。未开展替代实验。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Problem & Task（稀疏观测下的生成式 NVS）
- **Keywords:** Sparse-view NVS; Conditional Video Generation; View Extrapolation; Camera Control
- **Highlight:** 将已有表面重投影为目标相机下的二维条件，并由视频先验补全未知区；输出为 RGB 视频，不等价于显式重建。
- **Evidence:** §1、§4.1–4.4、§5.2–5.3、Tables 1–2；精读 §3.1。
- **Connection Points:** 观测数量、已知相机依赖、目标轨迹、输出表示、外推覆盖。
- **Open Questions:** 何种覆盖范围下生成式 NVS 的图像优势伴随可靠的三维几何？

### Topic: Scene Representation（时空三维缓存）
- **Keywords:** Colored Point Cloud; Spatiotemporal 3D Cache; RGB-D Unprojection; World Coordinates
- **Highlight:** L×V 个独立点云区分时间状态与空间来源，静态沿时间复制，动态逐时刻渲染，支持显式空间编辑。
- **Evidence:** §4.1–4.2、Figures 3、8；精读 §3.4.1–2。
- **Connection Points:** 三维表示形式、时间建模、点云与场景参数的关系、可编辑性。
- **Open Questions:** 相比其他几何条件，独立点云缓存的存储成本、遮挡处理与几何准确性如何权衡？

### Topic: Observation & Prior（几何条件与生成先验）
- **Keywords:** Depth Reprojection; Disocclusion; Conditional Diffusion; Image-to-Video
- **Highlight:** 目标相机投影提供位置锚点，扩散模型修正投影缺陷、补全未观测区域；条件是近似而非硬约束。
- **Evidence:** §3、§4.2–4.4、Eq.1–3、Figures 3、14。
- **Connection Points:** 软硬约束、条件生成、已覆盖与未知区域、目标定义、真实几何验证。
- **Open Questions:** 怎样定量拆分可靠证据遵循与生成先验补全？

### Topic: Geometry Inference（深度错位与融合）
- **Keywords:** Independent Rendering; Shared Input Layer; View-wise Max Pooling; Permutation Invariance
- **Highlight:** 不先合并不一致点云，而在独立投影编码后共享首层、跨视角 max-pool，固定融合后形状；对来源顺序不敏感。
- **Evidence:** §4.3、Eq.2–3、Figures 4、10、Table 6；精读 §3.5.2。
- **Connection Points:** 融合阶段、参数量、来源保留、可变 V、计算成本、置信度。
- **Open Questions:** 最大激活是否与真实几何可靠性相符？对于系统性错位，何种融合更稳健？

### Topic: Generalization（掩码及深度噪声）
- **Keywords:** Coverage Mask; Min Pooling; Element-wise Multiplication; Out-of-Distribution; Depth Perturbation
- **Highlight:** 掩码乘法避免新增 mask 通道参数；人工深度噪声增加时质量退化，说明几何仍是重要条件。
- **Evidence:** §4.3、§5.6、Table 5、Appendix C.1、Figure 16。
- **Connection Points:** 掩码分布、噪声类型、预训练参数、遮挡与误差、鲁棒性测试。
- **Open Questions:** 局部边缘误差或尺度系统误差是否具有与独立噪声不同的影响？

### Topic: Inference & Optimization（持久历史与深度对齐）
- **Keywords:** Autoregressive Inference; Cache Update; Depth Scale and Shift; Loop Consistency
- **Highlight:** 长序列按一帧重叠的 chunk 生成，以旧缓存渲染深度为参考对齐新深度再追加；可能同时保留历史和传播错误。
- **Evidence:** §4.5、Appendix A.1、Eq.4–5、Figure 2；精读 §3.5.4。
- **Connection Points:** 更新规则、重叠条件、缓存增长、漂移、回环评价、真实/生成来源分离。
- **Open Questions:** 有哪些可以区分真实历史与生成历史的写回策略，能否校准长期误差？

### Topic: Training & Supervision（静态真实与动态合成互补）
- **Keywords:** Paired Data Curation; RE10K; DL3DV; Waymo Open Dataset; Kubric4D; Progressive Finetuning
- **Highlight:** 用静态真实视频模拟目标视角，三相机驾驶及合成同步动态视频补足多源/时间监督；训练 V≤4。
- **Evidence:** §5.1、Appendix B.1；精读 §3.6.1。
- **Connection Points:** 训练配对、真实/合成域、运动条件、视角数、数据依赖。
- **Open Questions:** 真实动态配对缺失时，如何评估合成监督的迁移及不确定性？

### Topic: Technical Limitations（视觉质量与几何真实性）
- **Keywords:** FID; LPIPS; TSED; Geometric Fidelity; Dynamic NVS; Missing Ground Truth
- **Highlight:** 驾驶新轨迹无 GT 用 FID，动态 Kubric 的 LPIPS/FID 优而 SSIM 低；不同指标不能互相替代，动态输入已提供运动。
- **Evidence:** §5.4–5.5、Tables 3–4、§6 Limitations；精读 §4.2。
- **Connection Points:** 可见区域评价、几何 GT、感知质量、动态状态、域外验证。
- **Open Questions:** 如何为真实无 GT 极端视角建立可证伪的几何/运动验证协议？


## RESEARCH-DIRECTION：研究定位与可迁移性

本次输入只指定论文，未给本篇独立研究主线。以下首先按本篇开放问题做一般研究分析；领域扩展只是公开可讨论的假设，不绑定任何未公开项目或既有实验。

| 层次 | 对本篇的定位 |
|---|---|
| Field | 3D Vision、Video Generation、Computer Graphics |
| Task | 给定目标相机的视频/NVS，兼顾时空一致性 |
| Task Setting | 单/稀疏/多视图，源/目标位姿，静态或给定动态，极端外推 |
| Research Problem | 稀疏、噪声或不完整几何观测下如何利用证据与生成先验实现可靠新视角推断，并维护长程世界一致性？（研究抽象，不是作者的逐字 RP） |
| Method | 时空 point cache、目标投影、mask、共享首层跨视图 max、扩散生成、仿射对齐写回 |

**直接联系**：GEN3C 展示一种把观测转成显式几何条件的混合路线。对稀疏观测研究，首先应明确真实未知与重建失败的区别：空洞可能因为观测缺失，错位可能因为深度估计错误，两者不能以同一种置信度处理。Table 5 对误差敏感性提供直接证据，Table 2 外推结果说明生成先验有补全潜力；但不能从这里得到真实三维恢复能力。

**领域扩展假设：水下稀疏观测**。水下相机空间外观变化可同时来自几何、光照、介质吸收/散射与相机响应。如果简单将源像素颜色附在三维点上，在新相机距离变化时，不同路径长度的传播效应可能使正确几何点的外观仍不一致；此时 Fig.10 所示跨来源颜色冲突不再仅是光照/ISP/估计误差，而可能有系统性的路径依赖。一个待验证的问题是能否把真实几何证据与外观/介质导致的观测不一致分开，而不强制使用某种特定渲染或扩散结构。该判断是通用成像知识与本篇机制结合的假设，GEN3C 未做水下验证，也没有水下介质传输模型。

**迁移条件**：可靠源位姿/尺度，合适的深度先验，能区分已观测与遮挡的可见性，目标相机或相机估计，足够领域配对数据或可验证合成数据。若介质导致深度先验偏移，则几何条件可能系统性错误；若训练数据不足，生成模型可能把水下外观变化解释为对象或几何改变。是否采用生成式 NVS、几何恢复、深度推断或其他任务是后续选择，不能预先当成研究结论。

**评价要求**：至少分开观测内的真实几何/外观保持、未观测部分不确定性、视角变换一致性与生成质量。已知 GT 的合成可控场景与少量真实验证应互补；仅用 FID/LPIPS 无法证明恢复真实几何或介质。


## RESEARCH-PROBLEMS：从技术现象形成可证伪问题

以下不是已验证创新，也不是全面文献调研；仅依据 GEN3C 证据形成值得探索的候选。优先考虑能够区分真实几何证据与生成先验、且有可控数据验证的问题，不将具体网络结构当作科学问题。

### RP-A · 稀疏观测中如何校准几何条件的可靠性？

**现象与科学未知**：Table 5 显示深度误差增加时 NVS 性能退化；Fig.10/Table 6 显示独立渲染比显式合并稳健。未知是网络是否能够识别错误几何及可靠观测，而非仅以强生成先验掩饰错误。本文 max-pool 没有明确置信度输出。

**可证伪假设**：在已知 GT 几何和相机的稀疏观测下，若区分缺失（mask=0）、错误覆盖（mask=1 但位置不可靠）和多源颜色冲突，并利用可校准的可靠性信号约束融合，则已覆盖区域的真实几何保持/新视图一致性优于无可靠性融合，同时不必显著降低未覆盖区域的感知质量。失败情形是只提高 LPIPS/FID，真实几何误差不减或校准失效。

**验证设计**：固定视频底座、训练/测试集、目标轨迹、采样步骤与输入视角数。由同一 GT 几何生成 ①真深度 oracle，②实际估计深度，③控制全局尺度/偏移、局部边缘错位、随机噪声，④独立颜色变化，⑤组合扰动；按可见性和深度误差分组。比较独立渲染+max、显式融合、置信度加权/过滤候选以及无几何条件；不默认某种方案有效。分别报告已覆盖区 reprojection/深度误差、GT 图像指标，未知区 LPIPS 与多样性、置信度—真实误差校准曲线。若降低置信度只来自拒绝困难像素，要同时报告覆盖率与风险，不能靠减少输出区域“提高”准确率。每种扰动独立控制、重复随机种子。

**可行性与限制**：合成已知几何场景可生成真深度和遮挡标签；真实数据几何 GT 不足，需独立校准。潜空间置信度和像素置信度并非天然对应。应先核查现有可靠融合及 uncertainty 工作，不能宣称候选原创。

### RP-B · 长视频的显式记忆如何防止生成误差固化？

**现象与未知**：Fig.2 提示回访一致性；Appendix A.1 将生成帧的深度仿射对齐后写回。旧 cache 既是约束也可能包含先前幻觉。未知：不同长度和覆盖范围下，生成历史能否提升真实一致性而不是只提升对自身幻觉的一致性？

**可证伪假设**：区分原始观测与生成观测的来源、写回置信度和冲突检测，可降低长程回访误差随片段数增加的斜率，并在相同新区域覆盖量下优于无条件追加。若仅因拒绝写回导致新区域完全丢失、生成质量明显退化或真实回访误差无改善，则假设不成立。

**验证设计**：静态场景用已知环形轨迹及真实几何、固定初始视图与 chunk 长度；改变回访间隔、重叠比例和生成片段数。对比不写回、原方法仿射写回、只使用真实来源、可靠性门控/冲突回收、真值 RGB-D 追加上限。量化重复位姿相对 GT 的像素及几何误差、重复观测一致性、已知与新生成区分别误差、覆盖率、存储与推理时间。固定底座和去噪随机种子，多次重复；记录回访误差随时间曲线，区分“相同假象被再现”与“真实场景保持”。

**条件与限制**：对齐 Eq.4 需要非退化的覆盖深度；重叠不足时应单独分组。动态场景不能直接把旧时刻 GT 用于新时刻比较，需保持运动状态一致；可先在静态可控环境证伪再扩展。

### RP-C · 外观变化与几何不一致如何在跨域稀疏观测中解耦？

**现象与未知**：Fig.10 包含深度错位和照明差异，WOD 的多相机 ISP 差异由训练学习协调。在介质参与成像场景中，即使几何正确也可存在随视线/距离的系统性外观变化。未知：跨视角特征不一致是否可分解为真实几何错位与成像变化，且能否提升可靠推断？这是领域扩展假设，未在本文验证。

**可证伪假设**：在几何、位姿已知的受控模拟中，显式区分成像变化和几何误差的可靠性机制，能在不同介质/外观条件下保持正确对应的识别能力，减少几何误判。若只在训练介质分布内有效，或只改善图像外观而错误几何无法区分，则不支持假设。

**验证设计**：固定场景几何、相机与照明初值，分别改变传播/颜色成像参数以及深度错误；用真深度、同光照对照、无成像变化条件、不同错误深度建立可辨识基线。比较直接 RGB/latent 一致性、条件化特征、可靠性模型、必要的物理先验候选；对真/伪对应计算 ROC、校准、几何误差和最终 NVS/深度任务质量。保持视图数和视角轨迹相同，先验证同一表面正确匹配而非优化最终 RGB 指标。合成到真实迁移需要独立测量或可信参照；不能将生成模型视觉去雾当作真实介质恢复。

**优先级**：RP-A 最直接由 Table 5/6 支撑、可用真深度构造因果对照；RP-B 与论文记忆机制直接相关，但长程计算及动态控制成本更高；RP-C 跨领域意义明确但数据与介质参数可辨识性风险更高。排序仅基于当前证据、可证伪性和初始实验成本，未做外部研究查新，不能作为正式选题决定。


## INTERNALIZATION：独立复述与知识地图

**一句话复述**：GEN3C 用预测深度产生可投影的显式三维历史，在目标相机下把已有内容转换成不完整二维条件，借预训练视频扩散修复错误及生成未知区域；它在多来源特征层聚合、在长序列中更新缓存，以改善相机控制和世界一致性。

**三条相互区别的链**：研究逻辑＝欠约束观测/隐式历史不足 → 显式几何约束与数据先验分工 → 相机控制/补全/一致性；信息流＝RGB-D→P^{t,v}→I^v,M^v→E→masked latent→与 noisy latent concat→共享首层→view max→denoise→D→RGB；监督流＝同一目标轨迹的 GT latent E(x) 与预测 latent 的 denoising objective，配对数据来源于真实静态/驾驶、合成动态。监督流不是推理输入。

**可迁移知识地图**：几何投影（坐标、深度、遮挡）连接表示（时空缓存、源/目标相机）；表示连接融合（来源独立、max、mask）；融合连接生成（条件分布、潜空间、去噪）；生成与表示通过写回形成反馈。评价应沿同一图谱分别检查 geometry fidelity、perceptual quality、temporal consistency、OOD、cache reliability。学习目标是掌握这一分工，而非把特定视频底座当作科学问题。

**自测**：

1. 给定两张图，先拼接所有点云和先独立投影有何差别？前者要求几何对齐且冲突可能在投影前发生，后者保留各来源候选，网络内部协调。后者仍不保证正确。
2. max-pool 是否能选出最准确的深度？不能直接推出；它作用于共享首层的特征数值，未显式计算深度置信度。
3. mask=1 是否等同可靠观测？不等同。它表示点云覆盖，预测深度可能错误。mask=0 则是没有覆盖而不是“真实黑色”。
4. 单目动态 NVS 是否从文字生成物体动作？不是。运动来自源视频（§6 明示限制），新相机仅改变观察过程。
5. 已知位姿、预测深度与目标新图之间的映射为什么可解释？目标投影由针孔变换确定；深度误差传播到像素，扩散对缺失和错误条件学习修正，不能保证真实几何。
6. 为什么更低的 FID 不足以证明真实世界一致性？FID 只比较集合特征分布，不能检验每个物体坐标或回环一致。
7. 长程写回是否一定改善？不一定。无重叠、常深度时对齐欠定，错误 RGB-D 可能成为持久历史。

**下一轮阅读重点**：辨析不同方法何时直接重建、何时生成新视角；先验如何进入模型、是否有置信度；评价是否独立验证几何真实性。后续跨论文对比须读取各自原文，本笔记索引只提供待比较维度。


## GLOSSARY：术语表

| English | 中文 | 技术含义/相近概念辨析 |
|---|---|---|
| Novel View Synthesis (NVS) | 新视角合成 | 给定观测与新相机生成目标图；不必输出显式三维重建 |
| Sparse-view NVS | 稀疏视角新视角合成 | 少量输入造成欠约束，插值/外推难度不同 |
| Interpolation / Extrapolation | 插值 / 外推 | 目标在输入范围内 / 外；Table 2 分列 |
| World Consistency | 世界一致性 | 相机回访同一世界的内容相容，不等于短时平滑或已证明真实性 |
| Spatiotemporal 3D Cache | 时空三维缓存 | L×V 个独立有色点云，时间对应状态，视角对应来源 (§4.1) |
| Colored Point Cloud | 有色点云 | 三维位置与颜色样本；不等同 3DGS 的高斯参数和体渲染 |
| RGB-D Unprojection | RGB-D 反投影 | 由像素深度和相机得三维点，深度约定须一致 (MATH-03) |
| Camera Intrinsics / Extrinsics | 相机内参 / 外参 | 成像参数 / 相机坐标变换；Dolly Zoom 同时改变两者 |
| Plücker Embedding | Plücker 射线嵌入 | 射线几何条件，不直接提供表面颜色和深度 |
| Disocclusion | 去遮挡 / 新暴露区域 | 新视角原来被遮挡的区域，无已知点不等于黑色物体 |
| Coverage Mask | 覆盖掩码 | 表示投影覆盖，不直接代表深度置信度 |
| VAE | 变分自编码器 | E 编码 RGB，D 解码 latent；本文冻结 E |
| Latent Diffusion | 潜空间扩散 | 在压缩空间加噪去噪，本文回归 clean latent (Eq.1) |
| Denoising Score Matching | 去噪分数匹配 | 原文 Eq.1 的一般训练目标；实际 y=z0 |
| SVD / Stable Video Diffusion | 稳定视频扩散 | 本文主要预训练 image-to-video 底座，空间压缩 8 倍 |
| Cosmos | Cosmos 视频基础模型 | §5.7 更强底座，用 tokenizer 与条件拼接；非直接等同 SVD |
| View-wise Max Pooling | 跨视角最大池化 | Eq.3 共享首层后的逐位置最大值，非空间降采样 |
| Permutation Invariance | 排列不变性 | 共享首层+max 对视角输入顺序不敏感，不等于任意 V 效果不变 |
| Min Pooling | 最小池化 | 对覆盖 mask 保守降采样，任一无覆盖则抑制对应 latent |
| Latent Concatenation | 潜变量拼接 | 沿 channel；多视角拼接与额外 mask 通道拼接是两种不同策略 |
| CLIP Embedding | CLIP 嵌入 | 首帧语义/外观额外条件，不是逐像素几何约束 |
| Classifier-free Guidance (CFG) | 无分类器引导 | 条件/无条件预测组合引导；附录 B.2 weight=3 |
| Autoregressive Inference | 自回归推理 | 分 chunk 依次生成，邻段一帧重叠 |
| Scale and Shift Alignment | 尺度与偏移对齐 | Eq.4–5 在覆盖处最小二乘对齐深度，非相机位姿优化 |
| DROID-SLAM | DROID-SLAM 位姿估计 | 源相机缺失时的估计工具；真实训练数据尺度参考 |
| DAV2 / Depth Anything V2 | 深度估计模型 | 预测每像素深度，非真值传感器 |
| LiDAR | 激光雷达 | WOD 提供尺度对齐参考 |
| ISP | 图像信号处理 | 多相机不同响应/处理可能造成颜色冲突 |
| PSNR | 峰值信噪比 | 像素误差对数指标 dB，越高越好 |
| SSIM | 结构相似性 | 局部亮度、对比度与结构相似，越高越好 |
| LPIPS | 感知相似距离 | 深层特征距离，越低越好 |
| FID | Fréchet Inception Distance | 集合特征统计距离，越低越好，不直接评价逐帧几何 |
| TSED | 论文引用的三维一致性评分 | Table 1 越高越好；本篇未详述定义/阈值，不冒充位姿误差 |
| OOD | 分布外 | 与训练分布不同的数据/轨迹/mask；不能由单个案例证明普遍泛化 |
| Observability / Identifiability | 可观测性 / 可辨识性 | 观测是否包含相关信息 / 观测是否唯一确定未知量，生成先验不能等同新观测 |

**使用说明**：术语含义结合本篇实际语境；标准背景解释不替代具体实现。详细公式与实验条件见冻结精读报告相应章节。


## SOURCES：来源、可追溯性与未证实内容

**实际来源**：用户提供的 arXiv:2503.03751v1 PDF（16页），日期2025-03-05，正文§1–6、Appendix A–C、Figures 1–16、Tables 1–6、Eq.1–5、References [1]–[76]。同下载 ID 的 Source/HTML 为辅助载体，Source 无独立版本印记、HTML 缺外部装饰资源；PDF 为最终权威。PDF SHA-256：`5f2462a60f2bed4f097ea79680de022eec4e5ea2f2a30660af9c2a2e80a067d7`。

**冻结报告绑定**：`GEN3C_精读报告.md` SHA-256 `cdb2b2330787909ce7d5e666cd44c954dec074def3c4f97d65c62da256e2aff6`。本笔记的技术定位以该报告为锚，必要时回查同版 PDF。官方元数据仅核验论文版本和官方项目资源；未读取其他论文、代码/权重、用户私密材料或外部研究全文。

**证据分类**：原文事实指作者确实陈述或图表实际显示，未必等于独立证明；背景知识提供解释；补充推导/合理推断注明假设；独立评价和 RP 候选是本篇分析延伸；尚未证实事项不得写成论文结论。

**未证实项**：任意 V 的泛化、长程回环真实几何保证、生成历史可靠性、独立掩码/融合因果、动态自由运动生成、Cosmos 全任务定量、真实水下领域迁移；研究笔记提出的验证机制均未实际实施。原文 Table 5 插值 LPIPS 差异、§4.5 与附录 A.1 的写回范围措辞差异、Fig.13 题注与附录定性/定量表述差异均按原样保留。

**跨文献分析延期**：CROSS-PAPER 只记录本篇技术节点与未来关联问题，不建立已核实的跨论文比较或创新性/优先权结论。笔记可进入隐藏完整版，隐藏不是私密访问控制；未写入个人路径、未公开研究记录或私人数据。



