---
title: "Depth Anything 3：从任意视图恢复视觉空间（完整精读与研究笔记）"
date: 2026-11-27 03:53:59
index_img: "/DA3/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/DA3/)

# 第一部分：论文精读报告

# DA3 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2511.10647v1（2025-11-13）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2511.10647) | [PDF](https://arxiv.org/pdf/2511.10647v1) | [Project Page](https://depth-anything-3.github.io/) | [Code](https://github.com/ByteDance-Seed/Depth-Anything-3)

<!-- PAPER-METADATA END -->

> 论文：Depth Anything 3: Recovering the Visual Space from Any Views
> 版本：arXiv:2511.10647v1，用户上传 arxiv_2511.10647v1.zip。
> 范围：PDF 全32页，正文、参考文献与附录。PDF 为权威，HTML 与 LaTeX 为辅助；在线摘要存在数值差异，科学分析采用本地 PDF。展示图是 PDF 区域渲染的 JPG，不是原始矢量原件。
> 阅读方式：按计划逐单元直接写入并验收；原文陈述、背景知识、补充推导和评价分别标识。

<!-- PTH:BEGIN INIT -->
## 0. 初始化：资源、结构与精读范围

### 资源与证据

本次阅读固定为用户上传的 arXiv:2511.10647v1，论文作者为 Haotong Lin、Sili Chen、Jun Hao Liew、Donny Y. Chen、Zhenyu Li、Guang Shi、Jiashi Feng、Bingyi Kang，机构为 ByteDance Seed（PDF p.1）。PDF 首页带有 v1 及 2025-11-13 印记，32 页均已成功读取与渲染。LaTeX 主文件与 HTML 的标题和摘要一致；Source 没有独立的版本印记，固定版本下载地址和正文对应关系作为辅助证据。HTML 缺少两项引用资源，无缺失的候选图像资源。原始 PDF、源压缩包、HTML 文件保留，不修改。

arXiv 无版本 abs 页仅列 v1，但摘要的提升幅度与本地版本不同：网页为 44.3%/25.1%，本地 PDF 摘要为 35.7%/23.6%，且正文 p.15 报告相对 VGGT 的几何提升 25.1%，存在文内口径差异。同包 HTML、Source 及官方项目页采用本地 PDF 摘要的 35.7%/23.6%。本文不尝试把两个数值集合合并；所有科学结论以本地 PDF 为准。版本核验日期为 2026-11-27。外部链接仅核验身份与资源，不以外部版本正文补充本文。

### 全文结构

| PDF 页码 | 内容 | 阅读目标 |
|---|---|---|
|1–2|标题、摘要、未编号比较图、目录|总体主张与范围|
|3–4|§1 Introduction|最小预测目标及最小架构问题|
|4–5|§2 Related Work|传统及学习型几何、单目与前馈 NVS|
|6–8|§3 Depth Anything 3|几何表示、射线到相机、Transformer、Dual-DPT、目标与训练|
|8–11|§4 Teacher–Student Learning|教师数据、相对深度及法向损失、对齐、单目与度量学生|
|11–12|§5 Application: Feed-Forward 3D Gaussian Splattings|有姿态和自适应姿态的 3DGS|
|12–13|§6 Visual Geometry Benchmark|位姿、重建及渲染评估协议|
|14–21、23|§7 Experiments|主比较、消融、效率及定性结果；图11位于p.23|
|22|§8 Conclusion and Discussion|作者结论及未来方向|
|24–30|References|在本文范围内定位相关工作，不展开外部文献全文|
|31–32|Appendix A Data Processing|合成数据异常与清理|

### 图表目录

| Figure | 页 | 功能 |
|---|---:|---|
|1|3|总览：任意输入、深度与射线、点云、几何及渲染|
|2|5|主干、相机 token、Dual-DPT 和训练关系|
|3|7|共享重组及双融合解码器|
|4|8|真实数据深度监督缺陷|
|5|14|视频相机轨迹比较|
|6|16|点云规则性与噪声比较|
|7|16|深度图细节与结构比较|
|8|19|教师标签对深度细节的影响|
|9|20|1、2、13 张真实图像的几何实例|
|10|21|度量深度教师监督定性消融|
|11|23|域内外新视角渲染定性比较|
|12|31|TartanAir 无效背景|
|13|31|IRS RGB–depth 空间错位|
|14|32|UnrealStereo4K 数据错误|

| Table | 页 | 功能 |
|---|---:|---|
|1|9|训练及位姿–几何 benchmark 数据|
|2|14|位姿 AUC|
|3|15|重建 F1/CD，有无已知相机|
|4|15|单目深度 δ1|
|5|17|前馈 NVS 三指标|
|6|17|预测目标组合|
|7|18|架构、解码、教师和相机条件|
|8|18|容量、参数及速度|
|9|19|教师数据、表示和损失|
|10|20|单目学生|
|11|21|度量深度与教师消融|

合计正文 11 个编号 Figure、附录 3 个，Table 11 个，未发现独立编号 Algorithm。首页还有无题注的柱状/雷达性能图，列为额外可见内容，不人为赋号。其源文件为 `figs/pdfs/da3_radar.pdf`（paper.tex 引用），分析见图表总览。14 幅编号图已实际核验保真区域及展示文件。此单元为目录与证据登记，不涉及机制推导；此处不展开数学。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1.1 标题精读

### 标题译文

Depth Anything 3: Recovering the Visual Space from Any Views —— **Depth Anything 3：从任意视图恢复视觉空间**。

Depth Anything 3 是模型/系列名称，不强行译为“深度任何事物”。副标题的中心是 recover visual space，而不是只输出单张 RGB 的深度图。PDF p.1 脚注解释，系列从单目拓展至任意视图，以深度作为理解物理世界的基础。标题中的 3 是系列代际，不是视图数或三维坐标维度。

### 术语界定

- Visual space：这里指可由视觉输入推断的三维几何空间。§3.1 给出具体对象：每视图深度、相机射线及可反投影至统一坐标系的点。它不表示已经恢复整个世界的隐藏体积、材料或语义。
- Any views：输入可以是单张图像、多视角图像集或视频帧集合。与传统固定双视图/多视图模型不同，不为不同输入数量重新设计一套模型。论文还允许提供或省略相机参数（p.3–4、§3.2）。这不是“从任意无关的图片一定可以重建一个共同场景”。
- Recovering：从观测推断几何与相机而非只是生成视觉上合理的图像。若输入为 $N$ 张图，模型产生 $N$ 对像素对齐的深度、射线图。世界点由 $P=t+Dd$ 给出；$t$ 是射线起点，$d$ 是不归一化的方向，$D$ 是相机 z 深度。输出的空间一致性需要不同视图中同一表面的反投影接近，而不是仅单张深度轮廓合理。

### 标题边界

任意视图数描述接口灵活性，不保证任意基线、无重叠、任意动态内容或无限视图数的准确恢复。单目观测的真实尺度歧义不会因统一表示而消失；一般相对深度与额外训练的 metric 模型需分别理解。单目看到的表面不是完整不可见三维结构。§6 的相机/几何协议与 §5 的新视角应用分别评估不同能力，不能用标题把相机估计、点云、度量深度和新视角合成视为完全相同的任务。标题未承诺透射/散射建模、动态重建或严格度量精度。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 摘要的论证链条（PDF p.1）

### 问题与困难

作者目标是用一个模型处理单张、少量或大量视觉输入，并在可选相机条件下恢复空间一致几何。深度任务本身在每个像素沿成像光线定位表面；多视图情形还需要相机关系，使多个局部深度图可以进入共同坐标系。传统把单目深度、SfM、MVS 作为独立任务；近期统一模型虽有多任务预测能力，却常依赖多阶段或专门主干，以及多个重叠的预测目标（§1）。冗余目标不仅增加头部或训练约束，预测出的 depth、point map、pose 还可能无法互相满足同一投影方程。实际深度标签亦异质：真实重建的点稀疏、传感器可能有孔洞、合成深度的密集细节较丰富但域不同（Fig.4）。因此有三条必须分开处理的困难链：几何变量如何表示、共享网络如何跨视图交流、如何从不同质量的数据建立可信监督。

### 方法链条

**表示。** 作者主张深度–射线联合预测是最小且足以编码该任务的目标。对于像素 $p=(u,v,1)^\top$、相机内参 $K$、camera-to-world 旋转 $R$ 与平移 $t$，相机 z 深度 $D$ 给出世界点 $P=R(DK^{-1}p)+t$。将 $d=RK^{-1}p$ 预测为像素对齐向量，与射线起点 $t$ 及深度组合，就把相机作用并入密集表示。点云和相机不是必须完全独立地回归的量；轻量相机 head 仍为便捷推理而保留（§3.1）。这里“最小”指作者考察的目标组合，而不是对所有可能几何参数化的理论最优证明。

**架构。** 复用 DINOv2 预训练 ViT 主干，通过部分层 token 重新排列实现图内与跨图 self-attention；输入只有一张时，自然退化为单图序列。相机参数可被编码为每图一个 token；未知时使用共享可学习 token。Dual-DPT 的共享重组层使两路预测从对齐的特征产生，分别融合并输出深度/射线；并非完全取消专用解码头。这一机制让不同输入视图的几何线索在主干内交流，而不必新增一套每个任务专用的深度或 SfM 主干。

**监督。** 仅依赖噪声真实深度会把坏标签传入模型。先在大量经过整理的合成数据上训练单目相对深度教师，再用 RANSAC 估计正尺度 $s$ 与平移 $t$，使教师深度在真实观测有效像素上对齐，获得 $D_{T\to M}=s\tilde D+t$（Eq.8）。教师提供稠密结构，真实测量提供场景尺度及与相机关系的约束。若教师局部结构错误或有效测量集中在狭窄深度范围，对齐也无法保证全图真实。

### 结果与边界

作者摘要报告：DA3 在新的视觉几何 benchmark 的位姿、任意视图几何和视觉渲染任务中建立新成绩，相对 VGGT 的位姿和几何准确性平均增幅分别为 35.7% 与 23.6%（本地 PDF 摘要数字）。注意 PDF 正文 p.15 的几何提升另写为 25.1%（相对 VGGT），与摘要 23.6% 不同；正文与表格的核对将在 Table 3 与 EXP-POSE-GEOM 单元独立进行，不擅自统一。相对单目模型也超过 DA2，并借助 teacher–student 获得同等级别的细节与泛化；所有模型使用公开学术数据。§1 报告 20 个设置中 18 个 SOTA，而不是全部每项指标都获第一；Table 2–5 的模型、条件和评估口径需逐一分析。

几何基准考察预测相机和融合几何，包含带 GT 相机融合与不带已知相机设置；渲染比较使用 12 张 context 与给定相机，因此该结果不等于已证明任意无姿态少视图场景中的渲染可靠性。摘要的相对增益也不是绝对百分点。首页柱状图的 DA3-Teacher、主 DA3、DA2 是不同模型配置，不与 §4.3 的单目学生混为一谈。

一句话：作者将深度和射线作为统一前馈几何估计的核心目标，用预训练 Transformer 的适应性跨视图交流与质量改进后的深度监督，使统一表示可以服务位姿、三维重建、单目深度和进一步的前馈 3DGS。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 核心贡献与相互关系

作者未在 §1 单列固定编号贡献，以下按 p.3–4 的两个中心问题与后续章节归纳，不将已有 DINOv2、DPT、DLT、RANSAC、3DGS 算作本论文首创。

### 表示贡献：减少独立回归目标

输入为 $N$ 张图像；每图预测 $D\in\mathbb R^{H\times W}$ 和 $M\in\mathbb R^{H\times W\times6}$。世界点由 $P=t+Dd$ 导出，因此表示将局部深度与统一参考系绑定。相机起点可平均，方向集合可以通过线性投影几何求相机；额外的小型 camera head 只是降低显式求解成本（§3.1）。关键不在预测输出的通道总数最少，而在把几何相关任务统一为具有明确组合关系的目标。Table 6 将 depth+cam、depth+pcd+cam 与 depth+ray 等比较；前者在作者设置中整体较弱，后者也有增添 camera head 后局部改善的情况。故支持“足够有效”而非绝对无冗余/所有情形唯一最优。

### 架构贡献：共享预训练 ViT 与轻量输出设计

主干仍采用通用 ViT，前约 2/3 层仅图内注意力，后约 1/3 交替图内和全视图注意力。其实现机制是重排 token 而不是另起额外的全局 Transformer 主干；使跨视图关联能建立在单图预训练特征上。每图一个 camera token 提供可选几何条件。Dual-DPT 复用 reassembly，之后分离 fusion 与 output。Figure 2/3 给出数据流，Table 7 的 a–d 分别检查架构、注意力排布、独立 DPT 设计。它不是没有架构设计的纯 DINO 模型：注意力作用范围、camera token、输出头是重要适配。

### 监督贡献：密集教师与原真实几何的尺度联系

合成教师从更大数据集学习相对深度、指数深度表示及法向/掩码监督（§4.1）。真实训练深度通过 $s\tilde D+t$ 由教师补足细节并与可用真实数据对齐（Eq.8）。这使“合成数据的稠密边缘与真实数据的相机/尺度”结合，而非把合成/真实数据简单拼接为同质量标签。Fig.4 是问题证据，Fig.8 与 Table 7(e) 是教师对主模型的消融，Table 9 验证教师数据与损失选择，Table 10–11 验证额外的单目和度量模型。改善细节不保证所有 metric 指标都改善；教师 supervision 不是物理测量的新真值。

### 验证与应用

- 新建综合视觉几何评估，包含 5 个位姿/重建数据集及另一个 165 场景的前馈 NVS benchmark，尽量让方法通过同一协议比较。作者的 HiRoom 为自建合成集合，其他数据集并非新发明。
- 在已学几何主干上添加 GS-DPT，输出像素级 Gaussian 参数，将深度反投影作为中心，再进行 rasterization。另提出 pose-adaptive 版本，支持有/无相机并采用冻结主干、深度偏移和 SH 等设计（§5）。Table 5 支持该主干在已知姿态的前馈渲染协议下优势，但没有证明所有 pose-free 设置都同样有优势。

逻辑关系：表示确定预测与几何的接口；架构负责从可变输入中获取并融合信息；教师改善学习到的监督目标；综合基准验证是否兼顾多类几何任务，GS 扩展检验几何特征的可迁移性。这四部分不是相互独立的四个模型。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 结论与讨论精读（PDF p.22, §8）

### 作者结论

作者认为通用 Transformer、depth-and-ray 目标和 teacher–student 监督就能够统一 any-view geometry，不需要复杂的专门主干。深度包含场景几何的局部信息，射线将像素关联到统一空间；自适应 attention 使每图特征能够接受其他图像上下文。尺度感知的深度与每像素射线共同服务重建，不能理解为单张 RGB 在无任何额外条件下必然决定绝对世界尺度。作者指出巨型与紧凑配置均有竞争力，并认为同一主干可以支撑前馈 NVS。证据分别来自 Table 2/3（位姿与重建）、Table 6/7（表示与架构）、Table 8（效率）及 Table 5（给定相机渲染）。这些是模型族、数据和协议层面的验证。

### 未来方向

原文明确提出未来扩展至动态场景、加入语言与交互线索、探索更大规模预训练，并希望几何理解与可行动世界模型形成闭环。动态场景扩展属于未来工作，不能因 Fig.5 使用动态视频就认定本文已完整恢复随时间变化的三维场。语言/交互也不是本文的模型输入或训练任务。更大预训练是发展方向，不是已验证的某种缩放定律。

### 结论边界

独立评价：实验有力支持作者定义的统一任务、比较模型和训练设置中的有效性；但不能把“足够”扩展为数学上对任意场景、观测噪声和姿态条件的充分唯一可识别性。尤其 Table 6 的附加 camera head 不是在所有列上无提升，Table 7 的 teacher/pose conditioning 也存在反向变化，因此不能由作者的概括性语言推出全指标单调获益。Table 5 的渲染实验采用给定相机姿态，不能单独支持 pose-free 3DGS 的定量优势。未见对遮挡背面真实几何、物体非刚性运动或参与介质物理参数的专门评估。

作者强调简洁与拓展性，这是相对多主干/多头系统的设计主张，不等于整个训练低成本；§3.4 的 128 H100 与 200k steps 表明训练资源仍很大。推理效率的比较以指定 A100 与输入分辨率为条件。未来研究主张和实际结果应分列。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2.1 图表总览

### 图表功能

图表形成四条证据链，而不是 25 个彼此独立的展示对象。

| 证据链 | 图 | 表 | 回答的问题 |
|---|---|---|---|
|表示与模型|Fig.1、2、3|Tab.6、7、8|输入输出如何统一？单主干、depth-ray、共享解码是否有实证支持？|
|监督质量|Fig.4、8、10、12–14|Tab.1、7(e)、9、10、11|真实深度为何不够好？教师及清洗分别补足什么，代价如何？|
|主几何能力|Fig.5、6、7、9|Tab.2、3、4、10、11|相机、点云、相对/度量深度能否在各自协议下改善？|
|渲染迁移|Fig.1、11|Tab.5|几何 backbone 能否用于高质量前馈新视角合成？|

第一类架构图说明数据流，不能证明其设计最优；第二类包含动机样例、定性消融和量化表；第三类不能混淆 GT pose 与预测 pose 以及 relative vs metric；第四类用 RGB 渲染指标验证外观，不能代替几何真值误差。

### 首页图（无编号）

PDF p.1 显示三个紧邻图形。左图为 Monocular Depth 柱状对比：DA2=90.3、DA3=92.4、DA3-Teacher=94.6（图内直接数值，百分比制 δ1 的聚合值；Table 4 给出各项，图中值与五项算术均值吻合）。这区分了 any-view 主模型和专门的教师；教师不等同于额外训练的 mono-student。中图 Pose Accuracy、右图 Reconstruction Accuracy 的雷达轴为 HiRoom、ETH3D、DTU、7Scenes、ScanNet++，模型为 DA3、Pi3、VGGT；刻度 0.2、0.4、0.6、0.8。轴表示不同数据集而非时间步，雷达面积不等于表中统一定义的平均误差。DTU 的重建表使用 CD（越小越好），而其他数据集用 F1（越大越好），图形为概览而不能由半径直接反推原始 CD。首页无独立编号或题注，逐表数据见 Table 2–4。

### 图表边界

所有 14 幅编号 Figure 的展示图来自 PDF 同一页面区域并保留标签；Figure 1/3/4/9 为带记录通过，其他 10 幅为精确通过（实用保真分类，不把 JPG 当原始测量数据）。对深度色图只根据可见边缘与相对色块描述，不从颜色直接计算绝对深度；对雷达或点云不从视觉大小估计定量结果。所有定量指标以实际表格及其协议为准。附录图不是实验测试样本的完整统计，只是作者识别数据缺陷的证据。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
## 2.2.1 Figure 1（PDF p.3）

![Figure 1（PDF 区域渲染展示副本）](DA3/Figures/Figure_01.jpg)

**Caption 原文**（PDF p.3）：Figure 1 Given any number of images and optional camera poses, Depth Anything 3 reconstructs the visual space, producing consistent depth and ray maps that can be fused into accurate point clouds, resulting in high-fidelity 3D Gaussians and geometry. It significantlyoutperforms VGGT in multi-view geometry and pose accuracy; with monocular inputs, it also surpasses Depth Anything 2 while matching its detail and robustness.

**Caption 中文**：给定任意数量的图像和可选的相机位姿，DA3 重建视觉空间，输出一致的深度图和射线图，可将它们融合为准确点云，进而获得高保真的三维高斯与几何。它的多视角几何和位姿准确性显著超越 VGGT；单目输入时也超越 DA2，同时达到相当的细节和稳健性。

**布局与子图**：上方从单张悉尼歌剧院图像、多张不同视角图像到视频片段的三种输入形态，用同一中间蓝色 Single Transformer 模块连接。输入说明中的 with or without camera poses 表示可选外部相机信息。输出区域左侧先展示 Depth & Ray Maps（色图与相机小图），再展示 Geometry & Rendering（灰色几何与着色渲染）；中间大图是歌剧院的彩色点云，右侧雷达按 HiRoom、7Scenes、ETH3D、ScanNet++、DTU 分组显示 Geo. & Pose Accuracy，带星号的轴为 pose 项；左下角柱状条比较 DA3-Teacher、DA3、DA2 的单目表现。图中没有给出绝对深度标尺或渲染损失数值，柱图和雷达仅作整体效果提示。

**机制与证据**：输入从 $N=1$ 到 $N>1$ 变化时，不需要换用另一套多视角专用主干。由 §3.1，对于图像 $i$ 的像素 $p$，深度 $D_i(p)$ 与射线 $M_i(p)=(t_i(p),d_i(p))$ 产生世界点 $P_i=t_i+D_i d_i$。若不同视图中同一个可见物理点的位置一致，叠加反投影就得到更密集且一致的场景几何；相机头或从射线求解的相机可用于将点投影到目标视图。§5 的 3DGS 需要额外 GS-DPT，预测 opacity、rotation、scale、color 等属性，使用深度确定中心；不是从点云直接得到完整 3DGS 参数。图中的“统一”是几何主干共享，而不是只通过两个输出张量无条件产生可渲染全部场景。Table 2/3 对应雷达中的位姿/重建任务，Table 4 单目深度，Table 5 是附加 head 的给定 pose 渲染实验。DA3-Teacher 柱条更长与 Table 4 的指标一致，但它并非 any-view 模型本身。

**证据边界**：这是 teaser，展示方法意图及案例，不是受控消融。可见点云质量不能证明不可见背面也正确；单张图的视觉合理性不解决真实尺度和遮挡歧义。雷达横轴/单位因评估协议而异，尤其 DTU 的重建使用 CD，不能从雷达半径反推具体绝对值。比较应查后续实际表格。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
## 2.2.2 Figure 2（PDF p.5）

![Figure 2（PDF 区域渲染展示副本）](DA3/Figures/Figure_02.jpg)

**Caption 原文**（PDF p.5）：Figure 2 Pipeline of Depth Anything 3. Depth Anything 3 employs a single transformer (vanilla DINOv2 model) without any architectural modifications. To enable cross-view reasoning, an input-adaptive cross-view self-attention mechanism is introduced. A dual-DPT head is used to predict depth and ray maps from visual tokens. Camera parameters, if available, are encoded as camera tokens and concatenated with patch tokens, participating in all attention operations.

