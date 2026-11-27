---
title: "VGGT-Diff: Visual Geometry Meets Diffusion for Sparse-View Novel View Synthesis（完整精读与研究笔记）"
date: 2026-11-27 04:29:30
index_img: "/VGGT-Diff/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/VGGT-Diff/)

# 第一部分：论文精读报告

# VGGT-Diff 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2609.33253v2（2026-10-14）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2609.33253) | [PDF](https://arxiv.org/pdf/2609.33253v2) | [Project Page](https://chenkangjie1123.github.io/VGGT-Diff/) | [Code](https://github.com/chenkangjie1123/VGGT-Diff)

<!-- PAPER-METADATA END -->

> 论文：VGGT-Diff: Visual Geometry Meets Diffusion for Sparse-View Novel View Synthesis（PDF 首页题名无 VGGT-Diff 前缀）
> 版本：arXiv:2609.33253v2，用户上传资源包。正文、参考文献和附录共24页。PDF权威，Source与HTML辅助。
> 原论文Figure为PDF区域保真提取；JPEG仅用于阅读展示。
> 原文事实、背景知识、合理推断、独立评价、尚未证实分别标明。

<!-- PTH:BEGIN INIT -->
## 0. 论文与资源

【原文事实】本文是视觉几何与扩散融合的稀疏视角 NVS 预印本，arXiv v2（2026-10-14）。官方题名含 VGGT-Diff 前缀，PDF 首页及 TeX 标题为 Visual Geometry Meets Diffusion for Sparse-View Novel View Synthesis。作者：Kangjie Chen, Xiangyu Li, Dongbin Zhang, Chaoda Zheng, Shijia Chen, Jinhao Deng, Hongbin Lin, Choo Sin Wai, Minqi Wang, Minghao Yang, Dake Zhong, Guorui Song, Yu Zhang, Xianming Liu, Boyang Wang。机构：XPeng Motors、The Chinese University of Hong Kong、Tsinghua University；前两位共同贡献，末位通讯作者（p.1）。

### 结构

| 内容 | PDF页 | 阅读重点 |
|---|---|---|
| Abstract / Introduction | 1–2 | 几何约束和生成补全的矛盾 |
| Related Work | 2–3 | 回归/生成/几何引导 |
| Method §3.1–3.4 | 3–6 | 联合扩散、几何路由、PTRC、训练推理 |
| Experiments §4.1–4.3 | 6–10 | 主结果、分层姿态、几何一致性、消融 |
| Conclusion / References | 10–14 | 结论、引文 |
| A.1–A.2 | 15–17 | 采样、gauge、姿态和超参数 |
| A.3–A.5 | 17–20 | 规模、联合生成、特征深度 |
| A.6–A.7 | 20–21 | 完整难度表与点云 |
| A.8–A.9 | 22–23 | 残差目标、优化与统计 |
| A.10–A.11 | 23–24 | 少视图、VAE 协议 |
| A.12 | 24 | 未来方向 |

### 图表目录

| 对象 | PDF页 | 范围 |
|---|---:|---|
| Figure 1 | 1 | 正文 |
| Figure 2 | 4 | 正文 |
| Figure 3 | 5 | 正文 |
| Figure 4 | 8 | 正文 |
| Figure 5 | 9 | 正文 |
| Figure 6 | 16 | 附录 |
| Figure 7 | 18 | 附录 |
| Figure 8 | 18 | 附录 |
| Figure 9 | 19 | 附录 |
| Figure 10 | 21 | 附录 |
| Figure 11 | 24 | 附录 |
| Table 1 | 7 | 正文 |
| Table 2 | 8 | 正文 |
| Table 3 | 8 | 正文 |
| Table 4 | 9 | 正文 |
| Table 5 | 17 | 附录 |
| Table 6 | 18 | 附录 |
| Table 7 | 19 | 附录 |
| Table 8 | 19 | 附录 |
| Table 9 | 20 | 附录 |
| Table 10 | 21 | 附录 |
| Table 11 | 22 | 附录 |
| Table 12 | 22 | 附录 |
| Table 13 | 23 | 附录 |

**证据边界。** 24页均读取并渲染；11 图、13 表，无独立编号 Algorithm。34 个原始文件哈希逐项一致。HTML 缺数学字体，Source 无独立版本印记，题名和作者匹配；PDF v2 明确印记为最终证据。图片 5 幅精确通过、6 幅实用带记录通过；JPEG 不是原件。初始化单元不涉及数学推导或方法机制。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1.1 标题

**译名：VGGT-Diff：视觉几何与扩散融合的稀疏视角新视角合成。**

【原文事实】题名描述的任务是 novel-view synthesis (NVS)，而非由图片直接输出一组可编辑的高斯或准确的完整三维模型。sparse-view 表示有限来源相机；视觉几何在文中专指 VGGT-Ω 提供的含外观信息的特征与对应点、置信度。Diffusion 表示以视频扩散先验作条件生成，而不是对已知点云做确定性投影即可得到最终 RGB。

【背景知识】NVS 与一般图像生成的关键不同：同一场景身份及给定目标相机是约束。某个结果即使很像自然照片，如果窗、椅或文字的位置与指定相机看到的实际场景不符，仍然不能算正确合成。在稀疏观测下，背景被遮挡、反向表面未拍摄，成像信息无法唯一决定未知内容。

**证据边界。** 标题本身不能确定输入张数、是否给定相机、生成多个视角是否共享一个 3D 表示、模型大小及是否能还原真实未观测面。§3.1 明确是有位姿的输入与指定目标相机；其余需读方法和实验。这里不涉及公式推导。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 摘要

### 研究背景与问题

【原文事实】p.1 摘要把现有 NVS 的主要困难概括为重建范式与生成范式的互补弱点。前者能保存被看到的几何，但面对未知区域缺少信息；后者借助预训练视频模型学习真实图像分布，能填补空白，却通常把源图与目标像素的对应隐藏在网络内部。这里不是说回归模型不能输出未知像素，或者生成模型没有几何能力，而是它们给定稀疏观测时的错误类型不同。

【背景知识】设源图只看到桌子前沿，目标相机转到桌后。确定性系统要由有限约束估计新露出的后沿，可能把已观察的纹理涂到未知位置；生成模型可以构造合理后沿，但如果把原本存在的杯子移位，也违背已观测事实。仅用相机射线说明目标像素朝哪里看，无法自动告诉网络那条射线对应哪个源图局部。

### 技术路径

【原文事实】VGGT-Diff 将每个视觉 token 与三维点、置信度绑定，再通过 Visual Geometry Router (VGR) 将源特征投影到 query 相机对应的网格。hard anchor 给出尖锐近前表面条件；front/back 两层与路由统计只参与学习残差修正。未获得可靠支持的地方仍由扩散先验补全。此处三维点是证据传输媒介，不是完整重建输出。

多个 query slot 同时去噪，使目标视图能在 DiT 内交换信息；但共享注意力不是三维一致性的显式保证。Point-Track Residual Consistency (PTRC) 对同一可靠点的多视图预测误差作约束，不强制不同视角的真实颜色或潜变量一样。训练随机削弱几何，推理选择性增强 query 几何，目的在于不把不完善的几何当作无误的真实世界。其详细计算见 §3.2–3.4（pp.4–6）。

### 结果与结论

【原文事实】摘要声称在不同插值、外推难度具有竞争力或领先表现；并声称改善多视图稳定性。Table 1 是逐目标图像指标，Table 2 与 9/10 是姿态分层，Table 4/7 是联合视图重建代理，Table 3/11 检查模块作用。不能用单个 PSNR 代表所有这些主张。

**证据边界。** 这些是作者陈述及指向实验证据的摘要解析。生成合理内容不意味着真实未观测内容被确定性恢复；具体协议与量化边界后文展开。摘要单元不承担逐式推导。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 贡献

【原文事实】p.2 Introduction 明列三项贡献。

1. **Geometry-routed generation（几何路由生成）**。问题：有了目标射线仍不能建立显式源到目标对应。设计：用冻结 VGGT-Ω 的外观特征、预测三维位置与置信度投影；hard anchor 保留前表面，layered refinement 给出纠正线索，而非把恢复点云视作场景完整模型。原文 §3.2、Eq. (3)–(5)、Fig. 3。Table 3 比较无 Ω、point-rendered RGB 和无 layered refinement。创新是把视觉几何作为空间对齐的生成条件，并设计处理不确定可见性的路由，而非发明透视投影或 z-buffer。
2. **Geometry-grounded consistency（几何约束的一致性）**。问题：每张图单独优化和联合生成仍允许同一物点的残余错误漂移。设计：沿共同可见、通过 z-buffer 的轨迹对预测 clean latent 相对各自 GT 的残差做 robust matching，而不是直接强制速度/RGB相同。§3.3 Eq. (6)–(7)，A.8 Eq. (12) 对照，Table 3/11。创新在约束对象和几何轨迹使用，而非 Smooth-L1 自身。
3. **Geometry-condition regularization（几何条件正则）**。问题：稀疏源的几何存在错误，过强条件使扩散继承错误。训练对全局和密集条件用共享随机 scale，推理 target-only reference 做随步数衰减的几何 CFG。§3.4/A.2；Table 3/5。创新是针对几何条件的训练-推理配合，不是 CFG 基本公式本身。

三者不是互相独立的流水线：VGR 告诉网络何处能用观察证据；联合去噪/PTRC 约束各目标间如何分享错误修正；regularization 处理条件可能失真的情况。既有 Wan 视频扩散、VGGT-Ω、VAE、Plücker rays、flow matching 是基础组件。实验另提供多难度、新领域、双重建器、多目标与规模诊断，是经验性贡献。

**证据边界。** 不把整套网络收益归因于单个模块，也不宣称证明了所有相关研究中首次使用几何/扩散。表3无Ω同时移除 Ω 相关组件，不是纯 VGR 单变量；A.8 更直接检验残差形式。公式完整推导归后续数学单元。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 结论

【原文事实】§5 (p.10) 总结 VGR 建立 query-aligned 几何条件、PTRC 对齐可靠对应上的预测 clean 残差、随机训练时削弱和推理时选择性加强改善几何不完美时的表现。其最终目标是在保留观测场景结构的同时完成未知区域的合成。

【独立评价】作者的“cross-view coherence”应当按实验定义理解：同组生成视角经冻结重建器后与 GT 视角重建参考更接近（Table 4）；不是从传感器获得真值点云测量绝对精度。图像指标支持目标视角相似性而非全场景物理真实性。原文同时明确了预测几何并非完整 scene explanation。

【原文事实】A.12 (p.24) 提出五类未来方向：多层 VGGT-Ω 特征自适应融合；在不破坏几何先验情况下适配视觉编码器；更丰富的可见性分布、多遮挡层和可信度标定；更省内存的联合去噪及分块保持跨块一致；动态/非刚性内容的时空轨迹。它们不是本文已实现的能力。

**证据边界。** Table 1 v2 加入 scaled 模型，表中排名与主文描述早期模型的排名要分别理解。原文没有保证任意场景、任意来源相机或未知物体的真实性；也未将动态场景作为已验证范围。本结论单元不新增公式或推导。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2.1 图表功能总览

【原文事实】完整11图13表服务于不同问题。不能把视觉案例、总体平均和受控诊断混为同一组实验。

| 功能 | Figure | Table | 问题 |
|---|---|---|---|
| 全局概念 | 1 | 1, 2 | 是否兼顾目标 fidelity 和感知质量 |
| 机制 | 2, 3 | 3, 5, 8, 11, 12 | 如何融合几何与扩散，哪个条件/损失有效 |
| 单视图定性 | 4, 5, 11 | 1 | 细结构、目标相机对齐、跨域 |
| 姿态分层 | 6 | 2, 9, 10 | 近/中/远，插值与外推 |
| 数据/步数 | 7, 8 | 6 | 训练预算、损失和指标演化 |
| 特征层选择 | 9 | 8 | 中间层、冻结与联合优化 |
| 多视图结构 | 10 | 4, 7 | 重建器代理和 joint generation |
| 输入鲁棒性 | — | 13 | 六源训练后少源测试 |

【背景知识】PSNR 是像素误差的对数变换，SSIM 衡量局部结构相似，LPIPS 与 DreamSim 是学习型感知距离。数值方向前二者高优，后二者低优。Chamfer-L1 与 F-score 衡量点云参考匹配，前低后高。分层实验把总体平均隐藏的难度差异显露出来；消融用于在受控情况下隔离贡献；定性图仅展示选定案例。

**证据边界。** Table 1 LPIPS-AlexNet；192 个 DL3DV 受控诊断多数为 LPIPS-VGG；Table 10 Mip-NeRF 360 用 AlexNet。表4/7的重建指标只在同一协议下比较。scaled（100K+6K）与原 1K 检查点不等价。总览不展开公式推导。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
### Figure 1｜全局目标与效果

![Figure 1](VGGT-Diff/Figures/Figure_01.jpg)

*图源：PDF p.1，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 1: VGGT-Diff grounds a pretrained video diffusion prior with geometry-routed visual features for sparse-view novel-view synthesis. It maintains strong target-view fidelity across varying pose difficulties while better preserving source-observed details than diffusion-based baselines.

