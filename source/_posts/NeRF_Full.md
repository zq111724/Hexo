---
title: "NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis（完整精读与研究笔记）"
date: 2026-11-26 00:00:01
index_img: "/NeRF/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/NeRF/)

# 第一部分：论文精读报告

# NeRF 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2003.08934v2（2020-08-03）；核验时为最新版（2026-11-26）。

**官方资源：** [arXiv](https://arxiv.org/abs/2003.08934) | [PDF](https://arxiv.org/pdf/2003.08934v2) | [Project Page](https://www.matthewtancik.com/nerf) | [Code](https://github.com/bmild/nerf) | [Dataset](https://drive.google.com/drive/folders/1cK3UDIJqKAAm7zyrxRYVFJ0BRMgrwhh4)

<!-- PAPER-METADATA END -->

> 论文：NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis
> 来源：上传 arxiv_2003.08934v2.zip；同版本 PDF 25 页（正文、参考文献、附录 A–D），HTML 辅助。
> 阅读方式：Markdown-first 逐单元精读。图片为已验收 PDF 区域的 JPEG 展示副本，并非原件；客户端显示未验证。
> 证据规则：【原文事实】【背景知识】【合理推断】【独立评价】【尚未证实】分别标识。页码为 PDF 1-based。

<!-- PTH:BEGIN INIT -->
## 0. 初始化：原文范围与目录

**版本与来源**

【原文事实】NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis，Ben Mildenhall、Pratul P. Srinivasan、Matthew Tancik、Jonathan T. Barron、Ravi Ramamoorthi、Ren Ng。前三作者等贡献；单位为 UC Berkeley、Google Research、UC San Diego（PDF p.1）。采用 arXiv:2003.08934v2，2020-08-03；ECCV 2020（会议信息来自 B1 元数据核验）。上传 ZIP 内 PDF 和离线 HTML 同版本，PDF 权威。56 项原始文件完整性已复核；本报告只使用该版本及 B1 确认的元数据，不读取引用文献全文或源码，不执行 HTML 脚本。

**章节目录**

| 原章节 | PDF 页 | 作用 |
|---|---|---|
| Abstract | 1 | 任务与方法主张 |
| 1 Introduction | 1–3 | 任务、方法、三项贡献 |
| 2 Related Work | 3–4 | 神经隐式表示、光场、网格/体积表示 |
| 3 Neural Radiance Field Scene Representation | 4–5 | 5D 场、几何与方向颜色分支 |
| 4 Volume Rendering with Radiance Fields | 5–7 | 公式 (1)–(3)、光线积分与数值积分 |
| 5 Optimizing a Neural Radiance Field | 7–9 | 5.1 PE；5.2 层次采样；5.3 优化配置；公式 (4)–(6) |
| 6 Results | 9–14 | 6.1 数据；6.2 基线；6.3 讨论；6.4 消融 |
| 7 Conclusion | 14 | 总结、效率与可解释性 |
| Acknowledgements / References | 14–17 | 致谢及 52 条引用（仅作为本篇来源索引） |
| A Additional Implementation Details | 17–18 | 网络、边界、噪声、渲染耗时 |
| B Additional Baseline Method Details | 18–19 | 基线实现及分辨率 |
| C NDC ray space derivation | 19–21 | 公式 (7)–(26)，投影射线推导 |
| D Additional Results | 22–25 | 逐场景结果和 Fig. 8 |

**图表目录**

| 原编号 | 页 | 章节/范围 | 内容 |
|---|---:|---|---|
| Fig. 1 | 2 | §1 | 鼓组输入、优化、渲染 |
| Fig. 2 | 5 | §3–4 | 场查询、渲染和损失 |
| Fig. 3 | 6 | §3–4 | 两个位置的视角依赖辐射 |
| Fig. 4 | 7 | §5 | VD、PE 定性消融 |
| Fig. 5 | 11 | §6 | 复杂合成场景对比 |
| Fig. 6 | 12 | §6 | 真实前向场景对比 |
| Fig. 7 | 18 | App. A | MLP 结构 |
| Fig. 8 | 22 | App. D | 简单漫反射合成对比 |
| Table 1 | 10 | §6 | 三类数据集综合指标 |
| Table 2 | 14 | §6.4 | 九组消融 |
| Table 3 | 23 | App. D | DeepVoxels 逐场景 |
| Table 4 | 23 | App. D | 复杂合成逐场景 |
| Table 5 | 24 | App. D | 真实逐场景 |
| Table 6 | 25 | App. D | 逐场景消融 |

正文 7 个一级章节，正文 6 图 2 表；附录 4 节，2 图 4 表；总计 8 图 6 表，独立编号算法 0。编号连续。PDF 25 页均可读；HTML 缺 Figure 6 四张 T-Rex 图与七项装饰资源，PDF 中图内容完整。B2 区域 PDF 与 JPEG 均有 8 个；1 幅精确通过，7 幅实用保真带记录通过，Figure 2 有内联流覆盖限制；显示未验证。补充视频未提供，涉及视频的作者主张不独立确认。初始化仅整理资料，不适用方法机制或数学推导。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1. 全局认识

### 1.1 标题

**中文标题**

NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis —— NeRF：将场景表示为神经辐射场以进行视角合成（PDF p.1）。这里的 view synthesis 指从新相机视角生成同一个场景的图像，而非生成任意场景。

**术语解析**

- Neural：场是由神经网络参数表达的函数；此处不是把图像直接输入卷积网络来预测整张输出，也不意味着预训练泛化网络。
- Radiance field：辐射场。背景知识：辐射亮度描述某位置、方向的光辐射；本文用位置和方向映射到 RGB 颜色和体密度，RGB 并非完整光谱辐射测量。NeRF 的几何信息由位置相关的体密度隐式表达，颜色可以随方向变化；后文 §3 定义 FΘ。
- Representing scenes：表示的是一个连续三维场，而非只存储训练图片、显式三角网格或体素颜色。连续指可在连续坐标查询，不等于无限实际分辨率。
- View synthesis（新视角合成，NVS）：给定新的相机参数，计算每个像素对应的光线，对场进行采样和体渲染。

【合理推断】标题将表示与渲染联系起来：先得到能跨视图复用的空间场，再用它合成其他视图，而不是把新视图作为独立图像回归问题。

**标题边界**

标题只确定技术方向与目标；输入视角数、相机姿态是否已知、具体场函数、是否有深度监督、优化时间、是否支持动态场景，均需正文核查。不能凭标题断言恢复真实几何、材质、光照或实现任意场景泛化。标题单元不涉及具体数学推导，公式在阶段 3 展开。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
### 1.2 摘要逻辑拆解

**背景与困难**

【原文事实】p.1 的摘要开篇针对复杂场景新视角合成：从有限输入图像优化一个连续体积函数，并声称达到当时先进水平。摘要没有给出具体网格或表面方法的失败原因；§1–2 才阐述离散体积的存储与分辨率代价，以及早期连续隐式场的细节不足。

【背景知识】相机仅测量每条光线最终到达的 RGB，观测不直接提供沿该光线的三维颜色和密度分布；因此从少量视角重建一个可供其他视角渲染的场是逆问题。摘要中的 sparse 是相对密集光场采样而言，不等价于今日常说的 1–3 张图像；本文有 25/50/100 张消融及 479 张漫反射实验。

**方法链**

【原文事实】输入是一个连续 5D 坐标：3D 空间点 (x,y,z) 加 2D 方向 (θ,φ)。全连接、无卷积网络输出体密度和该位置随视角变化的辐射颜色。5D 是自变量自由度，方向在实现中以三分量单位向量表示；输出为 RGB 与标量密度。渲染时沿每个像素的相机光线查询多个坐标，利用经典体渲染把输出投影成二维图像。渲染可微，故将观测与合成像素之间的误差反传至场参数。摘要强调只需已知姿态的图像；§5.3 还要求内参和场景边界，合成用真值，真实用 COLMAP 估计。

【背景知识】这不是把单个输入 RGB 编码成一张预测图：训练阶段每个场景的全部观测共同优化共享函数；测试阶段使用优化好的场与新相机，逐射线计算输出。若每个视角都独立拟合一张图，就无法共享空间位置，因此缺少 NeRF 的多视角结构约束。

**结果与边界**

【原文事实】作者声称经过有效优化可产生具有复杂几何和外观的照片级新视图，并优于神经渲染和视角合成的前人方法。具体论证在 Fig. 5、6、8、Table 1–6；真实数据 LLFF 的 LPIPS 优于本文（Table 1），因此摘要的优于前人不是所有指标和所有场景无条件优势。作者建议看补充视频；当前未访问，不能确认时序一致性。摘要不展开具体推导，后文公式 (1)–(6) 及附录 C 才给出数学细节。

一句话概括：以已知相机的多视图 RGB 图像监督，逐场景优化位置与视角条件化的神经体积场，再用可微体渲染生成未见视图。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
### 1.3 作者列出的技术贡献

【原文事实】§1 p.2–3 确实有三条明确贡献，而不是我们事后拟定的列表。

**贡献一：场表示**

将复杂几何和材料的连续场景表示成由普通 MLP 参数化的 5D 神经辐射场。要解决的是早期隐式表面模型细节不足以及离散体积表示在高空间分辨率的存储负担。位置只决定密度；位置和方向共同决定颜色，使同一空间点可以有不同视角外观但不能随视角随意切换密度。证据：§3、Fig. 2/3/7，Table 1/2 的表现。属于方法表示贡献，不是首创 MLP、隐式场或光场概念。

**贡献二：可微渲染与采样**

将经典体渲染应用于连续神经场，使模型仅靠带相机参数的标准 RGB 图像即可训练。沿射线求积并使误差对密度和颜色可微；粗网络估计颜色累积权重，细网络在可见内容处增加样本，同时保留粗样本。该策略解决大量查询落在空处、遮挡后的浪费，而不是显式在训练前确定表面。证据：§4/5.2/5.3、公式 (1)–(3)、(5)–(6)、Table 2 第4行。贡献是渲染、场表示、采样和训练的组合；体渲染定律和 alpha 合成来自已有工作，非新物理定律。

**贡献三：位置编码**

固定的高频正弦/余弦映射将每个坐标提升到高维，缓解 MLP 直接输入低维坐标时对高频细节拟合不足的问题。对空间使用 L=10，方向 L=4；频带映射本身不可学习，网络学习映射后的函数。证据：§5.1 公式 (4)、Fig. 4、Table 2 第2/7/8行。位置编码在其他研究中已有类似思想，本文贡献是将其用于此连续场与图像监督问题。

**证据定位**

三项都是方法设计创新；实验贡献是三类数据集比较及九组消融，并新建复杂合成数据集。附录 C 提供标准投影到 NDC 的射线推导，属于技术实现推导，不声称证明新理论。p.3 作者提出截至所知首次实现真实场景高分辨率照片级连续神经表示；这是一项历史优先权主张，未查阅外部文献，不能独立确认。三项贡献共同作用，单因素消融支持组件有效性但不能将全方法收益严格分解。此贡献单元不逐步推导公式，数学细节在阶段 3。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
### 1.4 结论

**结论主张**

【原文事实】§7 p.14：本文针对先前用 MLP 表示连续物体和场景时的不足，表明用 5D 神经辐射场（位置与方向决定输出颜色及位置决定密度）可产生优于常见 CNN 输出离散体素表示的渲染结果。作者设想未来图形管线可由真实图片优化得到的多个神经辐射场构成复杂场景。

【独立评价】这一结论应限定在已知相机、静态场景、本文数据与基线条件下的图像合成效果。Table 1 中真实 LPIPS 例外说明不能用“全部指标更好”替代原有条件。图像更清晰并不等同于密度即真实物理密度或恢复了真实三维表面。

**局限与未来方向**

作者明确承认两项未解决问题：其一，即使分层采样已经提高了采样效率，优化及渲染效率还需进一步改善；其二，可解释性，显式网格和体素较容易分析渲染质量和失效模式，网络权重中的场尚不知如何有效分析。§5.3、App. A 给出单 V100 约 1–2 天训练、约 30 秒/帧；这不是实时方法。多场景组合是未来愿景而非本文已有验证。动态、可重光照、未知姿态联合优化也并未在本文验证。结论段不含新公式或独立数学证明。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2. 图表精读

### 2.1 功能总览

**Figure 功能**

| 图号（PDF 页） | 简称 | 功能 |
|---|---|---|
| 1 (2) | Drums 任务概览 | 输入多视图→单场优化→渲染新视角 |
| 2 (5) | 可微体渲染 | 展示5D输入、颜色密度输出、射线积分和监督 |
| 3 (6) | 方向辐射 | 固定空间点随方向改变颜色，示例船侧/水面 |
| 4 (7) | 消融 | 视角依赖和PE对细节的影响 |
| 5 (11) | 复杂合成 | 四个复杂几何/材质对象的局部细节对比 |
| 6 (12) | 真实前向 | 植物、骨架、遮挡与边缘对比 |
| 7 (18) | 网络 | 附录A精确维度、激活和skip连接 |
| 8 (22) | 漫反射合成 | 附录D简单物体与细纹理定性对比 |

**Table 功能**

| 表号（PDF 页） | 功能 |
|---|---|
| 1 (10) | 三数据集聚合PSNR/SSIM/LPIPS与各基线 |
| 2 (14) | 九组结构/频率/视图数消融 |
| 3 (23) | DeepVoxels逐场景（含原DeepVoxels方法） |
| 4 (23) | 复杂合成逐场景 |
| 5 (24) | 真实前向逐场景 |
| 6 (25) | 九组消融逐场景 |

【独立评价】这是功能导航，不代替每幅图的视觉观察或每张表的实验分析。功能总览不需要机制或公式推导。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
### 2.2 逐幅 Figure 精读

#### Figure 1：由多视图优化辐射场并合成新视图

![Figure 1：由多视图优化辐射场并合成新视图](NeRF/Figures/Figure_01.jpg)

来源：PDF p.2，区域坐标（pt，左上原点）[31.920000076293945, 29.760000228881836, 381.1199951171875, 116.16000366210938]；B2 验收 JPEG 展示副本（300 DPI/Q90/4:4:4），保真区域 PDF 为 `source/Figures_PDF/Figure_01.pdf`；原件不改动，客户端渲染未验证。

**Caption 原文**（PDF p.2）

Fig. 1: We present a method that optimizes a continuous 5D neural radiance field representation (volume density and view-dependent color at any continuous location) of a scene from a set of input images. We use techniques from volume rendering to accumulate samples of this scene representation along rays to render the scene from any viewpoint. Here, we visualize the set of 100 input views of the synthetic Drums scene randomly captured on a surrounding hemisphere, and we show two novel views rendered from our optimized NeRF representation.

**Caption 中文**

我们提出一种从一组输入图像优化场景的连续 5D 神经辐射场表示（在任意连续位置的体密度和视角依赖颜色）的方法。我们用体渲染技术沿射线累积该场景表示的样本，以从任意视点渲染场景。这里展示合成 Drums 场景在周围半球上随机采集的 100 个输入视图，并展示从优化后的 NeRF 表示渲染的两个新视图。

**布局与子图**

【视觉观察】左侧 Input Images 为鼓组的 10×10 缩略图阵列，中间 Optimize NeRF 示意为鼓组周围的半球相机视点，箭头从输入阵列指向场，再指向右侧两张不同相机视角的鼓组。两个输出的红色鼓身、金属支架、镲片外观一致但透视和可见部分变化。半球相机符号表示多视角观测，并非真正展示训练出的密度体积。

**正文与机制**

§1 p.2 给出三步：射线步进得到三维点、以点与视角查询 MLP、累积颜色和密度；对所有输入图像渲染误差优化同一场，测试时换相机。§6.1 合成复杂数据有 100 输入/200 测试，Fig.1 是 Drums 例子。背景知识：同一根支架可能从多个相机可见；若场把它错误放在某位置，来自其他视角的投影会错开，从而引入重投影误差。多视图提供这种耦合约束，但不保证唯一场。公式 (1) 给出每像素连续积分，式 (6) 给出训练误差。

**证据边界**

图展示任务与两个新视点，不能仅凭两张渲染结果验证所有方向、几何真值或少至 1–3 张的表现。“任意视点”是场可查询的功能，并不等同于任意外推视点的质量保证。图中不含数值曲线或公式推导；渲染及损失公式在后续单元。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
#### Figure 2：神经场查询、体渲染与像素监督

![Figure 2：神经场查询、体渲染与像素监督](NeRF/Figures/Figure_02.jpg)

来源：PDF p.5，区域坐标（pt，左上原点）[30.959999084472656, 28.799999237060547, 382.0799865722656, 139.1999969482422]；B2 验收 JPEG 展示副本（300 DPI/Q90/4:4:4），保真区域 PDF 为 `source/Figures_PDF/Figure_02.pdf`；原件不改动，客户端渲染未验证。

**Caption 原文**（PDF p.5）

Fig. 2: An overview of our neural radiance field scene representation and differentiable rendering procedure. We synthesize images by sampling 5D coordinates (location and viewing direction) along camera rays (a), feeding those locations into an MLP to produce a color and volume density (b), and using volume rendering techniques to composite these values into an image (c). This rendering function is differentiable, so we can optimize our scene representation by minimizing the residual between synthesized and ground truth observed images (d).

**Caption 中文**

我们的神经辐射场场景表示与可微渲染过程概览。我们通过沿相机射线采样 5D 坐标（位置和视角方向）（a），将这些位置输入 MLP 以获得颜色和体密度（b），再用体渲染技术将这些值合成为图像（c）来合成图像。该渲染函数可微，因此我们可以通过最小化合成图像与真实观测图像之间的残差（d）来优化场景表示。

**布局与子图**

【视觉观察】(a) 两个相机图像平面与一个虚线场景立方体，两条红色光线穿过黄色推土机，黑点表示采样位置；蓝箭头表示方向；位置与角度标成 $(x,y,z,\theta,\phi)$。中央蓝色 $F_\Theta$ 表示所有点查询共享网络，而非每条射线单独学习。(b) 原采样点变为带色小圆点，旁边标 RGBσ；密度不由小圆点颜色定量编码。(c) Ray 1 与 Ray 2 分别画 σ 对 Ray Distance 的示意曲线：上方主要一峰、下方两峰；高度与遮挡不是同一量，后峰要乘前段透射。曲线无数值刻度/单位，不能读出物理密度。(d) 棕、橙像素色块各和 g.t.（ground truth，真实观测像素）构成平方 L2 残差。蓝色曲线箭头表示合成后与真值比较，而非密度监督。

**正文与机制**

【原文事实】§3 p.5 $F_\Theta(\mathbf x,\mathbf d)=(\mathbf c,\sigma)$；§4 p.5–6 式 (1)/(3) 定义射线积分和离散权重。对第 i 点，$w_i=T_i(1-e^{-\sigma_i\delta_i})$，颜色是 $\sum_iw_i\mathbf c_i$。例如一前一后两个高密度峰，若前峰已近不透明，后峰虽大也几乎不可见。这里的 (c) 是密度分布，不是积分权重曲线或深度图。损失的梯度可穿过加权和及透射积回到每个 MLP 输出，最终更新共享参数。§5.3 式 (6) 实际对粗、细两个渲染都施加 RGB 残差。图没有画位置编码、层次采样及精细网络分支，完整实现见 Fig.7/§5。

**证据边界**

该图是算法解释，不是对密度峰准确性的实验验证；不能认为每条射线只有一个表面或两峰就是两个真实表面。图中没有定量曲线值、误差大小；这些图示仅支持信息流的理解。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
#### Figure 3：固定三维位置的方向辐射分布

![Figure 3：固定三维位置的方向辐射分布](NeRF/Figures/Figure_03.jpg)

来源：PDF p.6，区域坐标（pt，左上原点）[32.880001068115234, 37.91999816894531, 381.1199951171875, 131.0399932861328]；B2 验收 JPEG 展示副本（300 DPI/Q90/4:4:4），保真区域 PDF 为 `source/Figures_PDF/Figure_03.pdf`；原件不改动，客户端渲染未验证。

**Caption 原文**（PDF p.6）

Fig. 3: A visualization of view-dependent emitted radiance. Our neural radiance field representation outputs RGB color as a 5D function of both spatial position x and viewing direction d. Here, we visualize example directional color distributions for two spatial locations in our neural representation of the Ship scene. In (a) and (b), we show the appearance of two fixed 3D points from two different camera positions: one on the side of the ship (orange insets) and one on the surface of the water (blue insets). Our method predicts the changing specular appearance of these two 3D points, and in (c) we show how this behavior generalizes continuously across the whole hemisphere of viewing directions.

**Caption 中文**

视角依赖发射辐射的可视化。我们的神经辐射场表示将 RGB 颜色作为空间位置 x 和视角方向 d 的 5D 函数输出。这里展示 Ship 场景神经表示中的两个空间位置的方向颜色分布示例。在 (a) 和 (b) 中展示从两个不同相机位置观察两个固定 3D 点时的外观：一个位于船侧（橙色局部图），另一个位于水面（蓝色局部图）。我们的方法预测这两个 3D 点不断变化的镜面外观，(c) 展示该行为如何在整个半球的视角方向上连续泛化。

**布局与子图**

【视觉观察】(a) View 1 和 (b) View 2 为同一船模型不同方向。橙色圆框指向船侧，蓝青色框指向水面；船侧分别用菱形、方形标志对应不同观察方向，水面用三角形、圆形。船侧局部红棕色与亮斑不同，水面局部由暗绿至偏灰亮。右侧 (c) 两个圆盘分别是橙色船侧、蓝色水面的方向分布，内部由暗到亮的平滑变化，并以相同符号标记两次观察的方向。圆盘不是几何深度图、不是 RGB 空间坐标轴；无定量亮度刻度，不应读取具体颜色值。

**正文与机制**

§3 将 σ 限制为 σ(x)，但 c 为 c(x,d)。背景知识：漫反射在固定照明下外观可近似方向不变，镜面反射却依赖观察角度；若把颜色也限制为 c(x)，同一点无法同时解释不同视图中的高光，优化会平均外观或把错误密度分配给其他点。NeRF 的方向分支将位置特征和 γ(d) 拼接，可连续查询方向，空间密度不随视角变化。Fig.4 去除 VD 的镜面消融与此相呼应。方向圆盘展示学习函数对该位置的评估，而非独立测量的 BRDF（双向反射分布函数）。本图无独立数学推导，方向编码和渲染公式在后续展开。

**证据边界**

作者解释了两个点的镜面外观，图可展示方向连续变化的能力；不能证明整个半球每个方向的物理真实性，不提供入射光、法线、材料或 BRDF 真值。水面只是 Ship 合成物体一部分，本图不是参与介质水下传输模型。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
#### Figure 4：视角依赖与位置编码的定性消融

![Figure 4：视角依赖与位置编码的定性消融](NeRF/Figures/Figure_04.jpg)

来源：PDF p.7，区域坐标（pt，左上原点）[33.84000015258789, 29.760000228881836, 380.1600036621094, 129.1199951171875]；B2 验收 JPEG 展示副本（300 DPI/Q90/4:4:4），保真区域 PDF 为 `source/Figures_PDF/Figure_04.pdf`；原件不改动，客户端渲染未验证。

**Caption 原文**（PDF p.7）

Fig. 4: Here we visualize how our full model benefits from representing viewdependent emitted radiance and from passing our input coordinates through a high-frequency positional encoding. Removing view dependence prevents the model from recreating the specular reflection on the bulldozer tread. Removing the positional encoding drastically decreases the model’s ability to represent high frequency geometry and texture, resulting in an oversmoothed appearance.

**Caption 中文**

这里可视化完整模型如何受益于视角依赖发射辐射的表示，以及将输入坐标通过高频位置编码。去除视角依赖使模型无法重现推土机履带上的镜面反射；去除位置编码使模型表示高频几何和纹理的能力显著下降，导致过度平滑的外观。

**布局与子图**

【视觉观察】四列依次为 Ground Truth、Complete Model、No View Dependence、No Positional Encoding，均展示黄色推土机的同一局部。GT 的深色履带上有亮金属横片、孔状细节、黄梁边缘；完整模型基本保留轮廓、孔与高光；无 VD 履带上的白亮金属条变暗/消失，但部分孔和纹理仍在；无 PE 履带的孔和节块模糊成连续暗带，黄色结构边缘亦变圆钝。前后两类退化不同：方向不一致的外观丢失 vs 高频空间细节丢失。

**正文与机制**

§3 方向分支允许同一空间点表达随方向变化的颜色，§5.1 高频映射使 MLP 更易拟合快速空间变化；Fig.4 用局部视觉显示两个独立设计的作用。Table 2 第2行无 PE 28.77 dB、0.924 SSIM、0.108 LPIPS；第3行无 VD 27.66、0.925、0.117；完整 31.01、0.947、0.081。相对完整模型，PSNR 分别低 2.24 与 3.35 dB。Table 6 进一步展示不同场景对组件敏感程度。PE 与 VD 的作用并非严格可加：PE 同时用于方向和位置，移除 PE 也影响方向表达。

**证据边界**

图只有一个选定局部，不足以对全部场景做因果或平均性能推断；量化依赖 Table 2/6 的受控设定。无 PE 的几何与纹理模糊不能单独量化真实几何误差。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
#### Figure 5：复杂合成场景局部渲染对比

![Figure 5：复杂合成场景局部渲染对比](NeRF/Figures/Figure_05.jpg)

来源：PDF p.11，区域坐标（pt，左上原点）[31.920000076293945, 32.880001068115234, 381.1199951171875, 465.1199951171875]；B2 验收 JPEG 展示副本（300 DPI/Q90/4:4:4），保真区域 PDF 为 `source/Figures_PDF/Figure_05.pdf`；原件不改动，客户端渲染未验证。

**Caption 原文**（PDF p.11）

Fig. 5: Comparisons on test-set views for scenes from our new synthetic dataset generated with a physically-based renderer. Our method is able to recover fine details in both geometry and appearance, such as Ship’s rigging, Lego’s gear and treads, Microphone’s shiny stand and mesh grille, and Material’s nonLambertian reflectance. LLFF exhibits banding artifacts on the Microphone stand and Material’s object edges and ghosting artifacts in Ship’s mast and inside the Lego object. SRN produces blurry and distorted renderings in every case. Neural Volumes cannot capture the details on the Microphone’s grille or Lego’s gears, and it completely fails to recover the geometry of Ship’s rigging.

**Caption 中文**

我们新建的、使用基于物理的渲染器生成的合成数据集测试视图对比。我们的方法能够恢复几何和外观中的细节，例如 Ship 的索具、Lego 的齿轮和履带、Microphone 的亮面支架和网状格栅，以及 Materials 的非朗伯反射。LLFF 在 Microphone 支架与 Materials 物体边缘出现条带伪影，在 Ship 桅杆与 Lego 物体内部出现重影。SRN 在每种情况下都产生模糊且扭曲的渲染。Neural Volumes 无法捕捉 Microphone 格栅或 Lego 齿轮上的细节，并完全无法恢复 Ship 索具的几何。

**布局与子图**

【视觉观察】左列是四个完整物体参考图，右侧每物体两个局部裁剪，列序 GT、NeRF、LLFF、SRN、NV。逐组：

- Ship：上方桅杆、帆桁与细线，下方船舷索网。GT 有细且规则的索具；NeRF 较完整，LLFF 桅杆重影、下部索网局部混杂；SRN 两处都近似色团，NV 桅杆轮廓重复模糊、细绳大幅缺失。
- Lego：上方齿轮和黑色斜杆及黄色孔结构，下方车轮/齿轮及履带。NeRF 的孔、齿、边缘较清晰；LLFF 有重叠边和局部颜色泄漏，SRN 齿轮模糊，NV 轮廓可见但齿轮细节退化。
- Microphone：上方高光支架和交叉杆件，下方金属格栅。NeRF 保持支架明暗和密集孔纹；LLFF 支架边出现不规则带状、格栅局部重影；SRN 支架变形且格栅几乎光滑；NV 的格栅明显模糊。
- Materials：红色球形材质和灰色球形材质分别为两裁剪，带有弯曲的凸起纹样和镜面外观。NeRF 近似 GT 的高光与轮廓，LLFF 边缘多重/条带、SRN 光滑化、NV 模糊且有斑驳；这些是所示视图的观察，不是材质参数恢复结果。

**正文与机制**

§6.1 复杂合成数据含 8 个物体，Blender Cycles 路径追踪生成，100 输入、200 测试，800×800；Fig.5 展示其中四个。§6.2–6.3：SRN 单一表面/每射线单颜色、NV 128³ 显式体素受空间分辨率限制、LLFF 多 MPI 融合在输入视差大时可能估计错误几何。作者认为这些是伪影来源；图本身不能隔离全部因素。NeRF 的连续场、方向颜色和 PE 是组合方案。Table 4 中 Ship PSNR NeRF 28.65 vs NV 23.93；Microphone（Mic）32.91 vs LLFF 27.48；Materials 29.62 vs NV 24.22；Lego 32.54 vs NV 26.08。四个局部定性优势与这些完整图指标同向，但每幅图并无物体几何真值指标。

**证据边界**

图是作者选取的测试局部，并非全数据无偏采样；看见索网较清晰支持渲染细节而非严格几何准确性。App. B 的 SRN 比较分辨率较低，亦可影响清晰度。视频一致性未核验。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
#### Figure 6：真实前向场景的细结构与遮挡区域

![Figure 6：真实前向场景的细结构与遮挡区域](NeRF/Figures/Figure_06.jpg)

来源：PDF p.12，区域坐标（pt，左上原点）[30.959999084472656, 45.84000015258789, 382.0799865722656, 439.20001220703125]；B2 验收 JPEG 展示副本（300 DPI/Q90/4:4:4），保真区域 PDF 为 `source/Figures_PDF/Figure_06.pdf`；原件不改动，客户端渲染未验证。

**Caption 原文**（PDF p.12）

Fig. 6: Comparisons on test-set views of real world scenes. LLFF is specifically designed for this use case (forward-facing captures of real scenes). Our method is able to represent fine geometry more consistently across rendered views than LLFF, as shown in Fern’s leaves and the skeleton ribs and railing in T-rex. Our method also correctly reconstructs partially occluded regions that LLFF struggles to render cleanly, such as the yellow shelves behind the leaves in the bottom Fern crop and green leaves in the background of the bottom Orchid crop. Blending between multiples renderings can also cause repeated edges in LLFF, as seen in the top Orchid crop. SRN captures the low-frequency geometry and color variation in each scene but is unable to reproduce any fine detail.

**Caption 中文**

真实场景测试视图对比。LLFF 特别针对这一使用场景设计（真实场景的前向采集）。我们的方法比 LLFF 更能跨渲染视图一致地表示精细几何，如 Fern 的叶片及 T-Rex 的骨架肋骨和栏杆。我们的方法还正确重建了 LLFF 难以干净渲染的部分遮挡区域，例如 Fern 下方裁剪中叶片后的黄色搁板和 Orchid 下方裁剪中背景的绿色叶片。在多次渲染之间混合也可能使 LLFF 出现重复边缘，如 Orchid 上方裁剪。SRN 捕捉了各场景的低频几何和颜色变化，却不能再现任何精细细节。

**布局与子图**

【视觉观察】左侧 Fern、T-Rex、Orchid 三个完整场景示例，右侧各两个局部，四列 GT、NeRF、LLFF、SRN（无 NV）。Fern 上图细长叶片与墙上图案、下图交叠浅绿叶片及后方黄架；NeRF 叶片排列和空隙较清楚，LLFF 有叶片重叠边与搁板混合，SRN 基本只有色块。T-Rex 上图弧状肋骨、下图斜交金属栏杆与背景，NeRF 保持较连贯的细线，LLFF 有重合/混合边，SRN 丢失多数细节。Orchid 上图粉色花瓣边与绿叶，下图多个花瓣后方的交错绿叶，NeRF 的外轮廓和背景叶比 LLFF 更接近 GT，LLFF 在花瓣轮廓/遮挡后叶出现重复边，SRN 颜色轮廓粗略且模糊。B1 HTML 缺 T-Rex 四张图片已通过 PDF 区域保留，此处实际查看的是完整 B2 展示图。

**正文与机制**

§6.1 真实数据 8 个前向场景、20–62 张观测、留出 1/8 测试；LLFF 使用逐输入视图 MPI（多平面图像）及融合，作者认为融合可导致不一致；NeRF 同一共享三维场约束跨视图。部分遮挡点可由其他输入视图提供信息并经渲染的透射权重参与合成；在所有输入都不可见的区域则没有直接颜色约束。Table 5 Fern PSNR 25.17 vs LLFF 22.85，T-Rex 26.80 vs 24.15，Orchids 20.36 vs 18.52；对应 LPIPS 分别 0.280 vs 0.247、0.249 vs 0.222、0.321 vs 0.313，NeRF 反而较差。局部视觉细节与整体感知指标并非同义。

**证据边界**

真实场景没有真实几何和无遮挡的完整 GT；GT 指测试图。作者称跨视图一致性更好需要视频或多视角定量指标；本材料只有静态比较，视频缺失，不能独立核验。NV 因有界物体假设不适用，不能将空白结果当零分。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
#### Figure 7：全连接神经辐射场结构

![Figure 7：全连接神经辐射场结构](NeRF/Figures/Figure_07.jpg)

来源：PDF p.18，区域坐标（pt，左上原点）[63.84000015258789, 29.760000228881836, 350.1600036621094, 149.0399932861328]；B2 验收 JPEG 展示副本（300 DPI/Q90/4:4:4），保真区域 PDF 为 `source/Figures_PDF/Figure_07.pdf`；原件不改动，客户端渲染未验证。

**Caption 原文**（PDF p.18）

Fig. 7: A visualization of our fully-connected network architecture. Input vectors are shown in green, intermediate hidden layers are shown in blue, output vectors are shown in red, and the number inside each block signifies the vector’s dimension. All layers are standard fully-connected layers, black arrows indicate layers with ReLU activations, orange arrows indicate layers with no activation, dashed black arrows indicate layers with sigmoid activation, and “+” denotes vector concatenation. The positional encoding of the input location (γ(x)) is passed through 8 fully-connected ReLU layers, each with 256 channels. We follow the DeepSDF [32] architecture and include a skip connection that concatenates this input to the fifth layer’s activation. An additional layer outputs the volume density σ (which is rectified using a ReLU to ensure that the output volume density is nonnegative) and a 256-dimensional feature vector. This feature vector is concatenated with the positional encoding of the input viewing direction (γ(d)), and is processed by an additional fully-connected ReLU layer with 128 channels. A final layer (with a sigmoid activation) outputs the emitted RGB radiance at position x, as viewed by a ray with direction d.

**Caption 中文**

我们的全连接网络结构可视化。绿色表示输入向量，蓝色表示中间隐层，红色表示输出向量，块内数字表示向量维数。所有层都是标准全连接层；黑色箭头表示 ReLU 激活层，橙色箭头表示无激活层，虚线黑色箭头表示 sigmoid 激活层，“+”表示向量拼接。输入位置的编码 γ(x) 通过 8 个全连接 ReLU 层，每层 256 通道。我们沿用 DeepSDF [32] 结构，加入一个跳跃连接，将输入拼接到第 5 层的激活。附加层输出体密度 σ（用 ReLU 矫正以确保非负）和 256 维特征向量。该特征向量与输入视角方向的编码 γ(d) 拼接，经过一个额外的 128 通道全连接 ReLU 层。最终层（sigmoid 激活）输出从方向 d 的射线观察位置 x 时发射的 RGB 辐射。

**布局与子图**

【视觉观察】从左至右绿色 γ(x) 60，连续八个蓝色 256 块（黑箭头），第五个块上方另一个绿色 γ(x) 60 与 “+” 拼接；最后的蓝色 256 块和红色 σ 由橙箭头关联，表示线性输出部分，密度另用 ReLU 保证非负；下方绿色 γ(d) 24 经 “+” 与 256 特征拼接，黑箭头进入 128 蓝块，虚线黑箭头连接红色 RGB。图中的第五层拼接按照题注为第五层激活之后拼接；不应根据其他代码把层号改写。按该拼接位置，下一全连接层输入为 $256+60=316$ 维，方向分支输入为 $256+24=280$ 维。输入60与24来自 Eq.4 的 $3\times2L$（L=10、4），图没有额外原始坐标通道。

**正文与机制**

§3 与 App. A 明确密度只由位置产生；位置特征先生成，再加入方向。单点查询得到 1 个非负 σ 和 3 个 [0,1] RGB 值。若射线固定方向，沿射线查询不同位置时 γ(d) 相同、γ(x) 不同；若两视角查询同一点，位置特征和 σ 相同而颜色允许不同。skip 拼接提供原始高频位置特征通路，避免中间层完全丢失信息（合理解释，不是作者单独实验）。按 Fig.7 结构表示为 $\sigma=\operatorname{ReLU}(g_\sigma(h_x))$，$\mathbf c=\operatorname{sigmoid}(g_c([h_x,\gamma(d)]))$，$h_x\in\mathbb R^{256}$；细化的参数张量见 METHOD-FIELD。

**证据边界**

架构图确认维度、激活与信息流，不直接证明视角一致几何或 skip 的收益；图未给各层初始化与精确代码实现。原论文只画 60/24 维，不根据外部实现补入原始坐标。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
#### Figure 8：简单漫反射合成数据的对比

![Figure 8：简单漫反射合成数据的对比](NeRF/Figures/Figure_08.jpg)

来源：PDF p.22，区域坐标（pt，左上原点）[38.880001068115234, 29.760000228881836, 381.1199951171875, 254.16000366210938]；B2 验收 JPEG 展示副本（300 DPI/Q90/4:4:4），保真区域 PDF 为 `source/Figures_PDF/Figure_08.pdf`；原件不改动，客户端渲染未验证。

**Caption 原文**（PDF p.22）

Fig. 8: Comparisons on test-set views for scenes from the DeepVoxels [41] synthetic dataset. The objects in this dataset have simple geometry and perfectly diffuse reflectance. Because of the large number of input images (479 views) and simplicity of the rendered objects, both our method and LLFF [28] perform nearly perfectly on this data. LLFF still occasionally presents artifacts when interpolating between its 3D volumes, as in the top inset for each object. SRN [42] and NV [24] do not have the representational power to render fine details.

**Caption 中文**

DeepVoxels [41] 合成数据集场景测试视图的比较。该数据集的物体具有简单几何与完全漫反射的反射特性。由于输入图像很多（479 个视图）且渲染物体简单，我们的方法与 LLFF 在该数据上的表现都近乎完美。LLFF 在其三维体积之间插值时仍偶尔出现伪影，如每个物体的上方局部图。SRN 和 NV 不具备渲染细节所需的表示能力。

**布局与子图**

【视觉观察】两组完整物体 Pedestal 和 Cube 左列，右侧五列 GT、NeRF、LLFF、SRN、NV，各两裁剪。Pedestal 上部为圆孔内壁和装饰花边，下部是凹凸纹样的细条；NeRF 保留上部边缘和细纹，LLFF 上部内壁有模糊/重复纹，SRN 和 NV 细花纹较模糊。Cube 上部为红绿两个相交面上的字母和棱线，下部为红面字母；GT/NeRF/LLFF 均有较清楚字符，LLFF 在上部边缘和部分字母有插值伪影，SRN 字符模糊且有水平条带，NV 字符模糊变形。

**正文与机制**

§6.1 与 App. D：4 个简单漫反射物体，479 输入、1000 测试、512×512；视点远多于复杂合成数据。密集观察使 LLFF 的局部插值更容易；NeRF 的优势此处更多体现在细纹/边缘。Table 3 Pedestal PSNR 41.44 vs LLFF 35.87，Cube 39.19 vs LLFF 32.58；LPIPS 0.024 vs 0.039、0.006 vs 0.064。Table 1 漫反射整体 PSNR 40.15、SSIM 0.991、LPIPS 0.023；即便都视觉较好，PSNR仍明显有差异。此数据主要验证简单物体高分辨率表达，不是镜面能力关键证据。

**证据边界**

输入视图数与场景复杂度同时变化，不能从 Fig.8 与 Fig.5 的对照单独归因于视图数或材质。图只选择两物体局部；作者“近乎完美”是定性描述，仍存在非零误差和 Chair SSIM 例外（Table 3）。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN TABLE-01 -->
### 2.3 逐张 Table 精读

#### Table 1：三类数据集的定量比较

**Caption 原文**（PDF p.10）

Table 1: Our method quantitatively outperforms prior work on datasets of both synthetic and real images. We report PSNR/SSIM (higher is better) and LPIPS [50] (lower is better). The DeepVoxels [41] dataset consists of 4 diffuse objects with simple geometry. Our realistic synthetic dataset consists of pathtraced renderings of 8 geometrically complex objects with complex non-Lambertian materials. The real dataset consists of handheld forward-facing captures of 8 realworld scenes (NV cannot be evaluated on this data because it only reconstructs objects inside a bounded volume). Though LLFF achieves slightly better LPIPS, we urge readers to view our supplementary video where our method achieves better multiview consistency and produces fewer artifacts than all baselines.

**Caption 中文**

我们的方法在合成和真实图像数据集上定量优于先前工作。我们报告 PSNR/SSIM（越高越好）和 LPIPS [50]（越低越好）。DeepVoxels [41] 数据集包含 4 个简单几何的漫反射物体。我们的真实感合成数据集包含 8 个具有复杂几何和复杂非朗伯材料的物体的路径追踪渲染。真实数据集包含手持相机前向采集的 8 个真实场景（NV 无法在这些数据上评价，因为它只能重建有界体积内的物体）。虽然 LLFF 的 LPIPS 略好，我们建议读者观看补充视频，其中我们的方法实现了更好的多视角一致性，并产生比所有基线更少的伪影。

**表格数据**

【原文事实】PSNR、SSIM 越高越好，LPIPS 越低越好；PSNR 单位 dB，SSIM 与 LPIPS 为无量纲指标（背景知识：PSNR 反映像素均方误差，SSIM 关注局部结构，LPIPS 使用深层感知特征距离，不能互相替代）。下表保留论文精度，“—”为未报告/不适用，不视为零。

| Method | Diffuse Synthetic 360° PSNR↑ (dB) | Diffuse Synthetic 360° SSIM↑ | Diffuse Synthetic 360° LPIPS↓ | Realistic Synthetic 360° PSNR↑ (dB) | Realistic Synthetic 360° SSIM↑ | Realistic Synthetic 360° LPIPS↓ | Real Forward-Facing PSNR↑ (dB) | Real Forward-Facing SSIM↑ | Real Forward-Facing LPIPS↓ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SRN [42] | 33.20 | 0.963 | 0.073 | 22.26 | 0.846 | 0.170 | 22.84 | 0.668 | 0.378 |
| NV [24] | 29.62 | 0.929 | 0.099 | 26.05 | 0.893 | 0.160 | — | — | — |
| LLFF [28] | 34.38 | 0.985 | 0.048 | 24.88 | 0.911 | 0.114 | 24.13 | 0.798 | 0.212 |
| Ours | 40.15 | 0.991 | 0.023 | 31.01 | 0.947 | 0.081 | 26.50 | 0.811 | 0.250 |

**实验分析**

【原文事实】三组分别有 4/8/8 场景。按每组与最优可比较基线比较：漫反射合成：NeRF 40.15 dB vs LLFF 34.38，差 +5.77 dB；SSIM 0.991 vs 0.985（+0.006），LPIPS 0.023 vs 0.048（-0.025）。复杂合成：PSNR 31.01 vs NV 26.05（+4.96 dB），SSIM 0.947 vs LLFF 0.911（+0.036），LPIPS 0.081 vs LLFF 0.114（-0.033）。真实前向：PSNR 26.50 vs LLFF 24.13（+2.37 dB），SSIM 0.811 vs LLFF 0.798（+0.013），LPIPS 0.250 vs LLFF 0.212（+0.038，NeRF 更差）。NV 在真实数据三项为空，不能参加该组排名。作者声称视频中更一致，是独立于这三个指标的主张。背景知识：同一峰值下 PSNR 提升 Δ dB 对应 MSE 比例 $10^{-\Delta/10}$，例如 +4.96 dB 对应约0.319倍；这里为聚合 PSNR，不能直接解释成全数据集像素总MSE比。

**证据边界**

§6.2 各基线使用相同输入集，LLFF 有预训练网络，SRN 在较低分辨率评估（App. B）。表不能证明逐场景所有指标都领先，也不提供几何真值误差、置信区间或视频一致性测量。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
#### Table 2：三设计组件、输入视图数、编码频率消融

**Caption 原文**（PDF p.14）

Table 2: An ablation study of our model. Metrics are averaged over the 8 scenes from our realistic synthetic dataset. See Sec. 6.4 for detailed descriptions.

**Caption 中文**

我们模型的消融研究。指标在真实感合成数据集的 8 个场景上取平均。详细描述见 §6.4。

**表格数据**

【原文事实】PSNR、SSIM 越高越好，LPIPS 越低越好；PSNR 单位 dB，SSIM 与 LPIPS 为无量纲指标（背景知识：PSNR 反映像素均方误差，SSIM 关注局部结构，LPIPS 使用深层感知特征距离，不能互相替代）。下表保留论文精度，“—”为未报告/不适用，不视为零。

| Setting | Input | #Im. | L | (Nc,Nf) | PSNR↑ (dB) | SSIM↑ | LPIPS↓ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1) No PE, VD, H | xyz | 100 | — | 256,— | 26.67 | 0.906 | 0.136 |
| 2) No Pos. Encoding | xyzθφ | 100 | — | 64,128 | 28.77 | 0.924 | 0.108 |
| 3) No View Dependence | xyz | 100 | 10 | 64,128 | 27.66 | 0.925 | 0.117 |
| 4) No Hierarchical | xyzθφ | 100 | 10 | 256,— | 30.06 | 0.938 | 0.109 |
| 5) Far Fewer Images | xyzθφ | 25 | 10 | 64,128 | 27.78 | 0.925 | 0.107 |
| 6) Fewer Images | xyzθφ | 50 | 10 | 64,128 | 29.79 | 0.940 | 0.096 |
| 7) Fewer Frequencies | xyzθφ | 100 | 5 | 64,128 | 30.59 | 0.944 | 0.088 |
| 8) More Frequencies | xyzθφ | 100 | 15 | 64,128 | 30.81 | 0.946 | 0.096 |
| 9) Complete Model | xyzθφ | 100 | 10 | 64,128 | 31.01 | 0.947 | 0.081 |

