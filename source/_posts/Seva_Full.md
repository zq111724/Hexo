---
title: "Stable Virtual Camera: Generative View Synthesis with Diffusion Models（完整精读与研究笔记）"
date: 2026-11-27 05:46:52
index_img: "/Seva/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/Seva/)

# 第一部分：论文精读报告

# Seva 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2503.14489v2（2025-04-01）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2503.14489) | [PDF](https://arxiv.org/pdf/2503.14489v2) | [Project Page](https://stable-virtual-camera.github.io/) | [Code](https://github.com/Stability-AI/stable-virtual-camera)

<!-- PAPER-METADATA END -->


> 论文：Stable Virtual Camera: Generative View Synthesis with Diffusion Models
> 作者：Jensen (Jinghao) Zhou、Hang Gao、Vikram Voleti、Aaryaman Vasishta、Chun-Han Yao、Mark Boss、Philip Torr、Christian Rupprecht、Varun Jampani。
> 版本：arXiv:2503.14489v2，2025-04-01。原始上传 ZIP arxiv_2503.14489v2.zip；PDF 页码采用文件实际顺序，附录重新编页。
> 阅读方式：Markdown-first 完整精读；PDF 为权威，Source/HTML 仅辅助；Figure 为经 B2 验收的 PDF 区域 JPEG 展示副本，并非无损原件。
> 证据：原文事实、背景知识、合理推断、独立评价、尚未证实分开。

<!-- PTH:BEGIN INIT -->
## 0. 原论文与阅读范围

【原文事实】PDF 文件共 25 页：p.1 标题、摘要和 teaser；p.2–4 §1 Introduction、§2 Background；p.4–6 §3 Method（设计与训练、采样、扩展）；p.6–12 §4 Experiments（基准、set、trajectory、长轨迹、讨论）；p.12 §5 Conclusion 与致谢；p.13–15 References；p.16 附录目录；p.17 Appendix A（影响与局限）及 B（相关工作）；p.17–20 C（Benchmark）；p.21–25 D（额外实验）。后文 p.n 始终指文件页序而不是附录印刷页码。九位作者与三个机构为首页原文记录，前两位共同贡献。

【资源证据】实际导入的 PDF 有 2503.14489v2 日期戳。PDF SHA-256：ed2fe1926f43f5a9175465f810207a4d5a4a3c631cc65d10aaaec5eec70bc8。36 个原始归档文件重新计算哈希一致。Source 与 HTML 为可选辅助；前者缺独立版本戳，后者缺部分字体资源，未用其覆盖 PDF。图像全部从同版 PDF 生成区域 PDF/JPEG，非原图替代生成，1 精确通过、17 带记录通过。Figure 8 题注目录截断已用原文补全，区域未改变。客户端网页渲染未验证。

### 图表目录
正文 Figure 1–12、Table 1–6；附录 Figure 13–18、Table 7–11；共 18 图、11 表、无独立编号 Algorithm。以下完整目录的完整题注见对应逐图逐表小节。

| 标签 | PDF 页 | 章节 | 资源状态 |
|---|---:|---|---|
| Figure 1 | 1 | Abstract / teaser | 可访问 |
| Figure 2 | 3 | §1–2 camera-control gallery | 可访问 |
| Figure 3 | 4 | §2 Background | 可访问 |
| Figure 4 | 5 | §3.1 Method | 可访问 |
| Figure 5 | 6 | §3.3 Scaling sampling | 可访问 |
| Figure 6 | 9 | §4.2–4.3 | 可访问 |
| Figure 7 | 10 | §4.3 | 可访问 |
| Figure 8 | 11 | §4.4–4.5 | 可访问 |
| Figure 9 | 11 | §4.4–4.5 | 可访问 |
| Figure 10 | 12 | §4.5 | 可访问 |
| Figure 11 | 12 | §4.5 | 可访问 |
| Figure 12 | 12 | §4.5 | 可访问 |
| Figure 13 | 21 | Appendix D.3 | 可访问 |
| Figure 14 | 21 | Appendix D.3 | 可访问 |
| Figure 15 | 22 | Appendix D.1 | 可访问 |
| Figure 16 | 23 | Appendix D.1 | 可访问 |
| Figure 17 | 24 | Appendix D.1 | 可访问 |
| Figure 18 | 25 | Appendix D.1 | 可访问 |
| Table 1 | 2 | §1 Introduction | 可访问 |
| Table 2 | 8 | §4.2 Set NVS | 可访问 |
| Table 3 | 8 | §4.2 Set NVS | 可访问 |
| Table 4 | 8 | §4.2 Set NVS | 可访问 |
| Table 5 | 10 | §4.3 | 可访问 |
| Table 6 | 10 | §4.3 | 可访问 |
| Table 7 | 18 | Appendix C | 可访问 |
| Table 8 | 19 | Appendix C/D.2 | 可访问 |
| Table 9 | 20 | Appendix C/D.2 | 可访问 |
| Table 10 | 20 | Appendix C/D.2 | 可访问 |
| Table 11 | 20 | Appendix C/D.2 | 可访问 |

本单元不涉及方法机制推导，仅确定材料范围。本单元不涉及数学分析，仅记录资源目录。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1.1 标题

Stable Virtual Camera: Generative View Synthesis with Diffusion Models，译为“稳定虚拟相机：基于扩散模型的生成式视图合成”。SEVA 为作者使用的模型简称（p.1）。Virtual Camera 是摄影制作中用于虚拟环境预览相机运动的技术；作者在 p.2 脚注说明其命名致意于该技术。这里的 stable 强调多视角与长序列生成的一致性目标，而不是物理防抖硬件，也不是某种数学稳定性定理。

Generative View Synthesis 强调给定已有视图和相机条件，合成未观测视图。Novel View Synthesis（NVS）有一个明确的反事实目标：同一场景若由另一个指定相机观察，将出现什么画面？稀疏观测下目标像素不唯一，扩散模型学习的是条件生成分布，而非必然的确定性映射。相机的位置、方向和内参是控制变量；输入照片提供观测约束；生成的 RGB 图片是直接输出。

**标题的边界**：标题本身只能表明生成式视图合成、扩散模型和虚拟相机概念，不能据此断定相机姿态是估计的、输出包含深度或可编辑网格、可处理任意真实动态场景、具有真实三维重建精度或已证明长程稳定。具体需看 §2–4：模型假设已知输入及目标相机内外参，不以显式三维结构为中间表示。标题中的 stable 是命名和目标，不是无失败保证。本单元不涉及具体数学推导，数学定义留在方法与公式单元。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 摘要逐层拆解

【原文事实，p.1】作者提出一个 generalist diffusion model SEVA，根据任意数量的已知输入视图和目标相机产生同一场景的新视图。摘要指出既有工作常难兼顾大幅视角改变和时间平滑，并受特定任务配置限制。方案由简洁模型、优化训练与灵活采样组成；结果不需要额外基于三维表示的蒸馏即可保持较高一致性，展示约半分钟视频与无缝闭环，并声称跨数据集和设置取得更优结果。

### 背景—困难—方法—结果
背景：稀疏相机采集不能保证对全场景表面形成充分覆盖。一个正面的输入既可对应背面有纹理的物体，也可对应背面无纹理的物体；两者在已观察视图完全一样。若直接回归像素，模型可能选择条件均值而形成模糊；生成模型可以选取一个合理假设。扩散的关键不是显式补出真实缺失几何，而是用数据先验生成条件下的可能画面。【背景知识】

困难是生成能力与插值平滑性之间的训练分布张力：远离输入相机意味着更多不可见区域，需要模型完成语义、结构和纹理推断；轨迹视频意味着邻接帧不能随机切换假设，需要时间连贯；而输入/目标数量变化改变网络上下文和计算预算。同一场景的两种任务若采用不同网络，使用限制和额外融合成本增加。

方法分三层而非同一模块的三种称谓。第一，保留预训练二维扩散的图像先验，在潜空间 U-Net 内增加跨视图注意力，使多视图目标在一次采样中交换信息；已知相机射线提供空间条件。第二，训练由短到长窗口，同时覆盖较大与较小帧间隔，不将生成与平滑完全隔离。第三，测试以固定训练窗口构建锚点，再将其作为跨窗口共同条件生成目标片段；在长序列中用空间近邻检索历史锚点。三者分别解决单次网络表达、训练分布、超窗推理的问题。实际机制见 §3 与 Figure 4–5。

【证据边界】摘要的“任意数量”表示任务接口与灵活采样，不是任意规模均具有可接受计算成本或可靠质量。半分钟、闭环和泛化属于展示与实验结论，而非证明所有场景的三维几何一致。Table 2–6 对应量化结果，Figure 6–12 对应视觉和经验讨论；附录 A 明确列出动态纹理、人和动物、极端路径等失败边界。摘要的优越性不能覆盖每个单元格，后文表格保留反例。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 核心贡献

作者在 §1 末尾（p.2）明确列出四项贡献，而不是由读者凭结果自行增加贡献数。

| 贡献 | 所针对矛盾与实现 | 证据与边界 |
|---|---|---|
| 训练策略 | 兼顾大视点变化和时间平滑。先 T=8，再 T=21；随机输入帧数，部分小步长采样，另训练可选时序路径 | §3.1 p.4–5；§4 的 set/trajectory 两类结果。未见对训练课程每个组件的完整单独消融 |
| 两阶段程序化采样 | 固定网络窗口难覆盖任意长轨迹；生成共享锚点后补目标，长锚点序列通过空间记忆检索降低远程漂移 | §3.2–3.3 p.5–6，Table 6，Figure 7–8；未建立显式全局几何约束 |
| 综合基准 | 先前设置分散，不同任务类型和 P 值难放在同一框架比较；汇集 10 个数据集，多种已有划分与作者划分，按视差区分难度 | §4.1、Appendix C/Table 7，Table 2–5/8–11；存在缺项、专有基线及单视图尺度扫描 |
| 开源权重 | 便于后续研究与实际使用，降低重复大规模训练门槛 | p.2 作者主张；首页官方资源链接；本次未执行下载权重或源码复现 |

【独立评价】技术实质是组合创新：预训练 SD 2.1、标准扩散、跨视图注意力、Plücker 及 CFG 等已有思想被重新组织到一个覆盖 set/trajectory 的系统；不应声称发明扩散本身或证明新的三维成像定律。其较具辨识度的部分是训练课程与视图选取面向双任务、锚点驱动的测试时任务分解，以及空间记忆对长轨迹复访的支持。模型层提供多视图联合条件生成能力，采样层使固定窗口可连接；基准是验证而非模型组成部分；开放权重是资源贡献而非新数学方法。

贡献的逻辑依赖：如果单窗输出本身严重不一致，复用锚点只会共享错误；如果训练只见连续近视图，跨大视角的锚点本身难以生成。因此双阶段并不能替代训练，也不能从架构名称推出各组件独立增益。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 结论、适用范围与局限

【原文事实，§5 p.12】SEVA 被总结为兼顾大视角变化与平滑插值、支持灵活输入目标配置的通用扩散 NVS 模型。依靠无显式三维表示的扩散架构、结构化训练和两阶段程序采样，作者报告跨任务的三维一致渲染和真实场景泛化。这里“三维一致”指图像表现和相机控制下的跨视角一致性，不表示网络直接输出可量测几何；同一错误的三维假设也可能产生内部一致的图像。

【原文事实，Appendix A p.17】作者讨论对虚拟摄影、游戏和数字遗产等创作体验的潜在价值；训练数据范围限制质量，人、动物、动态纹理（例如水表面）尤其容易退化。高度歧义的场景、复杂轨迹（例如穿过表面）、极不规则物体及远离输入的视点也会产生闪烁或其他伪影。Appendix D.3 p.21 进一步指出开放式探索中距输入很远且无内容重叠时，输出越来越饱和；作者提出对输入锚点施加不同噪声、参考 Diffusion Forcing 的思路，作为未来工作而非本方法已实现功能。

【独立评价】对本论文最合适的评价是“在已知相机条件下，用共享生成先验与跨窗条件减少多视角矛盾，扩大可合成任务范围”；不能推广为稀疏输入唯一恢复真实场景。传统三维重建和直接生成具有不同评估标准。若任务为内容制作，合理未见区域可能足够；若任务为测量、导航或科学调查，则未观测区域的幻觉会构成风险，必须另外提供几何或观测验证。原文没有验证水下介质反演、动态流体或物理传输参数恢复。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2.1 图表功能总览

18 幅图与 11 张表的作用不是简单按出现顺序排列，而是从任务展示过渡到设计机制、对照验证及适用边界。

| 对象 | 功能与阅读问题 |
|---|---|
| Figure 1–2 | 总体任务接口与自由相机路径：输入数量、控制范围如何变化？ |
| Figure 3 | 定义 set NVS 和 trajectory NVS；无序集合与有序视频为何需要不同采样？ |
| Figure 4 | 训练固定窗口、跨视图扩散架构、推理双阶段；相机如何成为生成条件？ |
| Figure 5 | 长轨迹锚点空间检索；离开当前时间邻域时如何维持复访一致？ |
| Figure 6 | set 与 trajectory 的定性基线对照，输入数与场景难度不同 |
| Figure 7–8 | 时空切片检查局部闪烁；三圈相机路径检查远程闭环 |
| Figure 9–12 | 扩展窗口、非方形图像、CFG 与随机种子；经验泛化及生成歧义 |
| Figure 13–14 | 样本与 3DGS 蒸馏后渲染的相似性；padding 与变长窗异常 |
| Figure 15–18 | 单视图、文本合成/真实场景，物体中心/场景级，相机运动与效果 |
| Table 1 | 方法类别和属性判断，非统一定量排名 |
| Table 2–4 | 小视差、大视差 set PSNR，蒸馏后 3DGS PSNR |
| Table 5–6 | trajectory PSNR；采样和可选时序路径消融（TSED、PSNR、MS） |
| Table 7 | 十个数据集及视差分组、拆分、条件与样本规模 |
| Table 8–11 | 前述设定的 LPIPS 与 SSIM 补充结果 |

**证据链**：先读 Fig.3 确定任务，然后 Fig.4–5 解释设计，继而用 Table 7 解读 Table 2–6/8–11 的协议，最后 Fig.6–14 检验质量与限制。Fig.1–2、15–18 是使用示例，不等同于受控量化对照。数值表、图像局部证据、方法机制和作者主张需相互对照，不能以一张漂亮图推导整体科学结论。本单元不涉及数学分析，数值、指标和推导在逐表及公式单元展开。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
### Figure 1 — Abstract / teaser

![Figure 1](Seva/Figures/Figure_01.jpg)

来源：PDF 文件 p.1，区域 [55.91999816894531, 244.8000030517578, 556.0800170898438, 531.1199951171875] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_01.pdf。

**Caption 原文**：Figure 1. Generative view synthesis. STABLE VIRTUAL CAMERA generates novel views from any number of input views and target cameras, which the user can specify anywhere. We show three examples: single view with simple orbit camera trajectory (top); two views with long camera trajectory (middle); and nine views with large spatial range (bottom). Please visit our website for video samples.

**题注译文**：生成式视图合成。Stable Virtual Camera 能从任意数量的输入视图以及用户可指定于任意位置的目标相机生成新视图。展示三个示例：单视图、简单环绕相机轨迹（上）；双视图、长相机轨迹（中）；九视图、大空间范围（下）。视频示例见项目网站。

**布局与观察。** 从上到下是单视图、双视图、九视图三组任务，左侧是输入照片与相机位置示意，右侧是连续输出样帧。上排 lily dragon 黄色龙形对象，视点环绕后可见正面至背侧纹理；中排红色电话亭，从两张图像沿较长路线生成不同观察角度，包含周围植被与建筑；下排房间以 x9 表示九幅输入，生成跨越窗边、椅子和书架的广范围画面。不同输入数量和目标范围共用同一模型，而不是三个不同专门网络。

**机制联系。** 仅一张图时，背后区域没有直接观测证据，模型须依赖扩散先验决定形状与纹理；两张输入能约束更多区域但不会保证后续长轨迹始终有重叠；九张图为室内更多角度提供约束。在 §3.2–3.3 中这些分别关联单窗、双阶段与更大窗口的推理需求，但题注未为每一行给出实际采样参数，因此不能确定各自使用哪个策略。相机示意表达用户提供的目标轨迹而非网络估计的轨迹。

**证据边界。** 这是一张静态拼图，可看局部外观在不同视点是否明显断裂，却不能从抽样静帧证明视频无闪烁或精确几何。没有真实目标对照、几何误差或受控输入数实验，不能把三组例图当成 P 越大越好的因果证据；Table 2–6 和 Fig.7–9 更直接验证这些问题。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
### Figure 2 — §1–2 camera-control gallery

![Figure 2](Seva/Figures/Figure_02.jpg)

来源：PDF 文件 p.3，区域 [55.91999816894531, 78.95999908447266, 556.0800170898438, 726.0] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_02.pdf。

**Caption 原文**：Figure 2. Diverse camera control. SEVA generates photorealistic novel views following diverse camera trajectories. This includes orbit, spiral, zoom out, dolly zooms, and any user-specified trajectories. Please visit our website for more visual results.

**题注译文**：多样的相机控制。SEVA 沿不同相机轨迹生成逼真的新视图，包括环绕、螺旋、拉远、滑动变焦，以及用户指定的任何轨迹。更多视觉结果见项目网站。

**视觉布局。** p.3 为多行画廊，每行以上方相机轨迹示意和输入视图、下方沿目标路径生成的帧组成；前两组为虚构/生成图像风格的物体和场景，多个路径围绕对象或改变景深感，红色框表示输入、蓝色框表示目标。第三组分别展示 FERN 的植物及室内雕塑，路径穿行或贴近前景；第四组是物体丰富的室内工作台、盘旋路径；第五组庭院的多视图输入与弯曲长路线，第六组房间多视图输入与非直线的长路径。依据 §4.3 p.8–9，第二行包含 dolly zoom，第三行 FERN 甚至靠近或穿过对象，最后一行包含未被输入覆盖的区域。

**相机机制。** 【背景知识】orbit 是相机绕目标转动；spiral 同时改变方位与高度/半径；zoom 通过焦距变化改变视场，dolly 是相机平移。滑动变焦（dolly zoom）通过相反方向的焦距与距离改变保持主体成像尺度近似不变。若主体深度为 Z、焦距 f，则近似像高正比 f/Z；选择 f 随 Z 成比例变化，可保持主体大小，而背景因深度不同出现相对缩放。实际 SEVA 使用内外参作为射线条件，而不是将输入图单纯缩放。这个近似关系是成像背景解释，不是论文另设的损失。

**控制不等于重建。** 图中不同帧对相机运动有合理响应，但没有真值姿态误差、连续帧统计或真实三维测量。作者称能沿任意指定轨迹生成，不代表穿墙和不可行相机路径必然无伪影；附录 A 恰指出与表面相交可能闪烁。多样化的例子说明任务接口和外观表达性，不能证明全部路径上的精确几何或无损渲染。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
### Figure 3 — §2 Background

![Figure 3](Seva/Figures/Figure_03.jpg)