**中文翻译：** VGGT-Diff 用几何路由的视觉特征约束预训练视频扩散先验，实现稀疏视角新视角合成；在多种姿态难度下保持较强目标视角保真度，并比扩散基线更好保留源图观测细节。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】左上四个放大的花卉局部，依次对应下方目标 GT、VGGT-Diff、SEVA、FrameCrafter 的整图。花枝形状、花团位置、背景沙发边界和遮挡是主要观察点：VGGT-Diff较接近目标局部，两个基线可见形状与位置差异；不能仅凭某一朵花判断整图好坏。右侧圆形雷达由 PSNR、SSIM、LPIPS 三组分区，每组包含插值近中远、外推近中远；图例包括 scaled、1K、SEVA、FrameCrafter、LVSM。外圈距离是图形化比较而非直接同一单位的原始数值。

#### 技术机制与上下文

这是 Fig. 1 的概念结果图，左侧表现观测证据保存，右侧对应 Table 2、9、10 的难度分层及 v2 的 scaled 新行。I/E 是按轨迹帧索引划分的 interpolation/extrapolation；N/M/F 是姿态新颖度分位，二者不是同一概念。PSNR/SSIM高优，LPIPS低优；具体数字必须回到表格。

**证据边界。** 选取的花卉案例不能证明所有场景；雷达各指标量纲不同，不能用面积作统一性能量。scaled和1K不同训练预算。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
### Figure 2｜系统架构

![Figure 2](VGGT-Diff/Figures/Figure_02.jpg)

*图源：PDF p.4，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 2: Overview of VGGT-Diff. The Visual Geometry Conditioner uses VGGT-Ω points and confidence to route appearance-bearing source features into query-aligned conditions. The diffusion state, clean-source image condition, Plücker rays, and routed visual condition are concatenated channel-wise, embedded independently within each view, and jointly processed by a Wan-initialized multi-view DiT. Geometry conditioning is stochastically regularized during training, while flow matching and PTRC supervise target fidelity and cross-view consistency, respectively.

**中文翻译：** 视觉几何条件器使用 VGGT-Ω 的点和置信度，将含外观的源特征路由为目标对齐条件。扩散状态、干净源图条件、Plücker 射线及路由视觉条件沿通道拼接，在每视图内独立嵌入并由 Wan 初始化的多视图 DiT 联合处理。训练时随机正则几何条件，flow matching 与 PTRC 分别监督目标保真和跨视图一致性。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】图从左到右：源图及相机进入 Visual Geometry Conditioner（橙色 token），VAE Encoder 提供干净源 latent（蓝色）；源/目标射线经 PixelUnshuffle 变成紫色相机 map token。目标噪声状态以斜纹 token 表示。中心 channel-wise pixel-aligned concat 不是简单的序列末尾附加几何 token，而是在对应视图位置拼接条件，再 Conv3D Patch Embed。40个 DiT block 示出 timestep AdaLN、self-attention、FFN；linear/unpatchify 输出预测速度，经过迭代去噪和 VAE Decoder 生成目标 RGB。右侧 Training Target 框明确 flow loss 和 predicted-clean residual；实线蓝点各自的 GT，虚线表示残差匹配关系。

#### 技术机制与上下文

三路条件的分工是外观、相机方向/位置、显式局部对应。训练 Query GT 只参与监督（clean latent Z0 和真实 velocity U），不会作为条件输入。Eq. (2) 给出流匹配速度损失；Eq. (6) 将预测速度换算为 clean residual；Eq. (7) 计算轨迹损失。图顶部随机几何 dropout 仅涉及几何条件，不是把源图一起丢弃。联合 attention 在 view axis 去掉时间顺序位置编码，保留空间编码（p.4）。

**证据边界。** 图中的 block schematic 不能推断未给出的 tensor 维数或具体 kernel；训练监督框不是推理阶段需要 GT。A.11 明确正式定量不采用目标时间压缩。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
### Figure 3｜视觉几何路由机制

![Figure 3](VGGT-Diff/Figures/Figure_03.jpg)

*图源：PDF p.5，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 3: Visual Geometry Conditioning. Our conditioner constructs a query-aligned hard anchor from z-buffered near-front source features. A lightweight layered router forms confidence-weighted front and secondary hypotheses and predicts a residual correction from them and routing statistics.

**中文翻译：** 条件器由 z-buffer 筛选的近前表面源特征构造目标对齐的 hard anchor；轻量分层路由器生成置信度加权的前层与次层假设，结合路由统计预测残差修正。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】上方从多个输入图经 VGGT-Ω 得到 RGB/feature point cloud，query camera 将三维点投影到新视角；特征颜色是可视化，不是直接输出的 RGB。下方四组 map 分别为 hard anchor、front hypothesis、back hypothesis、routed statistics。hard anchor 显示稀疏锐利区域；前层图补入邻域投影支持，后层图保留更远表面；统计图示出四个不同通道的可视化。Concat 将前后假设和统计交给 Residual Refiner，Add 将修正加回 hard anchor，输出 routed condition 橙色 token。箭头从 hard anchor 直接到加法，是保留几何主支路的关键。

#### 技术机制与上下文

Eq. (3) 将源预测 3D 点变换至目标相机并投影；Eq. (4) 双线性 splat 乘置信度及相对深度衰减；Eq. (5) 的零初始化残差令初始输出等于硬锚。后层不是必然被目标真正看见，而是在遮挡边界或点误差时作为候选证据；没有源支持的位置交给扩散补全。Table 3 无 layered refinement 损失 0.129 dB，相比完全去 Ω 的 1.730 dB 小得多。

**证据边界。** 图示 RGB point cloud 不是最终场景模型；两层不是对真实所有遮挡面的重建。投影错配、透明/镜面、薄结构、置信失准仍可能造成错误。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
### Figure 4｜DL3DV 定性对比

![Figure 4](VGGT-Diff/Figures/Figure_04.jpg)

*图源：PDF p.8，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 4: Qualitative comparison on diverse DL3DV scenes. VGGT-Diff better matches the requested viewpoints while preserving fine geometry, legible text, and object identity; previous methods exhibit blur, structural distortion, or content drift.

**中文翻译：** 在多样 DL3DV 场景上，VGGT-Diff 更好匹配请求视角，同时保留细几何、可辨文字和物体身份；其他方法出现模糊、结构变形或内容漂移。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】5行7列，以每行 GT 为参照，后续 Ours、FrameCrafter、SEVA、MVSplat360、DepthSplat、LVSM。第一行教室椅腿、桌边及排列，第二行建筑入口标牌、立柱和地面，第三行站台柱体/扶手结构与数字标记，第四行车轮辐条与框架，第五行货品/装饰与背景。GT 红色虚框定位关注部位，并不是输出算法的 mask。可见 LVSM 在复杂几何处严重模糊；部分生成基线物体清晰却布局有偏移；Ours 在多个案例更接近 GT 的边界和局部结构。

#### 技术机制与上下文

此图对应 Table 1 的总体图像指标及 §4.2 (2)。像素 PSNR 下降可能是纹理不清，也可能是清晰但位置错；视觉观察配合图11误差图有助分辨。VGR 可使目标局部继承源图证据，但可见结果无法单独证明是哪一模块贡献。

**证据边界。** 作者选择的例图，不是随机样本统计；不能把红框内视觉相似推论为真实三维坐标或所有未见内容正确。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
### Figure 5｜Mip-NeRF 360 零样本对比

![Figure 5](VGGT-Diff/Figures/Figure_05.jpg)

*图源：PDF p.9，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 5: Qualitative comparison on Mip-NeRF 360. Without dataset-specific fine-tuning, VGGT-Diff better preserves fine structures, occlusion boundaries, and source-observed appearance while more accurately matching the target view.

**中文翻译：** 无需数据集特定微调，VGGT-Diff 更好保留精细结构、遮挡边界与源观测外观，同时更准确匹配目标视角。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】4行7列：GT、Ours、FrameCrafter、SEVA、MVSplat360、GeoNVS、Aether（以实际图上标题为准）。树干与木平台，横向栅格与自行车，粉色花卉与背景，厨房台面与锅具。第一行平台边缘和树干遮挡，第二行栏杆格栅/细结构及背景自行车，第三行花团与物体摆放，第四行容器轮廓和相对位置。Ours较接近多个 GT 布局；其他方法出现模糊、漏物体或生成位置变化。

#### 技术机制与上下文

跨数据集不再微调而仍保留观测结构，是作者用于支持 generalization 的定性证据。与 Table 1 Mip-NeRF 360 指标合读：早期 VGGT-Diff 不是 PSNR/SSIM 都第一，但 LPIPS/DreamSim 好；scaled 更好但 SSIM 仍不是最佳。

**证据边界。** 不证明任意 OOD（如水下、透明介质）泛化；该 benchmark 仍是普通场景，预训练分布未隔离。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
### Figure 6｜姿态难度协议

![Figure 6](VGGT-Diff/Figures/Figure_06.jpg)

*图源：PDF p.16，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 6: Pose-difficulty protocol and representative predictions. Blue frustums denote the six source cameras and purple denotes the query. Rows distinguish trajectory interpolation from extrapolation; columns increase pose novelty from near to far. Frozen VGGT-Ω point clouds, fused from multiple continuous GT-view windows, provide scene context only. Insets compare VGGT-Diff with GT and do not determine the bins.

**中文翻译：** 蓝色视锥表示六源相机，紫色表示目标；行区分轨迹内插与外推，列从近到远增加姿态新颖度。由多个连续 GT 视角窗口融合的冻结 VGGT-Ω 点云仅用于场景上下文；插图比较 VGGT-Diff 和 GT，不决定分箱。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】2×3 格：上排 interpolation，下排 extrapolation；列 Near/Mid/Far。每格上方给出点云中的源蓝相机和 query 紫相机相对位置，下方两张是 Ours 与 GT。近处相机与已有视角方向/中心接近；远处位置与方向偏离更明显。相同场景的上下行可能由不同 query 得到，不应把紫色视锥到点云视觉距离当公式分数。

#### 技术机制与上下文

A.1 Eq. (8) 使用中心距离除源轨迹直径，加上 forward direction 夹角除 π，再对源取最小。轨迹索引是否在源首尾间决定行；不是三维凸包判定。I/E各自按全局 tertile 得到独立阈值，32例/bin，图中点云、预测和 GT 不参与选样。

**证据边界。** 单格仅代表场景案例；不同 I/E 阈值导致同为 near 不表示相同绝对姿态差。用于展示的 GT 点云不等于模型推理输入。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
### Figure 7｜训练预算和指标

![Figure 7](VGGT-Diff/Figures/Figure_07.jpg)

*图源：PDF p.18，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 7: Evaluation metrics across the clean-10K optimization budget. Under the 192-case half-resolution protocol, PSNR, SSIM, and LPIPS-VGG are measured as training extends from 18K to 100K effective steps. Despite local fluctuations, all metrics improve substantially, with the best results at 100K. The open square denotes the clean-1K model evaluated at the matched 18K budget.

**中文翻译：** 在192案例半分辨率协议下，训练18K到100K有效步数的PSNR、SSIM和LPIPS-VGG被测量；尽管局部波动，三指标显著改善，100K最佳；空心方块是相同18K预算clean-1K模型。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】三个子图共享横轴 Effective training steps (K)，从左至右 PSNR(dB) 蓝、SSIM 紫、LPIPS-VGG 橙。实心点及线为clean-10K各checkpoint，空心方块是clean-1K 18K参考。PSNR整体增加、SSIM整体增加、LPIPS整体下降，42K、60K、88K局部回退；到100K达到最佳，末段PSNR增长趋缓。

#### 技术机制与上下文

Table 6 给出精确值：18K→100K PSNR18.7861→20.2780、SSIM0.5272→0.6166、LPIPS0.2793→0.2335。A.3指出96K→100K PSNR仅增加0.0136 dB（正文写0.0135，可能为四舍五入），不表示所有指标饱和。数据规模变大但固定计算预算可能欠优化；曲线是相同clean-10K模型的步数变化，空心方块仅为18K不同数据规模参照。

**证据边界。** 不能从步数增长的曲线单独定量分离数据规模、优化长度及后续全分辨率适配的贡献。与Table1的AlexNet LPIPS/分辨率不可直接比较。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
### Figure 8｜训练损失动力学

![Figure 8](VGGT-Diff/Figures/Figure_08.jpg)

*图源：PDF p.18，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 8: Training dynamics on DL3DV clean-10K. Curves connect non-overlapping 1K-step averages across the full training trajectory. The dashed line marks the original 18K fixed-compute budget. PTRC is reported as the raw, unweighted consistency loss. Its initial rise reflects scheduled activation and the increasingly difficult target curriculum rather than optimization divergence.

**中文翻译：** 曲线连接完整训练中不重叠的1K步平均值，虚线表示原18K固定计算预算。PTRC为原始未加权损失；初期上升反映调度激活与目标课程变难，而非优化发散。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】左图纵轴Flow matching loss、横轴有效训练K步；前段快速下降，18K附近再降，后段缓慢下降；右图PTRC loss (unweighted) 初期从接近零爬升，到18K附近峰值后缓慢降低。两图分别独立纵轴，不能比较曲线绝对量级。18K竖虚线表示预算分界而非停止点。