PE = positional encoding，VD = view dependence，H = hierarchical sampling；L 表示位置编码频带数，方向频带按比例调整；#Im. 为输入图像数；Nc/Nf 为粗采样/新增细采样数，而非细网络的总样本数。

**实验分析**

【原文事实】完整第9行作控制，三指标为31.01/0.947/0.081。

| 设置 | 完整减该行 PSNR (dB) | 完整减该行 SSIM | 该行减完整 LPIPS |
|---|---:|---:|---:|
| 1 三项皆无 | 4.34 | 0.041 | 0.055 |
| 2 无PE | 2.24 | 0.023 | 0.027 |
| 3 无VD | 3.35 | 0.022 | 0.036 |
| 4 无H | 0.95 | 0.009 | 0.028 |
| 5 25图 | 3.23 | 0.022 | 0.026 |
| 6 50图 | 1.22 | 0.007 | 0.015 |
| 7 L=5 | 0.42 | 0.003 | 0.007 |
| 8 L=15 | 0.20 | 0.001 | 0.015 |

1 与 9 对比是组合效果；1 同时改变输入维、编码与采样，无法拆单因果。2/3/4 分别去掉一个组件，在其余表列设置基本相同的条件下评估。无 VD 的 PSNR 降最大，无 PE 的 SSIM 降略大，无 H 的 LPIPS 退化0.028略大于无 PE 0.027，因此作者“PE、VD 收益最大，其次 H”是总体判断而非三个指标同序。4 的单网络256次查询 vs 完整64粗+192细共256查询，说明样本分配方式在同总查询次数下有益，但参数量/双网络结构和耗时未严格匹配。5/6/9 25→50→100视图，PSNR 27.78→29.79→31.01，SSIM 0.925→0.940→0.947，LPIPS 0.107→0.096→0.081；作者强调25图仍优于表1复杂合成基线（最高PSNR26.05，最高SSIM0.911，最低LPIPS0.114）。7/8/9 L=5/15/10 对比，L=10三指标最佳；增加频率不保证更好。