**Caption 中文**：DA3 的流程。DA3 使用单个 Transformer（原始 DINOv2 模型），无主干结构改造。通过输入自适应的跨视图自注意力实现跨视图推理。Dual-DPT head 从视觉 token 预测深度和射线图。若有相机参数，则编码为相机 token 并与 patch token 拼接，参与所有注意力操作。

**布局与子图**：从左到右分四部分：(1) 多张 RGB 通过 Image Patch Embed 产生图像 patch token；相机参数可经编码器 $E_c$ 转为紫色 token，否则用同一 learnable camera token；图中的加号指序列拼接。(2) Single Transformer 由 $L_s$ 前段和 $L_g$ 后段组成，白色/浅绿色块分别为图内与跨图 self-attention，末段交替出现两种颜色，而不是在每层都全局注意力。(3) 图像 token 进入 Dual DPT Head，输出 $N\times H\times W$ 的 Depth 与 $N\times H\times W\times6$ 的 Ray；图中 Points 为 $N\times H\times W\times3$，由前两者关联产生。深度教师位于右上方虚线训练支路；$\mathcal L_D,\mathcal L_M,\mathcal L_P$ 标出深度、射线及点监督。紫色相机 token 进入 $D_c$ 输出相机参数，配 $\mathcal L_C$。(4) 右下图例说明两类注意力。图中既画计算流，也画损失连接，虚线 teacher 不表示推理还要跑教师。

**机制与证据**：设每视图 $S$ 个 patch token，加相机 token 后序列长度 $S+1$。图内注意力可将输入整理为 $N$ 个长度 $S+1$ 的序列；跨视图注意力把同组 $N(S+1)$ 个 token 视为一个序列，使每视图的 query 能访问其他视图的 key/value。权重结构仍是预训练 ViT 自注意力，这就是“无架构改造”的实际含义；改变的是 tensor 排列及注意力范围。跨视图注意力计算量随序列长度增加，作者选择 $L_s:L_g=2:1$（§3.2），避免所有层都是高成本跨图操作。$N=1$ 两种排列等价。Dual-DPT 使用同源重组特征、独立融合，详细见 Fig.3。点图并非一套额外回归头，而是 $P=t+Dd$（p.6）。p.8 训练目标还含深度梯度损失、共同尺度归一化和 confidence 正则；图中未画全部公式。相机 head 是直接输出 FOV、四元数、平移的便捷支路，与射线 DLT 解算不同。

**证据边界**：图示结构支持论文关于信息流与可变输入的解释，实际计算速度需看 Table 8，设计收益由 Table 6/7 的消融支持。没有画出或未给出的 patch 维度、精确 backbone block 序号、所有 DPT 运算参数不能仅从示意图猜测；单一 Transformer 不等于没有相机编码器和解码器，也不意味着只用一个 loss。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
## 2.2.3 Figure 3（PDF p.7）

![Figure 3（PDF 区域渲染展示副本）](DA3/Figures/Figure_03.jpg)

**Caption 原文**（PDF p.7）：Figure 3 Dual-DPT Head. Two branchs share reassembly modules for better outputs alignment.

**Caption 中文**：Dual-DPT head。两条分支共享重组模块，以获得更好的输出对齐。

**布局与子图**：左侧竖排 Feature tokens 四个输入箭头接到四个 Reassemble；每个 Reassemble 同时送入浅绿色 fusion 和蓝色 fusion。两列各有四层 fusion，列内竖直箭头从上到下，分别接到底部独立 Output Layer；两列之间没有画跨列直接融合箭头。颜色只是区分分支，正文说明一列对应 depth，另一列对应 ray，图本身没有标明具体颜色对应哪一个。不能从图中判断某一层 token 的确切分辨率或通道数。

**机制与证据**：DPT 类 dense decoder 需要把序列 token 重新排列成空间结构，并把不同层的特征融合到高分辨率像素网格。该设计先共享重组映射 $A_k(F_k)$（$F_k$ 是第 $k$ 个输入尺度的视觉特征，$A_k$ 为对应重组模块；此为解释性记号），再分别用 $G_{D,k}$ 与 $G_{M,k}$ 融合。两条支路都从相同的空间对齐中间表征开始，不必独立学习两套 token-to-map 投影，同时保留 depth 和 6 通道 ray 的不同表达需求。预测任务的耦合在共享特征及损失监督上，而非把两张输出强制设成同一通道。由 $P=t+Dd$ 可知，如果像素网格的深度与射线在边缘处错位，就会把深度与错误方向配对，造成三维点偏差。共享重组是作者试图减少这种不一致的架构措施。

Table 7(d) 与 (a) 的对照评估两个独立 DPT vs Dual-DPT。在 HiRoom Auc3 39.2→5.59，F1 47.0→11.5；ETH3D F1 55.4→33.4，DTU CD 3.82→5.14 mm，表明大部分结果明显下降；但 7Scenes F1 从 47.6→49.4，有例外，不能说所有指标都下降。图示与表格支持该配置整体有利，而非证明共享重组本身必然消除误差。

**证据边界**：图中四个 reassembly 是结构示意，论文未在此给出精确的多尺度尺寸和全部融合算子。共享特征不是严格数学对齐约束，不保证遮挡边界一致，仍依赖训练监督。表7的优势也不能独立排除参数数量、优化动态等混杂因素。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
## 2.2.4 Figure 4（PDF p.8）

![Figure 4（PDF 区域渲染展示副本）](DA3/Figures/Figure_04.jpg)

**Caption 原文**（PDF p.8）：Figure 4 Poor quality real-world datasets. We show some examples of the poor quality real-world datasets.

**Caption 中文**：质量较差的真实世界数据集。图中展示若干质量不佳的真实世界数据集样例。

**布局与子图**：左上 DL3DV Colmap：网球场 RGB 的球网、围栏及远景对应深度/几何图只保留稀疏、断裂的彩色结构，大量黑色空缺。左下 WildRGBD LiDAR：图中粉色球和桌面，深度图在球体轮廓内部有明显黑环/孔洞，远处也有缺失。右列 Co3dV2 Colmap：苹果的 RGB 图和其下彩色稀疏三维点示意，细节和完整度远低于图像。三组并列不是对比模型输出，而是来源于不同真实数据标签机制。没有深度颜色标尺，不能给出各色深度值。

**机制与证据**：§3.3、§4 的动机是现实数据的几何标签不够密集/可靠。传统 SfM/MVS 的几何依赖可追踪纹理和多视图匹配，重复纹理、弱纹理、遮挡或细线结构使匹配不足；主动深度传感器也可能因材质、深度边界、测距有效范围等造成孔洞（这些是一般机制，不是论文逐图确认的具体原因）。直接把这些标签作为逐像素监督时，有效 mask $m_p$ 缺失处无法传递可靠训练信号；有噪声处会向错误深度提供梯度，细结构被削弱。教师由密集的合成标签训练，生成稠密相对深度 $\tilde D$，再用有效真实标签 $D$ 通过 Eq.(8) 拟合 $s\tilde D+t$，以真实几何维持尺度联系。教师的目的不是生成相机位姿，而是改善深度监督。相机/跨视图约束仍来自训练数据及联合目标。Fig.8 与 Table 7(e) 分别显示加入教师后的细节与几何结果；Appendix Fig.12–14 又说明合成数据也须清理。

**证据边界**：3 个样例足以展示质量问题存在，不给出全数据集坏标签比例，也不能证明三种采集方式的一般优劣。图中黑色部分可描述为无效/缺失的可见示例，不应赋予具体零深度值。对齐依赖可靠有效深度的覆盖与教师结构，不能把伪标签当成绝对真值。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
## 2.2.5 Figure 5（PDF p.14）

![Figure 5（PDF 区域渲染展示副本）](DA3/Figures/Figure_05.jpg)

**Caption 原文**（PDF p.14）：Figure 5 Comparisons of pose estimation quality. Camera trajectories for two videos are shown. Ground-truth trajectories are derived using COLMAP on images with dynamic objects masked.

**Caption 中文**：相机位姿估计质量对比。展示两段视频的相机轨迹，真值轨迹使用将动态物体掩蔽后的图像经 COLMAP 获得。

**布局与子图**：左侧两行 Reference video。第一行红帽人物/自拍镜头及背景发生变化；第二行为骑行头盔人物的不同帧。右侧逐列为 VGGT、Pi3、Ours、GT，蓝色轨迹与网格提供形状对照，GT 使用红线等标记。第一行 VGGT 的折返/曲折明显，Pi3 较平滑但仍弯折，DA3 接近 GT 的短直线变化；第二行 VGGT 仍有多段回折，Pi3 轨迹横向展开，DA3 接近 GT 的斜向线性轨迹。颜色并非误差热图，也没有明确坐标单位、时间轴或误差数值。

**机制与证据**：输入视频中动态前景会使常规静态场景匹配困难。GT 的构造显式将动态物体 mask 后再用 COLMAP 推断相机轨迹，从而尽可能依据稳定背景；因此图示比较的是相机运动估计，不是对运动人物的三维形变估计。DA3 通过跨视图 attention 建模共同场景，再由 ray map 的 origin/direction 或轻量相机头给出相机参数（§3.1–3.2），若几何方向和相机中心一致，则同一静态结构可约束多帧位姿。Table 2 的 RRA/RTA 阈值积分 AUC3/AUC30 是量化位姿评估，Fig.5 提供额外定性轨迹示例，不能直接从红蓝线的长度计算 AUC。

**证据边界**：GT 是 COLMAP 估计而非外部运动捕捉真值；作者对动态区域的掩蔽也不能消除所有误差。未明示各轨迹图的可视化尺度是否统一，不能凭网格比较绝对平移长度。仅两段视频，不能据此推出对所有动态/长视频稳定，§8 仍将动态推理作为未来方向。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
## 2.2.6 Figure 6（PDF p.16）

![Figure 6（PDF 区域渲染展示副本）](DA3/Figures/Figure_06.jpg)

**Caption 原文**（PDF p.16）：Figure 6 Comparisons of point cloud quality. Our model produces point clouds that are more geometrically regular and substantially less noisy than those generated by other methods.

**Caption 中文**：点云质量对比。我们的模型生成的点云较其他方法更具几何规则性，噪声显著更少。

**布局与子图**：两行分别是室内桌椅/屏幕场景和物体近景；列依次 Reference、Ours、VGGT、Pi3、Fast3R。第一列参考 RGB 提供场景外观定位。第一行 DA3 的桌面、墙面及细长支架较清晰，VGGT、Pi3、Fast3R 的背景或几何边界可见较多离散噪声、模糊点；第二行 DA3 的近景物体轮廓与内部环状结构较连续，VGGT/ Pi3 存在可见点噪声或结构变形，Fast3R 更模糊。根据参考图只能判断外观对应，不能确定各点的真实三维坐标。

**机制与证据**：用 $P_i(p)=t_i(p)+D_i(p)d_i(p)$ 将每视图深度映射到世界空间，如果 $D$ 或相机 ray 有偏差，同一物体从多视角得到的点就会形成重影、漂移或点状噪声。在小误差近似下（补充推导）$\delta P\approx\delta t+d\,\delta D+D\,\delta d$，因此远处结构尤其受方向误差影响。联合 depth-ray 监督、跨图推理和教师补足细节共同可能改善规则性；不是仅靠更锐利深度图就能保证融合正确。§6 通过相机对齐与 TSDF 融合计算 F1 或 CD，Table 3 提供带/不带 GT pose 的量化评价。Fig.6 是可视化例证，不明确展示点数量、重建筛选阈值及每场景对应的数值。

**证据边界**：规则性和噪声是视觉判断，平滑也可能移除真实细节。无法由图计算几何精度或完整率，也不能证明跨视图融合中每个点都正确。实验表的 F1 综合 precision/recall，CD 反映双向邻域距离，应以原协议的对齐和阈值为条件。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
## 2.2.7 Figure 7（PDF p.16）

![Figure 7（PDF 区域渲染展示副本）](DA3/Figures/Figure_07.jpg)

**Caption 原文**（PDF p.16）：Figure 7 Comparisons of depth quality. Compared with other methods, our depth maps exhibit finer structural detail and higher semantic correctness across diverse scenes.

**Caption 中文**：深度质量对比。与其他方法相比，我们的深度图在不同场景中具有更精细的结构细节及更高的语义正确性。

**布局与子图**：两行场景（近景物体/人物与室内广角大厅），六列依次为 RGB、Ours、VGGT、Pi3、Fast3R、MapAnything。每个预测列的红色矩形/放大块强调物体顶部轮廓、背景遮挡及室内横向细结构。第一行 DA3 在物体/背景交界的局部色阶轮廓相对清楚，其他方法出现边界模糊、孔洞或色块不规则；第二行深度预测的近端栏杆/座位与远端结构边缘，DA3 的局部结构更连续；不同预测间有色彩分布差别。原图未提供绝对色标，不把颜色深浅直接解释为统一数值。这里 semantic correctness 指与物体/场景结构相符的几何分区，而非语义分割任务的精度。

**机制与证据**：相对深度预测既需要整体深度排序也需要物体边界局部梯度。§3.3 的 $\mathcal L_{grad}=\|\nabla_x\hat D-\nabla_xD\|_1+\|\nabla_y\hat D-\nabla_yD\|_1$（Eq.3）约束深度梯度，教师的密集伪标签在有效深度缺失区域补足细节；Dual-DPT 让 depth-ray 从共同空间特征起步。假设相邻像素分别观察前景 $D_1$ 与远背景 $D_2$，模糊预测会把边界像素混成中间深度，反投影产生错误过渡表面；锐利且位置正确的梯度能减少此类过渡点。这是机制解释，不是该图对每个模块的独立消融。Fig.8 才直接展示有无 teacher 的对照；Table 3/4 从融合几何和单目 δ1 提供补充指标。

**证据边界**：定性色图不提供 ground-truth depth、统一颜色范围或每框误差，边界更锐利不必然深度值更准确。不能把语义合理性当语义分割准确率，也不能用此图单独断定相机准确性或度量尺度。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
## 2.2.8 Figure 8（PDF p.19）

![Figure 8（PDF 区域渲染展示副本）](DA3/Figures/Figure_08.jpg)

**Caption 原文**（PDF p.19）：Figure 8 Comparison of teacher-label supervision. Supervision with teacher-generated labels yields depth maps with substantially richer detail and finer structures.

**Caption 中文**：教师标签监督的比较。用教师生成的标签监督训练，得到的深度图具有明显更丰富的细节和更精细的结构。

**布局与子图**：四列为树林中的足球、鸟类、工程车辆及林间道路上的骑行者；每列三行依次 Image、w/o Teacher、w/ Teacher。第一列未使用教师时树干及背景容易呈现较粗糙的色块，使用后球、树干、枝条边界更清楚；第二列鸟的身体、颈部及细长腿在教师监督下具有更细的结构；第三列工程车辆的边缘、车身轮廓、局部机械形状分辨得更清楚；第四列林间背景树干及骑行者轮廓更分明。深度色彩也随预测变化，原图未提供统一颜色标尺。对比对象是学生模型有无教师标签，而不是原图与教师预测直接对比。

**机制与证据**：真实深度标签本来稀疏或含孔洞（Fig.4）；学生仅以这些标签训练，边缘和缺失处的局部深度梯度信息不足。合成训练的教师通过 depth gradient、global-local 与法向等监督（§4.1）学习更密集相对深度；Eq.8 通过 $D_{T\to M}=\hat s\tilde D+\hat t$ 与有效真实深度对齐，使细节学习与真实多视图尺度联系并存。若拟合参数 $\hat s>0$，对同一图内任意两像素的深度差有 $\Delta D_{T\to M}=\hat s\,\Delta\tilde D$，因此仿射对齐保留教师的相对排序与归一化边缘结构，而不需要每个像素都存在真实深度。训练时 p.8 指出在 120k steps 转换到 teacher 标签，故该策略不应解释为额外教师推理模型。

Table 7(a)/(e) 为量化补充：HiRoom 的 Auc3/F1（39.2/47.0 vs 11.2/16.0），7Scenes 和 ScanNet++ 有多项下降；但无教师在 ETH3D F1（57.6 vs 55.4）、DTU Auc3/CD（52.5/3.29 vs 45.8/3.82）更好。表明教师主要改善总体表现及细节，不是全数据集的单调提升。

**证据边界**：图的局部清晰度由定性视觉判断支持，没有对应的 GT depth 或逐像素误差，不能据此推出所有度量值更准确。若教师结构本身错误、真实锚点深度分布很窄，$s,t$ 的拟合不会纠正所有局部变形。图中差异不等价于受控测量每个损失项的贡献，后者应查 Table 9。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
## 2.2.9 Figure 9（PDF p.20）

![Figure 9（PDF 区域渲染展示副本）](DA3/Figures/Figure_09.jpg)

**Caption 原文**（PDF p.20）：Figure 9 Visualizations of camera pose and depth estimation on in-the-wild scenes.

**Caption 中文**：真实开放场景中的相机位姿与深度估计可视化。

**布局与子图**：左上 The Great Wall，13 images，展示部分输入及其深度色图，右侧为较大范围的长城及周围山地几何，并可见多个彩色相机视锥；左下 Colosseum，2 images，将两张不同视图的输入与深度配对并展示斗兽场及邻近区域几何、相机视锥；右侧 Statue of Liberty，1 image，上半区为自由女神像及基座周边的重建几何与相机视锥，下半区显示输入 RGB 和预测深度。三块用边框划分，输入数量直接标在地标名称下方。长城示意省略部分帧，不是只用图中两张图；右侧自由女神像只有一个实际输入。没有显示 GT mesh、测量尺度或定量误差。

**机制与证据**：§3.1 允许任意视图数量，$N$ 张图各自输出深度和 ray map；$N=13$ 时跨图 attention 有多个视图提供共享结构信息，$N=2$ 时只有少量跨视图约束；$N=1$ 时只能由单图内容及学习先验预测深度，没有其他视图提供几何三角测量约束。若已估计统一世界坐标的 ray map，就能组合 $P=t+Dd$ 并绘制相机视锥。Fig.9 证实作者实际展示了三类输入数量的几何结果；可见自由女神例的细节并非对其背面或全场景的真实恢复证明。此图也与 §7.2.3 的“More visualizations”对应，目的是展示 in-the-wild 适应性，而不是特定数据集量化 benchmark。Table 2/3 的协议用于另行比较位姿和几何准确性。

**证据边界**：不同场景、输入视图数和分辨率变化，图中不是控制场景后的视图数消融；无法据此计算从 1 到 2 到 13 张图的精度收益或最小视图需求。图中可见重建范围不等同真实全空间，单目绝对尺度不能由此证明。也没有无重叠输入、极端遮挡或动态场景下的专项量化。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN FIG-10 -->
## 2.2.10 Figure 10（PDF p.21）

![Figure 10（PDF 区域渲染展示副本）](DA3/Figures/Figure_10.jpg)

**Caption 原文**（PDF p.21）：Figure 10 Effectiveness of Teacher model for supervising metric depth estimation. Incorporating Teacher model for supervision significantly improves the metric depth sharpness.

**Caption 中文**：教师模型对于监督度量深度估计的有效性。引入教师模型进行监督，显著改善了度量深度的清晰度。

**布局与子图**：左右各一组三列，标签均为 Input Image、w/ DA3 teacher、w/o DA3 teacher；每组三列纵向四行，共八个样例。左组依次是室内工作/桌椅环境、室内沙发/座椅场景、带花瓶/物件的桌边近景及远处林地；右组为室内厨房/柜体、桌椅与家具场景、室内植物与柜体、户外场景。每行对应输入及两张度量深度预测，教师版本在椅背、柜体、沙发、花朵/植物轮廓等处总体更清楚。某些大平面颜色也有变化，但图没有标尺，不能从色阶断言更接近真值。此处教师不是运行在推理输入上的额外后处理，而是 training supervision。

**机制与证据**：§4.4 以 canonical focal 处理不同焦距下深度的尺度问题：使用 $D^c=(f^c/f)D$ 转换训练真值，$f^c=300$。教师产生 relative depth，再以有效 metric label 对齐尺度和平移，使其可监督度量学生；训练有 20% 概率使用原 GT，以保留真实深度的监督。因为仿射变换在 $s>0$ 时维持相对边缘梯度，教师可以为局部结构提供比稀疏真值更密集的目标，但这种细节增强不保证绝对值更准确。

Table 11 的 ablation 明确呈现指标取舍：w/ teacher 的 NYUv2 δ1=0.966、AbsRel=0.073，w/o teacher 的 δ1=0.969、AbsRel=0.066；ETH3D 和 DIODE 的指标也可能出现相反变化（逐值见 Table 11 单元），因此作者强调的 sharpness 与数值绝对误差是不同维度。原文 §7.4 还说明最终模型和消融训练分辨率略不同，不应直接把最终行与消融行作为单变量严格对照。

**证据边界**：八个可见案例支持教师监督改善细节的定性主张，但没有同时显示真值深度图，不支持仅凭锐度推断 metric accuracy。相机焦距归一化只是训练坐标规范，不等于单张图的真实尺度从观测严格唯一确定；度量预测依赖训练中学到的先验和标签。

<!-- PTH:END FIG-10 -->

<!-- PTH:BEGIN FIG-11 -->
## 2.2.11 Figure 11（PDF p.23）

![Figure 11（PDF 区域渲染展示副本）](DA3/Figures/Figure_11.jpg)

**Caption 原文**（PDF p.23）：Figure 11 Qualitative comparisons with state-of-the-art methods for visual rendering. The first column shows the selected input views, while the remaining columns display novel views rendered by comparison models and ground truth. For each scene, two rendered novel viewpoints are presented in consecutive rows. The first three scenes are from DL3DV, the following two are from Tanks and Temples, and the last three are from MegaDepth. Compared to other methods, our model consistently achieves superior rendering quality across diverse and challenging scenes.

**Caption 中文**：与最先进视觉渲染方法的定性比较。第一列为选取的输入视图，其余列显示比较模型和真值的新视角渲染结果。每个场景连续两行表示两个新视点。前三个场景来自 DL3DV，接下来两个来自 Tanks and Temples，最后三个来自 MegaDepth。作者认为，相较其他方法，本模型在多样且富有挑战性的场景中持续取得更优渲染质量。

