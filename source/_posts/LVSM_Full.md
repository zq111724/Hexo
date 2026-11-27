---
title: "LVSM：以最小三维归纳偏置实现稀疏视角新视角合成（完整精读与研究笔记）"
date: 2026-11-27 07:03:36
index_img: "/LVSM/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/LVSM/)

# 第一部分：论文精读报告

# LVSM 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2410.17242v2（2025-04-02）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2410.17242) | [PDF](https://arxiv.org/pdf/2410.17242v2) | [Project Page](https://haian-jin.github.io/projects/LVSM/) | [Code](https://github.com/Haian-Jin/LVSM) | [Publication](https://proceedings.iclr.cc/paper_files/paper/2025/hash/9676c5283df26cabca412ca66b164a7d-Abstract-Conference.html)

<!-- PAPER-METADATA END -->

> 论文：LVSM: A Large View Synthesis Model with Minimal 3D Inductive Bias
> 来源：用户上传 arxiv_2410.17242v2.zip；正文与附录共21页；PDF为最终证据，HTML与Source仅辅助，后者无内部版号。
> 证据分类：【原文事实】【背景知识】【合理推断】【独立评价】【尚未证实】。图像为PDF区域渲染展示副本。

<!-- PTH:BEGIN INIT -->
## 0. 来源与结构

### 版本与证据

【原文事实】论文完整英文标题 LVSM: A Large View Synthesis Model with Minimal 3D Inductive Bias；作者 Haian Jin、Hanwen Jiang、Hao Tan、Kai Zhang、Sai Bi、Tianyuan Zhang、Fujun Luan、Noah Snavely、Zexiang Xu。PDF 首页印记 arXiv:2410.17242v2，2025-04-02。全文21页，第一页标题与摘要；正式发表 ICLR 2025（Oral，由官方项目页核验）。PDF 与 HTML 的标题及身份匹配；Source 中 main.tex 定义 paperName=LVSM 并以该完整标题编译，但无独立版本印记，故不把文件路径作为内部同版证明。原始 PDF、HTML、Source 压缩包及图像按接收字节保留。阅读以 PDF 为最终权威，Source 只辅助核对宏和公式代码，HTML 只辅助定位；不读取相关文献全文。

### 章节与目录

| 部分 | PDF 页码 | 内容 |
|---|---:|---|
| Abstract、1 Introduction | 1–2 | 研究目标，两级归纳偏置与两种架构 |
| 2 Related Work | 2–3 | IBR、3D representation optimization、generalizable NVS |
| 3 Method | 4–5 | §3.1 token 化与预测，Eq.1–5；§3.2 两种架构，Eq.6–8 |
| 4 Experiments | 6–10 | §4.1–4.5 数据、训练、对比、消融、更多视图与效率 |
| 5 Conclusion | 10 | 质量、速度、扩展性的总结 |
| References | 11–17 | 原论文参考文献列表；本报告未独立精读被引论文 |
| Appendix A.1–A.7 | 18–21 | 命名、训练细节、256 图、架构、掩码、生成式区别、局限 |

| 图表 | 页码 | 功能 |
|---|---:|---|
| Fig.1 | 2 | 场景/物体示例 |
| Fig.2 | 4 | 总体模型 |
| Fig.3 | 7 | 512 物体定性 |
| Fig.4 | 8 | 场景定性 |
| Fig.5 | 10 | 输入数泛化 |
| Fig.6 | 10 | 解码 FPS |
| Fig.7 | 19 | 256 物体定性 |
| Fig.8 | 19 | 详细结构 |
| Fig.9 | 20 | attention mask |
| Tab.1 | 6 | 主要性能比较 |
| Tab.2 | 9 | 模型规模 |
| Tab.3 | 9 | 结构消融 |

正文6幅图、3张表，附录3幅图，无独立编号 Algorithm 块。图像全部基于同版 PDF 区域提取。HTML 的外部装饰资源缺失，科学图像可定位；不影响 PDF 阅读。所有章节均可读，网站视频未纳入证据。初始化只核查范围，不将元数据当科学论证。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

本单元承担来源、术语或目录核查，不单独推导网络机制；机制详见第3.4节。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1.1 标题

### 标题翻译

**LVSM：一种具有最少三维归纳偏置的大规模新视角合成模型。** Large View Synthesis Model 指在跨场景训练、具有可扩展 Transformer 容量的视图合成网络，而不是每场景优化的大型三维模型。View Synthesis 指由已知的少量视图生成指定新相机下图像；不是直接输出深度、mesh 或点云。Inductive bias 是训练样本不足以唯一决定函数时模型所预设的结构与偏好。3D inductive bias 在本篇中特指显式三维表示、固定物理渲染规则以及利用极线/平面扫描的网络结构等。

### 任务边界

【原文事实】§3.1 给定已知相机内外参及图像，目标相机也已知；LVSM 保留逐像素 Plücker 射线条件。因而 minimal 不是 geometry-free 输入，也不是完全无先验。网络仍有 patch 化、全自注意力、模型参数与图像监督等归纳偏置。Encoder-Decoder 保留可学习的固定 latent；Decoder-Only 消除的是单独、可复用的中间场景表征，内部依然有隐特征。标题无法推出输入相机可未知、模型具备真实几何输出、任意观测数都可靠或任意域均可泛化。论文方法是 deterministic RGB prediction，非随机扩散采样。依据 p.1、§3、A.1/A.6。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

本单元承担来源、术语或目录核查，不单独推导网络机制；机制详见第3.4节。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 摘要

### 问题与动机

【原文事实】p.1 摘要提出稀疏输入下可扩展、可泛化的新视角合成。先区分两种既有先验：NeRF/3DGS 等规定场景表示和渲染；极线投影、plane sweep 等规定网络信息流。它们有利于几何一致性和样本效率，但当固定表示/操作难覆盖复杂结构和视角依赖外观时，可能形成表达上限。作者选择从数据学习 NVS 映射，不再先把输入强制变成某种预定义 3D 参数。这里的困难不是证明所有几何先验有害，而是探究能否减少先验并借助模型容量与多视图数据获得高质量。

### 两条路线

Encoder-Decoder 将输入 patch tokens 压到固定数量的一维 latent tokens，再条件于目标射线生成图像。Encoder 学习聚合、latent 学习表征、Decoder 学习渲染；输入数变化不直接改变已编码后每个目标的解码序列长度。Decoder-Only 则将输入和目标射线 tokens 一并送入单流 Transformer，仅取目标位置的输出；取消中间信息瓶颈，输入与目标可在各层交互。两者共同点是不用显式体渲染或 Gaussian splatting，且直接回归 RGB。

【原文事实】摘要称 Decoder-Only 相对领先方法约提升 1.5–3.5 dB PSNR，并且零样本输入数量泛化更强；Encoder-Decoder 已编码后渲染较快。Table 1 支持前者在所有报告主比较中领先，但 Encoder-Decoder 并非每一指标领先。主模型64 A100训练，单/双卡结果是更小模型和 batch，不等同于主模型单卡复现。Fig.5 的 zero-shot 是相同任务下输入视图数变化，不能解释为水下等任意域零样本泛化。A.7 还说明未见区域及训练外长宽比局限。摘要一句话：将带相机条件的稀疏视图合成转化为直接的多视图 token 到目标 RGB 的监督学习。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 贡献

### 技术贡献

作者在引言 p.1–2 并未以固定编号列出贡献；以下为归纳。第一，提出纯 Transformer 的 Encoder-Decoder NVS，不再规定 latent 的三维坐标结构或显式体/点渲染；固定数量 learnable tokens 从输入中聚合场景条件，目标射线与其全自注意交互。第二，提出更彻底的 Decoder-Only LVSM，移除独立 latent bottleneck，直接联合处理原始图像/射线 tokens 与目标射线 tokens。第三，统一使用 patch 级的相机条件和 RGB 回归，以及双向自注意力，不是将语言模型的因果 mask 移植到图像；通过掩码消融检验信息交换，而非只报告模型名字。

这些贡献的性质不同：第一、二项属于建模与架构选择，第三项是具体网络设计与验证。Plücker 射线、ViT patchify、Transformer、感知损失和 QKNorm 并非本文发明。单卡、双卡结果和尺度实验是经验贡献。未见作者证明关于三维归纳偏置的普遍定理。

### 证据对应

| 主张 | 直接证据 | 解释 |
|---|---|---|
| 两种 NVS 可不使用固定 3D 表示 | Eq.1–8、Fig.2、8；Tab.1 | 架构与性能证据，不是几何真实性证明 |
| Decoder-Only 质量更高 | Tab.1–2、Fig.3–5、7 | 在报告的训练和测试条件成立 |
| 双向信息交换重要 | Tab.3、Fig.9 | mask 干预后 PSNR 下降；不能独立归因于唯一机制 |
| 固定 latent 有效率优势 | Fig.6 | 只计解码，不能直接代替端到端延迟 |
| 小型模型可降低训练资源门槛 | §4.3 p.7–8 | 1/2 A100、7天，小模型配置；不等同主模型成本 |

【独立评价】设计上减少约束、数据上补足学习先验是核心逻辑。实验能支持具体选择的有效性，不意味着传统三维表示无价值，或相同结构在数据不足时仍领先。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 结论

### 作者结论

p.10 §5：LVSM 旨在减小预先写入模型的三维归纳偏置，以可扩展纯 Transformer 从训练数据习得 NVS。Encoder-Decoder 固定 latent 支持快速重复解码；Decoder-Only 以较长序列保留原始观测信息，质量及输入数泛化更强。物体和场景实验分别训练，Table 1 上主模型有较优 RGB 指标。用低资源配置也能得到有竞争力的结果（§4.3）。两条路线不是速度与质量都占优的同一模型，而是两种不同的权衡。

### 结论边界

【原文事实】A.6 讨论 deterministic model 逐目标独立预测与概率生成方法的差别，作者观察在已观测区域能生成一致视图；A.7 明确未见区域难以高质量预测，常出现噪声、闪烁和固定图案，建议结合生成方法。训练外长宽比与分辨率也可能退化，作者提出扩大训练数据多样性。原文没有真实深度误差或几何重建指标；“understand depth” 是单视图表现引出的解释，不是深度已得到准确测量的证据。本文没有验证未知相机、跨水下介质、动态场景，也没有证明任意输入视角分布或无限视图都稳定。质量、速度和泛化的结论要保留条件。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2.1 图表功能总览

### Figure 功能

| 图 | 问题 | 阅读重点 |
|---|---|---|
| 1 | 这种无预设显式三维表示的 NVS 能生成什么？ | 场景与物体、输入/输出、单图案例；不是完整统计 |
| 2 | 怎样从图像与目标相机输出 RGB？ | 两条 token 流程、是否有 latent |
| 3 | 512 物体高分辨率表现如何？ | 细结构、孔洞、反光、纹理与 GT |
| 4 | 真实室内场景表现如何？ | 窗、反射、台面、栏杆及伪影 |
| 5 | 训练4视图后能否适应不同输入数？ | 三条 PSNR 曲线、训练外输入数 |
| 6 | 多视图成本如何？ | 解码 FPS 与输入数量的关系 |
| 7 | 256 物体表现如何？ | LGM 与 GS-LRM 及两种 LVSM |
| 8 | 具体网络张量与层如何实现？ | 输入9通道、目标6通道、3072 latent、QKNorm |
| 9 | 为什么完整 self-attention 有价值？ | 四个注意力掩码的信息方向 |