来源：PDF 文件 p.4，区域 [314.8800048828125, 68.87999725341797, 556.0800170898438, 141.1199951171875] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_03.pdf。

**Caption 原文**：Figure 3. Set NVS versus trajectory NVS. Set NVS generates target views as an image set, whereas trajectory NVS produces them as a trajectory video.

**题注译文**：集合 NVS 与轨迹 NVS。集合 NVS 将目标视图生成为图像集合，而轨迹 NVS 将它们生成为轨迹视频。

**布局。** 左图是同一场景中散布的目标相机（蓝）与已知输入（红）；右图目标相机沿蓝色虚线排列。图例 Input/Target 区分提供的图像与要求生成的视图。区别在于目标相机的组织方式，而不在于是否属于同一场景。

**任务与机制。** §2.1 给定 $I^{inp}\in\mathbb R^{M\times H\times W\times 3}$ 与相机 $\pi^{inp}$、目标 $\pi^{tgt}$，求 $I^{tgt}\in\mathbb R^{N\times H\times W\times3}$。无序目标集合对帧的排列不应依赖时间方向（集合中的相机仍不同），因此可以基于空间关系聚类；轨迹目标 $\pi^{tgt}_1,\ldots,\pi^{tgt}_N$ 有连续相机路径顺序，相邻帧应在可见区域维持一致的外观与运动。图像集合也可能是小视差，轨迹也可能跨很大范围；不应把 set 与大视差、trajectory 与小视差定义成同义词，作者只强调通常倾向。

**方法联系。** 对集合采用 nearest 空间分配，对轨迹采用 interp 端点条件，并可开启有序时序卷积。二者都依赖相机条件，但只有后者有固定路径顺序可用。对于两个空间相近、时间索引很远的目标，纯时间邻接不是最佳空间检索，这在 Fig.5/8 进一步体现。图是任务概念示意，不包含模型性能或严格证明。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
### Figure 4 — §3.1 Method

![Figure 4](Seva/Figures/Figure_04.jpg)

来源：PDF 文件 p.5，区域 [55.91999816894531, 68.87999725341797, 556.0800170898438, 302.1600036621094] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_04.pdf。

**Caption 原文**：Figure 4. Method. SEVA is trained with fixed sequence length as a “M-in N-out” multi-view diffusion model with standard architecture. It conditions on CLIP embeddings, VAE latents of the input views, and their corresponding camera poses. During sampling, SEVA can be cast as a generative “P-in Q-out” renderer that works with variable sequence length, where P and Q need not be equal to M and N. To enhance temporal and 3D consistency across generated views, especially when generating along a trajectory, we present procedural two-pass sampling as a general strategy.

**题注译文**：方法。SEVA 以固定序列长度训练为“M 输入 N 输出”的标准架构多视图扩散模型，以输入视图的 CLIP 嵌入、VAE 潜变量及对应相机位姿为条件。采样时，SEVA 可作为可变序列长度的生成式“P 输入 Q 输出”渲染器，P 和 Q 不必等于 M 和 N。为提升生成视图之间的时序和三维一致性，特别是沿轨迹生成时，作者提出程序化两阶段采样通用策略。

**上半部：训练固定总长度。** 左侧示意已知帧与待生成帧共同构成 M-in N-out 窗口（T=M+N）；右侧画出单个多视图扩散模型的条件与残差块。图示输入图像由 VAE（Variational Autoencoder，变分自编码器）编码为 h×w 潜变量，避免直接对 H×W RGB 像素做全部去噪运算。目标帧使用加噪潜变量，输入帧为清洁潜变量；二元 mask 区分输入和目标。相机通过 Plücker embedding（普吕克射线编码）表示每像素空间直线，经过与潜变量相容的处理后连接到模型输入，并通过 Adaptive LN（自适应层归一化）注入条件。图中未明示全部投影尺寸，不能捏造具体 latent 通道或下采样实现。CLIP 图像嵌入经输入视图平均，作为 cross attention 的语义条件；它不是目标像素真值或硬几何投影。

右侧绿色模块中 2D Conv 对图像特征局部处理，2D/3D Attn 表明跨视图注意力扩展；正文 §3.1 说明将低分辨率残差块的二维 self-attention 扩为三维，并在各 self-attn 后加入沿视图轴的 1D Attn。虚线/可选 3D Conv 为 temporal pathway，作用于有顺序帧，使用额外小步长训练；并非必须对无序集合打开。模型继承 SD 2.1 潜空间 U-Net（870M 到 1.3B，可选时序后 1.5B）。图中 L× 指重复结构，不代表论文给出了可推断的全部块数。

**下半部：推理可变 P-in Q-out。** 左侧给出相机路径任务；中间 Anchor Generation 从真实输入先生成粉色锚点；右侧 Target Generation 将剩余蓝色目标分成 chunk 1–3，并复用已生成的锚点作为共同条件。路径上的锚点提供中间位置的外观假设，后一窗不必完全重新独立补全。训练时随机 M∈[1,T−1]，推理时 P、Q 可变；若 P+Q≤T，可在一个窗口补齐 T；若超过，采用两阶段或特殊大窗口/记忆策略。

【合理推断】共享条件降低两个子采样独立产生不同假设的机会，但不保证强几何一致性。此图没有显式深度、点云、体渲染模块，和训练监督中的目标潜变量不是同一概念。公式、窗口约束及帧顺序在 §3.1–3.3 展开，效果由 Table 6、Figure 7–8 检验。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
### Figure 5 — §3.3 Scaling sampling

![Figure 5](Seva/Figures/Figure_05.jpg)

来源：PDF 文件 p.6，区域 [314.8800048828125, 68.87999725341797, 556.0800170898438, 176.16000366210938] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_05.pdf。

**Caption 原文**：Figure 5. Anchor generation when Q >> T . We introduce a memory bank composed of previously-generated anchor views and their corresponding camera poses. The lookup of spatial neighbors helps improve long-term 3D consistency.

**题注译文**：当 Q 远大于 T 时的锚点生成。引入由先前已生成锚点视图及相应相机位姿组成的记忆库。查找空间邻居有助于改善长期三维一致性。

**布局。** 左半 Previous chunk 表示上一个锚点批次，红为真实 Input、黄色为 Prev. Anchor、紫色为待/已生成 Anchor；上方虚线框 Memory Bank 保留历史锚点。右半 Current chunk 中 NN 箭头从当前锚点指向空间近邻历史锚点（例如轨迹绕回场景左下或右上时）。记忆库内容随已生成锚点增加，表示存储图像和相机而不是显式网格。

**机制。** 当目标 Q 很大，第一阶段自身也超过模型窗口，不再能一次生成全部锚点。若只用上一时间段的输出作为条件，当前相机回到以前拍过的位置时却可能不再看到那些条件。作者在 §3.3 中让当前批从历史库检索空间邻居，再生成新锚点，后者放回库参与下一批。概念上可写 $\mathcal B_{k+1}=\mathcal B_k\cup\{(\hat I_a,\pi_a):a\in A_k\}$，当前条件由 $NN(\pi_a,\mathcal B_k)$ 选取。这是对原文程序的形式化说明，不指定论文未给的距离度量、索引结构、检索数量或计算复杂度。

**误差传播。** 有重叠的历史视图为当前生成提供更强外观约束，尤其相机回环时，空间距离与时间距离相反。错误锚点也可能被反复引用；并未进行全局几何优化或自动纠错，远离所有真实输入时仍有颜色和纹理漂移（Appendix D.3）。Fig.8 的三圈例证空间最近与时间最近的区别，不足以证明所有拓扑路径的闭环精度。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
### Figure 6 — §4.2–4.3

![Figure 6](Seva/Figures/Figure_06.jpg)

来源：PDF 文件 p.9，区域 [55.91999816894531, 68.87999725341797, 556.0800170898438, 519.1199951171875] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_06.pdf。

**Caption 原文**：Figure 6. SOTA comparison on set NVS (top) and trajectory NVS (bottom) across varying numbers of input views. We compare with open-source approaches—ViewCrafter [9] (VC) and DepthSplat [45] (DS)—as well as proprietary ones including LVSM [11], LongLRM [18] (LLRM), 4DiM [12], and CAT3D [8]. When the input comprises multiple views, we arrange them so that the view closest to the target is placed at the top of each set.

**题注译文**：集合 NVS（上）和轨迹 NVS（下）在不同输入数量下的最先进方法对比。与开源 ViewCrafter（VC）、DepthSplat（DS），以及专有 LVSM、Long-LRM（LLRM）、4DiM、CAT3D 比较。输入有多视图时，将最靠近目标的视图置于每组顶部。

**上半部（set）**：两组四列 Input / Target / Ours / Baseline 各有四行。首行建筑及室外物体/雕塑，与 VC 对照；次行室内画面与 LVSM 对照；第三行建筑立面和街边画面与 DS 对照；第四行 x32 输入的室内床铺与家具，对照 LLRM。§4.2 指出小视差中 SEVA 的纹理往往更锐利，例如与 LVSM 对照 RE10K；大视差的 DL3DV 样例中 DS 出现明显重建失真，SEVA 给出更完整外观。32 输入时差异小于稀疏例图，不代表该配置总体指标超过 Long-LRM。最近输入显示在上方，目标及其他输入不应被误认为模型输出。

**下半部（trajectory）**：三组 Input / Ours / Baseline 列，包含建筑塔楼、室内、汽车、花卉、庭院、室内工作台等多种样例，与 4DiM、VC、CAT3D 对照。可看到 SEVA 的远处结构或表面纹理在一些案例较稳定，而基线部分输出模糊或形状失真。轨迹只展示抽样关键帧，不能单凭此图量化闪烁；Fig.7 用相同路径的垂直切片更直接观察局部不连续。

**证据界限。** set 上半含 Target 可作同一目标相机下真实图像的局部对照；trajectory 下半不是逐帧 GT 表格。基线来源与实验设定由 Table 2/3/5 和 Appendix C 解释，存在专有模型、作者复现以及已有报告等差异。相机归一化 P=1 会影响 SEVA 像素对齐，视觉锐利并不一定意味着更高 PSNR。此图支持样例中的可见差异，不证明数据集均值全面领先、所有场景真实几何正确或任意 baseline 的失败原因。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
### Figure 7 — §4.3

![Figure 7](Seva/Figures/Figure_07.jpg)

来源：PDF 文件 p.10，区域 [314.8800048828125, 68.87999725341797, 556.0800170898438, 221.0399932861328] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_07.pdf。

**Caption 原文**：Figure 7. Temporal quality. Vertical slices of a rendered novel camera path on the BONSAI scene from Mip-NeRF360 [42] illustrate the temporal quality across adjacent viewpoints. One-pass or gt + nearest procedural sampling results in notable flickering, whereas interp procedural sampling ensures temporally smooth rendering.

**题注译文**：时序质量。在 Mip-NeRF360 的 BONSAI 场景中，沿新相机路径渲染后的垂直切片显示相邻视点之间的时间质量。单阶段或 gt + nearest 程序化采样产生明显闪烁，而 interp 程序化采样使渲染在时间上平滑。

**如何阅读时空切片。** 三行依次为 One-pass、Two-pass: gt+nearest、Two-pass: interp，横轴 Time，图像中的每一竖列来自相应帧固定横向位置的垂直像素切片（具体像素索引原文未给）。在平滑运动下，同一结构的时空纹理应随时间连续移动；突变形成突然断开的带状线条，细节亮暗反复改变形成闪烁。上排红框标出 discontinuity 和 flicker；中排仍有局部 flicker；底排相同区域纹理更加连续。这比单张关键帧更直接揭示相邻帧不一致。

**采样因果链。** one-pass 分段时每一窗口可生成不同的局部假设；gt+nearest 复用空间近邻和真实输入，但最近锚点的分配在轨迹上可能突变，不能主动提供连续区间两端。interp 以相邻两个锚点为共同边界，生成中间有序视图，片段交界处复用一个端点，因此能改善相邻帧的条件连续性。原文对照中这一方法有更高 VBench motion smoothness（Table 6：95.76，对 one-pass 95.39 和 gt+nearest 95.22）。

【证据边界】时空切片位置有限，看不到画面所有区域；空间颜色连续也可能源于过度模糊，而非几何正确。表中 PSNR 及 3DGS 蒸馏后 PSNR、TSED 提供互补指标，不能以切片外观独立证明全局三维重建。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
### Figure 8 — §4.4–4.5

![Figure 8](Seva/Figures/Figure_08.jpg)

来源：PDF 文件 p.11，区域 [55.91999816894531, 68.87999725341797, 297.1199951171875, 235.1999969482422] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_08.pdf。

**Caption 原文**：Figure 8. Long-range 3D consistency. We visualize samples following a camera path looping three times around the TELEPHONEBOOTH scene. Lookup using spatial neighbors from the memory bank (ours) notably improves view consistency and reduces artifacts in recurring locations across different loops, compared to lookup using temporal neighbors (baseline).

**题注译文**：长期三维一致性。可视化沿 TELEPHONE-BOOTH 场景环绕三次的相机路径生成的样本。相比检索时间邻居（基线），从记忆库检索空间邻居（本文）显著改善重复访问位置的视图一致性并减少伪影。

**视觉布局。** 上方 Input 为两张真实红色电话亭图和三圈路径；图中标注 111th/496th 与 242nd/582nd/838th 等不同时刻的相近空间位置。下方 Baseline 和 Ours 分两组复访画面：左组 Loop 1/2 的电话亭远景、建筑和植被，基线红框标出背景建筑变化；右组 Loop 1/2/3 的近景，基线在电话亭旁植被等区域出现不一致，本文行较稳定。原文 §4.4 p.10–11 描述最长约 1000 帧的演示。

**为什么空间检索适合闭环。** 若相机位姿沿路径 $\pi(t)$ 变化，闭环意味着存在 $t_i\ne t_j$ 而 $\pi(t_i)\approx\pi(t_j)$。例如第一圈第 242 帧与后续圈约第 582、838 帧虽时间差很大，却看向相似区域。按时间邻近只保留最近输出时，旧位置外观条件已退出上下文；按空间最近邻可直接找回该区域曾生成的锚点，与当前帧共同生成。这相当于图像—位姿条件记忆，而不是融合点云。它能降低再次生成相同位置时随机采样到另一个场景假设的风险。

【独立评价】闭环一致性是相同/近似相机观察结果的一致，不等于真实场景正确。如果第一圈有幻觉，空间记忆可以稳定复现这个错误；也没有回环检测的独立相机误差统计、全局优化或全部帧的数值对比。图示只支持这一案例的视觉改善，不宣称任意 1000 帧视频都无缝。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
### Figure 9 — §4.4–4.5

![Figure 9](Seva/Figures/Figure_09.jpg)

来源：PDF 文件 p.11，区域 [314.8800048828125, 68.87999725341797, 556.0800170898438, 249.1199951171875] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_09.pdf。

**Caption 原文**：Figure 9. Generation quality on the number of input views. PSNR↑(top) and Image Quality↑(bottom) on set NVS. Results are reported on our split of T&T. Extending T to more input views in a zero-shot manner produces more consistent samples in the semi-dense-view regime. Dense 3DGS denotes results of [3] with full views.

**题注译文**：输入视图数量对生成质量的影响。在 T&T 的本文划分上报告集合 NVS 的 PSNR（越高越好，上）及 Image Quality（越高越好，下）。零样本扩展 T 来容纳更多输入，在半稠密视图区间生成更一致的样本。Dense 3DGS 表示使用全部视图的 3DGS 结果。

**读图。** 左侧上图横轴 P（1、3、6、9、16、32）纵轴 PSNR(dB)，下图同横轴、纵轴 Image Quality；P≤8 为稀疏，P≥9 为半稠密。蓝色三条曲线对应两阶段 nearest-6/9/16，橙色 one-pass (T=P+Q)；绿色 Dense 3DGS 水平参考分别 23.7 与 78.4。点值没有原文数字表，不把像素读数冒充精确数值。上图大 P 时 one-pass 上升到约 16–17 dB 区间，近邻较少的曲线饱和/下降；下图 P<9 时扩窗 one-pass 的图像质量明显较低，输入增多后接近/超过部分蓝线，仍低于稠密参考。右侧 P=3 和 P=32 两列，逐行 Input、Groundtruth、two-pass (nearest-16)、one-pass；车辆正脸在 P=3 的 one-pass 明显模糊，在 P=32 的 one-pass 结构更清晰且更接近目标。

**为何出现两种制度。** nearest-K 在第一阶段只选有限邻居，P 增大但选入集合不一定扩大，新增观测可能不被利用，跨窗的锚点质量也受生成先验限制。扩展 T 的 one-pass 将所有输入与目标共同放入注意力范围；当覆盖大部分场景时，任务转为有较强观测约束的插值，额外视图有价值。稀疏输入时仍要生成大量不可见区域，而长于训练窗口的注意力分布存在外推问题，作者观察到过度模糊；附录 D.3 仅提出注意力分数变化的假说，未证明机制唯一。

**证据边界。** 这是 T&T 一个划分的输入规模实验，能支持该分布上 P 与策略的交互，不表明 P=9 是任何数据集的物理阈值。Dense 3DGS 使用全部视图、监督条件更强，仅作参照；与它的差距不能当同等观测条件的公平比较。作者称可扩到数百帧是经验观察，定量图只到 P=32。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN FIG-10 -->
### Figure 10 — §4.5

![Figure 10](Seva/Figures/Figure_10.jpg)

来源：PDF 文件 p.12，区域 [55.91999816894531, 68.87999725341797, 297.1199951171875, 315.1199951171875] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_10.pdf。

**Caption 原文**：Figure 10. Generation quality on different image resolutions. Our model generalizes to different image resolution of varying aspect ratios, including both portrait (top) and landscape orientations (bottom). Results are presented as a pair of the input view and the target views.

**题注译文**：不同图像分辨率下的生成质量。模型可泛化至具有不同长宽比的图像分辨率，包括纵向（上）与横向（下）。结果以输入视图和目标视图的配对形式展示。

**视觉区域。** 顶部两组纵向图像分别呈现黑金色虚构人物/物体及户外桌椅；每组输入和目标视图可看见主体轮廓、桌面和背景位置变化。底部两行横向图像是建筑庭院和室内房间，以输入—目标配对展示，视野扩展到新的建筑立面或墙面。不是将同一张图做普通裁剪，而是给定目标相机产生相应视角的 RGB 结果。

**训练与外推。** §3.1 明确训练 H=W=576。潜空间卷积与注意力可以在不同空间网格上运算，因而理论接口不一定锁死方形输入。相机射线条件仍应匹配目标像素网格与内参；否则改变长宽比会同时改变视场含义。论文没有给出每个案例的精确 H、W、内参处理与速度，也没有全面分辨率曲线。§4.5 的 portrait (16:9)、landscape (9:16) 与常用 W:H 书写顺序相反，可能采用 H:W；原文未定义比值约定，不能据此断言颠倒了图像方向。

【证据边界】定性配对说明方形训练不妨碍部分非方形样本生成；不证明任何分辨率、视场或超高像素数量均可。高分辨率可能使注意力成本上升。此处不对未知输出尺寸作推测。

<!-- PTH:END FIG-10 -->