**布局与子图**：整图八个场景组，每组两行目标视点、最左列用缩略输入和省略号示意 context。列标题从左到右 Inputs、MVSplat、DepthSplat、Fast3R、VGGT、DA3 (Ours)、Ground Truth。第一组玻璃/金属屋顶及支柱，第二组停车区域/工地护栏，第三组室外建筑及屋檐，属于 DL3DV。第四组蓝色汽车与街景、第五组金色室内装饰为 Tanks and Temples；第六组室内红地毯与柱廊、第七组古典建筑/神庙、第八组砖石古迹为 MegaDepth。每组两个视点帮助区分固定视角下的外观拟合与不同视角时的几何一致性。第一组三角棚顶与立柱、第三组屋顶与门窗、第四组车辆轮廓、第七组柱廊以及最后一组砖石边缘，在 DA3 列整体更接近 GT；部分其他列可见模糊、漂浮结构、重影或纹理边缘变化。DA3 也不与 GT 完全一致，如大场景细纹理、遮挡边缘仍存在可见差异。没有把每个像素的色差从 JPG 量化为 PSNR。

**机制与证据**：§5 的 pose-conditioned GS-DPT 以几何主干视觉 token 为条件，预测像素对齐的 3D Gaussian 参数，使用预测深度经相机反投影确定世界中心 $P$，再从目标相机 rasterize。与仅学习外观的目标不同，训练还使用深度监督，几何 backbone 能把不同视图的公共结构编码到同一空间，减少投影错误造成的重影。若中心位置误差 $\delta P$ 在源视角可被颜色/不透明度补偿，换一个目标相机时投影误差仍会表现成结构偏移；这解释为什么几何质量有助于 NVS，但不是对所有画质因素的充分解释。Table 5 在 DL3DV(140)、Tanks and Temples(6)、MegaDepth(19) 三域的同一输入数 12、270×480、目标每隔8帧协议下，DA3 在 PSNR/SSIM/LPIPS 各项均最佳。作者 §7.1、§7.5 说明 baseline 使用大体统一训练设置并加入深度训练损失（pixelSplat 为 100k steps，其余为 200k），几何主干组与专门 NVS 模型比较验证几何表征价值。

**证据边界**：输入 GT pose 的对比不能视为 pose-free NVS 的定量验证。不同模型的架构和预训练数据可能仍不同，不能把所有收益唯一归因于表示或教师。图中八组样例不是测试集全部 165 场景，且真实 GT 是图像真值，不是完整三维几何真值。更好的 RGB 指标可与不正确几何并存，需结合 Table 3 几何指标。

<!-- PTH:END FIG-11 -->

<!-- PTH:BEGIN FIG-12 -->
## 2.2.12 Figure 12（PDF p.31）

![Figure 12（PDF 区域渲染展示副本）](DA3/Figures/Figure_12.jpg)

**Caption 原文**（PDF p.31）：Figure 12 Invalid background of amusement scene in TartanAir dataset.

**Caption 中文**：TartanAir 数据集中 amusement 场景的无效背景。

**布局与子图**：左侧 RGB 为带娱乐设施/树木的户外场景，旁边伪彩深度图前景结构清晰但背景存在异常。右侧两个较大三维点云视图显示由深度生成的场景，右图红框圈出上方异常背景区域，能看到背景几何形成不自然的曲面/堆积。此图是合成数据的标签问题，不是 DA3 模型预测失败示例；原图没有深度色标、坐标比例或具体异常像素统计。

**机制与证据**：附录 A（p.31）作者明确移除 TartanAir 的 amusement 场景，因为存在无效 skybox（天空背景）。合成渲染的深度可能使用有限远平面、无效背景或不合理的天空几何；将错误深度 $D$ 与相机射线 $d$ 组合 $P=t+Dd$ 会把原本不可定义或极远的区域强行放到有限空间，出现错误三维表面。训练相对深度教师时，即使不需要真实场景尺度，也会从这种局部深度排序/梯度错误学习错误结构；法向监督还会放大非物理表面信息。移除不可靠场景是教师数据质量控制的一部分，而非仅依赖数据量。§4.1 的数据扩展与 Table 9 反映综合训练结果，不能单独定量该场景的清除贡献。

**证据边界**：图证实作者发现该场景异常、采取清理，不代表所有 TartanAir 样本都无效；背景错误的具体成因/异常深度数值未由题注明确，不假定真实深度为零或无限。此图不是 DA3 的测试结果。

<!-- PTH:END FIG-12 -->

<!-- PTH:BEGIN FIG-13 -->
## 2.2.13 Figure 13（PDF p.31）

![Figure 13（PDF 区域渲染展示副本）](DA3/Figures/Figure_13.jpg)

**Caption 原文**（PDF p.31）：Figure 13 Image-depth spatial misalignment in IRS dataset.

**Caption 中文**：IRS 数据集中的图像与深度空间错位。

**布局与子图**：左右两组各两张图，左图是室内沙发、茶几等 RGB，右图为与深度伪彩叠加后效果；右组是室内架子/柜体的 RGB 与对应叠加图。红色箭头标出 RGB 边缘与深度边缘不重合的位置，例如物体/家具轮廓和柜体深度变化处。可见深度色彩区域与图像结构存在系统偏移，而不是单一深度值有随机误差。叠加图不是经过模型修复后的输出。

**机制与证据**：图像–深度配对的监督假定 $I(u,v)$ 与 $D(u,v)$ 描述同一像素射线；若真实对应 $D(u+\Delta u,v+\Delta v)$ 被放在 $(u,v)$，会把另一表面的深度错误赋给当前像素。沿深度不连续边缘尤其严重，梯度损失会奖励错位边界，反投影 $P=t+Dd$ 也会形成错位几何。这解释数据清理为何不能用简单尺度对齐替代：$sD+t$ 可以调整深度值，不能纠正空间索引偏移。附录 A 的 IRS 段落具体使用图像/深度空间对齐检测，作者先对灰度 RGB 与深度分别做 Canny 边缘检测，膨胀 1 像素后计算交集百分比，再膨胀 3 像素计算交集，以两次结果的比值筛除低于阈值的样本（p.31）。

**证据边界**：图中红箭头证实错位存在，但没有标定误差的定量分布；不能从示例推断所有 IRS 场景都有相同偏移方向和像素量，也不能把作者筛选流程视为对全部训练标签的无误差保证。

<!-- PTH:END FIG-13 -->

<!-- PTH:BEGIN FIG-14 -->
## 2.2.14 Figure 14（PDF p.32）

![Figure 14（PDF 区域渲染展示副本）](DA3/Figures/Figure_14.jpg)

**Caption 原文**（PDF p.32）：Figure 14 Erroneous samples in UnrealStereo4K dataset. The windows in scene 00003 are transparent; scene 00004 suffers from clipping issue; the sea in scene 00008 does not have depth values.

**Caption 中文**：UnrealStereo4K 数据集中错误样本。00003 场景中的窗户是透明的；00004 存在裁切问题；00008 的海面没有深度值。

**布局与子图**：(a) 00003 位于左上，建筑窗户 RGB 及其深度伪彩图，窗格/玻璃区域的深度行为与可见表面不一致。(b) 00004 位于右上，室内/街巷结构及深度图，红色箭头指向深度图中不自然的亮色裁切边缘。(c) 00008 位于下方两组 RGB 与深度，海岸/树木景观中海面区域存在缺失/不合理的深度（题注明确没有 depth values）。三种不同机制分别对应透明材质、裁切和未定义深度。原图保留 (a)(b)(c) 标签与场景 ID，没有统一深度数值色条。

**机制与证据**：相对深度教师需要从合成 RGB–depth pairs 学习真实可见表面。透明材质的 RGB 可能显示透射后的背景，深度渲染却记录玻璃面或不一致的几何；深度裁切使不同真实距离映射为同一边界值，缺失海面深度则无可靠监督。若将错误像素作为有效 mask 送入 $\mathcal L_{grad}$、法向误差或深度误差，会使模型学习错误边界和表面。附录 A 作者完整删除场景 00003 与 00004，并删除 00008 中指定编号的错误图像；具体保留/删除规则需依据原文文字而非从红箭头猜测，见后续 EXP-APPENDIX。它与 Fig.12/13 共同构成合成数据质量控制的证据，说明“合成数据密集”不自动等于正确深度。

**证据边界**：样例不能证明 DA3 对一般透明体、镜面或水面能正确恢复物理结构；作者只说明训练数据中的错误及清理。未提供三类问题各自的独立消融或剩余错误率，不能量化清理对 Table 9 的单独贡献。

<!-- PTH:END FIG-14 -->

<!-- PTH:BEGIN TABLE-01 -->
### Table 1（PDF p.9）

#### 题注与协议

**Caption 原文：** Datasets used in Depth Anything 3, including number of scenes, data type.

**Caption 中文：** Depth Anything 3 所用数据集，包括场景数量和数据类型。

【原文事实】§3.3、§6.3。此表列出 pose-geometry benchmark 与 pose-geometry training，不是单目教师训练集或 NVS 划分。场景数不是图像数。

#### 表格数据

| 用途 | Dataset | 场景数 | 类型 |
| --- | --- | --- | --- |
| Benchmark | HiRoom(ours) | 29 | Synthetic |
| Benchmark | ETH3D | 11 | LiDAR |
| Benchmark | DTU | 22 | LiDAR |
| Benchmark | 7Scenes | 7 | LiDAR |
| Benchmark | ScanNet++ | 20 | LiDAR |
| Training | AriaDigitalTwin | 237 | Synthetic |
| Training | AriaSyntheticENV | 99950 | Synthetic |
| Training | ArkitScenes | 4388 | LiDAR |
| Training | BlendedMVS | 503 | 3D_Recon |
| Training | Co3dv2 | 30616 | Colmap |
| Training | DL3DV | 6379 | Colmap |
| Training | HyperSim | 344 | Synthetic |
| Training | MapFree | 921 | Colmap |
| Training | MegaDepth | 268 | Colmap |
| Training | MegaSynth | 6049 | Synthetic |
| Training | MvsSynth | 121 | Synthetic |
| Training | Objaverse | 505557 | Synthetic |
| Training | Omniobject | 5885 | Synthetic |
| Training | OmniWorld | 1039 | Synthetic |
| Training | PointOdyssey | 44 | Synthetic |
| Training | ReplicaVMAP | 17 | Synthetic |
| Training | ScanNet++ | 230 | LiDAR |
| Training | ScenenetRGBD | 16866 | Synthetic |
| Training | TartanAir | 355 | Synthetic |
| Training | Trellis | 557408 | Synthetic |
| Training | vKitti2 | 50 | Synthetic |
| Training | WildRGBD | 23050 | LiDAR |

#### 实验分析

【原文事实】测试覆盖合成室内、真实室内/室外、物体级扫描；§6.3 的 HiRoom 描述为 30 个场景，而表中是 29，未说明具体少了哪个。ETH3D 选 11 个，DTU 选 22 个 evaluation scans（49 视角/场景），7Scenes 以 11 为间隔下采样，ScanNet++ 选 20 个场景、以 5 为间隔下采样；一般位姿协议最多固定种子随机选 100 张。训练中 ScanNet++ 230 与测试 20 分列。

【合理推断】Objaverse、Trellis 的数量比小型室内扫描集高数个数量级，但合成物体的场景计数与真实视频场景不能直接视为同等监督量。合成深度可覆盖细边缘；LiDAR 易受无效点/采样密度影响；COLMAP 来源的重建依赖匹配和几何求解。真实与合成混合有利于扩展外观与几何覆盖，但不同质量的深度需要 §3.3 的监督设计与 §4 的教师补充。

#### 证据边界

【证据边界】表中 5 个测试集共 89 个场景（29+11+22+7+20），不应把 §6.3 所述 30 个 HiRoom 直接加进来。ScanNet++ 列出训练及测试不同数量，但不能仅靠表中数量证明场景完全无交叉。各数据源具体采样权重和监督密度未由表给出，不据场景数推导训练贡献或泛化因果。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
### Table 2（PDF p.14）

#### 题注与协议

**Caption 原文：** Comparisons with SOTA methods on pose accuracy. We report both Auc3 ↑ and Auc30 ↑ metrics. The top-3 results are highlighted as first, second, and third.

**Caption 中文：** 与先进方法比较位姿准确率，报告 Auc3 和 Auc30（越大越好），原表突出前三名。

【原文事实】§6.1–6.2、§7.1。相对旋转与平移角误差的阈值准确率，曲线取两者较小值后积分，Auc3 对小误差更敏感，Auc30 容许更大误差。表中为百分数尺度，参数 B 为十亿。

#### 表格数据

| 方法 | Params | HiRoom Auc3 | Auc30 | ETH3D Auc3 | Auc30 | DTU Auc3 | Auc30 | 7Scenes Auc3 | Auc30 | ScanNet++ Auc3 | Auc30 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DUSt3R | 0.57B | 17.6 | 54.3 | 4.30 | 27.3 | 4.00 | 74.3 | 6.90 | 61.6 | 8.10 | 33.9 |
| Fast3R | 0.65B | 25.9 | 77.0 | 8.10 | 44.4 | 9.50 | 79.1 | 19.0 | 78.6 | 17.9 | 72.5 |
| MapAnything | 0.56B | 17.9 | 82.8 | 19.2 | 77.4 | 6.50 | 72.7 | 12.6 | 79.7 | 20.2 | 84.1 |
| Pi3 | 0.96B | 67.0 | 94.8 | 35.2 | 87.3 | 62.5 | 94.9 | 25.5 | 86.3 | 50.7 | 92.1 |
| VGGT | 1.19B | 49.1 | 88.0 | 26.3 | 80.8 | 79.2 | 99.8 | 23.9 | 85.0 | 62.6 | 95.1 |
| DA3-Giant | 1.10B | 80.3 | 95.9 | 48.4 | 91.2 | 94.1 | 99.4 | 28.5 | 86.8 | 85.0 | 98.1 |
| DA3-Large | 0.36B | 58.7 | 94.2 | 32.2 | 86.9 | 70.2 | 96.7 | 29.2 | 86.6 | 60.2 | 94.7 |
| DA3-Base | 0.11B | 19.0 | 83.2 | 15.1 | 74.6 | 60.1 | 95.9 | 20.1 | 82.9 | 25.1 | 83.4 |
| DA3-Small | 0.03B | 9.49 | 75.2 | 8.59 | 62.1 | 30.6 | 91.2 | 14.0 | 78.7 | 10.9 | 71.9 |

#### 实验分析

【独立评价】逐域与最佳外部 baseline 比，Giant 的 Auc3：HiRoom 对 Pi3 +13.3 点（19.9%）；ETH3D 对 Pi3 +13.2 点（37.5%）；DTU 对 VGGT +14.9 点（18.8%）；7Scenes 对 Pi3 +3.0 点（11.8%）；ScanNet++ 对 VGGT +22.4 点（35.8%）。原文称 ScanNet++ 33%，与直接表值计算 35.8% 有差异。Auc30 同样比最佳外部 baseline 分别 +1.1、+3.9、−0.4、+0.5、+3.0 点。Giant 不是所有 DA3 中每列最优：Large 的 7Scenes Auc3 为 29.2，高于 28.5。DTU Auc30 VGGT 99.8 接近饱和；Auc3 差异仍明显，说明宽松阈值掩盖精细位姿差异。与 VGGT 比，Large 在 HiRoom/ETH3D 两阈值及 7Scenes 两阈值领先，DTU、ScanNet++ 落后；小模型提升并非随参数量单调逐列成立。

#### 证据边界

【证据边界】表 2 的角误差衡量相对姿态/平移方向，不等于绝对相机位置或度量尺度准确。不同模型数据、训练与结构不完全受控。按平均值比、逐域相对比再平均、只取 Auc3 或同时取两阈值会产生不同数字；摘要 35.7% 不直接用某个单域百分比代替。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
### Table 3（PDF p.15）

#### 题注与协议

**Caption 原文：** Comparisons with SOTA methods on reconstruction accuracy. For all datasets except DTU, we report the F-Score (F1 ↑). For DTU, we report the chamfer distance (CD ↓, unit: mm). w/o p. and w/ p. denote without pose and with pose, indicating whether ground-truth camera poses are provided for reconstruction. The top-3 results are highlighted as first, second, and third.

**Caption 中文：** 重建准确率比较：除 DTU 使用毫米单位的 Chamfer Distance（越低越好），其余使用 F1（越高越好）。w/o p.、w/ p. 表示重建是否提供真值相机位姿，原表标注前三名。

【原文事实】§6.1–6.3、§7.1。无位姿时用预测深度和位姿，经 evo/RANSAC 对齐再 TSDF 融合；有位姿时用 GT pose 融合，支持相机条件的模型还可输入已知相机。F1 是阈值距离内 precision/recall 的调和均值，CD 是双向点集距离平均。

#### 表格数据

| 方法 | Params | HiRoom 无/有pose F1（无） | 有 | ETH3D 无 | 有 | DTU 无 CD | 有 | 7Scenes 无 F1 | 有 | ScanNet++ 无 F1 | 有 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DUSt3R | 0.57B | 30.1 | 39.5 | 19.7 | 18.8 | 7.60 | 7.97 | 26.6 | 39.8 | 18.9 | 27.3 |
| Fast3R | 0.65B | 40.7 | 48.2 | 38.5 | 50.3 | 6.88 | 8.20 | 41.0 | 49.8 | 37.1 | 53.7 |
| MapAnything | 0.56B | 32.4 | 69.2 | 54.8 | 71.9 | 7.91 | 3.97 | 44.8 | 55.2 | 39.4 | 71.3 |
| Pi3 | 0.96B | 75.8 | 85.0 | 72.7 | 80.6 | 3.28 | 1.72 | 44.2 | 57.5 | 63.1 | 73.3 |
| VGGT | 1.19B | 56.7 | 70.2 | 57.2 | 66.7 | 2.05 | 1.44 | 47.9 | 51.4 | 66.4 | 70.7 |
| DA3-Giant | 1.10B | 85.1 | 95.6 | 79.0 | 87.1 | 1.85 | 1.85 | 53.5 | 56.5 | 77.0 | 79.3 |
| DA3-Large | 0.36B | 69.5 | 87.1 | 65.8 | 75.2 | 2.08 | 1.23 | 56.3 | 49.2 | 67.9 | 75.7 |
| DA3-Base | 0.11B | 25.9 | 71.4 | 49.5 | 66.7 | 2.87 | 2.36 | 49.9 | 50.6 | 47.2 | 67.8 |
| DA3-Small | 0.03B | 18.3 | 52.2 | 41.6 | 63.4 | 5.83 | 2.49 | 41.0 | 46.8 | 32.3 | 53.8 |

#### 实验分析

【独立评价】Giant 相对 VGGT 的无位姿 F1 增益：HiRoom +28.4 点（50.1%）、ETH3D +21.8（38.1%）、7Scenes +5.6（11.7%）、ScanNet++ +10.6（16.0%）；DTU CD 减少 0.20 mm（9.8%）。五项按改善方向分别相对化后等权平均约 25.1%，与 p.15 正文一致，而非本地摘要 23.6%。无位姿时 Giant 胜五种外部 baseline，但 7Scenes Large 56.3 更优。有位姿相对无位姿，Giant F1 分别 +10.5、+8.1、+3.0、+2.3 点，DTU CD 不变；Large 7Scenes 反降 7.1 点。DTU 已知位姿 Large 1.23 优于 VGGT 1.44，Giant 1.85 反逊；7Scenes 已知位姿 Pi3 57.5 最高。HiRoom/ETH3D/ScanNet++ 已知位姿 Giant 领先。Large 与 VGGT 的十列逐项比较为八列领先、两列落后（DTU 无 pose、7Scenes 有 pose），与正文称五项不同；应保留原表与正文差异。

#### 证据边界

【证据边界】F1 的距离阈值和 voxel size 随数据集变化（HiRoom 0.05m/0.007m，ETH3D d=0.25/0.039m，7Scenes 0.05m/0.007m，ScanNet++ 0.05m/0.02m；ETH3D 原文 d 未标单位）。跨域原始 F1 不直接衡量同一精度。对齐消去部分全局误差；有位姿的两种方法可能利用相机条件，而其他模型仅用真值融合，不是严格同结构干预。不能据此证明深度本身拥有真实绝对尺度。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
### Table 4（PDF p.15）

#### 题注与协议

**Caption 原文：** Monocular depth comparisons. δ1 ↑

**Caption 中文：** 单目深度比较，δ1 越大越好。

【原文事实】§7.1、§7.3。采用 DA2 标准单目评估；δ1 是预测/真值比例及其倒数均小于 1.25 的有效像素比例（背景知识定义），表为百分尺度。该表不是 Table 11 的 metric depth 无尺度对齐任务。

#### 表格数据

| Method | KITTI | NYU | SINTEL | ETH3D | DIODE | Rank |
| --- | --- | --- | --- | --- | --- | --- |
| DA2 | 94.6 | 97.9 | 77.2 | 86.5 | 95.2 | 2.60 |
| VGGT | 91.7 | 97.9 | 67.9 | 97.5 | 95.3 | 3.75 |
| DA3 | 95.3 | 97.4 | 75.5 | 98.6 | 95.4 | 2.20 |
| Teacher | 97.2 | 97.9 | 81.4 | 99.8 | 96.6 | 1.00 |

#### 实验分析

【独立评价】DA3−DA2 各域为 +0.7、−0.5、−1.7、+12.1、+0.2 点；DA3−VGGT 为 +3.6、−0.5、+7.6、+1.1、+0.1 点。故总体 Rank 领先但不是逐域全胜，NYU 与 SINTEL 是相对 DA2 的例外。Teacher 对 DA3 五域均更高（+1.9、+0.5、+5.9、+1.2、+1.2 点），与教师专用单目监督目标一致；NYU Teacher 与 DA2/VGGT 均 97.9。Rank 为原表报告值，不从舍入值反推精确排序规则。

#### 证据边界

【证据边界】δ1 只计算阈值内比例，不说明大误差尾部、边缘连续性或跨视角一致性。Teacher 不是多视角 DA3 的另一个名称，不能将其得分直接当作 DA3 主模型性能；单目学生的单独实验见 Table 10。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
### Table 5（PDF p.17）

