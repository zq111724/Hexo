---
title: "FrameCrafter: Novel View Synthesis as Video Completion（完整精读与研究笔记）"
date: 2026-11-27 04:08:39
index_img: "/FrameCrafter/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/FrameCrafter/)

# 第一部分：论文精读报告

# FrameCrafter 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2604.08500v2（2026-08-31）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2604.08500) | [PDF](https://arxiv.org/pdf/2604.08500v2) | [Project Page](https://frame-crafter.github.io/) | [Code](https://github.com/szqwu/FrameCrafter) | [Model](https://huggingface.co/szqwu/FrameCrafter)

<!-- PAPER-METADATA END -->

> 论文：FrameCrafter: Novel View Synthesis as Video Completion
> 来源：用户上传 arxiv_2604.08500v2.zip，PDF 24 页，正文、参考文献、附录 A–C。
> 同版 PDF 为权威；Source 与 HTML 为辅助证据。图像为 PDF 区域展示 JPEG，并非原件。
> 证据分为原文事实、背景知识、合理推断、独立评价、尚未证实。

<!-- PTH:BEGIN INIT -->
## 0. 自动初始化

### 来源与范围

【原文事实】Qi Wu、Khiem Vuong、Minsik Jeon、Srinivasa Narasimhan、Deva Ramanan（Carnegie Mellon University）。论文为 arXiv:2604.08500v2，2026-08-31，ECCV 2026 接收。实际 PDF 首页标记与标题、作者匹配。来源为用户上传的 ZIP；权威 PDF 共24页，均可读取。正文 p.1–15（摘要、§1 引言、§2 相关工作、§3 方法、§4 实验、§5 结论），参考文献 p.15–19，附录 p.20–24（A 补充定量实验、B 定性实验、C 实现细节）。Source 安全解包18项，标题、作者、图表引用与 PDF 对应，但无独立 arXiv 版本印记。HTML 可辅助检索，字体资源缺失不影响 PDF；不以其替换原文。未独立阅读所引用文献。官方链接仅用于身份核验。

### 完整图表目录

| 编号 | 页码 | 题注主题/原文定位 | 内容范围 |
|---|---:|---|---|
| Figure 1 | 1 | Video Diffusion as Multi-view Prior | 正文概览 |
| Figure 2 | 5 | 冻结 VAE、射线条件与 DiT | 方法总览 |
| Figure 3 | 6 | 因果 VAE 与独立编码比较 | §3.2 |
| Figure 4 | 12 | Qualitative comparison under sparse inputs | §4.3 |
| Figure 5 | 14 | 训练场景数量对性能影响 | §4.4 |
| Figure 6 | 23 | 附加定性对比 | §B |
| Figure 7 | 24 | In-the-wild qualitative results | §B |
| Table 1 | 10 | 6-view NVS 双基准定量 | §4.2 |
| Table 2 | 11 | 192×336 消融 | §4.4 |
| Table 3 | 13 | 10 次随机排列 | §4.4 |
| Table 4 | 14 | 有序评估下训练设置 | §4.4 |
| Table 5 | 20 | 3-view NVS 双基准 | §A.1 |
| Table 6 | 21 | Flexible multi-target prediction | §A.2 |
| Table 7 | 22 | 计算时间与基线 | §A.3 |
| Table 8 | 22 | 下游三维重建一致性 | §A.4 |

正文5图4表，附录2图4表，共7图8表；无独立编号 Algorithm。图像路径为 `FrameCrafter_精读报告_assets/Figure_01.jpg` 至 `Figure_07.jpg`，均为 PDF 区域渲染 JPEG。每个图均已实际比较源 PDF 区域、保真区域 PDF 和展示副本。Figure 1/2/3/5 的保真区域回渲染非精确，但尺寸和可识别源流保持且视觉通过；其余3图精确通过（在检查范围内）。完整题注见后续逐图表单元及原 PDF。此单元仅整理资源，不涉及机制推导或数学分析。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1.1 标题

### 术语与任务

**FrameCrafter：将新视角合成视为视频补全。** Novel View Synthesis（NVS）指给定已有视图及摄像机条件，生成场景在另一指定相机位姿下的图像。Novel 表示相机视角未被直接观测，不意味着内容必然新颖。Video Completion 通常指依据已知视频信息预测缺失帧；本文的关键是把相机姿态不同的稀疏图像视为广义的低帧率视频观测，并利用预训练视频生成模型填补指定查询图像。这个比喻并非把 NVS 等同于自然时间插帧：原始视点可能相距很远且不构成连续轨迹。

### 标题的证据边界

【原文事实】标题（p.1）只承诺从视频补全视角重新表述 NVS。结合摘要才明确输入为约5张有位姿图片，目标为一张指定查询视图，且输入为无序集合。单从标题无法确定是显式重建还是直接生成、是否需要相机标定、具体扩散骨干、训练数据量、能否多目标联合生成、是否使用时间位置编码。后续 §3 和附录 A.2 分别说明这些问题。这里属于术语与任务识别，不需要数学推导。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 摘要

### 研究问题与矛盾

【原文事实，pp.1–2】摘要依次提出：给定少量带相机姿态的场景图像（K 约5），生成目标相机的新视角。许多生成式方法用图像扩散先验，但图像预训练不直接要求跨视角对应；作者认为视频预训练已包含隐式多视角知识，因而更适合迁移。关键矛盾是视频模型学习有时间顺序的连续序列，而稀疏 NVS 的输入可以任意重排，甚至不存在自然时间次序。

【背景知识】同一物体在相邻视频帧中的外观通常连续变化；网络若能生成相干视频，至少需要学习统计上的跨帧结构保持。视频还包含物体运动、遮挡和曝光变化，因此跨帧一致性并非严格静态三维几何。稀疏视图中的未观测表面有多个与输入相容的解释，直接回归可能平均多个可能解而模糊，条件生成模型可选择一个看似合理的解，却不保证其真实。视频先验的价值在于缩小候选解空间，而不是消除所有歧义。

### 方案与结论

【原文事实】作者将任务表述为低帧率视频补全，并采用三个配套改造：每视图作为独立单帧视频经过 VAE，避免时间压缩合并不同观察；以查询相机为坐标基准，不让任意首帧决定参考系；移除时间位置嵌入，不让注意力依据人为帧序。§3.3 另关闭首帧 CLIP 条件并用空文本提示。通过 LoRA 以及重新训练 patch embedding 在少量多视图数据上适配。

这三项分别解决潜表示、几何条件、注意力位置编码三个独立的顺序依赖来源，并非只打乱训练输入即可获得结构不变性。作者认为以较少额外监督便可让视频模型忘记时间，达到有竞争力的稀疏 NVS 性能（论文摘要用更强表述，表1须逐指标理解）。摘要没有证明预训练模型存储显式三维场，也没有证明它丢失了真实动态世界知识。实验依据在 §4 与附录。

一句话：把独立编码的稀疏图像和几何射线条件交给去时序偏置的视频扩散骨干，生成指定查询相机的图像。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 核心贡献

### 贡献主张

【原文事实，§1 pp.2–3】作者未提供独立编号贡献清单，以下为按引言末尾及方法归纳的四类主张，而非逐字贡献列表。

| 主张 | 性质 | 解决的问题 | 具体设计 |
|---|---|---|---|
| 视频先验迁移到稀疏 NVS | 问题重新表述/方法 | 图像扩散需学习跨视角关系、稀疏观测难 | 低帧率视频补全、预训练视频 DiT |
| 独立单帧 VAE | 方法 | 因果压缩及输入顺序依赖 | 每图单独编码，时间轴堆叠，独立解码 |
| 查询中心的几何条件及无时间位置编码 | 方法 | 首帧参考系、时间索引、首帧条件偏置 | query-centered Plücker rays、pixel unshuffle、去 temporal RoPE/CLIP |
| 少量数据及参数适配与可扩展性 | 实验/工程 | 大量标注多视图和全量更新成本 | 1K DL3DV 场景、LoRA r32、patch embed 更新、两分辨率训练 |

### 论证路径

Figure 1 解释总体动机；Figure 2 给出潜变量与条件输入、监督分支；Figure 3 和表2检验独立编码。表2还验证 pixel unshuffle、仅目标监督及位置编码；表3直接测量置换稳定性，表4比较有序学习和无序学习。表1比较骨干和数据量不同的模型，Figure 5 测数据规模；附录表6进一步检验多目标训练、表7速度、表8下游几何代理。

【背景知识】VAE、DiT、Plücker、RoPE、LoRA、pixel unshuffle 和 flow matching 都是本文所继承的工具。作者的独特主张是这些改造形成适合无序稀疏视图的视频生成机制，而非提出新的通用 VAE/扩散理论。独立编码先保住每个视图的条件，查询参考系使几何条件随输入排列等变，时间 RoPE 移除使跨视图处理不依赖槽位，三者共同构成技术逻辑。本文自称首个相关范式，未在本任务独立查证优先权。此处讨论创新类别及证据路线，而非数学证明。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 结论

### 作者结论

【原文事实，§5 p.15】作者认为视频生成模型已有较强的几何和多视角先验；独立逐视图 VAE、查询中心 Plücker 条件、去 temporal RoPE、轻量 LoRA 能将其适配为无序稀疏 NVS 生成器，仅使用1K训练场景。表1表明较大的 Wan2.1-14B 骨干比 CogVideoX-5B 更强，并与多数据的图像扩散方法有竞争力。作者由此提出将 NVS 作为视频基础模型轻量特化的可扩展路线，而不是完全独立的几何建模任务。

作者进一步提出一个关于视频世界模型的表示问题：时间顺序似乎容易通过轻量微调被移除，这是否意味着预训练模型存储更多跨视角知识而非动力学知识？作者将之作为启发式解释，期待进一步探索。

### 适用范围与未来问题

【独立评价】实验证据限定于论文所用数据集、预训练骨干、相机输入条件与训练设置。视频预训练总数据量并不等于下游1K场景；骨干变大同时改变架构、训练数据及预训练过程，不能只凭表1证明参数规模的独立因果作用。表3的小而非零的顺序标准差也不等于逐像素严格相同。表8提供下游重建一致性代理，但不是实测三维真值，不能推出真实几何恢复。论文未系统研究动态场景的时间演化、未知位姿、极端外推和生成置信度。作者关于动力学容易遗忘的解释未通过专门动态任务、表征探测或反事实试验验证。未来方向是把预训练视频模型迁移至非时序任务，以及区分其空间与动力学知识；本文没有给出这类问题的数学证明。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2.0 图表功能总览

### Figure 功能

| Figure | 页/章节 | 作用 |
|---|---|---|
| 1 | p.1，概览 | 说明视频先验与图像先验的不同，并概括三个结构改造 |
| 2 | p.5，§3 | 连接稀疏图片、相机条件、潜变量、DiT、监督、查询输出 |
| 3 | p.6，§3.2 | 解释时序缓存与步长造成的跨帧依赖，展示独立编码替代方案 |
| 4 | p.12，§4.3 | 比较室内外场景的目标图及生成结果 |
| 5 | p.14，§4.4 | 以训练场景数量为横轴、PSNR为纵轴比较数据效率 |
| 6 | p.23，§B | 增加场景以展示定性表现 |
| 7 | p.24，§B | 展示 COLMAP 位姿的野外样例 |

### Table 功能

| Table | 页/章节 | 研究问题 |
|---|---|---|
| 1 | p.10，§4.2 | 六输入、两基准的重建及感知指标，比较骨干和基线 |
| 2 | p.11，§4.4 | 各组件是否必要 |
| 3 | p.13，§4.4 | 输入排列变化是否影响结果 |
| 4 | p.14，§4.4 | 有序训练能否替代无序建模 |
| 5 | p.20，§A.1 | 三输入时能否保持表现 |
| 6 | p.21，§A.2 | 单目标模型与混合多目标训练的差异 |
| 7 | p.22，§A.3 | 推理速度与质量的步数折中 |
| 8 | p.22，§A.4 | 下游几何一致性代理的比较 |

图1–3建立机制，图4–7和表1–8共同支撑实验结论，图表不等于独立证明。此单元仅做功能导航，不展开机制和数学推导。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
## 2.1 Figure 1：总体思路与三项去时序设计

![Figure 1](FrameCrafter/Figures/Figure_01.jpg)

PDF p.1，区域渲染展示副本（300 DPI JPEG）。

### Caption 原文及中文

> Video Diffusion as Multi-view Prior. While prior generative NVS methods typically initialize from image diffusion models and rely on large pose-annotated multi-view datasets, videos naturally capture viewpoint changes and cross-view consistency, making them a more scalable source of supervision for NVS. We present FrameCrafter, which adapts pretrained video diffusion models to permutation-invariant multi-view model with lightweight modifications trained using LoRA on only 1,000 DL3DV-10K scenes, achieving strong performance and demonstrating a more data-efficient path to high-fidelity novel view synthesis.