**证据边界**

这是八个特定场景、指定采样数和有限频带的消融，无随机种子重复、方差或训练/推理时间对照；视图子集的具体挑选方式未给出，不可概括成任意25视图。L=5/10/15结论不代表全场景频率最优。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
#### Table 3：DeepVoxels 逐场景比较

**Caption 原文**（PDF p.23）

Table 3: Per-scene quantitative results from the DeepVoxels [41] dataset. The “scenes” in this dataset are all diffuse objects with simple geometry, rendered from texture-mapped meshes captured by a 3D scanner. The metrics for the DeepVoxels method are taken directly from their paper, which does not report LPIPS and only reports two significant figures for SSIM.

**Caption 中文**

DeepVoxels [41] 数据集逐场景的定量结果。该数据集的“场景”都是具有简单几何的漫反射物体，由 3D 扫描器采集的纹理映射网格渲染而成。DeepVoxels 方法的指标直接取自其论文，未报告 LPIPS，且 SSIM 仅报告两位有效数字。

**表格数据**

【原文事实】PSNR、SSIM 越高越好，LPIPS 越低越好；PSNR 单位 dB，SSIM 与 LPIPS 为无量纲指标（背景知识：PSNR 反映像素均方误差，SSIM 关注局部结构，LPIPS 使用深层感知特征距离，不能互相替代）。下表保留论文精度，“—”为未报告/不适用，不视为零。

**PSNR↑ (dB)**

| Method | Chair | Pedestal | Cube | Vase |
| --- | --- | --- | --- | --- |
| DeepVoxels [41] | 33.45 | 32.35 | 28.42 | 27.99 |
| SRN [42] | 36.67 | 35.91 | 28.74 | 31.46 |
| NV [24] | 35.15 | 36.47 | 26.48 | 20.39 |
| LLFF [28] | 36.11 | 35.87 | 32.58 | 32.97 |
| Ours | 42.65 | 41.44 | 39.19 | 37.32 |

**SSIM↑**

| Method | Chair | Pedestal | Cube | Vase |
| --- | --- | --- | --- | --- |
| DeepVoxels [41] | 0.99 | 0.97 | 0.97 | 0.96 |
| SRN [42] | 0.982 | 0.957 | 0.944 | 0.969 |
| NV [24] | 0.980 | 0.963 | 0.916 | 0.857 |
| LLFF [28] | 0.992 | 0.983 | 0.983 | 0.983 |
| Ours | 0.991 | 0.986 | 0.996 | 0.992 |

**LPIPS↓**

| Method | Chair | Pedestal | Cube | Vase |
| --- | --- | --- | --- | --- |
| DeepVoxels [41] | — | — | — | — |
| SRN [42] | 0.093 | 0.081 | 0.074 | 0.044 |
| NV [24] | 0.096 | 0.069 | 0.113 | 0.117 |
| LLFF [28] | 0.051 | 0.039 | 0.064 | 0.039 |
| Ours | 0.047 | 0.024 | 0.006 | 0.017 |

**实验分析**

【原文事实】四场景PSNR全胜：Chair 42.65 vs 最好SRN36.67（+5.98），Pedestal41.44 vs NV36.47（+4.97），Cube39.19 vs LLFF32.58（+6.61），Vase37.32 vs LLFF32.97（+4.35）。SSIM Chair LLFF0.992略胜NeRF0.991，DeepVoxels的0.99仅两位有效数字，不能用该精度判断微小差距；其余三场景NeRF 0.986/0.996/0.992领先。LPIPS四场景NeRF均最低，分别0.047/0.024/0.006/0.017，DeepVoxels无此项。Cube 0.006 vs LLFF0.064（-0.058），支持 Fig.8 的字母纹理改善；Pedestal0.024 vs LLFF0.039（-0.015）。NV在Vase PSNR20.39、SSIM0.857低于其他方法，说明同一数据集也有场景差异。

**证据边界**

原DeepVoxels结果从其论文转录，非相同代码复跑；SSIM 精度不同。简单漫反射和479视图不证明少视角复杂材料。表无几何误差或方差。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
#### Table 4：真实感合成逐场景比较

**Caption 原文**（PDF p.23）

Table 4: Per-scene quantitative results from our realistic synthetic dataset. The “scenes” in this dataset are all objects with more complex gometry and nonLambertian materials, rendered using Blender’s Cycles pathtracer.

**Caption 中文**

我们真实感合成数据集的逐场景定量结果。该数据集的“场景”均为具有更复杂几何和非朗伯材料的物体，由 Blender 的 Cycles 路径追踪器渲染。

**表格数据**

【原文事实】PSNR、SSIM 越高越好，LPIPS 越低越好；PSNR 单位 dB，SSIM 与 LPIPS 为无量纲指标（背景知识：PSNR 反映像素均方误差，SSIM 关注局部结构，LPIPS 使用深层感知特征距离，不能互相替代）。下表保留论文精度，“—”为未报告/不适用，不视为零。

**PSNR↑ (dB)**

| Method | Chair | Drums | Ficus | Hotdog | Lego | Materials | Mic | Ship |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SRN [42] | 26.96 | 17.18 | 20.73 | 26.81 | 20.85 | 18.09 | 26.85 | 20.60 |
| NV [24] | 28.33 | 22.58 | 24.79 | 30.71 | 26.08 | 24.22 | 27.78 | 23.93 |
| LLFF [28] | 28.72 | 21.13 | 21.79 | 31.41 | 24.54 | 20.72 | 27.48 | 23.22 |
| Ours | 33.00 | 25.01 | 30.13 | 36.18 | 32.54 | 29.62 | 32.91 | 28.65 |

**SSIM↑**

| Method | Chair | Drums | Ficus | Hotdog | Lego | Materials | Mic | Ship |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SRN [42] | 0.910 | 0.766 | 0.849 | 0.923 | 0.809 | 0.808 | 0.947 | 0.757 |
| NV [24] | 0.916 | 0.873 | 0.910 | 0.944 | 0.880 | 0.888 | 0.946 | 0.784 |
| LLFF [28] | 0.948 | 0.890 | 0.896 | 0.965 | 0.911 | 0.890 | 0.964 | 0.823 |
| Ours | 0.967 | 0.925 | 0.964 | 0.974 | 0.961 | 0.949 | 0.980 | 0.856 |

**LPIPS↓**

| Method | Chair | Drums | Ficus | Hotdog | Lego | Materials | Mic | Ship |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SRN [42] | 0.106 | 0.267 | 0.149 | 0.100 | 0.200 | 0.174 | 0.063 | 0.299 |
| NV [24] | 0.109 | 0.214 | 0.162 | 0.109 | 0.175 | 0.130 | 0.107 | 0.276 |
| LLFF [28] | 0.064 | 0.126 | 0.130 | 0.061 | 0.110 | 0.117 | 0.084 | 0.218 |
| Ours | 0.046 | 0.091 | 0.044 | 0.121 | 0.050 | 0.063 | 0.028 | 0.206 |

**实验分析**

【原文事实】PSNR/SSIM八场景NeRF全胜。与每场景PSNR最佳基线逐列比较：Chair 33.00−28.72=4.28；Drums25.01−22.58=2.43；Ficus30.13−24.79=5.34；Hotdog36.18−31.41=4.77；Lego32.54−26.08=6.46；Materials29.62−24.22=5.40；Mic32.91−27.78=5.13；Ship28.65−23.93=4.72 dB。Ship 的SSIM 0.856低于本方法其余七场景，但仍超过最优基线LLFF0.823，复杂细索仍有难度。LPIPS 七场景最优，Hotdog例外：NeRF0.121，高于LLFF0.061、SRN0.100、NV0.109。注意 Hotdog 同时有最高PSNR36.18/SSIM0.974并不保证感知指标最优。Materials PSNR +5.40与方向外观能力一致，但不能只归因于VD，需消融 Table 6。

**证据边界**

同一套RGB指标只衡量渲染，不能量化真值网格/反射参数。与SRN的图像分辨率差异和场景级训练成本需并列看待。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
#### Table 5：真实前向图像逐场景比较

**Caption 原文**（PDF p.24）

Table 5: Per-scene quantitative results from our real image dataset. The scenes in this dataset are all captured with a forward-facing handheld cellphone.

**Caption 中文**

我们真实图像数据集的逐场景定量结果。该数据集的场景全部用手持手机向前拍摄。

**表格数据**

【原文事实】PSNR、SSIM 越高越好，LPIPS 越低越好；PSNR 单位 dB，SSIM 与 LPIPS 为无量纲指标（背景知识：PSNR 反映像素均方误差，SSIM 关注局部结构，LPIPS 使用深层感知特征距离，不能互相替代）。下表保留论文精度，“—”为未报告/不适用，不视为零。

**PSNR↑ (dB)**

| Method | Room | Fern | Leaves | Fortress | Orchids | Flower | T-Rex | Horns |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SRN [42] | 27.29 | 21.37 | 18.24 | 26.63 | 17.37 | 24.63 | 22.87 | 24.33 |
| LLFF [28] | 28.42 | 22.85 | 19.52 | 29.40 | 18.52 | 25.46 | 24.15 | 24.70 |
| Ours | 32.70 | 25.17 | 20.92 | 31.16 | 20.36 | 27.40 | 26.80 | 27.45 |

**SSIM↑**

| Method | Room | Fern | Leaves | Fortress | Orchids | Flower | T-Rex | Horns |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SRN [42] | 0.883 | 0.611 | 0.520 | 0.641 | 0.449 | 0.738 | 0.761 | 0.742 |
| LLFF [28] | 0.932 | 0.753 | 0.697 | 0.872 | 0.588 | 0.844 | 0.857 | 0.840 |
| Ours | 0.948 | 0.792 | 0.690 | 0.881 | 0.641 | 0.827 | 0.880 | 0.828 |

**LPIPS↓**

| Method | Room | Fern | Leaves | Fortress | Orchids | Flower | T-Rex | Horns |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SRN [42] | 0.240 | 0.459 | 0.440 | 0.453 | 0.467 | 0.288 | 0.298 | 0.376 |
| LLFF [28] | 0.155 | 0.247 | 0.216 | 0.173 | 0.313 | 0.174 | 0.222 | 0.193 |
| Ours | 0.178 | 0.280 | 0.316 | 0.171 | 0.321 | 0.219 | 0.249 | 0.268 |

**实验分析**

【原文事实】对 LLFF（PSNR 每列均为最强基线）逐场景 PSNR差值 NeRF−LLFF：Room4.28、Fern2.32、Leaves1.40、Fortress1.76、Orchids1.84、Flower1.94、T-Rex2.65、Horns2.75 dB。NeRF 的 SSIM 5/8 最优，3个例外 Leaves 0.690 vs0.697，Flower0.827 vs0.844，Horns0.828 vs0.840。LPIPS 仅 Fortress 的0.171略优于LLFF0.173，其余7个LLFF更优，差值 NeRF−LLFF 依次为Room0.023、Fern0.033、Leaves0.100、Fortress−0.002、Orchids0.008、Flower0.045、T-Rex0.027、Horns0.075。NeRF 对 SRN 所有列三指标均优。Leaves 的 LPIPS差最大，提示繁密叶片的感知难点，不足以单独定位原因。Fig.6 中局部清晰度并不能替代感知指标。

**证据边界**

附录D文字“quantitatively outperforms all baselines”需按表限缩，不能无条件使用。LPIPS例外在七场景，非仅单个异常；作者视频一致性主张未由本表检验。NV未评估。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
#### Table 6：逐场景消融的完整数据

**Caption 原文**（PDF p.25）

Table 6: Per-scene quantitative results from our ablation study. The scenes used here are the same as in Table 4.

**Caption 中文**

我们消融研究的逐场景定量结果。这里使用的场景与 Table 4 相同。

**表格数据**

【原文事实】PSNR、SSIM 越高越好，LPIPS 越低越好；PSNR 单位 dB，SSIM 与 LPIPS 为无量纲指标（背景知识：PSNR 反映像素均方误差，SSIM 关注局部结构，LPIPS 使用深层感知特征距离，不能互相替代）。下表保留论文精度，“—”为未报告/不适用，不视为零。

**PSNR↑ (dB)**

| Method | Chair | Drums | Ficus | Hotdog | Lego | Materials | Mic | Ship |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1) No PE, VD, H | 28.44 | 23.11 | 25.17 | 32.24 | 26.38 | 24.69 | 28.16 | 25.12 |
| 2) No Pos. Encoding | 30.33 | 24.54 | 29.32 | 33.16 | 27.75 | 27.79 | 30.76 | 26.55 |
| 3) No View Dependence | 30.06 | 23.41 | 25.91 | 32.65 | 29.93 | 24.96 | 28.62 | 25.72 |
| 4) No Hierarchical | 31.32 | 24.55 | 29.25 | 35.24 | 31.42 | 29.22 | 31.74 | 27.73 |
| 5) Far Fewer Images | 30.92 | 22.62 | 24.39 | 32.77 | 27.97 | 26.55 | 30.47 | 26.57 |
| 6) Fewer Images | 32.19 | 23.70 | 27.45 | 34.91 | 31.53 | 28.54 | 32.33 | 27.67 |
| 7) Fewer Frequencies | 32.19 | 25.29 | 30.73 | 36.06 | 30.77 | 29.77 | 31.66 | 28.26 |
| 8) More Frequencies | 32.87 | 24.65 | 29.92 | 35.78 | 32.50 | 29.54 | 32.86 | 28.34 |
| 9) Complete Model | 33.00 | 25.01 | 30.13 | 36.18 | 32.54 | 29.62 | 32.91 | 28.65 |

**SSIM↑**

| Method | Chair | Drums | Ficus | Hotdog | Lego | Materials | Mic | Ship |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1) No PE, VD, H | 0.919 | 0.896 | 0.926 | 0.955 | 0.882 | 0.905 | 0.955 | 0.810 |
| 2) No Pos. Encoding | 0.938 | 0.918 | 0.953 | 0.956 | 0.903 | 0.933 | 0.968 | 0.824 |
| 3) No View Dependence | 0.948 | 0.906 | 0.938 | 0.961 | 0.947 | 0.912 | 0.962 | 0.828 |
| 4) No Hierarchical | 0.951 | 0.914 | 0.956 | 0.969 | 0.951 | 0.944 | 0.973 | 0.844 |
| 5) Far Fewer Images | 0.956 | 0.895 | 0.922 | 0.966 | 0.930 | 0.925 | 0.972 | 0.832 |
| 6) Fewer Images | 0.963 | 0.911 | 0.948 | 0.971 | 0.957 | 0.941 | 0.979 | 0.847 |
| 7) Fewer Frequencies | 0.959 | 0.928 | 0.965 | 0.972 | 0.947 | 0.952 | 0.973 | 0.853 |
| 8) More Frequencies | 0.967 | 0.921 | 0.962 | 0.973 | 0.961 | 0.948 | 0.980 | 0.853 |
| 9) Complete Model | 0.967 | 0.925 | 0.964 | 0.974 | 0.961 | 0.949 | 0.980 | 0.856 |

**LPIPS↓**

| Method | Chair | Drums | Ficus | Hotdog | Lego | Materials | Mic | Ship |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1) No PE, VD, H | 0.095 | 0.168 | 0.084 | 0.104 | 0.178 | 0.111 | 0.084 | 0.261 |
| 2) No Pos. Encoding | 0.076 | 0.104 | 0.050 | 0.124 | 0.128 | 0.079 | 0.041 | 0.261 |
| 3) No View Dependence | 0.075 | 0.148 | 0.113 | 0.112 | 0.088 | 0.102 | 0.073 | 0.220 |
| 4) No Hierarchical | 0.065 | 0.177 | 0.056 | 0.130 | 0.072 | 0.080 | 0.039 | 0.249 |
| 5) Far Fewer Images | 0.058 | 0.173 | 0.082 | 0.123 | 0.081 | 0.079 | 0.035 | 0.229 |
| 6) Fewer Images | 0.051 | 0.166 | 0.057 | 0.121 | 0.055 | 0.068 | 0.029 | 0.223 |
| 7) Fewer Frequencies | 0.055 | 0.143 | 0.038 | 0.087 | 0.071 | 0.060 | 0.029 | 0.219 |
| 8) More Frequencies | 0.047 | 0.158 | 0.045 | 0.116 | 0.050 | 0.064 | 0.027 | 0.261 |
| 9) Complete Model | 0.046 | 0.091 | 0.044 | 0.121 | 0.050 | 0.063 | 0.028 | 0.206 |

**实验分析**

【原文事实】第9行与 Table 4 Ours 完全相同。相对9行：去 PE 对 Lego PSNR 32.54→27.75（-4.79），Ficus仅30.13→29.32（-0.81）；去VD对 Materials 29.62→24.96（-4.66）、Ficus30.13→25.91（-4.22）尤甚，Hotdog36.18→32.65（-3.53），说明场景敏感性不同，不能把均值当每场景同幅收益。去H的Drums LPIPS 0.091→0.177（恶化0.086），其他如Ficus0.044→0.056（恶化0.012）。25图相对100图Ficus30.13→24.39（-5.74），Mic32.91→30.47（-2.44）；50图Ficus27.45，比25图高3.06仍低于100图2.68。减少频带(L=5)反而在 Drums PSNR25.29 vs25.01、Ficus30.73 vs30.13、Materials29.77 vs29.62 胜过完整；SSIM Drums0.928 vs0.925、Ficus0.965 vs0.964、Materials0.952 vs0.949更高。Ficus LPIPS0.038 vs0.044、Hotdog0.087 vs0.121、Materials0.060 vs0.063也更低。L=15 Mic LPIPS0.027 vs0.028略优；Hotdog L=5的LPIPS低于L=10/15，且最简设置0.104也优于完整0.121。LPIPS的个别场景/行优势并不等同于组件无效。Lego 完整与L=15 LPIPS同为0.050；Chair SSIM完整与L=15同为0.967、Lego SSIM同为0.961、Mic SSIM同为0.980。平均最优不等于每列最优。

**证据边界**

表中没有逐场景训练时间、重复种子或频率与视图数交互的全因子实验。频率最优、少视角退化及真实材料的原因不能仅凭单场景数值确认。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
### 2.4 图表间的逻辑关系

**方法链关系**

Fig.1 定义任务层面（输入、场优化、新视图）；Fig.2 展开每条射线上的局部计算（坐标→颜色/密度→积分→像素损失）。Fig.7 将 Fig.2 中抽象的 MLP 展开为位置主干、密度输出、方向分支和 RGB 输出。Fig.3 专门显示方向颜色能力，Fig.4 以受控去除 VD/PE 的定性结果说明方向输入与位置编码的不同作用。数学链条：公式 (1) 是连续投影，(2) 给随机位置，(3) 给可微离散求积；(4) 细化 Fig.2 中输入的编码，(5) 利用粗合成权重重采样，(6) 细化 Fig.2 中渲染损失为粗细两路。Fig.7 本身没有画粗细两套网络，§5.2 说明它们同时训练。附录 C 的公式 (7)–(26) 对真实前向射线提供坐标变换，图2 的示意不能直接看出 NDC。

**实验链关系**

| 证据链 | 对应内容 | 能回答什么 |
|---|---|---|
| Fig.8 + Table 1 第一组 + Table 3 | 简单漫反射 | 视图较密时细纹理与合成指标的差异 |
| Fig.5 + Table 1 第二组 + Table 4 | 复杂合成 | 复杂几何、非朗伯外观；逐场景反例 |
| Fig.6 + Table 1 第三组 + Table 5 | 真实前向 | 遮挡、细结构与真实指标；LPIPS例外 |
| Fig.4 + Table 2 + Table 6 | 组件消融 | VD、PE，H、视图数、频带控制 |
| Fig.3 + Fig.4 + Table 2/6 | 方向颜色 | 同一位置可查询多方向；无VD结果退化 |

【独立评价】前两类示意图建立方法的可计算性，消融检验设计有用性，基线比较检验整体表现。量化总表与逐场景表必须共同解读：均值可领先而某一场景某一指标落后。视频一致性属于另一维度，原材料未包含；Fig.6 的静态裁剪不能代替视频或几何真值评估。图表关系没有新增公式推导，已有公式在后续独立推导。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 3. 正文逐层精读

### 3.1 研究问题

**任务定义**

【原文事实】§1、§3、§5.3：在静态场景中，输入若干 RGB 图像及其相机姿态、内参和场景范围，优化一个场景专属的神经辐射场；输出一是网络权重表达的连续场，二是由给定新相机渲染的 RGB 图像。训练时观测有真值像素，测试时目标相机给定而目标RGB不作为输入。作者不需要真值3D几何或深度图监督；真实相机及范围由 COLMAP 估计。主要目标是新视角图像质量，而非直接输出网格/深度或恢复材质参数。

【背景知识/形式化】设训练集合 $\mathcal D=\{(I_k,K_k,R_k,\mathbf t_k)\}_{k=1}^M$，$K$ 为内参，$R,\mathbf t$ 为姿态（下式仅用通用相机表示，本文未指定该符号）。对像素 $u$，相机确定射线 $\mathbf r_{k,u}(t)$。定义渲染算子 $\mathcal V$，学习

$\Theta^*=\arg\min_\Theta\sum_{k,u}\|\mathcal V(F_\Theta,\mathbf r_{k,u})-I_k(u)\|_2^2$。

这是便于理解的单网络形式；原 Eq.6 使用粗细两个场的平方误差。对新相机 $k'$，以相同算子逐像素计算 $I_{k'}$，无需目标真值。该任务不是训练一个将 $M$ 张图直接输入、一次前馈就输出新场景的网络。

**逆问题**

每个像素仅有 3 个观测分量，而沿一条射线的连续函数 $\sigma(t),\mathbf c(t,\mathbf d)$ 有大量自由度。以两个段为例，若目标颜色 $C=0.5$（单通道）可由一个完全不透明段的颜色0.5表示，也可由 alpha=0.5且颜色1的段在零背景下表示；图像相同、密度不同。相机之间共享位置相关 σ 以及 MLP 参数可使同一空间点从不同视角的投影耦合，多观测约束降低歧义，但不存在仅凭损失保证几何唯一的证明。方向颜色提供非朗伯自由度，也可能吸收不正确几何带来的外观误差。§1“鼓励 coherent model”是设计目标，不是可辨识性定理。

**适用条件**

- 静态场景、相机参数与边界可获得；合成的有界物体用归一化立方体，真实数据是前向拍摄用 NDC 约束空间。
- 观测稀疏程度需相对于视差和场景结构讨论。复杂合成以100训练图为默认，消融最少25张，真实采集20–62张再留出1/8；并未建立极少观测/严重无重叠情形的结果。
- 【独立评价】未知相机、动态/光照变化、大范围无观测外推、真实几何及物理光学分离均不在已验证任务范围。新视角合成准确性与三维重建正确性须分别评价。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
### 3.2 研究背景与动机

**表示矛盾**

【原文事实】§1–2：显式体素便于体渲染和梯度优化，但高分辨率复杂场景要细化三维网格，存储与时间代价增长；网格需拓扑初始化，图像重投影优化可能局部极小/病态；早期隐式形状表示连续而紧凑，但截至作者讨论时复杂几何/外观渲染常过平滑。SRN用单一表面和颜色解释射线，复杂半透明或多个深度贡献时受表达限制。论文主张组合连续 MLP 和体积渲染，可同时保留连续查询和体积累积的灵活性。

【背景知识】分辨率每轴 n 的稠密网格含 n³ 个单元，若通道 C、每通道 b 字节，存储约 Cbn³；每轴加倍则容量八倍。MLP 参数量由层宽和深度决定，不由空间网格点数直接决定。但这只说明参数存储与固定离散网格解耦；高频拟合仍受网络容量、训练样本和每条射线求积密度约束。以同一像素上的前景细绳与背景为例，单表面模型只能择一个交点解释像素，体积模型可累积多个深度贡献；这不是说任何半透明/反射都由 NeRF 正确物理建模。

**高频与采样**

仅把 MLP 用作连续场还不足：§5.1 直接输入低维坐标训练结果对高频变化表达差，作者援引 [35] 的低频偏置研究；细边、纹理、镜面变化都有较快速的空间/方向变化，需位置编码。§5.2 在整段均匀密集查询也低效：空区域密度近零，遮挡后的内容虽密度大但累积透射率小，两类点对当前像素贡献少。基于可见权重而不只密度来重分配样本，解决高频场渲染与查询预算矛盾。