#### 题注与协议

**Caption 原文：** Comparisons with SOTA methods on NVS task. We report NVS comparsions with exisiting feed-forward 3DGS models and counterparts using other backbones. For each scene, we use 12 input context views and test on target views sampled every 8 views over a set of over 300 views. Image resolution is 270 × 480.

**Caption 中文：** NVS 与已有前馈 3DGS 及替换几何骨干网络的方法比较；每场景 12 张上下文图像，从 300 多张中每 8 张选目标，分辨率 270×480。

【原文事实】§6.1、§6.3、§7.1、§7.5：COLMAP 相机直接提供，剩余视图按相机旋转和平移最远点采样输入；DL3DV 训练集统一训练，全部加入 depth loss；8 A100、batch=1、200K 步（pixelSplat 100K）。PSNR dB/SSIM 越高越好，LPIPS 越低越好。

#### 表格数据

| 方法 | DL3DV(140) PSNR | SSIM | LPIPS | T&T(6) PSNR | SSIM | LPIPS | MegaDepth(19) PSNR | SSIM | LPIPS |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| pixelSplat | 16.55 | 0.456 | 0.480 | 13.81 | 0.347 | 0.558 | 13.87 | 0.367 | 0.561 |
| MVSplat | 18.13 | 0.559 | 0.393 | 14.81 | 0.391 | 0.508 | 14.67 | 0.398 | 0.533 |
| DepthSplat | 19.24 | 0.620 | 0.322 | 15.80 | 0.474 | 0.418 | 15.90 | 0.471 | 0.450 |
| Fast3R | 19.30 | 0.604 | 0.320 | 16.24 | 0.478 | 0.409 | 16.43 | 0.493 | 0.421 |
| MV-DUSt3R | 20.01 | 0.645 | 0.294 | 17.04 | 0.529 | 0.370 | 16.20 | 0.484 | 0.437 |
| VGGT | 20.96 | 0.697 | 0.253 | 17.18 | 0.550 | 0.347 | 16.45 | 0.500 | 0.417 |
| DAv3(Ours) | 21.33 | 0.711 | 0.241 | 18.10 | 0.578 | 0.311 | 17.89 | 0.561 | 0.351 |

#### 实验分析

【独立评价】DA3 比最佳外部基线 VGGT 在三域 PSNR +0.37/+0.92/+1.44 dB，SSIM +0.014/+0.028/+0.061，LPIPS −0.012/−0.036/−0.066。相对 DepthSplat 的 PSNR +2.09/+2.30/+1.99 dB。所有模型 DL3DV 的分数均明显优于域外，DA3 自身 PSNR 下降 3.23/3.44 dB；域外绝对优势更大但仍存在域偏移。几何 backbone + GS head 的路线总体领先专用模型，不过 Fast3R 在 DL3DV SSIM 0.604 低于 DepthSplat 0.620。背景知识：PSNR=10 log10(MAX²/MSE)，相同动态范围下 +1.44dB 对应 MSE 比约 10^(−0.144)=0.718，即约下降28%，不是 PSNR 提升 1.44%。

#### 证据边界

【证据边界】相同采样和监督增强了比较公平性，但 pixelSplat 训练步数不同，预训练、容量等仍是混杂因素。域外差距可能来自轨迹、相机、内容、曝光等，作者归因于轨迹/姿态分布未单独控制。该表是已知 GT pose 渲染，不能证明无 pose 的同等 NVS；颜色质量也不能直接等价于真实三维几何。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
### Table 6（PDF p.17）

#### 题注与协议

**Caption 原文：** Ablations of prediction-target combinations. Note that all experiments in this table do not have camera condition token. The best and second best are highlighted.

**Caption 中文：** 预测目标组合消融；所有实验均无相机条件 token，突出最优和第二优。

【原文事实】§7.2.1。pcd 为 point cloud/point map，cam 为显式相机头，ray 是稠密射线表示。§7.2.1（p.16–17）明确：无 pcd 的模型通过 depth 与 ray 或 cam 获得的相机参数构建点云；有 pcd 组直接预测三维点云，不能假定其最终评估完全采用相同的反投影路径。所有模型使用 ViT-L、10 views、batch128、120k steps；ray 使用 Dual-DPT，pcd 使用独立 DPT。

#### 表格数据

| 目标 | HiRoom Auc3 | F1 | ETH3D Auc3 | F1 | DTU Auc3 | CD | 7Scenes Auc3 | F1 | ScanNet++ Auc3 | F1 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| depth+pcd+cam | 9.1 | 12.8 | 19.0 | 60.4 | 42.3 | 4.918 | 20.8 | 43.4 | 22.0 | 43.0 |
| depth+cam | 10.8 | 16.5 | 9.9 | 48.0 | 23.3 | 5.316 | 13.0 | 38.5 | 13.3 | 41.0 |
| depth+ray | 48.7 | 60.3 | 25.5 | 65.4 | 46.5 | 3.919 | 24.0 | 46.5 | 35.5 | 53.4 |
| depth+ray+cam | 37.2 | 45.4 | 22.3 | 59.4 | 56.3 | 3.066 | 25.7 | 45.6 | 34.1 | 56.5 |

#### 实验分析

【独立评价】depth+ray 相对 depth+cam 五域 Auc3 分别 +37.9/+15.6/+23.2/+11.0/+22.2 点；平均 Auc3 36.04 对 14.06，按均值比 +156.3%，不是唯一能复现作者 nearly 100% 的口径。相对 depth+pcd+cam 五域位姿也全优，F1 四域全优、CD 更小。比较 depth+ray+cam 与 depth+ray：HiRoom (−11.5,−14.9)、ETH3D(−3.2,−6.0)、DTU(+9.8,−0.853mm)、7Scenes(+1.7,−0.9)、ScanNet++(−1.4,+3.1)。额外 cam 不是全无益处，而是取舍。机制解释：depth+ray 以逐像素稠密方向和共享原点约束相机，P=t+Dd 可导出点图，减少独立点图目标间不一致；cam head 的直接回归与 ray 几何求解是不同途径。作者仍采用三头目标，称 cam 额外计算约主 backbone 的 0.1%。

#### 证据边界

【证据边界】无相机条件 token 下的受控组合比较支持 depth-ray 的实用充分性，不是数学唯一最优或学习理论证明；cam 辅助头可能改善某些域，不宜将 no further benefit 解释为每列无收益。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN TABLE-07 -->
### Table 7（PDF p.18）

#### 题注与协议

**Caption 原文：** Ablation study. We evaluate three architectural designs with comparable model sizes (a-c), the effects of the dual-DPT head (d), teacher label supervision (e), and the pose conditioning module (f-g). The best and second best are highlighted. Methods marked with "*" are evaluated with ground-truth pose fusion.

**Caption 中文：** 消融研究：比较规模相近的三种架构(a–c)、Dual-DPT(d)、教师标签(e)、位姿条件(f–g)。原表突出最优和第二优；星号表示真值位姿融合。

【原文事实】§7.2.2–7.2.3。a–e 预测位姿融合；f–g 使用真值融合，Auc3 空白，不能填零。VGGT-style 是论文重建的同容量架构对照，而非主表 VGGT 成品。

#### 表格数据

| 配置 | HiRoom Auc3 | F1 | ETH3D Auc3 | F1 | DTU Auc3 | CD | 7Scenes Auc3 | F1 | ScanNet++ Auc3 | F1 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| a.Proposed | 39.2 | 47.0 | 21.0 | 55.4 | 45.8 | 3.82 | 26.2 | 47.6 | 30.3 | 51.1 |
| b.VGGT_style | 3.72 | 14.5 | 2.31 | 27.4 | 1.38 | 6.93 | 0.97 | 21.4 | 2.03 | 12.2 |
| c.Full_Alt. | 24.7 | 29.3 | 13.1 | 51.9 | 44.6 | 4.23 | 21.1 | 48.6 | 27.7 | 47.5 |
| d.w/o_Dual_DPT | 5.59 | 11.5 | 13.6 | 33.4 | 21.7 | 5.14 | 14.2 | 49.4 | 26.5 | 46.6 |
| e.w/o_Teacher | 11.2 | 16.0 | 16.2 | 57.6 | 52.5 | 3.29 | 23.3 | 40.3 | 26.2 | 47.7 |
| f.w/o_Pose_Cond.* | — | 65.8 | — | 63.2 | — | 3.65 | — | 58.4 | — | 62.8 |
| g.w/_Pose_Cond.* | — | 73.8 | — | 70.9 | — | 2.14 | — | 46.0 | — | 65.7 |

#### 实验分析

【独立评价】a 比 b 所有指标好；如 DTU Auc3 45.8 vs 1.38，CD 3.82 vs 6.93。作者将其归因于单骨干完整预训练与对照中新增块未预训练，但这是组合差异。a 对 c 五域 Auc3 分别 +14.5/+7.9/+1.2/+5.1/+2.6；F1 HiRoom/ETH3D/ScanNet++ +17.7/+3.5/+3.6，DTU CD −0.41，7Scenes F1 −1.0。部分交替保留早期逐图先验；不能说全交替逐列更差。a 对 d Auc3 +33.61/+7.4/+24.1/+12/+3.8，几何 +35.5/+22.0/−1.32mm/−1.8/+4.5（F1 为点）。作者“consistent drops”在 7Scenes F1 49.4>47.6 有反例。a 对 e Auc3 +28/+4.8/−6.7/+2.9/+4.1，几何 +31/−2.2/+0.53mm/+7.3/+3.4；教师非全指标改善，ETH3D F1 和 DTU 两指标是反例。图8 边缘更锐利，不能替代量化。g 对 f HiRoom/ETH3D/ScanNet++ F1 +8/+7.7/+2.9，DTU CD −1.51mm，7Scenes F1 −12.4。即 p.18 “Across metrics ... consistently outperform” 与表内 7Scenes 明确冲突。

#### 证据边界

【证据边界】f/g 同用 GT pose fusion 因此只比较输入条件对几何的作用；不能将 f/g 与 a-e 的差异单归因于 conditioning。对照未给出重复试验方差或所有计算量细节，无法判断统计显著性，也不能由此证明完整预训练是唯一原因。

<!-- PTH:END TABLE-07 -->

<!-- PTH:BEGIN TABLE-08 -->
### Table 8（PDF p.18）

#### 题注与协议

**Caption 原文：** Comparison of Models with Parameters and Running Speed. The maximum number of images was tested on an 80 GB A100 GPU. If we store some intermediate tokens in CPU memory, we could process many more images. The running speed was measured on an A100 GPU with a scene of 32 images, and we report the average speed per image. The image resolution is 504 × 336.

**Caption 中文：** 比较模型参数与运行速度。最大图像数在 80GB A100 上测试；将部分中间 token 存入 CPU 内存还可处理更多。速度在 A100 的 32 图像场景测量，报告每图像平均速度，分辨率 504×336。

【原文事实】§7.2.3。FPS 为每秒图像数，参数为 B（十亿）；最大图像数量是测试区间，不是理论上限。

#### 表格数据

| Model | 最大图数 | Backbone(B) | DualDPT(B) | CameraHead(B) | FPS |
| --- | --- | --- | --- | --- | --- |
| VGGT(Reference) | 400–500 | 0.91 | 0.064 | 0.22 | 34.1 |
| DA3-Giant | 900–1000 | 1.130 | 0.050 | 0.018 | 37.6 |
| DA3-Large | 1500–1600 | 0.300 | 0.047 | 0.008 | 78.37 |
| DA3-Base | 2100–2200 | 0.086 | 0.015 | 0.004 | 126.5 |
| DA3-Small | 4000–4100 | 0.022 | 0.003 | 0.001 | 160.5 |

#### 实验分析

【独立评价】Giant 比 VGGT FPS 增 10.3%，最大图数区间中点约 950/450=2.11 倍。Large/Base/Small 速度分别 2.30/3.71/4.71 倍 VGGT；32 张场景若按 FPS 换算约 0.41/0.25/0.20 秒（Giant 0.85 秒、VGGT 0.94 秒），仅为平均吞吐的反算。三个组件相加 VGGT 1.194B、Giant 1.198B、Large 0.355B、Base 0.105B、Small 0.026B。Giant 组件和不同于 Table 2/3 的 1.10B，不能擅自更正；Large 0.30B 是 backbone 口径，表2 总量 0.36B。Giant camera head 0.018B 对 VGGT 0.22B 更轻，但不是完全相同结构的逐层参数对照。

#### 证据边界

【证据边界】32 图输入的吞吐不能外推到千图极限，也不等于实际端到端重建时间；注意力随 token 数增长、CPU offload 增加传输成本。未给出峰值显存、精度格式或每场景耗时分布。最大图数不证明在该规模下仍有相同几何准确率。

<!-- PTH:END TABLE-08 -->

<!-- PTH:BEGIN TABLE-09 -->
### Table 9（PDF p.19）

#### 题注与协议

**Caption 原文：** Ablations for teacher model. Training with V3 datasets and multi-resolution strategy yields the best performance. Depth-based geometry achieves the best AbsRel and SqRel. The full teacher-loss outperforms other variants. (AbsRel: ↓, SqRel: ↓, δ1: ↑). The results are averaged over KITTI, NYU, ETH3D, SUN-RGBD and DIODE.

**Caption 中文：** 教师模型消融。V3 数据及多分辨率策略取得最好性能；深度几何的 AbsRel、SqRel 最优；完整教师损失优于其他变体。指标按 KITTI、NYU、ETH3D、SUN-RGBD、DIODE 平均。

【原文事实】§7.3.1：ViT-L、batch64、DA2 基准协议。δ1 为比例尺度，AbsRel=mean(|D−G|/G)，SqRel=mean((D−G)^2/G)，后者按原文为平方差再以真值归一。

#### 表格数据

| 组 | 配置 | δ1↑ | AbsRel↓ | SqRel↓ |
| --- | --- | --- | --- | --- |
| Data | V2 | 0.919 | 0.087 | 0.596 |
| Data | V3 | 0.929 | 0.079 | 0.508 |
| Data | V3+mr. | 0.938 | 0.072 | 0.452 |
| Geometry | Disparity | 0.919 | 0.095 | 1.033 |
| Geometry | Pointmap | 0.912 | 0.096 | 0.693 |
| Geometry | Depth | 0.918 | 0.089 | 0.637 |
| Loss | MAE-Loss | 0.918 | 0.089 | 0.637 |
| Loss | w/o_Dist.Nor. | 0.918 | 0.087 | 0.600 |
| Loss | Full_loss | 0.919 | 0.087 | 0.596 |

#### 实验分析

【独立评价】V2→V3 的三项变化 +0.010/−0.008/−0.088；再加入多分辨率 +0.009/−0.007/−0.056。整体 δ1 +1.9 点，AbsRel −17.2%，SqRel −24.2%。Depth vs disparity δ1 −0.001，却 AbsRel −0.006、SqRel −0.396（38.3%）；Depth vs pointmap 三项 +0.006/−0.007/−0.056。深度参数化直接服务反投影和尺度平移对齐；视差反演非线性会让远距离误差被放大。完整 loss 相对 MAE +0.001/−0.002/−0.041；相对无距离加权法向项 +0.001/持平/−0.004。完整 loss 的提升较小，舍入后 AbsRel 相等，作者的 best 不意味着每个数严格优于所有行。

#### 证据边界

【证据边界】三组是各自消融，不宜跨组自由组合，不能把 V3+mr 与 Full loss 的差作为损失的单因素因果；数据规模与组成同时变化。均值掩盖单数据集差异，缺少重复试验方差。SqRel 与尺度/单位相关，不应和 δ1 混作统一量纲。

<!-- PTH:END TABLE-09 -->

<!-- PTH:BEGIN TABLE-10 -->
### Table 10（PDF p.20）

#### 题注与协议

**Caption 原文：** Monocular student depth comparisons. δ1 ↑

**Caption 中文：** 单目学生深度比较，δ1 越大越好。

【原文事实】§7.3.2。ViT-L 单目 student，区别于 Table 4 的多任务主模型和 teacher。

#### 表格数据

| Method | KITTI | NYU | SINTEL | ETH3D | DIODE |
| --- | --- | --- | --- | --- | --- |
| DA2 | 94.6 | 97.9 | 77.2 | 86.5 | 95.2 |
| mono-student | 97.1 | 98.0 | 82.3 | 98.8 | 96.5 |

#### 实验分析

【独立评价】学生对 DA2 +2.5/+0.1/+5.1/+12.3/+1.3 点，五域全正。ETH3D 相对增益 12.3/86.5=14.2%，符合作者 over 10% 描述；SINTEL 相对增益 5.1/77.2=6.6%，原文 +5.1% 实为数值百分点。与 Table 4 teacher 比，student 在 NYU +0.1、SINTEL +0.9，但 KITTI −0.1、ETH3D −1.0、DIODE −0.1。蒸馏引入大量无标签真实图像及教师伪标签，学生可以在某些域超过教师，非逐像素复制。

#### 证据边界

【证据边界】并未单独控制教师变化与 V3 数据规模；不能将性能提升全部归因于某一个改动。单目 δ1 无法验证多视角相机和三维一致性；Table 10 不是 metric 深度实验。

<!-- PTH:END TABLE-10 -->

<!-- PTH:BEGIN TABLE-11 -->
### Table 11（PDF p.21）

#### 题注与协议

**Caption 原文：** Comparison with state-of-the-arts on metric depth estimation. The best and second best are highlighted. Bottom rows show ablation results with and without teacher supervision. Note that the ablation setting is slightly different from the final model on training resolution, which leads to minor differences in performance.

**Caption 中文：** 度量深度与先进方法比较，突出最优及第二优。底部是有/无教师监督消融；消融训练分辨率与最终模型略不同，导致性能略有差异。

【原文事实】§4/metric 变体、§7.4。NYUv2、KITTI、ETH3D、SUN-RGBD、DIODE（indoor）；δ1 越高、AbsRel 越低。metric 目标是绝对深度，与相对深度指标协议不能混用。

#### 表格数据

| Method | NYUv2 δ1 | AbsRel | KITTI δ1 | AbsRel | ETH3D δ1 | AbsRel | SUN-RGBD δ1 | AbsRel | DIODE δ1 | AbsRel |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DepthPro | 0.932 | 0.093 | 0.843 | 0.121 | 0.386 | 0.349 | 0.950 | 0.126 | 0.734 | 0.173 |
| Metric3D_v2 | 0.971 | 0.067 | 0.976 | 0.051 | 0.830 | 0.138 | 0.954 | 0.132 | 0.018 | 0.154 |
| UniDepthv1 | 0.980 | 0.061 | 0.978 | 0.051 | 0.234 | 0.464 | 0.971 | 0.113 | 0.570 | 0.266 |
| UniDepthv2 | 0.968 | 0.064 | 0.968 | 0.076 | 0.863 | 0.152 | 0.977 | 0.111 | 0.856 | 0.123 |
| DA3-metric | 0.963 | 0.070 | 0.953 | 0.086 | 0.917 | 0.104 | 0.973 | 0.105 | 0.838 | 0.128 |
| w/_teacher | 0.966 | 0.073 | 0.947 | 0.086 | 0.906 | 0.105 | 0.973 | 0.104 | 0.824 | 0.132 |
| w/o_teacher | 0.969 | 0.066 | 0.965 | 0.067 | 0.907 | 0.105 | 0.975 | 0.099 | 0.816 | 0.134 |

#### 实验分析

【独立评价】ETH3D δ1 比 UniDepthv2 +0.054（5.4 点），AbsRel 比最佳外部 Metric3D v2 −0.034（24.6%）；SUN-RGBD AbsRel 0.105 最优（UniDepthv2 0.111），δ1 0.973 第二。DIODE 两项第二，NYUv2/KITTI 两项不及最佳。Metric3D v2 DIODE δ1 0.018 与 AbsRel 0.154 组合异常，照录原表，不能擅改。教师消融 w/−w/o：NYUv2 δ1 −0.003、AbsRel +0.007；KITTI −0.018、+0.019；ETH3D −0.001、0；SUN −0.002、+0.005；DIODE +0.008、−0.002。故仅 DIODE 两项改善，其他存在持平或下降。图10 展示更锐利结构，细节指标与绝对数值误差有不同关注点。

#### 证据边界

【证据边界】不能拿最终 DA3-metric 行与 w/o teacher 作严格单变量消融，原题注明确训练分辨率不同。教师对局部细节有补充作用，但边界清晰不自动保证绝对深度更准；没有边缘专门指标量化图10的改善。

<!-- PTH:END TABLE-11 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
### 图表证据链：从表示、监督到应用

#### 表示与架构

Figure 2 给出 token 到 depth/ray/camera 的整体信息流；Figure 3 说明 DPT 重组共享和独立融合；§3.1 的反投影及相机解算给出输出到世界点的数学联系。Table 6 在同一 ViT-L、10 views、batch128、120k steps、无 camera condition token 的设置下考察 target 组合，回答“什么输出最有用”；Table 7 的 a-c 对比 backbone 和注意力调度，d 检验共享重组。需要区分表示的数学充分性与模型学习效果：理想 ray 和 D 唯一确定对应像素点，不代表训练一定恢复真实几何；Table 6/7 是经验支持。

#### 教师与数据

Figure 4 是真实数据的监督问题：COLMAP 或 LiDAR 的深度不完整/含噪。Table 1 是主模型训练/测试来源，不是教师训练清单。§4.1 与 Table 9 给出合成教师的 data/representation/loss 消融；附录图12–14说明合成 GT 也会坏，需要筛除。§4.2 Eq.(8) 给出 teacher→metric 深度的对齐，Figure 8 比较主模型有无 teacher 的细节，Table 7(e) 则显示教师改善有条件（ETH3D F1、DTU 反例）。Table 10 说明单目学生蒸馏后的五域性能；Figure 10 和 Table 11 分别给出 metric 模型的边缘及标准数值，二者存在取舍。

#### 主实验