视频扩散作为多视图先验。以往生成式 NVS 方法通常从图像扩散模型初始化并依赖大规模有位姿标注的多视图数据，而视频自然包含视点变化与跨视图一致性，因而可作为更易扩展的 NVS 监督来源。FrameCrafter 将预训练视频扩散模型通过轻量改造适配为置换不变的多视图模型，仅在1,000个 DL3DV-10K 场景上使用 LoRA 训练，获得较强性能，展示更节省数据的高保真新视角合成路线。

### 视觉布局与机制

上方：左侧是六张建筑外观图及蓝色输入相机、橙色查询相机；中间灰色 Video-DiT 表示视频扩散 Transformer；右侧是指定视角的建筑图。图像与相机共同决定条件，输出是新视图，不是3D网格或点云。中部红色标识图像扩散先验和大量姿态标注多视图数据的依赖；绿色表示视频预训练和轻量微调。这里是作者立论，不表示所有图像扩散方法均缺乏几何能力。

下方三个面板从左至右：① 每帧独立走 VAE；不是从头训练独立图像 VAE，而是调用相同的冻结视频编码器，每次只有一帧。② 以橙色查询相机建立坐标轴，所有蓝色输入相机相对它表达，固定查询坐标矩阵为单位阵。③ 将各帧 temporal RoPE 的位置索引全部归零，消除相对帧序差异；仍保留空间坐标信息与扩散噪声时间。三个改造各处理不同的顺序依赖，不能互相替代。LoRA 为低秩适配、VAE 为变分自编码器、DiT 为扩散 Transformer。

### 正文关联与证据边界

§1 pp.2–3解释先验与无序任务的矛盾，§3.2 独立潜编码对应式(1)–(4)，§3.3 几何条件及位置编码对应式(5)，§3.4 LoRA、训练采样与目标损失。图示将既有相机姿态带入条件，不展示如何从无位姿图像估计相机。表2–4才提供设计消融及置换鲁棒性，表1与图5才提供数据效率证据。图本身不能证明先验含显式三维几何或不变性达到数值零误差。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
## 2.2 Figure 2：模型数据流和监督

![Figure 2](FrameCrafter/Figures/Figure_02.jpg)

PDF p.5，区域渲染展示副本。

### Caption 原文及中文

> Each input image is encoded by a frozen video VAE, and its camera pose is converted into a Plücker ray map that is concatenated with the image latent along the channel dimension. The resulting view tokens are then concatenated along the temporal dimension, where the first K tokens correspond to input views and the final token represents the query view. Only the patch embedding layer and the LoRA modules in the DiT backbone are trained. During both training and inference, only the predicted query latent is used for decoding and supervision.

各输入图像经冻结的视频 VAE 编码，相机位姿转为 Plücker 射线图，与图像潜变量沿通道拼接；随后沿时间维拼接各视图表征，前 K 个对应输入，最后一个对应查询。只训练 patch embedding 和 DiT 中的 LoRA 模块；只使用查询预测潜变量进行解码及监督。注意，最后一句中的监督实际适用于训练，推理没有 GT。

### 视觉布局与机制

左侧每张照片配有自己的相机符号、冻结 VAE 编码器和拼接块；灰/淡色块示意图像与射线条件，最下方红色查询相机对应查询块，目标没有可见图像输入。图中火焰表示可训练、雪花表示冻结。相机射线不是场景深度，而是每像素的投影几何，所有射线先经像素重排到潜空间网格。视图维与通道维承担不同作用：前者是整组相机观察的索引，后者是在同一空间位置融合图像、条件和射线信息。

中部 patch embedding 将扩张后的输入通道投影至 DiT token 特征空间。DiT Blocks × N 表示重复的 Transformer 块，冻结主体但 LoRA 可训练。右侧输出中上下文 token 被丢弃；黄色查询 token 经冻结 VAE Decoder 生成图像。GT Novel View 经同一冻结编码器形成 GT latent；损失的箭头表明只约束查询侧。此处是训练信息流简图，不应把黄色预测块与 GT latent 的箭头误读为直接 L2 像素回归。正文 §3.4 实际监督目标为噪声与干净潜变量之差所定义的速度场；解码对应完成去噪后的潜变量。

### 正文关联与证据边界

式(1)–(3)的独立编码和时间堆叠建立每视图一潜帧。式(5)还有噪声 latent、含 mask 的 VAE 条件和 ray map 三部分，图2为示意并未逐项标出。上下文被丢弃是指不将其输出用于损失/解码，不表示它不参与注意力或不向共享参数传递梯度。训练和推理均依赖已知相机，训练另有真实目标及噪声速度监督；推理不输入目标 GT。原文未给出 patch embedding 的完整核尺寸及每层 token 形状，不能从图中立方体推断数值。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
## 2.3 Figure 3：标准因果 VAE 与独立编码

![Figure 3](FrameCrafter/Figures/Figure_03.jpg)

PDF p.6，区域渲染展示副本。

### Caption 原文及中文

> Comparison between standard causal video VAE encoding and our per-view independent encoding. (a) Standard causal encoding mixes frame information through feature caching and temporal stride, leading to inter-frame dependency. (b) Our per-view encoding eliminates temporal interaction, ensuring independent view-to-latent mapping suitable for unordered sparse-view inputs.

标准因果视频 VAE 与逐视图独立编码的比较。(a) 标准因果编码通过特征缓存和时间步幅混合帧信息，引入帧间依赖；(b) 独立逐视图编码消除时间交互，获得适合无序稀疏视图的独立视图—潜表示映射。

### 视觉布局与机制

左图(a)：上方蓝色圆圈0至4表示顺序帧或其特征，斜线圆圈表示填充。第一帧用零填充；后续使用 Cache Padding 将之前帧的特征接到当前输入前端。虚线箭头表示卷积感受野，第一层 Causal Conv3D 仍有5个特征位置，第二层标注 Stride 2，示意压缩至0、1、2三个位置。首帧有独立输出，后面输出聚合多帧信号。图中0、1、2是压缩后的槽位，不能把槽位1当成仅来自输入帧1。

右图(b)：三个输入0、1、2各自配两处零填充，从单帧输入得到单帧输出，不使用其他视图缓存。蓝色圆圈与虚线连线保持各自独立。其卷积仍有空间感受野，仍会对空间尺寸下采样；去除的是跨视图时间混合和时间压缩，不是所有信息损失。

【背景知识】简化一维因果核可写为 $y_i=\sum_{j=0}^{q-1} W_j x_{i-j}$。若改变排列而保持同一卷积顺序，$y_i$ 的历史邻居会变，无法简单将旧输出按相同排列重排。时间 stride 还会减少输出槽位。独立编码改为 $z_i=\mathcal E(I_i)$，重新排序只重新排序潜表示，随后在 DiT 跨视图推理。图中单层 stride 2 是解释性示意，§3.1 所述整个视频 VAE 时间压缩通常约4，不能把示意中的2当成完整骨干配置。

### 正文关联与证据边界

§3.1–3.2、式(1)–(4)对应图3；表2移除逐视图编码时 PSNR 从15.69降到11.50，DreamSim 从0.114升到0.656，说明联合时序编码在这一设置下严重不适合。该消融同时影响压缩、独立性及潜变量与姿态对应，不能仅归因于顺序偏置。独立单帧 VAE 自身只证明编码映射等变，不单独保证后续整个模型严格不变。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
## 2.4 Figure 4：稀疏输入定性比较

![Figure 4](FrameCrafter/Figures/Figure_04.jpg)

PDF p.12，区域渲染展示副本，已校正下边缘至完整末行。

### Caption 原文及中文

> Qualitative comparison under sparse inputs. FrameCrafter produces more geometrically consistent and sharper novel views compared to SEVA [57], Aether [59], LVSM [17], with improved viewpoint alignment and detail preservation.

稀疏输入下的定性比较。与 SEVA、Aether、LVSM 相比，FrameCrafter 生成几何更一致、更清晰的新视角，提高视点对齐和细节保持。（作者表述）

### 视觉布局与机制

六列按 Input Views、Target View（GT）、Ours、SEVA、Aether、LVSM 排列。三组场景每组左侧六张输入，右边两行对应两个目标相机。第一组健身房：目标第一张包含白墙、红球、墙上矩形屏及画框。Ours 对墙面和红球的位置较接近 GT；SEVA 可见不连贯的高亮物体/异常纹理，Aether 有重影和布局偏移，LVSM 明显模糊。第二目标中健身器械的细杆与柱子是检查几何对齐的线索。

第二组街道建筑：招牌与门窗、遮棚透视、地面边缘提供高频细节及几何线索。Ours 保留相对清晰的店面结构；其他方法可见招牌/门窗及地面区域的局部差异，LVSM 较模糊。文字是否逐字准确不能仅凭这张缩略图断言。第三组室内客厅：沙发、地毯、窗帘、桌子及玩偶是局部外观线索；Ours 较好保持主要轮廓，SEVA 个别图出现醒目异常区域，Aether 与 LVSM 的摆放及清晰度有偏差。

【合理推断】回归结果的模糊可能与不确定目标的平均估计有关，但图中不能直接确认训练损失或内部机制；视频先验可补足外观，但新增细节也可能是假设而非真实观测。

### 正文关联与证据边界

§4.3 的定性比较对应此图；表1以 PSNR、SSIM、LPIPS、DreamSim 定量衡量。图中例子支持作者所称外观和视点对齐优势，但不提供每场景量化误差、三维真值或跨随机种子的稳定性。对未观测区域的合理填补和准确还原必须区分。不能把选择的样例直接等同于全部测试集平均性能。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
## 2.5 Figure 5：数据规模与性能

![Figure 5](FrameCrafter/Figures/Figure_05.jpg)

PDF p.14，区域渲染展示副本。

### Caption 原文及中文

> Performance on different numbers of training scenes. Even with only 20 scenes, our model surpasses EscherNet trained on 10K scenes. Performance improves consistently as training data increases, demonstrating strong data efficiency and scalability.

不同训练场景数量下的性能。即便仅用20个场景，本文模型仍超过用10K场景训练的 EscherNet；随训练数据增多，性能持续提升，展示数据效率及可扩展性。

### 视觉布局与机制

横轴 Number of Training Scenes 为对数刻度（10²、10³、10⁴），纵轴 PSNR (dB) 越大越好。蓝色折线带空心圆表示 Ours，灰色星号为10K场景 EscherNet；水平虚线约12.1 dB。按图读取四个 Ours 点：20场景约14.6、50场景约14.9、1,000场景约15.3、2,000场景约15.9 dB（均为图读近似，并非作者列出的精确数字）。20场景点高于 EscherNet，箭头标注500× less data，对应10,000/20=500倍下游场景数差异。横轴不是线性增长，因此折线斜率不能直接理解成每新增一个场景的固定收益。未报告误差带。

【背景知识】PSNR 定义为 $10\log_{10}(L^2/\mathrm{MSE})$；同一峰值 L 下高2.5dB意味着 MSE 比约 $10^{-2.5/10}\approx0.56$，这里只说明分贝差如何解释，不能将图读近似差值替代原始统计。

### 正文关联与证据边界

§4.4 Data Scaling（pp.14–15）认为视频预训练已有较强先验，轻量训练主要用于迁移与相机条件。图5与表1不同：表1的 Wan 14B 高分辨率 DL3DV 结果为17.18，不能把它填为图中1K点；此图正文没有明确列出与表1完全相同的训练/评估配置或随机种子。数据效率只针对额外有姿态多视图场景，不衡量视频预训练规模、总图像数、总训练成本或预训练泄漏。没有控制骨干与所有训练设置，不能从图5单独证明视频先验是全部收益的唯一原因。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
## 2.6 Figure 6：附加定性对比

![Figure 6](FrameCrafter/Figures/Figure_06.jpg)

PDF p.23，区域渲染展示副本。

### Caption 原文及中文

> Additional Qualitative Comparisons with SEVA [57], Aether [59] and LVSM [17].

与 SEVA、Aether 和 LVSM 的额外定性比较。

### 视觉布局与机制