**原文证据**

- Fig.4 的去VD/PE定性退化及 Table 2 的PSNR差分别3.35/2.24 dB，直接对应设计；Table 6 表明效果随场景变化。
- Table 2 无H在相同256总查询预算下PSNR低0.95 dB，支持采样分配有益，未独立测量效率增益或严格匹配模型参数量。
- Fig.5/6/8与Table 1展示整个方案的竞争力；§6.3指出NeRF权重约5 MB，相比LLFF复杂合成一场景>15 GB，但训练时间更长。

【独立评价】这里既有已知领域难点（离散网格立方增长、光场采样、非朗伯外观）也有作者通过基线/消融揭示的具体失败。没有证明所有MLP或网格必然失败，不能把经验观察变成普适理论。具体采样权重与编码数学随后展开。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
### 3.3 相关工作与技术定位

**隐式表面**

【原文事实】§2 p.3–4：作者回顾将三维形状表达为连续函数的 MLP。SDF（signed distance function，符号距离函数，[15,32]）以零水平集表达表面；occupancy（占据场，[11,27]）用空间位置预测内外/占据值。作者指出早期这类模型需要来自合成形状数据如ShapeNet [3] 的3D真值。为改用图像监督，Niemeyer等 [29] 对占据场沿射线求交，用隐式微分算精确导数，交点经神经3D纹理场预测漫反射颜色。SRN [42] 用坐标→特征，并训练递归网络逐步决定沿射线的步长，最后表面特征解码为颜色。本文认为这些方法在其所述结果中偏简单几何、渲染过平滑。

【背景知识】表面式颜色可写为交点 x* 的 c(x*)，其中 x* 来自求解隐式场零点；梯度需经过求交。NeRF 不先决定唯一交点，以连续密度和发射颜色积分。它仍是隐式连续函数，但语义从隐式表面改成可微可积的体场。早期神经函数亦被用来表达2D图像 [44]、纹理材料 [12,31,36,37]、间接光照 [38]；本文没有说神经低维坐标函数本身是新概念。

**图像式与网格**

| 路线（本文引用） | 基本机制 | 作者指出的难点 / 与本方法关系 |
|---|---|---|
| 密集光场插值 [21,5,7] | 采样相机光线并对邻近样本插值 | 视角密集时有效；较稀疏时缺少中间光线观测，需要更显式的空间约束 |
| Mesh 及视角依赖纹理 [48,2,8,49] | 几何网格投影、纹理着色 | 可微光栅化[4,10,23,25]或路径追踪[22,30]优化，作者指出病态/局部极小、固定拓扑初始化 [22] |
| 早期体积 [19,40,45] | 直接由观测给体素赋色 | 体积累积适于梯度和复杂形状，但分辨率/存储受网格影响 |
| 跨场景体积预测 [9,13,17,28,33,43,46,52] | 训练CNN预测离散体积，再alpha合成 [34]或学习合成 | 利用数据集先验，测试处理新场景；仍有离散体积空间代价 |
| 单场景神经网格 [41,24] | CNN 与显式体素联合优化，补偿体素离散或随时间/控制变化 | 仍受体素分辨率限制；NV有界体积与独立背景要求 |
| LLFF [28] | 预训练3D CNN 预测每输入视图的MPI，再融合附近MPI | 针对前向且良好采样；作者指出输入视差过大会出错、多表示混合可能不一致 |

**关系与边界**

NeRF 继承经典体渲染 [16,26]、分层采样启发 [20]、DeepSDF skip [32]、高频映射思想 [35]，与 Transformer [47] 的位置编码形式类似但目的不同：不是为离散 token 给顺序，而是连续坐标频率提升。作者还提到 cryo-EM 蛋白质结构投影 [51] 的同期类似映射。本文改进在于用连续体场而非显式高分辨率网格来承担复杂外观、让经典体渲染直接耦合 RGB 监督，再通过编码与采样改善学习及查询。SRN、NV、LLFF是§6的直接基线，DeepVoxels 原论文数据仅附录表3。

【独立评价】这些相关工作信息只来自本篇§2/5/6及其参考文献索引，未独立阅读全文或复现，故对 prior work 的评价应归于本文作者；不能据此断言其它方法在所有条件下都必须固定拓扑或不能高质量重建。本单元不推导其他论文公式；本篇数学在后续独立解释。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-FIELD -->
### 3.4.1 方法模块一：连续场与 MLP

**输入与结构**

【原文事实】§3 p.4–5、App.A Fig.7 p.18。连续函数的输入是 $\mathbf x=(x,y,z)$ 与二维视角 $(\theta,\phi)$；实践中 $\mathbf d\in\mathbb R^3$ 为单位方向向量，只有两个独立自由度。因此不是5个数值直接进网络，而是空间与方向各三分量经 Eq.4 分别映射至 60 和24维。输出为 $\mathbf c=(r,g,b)$ 和密度 $\sigma$。粗网络与细网络分别学习此类函数，参数不同。

按 Fig.7 的结构（补充张量说明，图中给的是每个点的维度），设批量有 Q 个查询：

| 张量/层 | 输入形状 | 输出形状 | 作用 |
|---|---|---|---|
| γ(x) | Q×3 | Q×60 | 位置频率特征 |
| 位置主干第1层 | Q×60 | Q×256 | 仿射+ReLU |
| 2–5层 | Q×256 | Q×256 | 逐点非线性特征 |
| skip | Q×256 与 Q×60 | Q×316 | 按题注在第5层激活拼接 |
| 6层 | Q×316 | Q×256 | 接收位置直通特征 |
| 7–8层 | Q×256 | Q×256 | 共享位置主干 |
| 附加线性输出 | Q×256 | Q×256 特征及 Q×1 密度 | 特征无激活、σ 用ReLU |
| γ(d) | Q×3 | Q×24 | 方向频率特征 |
| 拼接及颜色隐藏层 | Q×280 | Q×128 | 全连接+ReLU |
| RGB输出 | Q×128 | Q×3 | 全连接+sigmoid |

表中的 Q 和输入维度是按图推导，不是作者指定的 batch 张量布局。若全连接采用 $h'=\operatorname{ReLU}(Wh+b)$ 的列向量约定，第一层 $W\in\mathbb R^{256\times60}$，skip 后一层 $W\in\mathbb R^{256\times316}$，颜色隐层 $W\in\mathbb R^{128\times280}$，最终RGB $W\in\mathbb R^{3\times128}$；实际代码存储可转置。原文图明确60/24维，不额外加入 xyz/d 原值。

**密度与方向分支**

将位置主干写成 $h=H(\gamma(\mathbf x))$，附加线性映射给出 $\tilde\sigma=g_\sigma(h)$ 和 $z=g_z(h)\in\mathbb R^{256}$，然后 $\sigma=\max(0,\tilde\sigma)$。RGB支路 $\mathbf c=\operatorname{sigmoid}(W_c\operatorname{ReLU}(W_v[z;\gamma(\mathbf d)]+b_v)+b_c)$。密度支路非负；色彩归一化到(0,1)（sigmoid极限为0或1）。两条支路共享位置特征，但方向不会输入密度。若把方向提前拼入所有层，密度就可能按视角变化、同一空间结构的约束减弱；论文有意限制该自由度。连续性源于正余弦、线性、ReLU和sigmoid连续复合；不能因此保证高频内容准确或无限分辨率。

【背景知识】对同一空间点的不同方向 $d_1,d_2$，$\sigma(\mathbf x)$ 相同、$\mathbf c(\mathbf x,d_1)$ 与 $\mathbf c(\mathbf x,d_2)$ 可不同。镜面高光因观察方向改变而移动；用仅位置的颜色会把不同方向监督压向同一个值。Fig.3 的船侧和水面示例、Fig.4 无VD与 Table 2 第3行支持方向分支。skip 将原编码直接送到较深层，可能缓解深层中坐标信息丢失；原文称沿用DeepSDF，未单独消融此连接。

**训练与边界**

MLP 每个坐标独立查询，但所有点共享权重；损失通过体渲染把一条射线的多个点以及多射线/多视角耦合。训练不是直接监督每点的σ和c；输出颜色与密度由像素误差共同调整。测试固定网络，对新射线查询。没有显式训练相机网络、CNN特征或条件编码；没有单独材质、照明、散射或反射方程参数。σ 是学习到的有效密度，不保证真实物质密度。方向分支可拟合非朗伯外观，但不保证反射定律、BRDF或光照变化下重渲染。结构成本与层数/通道和查询数相关，不随显式网格分辨率直接立方增长；实际渲染仍慢（App.A）。

<!-- PTH:END METHOD-FIELD -->

<!-- PTH:BEGIN METHOD-RENDER -->
### 3.4.2 方法模块二：射线构造与可微体渲染

**射线构造**

【原文事实】§4、§5.3：相机光线 $\mathbf r(t)=\mathbf o+t\mathbf d$，近远边界 $t_n,t_f$；相机姿态、内参、场景 bounds 为输入。合成使用真实参数，真实数据用COLMAP估计；合成场景缩放入边长2的中心立方体，前向真实用NDC（App.A/C）。

【背景知识】以一种通用相机到世界姿态 $R,\mathbf o$ 和内参 K 为例，像素齐次坐标 $\tilde u=(u,v,1)^\top$，$\mathbf d=RK^{-1}\tilde u/\|RK^{-1}\tilde u\|$（采用相机看+z的约定；附录 C 使用看−z，轴约定需一致）。这是相机几何补充，不是论文给出的代码公式。像素决定射线方向、相机中心决定原点。仅有RGB而没有这些参数无法直接构造正确跨视角共用的空间查询。

**可微合成**

给定 N 个排序采样点 $t_i$（Eq.2），网络得到 $\{(\sigma_i,\mathbf c_i)\}_{i=1}^N$，形状分别 N×1、N×3。计算段长 $\delta_i=t_{i+1}-t_i$、光学厚度 $\tau_i=\sigma_i\delta_i$，$\alpha_i=1-\exp(-\tau_i)$，前序透射 $T_i=\prod_{j<i}(1-\alpha_j)$，加权 $w_i=T_i\alpha_i$。RGB预测 $\widehat{\mathbf C}=\sum_i w_i\mathbf c_i$。这是前向 alpha 合成而非直接平均。若某点之前有遮挡，其 $T_i$ 减小，贡献低；若当前点密度为0，其 alpha为0。透射有递推 $T_{i+1}=T_i(1-\alpha_i)$，可以逐点计算或累计和并行实现。复杂度（补充）每射线N次场查询、合成O(N)，若隐藏层宽h、层数D，主干计算量约O(NDh²)，全图还乘像素数。

【原文事实】Fig.2 表明渲染像素与观测像素比较；梯度可穿过求积与网络更新参数。密度不仅改变当前段发射，还通过后续透射改变后面的所有段。Eq.3不显式给出远端背景项，不能凭外部实现补白背景策略；详见 MATH-VOLUME。

**训练与推理**

训练对随机射线批量构造采样、同时渲染粗细颜色、比较真值；推理每像素按已知新相机执行相同粗细查询，只保留细渲染，无目标真值或更新。高分辨率图像不是放大低分辨率渲染图，而是更多射线；若目标含更高频细节，场与求积仍要足够精确。App.A 的64+192查询/射线导致800×800约1.64亿次网络查询。NDC坐标改变位置与深度参数，不能将原坐标物理距离直接套到NDC。若原射线方向未归一化，在原度量空间实际弧长是 $\|\mathbf d\|\Delta t$，公式(3)将 $\delta_i$ 称相邻距离。作者定义 d 为单位向量；NDC 变换后的坐标需按变换域解释。图像损失不能确保真实几何或物理消光正确。

<!-- PTH:END METHOD-RENDER -->

<!-- PTH:BEGIN METHOD-PE -->
### 3.4.3 方法模块三：位置编码

**频率映射**

【原文事实】§5.1 p.7–8，Eq.4 对标量 p 使用频率倍增的正余弦映射 $\gamma(p)=(\sin(2^0\pi p),\cos(2^0\pi p),\ldots,\sin(2^{L-1}\pi p),\cos(2^{L-1}\pi p))$。x 三分量归一化在[-1,1]，d 为单位向量三分量也在此范围。位置 L=10、方向 L=4，独立逐坐标后拼接为60和24维。固定映射本身没有训练参数；$F_\Theta=F'_{\Theta}\circ\gamma$，MLP 学习编码后的关系。方向仅拼到颜色分支（Fig.7），不改变密度的位置依赖约束。

**设计原因**

连续函数可表示不等于容易优化。作者依据 §5.1 的观察与 [35] 指出 MLP 偏向低频；用普通坐标输入，网络需要通过多层非线性自行构造快速变化，容易在有限训练下得到平滑场。固定高频特征让后续线性/非线性层直接组合快速变化信号。例如单维目标 $f(p)=\sin(8\pi p)$，L≥4 时 γ 中已经含此分量（k=3），线性读出即可得到该函数；直接输入 p 需网络近似整个振荡曲线。真实颜色/密度不是简单单频，MLP仍需组合多维编码；PE不是傅里叶系数直接反演，也不是离散位置索引或增加输入图像观测量。Fig.4 的无PE履带孔纹丢失与该机制一致。

**参数与边界**

位置最高频率为 $2^{L-1}\pi$ rad/归一化坐标单位；L=10 时在长度2的区间有512个完整周期的最高分量，L=4方向有8个周期（若只沿单方向分量考察）。方向三分量有单位球约束，不能把24维看作24个独立自由度。Table 2：L=5时PSNR30.59，L=10为31.01，L=15为30.81；方向L同比调整但具体整数未给。作者认为一旦 $2^L$ 超过输入图像含有的最大频率（约1024），增加L收益有限。这里图像频率和归一化三维坐标频率并非简单等同，该说法为作者经验解释。Table 6 的 Drums、Ficus、Materials 在L=5的PSNR反而高于完整，说明不存在从本文直接得到普适L值的结论。更高频可能对采样密度、噪声、观测覆盖提出要求；数学推导见 MATH-ENCODING。

<!-- PTH:END METHOD-PE -->

<!-- PTH:BEGIN METHOD-HIER -->
### 3.4.4 方法模块四：层次体积采样

**粗细流程**

【原文事实】§5.2 p.8：同时优化 coarse 和 fine 两个网络。设每射线区间 $[t_n,t_f]$，先分层均匀采样 $N_c$ 个点，以 coarse 场得到密度/颜色，按 Eq.3 合成粗色。Eq.5 将其写作 $\widehat C_c=\sum_i w_i\mathbf c_i$，$w_i=T_i(1-e^{-\sigma_i\delta_i})$。每段权重同时反映该段出现内容的机会及前序可见性。将权重归一化为 $\widehat w_i=w_i/\sum_j w_j$，据此构建分段常数概率密度函数（probability density function，此处PDF不是文档格式）。用 inverse transform sampling（逆变换采样）抽取 $N_f$ 个新增位置；将这些位置与粗位置合并排序，重新以 fine 场查询全部 $N_c+N_f$ 个位置，并按 Eq.3 重新求积。粗网颜色不是直接混入最终颜色，fine 使用自己的颜色与密度值。若 B 条光线每条有64粗点、128新点，则粗输入形状B×64×坐标维，细输入B×192×坐标维；粗权重B×64，最终输出B×3。实际张量排布原文未规定。

**训练与输出**

两网络各自从真实RGB监督。粗图损失促使其权重分布能找到可见内容；细图损失则优化高精度渲染。测试时仍先粗后细，最终仅输出 $\widehat C_f$。每射线的总查询次数64+(64+128)=256。粗细是两个网络，不是单一 MLP 的两层，也不是粗图超分辨率。重要性估计通常按样本概率加权得到整个积分的独立估计，而本文是按权重重新划分积分区间，仍使用确定性形式的非均匀求积权重，不在 Eq.3 中为每个细样本乘 $1/p(t)$。

**设计与边界**

【背景知识】只用密度重采样会在前面物体已不透明的后方仍密集采样；w 额外乘 T，抑制被遮挡段。比如粗段 α=[0,0.8,0.9]，透射=[1,1,0.2]，w=[0,0.8,0.18]，后面尽管alpha更高，权重反而较低；归一化约[0,0.816,0.184]，新增样本倾向第二段。保留粗样本可以避免完全丢弃均匀探测范围。

【独立评价】若 coarse 尚未学到极细结构，权重小导致细网也难增加样本；粗均匀采样、两路监督缓解但不能保证所有结构都发现。若所有权重为0，归一化分母为0，需实现定义保护；论文未给出 epsilon、边界插值细则、是否停止采样位置梯度等，不应凭其他代码补全。Table 2 无H保持256总网络查询而PSNR少0.95 dB，LPIPS增加0.028，说明采样组合有效；但不同网络数量/参数、GPU耗时并未完全受控。

<!-- PTH:END METHOD-HIER -->

<!-- PTH:BEGIN METHOD-TRAIN -->
### 3.4.5 方法模块五：训练、推理与实际配置

**训练配置**

【原文事实】§5.3 p.8–9、App.A p.17–18。每个场景独立优化 coarse/fine 场；每次随机抽取所有训练图像像素中的4096条射线。$N_c=64,N_f=128$，粗网络4096×64=262144次查询，细网络4096×192=786432次查询，共1,048,576次点查询/训练迭代（由参数推算，未计反向传播）。Adam 学习率从 $5\times10^{-4}$ 指数下降到 $5\times10^{-5}$，$\beta_1=0.9,\beta_2=0.999,\epsilon=10^{-7}$，迭代约100–300k，一张 NVIDIA V100 约1–2天；TensorFlow实现。对真实场景，作者在密度输出通过ReLU之前加入零均值、单位方差的随机高斯噪声，声称略改善新视图视觉表现；没有单独表格报告该项收益。

**算法全流程**

以下伪代码是从 §3–5/附录 A 重构的流程说明，不是原文独立编号 Algorithm 或已核验源代码。

```text
输入：每场景的 RGB、相机内参/姿态、near/far bounds
准备：合成场景归一化；真实前向场景使用 NDC
初始化：独立 coarse/fine MLP 参数
重复迭代：
    从训练图像所有像素中随机抽取 4096 射线及 GT RGB
    对每射线在 bounds 上分层采样 64 点 (Eq.2)
    编码点/方向，coarse 预测 σ,c
    用 Eq.3 合成 C_coarse，保留 w_i
    归一化粗权重、逆变换抽取 128 新点 (Eq.5)
    合并排序 192 点，用 fine 网络重新预测 σ,c
    用 Eq.3 合成 C_fine
    计算粗细渲染对 GT 的 L2 平方误差和 (Eq.6)
    反向传播，用 Adam 更新两个网络
测试：新相机逐像素执行粗→细采样与渲染，输出 C_fine
```

【背景知识】梯度自颜色残差出发，可经当前点颜色、密度及后续透射回到两路网络。模型参数跨射线共享使多视角信息形成联合约束。每帧图像H×W条光线；800×800时640000×256=163,840,000网络查询；原文称真实图约762k条射线，乘256约1.95亿次；作者概括每帧150–200百万查询、V100约30秒。训练迭代和帧时间不能直接转换为对其他GPU的速度保证。

**边界与附录**

合成场景限制在原点中心边长2立方体，真实场景可由近点至无限远，NDC把深度映射到[-1,1]并用视差线性采样（详细推导 MATH-NDC）。相机参数合成是真值，真实由COLMAP估计。App.B 基线SRN显存限制，合成指标在512×512、真实504×376评估，而其他高分辨率方法在800×800、1008×752（原文这里与§6.1的1008×756不同，保持原始陈述，不推定哪个为实际真实评价尺寸）。LLFF不逐场景长时间训练，使用预训练网络。原文未报告训练随机种子、网络初始化、完整边界/背景策略和采样梯度实现，不应凭外部代码补齐。

<!-- PTH:END METHOD-TRAIN -->

<!-- PTH:BEGIN MATH-VOLUME -->
### 3.5.1 数学精读：连续积分、分层随机采样、离散合成

**公式 (1)**（§4，PDF p.5–6）

【原文事实】给定相机射线 $\mathbf r(t)=\mathbf o+t\mathbf d$ 及 near/far bounds $t_n,t_f$，原式为：

$$
C(\mathbf r)=\int_{t_n}^{t_f} T(t)\sigma(\mathbf r(t))\mathbf c(\mathbf r(t),\mathbf d)\,dt,\qquad
T(t)=\exp\left(-\int_{t_n}^{t}\sigma(\mathbf r(s))\,ds\right). \tag{1}
$$

| 符号 | 意义 |
|---|---|
| $\mathbf o,\mathbf d$ | 相机光线原点与方向；§3 方向为单位向量 |
| $t,s$ | 光线参数与积分哑变量 |
| $t_n,t_f$ | 所考虑的近端、远端界限 |
| $\sigma(\mathbf r(t))$ | 非负体密度（单位弧长的终止风险/消光式参数） |
| $\mathbf c(\mathbf r(t),\mathbf d)$ | 位置及方向决定的 RGB 发射颜色 |
| $T(t)$ | 从近端到 t 未遇到终止事件的累积透射率，无量纲 |
| $C(\mathbf r)$ | 连续场的射线期望颜色，三维 RGB |

【背景知识：补充推导，非作者新定律】若光线在短距离 $dt$ 内发生首次终止的条件概率为 $\sigma(\mathbf r(t))dt+o(dt)$（在 $\|\mathbf d\|=1$ 的度量空间），则

$$T(t+dt)=T(t)[1-\sigma(\mathbf r(t))dt]+o(dt).$$

两边相减除以 dt 并取极限：$dT/dt=-\sigma(\mathbf r(t))T(t)$，初值 $T(t_n)=1$。分离变量，$d\log T=-\sigma(\mathbf r(t))dt$，积分得到 $T(t)=e^{-\int_{t_n}^{t}\sigma(\mathbf r(s))ds}$。因此终止概率密度为 $p(t)=-T'(t)=T(t)\sigma(\mathbf r(t))$；在 t 终止时贡献颜色 $\mathbf c(t,\mathbf d)$，求期望即 Eq.1。这里的 p(t) 在有限区间一般仅为次概率密度，因为

$$\int_{t_n}^{t_f}p(t)dt=T(t_n)-T(t_f)=1-T(t_f).$$

剩余概率 $T(t_f)$ 对应穿过区间仍未终止。若另外定义远端背景颜色 $\mathbf c_{bg}$，一般合成可加 $T(t_f)\mathbf c_{bg}$；这项没有写在原 Eq.1，不将它当成作者公式或具体白背景实现。若物理弧长以米计，σ 量纲为 $\mathrm{m}^{-1}$，积分指数必须无量纲；作者归一化空间/NDC下学习的密度是相应坐标域的有效参数，不能自动解释成真实介质消光系数。

**补充常密度例子**：若区间长度 D、σ 和颜色 $\mathbf c$ 恒定，令 u=t−tn，则 $C=\int_0^D e^{-\sigma u}\sigma\mathbf c\,du=(1-e^{-\sigma D})\mathbf c$。σ=0 时 C=0（无额外背景），σD→∞ 时 C→c，说明颜色和密度/透射共同决定最终像素，密度不是直接亮度。若先穿过完全不透明段，则后续项几乎无贡献。NeRF将这种发射—吸收式的积分用于学习场；不显式求多次散射、入射光场与材料的物理耦合。

**公式 (2)**（§4，PDF p.6）

【原文事实】将区间 $[t_n,t_f]$ 分成 N 个均匀 bin，每个 bin 内均匀随机采样一次：

$$
t_i\sim\mathcal U\left[t_n+\frac{i-1}{N}(t_f-t_n),\;t_n+\frac{i}{N}(t_f-t_n)\right],\quad i=1,\ldots,N.\tag{2}
$$

U 为连续均匀分布。令 $\Delta=(t_f-t_n)/N$、$u_i\sim U[0,1]$，等价 $t_i=t_n+(i-1+u_i)\Delta$。每个bin至少有一个点，且自然前后排序（边界相等是概率零情形）。相比固定网格，每次迭代位置可变化，长期训练可覆盖连续坐标；这解释了作者不希望 MLP 只在固定离散点上受监督。分层随机采样不是每条射线 N 点在全区间无约束独立均匀抽样，后者可能集中而遗漏某些区间。比如区间[0,4]、N=4，每个区间 [0,1]、[1,2]、[2,3]、[3,4] 各抽一点。即使长期覆盖连续，有限样本和近似积分仍无法确保识别任意高频结构，公式 (2) 不保证最终非线性求积无偏。