正文 Fig.1–6；附录 Fig.7–9。图1、3、4、7是定性证据，图2、8为架构示意，图9说明消融干预，图5–6为定量曲线。读图不能以架构的箭头证明实验效果，也不能以好看的 RGB 代替真实三维指标。

### Table 功能

Tab.1（p.6）在 ABO/GSO 和 RealEstate10K 比较 RGB 指标；Tab.2（p.9）改变层数和 encoder/decoder 分配，固定小规模资源；Tab.3（p.9）改变 tokenizer 和 attention mask，直接联系 Fig.9。表1为主比较，表2、3使用特定控制配置，不能混用数值当作单一公平排名。原论文没有独立 Algorithm 框，方法由 Eq.1–8 及 Fig.2/8 描述。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

本单元承担来源、术语或目录核查，不单独推导网络机制；机制详见第3.4节。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
## 2.2 Figure 1

### 题注与图像

![Figure 1](LVSM/Figures/Figure_01.jpg)

来源：原论文 PDF p.2，§1；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 1: LVSM supports feed-forward novel view synthesis from sparse posed image inputs (even from a single view) on both objects and scenes. LVSM achieves significant quality improvements compared with the previous SOTA method, i.e., GS-LRM (Zhang et al., 2024). (Please zoom in for more details.)

**中文翻译：** LVSM 支持由稀疏的已知位姿图像（甚至单幅图像）对物体和场景执行前馈新视角合成。与此前领先方法 GS-LRM 相比，LVSM 显著改善质量（请放大查看细节）。

### 视觉与机制

左侧场景组上两行是已知图像、GS-LRM、Ours、GT；卧室中的床铺/窗边与厨房或餐厅的反射区域被标框。输入列为少量原视图，不是目标视图的真值；预测列尝试在目标相机下恢复新图。底部单幅输入厨房图及两个新视角表明模型不必总有多图对应，单图时主要依赖可见外观、射线与学习先验。右侧是物体级组，三行分别为玩具、家具组合、蓝色金属架；每行四张输入，两个合成新视角。金属架的细杆、玩具的颜色和形状在新方向保留，体现细结构渲染能力。此图没有为右侧所有物体提供逐方法/GT列，因此只能看输出例子，不能计算其精度。

【机制解释】给定输入 RGB 和逐像素射线以及目标射线，Transformer 学习跨输入位置聚合并输出目标 patch；这里的 feed-forward 是共享网络权重一次或固定次数计算，不需为每个场景梯度优化 3D 参数（§3.1）。

### 证据边界

Fig.1 是定性 teaser，不能说明失败率、单图重建真实深度或不同域泛化。单图新视角必须依赖不可观测区域的学习先验；A.7 明确未见区域存在噪声和闪烁。Tab.1、Fig.5 才是相应统计证据；图中 Ours 的具体模型变体不能从图1题注唯一确定。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
## 2.2 Figure 2

### 题注与图像

![Figure 2](LVSM/Figures/Figure_02.jpg)

来源：原论文 PDF p.4，§3；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 2: LVSM model architecture. LVSM first patchifies the posed input images into tokens. The target view to be synthesized is represented by its Plücker ray embeddings and is also tokenized. The input view and target tokens are sent to a full transformer-based model to predict tokens that are used to regress the target view pixels. We study two LVSM transformer architectures, a Decoder-only architecture (left) and a Encoder-Decoder architecture (right).

**中文翻译：** LVSM 模型架构。首先将带位姿的输入图像切成 patch tokens；待合成目标视图由 Plücker 射线嵌入表示，也被 token 化。输入与目标 tokens 进入完全基于 Transformer 的模型，预测用于回归目标像素的 tokens。研究 Decoder-Only（左）与 Encoder-Decoder（右）两种架构。

### 视觉与机制

左侧输入两类：带 RGB 的输入视图/射线（绿色、黄色 token）和仅有目标射线（粉色 token）。目标相机已知但目标 RGB 不提供。二者进入同一 Decoder-Only Transformer，输出仍按输入和目标位置排列；淡化的输入输出 tokens 不用于像素回归，目标输出经 linear/unpatchify 形成中央阳台图。右侧 Encoder-Decoder 的 Encoder 同时接收多视图输入和灰色 learnable latent tokens；输入输出丢弃、更新 latent 进入 Decoder，再与目标粉色 tokens 交互。Decoder 的输出 latent 也被丢弃，使用目标 token 回归同一目标图。

Eq.(1)(2) 对输入 RGB+6维射线和目标6维射线分别线性投影；Eq.(6)(7) 为两阶段拼接，Eq.(8) 为单阶段拼接。相同输出头 Eq.(4)，相同 RGB+感知监督 Eq.(5)。右侧 latent 与目标无关，适合跨目标复用；左侧从第一层开始就让目标条件参与输入信息聚合，不能把输入上下文当独立缓存的最终 scene latent。两侧没有显式 3D 点、深度图或渲染方程接口。

### 证据边界

图2说明模型定义，不说明真实隐式几何是否正确。没有因果掩码，decoder 是输出职责的命名（A.1），不是 LLM 自回归。详细张量及注意力块见 Fig.8/9；质量与速度用 Tab.1、Fig.5/6检验。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
## 2.2 Figure 3

### 题注与图像

![Figure 3](LVSM/Figures/Figure_03.jpg)

来源：原论文 PDF p.7，§4.3；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 3: Object-level visual comparison at 512 resolution. Given 4 sparse input posed images (leftmost column), we compare our high-res object-level novel-view rendering results with two baselines: Instant3D’s Triplane-LRM (Li et al., 2023) and GS-LRM (Res-512) (Zhang et al., 2024). Both our Encoder-Decoder and Decoder-Only models exhibit fewer floaters (first example) and fewer blurry artifacts (second example), compared to the baselines. Our Decoder-Only model effectively handles complex geometry, including small holes (third example) and thin structures (fourth example). Additionally, it preserves high-frequency texture details (last example).

**中文翻译：** 512 分辨率的物体级视觉比较。给定4幅稀疏已知位姿图像（最左列），与 Instant3D 的 Triplane-LRM 和 GS-LRM（Res-512）比较高分辨率渲染。两种 LVSM 相比基线具有更少漂浮物（首例）与模糊（第二例）；Decoder-Only 能处理小孔（第三例）、细结构（第四例）等复杂几何，并保存高频纹理（末例）。

### 视觉与机制

从左到右：4输入视图、Triplane-LRM、GS-LRM、Ours Encoder-Decoder、Ours Decoder-Only、GT。五行物体分别为柜架、编织篮、带孔和小部件的玩具、线框篮架、印花长椅。第一行内部区域的浮状颜色/几何伪影在 GS-LRM 红框处可见，LVSM 的内侧平面更接近 GT；第二行篮内明暗及布料边缘，对照看模糊与漂浮色；第三行小孔和部件间隙显示细小结构的可辨认性；第四行线框杆件易被错误填充或抹除，Decoder-Only 更接近参考；末行图案边缘和色块对比体现高频纹理保存。红框为作者指定比较位置，不代表误差图。

【合理推断】显式三维原语/固定渲染约束可能限制某些薄结构和视角依赖外观；直接 patch 预测能学习更灵活的成像规律。但这组定性图未控制表示以外的训练差异，不能单独证明每一视觉差异由 3D bias 造成。Tab.1 左侧 512 列才提供平均指标。

### 证据边界

这是对象 RGB 的观察而非重建 mesh 的检查；孔洞在单一目标图呈现不证明其三维拓扑正确。五个选例不足以估计总体概率。注意 Tab.1 的 GSO-512 Encoder-Decoder PSNR 低于 GS-LRM；不能把两变体都描述为所有指标领先。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
## 2.2 Figure 4

### 题注与图像

![Figure 4](LVSM/Figures/Figure_04.jpg)

来源：原论文 PDF p.8，§4.3；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 4: Scene-level visual comparison. We evaluate encoder-decoder and decoder-only LVSM on scene- level view synthesis, comparing them to the prior leading baseline methods, namely pixelSplat (Charatan et al., 2024), MVSplat (Chen et al., 2024), and GS-LRM (Zhang et al., 2024). Our results exhibit fewer texture and geometric artifacts, feature more realistic specular reflections, and are closer to the ground truth images.

**中文翻译：** 场景级视觉比较。将 Encoder-Decoder 与 Decoder-Only LVSM 和此前领先的 pixelSplat、MVSplat、GS-LRM 比较；结果显示更少纹理/几何伪影，更真实的镜面反射，更接近真实图。

### 视觉与机制

五行室内场景，每行两幅输入，随后 pixelSplat、MVSplat、GS-LRM、两种 LVSM 和 GT。首行大门/窗与地面光斑，基线出现边缘模糊及结构扭曲；第二行餐桌、椅背、窗口和桌面反射，比较倒影与可见结构是否一致；第三行厨房台面/水槽，基线有模糊和遮挡区域伪影，LVSM 台面边缘更清楚；第四行楼梯栏杆，细竖杆是易出现重复、断裂的结构；末行餐桌和窗，桌上物体/桌面边界及反射可对照 GT。GS-LRM 的红色标框突出特定失败区域。图中 RGB 画面支持视觉层面的改善，不能确定内部显式几何，亦不能定量分离材质与几何的贡献。

场景级训练使用2输入、6目标（A.2）；Tab.1 右侧使用与 pixelSplat 相同256验证协议，Decoder-Only 29.67 dB 对 GS-LRM 28.10 dB。像素重投影或 3D 原语并非唯一实现 RGB 一致性的途径，模型可经射线条件和全注意力直接拟合视角依赖变化。

### 证据边界

样本主要展示真实室内画面，不代表所有户外或动态情景；没有独立反射率或遮挡误差评估。GT 指目标 RGB，不是几何或材质真值。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
## 2.2 Figure 5

### 题注与图像

![Figure 5](LVSM/Figures/Figure_05.jpg)

来源：原论文 PDF p.10，§4.5；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 5: Zero-shot generalization to different number of input images on the GSO dataset (Downs et al., 2022). We note that all models are trained with just 4 input views.

**中文翻译：** 在 GSO 数据集上对不同输入图像数量的零样本泛化。所有模型训练时仅使用4幅输入视图。

### 视觉与机制

横轴 Input view number，纵轴 PSNR（dB，越高越好），蓝色方块虚线是 Decoder-Only，绿色圆点是 Encoder-Decoder，红色菱形点划线为 GS-LRM。横轴实际采样 1、2、4、6、8、12、16；以下为图读近似而非精确原始数据。Decoder-Only 从单图约22 dB、4图约32 dB 上升到16图约35.5 dB；Encoder-Decoder 从约21 dB 上升到6–8图约30.5 dB，随后轻微下降至约29.8 dB；GS-LRM 约20 dB 起步，6–8图约31 dB，16图约29.4 dB。仅4图是训练输入数，其余为测试时直接改变 token 数，不是重新训练。作者在 §4.5 猜测固定中间压缩约束限制多图信息聚合。