结构与 Figure 4 一致：输入六视图、真实目标、Ours、SEVA、Aether、LVSM。三组场景每组两张查询图。第一组为盆栽/花枝与紫色桌布，细花枝、花盆轮廓和背景直线是较难合成的细结构。Ours 的整体摆放与桌布接近目标，SEVA 和 Aether 的枝叶及背景有不同程度偏差，LVSM 显著平滑。第二组为商店内并列家电，重复矩形柜门和天花板照明体现空间布局；Ours 和 GT 的主要排列较一致，其他生成方法可见局部柜门、反射或视点差异，LVSM 细节弱。第三组为植物与彩色雕塑/装饰，树叶、花朵、棚顶线条及遮挡关系较复杂；Ours 相对保留前景结构，SEVA/Aether 有局部纹理或位置误差，LVSM 较模糊。不能精确辨别每个花朵或柜门是否恢复正确。

### 正文关联与证据边界

§B p.22明确补充 Figure 6，服务于§4.3。图中多样场景表明方法在某些重复纹理、细物体和复杂背景中能生成更接近目标的外观，不单独验证每类场景的成功率。输出均为图像而非显式深度，本图不提供直接几何误差或时序轨迹一致性；定性与定量的互补关系见表1和表8。本图是视觉对照，不涉及新增数学公式。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
## 2.7 Figure 7：野外样例及相机误差

![Figure 7](FrameCrafter/Figures/Figure_07.jpg)

PDF p.24，区域渲染展示副本。

### Caption 原文及中文

> In-the-wild qualitative results. We evaluate FrameCrafter using COLMAP-estimated camera poses, which contain realistic noise and reconstruction artifacts. The results suggest robustness beyond curated benchmark settings.

野外定性结果。使用包含真实噪声与重建伪影的 COLMAP 估计相机位姿评估 FrameCrafter，结果提示其在精心整理的基准之外仍具一定鲁棒性。

### 视觉布局与机制

四个例子采用2×2排列，每组上方为 Ground Truth 与 Ours 并列，下方为6 Input Views 条带。左上是带树和窗户的建筑立面，右上是另一处砖墙建筑外观；左下从较高视角俯瞰楼顶与停车区域，右下是砖墙建筑的侧面。生成图大体保持相机视角、墙面或屋顶主轮廓，但一些细纹理、天空及远处背景仍有差别。输入条带里部分图很模糊或亮淡，这是原图本身可见现象，并非展示文件生成损坏；具体成因原文未逐图说明。

【背景知识】COLMAP 为基于多视图匹配与结构运动恢复的相机/结构估计系统。估计内外参可能有误差；若用错误姿态生成 Plücker 条件，网络接收的几何射线与输入真实光线不完全对应。能够容忍一定偏差有实际意义，但鲁棒性大小需通过受控噪声实验确定。

### 正文关联与证据边界

§B p.22将此图作为超出 curated benchmark 的补充验证。此处仅有四组定性结果，不提供位姿误差的度量、输入选择规则或与噪声为零的对照，不能推断噪声容限、无位姿推理或在动态场景的可靠性。COLMAP 在此提供估计相机，不等于模型自身预测位姿。图7是外部场景视觉示例，不涉及新增数学公式。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN TABLE-01 -->
## 2.8 Table 1：六输入主实验

### Caption 原文及中文

> Quantitative comparison on 6-view NVS on DL3DV-Benchmark and Mip-NeRF 360. FrameCrafter uses pretrained video diffusion backbones and is trained on only 1K multi-view scenes. Scaling the backbone (CogVideoX-5B → Wan2.1-14B) consistently improves performance, achieving competitive results with prior state-of-the-art methods. Best results are highlighted as first, second, third. DS denotes DreamSim [9]. For completeness, we also compare to regression baselines (top two rows) which tend to consistently output blurrier images that produce poor LPIPS and DS but better PSNR and SSIM.

DL3DV-Benchmark 与 Mip-NeRF 360 上六视图 NVS 的定量比较。FrameCrafter 使用预训练视频骨干，仅在1K多视图场景训练；扩大骨干后各指标改善，结果与已有先进方法有竞争力。排名以颜色区分，DS 为 DreamSim。另列两个回归基线，作者指出它们通常输出较模糊图像，感知指标较差但像素/结构指标较好。

### 原始数据与协议

【原文事实】PDF p.10 表1。DL3DV-Benchmark 与 Mip-NeRF 360，遵循 SEVA 划分，生成设置保持各自原本配置，最终统一 resize 或中心裁剪到480×480计算指标（§4.1–4.2）。PSNR单位dB、SSIM无量纲越大越好，LPIPS、DS感知距离越小越好。#Scenes 是论文表内列出的下游训练场景数，“–”为未报告而非零。

| 方法 | 骨干 | #Scenes | DL3DV PSNR↑ | SSIM↑ | LPIPS↓ | DS↓ | Mip360 PSNR↑ | SSIM↑ | LPIPS↓ | DS↓ |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| LVSM | – | – | 17.09 | .478 | .333 | .204 | 15.25 | .317 | .609 | .577 |
| E-RayZer | – | – | 16.85 | .442 | .455 | .254 | 16.56 | .343 | .621 | .340 |
| EscherNet | SD1.5 | 10K | 12.07 | .251 | .484 | .227 | 11.14 | .126 | .540 | .315 |
| Aether | CogVideoX-5B | – | 12.66 | .258 | .469 | .140 | 12.60 | .220 | .651 | .334 |
| SEVA | SD2.1 | 80K | 16.15 | .470 | .253 | .088 | 14.59 | .294 | .372 | .137 |
| Ours | CogVideoX-5B | 1K | 13.65 | .272 | .389 | .113 | 13.09 | .209 | .540 | .204 |
| Ours | Wan2.1-14B | 1K | 17.18 | .445 | .223 | .066 | 15.64 | .279 | .365 | .111 |

### 对比与结论边界

以 SEVA 为基准，Wan 版本 DL3DV PSNR +1.03 dB，SSIM −.025，LPIPS −.030，DS −.022；Mip360 PSNR +1.05 dB，SSIM −.015，LPIPS −.007，DS −.026。感知指标两数据集均为表内最好，但 SSIM 都不领先；Mip360 PSNR 最高为 E-RayZer 16.56。DL3DV Wan PSNR最高17.18，仅比 LVSM 高.09 dB；所以不能写成全部指标 SOTA。

同 CogVideoX 骨干下，Ours 对 Aether 的 DL3DV 指标全面改善，PSNR +.99、SSIM +.014、LPIPS −.080、DS −.027；Mip360 PSNR +.49、LPIPS −.111、DS −.130，但 SSIM −.011，作者正文的“一致改善”应理解总体，而非逐项全赢。Ours Cog→Wan 的 DL3DV 四项变化为+3.53、+.173、−.166、−.047；Mip360 +2.55、+.070、−.175、−.093。此比较支持选择更强骨干，但同时更换家族和预训练，不是隔离模型大小的受控试验。1K vs 80K 为80倍下游场景数，不能当成总数据成本相差80倍。PSNR由像素误差决定、感知指标由学习特征决定，对几何正确性的敏感性不完全相同。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
## 2.9 Table 2：结构消融

### Caption 原文及中文

> Ablation study on 192×336 resolution generation (on DL3DV-Benchmark). Removing the per-view independent encoding, pixel unshuffle, or target-only supervision degrades performance, with per-view “single-frame” video encodings being the most crucial design choice.

DL3DV-Benchmark 上192×336生成分辨率的消融。移除逐视图独立编码、pixel unshuffle 或仅目标监督均导致性能下降，其中逐视图单帧视频编码最关键。

### 原始数据与协议

PDF p.11，§4.4。✗ 表示去除/替换对应设计，w/ 表示采用该替代设计。不能与高分辨率表1直接作受控消融比较。

| Variant | PSNR↑ | SSIM↑ | LPIPS↓ | DreamSim↓ |
|---|---:|---:|---:|---:|
| ✗ Per-View Encoding | 11.50 | .147 | .676 | .656 |
| ✗ Pixel Unshuffle | 13.53 | .212 | .359 | .129 |
| ✗ Target-Only Supervision | 15.08 | .287 | .270 | .116 |
| w/ PRoPE | 14.91 | .289 | .315 | .174 |
| w/ Original RoPE | 15.30 | .300 | .265 | .122 |
| Full model | 15.69 | .326 | .246 | .114 |

### 对比与结论边界

以 Full 为基准，去独立编码 PSNR −4.19、SSIM −.179、LPIPS +.430、DS +.542，是最严重退化。联合视频编码压缩时间并混合不同相机信息，且引入输入顺序偏置（Fig3）。由于这三个机制同时变化，不能将差值全部归于时间 RoPE。去 pixel unshuffle 时 PSNR −2.16、SSIM −.114、LPIPS +.113、DS +.015；标准空间下采样对射线场采样的细节聚合可能降低相机控制精度。去仅目标监督（改为全序列监督）−.61 dB、−.039、+.024、+.002；作者解释已知上下文重建使网络把容量用于较容易的任务，而未知目标是主要目标。图2表明不监督上下文输出并不代表上下文无梯度。采用 PRoPE −.78 dB、−.037、+.069、+.060，原始 RoPE −.39 dB、−.026、+.019、+.008。§4.4认为 PRoPE 改变注意力结构更深，轻量适配未充分发挥；这是作者推测，未以更大训练预算验证。原始 RoPE保留时间索引，表3进一步检验对排列的敏感性。

【独立评价】全模型四项最优，支持这些配置在固定实验预算下的联合实用价值，但无完整因子交叉消融、重复训练及误差条，不能据此确定每个模块独立因果贡献或对其他骨干也同样有效。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
## 2.10 Table 3：输入置换稳定性

### Caption 原文及中文

> Performance under 10 random permutations of input views.

输入视图的10种随机排列下的性能。

### 原始数据与协议

PDF p.13。DL3DV-Benchmark 上同一输入集合随机排列10次，报告各运行指标均值及跨排列标准差的平均值。Mean 表示平均质量，Std 越低越不敏感。三种训练设置：首视角参考系、原始时空RoPE、本方法查询参考系及去 temporal RoPE。

| Training Setup | PSNR Mean↑ | Std↓ | SSIM Mean↑ | Std↓ | LPIPS Mean↓ | Std↓ | DreamSim Mean↓ | Std↓ |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| First-view Coordinate | 15.67 | .4730 | .309 | .0276 | .262 | .0203 | .112 | .0111 |
| w/ original RoPE | 15.61 | .3106 | .310 | .0212 | .254 | .0095 | .115 | .0063 |
| Ours | 16.21 | .0081 | .348 | .0006 | .224 | .0003 | .102 | .0005 |

### 对比与结论边界

Ours 对首视角参考系 PSNR均值 +.54、Std从.4730降至.0081（约58.4倍）；对原RoPE +.60、Std从.3106降至.0081（约38.3倍）。SSIM均值分别+.039、+.038；LPIPS均值−.038、−.030；DS−.010、−.013；所有标准差也显著降低。参考系改变会使同一物理场景的射线条件整体旋转平移，虽理论上场景不变，但网络未必对这种坐标变换等变。时间RoPE使注意力依据任意帧索引，故变更顺序可改变权重。两者去除后更稳定。

【独立评价】均值标准差是经验鲁棒性证据，而非像素级排列不变的严格验证；原文未说明全部随机噪声是否锁定、跨样例聚合及浮点实现细节。小的非零标准差不能直接归因某一种因素。理论还要求所有跨视图算子、mask、噪声随排列一致变换。表3均值与表2全模型15.69不同，协议不同且不能直接相减解释为设计收益。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
## 2.11 Table 4：有序与无序训练的比较

### Caption 原文及中文

> Effect of training setup under ordered evaluation on 192 × 336 resolution 6-view NVS (DL3DV-Benchmark [23]). During evaluation, input views follow their original temporal order in the source video.

DL3DV-Benchmark 上192×336分辨率六输入 NVS 有序评估下的训练设置影响；评估时输入遵循原视频时间顺序。

### 原始数据与协议

PDF p.14 表4、pp.13–14正文。Ordered 保留时间RoPE并按有序序列训练，Ours 置换不变并以随机打乱输入训练；两者用相同有序测试输入。指标定义同表2。

| Training Setting | PSNR↑ | SSIM↑ | LPIPS↓ | DreamSim↓ |
|---|---:|---:|---:|---:|
| Ordered | 15.04 | .292 | .276 | .136 |
| Ours (Permutation-Invariant) | 15.69 | .326 | .246 | .114 |

### 对比与结论边界

Ours 比 Ordered +.65 dB、+.034 SSIM、−.030 LPIPS、−.022 DS。即使评估顺序有利于有序模型，无序模型仍领先。作者认为稀疏视角的视点间距和空间布局不规则，原视频顺序不一定有利于几何推理；打乱训练可以把同一组 K 输入变成 K! 个形式不同、目标一致的训练样本（K=6 时720种）。然而若网络严格不变，这些排列在理论上不是新增独立信息；更多排列是训练配置变化，不应视作720倍真实几何观测。表4同时改变架构与训练方式，不能隔离随机打乱、RoPE删除或参考系改变的单项贡献，也未比较所有可能的几何排序策略。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
## 2.12 Table 5：三输入泛化