<!-- PTH:BEGIN FIG-11 -->
### Figure 11 — §4.5

![Figure 11](Seva/Figures/Figure_11.jpg)

来源：PDF 文件 p.12，区域 [55.91999816894531, 384.0, 297.1199951171875, 515.0399780273438] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_11.pdf。

**Caption 原文**：Figure 11. Generation uncertainty on CFG. The CFG scale should be increased as generation uncertainty rises. For singleview conditioning (top), a higher CFG scale is typically required, whereas few-view conditioning (bottom) benefits from a lower scale.

**题注译文**：生成不确定性与 CFG。随着生成不确定性增加，CFG 尺度应增大。单视图条件（上）通常需要较高 CFG，少量多视图条件（下）则受益于较低尺度。

**布局与观察。** 两行同类室内厨房场景，左边输入及相机示意，右方 CFG=2/3/5/8。单输入上排不确定性较大，2 和 3 的红框区域模糊，5 视觉效果较佳，8 局部过饱和。多输入下排观测约束增强，较低 CFG 已能给出清晰画面，5、8 有过饱和标记。底部从 Uncertain 到 Certain 的颜色条表达作者定性概念，不是测量过的概率校准轴。

**背景数学。** Classifier-free guidance（CFG，无分类器引导）常以条件与无条件预测混合，例如噪声参数化下 $\hat\epsilon_s=\epsilon_u+s(\epsilon_c-\epsilon_u)$。当 s=1 为条件预测，s>1 将条件方向外推；实际论文未披露该实现的参数化和哪些条件被置空，因此此式仅说明常见机制。s 太低时强先验不足以形成清晰模式；太高时超出条件分布的自然范围，容易对比度、饱和度失真。多视图本身已经提供较强信息，不需要同样大的外推幅度。不能从这两个样例推出不确定性和最优 s 的精确函数。

【原文事实】§4.5 p.11–12 经验建议 CFG 2–5，单视图高不确定性示例约 5，少量多视图约 3。该参数影响外观与一致性，但模型未输出可信的 per-pixel uncertainty map，也未自动估计最优 CFG。人工设置和场景歧义、观测覆盖有关，而不是仅由 P 唯一决定。

<!-- PTH:END FIG-11 -->

<!-- PTH:BEGIN FIG-12 -->
### Figure 12 — §4.5

![Figure 12](Seva/Figures/Figure_12.jpg)

来源：PDF 文件 p.12，区域 [314.8800048828125, 68.87999725341797, 556.0800170898438, 209.0399932861328] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_12.pdf。

**Caption 原文**：Figure 12. Generation diversity in unseen regions. Our model generates diverse samples by varying randomization seeds during the sampling process.

**题注译文**：未见区域的生成多样性。模型通过在采样过程中改变随机种子生成多样的样本。

**布局。** 两行、四列 Input/Seed 1/Seed 2/Seed 3。第一行正面古典雕像输入，图示相机转到背面后，三个结果在姿态轮廓、衣褶/背面纹理及背景上不同，却仍像同一类别对象；第二行输入为秋季风景道路，三个新视图出现不同道路延伸、房屋与树木布置。共享输入与目标相机，改变初始随机种子得到不同合理结果（§4.5 p.12）。这些差异集中在不可见区域，并不意味着每个已观察像素一定完全不变。

**为什么生成可多解。** 设两个可能场景 $S_1,S_2$ 在输入相机下投影相同，但在新相机下不同；输入数据不能排除其中任意一个。回归点估计倾向给出某种平均/单一解，而条件生成 $I^{tgt}\sim p_\theta(\cdot\mid I^{inp},\pi^{inp},\pi^{tgt})$ 能抽取不同模式。随机种子是改变采样轨迹的途径；训练数据和相机条件仍限制可产生的解。

**合理不等于真实。** 图证明至少所展示样本有多样性，不证明生成分布与真实后验匹配，不提供多样性统计、置信区间或真实背面。多种 plausible predictions 对虚拟摄影有利，但若用于测量、科研重建，必须明确未观测区域是假设而非观测。

<!-- PTH:END FIG-12 -->

<!-- PTH:BEGIN FIG-13 -->
### Figure 13 — Appendix D.3

![Figure 13](Seva/Figures/Figure_13.jpg)

来源：PDF 文件 p.21，区域 [55.91999816894531, 68.87999725341797, 297.1199951171875, 208.0800018310547] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_13.pdf。

**Caption 原文**：Figure 13. 3DGS versus samples. The model generates consistent renderings that closely resemble those from 3DGS [3], with minimal perceptual differences.

**题注译文**：3DGS 与样本。模型生成一致的渲染结果，它们与 3DGS 渲染非常相似，感知差异很小。

**布局。** 两行 DL3DV 的 O 划分案例，四列 Input（叠放多张真实输入）、Target（真实测试图）、Ours (sample)（扩散直接采样）、Ours (3DGS)（从输入和生成图像蒸馏到高斯表示后渲染）。首行现代建筑和树木：右侧建筑部位生成结果与 Target 存在合理幻觉，作者在 Appendix D.3 明确指出；蒸馏后仍保持近似清晰外观。次行温室/植物建筑，玻璃、屋顶和植被结构在两种本文结果中接近。

**为何是间接一致性证据。** 一个持久化 3DGS 表示必须共享一套空间高斯参数以同时解释多个视图；若扩散样本严重互相矛盾，拟合可能在颜色/几何间折中，产生模糊或伪影。直接采样与蒸馏渲染接近，说明生成样本能较好地被某一三维表示共同解释。实际 §4.2 使用 720 生成视图、真实输入、3DGS-MCMC（无点云初始化）、优化相机及 LPIPS 损失；不是 SEVA 正常推理的必需步骤。

**证据边界。** 感知差异小不等于真实几何精确：高斯可以用视角依赖外观和调整后的相机拟合错误但一致的幻觉。图没有深度/点云真值，也没有报告每个样本的几何误差。Table 4/10 和 6 给出相关蒸馏后图像指标，能补充但非直接测量深度。

<!-- PTH:END FIG-13 -->

<!-- PTH:BEGIN FIG-14 -->
### Figure 14 — Appendix D.3

![Figure 14](Seva/Figures/Figure_14.jpg)

来源：PDF 文件 p.21，区域 [314.8800048828125, 68.87999725341797, 556.0800170898438, 208.0800018310547] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_14.pdf。

**Caption 原文**：Figure 14. Padding. Padding the last elements within one forward reduces artifacts compared to changing T.

**题注译文**：填充。在单次前向中填充最后元素相比改变 T 可以减少伪影。

**布局。** 两行分别为红花及盆栽叶片，四列 Input、Target、Padding: none、Padding: last。无填充第三列可见颜色溢出/斑点与不正常局部纹理，红花背景尤其明显；填充最后元素的第四列整体更接近目标外观。Appendix D.3 p.21 说明这是 $P+Q<T$ 时，保持训练窗口长度与直接将 T 改为 P+Q 的比较。

**机制与假说。** 训练总长度固定，少 token/帧可能使同一注意力模块在训练外的上下文长度下归一化分布变化。附录作者假设较小 T 使注意力分布更尖锐、较大 T 使其更分散，分别与颜色溢出和稀疏扩窗模糊相关。这不是严格数学定理：softmax 权重还取决于每个 logit，新增重复帧可能提高该视图总注意力质量，并非单纯无影响填零。padding 可以保持长度却改变视图权重。

**措辞核对。** §3.2 p.6 明确说重复第一张输入图以填至 T；Figure 14 题注与列标签写 padding the last elements / Padding: last。两处措辞不同，不能凭此断定实现重复的是同一张首帧或末帧，或将位置与内容强行等同。本报告保留原文表述，未读取源码实现验证。作者建议训练时动态变化 T 可能改善分布外长度，尚未实证。

<!-- PTH:END FIG-14 -->

<!-- PTH:BEGIN FIG-15 -->
### Figure 15 — Appendix D.1

![Figure 15](Seva/Figures/Figure_15.jpg)

来源：PDF 文件 p.22，区域 [64.80000305175781, 68.87999725341797, 536.1599731445312, 704.1599731445312] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_15.pdf。

**Caption 原文**：Figure 15. Diverse camera motions and effects. Single-view conditioning with a text-prompted object-centric scene. The image is generated using SD 3.5 [73] with the text prompt, “A cute firefly dragon in its natural habitat.”

**题注译文**：多样的相机运动与效果。文本提示生成的物体中心场景，单视图条件。图像由 SD 3.5 依据提示词“A cute firefly dragon in its natural habitat.” 生成，即“一只处于自然栖息地中的可爱萤火龙”。

**布局与运动。** 顶部唯一输入是黄橙色小龙。左侧每行是相机轨迹图，右侧是同一路径的生成帧。根据实际 PDF 图像放大核对，从上到下 12 行分别为 lookat 360（朝向主体环绕）、spiral（螺旋）、fronto-parallel 360（正面平行方向的环行）、panorama 360（全景运动）、tilted forward（倾斜前移）、tilted backward（倾斜后移）、zoomin（拉近视场）、zoomout（拉远视场）、dolly zoomout（平移与焦距联动）、wander 360（绕行）、up（向上）、down（向下）。PDF 文本抽取顺序与视觉顺序不一致且重复出现 tilted backward，以视觉标签为准；作者未给出各路径精确位姿与内参。横向生成帧体现相机路径与外观变化的对应，不表示时间中的物体真实运动。

**逐类观察。** 首三行环绕/螺旋/正面平行环行看到龙的侧面、背面和翅膀，不同运动方向改变遮挡关系；第四行全景运动展示较多环境；第五、六行前后倾斜改变主体尺度和地面/背景透视；第七、八行变焦显著改变画面中龙的大小和上下文范围；第九行滑动变焦的主体相对尺度与背景改变；第十行绕行改变环境观察方向；末两行垂直移动使头部与地面相对构图变化。所有帧依赖同一张单视图，文本只用于预先产生输入图，原文没有把该文本作为 SEVA 的必要提示条件。

**技术边界。** SD 3.5 合成的图像本来就没有一套提供给模型的真实三维对象。未观测的龙背部、环境属于条件推断；多角度合理是外观证据，不是相对真实三维模型的误差。此类输入有助于展示创造性外推而不能验证对现实物体的尺度精度。

<!-- PTH:END FIG-15 -->

<!-- PTH:BEGIN FIG-16 -->
### Figure 16 — Appendix D.1

![Figure 16](Seva/Figures/Figure_16.jpg)

来源：PDF 文件 p.23，区域 [60.959999084472656, 68.87999725341797, 532.0800170898438, 709.2000122070312] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_16.pdf。

**Caption 原文**：Figure 16. Diverse camera motions and effects. Single-view conditioning with a text-prompted scene. The image is generated using SD 3.5 [73] with the text prompt, “Wide view of the interior of the famed Library of Alexandria, elegantly set behind a time-worn wreckage by a lake, hinting at the relentless passage of time. The surroundings are lit by the light of a late afternoon sun, gently cast, immersing the area in a sentimental luminescence. ”

**题注译文**：多样的相机运动与效果。文本提示生成的场景，单视图条件。图像由 SD 3.5 生成，提示词为：“著名亚历山大图书馆内部的宽广视图，优雅地坐落在湖畔饱经时间磨损的残骸之后，暗示时间无情地流逝。周围环境由柔和的午后阳光照亮，浸润于感伤的光辉之中。”

**布局与内容。** 顶部是 Library of Alexandria 风格的虚构图书馆内部；左侧每行是相机轨迹图，右侧是同一路径的生成帧。根据实际 PDF 图像放大核对，从上到下 12 行分别为 lookat 360（朝向主体环绕）、spiral（螺旋）、fronto-parallel 360（正面平行方向的环行）、panorama 360（全景运动）、tilted forward（倾斜前移）、tilted backward（倾斜后移）、zoomin（拉近视场）、zoomout（拉远视场）、dolly zoomout（平移与焦距联动）、wander 360（绕行）、up（向上）、down（向下）。PDF 文本抽取顺序与视觉顺序不一致且重复出现 tilted backward，以视觉标签为准；作者未给出各路径精确位姿与内参。横向生成帧体现相机路径与外观变化的对应，不表示时间中的物体真实运动。

**逐区域解读。** 环绕、螺旋与正面平行环行的前三行显示木质书架、拱窗、地面和水/光影风格区域从不同方向被观察；第四行全景运动更强调场景全景，书架在画面中的相对位置改变。第五、六行前后移动，近远物体发生不同透视变化；第七、八行视野收放表现室内结构扩展/收缩，第九行滑动变焦考察相对尺度，第十行绕行和末两行垂直移动使高处结构与地面进入或退出视野。原 PDF 该页可见一个额外命令式文字 “--visual_test_endpoint True”，不是本文新增算法公式或需要执行的命令；本报告仅保留图像，不解释为论文参数证据。

**证据边界。** 与 Fig.15 相同是单视图文本生成图像，但任务从单物体转为场景整体，意味着未观测结构更复杂、遮挡更广。书架背后和远景由模型推断；未给真实目标、重建几何或准确率。文本生成图不等于动态真实场景测试。

<!-- PTH:END FIG-16 -->

<!-- PTH:BEGIN FIG-17 -->
### Figure 17 — Appendix D.1

![Figure 17](Seva/Figures/Figure_17.jpg)

来源：PDF 文件 p.24，区域 [60.959999084472656, 68.87999725341797, 532.0800170898438, 709.2000122070312] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_17.pdf。

**Caption 原文**：Figure 17. Diverse camera motions and effects. Single-view conditioning with a real-life object-centric scene.

**题注译文**：多样的相机运动与效果。真实物体中心场景，单视图条件。

**布局。** 顶部唯一真实输入为一只棕黑色包，背景有货架/展示环境。左侧每行是相机轨迹图，右侧是同一路径的生成帧。根据实际 PDF 图像放大核对，从上到下 12 行分别为 lookat 360（朝向主体环绕）、spiral（螺旋）、fronto-parallel 360（正面平行方向的环行）、panorama 360（全景运动）、tilted forward（倾斜前移）、tilted backward（倾斜后移）、zoomin（拉近视场）、zoomout（拉远视场）、dolly zoomout（平移与焦距联动）、wander 360（绕行）、up（向上）、down（向下）。PDF 文本抽取顺序与视觉顺序不一致且重复出现 tilted backward，以视觉标签为准；作者未给出各路径精确位姿与内参。横向生成帧体现相机路径与外观变化的对应，不表示时间中的物体真实运动。

**视觉证据。** 环绕、螺旋和正面平行环行三行可看到提手、侧面和背面方向的变化，环境遮挡随角度改变；第四行全景运动扩大周围场景视野；前移后移两行在相近正面方向显示主体尺度变化；变焦两行突出包面图案与背景的视场改变；滑动变焦同时改变透视与尺度；绕行与上下运动的末三行更明显改变货架、桌面、包的上部和下部相对位置。原图展示多个生成帧而非同一输入的真实多视图采集。

**机制与边界。** 真实物体的输入相比 SD 合成图更贴近真实成像分布，但单视图对包后部仍存在遮挡，不能确定实际背部标志、材料纹理和环境。模型由预训练语义和跨视图扩散先验补全这些信息。实验没有给出真实多视角目标、严格几何或视频测量，不证明任意真实物体的商品级精确复现；Appendix A 的不规则物体失败仍适用。

<!-- PTH:END FIG-17 -->

<!-- PTH:BEGIN FIG-18 -->
### Figure 18 — Appendix D.1

![Figure 18](Seva/Figures/Figure_18.jpg)

来源：PDF 文件 p.25，区域 [60.959999084472656, 68.87999725341797, 532.0800170898438, 709.2000122070312] pt（左上坐标系）；B2 已验收的 JPEG 展示副本，原始区域 PDF：Figures_PDF/Figure_18.pdf。

**Caption 原文**：Figure 18. Diverse camera motions and effects. Single-view conditioning with a real-life scene.

**题注译文**：多样的相机运动与效果。真实场景，单视图条件。

**布局。** 顶部是一张真实室内起居空间图，含桌椅、沙发、吊顶和大窗。左侧每行是相机轨迹图，右侧是同一路径的生成帧。根据实际 PDF 图像放大核对，从上到下 12 行分别为 lookat 360（朝向主体环绕）、spiral（螺旋）、fronto-parallel 360（正面平行方向的环行）、panorama 360（全景运动）、tilted forward（倾斜前移）、tilted backward（倾斜后移）、zoomin（拉近视场）、zoomout（拉远视场）、dolly zoomout（平移与焦距联动）、wander 360（绕行）、up（向上）、down（向下）。PDF 文本抽取顺序与视觉顺序不一致且重复出现 tilted backward，以视觉标签为准；作者未给出各路径精确位姿与内参。横向生成帧体现相机路径与外观变化的对应，不表示时间中的物体真实运动。

**视觉区域。** 前三行环绕/螺旋/正面平行环行表现家具侧面、天花板与空间深处变化；第四行全景运动中窗、室内柱面和墙面交替进入视野。第五、六行前后倾斜移动使近处家具与远处窗景相对投影发生变化；第七、八行收放视野决定家具占据画面的比例；第九行滑动变焦显示主体和背景的相对尺度变化；第十行绕行与末两行垂直移动强调室内上部、地面及家具的遮挡关系。这比单物体例子需要对整个房间的不可见部分做补全。

**证据边界。** 真实场景的单视图只有可见表面的 RGB 约束，空间拓扑、真实尺度、其他房间布局未必可识别。图支持在选定真实输入上的可控生成，而不是场景重建质量的统计验证。没有真实路径配对数据、连续帧评估或失败率，不应据此否定 Appendix A/D.3 指出的复杂路径和长期饱和问题。

<!-- PTH:END FIG-18 -->

<!-- PTH:BEGIN TABLE-01 -->
### Table 1 — §1 Introduction

PDF p.2。

**Caption 原文（PDF 文本层，图标另释）**：Table 1. Comparison of existing NVS models based on the source of training data and key attributes. SEVA is trained on both object-level (ò) and scene-level (U) data, offering flexible input conditioning, strong generation capacity, and smooth view interpolation. We define generation capacity and interpolation smoothness of each work based on their evaluation setting and our benchmark results. ∗This upper-bound can be up to hundreds for dense captures, we test our model up to 32 views in practice.

**题注译文**：根据训练数据来源和关键属性比较既有 NVS 模型。SEVA 同时使用物体级（立方体图标）及场景级（山形图标）数据训练，支持灵活输入条件、强生成能力和平滑视图插值。作者根据各方法评估设置及本文基准结果定义生成能力、插值平滑性。* 对稠密捕获可扩至数百，实际测试输入最多 32。