机制上输入长度 $l_x=NHW/p^2$ 随 N 增长，单流直接将所有 token 置于同一 attention，目标可从新增观测中更新；固定 latent 的编码器必须从更长的 token 序列提取同样数量的 scene tokens。新增图像理论上可能提供新的表面、遮挡和纹理证据，也可能重复/质量低，不必然改善每个样本。

### 证据边界

这是 GSO 上输入数量的 zero-shot，不是跨领域/相机误差/任意视图分布泛化。不能由曲线直接证明固定长度是下降的唯一原因；训练分布偏移、融合学习及 latent 分配亦可能影响。与 Fig.6 一起读，更多视图的质量收益伴随计算成本。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
## 2.2 Figure 6

### 题注与图像

![Figure 6](LVSM/Figures/Figure_06.jpg)

来源：原论文 PDF p.10，§4.5；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 6: Rendering FPS with different number of input images. We test the FPS on the object level under 256 × 256 resolution. We refer to rendering as the decoding process, which synthesizes novel views from latent tokens or input images.

**中文翻译：** 不同输入图像数量下的渲染 FPS。在物体级256×256分辨率测试；渲染定义为从 latent tokens 或输入图像合成新视图的解码过程。

### 视觉与机制

横轴输入视图数，纵轴 Rendering FPS（帧每秒，越高越好），绿色圆点 Encoder-Decoder 几乎固定约90 FPS；蓝色方块 Decoder-Only 单图接近97 FPS，2图约65、4图约35、6图约22、8图约16、12图约9、16图约6。灰虚线为约30 FPS 的实时参考。数值均图读近似。单图时 Decoder-Only 甚至更快；多图后显著变慢，不能简单说任意输入数下 Encoder-Decoder 都快。

【原文事实】此处只计从固定 latent 或输入图像到目标图的 decoding，不把 Encoder-Decoder 的一次编码计入 FPS。假设固定分辨率及模型宽度，Encoder-Decoder 每目标 attention 长度 $l+l_q$ 不依赖 N；Decoder-Only 为 $l_x+l_q=(N+1)HW/p^2$，dense attention 算术量含序列长度的二次项。编解码式在同一场景多目标渲染时可摊销编码，但首次延迟及输入变化时必须编码。实际 FPS 还受实现、显存带宽和硬件影响。

### 证据边界

图6没有给出完整端到端耗时和所有设备条件，不宜外推其它分辨率或硬件；曲线测量的是图像生成速度，不是 3D 重建速度。与图5共同说明质量—吞吐的权衡。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
## 2.2 Figure 7

### 题注与图像

![Figure 7](LVSM/Figures/Figure_07.jpg)

来源：原论文 PDF p.19，§A.3；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 7: Object-level visual comparison at 256 resolution. Comparing with the two baselines: LGM (Tang et al., 2024) and GS-LRM (Res-256) (Zhang et al., 2024), both our Encoder-Decoder and Decoder-Only models have fewer floater artifacts (last example), and can generate more accurate view-dependent effects (third example). Our Decoder-Only model can better preserve the texture details (first two examples).

**中文翻译：** 256分辨率物体级视觉比较。与 LGM 和 GS-LRM（Res-256）相比，Encoder-Decoder 和 Decoder-Only 的漂浮伪影更少（末例）、视角依赖效果更准确（第三例）；Decoder-Only 更好保留纹理（前两例）。

### 视觉与机制

六列分别为四输入、LGM、GS-LRM、两种 LVSM、GT。四行：格纹布/软垫（密集网格是高频纹理）；彩色食物拼盘（圆形食物、边界与细花纹）；灰色圆桌（光滑表面、腿部明暗和视角相关亮度）；米色扶手椅（内部阴影、坐垫与漂浮色）。前两行比较规律纹理是否连续及局部色块是否混淆；第三行表面光泽和桌腿形态，第四行坐垫前侧红框突出不自然漂浮/模糊。两种 LVSM 的 RGB 与 GT 视觉上更接近，Decoder-Only 细纹理保留突出。Fig.3 使用不同分辨率和基线（Triplane-LRM），不能直接把图7称同一组512降采样。

### 证据边界

作者对镜面反射的判断是 RGB 视觉对照，没有 BRDF 参数估计；对几何伪影的判断同样未验证真实三维拓扑。Tab.1 Res-256 比较支持 Decoder-Only 总体提升，但 Encoder-Decoder 在部分指标不优于 GS-LRM。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
## 2.2 Figure 8

### 题注与图像

![Figure 8](LVSM/Figures/Figure_08.jpg)

来源：原论文 PDF p.19，§A.4；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 8: Model Details. We introduce two architectures: (1) an encoder-decoder LVSM, which encodes input image tokens into a fixed number of 1D latent tokens, functioning as a fully-learned latent scene representation, and decodes novel-view images from them; and (2) a decoder-only LVSM, which directly maps input images to novel view outputs, completely eliminating intermediate scene representations. Both models consist of pure self-attention blocks.

**中文翻译：** 模型细节。Encoder-Decoder 将输入图像 tokens 编码为固定数量的一维 latent tokens（完全学习的潜在场景表示），再解码新视图；Decoder-Only 直接将输入图像映射到新视图输出，移除中间场景表示。两种模型均由纯自注意力块组成。

### 视觉与机制

图从下到上读。左侧输入 $[N,9,H,W]$（RGB+Plücker），目标 $[6,H,W]$；分割为 $[l_x,9,p,p]$ 和 $[l_q,6,p,p]$，分别线性嵌入 $[l_x,d]$ 和 $[l_q,d]$，其中 $l_x=NHW/p^2,l_q=HW/p^2$。拼接并 LayerNorm，经过24个 Self-Att Transformer layers，更新后的所有 tokens 为 $[l_x+l_q,d]$，丢弃输入对应部分，保留 $[l_q,d]$。LayerNorm、Linear_out、Sigmoid 回归 $[l_q,3p^2]$，重排成 $[H,W,3]$。目标 RGB 不是输入。

中间：Encoder 输入 $[l_x,d]$ 与 $[3072,d]$ 可学习 latent，拼接为 $[l_x+3072,d]$，12层自注意力，取更新 latent 并 LayerNorm，丢弃输入位置输出；再与目标 $[l_q,d]$ 拼接为 $[l_q+3072,d]$，12层 Decoder，丢弃 latent 位置输出，保留目标位置。默认 A.2 $d=768,p=8$；3072 是固定数量，不是 3D 网格分辨率或语义深度。

右侧：单块 pre-LayerNorm → Query/Key/Value 线性投影 → QKNorm（Q/K）→ Multi-Head Attention → 残差加；再 LayerNorm → MLP → 残差加。归一化用于训练稳定，不改变目标定义。注意 Q（attention 的查询向量）不同于 §3.1 中由相机射线构造的目标 query tokens；输入和 latent 同样产生 Q、K、V。

### 证据边界

此图给出数据流及默认层数，没给完整每个头的维度、激活及具体 QKNorm 参数，不能由其它代码猜测。右侧模块不是交叉注意力 decoder；A.1 明确名称基于输出职责。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
## 2.2 Figure 9

### 题注与图像

![Figure 9](LVSM/Figures/Figure_09.jpg)

来源：原论文 PDF p.20，§A.5；PDF 区域渲染展示副本。

**Caption（原文）：** Figure 9: Attention Mask Visualization. Both our encoder-decoder and decoder-only models employ bidirectional self-attention modules. This figure visualizes the corresponding attention masks for the various design choices discussed in Sec. 4.4. (We use green color for the columns of latent/input tokens and blue for the columns of output pixel patch tokens.) In our encoder-decoder architecture, the decoder utilizes pure self-attention, enabling latent tokens and different output target image tokens to jointly attend to each other. Consequently, latent tokens can be updated across transformer layers, while different output patch pixels can also attend to each other for joint updates. As shown in Table 3, disabling either the joint updating of output patch pixels (ours w/ per-patch prediction) or the latents’ updating (ours w/o latents’ updating) significantly degrades performance. Prior work, SRT (Sajjadi et al., 2022), eliminates both mechanisms by employing a decoder with pure cross-attention (ours w/ pure cross-att decoder), leading to even worse performance. Similarly, in our decoder-only model, disabling the joint updating of output patch pixels (ours w/ per-patch prediction) also results in a notable performance drop.

**中文翻译：** 注意力掩码可视化。两种模型均使用双向自注意力模块；图中展示§4.4不同设计对应的掩码（绿色列为 latent/输入 tokens，蓝色为输出像素 patch tokens）。Encoder-Decoder 的 Decoder 中纯自注意力使 latent 与输出 tokens 相互注意，latent 可以在层间更新，不同输出 patch 也可以联合更新。Table 3 表明关闭输出 patch 的联合更新或 latent 更新都显著降低性能。SRT 的纯交叉注意力 decoder 同时取消二者，导致更差性能；Decoder-Only 关闭输出 patch 联合更新也明显退化。

### 视觉与机制

矩阵行是 Q（更新位置）、列是 K（可读取的上下文位置，V 提供值）；绿色是 latent/input 列，蓝色是 target/output 列，灰色表示不允许。将 L 表示 latent（或单流中的输入），T 表示目标 patch。四幅掩码从左到右：

| 设计 | L读L | L读T | T读L | T读T |
|---|---|---|---|---|
| Ours bidirectional | ✓ | ✓ | ✓ | ✓ |
| per-patch prediction | ✓ | × | ✓ | × |
| pure cross attention | × | × | ✓ | × |
| w/o latents updating | × | × | ✓ | ✓ |

第一种可实现 L 与 T 双向信息传递，跨层目标状态可通过 L 间接影响其它目标位置，也可直接读取其它 T。第二种将 K/V 限于 L，T 不互相读取且不能反馈至 L；各目标独立由共享输入上下文预测。第三种只有 T 对 L 的交叉注意，L 固定且 T 互不交流。第四种保持 L 固定，但 T 可以相互注意。这里的灰色整行表示该分组不做注意更新；不能误解为对所有 $-\infty$ 的 logits 直接 softmax（那会无定义）。原文给出的掩码是可见性示意，不等于具体 kernel 实现。

### 证据边界

Tab.3 在 GSO 的 Encoder-Decoder 完整模型28.07 dB，对关闭 latent 更新26.61、per-patch 26.27、纯 cross-att decoder24.60。场景 Decoder-Only 29.67→28.98（−0.69）。这些是实际受控架构干预的结果，但 per-patch 同时改变双向反馈及目标间联系，无法独立分离两效应；并不证明 attention 学到了准确匹配或跨不同目标图一致性（这里 T 是同一目标图的 patch）。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN TABLE-01 -->
## 2.3 Table 1

### 题注与协议
原文（PDF p.6）：Table 1: Quantitative comparisons on object-level (left) and scene-level (right) view synthesis. For the object-level comparison, we matched the baseline settings with GS-LRM (Zhang et al., 2024) in both input and rendering under both resolution of 256 (Res-256) and 512 (Res-512). For the scene-level comparison, we use the same validation dataset used by pixelSplat (Charatan et al., 2024), which has 256 resolution.