### Caption 原文及中文

> Quantitative comparison on 3-view NVS on DL3DV-Benchmark [23] and Mip-NeRF 360 [2]. Best results are highlighted as first, second, third. DS denotes DreamSim [9]. All outputs are resized or center-cropped to 480 × 480 for fair comparisons.

DL3DV-Benchmark 与 Mip-NeRF 360 三视图 NVS 定量比较。最佳、次佳与第三名以颜色标识，DS为DreamSim，所有输出缩放或中心裁剪到480×480以统一评估。

### 原始数据与协议

PDF p.20，§A.1。原文称未专门针对三输入训练。场景数为下游训练量，未报告用–标示；指标方向同表1。

| Method | Backbone | #Scenes | DL3DV PSNR↑ | SSIM↑ | LPIPS↓ | DS↓ | Mip360 PSNR↑ | SSIM↑ | LPIPS↓ | DS↓ |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| EscherNet | SD1.5 | 10K | 11.43 | .228 | .524 | .266 | 10.44 | .171 | .627 | .383 |
| Aether | CogVideoX-5B | – | 11.98 | .245 | .531 | .204 | 13.11 | .228 | .614 | .239 |
| SEVA | SD2.1 | 80K | 13.52 | .359 | .364 | .167 | 12.73 | .242 | .443 | .198 |
| Ours | Wan2.1-14B | 1K | 14.73 | .334 | .351 | .129 | 14.14 | .235 | .491 | .166 |

### 对比与结论边界

对 SEVA，DL3DV PSNR +1.21、SSIM −.025、LPIPS −.013、DS −.038；Mip360 +1.41、−.007、+.048、−.032。两数据集PSNR、DS均为表内最佳，但 Mip360 LPIPS 不如 SEVA、SSIM也不是最佳。相较表1的六输入，Ours 三输入的 DL3DV PSNR 从17.18降至14.73（−2.45 dB），Mip360从15.64降至14.14（−1.50 dB），说明视角减少仍造成可观退化；跨表不是固定样本控制试验，需谨慎解释差异。独立视图编码及共享注意力使输入维度原则上灵活，并不自动保证任意K的性能。结果只支持这里的三视图条件，不支持单视图或完全无几何条件。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
## 2.13 Table 6：多目标联合预测

### Caption 原文及中文

> Flexible multi-target prediction. Performance comparison before and after training with mixed m-to-n supervision (T = 10). Evaluation is performed at 192 × 336 resolution on DL3DV-Benchmark [23].

灵活多目标预测。在混合 m-to-n 监督（T=10）训练前后比较性能，DL3DV-Benchmark上以192×336分辨率评估。

### 原始数据与协议

PDF p.21 §A.2。主模型为6-to-1；新增混合训练固定视图轴总长10，输入数随机3至9，其余为目标；表中评估输入固定6，目标从1至4。原文未说明如何处理多目标参考系的具体选择。

| Setting | Original PSNR↑ | SSIM↑ | LPIPS↓ | DS↓ | After Training PSNR↑ | SSIM↑ | LPIPS↓ | DS↓ |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 6-to-1 | 15.69 | .326 | .246 | .114 | 16.27 | .365 | .226 | .100 |
| 6-to-2 | 14.84 | .261 | .347 | .200 | 16.20 | .374 | .229 | .102 |
| 6-to-3 | 14.62 | .239 | .419 | .280 | 16.42 | .374 | .223 | .096 |
| 6-to-4 | 14.18 | .217 | .511 | .384 | 16.23 | .367 | .225 | .097 |

### 对比与结论边界

原模型目标从1到4，PSNR −1.51、SSIM −.109、LPIPS +.265、DS +.270，说明单目标训练不能无代价外推至多目标。混合训练后 6-to-1 PSNR +.58，6-to-2 +1.36，6-to-3 +1.80，6-to-4 +2.05；对应 DS 分别−.014、−.098、−.184、−.287。训练后PSNR范围16.20–16.42，跨度仅.22dB，感知指标相近。作者认为同时预测多个目标使多视图监督更充分，具有几何正则化作用；没有单独控制训练迭代数和数据，不能将全部收益归因于联合约束。表中是每目标图像指标而非直接测量多输出之间极线一致性；没有证实联合生成一定得到同一严格3D场。单查询中心在多目标情况下如何定义未在正文/附录详细说明，不臆造实现。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN TABLE-07 -->
## 2.14 Table 7：速度与扩散步数

### Caption 原文及中文

> Compute time and comparison with baselines on DL3DV (6-view NVS) using a single A6000 GPU.

在单张 A6000 GPU 上，DL3DV 六输入 NVS 的计算时间与基线比较。

### 原始数据与协议

PDF p.22 §A.3。轨迹生成类基线采用最接近查询的两视图插值适配，不完全是同样的六视图集合输入。Avg. Time 单位为秒，近似值。不同扩散采样步数的比较是质量与速度折中。

| Method | PSNR↑ | SSIM↑ | LPIPS↓ | DreamSim↓ | Avg. Time↓ |
|---|---:|---:|---:|---:|---:|
| ViewCrafter | 13.55 | .312 | .443 | .181 | ~247s |
| GEN3C + VGGT | 15.95 | .398 | .333 | .099 | ~1460s |
| SEVA | 16.15 | .470 | .253 | .088 | ~24s |
| Ours (5 steps) | 18.13 | .485 | .298 | .114 | ~22s |
| Ours (10 steps) | 17.72 | .469 | .254 | .089 | ~42s |
| Ours (25 steps) | 17.35 | .454 | .226 | .070 | ~99s |
| Ours (50 steps) | 17.18 | .445 | .223 | .066 | ~177s |

### 对比与结论边界

从5到50步，PSNR 18.13→17.18（−.95），SSIM .485→.445（−.040），LPIPS .298→.223（−.075），DS .114→.066（−.048），时间22→177秒（约8倍）。作者解释少步输出更模糊，有利像素/结构平均指标；多步产生更丰富外观，感知更好但像素不一定对齐。5步时间接近且略快于SEVA，PSNR高1.98 dB，但两感知指标较差。10步 LPIPS/DS只分别比SEVA差.001，但时间约1.75倍；25步感知已优于SEVA。50步约SEVA的7.4倍，仍快于采用两近邻适配的轨迹基线。不能简单声称全面比SEVA快。

附录还报告 LoRA 30K迭代、1K场景，相比SEVA可训练参数少7.8倍、迭代少30倍、下游数据少80倍，不能转换为总训练FLOPs或总成本；预训练开销未统一比较。DMD蒸馏仅作为未来加速可能，未提供本文实测。不同模型计时是否包括所有预处理与重建环节未完整说明，比较限定于论文给定协议。

<!-- PTH:END TABLE-07 -->

<!-- PTH:BEGIN TABLE-08 -->
## 2.15 Table 8：下游三维一致性代理

### Caption 原文及中文

> Downstream 3D reconstruction consistency. Using the same reconstruction pipeline, our generated views yield lower depth errors and better pointmap accuracy, completeness, and overall error than SEVA.

下游三维重建一致性。使用相同重建流程，本文生成视图相比SEVA产生更低深度误差和更优点图精度、完整性及总体误差（作者概括，δ指标存在例外）。

### 原始数据与协议

PDF p.22 表8；§A.4 p.21。在DL3DV六输入条件下，同一 MapAnything 分别接受（相同真实输入 + 生成目标）与（相同真实输入 + GT目标），对比查询视图深度/几何。所有指标在查询相机系、米制尺度计算，GT目标的重建是参照而非真实扫描几何。

| Method | AbsRel↓ | δ<1.25↑ | RMSE(m)↓ | Acc.(m)↓ | Comp.(m)↓ | Overall(m)↓ |
|---|---:|---:|---:|---:|---:|---:|
| SEVA | .110 | .947 | .857 | .254 | .249 | .251 |
| Ours | .073 | .940 | .789 | .163 | .188 | .176 |

【背景知识】设参考深度 $d_i$、预测深度 $\hat d_i$，AbsRel 是 $\frac1N\sum_i|\hat d_i-d_i|/d_i$，RMSE是 $\sqrt{\frac1N\sum_i(\hat d_i-d_i)^2}$，阈值准确率δ为满足 $\max(\hat d_i/d_i,d_i/\hat d_i)<1.25$ 的比例（正有效深度）。点图 Accuracy 通常为预测到参照的最近邻距离，Completeness为反向；Overall为两方向 Chamfer-L1 汇总，原文没有给出采样及具体归约细节。这里说明常用定义，不冒充实现源码。

### 对比与结论边界

Ours AbsRel低.037（约33.6%），RMSE低.068m（约7.9%），Accuracy低.091m（约35.8%），Completeness低.061m（约24.5%），Overall低.075m（约29.9%）。但 δ从.947下降为.940，即低0.7个百分点，不能称全部深度指标更优。误差均值和阈值比例可能不同趋势：减少少量大误差可降低RMSE，同时略增越过阈值的像素数。

【独立评价】控制重建器与输入的设计有意义，说明生成图在该几何估计器下更接近真实目标参与时的结果。两次重建都存在系统误差，且共享输入可产生相关误差，未提供真实深度/点云或跨重建器验证；所以这只能作为一致性代理，不代表直接恢复真实3D或生成结果必定满足严格多视图几何。该表是本文从图像评价迈向几何评价的重要补充，而非证明。

<!-- PTH:END TABLE-08 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 2.16 图表关系总览

### 设计—实验链

Fig. 1 提出稀疏观测补全的整体设想；Fig. 2 将设想落实到独立编码、射线条件、DiT 和目标监督；Fig. 3 展示联合因果编码为何破坏视图独立性。Table 2 将结构拆分为逐视图 VAE、射线重排、仅目标监督、相机条件形式和时间 RoPE 五项对照：尤其逐视图 VAE 对 PSNR、LPIPS 的影响最大。Table 3 则直接测量输入排列引起的波动，而非只测平均图像质量。Table 4 对比随机采样策略，说明训练分布也是视图集合适配的一部分。三张表共同支持设计，但没有构成对整个网络严格置换不变性的数学证明。

### 性能—泛化链

Table 1 与 Fig. 4、6 分别从量化和视觉效果检验六视图 NVS：回归基线可在像素指标领先但纹理模糊，生成方法可能锐利却不精确。Fig. 5 隔离下游训练场景规模，表明小规模适配也有价值。Table 5 检验三输入，Table 6 检验多目标训练，Table 7 将质量与时间放在共同的比较框架，Table 8 用固定重建器检验补全图是否有利于下游几何。Fig. 7 扩展到 COLMAP 估计相机的非基准示例；未给定噪声量与统计指标，不能替代鲁棒性曲线。上述证据分别回答质量、适配数据成本、输入输出灵活性、推理代价和几何代理一致性，不能合成一个无条件的 SOTA 结论。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 3.1 研究问题

### 任务定义

原文 §3.1（p.4–5）：给定同一场景 K 张图像及已知相机姿态，结合 §3.3 的内参和外参，为指定查询相机生成图像。记条件集合 $S=\{(I_k,K_k,T_k)\}_{k=1}^{K}$、查询 $q=(K_q,T_q)$；输出是 $\hat I_q$，而非必须可渲染的显式 3D 表示。补充形式化为条件分布 $p_\theta(I_q\mid S,q)$。当输入视图及其相机成对重排时，要求条件分布不改变；固定采样噪声的逐次输出一致性是更强的条件。多目标扩展见附录 A.2，主模型训练为 6-to-1。

### 歧义与边界

稀疏观测只约束已见表面和投影范围。遮挡后方、缺失视角的纹理及几何可能有多种与输入相容的解释；这不是单纯增加渲染分辨率就能解决的问题。视频预训练提供统计先验，用以选择合理的补全，但不能使未观测区域唯一可辨识。目标是与指定相机和已有观察协调的真实感图像，而不是证明每个生成像素对应真实 3D 点。论文使用已标定视角；Fig. 7 的估计姿态实验也未把相机估计变成网络的输出任务。动态场景、严重姿态误差及物理介质变化未被该设定系统处理。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 3.2 研究动机

### 现有路线的约束

原文 §1–2（p.2–4）指出：回归式方法受有限几何约束制约，未见内容往往模糊；图像扩散能以生成先验补足外观，但跨视角一致性仍常需大量带位姿数据；视频扩散从大量视频学习跨帧相关性，作者假设其几何先验更贴合 NVS。视频模型的既有设计却默认有序且局部连续的帧序列，与无序、相距很远的稀疏视图集合冲突。