| 类别 | 模型 | 训练数据 | 生成能力 | 插值平滑性 | 输入灵活性 |
|---|---|---|---|---|---|
| 回归 | pixelNeRF | 物体 | ✗ | ✓ | sparse (1) |
| 回归 | pixelSplat | 场景 | ✗ | ✓ | sparse (2) |
| 回归 | MVSplat | 场景 | ✗ | ✓ | sparse (2) |
| 回归 | Long-LRM | 场景 | ✗ | ✓ | semi-dense ({16,32}) |
| 回归 | LVSM | 物体/场景 | ✗ | ✓ | sparse ({2,4}) |
| 图像扩散 | Zero123 | 物体 | ✓ | ✗ | sparse (1) |
| 图像扩散 | ZeroNVS | 物体、场景 | ✓ | ✗ | sparse (1) |
| 图像扩散 | ReconFusion | 物体、场景 | ✓ | ✗ | sparse (3) |
| 图像扩散 | CAT3D | 物体、场景 | ✓ | ✗ | sparse ([1,9]) |
| 视频扩散 | SV3D | 物体 | ✗ | ✓ | sparse (1) |
| 视频扩散 | MotionCtrl | 场景 | ✗ | ✓ | sparse (1) |
| 视频扩散 | ViewCrafter | 场景 | ✗ | ✓ | sparse (2) |
| 视频扩散 | 4DiM | 场景 | ✓ | ✓ | sparse ({1,2,8}) |
| 本文 | SEVA | 物体、场景 | ✓ | ✓ | sparse ([1,8]), semi-dense ([9,32*]) |

**机制及含义。** 回归路线用确定性预测或显式场景渲染，稳定但缺失区域常不具备足够生成自由度；图像扩散学习无序集合，善于补全但跨独立采样窗口可能不连续；视频扩散学习相邻帧时序，善于连续变化但训练视频通常不覆盖很大相机跳变。4DiM 同时有两项勾选，因此 SEVA 并非唯一兼顾两者，作者强调的是灵活输入与跨任务组合。

**证据边界。** 勾叉是作者依据评估设定与自有基准做的定性分类，不是严格二值能力或客观任务的充分必要条件。例如一个显式表示回归模型仍可能通过训练先验推断未见结构，不能据叉号宣称完全不能生成。P 的范围是作者表中测试/设定，而非理论支持上界。无统一单位或公式，本单元不涉及数学推导，分类的定量补充见后续表。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
### Table 2 — §4.2 Set NVS

PDF p.8。

**Caption 原文（PDF 文本层，图标另释）**：Table 2. PSNR↑on small-viewpoint set NVS. P denotes the number of input views. For all results with P = 1, we sweep the unit length for camera normalization due to the model’s scale ambiguity. Oe and Oh denote the easy and hard split of our split, respectively. Underlined numbers are run by us using the offical released code.

**题注译文**：小视差集合 NVS 的 PSNR（越高越好）。P 为输入视图数量。所有 P=1 的结果中，因模型存在尺度歧义，本文扫描相机归一化单位长度。Oe/Oh 表示本文易/难划分。下划线数值由作者用官方代码运行。

| 数据集/划分/P | Long-LRM | MVSplat | DepthSplat | LVSM | MotionCtrl | 4DiM | ViewCrafter | SEVA |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| OO3D/O/3 | - | 14.78† | 15.67† | - | - | - | 14.64† | 30.30 |
| GSO/O/3 | - | 15.21† | 16.52† | - | - | - | 15.93† | 31.53 |
| RE10K/D/1 | - | 20.42† | 20.90† | - | 12.74 | 17.08 | 20.43† | 17.99 |
| RE10K/V/1 | - | 20.32† | 19.24† | - | 16.29 | - | 22.04 | 18.56 |
| RE10K/P/2 | - | 26.39 | 27.44 | 29.67 | - | - | 21.42† | 25.66 |
| RE10K/R/1 | - | 21.56† | 21.87† | - | - | - | 20.88† | 18.11 |
| RE10K/R/3 | - | 25.64† | 22.54† | - | - | - | 22.81† | 27.57 |
| LLFF/R/1 | - | 11.23† | 12.07† | - | - | - | 10.53† | 14.03 |
| LLFF/R/3 | - | 12.50† | 12.62† | - | - | - | 13.52† | 19.48 |
| DTU/R/1 | - | 13.87† | 14.15† | - | - | - | 12.66† | 14.47 |
| DTU/R/3 | - | 15.52† | 16.24† | - | - | - | 16.40† | 20.82 |
| CO3D/V/1 | - | 12.52† | 13.23† | - | 15.46 | - | 18.96 | 18.40 |
| CO3D/R/3 | - | 13.52† | 13.77† | - | - | - | 14.72† | 19.25 |
| WRGBD/Oe/3 | - | 14.56† | 15.93† | - | - | - | 16.42† | 19.75 |
| WRGBD/Oh/6 | - | 12.54† | 14.23† | - | - | - | 12.66† | 18.91 |
| Mip360/R/6 | - | 13.56† | 14.01† | - | - | - | 14.59† | 16.70 |
| DL3DV/O/6 | - | 14.34† | 15.72† | - | - | - | 13.78† | 17.80 |
| DL3DV/L/32 | 23.86 | 16.24† | 16.78† | - | - | - | - | 20.96 |
| T&T/V/1 | - | 13.22† | 14.35† | - | 13.29 | - | 18.07 | 15.16 |
| T&T/L/32 | 18.20 | 12.63† | 13.12† | - | - | - | - | 16.50 |

以上转置保留全部 20 设置；† 对应原表下划线，—/− 为未报告（表中保留 -），不是零分。D/V/P/R/L/S/O 划分见 Table 7，PSNR 单位 dB。

**关键比较。** OO3D 30.30 vs 15.67（DepthSplat）差 14.63，GSO 31.53 vs 16.52 差 15.01；LLFF R/3 19.48 vs 13.52（ViewCrafter）差 5.96，说明稀疏多视图下优势大。RE10K/R/3 27.57 vs MVSplat 25.64 差 1.93；P/2 25.66 vs ViewCrafter 21.42 差 4.24，但低于 MVSplat 26.39、DepthSplat 27.44、LVSM 29.67。DL3DV/L/32 20.96 vs Long-LRM 23.86，落后 2.90；T&T/L/32 16.50 vs 18.20，落后 1.70。RE10K D/V/R 的 P=1 分别 17.99/18.56/18.11，低于相应最优基线 20.90/22.04/21.87；CO3D/V/1 和 T&T/V/1 也低于 ViewCrafter。DTU、CO3D R 多视图改善明确，但不能说每组均最好。

**机制与证据边界。** 小视差以插值和像素对齐为主；确定性回归或专门的半稠密模型可有优势，生成先验不是所有指标均占优的保证。P=1 单位长度扫描（§4.2，0.1–2.0，选每场景最佳）使评估含真值选择，实际部署不能默认已知最优尺度。表 7 的 V 列 P=2 而此表 V 列 P=1，原文不一致，保留各表定义。原表颜色排序不作为数据本身，完整数值逐格核对。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
### Table 3 — §4.2 Set NVS

PDF p.8。

**Caption 原文（PDF 文本层，图标另释）**：Table 3. PSNR↑on large-viewpoint set NVS. For all results with P = 1, we sweep the unit length for camera normalization due to the model’s scale ambiguity. Underlined numbers are run by us using the officially released code.

**题注译文**：大视差集合 NVS 的 PSNR（越高越好）。所有 P=1 的结果中，因模型尺度歧义，扫描相机归一化单位长度。下划线数值由作者使用官方发布代码运行。

| 数据集/划分/P | SV3D | DepthSplat | CAT3D | ViewCrafter | SEVA |
|---|---:|---:|---:|---:|---:|
| OO3D/S/1 | 19.28 | 11.56† | - | 10.56† | 19.25 |
| GSO/S/1 | 20.38 | 12.32† | - | 11.42† | 20.65 |
| CO3D/R/1 | - | 10.42† | - | 10.11† | 15.30 |
| WRGBD/Oh/1 | - | 9.35† | - | 9.12† | 14.37 |
| WRGBD/Oh/3 | - | 13.53† | - | 13.45† | 17.28 |
| Mip360/R/1 | - | 10.49† | - | 9.79† | 12.93 |
| Mip360/R/3 | - | 12.54† | 15.15 | 10.34† | 15.78 |
| DL3DV/O/1 | - | 9.63† | - | 8.97† | 13.01 |
| DL3DV/O/3 | - | 12.52† | - | 11.50† | 15.95 |
| T&T/O/1 | - | 8.63† | - | 9.23† | 11.28 |
| T&T/O/3 | - | 9.78† | - | 9.88† | 12.65 |
| T&T/O/6 | - | 10.12† | - | 10.32† | 13.80 |
| T&T/O/9 | - | 11.20† | - | 11.08† | 14.72 |

† 为原表下划线，− 为未报告。

**比较。** OO3D/S/1 SEVA 19.25，比 SV3D 19.28 低 0.03；GSO/S/1 20.65 vs 20.38，高 0.27，几乎同一水平。CO3D/R/1 15.30 vs DepthSplat 10.42，高 4.88。WRGBD/Oh/3 17.28 vs DepthSplat 13.53，高 3.75；同组 P=1 时 14.37 vs 9.35 高 5.02。Mip360/R/3 15.78 vs CAT3D 15.15，高 0.63。DL3DV/O/3 15.95 vs DepthSplat 12.52，高 3.43；T&T/O P=1/3/6/9 的 SEVA 11.28/12.65/13.80/14.72，总体随 P 增加，P=9 与最优基线 DepthSplat 11.20 相差 3.52。

**证据边界。** 大视差更依赖不可见区域生成，扩散模型较回归在多个条件有更强优势；原文对未报告项保持空白，不可推断专有 CAT3D 在其他所有数据均输。P=1 扫描同样限制真实部署可复现性；PSNR 只比较像素真值，合理但不同的未见场景可能得低分。没有深度几何准确率，因此结论应限定为测试图像指标。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
### Table 4 — §4.2 Set NVS

PDF p.8。

**Caption 原文（PDF 文本层，图标另释）**：Table 4. PSNR↑on 3DGS renderings for set NVS. Results are reported on the ReconFusion [7] split with P = 3.

**题注译文**：集合 NVS 的 3DGS 渲染 PSNR（越高越好）。在 ReconFusion [7] 划分、P=3 下报告结果。

| 数据集/划分/P | ZipNeRF | ZeroNVS | ReconFusion | CAT3D | SEVA |
|---|---:|---:|---:|---:|---:|
| RE10K 小 | 20.77 | 19.11 | 25.84 | 26.78 | 27.95 |
| LLFF 小 | 17.23 | 15.91 | 21.34 | 21.58 | 21.88 |
| DTU 小 | 9.18 | 16.71 | 20.74 | 22.02 | 22.68 |
| CO3D 小 | 14.34 | 17.13 | 19.59 | 20.57 | 21.88 |
| Mip360 大 | 12.77 | 14.44 | 15.50 | 16.62 | 17.82 |

**实验过程。** 与 Table 2–3 直接图像采样不同，本表用生成的 8 段视频共 720 视图和真实输入蒸馏到 3DGS-MCMC，点云不作初始化，优化相机参数、使用 LPIPS 损失，最终渲染测试视角（§4.2 p.8）。ZipNeRF/ZeroNVS/ReconFusion/CAT3D 为对照方法。

**结果。** SEVA 五列分别 27.95、21.88、22.68、21.88、17.82 dB；相对 CAT3D 26.78、21.58、22.02、20.57、16.62 提升 1.17/0.30/0.66/1.31/1.20。五列简单等权差均值 0.928 dB（补充计算，非论文汇总指标）。论文引言另声称 CAT3D 原设定 +1.5 dB，不将其等同此表均值或凭空解释汇总口径。

**证据边界。** 结果说明生成视图可提供有助于少视图三维表示拟合的附加约束，但该步骤不是 SEVA 必需运行机制。相机优化和高斯表示也影响最终像素对齐，不能单独隔离生成模型因果，更不能证明真实深度。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
### Table 5 — §4.3

PDF p.10。

**Caption 原文（PDF 文本层，图标另释）**：Table 5. PSNR↑on trajectory NVS. temp. denotes optional temporal pathway. RE, WR, and DL denotes RE10K, WRGBD, and DL3DV, respectively. For the V [9] split, P = 1 with unit length swept; for the O split, P = 3. Underlined numbers are run by us using the officially released code.

**题注译文**：轨迹 NVS 的 PSNR（越高越好）。temp. 为可选时序路径。RE、WR、DL 分别表示 RE10K、WRGBD、DL3DV；V 划分 P=1 并扫描单位长度，O 划分 P=3。下划线为作者使用官方代码运行数值。

| 数据集/划分/P | MotionCtrl | DepthSplat | ViewCrafter | SEVA | SEVA (+ temp.) |
|---|---:|---:|---:|---:|---:|
| RE/V/1 小 | 16.29 | 19.24† | 22.04 | 18.56 | 18.62 |
| CO3D/V/1 小 | 15.46 | 13.23† | 18.96 | 18.40 | 18.43 |
| T&T/V/1 小 | 13.29 | 14.35† | 18.07 | 15.16 | 15.13 |
| RE/O/3 小 | - | 25.23† | 26.54† | 27.34 | 27.36 |
| DTU/O/3 小 | - | 14.68† | 18.99† | 19.99 | 20.19 |
| WR/O/3 大 | - | 12.45† | 13.44† | 17.79 | 17.93 |
| DL/O/3 大 | - | 11.32† | 11.45† | 15.76 | 15.78 |
| T&T/O/3 大 | - | 9.11† | 9.68† | 11.92 | 11.99 |

原 PDF 表头 small-viewpoint 覆盖 V 三列及 O 的 RE、DTU 两列，large-viewpoint 覆盖 O 的 WR、DL、T&T 三列；不要把 O 整组等同于大视差。† 表原下划线。

**比较。** V 的 SEVA 相对 VC（22.04、18.96、18.07）为 18.56、18.40、15.16，差 −3.48、−0.56、−2.91。O 中 RE/DTU 的 SEVA 27.34、19.99 比 VC 26.54、18.99 高 0.80、1.00；WR/DL/T&T 17.79、15.76、11.92 比 VC 13.44、11.45、9.68 高 4.35、4.31、2.24。时序路径对 O 五列增量 +0.02/+0.20/+0.14/+0.02/+0.07；V 中 T&T 反降 0.03。

**机制与证据边界。** 用同组真实输入、完整有序目标帧生成轨迹，PSNR 是单帧与真值接近程度，不直接度量闪烁。作者将 V 上 VC 的优势归因于高分辨率/宽比例输入和训练，这不是隔离分辨率的受控消融。可选 temporal 对此表多数 PSNR 提升小，需结合 Table 6 的 MS/TSED 与 Fig.7，不能从 PSNR alone 得出无闪烁。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
### Table 6 — §4.3

PDF p.10。

**Caption 原文（PDF 文本层，图标另释）**：Table 6. 3D consistency (TSED↓and PSNR↑) and temporal quality (MS↑) on trajectory NVS. SEVA uses interp procedural sampling by default. temp. denotes the optional temporal pathway. MS denotes motion smoothness from VBench [52]. Results are reported on our split of DL3DV with P = 3.

**题注译文**：轨迹 NVS 的三维一致性（TSED 越低越好、PSNR 越高越好）和时序质量（MS 越高越好）。SEVA 默认使用 interp 程序化采样，temp. 是可选时序路径。MS 为 VBench 的运动平滑性；本文 DL3DV 划分 P=3。

| 方法 | samples PSNR↑ | samples TSED↓ | 3DGS PSNR↑ | video MS↑ |
|---|---:|---:|---:|---:|
| one-pass | 15.73 | 115.1 | 16.03 | 95.39 |
| two-pass nearest | 13.74 | 120.9 | 14.21 | 94.71 |
| two-pass gt + nearest | 15.58 | 116.2 | 15.96 | 95.22 |
| two-pass gt + interp | 15.66 | 120.1 | 15.98 | 95.56 |
| SEVA (interp) | 15.76 | 116.7 | 16.11 | 95.76 |
| SEVA (+ temp.) | 15.78 | 109.0 | 16.17 | 95.77 |

**指标定义及读法。** samples PSNR 是生成图像与真实目标的像素误差；3DGS PSNR 是将生成样本统一拟合到表示后重新渲染的指标；正文将 SED 描述为 SIFT 特征匹配与极线最短距离，表头为 TSED，未给出额外聚合、阈值、单位及 T 的详细定义，保留原标签而不自造定义；MS 来自 VBench 运动平滑性，越高越好，原文不明确百分比单位。三者分别表征像素对齐、跨视角匹配/可拟合性与时序平滑度。

**控制变量。** nearest 只用锚点，加入 gt 原输入后 samples PSNR 从 13.74 到 15.58、TSED 从 120.9 到 116.2、MS 从 94.71 到 95.22；说明真实输入仍能补充空间锚点条件。interp 与 one-pass 比：PSNR +0.03、3DGS +0.08、MS +0.37，但 TSED 从 115.1 升至 116.7（变差 1.6），所以不是所有指标都改善。gt+interp 与默认 interp 比，MS 95.56 vs 95.76、TSED 120.1 vs 116.7；增加真实输入并非在此有序策略中必然提升，可能改变窗口预算，作者未隔离原因。加 temporal 后相对默认 TSED 降 7.7、PSNR +0.02、3DGS +0.06、MS +0.01；该组合四列为表中最优。结合 Fig.7，端点共享主要改善轨迹连续性，可选时序层可进一步提升部分跨视角匹配。

**证据边界。** 这是一组有明确策略对照的消融，但只在 DL3DV/O/3 设置测试，无方差、重复 seed 或显著性检验，不能将小差值当普遍因果收益。极线匹配也受纹理、特征检测及相机误差影响，并不等于准确深度。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN TABLE-07 -->
### Table 7 — Appendix C

PDF p.18。

**Caption 原文（PDF 文本层，图标另释）**：Table 7. Statistics for NVS benchmark. We consider 10 publicly available datasets commonly used for evaluating NVS, encompassing both object-level and scene-level data. Views from Mip-NeRF360 [42] derive from several disjoint captures following different camera trajectories, thus all views (Iinp, Itgt) ∼I. P denotes the number of input views. Depending on the disparity between Iinp and Itgt, we group NVS tasks into small-viewpoint NVS (top panel) where target views are similar to input views and large-viewpoint NVS (bottom panel) where target views are more different to input views.

**题注译文**：NVS 基准统计。考虑 10 个常用于评估 NVS 的公开数据集，包含物体级和场景级数据。Mip-NeRF360 视图来自多个不连续的相机轨迹，因此所有视图属于无序集合 I。P 是输入数量。按输入与目标视差分组为目标接近输入的小视差 NVS（上）及目标差异更大的大视差 NVS（下）。