**公式 (3)**（§4，PDF p.6）

【原文事实】论文采用数值求积规则：

$$
\widehat C(\mathbf r)=\sum_{i=1}^{N}T_i(1-\exp(-\sigma_i\delta_i))\mathbf c_i,\qquad
T_i=\exp\left(-\sum_{j=1}^{i-1}\sigma_j\delta_j\right).\tag{3}
$$

$\delta_i=t_{i+1}-t_i$ 为相邻采样距离，$\sigma_i=\sigma(\mathbf r(t_i))$，$\mathbf c_i=\mathbf c(\mathbf r(t_i),\mathbf d)$；定义 $\alpha_i=1-e^{-\sigma_i\delta_i}$，$w_i=T_i\alpha_i$，得到传统 alpha compositing。原文 Eq.3 未详细规定最后一个 $t_{N+1}$ 的构造以及首采样与近端之间的处理；下方推导解释在近似分段覆盖区间的理想化情况下的形式，不补写实现细节。

【背景知识：分段常数求积推导】在区间 $[t_i,t_i+\delta_i]$ 以常数 $\sigma_i,\mathbf c_i$ 近似连续函数。第 i 段起点透射 $T_i$ 是此前各段透射乘积：

$$
T_i=\prod_{j<i}\exp(-\sigma_j\delta_j)
=\exp\left(-\sum_{j<i}\sigma_j\delta_j\right).
$$

段内位置 u∈[0,δi] 的透射为 $T_i e^{-\sigma_i u}$，段贡献

$$
\int_0^{\delta_i}T_i e^{-\sigma_i u}\sigma_i\mathbf c_i\,du
=T_i(1-e^{-\sigma_i\delta_i})\mathbf c_i.
$$

对所有段累加即 Eq.3。这个表达在分段常值假设下对各段精确，在连续场随位置变化时是近似。对 σ=0 用连续极限，段贡献0。记 $\alpha_i$ 为段内条件终止概率，$1-\alpha_i=e^{-\sigma_i\delta_i}$，则

$$
T_i=\prod_{j<i}(1-\alpha_j),\qquad T_{i+1}=T_i(1-\alpha_i),\qquad
w_i=T_i-T_{i+1}.
$$

由望远镜和得 $\sum_i w_i=1-T_{N+1}\leq1$（非负σ和距离）。这解释了它并非普通归一化加权平均；未终止概率可以对应背景或剩余透射。

**构造数值例子**：两段 $\delta_1=\delta_2=1$，$\sigma_1=\ln2,\sigma_2=\ln4$，第一段 RGB=(1,0,0)，第二段=(0,1,0)。α1=0.5、α2=0.75；T1=1、T2=0.5，权重 w1=0.5、w2=0.375；预测 RGB=(0.5,0.375,0)，剩余透射0.125。即使第二段密度更大，贡献仍受前段遮挡抑制。加上白色背景（仅示例）会变为(0.625,0.500,0.125)，不能认为 Eq.3 自带该白背景。

【补充梯度推导】对固定采样位置与距离，直接颜色梯度为 $\partial\widehat C/\partial\mathbf c_k=w_k I_3$。密度影响两个路径：$\partial w_k/\partial\sigma_k=T_k\delta_k e^{-\sigma_k\delta_k}$；对后续 i>k，$\partial w_i/\partial\sigma_k=-\delta_k w_i$，对 i<k 为0。于是

$$
\frac{\partial\widehat C}{\partial\sigma_k}
=\delta_k\left[T_k e^{-\sigma_k\delta_k}\mathbf c_k-\sum_{i>k}w_i\mathbf c_i\right].
$$

由链式法则，像素误差梯度经密度和颜色输出回传至 MLP 参数。高密度段后 T 很小，后面点的颜色梯度很弱；未观察/长期遮挡位置受监督不足。若 $\sigma_k$ 由 ReLU 输出，负预激活区域对其自身参数的直接密度梯度为0（共享特征仍可能由其他点/颜色更新）。上述只对固定采样推导，不主张作者具体重采样梯度实现。

**成立条件与边界**

- 近远区间、非负密度、前向有序采样；把 σ 解释为单位长度风险须匹配参数单位，单位方向时 $dt$ 是弧长。非单位方向须使用相邻点真实弧长或相应重参数化密度；NDC 情形使用其坐标域解释。
- 颜色在 sigmoid 下有界，透射在[0,1]；很大光学厚度时后段梯度趋弱。数值计算可用稳定的 exp/cumulative sum（背景建议，原文未规定实现）。
- Eq.1–3 给可微的图像投影，不是材质/真实几何唯一恢复证明。密度、颜色和遮挡可相互补偿；有限射线求积误差与观测歧义是不同问题。
- 公式作用：Eq.1 定义连续目标，Eq.2 给粗采样，Eq.3 对粗细场实际渲染；Eq.5复用 w，Eq.6 对渲染施加监督。

<!-- PTH:END MATH-VOLUME -->

<!-- PTH:BEGIN MATH-ENCODING -->
### 3.5.2 数学精读：位置编码与频率性质

**公式 (4)**（§5.1，PDF p.7–8）

【原文事实】对每一个标量坐标 $p\in\mathbb R$，

$$
\gamma(p)=\left(\sin(2^0\pi p),\cos(2^0\pi p),\ldots,\sin(2^{L-1}\pi p),\cos(2^{L-1}\pi p)\right).\tag{4}
$$

映射 $\gamma:\mathbb R\to\mathbb R^{2L}$，L为频带数，k=0,…,L−1 的角频率为 $\omega_k=2^k\pi$。每个频带两维 sin/cos。周期 $2\pi/\omega_k=2^{1-k}$；输入 p∈[-1,1] 时长度2，频带k在该区间振荡 $2^k$ 个完整周期。对位置 x，分别计算三分量并拼接，L=10得 $\gamma(\mathbf x)\in\mathbb R^{60}$，对方向向量 d，L=4得 $\gamma(\mathbf d)\in\mathbb R^{24}$。Fig.7 就是这两个维度。作者用高频映射改善低维输入 MLP 学习高频函数；编码固定，只有后续 $F'_\Theta$ 的参数训练。

**机制推导**（以下为背景数学补充）

1. 为什么一组频率需要正余弦？对于任意相位 φ，$A\sin(\omega p+\varphi)=A\cos\varphi\sin(\omega p)+A\sin\varphi\cos(\omega p)$；后续线性层即可学习任意相位与振幅的该频率分量。只有正弦会限制线性组合中的相位。
2. 变化敏感性：对单频对 $g_k(p)=(\sin\omega_kp,\cos\omega_kp)$，导数 $g'_k=(\omega_k\cos\omega_kp,-\omega_k\sin\omega_kp)$，平方范数 $\omega_k^2$。全部编码 $\|\gamma'(p)\|^2=\sum_{k=0}^{L-1}\omega_k^2=\pi^2(4^L-1)/3$，随 L 增大快速增长；这使细微坐标位移能引起高频特征变化。但导数大不是自动学习准确，也可能使噪声和坐标误差敏感。
3. 编码的相对位置内积：对 p,q，利用 $\sin a\sin b+\cos a\cos b=\cos(a-b)$，得 $\gamma(p)^\top\gamma(q)=\sum_{k=0}^{L-1}\cos(2^k\pi(p-q))$，特征平方范数为 L。所以距离平方 $2L-2\sum_k\cos(\omega_k(p-q))=4\sum_k\sin^2(\omega_k(p-q)/2)$。在充分小的 Δ=p−q 时近似 $\Delta^2\sum_k\omega_k^2$；对较大间距因周期性不一定单调。该关系帮助理解高频映射使网络区别相近位置的能力，不是本文对 MLP 核或收敛性的证明。
4. 例子：L=2 时 γ(p)=[sinπp,cosπp,sin2πp,cos2πp]。p=0 编码[0,1,0,1]；p=0.5编码[1,0,0,-1]（理想精确三角值）。后续选第三维即可表达 sin2πp。p=-1 与 p=1 在整数倍 π 的频率下编码相同（均周期2），提醒仅编码的全局可辨识性有边界；论文的实际图只展示 Eq.4 编码，不引入未报告的原坐标通道来消解该问题。

【背景知识】对于离散观测，频率可恢复性由采样密度与场景投影共同决定，超过可解析带宽的变化可能混叠。3D位置在图像中的投影因视角、深度而变化，不可简单把像素频率等同于三维坐标频率。MLP 非线性可组合不同频率，本映射并非严格限定函数输出为这L个频率。

**证据边界**

§5.1 援引低频偏置 [35]、Transformer 类似编码 [47]（目的不同）以及同期蛋白质工作 [51]；未读这些原论文，不独立评价其理论。Fig.4 无PE造成局部平滑，Table 2 完整PSNR31.01 vs无PE28.77；L=5/15分别30.59/30.81，L=10在该均值中最佳，Table 6 存在单场景例外。作者的 $2^L$ 大于输入约1024最高频时收益有限是经验解释，并非严格采样定理。本补充推导说明编码性质，不保证有限观测/容量/优化下真实几何高频被恢复。算法中 x编码进入密度及共享特征，d编码只用于方向颜色。

<!-- PTH:END MATH-ENCODING -->

<!-- PTH:BEGIN MATH-HIERLOSS -->
### 3.5.3 数学精读：可见性权重、逆变换重采样与损失

**公式 (5)**（§5.2，PDF p.8）

【原文事实】作者重写粗网络的 alpha 合成色：

$$
\widehat C_c(\mathbf r)=\sum_{i=1}^{N_c} w_i\mathbf c_i,\qquad
w_i=T_i(1-\exp(-\sigma_i\delta_i)).\tag{5}
$$

$c$ 下标指 coarse，不等同于点颜色 $\mathbf c_i$。$T_i,\sigma_i,\delta_i$ 为粗场和粗采样定义的量，权重 $w_i$ 为该段对射线终止概率的贡献，而非仅密度或颜色强度。作者将 $\widehat w_i=w_i/\sum_{j=1}^{N_c}w_j$ 归一化为沿光线的分段常数 PDF，并用逆变换采样获得 $N_f$ 个新点。最终 fine 网络在粗细样本的并集重新估计 Eq.3。

【补充推导】由于 MATH-VOLUME 已证 $w_i=T_i-T_{i+1}$，有 $\sum_{i=1}^{N_c}w_i=1-T_{N_c+1}$，故 $\widehat w_i$ 是有终止的条件下在第 i 区段的离散质量。若区段 $[b_i,b_{i+1}]$ 宽度 $\Delta_i>0$，定义段内均匀密度 $q(t)=\widehat w_i/\Delta_i$，总积分=1。论文仅说明 piecewise-constant PDF，未规定具体 bins 边界，这里是解释用的一般构造。累积分布 $F(b_i)=\sum_{j<i}\widehat w_j$，对于 $t\in[b_i,b_{i+1}]$，

$$F(t)=F(b_i)+\widehat w_i\frac{t-b_i}{\Delta_i}.$$

取 u∼Uniform[0,1]，找到 $F(b_i)\le u<F(b_{i+1})$ 的正质量区段，反解

$$t=F^{-1}(u)=b_i+\Delta_i\frac{u-F(b_i)}{\widehat w_i}.$$

因此 u 的落入每段概率正好等于归一化权重，区段内部连续取样。构造例子：三个等长区间 [0,1]、[1,2]、[2,3] 的粗 w=[0.1,0.6,0.1]（剩余透射0.2），归一化[0.125,0.75,0.125]，CDF边界[0,0.125,0.875,1]。若u=0.5，落第二段，$t=1+(0.5-0.125)/0.75=1.5$。大多数新增点位于第二段。若w全部为0，分母为0；若某段权重0，CDF 平坦，该段无正测度抽样概率，需跳过或约定逆函数。论文未指定数值保护/边界处理，不能将某个工程策略当原文事实。

**和经典 Monte Carlo 重要性采样的区别**（背景）：积分 $I=\int f(t)dt=\mathbb E_{q}[f(t)/q(t)]$（要求在 f 非零处 q>0），通常估计 $N^{-1}\sum_k f(t_k)/q(t_k)$。但 Eq.3 的最终颜色不是这种独立估计；作者明确把新增点当作整个区间的非均匀离散求积，先合并粗/细位置排序，再计算相邻距离和透射，每个样本与其他样本耦合。故不能为 fine color 人为加入 $1/q$，也不能保证传统 Monte Carlo 意义的无偏性。

**公式 (6)**（§5.3，PDF p.9）

【原文事实】总损失是两路渲染的像素平方误差：

$$
\mathcal L=\sum_{\mathbf r\in\mathcal R}\left[\left\|\widehat C_c(\mathbf r)-C(\mathbf r)\right\|_2^2+\left\|\widehat C_f(\mathbf r)-C(\mathbf r)\right\|_2^2\right].\tag{6}
$$

$\mathcal R$ 是训练批的光线集合，$C(\mathbf r)$ 在 Eq.6 表示观测真值RGB（注意 Eq.1 中同样符号为连续场期望颜色，语境不同），$\widehat C_c,\widehat C_f\in\mathbb R^3$ 为粗、细预测。原式是和而非显式平均；采用均值会只改变固定批大小下损失的尺度，本文未指定代码的归约方式。没有显式几何、深度、语义或材质损失。因为粗网络需要可靠权重分布，即使最终输出 fine，也需要粗渲染真值监督；细网络则拟合更精细的颜色、密度。

【补充梯度】单条光线、单一路预测 $\widehat C$，残差 $e=\widehat C-C$，$\partial\|e\|^2/\partial\widehat C=2e^\top$。固定采样位置时，$\partial\mathcal L/\partial\mathbf c_i=2w_i e$（列梯度），$\partial\mathcal L/\partial\sigma_i=2 e^\top \partial\widehat C/\partial\sigma_i$，后一导数见 MATH-VOLUME。再通过 $\partial(\mathbf c_i,\sigma_i)/\partial\Theta$ 累积到 MLP 参数。举例若一通道 GT=0.8、粗预测0.5、细预测0.7，其余通道相同，则单光线损失0.09+0.01=0.10，两预测梯度分别−0.6和−0.2；均推动向观测靠近。粗和细采样位置的依赖可能产生额外梯度路径，本文没有说明其代码是否截断该路径，不能断言具体实现。

**成立条件与边界**

非负 w、总权重大于0才定义上述 PDF；原文未提供零权重保护。粗权重可能因错误密度而集中在错误位置，导致 fine 难发现内容；保留粗样本与粗损失能缓解但不证明全空间覆盖。Eq.6 的误差能反传并不等于全局最优或真实几何可辨识；同一像素的颜色/密度可能补偿。§6.4 Table 2 无H对照验证的是整体性能，未单独证明重要性采样方差或收敛速度。

<!-- PTH:END MATH-HIERLOSS -->

<!-- PTH:BEGIN MATH-NDC -->
### 3.5.4 数学精读：NDC 射线空间的完整推导

**公式 (7)–(9)**（App. C，PDF p.19）

【原文事实】真实前向采集用 NDC（normalized device coordinates，归一化设备坐标）重参数化射线。原空间相机看负 z 方向，近远平面在 z=−n、−f，$0<n<f$，右上边界为近平面上的 r,t（此处 t 是 top，后续光线参数也记 t，应按语境区分）。视锥左右、上下对称。标准齐次透视矩阵：

$$
M=\begin{pmatrix}
\frac n r&0&0&0\\
0&\frac n t&0&0\\
0&0&-\frac{f+n}{f-n}&-\frac{2fn}{f-n}\\
0&0&-1&0
\end{pmatrix}.\tag{7}
$$

左乘齐次点 $(x,y,z,1)^\top$：

$$
\begin{pmatrix}
\frac n r&0&0&0\\
0&\frac n t&0&0\\
0&0&-\frac{f+n}{f-n}&-\frac{2fn}{f-n}\\
0&0&-1&0
\end{pmatrix}
\begin{pmatrix}x\\y\\z\\1\end{pmatrix}
=\begin{pmatrix}\frac n r x\\\frac n t y\\-\frac{f+n}{f-n}z-\frac{2fn}{f-n}\\-z\end{pmatrix}.\tag{8}
$$

投影须除以第四分量 −z（z≠0）：

$$
\pi(x,y,z)=\begin{pmatrix}
\frac n r\frac{x}{-z}\\
\frac n t\frac{y}{-z}\\
\frac{f+n}{f-n}-\frac{2fn}{f-n}\frac1{-z}
\end{pmatrix}.\tag{9}
$$

【补充检查】$z=-n$ 时第三分量 $(f+n-2f)/(f-n)=-1$；$z=-f$ 时 $(f+n-2n)/(f-n)=1$。在近面 x=r 得x'=1，y=t得y'=1。透视视锥按此映射到[-1,1]³；x'、y' 为归一化透视图坐标；z' 是 1/z 的仿射变换，不是原米制深度。对于从投影中心发出的理想针孔光线，x/z、y/z 沿线恒定，因此 NDC 中为平行于 z' 轴的射线；一般具有不同原点的三维射线仍映成直线，但不应将“保持平行”理解为透视变换对任意方向和任意直线都保存平行性。

**公式 (10)–(16)**（App. C，PDF pp.19–20）

【原文事实】目标：找到原点 $\mathbf o'$, 方向 $\mathbf d'$ 和新参数 $t'$，使原射线 $\mathbf o+t\mathbf d$ 与 NDC 射线 $\mathbf o'+t'\mathbf d'$ 描述投影后的相同点；不要求 t 与 t' 相同。将 Eq.9 改写为 $\pi(x,y,z)=(a_x x/z,a_y y/z,a_z+b_z/z)^\top$。$a_x,a_y,a_z,b_z$ 是投影常数；此处 t 是光线参数。

代入原射线 $\mathbf o=(o_x,o_y,o_z)$、$\mathbf d=(d_x,d_y,d_z)$：

$$
\begin{pmatrix}
 a_x\frac{o_x+t d_x}{o_z+t d_z}\\
 a_y\frac{o_y+t d_y}{o_z+t d_z}\\
 a_z+\frac{b_z}{o_z+t d_z}
\end{pmatrix}
=\begin{pmatrix}o'_x+t'd'_x\\o'_y+t'd'_y\\o'_z+t'd'_z\end{pmatrix}.\tag{10}
$$

规定 t=0 与 t'=0 对应同一点，消去一个自由度，得到

$$
\mathbf o'=\begin{pmatrix}o'_x\\o'_y\\o'_z\end{pmatrix}
=\begin{pmatrix}a_x\frac{o_x}{o_z}\\a_y\frac{o_y}{o_z}\\a_z+\frac{b_z}{o_z}\end{pmatrix}=\pi(\mathbf o).\tag{11}
$$

将 Eq.11 从 Eq.10 减去：

$$
\begin{pmatrix}t'd'_x\\t'd'_y\\t'd'_z\end{pmatrix}
=\begin{pmatrix}
 a_x\frac{o_x+td_x}{o_z+td_z}-a_x\frac{o_x}{o_z}\\
 a_y\frac{o_y+td_y}{o_z+td_z}-a_y\frac{o_y}{o_z}\\
 a_z+\frac{b_z}{o_z+td_z}-a_z-\frac{b_z}{o_z}
\end{pmatrix}.\tag{12}
$$

通分（a_z 抵消）：

$$
=\begin{pmatrix}
 a_x\frac{o_z(o_x+td_x)-o_x(o_z+td_z)}{(o_z+td_z)o_z}\\
 a_y\frac{o_z(o_y+td_y)-o_y(o_z+td_z)}{(o_z+td_z)o_z}\\
 b_z\frac{o_z-(o_z+td_z)}{(o_z+td_z)o_z}
\end{pmatrix}.\tag{13}
$$

x 的分子是 $o_zo_x+o_ztd_x-o_xo_z-o_xtd_z=t(o_zd_x-o_xd_z)$，y 同理；z 的分子为−td_z。对前两式提取 td_z（要求 d_z≠0），得到

$$
=\begin{pmatrix}
 a_x\frac{td_z}{o_z+td_z}\left(\frac{d_x}{d_z}-\frac{o_x}{o_z}\right)\\
 a_y\frac{td_z}{o_z+td_z}\left(\frac{d_y}{d_z}-\frac{o_y}{o_z}\right)\\
 -b_z\frac{td_z}{o_z+td_z}\frac1{o_z}
\end{pmatrix}.\tag{14}
$$

提取唯一共同依赖 t 的因子，定义

$$
t'=\frac{td_z}{o_z+td_z}=1-\frac{o_z}{o_z+td_z},\tag{15}
$$

剩余因子就是 NDC 射线方向

$$
\mathbf d'=\begin{pmatrix}
 a_x\left(\frac{d_x}{d_z}-\frac{o_x}{o_z}\right)\\
 a_y\left(\frac{d_y}{d_z}-\frac{o_y}{o_z}\right)\\
 -b_z\frac1{o_z}
\end{pmatrix}.\tag{16}
$$

如此 Eq.11/15/16 乘回就等于 Eq.10。原文指出 t=0→t'=0，t→∞→t'→1；注意后者须 d_z≠0 并且沿有效相机前方区间不穿过 z=0。$\mathbf d'$ 的长度通常不为1，它是重参数化方向/位移，不是 Eq.4 中原始单位视角向量的必然替代物。其分量是无量纲 NDC 坐标差，$t'$ 为无量纲。

**公式 (17)–(26)**（App. C，PDF p.21）

【原文事实】将 Eq.9 与 $\pi(x,y,z)=(a_xx/z,a_yy/z,a_z+b_z/z)^\top$ 对应，得到

$$a_x=-\frac n r.\tag{17}$$

$$a_y=-\frac n t.\tag{18}$$

$$a_z=\frac{f+n}{f-n}.\tag{19}$$

$$b_z=\frac{2fn}{f-n}.\tag{20}$$

这里 r,t 是 near plane 上右/上边界，f 是 far clipping plane，n 是 near clipping plane。注意 z 为负，Eq.20 的 b_z 为正。使用标准针孔相机，以图像像素宽 W、高 H 和相机焦距 $f_{cam}$（像素单位）重参数化：

$$a_x=-\frac{f_{cam}}{W/2}.\tag{21}$$

$$a_y=-\frac{f_{cam}}{H/2}.\tag{22}$$

【补充推导】对称视锥且同一焦距时，相似三角形 $r/n=(W/2)/f_{cam}$，$t/n=(H/2)/f_{cam}$；代入17、18即可。不同焦距或偏心主点的推广需要额外相机投影矩阵，不是本式直接涵盖。

原文真实前向采集取远端 f→∞，由于 NDC z 轴线性表示视差/逆深度，远距空间压缩到有限区间。Eq.19、20分别分子分母除以f，得

$$a_z=1.\tag{23}$$

$$b_z=2n.\tag{24}$$

最终将 Eq.21–24 代入 Eq.11、16：

$$
\mathbf o'=\begin{pmatrix}
-\frac{f_{cam}}{W/2}\frac{o_x}{o_z}\\
-\frac{f_{cam}}{H/2}\frac{o_y}{o_z}\\
1+\frac{2n}{o_z}
\end{pmatrix}.\tag{25}
$$

$$
\mathbf d'=\begin{pmatrix}
-\frac{f_{cam}}{W/2}\left(\frac{d_x}{d_z}-\frac{o_x}{o_z}\right)\\
-\frac{f_{cam}}{H/2}\left(\frac{d_y}{d_z}-\frac{o_y}{o_z}\right)\\
-\frac{2n}{o_z}
\end{pmatrix}.\tag{26}
$$

原文最后还有一个关键实现步骤（未单独编号）：若相机原点在 z=0 或不在近面，直接代入 o_z 会有奇点或不统一。先令 $t_n=-(n+o_z)/d_z$，平移到 $\mathbf o_n=\mathbf o+t_n\mathbf d$；因为 $o_{n,z}=o_z-(n+o_z)=-n$，然后用新原点进行 Eq.25/26。

【补充推导】替换后 $o'_z=1+2n/(-n)=-1$，$d'_z=-2n/(-n)=2$，从而 $z'(t')=-1+2t'$。原空间正深度 $Z=-z$，由 Eq.23/24，$z'=1-2n/Z$；令两式相等：