### 原因与证据

联合时序 VAE 的跨帧因果缓存引入顺序依赖，时间下采样会把若干不同相机图像混在同一潜变量；时间 RoPE 赋予人为帧序号；首输入相机作为原点使交换顺序改变所有射线；首帧 CLIP 也赋予单个输入特殊身份。Fig. 3 是结构证据，Table 2 的逐项移除与 Table 3 排列方差是实验证据。与 EscherNet/SEVA 的数据效率比较（Table 1、Fig. 5）符合视频先验假说，但架构、模型大小、预训练语料和微调预算不同，不能单独识别视频数据本身的因果效应。与同 CogVideoX 骨干的 Aether 比较更接近同骨干控制，但目标设定和训练方案依旧不同。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 3.3 相关工作

### 相关工作分类

按原文 §2（p.3–4）组织：NeRF [27]、3DGS [19] 先建立场景表示再渲染；稀疏条件下正则化 [28]、前馈预测 [15,17,52,54] 减少逐场景优化或强化先验；图像扩散 NVS 如 EscherNet [21]、SEVA [57] 从生成先验获得未见区域补全能力，但多视角监督仍重要。视频路线分为先重建再视频补洞（如 ViewCrafter [50]、GEN3C [32]），和直接向视频网络注入位姿、射线或注意力几何条件（如 Aether [59]）。前者依赖中间几何，后者通常服务有序轨迹生成。

### 本文位置

FrameCrafter 不要求先重建，也不要求指定连续路径，而将离散目标视图视为无序集合中的缺失元素。其贡献在于重用视频基础模型并消除与任务不匹配的时间索引偏置，而非发明 VAE、DiT、LoRA 或 Plücker 坐标。直接生成路线的优势是利用先验完成欠约束任务，代价是没有显式场景作为一致性保证。这里分类依据本文对前人工作的描述，未独立审查各论文，不将作者的首创性陈述升级为全面优先权结论。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-VAE -->
## 3.4 独立 VAE 编码

### 输入与步骤

原文 §3.1–3.2（p.5–7），Fig. 2–3。冻结视频 VAE；标准因果 3D VAE 首帧独立，后续每四帧形成一个潜时间步（适配的长度满足 $T'=1+(T-1)/4$）。本文把每个 $I_k$ 当作长度 1 的视频独立编码，输出 $d_z\times1\times h\times w$，$h=H/f_s,w=W/f_s$。沿时间轴拼接 K 个潜帧，附加零图像经 VAE 编码的目标占位。这里的时间轴仅是集合槽位而非物理时刻。条件分支与实际加噪潜变量不同。推理时只取生成的查询潜变量，单帧解码得到 RGB。

### 设计原因与边界

时间卷积缓存允许后续帧使用之前特征，交换输入可能改变所有后续潜变量；步长卷积还把若干不同视图的空间内容压到较少潜帧中。独立编码相当于每次重新开始首帧路径，无之前视图的缓存，保证每个视图的编码不随其他视图的排列改变。保留的是逐视图潜变量与相机的对应而非原图无损：空间 VAE 仍有压缩与重建误差。跨视图关系并未被禁止，而移交给 DiT 的联合注意力；这是把几何关联从固定时序压缩转到学习的集合交互中。Table 2 无逐视图 VAE 的 PSNR 从 15.69 降至 11.50，支持该设计在此配置下必要性。

<!-- PTH:END METHOD-VAE -->

<!-- PTH:BEGIN METHOD-CAMERA -->
## 3.5 相机几何条件

### 输入与步骤

§3.3（p.7–8）。以查询相机为参考坐标系，而非任一输入图像。将各输入 camera-to-world 外参变换到查询坐标系（按列向量约定可写 $T_q^{-1}T_k$），以输入相机平均距离为单位尺度进行归一化；原文未展开距离的确切计算式。内参将像素反投影到相机射线方向，外参变换到共同坐标。每个像素构造六维 Plücker 方向与矩 $[\hat d;o\times\hat d]$。全分辨率 $6\times H\times W$ 射线图经 factor $f_s$ 的 pixel unshuffle 得到 $6f_s^2\times h\times w$，与同视图潜变量拼接。查询相机也有相应射线条件。

### 设计原因与边界

若以第一个输入为原点，交换两个输入不只是交换张量槽位，还改变所有相机的数值表达；以固定查询为参考则排除了该人为依赖。内参决定光线的相机方向，外参决定光心与坐标方向，两者缺一不可。pixel unshuffle 不对一个局部块求均值，而把块内每条射线的位置映射到独立通道；只在 H、W 可被因子整除且没有后续截断时该重排严格可逆。它保留传入 patch embedding 的像素几何，不能保证模型此后完整保留全部射线信息。尺度归一化也并非恢复绝对物理尺度；退化基线和姿态误差的影响没有被全面量化。

<!-- PTH:END METHOD-CAMERA -->

<!-- PTH:BEGIN METHOD-DIT -->
## 3.6 DiT 适配

### 输入与步骤

§3.3–3.4，Fig. 2。网络输入含三部分：加噪潜变量 $z_t$、由图像条件潜变量与二值输入/目标掩码构成的 $y$、逐视图射线 $r$。沿通道拼接，沿槽位轴包含 K+1 视图。重新初始化 3D 卷积 patch embedding 以容纳扩展通道；预训练 DiT 其余权重冻结，用注意力及 FFN 的 LoRA 适配。删除 3D RoPE 的时间分量而非空间分量或扩散步条件；停用 Wan2.1-I2V 首帧 CLIP 条件，以空文本作为全局文本条件。通过联合注意力，查询 token 可利用所有观测视图；预测后舍弃上下文输出，仅目标进入监督与最终解码。

### 设计原因与边界

移除时间位置编码意味着几何身份来自相机射线而非任意帧号。共享的逐 token 运算和无索引偏置的注意力具有置换等变性：输入改变排列时，对应输出随之排列；固定查询槽位则给出查询输出不变。不能把等变性直接扩大到任意 3D 卷积 patch embedding、掩码或随机采样：时间核若混合相邻槽位仍可破坏性质，原文未列出完整核参数。Table 3 实际显示很小而非严格为零的输出波动，是经验稳定性证据。删除首帧 CLIP 避免通过额外图像分支赋予首输入不对称权重；没有单独 CLIP 消融，不能定量分离该操作贡献。

<!-- PTH:END METHOD-DIT -->

<!-- PTH:BEGIN METHOD-TRAIN -->
## 3.7 训练与推理

### 输入与步骤

§3.4、§4.1 及附录 A.2、A.3、C：从 DL3DV 场景采 K+1 帧，80% 从整段视频随机顺序均匀采样，20% 从局部时间窗口采样；分别覆盖大基线和附近视点。主配置 K=6，目标 GT 经冻结 VAE 获得干净潜变量，在训练时与高斯噪声按流匹配插值。DiT 预测速度，仅查询槽位计算平方误差。训练 LoRA（r=32，attention/FFN）及全参数 patch embedding，约 1% 参数更新。先 192×336（A6000），再 480×832（H100）；作者报告 1K 场景、30K 迭代。推理查询 GT 不可用，利用已知图像条件、相机、掩码和噪声反向生成查询潜变量后解码。原文未给完整采样求解器配置。

### 设计原因与边界

仅目标损失使更新指向条件补全而非复现已知输入；上下文 token 仍通过注意力参与目标预测，也可沿梯度影响共享 LoRA 参数。训练速度场与测试生成分开理解：前者已知 GT 构造监督，后者由噪声出发估计 GT 的潜空间样本。附录 A.2 使用固定 T=10、随机 3–9 输入、其余为目标的混合 m-to-n 训练，缓解单目标模型直接多目标推理的退化；多查询时如何选共同参考相机未明确。LoRA 节约可训练参数，不等价于预训练或推理成本小；单目标/多目标训练预算亦非严格等量控制。

<!-- PTH:END METHOD-TRAIN -->

<!-- PTH:BEGIN MATH-LATENTS -->
## 3.8 潜变量公式

### 原文公式与定义

§3.1–3.2（p.5–7）：令 $I_k\in\mathbb R^{3\times H\times W}$，$f_s$ 为空间缩小因子，$h=H/f_s,w=W/f_s$，$d_z$ 为潜变量通道数。原文式 (1)–(4)：

$$z_k=\mathcal E(I_k)\in\mathbb R^{d_z\times1\times h\times w},\quad k=1,\ldots,K.\tag{1}$$

$$z_{\mathrm{ctx}}=[z_1;\ldots;z_K]\in\mathbb R^{d_z\times K\times h\times w}.\tag{2}$$

$$z_{\mathrm{all}}=[z_{\mathrm{ctx}};z_{\mathrm{tgt}}]\in\mathbb R^{d_z\times(K+1)\times h\times w},\quad z_{\mathrm{tgt}}=\mathcal E(0).\tag{3}$$

$$\hat I_k=\mathcal D(z_k),\quad k=1,\ldots,K+1.\tag{4}$$

分号在式 (2)(3) 中代表时间槽位拼接。标准视频 VAE 的 $T'=1+(T-1)/4$ 适用于其支持的序列长度；独立编码使每个视图各占一槽，不再发生跨视图时间降采样。

### 补充推导与条件

定义 $F(S)=[\mathcal E(I_1);\ldots;\mathcal E(I_K)]$。对槽位排列矩阵 $P$，同一确定性逐视图编码满足 $F(PS)=PF(S)$。因此编码本身严格说是等变：集合内容不变，但数组顺序随输入变。若 VAE 使用随机采样，则需要相应噪声跟随视图一起排列，或只论分布。式 (3) 的 $\mathcal E(0)$ 是条件占位，不保证数值为零，也不是目标 GT 或扩散噪声。式 (4) 描述逐帧解码机制；实际目标合成只需要解码预测的查询槽，而不能把输入占位直接解码当作预测。独立编码不意味着跨视图无交互，后续 DiT 是信息交换位置。

<!-- PTH:END MATH-LATENTS -->

<!-- PTH:BEGIN MATH-GEOMETRY -->
## 3.9 相机射线公式

### 原文公式与定义

§3.3 p.7–8：每台相机的内参为 $K_k$，外参为 $T_k\in SE(3)$。对像素 $(u,v)$，原文定义 $p(u,v)=[\hat d;o\times\hat d]\in\mathbb R^6$，$\hat d=d/\|d\|$。以查询视图定义坐标并归一化输入相机平均距离。pixel unshuffle 将射线图从 $6\times H\times W$ 变为 $6f_s^2\times h\times w$。原文式 (5)：

$$x=[z_t;y;r]\in\mathbb R^{D_{\mathrm{in}}\times(K+1)\times h\times w}.\tag{5}$$

这里 $z_t$ 为加噪潜变量，$y$ 为图像条件与输入/目标二值掩码的拼接，$r$ 为重排后的射线条件。原文未指定掩码通道数的完整实现细节，故不填造 $D_{\mathrm{in}}$ 数值。

### 补充推导与条件

采用列向量、camera-to-world 约定，$T_q^{-1}T_k=[R'_k,t'_k;0,1]$。归一化尺度前有 $o_k=t'_k$，$d_k=R'_kK_k^{-1}[u,v,1]^\top$，再将 $d_k$ 归一化。正尺度归一化只改变光心和 Plücker 矩的尺度，方向不变。若平均距离是关于所有输入的对称函数，它不随重排变化；原文没有展开距离具体分母。Plücker 矩满足 $\hat d^\top(o\times\hat d)=0$，沿同一射线改变原点 $o+\lambda\hat d$ 时矩不变。该表示描述几何直线，不给出表面交点或深度。

设原射线图 $R[c,u,v]$ 的两个空间索引按行列约定；一种重排索引为 $R'[c f_s^2+a f_s+b,i,j]=R[c,f_si+a,f_sj+b]$，$0\le a,b<f_s$。每个原元素都有唯一新索引且元素总数不变，因此重排本身不丢像素射线；之后的 patch embedding 并不必然可逆。这里的索引公式为补充推导，不是作者给出的实现代码。

<!-- PTH:END MATH-GEOMETRY -->

<!-- PTH:BEGIN MATH-FLOW -->
## 3.10 流匹配与监督

### 原文公式与定义

§3.1 p.5、§3.4 p.8：在冻结 VAE 得到的干净潜变量 $z_0$ 上定义线性流匹配路径

$$z_t=(1-t)z_0+t\epsilon,\quad \epsilon\sim\mathcal N(0,I),\quad t\in[0,1].$$

