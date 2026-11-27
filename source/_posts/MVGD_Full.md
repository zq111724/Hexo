---
title: "Zero-Shot Novel View and Depth Synthesis with Multi-View Geometric Diffusion（完整精读与研究笔记）"
date: 2026-11-27 06:09:39
index_img: "/MVGD/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/MVGD/)

# 第一部分：论文精读报告

# MVGD 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2501.18804v1（2025-01-30）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2501.18804) | [PDF](https://arxiv.org/pdf/2501.18804v1) | [Project Page](https://mvgd.github.io/)

<!-- PAPER-METADATA END -->


> 论文：Zero-Shot Novel View and Depth Synthesis with Multi-View Geometric Diffusion
> 原件：用户提供 arxiv_2501.18804v1.zip；PDF 20 页为权威，LaTeX 与 HTML 为同篇辅助。
> 范围：正文 §1–5、附录 A–E、参考文献目录。原文事实、背景知识、合理推断、独立评价及尚未证实分开。

<!-- PTH:BEGIN INIT -->
## 0. 阅读范围与证据规则

**来源与版本**：原始 ZIP 经 CRC 与逐文件 SHA-256 复核；PDF 首页印记明确为 arXiv:2501.18804v1（2025-01-30）。全篇 20 页均提取文本及渲染。Source 的 main.tex 标题、五位作者和方法章节与 PDF 一致，安全解包 94 个文件；无独立 arXiv 版本印记，不能仅以文件名证实版本，下载记录指向同一 v1，作为辅助静态证据。HTML 标题及 ID 对应，外部装饰资源部分缺失，不以 HTML 替代 PDF。arXiv 官方历史仅有 v1，2026-11-27 核验。论文作者为 Vitor Guizilini、Muhammad Zubair Irshad、Dian Chen、Greg Shakhnarovich、Rares Ambrus，机构 TRI 与 TTIC。

**篇章与目录**：正文 p.1–8：§1 Introduction（p.1–2），§2 Related Work（p.2–3），§3 Multi-View Geometric Diffusion（p.3–5，3.1–3.7），§4 Experiments（p.5–8，4.1–4.6），§5 Conclusion（p.8）；附录 A 数据准备 p.9，B 更多定性结果 p.9–11，C 隐式动态 p.9、12，D 增量条件 p.12–15，E 局限 p.15–16；参考文献 p.16–20。正文 Figure 1/2/3 分别 p.1/3/6；附录 Figure 4–9 分别 p.10/11/12/13/14/15。Table 1–8 分别 p.2/6/7/7/7/8/8/8。共 9 图、8 表，无编号 Algorithm；公式编号 (1)–(4)，其余非编号定义同样纳入。

图像全部为 PDF 区域独立保真 PDF 加 300 DPI JPEG 展示副本，Figure 5/7/8 存在非精确回渲染差异但实用视觉通过；不把 JPEG 用作像素级证据。事实以对应页为准；背景推导与独立评价明确标出；不访问引用论文全文，不声称已运行作者代码。本单元不承担独立机制解释。本单元不承担独立数学推导。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1. 全局认识

### 1.1 标题

**标题译解**：基于多视角几何扩散的零样本新视角与深度合成。Zero-shot 强调训练完成的共享模型可以对未见场景或评测域直接推断，不等于无输入、无训练或完全无监督。Novel View Synthesis 指从若干已知相机的观测图像预测另一相机所能看到的颜色图像；Novel Depth Synthesis 是同一目标相机下的深度合成，目标 RGB 不必已存在。Multi-View Geometric Diffusion 是作者对采用相机射线条件、共享模型学习颜色与深度的条件扩散机制的命名；不是对三维体素、点云或高斯参数直接加噪。

**任务边界**：§3（p.3）规定输入为 $N$ 张已知内参和外参的 RGB 图像以及目标相机，输出目标 RGB 和深度。因此本文不是从未知位姿图片联合求 SfM，也不是单纯单目深度估计。与传统先拟合 NeRF/3DGS 再渲染不同，扩散模型在像素层生成目标结果；共享 latent tokens 是隐式条件特征，不是一个定义明确的连续辐射场或可直接投影的三维高斯集合。图1点云是从预测深度反投影后着色的展示/派生结果，而非模型内部必须构造的显式几何。题目并不保证每个测试域都 zero-shot，也不保证绝对公制尺度、精确一致性或任意视角的真实性，需结合各实验协议。 本单元不承担独立数学推导。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
### 1.2 摘要逻辑

**问题与困难**（p.1 摘要、§1 p.1–2）：已有稀疏输入三维重建常用神经场、体素或高斯形成中间表示，通过统一的三维结构渲染不同视角。这样提供跨视角耦合，但要求重建表示、处理尺度与稀疏可见性；作者指出泛化与视角数量扩展仍有局限。另一条路线是给定观测与目标相机直接生成图像和深度。没有统一可渲染表示时，各目标相机分别采样，未观测区域可能出现多种都合理的补全；独立生成可能彼此矛盾。【背景知识】这是条件逆问题欠定，不是所有显式方法都必然失败。深度监督可以促进几何推断，但不能从无观测中保证真值。

**方法链**：输入相机相对目标重新表达并进行 Scene Scale Normalization（SSN），消除共同尺度变化的影响；条件图像特征和对应射线编码组成 scene tokens，目标射线、任务嵌入及扩散状态组成 prediction tokens。RIN（Recurrent Interface Networks）用固定数量的 latent tokens 读写大量像素/场景 token，避免所有 token 之间的全局自注意力，从而直接在像素空间扩散而不是先编码到 VAE。每个预测 token 独立携带图像或深度任务标识，共享 latent 允许只有 RGB 标注的样本和具有稀疏/稠密深度的样本参与训练。作者使用超过 6000 万多视角样本，涵盖不同数据域、标注与尺度，并将较小模型的 latent 复制扩展后短程微调。

**结果边界**：摘要声称在多个 NVS、MVS、video depth benchmark 达到当时较优结果。Table 2 两视角 NVS，Table 3 三/六/九视角，Table 4 不做测试时尺度对齐的稀疏深度比较，Tables 6–7 目标图已知时的多视角深度估计，Table 5 设计消融，Table 8 扩容。Table 3 的 RE10K/CO3D 属 in-domain，其余按题注为 zero-shot；Table 6 ScanNet in-domain，SUN3D/RGB-D zero-shot。不能将论文全部评测标为零样本，也不能以 PSNR 代替几何准确性。零样本不是对任意未见物理介质的保证。 本单元不承担独立数学推导。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
### 1.3 论文贡献

【原文事实】§1 p.2 明列三项贡献，以下按各自功能拆解，而不是将所有借用组件均称为新发明。

**贡献一**：MVGD 多任务扩散架构可在任意数量带位姿图像条件下产生跨视角相容的颜色和深度，不依赖显式中间 3D 表示。Raymap 将目标像素位置和相机方向、条件图像位置联系；任务 token 在共享 latent 中建模颜色和几何。贡献是组合机制与任务设定，不是首次提出扩散、傅里叶编码或 RIN。证据：Figure 2、Tables 2–4、6–7，Figure 1/5 展示独立预测点云的叠加。

**贡献二**：异质大规模训练技术。61,126,321 样本（Table 1）来自合成/真实、动态/静态、内视/外视、公制/非公制数据，深度标签并非每像素可用。像素扩散让每个有效深度像素可独立构成 token，任务嵌入兼容缺深度标签，SSN 让不同标定尺度在条件与监督中一致。其协同效果用 Table 5 的 B/C/D/E 变化评估。数据多样性与规模共同变化，不能仅据 E 判定是动态信息本身造成提升。

**贡献三**：增量扩容微调。RIN 的可变 latent 序列长度与共享权重使模型可以从较小瓶颈出发，再复制 latent 并在后续阶段微调。参数数目只小幅增加，计算量却增长；Table 8 证明报告设置下逐步扩容改善多种任务，Table 5 A 表明直接训练大模型在给定步数下不一定更优。作者报告训练时间最多缩减约 70%，不是普遍复杂度保证。

三项贡献形成依赖：高效架构可承载大规模多任务与稀疏标签训练，尺度规范和深度任务为直接生成提供几何信息，扩容减少瓶颈信息损失。单个因素的独立增益仅以实际消融为准。 本单元不承担独立数学推导。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
### 1.4 结论及约束

**作者结论**（§5 p.8）：MVGD 直接以相机射线调节像素级条件扩散，为新视角颜色与深度生成统一建模，训练数据异质且大规模；通过 SSN 处理任意尺度，通过任务嵌入适配数据标签，使用高效 RIN 处理多个观测。作者报告新视角合成、多视角立体和视频深度的 benchmark 进步，并展示 latent 扩容的表现。Table 5 支持多任务和 SSN 在指定训练设置的价值，Table 8 支持扩大 latent 容量。结论是经验性证据，不是跨视角一致性的严格数学定理。

**技术局限**（Appendix E p.15–16）：每个目标相机单独作为规范坐标原点，现有 SSN 难以同时生成多个不同目标相机（可在单个目标上生成两种模态）；增量条件缓解未观测区域的采样随机性，但不能消除误差累积。模型无显式动态建模，Figure 6 只能说明特定 DDAD 序列中有一定隐式插值能力；作者提出未来加入 temporal embeddings / motion tokens，并指出准确相机信息的大规模动态数据不足。精确的相机参数仍是前提；由任意尺度的相机无法恢复米制绝对尺度。局部点云相容不代表闭合、完整或真值一致的表面；没有观测证据的区域必须区分合理生成与真实测量。 【独立评价】数个原文参数化公式表面不自洽，应保留原式并推导，不据此断言实际代码错误。 本单元不承担独立数学推导。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2. 图表精读

**图表功能**

|编号|PDF页|作用|
|---|---:|---|
|Figure 1|1|多目标点云叠加总览，色标区分条件/生成相机|
|Figure 2|3|整体架构：SSN、双编码、任务/状态、RIN 与反归一化|
|Figure 3|6|8 组目标 RGB、深度与点云定性样例|
|Figure 4|10|12 组附录补充零样本样例|
|Figure 5|11|8 组同条件多目标点云独立合成后叠加|
|Figure 6|12|DDAD 动态插值示例|
|Figure 7|13|固定条件视角数量扫描，PSNR/AbsRel 曲线|
|Figure 8|14|固定和增量条件对照|
|Figure 9|15|长序列增量合成、149 至 5578 帧|
|Table 1|2|训练数据来源与异质性统计|
|Table 2|6|2-view NVS 比较|
|Table 3|7|3/6/9-view NVS 多域比较|
|Table 4|7|目标深度对稀疏 COLMAP、无测试时尺度对齐|
|Table 5|7|单任务、SSN、动态数据及任务迁移等消融|
|Table 6|8|stereo depth, 2 conditioning views|
|Table 7|8|video depth, 10 conditioning views|
|Table 8|8|latent 数和参数、性能的扩容实验|

**证据链**：Figure 2 解释机制；Table 1 说明先验来源和数据异质性；Figures 1/3–6 说明输出形态和可见的跨视角效果；Tables 2–4/6–7 测试经验性能；Table 5 针对部分关键假设作消融，Table 8 扩容；Figures 7–9 则考察输入数量和增量条件行为。定性结果仅支持图中样例的现象，定量比较须保留协议和域，不从单张示例得出泛化证明。 本单元不承担独立机制解释。本单元不承担独立数学推导。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
### Figure 1（PDF p.1）

![Figure 1](MVGD/Figures/Figure_01.jpg)

*PDF p.1 区域渲染展示副本。*

**Caption 原文**：Figure 1. MVGD is a state-of-the-art method that generates images and scale-consistent depth maps from novel viewpoints given an arbitrary number of posed input views. In the above, red cameras are used as conditioning to directly generate RGB-D predictions from green cameras. To highlight the multi-view consistency of our method, predicted colored pointclouds from all novel viewpoints are stacked together for visualization without any post-processing. More examples and videos can be found in https://mvgd.github.io/

**Caption 中文**：MVGD 是一种先进方法，可从任意数量的已知位姿输入视图生成新视角图像与尺度一致的深度图。上图红色相机作为条件，直接生成绿色相机的 RGB-D 预测。为突出方法的多视角一致性，将所有新视角预测的彩色点云直接堆叠可视化，未作任何后处理。更多示例和视频见 https://mvgd.github.io/。

**布局与子图**

图中从左至右呈现四组场景：小物体周围分布的离散相机、环绕桌面物体的相机轨迹、室内家具空间、较大范围房间。红色锥体表示真实条件相机，绿色锥体表示目标；不是每个绿色相机都有真实 RGB 输入。第一、二组有环绕物体的稀疏条件，后两组沿室内路径延伸，故在视角拓扑上有不同的空间覆盖。点云重叠形成连贯的外观和大体表面，不表示已先构建全局 NeRF 或 3DGS。

【背景知识】若某个目标像素有颜色 $\hat I(u,v)$、相机 z 深度 $\hat D(u,v)$，相机坐标点可由 $\hat D K^{-1}[u,v,1]^T$ 得到，再用相机外参的逆变换放到共同坐标系；将 RGB 作为点颜色。若各目标深度尺度不一致，平移部分不能与深度同时兼容，重复的墙壁、物体等可能出现错层。Figure 1 的堆叠恰是此类一致性的直观诊断（实际反投影应遵循作者的深度及外参约定）。

**证据边界**

Figure 1 题注强调无点云后处理，§3.2 为尺度一致性提供机制，Appendix B Figure 5 给出更多例子。图只能证明所选示例中点云外观和尺度的相容性，不能量化绝对几何误差、表面完整性或遮挡区域真值；可见白洞和稀疏部分不应被填成完美重建。Table 4 是尺度相关定量补充。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
### Figure 2（PDF p.3）