| 视差组 | 数据集 | 类型 | 划分 | 场景数 | 有序 V | P | DCLIP |
|---|---|---|---|---:|---|---:|---:|
| 小 | OO3D | 物体 | O 动态环绕 | 308 | ✓ | 3 | 0.11 |
| 小 | GSO | 物体 | O 动态环绕 | 300 | ✓ | 3 | 0.11 |
| 小 | RE10K | 场景 | D | 128 | ✓ | 1 | 0.09 |
| 小 | RE10K | 场景 | R | 10 | ✓ | 1 | 0.08 |
| 小 | RE10K | 场景 | R | 10 | ✓ | 3 | 0.03 |
| 小 | RE10K | 场景 | P | 6474 | ✓ | 2 | 0.04 |
| 小 | RE10K | 场景 | V | 10 | ✓ | 2 | 0.11 |
| 小 | LLFF | 场景 | R | 8 | ✓ | 1 | 0.04 |
| 小 | LLFF | 场景 | R | 8 | ✓ | 3 | 0.03 |
| 小 | DTU | 场景 | R | 15 | ✓ | 1 | 0.07 |
| 小 | DTU | 场景 | R | 15 | ✓ | 3 | 0.06 |
| 小 | CO3D | 场景 | R | 20 | ✓ | 3 | 0.09 |
| 小 | CO3D | 场景 | V | 10 | ✓ | 2 | 0.09 |
| 小 | WRGBD | 场景 | Oe 1/3环绕 | 20 | ✓ | 3 | 0.07 |
| 小 | WRGBD | 场景 | Oh 全环绕 | 20 | ✓ | 6 | 0.11 |
| 小 | Mip360 | 场景 | R | 9 | ✗ | 6 | 0.11 |
| 小 | DL3DV | 场景 | O | 10 | ✓ | 6 | 0.10 |
| 小 | DL3DV | 场景 | L | 140 | ✓ | 32 | 0.05 |
| 小 | T&T | 场景 | V | 22 | ✓ | 2 | 0.10 |
| 小 | T&T | 场景 | L | 2 | ✓ | 32 | 0.10 |
| 大 | OO3D | 物体 | S 动态环绕 | 308 | ✓ | 1 | 0.16 |
| 大 | GSO | 物体 | S 动态环绕 | 300 | ✓ | 1 | 0.18 |
| 大 | CO3D | 场景 | R | 20 | ✓ | 1 | 0.15 |
| 大 | WRGBD | 场景 | Oh 全环绕 | 20 | ✓ | 1 | 0.19 |
| 大 | WRGBD | 场景 | Oh 全环绕 | 20 | ✓ | 3 | 0.14 |
| 大 | Mip360 | 场景 | R | 9 | ✗ | 1 | 0.19 |
| 大 | Mip360 | 场景 | R | 9 | ✗ | 3 | 0.13 |
| 大 | DL3DV | 场景 | O | 10 | ✓ | 1 | 0.21 |
| 大 | DL3DV | 场景 | O | 10 | ✓ | 3 | 0.12 |
| 大 | T&T | 场景 | O | 2 | ✓ | 1 | 0.21 |
| 大 | T&T | 场景 | O | 2 | ✓ | 3 | 0.18 |
| 大 | T&T | 场景 | O | 2 | ✓ | 6 | 0.16 |
| 大 | T&T | 场景 | O | 2 | ✓ | 9 | 0.14 |

**划分与难度。** D=4DiM，V=ViewCrafter，P（划分符号）=pixelSplat，R=ReconFusion，S=SV3D，L=Long-LRM，O=本文，Oe/Oh 为易/难；P（数字列）是输入图像数，不与 P 划分混淆。表中 33 条设置；CLIP 距离定义为目标视图与最近输入的 CLIP 特征距离在所有目标上的平均：$D_{CLIP}=Q^{-1}\sum_j\min_i d(\phi(I_j^{tgt}),\phi(I_i^{inp}))$，$\phi$ 为 CLIP 特征，原文未指定具体距离度量（不能擅定欧氏或余弦）。阈值 ≤0.11 小、>0.11 大。相同 T&T/O 随 P=1→9 距离 0.21→0.14，说明输入增加减少最近语义差异；WRGBD 120°与360°的裁剪亦改变难度。CLIP 距离不是几何基线长度或像素遮挡精确量。

**样本选择。** 继承已有划分时按原工作输入/目标；自有 WRGBD 先选 21 帧再随机选 P、剩余目标；DL3DV/T&T 先每第八帧取目标，对剩余帧的相机平移+单位方向六维向量做 K-means（K=32），原文在此处结束，未明确选择中心附近实际帧及 P 个中心的细节。T&T/O 为 TRAIN、TRUCK；DL3DV/O 为附录 p.19 所列 10 个场景 ID（精确复现实验时以 PDF 原始清单为准，后续 EXP-01 保留）。WRGBD 正文称 20 个但 p.19–20 的列表实际 19 项，另有重复引介句；不臆造第 20 个。

**证据边界。** Table 7 的 V 划分三组 P=2，Table 2/5/8/11 则列 P=1；RE10K/R 的若干设定与 Table 2 一致，但不要将表 7 所有行强行等同 trajectory 结果列。本文复核保留原表条件，精确复现需要作者澄清。原文 §2.1 有序性指相机轨迹，不等于 set 输出按顺序生成。

<!-- PTH:END TABLE-07 -->

<!-- PTH:BEGIN TABLE-08 -->
### Table 8 — Appendix C/D.2

PDF p.19。

**Caption 原文（PDF 文本层，图标另释）**：Table 8. LPIPS↓(top) and SSIM↑(bottom) on small-viewpoint set NVS. For all results with P = 1, we sweep the unit length for camera normalization due to the model’s scale ambiguity. Oe and Oh denote the easy and hard split of our split. Underlined numbers are run by us using the officially released code.

**题注译文**：小视差集合 NVS 的 LPIPS（越低越好，上）和 SSIM（越高越好，下）。P=1 因模型尺度歧义扫描相机归一化单位长度。Oe/Oh 是易/难划分。下划线为作者使用官方发布代码运行。

**(a) LPIPS ↓**

| 数据集/划分/P | Long-LRM | MVSplat | DepthSplat | LVSM | MotionCtrl | 4DiM | ViewCrafter | SEVA |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| OO3D/O/3 | - | 0.411† | 0.404† | - | - | - | 0.427† | 0.049 |
| GSO/O/3 | - | 0.387† | 0.372† | - | - | - | 0.379† | 0.041 |
| RE10K/D/1 | - | 0.224† | 0.217† | - | 0.500 | 0.302 | 0.220† | 0.194 |
| RE10K/V/1 | - | 0.237† | 0.245† | - | 0.386 | - | 0.178 | 0.231 |
| RE10K/P/2 | - | 0.128 | 0.119 | 0.098 | - | - | 0.203† | 0.061 |
| RE10K/R/1 | - | 0.254† | 0.236† | - | - | - | 0.287† | 0.308 |
| RE10K/R/3 | - | 0.142† | 0.177† | - | - | - | 0.164† | 0.073 |
| LLFF/R/1 | - | 0.542† | 0.530† | - | - | - | 0.620† | 0.389 |
| LLFF/R/3 | - | 0.497† | 0.465† | - | - | - | 0.435† | 0.181 |
| DTU/R/1 | - | 0.386† | 0.369† | - | - | - | 0.485† | 0.316 |
| DTU/R/3 | - | 0.310† | 0.304† | - | - | - | 0.272† | 0.158 |
| CO3D/V/1 | - | 0.634† | 0.618† | - | 0.443 | - | 0.324 | 0.318 |
| CO3D/R/3 | - | 0.614† | 0.603† | - | - | - | 0.513† | 0.278 |
| WRGBD/Oe/3 | - | 0.504† | 0.499† | - | - | - | 0.324† | 0.215 |
| WRGBD/Oh/6 | - | 0.643† | 0.530† | - | - | - | 0.639† | 0.237 |
| Mip360/R/6 | - | 0.556† | 0.534† | - | - | - | 0.464† | 0.319 |
| DL3DV/O/6 | - | 0.527† | 0.481† | - | - | - | 0.558† | 0.232 |
| DL3DV/L/32 | 0.262 | 0.425† | 0.404† | - | - | - | - | 0.155 |
| T&T/V/1 | - | 0.519† | 0.462† | - | 0.473 | - | 0.283 | 0.354 |
| T&T/L/32 | 0.375 | 0.568† | 0.528† | - | - | - | - | 0.236 |

**(b) SSIM ↑**

| 数据集/划分/P | Long-LRM | MVSplat | DepthSplat | LVSM | MotionCtrl | 4DiM | ViewCrafter | SEVA |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| OO3D/O/3 | - | 0.554† | 0.636† | - | - | - | 0.538† | 0.935 |
| GSO/O/3 | - | 0.621† | 0.689† | - | - | - | 0.647† | 0.942 |
| RE10K/D/1 | - | 0.788† | 0.801† | - | 0.267 | 0.463 | 0.792† | 0.615 |
| RE10K/V/1 | - | 0.769† | 0.745† | - | 0.587 | - | 0.798 | 0.693 |
| RE10K/P/2 | - | 0.869 | 0.887 | 0.906 | - | - | 0.710† | 0.847 |
| RE10K/R/1 | - | 0.812† | 0.820† | - | - | - | 0.806† | 0.700 |
| RE10K/R/3 | - | 0.857† | 0.824† | - | - | - | 0.830† | 0.892 |
| LLFF/R/1 | - | 0.283† | 0.299† | - | - | - | 0.146† | 0.384 |
| LLFF/R/3 | - | 0.358† | 0.396† | - | - | - | 0.454† | 0.602 |
| DTU/R/1 | - | 0.576† | 0.601† | - | - | - | 0.542† | 0.652 |
| DTU/R/3 | - | 0.624† | 0.638† | - | - | - | 0.671† | 0.750 |
| CO3D/V/1 | - | 0.403† | 0.429† | - | 0.502 | - | 0.641 | 0.585 |
| CO3D/R/3 | - | 0.370† | 0.402† | - | - | - | 0.483† | 0.647 |
| WRGBD/Oe/3 | - | 0.405† | 0.436† | - | - | - | 0.465† | 0.670 |
| WRGBD/Oh/6 | - | 0.368† | 0.417† | - | - | - | 0.376† | 0.646 |
| Mip360/R/6 | - | 0.312† | 0.324† | - | - | - | 0.354† | 0.395 |
| DL3DV/O/6 | - | 0.487† | 0.513† | - | - | - | 0.469† | 0.546 |
| DL3DV/L/32 | 0.775 | 0.512† | 0.564† | - | - | - | - | 0.661 |
| T&T/V/1 | - | 0.394† | 0.413† | - | 0.384 | - | 0.563 | 0.437 |
| T&T/L/32 | 0.590 | 0.314† | 0.359† | - | - | - | - | 0.505 |

† 为原表下划线，缺值非零。LPIPS 是学习特征的感知距离，低较好；SSIM 用结构/亮度/对比等图像统计比较，高较好，通常不以 dB 表示。与 Table 2 的 20 设置一致。

**结果。** OO3D/GSO 的 LPIPS 0.049/0.041，SSIM 0.935/0.942，显著优于相应基线。RE10K/P/2 的 LPIPS 0.061 优于 LVSM 0.098，尽管 PSNR 25.66 低于 LVSM 29.67；SSIM 0.847 低于 0.906，体现锐利感知与精确对齐不完全等价。LLFF/R/3 LPIPS 0.181、SSIM 0.602；相对 ViewCrafter 0.435、0.454 改善，配合 Table 2 的 +5.96dB。DL3DV/L/32 LPIPS 0.155 优于 Long-LRM 0.262，SSIM 0.661 低于 0.775，PSNR 也较低；T&T/L/32 同样 LPIPS 0.236 vs 0.375、SSIM 0.505 vs 0.590。RE10K/R/1 LPIPS 0.308 劣于 DepthSplat 0.236；V 的 RE10K/T&T 也非最佳。

**证据边界。** 相对 PSNR，LPIPS 对部分感知模式更宽容，能更好支持作者的“photorealism”主张，但仍是一项特定特征距离，不等于人评或真实几何。P=1 扫描及 V 的条件冲突沿用 Table 2，不能把最佳感知当部署可达精度。

<!-- PTH:END TABLE-08 -->

<!-- PTH:BEGIN TABLE-09 -->
### Table 9 — Appendix C/D.2

PDF p.20。

**Caption 原文（PDF 文本层，图标另释）**：Table 9. LPIPS↓(top) and SSIM↑(bottom) on large-viewpoint set NVS. For all results with P = 1, we sweep the unit length for camera normalization due to the model’s scale ambiguity. Underlined numbers are run by us using the officially released code.

**题注译文**：大视差集合 NVS 的 LPIPS（越低越好，上）与 SSIM（越高越好，下）。所有 P=1 的结果扫描相机归一化单位长度，因模型尺度歧义。下划线数值为作者使用官方发布代码运行。

**(a) LPIPS ↓**

| 数据集/划分/P | SV3D | DepthSplat | CAT3D | ViewCrafter | SEVA |
|---|---:|---:|---:|---:|---:|
| OO3D/S/1 | 0.158 | 0.610† | - | 0.634† | 0.160 |
| GSO/S/1 | 0.140 | 0.543† | - | 0.559† | 0.137 |
| CO3D/R/1 | - | 0.756† | - | 0.789† | 0.445 |
| WRGBD/Oh/1 | - | 0.732† | - | 0.775† | 0.423 |
| WRGBD/Oh/3 | - | 0.588† | - | 0.603† | 0.289 |
| Mip360/R/1 | - | 0.691† | - | 0.723† | 0.573 |
| Mip360/R/3 | - | 0.491† | 0.488 | 0.540† | 0.364 |
| DL3DV/O/1 | - | 0.580† | - | 0.616† | 0.484 |
| DL3DV/O/3 | - | 0.405† | - | 0.576† | 0.316 |
| T&T/O/1 | - | 0.774† | - | 0.755† | 0.571 |
| T&T/O/3 | - | 0.706† | - | 0.671† | 0.463 |
| T&T/O/6 | - | 0.611† | - | 0.604† | 0.387 |
| T&T/O/9 | - | 0.487† | - | 0.546† | 0.328 |

**(b) SSIM ↑**

| 数据集/划分/P | SV3D | DepthSplat | CAT3D | ViewCrafter | SEVA |
|---|---:|---:|---:|---:|---:|
| OO3D/S/1 | 0.850 | 0.549† | - | 0.463† | 0.857 |
| GSO/S/1 | 0.880 | 0.612† | - | 0.575† | 0.873 |
| CO3D/R/1 | - | 0.385† | - | 0.277† | 0.536 |
| WRGBD/Oh/1 | - | 0.234† | - | 0.225† | 0.505 |
| WRGBD/Oh/3 | - | 0.335† | - | 0.321† | 0.603 |
| Mip360/R/1 | - | 0.206† | - | 0.199† | 0.282 |
| Mip360/R/3 | - | 0.291† | 0.294 | 0.264† | 0.377 |
| DL3DV/O/1 | - | 0.349† | - | 0.323† | 0.360 |
| DL3DV/O/3 | - | 0.452† | - | 0.400† | 0.480 |
| T&T/O/1 | - | 0.304† | - | 0.312† | 0.342 |
| T&T/O/3 | - | 0.315† | - | 0.328† | 0.385 |
| T&T/O/6 | - | 0.326† | - | 0.337† | 0.427 |
| T&T/O/9 | - | 0.367† | - | 0.343† | 0.452 |

**结果。** OO3D/S/1 LPIPS 0.160 vs SV3D 0.158（略差 0.002），SSIM 0.857 vs 0.850（高 0.007）；GSO LPIPS 0.137 vs 0.140，SSIM 0.873 vs 0.880，非所有指标一致。CO3D/R/1 LPIPS 0.445 vs DepthSplat 0.756、SSIM 0.536 vs 0.385；WRGBD/Oh/3 0.289/0.603 vs 0.588/0.335；Mip360/R/3 0.364/0.377 vs CAT3D 0.488/0.294；DL3DV/O/3 0.316/0.480 vs DepthSplat 0.405/0.452。T&T/O 随 P 增加，LPIPS 0.571→0.328，SSIM 0.342→0.452，与 PSNR 提升互补。

**证据边界。** 未观测区域的合理生成不必像素完全一致；同时多指标优势在此更集中于大视差，仍不能证明几何误差更小，且 CAT3D 仅报告 Mip360 一列。† 原下划线，- 未报告。

<!-- PTH:END TABLE-09 -->

<!-- PTH:BEGIN TABLE-10 -->
### Table 10 — Appendix C/D.2

PDF p.20。

**Caption 原文（PDF 文本层，图标另释）**：Table 10. LPIPS↓(top) and SSIM↑(bottom) on 3DGS renderings for set NVS. Results are reported on the ReconFusion [7] split with P = 3.

**题注译文**：集合 NVS 的 3DGS 渲染 LPIPS（越低越好，上）与 SSIM（越高越好，下）。在 ReconFusion 划分、P=3 下报告。

**(a) LPIPS ↓**

| 数据集/划分/P | ZipNeRF | ZeroNVS | ReconFusion | CAT3D | SEVA |
|---|---:|---:|---:|---:|---:|
| RE10K 小 | 0.332 | 0.422 | 0.144 | 0.132 | 0.078 |
| LLFF 小 | 0.373 | 0.512 | 0.203 | 0.181 | 0.164 |
| DTU 小 | 0.383 | 0.223 | 0.124 | 0.121 | 0.107 |
| CO3D 小 | 0.652 | 0.566 | 0.398 | 0.351 | 0.256 |
| Mip360 大 | 0.705 | 0.680 | 0.585 | 0.515 | 0.435 |

**(b) SSIM ↑**

| 数据集/划分/P | ZipNeRF | ZeroNVS | ReconFusion | CAT3D | SEVA |
|---|---:|---:|---:|---:|---:|
| RE10K 小 | 0.774 | 0.675 | 0.910 | 0.917 | 0.961 |
| LLFF 小 | 0.574 | 0.359 | 0.724 | 0.731 | 0.735 |
| DTU 小 | 0.601 | 0.716 | 0.875 | 0.844 | 0.867 |
| CO3D 小 | 0.496 | 0.581 | 0.662 | 0.666 | 0.702 |
| Mip360 大 | 0.271 | 0.316 | 0.358 | 0.377 | 0.454 |

**结果。** SEVA 五列 LPIPS 均最低，0.078/0.164/0.107/0.256/0.435，相对 CAT3D 0.132/0.181/0.121/0.351/0.515 均改善。SSIM 四列最好，DTU 0.867 低于 ReconFusion 0.875 但高于 CAT3D 0.844。Mip360 大视差 LPIPS 0.435 vs CAT3D 0.515，SSIM 0.454 vs 0.377，Table 4 PSNR 17.82 vs16.62。

**证据边界。** 这些数值是下游 3DGS 重渲染后的图像指标，非模型直接生成质量（Table 8–9），也非几何真值。Fig.13 说明生成样本可被较一致拟合，但仍有合理幻觉。

<!-- PTH:END TABLE-10 -->

<!-- PTH:BEGIN TABLE-11 -->
### Table 11 — Appendix C/D.2

PDF p.20。

**Caption 原文（PDF 文本层，图标另释）**：Table 11. LPIPS↓(top) and SSIM↑(bottom) on trajectory NVS. For the V [9] split, P = 1 with unit length swept; for the O split, P = 3. RE, WR, and DL denote RE10K, WRGBD, and DL3DV, respectively. Underlined numbers are run by us using the officially released code.

**题注译文**：轨迹 NVS 的 LPIPS（越低越好，上）与 SSIM（越高越好，下）。V 划分 P=1 并扫描单位长度，O 划分 P=3。RE、WR、DL 分别为 RE10K、WRGBD、DL3DV。下划线数值为作者使用官方发布代码运行。