速度目标 $v_t=\epsilon-z_0$，网络记为 $\epsilon_\theta(z_t,t,c)$（作者用 epsilon 符号但预测的是速度而非单独噪声）。目标掩码损失为

$$\mathcal L=\mathbb E_{t,\epsilon,z_0}\left\|\epsilon_\theta(z_t,t,c)-(\epsilon-z_0)\right\|_{\mathrm{tgt}}^2.$$

$c$ 汇集条件潜变量、掩码、射线图与文本 embedding。损失只取目标槽位。LoRA r=32 用于注意力与 FFN，patch embedding 全梯度更新。

### 补充推导与条件

对固定的 $z_0,\epsilon$ 求导得 $dz_t/dt=\epsilon-z_0$，端点 $z_{t=0}=z_0,z_{t=1}=\epsilon$。因此采样从 $t=1$ 的噪声沿学习速度场反向积分到 0；一般不是单次减去噪声。补充欧拉例：$z_{t-\Delta t}\approx z_t-\Delta t\epsilon_\theta(z_t,t,c)$，实际求解器原文未明确。这里 $t$ 是生成噪声时间，不是视频帧槽位，移除 temporal RoPE 不移除 $t$。

令掩码 $M$ 仅在目标位置为 1，则损失写作 $\|M\odot(\epsilon_\theta-v_t)\|_2^2$（归一化细节未给出）。对参数 $\theta$ 的梯度由目标残差和目标速度对参数的雅可比相乘而来；上下文虽然没有直接残差，仍通过跨视图注意力影响该雅可比。补充 LoRA 形式 $W'=W+BA$（缩放可并入系数，原文未给），$A\in\mathbb R^{r\times d_{in}}, B\in\mathbb R^{d_{out}\times r}$，训练参数量 $r(d_{in}+d_{out})$，不是训练完整 $d_{in}d_{out}$ 权重。此参数节省不等于推理 FLOPs 同比例下降。

<!-- PTH:END MATH-FLOW -->

<!-- PTH:BEGIN MATH-PERMUTATION -->
## 3.11 置换性质推导

### 原文公式与定义

原文 §3.3 仅文字说明删除 3D RoPE 时间分量，未给出展开的注意力或 RoPE 公式。以下为机制推导：标准旋转位置编码对二维通道子空间施加旋转 $R(\omega j)$，使点积包含相对索引 $j-i$：$(R(\omega i)q_i)^\top R(\omega j)k_j=q_i^\top R(\omega(j-i))k_j$。若索引为任意输入帧号，交换视图会改变其相对相位；删除时间旋转可消除此特定依赖，空间旋转仍可保留。

对共享投影 $Q=XW_Q,K=XW_K,V=XW_V$，注意力 $A(X)=\mathrm{softmax}_{row}(QK^\top/\sqrt d)V$。设 $P$ 为完整 token 的排列矩阵，且几何/掩码/空间坐标一同排列，则

$$A(PX)=\mathrm{softmax}_{row}(P QK^\top P^\top/\sqrt d)PV=PA(X).$$

这是无额外不对称位置偏置的注意力的等变性；若 $P$ 只交换输入槽、固定查询槽，取查询行便得到不变输出。

### 补充推导与条件

上述推导要求 patch embedding 不跨帧不对称混合，条件与特征使用相同排列，任何注意力掩码也满足共轭变换，MLP、归一化等逐 token 共享；对空间 RoPE 需要同一视图块内空间索引保持对应。完整扩散轨迹还需初始噪声相应排列，或讨论交换对称的噪声分布而非固定样本；有限精度计算可产生微小差异。原文未给完整 patch 核、随机性和注意力实现细节，故不能由本推导宣称整个模型严格不变。Table 3 测得 PSNR 标准差 0.0081，明显低于原始时间 RoPE 的 0.3106 和首视图坐标的 0.4730，支持实测近似不变，而非数值严格为零。

<!-- PTH:END MATH-PERMUTATION -->

<!-- PTH:BEGIN EXP-SETUP -->
## 4.1 实验协议

### 实验设计与结果

§4.1–4.2，p.9–10：下游训练用 DL3DV-10K 的 1K 场景子集，测试 DL3DV-Benchmark 与 Mip-NeRF 360，遵循 SEVA 划分；主设定六输入、单目标。Wan2.1-I2V-14B 先192×336再480×832训练，LoRA r32，约1%参数更新；CogVideoX-5B 为另一骨干。比较 LVSM、E-RayZer 回归模型，EscherNet（按其方案在10K场景微调）、Aether、SEVA（80K场景）。主表及三视图表将各方法输出 resize 或中心裁剪至480×480后评估；不改变各自原生生成设置。

PSNR（dB，越高越好）衡量像素误差，SSIM（越高越好）衡量结构相似性，LPIPS、DreamSim（越低越好）是学习特征空间的感知距离。补充定义 $\mathrm{PSNR}=10\log_{10}(L^2/\mathrm{MSE})$，$L$ 为像素动态范围；原文没有明确颜色空间、汇总方式等细节。低分辨率消融另用192×336，不能直接用其 PSNR 与主表比较。

### 证据解释与局限

统一评估尺寸减少分辨率不一致造成的部分误差，但裁剪可能改变有效视野，不能确保所有生成输入、训练预算和预训练语料都相同。场景数只表示下游多视图训练量。跨骨干比较不能把模型参数量、预训练视频与网络结构三个因素分离；回归与生成模型也面临不同损失偏好。原文未报告完整逐场景方差、显著性或总预训练算力。图像相似性与真实三维几何正确性需分开评价。

<!-- PTH:END EXP-SETUP -->

<!-- PTH:BEGIN EXP-MAIN -->
## 4.2 主实验

### 实验设计与结果

§4.2、Table 1（p.10）、Fig.4（p.12）及 Fig.6（p.23）。详细数据见本报告 Table 1 单元。六输入时，Wan 版本 DL3DV PSNR/SSIM/LPIPS/DS 为17.18/.445/.223/.066，SEVA 为16.15/.470/.253/.088；Mip360 为15.64/.279/.365/.111，对 SEVA 14.59/.294/.372/.137。两数据集感知距离均更低，SSIM却低于SEVA；Mip360 PSNR最高为E-RayZer的16.56。Cog版本相对同骨干Aether，DL3DV四项均改善，Mip360 SSIM下降.011。由 Cog 到 Wan，八项全部改善，支持骨干能力迁移的经验趋势。

Fig.4、6 的同一目标视角并列比较呈现纹理补全、较大视角变化下的结构和细节差异：回归结果倾向模糊，生成方法可给出更锐利纹理，但真实 GT 仍是对照。主结果使用统一480×480评估，低分辨率消融不混入此比较。

### 证据解释与局限

生成模型可能减少感知距离却出现偏离GT的细节，回归均值则可能提高PSNR但导致模糊。因而作者的“competitive”不能扩写为全指标全面优于所有方法。骨干更换同时改变模型家族和预训练，不能认定性能改善完全来自规模。图像质量的提高有助于但不保证三维一致性；附录A.4才通过共同重建器提供额外代理指标。Fig.4、6 是选择性展示，不代表每个场景必然改善。

<!-- PTH:END EXP-MAIN -->

<!-- PTH:BEGIN EXP-ABLATION -->
## 4.3 消融与数据规模

### 实验设计与结果

§4.4，Tables 2–4 与 Fig.5（p.11–15）。Table 2 全模型低分辨率PSNR/SSIM/LPIPS/DS=15.69/.326/.246/.114；取消逐视图VAE=11.50/.147/.676/.656；不用pixel unshuffle=13.53/.212/.359/.129；不用仅目标损失=15.08/.287/.270/.116；以PRoPE代替Plücker=14.91/.289/.315/.174；保留原始时间RoPE=15.30/.300/.265/.122。逐视图编码带来的单项差距最大。Table 3 的排列PSNR标准差由首输入参考系.4730、原始RoPE .3106降至.0081，均值分别15.67、15.61、16.21。Table 4 在有序评估中，无序模型15.69 dB优于有序模型15.04 dB，四项指标均有改善。Fig.5 训练场景20、50、1000、2000时PSNR图上约14.6、14.9、15.3、15.9 dB，EscherNet 10K约12.1 dB；读数为图估计，不作精确表值。所有数据见图表单元。

### 证据解释与局限

逐视图VAE消融同时改变时间压缩与因果依赖，不能独立衡量二者。pixel unshuffle 与PRoPE结果是具体条件编码实现的比较，而非证明所有插值或注意力相机条件均劣。仅目标损失消融支持监督焦点，但目标与上下文梯度权重也改变。Table 3 更接近排列稳定性实验，非严格零差异。Table 4 同时修改架构与采样；不能把改善完全归于随机顺序。Fig.5 显示较小下游数据规模可训练，20 vs 10K为500倍场景数量差，预训练成本不受控，不意味着只需500分之一总训练数据。没有隔离骨干预训练先验的对照。

<!-- PTH:END EXP-ABLATION -->

<!-- PTH:BEGIN EXP-APPENDIX -->
## 4.4 附录实验

### 实验设计与结果

附录A.1–A.4、B、C（p.20–24）。Table 5 三输入：Wan DL3DV 14.73/.334/.351/.129，Mip360 14.14/.235/.491/.166，两套PSNR与DS领先，但SSIM低于SEVA，Mip360 LPIPS也较差。Table 6 单目标训练直接6-to-1–4的PSNR 15.69/14.84/14.62/14.18，经固定T=10、3–9输入混合训练变为16.27/16.20/16.42/16.23，其他指标也改善。该表为192×336。

Table 7 单A6000，5/10/25/50步约22/42/99/177s，对应PSNR18.13/17.72/17.35/17.18，DS .114/.089/.070/.066；SEVA约24s，ViewCrafter约247s，GEN3C+VGGT约1460s。作者将轨迹式基线适配为两个最近视图插值；报告相对SEVA下游7.8倍更少可训练参数、30倍更少迭代、80倍更少场景。Table 8 对相同输入加入预测或GT目标，使用MapAnything重建；相比SEVA，AbsRel .110→.073，RMSE .857→.789m，pointmap总体 .251→.176m，但δ阈值正确率 .947→.940。Fig.7 给出COLMAP估计姿态的非基准示例。附录C低分辨率A6000、高分辨率H100训练。

### 证据解释与局限

三输入不是对所有稀疏程度泛化的保证；混合训练额外预算与多目标监督共同变化，几何正则解释仍是作者推测。少步结果有更高PSNR但较差感知质量，不能简单用时间或单指标判断；50步远慢于SEVA，轨迹式基线只用两最近输入的适配也改变了原任务。MapAnything GT目标重建是代理参考，不是真实扫描几何；不同生成纹理可能影响同一重建器，不能直接宣称真实3D误差降低。Fig.7 无噪声量化或系统误差曲线。DMD加速仅作为潜在方向提出，未实际验证。

<!-- PTH:END EXP-APPENDIX -->

<!-- PTH:BEGIN SYNTHESIS-EVIDENCE -->
## 5.1 贡献与证据再评价

### 主张—证据矩阵

| 主张 | 直接证据 | 支持与边界 |
|---|---|---|
| 跨帧VAE压缩不适合无序视图 | Fig.3、Table 2 | 结构理由明确，性能对照支持；压缩与因果依赖未分离 |
| 查询系与去时间RoPE稳定输入排列 | Table 3、§3.3 | 强经验支持；非零标准差，未证明全网不变 |
| 视频先验支持低数据微调 | Table 1、Fig.5、A.3 | 下游场景/迭代更少；预训练语料、架构与规模混杂 |
| 更强视频骨干有益 | Table 1 Cog→Wan | 两数据集八项改善；非单一规模变量对照 |
| 支持可变输入/多目标 | Tables 5–6 | 3输入及6-to-1–4实测；不能推断任意数量或无界泛化 |
| 补全改善下游几何 | Table 8 | 多数代理误差改善；δ下降且无真实几何GT |
| 对姿态噪声有鲁棒性 | Fig.7 | 定性可行，未有系统受控噪声实验 |
| 视频模型易于忘记时间 | 结论、Tables 2–4 | 无序NVS适配支持；未测试原动态能力是否被擦除 |

### 创新性质与局限

方法创新是三项关键适配的组合：独立VAE编码、查询中心射线条件、无时间位置偏置；额外首帧CLIP禁用、目标损失、LoRA及随机采样形成可训练系统。VAE、视频DiT、RoPE、Plücker、pixel unshuffle、LoRA、flow matching均为已有技术，论文没有提出新成像方程或显式三维重建表示。理论层面给出任务匹配的结构理由，但完整不变性证明与严格因果识别未提供；最强贡献是方法与实验证据。作者声称首个此类方法属其文献判断，本次不独立验证。生成先验使欠约束区域更可信，却也可能产生不真实的结构与细节。