#### 技术机制与上下文

A.2 Eq. (9) stage内前10%关闭、接下来20%升权；目标数课程逐步增加；右图原始未加权误差还受点轨迹难度和训练样本构成改变影响。Table6 18K→100K flow 0.074406→0.050209、PTRC0.087436→0.073912，分别约32.5%和15.5%降低。图7评估指标同步改善提供相互证据。

**证据边界。** 损失上升原因是作者解释，不是严格因果消融；损失低不自动代表真实几何准确；两损失的数学单位与尺度不同。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
### Figure 9｜冻结层选择及联合优化

![Figure 9](VGGT-Diff/Figures/Figure_09.jpg)

*图源：PDF p.19，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 9: VGGT-Ω feature depth and optimization. Intermediate frozen features perform best, while jointly optimizing the Layer 23 visual pathway degrades PSNR.

**中文翻译：** 中间层冻结特征表现最好；联合优化Layer 23视觉路径使PSNR下降。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】横轴 VGGT-Ω configuration，前四点冻结L4/L11/L17/L23，右侧紫点joint optimization L23；纵轴 PSNR dB。标注约18.59、18.73、18.67、18.58、17.93，虚线标出冻结到联合优化L23约-0.65 dB。横轴是离散配置，不是连续网络深度函数；折线只是配置对比。

#### 技术机制与上下文

A.5/Table8固定半分辨率诊断，L11在三指标最佳，L23为预先主实验配置避免基于测试集后验选型。联合优化视觉aggregator和encoder（1e-6）但冻结相机/深度头，可能破坏原预训练视觉组织和冻结头之间的相容性。

**证据边界。** L11优势有限，不能推广到所有数据或改进策略；联合优化结果仅限当前冻结头、学习率和数据。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN FIG-10 -->
### Figure 10｜两种重建器的跨视图几何代理

![Figure 10](VGGT-Diff/Figures/Figure_10.jpg)

*图源：PDF p.21，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 10: Point-cloud reconstruction from jointly generated views. The upper and lower blocks use VGGT-Ω and Pi3, respectively. Within each block, normalized point-to-reference errors are shown above the corresponding reconstructions using the displayed color scale. Both reconstructors show fewer displaced points and more coherent geometry for VGGT-Diff, indicating stronger cross-view consistency.

**中文翻译：** 上、下组分别使用VGGT-Ω和Pi3重建；每组上排归一化点到参考误差及色条，下排对应点云。两个重建器均显示VGGT-Diff位移点更少、几何更连贯，表明跨视图一致性更强。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】上组标题VGGT-Ω、下组Pi3。每组列为SEVA、FrameCrafter、Ours、GT-view Reconstruction（参考列没有自身误差图，显示色条）。误差图蓝色表示小误差，红橙为大误差，色条显示归一化比例0、约2.4/2.5%、≥4.9%。建筑正面窗格、轮廓和前景点为关注区域。Ours误差图大面积偏蓝，窗面和结构在重建点云中较连贯；SEVA/FrameCrafter较多暖色区域及漂移点。

#### 技术机制与上下文

Table4同一52场景8目标视角、同样相机设置，独立于训练的Pi3也给出相同排序。比较在同一reconstructor内，以GT目标RGB经同一重建器得到的点云作为参考，而不是激光扫描。两重建器绝对误差值不同不能直接横比；此设计减弱共享VGGT-Ω先验的评估偏置。

**证据边界。** 点云可重建性同时受生成纹理、几何、重建器匹配能力影响；一致但错误的场景仍可能得高分。选例仅说明可见的误差分布，不证明物理GT。

<!-- PTH:END FIG-10 -->

<!-- PTH:BEGIN FIG-11 -->
### Figure 11｜目标视角空间对齐与像素误差

![Figure 11](VGGT-Diff/Figures/Figure_11.jpg)

*图源：PDF p.24，完整区域渲染 JPEG 展示副本，非原始图像文件。*

**原文 Caption：** Figure 11: Target-view alignment and pixel error. Predicted RGB views are shown above their absolute pixel differences from GT under a shared error scale. VGGT-Diff produces lower and more localized errors than the baselines, indicating closer alignment with the GT viewpoint and fewer spatial or structural mismatches.

**中文翻译：** 预测RGB在其与GT的绝对像素差之上展示，使用共享误差色标。VGGT-Diff比基线误差更低、更局部，说明更接近GT视点且空间或结构错配更少。

#### 视觉布局与逐区域理解

【原文事实/实际视觉观察】两列场景组，各三行（6场景），每组GT/Ours/FrameCrafter/SEVA四列，预测RGB下方为各方法绝对差值热图，GT列为空。上左室外梁柱/草坪，上右建筑窗格；中左桌椅室内、中右货架；下左餐厅室内、下右建筑台阶与草坪。多数 Ours 误差集中于边缘或纹理，基线部分出现大范围高误差区域如错位墙/梁、货架、窗格与前景。

#### 技术机制与上下文

和Fig4/5互补：即使生成图视觉上合理，像素位置偏移也会造成边缘周围高误差。误差定位支持目标相机条件和源对应改善的解释，但同样受亮度、纹理和曝光差影响。与Table1 PSNR定量结合阅读。

**证据边界。** 绝对RGB误差没有单独分解几何与光度误差；图没有提供逐案例数字或误差归一化之外的尺度信息，不能由热图计算深度或物理误差。

<!-- PTH:END FIG-11 -->

<!-- PTH:BEGIN TABLE-01 -->
### Table 1｜六视图主结果（p.7）

**原文题注（PDF）：** Quantitative comparison of 6-view NVS on DL3DV and Mip-NeRF 360. All methods follow the same protocol within each benchmark; #Scenes excludes generic foundation-model pretraining. LPIPS uses AlexNet features, and DS denotes DreamSim. VGGT-Diff (scaled) uses the clean 9,510-scene split, with 100K effective half-resolution steps followed by 6K full-resolution adaptation steps. Best, second-best, and third-best results use the corresponding rank colors.

**中文：** 六视图NVS比较；训练场景数不含基础模型预训练；LPIPS-AlexNet，DS为DreamSim；规模版使用9510场景、100K半分辨率有效步及6K全分辨率适配步。

| 方法 | 骨干 | 场景数 | D PSNR↑ | D SSIM↑ | D LPIPS↓ | D DS↓ | M PSNR↑ | M SSIM↑ | M LPIPS↓ | M DS↓ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AnySplat | VGGT | 254K | 12.388 | 0.298 | 0.510 | 0.214 | 11.009 | 0.232 | 0.594 | 0.252 |
| E-RayZer | – | 10K | 16.850 | 0.442 | 0.455 | 0.254 | 16.560 | 0.343 | 0.621 | 0.340 |
| LVSM | – | 67.5K | 17.090 | 0.478 | 0.333 | 0.204 | 15.250 | 0.317 | 0.609 | 0.577 |
| DepthSplat | Depth Anything V2 | 77.5K | 17.284 | 0.566 | 0.307 | 0.158 | 15.977 | 0.370 | 0.410 | 0.215 |
| EscherNet | SD1.5 | 10K | 12.070 | 0.251 | 0.484 | 0.227 | 11.140 | 0.126 | 0.540 | 0.315 |
| Aether | CogVideoX-5B | – | 12.660 | 0.258 | 0.469 | 0.140 | 12.600 | 0.220 | 0.651 | 0.334 |
| MVSplat360 | SVD | 69.5K | 14.150 | 0.358 | 0.513 | 0.174 | 13.859 | 0.292 | 0.636 | 0.278 |
| SEVA | SD2.1 | 80K | 16.150 | 0.470 | 0.253 | 0.088 | 14.590 | 0.294 | 0.372 | 0.137 |
| FrameCrafter | Wan2.1-I2V-14B | 1K | 17.180 | 0.445 | 0.223 | 0.066 | 15.640 | 0.279 | 0.365 | 0.111 |
| VGGT-Diff | Wan2.1-I2V-14B | 1K | 18.104 | 0.508 | 0.222 | 0.062 | 16.296 | 0.317 | 0.315 | 0.091 |
| VGGT-Diff scaled | Wan2.1-I2V-14B | 10K | 19.283 | 0.569 | 0.198 | 0.054 | 17.217 | 0.362 | 0.283 | 0.080 |

D=DL3DV，M=Mip-NeRF360；前四行为回归式，其余扩散式。PSNR衡量像素误差（dB，越大越好），SSIM衡量结构相似性，LPIPS与DreamSim衡量感知距离。原始版对同骨干FrameCrafter在D上+0.924 dB、M上+0.656 dB；规模版再增1.179/0.921 dB。原始版D的SSIM不及DepthSplat，M的PSNR不及E-RayZer；规模版仅M SSIM仍低于DepthSplat（0.362 vs 0.370）。

**证据边界：** 训练数据与预算不同，不可把规模版增益全归因于路由；跨方法公布协议不等于统一训练或几何真值。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
### Table 2｜DL3DV姿态分层PSNR（p.8）

**原文题注（PDF）：** PSNR across target-pose difficulty on DL3DV. All methods use independent 6-to-1 inference over 32 targets per bin.

**中文：** DL3DV按目标姿态难度的PSNR；每组32目标，独立6→1。

| 方法 | I近 | I中 | I远 | E近 | E中 | E远 |
| --- | --- | --- | --- | --- | --- | --- |
| AnySplat | 14.196 | 12.415 | 10.862 | 13.977 | 12.566 | 11.827 |
| E-RayZer | 17.388 | 14.878 | 13.824 | 16.833 | 14.477 | 14.712 |
| LVSM | 20.276 | 16.599 | 13.969 | 20.790 | 16.633 | 15.579 |
| DepthSplat | 20.376 | 17.424 | 14.820 | 20.653 | 17.943 | 16.374 |
| Aether | 14.030 | 12.167 | 11.065 | 15.739 | 12.295 | 11.795 |
| GEN3C | 13.620 | 12.356 | 11.548 | 14.288 | 13.050 | 12.288 |
| MVSplat360 | 15.282 | 14.420 | 13.389 | 15.542 | 14.479 | 14.176 |
| FrameCrafter | 20.338 | 17.485 | 14.649 | 21.168 | 17.888 | 16.594 |
| SEVA | 20.940 | 17.861 | 14.703 | 22.311 | 18.050 | 16.575 |
| VGGT-Diff | 20.920 | 18.391 | 15.853 | 22.396 | 18.793 | 17.793 |
| VGGT-Diff scaled | 22.772 | 19.640 | 16.755 | 24.222 | 20.282 | 18.673 |

I=插值，E=外推；原版五组领先，I近比SEVA低0.020 dB。规模版六组均首位。I远原版15.853相比FrameCrafter14.649提高1.204 dB。远组质量普遍更低，表明稀疏观测的难度并非均匀。

**证据边界：** 分组是轨迹关系及相对姿态新颖度而非几何遮挡真值；各组不同场景，不能把I/E当作唯一变化量。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
### Table 3｜组件消融（p.8）

**原文题注（PDF）：** Ablations on DL3DV. All variants use independent 6-to-1 inference on the 192 pose-stratified targets; LPIPS uses VGG features.

**中文：** 192姿态分层目标独立6→1消融，LPIPS-VGG。

| 配置 | PSNR↑ | SSIM↑ | LPIPS↓ |
| --- | --- | --- | --- |
| w/o VGGT-Ω prior | 17.294 | 0.501 | 0.357 |
| w/ point-rendered RGB | 18.553 | 0.536 | 0.316 |
| w/o PTRC | 18.586 | 0.530 | 0.312 |
| w/o layered residual refinement | 18.895 | 0.548 | 0.303 |
| w/o condition regularization | 18.722 | 0.538 | 0.309 |
| w/o CFG (full method) | 19.024 | 0.557 | 0.301 |
| w/ geometry-prior CFG | 19.139 | 0.563 | 0.296 |

以无CFG完整模型为基准，去Ω先验损失1.730 dB（同时取消依赖它的模块）；RGB替代路由特征−0.471，去PTRC −0.438，去分层−0.129，去条件正则−0.302；推理CFG单独+0.115。分层小增益与全几何先验总增益不能混为一谈。

**证据边界：** 去Ω不是独立只去一个条件；各训练变体之间不能直接相加贡献。VGG与表1 AlexNet不可直接比较。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
### Table 4｜双重建器几何代理（p.9）

**原文题注（PDF）：** Cross-reconstructor evaluation on DL3DV. Eight jointly generated views are reconstructed with frozen VGGT-Ω and Pi3 and compared with GT-view references produced by the same reconstructor. Pi3 is not used for training or conditioning VGGT-Diff. Metrics are trajectory-normalized and should be compared within each reconstructor.

**中文：** 八个联合生成视图分别由冻结VGGT-Ω和Pi3重建，同重建器GT图像重建为参考；轨迹尺度归一化。