译：物体级（左）及场景级（右）定量比较。物体实验在输入和渲染分辨率256与512上与GS-LRM设置匹配，场景实验使用pixelSplat的256验证集。PSNR 衡量像素误差（dB），SSIM 衡量结构相似度，LPIPS 衡量学习特征的感知距离。前两项越大越好，后一项越小越好；它们不是深度误差。

### 完整数据

| 模型 | ABO PSNR↑ | SSIM↑ | LPIPS↓ | GSO PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---:|---:|---:|---:|---:|---:|
| Triplane-LRM 512 |27.50|.896|.093|26.54|.893|.064|
| GS-LRM 512 |29.09|.925|.085|30.52|.952|.050|
| ED 512 |29.81|.913|.065|29.32|.933|.052|
| DO 512 |32.10|.938|.045|32.36|.962|.028|
| LGM 256 |20.79|.813|.158|21.44|.832|.122|
| GS-LRM 256 |28.98|.926|.074|29.59|.944|.051|
| ED 256 |30.35|.923|.052|29.19|.932|.046|
| DO 256 |32.47|.944|.037|31.71|.957|.027|

| RealEstate10K 256 |PSNR↑|SSIM↑|LPIPS↓|
|---|---:|---:|---:|
|pixelNeRF|20.43|.589|.550|
|GPNR|24.11|.793|.255|
|Du et al.|24.78|.820|.213|
|pixelSplat|26.09|.863|.136|
|MVSplat|26.39|.869|.128|
|GS-LRM|28.10|.892|.114|
|ED|28.58|.893|.114|
|DO|29.67|.906|.098|

### 差异与边界
DO 相对 GS-LRM 的 PSNR 增益：ABO512 +3.01，GSO512 +1.84，ABO256 +3.49，GSO256 +2.12，RE10K +1.57 dB。GSO512 的正文 §4.3 写作 2.8 dB，而表格 32.36−30.52=1.84 dB；这是原文不一致，报告按表复算，不擅自判定哪一个是作者实际实验结果。DO 全部报告三指标领先。ED 对比 GS-LRM 的 ABO512 PSNR +0.72，但 SSIM −.012；GSO512 PSNR −1.20 且其他两指标也差；GSO256 −0.40 dB，LPIPS 改善 .005；场景 +0.48 dB 而 LPIPS 持平。512/256 分别为训练/微调条件，不宜通过跨行差异断定分辨率本身的收益。本文没有统一控制所有基线的模型规模和训练数据，RGB 指标也不证明真实几何；应结合定性图与消融。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
## 2.3 Table 2

### 题注与协议
原文（p.9）：Table 2: Ablations studies on model sizes. The following experiments are all run with 8 GPUs and a total batch size of 64.

译：模型规模消融。下列实验均在8 GPU、总 batch size 64下运行。ED 表示 Encoder-Decoder，括号为编码层数+解码层数；DO 为 Decoder-Only 总层数。M 为百万参数。

### 完整数据
| 模型 |参数|GSO PSNR↑|SSIM↑|LPIPS↓|RE10K PSNR↑|SSIM↑|LPIPS↓|
|---|---:|---:|---:|---:|---:|---:|---:|
| ED 6+18 |173M|26.48|.901|.065|28.32|.888|.117|
| ED 12+12 |173M|25.69|.889|.076|27.39|.869|.137|
| ED 18+6 |173M|24.74|.873|.091|26.80|.855|.152|
| DO 24 |171M|27.04|.910|.055|28.89|.894|.108|
| DO 18 |128M|26.81|.907|.057|28.77|.892|.109|
| DO 12 |86M|26.11|.896|.065|28.61|.890|.111|
| DO 6 |43M|24.15|.865|.092|27.62|.869|.129|

### 差异与边界
ED 参数数固定，6+18 对18+6 改善 GSO 1.74 dB、场景1.52 dB；把容量用于目标条件处理在该设置中收益更强。DO 24 相对 6 层提升 GSO 2.89、场景1.27 dB；DO 12到18收益分别0.70/0.16，18到24为0.23/0.12；深度收益呈减小趋势但不是严格规模律。近同参数 DO24 相较最优 ED6+18 增益0.56/0.57 dB，说明非仅参数规模因素；序列成本与训练 FLOPs 并未等同。作者解释固定 latent 可能限制信息，但实验没有直接测量信息丢失量。8 GPU 结果与表1主模型资源不同，不能横向把不同表的数字当作同一训练运行。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
## 2.3 Table 3

### 题注与协议
原文（p.9）：Table 3: Ablations studies on model architecture.

译：模型架构消融。§4.4 中物体 ED 采用32 GPU、batch256、8目标视图；场景 DO 使用原主训练设置。表内每一组以本组完整模型作比较，不与表2同基线混算。

### 完整数据
| GSO（ED）|PSNR↑|SSIM↑|LPIPS↓|
|---|---:|---:|---:|
|完整|28.07|.920|.053|
|CNN tokenizer|27.59|.914|.052|
|w/o latents’ updating|26.61|.903|.061|
|per-patch prediction|26.27|.897|.072|
|pure cross-att decoder|24.60|.876|.085|

| RE10K（DO）|PSNR↑|SSIM↑|LPIPS↓|
|---|---:|---:|---:|
|完整|29.67|.906|.098|
|per-patch prediction|28.98|.897|.103|

### 差异与边界
ED 的 CNN tokenizer PSNR −0.48、SSIM −.006，LPIPS 反而降低 .001（较好），不能声称所有指标退化。关闭 latent 更新下降1.46 dB；per-patch 下降1.80；纯 cross attention 下降3.47。图9表明后者同时取消 latent 更新和目标 patch 间交换；per-patch 保留 latent 自更新但不接收目标反馈。DO 去掉目标 patch 联合处理下降 .69 dB，SSIM −.009、LPIPS +.005。该控制说明全双向交互有经验价值，却未直接测量内部几何、训练优化差异或各机制独立贡献。SRT 只作为结构设计启发，表3的 pure cross-att decoder 并非完整复现 SRT。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 2.4 图表关系

### 方法与消融
Fig.2 给总体数据流，Fig.8 展开张量和 Transformer 层，Eq.1–8 将箭头形式化。Fig.9 的四个掩码是 Tab.3 的实际干预定义：固定 latent、限制 patch 联合预测与纯交叉注意。Tab.2 则改变容量及编码/解码层数，不能与 Tab.3 混为掩码实验。读图8不能推出表3的性能，必须由训练实验支撑；同样表3不证明潜特征等于三维坐标。

### 质量与速度
Fig.1 展示目标和单图可能性；Fig.3/7 对应 Tab.1 物体512/256；Fig.4 对应 RE10K；定性图解释高频纹理、薄结构和视角相关外观等局部 RGB 差异，而平均 PSNR、SSIM、LPIPS 不解释具体失败原因。Fig.5 与 Fig.6 共同描述观测增多时的质量—解码吞吐权衡：DO 质量增长却序列增长，ED 固定 latent 的解码成本稳定但质量饱和。主实验、消融和输入数泛化共同支持两种设计在给定设置下有效；没有真实深度或三维拓扑评测，不能用三类 RGB 证据共同替代几何验证。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 3.1 研究问题

### 任务定义
【原文事实，§3.1 p.4】给定 $N$ 个带位姿的 RGB 图像 $I_i\in\mathbb R^{H\times W\times3}$ 及其外参 $E_i$、内参 $K_i$，以及目标相机 $E^t,K^t$，目标为预测 $\hat I^t$。输入图像/射线和目标射线以 patch 为单位进入共享网络；训练用真实目标 RGB $I^t$ 的 MSE 与感知损失。推理时目标 RGB 不可用，输入/目标相机参数已知，不输出显式 3D 模型。训练样本可来自不同对象/场景，权重跨场景共享，不需为每个场景重新优化。

### 欠定性
【补充理解】有限视角仅约束部分可见表面及射线，遮挡后区域、材质与光照可能有多个解释。固定相机只消除位姿未知，不消除场景推断欠定性。传统 3D bias 通过场景参数与渲染约束缩小解空间；LVSM 将许多约束交给大规模多视图训练的统计先验学习。目标相机射线仍携带几何条件，不能称模型完全不使用几何。确定性网络对于相同输入与目标只给一个 RGB 预测，不能表征全部合理假设或置信度。A.6/7 的未见区退化表明拟合可见区与恢复不可见真实信息是不同问题。核心研究问题：能否减少预定义三维表示和几何网络操作，仍高质量、高效率地从稀疏已知位姿视图合成新视图？

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 3.2 研究动机

### 两类先验
【原文，§1 p.1–2】representation-level bias：NeRF、3DGS 等指定表示与渲染，LRM 学习从图像到显式三维参数的映射。network-level bias：投影、极线特征采样、plane sweep 等预先限制信息交互。这些设计有利于几何推理，但表达、网络规模和外观建模可能受固定选择制约。作者希望通过纯 Transformer 将表示和渲染尽可能从数据学习；并非否认三维成像物理，而是改变 NVS 网络的参数化。SRT 已经采用无显式三维表示的潜在表示，但 CNN 与固定 latent/纯 cross-attention 仍限制信息流。

### 设计假设与证据
ED 以固定数量 learnable latent 压缩输入，保留可复用的场景条件；DO 移除该独立瓶颈，将 target 条件提前进入所有层。假设：大容量和训练数据可学习有利于新视图的隐式关系；减少压缩及掩码可以增加高频细节和多视图融合。Tab.1 的性能、Tab.2 的容量、Tab.3 掩码、Fig.5 输入数提供经验支持。Fig.6 同时暴露 DO 计算成本。原文并未在同一训练 FLOPs、数据量和网络规模下消除所有混杂，因此结论是有效的设计路线而非三维归纳偏置普遍无用。固定 latent 的信息容量归因是作者推测，不是信息论证明。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 3.3 相关工作

### 方法分类
依据 §2 p.2–3 的叙述（不独立读取被引论文）：

| 类别 | 核心机制 | LVSM 区别 |
|---|---|---|
| IBR/light field | 在已观测图像间以对应或光场重采样合成新视图 | 不预设插值与几何重采样操作 |
| 逐场景优化（NeRF、3DGS） | 利用多图约束优化显式或隐式场景参数，按渲染规则生成图像 | 共享权重前馈，不作逐场景优化 |
| Generalizable NeRF/3DGS（pixelNeRF、IBRNet、MVSNeRF、pixelSplat、MVSplat） | 通过跨场景学习、投影匹配或代价体等推断渲染参数 | 直接预测 RGB token，不强制几何中间接口 |
| Large Reconstruction Model（Instant3D 的 Triplane-LRM、GS-LRM、LGM） | 大数据大模型前馈预测三平面或高斯表示再渲染 | 取消显式 3D 表示及固定渲染 |
| Geometry-free（SRT 等） | Transformer 学习潜在场景表示，再基于目标射线生成像素 | ED 线性 patchify 与双向注意力；DO 取消独立场景 latent |

不能把全部基线归为逐场景优化：GS-LRM 与 LVSM 同为前馈。也不能把 SRT 和显式 LRM 等同；SRT 是最接近 ED 的思路之一。

