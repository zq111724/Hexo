---
title: "MVSplat：从稀疏多视角图像高效预测三维高斯（完整精读与研究笔记）"
date: 2026-11-26 21:13:55
index_img: "/MVSplat/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/MVSplat/)

# 第一部分：论文精读报告

# MVSplat 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2403.14627v2（2024-07-18）；缓存记录 v2（2024-07-18）；最新版未能确认。核验日期：2026-11-26。

**官方资源：** [arXiv](https://arxiv.org/abs/2403.14627) | [PDF](https://arxiv.org/pdf/2403.14627v2) | [Project Page](https://donydchen.github.io/mvsplat/) | [Code](https://github.com/donydchen/mvsplat) | [Publication](https://www.ecva.net/papers/eccv_2024/papers_ECCV/html/3187_ECCV_2024_paper.php)

<!-- PAPER-METADATA END -->


> 论文：MVSplat: Efficient 3D Gaussian Splatting from Sparse Multi-View Images
> 版本及来源：arXiv:2403.14627v2，用户上传 arxiv_2403.14627v2.zip。
> 实际访问范围：PDF 23 页，正文、参考文献、附录 A–C。
> 载体：PDF 为最终证据；HTML 与 Source 仅辅助，Source 无独立版本印记；Figure 为 PDF 区域展示副本。
> 证据规则：原文事实、背景知识、合理推断、独立评价、尚未证实分开标注。

<!-- PTH:BEGIN INIT -->
## 0. 来源、结构与完整目录

### 论文身份与来源

【原文事实，PDF p.1】正式标题为 MVSplat: Efficient 3D Gaussian Splatting from Sparse Multi-View Images。作者依次为 Yuedong Chen、Haofei Xu、Chuanxia Zheng、Bohan Zhuang、Marc Pollefeys、Andreas Geiger、Tat-Jen Cham、Jianfei Cai。PDF 首页印有 arXiv:2403.14627v2 [cs.CV] 18 Jul 2024。实际使用用户提交的该版本 PDF，23 页，SHA-256 为 `d1becb5361d7d8aa6f2e47d1e9e257031120b4207834f5effca049a8df7e4bb9`。HTML 可用于定位文本；Source 与 PDF 标题作者一致，但没有独立版本印记，二者都不是替代 PDF 的权威依据。arXiv 官方缓存为 v2，实时最新版未独立确认。

### 章节与覆盖范围

| 内容 | PDF 页码 |
|---|---|
| 标题、摘要 | 1 |
| 1 Introduction | 1–3 |
| 2 Related Work | 3–4 |
| 3 Method | 5–7 |
| 4 Experiments | 7–14 |
| 5 Conclusion | 14 |
| References | 15–17 |
| A More Experimental Analysis | 18–20 |
| B More Visual Comparisons | 20–22（图浮动至 p.22） |
| C More Implementation Details | 21–23 |

### Figure/Table 目录

- Figure 1 — PDF p.1。
- Figure 2 — PDF p.4。
- Figure 3 — PDF p.9。
- Figure 4 — PDF p.10。
- Figure 5 — PDF p.11。
- Figure 6 — PDF p.13。
- Figure A — PDF p.20。
- Figure B — PDF p.20。
- Figure C — PDF p.21。
- Figure D — PDF p.22。
- Figure E — PDF p.22。
- Table 1 — PDF p.8。
- Table 2 — PDF p.11。
- Table 3 — PDF p.12。
- Table A — PDF p.18。
- Table B — PDF p.18。
- Table C — PDF p.19。
- Table D — PDF p.19。

正文 6 图 3 表，附录 5 图 4 表，共 11 图 7 表。未发现独立编号 Algorithm。全部 Figure 已对照 PDF 区域、区域 PDF 回渲染和 JPEG 进行视觉核查；8 幅精确通过、3 幅带记录通过。图像实用保真通过不等同于所有像素或全部内联流都可证明无损。此单元仅建立来源和目录，不涉及方法机制或数学推导。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1. 标题解读

### 标题翻译

MVSplat：从稀疏多视角图像实现高效三维高斯泼溅。

### 术语拆解

- MVS 是 multi-view stereo（多视角立体视觉）。它利用同一三维点在不同已知相机中的投影关系估计深度。本论文将这种几何匹配思路用于高斯中心预测；具体见 §3.1。Splat 指高斯投影后的泼溅式栅格化。
- 3D Gaussian Splatting（3DGS）是由带中心、空间协方差、透明度与颜色参数的三维高斯组成的显式场景表示。每个高斯可投影至图像平面，再进行可微合成；因此重建表示与目标相机渲染是两个相连但不同的阶段。
- Sparse Multi-View Images 指少量不同视角的输入图像。§1 明确以少至两张为目标，§3 的输入还包括相机参数；稀疏不表示输入像素本身稀疏。
- Efficient 指在本文实验条件下兼顾推理耗时、参数量与画质，而不是声称任意分辨率或任意硬件均达到固定帧率。

### 标题的边界

仅由标题可知研究对象、稀疏多视角输入及效率目标；不能由标题推出具体深度公式、相机已知条件、是否前馈、训练监督或量化速度。这些必须通过摘要、§3、§4 确认。此单元解释术语而不展开数学推导。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 2. 摘要精读

### 问题与困难

【摘要 p.1，§1 pp.1–3】作者研究少量多视角图像的一次前向推理：直接预测能够进行新视角渲染的 3D 高斯。难点首先是中心定位，而非仅获得高分辨率图像。给定一个参考像素，单个视角主要确定一条空间射线，无法唯一确定沿射线的位置；另一视角可通过对应关系缩小深度范围。直接从图像特征回归深度分布不一定充分利用这种几何关系（§1，pixelSplat 对比）。错误深度可能表现为漂浮高斯和不稳定的几何。

### 方法与因果链

作者先进行平面扫掠：对参考视角的多个候选深度，按相机几何把其他视角的特征采样到参考像素位置。对齐后的特征相似度组成代价体。若候选深度贴近真实表面并且跨视角可见，匹配通常更好；网络可借此估计深度，而非无结构地回归三维位置。随后用 2D 网络精炼代价体，由深度反投影确定中心，同时预测协方差、透明度与球谐颜色。组合所有视角的高斯进行可微栅格化。训练使用新视角真实 RGB 与渲染 RGB 的图像损失，不需要显式深度真值监督。该流程的原文公式为 §3 Eq.(1)–(6)，后续专章展开。

### 结果与边界

摘要报告 RealEstate10K 和 ACID 上的领先画质、约 22 fps 前馈推理、相对 pixelSplat 约十分之一参数及超过两倍速度，并指出几何表现及跨数据集泛化改善。Table 1 的条件是 256×256、每场景两张输入、三个新视角用于评估；MVSplat 0.044 s、12.0 M，pixelSplat 0.104 s、125.4 M；耗时包含编码器与渲染，而非单纯渲染帧率。RealEstate10K PSNR 为 26.39 vs 25.89 dB，ACID 为 28.25 vs 28.14 dB。几何改善由 Figure 4/D 的可视化支持，不代表已通过真实深度误差的量化验证。泛化见 Table 2 和 Figure 5/E，不能推出对任意域稳健。

【理解】代价体不是保证真实几何的硬约束，而是把已知投影几何转化成网络可学习的候选深度证据。数学推导在后续专章给出；此处摘要没有独立公式。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 3. 贡献分析

### 几何匹配式中心预测

【原文 §1、§3.1】核心变化并非提出一种新的 3DGS 原语，而是重新组织从图像到中心位置的学习问题。pixelSplat 直接从跨视角特征预测深度分布，本文先按已知相机对各深度候选做平面扫掠（Eq.2），计算跨视角特征点积相关性（Eq.3），沿深度堆叠成代价体（Eq.4）。再预测残差精炼代价体（Eq.5），用 softmax 后的候选深度期望估计深度（Eq.6），最后反投影生成中心。真实对应的深度假设一般有更好的特征匹配，故几何线索可以减少直接回归负担；在遮挡、反射等情况下则不一定成立。

### 轻量联合预测

【原文 §3.2–3.3，Figure 2】用带跨视角注意力的 2D U-Net，而不是直接以昂贵的三维网络处理体素，精炼每参考视角的代价体；将各视角深度反投影后直接取并集，不执行传统 MVS 的复杂融合过滤。透明度、协方差及颜色与深度共同预测；训练通过渲染 RGB 的 L2 与 LPIPS 损失优化，未提供真值深度。编码完成后显式高斯可反复用于目标相机的栅格化。平面扫掠、Transformer、U-Net、3DGS 渲染及球谐表示均有前人基础，贡献在几何匹配式前馈高斯中心学习与轻量集成。

### 实验贡献

【原文 §4、附录 A–B】Table 1 对比画质、时间、参数；Table 2 验证 RE10K 训练模型零样本迁移至 ACID/DTU；Table 3 将代价体、跨视角注意力、U-Net、额外深度精炼分别消融。Table A 将代价体方案接入 pixelSplat 后也得到画质提升，支持该设计不限于本文单一完整网络。Figure 4/D 的几何证据是定性而非深度真值评估。作者没有证明任何特征匹配体系都能得到真实表面，也没有提出从稀疏视角唯一恢复任意场景的理论定理。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 4. 结论解读

### 结论及证据

【作者结论，§5 p.14】MVSplat 利用多视角对应构造代价体学习几何，并用针对高斯预测定制的编码器获得高效前馈重建。作者报告两个大规模场景级基准上的领先表现，与 pixelSplat 相比参数约缩小至十分之一、推理速度超过两倍，且画质、几何及跨数据集泛化更好。Table 1 给出画质及效率，Table 2 给出迁移指标，Table 3 和 Table A 对代价体作用提供实验支持，Figure 4/D 给出几何与深度可视化。这里的关键关系是：几何匹配提供定位依据，正确定位更可能使目标视角投影一致，从而减少重影、模糊等现象；但渲染误差下降不必然等于深度误差下降。

### 局限与未来方向

【原文 §5，附录 A p.20】反射表面（玻璃、窗等）及非朗伯表面可能导致不可靠结果：同一个物理表面在不同视角中的外观并不稳定，因而难以依靠跨视角外观匹配确定同一个三维点。Figure B 展示失败例。作者建议研究包含 BRDF 的渲染以及更加多样化的训练数据；即便 RealEstate10K 规模很大，其多样性也不足以保证真实开放场景稳健。附录强调在自动驾驶等安全关键用途使用渲染数据时应谨慎。

### 结论边界

【独立评价】本文结论是特定实验协议下的效率与画质改善，以及定性几何和有限数据集迁移表现，不是全场景可靠几何恢复的证明。未来方向并未在本文中实际验证。本单元没有新增公式，具体模型推导在方法与数学章节展开。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 5. 图表精读总览

### 正文图表功能

| 图表 | 功能与阅读问题 |
|---|---|
| Figure 1 | 总览效率、质量和高斯结构：为何可更小更快？ |
| Figure 2 | 端到端架构：图像到代价体再到高斯及目标图的路径。 |
| Figure 3 | 域内新视角定性对比：哪些区域渲染改善？ |
| Figure 4 | 高斯与深度可视化：漂浮伪影是否改善？ |
| Figure 5 | 域外零样本定性对比：从 RE10K 到 ACID/DTU 的变化。 |
| Figure 6 | 模块消融定性对照：共同可见区及单视图可见区的差异。 |
| Table 1 | 域内 PSNR、SSIM、LPIPS、耗时及参数量。 |
| Table 2 | 两目标域零样本迁移的图像指标。 |
| Table 3 | 代价体、跨视角注意力、代价体 U-Net、深度精炼的定量作用。 |

### 附录图表功能

| 图表 | 功能与阅读问题 |
|---|---|
| Figure A | 消融验证曲线，观察缺跨视角注意力时的过拟合。 |
| Figure B | 非朗伯及反射表面失败例。 |
| Figure C | 扩展域内定性对比。 |
| Figure D | 扩展几何与深度对比。 |
| Figure E | 扩展跨数据集定性结果。 |
| Table A | 将代价体深度分支引入 pixelSplat 的对照。 |
| Table B | Epipolar 与 Swin Transformer 的效率/效果。 |
| Table C | 单像素预测一个或三个高斯的画质/渲染耗时。 |
| Table D | 初始化与训练步数对画质的影响。 |

图表之间承担不同证据职责：Figure 2 说明机制流程而不直接证明有效性，Table 3/A 用控制对照支持模块贡献，Figure 4/D 仅作定性几何证据；Table 1/2 是图像质量与效率证据（Table 2 不报告时间）。此单元是图表功能索引，不是单独的数学推导；逐图逐表仍需分别审读。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
### Figure 1：几何、画质和效率的总览

![Figure 1](MVSplat/Figures/Figure_01.jpg)

来源：指定版本 PDF 第 1 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_01.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. 1: Our MVSplat outperforms pixelSplat [1] in terms of both appearance and geometry quality with 10× fewer parameters and more than 2× faster inference speed.

【原文事实】MVSplat 相比 pixelSplat，在外观和几何质量上均更优，参数约少一个数量级，推理速度超过其两倍。

#### 布局与逐区域分析

图中央上方是同一室内场景的两张输入图，箭头向左、向右分别进入 pixelSplat 和 MVSplat，而不是将前者输出传给后者。两侧各有一幅拉远观察的高斯场景、两张伪彩深度图、两张新视角合成图。绿色/红色相机框标识场景中的观察位姿；仅凭图不规定两种颜色对应的精确协议。左侧墙体及家具周边存在大量不连续的云团状高斯，深度图有明显颗粒状扰动；右侧墙、门、蓝色地毯的整体轮廓相对连续，深度图更平滑。两张新视角图中，右侧门框、墙上挂件和家具边缘更清晰，左侧局部泛模糊。没有真值深度及几何误差标尺，不能从伪彩颜色恢复绝对米制距离。

中央下方四个柱图用蓝色表示 pixelSplat、绿色表示 MVSplat：参数量从约 125.1M 到 12.0M；时间约 104 ms 到 44 ms；高斯数约 393K 到 131K；PSNR 约 25.9 到 26.4 dB。柱图数值为取整显示；精确时间与主实验 Table 1 的口径应以表和 §4.2 为准。PSNR 越高表示同一参考下平均平方误差越小（不代表深度更准）。参数量和时间越少通常越有利于轻量推理，但不直接等于峰值显存或训练成本。

#### 技术联系与证据边界

【原文事实】§3.1–3.2 将原先依赖图像特征的深度估计改为显式几何匹配代价体，预测深度后反投影成高斯中心；每个像素一个高斯，比较模型每个像素三个。§4.2 报告编码 0.043 s 对 0.102 s、渲染 0.0015 s 对 0.0025 s；效率优势既有编码器结构原因也有高斯数原因，不能只归功于 cost volume。对于两张 $256\times256$ 输入，MVSplat 产生 $2\times256^2=131072$ 个中心，而三个/像素对应 $393216$，与图中的数量级相符。125.1/12.0≈10.43，104/44≈2.36，图中的数量级主张有数值支撑。

【独立评价】该图是 teaser，联合呈现定性与定量优势；它并不是有真值几何监督的深度实验。图像相似度与几何一致性可能解耦，需结合 Figure 4、D 的几何可视化以及 Table 1、3、A–C 的对照判断。图中更光滑的深度本身也可能抹掉细节，不能证明真实几何恢复。

【背景知识】对归一化 RGB，$\mathrm{PSNR}=10\log_{10}(1/\mathrm{MSE})$，相同数据上的 0.5 dB 提升对应 MSE 比约 $10^{-0.5/10}=0.891$，不是几何误差减少 10.9%。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
### Figure 2：MVSplat 总体信息流

![Figure 2](MVSplat/Figures/Figure_02.jpg)

来源：指定版本 PDF 第 4 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_02.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. 2: Overview of MVSplat. Given multiple posed images as input, MVSplat first extracts multi-view image features with a Transformer. Then, the per-view cost volumes using plane sweeping are constructed. The Transformer features and cost volumes are concatenated together as input to a 2D U-Net (with cross-view attention) for cost volume refinement and predicting per-view depth maps. The per-view depth maps are unprojected to 3D and combined using a simple deterministic union operation as the 3D Gaussian centers. The opacity, covariance and color Gaussian parameters are predicted jointly with the depth maps. Finally, novel views are rendered from the predicted 3D Gaussians with the rasterization operation.

【原文事实】MVSplat 以多张带位姿的图像为输入，首先通过 Transformer 提取多视角特征，随后利用平面扫掠构造各视角的代价体。将 Transformer 特征与代价体拼接后输入带跨视角注意力的二维 U-Net，精炼代价体并预测各视角深度。深度图反投影到三维，利用简单确定性的并集运算得到三维高斯中心；透明度、协方差与颜色参数同深度一起预测，最后通过栅格化从预测的高斯渲染新视角。

#### 布局与逐区域分析

从左向右阅读。左端 $I_i,I_j$ 表示两个示意输入视角，省略号表示可接受多视角。浅绿色 Multi-View Transformer 的输入是图像，经 §3.1 的 CNN 下采样与跨视角注意力后得到灰色特征块，各视角仍保留自己的像素网格。中部交叉箭头表示特征匹配，不是把图像拼成一个全局纹理；对参考视角 i，把 j 的特征按照候选深度投影至 i，再与 i 的特征点积，反方向对 j 同样操作。橙色块为每视角 cost volume，沿候选深度组织。灰色块绕过 matching 连到 2D U-Net，表示原特征与代价体拼接，保留外观和上下文信息。

第二个绿色模块为带跨视角注意力的二维 U-Net，用于代价体的残差精炼；图中简化了后续上采样、softmax 期望及第二个深度 refinement U-Net。每视角输出两类信息：上方伪彩深度图（中心的几何依据）；下方 $(\alpha_i,\Sigma_i,c_i)$ 等是透明度、协方差和颜色。斜箭头 unproject 把像素坐标和深度转为共同世界坐标中的 $\mu_i$，其余参数跟随同一像素的高斯。虚线框中 $\cup$ 表示将不同输入视角的高斯集合直接组合，没有画出点云去重、显式融合或逐场景优化。右侧相机图标给定目标位姿，渲染箭头从统一高斯集产生 novel view；不需要为每一个目标重新编码输入。

#### 技术联系与证据边界

【原文事实】§3 Eq.(1) 把 $K$ 个带相机矩阵的图像映射到 $H\times W\times K$ 个高斯。Eq.(2–4) 定义候选深度下的 warp、点积和堆叠，Eq.(5) 精炼，Eq.(6) 从深度分布计算期望。§3.2 给出 opacity 由匹配置信度预测，协方差和颜色从特征、精炼代价体、原图的拼接输入预测；因此图中“2D U-Net”的两输出箭头是总体简图，不能推断所有参数均由单一 U-Net 最后一层直接产生。图中两处跨视角注意力对应特征提取与代价体精炼；深度精炼模块中也使用跨视角注意力。

【背景知识】将相机投影写成 $P^i=K_i[R_i|t_i]$，假设深度 $d$ 是相机坐标 z，齐次像素 $\tilde p$ 反投影为 $X_w=R_i^T(dK_i^{-1}\tilde p-t_i)$；因此同一像素深度的误差直接传播到世界空间中心。若深度定义为射线距离则需要归一化射线，不能混用。匹配证据经过学习的残差与 RGB 渲染损失调整，不是固定 MVS 算法加一个颜色头。

【独立评价】结构图支持端到端前馈流程的理解，不能单凭箭头判断哪一模块贡献最大。Table 3/A 才提供代价体及模块对照；Figure 6 说明仅单视图可见的区域需要上下文补全。并集也不保证重叠高斯完全一致、无浮点或无遮挡冲突。

【原文事实】训练不使用 GT depth，目标图的渲染损失通过可微 rasterization 传向高斯参数和深度估计（§3.3）。这是训练与图示推理链的区别。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
### Figure 3：同数据集新视角合成定性对比

![Figure 3](MVSplat/Figures/Figure_03.jpg)

来源：指定版本 PDF 第 9 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_03.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. 3: Comparisons with the state of the art. The first three rows are from RealEstate10K (indoor scenes), while the last one is from ACID (outdoor scenes). Models are trained with a collection of training scenes from each indicated dataset, and tested on novel scenes from the same dataset. MVSplat surpasses all other competitive models in rendering challenging regions due to the effectiveness of our cost volume- based geometry representation.

【原文事实】与先进方法比较。前三行来自 RealEstate10K（室内），最后一行来自 ACID（室外）。模型在各自所标注的数据集的训练场景上学习，并在该数据集的新场景测试。作者认为基于代价体的几何表示使 MVSplat 在困难区域的渲染优于其他竞争模型。

#### 布局与逐区域分析

共有四行六列。每行前两列是两个输入视角，第三列 MuRF、第四列 pixelSplat、第五列 MVSplat 为同一目标视角的预测，末列 Ground Truth 为目标真值。注意不要用输入列与预测列直接作像素误差对比，输入与目标相机不同。

第一行是窗边餐桌。大面积窗框、远处建筑和椅背具有重复直线纹理，MuRF 窗框模糊、外景重影，pixelSplat 在窗框、椅背附近也有错位或模糊；MVSplat 的窗框与桌椅轮廓相对清楚，但与 GT 仍有差异。第二行为楼梯栏杆；第一输入可见完整栏杆，第二输入几乎不见栏杆。MuRF 和 pixelSplat 的细杆密集模糊，MVSplat 能恢复较明确的竖杆及扶手方向，接近真值。第三行石墙、壁炉、吊灯和台灯，右侧台灯罩尤其依赖第二输入提供的信息。MuRF 和 pixelSplat 的灯罩边界及周边出现模糊/拉伸，MVSplat 的外轮廓较完整。第四行室外桥梁，第二输入没有清楚呈现整座桥的拱部；两个基线桥体被背景植被混合，MVSplat 的栏杆、桥面、桥拱更连贯。GT 左侧车辆在合成图中没有同样清晰出现，不能宣称完全一致。

#### 技术联系与证据边界

【原文事实】§4.2 将四行分别联系重复纹理窗框、仅单输入可见的栏杆和灯罩、远距离拍摄的大型室外物体桥梁。重复纹理可使若干候选深度具有类似相似度；跨视角与局部空间上下文有助于排除局部歧义。对于单视图可见点，匹配不能直接提供正确的另一视图对应；2D U-Net 的邻域和学习先验可补充，不是从不存在的观测中恢复唯一几何。桥梁对应的大深度和小视差也使深度估计难度增加。Figure 6 的模块消融可进一步检验这些解释。

【独立评价】这里是同域新场景，不是零样本跨数据集迁移。展示的四个场景属于选择性样例，不能证明所有场景都优于基线；Table 1 才是跨大量测试场景的聚合画质证据。图像渲染更清楚不直接证明预测的三维位置与真值一致。

本图没有独立数学推导；具体匹配几何与候选深度歧义将在 Eq.(2–4) 精读中展开。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
### Figure 4：三维高斯与深度图对照

![Figure 4](MVSplat/Figures/Figure_04.jpg)

来源：指定版本 PDF 第 10 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_04.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. 4: Comparisons of 3D Gaussians (top) and depth maps (bottom). We compare the reconstructed geometry quality by visualizing zoom-out views of 3D Gaus- sians predicted by pixelSplat and our MVSplat, along with the predicted depth maps of two reference views. Extra fine-tuning is not performed on either model. Unlike pixelSplat that contains obvious floating artifacts, our MVSplat produces much higher quality 3D Gaussians and smoother depth, demonstrating the effectiveness of our cost volume-based 3D representation.

【原文事实】上方比较 3D 高斯，下方比较深度图。作者将 pixelSplat 与 MVSplat 预测的三维高斯拉远观察，同时呈现两个参考视角的深度图；两种模型均未额外微调。pixelSplat 存在明显漂浮伪影，而 MVSplat 的高斯几何与深度更平滑，作者认为这说明代价体表示有效。

#### 布局与逐区域分析

全图左右各一组室内场景，每组上方先 pixelSplat 再 MVSplat；红色相机框和虚线分别把高斯场景与下方两个参考视角的伪彩深度图联系起来。总共四个三维场景可视化、八张深度图。左组暖色房间中，pixelSplat 墙体、顶部、家具附近有大量分散的棕色或灰色团块，几何边界难辨；MVSplat 形成较清楚的墙面、灯、门洞与拐角，仍有不完整边缘。对应的两张深度中，pixelSplat 在墙面上有细碎起伏与局部突变，而 MVSplat 的大平面和门洞呈较成片的颜色变化。右组白墙和窗附近，pixelSplat 的天花板上方及边缘有弥散片团，MVSplat 的墙、窗和家具轮廓可分辨；下方两图也显示前者的斑点与噪声较多，后者的近处家具及远墙过渡更连贯。无数值色标，不由颜色推断米制尺度、深度定义或绝对误差。

#### 技术联系与证据边界

【原文事实】§4.2 说明 pixelSplat 可以生成合理二维图像但底层高斯仍有漂浮物，需要额外 50,000 步深度正则微调才能取得作者所称合理几何；图中双方均未进行这种额外微调。MVSplat 通过式(2–6)构造匹配代价体并回归每像素深度，再反投影，训练仍只依赖渲染图像损失。漂浮物（floaters）可理解为位于场景表面之外的高斯，对目标视角合成可能产生雾状或不稳定贡献。

【背景知识】沿参考像素射线移动高斯中心并改变其尺度、透明度和颜色，可以在少量训练/测试视角上产生近似相同的投影；因此仅用 RGB 损失，深度不是唯一可辨识量。多视图匹配缩小候选集合，但遮挡、重复纹理和非朗伯表面仍可能造成错误位置。高斯中心数亦影响图中密度；Table C 的每像素高斯数对照没有直接报告几何误差。

【独立评价】这幅图比 Figure 3 更直接地观察三维结构，却仍是定性证据：未报告 GT depth、点云精度、完整度或 Chamfer 距离，也不能区分真正光滑表面与过平滑重建；Figure D 扩展样例，Table A/3 间接支持匹配机制，但不构成定量几何证明。

本图无独立公式或绝对深度数值；反投影和高斯中心的数学定义在 §3 精读中完整展开。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
### Figure 5：跨数据集零样本泛化

![Figure 5](MVSplat/Figures/Figure_05.jpg)

来源：指定版本 PDF 第 11 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_05.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. 5: Cross-dataset generalization. Models trained on the source dataset RealEstate10K (indoor scenes) are used to conduct zero-shot test on scenes from target datasets ACID (outdoor scenes) and DTU (object-centric scenes), without any fine- tuning. pixelSplat tends to render blurry images with obvious artifacts since feature distributions in the target datasets differ from the one in the source, while our MVSplat renders competitive outputs thanks to the feature-invariant cost volume based design.

【原文事实】源数据集 RealEstate10K（室内场景）训练的模型直接对目标数据集 ACID（室外场景）和 DTU（以物体为中心的场景）开展零样本测试，不作微调。作者认为 pixelSplat 因目标与源数据特征分布不同而出现模糊和明显伪影，MVSplat 凭借基于相对特征匹配的代价体设计取得有竞争力的结果。

#### 布局与逐区域分析

图分左右两个目标数据集，每侧两行、四列：Input（上下堆叠两张输入）、pixelSplat、MVSplat、Ground Truth。左侧 RE10K→ACID：第一行为海岸草地上的红屋顶灯塔建筑，pixelSplat 把红屋顶重复到草地并产生大范围模糊，MVSplat 的灯塔、道路和小屋位置与轮廓更接近 GT，不过海岸背景不完全一致；第二行为俯视绿色植被、道路及远处水体，pixelSplat 的纹理过平滑、地面道路不清楚，MVSplat 保留更多树丛及浅色道路结构。右侧 RE10K→DTU：第一行为深色圆柱容器及红色字样，pixelSplat 容器右侧和背景互相混叠，文字难辨；MVSplat 保留形状、字样和边缘，但较 GT 模糊。第二行为食品盒、罐头与瓶子，pixelSplat 出现明显重影和边缘混合，MVSplat 更清楚地恢复盒子、两罐的轮廓和部分包装文字，仍不等同 GT。室外远景与物体近景的尺度、相机分布、纹理均不同，图不能把域差只归于单一因素。

#### 技术联系与证据边界

【原文事实】Table 2 的同训练源对照：ACID 上 MVSplat 相对 pixelSplat 的 PSNR +0.51 dB、SSIM +0.011、LPIPS -0.013；DTU 分别 +1.05 dB、+0.091、-0.175，改善在 DTU 更大。这里两模型都仅在 RE10K 训练，不能与在目标域训练的模型混成同一个控制实验。§4.2 将优势解释为代价体相对特征相似性比绝对特征尺度更稳健。

【独立评价】式(3) 是缩放点积而非显式余弦归一化：若两特征同时乘以 $a$，得分乘以 $a^2$，softmax 也会改变尖锐程度。因此作者“feature-invariant”一词应理解为经验上相对匹配的可迁移性，不是严格数学尺度不变性。MVSplat 在 DTU 的绝对 PSNR 13.94、LPIPS 0.385，说明迁移仍具有相当残差；不能由这几张例图推断所有域的可靠性，尤其没有水下、雾等复杂介质的证据。Figure E 扩展定性样例。

【背景知识】PSNR 和 SSIM 越高越好，LPIPS 越低越好；跨不同数据集绝对指标不直接衡量域差大小。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
### Figure 6：代价体、注意力与 U-Net 的定性消融

![Figure 6](MVSplat/Figures/Figure_06.jpg)

来源：指定版本 PDF 第 13 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_06.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. 6: Ablations on RealEstate10K. Colored error maps obtained by calculating the differences between the rendered images and the ground truth are attached for better comparison. All models are based on the “base” model, w/o depth refinement module. “w/o cross-attn” is short for “w/o cross-view attention”. Compared to “base”, “base+refine” slightly reduces errors, “w/o cost volume” leads to the largest drop, “w/o U-Net” harms contents on the right that only exist in one input view, while “w/o cross- attn” increases the overall error intensity.

【原文事实】RealEstate10K 消融。附有根据渲染图与真值差异计算的彩色误差图。各消融均基于不含深度精炼的 base；w/o cross-attn 指去掉跨视角注意力。与 base 相比，base+refine 略微降低误差，去掉代价体使结果下降最多，去掉 U-Net 损害右侧仅存在于一个输入视角中的内容，去掉跨视角注意力则总体增加误差强度。

#### 布局与逐区域分析

左上是 GT，左下两幅小图为输入视角：左输入主要包含壁炉、植物、左侧家具，右输入包含落地窗、右侧沙发和灯具；它们并非完全重叠。右边五列从左到右为 w/o cost volume、w/o cross-attn、w/o U-Net、base、base + refine，上方对应渲染、下方对应误差图。误差图采用由暗紫到亮绿的伪彩，但没有数值色标，因而只比较显示的相对高误差区域。

去掉代价体：门窗竖框错位、茶几和壁炉出现模糊与重影，误差在窗框、家具等处大面积增亮。去掉跨视角注意力：画面总体仍可辨认，窗框、茶几和植物周边误差高于 base。去掉 U-Net：右边窗框和家具明显变形，误差特别集中在右侧（只有右输入可见的内容）。base 保留代价体、跨视角注意力和 U-Net，误差图大部分较暗；base+refine 的窗框、灯具和茶几局部误差有进一步减弱。该图未给出每组深度真值或逐像素深度误差。

#### 技术联系与证据边界

【原文事实】§4.3 和 Table 3：以 base 的 PSNR 26.12、SSIM 0.864、LPIPS 0.133 为共同参考，去掉 cost volume 得 22.83/0.753/0.197（-3.29 dB、-0.111、+0.064），去掉 cross-attn 得 25.19/0.852/0.152（-0.93、-0.012、+0.019），去掉 U-Net 得 25.45/0.847/0.150（-0.67、-0.017、+0.017），加入 refine 得 26.39/0.869/0.128（+0.27、+0.005、-0.005）。除跨视角注意力消融取最佳步外，均为 300K；Figure A 说明例外原因。并不能据一张图声称跨视角注意力与 U-Net 在所有指标上的贡献顺序相同。

【合理推断】无代价体时失去深度候选的显式对应证据，需主要依赖特征预测几何；无跨视角注意力削弱跨图像特征/上下文交换；无 U-Net 使低纹理、遮挡、单视图区域较难借助邻域上下文纠正模糊匹配。Refine 在已建立的深度上预测局部残差，改善有限而非重新构造几何。这些是机制解释，不能从 RGB 误差图分离出几何、透明度和颜色的各自误差，也不是模块独立相加的因果定量模型。

【背景知识】同一 GT 的误差可用 $|I_{pred}-I_{gt}|$ 或其他像素差异定义；题注未规定误差图精确通道归约/归一化，不能擅自指定实现。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-A -->
### Figure A：各消融配置的验证曲线

![Figure A](MVSplat/Figures/Figure_A.jpg)

来源：指定版本 PDF 第 20 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_A.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. A: Validation curves of the ablations. The setup of each model is illustrated on the top, which refers to the same one as in Tab. 3 of the main paper. The cost volume plays a fundamental role in our full model, and interestingly, the model without cross- view attention suffers from over-fitting after certain training iterations.

【原文事实】消融模型的验证曲线，顶部标记的模型设置对应主文 Table 3。代价体在完整模型中起基础性作用；去掉跨视角注意力的模型在一定训练迭代之后表现出过拟合。

#### 布局与逐区域分析

三个横排子图共用顶部图例：红 base+refine、黄 base、绿 w/o U-Net、蓝 w/o cross-view attention、灰 w/o cost volume。横轴为 trainer/global_step（训练更新步数，k 表示千步），标注 50k、100k、150k、200k、250k，末端约 300k。左纵轴 PSNR（dB，越高越好），中纵轴 SSIM（无量纲，越高越好），右纵轴 LPIPS（无量纲，越低越好）；三图均是验证曲线，不是训练损失曲线。

左图五条曲线初期均快速上升，随后红、黄继续缓升并领先，绿在约 25.5 dB 附近稳定；灰只升到约 22.8 dB，长期与其余配置有明显间隔。蓝曲线约 150k–200k 附近达到高点，后期跌至约 24.4 dB。中图灰停留在约 0.75，红黄接近 0.87，绿略低于 0.85；蓝先升至约 0.85，约 250k 后下滑至约 0.835。右图红黄持续下降，灰长期最高且约 0.197 收敛，绿约 0.15；蓝先接近绿，后期反升至约 0.185。曲线读数为可视估计而非额外表格精确值，不用曲线来构造逐步数值。三项指标方向一致显示蓝色配置晚期退化。

#### 技术联系与证据边界

【原文事实】附录 A 及主文 Table 3：除 w/o cross-view attention 外均取 300K 收敛步，后者因为后期退化而报告最佳性能 25.19/0.852/0.152。base+refine 在三个方向总体最佳，去掉代价体最差。灰色在较长训练下仍明显落后，说明在现有训练设置中不是单纯训练步数不足即可解释。蓝色最佳指标未必来自完全相同的 step，原文未给出所选精确步数。

【合理推断】跨视角注意力允许同一场景不同观察的特征交互，可能降低依赖单图训练外观统计的程度；但验证曲线仅证明此配置在训练后期验证性能恶化，缺少训练误差曲线、随机种子和独立验证协议的完整统计，不能唯一确定过拟合的内部因果机制。Table 3 的蓝色最佳步对照与其余最后步并非完全相同步数；仍落后于 base，但不能据此声称已控制每一种训练预算。

本图不含独立公式；数值只按实际曲线作近似读数，精确指标采用 Table 3。

<!-- PTH:END FIG-A -->

<!-- PTH:BEGIN FIG-B -->
### Figure B：反射与非朗伯表面的失败样例

![Figure B](MVSplat/Figures/Figure_B.jpg)

来源：指定版本 PDF 第 20 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_B.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. B: Failure cases. Our MVSplat might be less effective on the non-Lambertian and reflective surfaces.

【原文事实】失败案例：MVSplat 在非朗伯和反射表面上可能不够有效。

#### 布局与逐区域分析

左右两组厨房场景，每组第一列为两张输入，第二列为 MVSplat，第三列为 Ground Truth，第四列为 Error Map。左组亮色台面，红框标记水龙头附近的台面高光/反射；GT 的反射较集中清楚，而预测在其附近出现模糊的亮斑与不一致结构，误差图对应框有较亮的竖状区域。右组暗色橱柜，红框标出左侧反光的深色家电/玻璃表面，预测与 GT 的反射纹理和亮度不一致，误差集中于框内。其余静态橱柜、台面、灯具总体仍可辨认，因此是局部困难而不是整张图均完全失败。误差图无绝对色标，不能报告误差数值。

#### 技术联系与证据边界

【原文事实】Appendix A p.20 明确提出结合额外 BRDF 属性和多样化训练数据可能有助于解决此问题，并提醒复杂现实场景结果不可靠、需谨慎用于安全关键场合。这些是未来建议而非已实现组件。

【背景知识】朗伯假设指理想漫反射表面出射辐亮度不随观察方向而变（在固定光照条件下）。非朗伯/镜面反射情况下，同一空间点在不同方向的可见外观可以显著不同。即使投影几何正确，式(3) 的特征点积也可能因视角依赖外观而降低，反射中的虚像甚至可能提供错误对应。球谐颜色允许一定方向依赖外观，但它不是显式光照、BRDF 与几何分离的反射模型；不能保证从两张图估计新的方向下的反射。

【独立评价】图只展示 RGB 误差，没有深度真值或分离的反射分量，不能判断错误来自中心、颜色、遮挡或多个因素。也没有反射材质的统计样本比例；无法定量给出反射鲁棒性的边界。不能将 Figure E 中有金属光泽物体的相对成功视为与此矛盾，二者是不同实例与不同对照。

本图没有独立数学式或定量指标；非朗伯条件下匹配分数的变化属于机制解释。

<!-- PTH:END FIG-B -->

<!-- PTH:BEGIN FIG-C -->
### Figure C：同域新视角结果的扩展样例

![Figure C](MVSplat/Figures/Figure_C.jpg)

来源：指定版本 PDF 第 21 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_C.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. C: More comparisons with the state of the art. These are the extended visual results of Fig. 3. Scenes of the first four rows come from the RealEstate10K, whilst scenes of the last two rows come from ACID. Our MVSplat performs the best in all cases.

【原文事实】Figure 3 的扩展可视结果。前四行场景来自 RealEstate10K，后两行来自 ACID。作者称 MVSplat 在这些展示的情形中均表现最好。

#### 布局与逐区域分析

每行六列，前两列 Input Views，第三 MuRF，第四 pixelSplat，第五 MVSplat，第六 GT。第一行门框内卫生间，两个输入显露门框/柜台/马桶不同范围；MuRF 与 pixelSplat 的门缘与马桶轮廓变得模糊，MVSplat 较完整但边界仍不完美。第二行不规则木桌、电脑、壁画：基线在木桌边沿和腿部混叠较明显，MVSplat 更保留桌板形状，与 GT 仍有细节差异。第三行厨房吧台、白椅和玻璃杯，MuRF 和 pixelSplat 的椅子及台面边缘有模糊或重影，MVSplat 较清楚地分离白椅、台面及柜体。第四行为走廊，花瓶、栏杆、门框和红色房间，基线花瓶/栏杆局部较模糊，MVSplat 更接近真值的门框与走廊结构。

第五行室外海岸岩石及碧绿水面；基线岩壁、白色浪花、水陆边界模糊，MVSplat 保留较清楚的海岸形态，但水面纹理与 GT 不完全一致。第六行俯视道路和植被斜坡；两个基线植被细节较模糊，MVSplat 的地形纹理和道路更清楚，但真值道路上的车辆未完全重现。各行都对比相同目标位姿，不以输入和 GT 的不同视点作为预测错误。

#### 技术联系与证据边界

【原文事实】Appendix B 将本图定义为 Figure 3 的扩展，与 Table 1 的同域测试协议配套。窗框、遮挡、薄结构、远处纹理等情况与 §4.2 的几何困难相对应。对于桌腿、椅子等物体，正确的深度可帮助减少错误投影；但稀疏视角下仅可在一张图中见到的内容仍依赖上下文先验，不是由双目匹配直接确定。对于水面，动态浪花、反光等也可能破坏对应，图中没有专门控制此因素。

【独立评价】更多选择性样例提高了结果展示覆盖，却不是新的独立定量实验，也不应把作者的所有样例主张扩展成所有测试场景均占优。没有真实几何误差，因此图的结论限于可见渲染质量。

本图没有独立数学推导；聚合指标与评价口径见 Table 1。

<!-- PTH:END FIG-C -->

<!-- PTH:BEGIN FIG-D -->
### Figure D：扩展高斯几何及深度对比

![Figure D](MVSplat/Figures/Figure_D.jpg)

来源：指定版本 PDF 第 22 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_D.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. D: More comparisons of 3D Gaussians (top) and depth maps (bottom). These are the extended visual results of Fig. 4. Extra depth-regularized fine-tuning is not applied to either model. 3D Gaussians and depth maps predicted by our MVSplat are both of higher quality than those predicted by pixelSplat.

【原文事实】Figure 4 的扩展结果；上方比较三维高斯，下方比较深度图，两种模型均未额外应用深度正则微调。作者认为 MVSplat 预测的高斯及深度图质量均更好。

#### 布局与逐区域分析

左右各一个室内场景，组内分别展示 pixelSplat 与 MVSplat 的三维高斯，每幅高斯下方配有两个红框相机所对应的伪彩深度图，共四个高斯可视化和八张深度图。左组是黄色墙面与厨房/门洞场景，pixelSplat 在墙壁外侧及边缘形成大量棕灰色漂浮团块，门洞和厨房器具难以区分；MVSplat 的墙、门框、台面呈较连续的平面与轮廓，场景仍不完整。其两幅深度图中，基线门洞附近细碎变化与斑点较多，而 MVSplat 深度主要呈连贯的大块过渡。右组是木色厨房，pixelSplat 在左侧、上方和柜面附近可见扩散高斯、拉丝结构，MVSplat 的上下柜、炉灶、台面及拐角更明确；对应深度图中前者有显著噪声、突变，后者平滑并保留大体转角。没有统一数值色标及 GT depth，不能从不同伪彩图直接比较绝对深度。

#### 技术联系与证据边界

【原文事实】Figure 4/D 与 §4.2 的几何质量比较采用未额外微调的前馈输出。MVSplat 的代价体为深度预测提供候选层的匹配证据，并通过反投影直接约束高斯中心的像素对齐方式。相较于缺少显式匹配的回归，有机会在更多位置选择与多视角一致的深度。

【独立评价】这与 Figure 4 的相同现象相互印证：某些情况下较少漂浮、深度更连续；不是独立测量几何精度。图不显示所有遮挡表面，输出更规整可能包含先验平滑作用。像素对齐中心和视图并集并不保证场景是完整闭合表面，也不确保同一世界位置的高斯重复被消除。

本图没有独立数学式或标定深度数值；几何数学将在式(1–6)及高斯参数专章推导。

<!-- PTH:END FIG-D -->

<!-- PTH:BEGIN FIG-E -->
### Figure E：跨数据集泛化的扩展样例

![Figure E](MVSplat/Figures/Figure_E.jpg)

来源：指定版本 PDF 第 22 页，B2 区域提取的 JPEG 展示副本；保真 PDF 为 `source/Figures_PDF/Figure_E.pdf`。客户端渲染未验证。

#### Caption 原文与翻译

> Fig. E: More comparisons of cross-dataset generalization. These are the ex- tended visual results of Fig. 5. By training on indoor scenes (RealEstate10K), our MVSplat generalizes much better than pixelSplat to outdoor scenes (ACID) and object- centric scenes (DTU), showing the superior of our cost volume-based architecture.

【原文事实】Figure 5 的扩展结果。在室内 RealEstate10K 上训练时，MVSplat 比 pixelSplat 更好地泛化到室外 ACID 和物体中心 DTU；作者将其归因于代价体架构。

#### 布局与逐区域分析

左右两域各四组，组内四列为 Input（每组两张小图）、pixelSplat、MVSplat、Ground Truth。左侧 ACID：第一行海面栈桥，pixelSplat 桥脚及边缘较模糊，MVSplat 保留柱体和桥面形状；第二行海岸城镇，pixelSplat 的屋顶及道路被平滑，MVSplat 更清楚地保留布局；第三行俯视海滩及波浪，基线浪线、沙滩纹理较模糊，MVSplat 的浪线和斑驳地貌较清楚；第四行沙滩岩石，MVSplat 相比基线保留更多岩石边缘，仍有纹理差异。右侧 DTU：第一行深色杯状物体，红色心形花纹及物体边缘，基线有显著重影，MVSplat 较接近 GT；第二行红色南瓜与包装物，基线果梗、背后包装严重混叠，MVSplat 果梗和轮廓更明确；第三行为深色有光泽坐像，MVSplat 改善轮廓与肢体形态，但局部高光并非完全匹配 GT；第四行浅色雕像，基线面部及衣纹明显模糊，MVSplat 五官和纹理更清楚，仍可见细节损失。

#### 技术联系与证据边界

【原文事实】Appendix B 与主文 Table 2 的 RE10K→ACID、RE10K→DTU 零样本协议一致。DTU 物体中心拍摄与源域室内场景的构图和相机分布差异大；表中 LPIPS 的绝对改善在 DTU 为 0.175，在 ACID 为 0.013。匹配表示相对特征关系而非完全依赖特征到深度的直接映射，作者认为由此提高跨域鲁棒性。

【独立评价】两组不构成严格控制域距离大小的实验，ACID 与 DTU 同时改变了多种因素，且每行只展示一个目标图。跨域绝对性能特别是 DTU 仍明显不完美。带反光坐像的相对优势不意味着非朗伯问题已解决（Figure B）；水面出现在 ACID 图中也不等同于水下成像或介质建模。

本图没有独立数学推导；相似性尺度边界见 Figure 5 与后续式(3)分析。

<!-- PTH:END FIG-E -->

<!-- PTH:BEGIN TABLE-01 -->
### Table 1：与代表性前馈方法的主实验对照

来源：指定版本 PDF 第 8 页，4.2 Main Results。

#### Caption 原文与翻译

> Table 1: Comparisons with the state of the art. Running time includes both encoder and render, note that 3DGS-based methods (pixelSplat and MVSplat) render dramatically faster (∼500FPS for the render). Performances are averaged over thou- sands of test scenes in each dataset. For each scene, the model takes two views as input and renders three novel views for evaluation. MVSplat performs the best in terms of all visual metrics and runs the fastest with a lightweight model size.

【原文事实】与先进方法对比。运行时间包括编码器及渲染，基于 3DGS 的 pixelSplat 和 MVSplat 的独立渲染非常快（约 500 FPS）。各数据集性能是数千个测试场景上的平均结果，每场景输入两视角并渲染三个新视角用于评价。MVSplat 在各图像指标上表现最佳、运行最快，且模型规模轻量。

#### 原始数据与指标

| Method | Time (s) ↓ | Param (M) ↓ | RE10K PSNR↑ | SSIM↑ | LPIPS↓ | ACID PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| pixelNeRF [49] | 5.299 | 28.2 | 20.43 | 0.589 | 0.550 | 20.97 | 0.547 | 0.533 |
| GPNR [35] | 13.340 | 9.6 | 24.11 | 0.793 | 0.255 | 25.28 | 0.764 | 0.332 |
| AttnRend [10] | 1.325 | 125.1 | 24.78 | 0.820 | 0.213 | 26.88 | 0.799 | 0.218 |
| MuRF [44] | 0.186 | 5.3 | 26.10 | 0.858 | 0.143 | 28.09 | 0.841 | 0.155 |
| pixelSplat [1] | 0.104 | 125.4 | 25.89 | 0.858 | 0.142 | 28.14 | 0.839 | 0.150 |
| MVSplat | 0.044 | 12.0 | 26.39 | 0.869 | 0.128 | 28.25 | 0.843 | 0.144 |

时间单位秒，参数单位百万（M）；PSNR 为 dB，基于像素均方误差；SSIM 为结构相似性，LPIPS 为学习特征空间的感知距离，二者无量纲。PSNR/SSIM 越高越好、LPIPS 越低越好；参数/时间越小通常有利于部署但不能单独作为质量指标。§4.1：256×256；RE10K 和 ACID 分别有 7,289、1,972 个测试场景，提供估计内外参。各自训练域与测试域一致；训练场景不等于测试场景。

#### 比较、作用与限制

【原文事实】MVSplat 对最接近的 3DGS 基线 pixelSplat：RE10K 的 PSNR +0.50 dB，SSIM +0.011，LPIPS -0.014（相对下降 9.86%）；ACID +0.11 dB、+0.004、-0.006（相对下降 4.0%）。对 MuRF，RE10K +0.29/+0.011/-0.015，ACID +0.16/+0.002/-0.011；ACID 的最佳基线 SSIM 实为 MuRF 的 0.841，因此该指标相对最强基线仅 +0.002。pixelNeRF 在两域的指标均显著较低，GPNR、AttnRend 随更丰富匹配/聚合得到更高画质；不能仅凭该横向比较证明具体模块的因果作用。MVSplat 对 MuRF 的时间比 0.186/0.044≈4.23；对 pixelSplat 为 0.104/0.044≈2.36，减少约 57.7% 延时；125.4/12.0=10.45，即参数少约 90.4%。MuRF 5.3M 与 GPNR 9.6M 都比 MVSplat 小，不能说 MVSplat 全表参数最少。Figure 1 的柱图是近似可视值，精确表值 125.4M 为准。

§4.2 单独分解：MVSplat encoder 0.043s、render 0.0015s；pixelSplat encoder 0.102s、render 0.0025s。编码分别约 2.37 倍、单帧渲染约 1.67 倍加速；表中总时间约 0.044/0.104s，系独立测量/取整，不能将三目标视图时间随意累加来重建表值。总耗时倒数约 22.7 次/s，独立渲染倒数约 667/400 FPS；后者不是从输入到新视角的全流程 FPS。每像素一个 vs 三个高斯可减少渲染规模，Swin 与代价体结构也影响编码，Table B/C 提供更细的控制对照。

【原文事实】§4.2 说明 pixelNeRF、GPNR、AttnRend、pixelSplat 的指标直接取自 pixelSplat 论文；MuRF 使用公开代码重训与评价。因此不是所有基线均由作者在完全统一环境从零重训。硬件和具体计时重复统计未在表题完整列明，不能直接外推到其他分辨率、不同 GPU 或任意 K。

【独立评价】主实验支持本文在两域、该协议下的综合画质和效率优势，但不是统计显著性检验（未列方差/置信区间），也没有量化深度或几何真值。Figure 3/C 提供渲染可视证据，Figure 4/D 提供定性几何证据，不能将 Table 1 的 PSNR 改称几何精度。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
### Table 2：跨数据集泛化与题注数字冲突

来源：指定版本 PDF 第 11 页，4.2 Main Results。

#### Caption 原文与翻译

> Table 2: Cross-dataset generalization. Models trained on RE10K (indoor scenes) are directly used to test on scenes from ACID (outdoor scenes) and DTU (object- centric scenes), without any further fine-tuning. Our MVSplat generalizes better than pixelSplat, where the improvement is more significant when the gap between source and target datasets is larger (RE10K to DTU). It is also worth noting that our zero- shot generalization results on ACID even slightly surpass pixelSplat’s ACID trained model (PSNR: 28.14, SSIM: 0.843, LPIPS: 0.144) reported in Tab. 1.

【原文事实】跨数据集泛化：在 RE10K（室内）训练的模型直接用于 ACID（室外）和 DTU（物体中心）场景测试，不进行后续微调。作者指出 MVSplat 泛化优于 pixelSplat，在源与目标数据差距更大的 RE10K→DTU 时改善更明显。作者还称零样本 ACID 结果略优于 Table 1 中在 ACID 训练的 pixelSplat，并在题注给出 PSNR 28.14、SSIM 0.843、LPIPS 0.144（此处照录原文，后两项与 Table 1 的归属冲突，见下）。

#### 原始数据与指标

| Training data | Method | ACID PSNR↑ | SSIM↑ | LPIPS↓ | DTU PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---|---:|---:|---:|---:|---:|---:|
| RealEstate10K [54] | pixelSplat [1] | 27.64 | 0.830 | 0.160 | 12.89 | 0.382 | 0.560 |
| RealEstate10K [54] | MVSplat | 28.15 | 0.841 | 0.147 | 13.94 | 0.473 | 0.385 |

同训练数据、目标域不微调。PSNR(dB)、SSIM、LPIPS 为 RGB 图像质量指标，方向分别 ↑/↑/↓。DTU 采用 16 个验证场景、每场景 4 个新视角（§4.1）；表中两视图协议与 p.12 后续三视图实验不同，不混入此表。

#### 比较、作用与限制

以同样 RE10K 训练的 pixelSplat 为参考，ACID：28.15−27.64=+0.51 dB，0.841−0.830=+0.011，0.147−0.160=−0.013（下降 8.13%）。DTU：+1.05 dB，+0.091，−0.175（下降 31.25%）。两目标域的三个指标均改善，在 DTU 的绝对差值更大，支持作者的相对泛化主张；但 DTU PSNR 13.94 和 LPIPS 0.385 不代表绝对效果优良。

**原文内部数字冲突：** Table 2 题注把 28.14/0.843/0.144 归给 ACID 训练的 pixelSplat。实际 p.8 Table 1 中，pixelSplat 行为 28.14/0.839/0.150，0.843/0.144 属于 MVSplat 行（28.25/0.843/0.144）。因此在原始题注保留的同时，跨表比较必须使用 Table 1 的真实行：RE10K 训练 MVSplat 的 ACID 结果 28.15/0.841/0.147 对 ACID 训练 pixelSplat 28.14/0.839/0.150，三指标分别 +0.01、+0.002、−0.003，优势确实很小；若错误使用题注，则 SSIM/LPIPS 的方向相反。相对于 ACID 训练的 MVSplat，零样本模型 −0.10 dB、−0.002、+0.003，略弱。此处不是外部纠正，而是同版本两张表的交叉核查，未取得作者勘误。

【原文事实】作者将跨域性能归因于相对特征匹配，并指出 RE10K 训练规模比 ACID 大。按 §4.1 列出的训练场景数 67,477/11,075≈6.09，p.12 文本称约 7 倍；保留两种口径并指出场景数计算不同，不能确认作者是否使用其他样本口径。比较不同训练集的方法与数据规模同时变化，不是严格控制实验。

【独立评价】缩放点积本身不是严格尺度不变量，式(3) 的特征变化可能改变分布和 softmax；这里是经验支持，不是数学证明。Figure 5/E 的样例只能补充定性差异，未证明任意目标域或未见相机分布均可靠。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
### Table 3：RealEstate10K 主消融

来源：指定版本 PDF 第 12 页，4.3 Ablations (table float)。

#### Caption 原文与翻译

> Table 3: Ablations on RealEstate10K. The “base + refine” is our final model, where “refine” refers to the “depth refinement” detailed in Sec. 3.1. All other ablations are conducted on the “base” model w/o depth refinement. The cost volume module plays an indispensable role in MVSplat. Results of all models are obtained from the final converged step (300K in our experiments), except the one “w/o cross-view attention”, which suffers from over-fitting (details are shown in the supplementary material Fig. A), hence we report its best performance.

【原文事实】RealEstate10K 上的消融：base+refine 是最终模型，refine 指 §3.1 中的深度精炼；其余消融基于不含深度精炼的 base。代价体模块起不可或缺的作用。除去跨视角注意力的配置发生过拟合（详见附录 Figure A）而报告最佳性能外，其余均为最后收敛步（300K）。

#### 原始数据与指标

| Setup | PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---:|---:|---:|
| base + refine | 26.39 | 0.869 | 0.128 |
| base | 26.12 | 0.864 | 0.133 |
| w/o cost volume | 22.83 | 0.753 | 0.197 |
| w/o cross-view attention | 25.19 | 0.852 | 0.152 |
| w/o U-Net | 25.45 | 0.847 | 0.150 |

同 RE10K 协议，指标分别为 PSNR(dB)↑/SSIM↑/LPIPS↓。base 已包含匹配代价体、特征提取骨干的跨视图注意力及代价体 refinement U-Net；它缺少单独的第二个 depth refinement U-Net。§4.3 的 w/o cross-view attention 指移除 Transformer 骨干里的跨视角注意力，不能擅自解释为移除了全模型所有注意力。w/o U-Net 指移除代价体精炼 U-Net，不是本就不存在的深度精炼模块。

#### 比较、作用与限制

从 base 出发而不是最终模型：

| 修改（变体−base） | ΔPSNR (dB) | ΔSSIM | ΔLPIPS |
|---|---:|---:|---:|
| 加 depth refinement | +0.27 | +0.005 | −0.005 |
| 去 cost volume | −3.29 | −0.111 | +0.064 |
| 去 cross-view attention | −0.93 | −0.012 | +0.019 |
| 去 cost volume refinement U-Net | −0.67 | −0.017 | +0.017 |

去代价体的 LPIPS 恶化相对 base 为 0.064/0.133≈48.1%，PSNR 降幅最大且三指标方向一致。Figure 6 观察到输入视图直接叠加样伪影，作者解释为两个输入提取出的高斯未能在同一空间对齐。去 cross-attn 的 PSNR 下降约 1dB，蓝色 Figure A 验证曲线后期退化；去 U-Net 的中间重叠区域仍可渲染，而左右单输入可见区域变差，说明空间特征和上下文有助于补足匹配缺失。Depth refinement 在已有深度上预测残差，0.27dB 是相对 base 的增益，不能将全部差距归给它。SSIM 中去 U-Net 的降幅反而大于去 cross-attn，两个模块不能仅凭一项指标统一排序。

【独立评价】消融是对本文机制最直接的画质证据之一，但跨视角注意力采用最佳步而非统一末步；原文没有提供其具体选择 step/不同随机种子。移除模块还可能改变模型容量、计算及优化状态；不能把差值当作相互独立可相加的理论贡献，或把 RGB 指标下降直接等同为 3D 中心定位误差。附录 Table A–D 提供额外证据而不是本表所有因素的正交组合。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-A -->
### Table A：将代价体深度分支移植到 pixelSplat

来源：指定版本 PDF 第 18 页，A More Experimental Analysis。

#### Caption 原文与翻译

> Table A: Using cost volume in pixelSplat. Our cost volume-based depth predic- tion approach can also be used in the pixelSplat [1] model by replacing its probability density-based depth branch and its performance can be significantly boosted, which demonstrates the general applicability of our method.

【原文事实】MVSplat 基于代价体的深度预测方法也可用于 pixelSplat：替换其基于概率密度的深度分支后，性能显著提升，作者认为这验证了所提方法的一般适用性。

#### 原始数据与指标

| Setup | PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---:|---:|---:|
| pixelSplat (w/ probability density depth) [1] | 25.89 | 0.858 | 0.142 |
| pixelSplat (w/ our MVSplat cost volume depth) | 26.63 | 0.875 | 0.122 |

附录 A 开头说明除非另有说明，实验遵循 §4.3，在 RE10K 上训练并在完整测试集求平均。PSNR(dB)↑、SSIM↑、LPIPS↓为图像质量；这里没有时间、参数、几何误差列。作者明确写明替换深度模块时其他组件保持不变。

#### 比较、作用与限制

相对于原 pixelSplat，替换后 PSNR +0.74 dB、SSIM +0.017、LPIPS −0.020（相对下降约 14.1%）。这比单纯比较两种整体架构更能隔离深度分支的效果：固定 pixelSplat 其他组件，使多视角匹配显式进入深度估计。原分支依靠特征回归概率深度并采样（§2），替换后采用候选深度的几何匹配与预测。注意 MVSplat 仍有 softmax 深度分布，区别不是有无概率，而是分布构建所依据的匹配证据以及从分布到中心的估计方式。

与 Table 1 的完整 MVSplat（26.39/0.869/0.128）相比，此移植配置画质 +0.24/+0.006/−0.006。不能称它为本文默认更高效模型，因为没有报告该变体参数、推理时间、高斯数量及几何指标；更强画质可能受 pixelSplat 其余架构影响。没有额外独立基线架构移植或非 RE10K 实验，支持的是在该架构和协议下的可迁移性，而不是所有前馈模型都必然改善。

<!-- PTH:END TABLE-A -->

<!-- PTH:BEGIN TABLE-B -->
### Table B：Swin 与 Epipolar Transformer 的骨干对照

来源：指定版本 PDF 第 18 页，A More Experimental Analysis。

#### Caption 原文与翻译

> Table B: Comparisons of the backbone Transformer. Our Swin Trans- former [23]-based architecture is more efficient than the Epipolar Transformer counter- part in pixelSplat [1] since the expensive epipolar sampling process is avoided. Besides, there is no clear difference observed in their rendering qualities.

【原文事实】骨干 Transformer 对比：基于 Swin Transformer 的架构避免了 pixelSplat 中 Epipolar Transformer 所需的昂贵极线采样过程，因此效率更高；两者渲染质量没有明显差异。

#### 原始数据与指标

| Setup | Time (s)↓ | PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---:|---:|---:|---:|
| MVSplat (w/ Epipolar Transformer [1]) | 0.055 | 26.09 | 0.865 | 0.133 |
| MVSplat (w/ Swin Transformer) | 0.038 | 26.12 | 0.864 | 0.133 |

RE10K，采用附录 A 的默认消融协议；时间为秒，表头只写 Time 而非 Render Time，不能把它等同 Table C 的独立渲染时间。Swin 行画质恰与 Table 3 base 一致，不应同完整模型的 0.044s/26.39 直接混比。

#### 比较、作用与限制

Swin−Epipolar 的时间为 −0.017 s（减少约 30.9%；0.055/0.038≈1.45 倍速度），PSNR +0.03 dB、SSIM −0.001、LPIPS 0。Swin 并非三个图像指标全面提升，作者所说“无明显画质差异”与表中非常小的变化相符；缺少方差与多次运行统计，不能声称两者数学等价或统计等效。

【原文事实】Appendix A：在 MVSplat 多视图特征提取骨干中替换 Swin 为 Epipolar Transformer 进行对照。Swin 避免在极线上显式采样点；§3.1 仍通过 plane sweeping 在代价体构造时使用相机投影几何。也就是说，减少的是骨干注意力的极线采样开销，不是移除几何约束。局部窗口注意力在特征空间交互，再由显式 cost volume 进行深度候选匹配，二者职责不同。

【独立评价】该表支持作者对骨干效率的选择，但不提供完整 FLOPs、峰值显存或窗口设置扫描；不能断言 Swin 在其他分辨率、视角数和非 MVSplat 架构中都更快。

<!-- PTH:END TABLE-B -->

<!-- PTH:BEGIN TABLE-C -->
### Table C：每像素高斯数量与渲染成本

来源：指定版本 PDF 第 19 页，A More Experimental Analysis。

#### Caption 原文与翻译

> Table C: Comparisons of Gaussians’ number per pixel. Increasing the number of Gaussians improves the performance but slows down the rendering speed.

【原文事实】每像素高斯数量对比：增加高斯数量可以提高性能，但会降低渲染速度。

#### 原始数据与指标

| Setup | Render Time (s)↓ | PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---:|---:|---:|---:|
| MVSplat (1 Gaussian per pixel) | 0.0015 | 26.39 | 0.869 | 0.128 |
| MVSplat (3 Gaussians per pixel) | 0.0023 | 26.54 | 0.872 | 0.127 |

附录 A 中 RE10K 实验；时间表头明确 Render Time，单位秒，不含整个编码器。PSNR(dB)↑、SSIM↑、LPIPS↓均是图像质量。

#### 比较、作用与限制

以默认 1 个高斯/像素为基准，3 个高斯/像素的 PSNR +0.15dB、SSIM +0.003、LPIPS −0.001；渲染时间 +0.0008s（0.8ms，增加约 53.3%）。独立渲染时间倒数为约 667 FPS 与 435 FPS；增加数量使每秒渲染帧数下降约 34.8%，不是端到端推理帧数。对两个 256×256 输入，式(1) 的默认 $N=HWK=131072$；3 个/像素时 $N=3HWK=393216$。数量变成三倍但耗时并非严格三倍，因为投影覆盖、tile 分桶、光栅化并行和固定开销共同影响时间（通用背景解释，非作者计时分解）。

【原文事实】作者因此选择每像素一个高斯来平衡性能与渲染速度。该对照同一 MVSplat 架构内改变数量，比直接对 pixelSplat 的渲染时间比较更有针对性；后者在 §4.2 的 3 高斯/像素时间为 0.0025s，不同架构不应混淆。

【独立评价】不能由两个数量点拟合任意高斯数的耗时函数，也不能认定 3 个/像素几何一定更准确；增加表示容量可能让外观拟合更好，也可能产生多余中心。未报告变体的编码耗时、内存或深度真值，无法判断完整系统的最佳数量。

<!-- PTH:END TABLE-C -->

<!-- PTH:BEGIN TABLE-D -->
### Table D：骨干预训练与随机初始化

来源：指定版本 PDF 第 19 页，A More Experimental Analysis。

#### Caption 原文与翻译

> Table D: Comparisons of backbone initialization. By default we train our models for 300K iterations with the publicly available UniMatch pretrained weights as initial- ization. However, our model can also be trained with random initialization (“w/ random init”) and still outperforms previous state-of-the-art method pixelSplat. The random initialized model needs more training iterations (450K) to reach the performance of the UniMatch initialized model.

【原文事实】骨干初始化对比：默认利用公开 UniMatch 预训练权重初始化并训练 300K 步；模型也可以随机初始化并超越此前先进方法 pixelSplat。随机初始化模型需要更多训练迭代（450K）才能接近 UniMatch 初始化模型的性能。

#### 原始数据与指标

| Method | PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---:|---:|---:|
| pixelSplat w/ DINO init (300K) [1] | 25.89 | 0.858 | 0.142 |
| MVSplat w/ UniMatch init (300K) | 26.39 | 0.869 | 0.128 |
| MVSplat w/ random init (300K) | 26.01 | 0.863 | 0.133 |
| MVSplat w/ random init (450K) | 26.29 | 0.868 | 0.128 |

K 表示千训练更新步，非 epoch。指标 PSNR(dB)↑、SSIM↑、LPIPS↓。附录 A 说 UniMatch 的预训练数据与本论文实验数据集没有重叠，这是作者报告，并未独立访问其训练资料；pixelSplat 使用 ImageNet 上 DINO 目标训练的骨干。

#### 比较、作用与限制

300K 相同步数下，随机 MVSplat 对 DINO pixelSplat 的 PSNR +0.12dB、SSIM +0.005、LPIPS −0.009，说明在该实验中本文的优势不完全依赖 UniMatch 预训练；但方法架构和初始化不同，不是纯预训练对照。同一 MVSplat 300K 比较，UniMatch−random 为 +0.38dB、+0.006、−0.005，说明预训练有助于给定训练预算下的表现。随机初始化由 300K 延长至 450K（步数增加 50%）带来 +0.28dB、+0.005、−0.005。450K random 与 300K UniMatch 比为 −0.10dB、−0.001、0，LPIPS 相同，其他指标略低，因此“达到”应理解为近似性能，而非每项严格相等。

【合理推断】预训练特征有助于早期多视角相似性学习，几何代价体使得从随机初始化仍可逐步构建匹配证据。此解释与表一致但不是梯度/特征分布的直接测量。作者未给出同样 450K UniMatch 结果、预训练总计算成本或多个随机种子，无法证明训练 450K 的随机模型达到最佳性能，也不能比较包含预训练在内的总体算力消耗。Table D 回答的是初始化依赖与收敛预算问题，不是无先验学习——网络架构和 RE10K 监督仍提供强先验。

<!-- PTH:END TABLE-D -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
### 图表之间的逻辑关系

#### 几何与画质证据链

【原文事实】Figure 1 是总览主张，Figure 2 是实现架构。Figure 2 的跨视角匹配、代价体、精炼及反投影解释了为什么作者不直接从特征回归高斯位置；它只是机制说明，尚未证明效果。Table 1 在 RE10K、ACID 两域比较所有方法的 RGB 指标、参数及端到端时间，Figure 3 和 C 以同域新场景的定性图补充表值；Figure 4 和 D 在未额外深度正则微调情况下展示拉远的高斯和参考图深度，对 Figure 1 的几何主张提供定性支持。它们共同回答两个不同问题：可否得到更好的目标图像？在拉远视角可否看到更一致、更少浮点的结构？前者有大量测试场景上的平均数值，后者只有示例，无真值深度或表面准确率统计。

| 主张 | 直接证据 | 辅助证据 | 未能证明 |
|---|---|---|---|
| 快速、轻量 | Table 1：0.044s、12.0M | Figure 1、Table B/C 的时间消融 | 任意分辨率和设备的实时性能 |
| 同域画质 | Table 1：两域三指标最佳 | Figure 3/C 的窗框、栏杆、台面等 | 所有个体场景都优 |
| 几何更规整 | Figure 4/D 的高斯及深度 | Figure 1、Figure 6 的对齐伪影 | GT depth 或完整表面精度 |

#### 泛化与消融证据链

Table 2 固定源数据 RE10K，在 ACID/DTU 无微调测试，Figure 5/E 展示同协议的典型可视结果；二者共同支持相对迁移优势，但 Table 2 题注与 Table 1 的 ACID pixelSplat 行后两项数值冲突，跨表对照须使用实际表行（见 Table 2 单元）。Figure E 的金属坐像例图并不消除 Figure B 所展示的反射局限。

Table 3 以 base 为统一参照，四种变化对应 Figure 6 中误差强度及空间位置；Figure A 提供验证曲线，说明 cross-view attention 删除配置取最佳步而非末步的原因。Table A 固定另一整体架构中的其他组件、替换其深度分支，是代价体可迁移作用的补充对照；Table B 说明 Swin 减少骨干采样成本但画质变化很小；Table C 说明增加高斯数带来微小画质提升和渲染成本；Table D 显示随机初始化仍可获得有竞争力结果，但更多训练步有益。Figure B 则指出反射局部错误，解释何时匹配机制不充分。图与表并非独立重复试验：可视化通常来自相应实验样例，不能据两份展示把证据强度算成两次独立验证。

【独立评价】图表顺序构成“总览→机制→同域/几何/迁移结果→消融→失败”的论证链。最有力度的受控证据是 Table 3 与 A 的匹配分支对照；但仍用 RGB 指标验证，缺少直接的几何量化。性能、速度和几何应分别评价，不能由一项推出另外两项。此单元是证据组织，不新增数学推导；数字的定义和差值见各图表单元。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 3. 正文精读

### 研究问题

#### 任务与输入输出

【原文事实】§1、§3 与式(1)：给定 $K$ 张稀疏已知位姿的 RGB 图像 $I^i\in\mathbb R^{H\times W\times3}$ 及投影矩阵 $P^i=K_i[R_i|t_i]$，利用预先跨场景训练的映射 $f_\theta$ 一次前馈预测像素对齐的高斯集合：

$$f_\theta:\{(I^i,P^i)\}_{i=1}^{K}\mapsto\{(\mu_j,\alpha_j,\Sigma_j,c_j)\}_{j=1}^{HWK}.$$

$K$ 表示输入图数量（与内参矩阵 $K_i$ 区分），$H,W$ 为图像分辨率，$\mu\in\mathbb R^3$ 为世界坐标高斯中心，$\alpha$ 为透明度参数，$\Sigma$ 为三维协方差，$c$ 为颜色球谐系数。默认每输入像素生成一个高斯；不同视角输出对齐到同一世界坐标，直接组合。给定任意目标相机的已知参数可渲染目标图像。输入通常为两张图；论文还测试三张图的推理。输出不是只生成一个目标 RGB，而是可重复渲染的显式 3D 表示；不是相机定位或 SfM，也不是输入未知位姿的任意图像重建。

#### 科学问题与歧义

任务表述是 sparse-view scene reconstruction 与 novel view synthesis；技术科学问题是：稀疏已知相机图像下如何充分利用跨视图几何对应，可靠、快速地估计像素对齐高斯中心，而不是仅用数据驱动的图像特征回归未知深度？

【背景知识】单个像素 $\tilde p=(u,v,1)^T$ 的空间射线包含无穷多个候选点。以相机 z-depth 定义为例，$X_w(d)=R_i^T(dK_i^{-1}\tilde p-t_i)$，每个 $d>0$ 在同一参考图投影为 $p$；所以单图不能唯一确定沿射线的位置。第二视图使不同 $d$ 通常具有不同投影，提供匹配证据，却仍受遮挡、低纹理、重复图案、视差极小或反射影响。目标图的 RGB 相同也不一定对应同一组高斯：尺度、透明度、颜色和位置可相互补偿。稀疏观测的约束不足是几何与外观联合逆问题中的真实歧义，而不是网络容量不足的同义词。

【原文事实】作者以 pixelSplat 的浮点几何（Figure 1/4）及代价体删除后对齐失败（Figure 6/Table 3）说明现有前馈几何学习困难。本文的关键是将几何候选和跨视图相似度作为中间表示，再从匹配中预测深度，而非通过额外 GT depth 或逐场景优化恢复中心。跨场景训练仍使用大规模已知位姿、目标 RGB 图监督；无 GT depth 不等于无监督。

#### 评价目标与边界

任务目标具有至少四个方面：目标图视觉质量（PSNR/SSIM/LPIPS）、从输入到新视角的效率（时间与参数）、显式三维结构的视觉质量（高斯拉远图、参考深度图）、跨训练数据集和输入数变化的泛化。Table 1、2、Figure 4/D 及 §4.2 more-view 实验提供不同证据。作者并未用 GT geometry 建立误差最小化或证明所有表面可辨识，输入相机、近远深度范围和跨场景数据仍是前提。MVSplat 不是通用无条件 3D 生成模型，不保证不可见区域几何正确，也未验证复杂参与介质和未知折射的成像。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
### 为什么需要这种方法？

#### 现有方法困难

【原文事实】§1–2 的起点是稀疏输入（少至两张图）和一次前馈，而原始 NeRF/3DGS 多依赖密集视图及逐场景优化。逐场景梯度反传使新场景推理慢；NeRF 渲染需要每像素沿射线多次体积采样，即使将网络跨场景前馈化仍承担采样成本。3DGS 将场景表示为显式高斯，栅格化避免 NeRF 的逐像素密集体采样，提供快速渲染潜力。但前馈 3DGS 的难点随之转到高斯参数，特别是中心位置：中心预测错误可能造成重影、漂浮及结构不稳定。pixelSplat 从双视图特征预测概率深度，再采样得到中心；作者认为缺乏明确几何匹配信息导致映射歧义和不可靠的几何，甚至需额外深度正则微调。Splatter Image 的单视图物体级设置则不能直接解决复杂场景的多视图几何。

#### 几何约束为何有效

【背景知识】一个候选点在参考像素射线上随深度改变，通常会在另一相机中沿极线移动。如果两视角对同一表面的特征具有一致性，正确深度对应的源特征与参考特征更相似。平面扫掠枚举 $D$ 个候选深度，不需要网络凭空猜绝对深度：每个参考像素得到一个长度 $D$ 的相似度序列，网络通过局部上下文、跨视图注意力修正不可靠匹配。对更多视图取相关性平均，使通道数与视角数解耦。其作用是将几何约束显式变成学习输入，压缩需要由数据学习的映射范围，而不是将歧义彻底删除。

【合理推断】假设两个平行相机的有效焦距 $f=100$ px、基线 $b=0.2$m，一点深度 $z=2$m，则视差为 $fb/z=10$px；候选 $z=4$m 只有 5px。若对应位置可辨，网络面对这两个不同的匹配值比直接把参考特征映射到深度更容易选择正确层。若图像无纹理、两投影像素都为同色，则该约束没有提供足够辨识度。以 1/4 分辨率做匹配还能降低空间计算，但薄结构和深度边界细节需精炼补回。

#### 证据与未证实

【原文事实】Figure 4/D 展示无额外微调的更平滑深度与更少浮点；Table 3 去掉代价体 PSNR 下降 3.29dB、LPIPS 增加 0.064，Figure 6 出现两输入直接叠加样伪影；Table A 在 pixelSplat 保持其他组件时替换分支，PSNR +0.74dB。这些是支持显式匹配有效的具体证据。Figure 6 与 §4.3 还显示单视图可见区域缺少匹配，需要 U-Net 的上下文；Figure B 的反射场景验证了多视图特征一致性不是无条件成立。

【独立评价】作者提出的不是“发现稀疏重建歧义”这一新科学现象，而是在前馈 3DGS 的中心估计环节采用经典 MVS 约束的结构设计。该结构减轻学习难度是有实验支持的解释，但论文未测量中心误差分布、模型学习复杂度的理论下界、匹配峰值与真值深度的统计关系。更少参数和更快推理亦受 Swin、2D U-Net、单高斯/像素影响，不可全归因于代价体。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
### 相关工作：与本文实际问题的联系

证据范围：以下只依据本论文 §2（pp.3–4）及 §1、§4.2、参考文献表；未独立读取引用文献全文，不能把作者对竞争方法的描述当成完整独立评价。

#### 稀疏视角方法分类

| 路线 | 本文列举及机制 | 作者指出的边界 | 本文关系 |
|---|---|---|---|
| 逐场景优化 | RegNeRF [26]、SPARF [39]、ReconFusion [43] 等，通过正则、约束或先验缓解稀疏观测 | 每个新场景需要反向传播，推理代价高 | 本文学习跨场景映射，测试时无需再优化 |
| 跨场景前馈 NeRF | pixelNeRF [49] 使用像素对齐特征；[3,5] 利用匹配，[10,25,30] 引入 Transformer，[3,44] 体积表示 | NeRF 沿射线的多点采样昂贵 | 本文采用可复用的显式 3DGS 与栅格化 |
| MuRF [44] | 目标视图 frustum volume + (2+1)D CNN，每目标视图重建相应体积 | 作者称训练昂贵、渲染较慢 | 本文一次从输入视图产生 3D 高斯，可重复渲染 |
| LFN 类 | §4.2 把 GPNR [35] 与 AttnRend [10] 纳入 Light Field Network-based 前馈基线 | 主表报告画质与时间；§2 未详细展开每种架构 | 作为 RGB 新视角合成基线，不等同于几何精度基线 |

【背景知识】前馈指推理不再对每个新场景逐次梯度优化；训练网络本身仍需优化。逐场景优化可以使用明确的物理/几何约束，不天然比前馈方法几何差；比较重点在本文的稀疏输入和时间协议。

#### 前馈 Gaussian 路线

- Splatter Image [37]：作者称其采用 U-Net，从单图回归像素对齐高斯参数，主要面对物体级重建。单图深度缺少直接多视角对应，因此对一般大场景存在固有歧义。本文增加多视角匹配信息。
- Flash3D [36]：本文称其将单图前馈扩展到场景级，但复杂场景受单图信息限制。未从原文之外补入它的深度先验或实现细节。
- pixelSplat [1]：最接近本文，双图输入、Epipolar Transformer 交换跨视角特征，再由特征预测概率深度分布并采样。作者认为从特征直接映射到深度分布仍有歧义，导致浮点几何；MVSplat 将候选深度的相似度显式编码为代价体。两者都使用跨视角信息和像素对齐高斯，故区别不是“pixelSplat 完全没有几何”，而是明确的匹配分布及中心估计机制、骨干效率、每像素高斯数。Table A 的分支替换验证部分可移植性，Table B/C 分别检验骨干及高斯数影响。
- GPS-Gaussian [53]：本文称其面向人体，使用两个校正后的立体图像预测视差，并训练时需要 GT depth。MVSplat 面向一般非校正多视图场景，可利用给定任意相机投影矩阵，仅 RGB 渲染监督。此处不意味着任意相机参数误差或畸变都可处理。

#### MVS 的继承与区别

【原文事实】典型 MVS 从多个有标定相机的图像估计深度，之后点云融合（本文列 [13,31,48]）；很多 MVS 网络需要 GT depth 训练（[9,13,48]）。MVSplat 借鉴经典平面扫掠与匹配（[7,46,48]）、相关性计算（[46,47]）及 MVS 网络思想，但将深度变成可微高斯中心，直接组合各视角的像素对齐高斯，不设传统的独立点云融合步骤；由可微渲染 RGB 损失训练深度与其他参数。原文还选择二维卷积/注意力而非很多 MVS 与前馈 NeRF 的三维卷积，以提高效率。

【独立评价】这是一种从显式深度证据到可渲染表示的结合：plane sweep、cost volume、Transformer、U-Net、3DGS 原语本身是继承组件，本文贡献是选择匹配表征、有效集成和验证。不能只因为本文未用独立几何监督便推断其几何更准确，也不能将作者列举的“许多 MVS”改为“所有 MVS”。本单元只组织相关工作与技术差异，不新增数学推导；式(2–6)的几何推导另见后文。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-01 -->
### 方法一：整体架构与多视角特征提取

#### 整体数据流

【原文事实】§3，Figure 2 及式(1)定义 $f_\theta$。输入 $K$ 张 $H\times W\times3$ 的 RGB 图及投影矩阵，输出 $HWK$ 个位置、透明度、协方差和颜色。不同模块的信息流可以按下表阅读（维度不包括 batch）：

| 步骤 | 输入 | 输出 |
|---|---|---|
| CNN + multi-view Transformer | $K\times H\times W\times3$ | $K\times H/4\times W/4\times C$ |
| 平面扫掠匹配 | 多视角特征、相机参数、$D$ 候选深度 | $K\times H/4\times W/4\times D$ |
| 代价体 refinement | 特征和代价体拼接 | 同尺度残差、精炼代价体 |
| CNN 上采样、softmax 期望 | 精炼代价体、深度候选 | $K\times H\times W\times D$ 分布、$K\times H\times W$ 深度 |
| Depth refinement | 图像、特征、当前深度 | $K\times H\times W$ 最终深度 |
| Gaussian heads | 深度/相机、置信度、特征/代价体/图像 | 每像素一组高斯参数 |
| Splatting | 对齐世界坐标的高斯、目标相机 | 目标 RGB 图 |

深度决定中心、匹配置信度参与透明度预测、图像和匹配特征参与协方差与颜色预测。整体表示由输入视图生成而与目标视图无关，可以一次编码多次渲染。训练时目标图像参与监督而非作为输入给编码器；推理不使用目标 RGB。本文所说“前馈”针对新场景，无须新场景的反向传播或深度微调；训练仍需大量跨场景数据与参数优化。

#### 多视角特征提取

【原文事实】§3.1 p.5 及 Appendix C pp.21–23：先用浅层 ResNet-like CNN 分别提取各输入图像特征。CNN 由 6 个 residual blocks 组成；最后 4 个块中每两个连续块后通过 stride=2 卷积实现一次空间二倍下采样，总计 $4\times$。若 $H=W=256$，则输出空间网格为 $64\times64$。残差块通过捷径传递输入特征，使网络可以学习对已有表示的修正；具体卷积核、每块通道数、激活函数未在论文逐项列出，不能据其他实现补齐。符号 $C$ 为特征通道数，论文此段未显式给出其数值；不要把附录中代价体 U-Net 的 128 通道误认为 Transformer 的 $C$ 已确认。

随后使用 6 个 Transformer blocks，每个包含一个 self-attention 层与一个 cross-attention 层。Self-attention 在同一图像特征中交换上下文；cross-attention 对参考视图 i 从其他视图的特征中取信息，得到 cross-view aware 特征 $F^i\in\mathbb R^{H/4\times W/4\times C}$。使用 Swin Transformer 局部窗口注意力，所有实验将特征分为 $2\times2$ 个区域（是空间划分为四个区域，不等同窗口内只有 $2\times2$ 个 token）。在 $64\times64$ 的示例下，若均匀划分，对应约 $32\times32$ 的区域；论文没有逐层列出窗口移动、边界 mask 或具体 head 数。

【背景知识】标准注意力可理解为 $A=\mathrm{softmax}(QK^T/\sqrt{d_k})V$，此处 $K$ 为 key 矩阵而非视角数。Self-attention 的 Q/K/V 来源于同视图，cross-attention 的 query 来自参考视图而 key/value 来自其他视图；是机制解释，非本文公开的具体实现公式。局部窗口降低全图 token 互相交互的空间开销，同时让局部外观、结构及另一图的信息进入每个参考特征。匹配阶段仍需显式相机 warp；Transformer 本身不直接输出物理深度。Table B 替换为 Epipolar Transformer 后时间为 0.055s 对 Swin 的 0.038s，质量变化仅 +0.03dB/-0.001/0（Swin 相对 Epipolar），支持效率取舍。

当 $K>2$，作者令每视图对其他全部视图进行 cross-attention，与两视图时使用相同数量的可学习参数。因此改变视角数不必改变网络参数维度。参数共享不等于 FLOPs 与 K 无关：更多输入特征、更多跨图交互和之后的 $K(K-1)$ 参考/源匹配仍增加成本。具体融合多个 key/value 的张量实现未完全报告，不据此声称严格置换不变或任意规模实时。

【独立评价】特征提取旨在增强跨视图可匹配性，代价体才把特征对应与候选深度联系起来；去掉骨干跨视图注意力的 Table 3/Figure A 结果表明该信息流有价值，但仍不能仅凭视觉质量推断特征在真实几何点上的精确相似度统计。

<!-- PTH:END METHOD-01 -->

<!-- PTH:BEGIN METHOD-02 -->
### 方法二：Plane sweep 与多视图代价体

#### 代价体构造

【原文事实】§3.1 pp.5–6 与 Eq.(2–4)。对 $K$ 输入，每个视角都作为一次参考视角 i，其余 $K-1$ 个视角 j 提供源特征；为每个参考视角建立独立 cost volume。给定 near/far depth range，在逆深度域均匀选取 $D$ 个深度候选 $d_m$（实验 $D=128$）。逆深度采样不是深度线性采样：若 $d_n,d_f$ 为近远平面，令 $\rho_m=1/d_n+(m-1)(1/d_f-1/d_n)/(D-1)$，$d_m=1/\rho_m$（含端点的补充表达，原文未指定精确端点次序）。近处实际深度间隔较小，远处较大；典型透视立体中视差与逆深度近似线性，因而更符合匹配几何。Appendix C 给 RE10K/ACID 近远平面 1、100，DTU 2.125、4.525（论文未给统一物理尺度单位，不能默认都为米）。

对一个参考像素 $p$，假定其深度 $d_m$，借助 $P^i,P^j$ 将其三维候选点投影到源视图的特征网格，取源特征。对整幅参考特征网格重复，得到同参考尺寸的 warped features：

$$F^{j\to i}_{d_m}=\mathcal W(F^j,P^i,P^j,d_m)\in\mathbb R^{H/4\times W/4\times C}.\tag{2}$$

这是反向采样式的 warp：输出位置在 i 上，取值来自 j，不是将源图直接平移或在源图深度上做未知三维重建；显式相机几何决定候选对应。实现需要使内参匹配 1/4 分辨率；精确插值/无效边界处理未在论文列出（典型做法是双线性特征插值，仅背景说明）。然后逐像素沿特征通道做缩放点积：

$$C^i_{d_m}(p)=\frac{F^i(p)^T F^{j\to i}_{d_m}(p)}{\sqrt C}.\tag{3}$$

这里的 cost 是相似度，高值通常为较好的匹配证据，而不是传统“代价越低越好”的距离。给定多于两视图，把每个源 j 的上述相关性在相同 i、p、m 下逐像素平均：

$$\bar C^i_{d_m}(p)=\frac1{K-1}\sum_{j\ne i}\frac{F^i(p)^TF^{j\to i}_{d_m}(p)}{\sqrt C}$$

这是把原文“pixel-wise averaged”展开为数学表达；原文未描述可见性自适应权重或越界掩码的细节。堆叠所有 $D$ 个相关性平面：

$$\mathbf C^i=[C^i_{d_1},\ldots,C^i_{d_D}]\in\mathbb R^{H/4\times W/4\times D}.\tag{4}$$

一个位置对应一条随候选深度变化的相似度曲线；匹配特征在 C 维求和后只保留一个标量，而 D 维不被求和。最终 $K$ 个参考视图分别获得 cost volume，并非一个统一世界坐标体素网格。它们将由共享的 2D 网络分别精炼并跨视图交换信息。

#### 维度、复杂度与限制

以 $K=2,H=W=256,D=128$ 为例，每个视角 $F^i$ 为 $64\times64\times C$；每个 $m,j$ warp 后仍为该形状，逐点相关性为 $64\times64$，代价体为 $64\times64\times128=524288$ 标量，两个视图总 1048576 标量（只计初始代价体，不含特征、激活、上采样和 batch）。朴素全部参考/源的匹配计算量按 $O(K(K-1)DHW C/16)$ 估计，不代表实际 kernel 时间；实际内存取决于是否一次保存所有 warped features 等实现。平均后存储规模仅 $O(KDHW/16)$，且 D 通道固定，可以使用二维卷积，无须每个源视图独立增加后续 U-Net 通道。更大的 K 并非零成本。

【合理推断】如果真实深度附近的特征在多视图间一致，相似度峰将接近真实深度；对较大深度范围，候选离散化限制初始匹配分辨率。近处纹理重复产生多个峰；无纹理导致近乎平坦；遮挡或参考独有内容时其他图没有真实对应；非朗伯外观改变可能使正确候选分数降低；极小基线使候选投影变化小。平均融合可减少部分随机噪声，但并不自动屏蔽遮挡视图，错误相似度可能拖低真实候选。输入位姿误差也会令所有候选 warp 偏离。没有 K=1 的跨视图相关性定义（分母为 0），不能把“任意视图数量”解读为无需其他图像。

【原文事实】作者用后续 cost volume refinement U-Net 缓解无纹理等歧义。Figure 6/Table 3 去掉代价体损失最大；Table A 移植代价体改善 pixelSplat。此证据支持设计有效，不是代价体每个峰的统计校准或几何真值证明。

<!-- PTH:END METHOD-02 -->

<!-- PTH:BEGIN METHOD-03 -->
### 方法三：代价体精炼、深度期望和深度精炼

#### 代价体精炼

【原文事实】§3.1 p.6 与 Appendix C pp.21–23。初始 cost volume 对无纹理/单视图区域存在歧义。将每视图 Transformer feature $F^i\in\mathbb R^{h\times w\times C}$ 与 $\mathbf C^i\in\mathbb R^{h\times w\times D}$ 在通道维拼接（$h=H/4,w=W/4$），产生 $h\times w\times(C+D)$，送入 2D U-Net。U-Net 输出 residual $\Delta\mathbf C^i\in\mathbb R^{h\times w\times D}$，与初值相加：

$$\widetilde{\mathbf C}^i=\mathbf C^i+\Delta\mathbf C^i.\tag{5}$$

以候选深度轴为通道、二维空间轴为卷积轴，2D U-Net 可以融合邻域结构及不同深度通道，而不是在 $H,W,D$ 三轴使用 3D 卷积。残差连接保留初始几何相似度，学习哪些位置和候选需要校正；它不强制最终分数保持原始匹配峰。Appendix C 使用参考[28]的二维 U-Net 实现，通道维在 U-Net 内部保持 128，进行两次 2× 下采样，在 4× 下采样级加 self-attention；若输入 $64\times64$，最低网格 $16\times16$。§3.1 指明最低分辨率有三层 cross-view attention，用于不同 cost volume 信息交换；附录还说 flatten features 后应用 self-attention 以实现 cost volume 之间信息传播。原文未完全给出 self/cross-attention 的逐层融合次序与精确张量排布，不能擅自补全。跨视图交互与 K 的参数共享逻辑类似骨干，后续可接更多视图。精炼后低分辨率 $\widetilde{\mathbf C}^i$ 用 CNN-based upsampler 上采样为 $\widehat{\mathbf C}^i\in\mathbb R^{H\times W\times D}$；这是对每个深度通道的全分辨率分数恢复，而不是先回归一张低分辨率深度图再简单插值。论文没有给出具体上采样核。

#### 深度预测及精炼

对每个全分辨率像素，将 $D$ 个 logits 沿深度维 softmax 为匹配分布 $p_m^i(p)$：

$$p_m^i(p)=\frac{\exp\widehat C_m^i(p)}{\sum_{l=1}^{D}\exp\widehat C_l^i(p)},\quad V^i(p)=\sum_{m=1}^Dp_m^i(p)d_m.\tag{6}$$

这将原文式(6) $\mathrm{softmax}(\widehat{\mathbf C}^i)\mathbf G$ 展开，$\mathbf G=[d_1,\ldots,d_D]\in\mathbb R^D$；输出每视角 $H\times W$。它是对实际深度候选的加权均值，虽候选在 inverse depth 域采样，也不应替换成“先对逆深度期望再取倒数”。均值是连续且可微的，比 argmax 易于端到端梯度训练；例 $d=(2,4,8)$、$p=(0.1,0.8,0.1)$，预测 $4.2$（非仅 4）。如果两个相距很远的候选同样高，均值可能落在没有实际表面的位置；这也是单高斯中心面对多模态分布的潜在限制。最大概率 $q^i=\max_m p_m^i$ 还为透明度分支提供输入，不是校准过的几何置信度真值。

随后独立的轻量 2D U-Net 接受多视角图像、特征及当前深度预测，输出每视角 residual depth $\Delta V^i\in\mathbb R^{H\times W}$。最终 $V^i_{final}=V^i+\Delta V^i$（按正文描述补充记号）。该网络同样在最低分辨率应用跨视图注意力，Appendix C 描述采用 4 次 2× 下采样，在 16× 下采样级加 attentions，配置与前一个 U-Net 类似。论文未完整说明特征与 RGB 拼接前如何对齐尺度、深度残差限幅、激活或通道细节；残差本身没有数学上保证最终深度仍在原候选范围。不能将作者未报告的 clamp 写成已实现。

【背景知识】2D U-Net 的编码器、解码器和跳接允许高层上下文与局部细节融合；相似度平坦的像素可以受附近可靠区域、图像边界等先验约束，跨视图低分辨率注意力则提供其他视角的上下文。与代价体精炼区别：第一个网络修改所有 D 个候选分数，再做概率平均；第二个修改已回归的一个深度标量，不再显式输出新的 D 维分布。两者在训练中均经高斯反投影与目标 RGB 损失更新，原文没有给直接深度标签监督。

【原文事实】Table 3：去代价体精炼 U-Net 对 base 的 PSNR -0.67dB，Figure 6 中单视图可见区域变差；加入最终深度精炼对 base +0.27dB，LPIPS -0.005。两者不是同一消融，不能将后者误作 cost volume refinement 增益。【独立评价】图表支持画质改善；两级精炼可能补足细节，但未直接证明每个像素深度变准，尤其遮挡/重复纹理等情形。

<!-- PTH:END METHOD-03 -->

<!-- PTH:BEGIN METHOD-04 -->
### 方法四：高斯参数联合预测与显式表示

#### 高斯中心和透明度

【原文事实】§3.2 p.7。每输入像素 $(u,v)$ 的最终深度由前一模块提供，利用相机参数反投影为相机坐标三维点，并变换到共同世界坐标。以 $P^i=K_i[R_i|t_i]$ 且 z-depth 为例，$\mu^i(u,v)=R_i^T(V^i_{final}(u,v)K_i^{-1}(u,v,1)^T-t_i)$。这是对原文“unproject and transform”的数学说明而非新公式编号。各视角直接并集 $\mathcal G=\bigcup_i\mathcal G_i$，工程上可理解为拼接各图所有像素对应的高斯实例；即使不同图观测同一表面，仍可保留多个原语，集合符号并不意味着计算了严格去重或传统点云融合。默认 $HWK$ 个高斯，空间尺度与相机坐标统一是融合的前提。

匹配分布 $p_m^i(p)=\mathrm{softmax}(\widehat C_m^i(p))$ 的最大值 $q(p)=\max_m p_m^i(p)$ 为 matching confidence，值域 $[1/D,1]$（有限 logits 下严格小于1）；作者认为高匹配置信度意味着更可能位于真实表面，与 opacity 的意义具有联系。用两个 convolution layers 从置信度输入预测 $\alpha$。不能直接设 $\alpha=q$，也不能把 $q$ 当作几何正确的概率：它度量分布峰值强弱，错误匹配也可十分尖锐；两个相等高峰的分布甚至有较高最大概率但深度期望可能落在两表面之间。$\alpha$ 是用于光栅化合成的高斯不透明度参数，不是物理占据概率，也不等同深度置信度。论文没有列出卷积核、输出激活、精确通道数。

#### 协方差与颜色

【原文事实】用两个卷积层从拼接的 image features、refined cost volume、original multi-view images 预测协方差和颜色。原文给出 $\Sigma=R(\theta)^T\mathrm{diag}(s)R(\theta)$，$R$ 由四元数表示，$s$ 是尺度项；原文未明确这里是否已包含尺度平方、正值约束或归一化，不能擅改成另一套 convention。如果 $R$ 为正交矩阵、diag(s) 非负，则协方差对称半正定。若采用标准差参数 $\sigma_i$ 则特征值为 $\sigma_i^2$ 是一般背景，不代表本文具体 $s$ 定义。颜色由预测的 SH（spherical harmonics，球谐）系数计算，在一般 3DGS 中可用 $c(\omega)=\sum_{l,m}a_{lm}Y_{lm}(\omega)$ 表示视角方向 $\omega$ 依赖的 RGB；论文没有给出 SH 的阶数、方向 convention 或输出系数数量，不能套用默认阶数。

【机制解释】中心决定高斯在哪里，协方差决定空间伸展/旋转及投影屏幕后的椭圆范围，透明度决定对像素累积权重，SH 决定方向相关颜色。图像特征、匹配和原始 RGB 拼接允许同时保留纹理与几何线索：中心需要跨视图一致的深度，协方差和颜色则可适应局部结构和外观。不同参数并非独立后处理；在可微渲染目标监督下，误差沿 $I_{pred}\rightarrow(\mu,\alpha,\Sigma,c)\rightarrow$各预测分支反传，同时中心还经过反投影和深度精炼、代价体。最大值算子对概率分量的局部梯度传向唯一最大概率层（平局处次梯度依实现）；由于 softmax 归一化，该概率对其他候选 logits 也有梯度，详见数学精读五。深度期望直接使用全部候选概率。

【独立评价】图像重建优化不保证高斯各参数唯一。低置信度处可以靠颜色和尺度弥补，遮挡区域可由低透明度高斯减少错误投影，中心、尺度、透明度和颜色可能相互补偿。Figure 4/D 的浮点减少与几何中心机制一致，但没有定量真值；Figure B 表明 SH 与匹配仍不足以完全处理镜面反射。Table C 的 1/3 高斯/像素对照展示表示容量取舍，不证明单高斯适合所有多表面射线。

<!-- PTH:END METHOD-04 -->

<!-- PTH:BEGIN METHOD-05 -->
### 方法五：训练、推理与实验实现

#### 训练流程

【原文事实】§3.3 p.7，§4.1 p.8，Appendix C p.23。每个训练样本是同一场景的两张 context/input RGB 图与四张 target RGB 图及相机内外参。输入图先经过上述 encoder 得到像素对齐高斯，依据各 target 相机进行可微 splatting，得到 $\widehat I_t$；只有目标 RGB 用作监督。训练损失由 $\ell_2$ 和 LPIPS 线性组合，权重分别 1、0.05。若对四个目标取平均，可写作（此处聚合形式为解释，论文未具体给出 batch/target 归约细节）：

$$\mathcal L=\frac{1}{T}\sum_{t=1}^T\left[\ell_2(\widehat I_t,I_t)+0.05\,\mathrm{LPIPS}(\widehat I_t,I_t)\right],\quad T=4.$$

光度项约束像素颜色，LPIPS 比较感知特征；不需要 GT depth/点云、不需额外几何正则微调，但仍有目标图标签和已估计相机参数，不能叫无任何监督。端到端梯度从目标 RGB 误差沿 CUDA rasterizer 返回高斯颜色、透明度、协方差、中心，再通过深度反投影、残差精炼、代价体、跨视角特征提取优化 $\theta$。实际可微 warp 的位姿固定输入、未报告同时优化相机。可见性变化/排序处梯度不保证处处光滑。

使用 PyTorch、现成 CUDA 3DGS renderer，Adam，默认单张 A100，300,000 次迭代；骨干使用 UniMatch 预训练权重初始化，Table D 的随机初始化例外为 300K 或 450K。batch size=14；Appendix C 英文表述每个 batch 包含一个训练场景，其中有 2 输入、4 目标，同时给出 batch size=14。可确认每个场景样本是 2+4 视图，原文对 14 与“每 batch 一个 scene”的措辞未完全消歧，不能擅自说 14 个不同场景或 14 张图。作者按训练进展逐步增大两个输入视图的 frame distance；这可理解为先学习较易匹配的近邻再处理较大视差（合理推断），具体 schedule 未报告。图像分辨率 256×256，D=128；RE10K/ACID near/far=1/100，DTU=2.125/4.525。学习率、准确训练耗时、batch 聚合及具体 LPIPS 网络细节未由正文完整提供。

#### 推理与实现

推理时只有稀疏图像、已知相机与深度范围，从网络前馈计算高斯；换目标相机直接渲染，不再对该场景进行 gradient back-propagation。训练与测试任务均是跨场景映射，测试不是从零训练 3DGS。Figure 2 说明 encoder 与 render 分离；§4.2 报告 MVSplat encoder=0.043s、render=0.0015s，pixelSplat 为 0.102/0.0025s。Table 1 总时间 0.044s 对 0.104s，约 22.7 次输入到单次新视图预测/s；已经有同一组高斯时可独立渲染约 667 帧/s（简单倒数，实验分辨率与计时条件限定）。不能把独立渲染速率当成输入变化后的完整前馈速率，也不能由训练 A100 硬件推断所有计时均来自同硬件。

输入数变化：特征/精炼的跨视角注意力使用同一可学习参数，代价体对其余源视角的相关性取平均，之后每视图各产生一个固定大小表示，因此两图训练可以测试三图。§4.2 的 DTU 三 context 结果 MVSplat 14.30/0.508/0.371，对两图 13.94/0.473/0.385（PSNR/SSIM/LPIPS）有所改善；改造的 pixelSplat 三图 12.52/0.367/0.585，较两图变差。作者说明为其公开双图模型做了尽力扩展，因此不是原生三图基线。K 增大会提高匹配、attention 与高斯渲染工作量；没有更多 K 或高分辨率规模曲线。

【独立评价】方法的实用优势是把场景特定求解转化为离线学习和在线前馈，适合快速从少量位姿已知图像生成可复用场景表示。代价是仍需大规模训练数据、可靠相机及相似的外观/几何统计。若测试图不满足训练分布、多视图没有重叠或出现复杂反射，网络并不保证恢复可靠真实几何；Figure B 和 §5 已指出此类边界。

<!-- PTH:END METHOD-05 -->

<!-- PTH:BEGIN MATH-01 -->
### 数学精读一：输入输出映射与反投影

#### 公式 1：映射

【原文事实】p.5 式(1)：

$$f_{\boldsymbol\theta}:\{(\boldsymbol I^i,\boldsymbol P^i)\}_{i=1}^{K}\mapsto\{(\boldsymbol\mu_j,\alpha_j,\boldsymbol\Sigma_j,\boldsymbol c_j)\}_{j=1}^{H\times W\times K}.\tag{1}$$

$\theta$ 是前馈网络的可学习参数，$i$ 是输入视角索引，$j$ 是输出高斯索引（注意与式2的源视角 j 含义不同）。$I^i\in\mathbb R^{H\times W\times3}$，$P^i=K_i[R_i|t_i]\in\mathbb R^{3\times4}$；$K_i$ 为 $3\times3$ 相机内参矩阵，$R_i$ 为旋转，$t_i$ 为平移向量，$K$ 为输入数量。$\mu_j\in\mathbb R^3$，$\alpha_j$ 为不透明度标量，$\Sigma_j\in\mathbb R^{3\times3}$ 为协方差，$c_j$ 为球谐颜色系数（阶数未报告）。像素对齐使 $j$ 可以由 $(i,u,v)$ 确定；两张 $256\times256$ 图为 $131072$ 个高斯实例。符号 $f_\theta$ 是学习映射定义，不是解析求解几何的闭式公式，真正的跨视角约束由式(2–6)给出。

#### 补充推导：反投影

【背景知识／补充推导】按本文投影矩阵约定，世界点 $X_w\in\mathbb R^3$，齐次坐标 $\widetilde X=(X_w^T,1)^T$，世界到相机为 $X_c=R_iX_w+t_i$。针孔投影：

$$z\widetilde p=K_i X_c=K_i(R_iX_w+t_i),\quad\widetilde p=(u,v,1)^T,$$

其中 $z=(X_c)_z$，假设内参最后一行为 $(0,0,1)$、无未处理畸变、$z>0$。乘 $K_i^{-1}$，有 $X_c=zK_i^{-1}\widetilde p$；再乘旋转逆 $R_i^{-1}=R_i^T$：

$$X_w(z)=R_i^T(zK_i^{-1}\widetilde p-t_i)=o_i+z r_i,\quad o_i=-R_i^Tt_i,\quad r_i=R_i^TK_i^{-1}\widetilde p.$$

$o_i$ 为相机中心，$r_i$ 为世界方向的未归一化射线，$z$ 为相机轴向深度；若论文的实现用射线欧氏长度，则应将 $r_i$ 归一化而不直接代入该 $z$，原文未给实现细节。以标准内参 $K_i=[[f_x,0,c_x],[0,f_y,c_y],[0,0,1]]$ 为例，$K_i^{-1}\tilde p=((u-c_x)/f_x,(v-c_y)/f_y,1)^T$，故同一像素对所有 $z$ 都投影相同。这直接解释单图不能确定高斯中心。假设 $R_i=I,t_i=0,f_x=f_y=100,c_x=c_y=50$，像素 $(60,70)$，$z=2$ 得 $(0.2,0.4,2)$，$z=4$ 得 $(0.4,0.8,4)$，两点均投影为同一像素。反投影的深度梯度 $\partial X_w/\partial z=r_i$；误差在世界坐标沿该射线传播，影响后续投影位置。

【原文事实】§3.2 用最终深度反投影各视角并对齐世界系，随后直接组合高斯中心。因此对每个像素 $\mu_{i,u,v}=X_w(V^i_{final}(u,v))$（z-depth 条件下），此映射可微。多个参考视图输出共用世界坐标的必要条件是相机内外参一致、深度的尺度一致；若相机仅在未知全局尺度下估计，渲染可在该坐标尺度中工作，但不能推出度量真实尺度。特征网格匹配时使用相应分辨率内参；若简单四倍缩小且像素坐标原点一致，$K_i'=\operatorname{diag}(1/4,1/4,1)K_i$；严格像素中心 convention 可能需偏移修正。反投影最终用全分辨率像素网格。本文未报告相机误差鲁棒性，也不优化位姿。

<!-- PTH:END MATH-01 -->

<!-- PTH:BEGIN MATH-02 -->
### 数学精读二：平面扫掠的几何关系

#### 公式 2：平面扫掠

【原文事实】p.6 式(2)：

$$\boldsymbol F^{j\to i}_{d_m}=\mathcal W(\boldsymbol F^j,\boldsymbol P^i,\boldsymbol P^j,d_m)\in\mathbb R^{\frac H4\times\frac W4\times C},\quad m=1,2,\ldots,D.\tag{2}$$

$\mathcal W$ 为 warping operation；$i$ 是参考图，$j$ 是源图，$F^j$ 是源图的 $h\times w\times C$ 特征（$h=H/4,w=W/4$），$d_m$ 是候选深度。输出仍在参考图网格，通道 C 不变。作者在 near/far 深度范围的逆深度域均匀采样 $D$ 个候选，实验 D=128。若包含端点并从近到远排列，补充写成

$$\rho_m=\frac1{d_n}+\frac{m-1}{D-1}\left(\frac1{d_f}-\frac1{d_n}\right),\qquad d_m=\rho_m^{-1}.$$

需要 $0<d_n<d_f,D\ge2$。对于 $d_n=1,d_f=100,D=128$，相邻逆深度差的绝对值约 $0.007795$；深度间距非均匀，近处约 0.00786、最远两候选约 43.8（由端点公式示例，不冒充作者实现中精确的采样 convention）。连续近似 $|\Delta d|\approx d^2|\Delta\rho|$ 解释越远间距越大。对于 DTU 近远 2.125/4.525，搜索范围较窄；相机尺度及单位由数据决定。

#### 补充推导与退化

【背景知识／补充推导】假设相机 z-depth、针孔相机模型及特征网格内参 $K'_i,K'_j$（已适配 1/4 分辨率）。对参考齐次坐标 $\tilde p_i$，按上节得到 $X_w=R_i^T(d_mK_i'^{-1}\tilde p_i-t_i)$。源相机坐标：

$$X_{c,j}=R_jR_i^T d_mK_i'^{-1}\tilde p_i+t_j-R_jR_i^Tt_i = R_{ji}d_mK_i'^{-1}\tilde p_i+t_{ji}.$$

令 $R_{ji}=R_jR_i^T,t_{ji}=t_j-R_{ji}t_i$，源特征点为 $\tilde p_j\sim K'_j X_{c,j}$，使用 $\pi(x,y,z)=(x/z,y/z)$ 将投影转为非齐次像素位置：

$$p_j(d_m)=\pi\big(K'_j(R_{ji}d_mK_i'^{-1}\tilde p_i+t_{ji})\big),\qquad F^{j\to i}_{d_m}(p_i)=F^j(p_j(d_m)).$$

如 $p_j=(x,y)$ 非整数，通常使用双线性插值 $F^j(x,y)=\sum_{a,b\in\{0,1\}}w_{ab}F^j(\lfloor x\rfloor+a,\lfloor y\rfloor+b)$，其中 $w_{ab}$ 是两个方向线性距离权重乘积，权重和为1；可在网格单元内连续求导。作者仅给 $\mathcal W$ 与引用，未明确插值/边界实现，此处为背景。源相机 z 必须为正，投影在图内且实际可见才存在可靠物理对应；遮挡时数学投影仍成立但取到的是其他表面特征。

把深度平面固定，齐次关系也可写为平面诱导变换：在参考相机坐标中 $n=(0,0,1)^T,n^TX_{c,i}=d_m$，有

$$\tilde p_j\sim K'_j\left(R_{ji}+\frac{t_{ji}n^T}{d_m}\right)K_i'^{-1}\tilde p_i.$$

这是本文 warp 的一种背景几何推导（符号正负依所用坐标定义，此处由 $X_{c,j}=R_{ji}X_{c,i}+t_{ji}$ 直接得出）。固定 m 时整幅参考网格对应一平面；遍历 m 即平面扫掠。真实三维场景不必是多个平面，平面只是逐像素搜索深度的离散假设。

特别地，两个平行相机，焦距 f，源相机中心相对参考向右平移 b，$R_{ji}=I,t_{ji}=(-b,0,0)^T$，则 $u_j=u_i-fb/d_m,v_j=v_i$。视差 $\delta=u_i-u_j=fb/d_m$，逆深度均匀则视差均匀。例在特征网格上 f=100px,b=0.2，参考 u=50，$d_m=2,4$ 分别映射到40、45。相同场景点在正确深度处更可能有高特征相似度。深度误差传播可由 $\partial\delta/\partial d=-fb/d^2$ 得出：基线越小、距离越远，等量深度变化产生越小的视差变化，噪声下越难辨识。若 $b=0$ 且仅旋转，$p_j$ 与 d 无关（旋转投影的齐次尺度抵消），匹配本身不给深度约束。相机标定误差、越界、遮挡、重复纹理与反射亦可造成错误/平坦峰。本文未给出显式遮挡检测与置信度标定细节。式(2) 的作用是生成可比较候选，不直接完成深度回归。

<!-- PTH:END MATH-02 -->

<!-- PTH:BEGIN MATH-03 -->
### 数学精读三：相关性与代价体

#### 公式 3–4：相关性和堆叠

【原文事实】p.6 式(3) 与(4)：

$$\boldsymbol C^i_{d_m}=\frac{\boldsymbol F^i\cdot\boldsymbol F^{j\to i}_{d_m}}{\sqrt C}\in\mathbb R^{\frac H4\times\frac W4},\quad m=1,2,\ldots,D.\tag{3}$$

$$\boldsymbol C^i=[\boldsymbol C^i_{d_1},\boldsymbol C^i_{d_2},\ldots,\boldsymbol C^i_{d_D}]\in\mathbb R^{\frac H4\times\frac W4\times D}.\tag{4}$$

这里 $C$ 是特征通道数量，粗体 $\boldsymbol C^i$ 是 cost volume，避免同符号含义混淆。点积逐参考像素进行而不是两张特征图全局内积；对 $f=F^i(p)$、$g=F^{j\to i}_{d_m}(p)$，得 $s=f^Tg/\sqrt C=\sum_{c=1}^{C}f_cg_c/\sqrt C$。缩放将 C 个通道的累加控制在适当量级。若各乘积独立、均值为0、方差约1，则 sum 方差 C，除以 $\sqrt C$ 方差约1（一般统计解释，并非论文证明特征满足这些假设）。梯度 $\partial s/\partial f=g/\sqrt C$、$\partial s/\partial g=f/\sqrt C$，用于渲染损失经后续深度网络回传到特征。更高点积一般表示更强匹配，因而“cost volume”在此实际为 similarity volume。

例 $C=4$，参考特征 $f=(1,1,0,0)^T$，三个深度候选源特征 $g_1=(1,1,0,0),g_2=(1,0,1,0),g_3=(-1,-1,0,0)$，得相关性 $s=(1,0.5,-1)$。这些值形成该像素长度 D 的序列，而不是直接输出深度。若第二源视图相同候选的相关性为 $(0.6,0.9,-0.2)$，逐像素均值为 $(0.8,0.7,-0.6)$。根据原文 $K>2$ 的 average，可写成

$$C^i_{d_m}(p)=\frac1{K-1}\sum_{j\ne i}\frac{F^i(p)^TF^{j\to i}_{d_m}(p)}{\sqrt C}.$$

每个 m 输出一个 $h\times w$ 平面，堆叠变成 $h\times w\times D$。对于 $H=W=256,D=128$，每参考视图 $64\times64\times128$，K 个参考视图维度 $K\times64\times64\times128$。若平均前保留每源特征，所需内存可能远高于此。视角数的平均不改变 D 通道数，适配后续二维网络；源视图顺序对这一步简单平均无影响（不等于完整注意力网络严格 permutation invariant）。

#### 匹配相似度边界

【独立评价】§4.2 将跨域泛化优势归于相对特征相似性相较绝对特征尺度更稳定。但 Eq.(3) 并非严格尺度不变：若 $f'=af,g'=bg$，则 $s'=ab s$；相同正尺度时 argmax 在理想纯点积序列可能不变，但 softmax 分布通常改变，若各候选或视图尺度不同，峰排序也可能改变。即便除以 $\sqrt C$，也未除向量范数 $\|f\|\|g\|$，所以不等同余弦相似度。若上例 f、g 同时乘2，分数从 $(1,0.5,-1)$ 变 $(4,2,-4)$，峰位置相同而概率更集中。作者未给出特征归一化步骤足以证明不变性，不能将经验迁移结论转成定理。

真实深度对应一般有高相似度，但重复纹理可有多峰、低纹理可使所有分数接近、单视图可见/遮挡可能无有效匹配、反射导致非一致外观。简单平均的局限：若某源因遮挡不支持真深度，平均可能压低真候选；没有报告显式可见性权重。后续二维 U-Net 的作用是在空间上下文和跨视图信息下校正 logits，不保证消除所有歧义。Table 3/A 的受控对照支持使用代价体，但无匹配置信度/深度误差的真值校准。

<!-- PTH:END MATH-03 -->

<!-- PTH:BEGIN MATH-04 -->
### 数学精读四：残差精炼、softmax 深度与梯度

#### 公式 5–6：残差与期望

【原文事实】p.6 式(5)：

$$\widetilde{\boldsymbol C}^i=\boldsymbol C^i+\Delta\boldsymbol C^i\in\mathbb R^{\frac H4\times\frac W4\times D}.\tag{5}$$

$\Delta\boldsymbol C^i=U_\phi([F^i,\boldsymbol C^i])$ 为补充记号，输入通道 C+D，输出 D，与原代价体同尺寸。残差允许以显式匹配为初始证据进行修正；例初始 logits $(1,1,0)$，在邻域线索帮助下 residual $(0.1,1.0,-0.2)$，得 $(1.1,2,-0.2)$，第二候选更突出。二维网络处理空间 H/4,W/4，D 作通道，原文不施加残差大小约束。CNN upsampler 将其恢复到 $\widehat C^i\in\mathbb R^{H\times W\times D}$。

【原文事实】p.6 式(6)：

$$\boldsymbol V^i=\mathrm{softmax}(\widehat{\boldsymbol C}^i)\boldsymbol G\in\mathbb R^{H\times W},\quad \boldsymbol G=[d_1,d_2,\ldots,d_D]\in\mathbb R^D.\tag{6}$$

逐像素展开：设 $l_m=\widehat C_m^i(u,v)$，$p_m=e^{l_m}/\sum_k e^{l_k}$，$p_m>0,\sum p_m=1$，则 $V=\sum p_md_m$。深度均值是候选深度的凸组合，初始 V 必在最小/最大候选之间（有限 logits 且候选不同则严格内部）。注意 $d_m$ 为实际深度，虽然候选均匀采样 inverse depth，$E[d]$ 与 $1/E[1/d]$ 不等同；例如两个等概率 2 和 8，前者5，后者3.2。

用 $d=(2,4,8)$，若 logits 为 $(0,\ln8,0)$，softmax 为 $(0.1,0.8,0.1)$，深度 V=4.2，峰值候选4；匹配置信度 $\max p=0.8$。若分布接近 one-hot，期望近似 argmax；若均匀，V 为候选的算术均值，不等于近远范围的中点（逆深度采样非均匀）。最终另一个 U-Net 预测 $\Delta V$，$V_{final}=V+\Delta V$（未编号补充记号），该残差修正后不再必然属于候选区间，原文没有给出限幅/正深度约束。

#### 梯度与多峰歧义

【背景知识／补充推导】softmax 的偏导

$$\frac{\partial p_m}{\partial l_k}=p_m(\mathbb 1_{m=k}-p_k).$$

由链式求导：

$$\frac{\partial V}{\partial l_k}=\sum_m d_m p_m(\mathbb 1_{m=k}-p_k)=p_k d_k-p_k\sum_m d_mp_m=p_k(d_k-V).$$

该导数表明，增加大于当前均值的候选 logit 会提高 V，增加小于均值的候选 logit 会降低 V，概率越小影响越弱。以上例 V=4.2，三个导数分别 $0.1(2-4.2)=-0.22$、$0.8(4-4.2)=-0.16$、$0.1(8-4.2)=0.38$，和为0，符合所有 logits 同加常数不改变 softmax。若渲染损失对深度的局部梯度为 $g=\partial L/\partial V$，则仅这条深度路径贡献 $\partial L/\partial l_k=g p_k(d_k-V)$；还有置信度 opacity、其他参数头及深度精炼的梯度，不能将其视为总梯度。Softmax 可做数值稳定实现 $e^{l_m-\max l}$，这属于通用计算解释。

温度 $\tau$ 的背景分析（本文未使用明确温度）：$p_m\propto e^{l_m/\tau}$，导数增加 $1/\tau$ 因子。$\tau\to0$ 且峰唯一时接近 hard argmax，梯度可能因饱和变弱；$\tau\to\infty$ 接近均匀，深度退化为平均候选。匹配置信度最大值只反映概率峰尖锐程度，不是深度误差的可靠估计。可作为背景用方差 $\sigma_d^2=\sum p_m(d_m-V)^2$ 区别单峰/多峰，但论文没有用该方差监督或评价。例 $d=(2,4,8),p=(0.49,0.02,0.49)$，V=4.98，落在两个大峰之间且可能不是真实表面；若两个高峰分别代表前景与背景，单高斯中心的均值可能几何错误。用软期望可微回归有代价：没有直接深度监督时可出现匹配不确定性与渲染误差补偿。Depth refinement 可改善局部深度但未证明它解决所有多峰。Figure 6/Table 3 证实画质改善，不是单像素真实深度校验。

<!-- PTH:END MATH-04 -->

<!-- PTH:BEGIN MATH-05 -->
### 数学精读五：高斯参数、渲染与监督

#### 高斯参数数学

【原文事实】§3.2 定义中心来自最终深度反投影（详见式1推导）。由匹配分布最大值 $q^i(p)=\max_m p_m^i(p)$ 作为透明度头输入，两个卷积层得到 $\alpha$；这是学习映射 $\alpha=\mathcal H_\alpha(q)$（补充记号），不直接取最大值。对于归一化分布 $1/D\le q\le1$；有限 logits 下 q<1。若最大候选唯一，$\partial q/\partial l_k=p_{m^*}(\mathbb 1_{k=m^*}-p_k)$，平局处最大算子不可微但可由实现使用次梯度；这说明虽然只取某个最大概率，其 softmax 归一化仍使其他 logits 影响该概率。没有真值校准，因此不透明度不是统计正确匹配概率。

论文协方差原式：

$$\boldsymbol\Sigma=R(\theta)^\mathsf T\operatorname{diag}(s)R(\theta).$$

旋转矩阵由四元数表示，尺度项 s 构成对角矩阵，颜色由预测 SH 系数计算。这里保留原文转置顺序，不把 s 擅改成标准差平方；若 $R^TR=I,s_k\ge0$，则 $\Sigma$ 对称，任意 $x$ 满足 $x^T\Sigma x=(Rx)^T\operatorname{diag}(s)(Rx)=\sum s_k(Rx)_k^2\ge0$。若全部 s 正则正定可逆。例 $R=I,s=(1,4,9)$，协方差 diag(1,4,9)，主轴标准差为1、2、3（作为数学说明）。若使用通常的高斯函数，背景表达为 $g(X)=\exp[-\tfrac12(X-\mu)^T\Sigma^{-1}(X-\mu)]$；不包含概率密度归一化常数，因为此处是渲染原语强度表达。各向异性协方差可贴合墙面或薄结构。原文没有说明 s 正值实现、四元数归一化、SH 阶数或具体参数约束，不能宣称已验证。

【背景知识】方向相关颜色可写 $c(\omega)=\sum_{l=0}^L\sum_{m=-l}^l a_{lm}Y_{lm}(\omega)$，每个 $a_{lm}$ 可为 RGB 向量，$L$ 是最大阶数，$\omega$ 为单位观察方向。低阶 SH 表达平滑视角变化，但不是显式 BRDF/光照分解。论文没有给 L，不为其填默认数字。两个卷积层的输入是原图、image features、refined cost volume 的拼接，因此高斯空间伸展和颜色既可依赖匹配也可依赖外观。

#### 渲染与训练损失

【背景知识／非本文新推导】作者用现成 CUDA 3DGS renderer，未在本文重写栅格化数学。为理解反传，可近似描述针孔投影下三维高斯中心先变换到目标相机 $X_c=R_t\mu+t_t=(x,y,z)^T$，投影 $u=f_x x/z+c_x,v=f_y y/z+c_y$。投影在中心附近的雅可比

$$J=\begin{bmatrix}f_x/z&0&-f_xx/z^2\\0&f_y/z&-f_yy/z^2\end{bmatrix},\quad\Sigma_{2D}\approx J R_t\Sigma R_t^T J^T.$$

该局部线性化要求 z>0、尺寸相对于深度不过大，实际 renderer 可能有屏幕滤波、裁剪等未描述步骤。设投影均值 $m_j$，像素 p 的屏幕权重 $w_j(p)=\exp[-\tfrac12(p-m_j)^T\Sigma_{2D,j}^{-1}(p-m_j)]$，有效 alpha $a_j(p)=\alpha_jw_j(p)$。按近到远排序的背景 alpha compositing：

$$T_j(p)=\prod_{k<j}(1-a_k(p)),\quad\widehat I(p)=\sum_j T_j(p)a_j(p)c_j(\omega)+T_{end}(p)c_{bg},$$

$T_{end}=\prod_k(1-a_k)$，背景项取决于 renderer。它说明中心移动改变屏幕投影和排序，协方差改变覆盖范围，透明度改变透射累计，颜色改变 RGB；并非论文新提出的渲染公式。相较逐目标射线密集采样 NeRF，splatting 使用显式投影和混合；不代表完全无体积/可见性计算。

【原文事实】§3.3 仅目标 RGB 监督，L2 和 LPIPS 权重分别 1 和 0.05；可记为 $L=\ell_2(\widehat I,I_{GT})+0.05\mathrm{LPIPS}(\widehat I,I_{GT})$。若 $\ell_2$ 用均方误差 $\|\widehat I-I\|_2^2/n$（常见实现，论文未指定精确 reduction），则像素梯度为 $2(\widehat I-I)/n$，LPIPS 通过感知特征的差传递梯度。链式法则中心路径示意 $\partial L/\partial V=(\partial L/\partial\mu)^T\partial\mu/\partial V$，再经 Eq.(6) 的 $p_k(d_k-V)$ 回传给代价体 logits；此外透明度、协方差与颜色等并行路径共同优化特征。中心投影/屏幕权重对位置可局部求导，排序或遮挡切换可能非光滑，不能说所有操作全局处处可微。

【独立评价】仅 RGB 监督的不可辨识性：两个不同中心沿输入相机射线但通过改变尺度和颜色可能在少数目标视角产生相似图像；低 alpha 原语的位置可能几乎不受 RGB 梯度约束；SH 颜色可以部分补偿几何错误。代价体提供额外结构先验，减少但不消除歧义。Figure 4/D 的高斯和深度定性证据与此机制一致，Table 3/A 支持画质上的效果。没有 GT geometry、分支梯度/深度精度统计或物理材质真值，不能推出唯一正确的三维结构。

<!-- PTH:END MATH-05 -->

<!-- PTH:BEGIN EXP-01 -->
### 实验一：同域评估与效率

#### 数据和评估协议

【原文事实】§4.1（PDF pp.7–8）：RealEstate10K 来自房产视频，训练/测试场景 67,477/7,289；ACID 为航拍自然场景，11,075/1,972；均提供估计的相机内外参。两者分别训练模型，再在同域新场景测试；每场景输入两幅上下文图像，渲染三幅新视角。DTU 则是具有相机位姿的物体中心多视图数据集；在 16 个验证场景各评估四个新视角，供跨域实验使用。全部比较采用 256×256。PyTorch 网络及现成 CUDA 3DGS 渲染器，Transformer 六组自/跨视图注意力层，128 深度候选，单 A100，Adam，300K 次迭代。附录 C 的训练上下文与目标数为 2 和 4，不要误读为测试目标数。

三种指标分别覆盖像素差、局部结构和感知特征差。补充说明：PSNR = $10\log_{10}(M^2/\mathrm{MSE})$，同一动态范围 $M$ 下越大表示均方误差越小；SSIM 越大通常表示结构更近；LPIPS 越小表示指定感知特征空间更近。它们均不直接测量三维表面误差。以相同动态范围和同一评估聚合口径比较，PSNR +0.5 dB 对单个 MSE 对应约 $10^{-0.05}=0.891$ 倍，但数据集平均 PSNR 不能无条件换算为平均 MSE 的比例。

#### 主结果与效率

【原文事实】Table 1、Figure 3/4：与光场网络 GPNR/AttnRend、NeRF 方法 pixelNeRF/MuRF、3DGS 方法 pixelSplat 比较。前四个已有结果（pixelNeRF、GPNR、AttnRend、pixelSplat）来自 pixelSplat 论文，MuRF 由作者使用官方代码重训；并非全部由作者统一重新实现。MVSplat 在两个基准的三指标均最优（限表中比较）。RE10K 的 26.39/.869/.128 相对 pixelSplat 的 25.89/.858/.142 改善 +0.50/+0.011/−0.014；ACID 的 28.25/.843/.144 相对 28.14/.839/.150 改善 +0.11/+0.004/−0.006。差值更大的 RE10K 是本文证据而非普遍规律。Figure 3 的窗框、楼梯扶手、灯罩与桥梁支持困难区域的视觉改善，Figure 4 的放大视野高斯及深度支持少漂浮、较平滑的定性几何判断。

表中端到端时间 0.044 s 对 pixelSplat 0.104 s（约 2.36×），参数 12.0M 对 125.4M（约 10.45× 少；Fig.1 写 125.1M，保留差异）。分阶段的 encoder 为 0.043 对 0.102 s；render 为 0.0015 对 0.0025 s，对应渲染速度约 667 与 400 FPS，仅指各自已预测高斯的渲染耗时，不能等同完整推理 FPS。两阶段之和 0.0445 与 0.1045 s 和表格报告存在舍入/统计口径差，未见精确计时协议，不能人为修改。128 深度候选与四分之一尺度特征限制 encoder 规模，1 Gaussian/pixel 而非 3 解释部分 renderer 提速；两输入 256×256 时未剔除的原语数分别为 131,072 与 393,216。模型参数不等于运行显存，GPU、输入尺寸、视图数与时间统计方式均可能影响比较，原文未给完整可复现实测延迟分布或 FLOPs。

【独立评价】实验能证明在给定协议下画质/效率的优势；无需额外深度正则微调的定性重建是重要证据，但没有深度 RMSE、Chamfer、F-score、几何尺度误差或统计置信区间，不能以平滑程度替代准确性。对其他场景及硬件的泛化仍需验证。

<!-- PTH:END EXP-01 -->

<!-- PTH:BEGIN EXP-02 -->
### 实验二：泛化与输入数量

#### 跨数据集零样本

【原文事实】§4.2、Table 2、Figure 5/E：仅在 RealEstate10K 两视图训练的模型直接用于 ACID 和 DTU，不在目标域再训练。ACID 上 pixelSplat 27.64/.830/.160，MVSplat 28.15/.841/.147（+0.51/+0.011/−0.013）；DTU 上分别 12.89/.382/.560 与 13.94/.473/.385（+1.05/+0.091/−0.175）。DTU 的 LPIPS 相对下降约 31.25%，但绝对 PSNR 仍很低，优势不等于可用的精确几何。Figure 5/E 展示户外与物体场景的差异；DTU 外观、尺度、相机分布相对训练域变化较大，不能量化每个因素的因果贡献。

必须区分源域训练与目标域训练：ACID 零样本 MVSplat 28.15/.841/.147，比 ACID 专门训练 pixelSplat（Table 1 实为 28.14/.839/.150）略优 +0.01/+0.002/−0.003，却弱于 ACID 专门训练 MVSplat 28.25/.843/.144。Table 2 caption 将 .843/.144 错归给 ACID 专门训练 pixelSplat，正文定性叙述与 Table 1 的正确比较相容；不可静默改写原题注，也不可宣称该细小优势显著。作者解释为 RE10K 较大的训练集（文中称约 7×，所列训练场景比约 6.09×，未说明其他计数）及代价体跨域鲁棒性。此为合理机制解释，非控制训练量的因果实验。

#### 输入视图数泛化

DTU 同一 RE10K 两视图训练模型，测试视图数改为三：MVSplat 达 14.30/.508/.371（比二视图 +0.36/+0.035/−0.014），pixelSplat 12.52/.367/.585（−0.37/−0.015/+0.025）。作者尽力修改已公开的二视图专用 pixelSplat 以支持更多视图，但未将其按三视图重新训练，故只能解释所述扩展与设置，不能说其他合理多视图实现必然退化。

机制上，参考视图 i 的候选深度分数对另外 K−1 个视图取平均，每个参考视图共享网络参数，所以新视图可以加入匹配集合；当额外视图能提供一致的有效匹配时可能使真实候选更易区分。若其被遮挡、位姿错误或特征域变，则等权平均也可能冲淡证据。两视图到三视图的实验支持可变 K 的实际可行性；K=1 没有源视图相关性，不能直接套用式(3)，更高 K 的精度、计算和鲁棒性并无相应曲线。理论粗算对全部有序参考—源视图组合为 $K(K-1)$ 次候选匹配，3 视图比 2 视图的组合数 6 对 2，原语数也增加 1.5 倍，不意味着计算恒定。

【独立评价】本文将相机几何作为显式归纳偏置，减少只依赖训练域特征分布的风险，但未测量特征尺度、相机位姿扰动或遮挡比例对泛化的独立作用，不能排除预训练、网络容量和训练量的共同影响。

<!-- PTH:END EXP-02 -->

<!-- PTH:BEGIN EXP-03 -->
### 实验三：主消融的因果范围

#### 主消融设计

【原文事实】§4.3、Table 3/Fig.6/A。以 RE10K、相同主要训练设置为背景，base 是移除最终 depth refinement 的网络，不是所有模块都去掉。其 26.12/.864/.133，最终 base+refine 26.39/.869/.128；w/o cost volume 22.83/.753/.197；w/o cross-view attention 25.19/.852/.152；w/o U-Net 25.45/.847/.150。除去跨视图注意力的模型，其他结果取 300K 收敛步；该例因后期过拟合取其最佳表现步，图 A 可见相应验证退化。比较差值必须以 base 为对照，而不是把 full 和 w/o 项直接当单因素实验。

#### 实验机制与限制

- Cost volume：−3.29 dB/−.111/+.064，LPIPS 相对恶化 48.1%。图 6 中去掉成本体导致两幅输入图像直接重叠是作者解释，说明显式候选几何对三维对齐的重要性；它不是实际点云误差的定量测量。
- Backbone cross-view attention：−.93 dB/−.012/+.019。移除 Transformer 的跨视图信息交互后特征不再通过此模块互补，相关性判别更难；图 6 误差增强，图 A 后期验证退化。注意 U-Net 本身还存在跨视图交互设计，不能把该消融简单解释为取消整个网络所有跨视图信息。
- Cost volume refinement U-Net：−.67 dB/−.017/+.017。图 6 中间共同可见区域近似 base，左右仅单视图可见内容退化；当某点仅有一个有效观测，显式匹配不能可靠确定深度，网络上下文先验填补信息，而非产生新的几何观测。作者的“高频细节映射”是渲染证据支持的机制解释。
- Depth refinement：full 相对 base +.27 dB/+.005/−.005；对候选深度期望的直接残差可以修正标量细节与边界，与修改候选分数的 U-Net 不同。画质改善并不保证每点深度误差都减小。

若以 PSNR 粗略判断相对贡献，成本体 > backbone 跨视图注意力 > 成本体精炼 > 深度精炼；该排序是本协议下逐项删除的效果，不是彼此独立贡献值，也不可相加。移除模块改变容量、特征分布、优化轨迹等，且注意力变体用了不同选步规则，因此不能完全归因于单一内部物理机制。缺少多随机种子方差、联合消融、真值几何、分遮挡比例及纹理/反射条件的性能统计，不能直接证明几何误差变化。图 A 支持过拟合现象，不足以证明其唯一原因。

<!-- PTH:END EXP-03 -->

<!-- PTH:BEGIN EXP-04 -->
### 实验四：扩展消融和可靠性

#### 附录额外消融

Appendix A（pp.18–20），除另述外按 §4.3 设置，RE10K 全测试集平均。Table A 替换 pixelSplat 的概率密度深度分支为本文成本体深度，保持其他组件：25.89/.858/.142 → 26.63/.875/.122，+0.74 dB/+.017/−.020；甚至超过独立 MVSplat full 的画质指标（但结构、参数与速度不同）。说明成本体机制具有跨架构复用的实际证据，不证明任意架构或数据域均提升。

Table B 在 MVSplat 比较 Epipolar Transformer 和 Swin：0.055 s、26.09/.865/.133 对 0.038 s、26.12/.864/.133。Swin 时间缩短约 30.9%，PSNR +.03、SSIM −.001、LPIPS 相同，精度近似，不应写成所有指标提升；两者的空间注意力限制方式不同，视图对应仍由显式代价体承担。此表计时与 base 对应，不可把 .038s 直接替代 full 的 .044s。

Table C：1/3 Gaussian per pixel 的 render 时间 0.0015/0.0023 s，画质 26.39/.869/.128 对 26.54/.872/.127；提升 +.15/+.003/−.001 伴随约 53.3% 渲染时间增长。增加原语可提高表示自由度，也可能增加重叠与排序负担，不能认为原语数三倍即耗时三倍。

Table D：pixelSplat 的 DINO 初始化（300K）25.89/.858/.142；MVSplat UniMatch（300K）26.39/.869/.128；随机初始化（300K）26.01/.863/.133，450K 为 26.29/.868/.128。与预训练同 300K 比 −.38/−.006/+.005，多训练 50% 后差值 −.10/−.001/0。支持随机初始化仍优于对比的 pixelSplat，以及预训练有利于收敛，但没有完整等训练预算的所有基线或方差实验，不能断言预训练没有作用。作者声明 UniMatch 数据不与本文实验数据重叠，未独立检查外部训练数据。Figure A 各消融验证曲线补充收敛过程，Figure C/D/E 则扩充同域图像、定性几何与跨域视觉案例。

#### 实现、失败与应用边界

Appendix C：骨干 UniMatch 权重，成本体 U-Net 通道 128、两次 2× 下采样、在最低尺度交互；depth refinement 四次 2× 下采样、在 16× 下采样层注意力。原文主体描述 cross-view attention、附录描述 flatten 后 self-attention 实现跨视图传播，不据此擅自确定未报告的张量布局。训练单 A100，batch size 14；附录同句称 each batch contains one training scene（原文表述存在歧义），每训练场景两输入四目标，随训练增大输入帧距离。near/far RE10K 和 ACID 为 1/100，DTU 为 2.125/4.525；这是数据集特定范围选择，不能推断物理米制尺度或任何环境自动适应。D=128 等间隔 inverse depth 的候选会将更多采样集中在近处。

Appendix A Limitation / Fig.B：非朗伯和反射表面较难，视角变化导致同一三维点的外观变化，正确投影深度不必具有高特征相关性。作者提出 BRDF 与更多样数据作为未来方向，并未实现或验证。Potential negative societal impacts 指复杂现实场景可能生成不可靠结果，在自动驾驶等安全关键用途合成训练数据需谨慎。对于不可见区域或跨域失败，额外学习的外观/几何先验可能生成视觉上合理但不真实的内容；论文没有提供不确定性校准或安全认证。

【独立评价】上述补充实验把跨架构复用、注意力选择、原语数及预训练影响拆开，提升了机制论证的可信度；但整体仍是新视角图像指标为主，附录图不是几何测量或安全性评估。

<!-- PTH:END EXP-04 -->

<!-- PTH:BEGIN SYNTHESIS-01 -->
### 综合分析：从论文主张回到证据

#### 主张—证据矩阵

评级仅对当前论文、指定版本和实验条件，强＝有直接定量对照；中＝有直接但有限的定性/间接证据；待验证＝没有对应实测。不能把评级当成论文外的权威结论。

| 主张 | 主要证据 | 支持程度与边界 |
|---|---|---|
| 稀疏两视图 NVS 画质超过表内基线 | Table 1；Fig.1/3/C | 强：两个基准三指标；不是所有数据集或最新方法 |
| 编码轻量、渲染快 | Table 1/C、§4.2 | 强：指定分辨率下参数/时间；不是所有GPU、分辨率、K |
| 三维中心更符合场景结构 | Fig.4/D；Fig.6 | 中：少漂浮、深度较平滑及渲染对齐；无真值几何量化 |
| 几何引导有利于跨域 | Table 2、Fig.5/E | 强于表中 pixelSplat 的输出指标；中于作者的因果解释 |
| 增加输入视图可改进结果 | §4.2 DTU 2→3 | 中：实际改善一个数据集的三指标，缺多 K 曲线 |
| 成本体是重要机制且可复用 | Table 3/A、Fig.6/A | 强：删除和替换实验证明本配置画质贡献；不等于任意场景必需 |
| U-Net 修复单视图可见区域 | Table 3、Fig.6 | 中：区域示例与整体指标；没有分可见性统计 |
| 无额外深度正则微调也能重建 | §4.2 Fig.4 | 中：无需该额外流程的定性效果；不是绝对几何精度 |
| 反射面可靠性不足 | Fig.B | 中：真实失败示例，无按材质定量 |
| 深度概率是校准的不确定性 | Eq.(6)、§3.2 | 待验证：只使用最大匹配概率预测 opacity，无校准实验 |

#### 替代解释与实验边界

模型规模、Gaussian 数、初始化和训练分布都是竞争解释。Table A 将深度分支替换到 pixelSplat，有助于排除仅仅“整体网络更小”的解释；Table C 明确说明较少原语是 render 优势来源之一，不能把全部效率优势归给代价体；Table D 显示预训练帮助收敛但随机初始化仍有竞争力。没有统一计算量控制或跨模型等训练数据/预算实验，不能完全排除这些因素。

在几何机制层，RGB 损失约束投影结果而不是三维唯一解。若多个候选 $d_1,d_2$ 的渲染误差近似相同，光度监督不足以排除错误中心；显式匹配使正确深度更可能受到跨视图一致性支持，但重复纹理、弱纹理、遮挡和反射均能改变相关性。高斯尺度/不透明度/方向颜色补偿也可能使图像改善而几何仍偏。论文有定性深度与原语图，但缺真值几何误差、匹配校准、姿态敏感性、尺度误差及按场景类型分布。跨域的主因也未通过固定训练量、受控位姿和光照等干预实验分离。图像指标的平均值未附方差或显著性检验。

总体评价：最有说服力的是将多视图几何结构注入前馈 Gaussian 位置估计，既得到本基准画质增益，又减小编码器和原语负担；不是新的 Gaussian 光栅化器，也不是首次发明 cost volume。将贡献限定在该组合、实现及实证，能够避免把继承的 plane sweep、相关性、softmax 和 3DGS 基础理论说成全新理论。

<!-- PTH:END SYNTHESIS-01 -->

<!-- PTH:BEGIN SYNTHESIS-02 -->
### 综合分析：适用范围、未解问题及完整性审计

#### 论文自身技术局限

【作者明确指出】Appendix A / Fig.B：非朗伯与反射表面的效果较差，建议未来引入 BRDF 及更丰富的训练数据；真实复杂场景可能不可靠。§4.2 的跨域结果表明相对优势，不代表 DTU 的绝对画质令人满意；图5/E 及 Table2 的数值体现了剩余域差。

【基于机制的补充评价】(1) 可见性：共同可见才有真实多视图约束。Figure 6 中只见于单图的内容由 U-Net 先验改善，但先验不保证几何正确；完全未见区域更无直接证据。(2) 深度候选：固定 near/far 与有限逆深度分辨率使范围外结构无正确候选；远处候选间距更大，深度残差可能缓解但不保证恢复。(3) 位姿：Eq.(2) 依赖已知/估计内外参，误差会造成投影偏差，论文并未联合估计位姿或给扰动实验。(4) 更多视图：平均相关性不显式判别不同源视图可信度；增加有用观测与增加冲突观测不同。(5) 表面几何：每像素一个 Gaussian 的中心依赖深度及相机，跨参考视图重复原语直接合并而非显式融合为单一网格。没有拓扑/表面精确性保证。(6) 外观：SH 方向颜色不是显式 BRDF 或成像传输模型；光度监督可能经其他参数补偿深度错误。以上是分析而非作者已验证的新结论。

#### 未证实问题及覆盖审计

精读报告覆盖原 PDF pp.1–23：标题、摘要、贡献、结论、正文 §1–2 的问题与相关工作、§3 方法（3.1–3.3）、§4 实验（4.1–4.3）、§5 结论及 pp.15–17 参考文献的引用角色；Appendix A 额外实验与局限、B 更多可视化、C 实现训练。参考文献是作者引用语境，未声称逐篇独立阅读。按实际目录而非预设章节数审查。

正文 Fig.1–6 与附录 Fig.A–E 均已逐幅分析、保留题注及对应 JPEG/区域 PDF；正文 Tables 1–3 与附录 Tables A–D 已逐张转录和分析。公式覆盖 Eq.(1) 图像及相机到 Gaussian 参数的前馈映射、Eq.(2) 候选深度 warp、Eq.(3) 缩放点积、Eq.(4) 成本体、Eq.(5) 残差精炼、Eq.(6) softmax 深度期望，以及 §3.2 未编号协方差、Gaussian 参数、§3.3 训练目标；补充数学推导单独标记，未把它们归作作者原创。前文已经逐项解释维度、坐标、梯度与边界。

保留三个原文口径问题：Fig.1 与 Table1 的 pixelSplat 参数 125.1M/125.4M；Table2 题注的 ACID 基线 SSIM/LPIPS 归属与 Table1 冲突；训练场景数量比约 6.09 与正文“约7×”未给出完全相同计数依据。另外 Appendix C 的 batch size 与场景表述、分阶段计时与舍入均按原文保留。PDF 是事实主证据；Source/HTML 辅助，未独立验证 Source 版本。图像仅声明 B2 实际检验通过的区域保真，不承诺像素无损或聊天客户端显示。

开放问题：在受控真值几何下，成本体提高的是深度准确度还是主要改善可渲染性？候选置信度能否预测真实误差？不同位姿误差、纹理重复、反射、基线、遮挡和 K 是否有稳定的失败阈值？扩展到非静态、未标定位姿、复杂介质是否需要显式新成像模型？这些在本文均未验证。后续研究笔记可独立讨论，不修改本报告的事实边界。

<!-- PTH:END SYNTHESIS-02 -->



---

# 第二部分：学习与研究笔记

# MVSplat 学习与研究笔记

> 精读报告 SHA-256：0bfd74516ca40d8c80a98ba7571ec5249ffb497e2920cb4f1c0756f5ca410ab6
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## 一、概念内化：观测、候选与三维表示

### 1. 匹配是几何证据，而非深度本身

原文 §3.1/Eq.(2–4)，精读报告“数学精读二、三”：已知内外参时，一个参考像素对应空间射线；给它假设深度，就能确定另一个视角中的候选匹配位置。代价体的核心不是“有一个三维张量”，而是每个元素都带有一个可反驳的几何假设。直接预测深度 $z=h(F)$ 与 $z=\sum_m d_m\operatorname{softmax}_m(C)$ 的差别在于：后者把候选对应关系显式交给网络，前者则需从特征中隐式学习它。二者都可能学习先验；后者没有消灭学习误差。

补充推导：平行相机、已知基线 $b$ 和焦距 $f$ 时，视差 $\delta=fb/z$。测量噪声标准差 $\sigma_\delta$ 较小、无偏且真对应存在时，由一阶误差传播 $\sigma_z\approx z^2\sigma_\delta/(fb)$。例如 $f=100$ px、$b=0.2$ m、$\sigma_\delta=0.5$ px，$z=2$ m 时约 0.1 m，$z=10$ m 时约 2.5 m；后者线性近似已经较粗，不能看作真实误差分布。即使网络能完美提取局部特征，有限基线也会限制远距离的深度约束。如果两幅图仅纯旋转，几何对应完全不携带轴向深度信息；更大的模型可以学习深度先验，却无法创造平移视差。Fig.3 的远景案例与这个风险相符，但未独立测量基线。

**可观测性与可辨识性**：前者是改变状态会否改变观测，后者是是否能从观测唯一确定状态。设渲染函数 $I=R(\Theta)$，局部线性化 $\Delta I\approx J\Delta\Theta$。若 $J$ 存在零空间方向，沿该方向修改位置/尺度/颜色可几乎保持 RGB；即便每个参数都对图像有影响，也可能因为相互补偿而不唯一。条件匹配的显式几何证据可减少可行解，却不能保证 $J$ 满秩。Figure 4/D 中 NVS 较好而基线仍有浮点正是视觉与几何不完全一致的具体例证。

### 2. 代价体的概率解释不能越过证据

Eq.(6) 的 $p_m$ 是 softmax 归一化后的匹配分数；不是由可验证噪声模型推导的真实后验概率。若 score 等于 $\log P(I\mid d)+\log P(d)$ 加与 d 无关常数，则 softmax 才等于离散 Bayesian posterior；本论文没有建立该等价。最大概率作为 opacity 的输入不是经过校准的深度置信度。可以有同样最大值但误差风险截然不同的分布：构造 $d=(1,2,10)$，$p_A=(0.8,0.2,0)$、$p_B=(0.8,0,0.2)$（零概率是极限），均有 $q=0.8$，但 $E_A[d]=1.2$、$E_B[d]=2.8$，方差分别 0.16 与 12.96；真实深度若是1，期望误差也不同。熵相同也不能区分候选间物理距离。相反，重复纹理会让错误匹配尖峰非常锐利。

若希望在下游决策使用不确定性，一个备选是同时保留期望、方差、多峰质量与源视图一致性，使用真值误差对可靠性信号做校准（拓展假设）。但是没有 GT depth 的训练场景可能缺校准标签，需独立合成或可测几何验证；可见性未知时，置信度也可来自网络幻觉。不要把高斯协方差当作深度后验协方差：$\Sigma$ 描述渲染原语在空间中的形状，$\sigma_z^2$ 描述候选分布的不确定度，二者量纲与用途虽可能相关却不相同。把沿射线方差传播到中心不确定性 $\operatorname{Cov}(\mu)=r r^T\sigma_z^2$ 需要已知相机且深度确实为随机变量；这只是位置不确定度，不直接决定光栅化尺度。

### 3. 多视图聚合：信息量不必单调增加

原文 Eq.(3–4) 及 more-view 实验：对其他视图的相关性取均值，得到固定 D 通道，能接受更多视图。设真实候选和错误候选的分数差 $\Delta_j=s_j(d^*)-s_j(d')$。聚合后 $\bar\Delta=(K-1)^{-1}\sum_j\Delta_j$。当新视图差值为正且与已有独立时，有机会提高辨别稳健性；但若已有两个差为 0.5、0.5，新增遮挡视图差为 −2，则均值从 0.5 变成 −1/3，排序反转。简单均值不显式处理可靠性和可见性，不保证观测越多越正确。可以设想 $\sum_j w_j\Delta_j/\sum_j w_j$，$w_j\ge0$，条件是权重必须能区分几何证据与伪一致性，不能单凭 RGB 拟合验证。若所有视图都缺有效对应，加权并不能产生证据。DTU 的 2→3 结果只是一个经验点，不能代替上述条件。

### 4. 两级精炼与显式表示的边界

精读报告 METHOD-03/04 与 Table3：代价体精炼改变每个候选的 logits，深度精炼只对最终标量进行残差修改。前者可改变峰形、多峰和置信度；后者可突破离散候选的凸包但不保证正确。若先把双峰折成一个深度，后级只能依赖附带图像/特征和上下文来恢复失去的分布信息。因此保留候选分布、多高斯或迭代更新是一种备选，但会增加计算、表示自由度与非辨识性；Table C 说明增加原语虽改善图像指标，却没有证明多峰几何更真实。

显式高斯可用于不同目标视角，不代表它是可解释的网格、几何真值或独立物理介质模型。输出位置是条件网络推断，训练监督是图像误差；较好的图像可以由不准确的三维参数组合合成。若以机器人测距或几何测绘为目标，需要直接几何评估，而不是仅选择最高 PSNR 模型。此处所有例子和推导为内化拓展，并非论文作者新增实验或结论。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Problem & Task — Sparse-View Feed-Forward Reconstruction
- **Keywords:** Sparse-View Reconstruction; Novel View Synthesis; Feed-Forward 3DGS; Posed Images
- **Highlight:** 将带已知相机的稀疏图像直接映射为可复用的像素对齐高斯；训练跨场景，推理不逐场景优化。
- **Evidence:** §1、§3 Eq.(1)、§3.3、Table 1。
- **Connection Points:** 输入视角数量、姿态前提、目标输出、训练监督、推理是否优化。
- **Open Questions:** 相同输入数量及相机条件下，前馈推断与优化的精度和计算边界如何变化？

### Topic: Geometry Inference — Plane-Sweep Cost Volume
- **Keywords:** Multi-View Stereo; Plane Sweep; Inverse Depth; Feature Correlation; Cost Volume
- **Highlight:** 按逆深度候选进行跨视图 warp，缩放点积构成每参考视角的成本体，显式几何匹配取代完全依赖特征回归的深度分支。
- **Evidence:** §3.1 Eq.(2–4)、Fig.2、Table 3/A。
- **Connection Points:** 候选空间、特征匹配、相机条件、空间分辨率、源视图聚合。
- **Open Questions:** 在遮挡、位姿误差和视角外观变化下，候选响应与真实深度的关系是否稳定？

### Topic: Geometry Inference — Distribution and Depth Refinement
- **Keywords:** Residual Cost Refinement; Softmax Depth Regression; Depth Residual; Cross-View Attention
- **Highlight:** 先修正候选分数，再由期望回归深度，另一个轻量网络修正最终深度标量；单视图可见区域需上下文补充。
- **Evidence:** §3.1 Eq.(5–6)、§4.3、Fig.6、Table 3。
- **Connection Points:** 分布保留或压缩、可见性处理、空间先验与观测证据、深度边界。
- **Open Questions:** 精炼是消除错误匹配还是用数据先验生成更易渲染的表面？

### Topic: Scene Representation — Pixel-Aligned Gaussian Centers
- **Keywords:** Pixel-Aligned Gaussians; Unprojection; Deterministic Union; Spherical Harmonics
- **Highlight:** 深度反投影决定中心，opacity 来自最大匹配概率的学习映射，其余参数由特征、代价体及 RGB 共同预测；直接组合多视图高斯。
- **Evidence:** §3.2、Fig.2/4、Table C。
- **Connection Points:** 中心估计、重叠原语处理、高斯密度与表达能力、几何和外观解耦。
- **Open Questions:** 对应位置一致性及原语冗余能否在保持图像质量时减少几何误差？

### Topic: Training & Supervision — Photometric-Only Geometry Learning
- **Keywords:** Photometric Supervision; LPIPS; Differentiable Rasterization; Geometric Inductive Bias
- **Highlight:** 仅目标 RGB 的 L2 与 LPIPS 监督端到端训练；无需额外深度正则微调即可在可视化中减少漂浮高斯，但没有真值几何误差统计。
- **Evidence:** §3.3、§4.2、Fig.4/D。
- **Connection Points:** 深度监督、跨场景训练、渲染梯度路径、几何与图像评价。
- **Open Questions:** 光度监督下哪些几何自由度可由显式匹配稳定约束？

### Topic: Uncertainty — Matching Confidence vs Reliability
- **Keywords:** Matching Confidence; Softmax; Opacity; Calibration; Multimodal Depth
- **Highlight:** 最大候选概率用于预测不透明度；没有校准实验，概率尖锐程度不能直接代表几何误差。
- **Evidence:** §3.1 Eq.(6)、§3.2；精读数学四、五。
- **Connection Points:** 置信度定义、分布矩、错误峰、空间形状与位置不确定性。
- **Open Questions:** 哪些置信度指标在域移位和遮挡中仍能预测深度错误？

### Topic: Generalization — Domain and View Count
- **Keywords:** Cross-Dataset Generalization; Variable View Count; Feature Distribution; Geometric Matching
- **Highlight:** RE10K 训练模型在 ACID/DTU 的零样本结果超过所比较的 pixelSplat，DTU 三视图测试优于两视图，但未验证任意域或任意视角数。
- **Evidence:** §4.2、Table 2、Fig.5/E。
- **Connection Points:** 域差、训练量、输入相机分布、视图数聚合、跨域绝对误差。
- **Open Questions:** 可否分离几何归纳偏置与训练量、预训练和域距离的贡献？

### Topic: Efficiency & Scalability — Cost of Explicit Geometry
- **Keywords:** 2D U-Net; Swin Transformer; Gaussian Count; Rendering Latency; Inverse Depth Sampling
- **Highlight:** D 作为二维卷积通道，轻量 encoder 与默认每像素一高斯共同降低端到端耗时；增加 Gaussian 数带来画质/渲染时间取舍。
- **Evidence:** §3.1、§4.2、Table 1/B/C。
- **Connection Points:** 分辨率、深度候选数、视图数量、参数量、显存与延迟。
- **Open Questions:** 如何按区域或观测不确定性分配候选和原语预算？

### Topic: Technical Limitations — Appearance Consistency
- **Keywords:** Non-Lambertian Surfaces; Specular Reflection; Occlusion; Feature Matching; BRDF
- **Highlight:** 单视图可见区域的匹配证据不足，反射场景出现失败；建议 BRDF 与多样训练数据仍是未来方向。
- **Evidence:** §4.3、Fig.6/B、Appendix A。
- **Connection Points:** 视角相关外观、可见性、成像模型、几何错配与外观错配分离。
- **Open Questions:** 正确投影下的外观不一致如何与错误几何区分？


## 三、研究方向：稀疏观测与成像条件

本节以稀疏观测三维视觉为一般主线，以水下视觉作为可公开讨论的拓展示例，不预设具体项目、方法或原创性。原论文没有水下实验。以下不属于作者提出的水下算法。

| 层级 | 本论文 | 可拓展方向 |
|---|---|---|
| Field | 三维视觉 / 图形学 | Sparse-Observation Underwater Vision |
| Task | 场景级前馈三维重建、新视角合成 | 几何/深度/外观/介质推断或 NVS；须按可用真值选择 |
| Task Setting | 两个已知相机 RGB 输入，静态普通场景，跨场景 RGB 监督 | 少量水下视图；已知或带误差位姿；水体、照明、视距可变 |
| Research Problem | 如何在稀疏观测下以显式多视图对应可靠预测中心并高效渲染 | 稀疏且外观随传播路径变化时，哪些观测仍能约束几何、如何避免先验掩盖错误？ |
| Method | Plane sweep/cost volume + 2D refinement + pixel-aligned 3DGS | 待研究：物理感知特征、可靠性融合、条件先验或受控迭代；不预设必须用 3DGS |

### 直接联系与关键障碍

MVSplat 的方法可迁移性不取决于“水下也有多视图”这一表面类比，而在于能否保持投影几何与匹配证据之间的关系。原论文 Fig.6 显示共同可见区域比单视图区域有更好的匹配约束，Fig.B 显示正确投影也可能因反射外观变化而失败。水下成像中即便理想漫反射表面，在不同相机下由于传播距离变化也可能不满足跨图亮度一致性。

**补充建模（简化均匀介质，并非原文模型）**。对一个静态物理点 $X$，第 i 个相机观察的某颜色通道可写

$$I_i=J(X)e^{-\beta t_i}+B(1-e^{-\gamma t_i}),$$

$J(X)$ 是在固定照明下假设不依观察方向变化的清晰辐射，$t_i$ 为相机到点的路径长度，$\beta,\gamma\ge0$ 为简化的直达衰减及背散射有效系数，$B$ 为背景项。实际水体可能空间变化、波长依赖、光照不均、存在前向散射及折射，该简化式不是普遍正确的水下模型。若正确匹配同一 X，且 $t_i\ne t_j$，通常仍有 $I_i\ne I_j$；例如 $J=0.8,B=0.2,\beta=\gamma=0.3$，$t_i=1,t_j=4$ 时约 0.644 与 0.381。原始颜色差约0.263并非几何错配。对于 learned features，能否抵消差异仍是实证问题，不能由这个 RGB 例子直接推出网络一定失败。

若已知 $B,\beta,\gamma,t_i$，可逆地恢复 $J_i=[I_i-B(1-e^{-\gamma t_i})]e^{\beta t_i}$。但路径长度依赖候选几何；错误深度会改变补偿，且噪声 $\epsilon$ 放大为 $\epsilon e^{\beta t_i}$。如果未知介质参数，则同一 $I_i$ 可由不同 $J,t_i,\beta,B$ 组合解释，稀疏视图进一步降低可辨识性。举例在不含背散射 $I=Je^{-\beta t}$ 的简化条件，固定观测 I，对任意 $\beta t$ 都有 $J=I e^{\beta t}$（受物理辐射范围限制）。所以候选几何引导的成像补偿也不能脱离参数不确定性。

### 观测与学习先验如何协调

可将输入视角分为共同可见、有部分遮挡、单视图独见和完全不可见区域。第一类在位姿准确、足够基线、特征一致时具有直接几何约束；第二类需判断哪一源可靠；第三类缺乏三角化约束而依赖先验；第四类最多在统计意义上生成合理内容。介质使第一类也可能出现弱证据。与其把所有区域都施加相同一致性，不如明确证据质量与推断边界，区分“观测所支持”与“由训练分布补全”。模型不确定度可作为决策信号，但须校准、不能等同 opacity 或高斯协方差。

可迁移的是候选几何显式化、匹配可信度建模、可变视图聚合和分级先验，而非原论文训练权重必定适用。候选机制在未知折射/相机标定时需先修正投影；强衰减可能使结构不可观测，任何特征增强都不能恢复不存在的光子证据；域移位也可能使图像先验生成与真实环境不同的纹理和结构。应优先明确真实任务、观测协议和可证伪指标，再选择前馈、优化或混合方法。水下仅是此原理的一个重要例子，尚需独立文献与数据验证。


## 四、可检验研究问题候选

以下从本论文现象提出研究假设，不表示此前无人研究，不代表已完成其他论文调研。优先级依据是科学矛盾、可控实验与证据可获得性，而非臆测新颖性。先验证机制是否成立，再决定技术路线。各候选可单独研究，不要求合并为一个大模型。

### RP-A：成像差异与几何错配是否可区分？

**可观察现象与科学未知。** Fig.B 的反射失败、Fig.6 的单视图内容错误以及 §4.2 的跨域剩余误差均表明匹配不可靠可以由多种因素引起。对于稀疏水下观测，正确投影位置仍可能因距离相关衰减和背散射具有不同外观。未知的是：几何一致性线索在何种介质、基线和信噪比条件下仍可用？其失效是匹配特征本身不变性不足，还是几何信息已经不可观测？原文只验证普通场景，不回答该问题。

**假设与反证。** H1：在有共同可见、已知位姿、可辨纹理且非极端衰减的条件下，使用候选路径长度和可估计成像差异的匹配分数，能提高真实候选相对错误候选的可分性。若在同一几何、基线、图像噪声和训练数据下只改善图像画质，真实候选排序与深度误差不改善，或错误候选也因补偿获得更高分，则不能支持 H1。H2（更强）：物理参数未知时仍有可用可靠性线索；不能从 H1 自动推出。

**验证。** 先构建已知几何/位姿/清晰辐射的配对图：同一对相机、同一表面及光照，独立控制介质强度、传播差、背散射与测量噪声；对相同真值候选网格记录 $C(d^*)-\max_{|d-d^*|>\epsilon}C(d)$、真值候选 Top-k 率、实际深度 abs-rel/RMSE、遮挡分层错误率。候选的 d 必须按同一深度定义，真值不在候选格时取最近层并单独统计量化误差。基线：原始缩放点积、归一化特征、普通图像增强再匹配、oracle 已知介质补偿和实际估计补偿；固定特征容量和训练量，另做相同可见性掩码比较。oracle 仅界定潜在收益，不视为部署结果。用若干随机种子及按场景 bootstrap 区间报告稳定性。

**判据与限制。** 候选可分性和真实深度必须同时改善，介质增大后失效曲线可解释且不以严重图像退化换取，才有实际证据。若 oracle 也无法改善，说明可能是信号缺失、采样/几何或光照问题；若 oracle 有效而估计补偿失败，需研究介质可辨识性而非盲目增大网络。合成到真实的差距、色域校准、折射与不均匀光照须另测，不能以简单均匀介质式证明一般结论。优先级：高，因果变量容易在合成中独立控制；尚需核查水下 MVS、成像补偿与物理感知特征文献。

### RP-B：稀疏推断的可靠性是否能被校准？

**现象与科学未知。** §3.2 以最大 softmax 概率预测 opacity，而没有 GT 深度验证；Fig.4/D 的浮点说明图像指标无法完全反映三维误差。关键未知：在错误尖峰、双峰、单视图可见和域移位时，哪些不确定性信号能可靠区分可观测与先验补全？不是简单增加一个 confidence head。

**假设与机制。** H：组合候选分布的物理深度方差、源视图间分歧、可见性和几何投影敏感度，能比单独 $q=\max p$ 更可靠地预测大深度误差。可考虑固定阈值 $\tau$ 的事件 $Y=\mathbb 1(|\hat d-d^*|>\tau)$，估计风险 $\hat r=P(Y=1\mid \text{features})$。校准定义是给定预测风险 r 的样本组实际错误率接近 r，而非要求所有风险都小。

**验证。** 使用带真实几何的同域及留出域，独立划分训练、校准和测试场景，固定深度预测网络后比较 max-softmax、熵、候选方差、跨视角方差、组合模型和 oracle 可见性。按近远、纹理、反射、遮挡、视图数和位姿误差分层；报告 AUROC/PR（排序）、Brier/ECE（校准，注明 bins）、风险—覆盖曲线（拒绝高风险时剩余深度误差），同时报告覆盖率和 NVS 指标。以相同覆盖率比较误差，避免全拒绝的假成功；并检验模型在未见介质/相机分布的校准漂移。若调整深度网络也改变误差分布，需单独控制重训因素。

**成功与失败判据。** 新指标在留出条件下以相同覆盖率降低风险且校准改善，才支持该假设；同域有效、跨域失效或只在平均 ECE 上改善但高风险尾部恶化，则只能限定结论。模型的认知/观测噪声来源可能并不能从有限观测分离，未得到可用真值时只能报告代理风险。优先级：高至中；它能为后续自适应融合提供前置评价标准，但需真实/合成几何可靠。尚需核查 MVS 深度置信度和域外校准研究。

### RP-C：更多稀疏视角何时真正增加可辨识信息？

**现象。** §4.2 DTU 2→3 有改善；Eq.(3–4) 的等权平均对遮挡/域变敏感；Fig.6 显示单图内容需先验。科学问题不是单纯寻找最优视图数，而是判断额外观测的有效几何信息何时大于其带来的冲突、噪声和计算代价。

**假设与验证。** H：按候选深度和可见性评估源视图的贡献，比无条件平均更稳健；但错误的学习权重也可能压制罕见真实结构。构造固定参考和真实几何，将同一场景中新增视角按基线、重叠率、成像一致性、相机误差分组，逐步加到 K=2、3、4、6 等（按数据实际可用视图数），比较等权平均、基线/可见性启发权、学习权重、oracle 有效源选择及去掉匹配的同容量模型。测试时固定原模型及输入顺序实验以区分训练视图数分布变化；另做等训练预算可变 K 训练。测真实深度误差、几何覆盖、源视图排序正确率、显存与端到端耗时；按共同可见点与单视图点分层。与仅增加输入图数量但不增加平移基线的控制组比较，检验是否真正增加信息。

**判据。** 新增有效视图时改善深度，新增遮挡/冲突视图时至少不显著恶化，相同计算预算或明确增加代价下优于基线；若仅 RGB 变化而几何不改善、权重把有效视角排除、或显存成本不可接受，H 不成立或仅适用于局部条件。权重本身需可验证证据，不能由高权重反推真实可见性。优先级：中，依赖前两个问题的证据分离和可靠性评估；不默认加入主动采集或迭代优化。后续需要独立调研鲁棒多视图聚合和视角选择。

### 候选间关系

RP-A 检查外观与几何证据的根本冲突，RP-B 定量评价推断可靠性，RP-C 使用可靠性信号处理多观测。它们不是一项工作必须同时完成的三模块；可先用小规模受控实验判断关键矛盾，再决定任务是深度、恢复、介质推断还是 NVS。没有科学效果时应接受反证，不将方法复杂度或论文数量当成果。


## 五、知识内化与理解自检

### 用自己的语言完整解释论文

MVSplat 先解决“稀疏已知位姿图像怎样提供三维定位依据”，再解决“怎样用三维原语表达可渲染场景”。参考像素确定一条射线，网络在射线上选取许多可能的深度；通过投影到其他视角判断哪些候选在特征上更一致。对每一张输入图都这样构造代价体，因此能获得各图自己的深度。共同可见区域有显式对应，单视图可见、无纹理和反射区域则只有弱证据或错误证据，网络用上下文残差修正候选分数。softmax 后求真实深度候选期望，再精炼深度。将像素和深度反投影到公共坐标就是中心，和 opacity、协方差、SH 颜色组成 Gaussian；多个输入视图的原语直接合并。训练时渲染未作为输入的目标视角，与目标 RGB 比较，反传到所有分支；推理则一次编码再重复渲染，不需要新场景优化。

这不是“先用真实 MVS 深度监督，再拟合高斯”，也不是“只把两个输入图投影重叠”：深度的匹配表示参与端到端渲染学习。选择较轻的 CNN、局部注意力及二维 U-Net，在代价体中把候选深度作为通道；一高斯/像素控制原语数。Table 3 删除代价体的 PSNR 降幅远大于额外深度精炼增益；Table A 在另一架构中替换深度分支依然改善图像，支持核心机制。Figure 4/D 定性展示少漂浮与平滑深度，但不能量化准确度。Figure B 说明同一表面在不同视角出现不一致外观时仍失败。Table 2 与多视图实验支持有限迁移，而不是通用可靠性。

### 技术和概念地图

| 层次 | 关键知识 | 证据与下游连接 |
|---|---|---|
| 观测 | 带位姿稀疏图像、深度候选、可见性 | Eq.(1–2)，把投影模型显式引入 |
| 特征 | 1/4 网格 CNN、自/跨视角注意力 | §3.1，给匹配提供可比较描述 |
| 几何 | Warp → 点积 → 多源均值 → D 通道 | Eq.(2–4)，明确搜索空间 |
| 学习 | 候选残差、上采样、softmax 深度、深度残差 | Eq.(5–6)，观测与上下文融合 |
| 表示 | 反投影中心、置信度→opacity、协方差/SH | §3.2，渲染参数化 |
| 优化 | 可微渲染目标 RGB L2+LPIPS | §3.3，不依赖 GT depth |
| 评价 | 图像质量、计时、参数、定性几何、域外泛化 | §4、Table1–3/A–D，不能替代几何真值 |
| 研究线索 | 歧义、可信度、非一致外观、先验补全 | Fig.6/B，需受控验证 |

### 易错理解自检

1. 如果输入两张图之间只有旋转，为什么 cost volume 可能无法确定深度？因为在理想针孔模型中旋转的齐次尺度抵消，候选深度不会改变对应位置。网络仍可借助单目统计，但不属于双目三角化证据。
2. 为什么称成本体是“成本”却用 softmax 奖励大值？这里存储的是相似度分数，不是待最小化距离；较大匹配分数概率更高。缩放点积不等于余弦相似度。
3. 为什么深度期望不等于概率最大的候选？期望由所有候选加权，双峰时甚至落在空区域；后续深度精炼没有唯一恢复保证。
4. 为什么两级 U-Net 不能互换？前级修正候选分布，影响 confidence 与深度；后级直接修改深度标量，不能反向重建已压缩分布，除非另有特征支持。
5. opacity 是否等于匹配置信度？否，两个卷积层学习预测，opacity 是混合权重参数；置信度不是校准几何正确率。
6. 能否把 0.0015s 的倒数作为整个模型的实时推理速度？不能，它是已有高斯的 renderer；完整约 0.044s，编码主导。
7. 能否把三视图 DTU 优势推广到任意数量或单视图？不能，K=1 无匹配，K>3 计算量与遮挡影响未系统评估。
8. 为什么没有 GT depth 也不能称无监督？因为有目标 RGB 监督和已估计相机，训练需要跨场景数据。
9. 能否以较少漂浮、较高 PSNR 证明厘米级几何？不能，需要尺度对齐、真值深度/点云和误差统计。
10. 水下迁移最重要的检查是什么？正确几何候选在传播改变后是否仍能被识别，并能否区分图像补偿收益与真实几何改善；本文没有验证水下。

尚未解决：不同距离与基线下的候选覆盖、真实深度误差与匹配置信度关系、学习先验在弱观测区域的失效、反射和复杂介质的成像一致性。若读者能独立从已知相机推出 warp、从相关性解释深度回归、从渲染损失解释监督路径并指出这些限制，就抓住了本文核心，而不必记住每个数值。


## 六、术语表

| English | 中文 | 技术含义及联系 |
|---|---|---|
| Sparse-view | 稀疏视角 | 输入相机数量少，不指稀疏像素或稀疏体素 |
| Posed image | 带位姿图像 | 具有相机内外参的图像，本文是输入前提，非预测位姿 |
| Feed-forward | 前馈 | 跨场景训练后一次预测，不等于不用训练或不使用优化器 |
| Novel view synthesis (NVS) | 新视角合成 | 给定目标相机生成未作为输入的图像；质量不等同几何精度 |
| Multi-view stereo (MVS) | 多视角立体视觉 | 通过相机投影和对应约束深度，多数需要共同可见 |
| Plane sweep | 平面扫掠 | 对候选深度平面进行跨相机投影，本文 Eq.(2) |
| Inverse depth | 逆深度 | $\rho=1/d$，均匀采样意味着近处候选更密，非均匀实际深度 |
| Disparity | 视差 | 平行双目中 $fb/z$，与逆深度成正比；一般非校正视图非仅水平位移 |
| Warping | 特征变换/重采样 | 依据候选几何把源网格特征采到参考位置，区别反投影 |
| Cost volume | 代价体 | 每像素各深度候选的匹配值集合；本文是相似度大为好 |
| Scaled dot product | 缩放点积 | $f^Tg/\sqrt C$，非显式余弦归一化；Eq.(3) |
| Context view | 上下文视角 | 提供模型输入的图像；target view 提供训练监督/评估 |
| Cross-view attention | 跨视角注意力 | 参考特征与其他视角特征交互；区别明确的候选几何 warp |
| Self-attention | 自注意力 | 当前 token 集合内部交互；附录的 flatten 可含不同视图信息 |
| Swin Transformer | 移位窗口 Transformer | 局部窗口注意力架构；本文避免显式极线采样的额外开销 |
| Epipolar line | 极线 | 参考像素射线在另一图的投影轨迹，不直接给出真实深度 |
| U-Net | U形编解码网络 | 多尺度空间上下文与细节，本文两级残差任务不同 |
| Residual cost refinement | 代价体残差精炼 | 在初始候选分数上加预测残差；Eq.(5) |
| Softmax depth regression | Softmax 深度回归 | 对候选概率求实际深度期望；Eq.(6) |
| Depth refinement | 深度精炼 | 在已估计深度上加标量残差，不是再做一次成本体 |
| Unprojection | 反投影 | 像素和深度转成三维点；需要相机定义与尺度 |
| Pixel-aligned Gaussian | 像素对齐高斯 | 每输入像素对应预测原语，默认一像素一个，不是屏幕空间高斯固定不动 |
| Gaussian center | 高斯中心 | 三维位置 $\mu$，本文来自深度反投影 |
| Covariance | 协方差 | $\Sigma$ 描述空间原语形状、尺度及方向，不代表中心预测不确定度 |
| Opacity | 不透明度 | 渲染合成参数 $\alpha$，由匹配置信度输入卷积头预测，不等于置信度 |
| Spherical harmonics (SH) | 球谐 | 方向相关颜色基函数；不是显式物理 BRDF |
| Matching confidence | 匹配置信度 | 本文取候选 softmax 最大值；未证明校准 |
| Gaussian rasterization | 高斯栅格化 | 三维原语投影为屏幕椭圆并混合，已有高斯可快渲染 |
| Photometric supervision | 光度监督 | 渲染目标 RGB 与观测 RGB 对照，本文 L2 加 LPIPS |
| Floater | 漂浮伪影 | 空间位置可能不符合真实表面的高斯团；定性观察非几何指标 |
| Zero-shot cross-dataset | 跨数据集零样本 | 在源域训练、无需目标域再训练的直接测试 |
| Observability | 可观测性 | 状态变化是否在观测中留下可检测信息 |
| Identifiability | 可辨识性 | 在模型条件下不同参数是否可由观测唯一分离 |
| Calibration | 校准 | 预测可靠性与实测错误频率的对应；论文未验证 |
| BRDF | 双向反射分布函数 | 描述入射/出射方向相关反射；作者未来方向，本文未显式建模 |
| PSNR | 峰值信噪比 | 同动态范围下由 MSE 决定，越高越好 |
| SSIM | 结构相似性 | 图像局部结构/亮度等相似度，越高越好 |
| LPIPS | 学习感知图像距离 | 深层感知特征差，越低越好，不是三维误差 |

公式、模块细节及符号约定见冻结精读报告的 METHOD 和 MATH 单元；可观测性、校准等术语在本笔记用于研究拓展，不是声称论文已测量。


## 七、来源、证据与版本

- **论文：** MVSplat: Efficient 3D Gaussian Splatting from Sparse Multi-View Images，arXiv:2403.14627v2，2024-07-18，ECCV 2024。PDF 共23页，实际核查正文、附录及参考文献。[arXiv](https://arxiv.org/abs/2403.14627) · [指定版本 PDF](https://arxiv.org/pdf/2403.14627v2) · [Project Page](https://donydchen.github.io/mvsplat/) · [Code](https://github.com/donydchen/mvsplat)。PDF SHA-256 `d1becb5361d7d8aa6f2e47d1e9e257031120b4207834f5effca049a8df7e4bb9`。实时最新版未独立确认，Source 标题作者相符但无独立版本印记。
- **冻结精读报告 SHA-256：** `0bfd74516ca40d8c80a98ba7571ec5249ffb497e2920cb4f1c0756f5ca410ab6`。全部44单元、11图7表及公式1–6实际审读，三处定点纠正后冻结。原文题注/表格冲突保留，使用主表数值的口径注明。
- **证据分类：** 本文实验事实以同版本 PDF 和冻结报告定位为准；补充数学例子、误差传播、贝叶斯条件和水下成像模型为背景或推导；三个研究问题属于待检验假设，未证明新颖性及可行效果。
- **跨文献分析延期：** 当前只提取9个潜在关联主题索引，未独立下载或阅读其他论文，不能把作者 Related Work 描述当独立核查。后续比较应读取各自论文原件、统一任务及实验协议。
- **未证实项：** 真值几何精度、匹配置信度校准、姿态误差、可见性控制、更多视角、复杂介质和非朗伯反射的系统性能。本文没有水下实验；实际水下参数、折射、照明以及真实几何评估须另证。研究笔记只纳入可公开讨论的一般技术分析，不包含私人研究计划或未公开实验。
- **版本维护：** 当前笔记绑定上述冻结报告，初次独立封存后后续学习讨论可追加；不改冻结报告，不覆盖已封存版本。仅在明确授权的重新发布时纳入新版本。本文原件及图像保真验收与客户端显示是不同问题。