| 方法 | Ω Chamfer↓ | Ω F@1%↑ | Ω F@2%↑ | Pi3 Chamfer↓ | Pi3 F@1%↑ | Pi3 F@2%↑ |
| --- | --- | --- | --- | --- | --- | --- |
| LVSM | 0.2387 | 0.1116 | 0.2218 | 0.3610 | 0.0673 | 0.1525 |
| SEVA | 0.1944 | 0.2050 | 0.3244 | 0.2514 | 0.1640 | 0.2751 |
| FrameCrafter | 0.1000 | 0.3086 | 0.4793 | 0.1404 | 0.2140 | 0.3880 |
| VGGT-Diff | 0.0733 | 0.4231 | 0.6041 | 0.1174 | 0.3089 | 0.5021 |

52场景相同六源八目标。Chamfer是双向近邻距离类指标，F-score结合阈值内的精确率与召回率；阈值为轨迹尺度百分比。相对FrameCrafter，Ω距离减26.7%，Pi3减16.4%；F@1%分别+0.1145/+0.0949。Pi3未参与训练，降低同源评估偏好疑虑。

**证据边界：** 两重建器参考点云是GT视图经模型重建，不是实测三维扫描；只在同重建器内比较，不能证明绝对几何精度。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
### Table 5｜固定超参数（p.17）

**原文题注（PDF）：** Fixed geometry-conditioning hyperparameters. The relative-depth router constants are shared across scenes and resolutions.

**中文：** 几何条件固定超参；相对深度常数跨场景和分辨率共享。

| 组件 | 设置 | 值 |
| --- | --- | --- |
| VGR | 第二层相对边界δ | 0.04 |
| VGR | 相对深度可见性温度τ | 0.02 |
| PTRC | 权重；Smooth-L1转折 | 0.1；0.1 |
| PTRC schedule | 关闭/线性爬升/全权重 | 10%/20%/70% |
| Condition regularization | drop/weaken/full概率 | 0.10/0.20/0.70 |
| Geometry-Prior CFG | scale/步数/引导区间 | 2.0/50/前60% |
| Text conditioning | prompt/text CFG | 空/1 |

δ控制前后层按相对深度分隔，τ控制软可见性衰减；两者无绝对距离单位。PTRC权重0.1与Smooth-L1 β=0.1是不同含义；前者乘损失，后者定义二次/线性分段。

**证据边界：** 固定值而非系统敏感性扫描；不能推断最优超参或跨域有效。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
### Table 6｜数据与预算规模（p.18）

**原文题注（PDF）：** Checkpoint scaling on DL3DV clean-10K. Image metrics use the fixed 192-case half-resolution protocol. Losses average the preceding 1K effective steps; PTRC is reported before weighting.

**中文：** clean-10K不同训练步检查点；192固定半分辨率协议；损失为前1K有效步均值，PTRC未乘权重。

| 步数 | PSNR↑ | SSIM↑ | LPIPS↓ | Flow loss↓ | PTRC loss↓ |
| --- | --- | --- | --- | --- | --- |
| 18K | 18.7861 | 0.5272 | 0.2793 | 0.074406 | 0.087436 |
| 24K | 19.1529 | 0.5509 | 0.2621 | 0.057234 | 0.086353 |
| 30K | 19.1980 | 0.5579 | 0.2593 | 0.056245 | 0.085368 |
| 36K | 19.6120 | 0.5770 | 0.2507 | 0.055986 | 0.084214 |
| 42K | 19.5829 | 0.5762 | 0.2541 | 0.053832 | 0.080950 |
| 48K | 19.6875 | 0.5830 | 0.2490 | 0.053431 | 0.080535 |
| 54K | 19.8138 | 0.5889 | 0.2467 | 0.053542 | 0.079932 |
| 60K | 19.4351 | 0.5725 | 0.2534 | 0.052149 | 0.077931 |
| 72K | 19.9368 | 0.5973 | 0.2411 | 0.052154 | 0.077268 |
| 84K | 20.1706 | 0.6088 | 0.2373 | 0.052251 | 0.076402 |
| 88K | 20.1350 | 0.6059 | 0.2388 | 0.052218 | 0.076123 |
| 96K | 20.2644 | 0.6146 | 0.2342 | 0.050399 | 0.074175 |
| 100K | 20.2780 | 0.6166 | 0.2335 | 0.050209 | 0.073912 |

18K→100K：PSNR+1.4919，SSIM+0.0894，LPIPS−0.0458；Flow loss约减32.5%，PTRC约减15.5%。60K图像指标回落而训练损失继续下降，说明训练目标与测试效果非单调对应；96K到100K收益小。

**证据边界：** 同一clean-10K训练轨迹，不能识别独立数据量效应；半分辨率指标不能与表1 full-res直接比较。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN TABLE-07 -->
### Table 7｜联合与独立八目标（p.19）

**原文题注（PDF）：** Joint versus independent eight-view generation. Image metrics use 416 targets; geometry proxies are computed from the same outputs with frozen VGGT-Ω. LPIPS uses VGG features, and Chamfer-L1 is trajectory-normalized.

**中文：** 联合与独立八目标；416目标图像指标，同输出用冻结Ω计算几何代理；LPIPS-VGG。

| 模式 | PSNR↑ | SSIM↑ | LPIPS↓ | Chamfer↓ | F@1%↑ | F@2%↑ |
| --- | --- | --- | --- | --- | --- | --- |
| Joint-8 | 17.6401 | 0.4787 | 0.3568 | 0.0483 | 0.3438 | 0.5341 |
| Independent-8 | 17.1784 | 0.4453 | 0.3744 | 0.0741 | 0.2432 | 0.4047 |

同checkpoint、相同每视图噪声种子；联合可通过共享注意力互相约束而非仅批处理。六项改善：+0.4617、+0.0334、−0.0176、−0.0258（正文以未舍入值给−0.0259）、+0.1006、+0.1294；场景bootstrap 95% CI均排除0。

**证据边界：** 只验证该八目标协议；联合并不保证任意长轨迹一致或绝对几何正确。

<!-- PTH:END TABLE-07 -->

<!-- PTH:BEGIN TABLE-08 -->
### Table 8｜冻结特征层与联合优化（p.19）

**原文题注（PDF）：** VGGT-Ω feature and optimization diagnostics. All variants use the same half-resolution protocol; LPIPS uses VGG features.

**中文：** Ω特征及优化诊断；同半分辨率协议，LPIPS-VGG。

| 配置 | PSNR↑ | SSIM↑ | LPIPS↓ |
| --- | --- | --- | --- |
| 冻结L4 | 18.588 | 0.5160 | 0.2828 |
| 冻结L11 | 18.734 | 0.5253 | 0.2763 |
| 冻结L17 | 18.670 | 0.5197 | 0.2788 |
| 冻结L23 | 18.581 | 0.5155 | 0.2803 |
| 联合优化L23 | 17.934 | 0.4803 | 0.3021 |

L11相对L23+0.153 dB，最优层不必是最深层；联合训练视觉聚合器/编码器（LR 1e-6，冻结camera/depth heads）比固定L23低0.647 dB。主实验预先选定L23。

**证据边界：** 并未穷举学习率、层组合或冻结策略；不能断言联合微调总是有害。

<!-- PTH:END TABLE-08 -->

<!-- PTH:BEGIN TABLE-09 -->
### Table 9｜两数据集姿态PSNR（p.20）

**原文题注（PDF）：** PSNR across target-pose difficulty (↑). Results are reported separately for DL3DV and Mip-NeRF 360. The scaled checkpoint uses 100K half-resolution steps followed by 6K full-resolution adaptation. Ties at the reported precision share the same rank.

**中文：** 两数据集按姿态难度的PSNR；规模版100K+6K训练；报告精度下相同值并列。

| 方法 | D I近 | D I中 | D I远 | D E近 | D E中 | D E远 | M I近 | M I中 | M I远 | M E近 | M E中 | M E远 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AnySplat | 14.196 | 12.415 | 10.862 | 13.977 | 12.566 | 11.827 | 12.424 | 11.040 | 9.851 | 12.594 | 10.273 | 8.972 |
| E-RayZer | 17.388 | 14.878 | 13.824 | 16.833 | 14.477 | 14.712 | 15.964 | 15.085 | 14.028 | 15.404 | 14.760 | 12.609 |
| LVSM | 20.276 | 16.599 | 13.969 | 20.790 | 16.633 | 15.579 | 16.117 | 14.488 | 13.722 | 15.296 | 14.040 | 13.016 |
| DepthSplat | 20.376 | 17.424 | 14.820 | 20.653 | 17.943 | 16.374 | 17.478 | 16.046 | 15.303 | 16.208 | 15.518 | 12.511 |
| Aether | 14.030 | 12.167 | 11.065 | 15.739 | 12.295 | 11.795 | 12.977 | 12.994 | 12.867 | 11.841 | 10.873 | 10.894 |
| GEN3C | 13.620 | 12.356 | 11.548 | 14.288 | 13.050 | 12.288 | – | – | – | – | – | – |
| MVSplat360 | 15.282 | 14.420 | 13.389 | 15.542 | 14.479 | 14.176 | 14.283 | 13.928 | 13.585 | 14.407 | 13.463 | 12.794 |
| FrameCrafter | 20.338 | 17.485 | 14.649 | 21.168 | 17.888 | 16.594 | 16.933 | 15.700 | 15.016 | 16.222 | 15.330 | 13.017 |
| SEVA | 20.940 | 17.861 | 14.703 | 22.311 | 18.050 | 16.575 | 17.911 | 16.467 | 15.427 | 17.448 | 15.925 | 13.933 |
| VGGT-Diff | 20.920 | 18.391 | 15.853 | 22.396 | 18.793 | 17.793 | 17.254 | 16.630 | 15.611 | 16.336 | 16.061 | 13.989 |
| VGGT-Diff scaled | 22.772 | 19.640 | 16.755 | 24.222 | 20.282 | 18.673 | 18.658 | 17.458 | 16.125 | 17.448 | 17.077 | 14.694 |

D同表2；M原版四组最佳，I近及E近落后SEVA。规模版M六组最佳（E近17.448与SEVA并列），D六组最佳。GEN3C的M未报告。

**证据边界：** 不能把未报告项当零或失败；不同数据集姿态阈值不同，不宜直接比较同名组。

<!-- PTH:END TABLE-09 -->

<!-- PTH:BEGIN TABLE-10 -->
### Table 10｜两数据集姿态LPIPS（p.21）

**原文题注（PDF）：** LPIPS across target-pose difficulty (↓). We use VGG features on DL3DV and AlexNet features on Mip-NeRF 360. The scaled checkpoint uses 100K half-resolution steps followed by 6K full-resolution adaptation.

**中文：** 两数据集按姿态难度的LPIPS；D用VGG，M用AlexNet；规模版100K+6K。

| 方法 | D I近 | D I中 | D I远 | D E近 | D E中 | D E远 | M I近 | M I中 | M I远 | M E近 | M E中 | M E远 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AnySplat | 0.5052 | 0.5609 | 0.6214 | 0.5057 | 0.5483 | 0.5828 | 0.5513 | 0.5854 | 0.6135 | 0.5920 | 0.6349 | 0.6830 |
| E-RayZer | 0.5496 | 0.6357 | 0.6925 | 0.5527 | 0.6433 | 0.6508 | 0.7270 | 0.7978 | 0.8284 | 0.7494 | 0.7975 | 0.7859 |
| LVSM | 0.2911 | 0.4413 | 0.5656 | 0.2375 | 0.3913 | 0.4835 | 0.4980 | 0.6461 | 0.6897 | 0.5315 | 0.6589 | 0.7058 |
| DepthSplat | 0.2688 | 0.3592 | 0.4643 | 0.2600 | 0.3141 | 0.4077 | 0.3164 | 0.4058 | 0.4555 | 0.3965 | 0.4842 | 0.5703 |
| Aether | 0.4949 | 0.5891 | 0.6405 | 0.3952 | 0.5579 | 0.6098 | 0.5731 | 0.5855 | 0.5978 | 0.5965 | 0.6857 | 0.6980 |
| GEN3C | 0.5194 | 0.5808 | 0.6269 | 0.4773 | 0.5578 | 0.5990 | – | – | – | – | – | – |
| MVSplat360 | 0.5009 | 0.5374 | 0.5952 | 0.4855 | 0.5223 | 0.5724 | 0.5820 | 0.6453 | 0.6610 | 0.6404 | 0.6683 | 0.6743 |
| FrameCrafter | 0.2489 | 0.3558 | 0.4689 | 0.2150 | 0.3011 | 0.3996 | 0.2712 | 0.3639 | 0.3996 | 0.3148 | 0.4204 | 0.5152 |
| SEVA | 0.2460 | 0.3390 | 0.4546 | 0.2122 | 0.2959 | 0.3929 | 0.2564 | 0.3267 | 0.3854 | 0.2983 | 0.4013 | 0.4744 |
| VGGT-Diff | 0.2423 | 0.3227 | 0.4163 | 0.1922 | 0.2732 | 0.3572 | 0.2474 | 0.3085 | 0.3504 | 0.2922 | 0.3685 | 0.4424 |
| VGGT-Diff scaled | 0.2035 | 0.2831 | 0.3861 | 0.1628 | 0.2322 | 0.3248 | 0.2085 | 0.2772 | 0.3236 | 0.2645 | 0.3303 | 0.4122 |

原版在两个数据集的所有分组均优于其他方法，规模版进一步降低。例如D E远0.3572→0.3248，M E远0.4424→0.4122。