![Figure 2](MVGD/Figures/Figure_02.jpg)

*PDF p.3 区域渲染展示副本。*

**Caption 原文**：Figure 2. Diagram of our proposed Multi-View Geometric Diffusion (MVGD) framework, at inference time. N input images In c with cameras Cn c are used for scene conditioning, and a different camera Ct is selected for novel view and depth synthesis.

**Caption 中文**：所提出的多视角几何扩散（MVGD）框架在推理时的示意图。使用 N 张输入图像及其相机作为场景条件，选择另一相机作为新视角和深度合成目标。

**布局与子图**

从左到右四个层次：

1. Input Views 包含 $(I,C)_c^1,\ldots,(I,C)_c^N$，虚线相机是目标 $(I,C)_t$ 的位置示意。推理时目标真实图像不是必需的；不要把图中符号误解成已给目标 RGB。Scene Scale Normalization 将条件位姿改到目标相机坐标系并求场景尺度 $s$；底部连到输出的线保存尺度以恢复预测深度。目标相机平移为零，旋转为单位阵。
2. Conditioning Views 经 Image Encoder 和 Ray Encoder：EfficientViT 提取 $H/4\times W/4\times448$ 图像特征并投影为 $D_I$ 维，射线表示在相同低分辨率下定位；逐位置拼接得到 $E_c^n$，多视角沿 token 轴汇合。目标相机在完整预测分辨率编码射线；图中它不通过 Image Encoder。
3. 左下 State Embeddings 的黄色小块表示当前扩散状态，图像为 3 个分量，深度为 1 个；Task Embeddings 红/绿区分图像和深度。它们同蓝色目标射线特征拼成预测 token $E_t^I,E_t^D$。Scene Tokens 传入观测证据；Prediction Tokens 指定要在哪些射线上生成哪类量及当前噪声状态。任务并非两个独立 3D 模型，也非把任务 token 加入 latent 使每个任务完全分离。
4. 中间 Diffusion 内的 RIN 读取两类 token，通过固定长度黄色 latent tokens 做内部自注意力再写回输入；Timestep 作为去噪时间条件。多步迭代更新目标状态，反参数化得到 $\hat I_t,\hat D_t$；深度乘回场景尺度（底部箭头）。右侧彩色点云说明两种输出可用于后续反投影，不是 RIN 的显式中间结构。

【原文事实】§3.1–3.5 与 Figure 2 相互对应；训练时目标 GT 用于生成噪声状态和损失，而推理初态为高斯随机噪声。条件图像与目标射线不随 DDIM 步数变为新目标真值。

**证据边界**

架构示意给出信息流而非逐层代码。没有展示完整的 Q/K/V 投影细节、task state 维度对齐方式、精确的嵌入频率实现或采样聚合函数。SSN 与深度监督不能数学保证所有目标同一表面；Table 5 D/B 提供经验支持。图中的 Scene scale 只用于尺度变换，不会自动创造米制标定。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
### Figure 3（PDF p.6）

![Figure 3](MVGD/Figures/Figure_03.jpg)

*PDF p.6 区域渲染展示副本。*

**Caption 原文**：Figure 3. MVGD novel view and depth synthesis results randomly sampled from different evaluation benchmarks and in-the-wild datasets. Top images are conditioning views (colored cameras), and bottom images are the target view (black camera), showing from left- to-right: ground-truth image, predicted image, and predicted depth map. These predictions are used to produce a colored 3D pointcloud observed from the target view. For more examples and additional visualizations, please refer to the supplementary material.

**Caption 中文**：在不同评测基准与野外数据集中随机采样的 MVGD 新视角及深度合成结果。顶部图像是条件视图（彩色相机），底部是目标视角（黑色相机），由左至右分别为真实图像、预测图像和预测深度图。利用这些预测生成从目标视角观测到的彩色三维点云。更多样例和可视化参见补充材料。

**布局与子图**

2×4 布局，各组左上小图为条件 RGB，黑色目标相机与其余彩色相机示意位姿；点云是预测深度和 RGB 的投影，底部三联图对照颜色真值、生成颜色和预测深度。第一行从左到右：车辆及草地、建筑与道路、盆栽、室内空间；第二行：室外道路/树木及深色物体、红花、建筑立面、桌面/室内近景。各组不局限于统一相机轨迹。深度伪彩表现前景和背景的相对层次；无统一色条，不能从颜色直接推断物理深度数值。

第一行车辆与建筑能看出目标图中的大形状和颜色与真值大体对应；盆栽和花朵突出细粒度物体的轮廓；室内与建筑立面展示平面和边缘。点云仍有空洞、边缘条纹，尤其背景和遮挡边界。生成 RGB 的像素拟合与深度的三维真实性是两种不同性质的证据。例如色彩相似仍可能伴随错误深度，反之单视角深度接近也可能有纹理误差。

**证据边界**

p.6 §4.3 与 Tables 2–4 相关。题注只标随机选取不同基准及野外数据，不能从外观臆造每组具体数据集名或输入视角数。此图各组仅显示目标 RGB 真值而非目标深度真值，深度与点云的准确性需 Table 4、Tables 6–7 补充。未证明各目标联合的一致性；Figure 5 更直接展示同条件多目标的叠加。
本单元不承担独立数学推导。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
### Figure 4（PDF p.10）

![Figure 4](MVGD/Figures/Figure_04.jpg)

*PDF p.10 区域渲染展示副本。*

**Caption 原文**：Figure 4. Zero-Shot MVGD novel view and depth synthesis results randomly sampled from different evaluation benchmarks and in-the- wild datasets. Top left images are conditioning views (colored cameras), and bottom images are the target view (black camera), showing from left-to-right: ground-truth image, predicted image, and predicted depth map. These predictions are used to produce a colored 3D pointcloud observed from the target viewpoint.

**Caption 中文**：在不同评测基准及野外数据中随机选取的零样本 MVGD 新视角和深度合成结果。左上图像为条件视图（彩色相机），底部目标视角（黑色相机）从左至右依次为真实图像、预测图像和预测深度图。利用预测形成从该目标视角观测到的彩色三维点云。

**布局与子图**

图为 4 行×3 列，和 Figure 3 的视觉语法相同。第1行：桌面点心、白色室内墙门/地面、窗格或围栏；点心轮廓、地面交界、规则竖线分别考验小物体、低纹理平面和结构细节。第2行：有丰富纹理的室内摆放物、草地上的彩色小物、暗背景的金属色物体，显示颜色先验与几何范围的多样性。第3行：蓝色物体、复杂室内台面、建筑立面，包含平滑及复杂遮挡。第4行：蓝色椅子、室内植物/花盆、室外建筑与天空。每组有已知条件小图、相机相对位置、目标三联图和单目标点云。

【视觉观察】若干点云在不连续背景、边缘区域呈明显空白或点阵；白墙区域与建筑平面虽然形态相符，不能凭该图测量深度误差。颜色图与深度图也没有提供逐像素跨模态一致性的量化。

**证据边界**

Appendix B p.9 说明这些是 Figure 3 的补充。零样本是作者题注的实验描述，并非所有图像场景训练分布均可独立证明；没有各样本完整身份与真实深度，不能从 12 张例图断言一般化性能或绝对尺度。
本单元不承担独立数学推导。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
### Figure 5（PDF p.11）

![Figure 5](MVGD/Figures/Figure_05.jpg)

*PDF p.11 区域渲染展示副本。*

**Caption 原文**：Figure 5. Accumulated MVGD pointclouds, obtained by generating novel images and depth maps from various viewpoints (black cameras), using the same conditioning views (colored cameras), and stacking them together without any post-processing. Our zero-shot architecture is capable of directly generating multi-view consistent predictions that match the scale from conditioning cameras.

**Caption 中文**：将同一组条件视图（彩色相机）用于多个不同新视角（黑色相机）的图像与深度合成，得到累积的 MVGD 点云；不进行任何后处理，直接将它们堆叠。零样本架构能够直接生成多视角相容且与条件相机尺度一致的预测。

**布局与子图**

四行两列，每组左侧/上侧彩色边框图为同一组真实条件，黑色相机为各新目标，三维点云是所有目标结果合并。第一行左为室内地面及黑色箱状物，右为小型卡通玩偶；第二行左为红色消防栓，右为草地中蓝黄物体；第三行左为杂乱室内，右为白墙、门和室内走廊；第四行左为室内房间，右为桌面红色物体。点云能形成相对统一的场景结构，仍有缺口和重叠不完美处。相比 Figure 3/4 只从某一目标生成，本图更直接考察独立目标之间的几何兼容。

§3.2、Appendix B 的关键点：各目标被设作原点并独立估计相对尺度 $s_t$；将归一化深度还原为条件位姿原有单位后，再把各点转到共同坐标系叠加。如果只学习归一化深度而不恢复尺度，则不同 $s_t$ 的输出不能直接使用同一外参坐标。这就是 SSN 的算法作用。共享模型与相同观测不保证随机采样必然匹配，但可以从训练学习相容的几何先验。

**证据边界**

附录 B 说明各目标独立生成，且没有点云后处理；此图不代表模型在内部联合优化所有点。SSN 只保证尺度变换规则一致，不证明深度数值正确，不能将“same scale as cameras”理解为米制误差为零。Table 4 无测试时对齐的深度比较提供定量证据；Table 5 D 则为归一化消融。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
### Figure 6（PDF p.12）

![Figure 6](MVGD/Figures/Figure_06.jpg)

*PDF p.12 区域渲染展示副本。*

**Caption 原文**：Figure 6. Accumulated MVGD pointclouds on a dynamic dataset [30]. (left) Red cameras are used as conditioning views, and novel images and depth maps are generated from green cameras. (right) Colored pointclouds are calculated from these predictions and stacked together without any post-processing. Even though MVGD does not explicitly model dynamic objects, it implicitly learns how the scene should change when interpolating between views with objects in different locations (e.g., moving cars), while keeping the remainder static.

**Caption 中文**：动态数据集 [30] 上的累积 MVGD 点云。（左）红色相机用作条件，从绿色相机生成新图像和深度图。（右）用这些预测计算彩色点云并直接堆叠，无后处理。虽然 MVGD 未显式建模动态物体，却隐式学习到在物体位于不同位置（例如行驶的汽车）的视图间插值时场景如何变化，同时保持其余部分静态。

**布局与子图**

上下两个道路序列；每组左侧四个不同时刻的目标彩色点云及 GT/预测 RGB/深度，右侧将这些目标预测的点云放到统一坐标中。右上黑框标出汽车位置的变化；右下黑框中可见多辆车沿道路方向叠加。红色条件相机与绿色目标相机位于道路上；静态路面及道路标线总体连贯，移动物体在不同位置，叠加时出现多位置的“轨迹状”点云，不应将其当成静态场景中真实存在的多辆物体。

Appendix C p.9、12 说明 DDAD 的100帧序列每10帧取一个条件，其余目标生成。输入条件相机路径与时间相关；在未显式输入时间的情况下，模型可能根据视点和观测内容的相关性学习某些隐式时间插值。它并没有提供显式物体速度、轨迹、时间控制变量或运动分解。

**证据边界**

该定性案例支持“部分动态序列中出现合理插值”而不是通用时空重建。附录 C 将动态数据消融误指 Table 8，正文实际动态数据消融在 Table 5 E；Table 8 为 latent 扩容。数据量减少也是混杂因素；Appendix E 承认仍无显式动态模型。
本单元不承担独立数学推导。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
### Figure 7（PDF p.13）

![Figure 7](MVGD/Figures/Figure_07.jpg)

*PDF p.13 区域渲染展示副本。*

**Caption 原文**：Figure 7. MVGD per-frame novel view and depth synthesis results using different numbers of fixed conditioning views, on ScanNet (scene 0086 02, with 1267 images). The same model from all experiments reported in the main paper was used. Legend numbers indicate the stride (i.e., how many target images are positioned between each conditioning view). As expected, results consistently improve as more input information is available, eventually plateauing at around 100 conditioning views.

**Caption 中文**：ScanNet（场景0086_02，共1267张图）上，使用不同固定条件视图数量的逐帧新视角和深度合成结果。使用正文所有实验中的同一模型。图例数字表示步长，即每两个条件视图之间目标图像的采样间隔。随着可用输入信息增加，结果持续改善，约100个条件视图后趋于饱和。

**布局与子图**

上图横轴 Frame Number（约0–1267），纵轴 PSNR（dB，越高越好）；下图同一横轴，纵轴 Abs.Rel.（相对深度误差，越低越好）。黄/绿/蓝/紫/红线分别表示 stride 200、100、50、25、10。注意 legend 数字不是条件数量，间隔越小意味着真实输入越密集；粗略 $N_c\approx1267/\mathrm{stride}$，所以 stride 200 约7视角，stride 10 约127视角，确切数取决于首尾选帧规则。

上图黄线在远离条件帧处常降至约14–20 dB，较密条件的紫/红线多处接近约27–30 dB；下图稀疏线出现高尖峰（图纵轴至0.20），密集线普遍较低，局部仍有异常。增加条件让模型得到更直接的几何与纹理信息，但也存在视角冗余，曲线逐渐接近。作者正文称 plateau around 100 (stride 20)，与1267/20约63不完全一致，且图例无20；应保留为作者近似描述，不能把100或20当作精确阈值。

**证据边界**

同一模型和同一 ScanNet 序列比较支持真实条件密度与质量关系，但未覆盖不同相机轨迹或全部数据域，曲线不能推导任意场景的最小观测数；没有给出每条线的整体均值或置信区间。作者还称分成5个250帧片段更差，未提供详细表格；仅视为定性报告。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
### Figure 8（PDF p.14）

![Figure 8](MVGD/Figures/Figure_08.jpg)

*PDF p.14 区域渲染展示副本。*