### 生成式路线
【原文 A.6】3DiM 等概率生成路线以生成模型处理新视图，单图条件的 Zero-1-to-3 等能够生成未见区，但逐视图生成可能有不一致；多视图联合扩散试图改善。LVSM 采用确定性 MSE/感知 RGB 回归，训练/推理没有噪声逐步去除，不需扩散采样。其逐目标独立推理在可见区能产生较一致的输出，是作者观察而非跨视图误差度量；未见区仍噪声和闪烁。两类路线的关键区别是确定性条件映射与概率生成，而不是是否用了 Transformer。本文未证明概率方法一定不一致，亦未与所有概率生成方法在相同协议比较。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-01 -->
## 3.4.1 射线与 token 化

### 输入张量
每幅输入图像有 RGB 三通道与每像素 Plücker 射线六通道，拼成 $[N,9,H,W]$（Fig.8）。射线用相机内外参计算，在同一坐标系描述光线的位置与方向。将每个 $p\times p$ 非重叠 patch 展开，输入向量维数 $9p^2$，经同一个 $Linear_{input}$ 映射为 $d$ 维；每幅图 $HW/p^2$ tokens，全部输入拼接为 $[l_x,d]$，$l_x=NHW/p^2$。默认 $p=8,d=768$，每 patch 原始 576 维。没有把 RGB 压成固定二维 CNN 特征图的额外编码器；图8的线性 patchify 不是显式三维点云。

### 目标查询与序列
目标只有六通道射线 $[6,H,W]$，每 patch $6p^2=384$ 维，经 $Linear_{target}$ 得 $[l_q,d]$，$l_q=HW/p^2$。它提供希望合成的像素位置与相机条件，不是目标颜色，避免把 GT 泄漏到输入。256×256 时每图1024 tokens，4输入4096，目标1024；512 时每图4096，4输入16384。不同 patch 的射线随位置变化，提供条件，而原文未逐一说明额外位置编码。无相机误差建模；相机标定错误可能使射线与 RGB 对应错误（合理推断）。

【机制】同一场景不同图的 ray token 在共享坐标系中标注观测方向，Transformer 可学习跨视图关联，未显式用极线采样、三角测量或平面扫描，也未保证学到正确匹配。Eq.1–3 与 Fig.8 是直接依据。

<!-- PTH:END METHOD-01 -->

<!-- PTH:BEGIN METHOD-02 -->
## 3.4.2 Encoder-Decoder