**证据边界：** LPIPS低意味着在指定特征空间更接近参考图像，并不证明三维结构或不可见内容真实；不同特征网络数值不可比。

<!-- PTH:END TABLE-10 -->

<!-- PTH:BEGIN TABLE-11 -->
### Table 11｜残差与直接速度一致性（p.22）

**原文题注（PDF）：** PTRC versus direct velocity matching across pose difficulty. Both use the same half-resolution protocol; PTRC values are highlighted and LPIPS uses VGG features.

**中文：** PTRC与直接速度匹配的分层比较，同半分辨率协议，LPIPS-VGG。

| 难度 | Direct PSNR↑ | PTRC PSNR↑ | Direct SSIM↑ | PTRC SSIM↑ | Direct LPIPS↓ | PTRC LPIPS↓ |
| --- | --- | --- | --- | --- | --- | --- |
| I近 | 19.008 | 20.133 | 0.531 | 0.602 | 0.274 | 0.221 |
| I中 | 17.520 | 18.103 | 0.435 | 0.467 | 0.341 | 0.300 |
| I远 | 15.210 | 15.670 | 0.308 | 0.352 | 0.440 | 0.397 |
| E近 | 20.021 | 21.326 | 0.591 | 0.674 | 0.238 | 0.173 |
| E中 | 17.863 | 18.613 | 0.478 | 0.535 | 0.308 | 0.260 |
| E远 | 16.994 | 17.641 | 0.415 | 0.463 | 0.377 | 0.331 |

保持相同轨迹、可见性筛选、权重等，改变匹配量；六组全部改善，PSNR +0.460~1.305 dB。全192目标17.769→18.581。原因是相同3D点在不同视图的正确速度/latent本就可以不同，而相对各自GT的预测误差更适合约束。

**证据边界：** 这是作者具体直接匹配基线，不排除其他跨视图一致性设计更优。

<!-- PTH:END TABLE-11 -->

<!-- PTH:BEGIN TABLE-12 -->
### Table 12｜Wan适配方式（p.22）

**原文题注（PDF）：** Effect of Wan adaptation strategy. Both variants use the same half-resolution controlled protocol; LPIPS uses VGG features.

**中文：** Wan适配策略比较；同半分辨率受控协议，LPIPS-VGG。

| Wan适配 | PSNR↑ | SSIM↑ | LPIPS↓ |
| --- | --- | --- | --- |
| LoRA rank32 | 17.2563 | 0.4677 | 0.3554 |
| Full fine-tuning | 18.5807 | 0.5155 | 0.2803 |

全微调+1.3244 dB、+0.0478 SSIM、−0.0751 LPIPS。说明在本设置中需要较充分地改变预训练视频生成行为以适应无时序多视图条件。

**证据边界：** 仅比较rank32与全微调，未穷举其他参数高效方案、预算或成本；不是LoRA必然较差的定理。

<!-- PTH:END TABLE-12 -->

<!-- PTH:BEGIN TABLE-13 -->
### Table 13｜减少输入视图（p.23）

**原文题注（PDF）：** Robustness to sparse input views on DL3DV. Both methods use full-resolution independent M-to-1 inference on the same 192 targets. Source subsets are selected identically using only camera poses; LPIPS uses VGG features.

**中文：** 相同192目标，全分辨率独立M→1；仅用姿态选择相同源子集；LPIPS-VGG。

| 源数 | 方法 | PSNR↑ | SSIM↑ | LPIPS↓ | DS↓ |
| --- | --- | --- | --- | --- | --- |
| 3 | FrameCrafter | 16.9559 | 0.4647 | 0.3709 | 0.0807 |
| 3 | VGGT-Diff | 17.7372 | 0.4966 | 0.3391 | 0.0680 |
| 4 | FrameCrafter | 17.8545 | 0.5076 | 0.3390 | 0.0685 |
| 4 | VGGT-Diff | 18.4380 | 0.5307 | 0.3135 | 0.0592 |

同姿态排序从6源取嵌套3/4子集，同噪声种子，不再训练。三源相对FrameCrafter+0.7813 dB、四源+0.5835；三源时更大增益但绝对质量仍低于四源，说明几何先验不能完全替代真实观测。

**证据边界：** 只验证所选子集及3/4源，不保证任意位置、单视图或未知姿态条件。

<!-- PTH:END TABLE-13 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 2.4 图表关系

方法设计由Fig.2全局链条、Fig.3局部路由共同解释；表3检验完整先验、RGB替代、PTRC、分层、正则、CFG；表5提供固定常数；表8检查特征来源层和是否冻结；表11专门检验残差而非速度；表12验证全微调选择。Fig.6解释姿态采样，表2、9、10沿同样的两维分类比较；不能由Fig.6插图决定难度。Fig.7的图像指标和Fig.8的训练曲线对应表6检查点，均不直接等价全分辨率表1。Fig.4、5、11分别展示DL3DV、Mip-NeRF360的图像质量和目标对齐，表1给全测试集的平均指标；Fig.10与表4展示双重建器代理，表7隔离同checkpoint的联合生成收益。表13仅改变源子集，支持输入稀疏性讨论。

**证据边界：** 不能将定性样例视为总体统计，也不能将图像指标视为三维扫描误差。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 3.1 研究问题

原文§1、3.1（pp.2–3）：输入是M幅带已知内外参的源RGB图像，及N个给定查询相机；输出是N幅与查询相机对应的RGB图像。任务为静态场景稀疏视角NVS，主要训练6源；并非估计未知相机位姿，亦非直接输出可编辑3DGS/NeRF。观测场景在某一源视图可见，不代表同一点在任意查询视图可见；查询新暴露区域可能没有源证据。

从逆问题理解（补充）：设场景s、相机c、成像算子F，源约束F(s,c_i)=I_i。不同s可能同时满足这些源约束，却在F(s,c_q)上不同。因而即使姿态精确，也不能由有限观测唯一确定全部新视角内容。几何模型提供已见区域的空间对应，扩散模型提供由训练分布学习的补全先验。两者解决的是不同信息缺口：前者降低已知区域匹配歧义，后者使未观测区域具有统计上合理的外观，而非恢复唯一真实解。

**证据边界：** 论文未证明可辨识性；生成结果可能看似真实但与实际隐藏表面不一致。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 3.2 动机

§1指出重建式方法在几何支持较好的区域能保持忠实性，远视角和未见区域可能出现从邻近源颜色偏置而来的patch伪影；生成式模型可补全却可能覆盖可靠观测，产生结构漂移。图4/5提供具体样例，表3中去全部Ω依赖模块比完整无CFG低1.730 dB，说明加入几何信息有效但并非单模块归因。单纯点RGB投影比视觉特征路由低0.471 dB：RGB只是特定视点观测值，特征可包含纹理、结构和跨视图上下文；不意味着每个特征维度均可解释。VGR显式空间对齐，利用深度、置信度与软分层避免把硬遮挡当成无误真值。

为何再要PTRC？联合注意力建立通信通道，却不提供“同物理点的剩余误差应接近”这一监督。直接匹配目标latent会压制本应保留的视点差异。表11用相同点轨迹只改匹配量，支持残差形式。随机弱化几何则对应第三类风险：模型可能过分依赖不可靠的投影。

**证据边界：** 上述失败模式有实验证据，但未提供逐像素遮挡错误的独立因果分解；特征语义解释是合理机制推断。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 3.3 相关工作

§2（pp.2–3）分为可泛化/前馈NVS和生成/几何引导NVS。第一类由NeRF、Mip-NeRF、Gaussian Splatting等逐场景表示起步，经pixelNeRF、IBRNet、MVSNeRF等像素特征、MVS/极线/射线聚合实现跨场景泛化；SRT、ViewFormer、LVSM、Efficient-LVSM、RayZer降低显式3D归纳偏置；pixelSplat、MVSplat、Splatt3R、其他前馈3D表示及LagerNVS、SVSM、投影条件、SHARP改善效率/几何建模。共同边界是缺少观测的区域仍欠定，不等于所有重建式方法都无学习先验。

生成路线从pose-conditioned diffusion、多视图联合分布（Watson、Zero-1-to-3相关工作、SyncDreamer等原文引文）到视频先验（Kwak、Voleti、Müller、ViewCrafter、SEVA），以生成先验处理未见内容。FrameCrafter尤其接近：将预训练视频扩散适配为无序posed-view completion；本文与其同用Wan2.1-I2V-14B，但强调几何特征查询对齐及训练残差一致性。原文还列混合确定性/生成、test-time视频补全、动态任意视角、对应监督扩散。

几何基础模型DUSt3R、MASt3R、VGGT、VGGT-Ω提供相机、3D与对应信息；原文列MVSplat360（高斯与扩散结合）、稀疏3D优化、联合图像/几何扩散、latent生成细化、几何条件视频扩散、宽基线引导。本文没有把几何先验转换成完整可渲染场景，而是用特征—3D点—置信度作为生成条件及训练轨迹。实验证据是表1跨方法和表3/11受控消融，而非声称所有其他方法均无显式对应。

**证据边界：** 相关工作描述以本文§2为限，未逐篇独立核验引文。
本节不涉及独立数学推导。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-01 -->
## 3.4.1 多视图扩散

§3.1、Fig.2、A.1/A.11：M源与N查询组织为M+N个视图槽。独立编码的冻结Wan VAE每图16通道、空间下采样8倍；192×336→16×24×42，全分辨率480×832→16×60×104。训练Z0含源和GT查询，Xσ加噪；另有清洁源图latent条件，查询槽空白。推理仅编码源，查询从噪声开始，绝无查询GT条件。每个相机的Plücker射线为6通道，通常表示方向d与矩m=o×d（背景知识，原文未规定通道顺序）；8× PixelUnshuffle把每个8×8空间块重排到384通道，保留像素排列信息而非平均池化。

冻结VGGT-Ω给全局上下文和稠密几何条件。源特征直接按图像格对齐，查询特征通过VGR投影。Xσ、clean-source、ray、geometry在通道维拼接；扩展输入投影将其映射至预训练Wan token空间，原图像通路保留，新增相机/几何通路初始无影响。Fig.2中Conv3D patch embed按视图独立嵌入，40个DiT blocks共享处理所有视图token，含timestep AdaLN、self-attention、FFN，最终线性/反patch得到速度；保留空间位置编码，取消视图轴时间顺序编码。取消时间编码不是删除全部视图结构：相机和路由条件区分不同槽。联合注意力使目标间可以通信；仅对查询slot算FM损失并独立VAE decode。无场景专用文字，空文本固定。

坐标规约仅由源相机决定：选最近源中心质心的相机作为锚，变换到其原点和旋转，以源到锚的均距归一化平移；固定源集改变N或查询子集不会改变坐标系。

**证据边界：** 本文未公开每个中间token具体通道/参数量的全部实现细节；不假定移除时间编码即严格置换等变。

<!-- PTH:END METHOD-01 -->

<!-- PTH:BEGIN METHOD-02 -->
## 3.4.2 VGR

§3.2、Fig.3、A.2：冻结Ω为每源特征f_n关联3D点p_n与置信c_n；轻量adapter变换特征维度，3D点与置信用于路由，而不是生成完整辐射场。将p_n按查询外参变换、内参投影（Eq.3），剔除无效投影。硬锚将有效投影分配至最近格点，z-buffer保留接近最近有效深度的特征，再按归一置信平均。硬锚尖锐而便宜，但几何偏差或离散遮挡可能丢弃可用证据。

双层soft splat：对连续投影的邻格采用双线性核，每层再乘置信和相对深度可见性。前层锚在最小splat深度；后层只取深度超过前层1.04倍的点，再找后层最小深度。两层各自归一化聚合。四项统计为前/后层支持量（截断）、双线性加权平均置信、截断相对深度间隔。将前、后特征及统计输入轻量残差refiner，输出与Ghard相加。零初始化确保初始模型等于硬锚，而不是强制两层混色。后层是证据候选并非预测应可见的表面。无支持的查询区域不获得特定源特征，由扩散先验补全。表3的RGB替代与去分层比较给出分别0.471和0.129 dB的差距。

**证据边界：** 后层只代表从源预测点形成的候选；源中未见物体背面不会凭路由产生真实证据。硬锚保留阈值具体实现原文未完全给出。

<!-- PTH:END METHOD-02 -->

<!-- PTH:BEGIN METHOD-03 -->
## 3.4.3 PTRC

§3.3、A.2/A.8：FM为每视图误差之和，没有显式同一点误差关系。对每次训练采样的流时间σ，用预测速度得到clean估计Xσ−σV，再减各自真实clean latent得到E。用冻结Ω源点投影到所有查询对，只有两边在视野内、正深度并通过各自z-buffer的点进入轨迹集合C。将投影取最近latent格；同轨迹的E_a−E_b经逐通道Smooth-L1、置信度加权求平均。可靠点权重较高，极端残差在线性区受限。

梯度经E→V→DiT及可训练条件模块，不经过冻结几何/真实latent；不需VAE decoder。不同视图的理想latent可不同，PTRC只要求误差相近：例如两个真值分别3、8，预测4、9，误差同为1，PTRC为0而直接预测匹配差5；FM仍推动两者各自消除误差。因此PTRC不能单独监督绝对正确性。Eq.12直接速度对照、表11支持残差比该基线更好。