**Caption 原文**：Figure 8. MVGD per-frame novel view and depth synthesis results with and without our proposed incremental conditioning strategy, on ScanNet (scene 0086 02, with 1267 images). The same model from all experiments reported in the main paper was used. The blue line indicates a fixed number (25) of conditioning views, evenly spaced with a stride of 50. The red line indicates our proposed incremental conditioning strategy, in which each new generation uses previously generated images as additional conditioning. This strategy leads to consistently better and more stable results in novel view and depth synthesis, especially in regions further away from conditioning views.

**Caption 中文**：在 ScanNet（场景0086_02，共1267张图）上，有无增量条件策略的逐帧新视角与深度合成结果。采用正文实验的同一模型。蓝线为固定25个条件视图、均匀间隔50；红线为所提出的增量条件策略，每次新生成都将之前生成的图像作为额外条件。该策略使新视角及深度合成结果持续更好、更稳定，尤其在远离条件视图的区域。

**布局与子图**

上图纵轴 PSNR（约20–32 dB），红线通常高于蓝线，蓝线在多个局部发生急剧下降；例如帧约150和600附近有显著低谷，红线更平滑。下图纵轴 AbsRel（0–0.100），蓝线有更高频、更大幅的尖峰，例如约250、480和1050附近，红线整体较低，但仍出现误差峰。两图横轴 Frame Number。

条件初始为同样25张真实观测，固定方案每个目标用固定集合生成；增量方案把先生成的 RGB 与其已知相机加入后续 scene tokens。目标由离初始观测近到远生成，故已有生成样本提供跨目标关联与较稳定的补全先验。后续生成的深度不是额外真实监督，也没有引入新外部测量。

**证据边界**

Appendix D p.13–14 支持该序列中增量条件提高 PSNR、降低 AbsRel，但不等于所有帧必然改善，也不保证所有场景。其性能来源可能是约束采样空间或平滑生成结果，伪观测会自我强化错误；需更多场景和真实几何检验。作者报告 25 条件约0.5s、1250条件约20s，增量不是无成本。
本单元不承担独立数学推导。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
### Figure 9（PDF p.15）

![Figure 9](MVGD/Figures/Figure_09.jpg)

*PDF p.15 区域渲染展示副本。*

**Caption 原文**：Figure 9. MVGD novel view and depth synthesis results using our proposed incremental conditioning strategy. Red cameras indicate initial conditioning views, used to generate predictions for green cameras (ordered from closest to furthest away from the initial conditioning views). After each generation, the predicted image is added to the set of conditioning views for future generations. Even though MVGD was trained using only 2 −5 conditioning views, it can directly scale to thousands on a single GPU.

**Caption 中文**：使用所提出的增量条件策略的 MVGD 新视角与深度合成结果。红色相机为初始条件，绿色相机为预测目标（从距离初始条件最近到最远排序）。每次生成后将预测图像加入条件集合以供后续生成。尽管 MVGD 训练时只使用2–5个条件视图，也可以在单个 GPU 上直接扩展到数千视图。

**布局与子图**

2列×3行六个例子，绿色相机沿轨迹连接初始红色相机。子图标签：(a)149帧，5个初始条件（室外停车/道路）；(b)202帧，7个初始条件（红色背包/物体与环绕轨迹）；(c)252帧，5个初始条件（草地中的蓝色小物）；(d)337帧，6个初始条件（室内门口直线路径）；(e)1268帧，25个初始条件（房间）；(f)5578帧，60个初始条件（更长室内轨迹）。点云呈大范围外观与布局，局部存在缺口和模糊结构。初始真实输入占比可以非常低，例如(f) $60/5578\approx1.08\%$，但绿色相机位置本身已知，不是模型恢复的轨迹。

算法每步根据初始真实相机集合的距离选择尚未处理的最近目标，生成后把 RGB 和相机纳入条件；图中的数千绿色目标属于多次连续推理，不是一次 RIN 同时生成数千个目标。

**证据边界**

Appendix D p.14–15：全部此前生成图像作为条件，作者称单张40GB A100可运行，具体耗时随条件数增加。f 的5578帧表明在报告样例可扩展性，不能等价为真实几何、无遮挡区域真实性或同一时刻多目标采样。Appendix E 承认缺少同时多目标的SSN方案。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN TABLE-01 -->
### Table 1（PDF p.2）

**Caption 原文**：Table 1. Datasets used to train MVGD. Syn indicates if the dataset is synthetic, Dyn. if it contains dynamic objects, Met. if depth labels are metric, I/O if cameras are looking inwards and/or outwards, and M/V if it contains multiple cameras and/or videos.

**Caption 中文**：MVGD 训练数据集。Syn 表示合成，Dyn 表示动态物体，Met 表示度量深度标签，I/O 表示相机向内/向外，M/V 表示多相机/视频。

**表格数据**

| Dataset | Syn | Dyn | Met | I/O | V/M | Sequences | Samples |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ArgoVerse2 |  | ✓ | ✓ | O | VM | 1043 | 3,909,297 |
| BlendedMVG |  |  |  | O | M | 502 | 115,142 |
| CO3Dv2 |  |  |  | I | V | 25,243 | 5,088,873 |
| DROID |  |  | ✓ | I | M | 76,792 | 7,340,712 |
| LSD |  | ✓ | ✓ | O | VM | 17,647 | 1,057,920 |
| LyftL5 |  | ✓ | ✓ | O | VM | 394 | 347,508 |
| MVImgNet |  |  |  | I | V | 194,368 | 5,768,120 |
| NuScenes |  | ✓ | ✓ | O | VM | 850 | 204,894 |
| Taskonomy |  |  | ✓ | O | M | 533 | 4,584,462 |
| HM3D |  |  | ✓ | O | M | 900 | 9,531,876 |
| Hypersim |  |  | ✓ | O | M | 457 | 74,619 |
| P. Domain | ✓ | ✓ | ✓ | IO | VM | 5,449 | 555,000 |
| RealEstate10k |  |  | --- | O | V | 74,532 | 10,115,793 |
| RTMV | ✓ |  |  | I | M | 1,909 | 286,350 |
| ScanNet |  |  | ✓ | O | V | 1,513 | 2,477,378 |
| TartanAir | ✓ | ✓ | ✓ | O | VM | 369 | 613,274 |
| VKITTI2 | ✓ | ✓ | ✓ | O | VM | 45 | 38,268 |
| Waymo | ✓ |  | ✓ | O | VM | 1,000 | 990,340 |
| WildRGBD |  |  |  | I | V | 23,049 | 8,026,495 |
| Total |  |  |  |  |  | 426,595 | 61,126,321 |

**实验分析**

19 个数据集，原表合计 426,595 序列、61,126,321 样本。空格表示原表未标记，不等于已独立证明不存在相应属性；RE10K 的 Met 为 ---。DROID、HM3D、RE10K、WildRGBD 样本量大；既有室内外也有对象、行车和机器人数据。V/M 表示视频/多相机。注意原表将 Waymo 标为 Syn、未将 HM3D/Hypersim 标为 Syn，Taskonomy 标为 O；保留原表分类而不自行纠正。样本数并非独立场景数，标签不齐是多任务设计的直接背景。

本单元不承担独立数学推导。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
### Table 2（PDF p.6）

**Caption 原文**：Table 2. Novel view synthesis results with 2 input views.

**Caption 中文**：两个输入视角的新视角合成结果。

**表格数据**

| Method | RE10K PSNR | SSIM | LPIPS | ACID PSNR | SSIM | LPIPS |
| --- | --- | --- | --- | --- | --- | --- |
| PixelNeRF | 20.43 | 0.589 | 0.550 | 20.97 | 0.547 | 0.533 |
| GPNR | 24.11 | 0.793 | 0.255 | 25.28 | 0.764 | 0.332 |
| AttnRend | 24.78 | 0.820 | 0.213 | 26.88 | 0.799 | 0.218 |
| MuRF | 26.10 | 0.858 | 0.143 | 28.09 | 0.841 | 0.155 |
| PixelSplat | 25.89 | 0.858 | 0.142 | 28.14 | 0.839 | 0.150 |
| MVSplat | 26.39 | 0.869 | 0.128 | 28.25 | 0.843 | 0.144 |
| MVGD | 28.41 | 0.891 | 0.107 | 29.98 | 0.875 | 0.131 |

**实验分析**

PSNR/SSIM 越大、LPIPS 越小越好。对 MVSplat，MVGD 的 RE10K PSNR 从 26.39 到 28.41（+2.02 dB），ACID 从 28.25 到 29.98（+1.73 dB），三项指标均改善。训练规模和模型路线同时改变，不是单独证明扩散优于 3DGS。

本单元不承担独立数学推导。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
### Table 3（PDF p.7）

**Caption 原文**：Table 3. Novel view synthesis results with 3-9 conditioning views. ∗indicate methods fine-tuned and reported by [106]. † indicate the use of image diffusion models pre-trained with internal datasets. Colored numbers represent first , second , and third best ranked metrics. RealEstate10k and CO3D results are in-domain, and results on other benchmarks are zero-shot.

**Caption 中文**：3–9 个条件视角的新视角合成。* 表示由 [106] 微调并报告；† 使用内部数据预训练的图像扩散模型。颜色标记前三名。RE10K 与 CO3D 为域内，其余零样本。

**表格数据**

| Dataset | Method | PSNR 3 | 6 | 9 | SSIM 3 | 6 | 9 | LPIPS 3 | 6 | 9 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| RE10K | FreeNeRF | 20.54 | 25.63 | 27.32 | 0.731 | 0.817 | 0.843 | 0.394 | 0.344 | 0.332 |
| RE10K | SimpleNeRF | 23.89 | 28.75 | 29.55 | 0.839 | 0.896 | 0.900 | 0.292 | 0.239 | 0.236 |
| RE10K | ZeroNVS* | 19.11 | 22.54 | 23.73 | 0.675 | 0.744 | 0.766 | 0.422 | 0.374 | 0.358 |
| RE10K | ReconFusion† | 25.84 | 29.99 | 31.82 | 0.910 | 0.951 | 0.961 | 0.144 | 0.103 | 0.092 |
| RE10K | CAT3D† | 26.78 | 31.07 | 32.20 | 0.917 | 0.954 | 0.963 | 0.132 | 0.092 | 0.082 |
| RE10K | MVGD | 28.70 | 31.71 | 32.89 | 0.925 | 0.960 | 0.969 | 0.128 | 0.090 | 0.085 |
| LLFF | FreeNeRF | 19.63 | 23.72 | 25.12 | 0.613 | 0.773 | 0.820 | 0.347 | 0.232 | 0.193 |
| LLFF | SimpleNeRF | 19.24 | 23.05 | 23.98 | 0.623 | 0.737 | 0.762 | 0.375 | 0.296 | 0.286 |
| LLFF | ZeroNVS* | 15.91 | 18.39 | 18.79 | 0.359 | 0.449 | 0.470 | 0.512 | 0.438 | 0.416 |
| LLFF | ReconFusion† | 21.34 | 24.25 | 25.21 | 0.724 | 0.815 | 0.848 | 0.203 | 0.152 | 0.134 |
| LLFF | CAT3D† | 21.58 | 24.71 | 25.63 | 0.731 | 0.833 | 0.860 | 0.181 | 0.121 | 0.107 |
| LLFF | MVGD | 22.39 | 24.76 | 25.93 | 0.776 | 0.882 | 0.853 | 0.157 | 0.131 | 0.103 |
| DTU | FreeNeRF | 20.46 | 23.48 | 25.56 | 0.826 | 0.870 | 0.902 | 0.173 | 0.131 | 0.102 |
| DTU | SimpleNeRF | 16.25 | 20.60 | 22.75 | 0.751 | 0.828 | 0.856 | 0.249 | 0.190 | 0.176 |
| DTU | ZeroNVS* | 16.71 | 17.70 | 17.92 | 0.716 | 0.737 | 0.745 | 0.223 | 0.205 | 0.200 |
| DTU | ReconFusion† | 20.74 | 23.62 | 24.62 | 0.875 | 0.904 | 0.921 | 0.124 | 0.105 | 0.094 |
| DTU | CAT3D† | 22.02 | 24.28 | 25.92 | 0.844 | 0.899 | 0.928 | 0.121 | 0.095 | 0.073 |
| DTU | MVGD | 23.79 | 25.31 | 26.88 | 0.893 | 0.912 | 0.939 | 0.115 | 0.091 | 0.088 |
| CO3D | FreeNeRF | 13.28 | 15.20 | 17.35 | 0.461 | 0.523 | 0.575 | 0.634 | 0.596 | 0.561 |
| CO3D | SimpleNeRF | 15.40 | 18.12 | 20.52 | 0.553 | 0.622 | 0.672 | 0.612 | 0.541 | 0.493 |
| CO3D | ZeroNVS* | 17.13 | 19.72 | 20.50 | 0.581 | 0.627 | 0.640 | 0.566 | 0.515 | 0.500 |
| CO3D | ReconFusion† | 19.59 | 21.84 | 22.95 | 0.662 | 0.714 | 0.736 | 0.398 | 0.342 | 0.318 |
| CO3D | CAT3D† | 20.57 | 22.79 | 23.58 | 0.666 | 0.726 | 0.752 | 0.351 | 0.292 | 0.273 |
| CO3D | MVGD | 20.68 | 23.49 | 24.77 | 0.678 | 0.743 | 0.789 | 0.331 | 0.322 | 0.248 |
| MIP-NeRF360 | FreeNeRF | 12.87 | 13.35 | 14.59 | 0.260 | 0.283 | 0.319 | 0.715 | 0.717 | 0.695 |
| MIP-NeRF360 | SimpleNeRF | 13.27 | 13.67 | 15.15 | 0.283 | 0.312 | 0.354 | 0.741 | 0.721 | 0.676 |
| MIP-NeRF360 | ZeroNVS* | 14.44 | 15.51 | 15.99 | 0.316 | 0.337 | 0.350 | 0.680 | 0.663 | 0.655 |
| MIP-NeRF360 | ReconFusion† | 15.50 | 16.93 | 18.19 | 0.358 | 0.401 | 0.432 | 0.585 | 0.544 | 0.511 |
| MIP-NeRF360 | CAT3D† | 16.62 | 17.72 | 18.67 | 0.377 | 0.425 | 0.460 | 0.515 | 0.451 | 0.439 |
| MIP-NeRF360 | MVGD | 18.74 | 20.28 | 21.18 | 0.425 | 0.463 | 0.512 | 0.499 | 0.512 | 0.488 |