**(a) LPIPS ↓**

| 数据集/划分/P | MotionCtrl | DepthSplat | ViewCrafter | SEVA | SEVA (+ temp.) |
|---|---:|---:|---:|---:|---:|
| RE/V/1 小 | 0.386 | 0.224† | 0.178 | 0.231 | 0.228 |
| CO3D/V/1 小 | 0.443 | 0.532† | 0.283 | 0.318 | 0.312 |
| T&T/V/1 小 | 0.473 | 0.415† | 0.324 | 0.353 | 0.356 |
| RE/O/3 小 | - | 0.134† | 0.120† | 0.079 | 0.078 |
| DTU/O/3 小 | - | 0.253† | 0.187† | 0.159 | 0.156 |
| WR/O/3 大 | - | 0.452† | 0.346† | 0.284 | 0.280 |
| DL/O/3 大 | - | 0.572† | 0.566† | 0.329 | 0.328 |
| T&T/O/3 大 | - | 0.685† | 0.674† | 0.514 | 0.510 |

**(b) SSIM ↑**

| 数据集/划分/P | MotionCtrl | DepthSplat | ViewCrafter | SEVA | SEVA (+ temp.) |
|---|---:|---:|---:|---:|---:|
| RE/V/1 小 | 0.587 | 0.723† | 0.798 | 0.693 | 0.695 |
| CO3D/V/1 小 | 0.502 | 0.486† | 0.641 | 0.585 | 0.590 |
| T&T/V/1 小 | 0.384 | 0.408† | 0.563 | 0.437 | 0.436 |
| RE/O/3 小 | - | 0.844† | 0.868† | 0.890 | 0.891 |
| DTU/O/3 小 | - | 0.723† | 0.739† | 0.756 | 0.760 |
| WR/O/3 大 | - | 0.447† | 0.464† | 0.613 | 0.616 |
| DL/O/3 大 | - | 0.539† | 0.523† | 0.475 | 0.476 |
| T&T/O/3 大 | - | 0.497† | 0.456† | 0.363 | 0.369 |

† 为原表下划线。表头小/大视差分组同 Table 5：前五列小，后三列大。

**结果。** V/1 的 ViewCrafter LPIPS 0.178/0.283/0.324，均优于 SEVA 0.231/0.318/0.353；SSIM 0.798/0.641/0.563 同样更高。O/3 五列 SEVA LPIPS 0.079/0.159/0.284/0.329/0.514，全部优于 VC 0.120/0.187/0.346/0.566/0.674；temporal 在五列再小幅改善。SSIM 在 O 的 RE/DTU/WR 有优势（0.890/0.756/0.613 vs VC 0.868/0.739/0.464），但 DL/T&T 为 0.475/0.363，低于 DepthSplat 0.539/0.497 与 VC 0.523/0.456，虽有 LPIPS、PSNR 领先。temporal 后 V/T&T LPIPS 0.356 比 0.353 稍差、SSIM 0.436 vs 0.437，反例说明额外时序层并非普适提升。

**证据边界。** 逐帧平均的 LPIPS/SSIM 与 PSNR 互补，不能直接测量视频闪烁；应结合 Table 6 的 MS 与 Fig.7。O 中后两组 SSIM 较低提示结构与感知得分并不总一致，不能宣称全面 SOTA。

<!-- PTH:END TABLE-11 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 图表之间的设计—验证关系

Figure 1–3 与 Table 1 定义目标：任意给定相机的集合生成与平滑轨迹生成既要空间跨度，又要局部连续。Figure 4 是实现的中心，连接固定长度的多视图训练和可变长度推理；Figure 5 处理第一阶段锚点自身超过窗口的问题。Figure 6 与 Tables 2–4、8–10 对应集合任务和可选三维蒸馏；Figure 7 与 Tables 5–6、11 对应轨迹任务和采样消融；Figure 8 支持长序列闭环的定性主张。Figure 9–12 将覆盖率、尺寸、CFG 和随机性分开考察；Figure 13 是 DL3DV 样本及其三维蒸馏的补充例证，Table 7 是评测协议；Figure 14 对照 padding 与直接扩窗，Figure 15–18 展示额外轨迹例。

**机制与实验**：不把 Figure 4 的架构示意当作各部件的独立因果证明。Table 6 实际改变的是采样策略和最后一行的 temporal pathway，并未逐项消融 3D/1D 注意力或训练课程。Table 2/3 与 Table 5 分别测试不同条件，不能将跨表 PSNR 直接相减作为同一控制变量的收益。Table 4/10 的可选三维蒸馏说明生成视图可供后续拟合，不说明网络内部已经恢复真实场景。

**证据层级**：定量表是指定数据、相机与指标下的比较；图像实例提供直观反例和能力展示；机制解释中的一般扩散、投影几何推导是教学补充；从这些结果推到任意场景、精确几何或物理一致性仍是假设。Table 7 的 V 输入数量与主结果表不完全相符，须以各表所标条件分别引用。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 任务与科学问题（§1、§2.1–2.2，PDF p.2、4）

给定 $M$ 张有位姿的 RGB 图像 $I^{inp}\in\mathbb R^{M\times H\times W\times3}$，及其相机 $\pi^{inp}$，输出由目标相机 $\pi^{tgt}$ 指定的 $N$ 张图像 $I^{tgt}\in\mathbb R^{N\times H\times W\times3}$。内参、外参均视作已知；这不是从无位姿图像同时估计相机和几何。作者定义稀疏为至多 8 张，半稠密是稀疏与数百张稠密捕获之间的过渡，不是固定的数值区间。

目标不仅是复现输入已看到的表面，还要在新相机下生成合理的未观测区域。多个不同场景可能投影为完全相同的稀疏输入，因此真实场景一般**不可识别**；增加先验能缩小解的集合，却不能凭空增加观测证据。论文选择学习条件分布 $p_\theta(I^{tgt}\mid I^{inp},\pi^{inp},\pi^{tgt})$，而不是承诺恢复唯一真值。

Set NVS 的目标相机通常跨越较大空间，目标顺序本身无意义；trajectory NVS 的目标有明确顺序，形成视频。大/小空间范围是另一评测维度，与这两种任务不是同义。相机运动在静态场景内产生视频变化，不等于建模物体随时间运动。模型成功需要跨视角一致、局部无闪烁、相机控制准确且输入配置灵活；单一像素误差不能充分刻画这些要求。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 为什么需要新的统一模型（§1、§2.2–2.3）

较大相机位移和视角旋转造成显著视差、遮挡变化与新表面暴露；图像模型擅长以先验生成这些区域，但各次生成的外观可能不一致。视频模型通过有序序列和时间算子约束局部变化，通常更擅长小位移插值，却可能因训练序列空间范围受限而难以处理大视角生成。这个矛盾是本文针对现有模型的概括，不是两类模型的数学必然属性。

此外，固定输入/输出张数、规定输入在序列开头等限制，使自由选择相机的任务很难统一。即使单窗口的生成一致，跨窗口采样仍会因为未知区域在不同随机过程里被分别补全而产生相互矛盾的结果。仅依赖相邻时间帧不保证再次到访同一空间位置时重现先前内容。作者以 Figure 8 的回环实例说明空间条件检索的价值，并在 Table 6 比较轨迹采样方案；不能把这些实验当作对所有长轨迹一致性的证明。

设计目标因此分为：大跨度生成能力、顺滑插值、灵活条件；进一步通过共享生成锚点，让跨窗口有公共证据。与先生成再强制拟合 NeRF 的路线相比，作者希望直接输出可用图像；其结论不排斥为下游编辑、渲染或测量再构建三维表示。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 相关工作与方法边界（§2、Appendix B）

| 路线 | 代表性工作（原论文所述） | 机制与限制 |
|---|---|---|
| 直接回归 | LFNR、LVSM | 从有位姿视图及查询相机回归目标像素/特征，不必须输出持久三维场；在未知区域通常依赖训练先验。 |
| 中间三维表示回归 | pixelNeRF、Splatter Image、pixelSplat、MVSplat、LRM | 前馈生成辐射场、高斯或其他三维表示，再按相机渲染；共享场为不同查询提供一致的结构，但稀疏数据不足时并不保证场正确。 |
| 图像扩散 | Zero-1-to-3、ZeroNVS、MVDream、SyncDreamer、HexGen3D | 以图像或相机条件采样未知视角；单视图或固定多视图配置有不同能力，不能笼统视为同一实现。 |
| 生成后融合 | CAT3D、ReconFusion | 利用生成先验产生视图并蒸馏进场景表示；其优势是持久场，代价可能是额外拟合步骤。 |
| 视频生成/相机控制 | SV3D、MotionCtrl、ViewCrafter、4DiM | 有序序列与运动条件强调局部连贯；训练或推理的视图配置与空间范围可成为限制。 |

§2.2 的关键分类是确定性映射与条件概率采样；是否输出三维场是另一个轴。SEVA 是直接多视图扩散，不要求先构建显式三维场。可选 3DGS 蒸馏不会改变其生成器的性质。Table 1 的对照项是本文作者对各工作所列能力和训练数据的概括。引用这些工作的具体性能及限制仅限原论文比较协议，未独立复核其他论文，不将其上升为完整技术演进史。

比较时应分清三层：表示层（有无显式场）、概率层（回归或采样）、推理组织层（单次联合或跨窗检索）。例如共享场可支持有序或无序相机查询；联合扩散即使没有共享显式场，也可在一个窗口中学习一致性。两者不互斥。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-01 -->
## 网络：统一多视图扩散与可选时间路径（§3.1，Figure 4，PDF p.4–5）

以 SD 2.1 的 VAE 与 latent denoising U-Net 为基础。作者将 U-Net 低分辨率残差块中的 2D self-attention 扩展为 3D self-attention；每个 self-attention 后增加沿视图轴的 1D self-attention（跳接），模型规模由 870M 增为 1.3B。另在每个 residual block 后通过跳接加入 3D convolution，得到可选 temporal pathway（总 1.5B）。该路径仅在已知窗口中图像按空间顺序排列时启用，用来增加平滑性。

以背景张量示例说明（非论文给定的精确实现尺寸）：输入特征 $X\in\mathbb R^{B\times T\times h\times w\times d}$，$T$ 为窗口视图数，$h,w$ 为特征分辨率。2D attention 在每个视图的 $hw$ 个 token 内混合；3D self-attention 将视图与空间联合，允许某一视图的局部特征直接读取其他视图的不同位置；1D self-attention 沿视图轴组织信息，给跨视图交互另一个专门路径。假设朴素全连接注意力，联合 token 数 $Thw$，注意力矩阵元素数为 $(Thw)^2$；若在每个空间位置沿视图轴计算，则为 $hwT^2$。这些是帮助理解扩窗代价的条件性复杂度，不是实际显存/运行时间测量；原文没有给出底层内核实现。

一般注意力 $\operatorname{Attn}(Q,K,V)=\operatorname{softmax}(QK^\top/\sqrt{d_k})V$（教学补充），其相关性由学习特征决定，不是显式求两条射线交点。这里的“3D”是视图轴与二维特征空间的张量运算，不代表建立世界坐标体素或高斯场。3D convolution 的局部时序邻域依赖帧的排列，有序路径使邻近 index 更可能对应邻近相机；任意视图集合不应无条件启用时间路径。训练仅优化新增 temporal weights 的阶段见 METHOD-02。图中 CLIP cross-attention、Plücker 与 clean/noisy latent 共同构成条件入口。

<!-- PTH:END METHOD-01 -->

<!-- PTH:BEGIN METHOD-02 -->
## 条件输入及训练课程（§3.1，Figure 4，PDF p.5）

对于每个输入视图，VAE 编码干净 latent；目标视图使用当前扩散时刻的 noisy latent。每个视图拼接对应 Plücker 射线嵌入及二值 mask，mask 告知网络哪些槽是已观测条件、哪些是待生成目标；相机嵌入也通过 adaptive layer normalization 注入。相机统一转到第一输入相机的相对坐标系，并缩放到所有相机中心均位于 $[-2,2]^3$，降低世界坐标原点、旋转与尺度约定的变化。具体射线坐标见 MATH-02。Figure 4 中 CLIP 图像嵌入沿输入视图平均并通过 cross-attention 提供高层语义；平均语义条件不是逐像素几何对应。第一层新增通道权重零初始化，使原预训练路径初始不被新通道强烈扰动；作者报告约 5K 次迭代已能产生真实感图像，不意味着训练已完成。

训练窗口 $T=M+N$ 固定，随机选 $M\in[1,T-1]$ 并采样输入/目标帧。两阶段课程如下：

| 阶段 | T | batch size | iterations | 训练范围 |
|---|---:|---:|---:|---|
| 初始 | 8 | 1472 | 100K | 多视图模型 |
| 扩窗 | 21 | 512 | 600K | 多视图模型 |
| 可选视频 | 延续模型窗口设置（原文未另列 T） | 512 | 200K | 仅 temporal weights |

朴素大窗口训练容易发散，先短后长用于稳定学习。以 0.2 概率使用较小的 subsampling stride 取得更细的局部时间粒度；可选视频训练使用小步长数据。这里并非所有样本都来自均匀相机轨迹。训练图像 576×576。所有阶段做 SNR shift：作者观察更多帧提供更多信息，因此需要更强加噪来破坏相关信息（相关文献 [8,33,34]）。SNR 的一般数学解释见 MATH-01；论文没有给出可据此唯一复现的具体 SNR 曲线、完整优化器配置或训练样本配比。

训练时随机输入/目标角色让推理时 P、Q 不必等于训练的 M、N，但固定 $T$ 仍是重要统计约束。随机角色不等于模型在任意 P、Q 与任意相机分布上均被训练过。

<!-- PTH:END METHOD-02 -->

<!-- PTH:BEGIN METHOD-03 -->
## 从固定 T 到可变 P/Q：两阶段推理（§3.2，Figure 4，PDF p.5–6）

令测试时 $P=|I^{inp}|$，$Q=|I^{tgt}|$。一个 forward pass 指一个窗口的一次联合扩散采样过程，不应理解为去噪 U-Net 只执行一次。若 $P+Q\le T$，联合生成所有目标，用重复输入补足 T；正文说第一输入，Figure 14 写最后输入，两处不一致，不能据此唯一确定实现。若 $P+Q>T$，one-pass 基线将目标拆成若干独立批次，$P+Q_i\le T$。各批共享真实输入却可能分别生成不同的未知细节。

**Two-pass / set**：第一阶段由输入生成覆盖相机区域的 anchors，第二阶段按相机空间邻近性分组。对象中心场景使用预设 360° 路径，前向场景可用 spiral。nearest 仅使用对应生成锚点；gt + nearest 还将真实输入放回每个窗口，作者默认后者，降低将生成错误作为唯一条件的风险。没有合理轨迹先验时退回 one-pass。这里 gt 指真实输入，而非测试目标真值，目标尚未生成时也不能比较其像素进行最近邻查找。

**Two-pass / trajectory**：第一阶段从路径均匀选锚点，第二阶段让相邻锚点共同约束中间目标。相邻段复用端点，保证了端点图像作为条件完全共享，而不是数学上保证段内不存在闪烁。interp 仅用两端；gt + interp 增加真实输入，导致可用目标槽减少，原论文相应调整步长；默认 interp。可选时间路径适用于此类已排序窗口。Figure 7 与 Table 6 给出相应验证，具体步长和窗口可行性见 MATH-03。

两阶段的作用是先协调全局关键观测，再降低局部生成难度；不是逐点建立连续三维表面。第一阶段锚点依然是随机预测，第二阶段可能继承错误。

<!-- PTH:END METHOD-03 -->

<!-- PTH:BEGIN METHOD-04 -->
## 大输入数及长序列的扩展（§3.3、§4.4–4.5，Figure 5）

当 $P>T$，普通窗口无法容纳所有真实输入。半稠密场景中作者在第一阶段 zero-shot 延长 T，使真实输入与锚点一次联合生成；例如 32 输入视图，作者报告甚至能扩到数百帧而不严重损失真实感。关键条件是输入已经覆盖大部分场景，任务从生成向插值转变；稀疏输入扩窗则会产生模糊等退化（Figure 9、14）。这是一种经验泛化，而非任意输入数、任意覆盖率和计算资源下的保证。第二阶段仍沿用原采样思路；若所有真实输入都放不下，原文没有详细给出每个子窗口的选择算法。

当 $Q\gg T$，第一阶段锚点本身超过预算。memory bank 存放已生成 anchor 图像及其相机位姿；生成下一组锚点时，自回归地检索空间最近邻作为条件，而非只拿时间上最近的图像。路径离开后返回某个位置时，时间邻近帧可能位于空间远处，空间检索能够重用先前相近视点的图像。Figure 5 示意此信息流，Figure 8 的三次回环可视化支持长期一致性改善。检索距离的精确实现、容量淘汰、冲突锚点更新未完整报告，不能把它视作带全局优化的 SLAM 地图。

误差可以通过自回归传播，且作者观察随着长度增加质量收益饱和或退化。共享 anchor 与 loop closure 在本文是图像条件层面的实践：重新到访相近相机位置时生成内容更一致，不是由严格位姿图约束求得几何闭环残差为零。它适合长轨迹生成，却尚不能证明可用于精准三维测量。

<!-- PTH:END METHOD-04 -->

<!-- PTH:BEGIN MATH-01 -->
## 数学机制一：条件生成与加噪过程

原文 §2.2 定义两类映射（均未编号）：

**公式 1（原文）**：$f_\theta(I^{inp},\pi^{inp},\pi^{tgt})$，确定性地输出 $I^{tgt}$。**公式 2（原文）**：$p_\theta(I^{tgt}\mid I^{inp},\pi^{inp},\pi^{tgt})$，从条件分布中采样。前者通常给单一结果；后者允许同一观测对应多个可能的隐藏外观，Figure 12 是经验例证。对平方误差回归，若条件分布双峰，最优预测是条件均值（教学补充）：对 $a$ 求 $\partial_a E[(Y-a)^2\mid C]=2(a-E[Y\mid C])=0$，得到 $a=E[Y\mid C]$。均值可能并非任何一个真实模式；这不意味着所有回归模型都使用平方误差，也不证明扩散一定优于回归。

原文 §3.1 说采用标准 diffusion process，无修改；下列是通用 DDPM **补充推导**，不是原文给定的完整训练损失或具体预测参数化。设干净目标 latent $z_0$、噪声时刻 $t$、噪声 $\epsilon\sim\mathcal N(0,I)$，$\alpha_t=1-\beta_t$、$\bar\alpha_t=\prod_{s=1}^t\alpha_s$。一步前向加噪为 $z_t=\sqrt{\alpha_t}z_{t-1}+\sqrt{1-\alpha_t}\epsilon_t$。递推独立高斯噪声，条件均值为 $\sqrt{\bar\alpha_t}z_0$，条件方差递推 $v_t=\alpha_t v_{t-1}+1-\alpha_t$、$v_0=0$，所以 $v_t=1-\bar\alpha_t$。

**公式 3（补充）**：

$$z_t=\sqrt{\bar\alpha_t}z_0+\sqrt{1-\bar\alpha_t}\epsilon.$$