Figure 1 是统一模型及多任务能力的概览，不是独立证据；Figure 5/6/7 分别补充轨迹、点云与深度定性结果，Table 2/3 对应位姿与融合几何量化。Figure 9 用少量非标准场景显示不同图数的适应性，但没有真值指标；Table 4 为单目评估。Table 8 补充特定设备、分辨率下的速度和容量，不是精度证明。Figure 11 对应 Table 5 的已知 pose NVS，对照同一训练/测试协议；§5.2 的 pose-adaptive 版本不能以这张表当作未知 pose 的实验。综合证据由公式、定量受控试验、定性例图三部分组成，不能以一张漂亮渲染图替代真实三维验证。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
### 研究问题：任意视图视觉空间恢复

#### 问题形式

【原文事实】§1、§3.1（p.3–7）。作者将 monocular depth、SfM、MVS、视频等视为共享的视觉空间恢复问题。输入 I={I_i}_{i=1}^{N_v}，I_i∈R^{H×W×3}；N_v=1 为单目，N_v>1 为多视图/视频，可选已知相机参数 (K_i,R_i,t_i)。F_θ 预测每图像深度 D_i（H×W）与 ray map M_i（H×W×6），以及可选 9 维相机向量 (t,q,f)；世界点图由逐像素组合得到。研究任务不是仅拟合每张 RGB 对应的相对深度，而是在共享坐标下恢复可融合的观测几何和相机。核心科学问题是：能否减少主目标和专用结构，在视图数量/相机条件不同的情况下保持强几何能力？

#### 几何约束

【背景知识】深度图是相机坐标下沿 z 的距离；同一像素沿射线可以有无限个候选三维点，D 选择位置。射线方向/原点描述相机投影，单独 depth 无法知道跨相机坐标关系。P_i(u,v)=t_i+D_i(u,v)R_iK_i^{-1}p，将像素和世界坐标联系。相同场景点应在可见的多个视图的预测中落入同一世界位置。若将所有世界点、相机平移和深度同时缩放 λ>0，并保持内参/旋转不变，则透视投影相同；若改变全局旋转/原点，也不改变图像观测。在无 metric reference 时存在全局相似变换自由度。DA3 的训练归一化与 benchmark 对齐利用这种性质，而不是证明尺度歧义消失。

#### 任务界限

【证据边界】单目有不可观测深度歧义，网络只能凭训练先验推断；多视图亦受遮挡、纹理、视差和动态影响。DA3 输出每图像像素对应表面位置，不是完整场景的显式拓扑/隐面重建。视频作为视图集合输入不等于运动场估计或 SLAM 的持续状态更新。metric 变体另做单目绝对深度，不能从主模型相对几何直接推断绝对尺度。NVS 是 §5 的附加应用，需要 Gaussian 属性和目标相机；与几何恢复目标不同。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
### 动机：三种复杂性来自何处

#### 表示冗余

【原文事实】§1（p.3–4）、§3.1（p.6–7）。传统任务拆分与近期统一模型常用多目标。作者质疑 depth、point map、pose 是否都需要独立主监督。一个像素的点图由深度、相机反投影确定；若分别预测三个量，有限容量/噪声下可能各自最接近标签但互不满足几何约束。例如预测点 P 与由 D 和 K,R,t 重建的 P' 不同，后续需要选择哪一个。Depth-ray 把相机信息分布到每个像素，并使几何位置通过同一组合计算；Table 6 对比而非证明所有冗余目标有害。直接回归 R 还需满足 R^TR=I、det R=1；ray 不以矩阵元素为直接主目标，但稠密预测仍可能不满足单一合法相机。

#### 架构与预训练

【原文事实】复杂双骨干/多阶段模型不能完整复用单图预训练。DA3 采用标准 DINOv2 ViT 权重，并通过 reshape 让若干层跨视图。早期层保留单图图像统计，后期交换对应与空间线索。相同注意力块的参数与序列长度无关，故无需新建同规模专用 transformer。Table 7(b) 同规模 VGGT style 对照明显落后；作者归因于三分之二块未预训练（原文为 two-thirds untrained blocks）。【独立评价】完整预训练、深度/宽度和注意力模式是共同变化，不能将性能差唯一归因于预训练。

#### 监督质量

【原文事实】Figure 4、§4、附录A：真实 COLMAP/LiDAR 深度不完整或细节缺失，直接监督会鼓励学生复现噪声。合成教师具有稠密标签，但合成资源也有背景无效、错位、裁切等（Fig12–14）。教师学单目相对深度，使用原深度的可靠像素鲁棒拟合尺度/平移，尝试同时利用局部先验与实际几何锚点。表9 支持扩大数据集和多分辨率，表7/11 表明教师的标准指标并非普遍提升。研究动机是监督异质性，不是单纯“合成数据一定比真实数据好”。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
### 相关工作：三条技术路线

#### 传统到前馈

【原文事实】§2 p.4–5。传统 SfM 通过特征检测/匹配、鲁棒相对位姿、增量/全局求解与 bundle adjustment 恢复稀疏结构；MVS 在相机参数下估计深度并融合。纹理不足、反光、大基线会使对应不可靠；学习方法逐步替换检测器、描述子、优化层、cost volume 正则化。DUSt3R 以 transformer 回归成对 point map，预测局部几何后可全局对齐；§7.1 描述 Fast3R 扩展到大量图，MapAnything 支持相机条件，Pi3 采用置换等变设计，VGGT 联合 camera/depth/point。DA3 与 VGGT 类任务相近，但强调 ray 表示、完整单 ViT 预训练和简化多头；作者说与 Pi3 正交，不代表已验证二者结合。

#### 单目路线

【原文事实】单目早期在室内/驾驶等单域训练，泛化差；现代 generalist 用多数据、ViT/DiT 与 affine-invariant normalization。DA2 教师/学生是 DA3 的直接背景；DA3 teacher 从 scale–shift invariant disparity 改为 depth，并采用 exponential depth 及法向监督。背景：disparity∝1/Z，近处梯度敏感，远处小视差误差可能转成大深度误差。DA3 主模型首先服务跨视图几何，因此与 DA2 的单目相对深度任务不同；Table4/10 分别比较主模型和单目 student，不能混用。

#### 新视角路线

【原文事实】NVS 从逐场景 NeRF/3DGS 优化转向前馈 image-to-3D。原文以 pixelSplat 的 epipolar attention、MVSplat 的 cost volume 和 depth priors 为例，指出几何基础模型的预训练可用于 3DGS。§5 的方法新增 GS-DPT；§7.1 对比专用网络和 Fast3R/MV-DUSt3R/VGGT 等 backbone。区别在“几何基础模型 + 简单 Gaussian head”，不是提出新的 splatting 成像公式。已知和未知相机的分支分别见 §5.1/5.2。【证据边界】以上为 DA3 对引用工作的归纳，未独立查阅其他论文全文，不能作为其最新实现或历史优先权认定。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-FORMULATION -->
### 方法一：Depth-Ray 表示及恢复链

#### 输入与输出

【原文事实】p.6–7 §3.1，Fig2。输入 N_v 张相同 H,W 的图像；可选内外参。每图 D∈R^{H×W}，M∈R^{H×W×6}，前三通道 t、后三 d；显式 camera vector v∈R^9，t∈R^3，q∈R^4，f∈R^2（FOV）。将 p=(u,v,1)^T 作为齐次像素坐标，K^{-1}p 得到相机归一化坐标；R 为从相机到世界的旋转、t 为相机中心。原文将 P 写为四维齐次向量，但右侧为三维；实际等式作用于前三个欧氏坐标，最后一维补 1。

#### 几何恢复

【原文事实】d=RK^{-1}p 不作单位归一化，P=t+D(u,v)d。这使 depth 与 ray 按像素广播相乘即可形成 [N_v,H,W,3] 点图，不需单独预测 point map。理想针孔相机每个像素的 origin 相同、方向由 K,R 确定；深度决定在该射线上哪一个点。若多个视角中对同一点预测一致，融合能保持真实几何；否则同一表面会产生重复层或模糊点。若仅点图回归，不显式约束所有点来自一组相机模型，则单一合法相机的恢复较困难。作者引入轻量 camera head D_C，处理每图一个 camera token 预测九维参数，免去每次稠密射线 DLT。它是输出 head，不等同于可选的相机输入 encoder。

#### 算法边界

【原文事实】Eq.(1) 对预测 origins 平均估计中心；Eq.(2) 用 canonical ray 与 predicted ray 的叉积求 H，再 RQ 分解。这里存在符号方向疑点：前文 d=RK^{-1}p，后文写 H=KR，若同一 K,R 与世界方向则不是同一个映射。应以原文为记录，数学单元另给一致约定的推导，不能直接把 DLT 写成已证实实现。对齐与尺度：已知相机作为条件并不等价于硬编码输出，网络仍预测；未知 pose 输出存在全局尺度/坐标系自由度，benchmark 对齐处理。射线不能凭空恢复遮挡面；预测 origin、direction 不一致时直接点图和 camera head 解出的点图可能不同。

<!-- PTH:END METHOD-FORMULATION -->

<!-- PTH:BEGIN METHOD-BACKBONE -->
### 方法二：单 Transformer 的跨视图信息交换

#### token流

【原文事实】§3.2 p.7、Fig2。预训练 ViT 有 L 个 blocks，前 L_s 个逐图 self-attention，后 L_g 个交替 within-view/cross-view，L_s:L_g=2:1。每图 patch embedding 与一个 camera token 拼接。以 B=批大小、N=视图数、T=每图 token 数（含 camera token）、C=隐藏维为辅助记号，逻辑张量 [B,N,T,C]。逐图 attention 将其重排为 [BN,T,C]；跨图 attention 将其重排为 [B,NT,C]，再还原 [B,N,T,C]。这不是在特征通道拼接相机数，也不是增加第二套 transformer 权重。原文未给出所有 variant 的 T、C、各层精确索引或位置编码细节，不从通用 DINOv2 假定。

#### 注意力机制

【背景知识/补充推导】单头注意力令 Q=XW_Q,K=XW_K,V=XW_V，A=softmax(QK^T/√d_k)，Y=AV。逐图时每个 query 只访问同图 T 个 key；跨图时能访问所有 NT 个 key，使对应关系、遮挡线索、相机运动信息在 token 层传播。忽略投影、MLP 等成本，逐图注意力矩阵元素约 BN T²，跨图约 B N²T²（两者比 N）。因此把昂贵全局交互限制在后部部分层，既减少成本又保留前部单图特征分布；不是全局 attention 的稀疏匹配算法。N=1 时 reshape 后两个注意力域都只有 T 个 token，因此该模块自然退化为同一单图计算，仍可执行相同权重。

#### 设计与证据

【原文事实】Table7(a–c) 在相近容量下比较默认部分交替、VGGT style 和 Full Alt.：a 对 c 除 7Scenes F1 外领先；作者归因于预训练及调度。Table8 给出速度与数量上限。输入适应来自 token 数/序列重排，而非对任意输入保证常数计算。单图预测主要利用学习先验，多视图可利用对应约束；注意力学到的联系不等于显式极线约束，且可能在重复纹理、低视差或运动中错误关联。

<!-- PTH:END METHOD-BACKBONE -->

<!-- PTH:BEGIN METHOD-HEADS -->
### 方法三：相机条件、Dual-DPT 与轻量相机头

#### 相机条件

【原文事实】§3.2 p.7：有相机时从 K,R,t 转为 FOV f、四元数 q、平移 t，由轻量 MLP E_C 得到 c_i=E_C(f_i,q_i,t_i)；无相机则使用共享可学习 c_l。与 patch token 一起进入所有层。条件注入是在特征层让图像和其他相机 token 能交换信息，而不是直接用 GT 位姿替换输出；同一网络可接受两种输入。训练随机激活 pose conditioning 概率0.2 (§3.4)。四元数有单位范数约束且 q 与 −q 等价；论文未详述归一化或 FOV 转换细节。

#### Dual-DPT

【原文事实】Dual-DPT（Fig3）接收 backbone 多层视觉 token；各层先共享 reassembly，将序列恢复为空间特征并做不同尺度的重组；之后分别进入 depth 与 ray 的 fusion layers，再各自输出 map。与两个完全独立 DPT 相比，重组阶段两个目标基于同一空间表示，避免重复中间特征并鼓励坐标一致；独立融合保留 depth 的局部边界和 ray 的相机几何各自需求。背景：DPT 的多尺度融合通常将较粗语义特征与较细局部特征逐步上采样结合；具体卷积通道/操作未在本文给出，不能填写假定实现。共享参数受到两支损失梯度共同更新，最终参数仍可不同。

#### 训练与推理

【原文事实】相机预测 head 只读取每图 camera token，回归 f,q,t；理论上可用稠密射线解算相机，实际为成本选择显式 head。训练总损失包含 ray、depth、组合 point 和可选 camera 监督（p.8），并非“只学习两个 loss”。Table6 最终 depth+ray+cam 虽有取舍仍采用；Table7(d) 移除 Dual-DPT 除 7Scenes F1 外整体退化；f/g 在 GT fusion 下多域收益但 7Scenes 降 12.4 点。共享重组是否必然保证两个输出满足物理约束没有证明，推理的具体 head 优先级也不由表格确定。

<!-- PTH:END METHOD-HEADS -->

<!-- PTH:BEGIN METHOD-TRAINING -->
### 方法四：主模型训练、归一化与监督调度

#### 监督与归一化

【原文事实】§3.3 p.8。所有 GT 以有效反投影点范数均值为公共尺度，Z=(1/M)Σ||P||₂（辅助记号，勿与原文 Z_Ω 的损失归一化混淆）。当世界位置和 D 同除 Z 时，为保持 P=t+Dd，平移亦应除 Z、旋转/内参/射线方向不变；这是几何一致性的补充解释，论文未逐项列出归一化实现。总目标 L=L_D+L_M+L_P+βL_C+αL_grad，α=β=1。深度置信度 loss 由有效掩膜、残差加权与 −λ_c log confidence 组成；ray 监督相机射线，point 项在 D⊙d+t 的组合上约束三维，camera 监督显式 9 维相机，gradient 保留边缘。教师标签用于深度，并需与真实深度对齐；不等于用教师替换相机标签。详细符号及 P 中 d/t 歧义留在数学单元。

#### 数据与调度

【原文事实】p.8 §3.4/Table1：128 H100、200k steps、8k warm-up、peak LR 2e−4；base 504×504，随机 504×504/378/336/280，336×504，896/756/672×504（按原文给定分辨率）。504×504 时 view 数均匀采 [2,18]；动态 batch 保持每步 token 数近似固定。120k 从 GT depth 转为 teacher labels；pose conditioning 概率0.2；ScanNet++ 作者明确场景级训练/测试互斥。p.16 巨型模型约10天；消融32 H100约4天，ViT-L、max10 views；Table6 10 views、batch128、120k。训练阶段从 GT 到细节更密集的伪标签，保留早期几何锚点的意义是合理推断，原文未分离调度消融。

#### 限制

【证据边界】loss 的 α/β 相同不意味着不同物理量梯度同量级；confidence 的范围、λ_c 数值、不同 source 的采样权重未充分说明。归一化可以避免大场景大残差主导，但绝对尺度不再是主目标。点范数依赖坐标原点；若原点或有效mask变化，Z也变化；若全点零或无有效数据则需保护，论文未给出实现。teacher 的局部预测有系统偏差，鲁棒对齐不能消除非仿射畸变。

<!-- PTH:END METHOD-TRAINING -->

<!-- PTH:BEGIN METHOD-TEACHER -->
### 方法五：合成数据教师与鲁棒伪标签

#### 合成数据

【原文事实】§4.1 p.8–10，teacher 用 DINOv2+单 DPT，无多视图专用模块；仅合成数据训练。V3 扩展 Hypersim、TartanAir、IRS、vKITTI2、BlendedMVS、SPRING、MVSSynth、UnrealStereo4K、GTA-SfM、TauAgent、KenBurns、MatrixCity、EDEN、ReplicaGSO、UrbanSyn、PointOdyssey、Structured3D、Objaverse、Trellis、OmniObject。附录A 筛除错误背景/错位/无纹理/裁切等；Table9 V2→V3→V3+mr 指标逐步改善，不能只用样本数解释。与主模型的 Table1 训练集不同。

#### 深度表示与损失

【原文事实】由 DA2 的尺度平移不变 disparity 改为尺度平移不变 depth；为缓解线性深度近处不敏感，采用 exponential depth，原文未给出具体指数函数、符号或解码公式，不能指定 e^D 或 e^{-D} 为作者实现。监督由 gradient、ROE alignment/global-local loss、distance-weighted normal、sky-mask、object-mask 构成（Eq4–7）。每中心像素取四个邻点，构造未归一化 normal，较大模长在权重中被削弱；平均方向和各 normal 的角误差共同监督局部表面。背景知识：深度跳变导致邻接三维向量跨越两面，法向可能不代表本表面；用距离相关权重可减弱它，但模长还取决于夹角。mask 对天空或 object-only 无 GT 的背景防止其污染深度。ROE/global-local 的具体公式由原引用工作提供，本文未展开。

#### 教学对齐

【原文事实】§4.2 Eq8：对每图 teacher 预测 ~D 和稀疏有效测量 D（mask m），RANSAC 拟合 s>0 与 t，使 s~D+t 在可靠点贴近 D；内点阈值为残差中位数的 mean absolute deviation；得到密集 D_{T→M} 供多视角 student。通过一个共同 scale/shift 将相对深度嵌入原测量空间而不需要教师知道相机平移。推断例：教师在远处缺测区域填入连续面，但拟合只在有效测量上校准，若远区深度曲线错误或有域偏移，则填补结果未被实测约束。表7、图8 支持细节收益，同时 DTU/ETH3D 的反例表明 teacher 不是“真值升级”。

<!-- PTH:END METHOD-TEACHER -->

<!-- PTH:BEGIN METHOD-MONO-METRIC -->
### 方法六：单目学生与度量深度变体

#### 单目学生

【原文事实】§4.3 p.10：按 DA2 的 teacher-student framework，使用无标签图像、教师生成伪标签，student 预测相对 depth 而非 disparity，使用与 teacher 相同的 loss。无需稀疏 metric GT 才能训练该变体；由于学习相对深度，不能直接获得米制距离。Table10 以 ViT-L 比较，五域优于 DA2；Table4 teacher 和 DA3 是不同模型。训练中用真实无标签图像能帮助 student 吸收真实域的外观信息，属于机制解释而非单因素证明。

#### metric学生

【原文事实】§4.4 p.10–11，metric 模型以 canonical focal f^c=300 对深度标签乘 f^c/f；教师预测与原 metric 深度做 scale/shift 对齐。背景推导：透视关系 x−c_x=fX/Z，同一物体像素尺寸可由 f 或 Z 改变；用 canonical 焦距把目标空间标准化，预测 canonical depth D_c 后需乘 f/f^c 恢复原尺度（假设使用原文定义）。原文称14个数据集但列出 Taskonomy、DIML Outdoor、DDAD、Argoverse、Lyft、PandaSet、Waymo、ScanNet++、ARKitScenes、Map-free、DSEC、Driving Stereo、Cityscapes 共13个名称，未补猜第14个。stereo 训练标签用 FoundationStereo。base504、多 aspect ratios（1:1,1:2,16:9,9:16,3:4,1:1.5,1.5:1,1:1.8），AdamW encoder/decoder LR5e−6/5e−5，5%概率旋转90/270°，batch64、160K；20%用原GT，余用对齐 teacher label；目标 depth+grad+sky-mask 加权。

#### 比较与边界

【证据边界】Table11 ETH3D δ1 0.917、AbsRel 0.104，在所列方法中最优；NYUv2、KITTI 有强基线更优。w/ 与 w/o teacher 不是所有域标准指标改善，Figure10 展示局部细节。最终模型与消融训练分辨率不同。相对深度的 scale–shift 不变性与 metric 的物理标尺目标不同，focal normalization 需要可用焦距、训练分布及模型先验；并未从单张 RGB 的投影数学上唯一确定真实距离。

<!-- PTH:END METHOD-MONO-METRIC -->

<!-- PTH:BEGIN METHOD-GS -->
### 方法七：从几何基础模型到前馈 Gaussian

#### pose-conditioned

【原文事实】§5.1 p.11。Pose-conditioned 版本在 DA3 backbone 增 GS-DPT 进行 fine-tuning。对每图视觉 token 输出像素对齐 Gaussian 属性：opacity σ（confidence head），rotation quaternion q、scale s∈R³、RGB c∈R³（main GS-DPT）。原文排版将 σ 与 q 一并写作 ∈H，opacity 语义为标量，不能据此认定四元数 opacity。中心由估计深度按 K,R,t 反投影到世界点 P∈R³。Gaussian 的局部旋转/尺度描述空间分布，目标相机 rasterization 生成 RGB。训练目标 rendered target 的 MSE/LPIPS 与 observed view 的 scale-shift-invariant depth loss；后者避免纯颜色监督下 Gaussian 中心漂离合理表面。这是像素到 Gaussian 的前馈预测，不是每个测试场景从随机点优化。

#### pose-adaptive

【原文事实】§5.2 p.11–12：Pose-adaptive 版本兼容已知/未知 pose，局部 camera-space GS 与兼容两种条件的 backbone 为两项设计要求。已知 pose 用 Umeyama scaling 将预测深度/局部 Gaussian 对齐到给定相机，再反投影；未知则直接使用预测 pose。新增深度 offset，为渲染与表面几何之间留自由度；将 RGB 改为 SH 系数表示 view-dependent appearance。训练时冻结预训练 DA3，只调 GS-DPT；改变 resolution 和 context views，高分辨率少视图、低分辨率多视图以控制负担。背景：若 world similarity 为 x'=aRx+t，Gaussian mean 如此变换，covariance 变 a²RΣR^T，尺度也随 a；论文未详述具体 covariance 变换/offset 参数化/SH阶数。

#### 数据与边界

【原文事实】§5.3 DL3DV 10,015 training scenes，140 benchmark scenes 按作者陈述完全互斥。§7.5 已知相机 12-view、270×480 的对照（Table5、Fig11），所有模型加入 depth loss，8 A100 200K步，pixelSplat 100K。Table5 支持几何 backbone 提高 NVS，但无法证明未知相机、变化输入量的 pose-adaptive 版本具同样数值优势；预训练与模型能力也共同变化。高 PSNR 可由偏离真实表面的 primitive 拟合，offset/SH 特别强调渲染与几何的区分。

<!-- PTH:END METHOD-GS -->