**实验分析**

五个数据集的 3/6/9 视角比较。MVGD 在所有 PSNR 列均最佳，但不是每个指标均最佳，如 LLFF 9-view SSIM 0.853 小于 CAT3D 0.860；DTU 9-view LPIPS 0.088 大于 0.073；MIP-NeRF360 6/9-view LPIPS 不及 CAT3D，且 MVGD 3→6 LPIPS 0.499→0.512。相比 CAT3D，MIP-NeRF360 3-view PSNR +2.12 dB。不同预训练数据与生成流程是混杂变量，未验证所有训练重叠。

本单元不承担独立数学推导。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
### Table 4（PDF p.7）

**Caption 原文**：Table 4. Novel depth synthesis results with 3-9 conditioning views, relative to COLMAP [82] ground-truth. No test-time align- ment was used, which means that predicted depth maps share the same scale as the provided camera extrinsics.

**Caption 中文**：3–9 个条件视角的新深度合成，对照 COLMAP 真值；测试时不进行对齐，预测深度与提供的相机外参尺度相同。

**表格数据**

| Dataset | Split | Abs.Rel. ↓ | RMSE ↓ | δ1.25 ↑ |
| --- | --- | --- | --- | --- |
| LLFF | 3-view | 0.105 | 8.378 | 0.865 |
| LLFF | 6-view | 0.098 | 8.260 | 0.879 |
| LLFF | 9-view | 0.093 | 8.006 | 0.891 |
| MIPNeRF360 | 3-view | 0.143 | 2.445 | 0.842 |
| MIPNeRF360 | 6-view | 0.124 | 2.292 | 0.866 |
| MIPNeRF360 | 9-view | 0.109 | 2.106 | 0.883 |
| CO3D | 3-view | 0.101 | 4.654 | 0.874 |
| CO3D | 6-view | 0.086 | 4.436 | 0.898 |
| CO3D | 9-view | 0.079 | 4.195 | 0.914 |

**实验分析**

Abs.Rel. 为相对绝对误差均值，RMSE 为平方误差均方根，δ1.25 为预测/真值比率落于 1.25 阈值内的比例。每个数据集三指标随视角增加均改善；例如 MIPNeRF360 Abs.Rel. 0.143→0.109。稀疏 COLMAP 参考只覆盖匹配可靠的位置，其尺度可能非米制；未进行对齐支持尺度一致，但不能证明遮挡区深度正确。

本单元不承担独立数学推导。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
### Table 5（PDF p.7）

**Caption 原文**：Table 5. MVGD ablation analysis. Top results obtained with 200k steps and 256 latent tokens. Bottom results obtained by fine- tuning top models for 100k steps.

**Caption 中文**：MVGD 消融；上半部分训练 200k 步、256 latent tokens（A 为 2048 从头训练）；下半部分将上方模型微调 100k 步。

**表格数据**

| ID | Variation | RE10K(2) PSNR | DTU(3) PSNR | CO3D(3) PSNR | Abs.Rel. | LLFF(3) PSNR | Abs.Rel. | ScanNet(2) Abs.Rel. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A | 2048 (scratch) | 22.09 | 20.22 | 16.25 | 0.120 | 19.11 | 0.148 | 0.088 |
| B | - RGB | ----- | ----- | ----- | 0.132 | ----- | 0.138 | 0.089 |
| C | - Depth | 21.46 | 19.31 | 16.39 | ----- | 19.01 | ----- | ----- |
| D | - SSN | 20.15 | 17.98 | 14.67 | 0.175 | 16.87 | 0.224 | 0.329 |
| E | - Dynamic | 23.24 | 19.51 | 17.04 | 0.141 | 19.28 | 0.143 | 0.087 |
|  | MVGD (short) | 24.44 | 20.16 | 17.98 | 0.127 | 20.19 | 0.131 | 0.081 |
| F | + RGB (B) | 25.15 | 21.11 | 18.04 | 0.120 | 20.21 | 0.120 | 0.075 |
| G | + Depth (C) | 25.68 | 21.31 | 18.39 | 0.124 | 20.52 | 0.127 | 0.079 |
|  | MVGD (full) | 25.89 | 21.32 | 18.48 | 0.121 | 20.58 | 0.119 | 0.075 |

**实验分析**

A 为 2048 scratch，与表题注 256 例外；§4.5 描述 300k，§4.6 和题注 200k，无法仅从原文确定 A 的确切步数。B 去 RGB、C 去 depth；正文 §4.6 对 B/C 的文字次序与表格冲突，以表格标签及数值列为准。D 去 SSN，ScanNet Abs.Rel. 从 0.081 到 0.329，RE10K PSNR 24.44→20.15；E 去动态数据，数据量也减少约 12%，不能隔离动态内容效应。F/G 从单任务模型加入另一任务，达到接近 full 的性能。

本单元不承担独立数学推导。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
### Table 6（PDF p.8）

**Caption 原文**：Table 6. Stereo depth experiments, with 2 conditioning views. Results on ScanNet are in-domain, and results on SUN3D and RGB-D are zero-shot.

**Caption 中文**：两条件视角立体深度实验。ScanNet 为域内，SUN3D 和 RGB-D 为零样本。

**表格数据**

| Method | ScanNet Abs.Rel. | RMSE | δ1.25 | SUN3D Abs.Rel. | RMSE | δ1.25 | RGB-D Abs.Rel. | RMSE | δ1.25 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DPSNet | 0.126 | 0.315 | - | 0.147 | 0.449 | 0.781 | 0.151 | 0.695 | 0.804 |
| NAS | 0.107 | 0.281 | - | 0.127 | 0.378 | 0.829 | 0.131 | 0.619 | 0.857 |
| IIB | 0.116 | 0.281 | 0.908 | 0.099 | 0.293 | 0.902 | 0.095 | 0.550 | 0.907 |
| DeFiNe | 0.093 | 0.246 | 0.911 | 0.095 | 0.287 | 0.914 | 0.095 | 0.539 | 0.909 |
| EPIO | 0.086 | 0.229 | 0.923 | 0.090 | 0.260 | 0.912 | 0.080 | 0.433 | 0.912 |
| GRIN | 0.088 | 0.224 | 0.925 | 0.097 | 0.274 | 0.908 | 0.092 | 0.512 | 0.919 |
| MVGD | 0.065 | 0.202 | 0.946 | 0.088 | 0.247 | 0.926 | 0.074 | 0.405 | 0.938 |

**实验分析**

目标 RGB 本身进入条件，区别于 novel depth synthesis。MVGD ScanNet Abs.Rel. 0.065、SUN3D 0.088、RGB-D 0.074，均为表中最低，所有 RMSE/δ 数值亦最优。原表部分排名着色与数值顺序不符，按数值解释，不复制错误排名。

本单元不承担独立数学推导。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN TABLE-07 -->
### Table 7（PDF p.8）

**Caption 原文**：Table 7. Video depth experiments on ScanNet (in-domain), with 10 conditioning views.

**Caption 中文**：ScanNet（域内）视频深度实验，10 个条件视角。

**表格数据**

| Method | Abs.Rel. ↓ | Sq.Rel. ↓ | RMSE ↓ |
| --- | --- | --- | --- |
| DeMoN | 0.231 | 0.520 | 0.761 |
| MiDas-v2 | 0.208 | 0.318 | 0.742 |
| BA-Net | 0.091 | 0.058 | 0.223 |
| CVD | 0.073 | 0.037 | 0.217 |
| DeFiNe | 0.059 | 0.022 | 0.184 |
| DeepV2D | 0.057 | 0.010 | 0.168 |
| NeuralRecon | 0.047 | 0.024 | 0.164 |
| MVGD | 0.041 | 0.007 | 0.143 |

**实验分析**

MVGD 的 Abs.Rel. 0.041 对 NeuralRecon 0.047，RMSE 0.143 对 0.164，Sq.Rel. 0.007 对 DeepV2D 0.010；目标图像在输入中，支持几何先验可迁移到视频深度，不是无输入的时序生成。

本单元不承担独立数学推导。

<!-- PTH:END TABLE-07 -->

<!-- PTH:BEGIN TABLE-08 -->
### Table 8（PDF p.8）

**Caption 原文**：Table 8. Incremental fine-tuning experiments. The first model (256) was trained from scratch for 300k steps, and each subsequent model was obtained by duplicating the latents of the previous one and fine-tuning for 50k steps. Even though the total number of learnable parameters barely changes (+0.5%), results improve by a significant margin (up to +20%) across all benchmarks and tasks.

**Caption 中文**：增量微调实验：256 latent 的初始模型从头训练300k步，后续模型通过复制 latent 并微调50k步获得。可训练参数仅增加0.5%，各任务与基准改善可达20%。

**表格数据**

| Latents | Params | RE10K PSNR | SSIM | CO3D PSNR | SSIM | Abs.Rel. | RMSE | MIPNeRF360 PSNR | SSIM | Abs.Rel. | RMSE | LLFF PSNR | SSIM | Abs.Rel. | RMSE |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 256 | 417.9M | 25.89 | 0.841 | 18.48 | 0.567 | 0.121 | 5.347 | 17.71 | 0.415 | 0.156 | 2.789 | 20.58 | 0.706 | 0.119 | 8.807 |
| 512 | 418.2M | 26.33 | 0.859 | 19.56 | 0.590 | 0.109 | 5.048 | 18.15 | 0.429 | 0.149 | 2.581 | 21.14 | 0.738 | 0.111 | 8.686 |
| 1024 | 418.7M | 27.73 | 0.870 | 20.08 | 0.622 | 0.104 | 4.766 | 18.49 | 0.444 | 0.147 | 2.499 | 21.85 | 0.754 | 0.109 | 8.547 |
| 2048 | 419.7M | 28.41 | 0.891 | 20.68 | 0.678 | 0.101 | 4.654 | 18.74 | 0.465 | 0.143 | 2.445 | 22.39 | 0.776 | 0.105 | 8.378 |

**实验分析**

256→2048，417.9M→419.7M 参数，增幅约0.43%；所有列单调改善。RE10K PSNR 25.89→28.41，CO3D Abs.Rel. 0.121→0.101。总训练步数亦递增（300k→450k），不能将所有改善单独归因于 latent 个数。注意 Table 8 MIPNeRF360 3-view SSIM 0.465，与 Table 3 的 0.425 不同，原值保留，论文未解释。

本单元不承担独立数学推导。

<!-- PTH:END TABLE-08 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
### 图表共同构成的证据链

**机制与证据**：Figure 2 的 SSN 将不同数据集的外参尺度统一到相对尺度；Table 5 D 关闭这一机制，RGB 和深度均退化，尤其 ScanNet 深度误差从 0.081 到 0.329。它说明尺度预处理对于该训练系统有价值，但不是几何精确性的数学证明。Figure 2 的任务嵌入在预测端控制输出，Table 5 B/C 去掉某任务、F/G 再引入任务，分别考察共享表示的收益和可迁移性；模型共享并不保证任意两次采样一致。Table 8 扩容不改变 Figure 2 的接口，只增加内部 latent 序列长度。

**质量与尺度**：Figure 1/3/4 以图像和点云展示生成能力，Table 2/3 衡量 RGB 重建，Table 4 对 COLMAP 稀疏深度给出定量约束。Figure 5 将独立生成点云叠加，是跨相机一致性的定性证据，不能替代稠密几何 GT。Figure 6 动态示例仅定性，Table 5 E 混合了数据量与动态属性两个因素。Table 6/7 验证目标 RGB 可用时的多视角深度推断，不是新视角深度。Figure 7–9 检验输入数量、历史生成作为条件、长序列扩展，分别对应真实观测增加与生成伪观测增加两种不同情形。

本单元不承担独立数学推导。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
### 研究问题与形式化任务（PDF §1, §3）

**形式化任务**：输入 $\mathcal I_C=\{(I_n,\mathcal C_n)\}_{n=1}^{N}$，其中 $I_n\in\mathbb R^{H\times W\times3}$，$\mathcal C_n=(K_n,T_n)$ 是已知相机内外参；另给目标相机 $\mathcal C_t$。输出 $\hat I_t\in\mathbb R^{H\times W\times3}$ 及/或 $\hat D_t\in\mathbb R^{H\times W}$。论文表示为学习条件分布 $p_\theta(\hat I_t,\hat D_t\mid\mathcal C_t,\mathcal I_C)$，而不是每个场景优化一组 NeRF 或高斯参数。这里的联合分布是学习目标，实际模型允许单独任务或同目标的联合预测；没有严格证明所有采样构成一个全局相容的三维随机场。

**核心困难**：输入视角的已观测部分受多视角投影几何约束，而未见区域的颜色、遮挡后表面和深度可能有多解。显式场景表示能约束多个输出来自同一表示，但也有稀疏场景重建偏差；直接扩散可以利用跨场景学习先验，代价是采样歧义与一致性问题。目标相机的位姿是输入，不是位姿估计任务；深度应与给定相机的平移单位匹配，不是从任意尺度的 SfM 结果恢复米制绝对尺度。目标 RGB 在输入中时为多视角深度估计，在输入中缺失时才是 novel depth synthesis。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
### 为什么需要该方法（PDF §1–3）