**证据边界：** 2D latent最近格与物理点对应并非精确像素语义对应；错误轨迹和共同系统误差仍可能存在。单查询时没有跨查询对；空轨迹实现保护原文未明确。

<!-- PTH:END METHOD-03 -->

<!-- PTH:BEGIN METHOD-04 -->
## 3.4.4 训练与推理

§3.4、A.2：冻结Ω与VAE，全量训练Wan DiT及扩展投影、adapter、VGR。两阶段各有独立PTRC schedule：前10%关闭，随后20%线性升至0.1，最后70%保持，避免早期不稳定预测间强耦合；无额外σ门控。训练每个样本抽共享a：10%为0，20%在[0.2,0.8]均匀，70%为1。a同时乘global Ω context和源/查询稠密几何，而RGB/ray/Xσ/timestep不变；不是逐像素随机遮挡，也不是文本dropout。

推理可不用CFG（完整模型），也可用目标几何专用CFG：同Xσ做完整V_G与参考V_ref两次预测，参考仅清零查询稠密路由，保留global Ω及源路由等所有其他条件。两者速度之差代表在相同条件下模型响应查询几何的变化；用有效scale从2平滑降到1（50步的前60%），后段等于V_G。附录描述full-Ω CFG替代参考会同时移除global及源/查询几何，不是正式结果协议。表3相同checkpoint的目标几何CFG+0.115 dB；文本CFG=1，空文本。

训练全几何弱化与推理仅查询删除不是完全相同的条件分布，应区分正则化赋予的鲁棒性和正式推理分支实际定义。

**证据边界：** CFG差向量不是严格可解释的独立几何物理量；原文未给置信自适应CFG，也未证明任何错误几何都能被克服。

<!-- PTH:END METHOD-04 -->

<!-- PTH:BEGIN METHOD-05 -->
## 3.4.5 VAE与轨迹扩展

A.11（pp.23–24）明确所有定量结果采用独立视图latent：物理相机、射线、几何与slot一一对应。80帧连续演示另行在192×336微调，源仍独立，目标改用同一冻结因果视频VAE的时间压缩。长度L_T=1+ceil((T−1)/4)，80帧补末帧成81，21目标槽；6+80=86变成6+21=27（减少约68.6%槽数，不等于实际加速比例）。解码去掉重复末帧。

相机不经VAE。每帧射线6→384通道，窗口[0]、[1,2,3,4]…[77,78,79,79]；四帧384拼为1536通道，用1×1×1卷积adapter投回384，初始仅选择第四帧，等价于原endpoint-only checkpoint。首槽保留单帧路径。相机矩阵和路由几何仍按endpoint {0,4,8,…,76,79}对齐，只有射线覆盖窗口所有姿态。因压缩latent语义不同，必须另行微调；不能把任意顺序物理视图直接混压并期待同一条件解释。

**证据边界：** 演示不属于定量表；不宣称整段每个物理视图都有独立路由或已验证长轨迹精度。

<!-- PTH:END METHOD-05 -->

<!-- PTH:BEGIN MATH-01 -->
## 3.5.1 Flow matching

原文Eq.(1)–(2)（pp.3–4）：

$$\mathcal S=\{(I_i^s,K_i^s,T_i^s)\}_{i=1}^M,\quad \mathcal Q=\{(K_j^q,T_j^q)\}_{j=1}^N.$$

$I$为RGB，$K$内参，$T$外参；$Z_0=[Z_0^s;Z_0^q]$是逐图冻结VAE的clean latent，形状按视图为$(M+N)\times16\times h\times w$（省略batch）。

$$X_\sigma=(1-\sigma)Z_0+\sigma\epsilon,\quad U=\epsilon-Z_0,\quad \epsilon\sim\mathcal N(0,I).$$
$$\mathcal L_{\rm FM}=\frac1N\sum_{j=1}^N\|V_{\theta,j}-U_j\|_2^2.$$

$\sigma$是流时间，$V_{\theta,j}$是网络输出查询速度。对线性路径求导（补充推导）：$dX_\sigma/d\sigma=-Z_0+\epsilon=U$；所以$X_\sigma=Z_0+\sigma U$。当$\sigma=0$为clean，$\sigma=1$为纯噪声；理想速度下由任意路径点求得$Z_0=X_\sigma-\sigma U$。将U替换网络V即Eq.6。推理从噪声逆向积分条件向量场；形式上$dx/d\sigma=V_\theta(x,\sigma,c)$，从1至0，简单Euler示意$x_{k+1}=x_k+(\sigma_{k+1}-\sigma_k)V$；实际论文只明确50步，不指定这里的Euler为实际采样器。FM只针对查询slot，源条件始终另行以clean latent存在。

**证据边界：** 原文未明确范数在像素与通道上的具体实现归约、采样器细节；Euler仅教学示意。

<!-- PTH:END MATH-01 -->

<!-- PTH:BEGIN MATH-02 -->
## 3.5.2 几何路由

原Eq.(3)（p.5）：
$$\mathbf x^c_{nj}=(x^c_{nj},y^c_{nj},z^c_{nj})^\top=R_jp_n+t_j,\quad u_{nj}=\pi_j(p_n)=\left(f_{x,j}\frac{x^c_{nj}}{z^c_{nj}}+c_{x,j},f_{y,j}\frac{y^c_{nj}}{z^c_{nj}}+c_{y,j}\right)^\top.$$

$p_n$为源几何点，$R_j,t_j$世界到相机变换，$f,c$焦距和主点（像素单位）；正深度后映射到latent格。若全场景点和平移同时乘正尺度s，像素投影不变（分子分母同乘s）。附录A.2置信$\tilde c_n=\operatorname{clip}(c_n/\max_m c_m,0,1)$。

原Eq.(4)：
$$w^\ell_{nq}=\tilde c_n k(u_n-q_x)k(v_n-q_y)\nu^\ell_{nq},\quad k(t)=[1-|t|]_+,$$
$$G^\ell_q=\frac{\sum_{n\in\mathcal A^\ell_q}w^\ell_{nq}f_n}{\sum_{n\in\mathcal A^\ell_q}w^\ell_{nq}+\varepsilon},\quad \nu^{front}_{nq}=\exp\{- [z_n/z^{front}_q-1]_+/\tau\}.$$

$\ell$为front/back层，$q$格中心，$u_n,v_n$连续格坐标，$\mathcal A$对应候选集合，$\varepsilon$防止除零。back候选满足$z_n>(1+\delta)z^{front}_q$，以自身最浅深度计算同类衰减。$\tau=.02,\delta=.04$。相对深度无量纲，所以全局正比例尺度改变不改变深度比或层归属。例（补充）：前层深度10，候选10.2的visibility为$e^{-1}\approx0.368$，10.5符合back资格并可能为其锚；对front衰减$e^{-2.5}\approx0.082$。若投影距某格横0.25、纵0.5，则核为0.75×0.5=0.375；归一置信0.8时前层综合权重约0.110。多点按上述权重平均，不是RGB alpha compositing，也不是沿连续射线积分。

原Eq.(5)：
$$G_j=G_j^{hard}+\mathcal R(G_j^{front},G_j^{back},S_j).$$

$S$包含前后支持、平均置信、相对深度差。零初始化refiner保证初始$\mathcal R=0$；训练后可调整而不硬性混合。遮挡反例：真实前景点若被预测到后方，z-buffer可能错选背景；次层和置信统计能提供候选线索，但并无确定性纠错保证。

**证据边界：** 置信度仅归一而非经验证概率；原文没有给出所有阈值与数值保护实现。

<!-- PTH:END MATH-02 -->

<!-- PTH:BEGIN MATH-03 -->
## 3.5.3 残差一致性

原Eq.(6)（p.6）：
$$\hat Z_{0,j}=X_{\sigma,j}-\sigma V_{\theta,j},\quad E_j=\hat Z_{0,j}-Z_{0,j}.$$

代入Eq2（原A.8亦给结论）：$E_j=Z_{0,j}+\sigma U_j-\sigma V_{\theta,j}-Z_{0,j}=-\sigma(V_{\theta,j}-U_j)$。因此它比较的是速度相对各自监督目标的偏差，而非要求速度相同。

原Eq.(7)：
$$\Delta E_\gamma=E_a(u_{\gamma,a})-E_b(u_{\gamma,b}),\quad \mathcal L_{\rm PTRC}=\frac{\sum_{\gamma\in C} w_\gamma\bar\rho_\beta(\Delta E_\gamma)}{\sum_{\gamma\in C}w_\gamma},$$
$$\rho_\beta(x)=\begin{cases}x^2/(2\beta)&|x|<\beta\\|x|-\beta/2&|x|\ge\beta.\end{cases}$$

$C$为所有双视图可见轨迹，$w_\gamma$为归一置信，$u$为最近latent格，横杠表示通道均值，$\beta=.1$。补充推导：标量导数为$x/\beta$（小误差）或$\mathrm{sign}(x)$（大误差），边界连续；乘$-\sigma$回传至a速度，b符号相反。权重归一化降低轨迹总数引起的损失尺度变化，不消除采样偏差。若$\sigma=0$，残差理论上全零；较高σ的同等速度偏差产生更大E，但原文不另外设置σ gate。

原Eq.(12)（p.22）：
$$\mathcal L_{\rm Vel}=\frac{\sum_{\gamma\in C}w_\gamma\bar\rho_\beta(V_{\theta,a}(u_{\gamma,a})-V_{\theta,b}(u_{\gamma,b}))}{\sum_{\gamma\in C}w_\gamma}.$$

同位置不同视图的$U_j=\epsilon_j-Z_{0,j}$既含不同噪声也含视点差异。即使完美$V=U$，直接速度损失也可能非零；PTRC则为零。反之若两视图相同非零误差，PTRC也为零，必须由FM抑制。此机制保存视点差异的数学意义是“不强制预测相等”，不是证明任意视点相关外观都正确恢复。

**证据边界：** 没有有效track时分母为零的工程处理未交代；置信加权不证明轨迹准确。

<!-- PTH:END MATH-03 -->

<!-- PTH:BEGIN MATH-04 -->
## 3.5.4 姿态与训练控制

原Eq.(8)（p.15）：$D_S=\max_{j,k\in S}\|c_j-c_k\|_2$，$\bar D_S=\max(D_S,10^{-6})$，$\theta_j=\arccos(\operatorname{clip}(f_t^\top f_j,-1,1))$。
$$\nu_t=\min_{j\in S}\sqrt{\|c_t-c_j\|_2^2/\bar D_S^2+\theta_j^2/\pi^2}.$$
$c$为相机中心，$f$单位前向方向，角度为弧度。最小化必须对同一个源j的组合量，不是分别取距离和角度最小值。例：距离0.1D、转角18°（0.1π），组合0.1414；若这是最小，则在DL3DV I中档、E远档（阈值各自不同）。帧索引插/外推不等同3D凸包。

原Eq.(9)（p.17）：$u_{tr}=e/(E-1)$，总损失$\mathcal L=\mathcal L_{FM}+\lambda_{eff}\mathcal L_{PTRC}$。
$$\lambda_{eff}(u)=0.1\begin{cases}0&u<.1\\(u-.1)/.2&.1\le u<.3\\1&u\ge.3.\end{cases}$$
每训练stage重新计算，u=.2时λ=.05；u=.3以后=.1。

原Eq.(10)：
$$a\sim\begin{cases}0&\Pr=.10\\\mathcal U(.2,.8)&\Pr=.20\\1&\Pr=.70.\end{cases}$$
同一a乘所有Ω条件；补充：期望a=.8，但这不等同每次固定0.8，随机暴露于不同几何强度是训练目的。

原Eq.(11)：$u_{den}=k/(K-1)$，$K=50$。
$$V_{GeoCFG}=V_{ref}+s_g^{eff}(u_{den})(V_G-V_{ref}),$$
$$s_g^{eff}(u)=1+(s_g-1)\cos^2\left[\frac\pi2\min\left(\frac{u}{.6},1\right)\right],\quad s_g=2.$$
初始2，u=.3时1.5，u≥.6时1；代入1即完整条件V_G。也可写为$V_G+(s^{eff}-1)(V_G-V_{ref})$，清楚显示额外方向逐渐消失。此处u是去噪进度，不是流时间σ或训练epoch进度。

**证据边界：** 姿态分数不包含遮挡、纹理、介质等；CFG方向不是可证明的几何正确方向。

<!-- PTH:END MATH-04 -->

<!-- PTH:BEGIN EXP-01 -->
## 3.6.1 训练与评估协议

§4.1、A.1–2、A.11：DL3DV-10K 960P的1K子集去除1不可用及19与140场景测试集重叠，剩980。147 epochs半分辨率192×336，60 epochs全分辨率480×832，BF16、global batch8；Wan LR分别1e−5、5e−6，新模块1e−4。冻结VAE、Ω。每场景16个可复现源bundle，13全局六段各取1，3局部窗口同样分段；源顺序打乱。目标在线排除源，局部优先窗口；第一阶段N集合依次{1,2,4}→{2,4,8}→{4,8,12}→{4,8,12,16}，第二阶段{4,8}→{4,8,12}→{4,8,12,16}。N≥8按跨度均匀时间采样，小集合50%采用轨迹、否则无放回均匀采样。无显式姿态难度平衡。仅源决定gauge。扩规模版clean9510、100K有效半分辨率步+6K全分辨率适配。