<!-- PTH:BEGIN METHOD-BENCHMARK -->
### 方法八：几何与渲染的统一评估协议

#### pose协议

【原文事实】§6.1–6.3 p.12–13。位姿对每场景选所有可用图；超限固定种子随机100。每模型前馈生成一致 pose/depth，计算相对旋转角和平移方向角的准确率；各阈值取 RRA 与 RTA 较小的曲线积分，Auc3、Auc30。每对视图的相对运动不依赖世界坐标的统一平移/旋转，平移方向不含长度，因此不测绝对尺度。无基线或接近零相对平移时方向定义可能不稳定，原文未说明处理。数据集特定的帧下采样见 Table1 单元。

#### 几何协议

【原文事实】几何用同图集预测 pose+depth。evo/Umeyama 对随机 pose 子集拟合到 GT 坐标；RANSAC 以 translation error 低于整体 pose deviations 的 median 定义 inlier，选最大集合的 transformation。用其对齐预测几何再 TSDF fusion 得重建，和 GT 点集比较。背景：TSDF 将多个 depth 在 voxel 网格中对截断符号距离累积，能融合噪声，但 pose 错会造成同一物面不重合。F1 用阈值 d 的 precision、recall；DTU 用双向平均距离 CD(mm)；GT-pose fusion 使用已知 pose，支持条件的模型还能使用输入 pose。阈值/voxel：HiRoom .05/.007m；ETH3D .25（d未标单位）/.039m；7Scenes .05/.007m；ScanNet++ .05/.02m。DTU 使用 RMBG2.0 去背景、原文引用默认融合。HiRoom Table1 29 scenes、§6.3 30；记录差异。

#### render协议

【原文事实】NVS 用 DL3DV140/Tanks and Temples6/MegaDepth19（共165）。每场景通常300–400 views，每8张采1 target，剩余按 COLMAP pose 的旋转/平移距离做 farthest point sampling 取12 context。DL3DV官方benchmark，T&T训练场景排除Courthouse，MegaDepth5000–5018。使用给定 COLMAP pose，报告 PSNR、SSIM、LPIPS。该协议侧重已校准图像的渲染质量；不能用它推断相机估计误差传到 NVS 的鲁棒性，也不能认为 COLMAP pose 绝对无误。

<!-- PTH:END METHOD-BENCHMARK -->

<!-- PTH:BEGIN MATH-RAY -->
### 数学一：非单位射线与深度的反投影

#### 反投影公式

【原文公式】§3.1 p.6（未编号）：

$$\mathbf P=\mathbf R_i(D_i(u,v)\mathbf K_i^{-1}\mathbf p)+\mathbf t_i,\qquad \mathbf d=\mathbf R\mathbf K^{-1}\mathbf p,\qquad \mathbf P=\mathbf t+D(u,v)\mathbf d.$$

$\mathbf p=(u,v,1)^T$，$\mathbf K$ 为内参，$\mathbf R$ 为相机到世界旋转，$\mathbf t$ 为世界系相机中心；$D$ 是相机 z 深度。$\mathbf d\in\mathbb R^3$ 不归一化，$\mathbf r=(\mathbf t,\mathbf d)\in\mathbb R^6$。原文把 $\mathbf P$ 写作齐次 4 维，等式右侧为欧氏 3 维，以下对前三分量推导。

【补充推导】针孔投影 $\mathbf p\sim\mathbf K\mathbf X_c$。当 $X_{c,z}=D>0$ 且 $K$ 第三行为 $(0,0,1)$，齐次比例取 D，于是 $\mathbf X_c=D\mathbf K^{-1}\mathbf p$。变到世界系 $\mathbf X_w=R\mathbf X_c+t$；利用标量与矩阵乘法可交换即上式。

#### 补充推导

【补充推导】设 $K=\begin{bmatrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}$，则 $K^{-1}p=((u-c_x)/f_x,(v-c_y)/f_y,1)^T$。射线长度为 $\|P-t\|=D\|d\|$，除中心像素外一般不等于 D；若改用单位方向 $\bar d=d/\|d\|$，必须同步把深度改成沿射线距离 $\ell=D\|d\|$，否则反投影位置偏移。构造例（非论文实验）：R=I、t=(1,0,0)、归一化像素(0.6,0,1)、D=5m，则 d=(0.6,0,1)，P=(4,0,5)m；射线长度 $5\sqrt{1.36}\approx5.83$m。若错误用 $5\bar d$，z 坐标约4.29m。可见非单位射线不是任意放大，而是与 z 深度的定义配套。

张量层面 D:[N,H,W,1] 广播乘 d:[N,H,W,3] 加 t:[N,H,W,3]，输出同像素排列的三维点。若全局相似变换 $P'=aQP+b$，$t'=aQt+b$，$D'=aD$，$d'=Qd$（a>0、Q旋转），则 $t'+D'd'=aQP+b$。投影保持一致，说明仅图像不能确定全局尺度和坐标规范。

#### 边界

【条件及算法作用】理想 pinhole、有效像素、统一坐标及一致单位是成立条件。预测 ray 每像素 origin 可有偏差，不保证同图所有 t 完全相同；预测 direction 也未必对应唯一 K,R。Eq.(1)–(2) 尝试恢复一个合法相机，或由显式 head 直接输出相机参数。若误差小，$\delta P\approx\delta t+d\delta D+D\delta d$；深度误差和方向误差对位置的影响不同，远处 D 大时方向误差被放大（忽略二阶项 $\delta D\delta d$）。因此 ray 估计的稳定性对远景很重要；公式只建立已见像素的表面，不证明隐藏几何恢复，也不等于相机和深度在噪声下必然可辨识。

<!-- PTH:END MATH-RAY -->

<!-- PTH:BEGIN MATH-CAMERA -->
### 数学二：从射线图恢复相机（Eq.1–2）

#### 公式1与2

【原文公式】p.6 Eq.(1)：

$$t_c=\frac1{HW}\sum_{h=1}^H\sum_{w=1}^W M(h,w,:3).$$

每像素 origin 为世界坐标的相机中心估计，取均值可减小零均值独立噪声。若有大量系统偏差/异常值，均值未必鲁棒。

Eq.(2)：

$$H^*=\arg\min_{\|H\|=1}\sum_{h,w}\|H p_{h,w}\times M(h,w,3:)\|.$$

原文构造 $K_I=I$、$d_I=p$，写 $d_{cam}=KRd_I$、$H=KR$，以 DLT 解算 homography，再 RQ 分解 upper triangular K 和 orthonormal R。叉积为零表达两个方向平行，不要求长度相等；全局 H 的范数约束避免零矩阵成为平凡解。

#### 补充推导

【补充推导及记号疑点】取一个预测方向 $m_i=(a_i,b_i,c_i)^T$，$[m_i]_\times$ 为叉积矩阵。无噪声时 $[m_i]_\times Hp_i=0$。展开 $Hp_i=(h_1^Tp_i,h_2^Tp_i,h_3^Tp_i)^T$，例如第一分量 $-c_i h_2^Tp_i+b_i h_3^Tp_i=0$；其余两式类似。每对应最多两条独立线性约束，将所有像素堆叠成 $A\,\mathrm{vec}(H)=0$。标准 DLT 最小化 $\|Ah\|_2^2$，约束 $\|h\|_2=1$，取 A 最小奇异值对应右奇异向量（等价最小特征值）。注意 Eq.(2) 印刷为逐点范数之和而非平方和，原文称 standard least squares；在有噪声时二者目标一般不严格等价，不能将补充 DLT 解释为精确求解原印刷目标。

坐标方向：按前文 $d_i=R_{cw}K^{-1}p_i$（cw 此处定义 camera→world），从 pixel 到 world-ray 的 H 应与 $R_{cw}K^{-1}$ 成比例。其逆映射 $H^{-1}\sim KR_{cw}^T$ 才自然具有上三角内参与正交旋转的 RQ 形式。若作者后半段采用反向映射/转置约定，$H=KR$ 可以有意义；但与前文同名 R,K 的方向没有解释一致。本文不擅改原文或声称源码实现如何，记录为公式记号疑点。实际恢复通常还要确定符号、令 $K_{33}=1$、正焦距、$\det R=+1$，并检查尺度。

#### 条件

【成立条件及推理】H 需满秩，像素和射线对应需覆盖足够非退化方向（同一直线上点不足以确定一般 homography），测量误差和异常方向会影响估计；旋转应正交、内参应合法。叉积损失依赖方向模长；归一化和加权细节未写明。Eq.(1) 对 origin 与 Eq.(2) 对 direction 分开求解，在真实针孔相机理想假设下有几何意义。作者最终采用轻量 camera head 直接从每图 token 回归 FOV/四元数/translation，减少稠密 DLT 计算；不能把 DLT 视为所有实验的必经推理过程，也不能把 head 的预测与 ray 的一致性视为硬约束。

<!-- PTH:END MATH-CAMERA -->

<!-- PTH:BEGIN MATH-DA3-LOSS -->
### 数学三：主模型的联合训练目标与置信度

#### 原文损失

【原文公式】p.8 §3.3：

$$\mathcal L=\mathcal L_D(\hat D,D)+\mathcal L_M(\hat R,M)+\mathcal L_P(\hat D\odot d+t,P)+\beta\mathcal L_C(\hat c,v)+\alpha\mathcal L_{grad}(\hat D,D).$$

$$\mathcal L_D(\hat D,D;D_c)=\frac1{Z_\Omega}\sum_{p\in\Omega}m_p\big(D_{c,p}|\hat D_p-D_p|-\lambda_c\log D_{c,p}\big).$$

Eq.(3)：

$$\mathcal L_{grad}(\hat D,D)=\|\nabla_x\hat D-\nabla_xD\|_1+\|\nabla_y\hat D-\nabla_yD\|_1.$$

$\hat D$ 预测深度，D 监督深度；$\hat R$ 在此为预测 ray map（非 rotation R），M 为 GT ray map；P 点图，v 相机九维标签，$\hat c$ 显式相机输出；$m_p$ 有效像素掩膜，$D_{c,p}$ 深度 confidence，$Z_\Omega$ loss 归一化因子（本段未给精确定义），$\lambda_c$ 权衡系数未给具体数值。作者称 L1 型，$\alpha=\beta=1$。

#### 补充推导

【补充推导】固定有效像素残差 $e=|\hat D-D|\ge0$，令 $c=D_c>0$，单点 loss $ce-\lambda\log c$。若 $\lambda>0,e>0$ 且 c 无上界，$\partial_c L=e-\lambda/c$，二阶导 $\lambda/c^2>0$，驻点 $c^*=\lambda/e$；残差越大最优权重越低，log 项阻止所有 confidence 归零。若实际 c 有限制需按边界求最优；e=0 且 c 无上界时该形式会趋向负无穷，故不能从公式断定具体参数化或置信度校准。对于预测深度，固定 c 时 L1 次梯度为 $c\,\mathrm{sgn}(\hat D-D)$（乘 mask 和归一化）；较可靠标签提供更强训练信号。

梯度项用有限差分。例如 1D 连续两个像素 $D=(2,8)$，$\hat D=(4,6)$：GT 跳变6、预测跳变2，梯度误差4；同样的偏置 $\hat D=(3,9)$ 梯度误差0，绝对误差由深度项校正。它约束 GT 的斜率/边缘，不是简单惩罚所有高梯度的 TV 平滑。共同几何归一化 $s=(1/|\Omega_P|)\sum\|P\|_2$ 若 s>0，对 P,D,t 同除 s，方向 d 不变可保持 $P/s=t/s+(D/s)d$，帮助训练不同场景尺度。

#### 边界

【证据边界】原文 $L_P$ 写 $\hat D\odot d+t$，d/t 无帽子，未明确这里是从预测 ray 取出还是 GT 方向/原点。如果取预测 ray，链式法则给 $\partial P/\partial D=d$、$\partial P/\partial d=D I_3$、$\partial P/\partial t=I_3$，point loss 可以协同约束 ray/depth；若取 GT ray 则不向 ray head 反传。本文只给条件性推导，不擅自确认梯度路径。各 loss 的内部 reduction、confidence 参数域、相机四元数监督、梯度 mask 处理未详述；不能填入其他模型默认实现。教师监督切换只说明深度标签改变，不代表所有 GT 都被教师替代。

<!-- PTH:END MATH-DA3-LOSS -->

<!-- PTH:BEGIN MATH-TEACHER-LOSS -->
### 数学四：教师的距离加权法向和复合损失（Eq.4–7）

#### 公式4至7

【原文公式】p.10 §4.1。中心像素取四个邻点并计算未归一化法向 $n_i$，权重及均值：

$$w_i=\sum_{j=0}^{4}\|n_j\|-\|n_i\|.\tag{4}$$

$$n_m=\sum_{i=0}^{4}w_i\frac{n_i}{\|n_i\|}.\tag{5}$$

$$\mathcal L_N=\mathcal E(\hat n_m,n_m)+\sum_{i=0}^{4}\mathcal E(\hat n_i,n_i).\tag{6}$$

$$\mathcal L_T=\alpha\mathcal L_{grad}+\mathcal L_{gl}+\mathcal L_N+\mathcal L_{sky}+\mathcal L_{obj},\quad \alpha=0.5.\tag{7}$$

$\mathcal E$ 法向间角误差；$L_{gl}$ 为采用 ROE alignment 的 global–local loss（引用 [95]，本文无具体公式）；$L_{sky}$、$L_{obj}$ 为与 depth 对齐的 sky/object mask 的 MSE。后两者解决天空/纯物体图背景无深度标签的问题。

#### 补充推导

【补充推导】以中心三维点 P 与相邻三维点 A,B 构成三角面，未归一化法向 $n=(A-P)\times(B-P)$，$\|n\|=\|A-P\|\|B-P\||\sin\theta|$，等于三角形面积的两倍。跨越深度边界会使邻接向量很长，可能得到大模长的错误法向；Eq.(4) 令总模长 S 固定，对大模长项赋相对小的权重 $w_i=S-\|n_i\|$。构造例：若参与项模长 1,1,1,1,8，则 S=12，四个短法向权重各11，大法向权重4；若其方向离群，其影响低于未加权平均。这是权重机制的示例，并非作者提供的实际邻点索引。

Eq.(5) 先除模长只保留方向，再加权求和；未除权重和，因此 n_m 不一定单位长度，角度误差可在非零向量下使用 $\arccos((a^Tb)/(\|a\|\|b\|))$（背景标准定义，实际实现未给）。Eq.(6) 同时监督局部聚合法向和个体方向，避免仅平均方向正确而各邻域几何错误。Eq.(7) 与深度梯度共同维护局部平面/边缘，global-local 项负责对齐后的深度结构。

指数深度：原文说采用 exponential depth 提高近处敏感性，没有给出明确映射。一般指数变换 $g(D)=e^{kD}$ 的局部敏感度为 $|k|e^{kD}$；是否近处更敏感取决于 k 的符号或变量定义。不能把某个特定指数形式写成论文实现。

#### 边界

【证据边界】文字“四个邻点”，Eq.(4–6) 却从 0 到4 求和（5项）；是否包含中心或另外一项法向未说明，不擅改索引。法向模长不是纯距离，夹角/退化三角形也会影响；小模长不必然代表可靠法向。若 $\|n_i\|=0$ 或加权向量相互抵消，归一化/角误差需数值保护，本文未给细节。ROE 的完整实现只能通过引用工作进一步确认（本次未外查）。Table9 的 normal loss 消融改善有限，不能由公式推导出普遍精度保证。

<!-- PTH:END MATH-TEACHER-LOSS -->

<!-- PTH:BEGIN MATH-ALIGN -->
### 数学五：尺度平移对齐与度量尺度（Eq.8）

#### 公式8

【原文公式】§4.2 p.10 Eq.(8)：

$$(\hat s,\hat t)=\arg\min_{s>0,t}\sum_{p\in\Omega}m_p(s\tilde D_p+t-D_p)^2,\qquad D^{T\to M}=\hat s\tilde D+\hat t.$$

$\tilde D$ 教师相对深度，D 原稀疏测量，m 有效 mask，s 正尺度，t 标量 shift（不是相机平移向量）。RANSAC least squares 以残差中位数的平均绝对偏差为 inlier threshold。输出对齐后的 dense pseudo depth，供 DA3 depth 监督。正尺度保持教师深度顺序；shift 允许相对深度零点与 metric 不同。

#### 补充推导

【补充推导】固定 RANSAC 选出的有效内点集合 S（非空），写 x_i=teacher、y_i=GT，$J=\sum_{i\in S}(sx_i+t-y_i)^2$。对 t 求导为0 得 $t=\bar y-s\bar x$；代入并对 s 求导得

$$s_{OLS}=\frac{\sum(x_i-\bar x)(y_i-\bar y)}{\sum(x_i-\bar x)^2},\quad t_{OLS}=\bar y-s_{OLS}\bar x.$$

若分母非零且 s_OLS>0 为可行极小；若 s≤0，原约束开区间不能简单宣称有正最优解，实际需拒绝候选或限制到正值下界，原文未给。若内点 x 全相同，scale/shift 无法分离；至少两点且 x 不同方可确定仿射关系。RANSAC 通过抽样候选、按阈值统计 inliers 再拟合，减少离群测量影响。设残差 e_i，原文阈值按 mean absolute deviation from median，可写为 $\tau=(1/n)\sum|e_i-\mathrm{median}(e)|$，不要误写成 median absolute deviation。具体采样次数/退化处理未给。

构造例：teacher x=(1,2,3)，可信深度 y=(4,7,10)m，则 s=3,t=1，缺测 x=2.5 估计 8.5m；若缺测处教师真实相对结构错误，该值只是伪标签，不是测量。RANSAC 可排除一个离群 y，但无法校正由折射、域偏移等导致的系统非仿射形变。

#### metric变换

【原文事实/补充推导】§4.4 p.11 canonical focal transformation：$D_c=D\,f^c/f$，$f^c=300$。由透视 $x-c_x=fX/Z$ 可见焦距与深度共同影响尺度；按原文 convention，将输出 canonical depth 变回原相机深度需 $D=D_c f/f^c$（已知 f>0，且对应训练标签定义成立）。例如 D=10m、f=600、fc=300，canonical D_c=5m，逆变换恢复10m。此处是标签规范化，不是由图像单独证明 D 的绝对值。主 DA3 的共同点范数尺度、相对教师的 scale/shift 自由度、metric 模型的焦距规范是三种不同操作；不可混用。Table11 教师改善细节不保证绝对尺度数值。

<!-- PTH:END MATH-ALIGN -->

<!-- PTH:BEGIN MATH-GS-METRICS -->
### 数学六：Gaussian 坐标变换与评价指标

#### 渲染关系

【原文事实】§5.1–5.2，估计 depth 反投影确定 Gaussian 中心；局部相机属性含 opacity、rotation、scale、RGB/SH（pose-adaptive）。背景数学：令 $\Sigma_c=R_q\mathrm{diag}(s_x^2,s_y^2,s_z^2)R_q^T$。相机→世界若 $x_w=aR x_c+t$，则中心 $\mu_w=aR\mu_c+t$，协方差 $\Sigma_w=a^2R\Sigma_cR^T$；推导为 $E[(aR(x-\mu))(aR(x-\mu))^T]$。单位旋转 q、正尺度 s、a>0。pose-adaptive 的 offset 允许中心不严格落在原预测 surface；SH 使颜色随视线方向变化。背景 3DGS 渲染可写 $C=\sum_k T_k\alpha_k c_k$、$T_k=\prod_{j<k}(1-\alpha_j)$（按深度排序；不是本文新公式），同一 RGB 可由不同几何和 opacity 组合生成。因此 photometric loss 与 geometry loss 的作用互补。

#### 评估公式

【原文事实/背景定义】§6.2 p.12–13：pose AUC 来自阈值准确率 $a(\theta)=\min(RRA(\theta),RTA(\theta))$ 的积分。以常见归一化写法 $AUC_\tau=\tau^{-1}\int_0^\tau a(\theta)d\theta$（本文未明确归一化细节，表为百分数尺度）；RRA/RTA 分别为相对旋转、平移方向角误差的阈值比例。若平移向量 $t_1,t_2\ne0$，角度为 $\arccos(t_1^Tt_2/(\|t_1\|\|t_2\|))$，不含长度。几何令 GT G、预测 R，$d(R\to G)$ 是 R 各点到 G 最近距离均值；$CD=(d(R\to G)+d(G\to R))/2$，原文 DTU mm。$precision=|R|^{-1}\sum_i[\mathrm{dist}(R_i,G)<d]$，$recall=|G|^{-1}\sum_i[\mathrm{dist}(G_i,R)<d]$，$F1=2pr/(p+r)$。构造例 p=.9,r=.6 得 F1=.72；低完整度会限制高准确率的收益。p+r=0 需约定。距离阈值不同则 F1 不可直接跨域比。

深度（背景）：$\delta_1=\Pr(\max(\hat D/D,D/\hat D)<1.25)$，$AbsRel=\mathrm{mean}(|\hat D-D|/D)$，p.19 SqRel 定义 $\mathrm{mean}((\hat D-D)^2/D)$。有效 GT 需 D>0。

$PSNR=10\log_{10}(MAX^2/MSE)$（dB，MSE同动态范围）；SSIM 衡量局部结构、亮度、对比度，常见形式 $((2\mu_x\mu_y+C_1)(2\sigma_{xy}+C_2))/((\mu_x^2+\mu_y^2+C_1)(\sigma_x^2+\sigma_y^2+C_2))$；LPIPS 是深度特征感知距离，较低优，具体网络/归一化不由本论文详述。

#### 证据条件

【证据条件】几何使用对齐、TSDF与不同 threshold，pose AUC 衡量相对角度。相对深度 δ1 依赖基准对齐方式，不可与 metric δ1 直接当同条件比较。NVS 的 PSNR/SSIM/LPIPS 使用已知 COLMAP pose、270×480、12 context；这些评价不能单独验证预测相机、绝对几何或遮挡面的正确性。Table5 的优势可与表2/3作关联分析，但模型容量/预训练/训练协议差异未完全控制，相关不是严格因果证明。

<!-- PTH:END MATH-GS-METRICS -->

<!-- PTH:BEGIN EXP-POSE-GEOM -->
### 实验一：多视角位姿与三维重建