**根因链**：稀疏图像使多视角匹配与场景重建约束不足；直接生成式 NVS 可借助训练分布补全未知内容，但分别对多个相机采样时，随机生成的遮挡后结构可能彼此不一致。显式 NeRF/3DGS 通过共享几何限制输出，论文希望以隐式共享模型直接生成 RGB 和深度，以免必须构建中间表示。不同数据源的相机标定使用米制或 SfM 任意尺度，直接学习绝对数值会把表示差异误当成几何差异。又因为不是每个训练图像都具有稠密深度，强制 RGB-D 成对监督会减少可用数据。最后，全像素自注意力的二次复杂度会阻碍多视角输入和像素空间去噪。

**证据与目标**：作者的响应是目标相机坐标归一化和尺度恢复、每 token 任务控制、RIN 固定瓶颈、异质大规模训练及增量历史。Table 5 检验多任务和 SSN；Table 8 检验 latent 容量；Figure 7–9 检验更多条件。它们证明设计在本文评测协议下有经验收益，而不是证明显式表示一定不如直接生成。数据量、训练步数和预训练先验存在混杂，不能从表2/3分离各贡献的因果效应。

本单元不承担独立数学推导。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
### 相关工作与方法位置（PDF §2）

**显式与隐式路线**：原文相关工作实际按三个标题组织，可将第一类内部的两种路线区分为四条技术线。① 通用辐射场：PixelNeRF 用输入图像局部特征条件化 NeRF，学习跨场景可迁移先验；pixelSplat、MVSplat 将前馈少视图合成扩展到显式高斯。② 稀疏场景优化：RegNeRF、SimpleNeRF 等通过几何正则或更简单的代理约束稀疏重建，DietNeRF 引入语义一致性，DS-NeRF/SparseNeRF 以深度监督缓解欠约束。③ 生成式 NVS：Zero-1-to-3 等从相对位姿条件化单图生成；ZeroNVS 在真实场景上训练扩散；ReconFusion 结合 PixelNeRF 条件与扩散先验；CAT3D 学习多视角扩散并配合三维重建生成一致渲染。④ 多视角深度：单目零样本深度（MiDaS、Depth Anything 等）重视跨域，Marigold/DepthFM 用扩散先验；MVSNet 等利用多视角几何，DeFiNe 涉及新视角深度。文中也对 GRIN、EPIO、NeuralRecon 等进行深度比较。

**差别与限制**：MVGD 的关键区别不是首次使用扩散或相机条件，而是不要求场景级显式中间表示，通过目标射线、可选任务与共享隐式 latent 直接生成 RGB 或深度，兼顾不齐标签和更多条件视角。以上为作者组织的相关工作及技术背景，不等于对每篇基线独立精读；CAT3D 的重建细节、预训练集重叠和全部运行条件尚未独立核验。

本单元不承担独立数学推导。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-RIN -->
### RIN：像素空间扩散的计算接口（PDF §3.1.2，Figure 2）

**读写机制**：令输入 $X\in\mathbb R^{M\times D}$（$M$ 是输入 token 数，不与相机数 $N$ 混淆），内部可训练 latent $Z\in\mathbb R^{L\times D}$。每个块先用 latent 作 query、输入作 key/value 读取：$Z'\leftarrow\mathrm{CrossAttn}(Q=Z,K=X,V=X)$，随后在 $Z'$ 上做若干自注意力，再以输入作 query、latent 作 key/value 把整合后的信息写回 $X$。这里公式是依原文描述的机制示意，省略投影、残差和归一化；并非原文逐层代码。场景 token 与目标预测 token 可由固定 latent 汇聚，目标预测 token 写回后用于估计噪声。输入的当前噪声状态与相机射线共同决定预测方向。

**复杂度**：固定特征维时，直接自注意力成本约 $O(M^2D)$；两次读写交叉注意力约 $O(MLD)$，latent 自注意力约 $O(L^2D)$。因此只说主要 self-attention 不随输入增长，绝不是总体复杂度与输入无关。以正方形 256 图像为例，一个任务有 65,536 个目标 token，每条件视角 4,096 个；若有 5 个条件视角，$M=86,016$。$L=2,048$ 远小于 $M$，但交叉注意力仍需要处理所有输入。本文直接在 RGB/深度参数化像素上去噪，内部 RIN latent 不同于 VAE 编码的图像潜变量；无 VAE 不代表无神经特征压缩。

<!-- PTH:END METHOD-RIN -->

<!-- PTH:BEGIN METHOD-SSN -->
### SSN：场景尺度归一化（PDF §3.2）

**尺度归一化**：论文使用相对外参 $\tilde T_c^n=T_c^nT_t^{-1}$，目标相机相对外参为单位阵。将相对变换平移分量的绝对值最大者定义为 $s=\max_n\|\tilde t_c^n\|_\infty$；相对平移均除以 $s$，训练时深度标签也除以 $s$，预测时按 $s$ 恢复。这样输入相机的相对平移量与目标深度共享一个缩放，避免不同标定单位给模型带来不必要变化。例：相同场景用米或厘米表示时，平移和深度一起变为原来的 100 倍，归一化量相同，恢复出的深度也变为 100 倍；不是从图像推知米制单位。目标相机固定为原点也减少全局平移和旋转坐标系选择的影响。

**边界与错误**：若 $s=0$（纯旋转或完全同位姿），除法无定义；原文未给具体下限或退化处理。深度超过 $d_{max}$ 时原文提出 $s'=s D_{max}/\max\tilde D$，但若 $\max\tilde D>d_{max}$ 且 $D_{max}$ 为该阈值，$s'$ 变小，会让归一化深度更大，方向反了。满足目的的推导为 $s'=s\max\tilde D/d_{max}$，详见数学单元；未核验代码，不能断言实现采用错误式。推理无法依赖真实深度调整尺度，训练/测试可存在范围差异。附录 E 还明确由于每个目标都在原点，该设计不能同时生成多个目标相机。

<!-- PTH:END METHOD-SSN -->

<!-- PTH:BEGIN METHOD-ENCODERS -->
### 相机与图像条件编码（PDF §3.3）

**图像编码**：每个条件图像通过预训练 EfficientViT-SAM-L2（整体训练中端到端微调），得到 $F_I\in\mathbb R^{H/4\times W/4\times448}$；展平为 $HW/16$ 个 448 维向量后线性投影至 $D_I=256$。即每个 token 对应下采样空间位置的语义/外观特征，而不是每个输入像素直接作一个 3 通道值。

**射线编码**：为相机 $k$ 的像素 $(u,v)$ 计算原文所称起点 $t_{ijk}=[x,y,z]_k^T$ 和方向 $r_{ijk}=(K_kR_k)^{-1}[u,v,1]^T=R_k^{-1}K_k^{-1}[u,v,1]^T$。后者把相机像素反投影至归一化场景坐标系；原文使用 Fourier 编码，频率数 $N_o=N_r=8$，最高频率 100，并报告 $D_R=3(N_o+N_r+1)=51$。具体正余弦组合未展开，不能擅自改成常见双倍维数。条件相机内参随特征图分辨率缩放，例如原像素尺寸缩到 1/4 时焦距、主点也按坐标约定缩放。目标相机保留完整分辨率且 $t_t=0,R_t=I$，因此每目标像素都有空间方向条件。注意若 $T$ 按常见 world-to-camera 定义，矩阵平移并非相机中心（$C=-R^{-1}t$）；论文将 $t$ 称射线起点但未细述转换，重现时必须核实实现坐标约定。

<!-- PTH:END METHOD-ENCODERS -->

<!-- PTH:BEGIN METHOD-MULTITASK -->
### 多任务共享机制（PDF §3.4）

**任务嵌入**：直接将 RGB 与深度拼成固定 RGB-D 输出需要同时拥有两类标签；反之把任务只放在内部 latent 上会使不同任务分离。论文把可学习的 $E^{task}\in\mathbb R^{128}$ 拼到每个目标 token，模型根据具体 token 的任务条件选择生成 RGB 或深度，且两类目标 token 可以共同出现、共享同一组 latent。输入条件仍是 RGB 与相机，而不是必须有 RGB-D。训练可从不同数据集中抽取 RGB-only、稀疏深度或两任务样本，缺失深度的像素不参与深度监督。单个任务不依赖同一图像拥有另一任务标签。作者的期望是外观与几何先验共享，而不使用一个显式三维场景作为硬约束。

**状态与监督**：RGB 状态是三维，深度状态为标量；在归一化或对数参数化后加入噪声，预测网络按式(1)估计噪声。原文称 RGB 使用 L2、深度使用 L1。两类损失的具体权重、不同尺寸状态进入统一 Transformer 的投影细节没有充分给出。$d_{min}=0.1,d_{max}=200$ 为归一化后非米制深度范围。原文 RGB 与深度逆式存在代数不一致，不能直接照抄为正确实现，详见数学单元。Tab5 B/C 的单任务对照与 F/G 增任务微调支持共享先验的收益，但不保证两个任务生成的每个边界完全匹配。

本单元不承担独立数学推导。

<!-- PTH:END METHOD-MULTITASK -->

<!-- PTH:BEGIN METHOD-TOKENS -->
### 两类 token 与端到端生成（PDF §3.5、§4.1）

**两类token**：条件相机的图像和射线 embedding 按通道拼接，$E_c^n=E_c^{I,n}\oplus E_c^{R,n}\in\mathbb R^{HW/16\times307}$；再沿 token 轴拼接不同视角，$E_c\in\mathbb R^{NHW/16\times307}$（假定统一分辨率）。预测 token 则拼接目标射线、任务嵌入、当前扩散状态：$E_t^{pred}=[E_t^R,E^{task},S_t^{task}]$，原始特征维为 RGB $51+128+3=182$，depth $51+128+1=180$；模型内部统一维的映射为必要实现环节，论文未具体给出。每次训练抽 2–5 个条件视角，每视角随机取 $M_s=1024$ scene tokens；另外抽 $M_t=4096$ prediction tokens（单任务全部使用或两任务各半）。深度只从有效标签像素抽取。随机 token 抽样降低训练成本，不等同于只给少量相机。

**生成流程**：训练阶段从真实目标 RGB 或深度参数化出 $x_0$，按时间步加噪得 $x_t$，编码条件后 RIN 预测噪声并监督。推理阶段不需要目标 RGB/深度作为真值：从三通道或一通道标准高斯随机状态出发，按 DDIM 迭代10步，取 $t=0$ 参数化预测再反变换；相机与任务条件一直控制预测。论文报告五样本 ensemble 以缓和随机性，未说明用均值、中位数或其他融合，不能把它认作后验不确定性校准。目标相机方向在每个预测像素上提供定位信息；scene token 提供实际可见外观和几何线索。

<!-- PTH:END METHOD-TOKENS -->

<!-- PTH:BEGIN METHOD-INCREMENTAL -->
### 增量多视角生成（PDF §3.6、附录D）

**生成历史**：给定初始真实条件集合 $\mathcal I_C$，先生成目标 $j$ 的图像 $\hat I_j$，加入历史 $\mathcal I_G=\{(\hat I_g,\mathcal C_g)\}_{g=1}^G$，下一个目标由 $\mathcal I_C\cup\mathcal I_G$ 条件化。附录D按尚未处理的绿色相机中最靠近初始条件相机的顺序生成；原文写明全部已有生成视角均作为额外条件。这里附加的是生成 RGB 和已知相机，不是将预测深度直接融合为三维模型。动机是未观测区域一次采样选定外观后，后续邻近视角能够重复利用这一选择，减少独立采样产生相互矛盾的结果。

**复杂度与风险**：新增生成图像意味着更多 scene token，而 RIN 的 $L$ 不变，故主要自注意力成本保持；交叉注意力和存储仍随条件数增长。附录D在 A100 40GB 上报告 25 条件约0.5s，1250 条件约20s。生成历史不是新的真实传感器观测，无法从信息论上消除真实场景的歧义；错误生成还可能被后续条件强化。Figure 8 所示指标改善是特定序列经验结果，不证明误差不累积。附录提出仅使用靠近目标的生成视角作为未来效率启发式，本文未实证该方案。

本单元不承担独立数学推导。

<!-- PTH:END METHOD-INCREMENTAL -->

<!-- PTH:BEGIN METHOD-SCALING -->
### Latent 容量扩容与微调（PDF §3.7、§4.5）

**扩容策略**：模型训练完成后，把 latent 序列从 $Z_L\in\mathbb R^{L\times1024}$ 复制拼接成 $Z_{2L}=[Z_L;Z_L]$；注意力层使用同一权重处理可变长度序列，其他架构不变，再微调 50k 步。256→512→1024→2048 各扩容阶段逐步增加内部处理槽位。复制的向量初始相同，若所有操作严格置换对称且确定性，完全相同的槽位可能保持相同；具体如何打破对称性，原文未提供细节。不能杜撰噪声初始化、位置编码或独立梯度机制。可观察到的是 Table8 的实际性能改善。

**优化与代价**：256 latent 初始模型训练300k步，后续每阶段50k、半学习率；参数量只从417.9M增至419.7M，而 latent self-attention 长度项从 $256^2$ 增至 $2048^2$（64倍），交叉注意力长度项8倍。整体延时实测约0.3→1.1秒，并非64倍，因为多个子模块有不同成本。作者称2048模型单步训练约4倍，从头训练约比增量方案慢3倍；Tab5 A 的步数叙述与表注不一致，不能从这组数值确定精确计算节省。扩容还增加训练步数，因而不是等计算量的纯容量对照。

<!-- PTH:END METHOD-SCALING -->

<!-- PTH:BEGIN MATH-DIFFUSION -->
### 扩散数学：公式 (1)（PDF p.3–4，§3.1.1）