在单位信号方差假设下，信号功率与噪声功率之比为 $\mathrm{SNR}(t)=\bar\alpha_t/(1-\bar\alpha_t)$。增加噪声降低 SNR；作者在多帧时做 SNR shift，是因为跨视图关联会使联合输入携带更多可恢复信号。原文没有给出 shift 函数。通用噪声预测训练可写为 $E\|\epsilon-\epsilon_\theta(z_t,t,C)\|^2$；这里 $C$ 包括干净输入 latent、相机及掩码。仅用于解释条件去噪，不能视为作者已经披露了具体 $\epsilon$/v 参数化、加权损失或调度器。SEVA 联合生成的是整批目标 latent，而不是为每张图独立运行无条件扩散。每一步依赖当前全部窗口的条件信息；VAE 解码得到 RGB。扩散随机性表达可能解，不直接给出经过校准的几何不确定度。

<!-- PTH:END MATH-01 -->

<!-- PTH:BEGIN MATH-02 -->
## 数学机制二：相对坐标、射线条件与尺度歧义

原文 §3.1：相机相对第一输入相机归一化，所有相机位置在 $[-2,2]^3$；以 Plücker embedding 条件化。以下使用 camera-to-world 旋转 $R_i$、中心 $c_i$、内参 $K_i$ 说明原理（补充推导，原文没有指定 Plücker 分量排列或精确缩放代码）。

**公式 4（补充）**：$R'_i=R_1^\top R_i,\quad c'_i=sR_1^\top(c_i-c_1)$。第一输入相机变为单位旋转和原点；共同尺度 $s>0$。若 $m=\max_i\|R_1^\top(c_i-c_1)\|_\infty>0$，一种满足边界的方案是 $s=2/m$；若 $m=0$（所有中心重合），此式失效，须另选尺度。原文只规定立方体边界，并未确认这就是实际算法。

**公式 5（补充）**：像素齐次坐标 $\tilde u=(u,v,1)^\top$，在归一坐标系的射线方向 $d=R'_iK_i^{-1}\tilde u/\|R'_iK_i^{-1}\tilde u\|$，起点 $o=c'_i$，射线 $x(\lambda)=o+\lambda d$。Plücker 直线可写为 $(d,m_l)$，其中 $m_l=o\times d$。因为 $(o+\lambda d)\times d=o\times d$，沿同一条线改变起点不会改变 moment；$d\cdot m_l=0$，说明六维参数受约束。射线方向提供朝向，moment 提供相对空间位置；没有深度 $\lambda$，故不能单凭它确定表面点。像素级射线嵌入使去噪器能访问目标相机空间信息，但并未硬编码极线约束。

**公式 6（补充）**：$\tilde u\sim K_iR_i^\top(X-c_i)$。对任意共同尺度 $s>0$，若 $X'=sX,c_i'=sc_i$，则齐次投影仅乘 s，归一化像素不变。仅一张输入时没有第二个已观测相机基线，因此训练归一化尺度与目标运动相对于对象的距离之间缺少锚定信息。这里的尺度歧义不表示相机参数一定未知，而是模型依据单图难以判断应生成多大的相对视差。§4.2 将 RE10K 单视图问题归因于跨数据集尺度与单位归一化，并对 P=1 从 0.1 到 2 选每场景最佳 unit length。该选择是评测条件，不能当作无真值的自动尺度估计算法。

<!-- PTH:END MATH-02 -->

<!-- PTH:BEGIN MATH-03 -->
## 数学机制三：采样分组、步长与窗口预算

原文 §3.2（公式未编号）。**公式 7（原文）**：$P+Q_i\le T$。对 one-pass，若 $P<T$ 且目标不空，最大每窗目标数 $T-P$，至少 $\lceil Q/(T-P)\rceil$ 个窗口（补充，忽略 padding）。$P\ge T$ 时没有目标容量，必须另作扩窗或条件选择。

**公式 8（原文）**：

$$\mathrm{nearest}:\{I_i^{acr}\}\cup\{I_j^{tgt}\mid NN(I_j^{tgt},I^{acr})=I_i^{acr}\}.$$

其含义是目标依据相机空间关系归到 anchor 的邻域，并在同窗中生成；如采用 gt + nearest，还要占用 P 个真实输入槽，因此每窗未知目标至多 $T-P-1$（单个锚点假设）。分组大小超过该上限需要切分，但原文没有详细描述边界算法。最近邻不是用未知目标 RGB 作为检索特征。

**公式 9（原文）**：$\Delta=\lfloor Q/(T-2)\rfloor$；

$$\mathrm{interp}:\{I_i^{acr},I_{i\Delta+1}^{tgt},\ldots,I_{(i+1)\Delta-1}^{tgt},I_{i+1}^{acr}\}.$$

**公式 10（原文）**：gt + interp 时 $\Delta=\lfloor Q/(T-P-2)\rfloor$。分母中的 2 反映两端点预留；实际相邻端点相隔 $\Delta$ 时，内部 $\Delta-1$ 个目标，加两锚点共 $\Delta+1$ 个槽；如追加 P 个输入，共 $P+\Delta+1$。这是根据序列记号得到的补充计数，并不等价于作者公开了完整索引实现。

**窗口约束的具体例子**：T=21、P=1、Q=95 时，$\Delta=\lfloor95/19\rfloor=5$，一个完整相邻区间内 4 个中间目标和 2 个锚点，共 6 槽，可放进 21 槽。末尾余数、端点是否正好对齐需要实现决定。相邻段复用锚点，因此能固定边界图像，但不能保证中间三维真实。

边界反例：T=21、P=18、Q=5 时 P+Q>T 却有 $\lfloor5/19\rfloor=0$；若 P≥T−2，gt + interp 分母非正；Q=1000 时 $\lfloor1000/19\rfloor=52$，单段 53 槽超过原 T。原文 §3.3 对长序列补充 anchor memory bank，但没有提供足以唯一解决全部端点、步长与第二阶段超窗问题的伪代码。因此这些式子表达采样组织思想，不能无条件当作任意输入的可执行算法。工程实现需补齐合法步长、分段上界、重叠条件、余数和端点规则，并与作者源码核对；本次不擅自修正原文公式。

<!-- PTH:END MATH-03 -->

<!-- PTH:BEGIN EXP-01 -->
## 实验协议与评价（§4.1、Appendix C、Table 7）

对象 OO3D/GSO；对象中心场景 LLFF/DTU/CO3D/WildRGBD；场景 RE10K/Mip360/DL3DV140/Tanks and Temples，共 10 组。复用 4DiM(D)、ViewCrafter(V)、pixelSplat(P)、ReconFusion(R)、SV3D(S)、Long-LRM(L) 的划分；O 为自设。Table 7 完整列出 33 项任务配置，见逐表记录。RE10K/D 是 6711 个测试场景中的 128 个，P=1。自设 WRGBD Oe/Oh 分别取约 120°/360° 的序列，从 21 均匀抽样帧随机选 P 张作输入；DL3DV 与 T&T 每第 8 帧作目标，其他帧按相机平移和单位朝向组成 6 维向量进行 K=32 的 K-means 选择输入。Appendix C 提供选定场景名单；WRGBD 原文写 20 个，但实际列表可数 19 个，本报告保留差异，不补造第 20 个。Figure 13 展示 DL3DV 生成及蒸馏例。使用 CLIP 特征距离区分视差：逐目标计算其 CLIP 特征与所有输入的最小距离，再对目标求平均得到 DCLIP(I)，阈值 0.11（小视角不超过阈值）。这是语义感知代理而非严格几何视差；纹理/场景类型可能影响分类。

PSNR、SSIM 越高越好，LPIPS 越低越好。教学补充：同一动态范围 MAX 下 $\mathrm{PSNR}=10\log_{10}(\mathrm{MAX}^2/\mathrm{MSE})$，故 +3 dB 约为 MSE 减半；不同划分、分辨率和预处理不能直接合并。SSIM 衡量局部结构统计相似，LPIPS 衡量深层特征感知差异。轨迹另用 Image Quality、TSED、Motion Smoothness（见 EXP-03），它们分别提供失真、几何相关和运动平滑的代理。未报告完整指标聚合代码，不能假定每场景等权。

特别注意 P=1 的尺度扫描：每个场景选 0.1–2 内最佳 unit length，训练 2.0。这意味着表中单图结果带每场景选择条件，若没有已知目标评估或其他尺度线索，实际部署未必能选中最佳值；对公平性的结论需检查其他方法是否获得类似选择机会。ViewCrafter 宽幅输入更多像素是作者解释 RE10K/V 优势的可能原因，而非已控制像素数的独立消融。原文 Table 7 的 V 配置与主结果表 P 标注有冲突（详见 Table 7），应按具体表说明解读。

<!-- PTH:END EXP-01 -->

<!-- PTH:BEGIN EXP-02 -->
## Set NVS 主结果与可选 3DGS 蒸馏（§4.2、Tables 2–4、8–10）

以下摘自已经完整转录的表，所有单位 PSNR 为 dB。小视角 Table 2，OO3D/GSO 的 O/P=3 SEVA 30.30/31.53，对照 DepthSplat 15.67/16.52；LLFF/R/P=3 为 19.48 对 13.52（+5.96）；RE10K/R/P=3 为 27.57 对 25.64（+1.93）。RE10K/P/P=2 为 25.66 对 ViewCrafter 21.42（+4.24），仍落后 LVSM 29.67。单输入 RE10K 小视角不占优势，作者解释为尺度歧义。半稠密 P=32 的 DL3DV 20.96 对 Long-LRM 23.86，T&T 16.50 对 18.20；说明通用模型有竞争力但不应宣称全面领先专用回归模型。

大视角 Table 3，OO3D 为 19.25 对 SV3D 19.28，GSO 20.65 对 20.38，不能把小视角的巨大优势外推；Mip360/R/P=3 相比 CAT3D +0.63；T&T 自设 P=1/3/6/9 的 SEVA 为 11.28/12.65/13.80/14.72，更多观测与更高得分同时出现，但不同 P 的目标集合和输入覆盖率可能不同，不能由此拟合普适提升规律。Tables 8–9 补充 LPIPS/SSIM，大范围下 LPIPS 优势更明显，但仍存在不占优的格子。Figure 6 的物体、结构细节与大视角补全是定性例证。

3DGS 下游实验（Table 4/10）：同一输入生成 8 个视频，共 720 张生成视图，使用 3DGS-MCMC 拟合，不用点云初始化，同时优化相机参数并引入 LPIPS 损失。之后渲染评估目标，故这是「生成视图→三维拟合→渲染」的结果，不等同于直接生成器 PSNR。Table 4 五列全部优于 CAT3D；逐列差值 1.17、0.30、0.66、1.31、1.20，简单等权平均 0.928 dB（非官方聚合）。Table 10 LPIPS 五列均最佳，SSIM 的 DTU 0.867 低于 ReconFusion 0.875。该实验支持生成结果适合进一步建场，不证明已恢复真值深度、点云或相机。

<!-- PTH:END EXP-02 -->

<!-- PTH:BEGIN EXP-03 -->
## Trajectory NVS 与长程一致性（§4.3–4.4，Figures 7–8、Tables 5–6、11）

Table 5 V/P=1 的 RE10K/CO3D/T&T，SEVA 18.56/18.40/15.16，ViewCrafter 22.04/18.96/18.07；O/P=3 的 RE10K/DTU/WRGBD/DL3DV/T&T，SEVA 27.34/19.99/17.79/15.76/11.92，对 VC 26.54/18.99/13.44/11.45/9.68。原表 small 包含 V 三项及 O 的 RE/DTU，large 为 O 后三项。Table 11 O 的 LPIPS 全部优于对照，DL/T&T 的 SSIM 仍较低。可选 temporal pathway 大多小幅改善，不能表述为每项都改善（V/T&T 的 LPIPS 0.353→0.356）。

Table 6 按 one-pass、nearest、gt + nearest、gt + interp、interp、interp(+temp.) 对照，完整数值见逐表记录。关键对照：one-pass PSNR 15.73、TSED 115.1、3DGS PSNR 16.03、MS 95.39；interp 15.76、116.7、3DGS PSNR 16.11、95.76；加 temporal 后 15.78、109.0、3DGS PSNR 16.17、95.77。interp 对运动平滑有优势，但 TSED 未加 temporal 时反而略差于 one-pass；nearest 单独 13.74 明显低于 gt + nearest 15.58。没有误差条，0.01 的 MS 差值不能作为统计显著性证据。

TSED 越低越好，论文正文描述 SIFT 对应的 shortest epipolar distance，未完整给出 TSED 聚合定义。教学补充：若相对位姿将相机1坐标变到相机2为 $(R,t)$，$F=K_2^{-\top}[t]_\times RK_1^{-1}$，点 $x_1$ 的极线 $l_2=Fx_1$，匹配点 $x_2$ 到线的距离 $|x_2^\top Fx_1|/\sqrt{l_{2,1}^2+l_{2,2}^2}$。这是一侧误差示例而不是 TSED 的完整原式；依赖正确匹配与位姿，低值不保证恢复真实几何。IQ、MS 方向越高越好，论文未充分定义实际所有聚合细节。

Figure 7 沿视频帧抽取竖向切片表现局部闪烁；Figure 8 电话亭三回环、最长约 1000 帧，将空间检索与时间检索比较。再次到访同一位置能重用近邻 anchor，建筑、植被等更一致，支持闭环能力的定性主张。无全场景重复试验的闭环误差分布，也无 FPS 条件足以从 1000 帧换算具体时长；作者另称半分钟视频。Appendix A 指出动态物体、复杂纹理与极端轨迹限制，生成长度增长也可能累积误差。

<!-- PTH:END EXP-03 -->

<!-- PTH:BEGIN EXP-04 -->
## 推理泛化、随机性及局限（§4.5、Appendices A/D，Figures 9–14）

Figure 9 在 T&T 自设划分比较 P=1/3/6/9/16/32：两阶段 nearest-K 在 P 超过 K 后饱和（额外输入未被有效利用）；扩窗 one-pass 在半稠密覆盖较好时随输入增多提升，而稀疏扩窗常变模糊。稠密 3DGS 参考 23.7 PSNR、78.4 IQ；不是与稀疏条件同预算的比较。Figure 14 直接扩窗与 padding 对照支持保留训练 T 的经验价值。正文/图中的重复第一/最后输入描述不一致，报告不替作者选择实现。

Figure 10 用 576×576 训练的模型试验非方形图，说明零样本空间维度泛化的可能性；没有统一定量表证明任意分辨率和纵横比可用。Figure 11 考察 CFG：在高不确定生成情形，2/3 的权重可模糊，5/8 则在低不确定情形产生过饱和；作者经验建议 2–5。教学补充：常见 classifier-free guidance 写作 $\hat\epsilon=\epsilon_u+w(\epsilon_c-\epsilon_u)$，放大条件与非条件预测差异；论文未给出此处精确的分支置空规则，不得据此反推实现。CFG 强并不等于几何约束强。Figure 12 同一条件换种子可生成不同合理外观，说明歧义，不是后验概率校准实验证据。

Appendix A 展示局限：人/动物等变化主体、动态纹理（水面等）、极端轨迹与长程生成退化。静态多视图模型无法保证时变世界的物理一致性。Figures 15–18 扩展四类输入的 12 种轨迹定性展示，不能替代真实三维轨迹 GT 或可重复的统计实验。原文未完整公开训练配比、尺度选择的无监督办法或长序列所有窗口边界处理，因此单靠论文文字尚不足以完全复现。

<!-- PTH:END EXP-04 -->

<!-- PTH:BEGIN SYNTHESIS-01 -->
## 主张—证据矩阵

| 论文核心主张 | 最直接证据 | 支持强度与边界 |
|---|---|---|
| 同一模型适配 set 与 trajectory | Figure 4、Tables 2–5、8–11 | 多数据集直接比较支持广泛任务覆盖；不是所有配置最优。 |
| 灵活输入输出数量 | §3.1–3.3、Figure 9、Table 2 | 训练角色变化、P32 和长序列给出经验支持；稀疏扩窗退化。 |
| 大视角生成质量 | Table 3/9、Figure 6 | 真实结果支持多个大跨度配置，亦有 OO3D/GSO 差异。 |
| 局部平滑 | Table 6、Figure 7 | 有采样消融支持，temporal 增益有限；非全组件独立消融。 |
| 长程 3D 一致性与闭环 | Figure 5/8、§4.4 | 有定性对照，未见完整回环误差分布。 |
| 无需强制三维蒸馏 | 直接生成结果及 §4.2 Table 4/10 | 直接输出实用，但可选三维拟合仍有价值。 |

**未证实**：生成图像的几何深度达到真值、介质或光照参数可辨识、跨所有动态场景泛化、任意 P/Q 的计算成本可接受、所学分布已校准。模型避免显式场，不等于不存在隐式几何；相机射线条件不保证极线严格满足。对长路径空间检索，证据支持在示例回环中改善外观重现，不等于严格三维地图优化。若后续研究要求几何真值，应增加独立深度、重投影、尺度、闭环与遮挡区域评价。

<!-- PTH:END SYNTHESIS-01 -->

<!-- PTH:BEGIN SYNTHESIS-02 -->
## 全文覆盖核查与综合判断

本报告基于 arXiv:2503.14489v2 的 25 页 PDF。正文 §1–5、Appendix A–D 与其他附录图示均按主题分析；18 幅图逐一保留原题注、中文解释及原区域位置，11 张表完整转录并分析，公式保留原文记号且明确补充推导。目录的 Figure 8 题注曾漏末尾数行，依据实际 PDF 补齐；Figure 15–18 的轨迹标签依据页面图像而非错序文本层纠正。正文与附录的 padding 首/末输入、V 划分 P、WRGBD 场景数及部分表头与文字的差异未擅自抹平。原论文参考文献用于解释其 Related Work 与实验 baseline，未另行阅读其他论文，故不把作者评述当独立事实核验。

**综合判断**：本文真正有辨识度的设计是将条件扩散的灵活训练窗口和程序化生成组织结合起来：在窗口内用多视图注意力学习关联，在窗口间共享锚点，在长轨迹中依据空间位置取回已生成的条件。可选时间路径补充局部平滑，不是核心三维表示。多视角一致与生成合理性是它的主要成功标准；对高精度几何恢复，应额外区分真实观测和生成先验的证据权重。文章报告的优点与失败例相互一致：输入覆盖更多时插值更可靠，稀疏且跨度大时需要更强生成先验，而其结果未必唯一。本文适合研究相机控制、生成式 NVS 和跨窗口一致性；若要用于三维科学测量，必须另行验证几何真实性。

来源范围：原始 PDF 为权威；同包 LaTeX 和 HTML 仅作辅助，HTML 字体缺失不影响 PDF 图表。作者未充分披露的训练/采样细节不补造。当前内容审查是生产自检，独立 Review 须由隔离上下文执行。

<!-- PTH:END SYNTHESIS-02 -->



---

# 第二部分：学习与研究笔记

# Seva 学习与研究笔记