#### 设置

【原文事实】§6、§7.1（p.12–16），Table1–3、Fig5–7。五个基准：HiRoom 29 synthetic 场景（§6.3 说30）、ETH3D 11、DTU22、7Scenes7、ScanNet++20。每场景上限100图、固定种子采样，7Scenes/ScanNet++分别下采样11/5。位姿未知的模型估计 relative rotation/translation 角误差并计算 Auc3/30；重建按预测位姿与深度、RANSAC pose alignment、TSDF 得点云，对照 GT，除DTU用CD(mm)其余F1。已知位姿时 GT fusion，DA3/MapAnything 支持输入相机。比较 DUSt3R、Fast3R、MapAnything、Pi3、VGGT 与 DA3 四种规模；模型参数不是完全相同训练设置。§3.4 作者明确 ScanNet++ train/test 场景级互斥；无法独立验证清单。

#### 数值结果

【独立评价】Table2 的 DA3-Giant 对 VGGT 五域 Auc3 为 80.3/48.4/94.1/28.5/85.0 vs 49.1/26.3/79.2/23.9/62.6；逐域相对增益平均44.3%，两者均值之比39.5%，与在线摘要44.3%可对应一个口径，本地摘要35.7%无明示同一算法。Auc30 为95.9/91.2/99.4/86.8/98.1，仅DTU低于外部最佳99.8。7Scenes Large Auc3 29.2 超过 Giant。Table3 pose-free 相对 VGGT 四 F1 增、DTU CD 减的相对增益均值约25.1%，与正文一致；本地摘要23.6%未能从该口径复现。已知pose时 Giant 的 DTU1.85mm 比 VGGT1.44更差，Large1.23最优；7Scenes Pi3 F1 57.5 高于 Giant56.5。全文主张18/20 settings 可按比较 DA3 家族与外部模型理解，但若只取 Giant 逐列，有不同计数；不能擅选统计定义。Fig5 是两段视频轨迹，GT轨迹来自动态mask后COLMAP；Fig6点云、Fig7深度细节提供定性示例。

#### 结论边界

【证据边界】对齐后几何指标测量的不是无外部坐标校准时的绝对尺度误差，TSDF 及阈值也会影响点云质量；pose-free 与 GT pose fusion 不能视作仅替换网络结构的一次控制试验。7Scenes 有 pose 时部分模型反降，论文解释视频设定饱和，但并未控制模糊、相机分布或 fusion 参数分别检验。原论文未给多次训练方差、置信区间、不同视图数量的系统曲线，不能由固定协议推出任意场景和任意图数表现。

<!-- PTH:END EXP-POSE-GEOM -->

<!-- PTH:BEGIN EXP-MONO -->
### 实验二：相对单目深度与蒸馏

#### 设置

【原文事实】§7.1、§7.3（p.15,19–20），Table4/10。标准 DA2 benchmark，KITTI、NYU、SINTEL、ETH3D、DIODE，主要 δ1。Table4 比较 DA2、VGGT、DA3 多任务主模型在单图输入下及独立 synthetic teacher；Table10 是 ViT-L 单目 student 与 DA2。Teacher 以合成稠密深度训练、预测 relative depth，student 用无标签图和 teacher pseudo-depth；主模型用跨视图相机和深度联合训练。三者的用途与监督不同，不把 teacher 性能归入 DA3 主模型。

#### 数值与异常

【独立评价】主模型 vs DA2 δ1差 +0.7/−0.5/−1.7/+12.1/+0.2点，总体 Rank2.20优于2.60；Teacher97.2/97.9/81.4/99.8/96.6，五域高于主模型，但 NYU 与 DA2持平。单目 student97.1/98.0/82.3/98.8/96.5，对 DA2 +2.5/+0.1/+5.1/+12.3/+1.3点，ETH3D相对增益14.2%，SINTEL5.1点（相对6.6%）。单目 student 在 SINTEL/NYU 高于 teacher，说明学习真实数据分布可能有益，不代表伪标签总是精确。表9的几何目标消融 Depth 的δ1 0.918略低于Disparity0.919，但两种误差更小；指标间取舍需结合。

#### 证据

【证据边界】比较支持统一模型保有较强单目能力、专门蒸馏模型进一步提升，但 DA3 主模型没有逐数据集胜 DA2。δ1 不直接衡量绝对深度和法向/表面边缘；教师与学生数据和训练策略同时改变，不能将结果单独归因于 exponential depth 或教师结构。缺少水下、参与介质、动态遮挡等专门测试；不外推到这些域。

<!-- PTH:END EXP-MONO -->

<!-- PTH:BEGIN EXP-NVS -->
### 实验三：前馈 Gaussian 新视角合成

#### 设置

【原文事实】§5、§6.3、§7.1/7.5，Table5、Fig11。使用DL3DV 10,015 training scenes，140测试场景与训练互斥；另T&T6、MegaDepth19为域外。每场景300–400图、每8张目标视角，剩余12 context 用位姿最远点采样（rotation/translation）。已知COLMAP pose、270×480。对比pixelSplat、MVSplat、DepthSplat与替换backbone的Fast3R、MV-DUSt3R、VGGT；全部重训并加入depth loss，8 A100、batch1、200K步，pixelSplat因慢的epipolar attention训练100K。训练优化包括flash attention和fully shared data parallelism（原文用语，通常称FSDP）。

#### 结果

【独立评价】DA3在三个数据集 PSNR21.33/18.10/17.89、SSIM.711/.578/.561、LPIPS.241/.311/.351，9列均最佳；VGGT最强外部对照，PSNR优势 .37/.92/1.44dB，域外优势更大。相对DepthSplat +2.09/+2.30/+1.99dB，说明几何基础模型作为GS预测backbone是有竞争力的设计。原文Fig11展示细柱、宽基线户外等定性例，已知相机可降低投影错位的影响，较可靠depth减少Gaussian中心错误；SH或opacity仍可补偿几何，渲染指标并非几何真值。Fast3R在DL3DV SSIM低于DepthSplat，故“所有基础模型所有指标超过专用方法”不成立。

#### 局限

【证据边界】Table5验证的是pose-conditioned NVS，§5.2 pose-adaptive 未给独立无相机量化对照；不能将前者的优势直接称为unposed NVS优势。相同训练协议减少但不消除计算预算与预训练差别（pixelSplat步数少）；没有直接控制pose误差/深度误差或相同网络容量以证明几何能力是唯一原因。域外分数降低可能由场景、相机轨迹、光照/曝光和域移位共同引起，作者轨迹解释尚未独立证实。

<!-- PTH:END EXP-NVS -->

<!-- PTH:BEGIN EXP-ABLATION -->
### 实验四：最小表示、架构、监督与效率

#### 表示和架构

【原文事实】§7.2 p.16–18。Giant训练128 H100约10天；消融ViT-L、最多10 views、32 H100约4天。Table6相同10 views/batch128/120k steps，不用camera condition token；ray用Dual-DPT，pcd用独立DPT，无pcd才由depth+camera形成点云。depth+ray五域的 Auc3/F1(or CD) 都优于 depth+cam 和 depth+pcd+cam；加入cam后DTU Auc3 46.5→56.3、CD3.919→3.066，HiRoom F1 60.3→45.4，支持实用充分性而非辅助目标全无用。Table7(a–c)对照同规模Proposed/VGGT style/Full Alt.，VGGT style用较小ViT-B堆叠；a所有指标优于b，c在7Scenes F1 48.6>47.6例外。完整pretraining与架构同时变化，无法分开。

#### teacher/pose

【独立评价】Table7(d) 两个独立DPT替代共享reassembly，多数指标下滑但7Scenes F1 49.4>47.6；(e) 无teacher在HiRoom F1 16.0 vs47.0下降31点，却ETH3D F1 57.6>55.4、DTU Auc3 52.5>45.8且CD3.29<3.82，说明教师适合某些细节和域。Figure8展示深度边界质量。f/g使用GT pose fusion，不与a-e直接比较；已知pose conditioning使HiRoom F1+8点、ETH3D+7.7、DTU CD−1.51mm、ScanNet++ +2.9，7Scenes却−12.4点（58.4→46.0）。原文“一贯更好”不符最后一项。可能是视图选择、条件编码或域问题，但需新实验，不能指定原因。

#### 效率

【原文事实/独立评价】Table8 80GB A100、504×336，最大图数区间与32图FPS。VGGT400–500/34.1FPS，Giant900–1000/37.6，Large1500–1600/78.37，Base2100–2200/126.5，Small4000–4100/160.5；参数分组件见逐表报告。Giant吞吐约1.10倍、最大图数中点2.11倍，不能将最大图数直接等同于推理速度或精度。Table2/3 Giant1.10B和表8组件和1.198B不一致；正文Large0.30B是backbone而总量0.36B。CPU存储中间token可以扩展数量但作者未报告额外延迟。显存复杂度由图数/分辨率/注意力等共同决定，模型规模更小可能牺牲几何（Tables2–3）。

<!-- PTH:END EXP-ABLATION -->

<!-- PTH:BEGIN EXP-TEACHER-METRIC -->
### 实验五：教师数据、几何表示、损失与度量深度

#### teacher

【原文事实】§7.3.1 Table9，ViT-L batch64，采用DA2 benchmark，KITTI/NYU/ETH3D/SUN-RGBD/DIODE平均（与表4的SINTEL名单不同）。数据组 V2(.919,.087,.596)→V3(.929,.079,.508)→V3+mr(.938,.072,.452)，按δ1/AbsRel/SqRel顺序，三个指标一致提升。几何组Depth(.918,.089,.637) 相对Disparity(.919,.095,1.033) δ1略低但SqRel明显降低，相对Pointmap(.912,.096,.693)全面优；完整teacher loss(.919,.087,.596)略优于MAE(.918,.089,.637)及无Dist.Nor.(.918,.087,.600)。Fig8为主模型使用教师后的细节，不能视为教师法向loss单独消融。

#### metric

【原文事实】§7.4 Table11 metric：对比DepthPro/Metric3D v2/UniDepthv1/v2，五域NYUv2、KITTI、ETH3D、SUN-RGBD、DIODE(indoor)。DA3-metric 在ETH3D δ1=.917、AbsRel=.104均最优，在SUN-RGBD AbsRel=.105最优、δ1第二，DIODE第二，NYUv2/KITTI弱于最佳。教师消融 w/ teacher vs w/o teacher，NYUv2 (.966,.073) vs(.969,.066)，KITTI(.947,.086) vs(.965,.067)，ETH3D(.906,.105) vs(.907,.105)，SUN(.973,.104) vs(.975,.099)，DIODE(.824,.132) vs(.816,.134)；最后一组改善，其他标准指标多降/平。Fig10显示教师能改善视觉边缘。

#### 边界

【证据边界】V2→V3同时改变数据量及构成，不等价于样本数量唯一因果。多分辨率增加图像形态覆盖，是否来自计算量、增强或真实尺度差异需受控。Table11最终模型与消融训练分辨率不同，不用最终行减无教师行推断教师贡献。边缘清晰和度量深度准确有不同损失目标，应通过法向误差、边缘深度误差和尺度校准等额外评估才可充分量化（此为独立建议，非论文结果）。老师并不创造缺测区域的独立真值，RANSAC仅约束可信稀疏点。

<!-- PTH:END EXP-TEACHER-METRIC -->

<!-- PTH:BEGIN EXP-APPENDIX -->
### 实验与实现补充：附录 A 训练数据清洗

#### 数据处理

【原文事实】PDF p.31–32，Appendix A、Figures12–14。逐数据集：

| 数据集 | 原文处理 |
|---|---|
| TartanAir | 删除 amusement（无效 skybox）；carwelding/hospital/ocean/office/office2 最大深度分别截到80/30/1000/30/30 |
| IRS | RGB转灰度、深度图分别Canny，边缘膨胀1px和3px后计算交集及比值，低于阈值的样本剔除 |
| UnrealStereo4K | 删除00003（大范围错误、透明窗）、00004（clipping），00008 删除海水无深度值的指定图像 |
| GTA-SfM | 最大深度截到1000 |
| Kenburns | 深度截到50,000，依据原文脚注1 |
| PointOdyssey | 删除 animal2_s、dancingroom3_3rd，地面深度错误 |
| TRELLIS | 删除 suffix stl、abc、STL、PLY、ply（无纹理） |
| OmniObject3D | 最大深度截到10 |

00008的原文删除编号为9–13,23–29,80–82,86–88,96,103–111,126–136,144–145,148–154,173–176,178–179,186–187,191–192,198–199。截断值原文未逐项标单位，不擅自改成m。附录脚注只作为原文定位，本次未外查。

#### 作用机制

【机制解释】Fig12的skybox可导致背景被当作实物面；无效深度通过 P=t+Dd 传播到点云，法向与梯度监督也会受污染。Fig13的RGB-depth空间偏移使边界不对齐，仿射 scale/shift 只改深度数值，不能改像素坐标；用小膨胀/大膨胀后的交集比可区分轻微偏移与较大错位，但对低纹理、真实RGB和深度边界天然不一致的情况也可能误判。Fig14透明窗、clipping、海面深度缺失会给错误或无效的监督；过滤而非强制填充能避免错误标签教给模型。Depth clipping 防止极大值主导损失，但也可能减少远景的几何监督。

#### 局限

【证据边界】附录给出清洗流程和示例，未给每数据集删除比例、IRS具体阈值或完整独立消融；不能量化清洗的独立收益。过滤含透明/海面错误数据不等于证明模型能重建透明或水下介质场景；这仅是训练监督质量控制。合成数据本身也可能有域偏移与采样不足。

<!-- PTH:END EXP-APPENDIX -->

<!-- PTH:BEGIN SYNTHESIS-EVIDENCE -->
### 综合评价：贡献与证据强度

#### 主张—证据矩阵

【主张—证据矩阵】以下区分作者结论和独立评价。

| 主张 | 原文证据 | 支持强度与边界 |
|---|---|---|
| Depth-Ray 是最小且充分的主表示 | §3.1、Eq1–2、Table6 | 理想反投影几何充分；同规模消融支持实用优势，不证明所有目标组合中最优；Eq2方向/范数疑点需记录 |
| 单标准Transformer足够 | Fig2、§3.2、Table7a-c、Table8 | 经验支持完整预训练+重排架构，预训练/层数混杂；计算随输入增长 |
| Dual-DPT促进对齐 | Fig3、Table7d | 多数指标提升，7Scenes例外；无硬几何一致性保证 |
| Teacher统一异质监督 | Fig4、§4、Eq4–8、Table7e/9/11、Fig8/10、附录A | 细节有定性收益，数据规模消融支持；DTU/ETH3D/metric部分反例，伪标签非实测 |
| Benchmark支持更好geometry/pose | §6、Table1–3、Fig5–7 | 统一协议较有说服力，但对齐消除部分尺度误差，参数/数据不同 |
| DA3保有单目能力 | Table4/10 | 主模型Rank领先但非全胜；单目student五域胜DA2，任务不同 |
| Geometry foundation提升NVS | §5、Table5、Fig11 | 已知相机三域9项最优；未独立隔离预训练、容量与几何；pose-adaptive无同等定量 |

方法创新是表示选择、token重排、共享DPT和监督细化；针孔反投影、DLT、RQ、RANSAC、DINOv2、DPT、3DGS等基础方法继承已有工具，不应描述为新发明。

#### 未证实问题

【尚未证实及原文差异】本地摘要35.7%/23.6%与网页摘要44.3%/25.1%不一致，Table2逐域Auc3相对VGGT平均约44.3%，Table3 pose-free相对改善平均25.1%，不使用其替换本地摘要。正文 Table2 ScanNet++称33%，直接算35.8%；Table3称Large胜5/10，直接表值为8/10；Table7 7Scenes f/g 和 d 有反例；Table8 Giant参数和差异，Table11 Metric3D v2 DIODE δ1=.018 照录；HiRoom数量29/30、metric训练集14/13名称。数学 Eq2 的H=KR 与 d=RK^{-1}p、范数和/平方最小二乘，Eq4–6四邻点/五项、未标d/t预测或GT，均为需实现核实的原文歧义。论文无源代码实证检查（仅元数据官方仓库核验），不可根据合理猜测认定为已修复。

#### 结论范围

【独立结论】DA3展示了统一几何基础模型的有效路径：共享视觉先验，通过有结构的稠密输出结合相机和深度，再把几何作为其他三维任务的基础。强证据是多域表格和消融，弱证据是定性细节与泛化外推。不能把“最小”解释成信息论最优，“任意视图”解释成任意输入质量/数量的保证，“重建视觉空间”解释成可见和不可见几何都真实，“高保真渲染”解释成真实表面精度。§8提出动态、语言/交互等未来方向，尚未在本文完成。

<!-- PTH:END SYNTHESIS-EVIDENCE -->

<!-- PTH:BEGIN SYNTHESIS-COVERAGE -->
### 综合评价：技术边界与全文覆盖

#### 局限

【独立评价】1. 观测限制：图像无法唯一确定遮挡面，单目尤其依赖先验；稀疏/低纹理/模糊、基线小会减弱几何约束。2. 尺度限制：主模型共同归一化、benchmark对齐，metric另有焦距规范，三种不能混淆。3. 相机限制：ray或camera head的输出仍可不一致，真实透视/动态/反光可能破坏假设。4. 域限制：合成与真实数据分布不同，教师对齐不修复局部形变，未见水下特殊验证。5. 计算限制：128 H100训练成本，A100固定图数分辨率测试，不证明无限输入。6. 应用限制：pose-conditioned NVS强但pose-adaptive缺同条件量化；offset/SH允许渲染与几何脱耦。作者§8展望动态建模、语言与交互，未完成这些任务。

#### 完整性核查

【覆盖核查】PDF 共32页：p.1 摘要/首页未编号图、p.2目录、p.3–5引言/相关工作、p.6–8 formulation/architecture/training、p.8–11 teacher/student/metric、p.11–12 GS应用、p.12–13 benchmark、p.14–21全部实验、p.22结论、p.23图11、p.24–30参考文献、p.31–32附录A。已逐图 Fig1–14、逐表 Table1–11，并对首页未编号概览在INIT/FIG-OVERVIEW解释，未将未编号图擅自加入原图编号。关键数学原文 Eq1–8及反投影、总loss/confidence等未编号公式均已专门分析；实验覆盖pose/geometry/mono/NVS/表示架构消融/teacher/metric/附录清洗。参考文献用于作者自己的Related Work及baseline定位，不另读全部引用文献。全文证据定位和图像引用来自同版资源与B2验收资产。

#### 来源记录

【来源与证据边界】权威来源为用户提供的2511.10647v1 PDF、32页；同版LaTeX/HTML用于静态辅助定位公式、表格和图资源，不改变PDF权威。PDF/HTML/Source摘要相同，官方abs摘要数值不同，已在INIT与逐表及综合评价记录。四图PASS_WITH_NOTES为非精确回渲染但视觉完整，未宣称无损；HTML存在辅助资源缺失。未进行额外跨论文文献检索、第三方图搜、代码/权重下载、网站部署。所有构造数学例、机制推断和研究建议标明非作者实验。该报告的生产自检不是独立Review。

<!-- PTH:END SYNTHESIS-COVERAGE -->



---

# 第二部分：学习与研究笔记

# DA3 学习与研究笔记

> 精读报告 SHA-256：5c055363094368109cd67e1e51c25ba458571a0c0d0857f82bf33ae46f8e8c7c
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：核心概念与机制内化

### 表示充分性不等于可辨识性

§3.1 的核心是以射线的起点与方向表示相机，以沿射线的深度表示表面。把像素坐标记为齐次向量 $p$，在常见 camera-to-world 约定下令 $d=RK^{-1}p$、$t$ 为相机中心，则 $P=t+Dd$。若 $D$ 是相机轴向深度，$d$ 不一定是单位向量；将 $d$ 归一化后，配套的距离应改成 $D\|d\|$，不能保持原 $D$ 不变。以 $d=(0.6,0.8,1)$、$D=2$ 为例，位移为 $(1.2,1.6,2)$，距离 $2\sqrt2$。这解释为什么 ray 与 depth 的尺度约定是接口契约，而非实现细节。原 Eq.(2) 的旋转约定疑点详见冻结报告 MATH-CAMERA。

在理想条件下，两类输出足以生成点图和相机；但“足以描述”不意味着“能从图像唯一恢复”。例如对任意 $a>0$，同时把所有深度和相机中心放大 $a$，在相应场景与相机相似变换下透视图像不变。单目没有跨图匹配，绝对尺度、背面结构需要额外先验；即便多目相机未知，整体相似变换仍可存在。主模型归一化解决训练规范，不证明世界尺度可辨识。DA3-metric 的焦距规范和度量标签属于另一任务（§4.4）。

### 误差传播与冗余监督

补充推导：若 $P=t+Dd$，一阶扰动为 $\delta P\simeq\delta t+d\delta D+D\delta d$。忽略高阶项且假设三类误差独立，点位协方差近似 $\Sigma_P=\Sigma_t+dd^T\sigma_D^2+D^2\Sigma_d$；若联合预测导致相关误差，必须再加入交叉协方差。这不是论文的不确定性方法。它说明远处方向误差会被深度放大、近处深度误差可能占主导；只看单图深度指标不够。直接点图监督可能额外约束 $d,D,t$ 的组合，但会增加任务权衡；理想数学冗余也可能成为优化正则。Table 6 中 depth+ray+cam 局部指标改善就是不应把“最小”误作“所有辅助监督无益”的证据。

### 从 token 重排到计算代价

§3.2/Fig.2 对同一预训练 attention 权重改变 token 的批维和序列维：每图 $T$ 个 token（含相机 token），$N$ 图的图内注意力有 $N$ 个 $T$ 长序列，全局注意力有 $NT$ 长序列。仅计密集 attention score，前者 $O(NT^2)$，后者 $O(N^2T^2)$，比值 $N$；不含 MLP、显存优化及 token projection。单图时两种排列等价，跨图权重可以继承，但跨图统计分布仍可能变化。标准权重形状不变不等于推理成本不变，也不等于 attention 已学习几何匹配。替代路径可只交换少量跨图摘要 token 或按共视邻接稀疏交互，但必须检查是否损失小结构的匹配证据。Table 7(a–c) 和 Table 8 支持作者选择，不能隔离完整预训练的单独因果。