**公式 (1)**：$x_t=\sqrt{\alpha_t}x_0+\sqrt{1-\alpha_t}\epsilon$, $\alpha_t=\prod_{j=1}^t(1-\beta_j)$, $\epsilon\sim\mathcal N(0,I)$。$x_0$ 为参数化的 RGB 或深度目标；$t$ 是扩散时间步（不同于目标相机下标），$\beta_j$ 为方差日程。由单步高斯加噪 $x_j=\sqrt{1-\beta_j}x_{j-1}+\sqrt{\beta_j}\epsilon_j$ 出发，独立零均值噪声的方差递推 $v_j=(1-\beta_j)v_{j-1}+\beta_j$，初值 $v_0=0$，解为 $v_j=1-\alpha_j$，均值系数为 $\sqrt{\alpha_j}$，故任意时刻可直接采样，不需要训练时遍历所有加噪步。边界：$\alpha_0=1$ 无噪声；充分小的 $\alpha_T$ 使状态接近标准高斯。这里单步推导为标准扩散背景补充，原文只给闭式 (1)。

**训练与采样**：网络 $\hat\epsilon=f_\theta(x_t,t,c)$ 以 scene/ray/task 等条件 $c$ 估计噪声。若用预测噪声反解，$\hat x_0=(x_t-\sqrt{1-\alpha_t}\hat\epsilon)/\sqrt{\alpha_t}$。原文报告 RGB L2、depth L1，可概念性写成 $\|\epsilon-\hat\epsilon\|_2^2$ 与 $\|\epsilon-\hat\epsilon\|_1$；未给出归一化、权重、全部任务的具体聚合，不能视作原文显式联合目标。以确定性 DDIM 更新为背景示意（并非论文所列完整采样实现）：从当前步 $t$ 到选定较早步 $j$，$x_j=\sqrt{\alpha_j}\hat x_0+\sqrt{1-\alpha_j}\hat\epsilon$（随机性参数为零时）；本文只明确 DDIM 训练1000步、评估10步。不同初始噪声对应不同样本，E=5 ensemble 不等于经校准的可信区间；多模态的均值甚至可能不对应一个物理场景，具体融合未知。

<!-- PTH:END MATH-DIFFUSION -->

<!-- PTH:BEGIN MATH-GEOMETRY -->
### 相机几何与公式 (4)（PDF §3.2–3.3、附录A）

**相对相机与射线**：在 world-to-camera 约定中 $X_c=T_c X_w$，$X_t=T_t X_w$，所以 $X_c=T_cT_t^{-1}X_t$。这与论文 $\tilde T_c=T_cT_t^{-1}$ 一致；若世界坐标作任意同一刚性变换 $A$，则 $T'_c=T_cA^{-1}$、$T'_t=T_tA^{-1}$，故 $T'_c(T'_t)^{-1}=T_cT_t^{-1}$。射线方向 $r=R^{-1}K^{-1}[u,v,1]^T$，若以光轴 z-depth 为参数，世界点 $X=C+d r$，其中相机中心 $C=-R^{-1}t$；若深度定义为欧氏射线长度，必须先归一化方向。原文并未明示所有数据深度与射线规范，实现时应确认。全局单位扩大 $a>0$，平移与深度同乘 $a$，$s'=a s$，归一化相同、输出恢复为原深度的 $a$ 倍，故 scale equivariance 是归一化逻辑的性质，不保证模型几何准确。

**公式 (4)**：附录 A 原文印为