$$-1+2t'=1-2n/Z\ \Rightarrow\ t'=1-n/Z,\qquad Z=\frac n{1-t'}.$$

于是 $1/Z=(1-t')/n$。当 t' 从0到1线性变化，原始逆深度从1/n线性下降到0，深度由n至∞；严格 t'=1 为无穷远极限，不是可计算的有限点。均匀 t' 并非均匀真实深度。构造 n=1 的示例：t'=0,0.5,0.75,0.9 分别对应 Z=1,2,4,10；深度差越来越大，近处被更密集采样。若用等间隔 $\Delta t'$，有 $dZ/dt'=n/(1-t')^2$，远端相同步长对应极大深度跨度。它适合前向真实场景中近处细节与远处视差缓慢变化的设定，不能直接用于任意360°相机空间或穿过相机平面的射线。

**成立条件与边界**

1. 透视投影的齐次除法要求 z≠0，Eq.11–16 还需 o_z、d_z、o_z+t d_z 非零；near plane shift 是避免原点奇异并统一采样范围的步骤。实际需使用一致的相机坐标约定（本文−z）。
2. 若在一个参考相机投影空间内，来自该投影中心的光线变成平行射线；透视变换一般不保持度量、角度，也不必保持任意直线的平行关系。不要从作者简化表述推出物理等距。
3. 真实空间与 NDC 空间的距离、体密度参数存在重参数化差异。背景知识：在原空间用弧长 s 的物理消光积分若改成 u，须有 $\int\sigma(s)ds=\int\sigma(s(u))|ds/du|du$。本文在变换域学习有效场，不要求学习物理米制消光系数；不能直接把 Eq.25/26 的深度值或 σ 当真实深度、真实介质密度。若要恢复物理深度，须明确逆变换、尺度和渲染量定义。
4. 附录 C 给出的推导建立几何坐标等价（射线点集合对应），不是对 NDC 下图像精度或真实几何重建的数学保证。原文无 NDC vs 无 NDC 的单独消融，采用该变换只是其真实前向实验设置。

**公式作用总览**：Eq.7–9 定义透视投影，Eq.10–16 让原射线在投影空间仍以直线参数化，Eq.17–24 确定相机与无穷远参数，Eq.25–26 用于真实场景的实际射线起点与方向；near shift+均匀 t' 为逆深度采样提供有限定义域。

<!-- PTH:END MATH-NDC -->

<!-- PTH:BEGIN EXP-DATA -->
### 3.6 实验精读：数据、协议与结论

#### 3.6.1 数据集与评价（原文 §6.1）

**数据与划分**

§6.1（PDF pp.9–10）将难度不同的三类场景分开报告，而不是以单一数据集支持所有结论。

| 数据集 | 场景及成像 | 训练视角 | 测试视角 | 图像分辨率（正文） |
|---|---|---|---|---|
| Diffuse Synthetic 360° | DeepVoxels 的 4 个简单、Lambertian、带纹理扫描网格，半球视角 | 每场景 479 | 每场景 1000 | 512×512 |
| Realistic Synthetic 360° | Blender Cycles 路径追踪生成的 8 个复杂非 Lambertian 物体；6 个半球、2 个球面视角采集 | 每场景 100 | 每场景 200 | 800×800 |
| Real Forward-Facing | 手机手持拍摄 8 个近似前向场景，其中 5 个来自 LLFF 数据，3 个新采集 | 每场景总共采集 20–62 张，扣除测试图 | 每 8 张留 1 张测试 | 1008×756 |

三个数据集分别考察：大量视角下简单外观的表示精度；复杂几何、非漫反射外观与宽基线；真实采集下有限视域的重渲染。第二类仍是同一场景的已知相机训练与留出视图测试，不是训练物体与测试物体互斥的学习协议。第三类的 20–62 是采集总数，不是每个场景恰有同样数量的训练图；原文未给各场景确切测试数量。正文未指明第二类哪些具体物体对应球面采样，不能从名称补推。

**指标与比较协议**

PSNR 越高越好：在相同动态范围与 RGB 像素归一化条件下，比较均方误差的对数；SSIM 越高越好：评价局部结构等相似性；LPIPS 越低越好：评价深特征空间中的感知距离。三者都针对合成图像与对应测试图，不是三维点或表面精度。本文未列出各指标实现的全部细节、聚合顺序、裁剪边界或颜色处理，因此只以 Table 1–6 报告数值为准，不把指标一般定义冒充实验代码。

§6.2 的比较使用相同输入视图；LLFF 另有大数据预训练。Appendix B（p.19）说 SRN 的合成图评价为 512×512，而其余方法在 realistic synthetic 为 800×800；简单合成原本是 512×512。真实数据 SRN 为 504×376，其余方法在该附录写成 1008×752，而 §6.1 写 1008×756。原文未解释这 4 像素差异，保留两处原数值，不自行判定裁剪方案。不同分辨率下 PSNR/SSIM/LPIPS 不必直接同分布，尤其下采样会改变高频误差和感知距离；这种协议差异须与数值一起考虑。

**证据边界**

原文实验监督为已知相机的 RGB 图像（§3、5.3），未使用 GT 深度、法线或材质分解损失，也未报告对应独立几何/材质指标。即使合成场景生成时存在真实几何，本文也未把它作为 Table 1–6 的评价对象。真实图的像素误差混合场景、相机校准、外观与采样因素；不能从图像得分反推出唯一几何。数据覆盖 20 个具体场景，但不构成对任意新场景、动态场景、未知相机或极稀疏输入的泛化验证。

<!-- PTH:END EXP-DATA -->

<!-- PTH:BEGIN EXP-COMPARE -->
#### 3.6.2 基线、结果与代价（原文 §6.2–6.3）

**基线与协议**

| 方法 | 场景表示与新视图生成 | 本文比较中的限制或条件 |
|---|---|---|
| Neural Volumes（NV，[24]） | 深 3D CNN 生成 128³ RGBA 体素及 32³ warp grid，沿射线查询变形体积并合成 | 有界前景体积，背景分离捕获；真实前向组缺 NV 数值 |
| Scene Representation Networks（SRN，[42]） | 位置到特征的 MLP，递归射线行进预测步长，到达终点后从特征回归不透明表面颜色 | 表面交点式渲染不同于沿射线积分；附录 B 记录输出分辨率较低 |
| Local Light Field Fusion（LLFF，[28]） | 预训练 3D CNN 生成每输入视图的视锥 MPI（离散 RGBA 平面）；附近 MPI 经过 alpha 合成与混合生成新视图 | 依赖输入密度；作者引用其约 64 px 最大视差的采集建议，而复杂合成中视差可达 400–500 px |
| DeepVoxels（[41]） | 附录 Table 3 增列原论文报告的简单合成结果 | 未在全部数据集重新统一比较，LPIPS 缺失、SSIM 精度较低 |

前三种基线描述来自 §6.2（pp.10–13）及 Appendix B；DeepVoxels 的数据来源与主表不同。这里的比较体现表示与求解方案整体差异（离散体、单表面、局部 MPI、连续场），不是仅替换同一个组件的控制实验。LLFF 64 px 为采集经验而非不可超过的数学上限；其预训练成本与场景处理时间须分离。

**三组结果**

1. 简单漫反射合成：Table 1，NeRF 40.15/0.991/0.023，相比 LLFF 34.38/0.985/0.048；平均 PSNR +5.77 dB。Table 3 四个场景 PSNR、LPIPS 均领先有报告的基线，但 Chair 的 SSIM 0.991 低于 LLFF 0.992。Figure 8 的 Pedestal、Cube 用选定裁剪补充定性证据。
2. 复杂真实感合成：NeRF 31.01/0.947/0.081，对比各指标最强基线 NV 26.05 PSNR、LLFF 0.911 SSIM 与 0.114 LPIPS。平均 PSNR +4.96 dB；8 个场景的 PSNR、SSIM 均领先。然而 Hotdog LPIPS 0.121 不及 LLFF 0.061，不能把聚合领先写成逐场景全面领先。Figure 5 的 Ship、Lego、Mic、Materials 显示细结构与视角相关外观的可见差异，未展示全部场景。
3. 真实前向：NeRF 26.50/0.811/0.250 对比 LLFF 24.13/0.798/0.212；PSNR +2.37 dB、SSIM +0.013，但 LPIPS 增加 0.038（更差）。Table 5 显示 NeRF 8/8 PSNR 领先 LLFF、5/8 SSIM 领先、1/8 LPIPS 领先；Leaves、Flower、Horns 的 SSIM 例外，Fortress 是唯一 LPIPS 优于 LLFF 的场景。与 SRN 比较则 8 个场景三指标均优。Figure 6 展示 Fern、T-Rex、Orchid 的特定视图和局部。真实数据的像素指标与感知距离出现分歧，作者的总体优势主张须加上这一限定。

§6.3 认为 LLFF 在高视差下更难解决融合、SRN 在复杂几何中可能不易找到合适表面、NV 受有限体素分辨率约束。以上是作者的机制解释，不是基线内部误差来源的逐项实证分解。图中少量裁剪与聚合数值共同构成证据，但不能由单幅可见现象推出普遍原因。

**时间与存储**

§6.3 报告逐场景优化的方案均至少需要约 12 小时；NeRF 约 1–2 天/场景，单张 V100（§5.3）。LLFF 的场景处理可低于 10 分钟，但网络已经在大数据集预训练。其存储开销随输入 MPI 数量和分辨率变化；作者以一个 realistic synthetic 场景比较约 5 MB NeRF 权重与超过 15 GB 的 LLFF 表示（约 3000 倍量级的表示存储差异）。这不是训练显存、原始数据大小或运行时峰值内存的比较，也没有完整同硬件端到端预训练计时。

Appendix A 报告每条射线 256 次 MLP 查询（64 coarse + 192 fine），每张测试图约 150–200M 查询、约 30 秒/帧。800×800 图为 640,000 条射线，乘 256 为 163,840,000 次查询；真实前向 1008×756 为约 195,084,288 次（以正文分辨率估算）。紧凑表示不等于实时渲染，查询次数也不是 FLOPs 或 GPU 批处理时间的精确代理。原文没有提供统一硬件上所有方法的完整峰值内存/帧延迟表，不能据此排序所有效率指标。

<!-- PTH:END EXP-COMPARE -->

<!-- PTH:BEGIN EXP-ABLATION -->
#### 3.6.3 消融研究：控制变量与可支持的因果判断（原文 §6.4）

**设计与对照**

Table 2（p.14）是 realistic synthetic 八场景聚合，Appendix D Table 6（p.25）给每场景结果；完整模型与其余 8 组变体共 9 行，具体数值见前文逐表转录。PE 为位置编码，VD 为视角相关颜色分支，H 为分层采样（为阅读便于使用的简称）。

| 对照 | 主要改变 | 仍需注意的共同变化 |
|---|---|---|
| Full | PE + VD + H；100 images；位置 L=10、方向 L=4 | 参考 |
| No PE, VD, H | 三项同时移除 | 不可分辨各项交互贡献 |
| No PE | 不使用输入编码 | 输入通道数及网络输入参数也改变 |
| No VD | 不使用方向依赖 | 分支容量、外观表示假设可能同时变动 |
| No H | 单网络、每条射线 256 样本 | 查询预算对齐而非模型参数量、显存或训练时长对齐 |
| 25 / 50 input images | 减少输入视图数量 | 可见区域与采样间隔同时改变，未列出多种选择策略 |
| L=5 / L=15 | 改变位置频率数量；方向频率数成比例调整 | 未报告变体的确切方向整数 L；输入维数亦改变 |

完整方案 64 个 coarse 样本，额外 128 个 fine 样本并合并为 192 个 fine 查询，总计 256；无 H 用单网络 256 查询。此处比较的是等查询次数下的空间采样策略与粗细网络整体，而不是两种完全等复杂度的神经网络。

**数值与因果**

以 Full 的 31.01/0.947/0.081 为基准，PSNR、SSIM、LPIPS 分别按高、高、低优。三个组件全部移除后为 26.67/0.906/0.136；三指标下降，支持组合有效。单项移除：No PE 为 28.77/0.924/0.108（PSNR -2.24）；No VD 为 27.66/0.925/0.117（-3.35）；No H 为 30.06/0.938/0.109（-0.95）。三项均使整体指标变差，但不能由这三项差值相加预测联合去除的损失，也不能直接由差值排序所有场景中机制的物理重要性。Figure 4 为编码与方向分支的定性对照，未单独展示 H 的图像对照。

逐场景 Table 6 进一步显示：No PE 下 Lego PSNR 27.75 对 Full 32.54，支持对高频细节敏感；No VD 的 Materials 与 Ficus PSNR 分别由 29.62 降至 24.96、30.13 降至 25.91，符合非漫反射表示不足的解释；No H 下 Drums LPIPS 0.177 对 Full 0.091，说明平均 PSNR 小差距不能代表每个场景的感知质量。证据支持具体设计整体有效，不独立识别网络容量、采样位置和优化动力学各自贡献。

视图数：25 图 27.78/0.925/0.107，50 图 29.79/0.940/0.096，100 图 31.01/0.947/0.081。25 图模型的聚合三个指标仍优于 Table 1 的 100 图基线中最佳聚合值（PSNR 26.05、SSIM 0.911、LPIPS 0.114），但这是不同训练视图数量的跨表比较，不代表各基线都在同样 25 图条件下测试。视图变少质量下降，未给 1–9 视图、遮挡区域单独统计或学习先验对照，因此不能宣称解决任意稀疏视图问题。

频率：L=5 为 30.59/0.944/0.088，L=15 为 30.81/0.946/0.096，Full L=10 平均三指标更好。作者解释当最高频率尺度超过输入图像可提供的有效分辨率（讨论中约 1024 的量级）后收益有限；这是经验解释而非严格 Nyquist 保证。Table 6 中 L=5 的 Drums PSNR 25.29 高于 Full 25.01、Ficus 30.73 高于 30.13、Materials 29.77 高于 29.62，Hotdog LPIPS 0.087 优于 Full 0.121；不存在所有场景、所有指标都由 L=10 最优的结论。

**证据边界**

论文给出固定实验配置的消融而非完整析因实验（未列出所有两两组合），没有不同随机种子重复、误差棒或显著性检验。没有把 256 次查询的比较进一步统一为相同网络参数量、GPU 时间或真实场景训练代价。所观察的相关变化可以支持该任务配置中的设计选择，但不能作为全局最优超参数、理论可辨识性或跨场景泛化的证明。

<!-- PTH:END EXP-ABLATION -->

<!-- PTH:BEGIN EXP-APPENDIX -->
#### 3.6.4 附录 A–D：从正文概念到复现实验

**附录 A–B**

Appendix A（pp.17–18）补充网络和运行协议，Figure 7 的 60 维位置/24 维方向编码、8 层 256 单元主干、输入跳连、密度 ReLU、128 单元方向分支、RGB sigmoid 将 §3 的抽象映射落实到结构。密度不依赖方向；方向依赖颜色通过空间特征和方向编码联合建模。合成场景缩放至以原点为中心、边长 2 的立方体，真实场景使用 NDC。真实场景优化时在 σ 的 ReLU 前加入零均值、单位方差的随机高斯噪声，作者报告视觉效果略有改善；没有相应消融数值，不能断言对哪一项指标有多大收益。实现为 TensorFlow [1]。64+192=256 查询/射线、每帧约 150–200M 查询、单 V100 约 30 秒，补足正文只报告训练时间的效率图景。

Appendix B（pp.18–19）记录使用各基线作者开放的代码：NV 单场景无时间依赖训练，SRN 单场景训练，LLFF 使用已预训练模型。SRN 甚至用 4 张 V100 并行仍受 512×512 分辨率限制，这说明其报告配置的实际资源限制，不是方法理论上的绝对分辨率上界。此处仅核对论文叙述，未下载或运行基线代码。

**附录 C–D**

Appendix C（pp.19–21）用透视投影、齐次除法、射线重新参数化和无限远平面极限给出 NDC 射线变换（公式 7–26，前文逐式推导）。其中 t′ 与原参数 t 为非线性关系，等间隔 NDC 采样非等距离采样；要进行物理深度比较需回到一致坐标与单位。真实场景 σ 在变换域内拟合，因此不能将该参数直接解释为真实介质的单位米消光系数。

Appendix D（pp.22–25）补足 Figure 8 与 Tables 3–6；Table 3 引入原论文 DeepVoxels 指标，Table 4、5、6 将聚合均值分解到场景级，能观察到主表未展示的局部反例。§6.1 真实图 1008×756 与 Appendix B 的 1008×752 记载不同；前者与 Appendix A 的约 762k 射线一致，后者与 SRN 504×376 的两倍尺寸一致。上述算术只显示各处内部的对应关系，不证明究竟使用哪一分辨率计算指标。

**证据边界**

附录 D 指向补充视频中的平滑视角路径，作者以视频主张多视角一致性优势；本地资源未包含该视频，因此只记录这一主张，不能独立验证时间上的闪烁、融合不一致或完整路径表现。正文与附录的措辞如“outperforms”应结合逐指标反例理解，不以该措辞覆盖实际数值。附录提供更多复现信息，但终端间隔、背景合成细节、采样梯度路径、完整训练随机性、指标代码及场景具体相机分布仍非全部可从纸面确定。

<!-- PTH:END EXP-APPENDIX -->

<!-- PTH:BEGIN SYNTHESIS-CLAIMS -->
## 4. 综合判断与知识内化

### 4.1 贡献、证据与技术边界

**贡献—证据矩阵**

| 作者的设计或主张 | 主要证据 | 本文支持程度与限制 |
|---|---|---|
| 5D 连续辐射场：密度只依赖空间、颜色依赖空间与方向 | §3，Fig.2、3、7；Fig.4；Table 2 的 No VD | 可表达非 Lambertian 外观，且所测设置下方向分支有效；没有材质/光照或几何独立 GT，不保证物理分解 |
| 经典体渲染的可微离散化支持 RGB 监督下场优化 | §4 公式 (1)–(3)，§5.3 公式 (6)，Fig.2；Table 1 | 数学机制明确、实证表明能实现 NVS；体渲染和 alpha 合成是已有工具，不把积分定律视为本文首创 |
| 频率编码帮助 MLP 表示高频空间与外观 | §5.1 公式 (4)，Fig.4，Tables 2、6 | 设计动机与平均消融一致；场景级频率反例及没有容量匹配对照，不能保证任意频率有效 |
| 粗细网络分层采样以权重聚集有效查询 | §5.2 公式 (5)，Table 2、6 | 等查询预算下聚合指标改善；不等参数量、GPU 时间，且权重依赖粗模型，未证明全局最优采样 |
| 训练后高质量视图合成和紧凑存储 | Table 1、3–5，Fig.5、6、8，§6.3 | 合成聚合优势较强、真实 PSNR/SSIM 优于 LLFF，LPIPS 反例明确；5 MB 权重不能推出低训练显存或实时渲染 |

贡献的核心并非发明体渲染、MLP 或正余弦本身，而是将连续神经场的表示约束、频率映射、经典可微积分、数据自监督和分层查询组合成可优化的新视角合成流程，并给出针对该任务的实验支持。该判断是阅读者依据本文所述相关工作和实验作出的归纳，不是对全部先行研究的独立优先权调查。尤其作者称能够恢复复杂几何，应与任务上的几何相关可见效果和未报告几何真值指标区分开。

**核心机制闭环**

已知相机参数和训练图像为观测 → 从像素构造射线 → 在有界域或 NDC 域采样位置 → 编码位置与方向 → MLP 预测位置密度及方向颜色 → 用沿射线的透射率和局部 alpha 形成加权颜色 → coarse 权重归一化指导 fine 采样 → 两个网络的渲染 RGB 同真实像素做平方误差 → Adam 更新网络 → 测试用新相机射线重复查询与合成。跨视角约束的关键是各条射线共享空间密度函数与颜色函数，而不是每张图各训练独立图像映射。

具体理解：密度沿射线控制能见度和遮挡，颜色描述到观察方向的局部外观；位置编码解决网络表达/优化的频率偏置，分层采样把计算资源倾向于对当前颜色有贡献的区间。图像残差沿公式 (3)、(6) 的链条传播至共享参数。本文由图像重建约束间接学习几何相关结构，而非从输入直接回归显式深度；这个区别决定了评价范围和可辨识性边界。

**技术边界**

- 适用静态、已知相机的同一场景重渲染；不直接处理未知相机、运动对象、单次前馈跨场景推断或任意视角外推。
- 优化耗时 1–2 天且推理约 30 秒/帧；场函数虽紧凑，查询量大。作者在 §7 明确希望未来改进优化与渲染效率、引入更明确可解释的参数和组合式场景表示。
- 体密度和视角颜色是表示量，不是完整逆渲染中独立的光源、材质、几何或波长相关介质变量。固定颜色观测无法证明这些因素被唯一分解；NDC 也改变距离含义。
- Table 2/6 仅有限视图减少实验，仍为同一场景优化；不证明极稀疏视角下歧义解决、未知区域真值恢复或对动态/参与介质场景的可靠性。

<!-- PTH:END SYNTHESIS-CLAIMS -->

<!-- PTH:BEGIN SYNTHESIS-AUDIT -->
### 4.2 精读覆盖、边界与自检

**覆盖与来源**

本报告以本地 arXiv 2003.08934v2 论文 PDF 为主证据，HTML 用于辅助目录核对；使用 PDF 物理页码定位。覆盖范围：正文 §1–7、附录 A–D、全部 8 幅 Figure、全部 6 张 Table、公式 (1)–(26)。无独立编号 Algorithm，正文算法已按射线查询、渲染和优化步骤说明；参考文献部分用于追踪本文引述的相关工作及基线，未逐篇外部检索或复述全部文献。各图保留 PDF 区域坐标、实际 JPEG 展示资源、原题注和译文；各表保留原报告数值、缺项、指标方向和例外；数学部分将论文编号公式与阅读辅助推导分开。

**未证实事项**

- 固定阅读版本为 v2（2020-08-03），B0/B1 已核验当时最新记录；不表示今后不会更新，也不将其他发表版本或后续实现差异并入报告。
- HTML 有 4 项 Figure 6 T-Rex 图资源缺失及 7 项装饰资源引用未解析；PDF 完整，使用 B2 已验收的 PDF 区域副本。8 幅图中 1 幅 PASS_EXACT、7 幅 PASS_WITH_NOTES；Figure 2 另有内联流核验覆盖限制；未见内容缺失或明显失真，但不能称全部像素级无差异。JPEG 是展示副本而非原始矢量/无损数据；客户端 Markdown 或 Hexo 的实际排版和显示效果未验证。
- 补充视频未提供，多视角一致性路径主张无法独立核查；没有运行作者代码或重现实验。数据/指标协议有分辨率不一致及基线预训练/分辨率差别；真实数据 LPIPS 反例与逐场景反例均予以保留。
- 公式中未完全规定的边界、实际背景项、采样梯度路径与代码特例不补造。高质量图像不直接等于唯一且正确的三维几何、物理材质或真实距离，尤其 NDC 场中的深度和密度须区分变换域量与物理量。

**知识内化**

1. 问：NeRF 学习的是一张图还是一个场？答：同一静态场景的共享函数，输入为 3D 位置和观察方向，输出密度和颜色；从新相机构造射线后对场积分。
2. 问：为什么密度只依赖位置？答：约束不同视角共用几何相关遮挡结构，而方向颜色允许视角外观变化；这提高一致性但不是几何唯一性证明。
3. 问：体渲染的三个量如何区分？答：σ 是单位射线参数上的终止率，α 是区间终止条件概率，T 是到达该区间前的存活概率；权重 Tα 是对颜色贡献的概率式权重。在变换域，单位不必对应米。
4. 问：为什么要有 PE 和 coarse/fine？答：前者改变输入频率表示，帮助拟合高频；后者把有限查询预算分配到当前粗网络预测的贡献区间。Table 2 的平均消融支持设计，Table 6 显示不是所有超参数都逐场景最优。
5. 问：论文最强的实证结论是什么？答：报告的两类合成数据聚合指标均优于所比较基线，真实前向 PSNR/SSIM 聚合领先但 LPIPS 不及 LLFF；实验监督与评价是 NVS RGB，不能代替 GT 几何验证。
6. 待读者继续研究的问题（阅读者提出，不是作者已解决的问题）：如果相机不准或观察更稀疏，共享场的约束如何变化？如果需要物理距离与可解释介质参数，坐标变换和辐射传输量应如何定义？如果优化目标仅是图像残差，如何设计独立实验评估几何与外观分解？这三问均超出本文直接验证范围，后续研究笔记需另立证据。

一句话总结：NeRF 用一个经频率编码的连续五维场，通过经典可微体渲染、RGB 监督及粗细分层查询，在已知多视角的静态场景内学习可用于新视角合成的紧凑表示；它的 NVS 实证优势并不等价于真实物理属性的唯一恢复。

<!-- PTH:END SYNTHESIS-AUDIT -->



---

# 第二部分：学习与研究笔记

# NeRF 学习与研究笔记

> 精读报告 SHA-256：044621e97802f3de55d5a1a7c7928481342fc9c7626d9e48b3f3faa838066470
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：从可微渲染到逆问题的概念内化

### 1. 连续函数不是无限有效分辨率，也不是几何真值

【原文事实】§3、Fig.7，冻结报告 §3.4.1：NeRF 以场景专属 MLP 表达 $(\sigma(\mathbf x),\mathbf c(\mathbf x,\mathbf d))$，位置编码和方向编码分别进入位置主干和颜色支路，密度不依赖方向。它不是将图片直接编码成网格，也不是对每张图训练独立的二维生成器。连续坐标可查询，查询密度却不等于有效信息密度；有限观测、有限参数、优化和数值求积共同限定可恢复内容。

【补充推导】把渲染器视为算子 $\mathcal R$，场 $f=(\sigma,c)$，有限相机/像素集合 $\Omega$ 给出 $y=\mathcal R_\Omega(f)$。连续表示意味着可计算 $f(x)$，不意味着 $\mathcal R_\Omega$ 是单射。设 $J=\partial\mathcal R_\Omega/\partial\theta$ 为固定采样下模型参数的局部雅可比；若存在非零 $v$ 使 $Jv=0$，则参数扰动的一阶图像变化为零。参数冗余也会产生零空间，故参数不可辨识不能自动推出几何不可辨识；需进一步检验该扰动是否改变场或几何。反过来，即使局部满秩，也不证明全局唯一。增加观测可添加雅可比行，但重复视角可能不增加秩。实际优化可通过架构偏置选取一个解，不是恢复唯一真值的证明。

【设计反事实】若让 $\sigma(\mathbf x,\mathbf d)$ 也随方向自由变化，模型可能为不同观测方向放置不同遮挡，而保持各训练图像正确；若强制 $c(\mathbf x)$ 与方向无关，则非漫反射外观可能被迫折中。前者是扩大自由度的理论风险，后者在 Fig.4、Table 2 的 No VD 得到任务层面的证据（平均 PSNR 27.66 对完整 31.01）；并不能由此证明所学密度为真实表面。一个可辨认的区别是：改变观察方向时，同一点的密度保持一致，而颜色可以改变；改变照明并不属于原模型显式输入。

### 2. 光学厚度、颜色、权重与可辨识性

【原文事实】§4 Eq.(1)–(3)、冻结报告 §3.5.1：体渲染的核心为 $w_i=T_i\alpha_i$，其中 $\alpha_i=1-e^{-\sigma_i\delta_i}$，$T_i=\exp(-\sum_{j<i}\sigma_j\delta_j)$。$w_i$ 表示区间对当前视线的终止概率式权重，而非该点的绝对亮度。色彩为 $\widehat C=\sum_i w_i c_i$。求和质量不一定为 1，剩余透射为 $T_{N+1}$；原公式未写额外背景项。

【补充推导：一段的精确等价族】取长度 $D=1$、无额外背景的单通道常值段，测得 $y=0.4$。设 $\tau=\sigma D$，则 $y=(1-e^{-\tau})c$。令任意 $\alpha\in[0.4,1)$，可取 $c=0.4/\alpha\in(0.4,1]$ 与 $\tau=-\log(1-\alpha)$。例如 $\alpha=0.5,c=0.8$ 和 $\alpha=0.8,c=0.5$ 都给出 0.4，光学厚度分别约 0.693 与 1.609。即使无噪声，也有无穷多个光学厚度—颜色等价解；独立深度/透射观测或跨射线共享约束才可能消除部分自由度。雅可比为 $(\partial y/\partial\tau,\partial y/\partial c)=(ce^{-\tau},1-e^{-\tau})$；固定 y 的局部方向满足 $dc/d\tau=-ce^{-\tau}/(1-e^{-\tau})$，与等价族直接求导一致。该构造只证明受限单射线模型的歧义，不证明完整多视角 NeRF 必定产生同一种退化。

若同一均匀段以不同已知长度 $D_1=D,D_2=2D$ 被观测，且颜色在两观测中相同、无背景，则 $y_2/y_1=(1-e^{-2\sigma D})/(1-e^{-\sigma D})=1+e^{-\sigma D}$。在 $y_1>0$、比值 $r\in(1,2)$ 时，可得 $\sigma=-\log(r-1)/D$，再得 $c=y_1/(2-r)$。这说明观测的**独立性**比单纯观测数量更重要；两个同长度观测没有此额外约束。接近 $r=1$ 时 $d\sigma/dr=-1/[D(r-1)]$ 的绝对值很大，噪声会放大；接近 $r=2$ 时颜色估计也不稳定。此例需相同材料/均匀密度和颜色、已知物理长度，不适用于任意 NeRF 或不同方向颜色。

### 3. 梯度遮挡与层次采样的反馈

【原文事实】Eq.(3)、(5)、(6)，冻结报告 §3.4.4、§3.5.1、§3.5.3：coarse 场的权重引导新增采样位置，fine 场重新查询粗细合并位置；两个场分别获得 RGB 损失。对于后方颜色，梯度直接乘以 $w_i$。因此密度估错不仅影响图像，也能影响后面获得训练信号和新增样本的机会。

【补充推导】固定三段的颜色和采样位置，设第一段错误光学厚度 $\tau_1=5$，其后段的 $T_i\le e^{-5}\approx0.00674$；对于任意后方 $c_i$，单通道输出梯度 $\partial \widehat C/\partial c_i=w_i\le0.00674$。即使真实目标主要来自后方，直接颜色梯度也可能较弱。与此同时 coarse 归一化权重会偏向前段。在另一个构造例中，区间权重 $(0.80,0.19,0.01)$ 和总权重 1，则 128 次独立逆变换新增抽样中第三段期望仅 1.28 次，不抽中的概率 $0.99^{128}\approx0.276$。原文分层/细采样实现细节未完整规定，这只是独立抽样的概率示例，而不是作者代码的实测失败率。

【设计替代】可考虑混合采样分布 $q_i=(1-\lambda)\widehat w_i+\lambda/M$（$M$ 个区间，$0<\lambda\le1$），使每段概率至少 $\lambda/M$，独立新增 N 次样本完全漏掉某段的概率不超过 $(1-\lambda/M)^N$。它不是论文提出的采样规则；实际须明确保留粗点，按非均匀求积重算区间长度，不将 $1/q_i$ 随意乘入 alpha 合成。此替代可增加探索，却降低对已知贡献集中的利用，不能保证未观测表面被识别。原论文 No H 消融在等 256 查询下平均 PSNR 30.06，对完整 31.01（Table 2），支持原设计组合而非混合策略或时间等价结论。

### 4. 位置编码改变可表达的频率和优化条件，不增加测量

【原文事实】Eq.(4)、§5.1、Fig.4、Tables 2/6，冻结报告 §3.4.3、§3.5.2。原模型使用位置 $L=10$、方向 $L=4$；输入编码 60/24 维。No PE 平均 PSNR 28.77 对 31.01，L=5、15 的平均值分别 30.59、30.81；部分场景 L=5 更高，不能称频率越多越好。

【补充推导】对一维编码的两个坐标 $p,p+\Delta$，由三角恒等式得

$$
\|\gamma(p+\Delta)-\gamma(p)\|^2=2\sum_{k=0}^{L-1}(1-\cos(2^k\pi\Delta)).
$$

当所有 $|2^k\pi\Delta|\ll1$ 时，近似为 $\pi^2\Delta^2(4^L-1)/3$。故同样的微小坐标误差可能在高频编码下产生较大特征差异，但不能直接换算成颜色误差，后续网络权重与局部梯度同样重要。若只观察 $p_j=j/8$，目标 $f(p)=0$ 与 $g(p)=\sin(8\pi p)$ 在这些点完全相同，点间不同；编码可表示后一函数，却不能凭这些样本确定哪个是真实函数。这个点观测例仅用于解释采样混叠，真实图像是射线积分并受相机分布和像素足迹影响，不能由二维图像分辨率直接确定三维频率上限。

【反事实】低频优先或自适应频率可能对噪声与相机误差更稳健，高频则有助于细节；需同时控制观测、参数容量、计算预算和实际目标的频率。不能把低频限制当作几何真实性保证。

### 5. 坐标参数与物理量：渲染等价不等于密度等价

【原文事实】附录 C Eq.(7)–(26)、冻结报告 §3.5.4：真实前向场景用 NDC 约束域，深度坐标与逆深度相关。此变换改善无界远景的参数化，但场变量的单位必须和积分参数一致。

【补充推导】沿单条射线用单调可微变量 $u=g(t)$（$g'>0$）重参数化，若要保持物理透射 $e^{-\int\sigma_t(t)dt}$ 不变，应令 $\sigma_u(u)=\sigma_t(t(u))\,dt/du$，同时 $c_u(u)=c_t(t(u))$。例如 $u=1-a/t$（$t\ge a>0$），则 $t=a/(1-u)$，$dt/du=a/(1-u)^2$；即使物理消光常数 $\sigma_t=\beta$，变换域有效密度也随 u 变化。反之，变换域常密度通常不表示物理空间常密度。这里是单射线示例，不是直接替代附录 C 的完整三维 NDC 映射；不同射线的雅可比可能不同，若要求变换域位置唯一标量密度同时维持所有方向的物理消光，需要额外条件，不能由该一维公式保证。

### 6. 图像监督的目标与实验证据层次

【独立评价】像素 MSE 是任务代理。高 PSNR 说明相同协议下误差较小，不说明深度、光学参数或不确定性均正确；LPIPS 与 SSIM 又度量不同性质。Table 1 的真实前向数据 PSNR/SSIM 聚合领先而 LPIPS 不及 LLFF，是必须保留的例外。若研究目标变为真实几何或物理分解，应增加独立深度、透射或校准指标，并区分训练视角拟合、留出插值和覆盖域之外外推。本文没有报告这些几何/介质验证；上述内容是学习拓展，不能替代原文结果或声称已验证。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Problem & Task — Scene-specific Novel View Synthesis
- **Keywords:** Novel View Synthesis; Scene-specific Optimization; Posed RGB Images; Static Scene
- **Highlight:** 训练使用同一静态场景的已知相机与 RGB，逐场景优化共享函数；测试由目标相机查询，而非新场景的条件前馈推断。
- **Evidence:** §1、§3、§5.3；Fig.1–2；冻结报告 §3.1。
- **Connection Points:** 输入视图数、位姿假设、场景专属优化与跨场景推断、目标视角覆盖。
- **Open Questions:** 在固定观测预算下，哪些约束来自同一场景观测，哪些需要额外先验？

### Topic: Scene Representation — Factorized Radiance Field
- **Keywords:** Neural Radiance Field; View-dependent Radiance; Volume Density; Coordinate MLP
- **Highlight:** 空间位置决定密度，位置与方向决定颜色；共享几何相关分支与方向外观分支的自由度不同，但未显式分解材料、光照或真实表面。
- **Evidence:** §3；Fig.3、7；Table 2 No VD；冻结报告 §3.4.1。
- **Connection Points:** 几何与外观参数化、视角依赖、表示容量、物理可解释性。
- **Open Questions:** 共享密度在何种观测覆盖与方向外观容量下仍可能出现几何歧义？

### Topic: Appearance & Transport — Emission-Absorption Rendering
- **Keywords:** Volume Rendering; Transmittance; Optical Thickness; Alpha Compositing; Differentiable Rendering
- **Highlight:** 颜色积分由非负密度、累计透射和方向 RGB 构成；分段常值求积使遮挡与局部贡献形成可微链路，并未独立恢复介质物理变量。
- **Evidence:** §4 Eq.(1)–(3)；Fig.2；冻结报告 §3.5.1。
- **Connection Points:** 透射率定义、传播积分、物理参数单位、背景与散射建模、梯度路径。
- **Open Questions:** 若需要区分对象与参与介质，何种额外观测能够约束同一积分中的多个未知量？

### Topic: Representation Frequency — Sinusoidal Encoding
- **Keywords:** Positional Encoding; Fourier Features; Spectral Bias; Coordinate Sensitivity
- **Highlight:** 固定倍频正余弦让 MLP 更易表达高频；L=10 的平均结果优于 L=5/15，但逐场景存在反例。
- **Evidence:** §5.1 Eq.(4)；Fig.4；Tables 2、6；冻结报告 §3.5.2。
- **Connection Points:** 空间与方向频带、表达容量、采样分辨率、观测噪声。
- **Open Questions:** 如何将可恢复频率与空间观测覆盖而不是固定全局 L 联系起来？

### Topic: Efficiency & Scalability — Visibility-weighted Hierarchical Sampling
- **Keywords:** Hierarchical Sampling; Inverse Transform Sampling; Coarse-to-Fine; Quadrature
- **Highlight:** coarse 权重归一化构造沿射线的采样分布；fine 重新查询合并样本并求积。无层次采样对照等查询预算，不等网络参数量或运行时间。
- **Evidence:** §5.2 Eq.(5)；App.A；Table 2 No H；冻结报告 §3.4.4。
- **Connection Points:** 采样预算、探索与利用、可见性权重、计算效率和漏检风险。
- **Open Questions:** 如何检测粗场置信不足时细采样遗漏潜在结构的问题？

### Topic: Observation & Prior — View-count Ablation
- **Keywords:** Sparse Views; Observation Coverage; RGB Supervision; Geometry Ambiguity
- **Highlight:** 25/50/100 图的平均 PSNR 27.78/29.79/31.01；观察减少时性能下降，但未测试极少视角或不同几何覆盖的受控对照。
- **Evidence:** §6.4；Tables 2、6；冻结报告 §3.6.3。
- **Connection Points:** 视差、遮挡、可观测性、跨场景先验、几何与 RGB 的独立评价。
- **Open Questions:** 相同图数但不同基线与可见区域时，哪些几何量更难确定？

### Topic: Geometry Parameterization — Normalized Device Coordinates
- **Keywords:** NDC; Inverse Depth; Forward-facing Scenes; Ray Reparameterization
- **Highlight:** 前向拍摄采用 NDC 将远景压入有限域；域内深度和密度不能自动解释为物理距离与消光。
- **Evidence:** App.C Eq.(7)–(26)；冻结报告 §3.5.4。
- **Connection Points:** 远景采样、无界场景、坐标雅可比、真实尺度及传输参数。
- **Open Questions:** 在坐标压缩下，怎样分别保证求积效率和物理距离一致性？

### Topic: Technical Limitations — Image Quality versus Interpretability
- **Keywords:** PSNR; SSIM; LPIPS; Inverse Rendering; Identifiability
- **Highlight:** 合成数据聚合指标领先，真实前向 LPIPS 不及 LLFF；作者指出效率与解释性仍待改善，图像结果不构成独立几何或物理验证。
- **Evidence:** Table 1；§6.3、§7；冻结报告 §4.1–4.2。
- **Connection Points:** 评价协议、可解释变量、独立几何监督、优化时间、失败分析。
- **Open Questions:** 如何设计能识别外观补偿所掩盖的几何误差的评价协议？


## RESEARCH-DIRECTION：研究定位与可迁移机制

本笔记采用可公开的一般研究分析：以本文所暴露的稀疏观测下场景推断、渲染物理可解释性和计算效率为线索，不将特定个人项目或未经授权的私密研究计划纳入发布源。以下不是作者声明的后续研究路线，也不是独立跨文献调研。

### 1. 区分研究层级

| 层级 | 本文定位（据 §1、§3–6） | 可进一步提出的研究层级（独立分析） |
|---|---|---|
| Field（领域） | 三维视觉、计算机图形学与图像合成 | 受限观测下的三维视觉、逆渲染 |
| Task（任务） | 给定新相机合成 RGB | NVS、几何恢复、外观/传输估计可各自成为任务 |
| Task Setting（任务设定） | 已知姿态、多视图、静态场景、逐场景优化 | 视图数量少、覆盖弱、位姿噪声、参与介质等条件需分别定义 |
| Research Problem（科学问题） | 如何用连续神经函数与可微渲染表示复杂空间与视角外观，获得高质量新视角 | 观测不能区分多种场时，如何判定与缓解几何和外观的不确定性？ |
| Method（方法） | 位置/方向编码、共享密度场、体渲染、层次采样 | 候选约束、观测设计、先验、诊断与自适应采样，均待论证 |

任务是希望获得什么输出；设定是允许哪些输入与假设；科学问题是任务困难的根因。不能把某个网络模块直接当作科学问题，也不能认为 NVS 必然需要唯一的真实几何才能成功。冻结报告 §3.1、§4.1 说明了该区别的论文证据边界。

### 2. 稀疏观测：从视图数到约束结构

【原文事实】§6.4、Tables 2/6：25、50、100 图的平均 PSNR 为 27.78、29.79、31.01；该实验支持在此设置下更多训练图有助于 NVS，但未验证极少图、相同数量不同位姿分布的对照或跨场景先验。不能从下降趋势单独识别出原因，也不能将 NeRF 的 25 图直接定义为所有场景下的稀疏阈值。

【背景知识与合理推断】对同一候选空间点，来自多个视角的射线若从相近位置穿过，约束可能高度相关；若点被遮挡，图像并不提供该点直接的颜色观测。用理想针孔双目说明：基线为 B、焦距为 f（像素单位），前向深度 Z 与视差 d 满足 $Z=fB/d$。小误差线性化给出 $|\Delta Z|\approx Z^2|\Delta d|/(fB)$。假定 f=1000 px、B=0.1 m、Z=10 m，视差10 px；0.5 px 的匹配误差对应约0.5 m 深度误差。把基线减半时误差近似加倍。该例要求平行校正双目、可正确匹配、误差足够小；不是 NeRF 估计深度的公式，也不保证基线无限扩大都更好，因为遮挡与匹配困难可能增加。

因此，研究者应把观测数量、相机分布、覆盖率、成像可靠性与位姿精度分别控制。对某些区域，即使总体视图数多，真正能够形成独立约束的视图仍少。渲染误差低的区域不一定具备可观测的几何；未见区域可能由先验补全，但补全不等于证据支持。可迁移的关键是“共享场+明确投影算子+多观测约束”，不是直接复制原网络的视图数或超参数。

### 3. 从普通 RGB 场到含介质的逆渲染：改变的是观测方程

【原文事实】§4 Eq.(1)–(3) 的透射由学习密度定义，颜色依赖空间和方向；没有显式分离对象、介质消光、散射源、材料与光照。附录 C 的 NDC 又说明学习坐标并非总是米制。冻结报告 §3.5.1、§3.5.4、§4.1 对此已有边界说明。

【背景知识与研究拓展】以均匀介质、单波段、单个可见表面、固定有效背景辐射为简化模型，可写 $I=Je^{-\beta z}+A(1-e^{-\beta z})$，其中 J 是无介质表面辐射，A 是简化路径辐射的极限值，$\beta$ 是有效消光系数，z 为路径长度。此式不是 NeRF 的原公式，亦非一般水下光谱/多次散射模型。它显示即使 z 已知，若 J、A、$\beta$ 均未知，一次颜色观测仍不足。假定 A 已知且相同表面在两距离具有同一 J，且两次观测非零对比并同号，则

$$
\frac{I_1-A}{I_2-A}=e^{-\beta(z_1-z_2)},\qquad \beta=-\frac{\log[(I_1-A)/(I_2-A)]}{z_1-z_2}.
$$

需要 $z_1\ne z_2$。若目标存在方向外观变化、光照变化或 A 不恒定，求出的系数可能吸收其他成像差异；接近 A 时对比低，取对数也会放大噪声。该例给出独立距离约束为什么有用，以及为何稀疏观测下的介质/几何歧义值得单独研究，不代表本文已验证水下迁移。

实际迁移应先定义物理量、尺度与观测协议，再决定表示与算法。若改用非度量坐标，需要按 CONCEPTS 的重参数化推导处理光学厚度；若相机尺度不知，$\beta z$ 的尺度歧义无法靠增加颜色通道自动消除。可以利用 NeRF 的可微积分连接场与图像，同时需要独立物理约束或数据，不应简单把其 $\sigma$ 命名为介质真实消光。

### 4. 表达、求积与推断是三个不同瓶颈

| 机制 | 可迁移内容 | 成立条件与障碍 |
|---|---|---|
| 位置/方向分支（§3） | 限定跨视图共享量与方向相关量 | 共享属性必须适合真实成像；约束过强会产生系统偏差，过弱则可能让外观吸收几何错误 |
| 频率映射（§5.1） | 显式控制高低频表征 | 需要足够观测和求积精度；高频也能拟合错误场 |
| 分层采样（§5.2） | 用低成本粗估计分配后续查询 | 粗估计需可靠，弱可见/低权重不代表不存在；样本数量与新相机数量不是同一预算 |
| 可微渲染（§4–5） | 连接参数、观测与梯度 | 正向物理模型必须适配任务；低残差不保证正确参数 |
| NDC（App.C） | 用有限坐标域重新分配远景表示 | 原论文针对前向场景；真实距离与密度变换要另外处理 |

【独立评价】表达瓶颈指模型不能表示正确场；求积瓶颈指给定正确场也不能准确渲染；推断瓶颈指很多错误场同样解释观测。三者可能同时存在，不能仅从图像误差判断哪一个主导。一个诊断顺序：先在已知合成场上验证求积收敛，再固定真值几何测试表示外观的能力，最后释放几何和外观评估逆问题；每一步均对照独立真值或可验证参考。若只比较 PSNR，错误机制可能互相补偿而无法归因。

### 5. 研究边界与优先方向

建议先解决约束不足的诊断与评价问题，再研究约束/先验设计；可解释传输模型在参与介质条件下尤为相关，但不是由原论文实验证明的通用结论。效率方向也有本文明确动机（§7），可与可靠性协同研究，但须区别采样加速和观测信息增加。此顺序只是以证据可获得性与实验可控性为依据的建议，非已有研究优先权或新颖性结论。后续应在独立授权下核查相应文献，尤其需要避免把已发表的稀疏视图、介质模型、采样策略误认为首次提出。


## RESEARCH-PROBLEMS：可证伪问题候选与验证路线

以下均为阅读者提出的候选问题，不宣称原创、不声称已验证或完成相关文献调研。以冻结报告的具体证据和前述补充推导为出发点，先提出可观察量，再提出假设；不是预先选定方法然后寻找适用任务。

### RP-A：观测覆盖如何决定几何可辨识性？

**现象与科学未知。** Tables 2/6 中减少输入图数伴随平均 NVS 质量下降；§3.1 的单射线反例与 CONCEPTS 的等价族说明图像监督允许密度与颜色相互补偿。本文没有真值几何误差或视角分布控制实验。因此未知的是：在相同输入数量下，几何约束的独立性是否能比数量更好地预测恢复可靠性，以及方向外观自由度如何改变这一关系？

**可证伪假设 H-A。** 对已知静态场景、相机和足够纹理，在同样 M 张训练图、相同噪声及分辨率下，有更多独立视差且可见覆盖充分的配置，会降低可见区域的几何误差和多次优化解之间的几何差异；但若方向颜色允许过度补偿，RGB 改善与几何改善的相关性会减弱。该假设不要求特定方法，且允许大基线引发更多遮挡而使趋势失效。

**实验设计。** 构造具有已知表面、深度、纹理及可控非漫反射的合成场景；将 M 设为 3/6/12/25 等候选值（这是新实验设定，并非论文原配置），在各 M 下比较成簇相机、广覆盖相机、相同覆盖但不同基线配置。保持目标测试视角固定，按共同可见区域、仅部分训练可见区域、训练未见区域分层统计；对原始 NeRF 分支与方向无关颜色、受限方向容量分别运行，保持训练预算并记录参数差异。每组至少使用多次视图选择和初始化，固定随机种子清单，并同时报告成功与失败例。

候选观测诊断量：对局部几何参数 g，在允许外观参数 a 调整的局部线性模型 $\Delta y=J_g\Delta g+J_a\Delta a$ 中，外观可解释的子空间投影 $P_a=J_aJ_a^+$（Moore–Penrose 伪逆），几何扰动最小可见残差为

$$
\min_{\Delta a}\|J_g\Delta g+J_a\Delta a\|_2^2=\|(I-P_a)J_g\Delta g\|_2^2.
$$

因此局部有效几何信息可用 $H_g=J_g^\top(I-P_a)J_g$ 的非零奇异/特征谱或预先定义方向上的二次型诊断；若 $J_g\Delta g$ 落在外观列空间，扰动一阶不可辨。这里假设局部无约束最小二乘且固定采样，无噪声加权；实际需归一化参数尺度、正则化或计算受限的局部雅可比。它不是论文已有评价指标，也不等同于全局唯一性或真实 Fisher 信息（噪声模型需另设）。可先用低维显式几何代理验证，再扩展到神经场，避免在大型 MLP 中直接求全量伪逆。

**指标与判据。** RGB 用 PSNR/SSIM/LPIPS；几何用可见像素深度 MAE/RMSE（米制或明确尺度对齐）、表面距离、跨初始化几何离散度；覆盖与视差作为自变量而非事后选择。提前定义场景级配对差值及置信区间；若同 M 的覆盖/视差改善在多数控制场景不能降低几何误差，或诊断量与几何误差无稳健关系，则 H-A 不受支持。若 RGB 改善却几何不改善，仍是有效结果，不能删除。若只改善了训练集重建，则未证明泛化到留出视角。需要排除位姿误差、场景尺度、训练不足、表面提取阈值等混杂。真实场景真值取得成本高，先用可控合成验证，后再评估外推性。

### RP-B：学习密度与真实传输参数何时可以区分？

**现象与科学未知。** 原文 Eq.(1)–(3) 通过密度定义透射；§3 颜色允许方向变化；App.C 使用 NDC。本文不提供介质分离真值，因此不支持“密度就是真实消光”的主张。前述常值段与参与介质例显示不同光学厚度、表面辐射和路径辐射可能产生相同 RGB。未知的是：在给定度量尺度与观测数量时，哪些额外传输/几何证据能让分解对噪声和观测变化稳定？

**可证伪假设 H-B。** 在简化均匀介质且度量尺度已知的设置中，相比仅有 RGB，增加独立路径长度变化与已知参考辐射（或独立透射观测）可降低介质系数估计误差；若方向外观或介质非均匀未被模型表达，这种改善可能消失。假设不预设具体参数网络，也不将 NeRF 的 σ 当作介质系数。

**实验设计。** 第一层用已知 z、J、A、β 的单通道合成数据验证 RESEARCH-DIRECTION 的双距离解析估计：固定 β、改变距离差、对比度和高斯测量噪声；比较单距离自由拟合、双距离且 A 已知、双距离且 A 未知、加入独立透射/参考观测。第二层构造有独立几何和介质真值的多视角图像，比较通用有效密度/颜色场与显式对象—介质分解候选表示；两者可直接比较 RGB 和几何，只有定义了可比较物理量时才比较介质误差。显式模型内对照是否给真实尺度、额外参考观测、距离覆盖。先用准确前向模型，再引入非均匀介质、视角外观或模型误差，观察估计是否吸收未建模因素。保持图数、训练预算、噪声、场景、光照不变并报告模型容量。若引入 NDC，明确积分对应的物理弧长转换；若尺度未知，只评价尺度不变量或显式讨论尺度歧义。

**定量预期与反证。** 在已知 A、J 的简化模型中，$I-A=(J-A)e^{-\beta z}$，对 β 的敏感度为 $\partial I/\partial\beta=-z(J-A)e^{-\beta z}$。设独立同方差噪声 $\epsilon\sim\mathcal N(0,s^2)$，局部信息量为 $\mathcal I(\beta)=\sum_k z_k^2(J-A)^2e^{-2\beta z_k}/s^2$。单次测量的敏感度幅值在 β>0 时随 z 先升后降（极值 z=1/β）；太近变化弱，太远信号低。该推导仅在其他参数已知、噪声独立且线性化附近成立；若 J、A 未知则必须联合评估，不能直接套用单参数信息量。实际指标应包括 β 的绝对/相对误差、独立透射曲线误差、几何误差、无介质外观误差与 NVS RGB，以及跨初始化方差。若附加观测不能降低物理误差，或者仅 RGB 更好但物理误差没有改善，则不支持该假设；若仅在理想模型有效，应明确领域边界。真实介质的波长依赖、散射和光照可破坏上述简化，尚需文献和实验核查。

### RP-C：如何在粗模型不可靠时配置有限查询预算？

**现象与科学未知。** §5.2 Eq.(5) 的细采样由 coarse 的当前权重指导；Table 2 在 256 总查询下层次采样优于单网络对照，但网络数量和参数并非完全相同。CONCEPTS 的遮挡梯度与低权重概率示例提供粗估计错误可能影响后续训练的机制。未知的是，在等查询预算下，探索对初始不可靠的场能否改善细薄或弱可见结构的发现，同时保持常规渲染效率？

**可证伪假设 H-C。** 在训练早期或低权重但真实存在结构的条件中，保留粗样本并向细分布加入少量均匀探索，可能降低持续漏检和深度误差；当粗场已经准确时，过多探索会降低有效积分密度或提高误差。这里的探索只改变射线内部查询，不会增加相机观测信息，故不应声称解决未观测区域的几何歧义。

**实验设计。** 分开两个阶段：先固定已知连续场（解析薄层、前后遮挡、低密度与高频场），以高精度数值积分为参考，比较原权重、纯均匀、新增混合分布，记录积分误差与覆盖，单独控制粗权重受人为扰动的程度。随后训练相同粗细网络，固定每射线64+192查询（细包含128新增，合计256），比较 $\lambda=0$、固定小 λ、随迭代衰减 λ，保持网络参数、优化器、训练射线、随机种子和视角协议一致。另报告 GPU 实测时间与显存，不将查询等价冒充速度等价。原论文没有明确规定全零权重的保护规则；所有候选使用同一预先规定的保护，避免混杂。对于正确但真实遮挡不可见的结构，不要求模型仅凭 RGB 恢复它；评价限定在有独立观测证据的细结构或固定真值场求积。

**指标与判据。** 固定场评价每射线 RGB 积分绝对误差、与高精度参考的深度矩误差、真实贡献区间命中率；训练评价细结构掩膜内 RGB/几何误差、全图指标、达到指定误差阈值的时间、不同种子失败比例。以等预算原权重为对照，若弱结构错误或漏检没有稳定降低，或总体质量显著下降而计算无优势，则否定相应配置的实用价值；若固定场积分改善但训练无改善，则不能将求积优势推广为逆问题优势。后续才考虑利用可量化不确定性动态分配 λ，需要另行校准其与实际误差的关系。混合采样是否已经被其他研究提出尚未检索。

### 优先级与依赖关系

| 优先级（暂定） | 候选 | 依据 | 主要风险 |
|---|---|---|---|
| 1 | RP-A | 直接对应视图数消融的证据缺口；可用可控相机和已知几何开展，诊断有助于其他候选 | 局部雅可比依赖参数化，覆盖与遮挡难完全独立 |
| 2 | RP-C | 原方法已有等查询基准，固定场可先排除逆问题混杂，实验成本相对可控 | 改善可能只在特定粗误差条件下成立，已有工作可能覆盖 |
| 3（若任务明确含介质可提升） | RP-B | 可解释性是原文承认的开放方向；简化传输具备可证伪解析例 | 领域迁移证据不足，独立真值及尺度获取难 |

排序依据是当前单论文证据与实验可行性，不是学术重要性或发表价值的排名。三者可独立验证：RP-A 侧重观测约束，RP-C 侧重数值/优化预算，RP-B 侧重观测方程与参数解释。若没有独立几何和物理真值，不能靠 NVS 单项指标完成 RP-A/B 的证实。任何研究选择前都应另行授权进行相关文献核查与实验成本评估。


## INTERNALIZATION：建立可独立复述的技术认知

### 1. 不看原文，如何讲清楚 NeRF？

NeRF 的核心是将同一静态场景表达成可查询的连续函数，再以可微的体渲染把该函数与已经观测到的图像连接起来。输入图像不是网络逐次查询时的输入特征，而是训练监督；相机决定每条射线的空间位置和方向，网络在射线上回答“这里有多少终止风险、向这个方向呈现什么颜色”。积分将这些局部答案合成像素，残差反向传播到共享网络参数。训练结束后更换相机而不更换场，即可合成新视角。

三项主要设计分别针对不同瓶颈：密度共享而颜色具有方向依赖，约束不同视角之间的空间结构并允许非漫反射；位置编码使普通 MLP 更容易表达高频；层次采样根据粗模型的贡献权重，把有限查询更集中于当前可见内容。它们与经典体渲染共同构成系统，不能将任意一项单独视为完整贡献。证据见 §3–5、Fig.2–4、7、Table 2 及冻结报告 §3.4–3.6。

### 2. 从输入到输出的概念地图

| 阶段 | 已知输入或中间量 | 运算 | 获得的知识及边界 |
|---|---|---|---|
| 观测 | RGB、相机内外参、范围 | 构造射线 | 知道像素对应的空间路径，不知道路径上的真实表面 |
| 表示 | 位置和方向 | 正余弦编码、位置主干及颜色支路 | 连续场；位置密度共享，不是独立物理介质估计 |
| 渲染 | 有序采样点、密度、颜色 | 计算光学厚度、alpha、透射率、贡献权重 | 预测 RGB，可微；数值误差与场误差要分开 |
| 层次采样 | coarse 的贡献权重 | 归一化、重采样、合并，fine 重新查询 | 增加当前重要区间的查询，不增加真实图像观测 |
| 优化 | 粗细预测、真值像素 | 两路平方误差与反向传播 | 同一场景的网络参数；不构成唯一几何证明 |
| 测试 | 新相机、训练后参数 | 再次查询与积分 | 新图像；对未观测区域的准确性需独立检验 |

该地图对应原 Eq.(1)–(6)、Fig.2、7 及报告 §3.4–3.5。NDC 属于射线空间参数化，而非额外监督信号。

### 3. 自检：从公式独立推出结果

**练习 A：两段遮挡。** 取 $\alpha_1=0.25,\alpha_2=0.8$、$c_1=0.2,c_2=0.9$，不加远端背景。$T_1=1,T_2=0.75$，$w_1=0.25,w_2=0.60$，预测 $C=0.05+0.54=0.59$，剩余透射 $0.15$。交换两段顺序时，$C'=0.8\times0.9+0.2\times0.25\times0.2=0.73$；总透射仍为 0.15。颜色合成与顺序有关，透射乘积与同一组区间的顺序无关。此构造例基于 Eq.(3)，不是论文的实验数据。

**练习 B：查询预算。** 64 个 coarse 查询与 128 个额外 fine 点合并为 192 个 fine 查询；总数 256，而非 192，也不是将 128 个点单独渲染再与粗图融合。800×800 图像按此预算需要 163,840,000 次点级网络查询（按每像素一条射线；不是运行时间测量）。由此理解紧凑参数存储与高计算成本为何可以同时存在。依据 App.A 与冻结报告 §3.4.4。

**练习 C：图像正确而几何不确定。** 对零背景单段，$C=0.4$ 可由 $\alpha=0.5,c=0.8$ 或 $\alpha=0.8,c=0.5$ 获得。若位置仍可变化、单射线仍只观察同一颜色，也不能由该颜色确定深度。多个视角的共享密度可缩小解空间，但需要覆盖和可靠外观；结论是“不能仅据 RGB 断言唯一真值”，不是“多视角毫无作用”。参见 CONCEPTS 的等价族推导。

**练习 D：参数单位。** 若空间物理距离缩放 $s'=ks$，$k>0$，为了使 $\sigma' ds'=\sigma ds$，应有 $\sigma'=\sigma/k$。因此未知尺度时光学厚度可保持不变而消光数值改变；非线性 NDC 的变化更不能用一个常数普遍处理。此为补充量纲检查，不是作者报告的介质实验。

### 4. 易错概念自检

| 易错表述 | 更准确的理解 |
|---|---|
| 输入是一组图片，输出直接是三维模型 | 图片与位姿用于逐场景优化；查询输入是位置与方向，输出是密度与颜色；新图像需渲染 |
| 5D 意味着网络接收五个原始数 | 三个位置自由度和两个方向自由度，实践用单位方向向量并编码，Fig.7 给出 60/24 维输入 |
| alpha 等于密度 | alpha 是区间条件终止概率，依赖密度和长度；T 是前序透射；权重还乘 T |
| fine 按密度最大处取样 | coarse 的权重包含可见性，重采样依据归一化的贡献分布 |
| 位置编码补充了高频观测 | 它提供高频表示能力，不提供额外测量；高频可拟合错误场 |
| PSNR 上升意味着几何和物理参数均改善 | 只支持图像误差的变化；需独立几何/传输证据 |
| 25 个视图实验证明极稀疏视角问题已经解决 | 本文只检验特定数据下 25/50/100 图，未检验 1–9 图等设置 |
| 所有数据和指标均优于基线 | Table 1 真实前向 LPIPS 存在例外；逐场景表也需看例外 |

### 5. 将论文理解转化为研究能力

遇到另一种场表示，先问它是否改变可观测数据或只是表达自由度；遇到加速算法，先分开固定场积分误差与训练后的场误差；遇到水下或其他参与介质，先确认单位、路径和可解释物理量，然后判断图像监督是否能独立约束它们。进一步研究时要先给出能够推翻假设的实验，而不是只找一个更高的 NVS 指标。这里的启发属于本笔记的研究推论。

仍未解决的关键问题：哪些观测几何真正增加独立信息？如何在外观补偿下诊断几何错误？如何在非度量坐标中保存真实传输意义？哪些采样失败来自粗场误判，哪些来自完全没有观测？这些问题已在 RESEARCH-PROBLEMS 提出可检验路径，但尚无新实验或跨文献证据。阅读完成的标准是能独立说明这些区别，而不是把开放问题当作作者已经解决的内容。


## GLOSSARY：术语与概念边界

以下术语按论文机制及本笔记的研究拓展分组。前两组主要来自原文；第三组部分为背景知识与补充研究术语，并非声称作者均使用这些词。

### 1. 场表示与几何

| English | 中文 | 技术含义及相近概念区别 | 证据/关联 |
|---|---|---|---|
| Neural Radiance Field (NeRF) | 神经辐射场 | 以神经函数表示位置密度与方向相关颜色；不是单一图像映射 | §3、Fig.7 |
| Implicit representation | 隐式表示 | 通过函数查询场值，不直接存储规则体素或表面；连续查询不保证实际高频准确 | §3、§2 |
| Radiance | 辐亮度 | 物理上描述特定方向传播的辐射；本文网络颜色为归一化 RGB 表示，不保证物理辐射标定 | §3–4 |
| Volume density | 体密度 | 控制单位射线参数长度的终止风险；不是 alpha 或真实质量密度 | Eq.(1)–(3) |
| View-dependent appearance | 视角依赖外观 | 同一位置不同方向可有不同颜色；不是随视角改变共享密度 | §3、Fig.3 |
| Lambertian | 朗伯型 | 理想漫反射在给定照明下的出射辐亮度不随观察方向变化；不是所有材质的普遍性质 | Fig.4、§3 |
| Positional encoding (PE) | 位置编码 | 倍频正余弦映射，为 MLP 提供高频输入，不增加观测数据 | Eq.(4)、§5.1 |
| Spectral bias | 频谱偏置 | 网络学习不同频率难度不同的现象；此处解释编码动机，而非水下光谱 | §5.1 |
| Skip connection | 跳跃连接 | 将编码输入拼接至较深层，区别于将输出图像残差相加 | Fig.7、App.A |
| Camera pose | 相机位姿 | 描述相机相对于场的旋转和平移；内参另外决定像素方向 | §5.3、报告 §3.1 |
| Novel view synthesis (NVS) | 新视角合成 | 给定目标相机生成图像；与直接估计深度、材质的任务不等同 | §1、Fig.1 |
| Normalized device coordinates (NDC) | 归一化设备坐标 | 前向场景使用的投影坐标域；逆深度相关，不是米制坐标或 RGB 归一化 | App.C Eq.(7)–(26) |

### 2. 渲染、采样与训练

| English | 中文 | 技术含义及相近概念区别 | 证据/关联 |
|---|---|---|---|
| Ray parameter | 射线参数 | $t$ 决定沿射线位置；只有匹配单位方向与度量坐标时可直接解释为距离 | Eq.(1)、App.C |
| Optical thickness | 光学厚度 | $\tau=\int\sigma dt$，无量纲；与局部密度不同 | Eq.(1)、补充推导 |
| Transmittance | 透射率 | $T=\exp(-\int\sigma dt)$，到该处前未终止的比例 | Eq.(1) |
| Opacity / alpha | 不透明度 | $1-e^{-\sigma\delta}$，区间条件终止概率；不同于累积透射 | Eq.(3) |
| Rendering weight | 渲染权重 | $w_i=T_i\alpha_i$，包含前序遮挡；不直接等于颜色 | Eq.(3)、(5) |
| Alpha compositing | Alpha 合成 | 按有序区间累加权重乘颜色；不是等权平均 | Eq.(3) |
| Volume rendering | 体渲染 | 沿空间路径积分局部贡献；本文采用发射—吸收式机制而非完整散射求解 | §4 |
| Stratified sampling | 分层随机采样 | 均匀划分区间、每区间抽点，区别于全区间独立无约束抽样 | Eq.(2) |
| Hierarchical sampling | 层次采样 | 先粗估贡献分布，再抽细点并合并，fine 重新计算场值 | §5.2 Eq.(5) |
| Probability density function (PDF) | 概率密度函数 | 表达连续采样位置的概率密度；区间概率为其积分，不是文档 PDF | §5.2 |
| Inverse transform sampling | 逆变换采样 | 利用累计概率的反函数获得目标分布的样本 | §5.2、报告 §3.5.3 |
| Numerical quadrature | 数值求积 | 用离散样本近似连续积分；求积精度与场恢复准确性是不同问题 | Eq.(2)–(3) |
| Differentiable rendering | 可微渲染 | 渲染输出对场参数可求梯度，从图像误差优化表示 | Fig.2、Eq.(3)、(6) |
| Scene-specific optimization | 逐场景优化 | 每场景学习相应网络权重，不是对新场景单次前馈 | §5.3 |
| Mean squared error (MSE) | 均方误差 | 平均平方误差；原训练式写成粗细 RGB 平方误差之和，平均与求和尺度需区分 | Eq.(6) |
| PSNR | 峰值信噪比 | 在同一像素范围和评价协议下越大通常越好 | Tables 1–6 |
| SSIM | 结构相似性 | 比较图像局部结构等相似性，通常越大越好 | Tables 1–6 |
| LPIPS | 学习感知图像块相似度 | 使用学习特征衡量感知距离，越小越好；不是真实几何指标 | Tables 1–6 |

### 3. 研究拓展术语

| English | 中文 | 技术含义与使用边界 |
|---|---|---|
| Inverse problem | 逆问题 | 由投影或测量反推场；正向可计算不代表反解唯一 |
| Observability | 可观测性 | 这里指某个变量的变化能否在给定观测中产生可辨信号；与单纯是否可见有联系但不等价 |
| Identifiability | 可辨识性 | 给定模型与观测是否足以区分候选参数/场；局部一阶可辨不证明全局唯一 |
| Null space | 零空间 | 线性算子映射为零的方向；局部雅可比零方向表示一阶观测不变 |
| Gauge / scale ambiguity | 规范/尺度歧义 | 参数同时变化而观测不变；如距离和消光的反比例缩放，需额外尺度信息 |
| Epistemic uncertainty | 认知不确定性 | 信息或模型不足导致的不确定性；多初始化差异只是一种诊断，不等于已校准后验 |
| Aleatoric uncertainty | 数据不确定性 | 给定条件下的噪声或随机性；不能与网络预测误差直接画等号 |
| Participating medium | 参与介质 | 在路径上发生吸收、散射等作用的介质；本文并未显式分解它与对象 |
| Extinction coefficient | 消光系数 | 物理空间单位长度的消光率，必须有度量单位；不能自动用归一化 NeRF 密度替代 |
| Model mismatch | 模型失配 | 假定的成像或表示机制与真实过程不符；增加观测可能不能消除系统偏差 |
| Fisher information | Fisher 信息 | 在给定统计模型下衡量参数局部信息；RP-B 的简化推导依赖已知其他参数与噪声模型 |
| Exploration–exploitation | 探索—利用 | 查询预算在未充分探测区与已知高贡献区间之间分配；不增加真实视角观测 |

术语理解以原文定义及公式为主；研究拓展术语的使用条件见 CONCEPTS、RESEARCH-DIRECTION、RESEARCH-PROBLEMS。未开展其他论文术语沿革调查。


## SOURCES：证据边界与来源记录

### 1. 固定论文及依赖

- Ben Mildenhall, Pratul P. Srinivasan, Matthew Tancik, Jonathan T. Barron, Ravi Ramamoorthi, Ren Ng. *NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis*. ECCV 2020。
- 阅读版本：arXiv:2003.08934v2（2020-08-03），[版本记录](https://arxiv.org/abs/2003.08934)、[固定版本 PDF](https://arxiv.org/pdf/2003.08934v2)。实际来源为用户提交的同版本 HTML/PDF 资源 ZIP；以 PDF 为权威，HTML 仅辅助。B1 于 2026-11-26 核验版本元数据；本阶段不重新联网。
- 直接依赖：`NeRF_精读报告.md`（B3 冻结报告），SHA-256：`044621e97802f3de55d5a1a7c7928481342fc9c7626d9e48b3f3faa838066470`。报告覆盖正文 §1–7、附录 A–D、Figures 1–8、Tables 1–6 与 Eq.(1)–(26)。B4 未改变该报告或原始资源。
- 官方资源身份在 B1 核验；代码、数据内容、补充视频均未在本阶段运行、下载或独立验证。[项目页](https://www.matthewtancik.com/nerf)、[代码](https://github.com/bmild/nerf) 仅供后续定位，不用其内容补写论文未报告的实现。

### 2. 证据分类与使用范围

| 分类 | 含义 | 本笔记实例 |
|---|---|---|
| 原文事实 | 来自固定版本的机制、实验数值和作者主张，以冻结报告相应定位为锚 | Eq.(1)–(6)、分支结构、Table 2 视图数消融 |
| 背景知识/补充推导 | 通用数学、几何、物理定义与明确假设下的构造例 | 单段等价族、双目误差、重参数化雅可比 |
| 合理推断 | 从上述机制形成的有条件解释 | 粗场误判可能影响查询与梯度 |
| 独立评价 | 阅读者分析证据能否支持具体结论 | NVS 不等同于独立物理或几何验证 |
| 尚未证实/研究假设 | 尚待实验、比较与文献检索 | RP-A/B/C、探索采样、领域迁移 |

CROSS-PAPER 仅建立当前论文的潜在关联索引，不读取或比较其他论文、不构建全局知识库。未核查其他工作的优先权、新颖性或当前最优结果。研究候选是一般可公开的学术分析，并非私密研究方案或已确定的个人研究主线。若开展真正跨论文比较，需另行授权与核验。

### 3. 继承限制与本阶段未验证项

B3 中记录的 HTML 缺失 Figure 6 的 4 张对比图片已由同版本 PDF 完整区域保留；B2 的 1 幅 PASS_EXACT、7 幅 PASS_WITH_NOTES 状态不变。未验证实际客户端或 Hexo 渲染；未运行代码、复现实验或核查补充视频。本文的真实数据 LPIPS 例外、逐场景反例及分辨率记载差异均不因学习笔记而改变。补充推导使用明确的简化假设，不能推广为一般物理定律或原文证明。B4 的审计为模型内容核查加程序文件完整性校验，不是独立同行评审。发布快照仅供后续生成 Full 使用，隐藏页面不等于访问控制。