主测试：DL3DV 6188目标、140场景，Mip-NeRF360零样本；每目标独立6→1，50步，原生输出缩放或中心裁480×480；PSNR、SSIM、LPIPS-AlexNet、DreamSim。姿态诊断独立192目标，六bin各32不同场景，109场景可跨bin重复但scene-target唯一；固定噪声、无反馈/rollout；报告候选池5549 I、638 E（和6187，原文未解释与总6188差1，不合并推断）。I阈值.1165/.2147，E .0560/.1272。半分辨率诊断192×336、192×192中心裁剪，LPIPS-VGG。表9/10的Mip协议单独标注。几何实验52场景六源八查询，共416目标，联合生成与独立生成比较；演示80帧另微调。

**证据边界：** 不能将训练场景数解释为全部预训练数据；候选池差1按原文记录。

<!-- PTH:END EXP-01 -->

<!-- PTH:BEGIN EXP-02 -->
## 3.6.2 主结果与姿态难度

表1主结果原版DL3DV 18.104/0.508/.222/.062，FrameCrafter17.180/.445/.223/.066；规模版19.283/.569/.198/.054。Mip原版16.296/.317/.315/.091，规模版17.217/.362/.283/.080。原版与同骨干相比优势明确，但D SSIM、M PSNR及SSIM并非均最佳；规模版D四项第一，M SSIM仍第二。原文§4.2的原始版结论与v2增加的scaled行须分开读。

表2/9/10：近中远下所有方法普遍退化；D原版五组PSNR最佳、I近例外；M原版四组最佳。规模版D全胜，M E近与SEVA并列，其余领先；LPIPS原版已两数据集全组领先，规模版更低。Fig.4/5显示细部结构与语义一致性较好，Fig.11显示共享色标下像素误差较集中；并非所有区域无误差。Fig.6展示轨迹关系与姿态新颖度区别。

**证据边界：** 未控制不同方法训练预算；仅可见GT图像支持图像对齐结论。

<!-- PTH:END EXP-02 -->

<!-- PTH:BEGIN EXP-03 -->
## 3.6.3 组件消融与诊断

表3无CFG full=19.024；去Ω全依赖−1.730 dB，RGB替代−0.471，去PTRC−0.438，去分层−0.129，去正则−0.302。CFG同checkpoint推理+0.115。表5公开固定参数，不是敏感性结果。表8同半分辨率设置：冻结L11 18.734优于L23 18.581，联合优化L23 17.934；说明中层信息与冻结策略有效，但选择不是最优搜索。表11同轨迹同权重，残差替代直接速度使全192目标PSNR+0.812，六组各指标均改善，是最直接机制证据。表12全量微调18.5807对rank32 LoRA17.2563，代价可能更大，未报告完整成本曲线。

**证据边界：** 不能把多个训练消融差值相加，亦不能把全分辨率表3与半分辨率表11直接横比。

<!-- PTH:END EXP-03 -->

<!-- PTH:BEGIN EXP-04 -->
## 3.6.4 多视图一致性

表4与Fig.10：同六源八目标、52场景，用冻结Ω和Pi3分别重建，再与相同重建器对GT视图的输出比对；对齐、过滤、采样相同。Ω Chamfer .0733 对FrameCrafter .1000，Pi3 .1174 对 .1404；F-score提升一致。Pi3从未用于训练或条件，降低同源评估偏差。表7同checkpoint的joint/independent，使用相同目标和每视图噪声，联合PSNR17.6401对17.1784，SSIM .4787对.4453，LPIPS .3568对.3744；Chamfer .0483对.0741，F@1% .3438对.2432，F@2% .5341对.4047。全部六指标场景bootstrap CI排除0。表4与表7不是同一完整比较设置，不可替换数值。

**证据边界：** GT-view reconstruction不是GT扫描，独立探针也不排除共同数据偏差；不能把代理距离直接换算米。

<!-- PTH:END EXP-04 -->

<!-- PTH:BEGIN EXP-05 -->
## 3.6.5 规模、稀疏性及统计

表6/Fig.7/8：clean-10K由18K到100K半分辨率PSNR18.7861→20.2780，SSIM .5272→.6166，LPIPS .2793→.2335；60K短期退化，96K后收益小，训练损失并非测试表现单调代理。A.3原18K fixed-compute与更多场景/优化预算讨论说明模型欠训练；不能仅凭最终scaled对原版识别数据量和训练步的各自因果效应。A.9另做140场景6188目标本地配对：场景均值PSNR+0.7893 dB，92.14%场景获胜，bootstrap 95% CI [.7003,.8769]；SSIM+.0330、LPIPS-Alex−.0365、DreamSim−.0118。此为配对统计，不应强行等于表1跨方法公开行差值。

表13六源训练checkpoint不适配改用3/4源，姿态选同嵌套子集与噪声；三源17.7372对16.9559，四源18.4380对17.8545，其余指标同方向。减少观测会降低绝对质量，几何路由相对优势在这里仍成立。

**证据边界：** 未报告任意源选取或单源极端稀疏、计算成本全曲线；训练预算与数据数量混杂。

<!-- PTH:END EXP-05 -->

<!-- PTH:BEGIN SYNTHESIS-01 -->
## 4.1 主张—证据矩阵

| 主张 | 证据 | 评价 |
|---|---|---|
| VGR空间对齐支持生成 | Fig2/3、表3去Ω及RGB替代 | 强支持组合设计；全Ω去除同时改变多个模块 |
| 分层残差保留不确定证据 | 表3去分层−0.129dB | 有受控增益；未独立量化遮挡边缘准确率 |
| PTRC不强制视点latent相同 | Eq6/7/12、表11 | 机制数学成立，对该直接速度基线全六组优势 |
| 几何条件弱化减轻依赖 | 表3去正则−.302 | 支持整体效果；未按几何误差校准分组 |
| 目标几何CFG进一步改善 | 同checkpoint表3+.115 | 小但正向；未给每例不确定度 |
| 共同去噪增强一致性 | 表7六指标、表4双重建器 | 支持代理一致性，非绝对场景真值 |
| 泛化及扩展 | 表1 Mip零样本、表6/13 | 有实证，范围限定于所测数据/输入 |

继承：Wan视频先验、冻结VAE和Ω、射线表示、线性FM、注意力与CFG。方法新内容是查询几何特征路由的硬锚+分层残差、轨迹误差耦合、具体几何正则与目标参考引导；理论部分是目标定义与相对误差理由，不是可辨识性新定理。实验贡献包括姿态诊断与双探针。

**证据边界：** 新颖性评价只限本文描述及证据，未完成跨论文优先权调查。

<!-- PTH:END SYNTHESIS-01 -->

<!-- PTH:BEGIN SYNTHESIS-02 -->
## 4.2 方法局限与未解问题

论文A.12明确未来方向：自适应多层Ω特征融合；更稳健的参数高效/分阶段几何适配；多遮挡层和校准可见性；节省内存的joint及跨chunk一致性；扩展到动态时空对应。独立分析：VGR依赖源几何，置信未必在域外校准；遮挡层只描述已见预测点，不可补出真实未见内容。PTRC受错轨迹、粗latent格及共同偏差影响；FM仍需GT训练，不能认为测试时直接优化真实误差。测试相机已知，非未知位姿场景；静态场景实验不能外推动态。图像和代理重建质量不等于绝对几何。80帧时间压缩演示的条件仅射线全窗口，几何endpoint；未量化其效果。计算量受M+N、分辨率与双分支CFG制约，实际复杂度/速度未全面测量。

**证据边界：** 未来方案与独立分析均未验证，禁止作为作者实验结论。

<!-- PTH:END SYNTHESIS-02 -->

<!-- PTH:BEGIN SYNTHESIS-03 -->
## 4.3 覆盖审计与来源

实际原件为arXiv:2609.33253v2，24页。正文pp.1–10；参考文献pp.10–14（正文结论之后）；附录pp.15–24。逐项涵盖Fig.1–11（正文1–5、附录6–11）、Table1–13（正文1–4、附录5–13）、Eq1–12；A.1采样和姿态、A.2固定超参、A.3规模、A.4联合独立、A.5特征层、A.6难度全表、A.7点云、A.8残差、A.9适配与统计、A.10少源、A.11VAE、A.12未来方向；无编号算法。完整参考文献仅按当前论文的引用关系分析，不独立精读所有引用论文。PDF为最终权威，辅助HTML缺字体、Source无独立版本印记，未用其覆盖PDF。11图全部实用保真验收，5精确、6带记录；JPEG为独立展示副本。补充数学、推断和评价已按节标记。

**证据边界：** 本报告为生产自检，不是独立Review；本单元为覆盖审计，不新增机制。
本单元不新增数学推导。

<!-- PTH:END SYNTHESIS-03 -->



---

# 第二部分：学习与研究笔记

# VGGT-Diff 学习与研究笔记

> 精读报告 SHA-256：cd4aa896b657740af7cc0740f23fa2be5b13a9e0a3bfe6b23a554e803b7a6fe8
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：几何证据与生成先验

### 1. 观测约束与补全不是同一种信息

对应精读 §3.1–3.2。把图像生成看成条件分布 $p(I_q|I_s,c_s,c_q)$，并非唯一的物理逆映射。设两个物理场景在所有源相机的投影完全一致，但一个有被遮挡的红色背面，另一个有蓝色背面；新增查询暴露背面时，其真实图像不同。无论训练多好，给定源条件无法辨别这一差异。学习到的先验可以给出概率上合理的候选，却不增加真实测量。VGR处理源证据如何抵达目标坐标，扩散处理有条件的补全；两者不可互换。几何本身也由源图推断，所以是带误差的证据模型，不是额外实测真值。

### 2. 路由与渲染的区别

对应 §3.4.2、3.5.2。给定点特征 $f_n$ 与投影权重 $w_n$，归一加权聚合保留局部潜在特征；体渲染通常沿射线根据透射率和发射/吸收规律积分。VGR不预测完整辐射场，也不要求特征是辐射值。即便 $f_n$ 是RGB，该运算也不能替代物理体积积分。硬锚是对可见表面的局部选择；后层只是待解释的候选证据。假设两源点深度近似相等却处于遮挡边界，硬选择可能对几何微扰不连续，双线性支持与残差模块可减弱这一问题；但当真实表面未被观测时，任意层数不能提供它的真实外观。

### 3. PTRC可以看成轨迹图上的误差正则（补充推导）

对应 §3.5.3。令一条轨迹在k个视图的标量残差为$e_i$，并考虑置信相等、所有差值在Smooth-L1二次区间的简化情况。恒等式

$$\sum_{i<j}(e_i-e_j)^2=k\sum_i(e_i-\bar e)^2,\quad \bar e=\frac1k\sum_i e_i$$

可由展开平方得到：左边为$k\sum_i e_i^2-(\sum_i e_i)^2$。因此在完整等权轨迹图的局部二次近似下，PTRC抑制误差跨视图方差，但不约束均值。若所有误差皆为常量b，正则为零；FM会进一步惩罚非零b。一般置信加权和不完整可见轨迹对应图边差分；断开的轨迹图只在各连通分量内建立约束。这个视角提示可见边覆盖与置信可靠性的重要性，而不是简单增加query数量。论文未提出图拉普拉斯定理；此处只是由Eq.7得到的解释性数学扩展。

### 4. 条件正则与CFG的反事实

对应 §3.4.4、3.5.4。训练随机将几何条件整体缩放，而保持RGB及相机，促使模型在证据不足时仍可工作。推理查询-only参考使差分更集中于目标空间路由；若参考同时删除源几何和全局上下文，差分会混合不同几何作用。若投影错误，增强其模型响应可能增加错位而不是减少；固定引导强度非可靠性自适应。可在合成场景中按真实投影误差分层比较两种引导，不能把几何置信视为校准概率。

### 5. 信息槽与物理视图

对应 §3.4.5。独立VAE时每个latent槽与相机一一对应；压缩后四帧窗口共同编码，不能把一个槽误当成一幅真实图。即使射线adapter包含四帧姿态，路由仍仅endpoint；其余物理视图依赖时序先验。比较两布局需控制模型适配训练、轨迹长度及相机间隔，不能把86→27槽数直接换算实际显存或速度。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Observation & Prior（观测约束与生成补全）
- **Keywords:** Sparse-view NVS; Conditional Generation; Observability; Geometry Conditioning
- **Highlight:** 本文把冻结几何模型提供的源证据路由至查询空间，由预训练扩散模型补全无支持区域，而非把预测几何作为完整场景。
- **Evidence:** §1、§3.2、Fig.2/3、Table 3；精读§3.1–3.4。
- **Connection Points:** 未观测区域定义；确定性表示与条件生成；观测约束如何注入模型。
- **Open Questions:** 如何在未观测区统计合理性和已观测区真实性之间建立独立评价？