> 精读报告 SHA-256：1fd6094e521eed083c5909bd2c07daa3624eedf84c8926812fccf6b156449f1c
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：从生成式 NVS 内化知识

### 一、观测约束与先验不是同一回事

精读 PROBLEM/MATH-01 与 Figure 12 给出一个重要思想：给定输入 $C=(I^{inp},\pi^{inp},\pi^{tgt})$，目标随机变量可能存在多个模式。想象一张正面物体图，背面可以有图案 A 或 B，正面像素完全一致。两个候选世界都满足现有观测，生成器选中一个合理背面不代表它找到了真实背面。真实输入约束的是可观测信息；训练先验决定不可见部分如何补全。即使跨视角都呈现图案 A，几何、纹理和真实场景仍可能错误。

教学拓展：设渲染算子 $g_\pi(S)$，观测集合 $\mathcal O$，候选场景集合 $\mathcal S_C=\{S:g_{\pi_i}(S)\approx I_i, i\in\mathcal O\}$。稀疏条件下这个集合可能很大。增加真实视角，相当于增加交集约束；增加同一模型生成的视角不一定等价，因为它们共同依赖原 C，误差可能相关。数据处理角度，若生成 $G$ 只依赖 C 与独立随机种子，则真实场景 S、C、G 满足条件独立 $p(G\mid S,C)=p(G\mid C)$，因此 $I(S;G\mid C)=0$（假设模型权重固定、没有额外观测）。G 仍能改善有限算法的重建，因为它提供可计算的先验表达与优化初始化，但不能被当作独立新增测量。此为一般信息论推导，不是本文实验。

### 二、局部一致性和全局一致性

精读 METHOD-01/03/04、Figure 4/5/8：多视图注意力把窗口中各相机的像素特征联系起来；共享端点把多个窗口联系起来；空间检索又把时间上远离的重复位置联系起来。这三层作用尺度不同。可以将各窗口看作图节点，若共享同一 anchor 就连边。连通图只保证信息可传播；不能保证边界一致误差为零，更不能保证所有环路的几何可积性。反事实：完全独立的窗口即使各自都像真实图片，也可能在窗口 A 选择红门、窗口 B 选择蓝门；共享红门 anchor 会减少此冲突，但红门仍可能是错误先验。

如果改用一个持久三维表示，同一场景参数控制所有查询，输出一致性具有更强结构约束；但若表示本身错误，所有渲染会一致地错误。直接扩散更易借助二维图像先验，却缺少显式几何约束。二者是不同归纳偏置而非绝对优劣。

### 三、输入覆盖比输入张数更接近真正变量

Figure 9 对照近邻 K 饱和与扩窗在半稠密输入的改善。设目标可见表面集合 $V_q$，输入可见表面的并集 $V_\mathrm{obs}$，可用理想几何定义覆盖率 $c_q=|V_q\cap V_\mathrm{obs}|/|V_q|$（拓展示例；实际需遮挡和面积权重）。相机张数翻倍却都在同一位置附近，$c_q$ 可能几乎不变。少量分布合理的输入反而可能覆盖更多表面。SEVA 的实验证据提示覆盖的重要性，并未测量这个公式的覆盖率。一个更好的研究设计应分别固定 P 改变覆盖、固定覆盖改变 P，区分信息增益与模型窗口统计移位。

### 四、固定窗口泛化为何不是简单的形状兼容

训练 T=21，扩窗不改变网络权重但改变 token 之间的竞争关系。以单查询注意力为例，已有 n 个等分数 token，每个权重 1/n；加入 m 个同分数 token，原权重变成 1/(n+m)。即使输入张量维度合法，学习到的注意力统计仍改变。Appendix D 提出注意力分布移位作为解释，并非直接测量证明。可变 T 训练、重复 padding、基于覆盖的条件选择是不同干预：前者修改训练分布，第二种保留统计形状，第三种减少冗余。应区分是否增加真实信息，而不把所有长度效应归为内存不足。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Problem & Task（集合与轨迹 NVS）
- **Keywords:** Set NVS; Trajectory NVS; View Disparity; Sparse-view
- **Highlight:** SEVA 将目标顺序与输入—目标视角跨度分开，分别考察生成容量和插值平滑。其模型用同一训练 backbone 支持两种任务。
- **Evidence:** §2.1–2.3、Figure 3、§4.1、Table 7。
- **Connection Points:** 任务定义、目标相机分布、观测张数和视角跨度、是否要求序列连贯。
- **Open Questions:** 不同论文的小视差评估是否对应相近的覆盖率与可见区域比例？

### Topic: Observation & Prior（多视图条件扩散）
- **Keywords:** Conditional Diffusion; View-axis Attention; Plücker Rays; Latent Diffusion
- **Highlight:** 干净输入 latent、带噪目标 latent、二值 mask、相机射线与 CLIP 语义构成一个联合窗口，2D→3D 与视图轴注意力提供交互；可选 3D 卷积增强有序帧平滑。
- **Evidence:** §3.1、Figure 4、精读 METHOD-01/02。
- **Connection Points:** 观测注入层、相机编码、隐式/显式几何、目标联合噪声与时间条件。
- **Open Questions:** 跨视角注意力学习的相关性多大程度对应真实投影几何？

### Topic: Inference & Optimization（程序化分窗）
- **Keywords:** Anchor Generation; Procedural Two-Pass; Shared Conditioning; Interpolation
- **Highlight:** 第一阶段生成锚点，第二阶段在 set 中使用真实输入和空间近邻，在轨迹中使用共享两端点，以减少独立窗口的冲突。
- **Evidence:** §3.2、Figure 4、Table 6、精读 MATH-03。
- **Connection Points:** 窗口预算、锚点选择、真实/生成条件区分、分段交叠与端点处理。
- **Open Questions:** 锚点错误如何沿分段依赖图传播，是否需要置信度或回溯修订？

### Topic: Efficiency & Scalability（空间记忆与闭环）
- **Keywords:** Memory Bank; Spatial Retrieval; Autoregressive Sampling; Loop Closure
- **Highlight:** 长轨迹第一阶段按相机空间邻近性复用已生成锚点，而不是仅依赖时间最近的帧；电话亭三圈示例呈现重复视点的更高一致性。
- **Evidence:** §3.3、§4.4、Figures 5/8。
- **Connection Points:** 记忆单元、检索距离、可见性、回环修正、显存与长度扩展。
- **Open Questions:** 空间接近但朝向相反时应如何评估可共享信息？

### Topic: Generalization（上下文长度与覆盖）
- **Keywords:** Zero-shot Context Extension; Observation Coverage; Padding; Attention Distribution Shift
- **Highlight:** 半稠密覆盖条件下扩窗利用更多输入，而稀疏扩窗易退化；固定长度 padding 优于直接缩窗的示例表明训练分布仍重要。
- **Evidence:** §3.3、§4.5、Figures 9/14、Appendix D。
- **Connection Points:** 训练长度课程、条件冗余、覆盖率、分布外输入和计算代价。
- **Open Questions:** 能否通过变化 T 的训练解除覆盖率与长度的耦合？

### Topic: Observability & Identifiability（尺度与随机歧义）
- **Keywords:** Scale Ambiguity; Seed Sensitivity; Multimodal Posterior; Geometry Fidelity
- **Highlight:** P=1 的 unit-length sweep 是评测中的尺度选择；同一条件多种 seed 生成不同合理结果，外观一致性不保证真实场景可辨识。
- **Evidence:** §4.2、§4.5、Figure 12、Tables 2/3。
- **Connection Points:** 尺度来源、先验不确定性校准、真实几何评价与评测公平性。
- **Open Questions:** 不依赖目标真值的尺度估计与生成不确定性怎样独立验证？


## RESEARCH-DIRECTION：论文启发的开放研究方向

本任务未在当前论文指令中指定研究主线；以下按论文开放问题做一般分析，而非为读者预设新课题。

| 层级 | 本文对应内容 |
|---|---|
| Field | 三维视觉、生成建模、计算机图形学 |
| Task | 给定相机位姿的新视角合成 |
| Task Setting | 单/稀疏/半稠密、集合/轨迹、小/大视差 |
| Research Problem | 稀疏观测下如何兼顾大空间跨度、局部平滑与跨窗口一致性 |
| Method | 多视图扩散、两阶段锚点、空间记忆与可选时间路径 |

值得迁移的是约束组织机制，不必迁移整个模型。局部窗口可作为有界计算的推断单元，跨窗口共享高可靠条件可以缓解不一致；空间记忆能在重复观测区域恢复历史信息。若目标从视觉合成改为三维测量，任务的成功标准必须变化：RGB 真实感不足以评价深度、尺度和位姿，真实输入与生成锚点应有不同权重。原文 Table 4/10 仅说明样本可进一步拟合三维表示。跨成像域（如低照度、雾或水中参与介质）时，同一三维点的外观可能随传播路径变化，基于外观相似的检索可能与几何邻近性不同；这是物理条件下的迁移障碍假设，并非本文验证过水下场景。不能把动态水面失败直接当作水下静态介质实验。

三条方向可以并行：生成式 NVS 的可靠性（校准不确定性）、几何推断（真实观测约束与生成先验的可辨识性）、长程一致性（共享约束图与记忆）。其优先次序取决于实际目标是视觉交互、三维重建还是科学测量。本文为前者提供直接证据，后两者需额外任务设计与真实几何数据。


## RESEARCH-PROBLEMS：可证伪候选问题

以下是假设与实验设计，不宣称原创性；需要后续独立文献核查。优先级按本文直接证据与可控验证成本排序。

### RP-A（优先）：覆盖率与窗口泛化的解耦

现象：Figure 9 中固定 K 近邻饱和，半稠密扩窗提高指标，稀疏扩窗退化；未知的是决定扩窗有效性的究竟是 P、目标覆盖、输入冗余还是注意力统计。假设：在相同训练窗口和总 P 下，目标覆盖率更高时扩窗更有效；仅增加冗余视图不会产生同等收益。验证用有 GT 几何的静态场景，按目标可见表面分组构造等 P 高/低覆盖、等覆盖不同 P、复制视图与真实新增视图四类对照，固定目标相机、随机种子集合、分辨率、采样步数。比较固定 T+padding、nearest-K 与直接扩窗；记录 PSNR/LPIPS、已见/未见区域深度或重投影误差、计算预算和跨 seed 方差。成功判据：预先规定阈值，扩窗收益在控制 P 后仍随覆盖率变化；失败：控制覆盖后只与 P 或重复次数相关，或高覆盖同样退化。可行性依赖可见性真值和可执行模型，需核查其他可变上下文研究及覆盖定义。

### RP-B：生成锚点的可靠性如何传递

现象：Table 6 nearest 的 PSNR 13.74，gt + nearest 15.58，表明真实条件有价值；Figure 12 多 seed 表明生成不确定性。未知的是错误 anchor 是否会被邻域一致性掩盖。假设：明确区分真实与生成来源并按可验证一致性分配权重，可降低跨窗误差传播，而不牺牲可见区保真。用同一组真实输入与目标，构造 oracle 真值 anchor（上界）、自然生成 anchor、已知局部错误 anchor；对照固定权重、只用真实输入、增加真实输入、基于重投影/跨 seed 一致度的置信加权。控制锚点位置、数量、P、计算量和种子；分别报告已见/不可见区域的 RGB、深度误差，锚点到远端段落的误差传播曲线，置信度—错误校准曲线。成功判据为在相同预算下降低真实几何误差及错误传递，同时保持 LPIPS/平滑性；失败包括仅改善 LPIPS、错误置信度过高、牺牲可见区或只在 oracle 情况成立。工程上需要模型支持权重输入或外部选择；不能将候选算法假称已被 SEVA 验证。后续核查生成式三维重建、置信校准、合成数据误差相关性。

### RP-C：空间回环检索是否需要可见性与冲突处理

现象：Figure 8 支持空间检索的定性优势，但相机中心近并不保证共视，已存 anchor 自身也可能错。假设：相机中心、朝向及预测共视约束的检索，结合冲突检测，比纯位置或时间检索更能维持长程一致性。构造固定重复轨迹、折返、同位置反朝向、远距共视和长时间隔条件，对照时间邻近、位置邻近、位姿/可见性联合检索；统一 memory bank 大小、检索 K、总计算预算、锚点生成方法。指标为同相机回访的像素/特征差、真实深度重投影残差、误差随圈数增长率、运动平滑与检索代价。成功是闭环改善同时真实几何误差不增加；失败是反朝向仍错误关联、早期错误被强化，或一致性提高但真实误差增大。需要真实长轨迹或仿真 GT；后续核查视觉定位、图优化和长视频记忆相关工作。

优先做 A 可用现有模型和控制输入直接验证；B 的科学意义更接近生成与真实几何的根本矛盾，但需更复杂干预；C 可作为长轨迹扩展方向，不必先假定需要新网络。三者互有交叉，不能因为本文出现相似现象就认定存在尚无人研究的空白。


## INTERNALIZATION：自我复述与知识地图

如果只有几张已知相机图，网络无法直接知道背面的世界。SEVA 用学习到的条件分布生成合理候选，并让同一窗口里的各视图互相参考。训练时变化输入/目标角色，但固定总长度；推理时先用真实输入生成关键相机的图，再让附近目标共享这些锚点。长路径需要生成大量锚点，因此保存旧锚点并按空间位置重用。局部时间算子使有序图像更平滑。其亮点是以图像条件组织跨窗口一致性，而不是直接输出场景网格或高斯。

技术知识地图：稀疏观测 → 不可识别 → 条件分布 → 多视图去噪 → 训练长度统计 → 锚点生成与局部分组 → 空间记忆 → 局部/长程一致性 → 可选三维蒸馏。实验知识地图：Table 2/3 检验不同视差的生成/插值；Table 6 采样策略；Figure 8 空间回环；Figure 9 观察覆盖与窗口变化；Figure 12 多解；Table 4/10 下游拟合。

自检问答：

1. 输入视图数可变，是否代表训练 T 可变？否，角色 M/N 变化但阶段 T 固定。
2. 3D self-attention 是三维体素吗？否，联合视图和二维空间的 token；相机条件不自动产生显式表面。
3. gt + nearest 使用测试目标真值吗？否，gt 是给定真实输入。
4. 共享锚点能否证明真值恢复？否，只共享预测条件，错误也可能共享。
5. 为什么 P=1 评测需扫描尺度？缺少第二个真实输入基线，单位归一化下相对视差缺少锚定；评测选择不等于可部署的估计。
6. interp 的 TSED 是否全面优于 one-pass？否，116.7 高于 115.1；加 temporal 才降至 109.0。
7. 为什么半稠密扩窗有用？原文认为多数场景已被观察，更多真实信息把生成变为插值；其独立原因尚需受控试验。

尚未解决：窗口边界的完整算法、相机空间检索的具体距离、尺度无真值估计、生成先验的误差校准、真实几何与外观一致性的可区分评价。下一轮阅读应针对这些维度建立证据，而不是简单累加模型名。


## GLOSSARY：术语与概念区分

| 英文 | 中文 | 技术含义与易混点 |
|---|---|---|
| Set NVS | 集合新视角合成 | 无序目标集合，通常大空间跨度；不等于所有目标必定大视差。 |
| Trajectory NVS | 轨迹新视角合成 | 沿有序相机路径生成视频；静态场景的视角变化不等于物体运动。 |
| Sparse-view / Semi-dense | 稀疏/半稠密 | 本文稀疏至多 8 张；半稠密覆盖程度逐渐提高。 |
| Conditional diffusion | 条件扩散 | 给定观测与目标相机对联合目标分布采样；MATH-01。 |
| Latent / VAE | 潜变量/变分自编码器 | 在压缩表示中去噪，解码为 RGB；不是几何深度。 |
| 3D self-attention | 多视图联合自注意力 | 跨视图与空间 token，不等于显式三维场。 |
| View-axis attention | 视图轴注意力 | 专门沿视图轴的交互，区别于有序时序卷积。 |
| Temporal pathway | 时间分支 | 可选 3D 卷积，已知有序窗口时使用。 |
| Plücker embedding | 普吕克射线嵌入 | 射线方向与力矩，条件化相机；MATH-02。 |
| Adaptive LN | 自适应层归一化 | 注入条件到特征归一化；原文未披露全部映射。 |
| CLIP | 图像语义嵌入 | 语义条件或视差代理；不等于精确几何匹配。 |
| SNR shift | 信噪比调整 | 多帧联合信息更丰富时调整噪声强度，非具体已知曲线。 |
| Context window | 上下文窗口 | 联合采样的输入和目标槽数，训练 T 固定。 |
| Anchor view | 锚点视图 | 第一阶段生成的关键相机图像，不能等同真实观测。 |
| One-pass sampling | 单阶段分窗采样 | 超窗时独立生成各目标块；并非只执行一次 U-Net。 |
| gt + nearest | 真实输入加近邻锚点 | set 的默认策略，gt 指给定输入。 |
| interp | 两端锚点插值 | 轨迹默认策略，共享相邻段的边界条件。 |
| Memory bank | 记忆库 | 已生成锚点及相机元数据；区别于全局优化三维地图。 |
| Loop closure | 回环一致性 | 返回相近相机时外观重复一致；并非 SLAM 残差证明。 |
| CFG | 无分类器引导 | 调节条件生成强度，与严格几何约束不同。 |
| Scale ambiguity | 尺度歧义 | 单图缺少真实基线，归一化空间与实际场景尺度难匹配。 |
| PSNR/SSIM/LPIPS | 图像质量指标 | 像素/结构/感知比较，不直接验证深度。 |
| TSED / MS | 极线距离相关指标/运动平滑 | Table 6 的跨视角与视频代理；TSED 完整聚合未披露。 |
| 3DGS distillation | 三维高斯蒸馏 | 将生成图拟合到显式场后再渲染，独立于生成器输出。 |


## SOURCES：来源、版本与证据分类

- 论文：Stable Virtual Camera: Generative View Synthesis with Diffusion Models，arXiv:2503.14489v2，2025-04-01，25 页。
- PDF SHA-256：`ed2fe1926f43f5a9175465f810207a4a4d5a4a3c631cc65d10aaaec5eec70bc8`。
- 冻结精读报告 SHA-256：`1fd6094e521eed083c5909bd2c07daa3624eedf84c8926812fccf6b156449f1c`。
- 本笔记仅依据该版本 PDF 与冻结精读报告；HTML 及 LaTeX 作为辅助资源，HTML 缺字体，不替换 PDF 内容。论文的官方元数据与项目链接已在精读报告开头注明。
- 本文事实：模型配置、任务定义、采样策略和实际表格数值。教学补充：投影/扩散/信息论和窗口图解释，均附假设。研究假设：RP-A/B/C 的干预及成功失败判据，未实际实施。
- 跨文献分析延期：仅建立 CROSS-PAPER 潜在关联索引，未独立读取或比较其他论文，未验证候选问题原创性。
- 未证实项：生成图像对应真实几何、后验校准、任意长度无退化、训练配方及采样边界完整复现。原文配置差异见冻结报告，实际代码和模型未运行。

本笔记是学习与研究分析，不是独立审校报告；不含私人研究记录或隐含发布授权。隐藏完整版仍可访问，不应当作访问控制。