<!-- PTH:END SYNTHESIS-EVIDENCE -->

<!-- PTH:BEGIN SYNTHESIS-COVERAGE -->
## 5.2 开放问题与覆盖

### 尚未解决的问题

1. 固定同一骨干、预训练预算和微调集，分别去除跨帧压缩与缓存依赖，能否重现各自增益？
2. 在固定噪声、严格重排相机/掩码/潜变量的条件下，逐层记录最大误差，是否能定位Table 3残余波动来源？
3. 按输入数量、视点覆盖、姿态噪声与遮挡程度分层，用真实几何GT检验生成外观与几何正确性的关联，避免仅用重建代理。
4. 混合多目标训练与等迭代的单目标训练比较；同时检查跨目标3D一致性，隔离额外训练预算。
5. 测试适配后模型在原视频动态任务上的表现；若动态能力保留，则“忘记时间”应解读为任务输入顺序的解耦，而非动力学知识消失。

### 覆盖与来源

本报告使用同一版本 arXiv:2604.08500v2 PDF（24页）为最终依据，Source与HTML仅辅助核对。覆盖标题、摘要、引言、相关工作、§3.1–3.4、§4及结论、附录A.1–A.4/B/C；参考文献以正文关联的编号定位，未逐篇独立审查。7幅Figure及8张Table均设独立精读单元，含原题注、中文、技术作用、证据边界；式(1)–(5)、无编号流匹配/目标损失、相机射线、压缩公式均有单元；注意力等变与几何重排等补充推导已标出条件。Fig.5图读数为近似值。Source缺独立版本戳、HTML缺字体资源；原始PDF身份与覆盖已验收。此报告不含独立Review、源码复现或对引用论文的外部核验。研究笔记和公开版须继承上述科学边界，不将作者假说写作证明。

<!-- PTH:END SYNTHESIS-COVERAGE -->



---

# 第二部分：学习与研究笔记

# FrameCrafter 学习与研究笔记

> 精读报告 SHA-256：b55f7af4a1d8768fce32a5ca79bb44f67e19c94d196c04d5c8f856d17a2c76ad
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：关键概念内化与理论拓展

### 1. 视图集合不是时间序列

【依据】精读 §3.4、§3.6、§3.11，原论文 §3.2–3.3、Fig.3、Table 3。物理拍摄过程可能有时间顺序，但静态场景的多视图推断只依赖图像及其相机的对应关系。例：同一桌子的左、右、后视图组成集合，输入左→右→后和后→左→右应给同一查询相机相容的条件输出。对视频 VAE，序列的相邻性却影响缓存、卷积窗口、压缩；对普通视频 DiT，相对帧序号影响位置旋转。FrameCrafter 将这两种不必要的先验拆除，而非禁止跨视图信息交互。

【补充推导】令 $S$ 是图像—相机对，$P$ 是只置换观测的矩阵，$q$ 固定。更合适的判据为 $p_\theta(I_q\mid PS,q)=p_\theta(I_q\mid S,q)$；若用同一噪声且其观测槽也随排列变换，可以检验更强的逐样本误差 $D=\|F(PS,q,P\epsilon)-F(S,q,\epsilon)\|$（输出仅查询）。对仅含共享逐视图特征及池化的 Deep Sets 型模型，$F=\rho(\sum_k\phi(I_k,c_k),q)$ 自然满足不变；其困难在于固定汇总可能丢空间对应。Attention 可形成按查询 token 的自适应加权，保留更丰富的交互，但必须检查位置编码、掩码和 patch 操作是否满足等变。Table 3 测的是指标标准差，不能替代上述逐像素或逐层检验。

反事实：若输入是动态场景的多个时刻而输出必须对应指定时刻，时间便成为物理条件而非冗余索引。正确方案应将真实时间作为与相机绑定的观测属性随排列，而不一定使用任意槽位号；删除时间信息可能造成运动状态不可辨识。论文未验证该拓展。

### 2. 射线条件不是三维对应

【依据】精读 §3.5、§3.9；原文 §3.3。Plücker 坐标编码成像几何中的一条线，但未知表面深度仍在连续射线上。令归一化方向 $d$，点 $X=o+sd$；由于 $(o+sd)\times d=o\times d$，相同的射线表示不区分所有可能的 $s$。两个相机的射线若有可靠像素对应且有非退化交角，可以三角化；网络并未显式执行此过程。因而相机条件能够限定目标投影几何，却不能独立恢复缺失内容。

【构造例】在双视图平行相机、已知焦距 f、基线 b、同一可见点满足针孔模型时，视差 $\Delta u=fb/Z$，$Z=fb/\Delta u$。若测量视差误差为 $\delta u$，一阶 $|\delta Z|\approx Z^2|\delta u|/(fb)$，远点/小基线更不稳定。大基线则可能增加遮挡和对应困难。此几何关系说明“更多场景先验”不能替代真实观测约束；它也解释为何相机覆盖和数量应分开评价。此式是背景推导，不是作者模型的显式算法。

### 3. 生成先验与证据：从点估计到条件分布

【依据】精读 §3.1–3.2、§3.10、§4.2、§4.4。给定输入 $S$，若两个可能的目标图像 $I_A,I_B$ 具有相同观测条件，最小平方误差的确定性估计在等概率条件下为 $(I_A+I_B)/2$；若两者纹理错位，均值会模糊。生成式模型可从条件分布采样一个清晰候选，未必恰是实际场景。表1与表7中像素指标、感知指标不完全一致，符合此机制的直觉，但不能据此证明模型确实校准了条件分布。

补充概率视角：若显式三维场景为 $X$，观测成像模型为 $p(S\mid X)$，理想后验为 $p(X\mid S)\propto p(S\mid X)p(X)$；本文直接生成目标图像，不显式求解该后验，也没有显式场景变量供检查。多个采样的方差可以作为待验证的歧义信号，但低方差可能是模式坍塌，高方差也可能来自采样噪声；必须使用真实误差校准。

### 4. 训练监督与推理条件的分离

【依据】精读 §3.7、§3.10、Fig.2、Table 2。GT 查询只用来构造训练的干净潜变量与速度监督；零图像编码属于条件，不是该速度监督的替代。目标槽有直接损失，观测槽为条件并参与交互。可以设想加输入重建损失 $L=L_q+\lambda L_{ctx}$；提高 $\lambda$ 会使有限适配参数兼顾复现已知内容。表2显示本论文的非 target-only 配置较差，但未报告完整 λ 曲线；不能推断所有辅助监督都无益。

【设计替代】若有可验证的深度，可在目标损失之外加入跨视角重投影一致性；仅对可见区域施加，遮挡和反射区域应区分。代价是需要可用几何以及可靠对应，可能偏离本文无需预重建的优势。值得检验其是否降低外观正确而几何错误的概率，而不是只看图像感知评分。

### 5. 轻量训练不等于轻量模型

【依据】§4.1、Fig.5、Table 7。冻结骨干加LoRA与全梯度 patch embedding，使训练参数大幅减少，但14B骨干依然需要前向计算。可训练参数、训练显存、总训练FLOPs、数据标注成本、推理延迟是不同量。论文中5步22s对SEVA24s、50步177s，感知质量随步数改善但PSNR下降。部署应按应用成本选择质量—速度曲线，不因1%参数训练就推断实时性。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Problem & Task（无序稀疏视图生成）
- **Keywords:** Sparse-view Novel View Synthesis; Set-conditioned Generation; Permutation Invariance
- **Highlight:** 将离散相机查询视为低帧率视频补全，不要求预重建或连续轨迹。条件是图像—相机对的集合而不是时间序列。
- **Evidence:** §1、§3.1，Fig.1–2；精读 §3.1。
- **Connection Points:** 已知姿态、输入集合表示、输出图像与显式场景、轨迹约束。
- **Open Questions:** 在相同可见性与相机覆盖下，直接生成与显式场景重建的误差来源如何区分？

### Topic: Observation & Prior（视频生成先验）
- **Keywords:** Video Diffusion Prior; Image Diffusion; LoRA; Data Efficiency
- **Highlight:** 冻结视频骨干，适配输入层和LoRA；下游1K场景可获竞争力结果。Fig.5支持低数据微调，但不控制总预训练成本。
- **Evidence:** §3.4、§4.1–4.2、Table 1、Fig.5、A.3。
- **Connection Points:** 预训练模态、监督规模、可训练参数、场景数与预训练成本。
- **Open Questions:** 同骨干结构与匹配训练预算下，视频预训练相对图像预训练的独立收益是什么？

### Topic: Scene Representation（逐视图潜变量）
- **Keywords:** Causal 3D VAE; Temporal Compression; Per-view Encoding; Latent Diffusion
- **Highlight:** 把每张视图按独立单帧视频编码，不把远距离视图压到共享潜时间步。后续DiT承担跨视图交互。
- **Evidence:** §3.2，Eq.1–4、Fig.3、Table 2。
- **Connection Points:** 空间与时间压缩、视图特征对齐、跨视图特征融合位置。
- **Open Questions:** 视图独立编码与受几何约束的联合编码之间是否存在更优信息效率折中？

### Topic: Geometry Inference（射线条件与参考系）
- **Keywords:** Plucker Ray; Query-centered Coordinates; Pixel Unshuffle; Camera Conditioning
- **Highlight:** 以固定查询相机为参考，像素射线重排至潜分辨率，去掉因首输入导致的参考系依赖。射线条件未显式给出深度或对应。
- **Evidence:** §3.3、Eq.5、Tables 2–3。
- **Connection Points:** 内外参编码、相机规范化、射线/姿态/注意力条件、三维对应。
- **Open Questions:** 相机噪声、尺度退化和稀疏角度覆盖分别如何影响条件生成？

### Topic: Training & Supervision（集合注意力及目标掩码）
- **Keywords:** Temporal RoPE; Permutation Equivariance; Flow Matching; Target-only Supervision
- **Highlight:** 移除时间RoPE与首帧CLIP偏置，联合预测但仅对查询计算速度损失；排列实测波动很小而非严格为零。
- **Evidence:** §3.3–3.4、Fig.2、Tables 2–4。
- **Connection Points:** 跨视图交互、位置编码、训练掩码、噪声排列、监督分布。
- **Open Questions:** 完整网络在锁定噪声后是否严格等变，剩余偏差来自哪一算子？

### Topic: Generalization（多目标与下游几何）
- **Keywords:** Multi-target NVS; Mixed m-to-n Training; Geometric Consistency; Downstream Reconstruction
- **Highlight:** 混合输入输出训练缓解多个查询同时生成的退化；MapAnything代理比较多数误差改善，但非真实三维GT评价。
- **Evidence:** A.1–A.4、Tables 5–8、Fig.7。
- **Connection Points:** 查询数量、联合监督、推理速度、真实几何与代理重建评价。
- **Open Questions:** 多目标训练在等算力对照下是否改善真实跨目标几何一致性？


## RESEARCH-DIRECTION：稀疏观测水下视觉的研究联系

研究主线：Sparse-Observation Underwater Vision（用户已明确的研究方向）。本节只讨论公开的通用科学问题和本文机制的可能迁移，不纳入私密研究方案。所有水下结论为待验证拓展，非 FrameCrafter 实验结果。

### 研究层级与任务边界

Field 是水下视觉；Task 可以是恢复、深度、介质估计、三维重建或 NVS；Task Setting 是稀疏观测（少视角、有限覆盖或低有效信息）；Research Problem 是欠约束引起的推断歧义与可靠性；Method 是用何种建模和推断机制求解，不能由任务设置预先决定。FrameCrafter 的具体任务为已知相机的离散 NVS，直接输出图像，不等同于水下场景、几何或介质参数的联合估计。迁移价值是“利用大量预训练获得的先验缓解有限观测约束”，而不是默认采用视频扩散。

### 观测—先验冲突及可迁移机制

【依据】§3.2–3.4、Table 2–3、Fig.5。独立逐视图编码保留各观测与相机的对应，能防止任意顺序引入额外混合。水下不同视图的成像介质与路径长度可能不同，因而将多个视图提前压缩到一个潜变量，或许更难分离几何、固有外观和介质变化。这里是迁移假设，而不是本文证明；若跨视图早期融合有可靠物理对齐，它也可能提高抗噪性。

【构造模型】简化水下成像可写 $I_c=J_c e^{-\beta_c d}+B_c(1-e^{-\beta_c d})$；假设均匀介质、通道常数系数与简化散射。给定单像素 $I_c$，不同 $J_c,d,\beta_c,B_c$ 可产生同值，且稀疏视图未必提供足够独立条件。该公式仅用以解释可辨识性，不是完整水下物理或本文算法。几何条件只约束射线，不能直接确定距离和介质。视频先验可以补充纹理和几何概率结构，但若预训练更偏陆地影像，可能把真实水下外观误判成退化，也可能产生符合先验但与观测不相容的细节。