### Topic: Geometry Inference（特征到查询空间）
- **Keywords:** Feature Routing; Z-buffer; Bilinear Splatting; Relative-depth Visibility
- **Highlight:** 硬锚以近前景预测点建立主条件，双层软聚合及支持统计只通过学习残差修正，避免硬性混合候选表面。
- **Evidence:** §3.2 Eq.3–5、A.2、Fig.3、Table 3。
- **Connection Points:** 显式投影与隐式注意力；深度层数；置信校准；可见性建模。
- **Open Questions:** 预测几何误差增大时何种路由仍保有真实源证据？

### Topic: Training & Supervision（轨迹误差耦合）
- **Keywords:** Flow Matching; Point-track Residual Consistency; Robust Loss; View-dependent Appearance
- **Highlight:** PTRC在同3D点上匹配各视图预测clean相对各自GT的残差；对应视图无需共享真实latent或速度。
- **Evidence:** Eq.6/7/12、A.8、Table 11。
- **Connection Points:** 预测一致性与误差一致性；轨迹来源；遮挡筛选；训练期监督与推理约束。
- **Open Questions:** 不完整、误配或带系统偏差的轨迹如何影响联合预测？

### Topic: Uncertainty（几何条件强度）
- **Keywords:** Geometry Dropout; Classifier-free Guidance; Confidence; Domain Shift
- **Highlight:** 训练共享随机scale弱化全部Ω条件，而正式CFG参考只删除查询稠密几何，保留源条件。
- **Evidence:** §3.4、Eq.10–11、Table 3/5。
- **Connection Points:** 条件随机化；引导分支选择；全局与局部可靠性；域外几何误差。
- **Open Questions:** 固定几何引导能否由真实误差校准的可靠性控制替代？

### Topic: Efficiency & Scalability（多视图slot与时序压缩）
- **Keywords:** Independent-view Latent; Causal Video VAE; Plücker Adapter; Joint Denoising
- **Highlight:** 定量采用逐视图独立latent；连续演示另微调目标时序压缩，四帧窗口射线均输入而几何仅endpoint。
- **Evidence:** A.11、Table 7、A.12。
- **Connection Points:** 每相机对应slot；条件对齐；联合注意力与序列压缩；跨chunk一致性。
- **Open Questions:** 长轨迹压缩丢失的几何监督能否由更细粒度条件补偿？

### Topic: Generalization（姿态与源数量）
- **Keywords:** Pose Novelty; Interpolation; Extrapolation; Reduced-view Inference; Zero-shot Transfer
- **Highlight:** 本文把帧序关系和归一化姿态距离分开评估，并检验六源训练模型对三/四源及Mip-NeRF360的泛化。
- **Evidence:** Eq.8、Fig.6、Table 1/2/9/10/13。
- **Connection Points:** 输入覆盖度；姿态新颖度；域外测试；样本筛选与噪声控制。
- **Open Questions:** 姿态新颖度是否足以预测实际可见性、纹理与几何不确定性？


## RESEARCH-DIRECTION：稀疏观测视觉的研究定位

本笔记按当前论文的一般研究方向整理，不预设新的专属研究任务或私密研究方案。

| 层级 | 本文定位 | 可开放的更一般方向 |
|---|---|---|
| Field | 3D vision / generative vision | 受限观测下的视觉推断 |
| Task | novel view synthesis | 图像恢复、深度/几何推断、NVS等 |
| Task setting | 稀疏已知姿态RGB，静态场景 | 任意源数、观测退化或不完整位姿 |
| Research problem | 如何兼顾可靠源证据与未见区域补全 | 约束不足时如何辨别可靠推断与先验补全 |
| Method | VGR、PTRC、条件正则及CFG | 证据路由、可靠性估计、跨观测约束或混合推断 |

关键是不要把“使用扩散模型”本身当研究问题。本文给出的证据是，几何支持可以提高图像与几何代理质量（表3/4/11），但不能消除稀疏观测下的多解性。更一般的研究方向可先确定失败现象及根因，再决定使用确定性回归还是生成式推断。

### 领域迁移示例：水下稀疏观测（研究假设，不是本文实验）

在参与介质中，同一点跨视角的成像受传播距离及介质变化影响。简化示意 I_i=A_i J_i+B_i，其中A为透射衰减、B为散射贡献、J为清晰辐射；这不是VGGT-Diff的成像公式。若源RGB特征反映介质变化，正确几何投影也可能给出不同特征响应，错误几何与介质差异难以分离。相机姿态新颖度也不能描述浑浊度、光照、有效可见距离。潜在可迁移的是“将观测证据按可靠性路由”“避免强制同点预测相同”而不是直接复制冻结Ω及Wan权重。必须独立检验水下几何模型置信、介质条件和真实几何，避免先验生成清晰但错误的结构。

任务边界：本文只给定相机生成RGB，未恢复介质或估计相机；迁移时需要先规定目标是图像合成、清晰恢复、几何还是介质推断，并定义相应真值及成功标准。


## RESEARCH-PROBLEMS：候选科学问题与可证伪实验

以下均为从本文证据出发的研究候选，不是已验证成果或原创性判定。

### RP-A：如何区分可靠几何条件与错误投影？

现象：硬锚可能受遮挡和几何误差影响；表3弱化条件与分层带来增益，但没有按真实投影误差分组。科学未知：基础模型置信是否能预测源证据在查询域中的真实性？假设：校准过的可见性/投影可靠性可在固定计算预算下减少错误几何引导，同时不破坏正确证据。构造已知几何和位姿的静态合成数据，对点深度、置信和遮挡进行独立受控扰动；固定RGB、查询、seed及模型，比固定scale、无引导、真实几何oracle及可靠性自适应。按真实可见/不可见、投影误差幅度分组，报告RGB误差、对应重投影误差、置信校准曲线与不确定度覆盖率。成功：高误差组降低结构错位、低误差组不退化，且跨场景置信有稳定区分力；失败：可靠性只跟纹理强度相关、不能识别几何错误，或收益只来自更多计算。优先级高：直接对应本文未校准置信的边界；但需真值和独立验证，尚需查阅不确定性相关研究。

### RP-B：联合生成的跨视图误差能否由更可靠的轨迹约束控制？

现象：表11支持残差而非速度，表7支持联合上下文，但轨迹误配/共同偏差未单独测试。科学未知：在相同逐视图重建误差下，轨迹覆盖与误配率如何决定跨视图结构漂移？假设：误差图连通性与可信边密度共同影响一致性；误配率升高后更多轨迹可能反而有害。对合成场景的GT点轨迹设置覆盖率、错误边比例、遮挡筛选三个独立因素，固定FM、训练步、目标数与噪声；比较无PTRC、直接速度、原PTRC、GT轨迹PTRC，另测置信阈值。评价每视图PSNR/LPIPS、真实三维重投影/深度一致性、误差方差与均值、计算开销。成功：存在可复现的可靠轨迹收益及误配导致的阈值效应；反证：控制单视图质量后，轨迹数量和准确率对真实几何一致性均无稳定作用。优先级高，且需区分代理指标与扫描/合成真值。

### RP-C：姿态稀疏程度与真实信息缺失如何区分？

现象：Eq.8仅相机中心和前向角，表2/9/10显示远组难但并非直接量化可见性；表13源数变少时质量下降。未知：相同姿态距离下，不同源覆盖、纹理、遮挡或介质是否产生不同的推断歧义？假设：查询真实可见表面在源集中的支持比例及信噪/传输可靠性，比单一pose novelty更能解释失效。固定源数和姿态novelty，改变场景遮挡及（单独实验）介质，比较姿态分层与真实可见度分层对错误的预测能力；固定场景/seed对多种源集合重复测试。用覆盖比例、校准的几何误差、图像恢复误差及未观测区多样性区分现象。成功：在留出场景上新增信息指标显著改善失败预测；失败：指标不能跨场景稳定泛化或只是反映数据选择。优先级中高，先建立诊断而非直接指定生成模型；水下拓展需物理GT及可靠成像模型，未在本文验证。

上述候选相互关联但问题不同：A是条件可靠性，B是误差耦合，C是观测难度定义。需先进行独立文献调研与可行性评估，再决定是否形成正式研究问题。


## INTERNALIZATION：知识内化与自检

一句话理解：这不是让几何网络直接生成新视图，而是让它告诉扩散网络源证据在查询平面上的可能位置，再借助训练时的同点误差约束使多个查询更一致。

技术地图：源图/相机→冻结VAE和Ω；Ω→视觉特征、点、置信→VGR→查询对齐条件；相机→Plücker→空间重排；训练GT→clean latent→加噪及速度目标；三条件+扩散状态→共享DiT→速度→query FM与轨迹残差损失；测试噪声→逆向采样→独立解码。训练期的GT与PTRC不进入推理，几何条件在两阶段均由源证据提供。

五个理解自检：

1. VGR是否等于基于点云渲染？否，输出是条件特征，含硬锚和可学习残差；真实图像由扩散生成。
2. PTRC为何不直接匹配速度？因为 U=ε−Z0 不同视图具有不同噪声/clean值。正确预测也可能不同。Eq.6的残差对其自身目标定义。
3. PTRC为零能否保证预测无误？不能；相同的非零误差满足差分约束，FM仍不可缺失。
4. 80帧演示能否解释表1？不能；表1采用独立view latent，压缩演示另微调，条件对齐也有差异。
5. 双重建器指标能否证明真实3D？不能；GT-view重建是模型相关参考，而非绝对几何真值。

阅读后可迁移的思想：先辨认观测支持是否存在，再用模型估计其可靠性；学习先验只补信息缺口，避免掩盖测量错误；将监督目标的视点相关成分与真正应一致的误差分离。未解决的是实际错误校准、未观测内容的可辨识性和长轨迹压缩条件的充分性。


## GLOSSARY：术语与概念关系

| English | 中文 | 本文技术含义及相近概念区别 |
|---|---|---|
| Novel View Synthesis (NVS) | 新视角合成 | 从源观察和给定目标相机预测目标RGB；不必输出显式3D |
| Pose novelty | 姿态新颖度 | Eq.8最邻近源的归一化中心距与前向角组合；不是遮挡程度 |
| Camera gauge | 相机规约 | 仅源相机决定坐标原点、旋转和尺度；不同目标不改框架 |
| Plücker ray | 普吕克射线 | 六维射线表示，表达像素相机条件；不包含完整场景对应 |
| PixelUnshuffle | 像素反重排 | 空间8×8重排为通道，6→384，不是空间平均 |
| VAE latent | 变分自编码器潜变量 | 冻结Wan VAE的16通道编码；独立视图与时间压缩含义不同 |
| Flow matching | 流匹配 | Eq.2学习线性插值路径速度，不是直接预测RGB |
| DiT | 扩散Transformer | Wan初始化的40块模型，共享处理视图槽 |
| Visual Geometry Router | 视觉几何路由器 | Eq.3–5用3D点/置信投影特征，非完整3D渲染 |
| Hard anchor | 硬锚 | 最近格和近前层深度选择的主要条件 |
| Layered residual refinement | 分层残差细化 | 前/后软层及统计提供可学习修正，非强制混合 |
| Point track | 三维点轨迹 | 同一预测3D点投影到多个可见query的对应 |
| PTRC | 点轨迹残差一致性 | Eq.7对各自clean预测误差的跨视图差分正则 |
| Smooth-L1 | 平滑L1 | 小差值二次、大差值线性，β为转折阈值 |
| Geometry-condition regularization | 几何条件正则化 | Eq.10按样本全几何缩放，不等于文本dropout |
| Geometry-Prior CFG | 几何先验无分类器引导 | Eq.11按完整和仅删查询几何的速度差进行引导 |
| Independent-view layout | 独立视图布局 | 每个物理视图对应一个latent槽；本文定量协议 |
| Causal temporal compression | 因果时间压缩 | A.11每四帧形成一压缩窗口，演示另微调 |
| Chamfer-L1 / F-score | 点云距离 / 阈值F分数 | 归一化的GT-view重建代理评价，非实测扫描精度 |
| LPIPS / DreamSim | 感知距离 | 使用学习特征衡量图像差异，LPIPS骨干不同不能直接比较 |


## SOURCES：版本、来源与边界

- 原始论文：arXiv:2609.33253v2（2026-10-14），24页；官方链接 https://arxiv.org/abs/2609.33253v2 。本文主要证据是用户提供的同版PDF；HTML与LaTeX Source仅辅助，缺字体和独立Source版本印记已在精读记录。
- 冻结精读报告 SHA-256：`cd4aa896b657740af7cc0740f23fa2be5b13a9e0a3bfe6b23a554e803b7a6fe8`。本笔记的核心概念、方法和定量引用均可沿上述章节、公式和表格定位。
- 来源范围：仅当前论文及其精读报告。相关工作按作者描述理解；尚未独立核查各引用论文、代码实现、运行速度或公开模型效果。跨文献比较延期，索引只是待关联线索。
- 证据区分：原文机制/数值、补充数学推导（轨迹图方差）、独立评价和研究假设（RP-A/B/C、水下迁移）分别标明；不将后两类当作者结论。
- 未证实事项：置信校准、实际几何真值、未见区域真实外观、长序列定量及域外介质鲁棒性。无Hexo实际构建或部署保证。笔记为可公开一般分析，不包含私密实验路径和方案。