$[u',v',d',1]_t^T=(\tilde K_c^n T_c^n)^{-1}[u,v,d,1]_c^T(T_t\tilde K_t)$，其中 $\tilde K=\begin{bmatrix}K&0\\0&1\end{bmatrix}$。按列向量记法，4×1 列向量右乘 4×4 矩阵不成立；且像素 $(u,v)$ 需与 z-depth 同乘构成齐次投影向量，不能直接将 $(u,v,d)$ 当作三维点。这里必须保留原式，并另给标准列向量推导而不是默认为实现：$X_c=dK_c^{-1}[u,v,1]^T$，$[X_t,1]^T=T_tT_c^{-1}[X_c,1]^T$，令 $q=K_tX_t$，$d'=X_{t,z}$，$u'=q_x/q_z,v'=q_y/q_z$。等价齐次向量 $[ud,vd,d,1]^T$ 在合适内参约定下经 $\tilde K_tT_tT_c^{-1}\tilde K_c^{-1}$ 变换后，再除以目标 z。需检查 $d'>0$ 且 $u',v'$ 在图像边界；原文写 $u'\in[0,H],v'\in[0,W]$，常见 u 横 v 纵约定则为宽、高对应，应以数据坐标定义为准。论文仅描述投影落入视野、重叠阈值30%且至少64有效像素，没有说明 z-buffer 遮挡检查，不能称为精确可见性检测。

<!-- PTH:END MATH-GEOMETRY -->

<!-- PTH:BEGIN MATH-PARAM -->
### 深度、RGB 与尺度参数化审查（PDF §3.2、§3.4）

**原文公式 (2)–(3)**：

(2) $P_D=2\frac{\log(D/(s d_{min}))}{\log(d_{max}/d_{min})}-1$。

(3) 原文 $\hat D=\exp((2\hat P_D+1)\log(d_{max}/d_{min}))d_{min}s$。

$D>0$ 为原相机尺度下深度，$s>0$ 为 SSN，$d_{min}=0.1,d_{max}=200$ 是归一化范围，$P_D$ 是送入扩散的标量。设 $q=D/s,r=d_{max}/d_{min}=2000$。式(2)给 $\log(q/d_{min})=(P_D+1)\log r/2$，所以其严格代数逆式应为

$\hat D=s d_{min}\exp\big(\frac{\hat P_D+1}{2}\log r\big)$。

端点检验：$P_D=-1\mapsto0.1s$，$P_D=1\mapsto200s$；原文(3)却在 -1 时给 $0.00005s$、1 时给 $8\times10^8s$，不可能是式(2)的逆。中点 $P_D=0$ 对应 $s\sqrt{d_{min}d_{max}}=s\sqrt{20}$，约 $4.472s$，体现对数尺度的几何均值而不是线性均值。导数 $\mathrm dP_D/\mathrm dD=2/(D\log r)$，局部相同的参数误差对应相近相对深度误差。注意这只是变换的性质，实际去噪监督不是直接对最终相对误差优化。该代数纠正是读者核查，不宣称已检验实际实现。

**RGB与截断核查**：原文 $P_{RGB}=(I+1)/2$，声称映射至 [-1,1]；逆式写 $\hat I=2\hat P_{RGB}+1$。对所写正向式，逆应为 $I=2P_{RGB}-1$。若通常图像 $I\in[0,1]$，希望映射到 [-1,1]，应使用 $P_{RGB}=2I-1$ 与 $I=(P_{RGB}+1)/2$；若 $I\in[-1,1]$，正向原式反而映射到 [0,1]。原文未给进一步实现说明。另设 $m=\max(D/s)>d_{max}$，要求最大归一化深度不超过阈值，原文 $s'=s d_{max}/m$ 将新最大值变为 $m^2/d_{max}>m$；令 $s'=sm/d_{max}$ 才得 $\max(D/s')=d_{max}$。若 m=400、阈值200，原文使新最大800、正确方向使新最大200。公式不确定性另包含 $s=0$、低于下界和预测超出参数范围的处理，原文未明确裁剪细节。

<!-- PTH:END MATH-PARAM -->

<!-- PTH:BEGIN EXP-TRAIN -->
### 训练协议及数据预处理（PDF §4.1–4.2、附录A）

**训练设置**：RIN 6 blocks、depth 4、16 latent/read-write heads，初始 $Z\in\mathbb R^{256\times1024}$；图像最长边256，$D_I=256$、$D_{task}=128$，ray 频率数8/8、最大100。2–5条件视角，每视角1024 scene tokens、目标4096 prediction tokens。DDIM 1000 train/10 eval，sigmoid 噪声日程，EMA 0.999。AdamW $\beta_1=0.9,\beta_2=0.99$，batch512，weight decay 0.01，LR $10^{-4}$；线性warmup10k，cosine decay，初始300k步，各扩容50k步半学习率。约6000 A100 GPU-hours 为作者报告的总训练计算量，不是本地复现测得。图像特征编码器有预训练，不能将全文“from scratch”理解为所有参数随机初始化。

**数据准备**：Table1共19数据集；官方训练分割可得时按官方训练集。附录A统一图像、内外参、深度存储格式，缺失深度使用空图；把视频和多相机放入统一相机序列，不要求全套稠密RGB-D。以序列最大相机间距 $c_M$ 定义距离过滤 $0.05c_M<\|t_c-t_t\|<0.2c_M$；动态数据另约束时间距离，原文 $t_{min}=-8,t_{max}=8$ 与范数的非负性结合主要约束上界。相机方向夹角深度任务限制至 $\pi/2$，RGB 至 $\pi$，后者鼓励外推。若有深度则依据附录式(4)投影，要求重叠比例至少30%且至少64有效投影像素。WebDataset 优化存储/读取。稀疏与缺失标签采用有效像素训练，不应自动解读为所有数据都有深度；部分标签度量尺度、部分SfM尺度。数据筛选本身限制训练分布，远超该视角/时间范围的效果需另验证。

本单元不承担独立数学推导。

<!-- PTH:END EXP-TRAIN -->

<!-- PTH:BEGIN EXP-NVS -->
### 新视角 RGB 与新视角深度实验（PDF §4.3，Table2–4）

**新视角评测**：2-view 在 RE10K 和 ACID 对比 PixelNeRF、GPNR、AttnRend、MuRF、pixelSplat、MVSplat；MVGD PSNR分别28.41/29.98、SSIM 0.891/0.875、LPIPS 0.107/0.131。3/6/9-view 用 RE10K、LLFF、DTU、CO3D、MIP-NeRF360，与 FreeNeRF、SimpleNeRF、ZeroNVS*、ReconFusion†、CAT3D† 比较；域内数据集为 RE10K/CO3D，其余零样本。训练2–5视角却能评估6/9，支持可变条件规模。所有 PSNR 列优于表中基线；LPIPS 和SSIM不是全部最优，例如 RE10K 9-view LPIPS 0.085 对 CAT3D 0.082。背景定义：PSNR $=10\log_{10}(MAX^2/MSE)$，SSIM 衡量亮度/对比度/结构，LPIPS 特征距离反映感知差异；三指标都衡量目标图像而非完整三维几何。不同模型训练数据、场景优化与计算量并不一致，不能直接将增益解释为单一架构因素。

**深度评测**：Table4 以稀疏 COLMAP 点投影参考深度，不进行测试时尺度对齐。Abs.Rel. $=\frac1M\sum_i|\hat d_i-d_i|/d_i$、RMSE $=\sqrt{\frac1M\sum_i(\hat d_i-d_i)^2}$、$\delta_{1.25}=\frac1M\sum_i\mathbf1[\max(\hat d_i/d_i,d_i/\hat d_i)<1.25]$（标准指标解释，原文未列逐项实现）。MIP-NeRF360 3→9-view Abs.Rel. 0.143→0.109，LLFF 0.105→0.093，CO3D 0.101→0.079；均有更多真实观测帮助。Figure3/4 可视化点云展示几何，但稀疏参考不能覆盖所有遮挡和纹理不足位置。Table4 仅给 MVGD，不宜称其在 novel depth 上超过其他模型；未进行 test-time alignment 支持相机尺度一致，不证明绝对度量真值。

<!-- PTH:END EXP-NVS -->

<!-- PTH:BEGIN EXP-MVS -->
### 多视角深度估计（PDF §4.4，Table6–7）

**立体与视频深度**：与 novel depth synthesis 不同，这两组任务的目标图像也包含在条件集合，目标只是生成该视角的深度。Stereo 使用重叠的两图；Video 用10帧。虽然训练时目标视角为独立追加的视角，推理时可把目标图像本身加入条件，利用自身外观和其他帧的几何。Stereo 对比 DPSNet、NAS、IIB、DeFiNe、EPIO、GRIN，在 ScanNet/SUN3D/RGB-D 的所有报告指标上 MVGD 最优；Video 对比 DeMoN、MiDaS-v2、BA-Net、CVD、DeFiNe、DeepV2D、NeuralRecon，MVGD Abs.Rel. 0.041、Sq.Rel. 0.007、RMSE 0.143。Sq.Rel. 一般为 $\frac1M\sum(\hat d-d)^2/d$，具体实现未展开。

**结论边界**：ScanNet 域内，SUN3D、RGB-D 零样本；视频实验只在 ScanNet 域内。此实验显示训练的隐式几何先验还能转为已知目标图像的多视角深度估计，不能证明动态时序建模或新视角的所有遮挡几何正确。作者称无需专门几何保持增强、equivariance 架构或 TSDF；不表示不存在任何几何输入或归一化设计。

本单元不承担独立数学推导。

<!-- PTH:END EXP-MVS -->

<!-- PTH:BEGIN EXP-ABLATION -->
### 消融与容量实验（PDF §4.5–4.6）

**消融设计**：Table5 A 从头训练2048 latent；B去RGB，C去深度，D去SSN，E去动态训练数据；短模型200k步，F在B基础上加入RGB、G在C基础上加入深度，追加100k；full为256 latent的300k模型。A训练步数正文/表注冲突（§4.5 300k，表注上半部分200k），B/C正文的任务文字次序与表标签冲突，保留并以数据列界定。以short对照：C RE10K PSNR 21.46 对24.44，说明仅RGB不及共享任务；B LLFF Abs.Rel. 0.138 对0.131，但 CO3D 0.132 对0.127，差距较小。D LLFF 16.87/0.224 对20.19/0.131，ScanNet 0.329 对0.081，SSN影响显著。E数据量约少12%，ScanNet 0.087对0.081；数据多样性与数量共同变化。F/G增加任务后，F ScanNet 0.075 与full相同，G RE10K 25.68接近25.89，说明已学模型可拓展任务，而非一定从零开始。未报告多随机种子方差或等计算量控制。

**扩容成本**：Table8从256/417.9M到2048/419.7M，RE10K PSNR +2.52 dB，LLFF +1.81 dB，CO3D Abs.Rel. 0.121→0.101；虽然增加不到0.5%可训练参数，计算量仍提高。每阶段50k微调，整体450k；256模型约0.3s，2048约1.1s；2048从头训练单步约4倍，作者估计总成本较增量法约3倍，但确切等步/等时间比较尚不充分。Table8 MIPNeRF360 3-view SSIM 0.465与Table3 0.425不同，不能静默择优。

本单元不承担独立数学推导。

<!-- PTH:END EXP-ABLATION -->

<!-- PTH:BEGIN EXP-APPENDIX -->
### 附录实证与边界（PDF 附录A–E）

**附录实证**：A提供数据筛选（详见训练单元）。B Figure4展示更多单目标结果，Figure5展示同一条件下独立生成点云堆叠，无额外对齐后处理，支持 SSN 与隐式几何一致性。C Figure6 用 DDAD 100帧每第10帧作为条件，车辆位置沿生成序列变化、静态背景相对稳定；未显式使用时间/运动tokens，不能证明泛化的动力学重建。C文字引用Table8作为动态数据消融，实际对应Table5 E。D Figure7 ScanNet 1267帧，图示 stride 200/100/50/25/10；文本称约100条件（stride20）饱和，与图例/帧数存在差异，保留定性趋势不替原文确定精确饱和点。Figure8 固定25真实条件（stride50）与增量追加生成RGB比较，远离条件的位置相对收益较大；这种改进不同于真实观测增加。Figure9 长序列149、202、252、337、1268、5578帧，初始5、7、5、6、25、60个条件视角。生成顺序从最接近初始条件的未处理目标开始，每次加入此前生成RGB。

**扩展边界**：附录D在A100 40GB可处理千级条件，25→1250条件耗时0.5→20s；随输入增长并非免费。作者称整体复杂度 largely independent，应理解为主要 latent 自注意力固定，而非整体独立。附录E明确 SSN 单目标原点限制无法同时生成多个相机，且缺少显式动态建模；长序列一致性只是经验，错误历史有传播风险。动态场景真正的时空控制、时间embedding、motion token 与更大精确动态数据集被列为潜在未来方向。

本单元不承担独立数学推导。

<!-- PTH:END EXP-APPENDIX -->

<!-- PTH:BEGIN SYNTHESIS-EVIDENCE -->
### 主张、证据、边界矩阵

**主张—证据矩阵**：

| 作者贡献 | 最直接证据 | 可以支持 | 不足以支持 |
| --- | --- | --- | --- |
| 直接几何多任务扩散 | Figure2；Table5 B/C/F/G，Table2–4、6–7 | 同一隐式模型生成两任务，几何监督助益RGB，可增加任务 | 所有RGB-D样本严格全局一致，优于任何显式方法 |
| SSN与大规模异质训练 | Table1、Table5 D/E，Table4和Figure5 | 外参尺度一致的经验收益；多样数据适用、缺失深度可训练 | 米制深度可从任意尺度恢复；动态属性单独造成增益 |
| RIN与增量扩容/条件 | Table8、Figure7–9 | latent容量、训练微调、更多输入和伪条件有经验收益 | 恒定整体时间复杂度、无误差累积、等计算量严格因果证明 |

**未证实与局限**：单个目标坐标原点使同时多目标预测不直接成立；随机生成可逼真但与真实场景不唯一，SSN解决尺度单位而非不可辨识性；稀疏COLMAP深度只约束被可靠匹配的点；在已有输入RGB时的深度估计不可混同新视角深度；附录动态例子不代表显式时空模型；训练数据大、预训练encoder、基线预训练与数据重叠均是比较边界。论文的代码及全部数据复现本次未验证。

本单元不承担独立数学推导。

<!-- PTH:END SYNTHESIS-EVIDENCE -->

<!-- PTH:BEGIN SYNTHESIS-AUDIT -->
### 阅读覆盖与疑点记录

**覆盖审计**：本报告按计划覆盖标题、摘要、贡献、结论、Figure1–9逐图原题注与译文、Table1–8逐表原值与解释、§1研究问题动机、§2相关路线、§3.1扩散/RIN、§3.2 SSN、§3.3编码器、§3.4多任务、§3.5 token/采样、§3.6增量生成、§3.7扩容、§4.1–4.6训练与实验、§5结论及附录A–E。公式(1)–(4)逐项保留原文内容并解释；额外复杂度、标准投影、深度指标及代数逆变换均标作背景/推导，非作者额外结论。参考文献目录保留原始PDF，不将未独立精读的引用说成已审查。

**来源与疑点**：原件 arXiv:2501.18804v1、20页，Source及HTML辅助，HTML外部资源不全；Figure5/7/8区域PDF回渲染差异已在B2记录。原文RGB正逆变换、式(3)、SSN调整方向、附录式(4)列向量乘法及像素深度坐标存在代数或记法疑点；B/C文字、A训练步数、表3/8 SSIM、附录C错误表号、附录D饱和条件数内部差异均明确记录，不代替作者更正。射线原点坐标、尺度为零、状态投影、ensemble聚合等实现细节未给出或未验证。无本地训练、源码运行、独立审校或Hexo部署的声称。

本单元不承担独立机制解释。

本单元不承担独立数学推导。

<!-- PTH:END SYNTHESIS-AUDIT -->



---

# 第二部分：学习与研究笔记

# MVGD 学习与研究笔记

> 精读报告 SHA-256：8ec33019c0c3517da7312e3ae15f0f5a32925807a358bad96205c12b43e16d87
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：核心知识内化

### 1. 隐式生成与显式重建不是同一个逆问题

精读 §研究问题、§RIN、Figure 2：显式重建先从观测估计共享场景变量 $G$，再计算 $I_t=\mathcal R(G,C_t)$；生成路线直接对 $p(I_t,D_t\mid\mathcal I_C,C_t)$ 采样。两者都有先验：显式形式约束共享场景，隐式形式把知识放在网络和训练数据。假设只有墙的一面被观察，另一面是窗或门都可满足输入；独立生成可对两个目标相机选择不同假设，而一个固定 $G$ 强迫两次渲染采用同一假设。反之，如果固定 $G$ 重建错了，渲染一致性并不等于真实性。区别不在是否有几何，而在几何被表示、约束和验证的方式。

### 2. 相机尺度一致 vs. 真值深度

精读 §SSN、§深度参数化、Table 4/5：统一单位消除相机坐标尺度干扰。补充推导：针孔投影 $u=f_xX/Z+c_x$，$v=f_yY/Z+c_y$；若三维点与相机中心同乘 $a>0$，投影比值不变。因此在仅 RGB、相机姿态的平移只有 SfM 任意尺度时，米制尺度不具有可辨识性。SSN 传递已给尺度而不凭空恢复绝对尺度；如果平移准确为米，输出也使用米，但依然存在相对深度错误。反事实：若相机标定有尺度漂移而非单一全局系数，单标量 SSN 无法修复；若纯旋转，基线为零，$s$ 退化。不同目标相机各自归一化，恢复尺度相同也不保证点云相同。

### 3. 预测 token 的任务控制与几何监督

精读 §多任务、§两类 token、Table 5：将 task embedding 放在每个预测位置，使 RGB 与 depth 不必共享输出空间，却可以访问共享表示。抽象为 $h=F(E_C, E_{pred})$，任务输出 $g_k(h,k)$；仅当标签存在时对该任务计算损失。设计上的优势是图像数据和稀疏深度标签都可参与；不能从共享权重直接推出 $\hat I$ 和 $\hat D$ 遮挡边缘一致。若增加跨任务重投影一致性损失，可能强化几何，但也可能把错误的生成深度反馈给 RGB；需考虑可见性及置信度。本文未采用这种额外硬约束。

### 4. RIN 的容量与成本不是参数量

精读 §RIN、§扩容、Table 8：$M$ 个输入 token 读写 $L$ 个 latent，近似成本为 $O(MLD+L^2D)$，不等于 $O(L^2D)$。在固定 $M$ 时扩大 $L$ 可增加信息槽数，但也增大 attention FLOPs/显存；权重共享使参数增量主要来自槽向量，而非各层矩阵倍增。反事实：保持参数不变增加目标图像分辨率，$M$ 增长依然加大交叉 attention。网络能访问所有输入并不意味着每个输入的局部细节都被有效保留，固定瓶颈也可能稀释罕见证据。

### 5. 伪观测的一致性收益与信息边界

精读 §增量生成、Figure 8/9：真实观测 $O$ 与独立模型噪声 $\epsilon$ 生成 $Y_1=g(O,\epsilon)$；若噪声不包含真实未知场景 $S$ 的额外信息，则给定 $O$ 后 $S$ 与 $Y_1$ 条件独立，$I(S;Y_1\mid O)=0$。这是补充的理想化信息论解释，不是论文提出的定理；先验通过模型参数已经参与推断。将 $Y_1$ 作为条件可让两个生成结果协调，但不是从外部增加真实测量，甚至可能把不正确假设锁定。真实新增相机观测与伪条件应在研究设计中分开统计。若生成历史只被当作先验而非等权传感证据，理论上可能减弱错误传播；待检验。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Problem & Task — Novel Depth Synthesis
- **Keywords:** Novel View Synthesis; Novel Depth Synthesis; Multi-View Depth Estimation; Posed Images
- **Highlight:** MVGD 从已知相机的条件图直接预测未观测目标 RGB/深度；目标 RGB 可用时可转为立体或视频深度估计。两种任务的观测信息并不相同。
- **Evidence:** §3、§4.3–4.4、Table 4/6/7；冻结报告研究问题与多视角深度单元。
- **Connection Points:** 输入是否包含目标 RGB；是否给定目标内外参；是否有场景级优化；深度参考尺度。
- **Open Questions:** 目标 RGB 缺失时深度误差如何随遮挡和基线变化？

### Topic: Geometry Inference — Scale and Ray Conditioning
- **Keywords:** Scene Scale Normalization; Raymap; Scale Equivariance; Camera Extrinsics
- **Highlight:** 以目标相机为参考归一化所有相对平移，并对深度采用同一尺度；射线编码把相机条件作用到特征位置和目标像素。
- **Evidence:** §3.2–3.3、Figure 2、Table 4/5 D、附录 E。
- **Connection Points:** 相机表示方式；尺度归一化和恢复；深度参数化；相机误差敏感性。
- **Open Questions:** 纯旋转、基线极小或非一致尺度漂移下如何稳定条件化？

### Topic: Training & Supervision — Heterogeneous Pixel Tasks
- **Keywords:** Task Embedding; Partial Supervision; Multi-Task Diffusion; Sparse Depth Labels
- **Highlight:** 每个预测 token 接收 task embedding；RGB/深度可单独或共享 latent 联合学习，深度仅有效像素监督，不要求每个数据源为稠密 RGB-D。
- **Evidence:** §3.4–3.5、§4.1–4.2、Table 1/5 B,C,F,G。
- **Connection Points:** 标签缺失策略；共享与任务专用表示；任务监督对几何和渲染的影响。
- **Open Questions:** 不同域的监督密度和标签噪声如何影响共享先验？

### Topic: Efficiency & Scalability — Recurrent Interface Networks
- **Keywords:** Recurrent Interface Networks; Cross Attention; Latent Bottleneck; Pixel-Space Diffusion; Incremental Fine-Tuning
- **Highlight:** 输入读写固定数量 latent，主要自注意力在 latent 上进行；复制并微调 latent 可逐步提高容量，训练与推理成本仍随规模变化。
- **Evidence:** §3.1.2、§3.7、§4.5、Table 8、附录 D。
- **Connection Points:** 输入序列长度和自注意力对象；容量扩容的参数/算力；局部证据压缩。
- **Open Questions:** 等算力下，增加 latent 和选择有信息量的观测何者更有效？

### Topic: Uncertainty — Incremental Generative Conditioning
- **Keywords:** Incremental Conditioning; Multi-View Consistency; Autoregressive Generation; Epistemic Ambiguity
- **Highlight:** 将已生成 RGB 和目标相机加入条件，减少未来独立采样的不一致；新增的是生成伪条件，不是独立传感测量。
- **Evidence:** §3.6、Figure 5/8/9、附录 D–E。
- **Connection Points:** 历史条件顺序；显式与隐式一致性；伪条件可靠性；真实几何验证。
- **Open Questions:** 当早期生成与真实场景偏离时，历史追加会修复还是强化错误？


## RESEARCH-DIRECTION：研究方向与可迁移性

以下为公开、一般性的 Sparse-Observation Underwater Vision 拓展分析，不是本文已验证的水下结果，也不记录私人研究计划。

### 任务层级

| 层级 | 本文位置 | 可讨论的水下方向 |
| --- | --- | --- |
| Field | 3D vision / generative vision | 水下视觉和三维感知 |
| Task | 新视角 RGB/深度；多视角深度估计 | 水下几何、外观与介质效应推断，可为新视角生成或场景恢复服务 |
| Task Setting | 2–9 已知相机真实条件为主，未观测目标 | 少量观测、相机尺度已知或不确定、颜色与对比度受介质影响 |
| Research Problem | 无显式3D场景时如何学习可迁移的几何生成先验并维持尺度/一致性 | 受限观测下哪些几何/介质/外观信息可辨识，如何约束不可靠推断 |
| Method | SSN、ray/task embedding、RIN、扩容与历史条件 | 可选组件，不预设必须选择扩散或隐式表示 |

### 直接联系与不可直接迁移之处

本文提供一个明确的稀疏观测实例：同一模型可在不同输入数下输出目标图像与深度，几何监督提高 RGB 合成表现（Table5），也能把生成结果用于后续条件（Figure8）。这对“联合推断何种信息”有启发，但不是水下成像物理模型。本文不包含介质参数、散射场或水下恢复的明确训练/评估；不能把其零样本能力直接外推到水下。输入相机已知也意味着不解决低纹理水下 SfM 位姿误差。

补充的简化水下成像示例：$I=J e^{-\beta z}+B(1-e^{-\gamma z})$（仅示意均匀介质、简化宽带系数，并非 MVGD 模型）。即使只有一个颜色通道且观测 $I$ 固定，不同 $J,z,\beta,B,\gamma$ 组合也可能产生相同观测；增添多视角有助于约束共享几何，但不同传播距离导致同一物点颜色不完全一致，传统亮度恒常性可能失效。先验可能选择合理的清晰纹理或深度，却不保证符合真实物理量。可研究的是如何分离由几何欠约束和介质造成的歧义，如何利用真实多视角/尺度证据约束生成，而不是仅将水下图像输入 MVGD。

### 适用边界

对于可视化、视角导航预览，合理外观和跨帧稳定可作为评价维度；涉及测绘、精确测量、近障操作或科学记录时，应要求真实深度/位置的验证及不确定性，必要时拒绝输出缺少证据的结果。研究设定应分别报告 RGB、几何、介质及校准误差，不能用 PSNR/LPIPS 代替几何真实性。相机尺度正确、真实观测与生成历史区分、合适的水下 GT 和域外验证是迁移前提。


## RESEARCH-PROBLEMS：可证伪的研究问题候选

以下均为基于本篇证据的开放问题和验证设计，不主张原创性，不等同已完成文献调研。

### RP-A：稀疏观测下生成一致性是否等同几何可信度？

**现象及科学未知**：本文 Table4 表明稀疏 COLMAP 点的深度误差随条件数下降，Figure5/8 表明点云与增量生成在可视化/图像指标上更加一致；但遮挡后的真实几何未知，伪条件有可能让错误自洽。科学未知为：真实观测证据与学习先验对生成几何的贡献能否被区分，并产生可靠的错误风险信号？

**可证伪假设**：在相同初始真实视角与训练模型下，利用真实观测重投影残差和多样本离散度进行置信门控，较全量无权重伪条件追加可降低遮挡区真实深度误差或置信失准；如果只改善图像观感而误差/校准不改善，假设不成立。注意离散度小也可能是共同偏差。

**验证设计**：构造有 GT 几何、完整相机标定的合成场景，并用真实数据具有测量参考的部分验证；固定初始相机数、基线分布、总生成步数和模型权重，比较 (a) 固定真实条件，(b) 全量伪条件，(c) 真实证据置信门控伪条件，(d) 加入同数真实观测的上界。以可见/未见、近/远基线分区计算绝对/相对深度误差、跨视角点云距离、RGB PSNR/LPIPS，报告误差与置信度的风险覆盖曲线。多随机种子，记录首个明显错误出现位置及后续传播。成功判据在相同覆盖率下减少高置信大误差，并不牺牲主要已观察区域；失败包括只降低跨视角残差、未降低 GT 误差。

**条件与限制**：需要稠密独立 GT，无法用 MVGD 的生成图像作 GT；现有模型未必提供可信的内在不确定性，需要额外实现与校准。待查生成不确定性、伪标签训练/推断、稀疏三维一致性及风险校准相关研究。优先级高，因为直接针对 Figure8 的信息边界而非重做已证实消融。

### RP-B：介质变化与几何歧义能否在稀疏多视角推断中解耦？

**现象及科学未知**：Table5 说明共享深度监督和 SSN 有益，但未检验介质变化。水下介质传输会随路径长度改变颜色，同一几何点在不同视角出现外观差异，低匹配一致性不能唯一归因于几何错误。未知是物理约束与跨场景先验能否共同减少错误的几何/介质分解。

**可证伪假设**：固定真实场景及相机，在不同介质强度下，加入适用的传播约束与观测可辨识性信号，较只依赖 RGB/通用 ray 条件先验可降低深度和介质参数误差；若改善仅来自更多训练数据或强制先验，等数据/等算力对照将消除收益，则不支持机制假设。

**验证设计**：合成已知几何、表面外观、介质参数的水下场景，在相同真实相机下独立改变传播强度、散射与纹理，加入标定误差及不同稀疏程度；比较直接RGB-D先验、只加介质模型、只加几何监督、联合约束，保持 backbone/训练预算相近。按已见/未见空间分别评估深度RMSE/Abs.Rel、清晰外观误差、介质传输曲线误差以及相机单位恢复；比较不同介质的错误相关性与 OOD 稳健性。若外观改善但深度或介质误差不降，不视为解耦成功。对真实数据需独立参考测量或可控制场景，不把无参考图像质量当物理证据。

**可行性与限制**：简化传输模型未必适用非均匀或复杂照明环境；存在多个等效参数组合，必须说明可辨识所需额外观测/先验。真实水下 GT 获取成本可能较高，先做可控的反事实试验。待查参与介质成像、跨视图物理一致性、稀疏视角重建与不确定性文献。本候选优先级取决于能否获得可信介质和几何参考；当前论文没有水下实证。

### RP-C：固定 latent 容量与观测信息选择的资源分配

**科学未知**：Table8 扩容改善、Figure7 增加观测改善，但两者成本均增长，不能从原文判断等算力预算下优先增加 latent 还是更有信息的条件。假设：在高冗余条件下，按目标可见性/互补基线选择视角比单纯扩容更有效；在真实稀疏且信息互补时，扩容可能更有利。用相同模型初始化、固定总 FLOPs 或墙钟、匹配训练步数，交叉变化 $L$ 与条件集，记录几何/外观、峰值显存、时延和非冗余区域误差；若视角选择只缩短时间而不提高固定预算内精度，假设不成立。需要能够复现模型及公平计时，未核实原代码实现，优先级低于 A，作为工程与科学交界的待检验问题。


## INTERNALIZATION：知识地图与自检

### 用自己的话复述

给定少量带位姿的真实图片，MVGD 不是先估计一个可渲染的显式三维场景，而是把图片特征与相机射线组成条件，把目标射线、任务和带噪状态组成查询。利用 RIN 把大量 token 的相互信息汇聚到有限的隐式 latent，再将结果写回目标像素完成去噪。RGB 和深度通过同一表示学习；标签不齐可分别监督。SSN 先令目标相机为原点，并用相对平移确定尺度，再将生成深度恢复到相机尺度。多个目标顺序生成时可复用此前生成的 RGB；增大 latent 容量可改善性能，但不等同新观测或无限计算。

### 技术与概念地图

| 矛盾 | 设计 | 直接证据 | 必须记住的边界 |
| --- | --- | --- | --- |
| 稀疏输入缺少目标表面信息 | 条件扩散与几何先验 | Table 2–4 | 生成合理不意味着唯一真实 |
| 异质相机单位 | 相对变换与 SSN | Table 4/5 D；Figure5 | 不是米制尺度恢复器 |
| 深度标签不足 | 预测端任务嵌入 | Table 1/5 B,C,F,G | 共享不保证联合样本严格一致 |
| token 数量巨大 | RIN读写 | §3.1.2、§4.5 | 仍有线性交叉attention成本 |
| 容量受限 | 复制 latent 微调 | Table8 | 步数与容量共同增加 |
| 多目标采样冲突 | 增量历史 | Figure8/9 | 伪条件不增加真实观测 |

### 理解自检

1. 为什么不是输入 RGB-D？因为条件主要为 RGB+相机，深度用于生成目标监督，并允许缺失；推理目标深度是输出而非输入必需。
2. 为什么不是 latent diffusion？因为被加噪/去噪的状态在参数化像素空间，RIN latent 只是内部计算接口，不是 VAE 编码图像。
3. 相机整体单位改为厘米，深度输出会怎样？在一致单位缩放和非零基线下归一化输入保持，恢复深度同乘100，不意味着误差消失。
4. 有 6/9 条件视角性能改善是否证明对任意相机都稳定？不能；训练只用2–5，评测覆盖有限，附录千级输入实验亦是特定序列。
5. 是否能直接生成同一三维空间的多个目标？附录E明确 SSN 目标原点限制；逐目标生成后点云可叠加，但不构成严格联合多视角生成保证。
6. 若公式(3)按印刷形式实现会怎样？深度端点不匹配式(2)；正确代数逆系数为 $(P+1)/2$；须核验实现，不能猜作者代码。
7. Table6/7为何深度任务更有约束？目标RGB本身作为条件，与无目标RGB的Table4不同。
8. 动态数据收益证明显式运动建模吗？否，Table5 E改变训练数据规模，Figure6仅提供定性运动现象。

**尚未解决**：真实观测缺少时的校准、动态时空建模、相机误差鲁棒性、生成历史错误传播及原文实现疑点。优先分辨任务输出是否要求物理真实性，再决定评价和约束。


## GLOSSARY：术语表

| 英文 | 中文 | 本文含义与区别 |
| --- | --- | --- |
| Novel View Synthesis (NVS) | 新视角合成 | 给定输入视图和目标相机生成目标 RGB；未必生成显式3D |
| Novel Depth Synthesis | 新视角深度合成 | 目标 RGB 未观测时生成其深度；Table4 |
| Multi-View Depth Estimation | 多视角深度估计 | 目标 RGB 也在输入中；Table6/7 |
| Zero-shot | 零样本 | 无针对评测域的微调；论文区分 in-domain，非所有基准零样本 |
| In-domain | 域内 | 训练已含同数据域，不等于同一测试样本 |
| Scene Scale Normalization (SSN) | 场景尺度归一化 | 按目标相机相对平移的最大分量统一尺度；§3.2 |
| Scale equivariance | 尺度协变 | 相机单位按系数变换，深度输出随之变换；区别于尺度不变/米制恢复 |
| Raymap | 射线图 | 每像素相机起点和方向，需明确外参约定 |
| Fourier encoding | 傅里叶编码 | 将几何变量编码为频率特征；本文给频率和维数，具体组合未展开 |
| Scene token | 场景 token | 条件视图的下采样图像embedding和ray embedding |
| Prediction token | 预测 token | 目标ray、task及当前噪声状态 |
| Task embedding | 任务嵌入 | 指示目标token生成RGB还是深度；不要求同源RGB-D标签 |
| State embedding | 状态嵌入 | 训练加噪/推理迭代去噪的参数化像素值 |
| Recurrent Interface Network (RIN) | 循环接口网络 | 固定latent读写输入token，在latent进行主要self-attention |
| Latent bottleneck | 隐式潜变量瓶颈 | 内部计算容量，区别于VAE图像压缩潜变量 |
| Pixel-space diffusion | 像素空间扩散 | 状态直接对应RGB或深度，而非图像VAE潜空间 |
| DDIM | 去噪扩散隐式模型 | 本文采样日程；训练1000/评估10步 |
| EMA | 指数滑动平均 | 本文模型训练中使用0.999系数 |
| Incremental fine-tuning | 增量微调 | 复制latent序列再训练，非向条件集追加生成图 |
| Incremental conditioning | 增量条件 | 顺序生成后将先前RGB作为额外条件，非额外真实观测 |
| Abs.Rel. | 相对绝对误差 | $\mathrm{mean}(|\hat d-d|/d)$，越低越好 |
| Sq.Rel. | 相对平方误差 | 通常 $\mathrm{mean}((\hat d-d)^2/d)$；实现细节未明 |
| RMSE | 均方根误差 | 绝对尺度敏感，越低越好 |
| $\delta_{1.25}$ | 阈值精度 | 预测与GT比值在阈值内的比例，越高越好 |
| LPIPS | 感知特征距离 | 图像感知差异，低为好，不是几何精度 |
| Identifiability | 可辨识性 | 观测能否唯一确定未知量；区别于模型能输出一项预测 |
| Multi-view consistency | 多视角一致性 | 多视角输出相互符合；不一定对应真实场景 |

术语依据本文 §3–4 与附录，指标/可辨识性定义为明确标记的通用背景；不把未提供实现细节补写为作者结论。


## SOURCES：依据与范围

- 原论文：Zero-Shot Novel View and Depth Synthesis with Multi-View Geometric Diffusion，arXiv:2501.18804v1，20 页，正文 §1–5、附录 A–E、参考文献；[arXiv](https://arxiv.org/abs/2501.18804v1)、[PDF](https://arxiv.org/pdf/2501.18804v1)、[官方项目](https://mvgd.github.io/)。
- 精读报告 SHA-256：`8ec33019c0c3517da7312e3ae15f0f5a32925807a358bad96205c12b43e16d87`。笔记依赖已冻结报告及同版原始 PDF；原始 Source/HTML 仅辅助，HTML 有外部资源缺失，Source 无独立版本印记，均不替代 PDF。
- **证据分类**：原文事实以章节、图表或公式定位；RIN复杂度、投影和尺度推导、信息论与水下成像例子是背景/补充推导；研究候选是独立可证伪假设，不是作者结论或已证实实验。
- **跨文献分析延期**：当前只抽取本篇潜在连接点，未独立读取其他论文，不宣称建立跨论文技术谱系、排名或完成检索。
- **未核实项**：训练代码、数据清单可复现性、实现中的坐标/尺度退化与公式、五样本融合细节、其他论文完整协议与预训练数据重叠。保留公式及表格内部疑点，不静默修正作者结论。原论文 Figure 5/7/8 展示副本带保真记录；JPEG 不用于定量像素分析。当前没有额外私人研究计划、部署信息或个人数据进入此公开可访问笔记。