### Dual-DPT 与结构化一致性

Fig.3 共享 reassembly，两个 fusion/output 分支独立。共享空间重排让两任务的相同像素索引从同一中间特征开始，可能减少不同 decoder 学到不一致局部布局；但最终函数独立，不是硬约束 $P=t+Dd$ 的跨图一致性。设某像素深度边缘正确但 ray 方向来自错误对应，则点云仍会出现错位。可设计以重投影残差为补充的约束，但需要可见性、位姿可靠性及非刚体处理，不能直接认为它会改善全部数据集。Table7(d) 的 7Scenes F1 例外值得保留。

### 稀疏锚点对齐与密集教师先验

§4 Eq.(8) 采用仿射拟合将教师深度映射到真实有效标签的尺度。补充推导：在固定有效锚点集、平方误差及无约束的假设下，令教师值 $x_j$、真值 $y_j$，$s=\mathrm{Cov}(x,y)/\mathrm{Var}(x)$，$t=\bar y-s\bar x$。若锚点深度几乎恒定，方差接近零，尺度不稳定；若教师局部变形，任何全局 $s,t$ 都不能修复。原文实际使用 RANSAC 筛选鲁棒一致集，以上只解释其拟合子问题。只在 $s>0$ 下保序，跨视图分别对齐也不自动保证相同世界点一致。Figure 8/10 的锐度提升和 Table 11 中部分绝对误差变差说明局部高频先验与全局度量校准是两个目标。

### 渲染质量与几何真实性

§5 的 GS-DPT 预测 Gaussian 属性并借助深度确定中心，渲染 MSE/LPIPS 可监督外观；训练还用深度项。若中心沿视线偏移而颜色和 opacity 适配，源视图仍可能重建良好；横向移动的目标视图才揭示视差错误。在简单平行双相机近似下，视差 $b f/Z$ 对深度扰动的导数为 $-bf/Z^2$，小基线或远景弱化这个约束。该推导不是本文实验。Table5 使用已知位姿，不能推断 pose-free 渲染或真实表面精度。应并列阅读 Table3 和 Table5，避免单一 RGB 指标替代真实几何评价。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Scene Representation / Depth-Ray factorization
- **Keywords:** Depth Map; Ray Map; Point Map; Camera Calibration; Gauge Freedom
- **Highlight:** 将深度与世界射线分开预测，可组合三维点并恢复相机；同规模消融支持实用优势，但附加 camera head 仍可改善部分指标。
- **Evidence:** §3.1, Eq.(1)–(2), Table 6；精读 MATH-RAY、MATH-CAMERA。
- **Connection Points:** 主预测变量、相机规范、坐标系、冗余监督、可辨识性。
- **Open Questions:** 理想表示充分性和网络有限容量下的优化收益何时冲突？

### Topic: Geometry Inference / Adaptive cross-view attention
- **Keywords:** DINOv2; Self-Attention; Token Reshaping; Cross-View Reasoning; Variable View Count
- **Highlight:** 图内和跨图注意力使用相同预训练 ViT 块，通过重排 token 实现跨图访问；相机 token 参与注意力，可输入真实位姿或学习 token。
- **Evidence:** §3.2, Figure 2, Table 7(a–c,f,g), Table 8。
- **Connection Points:** 预训练复用、匹配范围、图数成本、位姿条件、视图顺序/参考系。
- **Open Questions:** 共视关系稀疏或位姿先验错误时全局注意力是否仍可靠？

### Topic: Dense Decoding / Aligned prediction heads
- **Keywords:** DPT; Reassembly; Multi-Task Learning; Feature Fusion; Depth-Ray Alignment
- **Highlight:** depth/ray 共享特征重组而保留各自融合与输出，不是显式一致性投影；Table7(d) 多数指标受益但有例外。
- **Evidence:** §3.2, Figure 3, Table 7(d)。
- **Connection Points:** 共享位置、参数量、预测边界、结构约束与梯度干扰。
- **Open Questions:** 对齐收益来自共享空间布局还是容量与优化动态？

### Topic: Training & Supervision / Heterogeneous labels
- **Keywords:** Synthetic Data; Relative Depth; Pseudo Label; Scale-Shift Alignment; RANSAC; Surface Normal
- **Highlight:** 合成训练的密集教师经真实有效深度做仿射对齐，提供局部结构监督；教师并非独立真值且不保证全域度量误差下降。
- **Evidence:** §3.3–4.4, Eq.(3)–(8), Figures 4,8,10, Tables 7,9,11, Appendix A。
- **Connection Points:** 缺测机制、教师先验、对齐稳定性、置信度、几何和度量监督。
- **Open Questions:** 锚点覆盖或教师域偏移变化时何时会发生负迁移？

### Topic: Observability / Monocular versus multi-view geometry
- **Keywords:** Monocular Depth; Sparse Views; Multi-View Consistency; Metric Depth; Similarity Alignment
- **Highlight:** 主模型兼容单图和多图，单图不具备跨视角几何约束；度量学生有专门焦距规范及度量监督，不能与主模型相对几何尺度混为一谈。
- **Evidence:** §3.1, §4.3–4.4, §6, Figure 9, Tables 3,4,10,11。
- **Connection Points:** 观测数量、几何先验、评价对齐、绝对尺度、相机内参。
- **Open Questions:** 如何评估稀疏条件下先验驱动的合理性与观测驱动的真实性？

### Topic: Geometry-to-Rendering / Feed-forward Gaussians
- **Keywords:** Feed-forward 3D Gaussian Splatting; GS-DPT; Novel View Synthesis; Pose Conditioning; Depth Regularization
- **Highlight:** 以几何模型为基础追加 Gaussian head，通过深度及相机确定中心，已知位姿渲染对比领先；未知位姿扩展缺少等价定量协议。
- **Evidence:** §5.1–5.2, Table 5, Figure 11。
- **Connection Points:** 高斯中心来源、位姿依赖、几何约束、外观拟合、域泛化。
- **Open Questions:** 几何先验准确性与 NVS 指标的改善在何种视点跨度下相关？

### Topic: Evaluation / Evidence boundaries
- **Keywords:** AUC; RRA; RTA; F1; Chamfer Distance; TSDF; Benchmark Protocol
- **Highlight:** 分开评价相机、几何、单图深度、渲染；对齐协议消除一部分规范差异，局部例外与平均提升可同时成立。
- **Evidence:** §6–7, Tables 2–5,7,11；精读 SYNTHESIS-EVIDENCE。
- **Connection Points:** 评估尺度、测试输入、数据隔离、指标方向、均值聚合口径。
- **Open Questions:** 在不允许尺度对齐且相机未知的设置下，模型排序是否改变？


## RESEARCH-DIRECTION：一般研究方向分析

本次任务未明确指定独立研究主线，以下仅从本论文提取可迁移问题，不将任何领域外推当作作者结果。Field 是三维视觉与学习式空间推断；Task 包括相机估计、可见表面重建、单目深度和新视角合成。Task Setting 是可变视图数量、可选位姿、混合来源监督及前馈预测；不同实验对尺度和位姿的条件不一致。Research Problem 则不是“做一个更大的 DA3”，而是如何在观测约束不完整、监督不一致的情况下推断可信的空间结构。Depth-Ray、token attention、教师是当前实现选择，不是问题定义本身。

### 观测与先验的区分

Figure 9 同时展示 1/2/13 图输入，但场景不同，不能作视图数消融。真正有价值的方向是在固定场景与相机轨迹下改变图数和有效共视区域，识别由少量几何观测引起的误差与由学习先验引起的误差。对有共视纹理的区域，可借助重投影证据检验相对深度/相机；对无共视区域，模型输出只能依赖单图或全局语义先验，合理不等于真值。需要同时报告两类区域性能，而不是把总体指标当作每像素置信度。

### 可观测性、可辨识性及输出接口

同一三维点在两张已知相机图中有可靠匹配时可三角化；基线短、遮挡、重复纹理或未知相机削弱约束。相似变换不改变普通针孔投影，故仅有 RGB 不一定能恢复真实米制尺度。Depth-Ray 提供显式接口，可诊断深度误差、相机误差和两者交叉项，适合作为研究对象；但若预测 ray 不满足同中心的刚性相机，稠密输出可能形成几何不一致而仍在部分指标上有效。可以对比显式合法相机参数化与自由 ray 场，再研究表达自由度与物理约束之间的取舍。

### 监督质量与先验迁移

§4 和附录 A 展示合成标签也可能错误：透明、裁切、海面缺失及 RGB-depth 错位分别破坏监督假设。迁移到特殊成像域时，教师相对深度的排序也可能错误，而尺度对齐只能处理全局线性差异。应在有准确几何的控制数据上验证教师结构与锚点覆盖，而非只用更清晰的深度可视化判断成功。对现实标签缺失具有系统性（如远景或特定材质）的情况，随机掩码模拟不足以代表真实难度。任何特殊领域应用均尚未由本论文验证。

### 从确定性输出到可靠性推断

论文的 confidence-weighted loss 是训练中的损失加权机制，不自动等同于校准的不确定性或真实误差。可以将深度、射线的预测残差和可见性支持程度分别估计，再检验二者是否能预测在留出视图或真值中的误差。标称置信区间覆盖率、误差分箱与 risk-coverage 曲线均比单一平均置信度更有诊断性。这是后续研究路线，非本文结果。

### 前馈与优化的分工

前馈模型给出跨场景可复用的先验和初始空间，但局部几何可以再用具体场景观测优化。若初始几何先验与观测冲突，应有可靠性判据决定是否允许更新；无可靠证据的区域盲目优化可能把模型推向伪解。§5.2 的额外偏移和高斯外观自由度强调渲染与几何可脱耦。合理评价应固定观测并比较前馈、仅观测优化、前馈+约束优化，不预设哪条路线必胜。


## RESEARCH-PROBLEMS：待验证科学问题

以下是由当前论文证据引出的候选问题，不是原创性声明或已完成文献调查。优先级依据可控性与对核心机制的解释价值，而非预计发表价值。

### RP-A（优先）：几何证据不足时，如何辨别可信空间推断？

**现象与未知**：Fig.9 展示不同图数，Table3 在有/无位姿下给出几何结果，Table7 条件消融有反例。图数不是有效几何约束的充分指标：新增视图若几乎同中心或不共视，不提供足够视差。未知在于何种观测特征能预测 depth-ray 输出的错误及其相关性。

**假设**：固定图数时，有效共视与三角化角度能解释部分误差；结合模型预测和观测几何支持的可靠性判据，在留出场景上优于仅使用网络置信度。可证伪：若在控制图数与图像质量后，条件可靠性不改善风险排序，或只在训练域有效，则不支持假设。

**验证**：构建已知真值表面/相机的多场景视图集合，固定分辨率和模型权重，在 1/2/4/8 图设置内独立改变基线和重叠率。将遮挡、反光、低纹理作为独立分层变量；同时设置真实位姿、预测位姿及受控扰动。比较原生 confidence、仅共视几何指标、二者结合；冻结验证集阈值后再测试。以世界点误差、深度误差、角度误差、risk-coverage 及分箱校准为指标，明确相似对齐与度量尺度两个协议。报告可见区域与仅先验区域各自覆盖率，不能只删除难点后报告平均误差。成功应在同覆盖率下降低风险且跨场景稳定；失败包括可靠性过度依赖纹理而拒绝所有无纹理正确区域。代价：需要精确相机和 GT、固定模型版本及足够计算；可先在小型合成场景验证，再扩展真实域。

### RP-B：教师仿射对齐的可靠性是否能由锚点与局部几何检验预测？

**现象与未知**：Eq.(8) 用可靠真实深度对齐教师，Fig.8/10 细节改善但 Table7/11 存在几何/绝对误差反例。未知是锚点少、分布集中与教师局部结构失真如何共同限制对齐后的监督质量。

**假设**：锚点的深度跨度、图像空间覆盖和对齐残差可以解释拟合不稳定性，但仅有低残差不能识别未覆盖区域的教师偏差；增加局部独立几何检验可降低负迁移。通过在完整 GT 上控制锚点掩码检验：保持锚点数量不变，比较均匀、深度集中、远景缺失、边缘缺失。对同一教师输出施加全局比例/平移、局部非线性深度扭曲两类受控扰动；比较 GT-only、原 RANSAC teacher、可靠性加权 teacher，确保训练步数、骨干和数据一致。

**判据与反证**：除 AbsRel/δ1，报告边缘误差、法向误差、锚点外误差和跨图点云误差；在有效锚点外评估而非重复拟合点。若局部检验不降低错误且损失细节，或效应由额外训练量造成，则否定机制收益。先离线分析拟合再开展学生训练以降低成本。锚点本身错误、传感器系统偏差或完全无锚点时问题不同，不能声称该方法解决全部。需后续核查稳健伪标签、跨视角教师与校准文献。

### RP-C：什么情况下几何改进能转化为新视角渲染收益？

**现象与未知**：Table3/5 分别表明几何与已知位姿 NVS 表现强，二者不是单变量因果对照。§5 高斯中心与外观自由度允许颜色误差掩盖几何误差。未知是视点外推距离、几何初值和外观自由度怎样决定这一关系。

**假设**：小视点变化下外观指标对几何错误不敏感，较大横向视点变化且可见性受控时，中心准确性更影响结构投影。实验固定场景和输入位姿、Gaussian 数量与外观 head，分别用 GT depth、原预测、受控深度偏差产生中心；冻结中心和允许小偏移作为两组，另外控制 SH/opacity 容量。将测试视点按内插与外推、视差和遮挡分层。报告 PSNR/SSIM/LPIPS，同时独立测量表面误差、深度边缘与重投影误差；不能把 RGB 质量作为几何标签。若外观自由度变化后几何误差与渲染收益仍无稳定联系，或收益仅由可见区域大小解释，则不支持假设。需真实几何与新视图 GT，避免单纯扩大基线导致不可见面增多混淆；可先对已有模型做推断扰动，完整 retraining 较昂贵。

### 优先顺序与研究边界

A 最直接回应任意图数的证据缺口，先做无需重训的受控实验；B 直接利用原文反例，可先离线验证对齐稳定性，再决定是否重训；C 与渲染应用相关，但训练与可见性混杂更多，放在前两项获得可解释几何诊断后。三者均需进一步文献检索确认是否已有解决方案及适用范围，不能将本文未覆盖等同于领域空白。


## INTERNALIZATION：知识内化与技术地图

### 用自己的话说明论文

DA3 将多项三维任务放到一个共同的像素几何接口上：每个像素既有深度，也有指向世界坐标的射线。射线携带相机几何，深度选出该射线上的可见表面点，二者合起来得到点图。通过把多个图像的 token 临时排列成共同序列，标准 DINOv2 主干既可处理单图，也可以互相读取其他视图信息。Dual-DPT 将共享的空间重组特征分别解码成两个输出。密集教师的作用是补充真实深度标签缺失的局部结构，而不是凭空提供绝对尺度。单图学生与度量学生、Gaussian head 则是从这一训练思路扩展出的不同产品，不应当作同一输入协议下的单一模型。

### 技术地图

观测 RGB/可选相机 → patch 与 camera token → 图内/跨图 attention → 共享 reassembly + 独立 fusion → D 与 ray → 点图、相机、融合几何；并行的监督链是合成 dense depth → relative teacher → 真实有效标签的 scale-shift 对齐 → 主模型监督。独立训练/微调链包括单目 student、metric student 和 pose-conditioned GS-DPT；pose-adaptive Gaussian 应用另有位姿预测及偏移等设计。各链用不同评估指标：AUC 相机、F1/CD 几何、δ1/AbsRel 深度、PSNR/SSIM/LPIPS 渲染。应先匹配任务条件再比较数值。

### 自检问答

**为什么不是仅预测点图？** 点图可以携带三维位置，但从稠密点反推相机和相对深度是另一种参数化；DA3 把局部深度和射线几何分开，并用 Table6 比较目标组合。不是信息论证明点图不足，也不能断言点图永远更难学习。

**射线图是不是相机参数的冗余表达？** 理想中心投影中，同图所有射线起点相同且方向由少量相机参数决定；每像素表示是冗余的，但适合密集解码。若自由预测不满足刚性相机约束，需聚合/求解或相机头；该差别正是表示充分性与合法性之别。

**预训练如何跨视图？** attention 的投影权重不依赖 token 数量，改变序列布局即可让 query 看到其他图的 token，但不保证它们正确匹配；需训练和几何监督。不能理解为 DINOv2 在原始预训练时已经完成多视角几何重建。

**为什么教师锐度提升而 AbsRel 可能变差？** 相对深度教师可以提供高频局部边缘，但全局仿射对齐只约束整体比例和平移。边缘形态、绝对尺度、局部表面位置可能存在取舍，Table11 与 Fig10 需要一起看。

**为什么位姿已知仍需要深度？** 位姿只定义射线，不确定表面在射线的哪个位置。已知相机降低部分几何歧义，并不替代表面估计；单图情况下尤为如此。

**好的 NVS 能否证明几何好？** 不能。输入/目标视图、位姿、外观自由度和可见性均影响 RGB 指标。作者额外测量几何是必要的；§5 的已知位姿评估不直接支持未知位姿的新视图性能。

### 后续需要验证的理解

查阅实现确认 ray 方向定义、相机提取坐标约定、confidence 的具体参数化、教师法向邻域索引与 metric focal 处理；查明 Table8 参数统计口径及原文部分数值不一致。上述事项未通过代码检验，不在本篇知识库中写成已解决事实。


## GLOSSARY：术语与概念关系

| 英文术语 | 中文 | 含义及区别 |
|---|---|---|
| Depth map | 深度图 | 每像素一个深度值；轴向深度和欧氏距离依 ray 规范不同，参见 Eq1。 |
| Ray map | 射线图 | 每像素起点/方向共六维；不是已经观测到的表面点。 |
| Point map | 点图 | 每像素三维位置；可由 depth-ray 合成。 |
| Intrinsics / Extrinsics | 内参 / 外参 | 成像投影与相机姿态，内参不描述场景表面。 |
| Gauge freedom | 规范自由度 | 多组坐标/尺度表示可有相同观测；与预测误差区分。 |
| DLT / RQ | 直接线性变换 / RQ 分解 | 由对应射线拟合投影关系、再分离相机内参与旋转；约定需核查 Eq2。 |
| Input-adaptive attention | 输入自适应注意力 | 根据视图数量/序列布局改变注意范围，而非新的专用权重。 |
| Camera token | 相机 token | 相机参数编码或可学习替代，与 patch token 一起参与注意力。 |
| Reassembly | 特征重组 | 将 token 重整为空间特征；不同于分支后续 fusion。 |
| Dual-DPT | 双分支 DPT | 共享 reassembly、分别融合和输出；不构成硬一致性约束。 |
| Relative / Metric depth | 相对 / 度量深度 | 前者重视相对几何且可有尺度歧义；后者需校准实际长度。 |
| Scale-shift alignment | 尺度平移对齐 | 深度值的仿射映射，不改变像素空间位置。 |
| RANSAC | 随机采样一致性 | 鲁棒估计拟合参数，不能修复无有效锚点或结构性偏差。 |
| Pseudo label | 伪标签 | 教师生成的训练目标，区别于独立实测 GT。 |
| Surface normal | 表面法向 | 局部表面方向，受深度梯度与相机投影共同影响。 |
| Confidence-weighted loss | 置信加权损失 | 训练目标的权重与防退化正则，不自动等价误差校准。 |
| TSDF fusion | 截断符号距离场融合 | 多视角深度融合表面的方法，属于评估/重建流程而非 ray 定义。 |
| RRA / RTA / AUC | 相对旋转/平移准确性、曲线下面积 | 位姿误差阈值下的累计准确性，AUC3 与 AUC30 严格度不同。 |
| F1 / CD | F1 / Chamfer 距离 | 前者综合精度与召回，后者测双向邻近距离；方向与阈值不同。 |
| AbsRel / δ1 | 平均绝对相对误差 / 阈值准确率 | 深度误差平均值与比例阈值内像素比例，后者越高越好。 |
| GS-DPT | Gaussian 属性预测头 | 像素对齐地输出渲染属性，以深度辅助中心确定。 |
| NVS | 新视角合成 | 给定观测生成新相机图像，不等于真实几何重建。 |
| Pose-conditioned / Pose-adaptive | 位姿条件 / 位姿自适应 | 已知位姿输入与面向未知位姿的扩展，不混用实验结论。 |

术语关系链：相机定义 ray，depth 选择点，point map 可融合几何；relative 与 metric 由尺度规范区分；confidence 是预测/训练信号，真实误差需要独立证据。上述概念定位于精读 MATH-RAY、MATH-CAMERA、METHOD-BACKBONE、METHOD-TEACHER、METHOD-GS、各实验单元。


## SOURCES：来源、版本与证据范围

权威原文：Haotong Lin et al., *Depth Anything 3: Recovering the Visual Space from Any Views*, arXiv:2511.10647v1，用户提供的 PDF 32 页（同包 HTML、LaTeX Source 为辅助证据）。[arXiv](https://arxiv.org/abs/2511.10647v1)、[官方项目](https://depth-anything-3.github.io/)。本笔记依据冻结报告 SHA-256 `5c055363094368109cd67e1e51c25ba458571a0c0d0857f82bf33ae46f8e8c7c`；B3 已覆盖 57 单元，14 编号图、11 表、Eq1–8 及附录。图源使用实际验收的原 PDF 区域副本，四幅有非精确回渲染记录，不宣称像素级无损。

原文事实由正文、图表和公式支撑；补充数学推导均注明假设；研究问题、替代设计和验证计划属于独立拓展，不是作者实验。网页摘要与本地 PDF 摘要数值不同，正文也有口径差异，保留于冻结报告，未静默修订。Eq2/法向公式记法、Table8 参数统计、部分表格口径等待实现或作者确认。未进行代码执行、模型复现或额外论文检索。

跨文献分析延期：CROSS-PAPER 仅是本篇节点索引，没有读取其他论文、比较跨论文优劣或更新全局知识库。一般研究方向未擅自绑定用户私有研究内容；此笔记拟作为完整版组成部分，隐藏展示不是访问权限控制。后续讨论的私有研究材料不得自动纳入公开成果。本次质量审查为生产自检，非独立 Review。