### 编码
Eq.6：$[x',z]=TransformerEnc(x,e)$。初始 $e\in\mathbb R^{l\times d}$ 为跨场景共享可学习参数，默认 $l=3072$；输入 $x\in\mathbb R^{l_x\times d}$ 随观测变化。拼接后12层全自注意力，使 latent 能从全部输入读取特征，输入也能读取 latent；只取更新的 $z\in\mathbb R^{l\times d}$，丢弃 $x'$。$z$ 不含预定义 voxel/高斯/mesh 结构；它是依赖当前输入的学习场景表征，不是每场景额外训练的参数。通过固定序列长度限制后续解码输入成本。

### 解码
Eq.7：$[z',y]=TransformerDec(z,q)$。经 LayerNorm 的 z 与 target ray tokens q 拼接，12层 self-attention 后取 $y$，丢弃目标相关的 $z'$，再 Eq.4 映射 RGB。z 可复用至不同目标相机，z′ 则依赖当前 q，不能把它当目标无关场景缓存。latent 自更新意味着在 Decoder 内目标与潜变量可反复交换信息，不是固定 K/V 的纯 cross-attention。Tab.3 关闭 latent 更新或 patch 联合预测均退化。

固定宽度下编码 attention 长度 $l_x+l$；每目标解码 $l+l_q$。若渲染 M 个相同分辨率目标，总成本约 $C_{enc}+M C_{dec}$（补充分析），输入变化须重新编码。Fig.6 只报告解码 FPS。固定 latent 不是必然丢失信息的证明；Fig.5 饱和与作者的压缩解释支持值得研究的假设，而非量化信息容量上限。

<!-- PTH:END METHOD-02 -->

<!-- PTH:BEGIN METHOD-03 -->
## 3.4.3 Decoder-Only

### 单流信息流
Eq.8：$[x',y]=Transformer_{Dec-only}(x,q)$。原始输入 RGB/射线 tokens 与目标射线 tokens 从第一层拼接，默认24层全双向自注意力，目标条件可在所有层影响输入特征，输入也可更新目标；最终只取 y，经 Eq.4 预测 RGB。没有 ED 的独立固定长 latent；仍有多层隐藏表示，不能称网络没有隐状态。Fig.8 展示拼接后 LayerNorm 及输出 LayerNorm/Linear/Sigmoid。对于同场景不同目标，相同输入 x 与不同 q 进入网络，因此不能直接复用完整输入隐藏状态作为目标无关缓存。

### 联合预测
与 LLM 的因果 decoder 不同，A.1 说明命名按职责，使用全自注意力不需要逐 token 自回归。一个目标图内全部 patch 联合预测；训练多个目标图，但本文的单目标方程及 A.6 说明推理各目标独立，不能断言同时联合生成整个视频。Tab.3 的 per-patch 消融说明禁用目标位置交互造成 RE10K −0.69 dB。输入数增长时长度 $L=(N+1)HW/p^2$，attention 算术量近似二次增长；Fig.5/6 显示 DO 的额外信息与吞吐损失。隐式几何关系是可学习的假设，不是有显式深度监督的重建模型。

<!-- PTH:END METHOD-03 -->

<!-- PTH:BEGIN METHOD-04 -->
## 3.4.4 Transformer 层与掩码

### 自注意力与稳定性
Fig.8：对输入隐状态 H 先 LayerNorm，线性投影 Q/K/V，Q 和 K 做 QKNorm，再多头注意力并残差相加；第二个 LayerNorm 后 MLP 并残差相加。补充示意 $U=H+MHA(LN(H)),\ H_{out}=U+MLP(LN(U))$，其中 MHA 内含 QKNorm。注意力按内容相关性聚合不同 token，可让不同视角的像素特征交换；这里没有预定义极线注意力。QKNorm 用于控制注意力数值稳定（§4.2，A.2补充结构），具体公式及头数未在图中标注。训练用混合 bf16、梯度检查点、FlashAttention-v2/xFormers、全局梯度裁剪1，梯度范数超过5的步跳过（§4.2）；这些是计算与优化措施，不构成几何监督。

### 掩码干预
Fig.9 行 Q 列 K，用 L 表示 latent/input、T 表示 target。完整允许 L←L/T 与 T←L/T；per-patch 仅 L←L、T←L；pure cross-att 仅 T←L；w/o latent updating 仅 T←L/T。第一种 T 可直接读取其他 patch，也可经 L 跨层间接交换。第二种禁止 T 反馈给 L 与 T 间直接交流；第三种在此基础上固定 L；第四种仍允许 T 之间相互更新。灰色整行是跳过该组注意更新的图示，不等于计算一个全部为负无穷的 softmax。Tab.3：ED完整28.07 dB，禁 latent 更新26.61，per-patch26.27，纯 cross24.60。掩码结果支持完整更新有益，无法凭该实验精确分离两个路径的独立因果效应。

<!-- PTH:END METHOD-04 -->

<!-- PTH:BEGIN METHOD-05 -->
## 3.4.5 输出、监督与训练流程

### 输出与监督
Eq.4 中 y 的每个 d 维向量经 $Linear_{out}$ 生成 $3p^2$ 个标量，Sigmoid 后重排为 $p\times p\times3$ patch；所有目标 patch 按原网格排列成 RGB 图。Fig.8 显示 head 前 LayerNorm。Eq.5 $\mathcal L=MSE(\hat I^t,I^t)+\lambda Perceptual(\hat I^t,I^t)$，像素项约束目标颜色，感知项衡量学习特征的差异；A.2 指定物体 λ=1，场景 λ=.5，论文未展开具体特征层选择。无需 GT depth/pose 回归（相机是输入），无需显式体渲染监督。训练目标 RGB 只用于损失。

### 训练与推理
A.2：默认64 A100，batch每GPU8，总512，物体4输入8目标，场景2输入6目标；视图选择及相机归一化沿GS-LRM协议，本文未展开全部实现。p=8,d=768，DO24层、ED12+12、3072 latent；无 bias 线性/LN；权重初始化标准差 $0.02/\sqrt{2(idx+1)}$；AdamW β=(.9,.95)，weight decay .05（LayerNorm权重除外），cosine schedule，峰值4e−4，warmup2500。物体256训练80k步（ED4天、DO7天），512微调10k步，lr4e−5、batch128、约2.5天；场景256训练100k步（约3天），512微调20k步、lr1e−4、batch128、约3天。训练技巧见上节。推理只需带位姿观测和目标相机，前馈生成 RGB；ED先编码一次再解码，DO每目标联合处理。不声称需要微调目标场景或拥有原始训练代码。

<!-- PTH:END METHOD-05 -->

<!-- PTH:BEGIN MATH-01 -->
## 3.5.1 射线与 patch 公式

### Plücker 构造
【补充推导；原文只指定6维 Plücker ray embeddings，未给以下具体参数顺序】设外参是 world-to-camera $E=(R,t)$，$X_c=RX_w+t$，齐次像素 $u=(u,v,1)^\top$。相机中心由 $0=Ro+t$ 得 $o=-R^\top t$，方向 $d_r=R^\top K^{-1}u/\|R^\top K^{-1}u\|$。光线为 $r(s)=o+s d_r$；Plücker 矩 $m=o\times d_r$，故 $m\cdot d_r=0$ 且沿线选另一点 $o'=o+s d_r$ 有 $o'\times d_r=m$。用 $(d_r,m)$ 或 $(m,d_r)$ 表示同一有向直线；本文未明确排列及坐标归一化细节，不能把补充约定当作作者代码。若外参为 camera-to-world 则 o 和 R 的公式不同。Plücker 表示给出每像素直线条件，不给出物体深度 s 或可见性；这正是后续网络必须学习的部分。

### 公式1–3
原文 §3.1 p.4–5：设第 i 图第 j patch 的 RGB $I_{ij}\in\mathbb R^{p\times p\times3}$，对应 $P_{ij}\in\mathbb R^{p\times p\times6}$。

公式(1)：
$$x_{ij}=Linear_{input}([I_{ij},P_{ij}])\in\mathbb R^d.$$

拼接按通道后展平为 $9p^2$，矩阵 $W_x\in\mathbb R^{d\times9p^2}$；对全部 N 图的非重叠 patch 得 $l_x=NHW/p^2$（假定 H,W 可整除 p）。

公式(2)：
$$q_j=Linear_{target}(P^t_j)\in\mathbb R^d.$$

$P^t_j$ 是目标相机射线，$W_q\in\mathbb R^{d\times6p^2}$，无目标 RGB，$l_q=HW/p^2$。

公式(3)：
$$y_1,\ldots,y_{l_q}=M(q_1,\ldots,q_{l_q}\mid x_1,\ldots,x_{l_x}).$$

M 是条件序列映射；每个 y 仍 d 维。条件竖线不意味着只能用 cross-attention 或不更新输入；Eq.6–8 给具体实现。以 p=8,d=768、H=W=256、N=4 为例，输入每 patch 576维，目标384维，$x\in\mathbb R^{4096\times768}$、$q,y\in\mathbb R^{1024\times768}$。用所有像素射线保留 patch 内相机几何条件；不代表将同一3D点跨图显式对齐。图8与公式维度一致。

<!-- PTH:END MATH-01 -->

<!-- PTH:BEGIN MATH-02 -->
## 3.5.2 输出与损失

### 公式4
原文 Eq.4 p.5：
$$\hat I^t_j=\operatorname{Sigmoid}(Linear_{out}(y_j))\in\mathbb R^{3p^2}.$$

每个 d 维 y 经 $W_o\in\mathbb R^{3p^2\times d}$（图8有前置 LayerNorm）产生 logits a；$\sigma(a_k)=1/(1+e^{-a_k})$，得到 (0,1) RGB 分量。p=8时输出192个值，即64像素×3通道。1024个 patch 重组为256×256×3；不是仅预测每 patch 一个颜色，也无显式沿射线积分。参数化适配归一化 RGB，但不直接支持任意 HDR 辐射（补充理解）。

### 公式5
原文：
$$\mathcal L=MSE(\hat I^t,I^t)+\lambda Perceptual(\hat I^t,I^t).$$

A.2 物体 λ=1、场景 λ=0.5。原文引用 Johnson et al.，未给特征层和权重详细公式；不能擅指定某个具体网络。补充推导若 MSE 按 K=3HW 个通道取均值，令 $c_k=\sigma(a_k)$，则 $L_{mse}=K^{-1}\sum_k(c_k-c_k^*)^2$，$\partial L_{mse}/\partial a_k=2(c_k-c_k^*)c_k(1-c_k)/K$，感知项通过特征网络链式求导，梯度传至 head 和 Transformer。Sigmoid 饱和时梯度变小。损失仅要求目标图接近 GT，并不显式约束三维几何或介质参数。

【补充推导】单独平方损失的总体最优预测满足 $\arg\min_c\mathbb E[(c-C)^2\mid x,q]=\mathbb E[C\mid x,q]$（对 c 求导置零），当遮挡区域有多个合理颜色模式时，条件均值可能模糊。本文加感知项，不能断言实际输出正好是均值；确定性单一预测仍不能给出多模态条件分布。A.7 未见区退化与这一风险相符，但不是严格因果证据。

<!-- PTH:END MATH-02 -->

<!-- PTH:BEGIN MATH-03 -->
## 3.5.3 架构及计算

### 公式6–8
原文 §3.2 p.5：
$$[x',z]=TransformerEnc(x,e)\tag{6}$$
$$[z',y]=TransformerDec(z,q)\tag{7}$$
$$[x',y]=TransformerDec\text{-}only(x,q)\tag{8}$$

括号表示对序列轴拼接和按长度分割输出；$e,z,z'\in\mathbb R^{l\times d}$、$x,x'\in\mathbb R^{l_x\times d}$、$q,y\in\mathbb R^{l_q\times d}$。Eq6 输入不含 q，固定 x、e、权重时 z 不随目标变；Eq7 的 z′ 随 q 变且丢弃。Eq8 的 x′ 可以受 q 影响。ED 的映射 $M(q|x)=D(q,Z(x))$ 显式经过固定长度 Z，DO 无此因子分解要求；这不意味着 DO 无任何压缩，也不证明 ED 信息必定丢失。

### 注意力复杂度
补充推导：对长度 L、宽度 d 的状态 H，第 h 个 head 使用 $Q=HW_Q,K=HW_K,V=HW_V$；QKNorm 后以 $A=softmax(\tilde Q\tilde K^\top/\sqrt{d_h}+B)$ 聚合 $AV$（标准 attention 示意，具体 QKNorm 细节原文未展开）；mask B 禁止的项置为负无穷，图9整行跳过不能直接做无定义 softmax。QK^T 产生 L×L 打分，全部头约 $O(L^2d)$；投影与 MLP 约 $O(Ld^2)$（宽度倍数固定）。FlashAttention 减少内存读写和显存中间量，不消除 dense attention 算术二次项。

256，N=4，p=8，l=3072：ED encoder L=7168（12层），每 target decoder L=4096（12层）；DO L=5120（24层）。以 L² 为每层注意力打分规模代理，DO 单层26.21M，ED decoder 16.78M；编码51.38M 单层。若有 M 个目标，则 ED 的代理总量 $12(7168)^2+12M(4096)^2$，DO 为 $24M(5120)^2$。M=1 时前者约818M，后者629M；M=2 时前者约1019M，后者1258M，说明一次编码的摊销重要。N=16时 DO L=17408（303.04M/层），ED 解码仍16.78M/层，编码序列变长。真实延迟也含线性层、硬件及目标 batch，不能用此代理直接预测 Fig.6 FPS；原图只测解码。512每图 patch 数是256的4倍，固定 N 时 DO 长度4倍、二次项16倍；固定 latent 的 ED 不等比增长。以上为理论算术分析，不是论文报告的 FLOPs 数字。

<!-- PTH:END MATH-03 -->

<!-- PTH:BEGIN EXP-01 -->
## 3.6.1 数据与主实验

### 训练和评估
§4.1–4.3、A.2：对象级用 Objaverse 约730K对象，每对象32个随机视角；测试为 GSO 1099对象与 ABO 1000对象，按 GS-LRM 的4输入、10目标视角协议，256及512输入/输出设置。场景级 RealEstate10K 含约80K短片、来源约10K YouTube视频，按 pixelSplat 训练/验证划分，主比较256分辨率、2输入。物体、场景模型分别训练，不能解读为一个共享权重同时泛化两域。训练目标数对象8、场景6，与物体测试的10不是同一量。原文并未评价水下、动态/未知位姿任务。

### 结果与限制
Tab.1 DO 对 GS-LRM 的 PSNR：ABO512 32.10 vs29.09（+3.01），GSO512 32.36 vs30.52（+1.84，正文2.8存在不一致），ABO256 32.47 vs28.98（+3.49），GSO256 31.71 vs29.59（+2.12），RE10K 29.67 vs28.10（+1.57）。SSIM/LPIPS同向改善；ED 表现混合（GSO512 三指标低于GS-LRM）。Fig.3/7 支持细结构、纹理和反光质量，Fig.4 支持室内场景视觉表现。PSNR在相同颜色范围下与均方误差呈对数关系；+3 dB 约代表 MSE 减半的单一图像/统一MSE口径，不可直接由平均PSNR差推出全数据像素MSE精确倍率。没有几何真值误差、置信度或遮挡区分层指标；不能将 RGB 指标领先直接写成几何重建领先。基线训练资源、表示、数据细节不完全控制。

<!-- PTH:END EXP-01 -->

<!-- PTH:BEGIN EXP-02 -->
## 3.6.2 训练资源与规模

### 训练资源
§4.3：RealEstate10K 的 LVSM-small（DO6层、batch64）单 A100 80GB 训练7天，27.66/.870/.129，对 MVSplat 26.39/.869/.128：PSNR +1.27、SSIM+.001、LPIPS差 .001。双GPU 7天 DO12层、batch64 得28.56/.889/.112，对GS-LRM 28.10/.892/.114：PSNR+.46、LPIPS改善.002、SSIM低.003。作者强调学术计算资源的潜力，但不能写作单/双卡全部指标优于基线。默认DO24层64 A100、batch512，不同训练配置；对象 DO 7天而 ED4天，场景两者约3天（A.2）。未实测用户机器训练成本。

### 模型规模
Tab.2 的8 GPU、batch64控制：173M ED 6+18 比18+6 GSO提升1.74、RE10K1.52；171M DO24 高于 ED6+18 各0.56/0.57。DO6/12/18/24 参数43/86/128/171M，GSO PSNR24.15/26.11/26.81/27.04，场景27.62/28.61/28.77/28.89，SSIM与LPIPS同步改善；规模收益边际递减但有限取样不能拟合一般 scaling law。作者猜更深 encoder 让压缩更难学习；可能还受有效计算、优化及目标处理深度影响。DO 和 ED 参数近似不等于 FLOPs 相同；需匹配计算、训练数据/步数才能作更强因果比较。

<!-- PTH:END EXP-02 -->

<!-- PTH:BEGIN EXP-03 -->
## 3.6.3 结构消融

### 控制条件
Tab.3 / §4.4：ED物体级32GPU、总batch256、8目标，DO场景级原设置。原文称ED目标数“decreased ... 8”，A.2默认也为8，故记录实际8而不推断更早配置。完整ED 28.07/.920/.053；CNN 27.59/.914/.052；无latent更新26.61/.903/.061；逐patch26.27/.897/.072；纯cross24.60/.876/.085。DO完整29.67/.906/.098、逐patch28.98/.897/.103。CNN LPIPS略优是例外，不能称线性 tokenizer 所有指标更佳。

### 机制解释
线性 patchify 让模型直接学习对原始像素和射线的映射，作者选择不引入 CNN 特征提取；CNN 版本是特定实现对照，不证明所有 CNN 都差。latent 更新允许目标条件回写，再供其他目标patch读取；T↔T 直接交流支持共同预测纹理/边缘。Fig.9 的 per-patch 同时删除 T→L 反馈和 T间交换，因此其 −1.80 dB 不可只归因于目标自注意；关闭 latent 更新 −1.46 dB，而纯cross同时删除两机制 −3.47 dB。DO场景per-patch −.69 dB，证明该任务下完整单流有益。这里的 patch 皆属同一目标图，不能把消融解释为多目标跨时间的一致性训练；真实对应、可见性、深度仍未显式度量。

<!-- PTH:END EXP-03 -->

<!-- PTH:BEGIN EXP-04 -->
## 3.6.4 推理泛化与限制

### 视图数与速度
Fig.5 GSO 所有模型4输入训练，直接在1/2/4/6/8/12/16输入测试。DO PSNR约22→35.5 dB，ED与GS-LRM约6–8输入后饱和或下降；作者猜测固定中间表示影响信息聚合。Fig.6 物体256解码FPS：ED近90，DO随输入从约97降至约6，4输入约35，8约16，单图DO可能比ED快。ED一次编码多目标摊销，不等于首次总延迟。DO 更长序列的质量收益与运行成本是同一设计取舍。输入更多并不必然为每个目标提供有效的新信息；实验并未控制新增观测的覆盖、基线与画质。

### 泛化边界
A.6：作者观察确定性逐目标独立预测在已观测区域具有较好的一致性，概率生成路线可处理未见区域；A.7：未见区域存在噪声、闪烁、固定图案，作者建议与生成模型结合。未见区域的真实内容从有限观测一般不可唯一识别，不等同于“视觉合理”。图1单图示例不证明模型真实恢复深度。A.7 还讨论训练之外的分辨率/长宽比，由512×512训练转为512×960输入时，目标图中心较好而横向边界模糊；作者将中心裁剪导致长边边缘射线密度低作为可能解释，建议扩大训练数据多样性。没有跨水下域、未知相机及任意观测数的证据。可扩展性结论限定在测试的对象和场景协议，论文未证明无限视图或分辨率外推。

<!-- PTH:END EXP-04 -->

<!-- PTH:BEGIN SYNTHESIS-01 -->
## 4.1 贡献证据矩阵

### 主张—证据矩阵
| 主张 | 原文证据 | 支持程度/替代解释 |
|---|---|---|
| 无预设显式3D表示也可NVS | Eq.1–8、Fig.2/8、Tab.1 | 直接定义及RGB实验支持；保留相机射线 |
| DO更高质量 | Tab.1、2；Fig.3/4/7 | 所报告主比较支持；训练计算和表示差异未全部控制 |
| 固定latent可能限制多观测融合 | Fig.5、Tab.2 | 间接支持；训练外N、优化、容量亦可能影响 |
| 全自注意力有益 | Fig.9、Tab.3 | 受控结构消融，部分掩码同时改变多个路径 |
| ED重复解码高效 | Fig.6及Eq.6–7 | 仅解码，输入固定时编码可摊销 |
| 小模型降低计算门槛 | §4.3 | 单双卡特定训练与PSNR竞争力，SSIM/LPIPS有例外 |
| 可见区域较一致 | A.6 | 作者定性观察，未独立验证视频或跨图几何误差 |

### 技术边界
LVSM 是已知相机、确定性、多场景前馈 RGB NVS，不是相机估计、三维场景恢复或概率生成。3D bias 的减少属于网络/表征选择，不意味着成像物理失效；给定射线仍含几何条件。ED和DO 的真实质量取决于数据、优化、场景内容及观测分布。严格研究几何、物理真实性、跨域或未见区域可信度必须引入相应真值/独立观测或不确定性验证，不能仅以图像好看替代。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END SYNTHESIS-01 -->

<!-- PTH:BEGIN SYNTHESIS-02 -->
## 4.2 局限与覆盖审计

### 原文局限与未证实
作者明确未见区域噪声/闪烁、分辨率长宽比外推不稳（A.7），以及确定性与生成式方法差异（A.6）。独立评价：少观测下信息不足是根本限制；确定性回归可学习有效统计先验但无法保证未见区域真实。训练资源在主模型中仍较高，DO 随输入数显著降低吞吐；固定 latent 可能牺牲质量但非普遍必然。未实测重建几何、材质、相机鲁棒性、任意域和长视频一致性。没有复现实验，不能称论文结果已独立验证。

### 覆盖与来源
本报告依据实际 arXiv:2410.17242v2 PDF（21页），覆盖摘要、引言、相关工作、§3 Eq.1–8、§4五个实验子节、结论及附录A.1–A.7；参考文献 p.11–17 用于定位引用，未精读外部文献。9图逐图题注、译文、图像、视觉与机制和边界，3表全数据及差异；8个原文编号公式与补充推导明确区分。图像原始区域PDF保留，JPEG为展示副本，图5/6裁切修正后验收；4图存在实用保真记录，不宣称像素级无损。Source无内部版印，HTML装饰资源缺失；均以PDF为准。GSO512正文/表格差异及CNN LPIPS例外保留。官方代码是重新实现版本，未运行代码或网站视频。以上覆盖是生产自检，非独立Review。

本单元承担来源、叙事或定性证据核查，无独立公式推导对象；数学机制见第3.5节。

<!-- PTH:END SYNTHESIS-02 -->



---

# 第二部分：学习与研究笔记

# LVSM 学习与研究笔记

> 精读报告 SHA-256：84a321d224c41272731e9927150f31d78b63e0d18d9607d7ddda6f7819ca992b
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：核心概念内化

### 1. 几何条件不等于三维表示
原文 Eq.1–2 以 Plücker 射线描述每个像素在共享相机坐标系的几何关系；它没有把未知深度作为输入。给定一条线，任何线上点都与同一像素位置兼容；多个视角的对应关系、遮挡与材质必须通过输入内容和学习先验推断。显式 3D representation 则给每个空间位置或原语某种可查询属性，并以已知渲染机制把它投影为 RGB。LVSM 取消后者的硬约束，不取消前者的条件。反事实：去掉射线只保留 RGB，两个不同目标相机将没有不同的条件信息（若没有其他相机编码），同一确定性模型无法区分两个目标。这说明相机信息是任务定义的一部分，而非可无条件删除的先验。

### 2. 固定表示与目标条件化
精读 §3.4.2–3：令 X 是所有观测、q 是目标条件。ED 规定 $F(X,q)=D(Z(X),q)$，Z 是固定长度场景摘要；DO 只要求 $F(X,q)$ 本身。若两个观测集 $X_1,X_2$ 在 Z 中不可区分而正确目标图不同，任何确定性 D 都无法同时预测正确。这是条件性的反例，不是“3072一定容量不足”的证明。理论上充分统计量 Z 可保存所有与任务有关的信息；实际固定长度和有限训练可能限制信息选择。Fig.5 的ED饱和、DO持续提升是可检验线索。可变长 latent、分层缓存或按 q 检索是可能的替代设计，但不是原文实现。缓存与直接计算的差异可从 Eq.6–8 的目标依赖关系理解：若 q 每次变化，ED 的 z 保持，DO 的输入隐藏状态通常不能完全保持。

### 3. 观测与先验的互补
【补充推导】令未知场景 S、观测 X、成像算子 A，$X=A(S)+\epsilon$。如果 $A(S_1)=A(S_2)$ 但目标视图 $B_q(S_1)\ne B_q(S_2)$，目标任务不可辨识，增大网络容量本身不能制造额外观测证据。先验 p(S) 可以改变条件分布 p(S|X) 的权重，但不能证明选择的场景真实。MSE条件最优是目标RGB均值；本文额外感知损失使预测不必等于均值，仍为单一确定性输出。生成式模型可以采样多种解释，但概率生成不自动意味着真实或已校准。原文A.6–7 的可见区域一致与未见区伪影符合该区分；未直接评价不确定性。应区分已观察到的区域、具有多视图互证的区域、低覆盖和完全未观察区域。

### 4. 双向注意力与代价
Fig.9 的 L、T 是分组 token，不是物理光线传播过程。L→T 表示目标读取观测，T→L 是目标改变上下文，T→T 使目标patch互相更新。与单次读取不同，多层传播可形成 q 条件下的迭代特征交互；却不保证每个头承担某一真实物理量。Tab.3 的掩码改变多个路径，不能直接计算各路径的独立贡献。精读 §3.5.3 的算术代理在4输入256下 ED 首目标约818M打分项，DO约629M；两目标 ED约1019M vs DO1258M；说明缓存节省与目标数有关而非必然首帧更快。算术项只是代理，实测需分解预处理、编码、解码和内存。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Observation（带位姿图像与射线条件）
- **Keywords:** Sparse-view NVS; Posed Images; Plücker Rays; Patch Tokenization
- **Highlight:** LVSM 将输入RGB和六维射线合并为输入token，将目标射线单独token化；目标RGB仅作监督，未要求显式深度。
- **Evidence:** §3.1 pp.4–5 Eq.1–5，Fig.2/8；精读§3.4.1。
- **Connection Points:** 观测数量、相机已知/未知、ray/pose encoding、目标条件表达。
- **Open Questions:** 相机误差和不均匀基线对纯学习关联的影响是否可被测量或校准？

### Topic: Scene Representation（固定场景latent与目标条件直连）
- **Keywords:** Latent Scene Representation; Information Bottleneck; Decoder-Only; Target-conditioned Attention
- **Highlight:** ED 以3072个latent承载目标无关场景信息，DO 取消独立瓶颈，使目标条件参与所有层；后者在GSO更多输入时继续增益。
- **Evidence:** Eq.6–8，Fig.2/5/8，Tab.2。
- **Connection Points:** 表示容量、显式/隐式、目标无关/条件化缓存、数据规模。
- **Open Questions:** 在匹配计算和训练数据下，是否存在可变容量或查询式缓存同时保留多观测质量与吞吐？

### Topic: Inference & Optimization（确定性联合patch预测）
- **Keywords:** Bidirectional Self-Attention; Per-patch Prediction; Cross-Attention; Feed-forward Regression
- **Highlight:** 全自注意允许目标patch联合更新及latent回写；GSO关闭latent更新、逐patch和纯cross分别下降1.46、1.80、3.47 dB，但部分掩码同时改变多条路径。
- **Evidence:** Fig.9、Tab.3，§4.4 p.9；精读§3.4.4。
- **Connection Points:** 信息流方向、目标间联合处理、结构消融与公平对照。
- **Open Questions:** 如何分离直接T→T与经L间接交换的贡献？

### Topic: Efficiency & Scalability（输入数泛化与解码代价）
- **Keywords:** Input-count Generalization; Attention Complexity; Cached Latents; Rendering FPS
- **Highlight:** 4视图训练后GSO测试1–16视图，DO质量继续增长却FPS下降；ED固定latent解码约90 FPS，图6未计编码。
- **Evidence:** Fig.5/6 p.10，Eq.6–8。
- **Connection Points:** 观测数、吞吐/延迟、缓存、计算预算、分辨率。
- **Open Questions:** 以新增视图的有效覆盖而非数量衡量质量收益是否更稳定？

### Topic: Observability & Identifiability（未观测区域与确定性预测）
- **Keywords:** Unobserved Regions; Deterministic NVS; Generative Prior; View Consistency
- **Highlight:** 作者报告可见区域可独立生成较一致的新视图，未见区域仍有噪声和闪烁；未显式输出不确定性或几何真值。
- **Evidence:** A.6–7 pp.20–21，Eq.5。
- **Connection Points:** 观测支持域、确定性/概率预测、真实几何与感知质量、置信度。
- **Open Questions:** 能否在观测支持区约束证据一致性，而在无证据区输出校准的多种解释？


## RESEARCH-DIRECTION：一般研究分析

本次未指定面向某项新研究的独立任务，本节只以论文的开放问题为一般研究分析，不设定用户的具体研究方向。

### 任务层级
| 层级 | 本文定位 |
|---|---|
| Field | 计算机视觉、计算机图形学、多视图学习 |
| Task | Novel View Synthesis |
| Task Setting | 稀疏带位姿 RGB、目标相机已知、跨场景前馈 |
| Research Problem | 减少预设三维表示和几何网络操作后，能否兼顾高质量、输入数泛化与效率？ |
| Method | Plücker token 化、纯 Transformer、ED/DO、光度+感知监督 |

这里的任务并不要求显式深度，不能把“减少3D先验”移植成所有几何任务的目标。对于需要可测量三维结构的应用，应重新定义输出和评价标准。

### 可迁移机制与条件
相机射线 token 可为不同任务提供统一的坐标条件，但需要可靠标定；如果输入位姿未知，必须联合估计或建立对相机误差鲁棒的表示，这是额外任务。全注意力能在不指定匹配规则时聚合跨视角特征，但要求足够数据和计算，存在分布移位风险。固定场景 latent 支持快速查询，可用于多目标共享上下文；DO的目标条件化可用于针对目标选择观测证据，但长序列代价要求有效信息筛选。对物理成像发生变化的场景，射线相同不意味着颜色一致；介质、光照、材质、遮挡差异可能混淆跨视图关联。本文未验证该类域，不能直接宣称迁移有效。

### 证据、先验和可靠性
Fig.5只改变输入数量，未分离观测质量、覆盖、角度及遮挡；因此应把数量 N 与有效信息量分开。对几何或物理参数的任务，需要独立GT或新观测检验。确定性 RGB 输出并不提供置信度；如果要求可靠推断，可引入观测支持分层、可预测误差校准或多个合理假设，并测试可靠性，不只是提高PSNR。研究路径可以在保留先验约束与学习灵活性之间找条件性权衡，不把任何一种结构预先认定为最优。


## RESEARCH-PROBLEMS：可证伪的研究候选

以下为基于本文现象的研究假设，未进行跨文献新颖性审查，也不声称原创或已验证。优先级按当前论文直接证据强度与控制实验可行性排序，不等同项目立项。

### RP1：固定长度场景表示何时限制新增观测的利用？（优先）
**现象与未知：** Fig.5 的 ED/GS-LRM 在6–8输入附近饱和，DO 仍上升；Tab.2 增深编码反而下降。未知是瓶颈容量、训练分布偏移还是训练优化导致。**假设：** 在相同训练视图分布与计算预算下，新增视图带来独立几何/外观证据时，固定latent长度不足会显著限制有效利用率，增加或自适应选择latent可缓解；若新增仅重复视图，则无等量改善。

**验证：** 在有GT几何的对象数据上按N、基线、可见面覆盖分层；控制网络宽度、训练样本、参数/训练FLOPs与目标深度；比较固定latent长度扫描、可变长度、同计算DO、重复视图对照。记录 PSNR/SSIM/LPIPS、可见/新揭示区域误差、输入删减敏感性、完整编码/解码延迟和峰值内存。若在匹配资源与训练分布后固定长度的饱和仍可被增加latent一致缓解且集中于新覆盖区域，支持假设；若容量变化不改善而视图数混合训练消除退化，反对容量主因。风险：匹配FLOPs与参数不能同时完全一致，需双轴报告；评估几何时必须额外定义深度读出或跨图对应，避免将RGB当几何GT。

### RP2：能否分离观测证据与学习先验对新视图可信度的贡献？
**现象与未知：** A.6–7 可见区一致而未见区噪声/闪烁，Fig.1单图能合成新视图。未知在稀疏观测下单一预测的高质量是否反映真实信息。**假设：** 观测支持度是预测误差的一个可校准因素；在无法由输入唯一确定的区域，明确不确定性比强制单一RGB预测更能表征风险。

**验证：** 构造同样输入而遮挡背面不同的成对场景（确保输入图真正一致），逐步添加能看到差异的视角；用GT几何投影定义覆盖mask，比较确定性基线、具置信头/概率输出的同容量模型，另设仅改训练数据的对照。测量分区RGB误差、跨目标重投影误差（GT几何）、预测区间覆盖率/误差排序、校准误差及多假设覆盖。支持条件：证据不足时区间扩大并有正确覆盖，新独立观测后收敛；若置信度仅追随颜色纹理或模型规模，不随可见性变化，则假设中的支持度机制不足。风险：真实数据可见性和GT不确定；合成数据可能偏离实际分布。生成概率不自动等于真实后验，需要实际校准实验。

### RP3：目标条件化上下文能否以更低成本保留长序列收益？
**现象与未知：** Fig.5–6显示DO质量随N增长、解码吞吐下降；ED可复用场景latent。**假设：** 针对目标射线选择少量有效输入token，并保留必要的T↔T和L↔T更新，可以在给定质量容差内降低完整延迟。**验证：** 在相同N、目标分辨率和设备上比较全DO、ED、随机token丢弃、几何可见性检索、学习检索；匹配训练数据与预算，分别测预处理/编码/首帧/多目标延迟、峰值内存、RGB质量及遮挡边缘误差。设目标数1/2/8/视频长度；预先设定质量容差及延迟目标（须根据预实验和实际应用确定，不虚构达成值）。若检索对新揭示区域导致不可接受的误差或开销抵消节省，否定候选设计。风险：几何检索本身引入先验，可能在错误位姿或复杂外观下失败。

三候选需要进一步核查可变latent/长序列高效注意力、稀疏视角不确定性和观测选取相关文献，不能仅据本篇声称空白。RP1直接对应两个实验，较易开展控制；RP2科学价值较广但需要可信GT和校准；RP3工程约束明确但创新与代价需核实。


## INTERNALIZATION：知识内化与自检

### 用自己的语言重建论文逻辑
已知相机条件的稀疏NVS不一定先回答“场景在三维空间是什么”，也可以直接回答“给定这些观测，目标射线上的RGB是什么”。LVSM把输入图像和相机射线变成token，把目标射线也变成token，用全注意力联合求解目标patch。ED先把场景压缩成固定latent，之后反复解码；DO允许目标更早参与对原始观测的处理。RGB+感知监督负责让预测接近真实目标，而不是显式训练深度或体渲染参数。质量结果表明取消独立latent的DO在报告协议更优，视图数实验说明它更能利用训练外多观测；代价是每目标计算随观测增长。对完全未见区域两者都没有真实性保证。

### 技术地图
观测条件（I、K、E）→ Plücker射线 → 输入RGB+ray patch/目标ray patch → ED固定scene latent或DO直接拼接 → 全双向自注意 → y目标token → 线性/Sigmoid/重排 → 目标RGB → 训练时与GT计算MSE+感知。逻辑链中的三个关键关系分别是：研究逻辑（减小表示约束）、信息流（token如何交换）、监督关系（GT只作用于输出损失）。不能把注意力权重解释为可测物理透射率或深度概率。

### 理解自检
| 问题 | 正确理解 |
|---|---|
| 没有3DGS/NeRF就是没有几何吗？ | 否，已知相机和Plücker射线是几何条件；网络可能学习隐式几何。 |
| Decoder-Only是否因果自回归？ | 否，全双向自注意，所有目标patch联合预测。 |
| ED latent e、z、z′一样吗？ | e是共享可学初值，z随观测变化、目标无关，z′随目标变化且丢弃。 |
| 目标patch间联合是否等于多目标视频一致？ | 否，原文单目标输出，A.6逐视图独立推理。 |
| 图6是端到端FPS吗？ | 否，只计解码；ED需另算首次编码。 |
| DO更好是否证明任意几何任务更好？ | 否，主比较是RGB指标，真实深度/拓扑未评估。 |
| 单图能预测新视图是否证明真实遮挡区恢复？ | 否，未知部分依赖统计先验且可能失败。 |

### 尚未解决
需要区分固定latent容量与训练分布效应，建立有效观测信息量指标；独立测试图像合成和几何准确性之间的关系；评估相机误差、长宽比分布和置信度。本文提供了有价值的基线与反例，却未回答上述问题。精读§4.1主张证据矩阵是后续研究评价的入口。


## GLOSSARY：术语表

| English | 中文 | 技术含义及关联 |
|---|---|---|
| Novel View Synthesis (NVS) | 新视角合成 | 已知观测及目标相机预测目标RGB；不同于直接三维重建。 |
| Posed Images | 已知位姿图像 | 图像含相机内外参；不等于已知场景深度。 |
| Inductive Bias | 归纳偏置 | 学习中预设的结构偏好；本文区分表示层与网络层。 |
| Plücker Ray Embedding | 普吕克射线嵌入 | 六维有向线表示，常用方向和矩；Eq.1–2输入，补充推导见精读§3.5.1。 |
| Patch Tokenization | 图像块token化 | 不重叠patch展平线性映射，输入9p²维、目标6p²维。 |
| Target Query Token | 目标查询token | 目标射线的d维表示；不同于注意力矩阵Q。 |
| Latent Scene Representation | 潜在场景表示 | ED的目标无关场景token z，固定3072；与显式3D原语不同。 |
| Encoder-Decoder / Decoder-Only | 编码器–解码器 / 仅解码器 | Eq.6–8两类映射分解；命名非因果mask。 |
| Bidirectional Self-Attention | 双向自注意力 | 拼接序列所有允许位置相互读取；与纯cross-att区别见Fig.9。 |
| QKNorm | 查询/键归一化 | 稳定注意力训练，§4.2；不是几何归一化。 |
| Per-patch Prediction | 逐块独立预测 | Fig.9限制target间交流与反馈；不等同仅输出一个像素。 |
| Deterministic Prediction | 确定性预测 | 给定输入条件输出单一结果；不同于条件分布采样。 |
| Perceptual Loss | 感知损失 | 依据特征差异补充像素MSE；具体实现细节未详述。 |
| PSNR / SSIM / LPIPS | 峰值信噪比 / 结构相似性 / 学习感知距离 | 主RGB评价，↑/↑/↓，不直接度量深度。 |
| Observability / Identifiability | 可观测性 / 可辨识性 | 补充概念：观测是否携带有效信息，未知量是否可唯一确定。 |
| Zero-shot Input-count Generalization | 输入数量零样本泛化 | 4输入训练、测试不同N（Fig.5）；不是任意域泛化。 |
| Rendering FPS | 渲染帧率 | Fig.6仅指解码，不计ED编码。 |

术语中的可观测性/可辨识性为教学拓展，论文未提出对应显式估计器；Plücker具体排列以实现约定为准，不把补充推导当作者代码。


## SOURCES：来源与证据说明

### 实际来源
论文 LVSM: A Large View Synthesis Model with Minimal 3D Inductive Bias，arXiv:2410.17242v2（2025-04-02），ICLR 2025。用户提供原始资源 ZIP：`arxiv_2410.17242v2.zip`，SHA-256 `7354690df60ec9ff050936c14fe49bdfa274b1ce6c9dc40d445a03acbce54290`。实际PDF共21页，PDF为最终科学证据；LaTeX Source辅助公式/宏，内部无独立版本印记；HTML辅助定位，部分外部装饰缺失。Figure源自PDF区域，保真PDF及展示图保留。官方[论文](https://arxiv.org/abs/2410.17242v2)、[项目主页](https://haian-jin.github.io/projects/LVSM/)与[代码](https://github.com/Haian-Jin/LVSM)仅用于身份和资源核验；代码为重新实现版本，未运行，不宣称验证实验或网站视频。

冻结精读报告 SHA-256：`84a321d224c41272731e9927150f31d78b63e0d18d9607d7ddda6f7819ca992b`。本笔记只依赖当前论文及其冻结报告；精读覆盖9图、3表、8个原文编号公式、正文与附录。原文GSO512 PSNR增益2.8dB与Tab.1复算1.84dB冲突，保留差异；CNN tokenizer LPIPS例外同样保留。

### 证据分类与未证实项
论文定义、公式、数值为原文事实；Plücker约定、attention算术量、不可辨识性反例、MSE条件均值等为明确的补充推导；固定latent容量解释是作者假设或后续可检验推断；研究候选、优先级、实验设计为独立评价，不是论文成果。CROSS-PAPER仅为潜在知识关联索引，跨论文独立分析与新颖性审查延期；没有读取相关论文全文。无几何GT验证、无外部域迁移实证、无概率校准、无复现实验。本笔记不包含私人研究方案和敏感信息。