在已知位姿、静态场景的水下稀疏多视图中，查询中心参考系和共享集合交互有直接的结构迁移价值；在未知相机、动态介质或随时刻变化场景中，真实时间、照明及介质状态应该作为绑定观测的条件，不应因删除任意槽位号而一并删除。真正的困难不是缺少视频帧顺序，而是观测的物理约束可能不足以区分多种解释。

### 评价原则

以已见/未见区域、几何距离、视角覆盖、介质强度及姿态误差分层。分别报告观测重投影误差、真实几何误差、清晰外观或目标水下图像误差，以及多次预测的稳定性与校准。感知分数和下游重建器代理指标不作为几何真实性的替代（本文 Table 8 正好提供该区别的例子）。比较方法前需匹配可获得的观测、先验和训练预算。该方向尚需针对水下成像与稀疏推断的独立文献调研，不能据单篇论文判断原创性。


## RESEARCH-PROBLEMS：候选科学问题与可证伪设计

以下是基于本文证据的开放问题，不是已有创新结论；跨文献查重及水下数据可用性尚未完成。优先级按科学问题清晰度、可控制程度和验证成本排序，不指定最终技术路线。

### RP-A：欠约束条件下，生成先验的补全何时能改善真实几何，何时只是外观合理？（优先）

现象：Table 1、7 的感知与像素指标并非同向，Table 8 的代理几何多数改善但δ下降；稀疏视图未观测区域天然多解。科学未知是补全的几何收益是否由新增可信信息产生，抑或只是符合重建器偏好的纹理。

可证伪假设：在固定姿态、相同稀疏输入与固定重建器下，生成目标在输入充分约束且先验相容区域改善真实几何；在未观测或强介质域移位区域，感知质量改善不必伴随几何误差下降。候选验证：用可控渲染得到真实深度、可见性与不同介质条件，从同一场景选择相机子集。对照输入-only、加GT查询（参考上界）、加生成查询、加简单插值查询，以及不同生成随机种子；所有方法用同一重建器和同样视图数（输入-only另作观测量对照）。记录区域分层真实深度RMSE、点云准确/完整性、可见区重投影误差、目标感知指标。预注册主要判断：若生成查询持续降低真实几何误差且不恶化观测一致性，支持在对应范围内有效；若只改善感知/代理误差或在未见区域系统恶化，则反驳无条件几何收益。对不同重建器交叉验证可降低代理偏好混杂。

可行性：合成有真实几何较容易控制，但真实水下数据及域差距需另测；需要独立检查预训练泄漏和合成场景真实性。不以提出特定扩散网络为问题本身。

### RP-B：稀疏观测的证据可靠性可否从先验驱动的预测分布中识别？（次优先）

现象：本文采用条件生成并测量排列稳定性，却未评价不同噪声采样的校准。几何、外观和介质可形成多种相容解释。科学未知是模型不确定性是否能指示观测约束不足，而非仅反映生成器采样行为。

假设：当相机覆盖减少或成像退化增强时，正确校准的几何置信度应下降，且高置信预测具有较低真实误差；若先验压倒观测，模型可能在错误解上过度自信。候选实验控制真实场景、位姿、介质，分别改变视图数量、基线、透射率；每一设置固定一组输入生成多次，重建或估计几何。对照单次生成、采样方差、重投影残差、基于几何可观测性指标的置信度，保持相同总测试样本及相机条件。报告误差—置信排序、风险—覆盖曲线、置信区间覆盖率（需先定义概率输出）、未见/已见区域误差。若低方差却高误差或跨介质无法校准，则反驳把采样方差直接视为可靠性的做法。若融合物理观测残差和采样信息提高风险排序，则支持进一步机制研究。

限制：没有明确后验或概率预测时不能把样本标准差当作可信区间；样本数、随机种子、重建器误差和域外数据均是混杂。需查证现有不确定性与水下可观测性研究。

### RP-C：任意顺序偏置与真实物理时空条件如何分离？（机制验证）

现象：Table 3 的顺序方差大幅减小，Fig.3说明缓存和时间压缩依赖；真实水下动态可能又依赖拍摄时间。未知是能否保留实际成像状态同时消除人为排列偏置。

假设：把真实时间、姿态和照明/介质状态作为视图属性随输入一起排列，比绝对序号编码更稳定；若动态状态缺失，完全去时序信息可能降低恢复精度。实验：静态与有受控动态介质/对象的合成场景，随机置换成对观测与真实条件，固定噪声；对照保留原时间RoPE、删除时间RoPE、绑定真实时间属性三种机制，等训练预算，分别测查询输出变化、真实图像与几何误差及动态状态误差。若有真实时间属性的模型既稳定又保留动态预测，则支持区分；若稳定性不改善或精度下降，则反证当前假设。应额外逐层测试 patch embedding、注意力、噪声映射的等变误差。

限制：动态任务超出本文；观测时间可能与位姿相关，需随机或正交设计以隔离。不能将作者关于“忘记时间”的猜测当作已有动力学分析。


## INTERNALIZATION：知识地图与理解自检

### 用自己的语言解释论文

已有视频生成网络擅长从部分画面生成合理的跨帧内容，却误把输入帧当成时间序列。对于静态多相机照片，真正有意义的是各像素对应的空间射线及可见性；输入文件编号没有物理意义。因此保留预训练视频模型的跨帧生成能力，同时让 VAE 按视图独立编码，令相机射线在查询坐标系中形成条件，删除时间位置偏置及首帧专用条件。训练时只让网络预测目标潜变量的流匹配速度，推理时根据多视角条件从噪声生成目标。结果表明这种适配在给定数据与评价范围内有竞争力，尤其感知质量、低下游数据量及输入排列稳定性，而非重建真实世界的确定性保证。

### 技术与概念地图

| 逻辑阶段 | 核心变量或机制 | 后续作用 | 关键验证 |
|---|---|---|---|
| 稀疏观测 | 图像—相机对 S、查询 q | 给条件分布而非唯一3D | Fig.1、§3.1 |
| 输入独立化 | E(I_k)，一图一潜帧 | 保留图像—射线对应 | Fig.3、Eq.1–4、Table2 |
| 几何条件 | 查询系、Plücker、unshuffle | 赋予几何身份 | Eq.5、Table2–3 |
| 集合交互 | 去时间RoPE、禁首帧CLIP | 条件融合而非顺序依赖 | Table3–4 |
| 目标生成 | flow matching、LoRA、mask | 生成查询潜变量 | Fig.2、Table2 |
| 泛化和评价 | 多目标、质量、时间、代理几何 | 验证适用范围 | Tables1、5–8，Fig.5–7 |

### 自检：必须能回答的误解

- “逐视图编码就没有跨视图交互？” 否。VAE 的早期混合被消除，DiT 的注意力仍联合处理条件和目标；区分编码独立与推断独立。
- “去 temporal RoPE 就不用扩散时间？” 否。视频帧序号与流匹配噪声变量 t 是两个轴；后者是速度场条件。
- “零占位是噪声或零潜变量？” 否。是 E(0) 条件，训练 GT 速度目标和推理噪声分别承担其他职责。
- “射线图就是点云？” 否。每条射线含方向与矩，缺失交点深度；未建立显式3D表示。
- “置换不变等于相机位姿无关？” 否。交换图像及相机对不应改变查询输出；只交换图像、破坏相机对应，理应影响结果。
- “高PSNR意味着几何更可靠？” 不一定。表7少步更模糊却PSNR高；表8代理深度和点图也不能替代真实几何GT。
- “1K训练场景意味着总训练数据更少？” 仅能说下游带姿态场景量更少，预训练数据规模不控制。
- “作者证明视频模型不会理解动态？” 否。只测无序NVS适配，动态知识是否仍保留是待验证问题。

### 研究启发与开放边界

可迁移的是消除与任务无关的结构偏置、以物理观测属性为条件、区分真实证据和先验生成，及对不确定性与数据效率的分解评价。RP-A/B/C 分别针对几何收益、可靠性信号及时间条件，均需额外验证。对本文仍应追问：哪些预训练数据形成几何先验？多查询是否需要共享空间表示？不同基线、遮挡、输入覆盖下的失败模式如何？这些问题不是现有结果的否定，而是其可推广边界。


## GLOSSARY：核心术语

| 英文 | 中文 | 技术含义与辨析 |
|---|---|---|
| Novel View Synthesis (NVS) | 新视角合成 | 已知观测与查询相机生成图像；不必输出显式3D。§3.1 |
| Sparse-view | 稀疏视角 | 观测少或覆盖不足，未见区域多解；不等同低像素分辨率。 |
| Camera-to-world (c2w) | 相机到世界变换 | 将相机坐标的点/方向变换到共同坐标；与 world-to-camera 互逆。§3.3 |
| Query-centered coordinates | 查询中心坐标系 | 以目标相机为参考，避免首输入参考系随排列改变。 |
| Plücker coordinates | 普吕克坐标 | 方向与矩组成六维直线表示；不同于射线上的深度或3D点。 |
| Pixel Unshuffle | 像素反重排 | 将空间块搬入通道，元素总数不变；不同于插值降采样。 |
| Variational Autoencoder (VAE) | 变分自编码器 | 把像素映射为紧凑潜变量，解码恢复图像；与DiT生成器职责不同。 |
| Causal Conv3D | 因果三维卷积 | 时间上使用当前/过去而非未来帧；缓存带来输入顺序依赖。 |
| Temporal compression | 时间压缩 | 减少潜帧数，不是普通空间降采样；逐视图VAE避免该项跨视图混合。 |
| DiT (Diffusion Transformer) | 扩散Transformer | 潜空间的去噪/速度预测网络，以注意力交互条件。 |
| RoPE | 旋转位置编码 | 相对索引影响注意力点积；本文删除时间分量，保留其他位置/噪声步条件。 |
| Permutation equivariance | 置换等变 | F(PX)=PF(X)，输出对应槽位同序重排。 |
| Permutation invariance | 置换不变 | 固定查询输出不随输入集合排列改变；分布不变与固定噪声样本一致有区别。 |
| Flow Matching | 流匹配 | 学习将噪声与数据潜变量连接的速度场；t不是视频时间。 |
| Target-only supervision | 仅目标监督 | 速度损失掩码只取查询槽，上下文仍影响预测和梯度。 |
| LoRA | 低秩适配 | 冻结原权重并训练低秩增量；不等于减少所有推理参数。 |
| LPIPS / DreamSim | 学习感知距离 | 比较图像特征，低为好；不直接测真实3D。 |
| MapAnything proxy | 下游重建代理评价 | 同一重建器输入生成/GT查询形成参考差异；不是扫描GT。 |
| Observability / Identifiability | 可观测性 / 可辨识性 | 前者关心观测能提供什么信息，后者关心未知量能否唯一确定；先验补全不消除所有歧义。 |
| Calibration | 置信度校准 | 预测置信度与真实误差频率对应；不同于仅测试排列稳定性。 |


## SOURCES：证据来源与边界

- 论文：Qi Wu et al., *FrameCrafter: Novel View Synthesis as Video Completion*, arXiv:2604.08500v2，2026-08-31（ECCV 2026 接收）；[arXiv](https://arxiv.org/abs/2604.08500)，[PDF](https://arxiv.org/pdf/2604.08500v2)，[项目主页](https://frame-crafter.github.io/)。核验时为最新版。
- 主证据为24页同版本PDF，SHA-256 `aafbd03fa7e7f929e7c9f1659a2a0f42a50dbde348b6c893829ee0b63df39109`；Source/HTML只作辅助。Source无独立版本戳，HTML缺字体，PDF内容完整。
- B3冻结精读报告 SHA-256 `b55f7af4a1d8768fce32a5ca79bb44f67e19c94d196c04d5c8f856d17a2c76ad`；本文机制与实验依据其§2–5及实际PDF。报告的5个编号式及流匹配等无编号式是原文数学依据；注意力、三角化、后验等推导属补充背景。
- CROSS-PAPER 仅建立当前论文主题索引，未独立阅读或比较引用文献。水下应用、RP候选、校准、动态时间属性等是研究假设，未验证、未查重，不作为本文贡献。无用户私密实验结果或未公开研究方案纳入本笔记。
- 本笔记是生产上下文内的研究与学习材料，不是独立审校结果；未复现代码、训练或真实站点部署。后续实质讨论可按新单元增补，冻结报告及发布快照不得原地改写。



