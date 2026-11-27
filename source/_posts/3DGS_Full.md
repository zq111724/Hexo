---
title: "3D Gaussian Splatting for Real-Time Radiance Field Rendering（完整精读与研究笔记）"
date: 2026-11-26 00:00:02
index_img: "/3DGS/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/3DGS/)

# 第一部分：论文精读报告

# 3DGS 论文完整精读报告

**原文：** 3D Gaussian Splatting for Real-Time Radiance Field Rendering

**作者：** Bernhard Kerbl, Georgios Kopanas, Thomas Leimkühler, George Drettakis

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2308.04079v1（2023-08-08）；缓存记录 v1（2023-08-08）；最新版未能确认。核验日期：2026-11-26。

**官方资源：** [arXiv](https://arxiv.org/abs/2308.04079) | [PDF](https://arxiv.org/pdf/2308.04079v1) | [Project Page](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/) | [Code](https://github.com/graphdeco-inria/gaussian-splatting)

<!-- PAPER-METADATA END -->

**精读依据：** 用户提交 arXiv:2308.04079v1 PDF（14页）；HTML仅辅助。页码以PDF物理页为准。

<!-- PTH:BEGIN INIT -->
## 0. 自动初始化与证据范围

**来源与目录**

论文作者为 Bernhard Kerbl、Georgios Kopanas（两人为共同第一作者）、Thomas Leimkühler、George Drettakis。上传的 PDF 首页标记 arXiv:2308.04079v1，2023-08-08；ACM 引用格式给出 ACM Transactions on Graphics 42(4), Article 1, August 2023，DOI 10.1145/3592433。论文 PDF 物理页共 14 页，正文从摘要、引言开始，章节依次为：

| 章节 | 内容 |
|---|---|
| 1 Introduction | 研究目标、三项贡献 |
| 2 Related Work | 重建、神经渲染、辐射场和点基渲染 |
| 3 Overview | 表示与优化、渲染总体思路 |
| 4 Differentiable 3D Gaussian Splatting | 高斯表示、协方差、投影 |
| 5 Optimization with Adaptive Density Control of 3D Gaussians | 优化、初始化、增密与剪枝 |
| 6 Fast Differentiable Rasterizer for Gaussians | 分块光栅化、前向与反向 |
| 7 Implementation, Results and Evaluation | 实现、比较、消融、局限 |
| 8 Discussion and Conclusions | 结论及未来工作 |
| Appendix A–D | 协方差导数、优化与增密算法、光栅器及数值稳定性、逐场景结果 |

正文 Figure 1–12、Table 1–3；附录 Table 4–9、Algorithm 1–2。共12幅图、9张表和2个算法。所有 PDF 页均实际可读；HTML 辅助文件版本一致，但缺一个数学显示字体，另有一个行内图标引用问题；未发现科学图片资源缺失。B2 从同一 PDF 区域提取保真 PDF 与 JPEG 展示副本，12幅均视觉审查通过，其中 Figure 2–5 属非像素级精确但带记录实用保真通过。Figure 8、9 初审有渲染指标复现差异，独立提取后通过全局复验。详见工作区 `figure-map.json` 与 `checks/B2-final.json`。JPEG 为展示副本而非原始图像流；Hexo/聊天客户端实际显示未验证。

**证据规范**

- 【原文事实】定位到 v1 PDF 的陈述、公式、图表或算法；所有页码均指 PDF 物理页。
- 【背景知识】为理解方法补充的通用数学或计算机图形学概念，不表示论文自行提出。
- 【合理推断】基于原文的逻辑推论或补充推导，明确假设。
- 【独立评价】对证据强度、技术局限与替代解释的审查。
- 【尚未证实】本篇论文未提供足够证据的事实或实现细节。

不读取其他论文或代码实现以填补本文缺口；官方链接仅用于论文身份、版本和资源核对。官方缓存记录为 v1，实时最新版未确认。原始 PDF 和 HTML 保持不变。

本单元是资源与目录初始化，不涉及算法机制解释。
本单元是资源与目录初始化，不涉及数学推导。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1.1 标题（Title）

**译名与术语**

3D Gaussian Splatting for Real-Time Radiance Field Rendering：**面向实时辐射场渲染的三维高斯泼溅（3DGS）**。

- 3D Gaussian（三维高斯）：【背景知识】具有三维中心与协方差的空间核；各向异性协方差可让核沿不同方向伸展。它不是单纯三维点，也不必是归一化概率密度。【原文事实】§4 使用非归一化高斯核及可优化的不透明度。
- Splatting（泼溅/溅射）：【背景知识】把空间基元投影至像平面，覆盖一定像素区域，再累积其贡献。与按像素发射射线、逐点查询连续场相比，计算组织方式从“像素询问空间”转向“空间基元贡献给像素”。这不是说所有 splatting 都无需可见性处理；本文的排序和 alpha 合成恰恰针对可见性。
- Radiance Field（辐射场）：【背景知识】通常刻画空间位置与观察方向相关的外观，以及决定光线可见程度的体密度或相应表示。本文不是传统仅存几何和纹理的表面网格，而是由一组高斯基元提供空间覆盖、alpha 和方向相关颜色（§3–4）。
- Real-Time Rendering（实时渲染）：关注给定相机视角的图像生成速度，不等同于实时训练、实时建图或动态场景重建。摘要给出的具体目标为1080p、至少30 fps，而不是从标题本身可读出该阈值。

**任务与边界**

标题直接表明研究的是一种以三维高斯为基础、适用于辐射场实时渲染的技术；它将核心基元、渲染策略与性能目标同时写入标题。标题没有明确输入、监督方式、初始化、是否需要相机标定、动态场景能力或真实几何精度。结合§1–3才可确认本篇主任务是从多张已标定图像优化静态场景表示，并在新视角渲染RGB图像；并非直接输出可供测量的三维表面。三维高斯如何变成二维椭圆、如何合成颜色、如何训练与增密，需后续公式及算法解释。

本单元只建立标题语义与研究任务，不展开公式推导。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 摘要（Abstract）

**问题链**（PDF p.1）

【原文事实】辐射场方法使多张照片或视频中的场景新视角合成获得显著进展。但高质量方法通常依赖训练和渲染昂贵的神经网络；较快方法需要在速度与质量间折衷。作者特意强调无界、完整场景（而非孤立物体）与1080p分辨率：截至论文写作时，作者认为当时方法无法在此设定下达到实时显示。

【背景知识】渲染速度与训练速度不同。对连续场，每个像素需要沿射线查询多个位置：即使某位置为空，采样也可能发生；网络查询或特征插值还带来额外开销。图像分辨率越高，像素和射线越多。无界场景也不能简单用固定小包围盒排除远背景。这解释摘要为何将完整场景、速度及质量绑定，而不是仅在单个小物体上报告帧率。具体实现和加速原因在§2–6展开。

**三项方案及主张**

1. 表示：从相机标定阶段产生的稀疏点开始，用三维高斯表示场景。每个基元在空间有连续的影响范围，且可投影成二维覆盖区域；作者希望保留有利于优化的体辐射场性质，同时避免空空间上的无效工作。这里的稀疏点是初始化而不是最终表面或最终点数。与密集网格不同，基元数量和分布由后续优化改变。
2. 优化：交替优化高斯参数与自适应密度控制，特别优化各向异性协方差，使基元可改变位置、形状和分布。位置/形状决定覆盖，颜色与不透明度决定图像贡献；仅移动一组固定数量的点，无法保证修复覆盖不足的细节。后续 §5 的 clone、split、prune 负责增删基元，与梯度更新互补。
3. 渲染：开发快速且考虑可见性的渲染算法，支持各向异性 splat，同时加速训练与实时显示。其因果链是前向投影、按深度排序、分块像素 alpha 合成，反向沿合成顺序回传损失，使渲染器既是推理工具又是训练的可微接口；仅有高斯表示本身还不足以保证实时。

【原文事实】摘要主张在多个已有数据集上达到当时先进视觉质量、具有竞争力的训练时间，并实现1080p至少30 fps的新视角合成。摘要未给出逐数据集数值。需结合§7、Table 1–9以及Figure 5–12判断主张在哪些场景和指标上成立，不能将“state-of-the-art”理解为所有指标都优于所有基线。

【一句话概括】3DGS通过可优化的各向异性三维高斯集合、密度控制和可微分块光栅化，尝试在完整静态场景的新视角合成中同时达到高质量、较快训练和实时渲染。

本单元解释摘要的研究逻辑，具体方程及推导在数学单元展开。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 核心贡献（Contributions）

**三项贡献**（§1，PDF p.2，作者明确列出的三项）

| 贡献 | 技术性质 | 对应困难 | 实现核心 |
|---|---|---|---|
| 各向异性3D高斯作为高质量、无规则的辐射场表示 | 表示方法创新 | 连续场射线查询开销大；固定点缺乏可调体覆盖 | 高斯均值、协方差、opacity、方向相关SH颜色；投影并alpha合成 |
| 高斯参数优化与自适应密度控制交替 | 优化方法创新 | 初始化稀疏、不同尺度结构的表示能力不均衡 | 图像监督的梯度更新，clone/split、prune及opacity reset |
| 快速、可微、考虑可见性的GPU渲染 | 渲染算法及系统方法创新 | 大量高斯的投影、遮挡合成和梯度回传难以实时 | tile实例化、GPU排序、前向alpha合成、逆向遍历 |

第一项决定“用什么表示场景”，第二项决定“如何从照片得到表示”，第三项决定“怎样高效把表示变成图像，并回传训练信号”。三者不是彼此独立的性能模块：没有投影与alpha合成，表示不能用于图像监督；没有自适应增密，稀疏SfM初始化未必覆盖细节；若渲染器耗时，高斯本身也不会自动带来实时渲染。

【原文事实】作者同时指出在多种场景中获得高质量及实时显示，并称实景中优化产生约100万至500万个高斯。这些是方法成果和实验证据，而非第四项作者明确编号的贡献。

**论证路径**

- §3–4及Eq. 1–6解释点基alpha合成与离散体渲染形式的联系、可优化协方差及投影；Figure 3展示各向异性形状，Figure 10与Table 3检验关闭各向异性的影响。
- §5和Appendix B/Algorithm 1给出初始化、梯度优化和密度控制；Figure 4为设计示意，Figure 7、8与Table 3分别检验初始化和增密分支。
- §6和Appendix C/Algorithm 2给出GPU算法及稳定处理；Figure 9与Table 3测试限制梯度传播的影响，Table 1提供整体渲染/训练性能比较。整体速度不能仅归因于某个单独算子，因为还同时改变表示和优化方法。

【独立评价的边界】此处先准确复原贡献主张。已有技术（高斯核、SH、alpha blending、SfM、GPU排序）的使用不等于本文首次提出这些单项技术；新颖性应考察其适配、组合和新算法，而不是把所有组成部分均称为原创。本文没有单独列出“发现新的物理规律”或“证明连续体渲染与3DGS严格物理等价”的贡献。

本单元整理贡献与证据关系，不单独展开数学推导。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 结论（Conclusion）

**成果与适用范围**（§8，PDF p.11）

【原文事实】作者认为本文首次实现了在多种场景和采集方式下具有高视觉质量的实时辐射场渲染，并在训练时间上与此前最快方法具有竞争力。作者强调3D Gaussian基元保留有利于优化的体渲染性质，又能直接适用于快速splat rasterization，因此全局连续的隐式表示不是获得快速、高质量辐射场训练的必要条件。作者还认为遵循实时图形学原则、利用GPU及软件光栅化管线是性能的关键。

【合理推断】“连续表示不是严格必要的”不能解释为本文完全没有连续函数：单个高斯核本身连续，但场景由无规则、离散数量的显式基元组成，渲染不需要密集查询一个由MLP定义的全局场。这是表示组织与渲染实现的区别，不是对任意体介质物理积分的严格替代定理。

该结论应限于论文的静态多视角、已标定相机、测试数据集与对应硬件。新视角RGB质量不是几何真实性、深度准确性、动态场景适应性或稀疏极端视角鲁棒性的证明。结论中的“first”和“best”是2023年论文的主张，不能当作今日最新排名。

**局限与未来**（§7.4、§8，PDF p.10–11）

1. 未见视角覆盖不足时会出现伪影，且缺细节区域可能由粗大的各向异性高斯表示（Figure 11–12）；基线Mip-NeRF360也在相同类型场景出现不同伪影。这说明图像监督和表示灵活性未消除观测歧义。
2. 排序按tile共享的深度顺序，不为每个像素独立排序；可能出现视角移动时的可见性变化或 popping，guard band也可能造成突变。作者指出仍有改善空间。
3. 模型内存明显高于NeRF类表示。大型场景训练峰值可超过20 GB；渲染需数百MB模型数据及随场景/分辨率变化的额外30–500 MB光栅器内存。紧凑性是相对于此前点基方法而言，不代表比隐式辐射场更省存储。
4. 作者的PyTorch原型约80%的训练时间消耗在Python代码；仅光栅化由优化的CUDA内核实现。作者设想将剩余优化逻辑迁移CUDA可进一步提速，并非报告已完成此优化。
5. 未来还可探索点云压缩适配到高斯表示，以及由高斯构建mesh来研究其在体与表面表示之间的位置。论文并未提供成熟的mesh提取方案或几何精度评估。

本单元复原作者的结论与适用范围，不单独展开数学推导。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2.1 图表功能总览

**Figure 总览**

| 编号（PDF页） | 内容 | 在论文中的功能 |
|---|---|---|
| Figure 1（1） | 代表场景速度、训练时间和质量对比 | 研究动机及总体成果 |
| Figure 2（5） | SfM→高斯优化/密度控制→渲染 | 总体训练与推理框架 |
| Figure 3（5） | 缩小后的高斯形状和渲染 | 表示的各向异性几何直观 |
| Figure 4（6） | clone 与 split 示意 | 自适应密度控制的两个分支 |
| Figure 5（7） | 9个真实场景定性比较 | 典型质量证据 |
| Figure 6（8） | 7k、30k训练迭代 | 训练速度与质量的折衷 |
| Figure 7（10） | 随机与SfM初始化 | 初始化消融 |
| Figure 8（10） | clone、split关闭的影响 | 增密消融 |
| Figure 9（10） | 限制梯度接收基元数 | 反向传播范围消融 |
| Figure 10（11） | 各向同性和各向异性 | 高斯形状消融 |
| Figure 11（11） | Train场景失败模式 | 不同表示的伪影性质 |
| Figure 12（11） | DrJohnson低视角重叠 | 观测不足的失败案例 |

**Table 与 Algorithm 总览**

| 编号（PDF页） | 内容 | 功能 |
|---|---|---|
| Table 1（8） | 3个实景数据集SSIM/PSNR/LPIPS、训练时间、FPS、内存 | 总体定量比较 |
| Table 2（8） | Synthetic NeRF逐场景PSNR | 随机初始化的合成场景验证 |
| Table 3（9） | 各组件消融逐场景PSNR | 设计贡献的受控比较 |
| Table 4（14） | Mip-NeRF360逐场景SSIM | Table 1细分证据 |
| Table 5（14） | Mip-NeRF360逐场景PSNR | Table 1细分证据 |
| Table 6（14） | Mip-NeRF360逐场景LPIPS | Table 1细分证据 |
| Table 7（14） | Tanks&Temples/Deep Blending逐场景SSIM | Table 1细分证据 |
| Table 8（14） | Tanks&Temples/Deep Blending逐场景PSNR | Table 1细分证据 |
| Table 9（14） | Tanks&Temples/Deep Blending逐场景LPIPS | Table 1细分证据 |
| Algorithm 1（13） | Optimization and Densification | 优化增密伪代码 |
| Algorithm 2（14） | GPU software rasterization of 3D Gaussians | GPU光栅化伪代码 |

【原文事实】以上按实际图表及算法编号列出，算法不混计入Table。后续图表分析依据已验收的PDF区域展示副本和同版原文，逐图解释而非仅复述题注。

本单元为图表功能目录，机制解释留在逐图及方法单元。
本单元为图表功能目录，不涉及数学推导。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
## Figure 1：速度、质量与训练时间的总体对比

**原图与题注**

![Figure 1：速度、质量与训练时间的总体对比](3DGS/Figures/Figure_01.jpg)

来源：同版 PDF p.1，区域 [50.88, 168.00, 561.12, 286.08]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 1. Our method achieves real-time rendering of radiance fields with quality that equals the previous method with the best quality [Barron et al. 2022], while only requiring optimization times competitive with the fastest previous methods [Fridovich-Keil and Yu et al. 2022; Müller et al. 2022]. Key to this performance is a novel 3D Gaussian scene representation coupled with a real-time differentiable renderer, which offers significant speedup to both scene optimization and novel view synthesis. Note that for comparable training times to InstantNGP [Müller et al. 2022], we achieve similar quality to theirs; while this is the maximum quality they reach, by training for 51min we achieve state-of-the-art quality, even slightly better than Mip-NeRF360 [Barron et al. 2022].

**Caption 中文：**

本文以与此前最高质量方法相当的质量实现辐射场实时渲染，而优化时间可与此前最快方法竞争。关键是新的三维高斯场景表示与实时可微渲染器，对场景优化和新视角合成都有显著加速。在与InstantNGP相近的训练时间下，本文达到相近质量；后者已达到其最高质量，而本文训练51分钟后达到当时领先的质量，甚至略高于Mip-NeRF360。

**视觉与证据**

六列为真值及不同训练/方法的自行车图像。原图标注的具体数值如下；不要用这里的单场景结果替代Table 1的跨场景平均。

| 方法 | 训练时间 | PSNR (dB) | FPS |
|---|---:|---:|---:|
| InstantNGP | 7 min | 22.1 | 9.2 |
| Plenoxels | 26 min | 21.9 | 8.2 |
| Mip-NeRF360 | 48 h | 24.3 | 0.071 |
| Ours（短训练） | 6 min | 23.6 | 135 |
| Ours（长训练） | 51 min | 25.2 | 93 |
| Ground Truth | — | — | — |

自行车的轮辐、车架、草地等细节是可视比较对象。长训练配置PSNR比短训练高1.6 dB；相对于图中Mip-NeRF360高0.9 dB，同时FPS为93而非0.071。这是图中给定配置比较，不能推广为所有数据集、相机和GPU都相同。训练更长时帧率下降，说明训练迭代数与渲染速度不是同一量；基元数及分布可能影响渲染成本，但仅凭本图无法识别下降原因。

**主要功能与正文关联：** §1用此图建立三目标：高画质、竞争性训练时间、实时渲染。§7 Table 1给出数据集平均；Figure 6说明训练预算影响质量。PSNR为峰值信噪比（dB，越高越好），FPS为每秒帧数（越高越好）；二者不能说明几何精度。

【证据边界】单场景定量标注可直接核对，不能解释完整因果或硬件统一性。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
## Figure 2：整体优化与渲染流程

**原图与题注**

![Figure 2：整体优化与渲染流程](3DGS/Figures/Figure_02.jpg)

来源：同版 PDF p.5，区域 [50.88, 76.80, 561.12, 175.20]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 2. Optimization starts with the sparse SfM point cloud and creates a set of 3D Gaussians. We then optimize and adaptively control the density of this set of Gaussians. During optimization we use our fast tile-based renderer, allowing competitive training times compared to SOTA fast radiance field methods. Once trained, our renderer allows real-time navigation for a wide variety of scenes.

**Caption 中文：**

优化从SfM产生的稀疏点云出发，建立一组3D高斯。随后优化这些高斯并自适应控制其密度。优化期间使用快速分块渲染器，使训练时间可与当时先进的快速辐射场方法竞争。训练完成后，渲染器允许在各种场景中实时浏览。

**视觉与证据**

图左SfM Points经过Initialization生成3D Gaussians；相机参数和高斯共同进入Projection，把三维基元转为像平面覆盖及深度等信息。Differentiable Tile Rasterizer得到Image。黑色箭头是操作流，蓝色箭头是梯度流。渲染图像与训练视图的比较产生损失（图中未单独画出损失框），反向经光栅器和投影传到高斯参数。Adaptive Density Control读取优化相关信息并改变高斯集合，再进入下一轮。高斯集合既是前向渲染输入也是训练更新对象；并非光栅化之后额外运行神经网络修复图像。

训练时相机是训练视角、Image与对应观测比较，输出更新后高斯集合；推理时选择新相机，只投影并渲染训练后的集合，不再要求目标图像或进行增密。图未提供精确张量尺寸及每步执行频率。

**主要功能与正文关联：** §3总览，§4投影（Eq.4–6），§5/Algorithm 1优化与增密，§6/Algorithm 2光栅器。此图为系统结构，不是单独证明速度的实验。

【证据边界】蓝色梯度流与黑色操作流区分，训练/推理差异有正文支持。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
## Figure 3：各向异性高斯的形状

**原图与题注**

![Figure 3：各向异性高斯的形状](3DGS/Figures/Figure_03.jpg)

来源：同版 PDF p.5，区域 [50.88, 227.76, 295.20, 327.12]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 3. We visualize the 3D Gaussians after optimization by shrinking them 60% (far right). This clearly shows the anisotropic shapes of the 3D Gaussians that compactly represent complex geometry after optimization. Left the actual rendered image.

**Caption 中文：**

我们将优化后的三维高斯缩小60%进行可视化（最右），清楚显示各向异性高斯的形状，以及它们如何紧凑地表示复杂几何。左侧为实际渲染图像。

**视觉与证据**

左侧完整火车车身，红框标出通风格栅；中部为实际局部渲染（Original），右部为相同局部的Shrunken Gaussians显示。右图的高斯形成多条细长、近似平行的空间结构，而非大量同形球状点。缩小仅为分离重叠基元、帮助观察形状；它不是正常渲染的质量对比，也不能把缩小后的空隙理解成真实几何空洞。

【背景知识】各向异性协方差允许长轴沿格栅、短轴垂直格栅，覆盖长条状区域并限制跨边缘模糊。若只允许一个尺度参数，基元必须在三轴同样伸缩，长条细结构常需要更多基元。此图直观展示形状可调的表示能力，但不量化节省了多少基元，也不证明高斯主轴严格等于真实表面切向。

**主要功能与正文关联：** §4 Eq.4/6，§5.1强调均匀区域使用少量大各向异性高斯，§7.3 Figure 10与Table 3提供关闭各向异性的对照。

【证据边界】原图、红框、Original与Shrunken图明确可见；几何解释标为背景。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
## Figure 4：自适应增密的两条分支

**原图与题注**

![Figure 4：自适应增密的两条分支](3DGS/Figures/Figure_04.jpg)

来源：同版 PDF p.6，区域 [50.88, 76.80, 295.20, 223.20]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 4. Our adaptive Gaussian densification scheme. Top row (under- reconstruction): When small-scale geometry (black outline) is insufficiently covered, we clone the respective Gaussian. Bottom row (over-reconstruction): If small-scale geometry is represented by one large splat, we split it in two.

**Caption 中文：**

本文的自适应高斯增密方案。上排（欠重建）：当小尺度几何（黑色轮廓）未被充分覆盖时，复制相应的高斯。下排（过重建）：当一个大splat表示小尺度几何时，把它分成两个。

**视觉与证据**

黑色轮廓示意目标几何，绿色区域代表高斯影响范围。上行开始只有一个小高斯，目标轮廓另一部分未覆盖；Clone增加同尺度基元，随后继续优化，新增自由度用于填补缺失。下行开始一个大高斯广泛覆盖轮廓之外，Split将其替换为两个较小基元，后续优化能沿弯曲目标分布。省略号明确表示拓扑调整只是初始化新增基元，非一步恢复真几何。

§5.2以视空间位置梯度平均模长高于0.0002识别候选，再按大小选择分支。Clone按作者描述复制大小并沿位置梯度方向移动；Split用原高斯作为概率密度采样两个中心、尺度除1.6。绿色二维示意不能直接作为严格三维体积守恒证明。注意密度控制是空间内基元数量，不是NeRF体密度sigma。

**主要功能与正文关联：** §5.2、Algorithm 1与Figure 8/Table 3。梯度大意味着图像损失对位置变化敏感，并非直接观测到正确几何；两个分支在相同梯度条件下分配不同尺度的自由度。

【证据边界】上下两行、轮廓与箭头已核对；阈值和分裂规则源自正文。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
## Figure 5：九个实景的定性新视角比较

**原图与题注**

![Figure 5：九个实景的定性新视角比较](3DGS/Figures/Figure_05.jpg)

来源：同版 PDF p.7，区域 [50.88, 72.96, 559.20, 649.20]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 5. We show comparisons of ours to previous methods and the corresponding ground truth images from held-out test views. The scenes are, from the top down: Bicycle, Garden, Stump, Counter and Room from the Mip-NeRF360 dataset; Playroom, DrJohnson from the Deep Blending dataset [Hedman et al. 2018] and Truck and Train from Tanks&Temples. Non-obvious differences in quality highlighted by arrows/insets.

**Caption 中文：**

展示本文与此前方法的比较及留出测试视图的对应真值。场景从上到下为Mip-NeRF360的Bicycle、Garden、Stump、Counter、Room，Deep Blending的Playroom、DrJohnson，以及Tanks&Temples的Truck和Train。箭头或局部放大标出不易察觉的质量差异。

**视觉与证据**

五列依次是Ground Truth、Ours、Mip-NeRF360、InstantNGP、Plenoxels。九行从室外植被到室内场景及车辆，图中红箭头和局部框聚焦难以凭缩略图发现的局部差异。

| 行/场景 | 可见内容和对比焦点 |
|---|---|
| Bicycle | 车架及车轮的细结构；箭头指向基线车轮附近细节 |
| Garden | 花盆、桌椅及后景植被；红框局部比较边缘和纹理 |
| Stump | 树根、地面和复杂叶片；部分基线局部模糊或出现斑块 |
| Counter | 金属盆、厨具、布纹等多材质；箭头指向左下局部 |
| Room | 沙发、桌面、墙面，观察边界与墙面纹理 |
| Playroom | 玩具、字母地垫；绿色框放大玩具边缘及局部重建差别 |
| DrJohnson | 壁面、画框与线条；部分基线有显著局部伪影 |
| Truck | 汽车及路面、边缘物体；箭头标示Mip-NeRF360局部 |
| Train | 火车和远背景；可比较整体结构及纹理表现 |

【原文事实】§7.2说明本文列为30K迭代训练，认为部分场景的植被和墙壁细节优于Mip-NeRF360。图像为留出视图而非训练照片的直接重投影展示。图中没有完整路径时序或深度真值，不能由单帧图证明无闪烁、几何完全正确或所有场景均占优。

**主要功能与正文关联：** §7.2、Table 1、附录Table 4–9。这里有九个例图，但实景评估共13个场景；必须用逐场景和平均指标补足选图偏差。

【证据边界】五列九行与放大图核对；不把主观视觉优势当全数据集排名。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
## Figure 6：7K和30K训练的视觉差异

**原图与题注**

![Figure 6：7K和30K训练的视觉差异](3DGS/Figures/Figure_06.jpg)

来源：同版 PDF p.8，区域 [50.88, 186.96, 295.20, 338.16]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 6. For some scenes (above) we can see that even at 7K iterations (∼5min for this scene), our method has captured the train quite well. At 30K itera- tions (∼35min) the background artifacts have been reduced significantly. For other scenes (below), the difference is barely visible; 7K iterations (∼8min) is already very high quality.

**Caption 中文：**

对某些场景（上），即使7K迭代（此场景约5分钟），方法也已较好地捕捉火车；30K迭代（约35分钟）后背景伪影显著减少。对另一些场景（下），差异几乎不可见，7K迭代（约8分钟）已达到很高质量。

**视觉与证据**

上下两行分别是Train和Garden，同一场景并排比较7K和30K。火车主体在短训练时已可识别，进一步训练改善较难的远背景；庭院中桌子与花盆等主要结构短训练已清楚，长训练的差异相对细微。说明迭代预算的边际收益受场景及区域影响，并非固定倍数增长。

这里K代表千次优化迭代，并非高斯数目。两种预算可能同时改变参数成熟度与增密进度，因此不能将差异唯一归因于更多梯度迭代而排除基元分布变化。

**主要功能与正文关联：** §7.2 Table 1同时给7K/30K速度、质量、训练时间和内存，Figure 1也展示短、长训练。图没有报告本视图PSNR；不从视觉差异补造数值。

【证据边界】双行迭代标签与caption一致，K的含义明确。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
## Figure 7：SfM点与随机初始化

**原图与题注**

![Figure 7：SfM点与随机初始化](3DGS/Figures/Figure_07.jpg)

来源：同版 PDF p.10，区域 [50.88, 78.00, 295.20, 272.16]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 7. Initialization with SfM points helps. Above: initialization with a random point cloud. Below: initialization using SfM points.

**Caption 中文：**

SfM点初始化有帮助。上：随机点云初始化；下：SfM点初始化。

**视觉与证据**

两行都是Garden，左侧整幅庭院、右侧放大植被区域。上行Random背景叶片相对模糊，下行SfM背景可分辨纹理更丰富；桌面和花盆等大结构均能出现。SfM（Structure-from-Motion，运动恢复结构）通过多视角匹配与三角化得到较可靠的初始空间支持，随机初始化则须从图像监督发现可用位置，尤其难处理大范围远景。

§7.3随机点均匀分布于边长尺度为输入相机包围盒范围三倍的立方体。Table 3的Garden 30K Random Init=22.19 dB、Full=27.70 dB，差5.51 dB；三场景平均差5.63 dB。这个消融支持此随机方案在真实场景劣于SfM，不证明任何随机策略都失败；§7.2的有界合成场景随机初始化可以表现良好。

**主要功能与正文关联：** §1、§5.1、§7.3与Table 3；初始化不是最终点数或完整表面重建。对比条件不能推广到已知边界与覆盖充分的合成场景。

【证据边界】Random/SfM标记、植被局部及Table 3数值均核对。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
## Figure 8：Clone 与 Split 的消融

**原图与题注**

![Figure 8：Clone 与 Split 的消融](3DGS/Figures/Figure_08.jpg)

来源：同版 PDF p.10，区域 [50.88, 319.92, 295.20, 615.12]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 8. Ablation of densification strategy for the two cases "clone" and "split" (Sec. 5).

**Caption 中文：**

对增密策略的两种情况（clone和split）进行消融（§5）。

**视觉与证据**

三行No Split-5k、No Clone-5k、Full-5k；左为自行车整图、右为局部放大。5k在这里是训练迭代标签。关闭split时，后景出现较宽、模糊的区域；关闭clone时，箭头处的细轮辐等结构更不完整；完整策略同时改善大尺度模糊与细尺度覆盖。

结合Table 3，Bicycle在5K的PSNR为No-Split 22.21、No-Clone 22.15、Full 23.18 dB；两种关闭分别低0.97和1.03 dB。30K的三场景平均Full 26.05、No-Split 23.90、No-Clone 25.91 dB。split对平均指标影响较大，而clone即使平均差较小仍可能影响局部细结构。图示支持两分支功能互补，而不是所有场景中每个分支都必然改善所有指标。

**主要功能与正文关联：** Figure 4为机制示意；§5.2及§7.3为设计与消融说明。关闭单一分支可影响后续基元总量和布局，不能将差值视为固定数学贡献。

【证据边界】三行标签、箭头和局部放大已核对；数值来自Table 3。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
## Figure 9：限制反向传播基元数量的影响

**原图与题注**

![Figure 9：限制反向传播基元数量的影响](3DGS/Figures/Figure_09.jpg)

来源：同版 PDF p.10，区域 [318.00, 78.00, 560.16, 157.20]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 9. If we limit the number of points that receive gradients, the effect on visual quality is significant. Left: limit of 10 Gaussians that receive gradients. Right: our full method.

**Caption 中文：**

若限制接收梯度的点的数量，视觉质量会受到显著影响。左：限制为10个接收梯度的高斯。右：本文完整方法。

**视觉与证据**

两幅Garden庭院图并排，左图整体细节较模糊，右图植被和桌面纹理较清晰。此实验限制的是每像素最前方N个点的梯度计算，而不是将整个场景只保留10个高斯，也不等价于前向仅渲染10个高斯。沿深度重叠多个半透明基元时，靠后的基元仍可能通过透射率对颜色有贡献；任意截断梯度会使这些贡献无法正确优化。

§7.3 N=10，Table 3 Garden 30K Limited-BW 22.88 vs Full 27.70，下降4.82 dB；Truck 13.84 vs 24.81，下降10.97 dB（约11 dB）。论文报告严重梯度近似导致不稳定。完整法仍有饱和与微小alpha阈值，不是无限计算所有高斯。

**主要功能与正文关联：** §6逆向遍历、Appendix C及Table 3 Limited-BW。图展示特定N的效果，不证明任意较大N都必然失效，也未给速度收益的完整权衡。

【证据边界】左右对应及N=10、PSNR差值按原文核对。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN FIG-10 -->
## Figure 10：各向异性与各向同性表示

**原图与题注**

![Figure 10：各向异性与各向同性表示](3DGS/Figures/Figure_10.jpg)

来源：同版 PDF p.11，区域 [50.88, 78.00, 560.16, 179.04]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 10. We train scenes with Gaussian anisotropy disabled and enabled. The use of anisotropic volumetric splats enables modelling of fine structures and has a significant impact on visual quality. Note that for illustrative purposes, we restricted Ficus to use no more than 5k Gaussians in both configurations.

**Caption 中文：**

在禁用及启用高斯各向异性的条件下训练。各向异性体splat能够建模精细结构，显著影响视觉质量。为便于说明，Ficus在两种配置中均限制不超过5k个高斯。

**视觉与证据**

三组局部图各包含Ground Truth、Full、Isotropic：左为Ficus植物枝叶，中为车辆及道路局部，右为汽车车轮。与各向同性配置比较，Full在细长枝叶和局部边界等具有更灵活的形状适配。Ficus特意限制两种配置均最多5000个高斯，这里的5k是基元数，与Figure 8的5k迭代不同。

【背景知识】若三轴尺度均为s，则协方差为s²I，旋转不影响形状；一般协方差允许旋转和三个独立轴尺度，从而在相同数量约束下用细长/扁平的基元覆盖不同方向结构。Table 3 30K平均Full 26.05 vs Isotropic 25.23 dB，差0.82 dB；这支持可调各向异性有利于质量，但不是各个场景均严格相同数量或物理表面恢复的证明。

**主要功能与正文关联：** §4 Eq.6、§7.3、Figure 3和Table 3。Ficus数量限制是明确的图示控制变量；不能将其自动套到全部消融设置。

【证据边界】三组三列实际标签与Ficus限定核对；区分迭代数和高斯数。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-10 -->

<!-- PTH:BEGIN FIG-11 -->
## Figure 11：不同表示的失败模式

**原图与题注**

![Figure 11：不同表示的失败模式](3DGS/Figures/Figure_11.jpg)

来源：同版 PDF p.11，区域 [51.84, 396.96, 294.00, 466.08]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 11. Comparison of failure artifacts: Mip-NeRF360 has “floaters” and grainy appearance (left, foreground), while our method produces coarse, anisoptropic Gaussians resulting in low-detail visuals (right, background). Train scene.

**Caption 中文：**

失败伪影比较：Mip-NeRF360出现漂浮物和颗粒感（左图前景），本文产生粗大的各向异性高斯，导致低细节外观（右图背景）。Train场景。

**视觉与证据**

左右为Train场景，左图火车附近可见漂浮式杂乱伪影，右图背景植被等区域外观粗糙、细节不足。作者用此例区分两种方法可能形成的错误类型，而不是证明本文无漂浮物或基线必然颗粒化。由于没有附上对应真值深度或明确误差图，不能把视觉诊断直接等同于测量到的几何误差。

【合理推断】当远景或某些区域观测不足，多个三维分布可能产生近似训练图像；高斯优化可以用大尺度基元拟合低频颜色而丢失细节。这解释一种可能机制，但仅凭此图不能判定每个失败基元的产生原因。

**主要功能与正文关联：** §7.4局限性，联系Figure 12低视角重叠及Figure 6远背景训练改善。作者指出各向异性既能表示结构也可能产生拉长伪影。

【证据边界】失败案例左右方法由题注确定；机制仅作推断，不冒充深度测量。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-11 -->

<!-- PTH:BEGIN FIG-12 -->
## Figure 12：低视角重叠区域的失败

**原图与题注**

![Figure 12：低视角重叠区域的失败](3DGS/Figures/Figure_12.jpg)

来源：同版 PDF p.11，区域 [51.84, 552.00, 294.00, 633.12]（PDF点，左上为原点）；B2 已核验的300 DPI JPEG展示副本。

**Caption 原文：**

> Fig. 12. In views that have little overlap with those seen during training, our method may produce artifacts (right). Again, Mip-NeRF360 also has artifacts in these cases (left). DrJohnson scene.

**Caption 中文：**

在与训练时所见视图重叠较少的视角中，本文可能产生伪影（右）。Mip-NeRF360在这些情况下也有伪影（左）。DrJohnson场景。

**视觉与证据**

左右均为室内角落、地毯及壁面透视，边缘和前景物体出现不自然的局部变形或模糊。右图的拉长或斑块式外观与§7.4描述的elongated/splotchy Gaussians相呼应。题注明确此处视图与训练视图重叠少。重叠少意味着该区域来自不同相机的重复观测约束不足，不能简单依靠增加高斯数保证正确外观。

【独立评价】此图是观测不足的具体失败反例，支持将3DGS性能主张限定在训练覆盖较充分的范围。没有视角重叠率数值、稀疏视角系统实验、真实深度或多种输入数量控制，因而不能推导“少于多少视图即失败”或比较方法在一般稀疏视角条件下的排序。

**主要功能与正文关联：** §7.4、Figure 11；与§5.1关于投影歧义及§5.2增密的讨论联系。作者提出未来可加入正则化，但并未在本文验证。

【证据边界】题注条件和左右方法核对，未将个例外推为完整稀疏视角研究。

本图以可视化或实验对照为主，不单独推导公式；相关公式在后续数学单元完整展开。

<!-- PTH:END FIG-12 -->

<!-- PTH:BEGIN TABLE-01 -->
## Table 1：三种真实数据集总体定量比较

**题注与数据**（PDF p.8）

**Caption 原文：**

> Table 1. Quantitative evaluation of our method compared to previous work, computed over three datasets. Results marked with dagger † have been directly adopted from the original paper, all others were obtained in our own experiments.

**Caption 中文：**

本文与此前方法的定量评估，在三个数据集上计算。带匕首符号†的结果直接采用自原论文，其余来自本文自行进行的实验。

### Mip-NeRF360

| 方法 | SSIM ↑ | PSNR ↑ | LPIPS ↓ | Train | FPS ↑ | Mem |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Plenoxels | 0.626 | 23.08 | 0.463 | 25m49s | 6.79 | 2.1GB |
| INGP-Base | 0.671 | 25.30 | 0.371 | 5m37s | 11.7 | 13MB |
| INGP-Big | 0.699 | 25.59 | 0.331 | 7m30s | 9.43 | 48MB |
| M-NeRF360 | 0.792† | 27.69† | 0.237† | 48h | 0.06 | 8.6MB |
| Ours-7K | 0.770 | 25.60 | 0.279 | 6m25s | 160 | 523MB |
| Ours-30K | 0.815 | 27.21 | 0.214 | 41m33s | 134 | 734MB |

### Tanks&Temples

| 方法 | SSIM ↑ | PSNR ↑ | LPIPS ↓ | Train | FPS ↑ | Mem |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Plenoxels | 0.719 | 21.08 | 0.379 | 25m5s | 13.0 | 2.3GB |
| INGP-Base | 0.723 | 21.72 | 0.330 | 5m26s | 17.1 | 13MB |
| INGP-Big | 0.745 | 21.92 | 0.305 | 6m59s | 14.4 | 48MB |
| M-NeRF360 | 0.759 | 22.22 | 0.257 | 48h | 0.14 | 8.6MB |
| Ours-7K | 0.767 | 21.20 | 0.280 | 6m55s | 197 | 270MB |
| Ours-30K | 0.841 | 23.14 | 0.183 | 26m54s | 154 | 411MB |

### Deep Blending

| 方法 | SSIM ↑ | PSNR ↑ | LPIPS ↓ | Train | FPS ↑ | Mem |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Plenoxels | 0.795 | 23.06 | 0.510 | 27m49s | 11.2 | 2.7GB |
| INGP-Base | 0.797 | 23.62 | 0.423 | 6m31s | 3.26 | 13MB |
| INGP-Big | 0.817 | 24.96 | 0.390 | 8m | 2.79 | 48MB |
| M-NeRF360 | 0.901 | 29.40 | 0.245 | 48h | 0.09 | 8.6MB |
| Ours-7K | 0.875 | 27.78 | 0.317 | 4m35s | 172 | 386MB |
| Ours-30K | 0.903 | 29.41 | 0.243 | 36m2s | 137 | 676MB |

**对比与结论**

【背景知识】PSNR（dB）基于均方像素误差，越高越好；SSIM衡量局部结构等相似性，越高越好；LPIPS基于感知特征距离，越低越好。Train为训练时间，FPS为帧/秒，Mem为存储/模型内存口径（不能视为训练峰值GPU内存）。INGP是InstantNGP，Base与Big为两种配置；7K、30K为训练迭代。原表横向18个数值列，此处按数据集拆成三个表，保持全部原数值及†。

【原文事实与比较】Mip-NeRF360：30K相对原论文†的Mip-NeRF360，SSIM +0.023，LPIPS −0.023，但PSNR −0.48 dB；不是三个指标全胜。134 fps相比0.06 fps明显提高。7K相对INGP-Big训练更短（6m25s vs 7m30s），PSNR仅高0.01 dB，但SSIM/LPIPS明显更好。

Tanks&Temples：30K相对M-NeRF360在三项质量指标均更好（PSNR+0.92 dB，SSIM+0.082，LPIPS−0.074）。7K的PSNR 21.20低于INGP-Big 21.92，也低于INGP-Base 21.72，不能笼统称短训练质量全面优于快速基线。

Deep Blending：30K相对M-NeRF360 PSNR仅+0.01 dB、SSIM+0.002、LPIPS−0.002；这些是平均值接近的证据，不足以说明统计显著优势。短训练的27.78 dB已明显高于INGP-Big 24.96。

三个数据集7K到30K：PSNR分别+1.61/+1.94/+1.63 dB，FPS分别160→134、197→154、172→137；模型存储增大。相比M-NeRF360 8.6MB，30K占用数百MB；速度收益伴随显著存储代价。与Plenoxels的GB级模型相比更紧凑，但不能概括为所有方法中最省内存。

【证据边界】§7.1–2的训练协议为每8张留出一张、13实景，硬件和分辨率影响速度。Mip-NeRF360 48h为4×A100训练12h折合单GPU总时数，非严格同硬件单卡实测；质量†来自原文，其余来自作者实验。附录D给出其复现逐场景值。图1为个例，与本表平均不同。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
## Table 2：合成NeRF场景PSNR

**题注与数据**（PDF p.8）

**Caption 原文：**

> Table 2. PSNR scores for Synthetic NeRF, we start with 100K randomly initialized points. Competing metrics extracted from respective papers.

**Caption 中文：**

Synthetic NeRF的PSNR结果；本文由100K随机初始化点开始。竞争方法的指标取自各自论文。

| 方法 | Mic | Chair | Ship | Materials | Lego | Drums | Ficus | Hotdog | Avg. |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Plenoxels | 33.26 | 33.98 | 29.62 | 29.14 | 34.10 | 25.35 | 31.83 | 36.81 | 31.76 |
| INGP-Base | 36.22 | 35.00 | 31.10 | 29.78 | 36.39 | 26.02 | 33.51 | 37.40 | 33.18 |
| Mip-NeRF | 36.51 | 35.14 | 30.41 | 30.71 | 35.70 | 25.48 | 33.29 | 37.48 | 33.09 |
| Point-NeRF | 35.95 | 35.40 | 30.97 | 29.61 | 35.04 | 26.06 | 36.13 | 37.30 | 33.30 |
| Ours-30K | 35.36 | 35.83 | 30.80 | 30.00 | 35.78 | 26.15 | 34.87 | 37.72 | 33.32 |

**对比与结论**

指标PSNR (dB，越高越好)；每列一个有界合成场景，Avg.为论文表中均值。Ours平均33.32比Point-NeRF 33.30仅高0.02 dB，比INGP-Base高0.14，比Mip-NeRF高0.23。逐列看Chair、Drums、Hotdog为表中最佳；Mic低于Mip-NeRF，Ship低于INGP，Materials低于Mip-NeRF，Lego低于INGP，Ficus低于Point-NeRF。不能把平均最高等同于每个场景最高。作者使用已知准确相机、较充分视图及白色背景，随机100K基元很快剪到约6–10K有意义基元，30K迭代最终约200–500K（§7.2）。随机点初始化能成功说明SfM不是所有设定的绝对先决条件。

【证据边界】竞争数字来自各自论文而非同一硬件复跑；表仅有PSNR，没有SSIM/LPIPS或时间统计。§7.2另报告180–300 FPS；与真实无界场景初始化消融条件不同。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
## Table 3：组件消融PSNR

**题注与数据**（PDF p.9）

**Caption 原文：**

> Table 3. PSNR Score for ablation runs. For this experiment, we manually downsampled high-resolution versions of each scene’s input images to the established rendering resolution of our other experiments. Doing so reduces random artifacts (e.g., due to JPEG compression in the pre-downscaled Mip-NeRF360 inputs).

**Caption 中文：**

消融运行的PSNR。作者将每个场景的高分辨率输入图像手动降采样至其他实验的既定渲染分辨率，以减少随机伪影（例如Mip-NeRF360预降采样输入中的JPEG压缩伪影）。

| 设置 | Truck-5K | Garden-5K | Bicycle-5K | Truck-30K | Garden-30K | Bicycle-30K | Average-5K | Average-30K |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Limited-BW | 14.66 | 22.07 | 20.77 | 13.84 | 22.88 | 20.87 | 19.16 | 19.19 |
| Random Init | 16.75 | 20.90 | 19.86 | 18.02 | 22.19 | 21.05 | 19.17 | 20.42 |
| No-Split | 18.31 | 23.98 | 22.21 | 20.59 | 26.11 | 25.02 | 21.50 | 23.90 |
| No-SH | 22.36 | 25.22 | 22.88 | 24.39 | 26.59 | 25.08 | 23.48 | 25.35 |
| No-Clone | 22.29 | 25.61 | 22.15 | 24.82 | 27.47 | 25.46 | 23.35 | 25.91 |
| Isotropic | 22.40 | 25.49 | 22.81 | 23.89 | 27.00 | 24.81 | 23.56 | 25.23 |
| Full | 22.71 | 25.82 | 23.18 | 24.81 | 27.70 | 25.65 | 23.90 | 26.05 |

**对比与结论**

PSNR(dB，↑)，5K与30K表示优化迭代；BW=backward，SH=spherical harmonics。Full为完整方法，其余各关闭或替换一个设计。定义差值为Full−对应变体，正表示完整方法更高。由原表Average列直接相减：

| 变体 | 5K差值(dB) | 30K差值(dB) | 解释 |
|---|---:|---:|---|
| Limited-BW | 4.74 | 6.86 | 梯度只传前10个，显著不稳定 |
| Random Init | 4.73 | 5.63 | 实景随机初始化失去几何支持 |
| No-Split | 2.40 | 2.15 | 无法有效细化大基元 |
| No-SH | 0.42 | 0.70 | 视角相关外观表示受限 |
| No-Clone | 0.55 | 0.14 | 局部细结构即使平均影响小仍可受损 |
| Isotropic | 0.34 | 0.82 | 单尺度限制细结构对齐 |

Truck Limited-BW 30K低至13.84，甚至比5K 14.66低，表明增加迭代不能弥补严重近似梯度；与Full差10.97 dB。No-Clone的Truck 30K为24.82，反而比Full 24.81高0.01；这是小例外，不能抹去。No-Split对三个场景30K均负向，Truck差4.22 dB。Figure 7–10提供对应定性解释。

【证据边界】题注明确另行降采样，因此不能将本表的Full逐场景数值直接等同于附录Table 5/8同名场景。消融是论文所设单项改动证据，未报告重复试验方差或完整速度/基元数变化。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
## Table 4：Mip-NeRF360逐场景SSIM

**题注与数据**（PDF p.14）

**Caption 原文：**

> Table 4. SSIM scores for Mip-NeRF360 scenes. † copied from original paper.

**Caption 中文：**

Mip-NeRF360场景的SSIM分数。†复制自原论文。

| 方法 | bicycle | flowers | garden | stump | treehill | room | counter | kitchen | bonsai |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Plenoxels | 0.496 | 0.431 | 0.6063 | 0.523 | 0.509 | 0.8417 | 0.759 | 0.648 | 0.814 |
| INGP-Base | 0.491 | 0.450 | 0.649 | 0.574 | 0.518 | 0.855 | 0.798 | 0.818 | 0.890 |
| INGP-Big | 0.512 | 0.486 | 0.701 | 0.594 | 0.542 | 0.871 | 0.817 | 0.858 | 0.906 |
| Mip-NeRF360† | 0.685 | 0.583 | 0.813 | 0.744 | 0.632 | 0.913 | 0.894 | 0.920 | 0.941 |
| Mip-NeRF360 | 0.685 | 0.584 | 0.809 | 0.745 | 0.631 | 0.910 | 0.892 | 0.917 | 0.938 |
| Ours-7k | 0.675 | 0.525 | 0.836 | 0.728 | 0.598 | 0.884 | 0.873 | 0.900 | 0.910 |
| Ours-30k | 0.771 | 0.605 | 0.868 | 0.775 | 0.638 | 0.914 | 0.905 | 0.922 | 0.938 |

**对比与结论**

SSIM无量纲，↑。与原论文†比较，Ours-30k在前8个场景均更高；bonsai 0.938低于0.941，而与本作者复现的0.938相等。Bicycle差+0.086，Garden+0.055，Treehill+0.006，Room+0.001，优势大小不均。Ours-7k在Garden 0.836已高于†的0.813，但flowers 0.525低于0.583。30K在全部9场景比7K更高。

【证据边界】结构相似性证据支持总体质量，但不是三维几何测量。附录D报告自行复现的Mip-NeRF360平均SSIM为0.790，Table 1采用†的0.792。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
## Table 5：Mip-NeRF360逐场景PSNR

**题注与数据**（PDF p.14）

**Caption 原文：**

> Table 5. PSNR scores for Mip-NeRF360 scenes. † copied from original paper.

**Caption 中文：**

Mip-NeRF360场景的PSNR分数。†复制自原论文。

| 方法 | bicycle | flowers | garden | stump | treehill | room | counter | kitchen | bonsai |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Plenoxels | 21.912 | 20.097 | 23.4947 | 20.661 | 22.248 | 27.594 | 23.624 | 23.420 | 24.669 |
| INGP-Base | 22.193 | 20.348 | 24.599 | 23.626 | 22.364 | 29.269 | 26.439 | 28.548 | 30.337 |
| INGP-Big | 22.171 | 20.652 | 25.069 | 23.466 | 22.373 | 29.690 | 26.691 | 29.479 | 30.685 |
| Mip-NeRF360† | 24.37 | 21.73 | 26.98 | 26.40 | 22.87 | 31.63 | 29.55 | 32.23 | 33.46 |
| Mip-NeRF360 | 24.305 | 21.649 | 26.875 | 26.175 | 22.929 | 31.467 | 29.447 | 31.989 | 33.397 |
| Ours-7k | 23.604 | 20.515 | 26.245 | 25.709 | 22.085 | 28.139 | 26.705 | 28.546 | 28.850 |
| Ours-30k | 25.246 | 21.520 | 27.410 | 26.550 | 22.490 | 30.632 | 28.700 | 30.317 | 31.980 |

**对比与结论**

PSNR(dB，↑)。对照†，30K在bicycle(+0.876)、garden(+0.430)、stump(+0.150)领先；flowers、treehill、room、counter、kitchen、bonsai落后，其中kitchen差−1.913 dB，bonsai差−1.480 dB。与作者自跑Mip-NeRF360相比，领先场景仍为bicycle、garden、stump，均值作者自跑27.58 vs Ours 27.21。30K全部场景比7K提高，bonsai增长3.130 dB。

【证据边界】不能用Figure 1的bicycle个例替代总体PSNR；Table 1的27.69†与作者复现27.58应区分，且Table 3有不同降采样协议。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
## Table 6：Mip-NeRF360逐场景LPIPS

**题注与数据**（PDF p.14）

**Caption 原文：**

> Table 6. LPIPS scores for Mip-NeRF360 scenes. † copied from original paper.

**Caption 中文：**

Mip-NeRF360场景的LPIPS分数。†复制自原论文。

| 方法 | bicycle | flowers | garden | stump | treehill | room | counter | kitchen | bonsai |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Plenoxels | 0.506 | 0.521 | 0.3864 | 0.503 | 0.540 | 0.4186 | 0.441 | 0.447 | 0.398 |
| INGP-Base | 0.487 | 0.481 | 0.312 | 0.450 | 0.489 | 0.301 | 0.342 | 0.254 | 0.227 |
| INGP-Big | 0.446 | 0.441 | 0.257 | 0.421 | 0.450 | 0.261 | 0.306 | 0.195 | 0.205 |
| Mip-NeRF360† | 0.301 | 0.344 | 0.170 | 0.261 | 0.339 | 0.211 | 0.204 | 0.127 | 0.176 |
| Mip-NeRF360 | 0.305 | 0.346 | 0.171 | 0.265 | 0.347 | 0.213 | 0.207 | 0.128 | 0.179 |
| Ours-7k | 0.318 | 0.417 | 0.153 | 0.287 | 0.404 | 0.272 | 0.254 | 0.161 | 0.244 |
| Ours-30k | 0.205 | 0.336 | 0.103 | 0.210 | 0.317 | 0.220 | 0.204 | 0.129 | 0.205 |

**对比与结论**

LPIPS无量纲，↓。30K相对†在bicycle、flowers、garden、stump、treehill五场景降低，counter同为0.204；room、kitchen、bonsai更高。Garden 0.103 vs 0.170下降0.067，Bicycle 0.205 vs 0.301下降0.096；bonsai 0.205 vs 0.176上升0.029。全部场景30K优于7K。作者自跑Mip-NeRF360均值0.240，原论文†平均0.237，Ours 0.214。

【证据边界】本指标衡量图像感知相似性；与Table5 PSNR的胜负不同并不矛盾。不同指标的尺度和意义不能直接相加。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN TABLE-07 -->
## Table 7：Tanks&Temples和Deep Blending逐场景SSIM

**题注与数据**（PDF p.14）

**Caption 原文：**

> Table 7. SSIM scores for Tanks&Temples and Deep Blending scenes.

**Caption 中文：**

Tanks&Temples和Deep Blending场景的SSIM分数。

| 方法 | Truck | Train | Dr Johnson | Playroom |
| --- | ---: | ---: | ---: | ---: |
| Plenoxels | 0.774 | 0.663 | 0.787 | 0.802 |
| INGP-Base | 0.779 | 0.666 | 0.839 | 0.754 |
| INGP-Big | 0.800 | 0.689 | 0.854 | 0.779 |
| Mip-NeRF360 | 0.857 | 0.660 | 0.901 | 0.900 |
| Ours-7k | 0.840 | 0.694 | 0.853 | 0.896 |
| Ours-30k | 0.879 | 0.802 | 0.899 | 0.906 |

**对比与结论**

SSIM↑。30K在Truck、Train、Playroom三场景最佳；Dr Johnson 0.899略低于Mip-NeRF360 0.901。Train比Mip-NeRF360高0.142，Truck高0.022；T&T均值0.841。DB两场景方向不同，平均0.903与基线0.901接近。30K全部高于7K。

【证据边界】T&T与DB各仅两场景，不由均值推导统计显著性；Dr Johnson的例外也与Figure 12低覆盖失败并存。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-07 -->

<!-- PTH:BEGIN TABLE-08 -->
## Table 8：Tanks&Temples和Deep Blending逐场景PSNR

**题注与数据**（PDF p.14）

**Caption 原文：**

> Table 8. PSNR scores for Tanks&Temples and Deep Blending scenes.

**Caption 中文：**

Tanks&Temples和Deep Blending场景的PSNR分数。

| 方法 | Truck | Train | Dr Johnson | Playroom |
| --- | ---: | ---: | ---: | ---: |
| Plenoxels | 23.221 | 18.927 | 23.142 | 22.980 |
| INGP-Base | 23.260 | 20.170 | 27.750 | 19.483 |
| INGP-Big | 23.383 | 20.456 | 28.257 | 21.665 |
| Mip-NeRF360 | 24.912 | 19.523 | 29.140 | 29.657 |
| Ours-7k | 23.506 | 18.892 | 26.306 | 29.245 |
| Ours-30k | 25.187 | 21.097 | 28.766 | 30.044 |

**对比与结论**

PSNR(dB,↑)。Ours-30k相对Mip-NeRF360：Truck+0.275，Train+1.574，Dr Johnson−0.374，Playroom+0.387 dB。DB正负相抵，均值约+0.0065 dB，表1四舍五入约+0.01。7K Train 18.892低于其他方法，Dr Johnson 26.306低于INGP-Base/Big及Mip-NeRF360；短训练不是普遍最高质量。30K四场景均提高。

【证据边界】附录细分解释Table 1平均；Table3手工降采样，PSNR口径不直接合并。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-08 -->

<!-- PTH:BEGIN TABLE-09 -->
## Table 9：Tanks&Temples和Deep Blending逐场景LPIPS

**题注与数据**（PDF p.14）

**Caption 原文：**

> Table 9. LPIPS scores for Tanks&Temples and Deep Blending scenes.

**Caption 中文：**

Tanks&Temples和Deep Blending场景的LPIPS分数。

| 方法 | Truck | Train | Dr Johnson | Playroom |
| --- | ---: | ---: | ---: | ---: |
| Plenoxels | 0.335 | 0.422 | 0.521 | 0.499 |
| INGP-Base | 0.274 | 0.386 | 0.381 | 0.465 |
| INGP-Big | 0.249 | 0.360 | 0.352 | 0.428 |
| Mip-NeRF360 | 0.159 | 0.354 | 0.237 | 0.252 |
| Ours-7k | 0.209 | 0.350 | 0.343 | 0.291 |
| Ours-30k | 0.148 | 0.218 | 0.244 | 0.241 |

**对比与结论**

LPIPS↓。30K相对Mip-NeRF360在Truck下降0.011、Train下降0.136、Playroom下降0.011，Dr Johnson上升0.007。T&T均值0.183 vs 0.257，DB均值0.243 vs 0.245（原表精度）；主要优势由Train推动。7K Train 0.350略优于Mip-NeRF360 0.354，但7K Truck及DB较弱。30K四场景均比7K下降。

【证据边界】此表揭示平均感知指标背后差异，不等价于深度准确度或跨任意新视角无伪影。

本单元聚焦原表数据和对照，不单独推导渲染或训练公式。
本表不直接呈现算法数据流，相关机制联系已在对比说明，完整算法在方法单元展开。

<!-- PTH:END TABLE-09 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 2.4 图表之间的逻辑关系

**方法证据链**

Figure 1将质量、训练时间与FPS放在同一示例上，提出本文想解决的矛盾，但不独立证明机制。Figure 2把答案组织为表示、学习、渲染三部分；Figure 3显示表示在优化后确实呈现细长的各向异性形状；Figure 4描述拓扑调整如何补足缺失的自由度。这三个方法图分别回答系统如何连接、基元为何有用、为何不只更新固定集合。Eq.1–6建立合成和投影数学，Eq.7与Algorithm 1描述监督优化，Algorithm 2把光栅化拆为视锥剔除、投影、实例化、排序、合成。

**实验与反例链**

| 问题 | 主要证据 | 补充与边界 |
|---|---|---|
| 整体NVS质量与速度是否兼得 | Figure 5、Table 1 | Table 4–9展开每场景，PSNR/SSIM/LPIPS不完全同向 |
| 短训练是否可用 | Figure 6、Table 1 | 不同场景边际收益不同；7K不是30K同一质量 |
| SfM初始化是否必要 | Figure 7、Table 3 | Table 2反例：有界合成场景可随机初始化 |
| 密度控制两分支是否有效 | Figure 4、8、Table 3 | split更影响平均质量，clone影响局部细结构 |
| 梯度是否可任意截断 | Figure 9、Table 3 | 只测试N=10，不是所有可能截断策略 |
| 各向异性是否重要 | Figure 3、10、Table 3 | Ficus限同一上界；不能证明真实几何精度 |
| 方法是否总能恢复未观测区域 | Figure 11、12 | 明确反例；没有系统稀疏视角评估 |

Table 2检验随机初始化在有界合成场景的可行性，而Table 3检验实景随机方案；表面上不同结果并不矛盾，因为边界与覆盖条件不同。Table 4–9用于解释Table 1均值；Figure 5使用作者自行复跑的基线图像而Table 1部分数字来自原论文，附录D弥补这个口径区别。Table 3的人工降采样不能与附录绝对值直接拼接。Algorithm 1与2提供实现逻辑，而非新的实验数据。

【证据边界】方法图和公式说明设计可如何工作，定量/定性对照提供经验支持，失败图明确限制。三种证据不能互相替代。

本单元聚焦该阶段的概念和算法机制，完整公式及必要推导另见数学单元。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 3.1 研究问题

**任务定义**（§1、§3、§7）

任务是静态场景的新视角合成（Novel View Synthesis, NVS）。输入为多张拍摄同一场景的RGB图像及其相机内外参；典型真实场景由SfM标定并附带稀疏点云。优化所得并非单一新视角图像，而是一个场景特定的高斯集合，之后给定新相机即可生成对应RGB图像。没有训练好的跨场景前馈网络，也不需要推理时提供目标图像。论文的随机初始化合成实验说明准确SfM点并非每种设定都必需，但相机参数仍已知。

【背景知识】设观测集合为 $\mathcal D=\{(I_k,P_k)\}$，场景参数 $\Theta=\{\mu_i,s_i,q_i,o_i,h_i\}_{i=1}^N$，$P_k$含相机标定。可把学习问题形式化为

$$\min_{\Theta,N}\sum_k \ell(\mathcal R(\Theta,P_k),I_k).$$

这是对本文机制的概括，不是作者单独给出的全局离散优化公式：实际用随机训练视图、Adam梯度更新和启发式基元增删，而非一次解析求解N。$\mathcal R$是投影和alpha合成的可微渲染器，$\ell$为Eq.7。推理为 $I_*=\mathcal R(\Theta_*,P_*)$。

**研究问题**

核心工程科学问题是：如何设计一种场景表示及与其匹配的优化/渲染算法，使完整静态场景在高分辨率下同时达到高质量新视角合成、竞争性训练时间和实时显示？目标是缓和既有质量—效率矛盾，而不是首先提出新的成像物理定律。

为何不能只用传统点云？SfM稀疏、缺少法线；离散点渲染有空洞/混叠，并不提供易优化的连续覆盖。为何不只用连续NeRF？射线采样和重复特征查询带来开销，高质量模型训练也慢。本文借鉴两类表示的优点：局部连续核提供可微覆盖，离散显式基元提供GPU可投影的计算结构。

【独立评价】从有限二维视图反求三维结构本身不适定。仅以RGB重建损失优化，颜色、位置、形状和不透明度的变化可能互相补偿。因此目标输出是可渲染的场景表示，并不保证唯一或真实的几何和介质参数。即使损失很低，也可能在新视角产生伪影（Figure 12）。

【证据边界】论文没有把输入明确限定为极少视图，也未将精确深度恢复、动态建模、实时建图或参与介质物理反演作为所验证的任务。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 3.2 研究背景与动机

**根因分析**（§1–2，PDF p.1–4）

1. 隐式场的计算结构：NeRF沿每条射线查询密度/颜色，图像有H×W条射线，若每条使用K个采样，查询量与H×W×K相关。网络或插值在每个采样点被调用，空空间上的查询也可能消耗时间；重要性采样、占据网格能减轻但不能完全改变逐射线查询结构。高分辨率和复杂深度层使成本更明显。此为通用复杂度说明，论文不提供固定K或精确运行复杂度定理。
2. 结构化加速的质量/效率限制：InstantNGP用hash grid、occupancy grid和较小MLP，Plenoxels用稀疏体素网格插值；作者指出网格表示及射线采样仍限制质量和速度。并非网格本身必然劣于无规则基元，场景、分辨率与优化都影响效果。
3. 传统点/面方法的覆盖和几何依赖：孤立固定大小点可能留孔、混叠；带法线圆盘需要可靠局部表面方向，而SfM极稀疏难稳健估计。MVS重建在弱纹理、镜面、细结构区域可欠重建或过重建；重投影混合依赖这套几何。本文不以MVS密集表面为必要输入。
4. 多层可见性：物体及背景同时存在，许多半透明基元在像面重叠。如果使用无序合成，颜色遮挡逻辑发生改变；若对每个像素分别排序，开销大。本文折中为tile共享近似排序及保留所有实际贡献基元的梯度。

**证据及目标**

§1描述当时高质量Mip-NeRF360训练可能达48小时，较快方法在某些场景约10–15 fps，难以达到1080p实时目标。Figure 1提供单个场景的具体质量/速度对比；Table 1提供3个实景数据集的均值（例如Mip-NeRF360数据集Ours-30K 134 fps，而M-NeRF360 0.06 fps）；Table 3与Figure 7–10检验主要设计。本文针对的是已存在的效率—质量矛盾，作者的贡献主要是提出新的表示、优化与GPU渲染组合，而非首次发现该矛盾。

【独立评价】效率需同时看训练时间、推理FPS、分辨率、GPU、存储与基元数。显式局部表示使稀疏计算成为可能，但如果存在许多巨大、深度重叠的高斯，tile实例数和像素处理量仍会增加。方法设计需同时让高斯可优化、分布适当且渲染足够高效，单独换掉MLP不充分。

【证据边界】作者关于前人方法的评论来自本篇Related Work与实验，未独立查阅其他论文实现。跨硬件结果不是严格的计算量定理。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 3.3 相关工作（Related Work）

**传统与神经辐射场**（§2.1–2.2，p.2–3）

| 方向或代表工作 | 本文所描述的基本机制 | 作者指出的局限/本文区别 |
|---|---|---|
| Light field | 密集或无规则采集视图，合成新的视图 | 不直接提供本篇可自由优化的高斯空间表示 |
| SfM + MVS + reprojection | SfM标定和稀疏点，MVS估计更密集几何，用几何引导输入图像重投影混合 | MVS错几何、漏重建影响合成；依赖输入图像存储。本文仅典型使用SfM点初始化 |
| CNN辅助神经渲染 | CNN估计混合权重、纹理或由特征图生成像素 | 依赖MVS几何；作者指出CNN最终渲染可能时间闪烁 |
| Soft3D/早期可微体渲染 | 通过连续可微体密度及ray marching合成 | 沿射线大量空间查询 |
| NeRF (Mildenhall et al. 2020) | MLP表示密度与外观，位置编码和重要性采样改善质量 | 大MLP和射线采样慢；本文保留alpha合成形式而改用显式核投影 |
| Mip-NeRF360 (Barron et al. 2022) | 作者选为完整场景的高质量基准 | 训练与渲染成本高；本文对比SSIM/PSNR/LPIPS、时间、FPS、内存 |
| InstantNGP | hash grid、占据网格、小MLP | 更快但仍逐射线采样；本文不需该网格及神经组件 |
| Plenoxels | 稀疏体素网格插值连续密度场，无需神经网络 | 仍依赖规则空间结构与射线查询；本文使用无规则显式基元 |

§2.2还讨论网格/编码/MLP容量、空间离散、码本与其他加速路线，强调这些设计并非都等同于NeRF的大型MLP。原文在SH用途的former/latter指代上存在阅读歧义（p.3，先列InstantNGP再Plenoxels，继而描述直接方向表征和网络输入编码）；本文不据此对两篇未独立阅读的工作指定确切SH实现。3DGS自身的SH颜色用途由§3、5、7.1确认。

**点基渲染关系**（§2.3、p.3–4）

| 方向 | 核心思路 | 与3DGS关系 |
|---|---|---|
| 固定点渲染 | 离散点光栅化 | GPU友好但空洞、混叠，本文使用有空间范围的核 |
| 经典splatting（EWA等） | 圆盘、椭球、surfel等带范围基元投影 | 继承投影协方差和椭圆核思想，不是本文首次发明splatting |
| 可微点基渲染 | 优化位置、颜色、alpha或神经特征，部分用CNN | 本文复用可微与alpha合成观念，不依赖MVS法线或CNN最终成像 |
| Pulsar | 快速球体光栅化和GPU组织 | 启发tile排序；作者认为其无序合成和梯度数量限制不适合本文深度复杂性目标 |
| ADOP | 快速点基神经渲染 | 速度启发，本文改为各向异性高斯及直接合成 |
| Neural Point Catacaustics | MLP处理外观/镜面等方向变化 | 原文说仍需MVS几何；3DGS以SH颜色及直接光栅化组织 |
| Zhang et al. 2022 | 基于点的可微辐射场，使用SH、无需MVS但需mask初始化 | 作者指出其对象/分辨率和点数范围；§7.2紧凑性对比使用其点云条件 |
| Point-NeRF | 径向基点场，优化中剪枝、增密，体ray marching | 与3DGS共有点分布优化思想；后者为投影splat与tile光栅化 |
| 人体捕获的高斯/体基元 | 使用3D Gaussian或神经体基元建模孤立人体/脸 | 启发表示选择；本文面向室内外完整场景、大深度复杂性，联合协方差/增密/排序 |

【背景知识】这些路线的关键差异不只是“显式还是隐式”，还包括连续空间如何查询、每个像素如何合成、基元是否可改变形状、是否需密集几何、可见性是否依顺序，以及梯度可达的基元范围。Eq.1–3显示体渲染和有序点合成有相同的乘积权重结构，为选择显式高斯提供理论连接；它不是证明各方法底层连续物理完全相同。

【证据边界】本节忠实整理本篇作者的相关工作论述和对比定位，不等于对所有引用文献的独立全文复核。新颖性应聚焦本文针对完整场景的表示—优化—渲染联合设计。

本单元聚焦该阶段的概念和算法机制，完整公式及必要推导另见数学单元。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-REP -->
## 3.4.1 场景表示：可优化的三维高斯

**高斯参数**（§3–4、§5，Figure 2–3）

输入是世界坐标中的SfM点与标定相机；输出是一组不规则分布的可学习高斯基元，每个基元包含：

| 参数 | 意义 | 维度（按数学定义） |
|---|---|---|
| $\mu_i$ | 世界坐标中心 | 3 |
| $s_i$ | 三个轴的正尺度 | 3 |
| $q_i$ | 旋转的单位四元数 | 4，受单位范数约束 |
| $o_i$ | 学习的不透明度（激活后） | 1 |
| $h_i$ | 方向颜色SH系数 | 依启用频带而定 |

空间核为$G_i(x)=\exp[-\frac12(x-\mu_i)^T\Sigma_i^{-1}(x-\mu_i)]$（Eq.4使用以中心为原点的坐标）；$\Sigma_i=R_i\operatorname{diag}(s_i^2)R_i^T$（Eq.6）。$o_i$乘以投影二维核值得到每像素alpha；SH评价当前视线方向下的RGB颜色。这里区分学习的基元opacity和在像素位置得到的alpha，后者随像素及视角变化。三维核非归一化，不能把opacity直接当作物理消光系数$\sigma$。

给定相机$V$：中心做世界→相机→像素的投影，协方差由投影在中心的局部仿射近似传播，取二维块；得到像素中心$u_i$、二维协方差$Q_i$、排序深度$z_i$，以及对应视角的颜色$c_i$。对于像素$p$，影响程度可写为（依据§2.3和§4的显式表达）

$$a_i(p)=o_i\exp[-\tfrac12(p-u_i)^T Q_i^{-1}(p-u_i)].$$

这一项经裁剪、排序和alpha合成影响渲染图。三维均值影响投影中心、深度和投影Jacobian；尺度/旋转影响形状；opacity影响透射；SH影响颜色。所有这些路径最终可由图像损失监督。

**表示设计**

不直接优化任意协方差的6个独立元素，因为梯度步可能产生非法负特征值；用尺度和旋转保证正定（非零尺度），也无需从极稀疏点估计法线。形状自由度允许大基元覆盖均匀区域、细长或扁平基元描述薄结构。Figure 3为火车格栅可视化，Figure 10/表3提供各向异性消融。

局部核是连续的，但基元索引、数量和分布是离散不规则的。相比用一个网络或网格插值产生连续场，渲染可直接投影与当前视锥相关的基元，减少无效空间查询；不要求对每个像素均匀采样所有距离。与显式点云相比，核有可调空间支持，不仅是点坐标或单像素。与网格表面相比，基元不要求构成拓扑连续曲面。

训练时高斯参数和集合大小都变；推理时固定集合，仅按相机变换与方向评价后渲染。论文真实场景优化得到约1–5百万高斯（§1）。

【证据边界】SH频带数量由§7.1提供，具体通道存储布局未从代码核查；Gaussian作为概率密度进行split采样不意味着常规渲染的非归一化核就是归一化概率模型。位置、尺度、颜色等存在耦合，仅由RGB监督不能证明精确表面。

<!-- PTH:END METHOD-REP -->

<!-- PTH:BEGIN METHOD-OPT -->
## 3.4.2 初始化、损失与优化循环

**初始化与监督**（§5.1、§7.1）

1. 从SfM稀疏点云构建高斯中心；初始尺度为到最近三个点距离的平均值，设为各向同性。局部点密度因场景和匹配结果不同而变化，因此用邻域距离估计初始支持范围比统一半径更有适配性。作者未给出所有初始化常数，不能补造初始opacity或SH数值。
2. 可优化实值经sigmoid生成opacity，实值尺度参数经exp生成正尺度；旋转四元数归一化。这样无约束优化器的参数更新可映射为有效几何与不透明度。位置的学习率采用指数衰减，§5.1特别说明仅位置采用此调度。附录B的Algorithm 1明确采用Adam；“SGD techniques”是随机梯度类方法的概述，不是断言原型使用普通SGD而非Adam。
3. 每轮采样一个训练相机及图像，光栅化得到同分辨率RGB预测$\hat I$，计算Eq.7：

$$\mathcal L=(1-\lambda)\mathcal L_1+\lambda\mathcal L_{D\text{-}SSIM},\qquad \lambda=0.2.$$

L1约束逐像素绝对误差；D-SSIM（结构不相似性）约束局部结构。作者没有在论文内详述SSIM窗口等实现细节。RGB损失通过合成及投影反传至位置、尺度、旋转、opacity和方向颜色；不需真实深度或高斯标签。位置梯度还用来判断密度控制候选。

§7.1低分辨率warm-up：起始用4倍更低分辨率，250与500迭代分别提升分辨率；以论文原有措辞记录，不擅自指定每维或总像素缩减的具体程序。先优化SH零阶分量，随后每1000迭代引入一个频带，直到4个频带均使用。理由是角度覆盖不足时同时优化高阶方向项可能得到错误的零阶颜色；分阶段提升表达能力为基础颜色提供较稳定训练开端。

**训练循环**（Appendix B，Algorithm 1，p.13）

```
M = SfM points
S, C, A = InitAttributes()
i = 0
while not converged:
    V, I_gt = SampleTrainingView()
    I_pred = Rasterize(M, S, C, A, V)  # Algorithm 2
    L = Loss(I_pred, I_gt)
    M, S, C, A = Adam(grad L)
    if IsRefinementIteration(i):
        for Gaussian (mu, Sigma, c, alpha):
            if alpha < eps or IsTooLarge(mu, Sigma):
                RemoveGaussian()
            if view_position_gradient > tau_p:
                if norm(S) > tau_S:
                    SplitGaussian(...)
                else:
                    CloneGaussian(...)
    i += 1
```

此为原文算法中文解释用的等价结构摘录；$M,S,C,A$分别存均值、协方差相关属性、颜色、opacity。原伪代码顺序列出Pruning与Densification，实际删除后遍历索引维护及优化器状态迁移没有详细指定，不能从此推断代码必然访问已删除对象。§5.2另述定期reset opacity，不在该伪代码中完整展开。

训练前向、反向、参数更新与拓扑调整是不同类型操作：连续参数更新可微，clone/split/prune是离散结构修改，通过启发式间隔触发，改变下一轮计算图规模。推理固定参数后不再需要损失或上述流程。

【证据边界】论文公开训练的关键阈值和时间表，但不等于完整源码级复现清单；未提供的学习率、初始opacity与属性复制细节保留未证实。

<!-- PTH:END METHOD-OPT -->

<!-- PTH:BEGIN METHOD-DENSIFY -->
## 3.4.3 自适应密度控制：clone、split、prune 与 reset

**密度控制**（§5.2，PDF p.5–6、Figure 4）

论文所谓density是单位空间内高斯基元的数量/分布，不是NeRF密度$\sigma$，也不是高斯概率密度的归一化。由于初始点稀疏，仅移动固定N个高斯很难同时覆盖所有新结构；同时一些基元可能位于错误空间区域。论文在warm-up后每100次迭代增密，统计各基元视空间位置梯度的平均模长，超过阈值$\tau_{pos}=0.0002$作为候选。这个量反映像平面位置微调对训练损失的敏感性，是启发式优化信号；不是深度残差或几何真值。

| 操作 | 条件（概念） | 更新及原因 |
|---|---|---|
| Clone | 位置梯度大，基元较小 | 新增一个同尺度高斯；原文描述沿位置梯度方向移动副本，目的是覆盖欠重建的邻域 |
| Split | 位置梯度大，基元较大 | 原基元被两个更小基元替代；新中心从原三维高斯作为PDF采样，轴尺度除以1.6 |
| Prune | opacity低于$\epsilon_\alpha$或高斯过大 | 删除近透明、过大世界空间或像平面footprint的基元，控制不必要的数量/伪影 |
| Reset | 每3000迭代 | opacity设得接近0，随后让优化重新增大需要的基元opacity，便于剪除不必要基元 |

**设计机制**

欠重建时，小高斯无法覆盖邻近缺失几何，单纯扩大可能损失已有精细形状；clone同时增加总支持与基元数量。过重建时，一个大高斯同时覆盖前景和背景、难以拟合弯曲或细节，split增加自由度并减小单个支持范围。两者并不是先判断真实几何错误，再执行完美修复；梯度大的区域可能也由颜色、视角依赖、遮挡或不良初始化引起，大小只是启发式分类。Figure 4的末列还需要继续优化。

【补充推导】作者用“conserve total volume”说明split设计意图。若将椭球固定等值面体积定义为$V=\frac43\pi k^3s_xs_ys_z$，两个尺度都除1.6且忽略相互重叠，则体积之和为$2V/1.6^3\approx0.488V$，并非严格等于V。因此本文不将原文定性描述解释为该具体体积定义的数学守恒；opacity与实际覆盖也会变化。分裂因子为经验设定，非守恒定理。Clone的“沿梯度方向”保留作者描述，具体位置更新符号及实现未查源码，不能与通常负梯度下降混为一谈。

reset帮助减轻靠近输入相机的floaters造成的无谓增密；它可能暂时降低渲染质量，但下一阶段优化重新建立opacity并提供剪枝机会。高斯过大世界/像平面阈值的具体数值未在该段提供。Table 3与Figure 8表明关闭split会明显降低平均PSNR，关闭clone的平均影响较小但细结构变差。训练以离散增删改变N，而推理无需密度控制。

【证据边界】不能将高梯度必然等同于正确的新几何位置，不能把基元增密当作真实场景物理密度提高；原文未提供每次增密的确定性全局最优证明。

<!-- PTH:END METHOD-DENSIFY -->

<!-- PTH:BEGIN METHOD-RASTER -->
## 3.4.4 GPU分块光栅器：前向渲染

**渲染前向**（§6、Appendix C，p.6、13–14）

输入为图像宽高$w,h$、高斯均值/协方差$M,S$、颜色和opacity $C,A$及相机$V$。输出为$h\times w\times3$ RGB图像。对于当前相机，执行如下步骤（Algorithm 2）：

1. Frustum culling：仅保留99%置信区间与视锥相交的高斯，并用guard band排除均值近近平面且极远离视锥的极端基元，防止投影协方差数值不稳定。无限支撑的数学高斯在实现中采用有限候选覆盖，而非真正计算每个无限远尾部。
2. ScreenspaceGaussians：中心投影，依据Eq.5得到2D协方差，再按相机方向评价SH颜色并取得深度。每个2D高斯有屏幕footprint。
3. CreateTiles：将图像切为16×16像素块。边缘tile可能不满16×16。对于1920×1080的例子，有120×68=8160个tile（背景计算）。
4. DuplicateWithKeys：一个高斯若覆盖多个tile，则生成多个引用实例，不是复制训练参数。每个实例key低32位编码投影深度，高位编码tile ID，最多64位。用一次GPU Radix sort按key全局排序，可使同tile的实例连续且内部深度有序。
5. IdentifyTileRanges：每个key和相邻key比较高位tile ID，识别对应tile连续区间。空tile无需遍历无关高斯。
6. BlendInOrder：每个tile一个thread block；线程协作把若干高斯数据加载进shared memory，tile中的像素线程按同一列表前→后遍历。每像素仍需计算二维高斯响应和alpha；不同像素可因覆盖不同而采用不同有效贡献。累积alpha接近1后停止，全部像素饱和时整个tile停止。

**算法边界**

```
Rasterize(w,h,M,S,C,A,V):
    CullGaussian(...,V)
    M2,S2 = ScreenspaceGaussians(M,S,V)
    tiles = CreateTiles(w,h)
    L,K = DuplicateWithKeys(M2,tiles)
    SortByKeys(K,L)
    ranges = IdentifyTileRanges(tiles,K)
    I = 0
    parallel for tile in tiles:
        parallel for pixel in tile:
            I[pixel] = BlendInOrder(pixel,L,ranges[tile],K,M2,S2,C,A)
    return I
```

该伪代码对应原Algorithm 2；颜色与opacity等量在Blend使用。前向每个像素按Eq.3积累颜色。相对于逐像素独立排序，复用tile排序、共享内存和并行协作是重要性能来源。若一个高斯跨多个tile，其引用实例量增加；复杂度依赖重叠规模、场景高斯数、像素贡献深度及终止条件，不能只写$O(N)$并认为与分辨率无关。

关键近似：以基元代表深度排序并在tile内共享，没有额外逐像素排序；当多个大或倾斜高斯在一个tile中交错时，实际每条视线的深度关系可能不同。作者认为splat接近像素大小时近似可忽略，但§7.4承认可出现popping。视锥/guard band的离散剔除同样可能造成视角变化时的不连续性。可微渲染是指固定当前有效集合及顺序下主要连续参数可回传梯度，不代表所有离散排序与裁剪决策全局光滑。

【证据边界】本文说明具体算法与运行帧率，但未提供任意场景的严格可见性误差上界，也未证明一个tile中所有像素的排序完全正确。

<!-- PTH:END METHOD-RASTER -->

<!-- PTH:BEGIN METHOD-BACKWARD -->
## 3.4.5 反向传播与深度复杂性

**反向机制**（§6、Appendix C、Figure 9）

训练需对每个实际混合的高斯计算图像损失梯度。若每像素前向按$1,2,\ldots,K$深度顺序合成，反向需要知道每一步剩余透射率和颜色贡献。直接保存每像素所有贡献列表与中间值，可能产生动态列表及大量显存。论文复用前向已排序的tile列表和区间，记录最终累积量以及每像素最后贡献基元的信息，反向从后往前遍历。对tile中最后贡献的深度之前的候选才进行像素覆盖/alpha测试，重新计算对应贡献和梯度。由此避免为每个像素存一条任意长链。

【补充推导】记前向透射率$T_i=\prod_{j<i}(1-a_j)$，结束后$T_{K+1}=\prod_{j\le K}(1-a_j)$。反向可用

$$T_i=\frac{T_{i+1}}{1-a_i}$$

逐步恢复。这是Eq.2–3在一致alpha定义下的代数关系。作者在§6与Appendix C文字说用逐个Gaussian的alpha做除法，且将累计量称为opacity；在本文统一符号下，实际应明确恢复的是剩余透射率并除以$(1-a_i)$，若用累积不透明度$A_i=1-T_i$需相应转换。此处保留原文措辞歧义并给出数学自洽解释，不宣称已核对源码。

反向沿颜色、alpha、投影协方差、均值等连续路径获得梯度并累加给原始基元索引；高斯出现在多个tile时并不是多个独立参数。原论文强调没有固定N个最前基元的硬梯度上限：只要在前向中有效贡献，就可参与反向。这比人为固定10个更适合不同场景/像素深度复杂性。

**数值稳定**（Appendix C，p.14）

- 前向和反向跳过alpha小于$1/255$的混合更新，减少极小贡献及保持两遍一致。
- alpha上限0.99，防止$1-a_i$为0并缓解逆向除法数值不稳定。
- 加入一个基元前预估累积opacity，在其超过0.9999之前停止前向混合，防止剩余透射率过小。

因此“unlimited depth complexity with gradients”不是无限遍历或没有任何阈值，而是不引入人为固定数量的梯度截断。前向实际参与的有限序列、alpha阈值和饱和仍决定梯度。Figure 9/Table 3表明限制前10个会严重降低质量，Truck 30K降低约11 dB。这是实际受控消融的证据，而非任何剪裁策略都不可能加速的理论证明。

训练需要反向及少量额外前向状态，推理仅前向渲染。§6称额外开销为每像素常数；这不意味着整个系统内存与场景规模无关，仍需保存排序实例及模型。

【证据边界】离散排序、候选剔除和早停均带来局部近似；论文未证明这些操作全局可微，亦未提供逆向除法的完整浮点误差界。

<!-- PTH:END METHOD-BACKWARD -->

<!-- PTH:BEGIN MATH-COMPOSITE -->
## 3.5.1 公式 1–3：体渲染与有序 alpha 合成

**公式1–3**（§2.3，PDF p.3）

公式 1（原文）

$$C=\sum_{i=1}^{N}T_i(1-\exp(-\sigma_i\delta_i))c_i,\qquad T_i=\exp\left(-\sum_{j=1}^{i-1}\sigma_j\delta_j\right).$$

$C,c_i\in\mathbb R^3$是像素RGB与采样点RGB；$\sigma_i\ge0$为沿射线第i段的体密度（长度倒数），$\delta_i\ge0$为段长，$T_i\in[0,1]$是从相机到第i段之前的透射率。$\sigma_i\delta_i$无量纲。N为按近到远排列的采样数。$\exp(-\sigma_i\delta_i)$是常密度段的剩余透射率，故$\alpha_i=1-\exp(-\sigma_i\delta_i)$是该段消除的透射比例。

【补充推导】采用NeRF式吸收—发射模型，射线$r(t)=o+td$（d单位向量），透射率满足$\frac{dT}{dt}=-\sigma(r(t))T$，$T(t_n)=1$，故

$$T(t)=\exp\left(-\int_{t_n}^{t}\sigma(r(u))du\right),\quad C=\int_{t_n}^{t_f}T(t)\sigma(r(t))c(r(t),d)dt.$$

在区间$[t_i,t_i+\delta_i]$内密度和颜色近似常量，$T(t_i+u)=T_i e^{-\sigma_i u}$。该段积分为

$$\int_0^{\delta_i}T_i\sigma_i e^{-\sigma_i u}c_i\,du=T_i(1-e^{-\sigma_i\delta_i})c_i.$$

逐段求和得公式1。背景若有固定颜色$b$，还可加$T_{N+1}b$；原式仅列积分贡献，不能假定所有实验背景均为黑色。$\sigma_i=0$时贡献为0，$\delta_i\to0$时$\alpha_i\simeq\sigma_i\delta_i$。

公式 2（原文合成式及符号辨析）

$$C=\sum_{i=1}^N T_i\alpha_i c_i.$$

由$1-\alpha_j=e^{-\sigma_j\delta_j}$，对第i段之前的所有段连乘：

$$T_i=\prod_{j=1}^{i-1}(1-\alpha_j),\quad T_1=1.$$

【原文核查】PDF p.3在Eq.2下方印出的乘积内下标为$\alpha_i$而非$\alpha_j$；已经查看该公式原始页面。按Eq.1指数展开及紧接的Eq.3可知乘积必须依赖前序各项j，本文明确记录这个下标问题并采用数学自洽的$\alpha_j$，并非静默重写作者公式。

公式 3（原文）

$$C=\sum_{i\in\mathcal N}c_i\alpha_i\prod_{j=1}^{i-1}(1-\alpha_j).$$

$\mathcal N$为与当前像素重叠、近到远排序的点基元索引集合；此处i、j按有序列表重新编号。本文用二维高斯值乘基元学习的opacity生成像素alpha，非按3D密度积分求得。以像素p、投影中心$u_i$、2D协方差$Q_i$、基元opacity $o_i$写为$a_i(p)=o_i\exp[-\frac12(p-u_i)^TQ_i^{-1}(p-u_i)]$，$c_i$由当前观察方向的SH评价。代入Eq.3即得到3DGS的合成形式（实际实现还包括阈值与早停）。

**机制与边界**

令权重$w_i=T_i\alpha_i$，则$w_i=T_i-T_{i+1}$，所以$\sum_i w_i=1-T_{N+1}\le1$；加固定背景项后各权重和为1。前→后顺序不可忽略：构造两个高斯A/B，颜色分别为(1,0,0)、(0,0,1)，alpha为0.6、0.5，背景黑。A先时权重0.6、0.2，像素(0.6,0,0.2)；B先时权重0.5、0.3，像素(0.3,0,0.5)，总opacity均0.8，但颜色不同。该例解释排序影响外观以及顺序交换可能导致popping。

【独立评价】Eq.2与Eq.3共享相同的有序alpha合成代数结构，不意味着3DGS对任意连续密度分布的真实射线积分完全等价。对给定alpha可形式化定义有效光学厚度$\tau=-\ln(1-\alpha)$；若任取段长$\delta$，得$\sigma_{eff}=\tau/\delta$，但$\delta$与物理空间分布不由二维splat唯一确定。3DGS不计算每个空间段的真实密度线积分；投影、排序与裁剪均存在近似。其高斯opacity不能直接解释为唯一的物理消光系数。

【证据边界】公式等价建立在相同有序alpha、颜色和背景约定下；无法据此证明真实三维几何、连续介质传输等价，或每像素排序严格正确。

<!-- PTH:END MATH-COMPOSITE -->

<!-- PTH:BEGIN MATH-GAUSS -->
## 3.5.2 公式 4–6：空间核、投影协方差与正定参数化

**公式4–6**（§4，PDF p.4）

公式 4（原文）

$$G(x)=\exp\left(-\frac12x^T\Sigma^{-1}x\right).$$

原文说明核中心为均值$\mu$；上式x应理解为相对于中心的位移。若绝对坐标为$X$，等价写$G(X)=\exp[-\frac12(X-\mu)^T\Sigma^{-1}(X-\mu)]$。$X,\mu\in\mathbb R^3$，$\Sigma\in\mathbb R^{3\times3}$为对称协方差；$\Sigma^{-1}$要求可逆，即实际需正定。指数中马氏距离的平方是无量纲。核最大值1，等值面为椭球，而非带归一化常数$(2\pi)^{-3/2}|\Sigma|^{-1/2}$的概率密度。opacity独立乘于核。

公式 5（原文）

$$\Sigma'=JW\Sigma W^TJ^T.$$

W表示视图变换作用于协方差的线性部分（若完整相机变换为齐次矩阵，则此处取旋转线性部分）；J为投影在高斯中心处的一阶仿射近似的Jacobian。原文将$\Sigma'$的第三行列去掉，得到屏幕2×2协方差。投影中心由相机投影求得。

【补充推导】若$X=\mu+\Delta X$，相机坐标$Y=W X+t$，有$\Delta Y=W\Delta X$。透视投影函数$\pi(Y)$在均值$\bar Y$附近线性化：$\pi(Y)\approx\pi(\bar Y)+J_2\Delta Y$，故对协方差取期望：

$$Q=\mathbb E[(J_2W\Delta X)(J_2W\Delta X)^T]=J_2W\Sigma W^TJ_2^T.$$

这是Eq.5的屏幕二维块，$J_2\in\mathbb R^{2\times3}$。对针孔相机$Y=(X_c,Y_c,Z_c)^T$，$u=f_xX_c/Z_c+c_x$，$v=f_yY_c/Z_c+c_y$，则

$$J_2=\begin{bmatrix}f_x/Z_c&0&-f_xX_c/Z_c^2\\0&f_y/Z_c&-f_yY_c/Z_c^2\end{bmatrix}.$$

由此可见深度越近，投影尺度通常越大；$Z_c\to0$时Jacobian不稳定，是§6 guard band/剔除的数值动机之一。该推导是假设局部针孔模型、无畸变及在中心线性化的背景解释，不是作者另提供的新公式。一般3D高斯经过非线性透视映射后的分布并非严格高斯，局部线性化对小尺度/远距离基元更合理。

公式 6（原文）

$$\Sigma=RSS^TR^T,\qquad S=\operatorname{diag}(s_x,s_y,s_z).$$

$R\in SO(3)$由单位四元数$q$构造，$s$为三个轴的尺度，经exp激活为正。对任意$v$，

$$v^T\Sigma v=\|S^TR^Tv\|_2^2\ge0.$$

当三轴尺度均正且R正交，非零v对应严格正值，所以$\Sigma$正定。反之，任意正定对称矩阵可谱分解为$R\operatorname{diag}(\lambda_i)R^T$，取$s_i=\sqrt{\lambda_i}$，并调整特征向量符号使$\det R=1$，故这种参数化足以表达任意正定协方差。直接更新协方差元素可能破坏正定性；通过旋转/尺度更新避免此问题。

**推导与条件**

等值面$G(X)=e^{-k^2/2}$等价于$(X-\mu)^T\Sigma^{-1}(X-\mu)=k^2$，变换到局部轴坐标$y=R^T(X-\mu)$得到$\sum_a y_a^2/s_a^2=k^2$，半轴长度为$ks_x,ks_y,ks_z$。这使各向异性具有明确几何意义：例如$(s_x,s_y,s_z)=(2,0.2,0.05)$时，核在x方向更长、z方向更薄，可近似细长/表面结构；若三轴相等则$\Sigma=s^2I$，R变得不可辨识。尺度越小，局部覆盖越小但可能需要更多基元；尺度过大可能跨过不连续边界并造成低细节。Figure 3、10和Table 3提供经验支持。

投影后Q若正定，可计算每像素$G_2(p)=\exp[-\frac12(p-u)^TQ^{-1}(p-u)]$。当W、J2满行秩且$\Sigma\succ0$，Q正定；若投影退化或数值精度不足，需采取实现中的稳定措施（论文未完整列出所有矩阵正则常数）。优化$\mu$既改变投影中心也改变J；s和q改变Q。Appendix A的Eq.8–11提供尺度与旋转的链式导数。

【证据边界】二维高斯为局部仿射投影近似，并非任意3D高斯沿透视光线积分的精确表达；高斯形状对齐有利于视觉拟合，但不能直接等同于显式真实表面法线或物理体密度。

<!-- PTH:END MATH-GAUSS -->

<!-- PTH:BEGIN MATH-SH-LOSS -->
## 3.5.3 SH颜色与公式 7：方向外观及图像监督

**SH与公式7**（§3、§5.1、§7.1）

【原文事实】每个高斯以spherical harmonics（球谐函数，SH）表示方向相关外观，训练从零阶项开始，每1000迭代引入一个频带，直至4个频带均使用。§7.3去掉SH时PSNR下降（Table 3）。原文没有展开球谐基的具体归一化常数、方向正负约定或存储排列。

【背景知识】在常见实球谐基展开下，若最高阶为L，方向单位向量$d\in\mathbb S^2$，可写

$$c_i(d)=\sum_{\ell=0}^{L}\sum_{m=-\ell}^{\ell}h_{i,\ell m}Y_{\ell m}(d),\qquad h_{i,\ell m}\in\mathbb R^3.$$

0阶$Y_{00}$为常数，对应不随方向变化的基础颜色；高阶描述不同方向的变化。若4频带为$\ell=0,1,2,3$，每通道系数数量$1+3+5+7=16$，RGB合计48个。这是标准基展开的数学维数推断，不宣称已查原始源码的实际内存布局、激活或颜色裁剪。对于某固定高斯，所有相机观察方向共同约束SH系数；角度覆盖不充分时，高阶分量与零阶颜色可相互补偿。分阶段加入高阶有助于先建立基础色，但不是消除角度不辨识的严格保证。

公式 7（原文，§5.1，p.5）

$$\mathcal L=(1-\lambda)\mathcal L_1+\lambda\mathcal L_{D\text{-}SSIM},\qquad \lambda=0.2.$$

$\mathcal L_1$为渲染图与观测图的L1像素颜色误差（具体求和/平均实现未明示）；D-SSIM为基于SSIM的结构不相似性。常见SSIM以局部均值、方差、协方差比较亮度、对比度和结构，D-SSIM随相似度减小而增大；原论文未给其完整归一化公式，因此不擅自选择$1-\mathrm{SSIM}$或$(1-\mathrm{SSIM})/2$作为作者具体实现。Eq.7即0.8 L1 + 0.2 D-SSIM。该权重是作者全部测试经验设置。

**训练数学**

颜色$C(p)=\sum_i w_i(p)c_i(d)$，在排序、alpha固定时，SH系数影响图像的导数为

$$\frac{\partial C(p)}{\partial h_{i,\ell m}}=w_i(p)Y_{\ell m}(d)I_3.$$

因此有可见性遮挡时$w_i$小，对颜色系数的监督弱；观察方向覆盖少时SH基矩阵可能近似低秩，即使像素很多也不意味着方向信息充分。位置、尺度、旋转、opacity则通过权重变化影响颜色，SH方向也可能依赖位置。真实训练中L1与局部D-SSIM的像素梯度经合成器和投影传播。若$\alpha=\operatorname{sigmoid}(\rho)$，$d\alpha/d\rho=\alpha(1-\alpha)$；若$s=e^\eta$，$ds/d\eta=s$。这些链式因子解释有效参数约束和饱和时梯度特征。以上为补充推导，不是作者另编号的SH梯度公式。

【数值例子】同一像素误差可能由RGB预测不正确，也可能由高斯位置错或opacity不足引起。若只调SH能降低训练视角误差，就不保证位置得到正确修正；角度缺失时外推视角可能失败。逐步加SH和密度控制可改变优化路径，仍不提供唯一物理解耦。No-SH的30K三场景平均25.35 dB vs Full 26.05，支持方向外观表示有用；并不表明SH显式估计了BRDF或光照。

【证据边界】RGB重建与结构相似性不是几何监督；未验证SH系数的物理光谱含义、独立光照与材质分解。

<!-- PTH:END MATH-SH-LOSS -->

<!-- PTH:BEGIN MATH-GRAD -->
## 3.5.4 公式 8–11：协方差的尺度与四元数导数

**公式8–11**（Appendix A，PDF p.13）

原文以$\Sigma,\Sigma'$为世界/视空间协方差，以$s\in\mathbb R^3$和$q\in\mathbb R^4$为尺度与旋转。记$U=JW$，屏幕块$Q=U_2\Sigma U_2^T$，$U_2$为U的前两行。原文Eq.8、9：

$$\frac{d\Sigma'}{ds}=\frac{d\Sigma'}{d\Sigma}\frac{d\Sigma}{ds}\tag{8}$$

$$\frac{d\Sigma'}{dq}=\frac{d\Sigma'}{d\Sigma}\frac{d\Sigma}{dq}\tag{9}$$

这表示复合函数Jacobian收缩而非不考虑维度的普通矩阵乘法。对屏幕坐标a,b=1,2，世界坐标i,j=1,2,3：

$$Q_{ab}=\sum_{i,j}U_{ai}\Sigma_{ij}U_{bj},\quad \frac{\partial Q_{ab}}{\partial\Sigma_{ij}}=U_{ai}U_{bj}.$$

对独立对称变量$\Sigma_{ij}$（i≠j同时修改$\Sigma_{ji}$），导数应包括$U_{aj}U_{bi}$；对角无需重复。原文以紧凑对称矩阵写出导数；这里用分量表达消除全矩阵与独立三角元素的口径歧义。定义$M=RS$，$\Sigma=MM^T$。微分：

$$d\Sigma=dM M^T+M dM^T.$$

若上游标量损失对$\Sigma$的Frobenius梯度为H，有$dL=\operatorname{tr}(H^Td\Sigma)$，所以

$$\nabla_M L=(H+H^T)M=2HM\quad(H=H^T).$$

原文用$\frac{d\Sigma}{dM}=2M^T$紧凑表达对称梯度链；不能将其当成9×9 Jacobian的完整普通矩阵。上游若给$\nabla_Q L=B$，则$H=U_2^T B U_2$（固定U时），因此可高效反传。

尺度导数：$M_{ij}=R_{ij}s_j$，

$$\frac{\partial M_{ij}}{\partial s_k}=\begin{cases}R_{ik},&j=k\\0,&j\ne k\end{cases}.$$

即尺度只影响M第k列；$\partial L/\partial s_k=\sum_i(\nabla_M L)_{ik}R_{ik}$，log-scale参数再乘$s_k$。等价地$\Sigma=\sum_k s_k^2r_kr_k^T$，$\partial\Sigma/\partial s_k=2s_kr_kr_k^T$，其中$r_k$为R的第k列。

公式 10（原文单位四元数旋转矩阵）。令$q=(q_r,q_i,q_j,q_k)$，r为实部，i,j,k为虚部：

$$R(q)=2\begin{bmatrix}\frac12-q_j^2-q_k^2&q_iq_j-q_rq_k&q_iq_k+q_rq_j\\q_iq_j+q_rq_k&\frac12-q_i^2-q_k^2&q_jq_k-q_rq_i\\q_iq_k-q_rq_j&q_jq_k+q_rq_i&\frac12-q_i^2-q_j^2\end{bmatrix}.\tag{10}$$

计算$\partial M/\partial q_a=(\partial R/\partial q_a)S$，即R各元素逐项求导、每列乘对应尺度。原文公式11给出四个矩阵：

$$\frac{\partial M}{\partial q_r}=2\begin{bmatrix}0&-s_yq_k&s_zq_j\\s_xq_k&0&-s_zq_i\\-s_xq_j&s_yq_i&0\end{bmatrix},$$

$$\frac{\partial M}{\partial q_i}=2\begin{bmatrix}0&s_yq_j&s_zq_k\\s_xq_j&-2s_yq_i&-s_zq_r\\s_xq_k&s_yq_r&-2s_zq_i\end{bmatrix},$$

$$\frac{\partial M}{\partial q_j}=2\begin{bmatrix}-2s_xq_j&s_yq_i&s_zq_r\\s_xq_i&0&s_zq_k\\-s_xq_r&s_yq_k&-2s_zq_j\end{bmatrix},$$

$$\frac{\partial M}{\partial q_k}=2\begin{bmatrix}-2s_xq_k&-s_yq_r&s_zq_i\\s_xq_r&-2s_yq_k&s_zq_j\\s_xq_i&s_yq_j&0\end{bmatrix}.\tag{11}$$

例如$M_{12}=2(q_iq_j-q_rq_k)s_y$，对$q_r$求导为$-2s_yq_k$、对$q_i$为$2s_yq_j$，对应上式第一行第二列。对角$M_{22}=2(1/2-q_i^2-q_k^2)s_y$，对$q_k$为$-4s_yq_k$，对应第四矩阵。其他项逐元素同理。标量损失的$q_a$梯度是$\langle\nabla_M L,\partial M/\partial q_a\rangle_F$。

**梯度与条件**

原文说四元数归一化梯度可直接推导。设未归一化参数$u\in\mathbb R^4$非零，$q=u/\|u\|$，则

$$\frac{\partial q}{\partial u}=\frac{1}{\|u\|}(I_4-qq^T),\qquad \nabla_u L=\frac{(I_4-qq^T)\nabla_q L}{\|u\|}.$$

因为$d\|u\|=q^Tdu$，代入$d(u/\|u\|)$得到上式。这个Jacobian投影到单位球切空间，抑制仅改变范数的方向；u趋近0不稳定，实际归一化需避免该退化。q与−q表示同一旋转；当尺度相同或存在重复特征值时，部分旋转不改变协方差，梯度可能不可辨识。

维度链：$Q$ 2×2（独立3元素），$\Sigma$ 3×3（独立6元素），M 3×3，s 3，q 4；梯度经$B\to H\to\nabla_M\to(s,q)$，不是把矩阵导数随意相乘。上面固定U解释尺度/旋转；对位置均值的梯度还包含投影中心及U=J(\mu)W变化，Eq.8/9本身不完整描述这一路径。作者采用显式导数以降低自动微分开销，Appendix A给出这些尺度、旋转导数，未给所有CUDA实现和数值常数。

【证据边界】Eq.10/11针对单位四元数旋转及链式归一化；不证明全局优化收敛或协方差能唯一从图像恢复。

<!-- PTH:END MATH-GRAD -->

<!-- PTH:BEGIN MATH-ALPHA-GRAD -->
## 3.5.5 Alpha 合成梯度与反向重构（补充推导）

**前后向合成数学**（§2.3、§6、Appendix C）

为理解原文逆向遍历，考虑固定排序的K个有效splat、背景$b\in\mathbb R^3$：

$$T_1=1,\quad T_{i+1}=T_i(1-a_i),\quad C=\sum_{i=1}^{K}T_i a_i c_i+T_{K+1}b.$$

令$B_i$为以单位入射透射率开始、从第i项到背景的相对合成颜色，$B_{K+1}=b$，$B_i=a_ic_i+(1-a_i)B_{i+1}$。于是$C=\sum_{j<i}T_ja_jc_j+T_i B_i$。固定顺序时$T_i$与当前$a_i$无关，得到

$$\frac{\partial C}{\partial c_i}=T_i a_i I_3,\qquad \frac{\partial C}{\partial a_i}=T_i(c_i-B_{i+1}).$$

前式表示颜色梯度按该高斯实际贡献权重缩放；后式表示提高当前alpha既增加自身颜色、又遮挡后方颜色，取决于自身与后方相对合成颜色之差。若$g=\nabla_C L\in\mathbb R^3$，则$\nabla_{c_i}L=T_i a_i g$，$\partial L/\partial a_i=T_i g^T(c_i-B_{i+1})$。这说明背后基元虽被部分遮挡仍可得到梯度，而当颜色相同或透射率几乎为0时，alpha梯度可很小。原文未单独编号这组公式；它们是从Eq.3推导的理解辅助。

二维高斯$G=\exp(-\frac12d^TQ^{-1}d)$、$d=p-u$、$a=oG$（未截断时），则

$$\frac{\partial a}{\partial o}=G,\quad \nabla_u a=aQ^{-1}d,\quad \nabla_Q a=\frac{a}{2}Q^{-1}dd^TQ^{-1}.$$

推导：$d(d^TQ^{-1}d)/du=-2Q^{-1}d$（对称Q）；$dQ^{-1}=-Q^{-1}(dQ)Q^{-1}$，代入指数并以Frobenius内积识别梯度。上游$\partial L/\partial a$分别乘三条局部导数，再经投影中心和Eq.5/8–11影响世界均值、尺度、旋转。方向SH也可因位置改变观察方向；排序与裁剪在当前局部固定的假设下没有把离散开关当连续导数。

**稳定性**

前向遍历得到最终透射率，反向通过$T_i=T_{i+1}/(1-a_i)$恢复中间值，同时从背景开始更新$B_i$。因此不需要为每像素保存完整T列表；前向需记录最后有效基元边界和最终透射（或等价累积不透明度），复用tile的有序索引。

例：两个高斯alpha分别0.6、0.5，背景黑。$T_1=1,T_2=0.4,T_3=0.2$。逆向先$0.2/(1-0.5)=0.4$，再$0.4/(1-0.6)=1$；若第二基元RGB为蓝$B_2=(0,0,0.5)$、第一基元红$c_1=(1,0,0)$，则$\partial C/\partial a_1=(1,0,-0.5)$。如果把累计opacity 0.8直接除以各alpha，不能恢复上述透射率；§6/附录C的文字歧义必须用一致符号解释。未核对源码变量名与实现。

附录C规定跳过$a<1/255$，将a截断至0.99，并在加入后累计opacity可能超过0.9999时停止。$1-a\ge0.01$避免除零，剩余透射不至于在实际参与序列中无限减小；重复除法仍可能传播浮点误差。硬裁剪使函数分段定义，阈值处可能不连续，数学导数仅针对当前有效集合、非阈值边界的连续变化。限制前N个梯度还会省略已贡献的后方基元导数；Figure 9的N=10试验显示明显质量下降。

【证据边界】以上为固定排序下的补充推导，和论文光栅器设计/稳定规则一致，未声称等同于全部CUDA细节或严格全局可微。

<!-- PTH:END MATH-ALPHA-GRAD -->

<!-- PTH:BEGIN EXP-REAL -->
## 3.6.1 真实场景总体评估

**真实数据协议**（§7.1–2，p.8–9）

研究问题：完整场景的NVS能否在竞争性的训练时间内优化到高质量且实现实时显示？数据集包括Mip-NeRF360全部9场景（bicycle、flowers、garden、stump、treehill、room、counter、kitchen、bonsai），Tanks&Temples的Truck/Train及Deep Blending的DrJohnson/Playroom，共13实景，覆盖室内有界和室外无界环境、不同采集方式。每8张图留出一张测试，其余图训练；已知标定相机，典型SfM点初始化，RGB监督Eq.7。测试以原有输入分辨率渲染，数据集间分辨率可不同，不将所有实测帧率假设为完全相同1080p分辨率。作者以同一超参数配置进行整套评估。

| 基线/配置 | 角色及协议 |
|---|---|
| Mip-NeRF360 | 高质量基准；自身数据集质量数值†引自原论文；作者自跑图像及分场景指标另见附录D |
| InstantNGP Base/Big | 快速辐射场基准；Base运行35K迭代，Big为作者建议的更大网络 |
| Plenoxels | 无神经网络的稀疏体素辐射场，快速基准 |
| Ours-7K、Ours-30K | 同方法两种迭代预算，考察质量、耗时、FPS、模型存储的折衷 |

除Mip-NeRF360外作者报告在A6000 GPU上运行。Mip-NeRF360训练为4块A100、12小时，折合48 GPU小时（A100比A6000快，不能当严格同硬件单卡实测）。原文使用SIBR交互式查看器测帧率。评价指标为PSNR(dB)与SSIM↑、LPIPS↓，另报告训练时间、渲染速度和优化参数存储。没有真实几何GT或深度误差作为核心指标。

**结果解释**

Table 1中，30K在Mip-NeRF360/T&T/DB分别取得134/154/137 fps、41m33s/26m54s/36m2s训练，质量可与高质量基准竞争：

| 数据集 | PSNR（ours / M-NeRF360） | SSIM（ours / M-NeRF360） | LPIPS（ours / M-NeRF360） |
|---|---:|---:|---:|
| Mip-NeRF360 | 27.21 / 27.69† | .815 / .792† | .214 / .237† |
| T&T | 23.14 / 22.22 | .841 / .759 | .183 / .257 |
| DB | 29.41 / 29.40 | .903 / .901 | .243 / .245 |

Mip-NeRF360平均PSNR落后但感知/结构指标领先；附录D作者自跑平均27.58/.790/.240也存在同样趋势。Table 4–9进一步显示Bicycle等部分场景的PSNR领先，而Kitchen/Bonsai等落后；DB两场景平均相当但DrJohnson在三指标上均略劣于Mip-NeRF360，Playroom更优。因此作者“on par and sometimes slightly better”的定性总结比“全面领先”准确。

7K约4m35s–6m55s，帧率160–197，质量有时已与快速基线竞争，但T&T的7K PSNR低于InstantNGP Base/Big。Figure 6说明某些场景短训练已有较好前景，进一步训练降低背景伪影，另一些视觉差别较小。30K模型参数存储734/411/676MB，较7K 523/270/386MB更大；对比M-NeRF360 8.6MB尤其明显。不能把参数存储量直接作为训练峰值显存。

Figure 5的九行留出视图支持细节质量，未覆盖全部13实景；完整分场景表提供更全面证据。论文还称补充视频展示远离训练照片的路径，但本任务仅核查实际PDF/HTML，不宣称看过未提供的视频或全测试渲染。

【证据边界】质量、速度与存储均受分辨率、硬件和实现影响。不存在重复试验置信区间、完整稀疏视角控制、几何真值测试或动态场景验证。总体对比同时改变表示、训练与渲染方式，不能单独归因于某个数学项。

本单元聚焦该阶段的概念和算法机制，完整公式及必要推导另见数学单元。

<!-- PTH:END EXP-REAL -->

<!-- PTH:BEGIN EXP-SYNTH -->
## 3.6.2 有界合成场景与随机初始化

**合成数据协议**（§7.2，p.9、Table 2）

研究问题：方法是否一定依赖真实场景SfM点？Synthetic Blender（NeRF synthetic）提供准确相机、覆盖较充分的视图、有限物体空间范围，与真实无界场景存在关键条件差异。作者在包含场景边界的体积内均匀随机初始化100,000个高斯，后续借助图像监督与剪枝把无用基元去掉，约保留6–10K有意义基元；再优化及增密，30K迭代模型约200–500K高斯/场景。没有MVS点或真实几何监督。为与已有论文数字兼容，渲染使用白色背景。

**结果和条件**

Table 2的八个场景平均PSNR：Plenoxels 31.76、INGP-Base 33.18、Mip-NeRF 33.09、Point-NeRF 33.30、Ours-30K 33.32 dB。作者报告合成场景训练后180–300 fps。Chair/Drums/Hotdog列本文最高；平均略高不意味着全部场景最优，尤其Ficus的Point-NeRF 36.13高于本文34.87。表中比较方法质量来自各自论文，非作者统一复跑；速度未逐个对照列出。

与Table 3/Figure 7的真实场景随机初始化比较：真实消融的立方体由相机包围盒尺度决定，远景无界、背景及视角覆盖不均；合成已知有限范围与较充分角度使初始化错误更易由RGB约束剪除并增密。作者明确观察真实随机方案主要恶化背景且低覆盖区域有floaters。于是本实验支持有利观测条件下不依赖SfM的可行性，并不推翻真实数据初始化的消融结论。

【独立评价】随机初始化效果取决于候选空间范围、覆盖、位姿和场景背景等多个变量，两个不同数据集的对比不能单独估计“有界性”或“视角数量”的因果贡献。该论文没有分别控制这些变量。

【证据边界】本文表2只有PSNR，未从合成GT提取深度或真实几何误差；相机准确不等于三维高斯必然重建为真实表面。

本单元聚焦该阶段的概念和算法机制，完整公式及必要推导另见数学单元。

<!-- PTH:END EXP-SYNTH -->

<!-- PTH:BEGIN EXP-COMPACT -->
## 3.6.3 表示紧凑性实验

**紧凑性实验**（§7.2 Compactness，p.9）

作者将本方法与Zhang et al. 2022报告的紧凑点基模型进行比较，问题不是所有方法的全局存储排序，而是在相近图像质量下，各向异性高斯能否用更少基元表达复杂形状。为提高可比性，实验使用对方的初始点云（由前景mask进行space carving获得），优化直到达到对方报告的PSNR，通常耗时2–4分钟。作者说该实验使用其SH的“两阶”（two degrees），与对方类似；不将此措辞擅自改成精确系数存储布局。此设置不同于真实完整场景标准SfM初始化及四频带SH。

**结果和口径**

作者报告达到/超过对方指标时约使用其四分之一点数，平均模型大小3.8 MB而非9 MB。以原数字计算，模型存储比例3.8/9≈42.2%，节省约57.8%；点数约25%，二者比例不同是可能的，因为不同基元属性与颜色系数导致每个元素存储代价不同，原文没有逐字段字节预算，不进一步反推精确单点存储。

机制上，大型各向异性高斯能够沿均匀表面或细长形状覆盖更大的有效范围，同一质量下可能比固定大小点需要更少元素；Figure 3给直观示例、Figure 10提供控制Ficus基元上界的质量对照。紧凑性段落报告的是与特定点基方案的匹配质量比较，而非所有基元数下的完整质量—模型大小曲线；论文没有给该实验逐场景PSNR、方差或完整FPS。

【证据边界】不能把3.8MB作为所有3DGS模型大小。真实场景Table 1的30K模型数百MB，Mip-NeRF360仅8.6MB；大型场景训练峰值可超过20GB。空间显式结构的运行效率与模型存储之间存在折衷，且前景mask初始化不等价于无界场景。

<!-- PTH:END EXP-COMPACT -->

<!-- PTH:BEGIN EXP-ABLATION -->
## 3.6.4 消融：每项设计实际贡献的证据

**消融协议**（§7.3、Table 3，p.9–10）

作者测试真实Truck、Garden、Bicycle，采用5K和30K两种训练预算，针对SfM初始化、clone、split、SH、各向异性及反向传播范围修改方法。特别说明该组输入由高分辨率原图手工降采样到既定渲染分辨率，避免原预降采样图的JPEG随机伪影；所以与Table 4–9同名场景的PSNR不应混算。Full为该协议完整对照，其他设置保持其余部分不变（作者明确说明分别禁用增密分支）。

**结果解释**

| 实验 | 修改与针对问题 | Average 30K（Full=26.05 dB） | 辅助证据/例外 |
|---|---|---:|---|
| Limited-BW | 只给最前10个贡献点梯度，检验反向范围 | 19.19（−6.86） | Figure 9 Garden模糊；Truck降10.97 dB，长训练甚至更差 |
| Random Init | 相机bbox范围3倍立方体内均匀随机点，检验初始空间支持 | 20.42（−5.63） | Figure 7背景变差；与有界合成随机成功并不矛盾 |
| No-Split | 关闭大基元分裂，检验局部细化 | 23.90（−2.15） | Figure 8后景模糊，三场景30K均降低 |
| No-SH | 去掉方向相关SH，检验视角外观 | 25.35（−0.70） | 30K三个场景均下降；不证明真实BRDF恢复 |
| No-Clone | 关闭小基元复制，检验覆盖不足 | 25.91（−0.14） | Figure 8细轮辐；Truck30K 24.82略高于Full 24.81 |
| Isotropic | 单尺度控制三个轴，禁用各向异性 | 25.23（−0.82） | Figure 10细结构；Ficus两种设置最多5000个高斯 |

括号为变体−Full，与Table 3单元中Full−变体的正向差值是同一差异的相反符号，这里显式标示口径。各设置平均影响的排序仅限本组PSNR及迭代预算。Limited-BW不是缩小前向列表；作者认为严重梯度近似使训练不稳定，实验只测试N=10（论文称为Pulsar默认数量两倍）。No-Split与No-Clone都是改变增密规则，可能同时改变最终基元数量和布局；不能在没有固定基元数量实验时把差值仅归因于一种几何属性。Isotropic的Ficus定性展示特意控制基元数上限，而Table 3没有说明每个场景所有设置都严格同数量。

5K的No-Clone平均差0.55 dB，30K差0.14 dB；其作用可能与早期收敛和细结构有关，而不是只看最终全图平均。No-Split在30K仍有2.15 dB平均差；较长训练不能完全补偿少一种拓扑更新。对于SH，颜色依赖方向的建模可改善像素拟合，但不能从No-SH结果认定视角相关外观物理上已与几何分离。

【证据边界】此组对照提供设计有效性的经验支持，不是证明各组件数学必需。没有报告随机种子重复、误差条、每种消融的FPS、最终高斯数或训练显存；不将0.01 dB差异过度解读为统计显著。

本单元聚焦该阶段的概念和算法机制，完整公式及必要推导另见数学单元。

<!-- PTH:END EXP-ABLATION -->

<!-- PTH:BEGIN EXP-IMPLEMENT -->
## 3.6.5 实现细节、附录与失败案例

**附录A–D**（p.13–14，结合§7.1及§7.4）

- A Details of Gradient Computation：Eq.8–11给出屏幕协方差经世界协方差到尺度/四元数的导数；精读3.5.4保留公式和补充归一化梯度。显式梯度减少通用自动微分开销，具体CUDA性能未单独量化。
- B Optimization and Densification：Algorithm 1为SfM/属性初始化、随机训练视图、Rasterize、Loss、Adam及剪枝/增密循环；§5.2补充每100步、阈值0.0002、分裂因子1.6、3000步opacity reset。完整优化过程见3.4.2–3.4.3。
- C Details of the Rasterizer：16×16 tile，最多64 bit key（低32深度、高位tile ID），一次GPU Radix sort，相邻键寻找区间；Algorithm 2给出并行前向。数值稳定：忽略alpha<1/255，上截断0.99，预计累计opacity超过0.9999前停止；逆向恢复中间透射率，见3.4.4–3.5.5。附录中累计opacity除alpha的措辞与一致的透射率定义有歧义，已从合成递推辨析而非静默改写。
- D Per-Scene Error Metrics：Table 4–9逐场景质量，原论文†与作者自跑Mip-NeRF360分列。作者复跑Mip-NeRF360数据集平均PSNR27.58、SSIM0.790、LPIPS0.240，支持用实际复现图像解释定性比较。

原型为Python/PyTorch与自定义CUDA rasterization kernels、NVIDIA CUB radix sorting，SIBR用于交互式查看器测FPS。分辨率warm-up 250/500步逐级提高；SH从零阶逐步每1000步加频带；位置学习率指数衰减；同评估超参不代表不同大型城市场景无需调整，§7.4说早期实验可能需减小位置学习率。论文未给完整所有常数或源码级消融，本报告未额外查代码或补充视频。

**成本与失效**（§7.4–8，p.10–11）

1. 不充分观测：Figure 11的Train场景中，Mip-NeRF360前景floaters/颗粒感，本文背景粗大各向异性基元导致细节低；Figure 12低训练视角重叠时两者均出现伪影。作者承认可能产生拉长、斑块式高斯。
2. popping：较大高斯、视角相关外观区域易见跳变；guard band剔除与共享近似深度排序改变混合顺序是作者指出的原因。未来更原则性的culling、antialiasing和regularization可能有益，但本篇没有验证这些改进。
3. 内存：大型场景训练峰值超过20GB；推理需要数百MB参数存储及30–500MB额外光栅器内存。Table 1与紧凑性实验应区分隐式基线、显式点基对照及训练内存。
4. 训练时间：§8约80%在Python，只有光栅化为优化CUDA；完全CUDA化可能加速，但不应当作已报告实现结果。压缩和mesh重建也属于未来工作。

【证据边界】算法的高并行性和近似设计有实际运行表现，但未提供所有GPU/分辨率/场景的确定性成本上界、排序误差界、稀疏视角系统测试或真实深度精度；没有动态场景或参与介质实验。

本单元聚焦该阶段的概念和算法机制，完整公式及必要推导另见数学单元。

<!-- PTH:END EXP-IMPLEMENT -->

<!-- PTH:BEGIN SYNTHESIS-CLAIMS -->
## 4.1 主张—证据矩阵与综合判断

**主张证据矩阵**

| 作者贡献/主张 | 直接证据 | 支持程度与替代解释 | 尚未验证 |
|---|---|---|---|
| 各向异性3D高斯能有效表示复杂场景 | Eq.4–6、Figure 3/10、Table 3 Isotropic、紧凑性段落 | 数学上有效正定参数化与局部投影；经验上禁各向异性质量下降，同上界Ficus细节改善；模型量和优化差异也可能影响 | 真几何精度、物理体密度可辨识性、所有场景固定基元数下的质量曲线 |
| 参数优化与自适应密度控制构成有效学习方案 | §5、Algorithm 1、Figure 4/7/8、Table 2/3 | 两类增密互补，关闭split影响明显，clone支持细结构；SfM降低实景优化困难，但有界合成可随机；不等于全局最优 | 梯度阈值的普适性、跨种子稳定性、最终几何是否唯一、极少视角 |
| 快速考虑可见性的可微光栅器 | §6、Appendix C/Algorithm 2、Figure 9、Table 1/3 | 有序alpha合成、tile排序、共享内存和逆向遍历机制明确；N=10梯度限制明显降低质量；全系统实际高FPS | 精确逐像素可见性、硬件无关速度保证、梯度截断所有备选、排序离散点全局可微 |
| 高质量与竞争训练时间、实时推理兼得 | Figure 1/5/6、Table 1/4–9 | 给定数据协议下整体支持，短训练有例外，Mip360平均PSNR略低；DB接近；模型内存更大 | 动态、稀疏输入、真实几何及所有分辨率/硬件普遍优势 |

**理论与实验边界**

3DGS理论上的关键连接是Eq.1–3的有序alpha合成形式；在相同alpha、颜色、顺序下代数权重一致，而NeRF的alpha来自沿ray密度积分，3DGS的alpha来自投影核乘学习opacity。本文提供参数化、投影及梯度等数学机制，不提供任意连续参与介质的严格物理等价证明。需要在原文中明确两处符号辨析：Eq.2乘积下标印为alpha_i，但指数展开和Eq.3要求alpha_j；§6/Appendix C描述累计opacity除alpha，按一致的透射率递推应除以1−alpha。两处均标注为文本/数学辨析，未宣称源码有错误。

作者的视觉与运行性能结果支持技术方案，但速度来自整体系统改变而不是只靠“显式高斯”，且受硬件和分辨率限制。图像质量高不推出三维几何、物理辐射场参数唯一正确：有限训练投影使形状、opacity和方向色可以相互补偿，低重叠视角反例证实外推仍可能失效。

【独立评价】最有证据支持的创新是把可优化的各向异性空间基元、结构自适应更新、有序可微GPU光栅化联合用于完整场景，而不是单个公式的原创性。方法兼顾优化连续性和显式渲染组织方式；核心研究贡献既是表示选择，也是与其匹配的训练和系统算法。作者的历史首创/领先主张按本论文和数据语境理解，不作独立现代排名。

【证据边界】本篇不含独立跨文献全文核验、真实几何指标或受控极稀疏视角测试；不把作者主张、公式联系与外部物理事实混为一谈。

本单元聚焦该阶段的概念和算法机制，完整公式及必要推导另见数学单元。

<!-- PTH:END SYNTHESIS-CLAIMS -->

<!-- PTH:BEGIN SYNTHESIS-LIMITS -->
## 4.2 完整技术边界、尚未解决问题与总结

**技术边界**

| 假设/条件 | 若条件不足可能发生什么 | 原文证据或解释 |
|---|---|---|
| 静态场景与相机标定 | 位姿偏差或动态变化可使同一基元无法同时解释各视图；未测试这种设置 | §3静态图像与标定相机；背景推断 |
| 训练视图空间、角度覆盖 | 未见区域外推伪影、粗大或拉长高斯、SH零阶误估 | §7.1、7.4、Figure 11–12 |
| 基元相对小、共享深度顺序足够合理 | 大基元交错时排序近似；视角切换popping | §6、Appendix C、§7.4 |
| GPU能容纳模型及光栅化实例 | 大型场景模型和训练显存较高；场景复杂度改变FPS | Table 1、§7.4 |
| RGB图像监督足以支持期望的新视角 | 位置、opacity、方向色存在耦合；不保证真实深度或介质 | §5.1投影歧义，Figure 12；补充分析 |
| 启发式超参数适配场景 | 超大场景可能需降位置学习率，固定阈值不保证全局收敛 | §5.2、§7.4 |

**综合理解**

一条完整技术链是：多张静态场景照片和相机标定→SfM点建立具有初始尺度的高斯集合→透视局部仿射投影三维均值/协方差→当前视角SH颜色与逐像素alpha→tile深度排序及前向有序合成→与训练图比较（L1+D-SSIM）→反向复用排序列表恢复透射、更新均值/尺度/旋转/opacity/SH→周期clone、split、prune、reset改变空间覆盖→固定训练后的集合对任意给定相机渲染。各组件的目标不是恢复显式mesh，而是可学习、可投影且能生成高质量图像的辐射场表示。真实场景的初始化与有界合成例外、排序近似、稀疏观测失败均是这条链的适用边界。

作者尚未解决或提出的方向：更可靠的剔除与抗混叠以减少popping；为未充分观测与排序问题增加正则化；更优化的CUDA训练实现；显式参数压缩；高斯与真实mesh表面关系。以上属于§7.4–8作者明确局限与未来工作。并非本文已完成的方法；进一步研究假设与可证伪问题应和原文结论分开。

### 证据索引与实际来源

| 内容 | PDF物理页 |
|---|---|
| 标题、摘要、Introduction、贡献 | 1–2 |
| Related Work、Eq.1–3 | 2–4（Eq.1–3在p.3） |
| Overview、Gaussian及Eq.4–6 | 4 |
| 优化、Eq.7、密度控制、Figure 2–4 | 4–6 |
| 光栅器与反向遍历 | 6、13–14 |
| 真实/合成实验、Table 1–3、Figure 5–10 | 7–11 |
| 局限、Figure 11–12、结论 | 10–11 |
| 引用文献列表 | 11–13 |
| Appendix A–D、Eq.8–11、Algorithm 1–2、Table 4–9 | 13–14 |

本报告使用用户提供的arXiv:2308.04079v1 PDF（SHA-256 `f4c0c9f27e2d02b0265017f66049d471eb66cb50203bc918f51ba07d20cbbe11`）及同版本HTML辅助解析。DOI为10.1145/3592433；论文身份官方页面已核对，官方版本历史仅缓存v1，最新状态未实时确认；DOI页面访问受限。未以项目页可能变化的摘要替代本地PDF。未查其他论文全文、源码或未提供的补充视频，未使用外部图片。图表全部源自同版PDF及已核验展示副本；原件保持不变。

【证据边界】本报告将作者明确陈述、背景数学、补充推导、独立评价及未证实内容区分。论文没有验证动态、水下介质或稀疏极端视角任务；进一步推断不作为论文已证实的事实。

本单元聚焦该阶段的概念和算法机制，完整公式及必要推导另见数学单元。

<!-- PTH:END SYNTHESIS-LIMITS -->



---

# 第二部分：学习与研究笔记

# 3DGS 学习与研究笔记

> 精读报告 SHA-256：753502c1dd138b783b73a292621c90f06107f0fc11b125d024ddc3dc5bf0f69d
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：关键概念内化与拓展

本节以精读报告3.4–3.5为基础，将论文机制转化为可复用的数学与科研理解。下面推导和构造例子是学习拓展，不是作者额外实验。

### 1. 局部连续与全局离散是两个不同层次

把三维高斯理解为“带空间尺度的可学习粒子”，比把它理解成无形状的点更准确。它的中心和协方差决定空间响应，opacity决定局部影响强度，SH决定方向颜色。整个集合由离散有限索引组成；单个核却是光滑函数。因而“显式”不等于“不连续”，“连续”也不等于“隐式神经网络”。原论文§2.3–4和§8强调这种结构适配光栅化与优化。

**反事实设计**：若只保留中心和RGB、统一尺度固定，参数减少但基元不能适应细长结构，可能需要更高密度；若保留完整高斯但不改变集合大小，则初始化缺失区域仍难兼顾覆盖；若采用连续密度场对每条ray积分，则能赋予明确的连续介质含义，却失去直接按tile投影的计算组织。比较设计时应分别控制表示、监督、渲染与基元数量，不能将其捆绑为“显式必优”。

### 2. 投影高斯与真实线积分的区别

精读3.5.1–2指出，3DGS用投影核乘学习opacity得到像素alpha，与NeRF的$1-e^{-\sigma\delta}$共享合成公式但来源不同。可进一步用一个严格构造说明差异：考虑以常量$\beta$乘非归一化3D高斯为消光系数，$\sigma(x)=\beta\exp[-\tfrac12(x-\mu)^T A(x-\mu)]$，$A=\Sigma^{-1}\succ0$。单位方向射线$x=o+td$，令$v=o-\mu$、$a=d^TAd>0$、$b=d^TAv$、$c=v^TAv$。展开平方得到

$$a t^2+2bt+c=a(t+b/a)^2+c-b^2/a.$$

所以整条直线（而非相机的有限半射线）的光学厚度为

$$\tau_\infty=\beta\sqrt{\frac{2\pi}{a}}\exp[-\tfrac12(c-b^2/a)].$$

从有限$t_n$到$t_f$的积分需要乘上高斯CDF的区间差；对应严格吸收alpha为$1-e^{-\tau}$，并非$\tau$本身。即使在正交投影和固定方向下，线积分的峰值因子也含$\sqrt{1/a}$，与沿射线尺度相关。3DGS的屏幕核opacity未显式使用这一归一化因子或有限区间积分。这证明不能把学习opacity直接解释为固定、方向无关的物理消光参数。弱光学厚度$\tau\ll1$时$1-e^{-\tau}\approx\tau$，在特定线性投影/方向等假设下可能形成近似联系；这不是本文已验证的物理模型。

### 3. 可见性、颜色与几何的耦合

固定排序下$\partial C/\partial a_i=T_i(c_i-B_{i+1})$（精读3.5.5补充推导）。若颜色与后方合成色相同，alpha梯度为零，即使该基元深度错误；若$T_i\approx0$，遮挡后方的几何和颜色都缺少有效监督。由此应将可见性顺序正确、图像拟合正确、三维位置正确视为三个相关但不同的问题。

**构造例子**：灰色前景与灰色背景的颜色同为0.5，任意alpha合成仍为0.5。观察该像素不提供前景厚度的辨识信息。增加相机视图若仍看见同样均匀外观，也未必补足几何约束。相反，有可追踪纹理、遮挡边界或不同观测方向时，图像约束可能增强。该例说明“观测数量”不等于“有效信息量”，也解释高位置梯度只是增密启发式而非真实几何证据。

### 4. 各向异性与优化参数可辨识性

$\Sigma=R\mathrm{diag}(s^2)R^T$保证有效协方差，但并不让每个参数唯一。当$s_x=s_y=s_z$时任意R得到相同协方差；当两个尺度相等，绕剩余主轴的旋转不可辨识。q和−q也表示同一旋转。对模型进行参数误差评估时，比较协方差或渲染/几何结果比直接比较四元数分量更可靠；若真实几何法线是任务目标，还需另外定义高斯局部主轴与真实表面的关系。表3和图10支持各向异性提高视觉表示能力，不证明其小轴必然等于正确法线。

### 5. 自适应拓扑不是连续梯度的附属细节

位置、旋转、尺度更新解决“已有基元怎么变”；clone/split/prune解决“表示自由度在哪里增加或减少”。二者共同决定优化轨迹。密度控制基于当前误差敏感性而非几何oracle，容易受错误初始化、曝光、外观和遮挡影响。设计替代可以以梯度方向一致性、残差局部频率或跨视角可见性辅助判断，但需验证新增自由度究竟改善新视角/几何，还是过拟合训练图像。论文Fig8/Table3只能支持其当前增密策略有用，不能宣称该替代方案已有效。

### 6. 速度是一种表示—算法匹配，而非一个标签

tile实例数量为$K_{inst}=\sum_i n_i$，$n_i$是每个可见高斯覆盖的tile数；像素计算还依赖每tile候选数与有效深度。若基元很大且重叠多，实例数量、共享列表遍历以及梯度累积成本上升。比较两种表示应同时记录参数数目、投影footprint分布、有效贡献深度、FPS、分辨率、存储和训练耗时，而非仅数N。论文§6的排序与早停明确为高负载小splat优化；§7.4的popping和内存是其折衷。

**未证实范围**：上述构造、反事实和线积分推导只用于学习及后续实验设计，不是论文进行过的参与介质实验、深度评估或新的算法成果。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Scene Representation — Anisotropic Explicit Primitives（各向异性显式基元）
- **Keywords:** 3D Gaussian; Anisotropic Covariance; Explicit Representation; Splatting; Positive Definite Parameterization
- **Highlight:** 使用独立尺度与单位四元数表示正定协方差，局部连续高斯核可投影为二维覆盖；Figure 10的各向同性对照表明形状自由度有利于细结构。
- **Evidence:** §4 Eq.4–6；Figure 3/10；Table 3 Isotropic；精读3.4.1、3.5.2。
- **Connection Points:** 基元属性、空间支持、协方差约束、表面/体表示、同质量参数与存储代价。
- **Open Questions:** 约束核形状或增加表面先验时，何种场景下能改善几何而不损失新视角质量？

### Topic: Appearance & Transport — Ordered Alpha Compositing（有序透明度合成）
- **Keywords:** Volume Rendering; Alpha Compositing; Transmittance; Optical Thickness; Visibility; Gaussian Projection
- **Highlight:** Eq.1–3给出密度积分离散形式与点基有序alpha合成的代数联系，但像素alpha来源不同；本文没有证明任意连续消光场的严格线积分等价。
- **Evidence:** §2.3 Eq.1–3；§4 Eq.5；§6；精读3.5.1/3.5.5。
- **Connection Points:** alpha生成规则、光学厚度、连续积分、背景项、方向依赖、遮挡排序、参与介质假设。
- **Open Questions:** 在明确物理密度与有限视线区间时，如何构造可检验的投影近似并控制误差？

### Topic: Inference & Optimization — Adaptive Density Control（自适应拓扑优化）
- **Keywords:** Densification; Clone; Split; Pruning; Position Gradient; Opacity Reset
- **Highlight:** 以平均视空间位置梯度模长选择候选，按基元尺度clone或split，并定期剪枝、重置opacity；关闭split显著降低平均质量，clone影响局部细结构。
- **Evidence:** §5.2、Algorithm 1；Figure 4/8；Table 3；精读3.4.3。
- **Connection Points:** 增密触发信号、拓扑更新、初始化、误差敏感性、自由度预算、停止/剪枝准则。
- **Open Questions:** 如何区分真实缺失结构、外观失配与观测不足导致的高梯度，减少不可靠增密？

### Topic: Efficiency & Scalability — Tile-Based Differentiable Rasterization（分块可微光栅化）
- **Keywords:** Tile Rasterization; GPU Radix Sort; Depth Ordering; Shared Memory; Early Termination; Backpropagation
- **Highlight:** 16×16 tile、tile ID与深度组合键单次全局排序、共享列表前后向遍历；避免逐像素长状态列表与固定梯度数上限，同时存在近似可见性和内存折衷。
- **Evidence:** §6；Appendix C Algorithm 2；Figure 9；Table 1/3；精读3.4.4–5。
- **Connection Points:** 深度排序粒度、footprint、并行组织、内存状态、梯度覆盖和数值阈值。
- **Open Questions:** 能否仅在深度交错高风险区域细化排序，在质量和性能之间获得更稳定的折衷？

### Topic: Observation & Identifiability — Initialization and Coverage（初始化与观测覆盖）
- **Keywords:** Structure-from-Motion; Random Initialization; View Coverage; Identifiability; Sparse Observations; Floaters
- **Highlight:** 实景随机初始化主要恶化背景，已知边界且充分观测的合成场景可随机成功；低重叠测试视角仍出现伪影，图像质量不等于三维几何可辨识。
- **Evidence:** §5.1、§7.2–4；Figure 7/11/12；Table 2/3；精读3.6.2、4.2。
- **Connection Points:** 位姿、空间与角度覆盖、初始几何支持、训练监督、背景范围、几何与图像误差。
- **Open Questions:** 固定观测数量时，视角基线、空间覆盖和初始化质量各自如何影响几何与新视角误差？

### Topic: Appearance — Angular Basis and Missing Directions（方向基与角度缺失）
- **Keywords:** Spherical Harmonics; View-Dependent Color; Angular Coverage; Frequency Scheduling; Appearance-Geometry Ambiguity
- **Highlight:** 先学习SH零阶、每1000步加入一个频带直至四频带，以缓和角度观测不足造成基础色误估；No-SH消融支持方向表达有用但未证明BRDF分解。
- **Evidence:** §7.1、§7.3、Table 3 No-SH；精读3.5.3。
- **Connection Points:** 方向表示阶数、频带调度、角度覆盖、外观先验、颜色和几何解耦。
- **Open Questions:** 能否基于局部方向观测的条件数选择SH有效阶数，并以留出角度误差验证？


## RESEARCH-DIRECTION：研究方向定位及可迁移机制

**定位说明**：本次仅收到论文资源，并未要求以特定私人研究主线开展定向构思。因此按本篇公开问题作一般研究方向分析，不自行指定用户的研究主线，不纳入个人未公开方案。相关方向只是从3DGS实际证据推导的可探索技术维度。

| 层级 | 对3DGS的理解 | 不应混淆的概念 |
|---|---|---|
| Field | 三维视觉、计算机图形学、神经/显式辐射场 | 研究领域不等于具体方法名称 |
| Task | 多视角静态场景的新视角合成 | 不直接等于真实几何重建或语义预测 |
| Task Setting | 标定相机、RGB图像、典型SfM初始化；场景可有界/无界 | 真实随机、有界合成、低重叠条件不能合并 |
| Research Problem | 复杂场景中兼顾高质量、训练效率、实时渲染的表示与算法 | 性能目标不同于“使用高斯”的方法选择 |
| Method | 各向异性高斯、图像优化与增密、tile光栅化、SH | 方法不是问题的充分必要定义 |

### 方向一：观测约束与几何可辨识性

直接依据：§5.1投影歧义；§7.1角度信息缺失；Figure 7/12及Table 2/3。由不同初始点与观测覆盖下的行为可知，强表示与高效渲染并未解决所有信息不足问题。可迁移机制是利用已标定多视角建立可微图像监督，同时根据空间/角度覆盖评估局部参数是否有足够观测支持。核心科学问题是如何识别并区分有效约束不足和模型表达不足，而不是默认更密的高斯必然更准确。

若迁移到视角更稀疏、位姿有噪声、非理想成像的场景，跨视角不一致可能来自几何、颜色、相机或介质/光照变化。原方法假设静态且使用SH表达方向颜色，没有独立识别这些因素。实验必须拆分观测数量、视角布局、位姿误差和光度模型，不以RGB PSNR替代几何误差。需要已知几何的合成测试、真实留出视图以及局部误差分析。

### 方向二：表示与光学/几何约束的一致性

直接依据：Eq.1–6及精读3.5.1–2，Figure 10与局限性。可迁移机制是将合法协方差参数化、投影支持和有序合成组成可优化渲染器。领域差异在于若任务要求真实消光、几何深度、表面法线或连续传播规律，则学习opacity与图像色不足以提供这些物理量的唯一解释。需要明确定义目标物理量、渲染方程、额外监督或先验，以及模型能够退化回何种已知情况。上节线积分推导仅提供可检查的数学起点，不能当成3DGS已经验证的介质模型。

### 方向三：精度—效率—存储的共同设计

直接依据：§6/Appendix C、Table 1、§7.4–8。可迁移机制是先确定表示的有效计算域，再组织GPU并行、缓存与排序。瓶颈可能来自过大footprint、候选重叠、排序实例数、内存、梯度计算；不同任务不一定都需要同样tile大小或梯度处理。科学未知是何种场景与负载下可用近似不会破坏目标精度，工程未知是怎样在资源约束下稳定实现。控制分辨率、硬件、点数、质量与帧间稳定性，才能判断加速来自算法而非削弱任务条件。

**边界**：以上方向为一般研究分析，不声称其新颖性，尚需相关文献调研及专门实验；本次不独立读取其他论文。


## RESEARCH-PROBLEMS：可证伪的问题候选

下列均是从本文证据出发的研究假设，未进行独立新颖性检索或实验，不作为作者贡献。优先级以与实际失败证据的联系、控制变量可行性及潜在知识价值评估，不以预计发表难度评定。

### RP-A：基于观测充分性的增密判别（优先探索）

**现象与科学未知**：Figure 7中真实随机初始化在背景较差，Figure 12低重叠视角有伪影；§5.2增密依赖视空间位置梯度。高梯度可能意味着待补足真实结构，也可能是错误初始化、颜色或遮挡导致的失配。现有证据未分离这两者。问题：局部观测是否足以支撑增密所增加的几何自由度，如何识别不可靠高梯度？

**可证伪假设**：在位置梯度相近时，跨视角位置约束/可见性一致性更高的候选区域更可能通过增密改善留出视图和几何；当一致性低时，无条件增密可能只提高训练拟合或制造floaters。若这些信号与真实误差无稳定关联，或者同计算预算下不改善几何与留出误差，则否定该假设。

**验证设计**：使用已知几何及相机的合成场景，保留相同图像数量，改变视角基线与遮挡覆盖；分别改变初始化噪声和外观条件，独立控制。以原始梯度阈值、随机/均匀增密、同N预算且附加候选一致性判断为对照。记录每候选区域实际可见相机数、空间/方向约束、梯度统计、增密后训练误差/留出误差、已知几何深度或表面距离、floaters指标、基元数与训练成本。可先做不改方法的离线统计，再按统计结果设计干预。成功条件为在相同预算和多场景/重复试验下，新视角与几何同时改善或在几何改善时不显著牺牲质量；若仅训练PSNR改善、仅高覆盖区域获益或增加大量基元，不支持主假设。局限：候选区域真实几何对应需稳定定义，跨视角一致性也可能受非朗伯外观影响。需后续调研增密/不确定性相关工作。

### RP-B：方向外观与几何耦合的可观测性（次优先，理论价值较高）

**现象与未知**：§7.1缺少角度信息时SH零阶颜色可能错误，No-SH消融改善/损失只用图像PSNR衡量；固定排序下颜色和alpha梯度受透射率共同约束。问题：方向基容量和局部角度覆盖如何影响几何—外观耦合，何时提高方向表达反而损害可辨识性？

**可证伪假设**：在真实方向颜色较简单、角度覆盖条件数差的区域，限制高阶SH或延迟启用可减少几何错误与未见角度误差；在充分观测的强方向性外观区域，限制SH会明显降低正确外观拟合。若相同观察条件下方向基容量对几何误差无稳定影响，或适应性控制不能在两个条件间形成预期差异，则不支持假设。

**验证设计**：生成已知几何和可控Lambertian/方向外观场景，固定像素数、相机位姿精度、优化器和高斯预算，改变角度分布及SH阶数，比较原始逐频带日程、固定低阶/高阶、局部角度条件驱动的频带策略。对每个可见基元构造SH设计矩阵$Y_{k,m}$，记录奇异值谱与可见性加权条件数（区分真实角度缺失与遮挡）；评价留出角度RGB、真实深度/几何、基础色误差、外观误差和收敛。成功判据不是单一PSNR最高，而是条件依赖趋势可重复且策略在困难区域改善几何/外推，同时保留充分观测时的外观表达。真实场景无GT基础色时只能用间接指标，需保留这一限制；不声称已完成BRDF估计。

### RP-C：深度交错风险与自适应排序（偏工程与数学）

**现象与未知**：§6用tile共享深度排序，§7.4承认顺序突变和guard band导致popping；原文说小splat时近似可忽略。问题：哪些几何/投影/重叠条件能预测排序误差，能否只在必要像素或tile内提高可见性排序精度？

**可证伪假设**：当投影footprint较大、沿视线方向尺度/倾斜显著且多个基元深度区间交错时，基元中心深度的共享排序更容易产生颜色不连续；局部细化可减少此类误差而保持大部分帧率。若风险指标与错误不相关或所有改善都要求接近全像素排序成本，则否定性能假设。

**验证设计**：构造有明确几何与层次的双层/多层半透明或不透明基元示例，改变相机轨迹、基元尺度、深度差、交错角度；使用高精度参考（定义明确的目标，例如有限线积分或指定表面排序，不能混用物理模型）及原tile中心排序对照，增加逐像素候选排序和风险触发的局部排序。固定表示、颜色/alpha及硬件，比较每帧图像误差、连续轨迹差分（需剔除真实运动变化）、popping区域比例、GPU时间、实例数和显存。然后再考虑训练是否受改善。成功需要在指定参考模型及多轨迹中减少伪影且相较全精细排序保留速度；没有统一真实顺序时需先说明目标合成定义。此方向与原文失败机制直接相关，但实际GPU实现工作量较高，需先数学/合成验证。

### 优先级与限制

A直接对应论文增密与低覆盖失败，初步离线统计可低成本开展；B有明确角度基数学工具，需可控材质与几何数据；C与已知排序近似强关联，但需定义参考和实现代价。三者均需要后续独立相关工作检索与实验可行性审查。若问题本身在后续文献中已被充分解决，应更新候选而不是强行宣称创新。


## INTERNALIZATION：知识地图与自我检验

### 用自己的语言解释

3DGS做的是场景级逆渲染式优化：把一个未知静态场景近似为一组具有方向颜色、可调形状和透明度的三维核，然后通过可微投影和有序合成，让它们生成的图像逼近已知照片。它区别于典型NeRF的关键不是“不使用任何连续函数”，而是场景基元可以直接投影到像素候选列表，省去大量逐ray空间查询；增删基元分配新的自由度，GPU光栅器和逆向重构使训练及推理更高效。结果是可用于新视角合成的参数集合，不是经过真实几何监督保证的表面模型。

### 技术与概念地图

| 观察/任务 | 所需机制 | 学到或产生的量 | 主要风险 |
|---|---|---|---|
| 多视角RGB与标定相机 | SfM、初始化 | 初始中心和尺度 | 低纹理/少视角/错误位姿 |
| 三维形状与方向色 | 正定协方差、SH、opacity | 可学习基元 | 方向/形状/alpha参数耦合 |
| 当前视角投影 | 局部Jacobian与2D核 | 屏幕中心、椭圆、alpha | 近平面不稳定、非线性近似 |
| 图像形成 | tile排序、合成 | 像素RGB和透射率 | 深度交错、透明度截断 |
| 训练监督 | L1+D-SSIM、反向重构 | 参数梯度 | 遮挡、角度缺失与弱约束 |
| 表示自适应 | clone/split/prune/reset | 新的集合与参数 | 过拟合、floaters、过密 |
| 推理 | 固定集合与新相机 | NVS图像 | 未见区域伪影与存储成本 |

### 易错理解自检（问题—参考答案）

1. 为什么3DGS需要协方差而不只是位置？——投影核的空间支持、方向和大小决定像素贡献，细长结构可用各向异性核紧凑表示。无直接法线输入，不等于法线必然从核正确恢复。
2. Eq.2与Eq.3形式相同，是否说明3DGS精确积分了NeRF密度？——否，alpha生成机制不同，投影和排序还包含近似；需定义相同光学厚度、空间段与颜色才可作进一步物理比较。
3. opacity=0.8是否意味着每个覆盖像素都吸收80%？——否，像素alpha是学习opacity乘二维核响应（另有实现阈值），不同像素和视角不同。
4. clone和split都是增密，为何不能只用一种？——小基元欠覆盖需要新增支持，大基元过宽需要更小、更灵活的基元；Figure 4/8与Table 3支持分支互补。
5. 为什么10个前方点有梯度还不够？——后方有实际贡献的基元依然影响颜色，其梯度被人为截断可能不正确；原文N=10实验证实，但完整方法仍按alpha阈值/饱和停止。
6. 为什么随机初始化在合成成功、实景较差？——合成已知边界、覆盖充分、无复杂背景，真实随机范围与远景约束不同。不能直接推出SfM始终必需或始终不需。
7. 训练时间增加，FPS必然增加吗？——不。Table 1的30K比7K质量更高、参数存储更大、FPS更低。速度和质量分别评价。
8. PSNR更高意味着几何更准吗？——没有这样的保证；图像误差受外观、alpha、遮挡与位置共同影响。需要真实几何或其他独立证据。
9. 3DGS的高质量是否意味着对任意少视角都可靠？——Figure 12显示低重叠视角仍失败，未报告系统稀疏输入评估。
10. “隐藏完整版”是否私密？——不是；作为研究内容只保存适合发布的通用知识和未验证公开研究假设，不纳入个人敏感记录。

### 尚未解决问题

如果希望把本文机制用于具有更弱观测或更强物理解释需求的任务，先定义需要恢复的量与可辨识性条件，再决定高斯参数、约束与渲染近似；不要先选择增密或SH修改后再构造问题。研究可从上一节A/B/C的可控验证入手，区分训练、留出视图、真实几何和运行效率。此为一般研究启发，不把候选假设写成论文结论。


## GLOSSARY：核心术语及相近概念辨析

| 英文 | 中文 | 技术含义与相近概念 |
|---|---|---|
| Novel View Synthesis (NVS) | 新视角合成 | 给定未训练相机生成图像；不同于直接输出几何 |
| Radiance Field | 辐射场 | 空间和方向相关的成像表示；不同方法对密度/可见性表示不同 |
| Structure-from-Motion (SfM) | 运动恢复结构 | 多图估计相机和稀疏点；本文真实场景初始化来源 |
| Multi-View Stereo (MVS) | 多视图立体 | 用多视角恢复更密集几何；本文不要求其密集初始几何 |
| 3D Gaussian | 三维高斯 | 具有中心与协方差的空间核；渲染核非归一化，Eq.4 |
| Mean | 均值/中心 | 三维基元位置，不保证是准确表面点 |
| Covariance Matrix | 协方差矩阵 | 描述核的方向与尺度，Eq.4–6，不是颜色方差 |
| Anisotropic / Isotropic | 各向异性 / 各向同性 | 不同轴尺度可独立变化 / 三轴相同 |
| Positive Definite | 正定 | 对任意非零v有$v^T\Sigma v>0$，逆协方差有定义；半正定允许零特征值 |
| Quaternion | 四元数 | 用单位四维向量表达三维旋转；Eq.10；q与−q等价 |
| Jacobian | 雅可比矩阵 | 局部导数线性映射，Eq.5用于传播协方差 |
| Splatting | 泼溅/溅射 | 投影带范围基元并累积贡献；区别于固定点渲染、逐ray marching |
| Rasterization | 光栅化 | 把空间基元转换为像素贡献，本文tile组织 |
| Alpha Blending | alpha合成 | 按顺序混合颜色，Eq.2–3；一般与顺序相关 |
| Opacity | 不透明度 | 学习的基元标量或像素alpha需区别；不等于物理消光系数 |
| Transmittance | 透射率 | 剩余可见光比例，$T_i=\prod_{j<i}(1-a_j)$ |
| Optical Thickness | 光学厚度 | $\tau=\int\sigma ds$，与透射率$e^{-\tau}$关联；本文不直接学习该物理量 |
| Spherical Harmonics (SH) | 球谐函数 | 球面方向基展开颜色；零阶常数、高阶方向变化 |
| View-Space Gradient | 视空间梯度 | 位置对图像损失的局部敏感性，作为增密候选信号，非几何误差 |
| Densification | 增密 | 增加/细化空间基元，非NeRF密度sigma |
| Clone | 复制 | 对较小高梯度基元增加同尺度基元，补覆盖 |
| Split | 分裂 | 对较大高梯度基元拆成两个小核，细化空间自由度 |
| Pruning | 剪枝 | 删除低opacity或过大不合理基元 |
| Opacity Reset | 不透明度重置 | 周期性降低opacity以重新判定有效基元 |
| Tile / Bin | 图块 | 16×16像素分组，共享候选列表，§6 |
| Radix Sort | 基数排序 | 按tile ID和深度组合键一次GPU并行排序 |
| Frustum Culling / Guard Band | 视锥剔除 / 保护带 | 排除不可见或投影不稳定基元，可能引起边界跳变 |
| Footprint | 屏幕覆盖范围 | 高斯投影影响区域；决定tile重叠实例成本 |
| Depth Complexity | 深度复杂性 | 同一像素射线多个贡献基元/层，不是图像深度本身 |
| Early Termination | 提前终止 | 前向累计opacity接近饱和后停止，区别于固定N个梯度上限 |
| Backpropagation | 反向传播 | 图像损失梯度经合成、投影回基元；训练需要，推理不需要 |
| Adam | Adam优化器 | 原文Algorithm 1的随机梯度类更新方法 |
| L1 | L1损失 | 像素绝对误差，Eq.7权重0.8 |
| D-SSIM | 结构不相似性 | SSIM相关图像结构损失，Eq.7权重0.2，具体实现未在原文展开 |
| PSNR | 峰值信噪比 | 像素误差指标，dB，越高越好 |
| SSIM | 结构相似性 | 局部图像结构等相似度，越高越好 |
| LPIPS | 学习感知图像块相似性距离 | 感知特征距离，越低越好 |
| FPS | 每秒帧数 | 渲染速度，非训练耗时或硬件无关复杂度 |
| Floaters | 漂浮伪影 | 错误空间基元或体密度导致的浮动物，Fig11 |
| Popping | 跳变伪影 | 相机移动时不连续的颜色/结构变化，§7.4排序和guard band原因 |
| Identifiability | 可辨识性 | 给定观测能否区分不同参数/几何；不同于优化是否收敛 |

**依据及边界**：术语对应原文§2–7、Eq.1–11和图表；光学厚度、可辨识性等定义为辅助背景知识。术语表不宣称论文估计真实物理量或具有这些背景概念的全部性质。


## SOURCES：来源、版本及知识边界

### 实际使用的来源

- 论文：Bernhard Kerbl、Georgios Kopanas、Thomas Leimkühler、George Drettakis，*3D Gaussian Splatting for Real-Time Radiance Field Rendering*，arXiv:2308.04079v1（2023-08-08）。PDF物理页14页，正文§1–8、Appendix A–D、Figure 1–12、Table 1–9、Algorithm 1–2。
- 原始 PDF SHA-256：`f4c0c9f27e2d02b0265017f66049d471eb66cb50203bc918f51ba07d20cbbe11`。同版HTML为辅助，缺一数学字体及一行内图标引用，未发现科学图片缺失。原始字节未改动。
- 精读报告 SHA-256：`753502c1dd138b783b73a292621c90f06107f0fc11b125d024ddc3dc5bf0f69d`；以上笔记锚定冻结报告及必要时回查实际PDF。
- 官方来源：[arXiv](https://arxiv.org/abs/2308.04079)、[PDF](https://arxiv.org/pdf/2308.04079v1)、[Project Page](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/)、[Code](https://github.com/graphdeco-inria/gaussian-splatting)。官方缓存记录仅v1，实时版本未确认，DOI页面访问受限。代码及项目页用于身份/链接核验，不作为原文之外的科学补充来源。

### 证据使用规则

【原文事实】来自PDF；【背景知识】提供通用概念；【合理推断】与补充推导写明条件；【独立评价】对证据和限制的判断；【尚未证实】未执行实验或无足够依据的假设。学习笔记中线积分、参数可辨识性、梯度信号诊断、局部排序策略等是公开研究性拓展，不是作者额外贡献。未查其他论文全文、未运行代码、未看未提供视频。跨文献实质比较延期，CROSS-PAPER仅本篇的待关联索引。

### 需后续核查的事项

源码变量命名、具体SH存储/方向约定、所有超参数和数值实现未做源码级复核；正文的Eq.2下标及§6逆向累计量措辞已按数学关系辨析；未确认实际站点渲染或其他版本变化。本文证据支持所测试数据集的NVS与效率结论，不证明物理介质参数、真实几何或一般稀疏观测任务。研究问题候选仍需后续文献调查、明确实验参考与重复验证，不宣称已有创新性或结果。



