---
title: "GeoNVS: Geometry Grounded Video Diffusion for Novel View Synthesis（完整精读与研究笔记）"
date: 2026-11-27 05:19:57
index_img: "/GeoNVS/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/GeoNVS/)

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2603.14965v2（2026-06-30）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2603.14965) | [PDF](https://arxiv.org/pdf/2603.14965v2) | [Code](https://github.com/MinJunKang/GeoNVS) | [Model](https://huggingface.co/HugMinjun/GeoNVS)

<!-- PAPER-METADATA END -->

# 第一部分：论文精读报告

# GeoNVS 论文完整精读报告

论文：GeoNVS: Geometry Grounded Video Diffusion for Novel View Synthesis

作者：Minjun Kang, Inkyu Shin, Taeyeop Lee, Myungchul Kim, In So Kweon, Kuk-Jin Yoon。

版本：[arXiv:2603.14965v2](https://arxiv.org/abs/2603.14965v2)，2026-06-30。PDF 43 页，正文与附录均可读取。[代码](https://github.com/MinJunKang/GeoNVS)，[权重](https://huggingface.co/HugMinjun/GeoNVS)；官方项目页直接访问未核验成功。

> Markdown-first；PDF 为权威，HTML/Source 为辅助；Figure 为已核验 PDF 区域的展示副本，不是原始独立图像。客户端渲染未验证。原文事实、背景知识、合理推断、独立评价和尚未证实内容分开。

<!-- PTH:BEGIN INIT -->
## 初始化

### 原件与阅读范围

【原文事实】PDF p.1 标题、六位作者和 arXiv:2603.14965v2 标记一致；PDF 共 43 页。正文 p.1–15；附录 p.16–38；参考文献 p.39–43。论文研究静态场景的新视角合成，通过显式三维几何调制视频扩散模型内部特征。上传的 HTML 与 LaTeX Source 可作结构或静态公式辅助；不执行源码中的脚本。

原件 PDF SHA-256：`52630797ac7c1a4ce100b7760b053fa038e19be594ca0e6912bc689ae953cc21`。绑定 Skill v0.6.2，SHA-256：`8bd4338bfe29551ad0e9bab60a4c38591ddb41deed21af12f42d11c04d333c52`。

### 目录、载体冲突及证据规则

【原文事实】§1 Introduction p.2–3；§2 Related Work p.3–4；§3 Method p.4–7；§4 Experiment p.8–14；§5 Conclusions p.15。附录 A 符号、B 基准、C 额外实验、D 网络、E 数据预处理、F 额外定量、G 定性、H 局限。正文 10 Figure、8 Table；附录 19 Figure 区域（含 C.4 未编号图）、6 Table（含 C.3 未编号表）、1 Algorithm。

【来源记录】图 11–13 与表 8 部分题注编号未印出，依据 PDF 区域及 Source 静态计数定位；HTML 表号与 PDF 部分不一致，保留 PDF 编号。Source 无独立版本戳，不据此声称版本已独立验证。29 个区域全部 B2 视觉复核，10 精确通过，19 带记录通过；图11/12 环绕排版边缘有极少文字碎片，不影响主体。

【证据规则】原文事实引用 PDF；通用数学作为背景知识；作者未提供的实现细节标尚未证实；合理推断及独立评价不冒充作者结论。此次不比较其他论文全文，不部署网站。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 标题

### 标题术语与任务

**中文标题**：GeoNVS：面向新视角合成的几何引导视频扩散。

【原文事实】题名 GeoNVS: Geometry Grounded Video Diffusion for Novel View Synthesis（p.1）。Novel View Synthesis（NVS）给定参考图像与对应相机，合成指定新相机的图像。Video Diffusion 指借助视频扩散模型的联合生成能力；输出的序列对应相机视角变化，不等于解决任意动态场景重建。Geometry Grounded 表示在去噪过程中显式利用 3D Gaussian Splatting（3D-GS）的结构，而非只依赖相机参数或二维图像条件。

【背景知识】相机位姿确定从相机坐标到世界坐标的空间关系；即使模型输入正确位姿，生成图像仍可能不遵循真实投影几何，因为学习到的条件映射不自动满足严格三维对应。

### 几何引导与几何保证的区别

【原文事实】§3.1 给出参考 P 张图像及位姿、目标 Q 个位姿的条件，§3.2–3.3 在每个去噪步将参考视角扩散特征提升到高斯、渲染目标视角特征、精炼并融合。高斯提供位置、尺度、旋转和不透明度等结构，而不是以其颜色作为待生成图像的固定真值。

【合理推断】“grounded”是信息约束和归纳偏置，并非严格几何定理。若目标区域未被观测、先验几何错误或特征受遮挡，几何约束可能不可靠；适应性门控允许生成先验补充或纠正。§5 和 Fig.10、28 说明纹理模糊及远离输入时退化仍存在。不可由题名推出完美三维一致性、任意模型通用性或未知物体的真实恢复。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 摘要

### 摘要的论证链

【原文事实】摘要 p.1 的论证可分为五步：① NVS 同时要求三维几何一致与不同视角的视觉连贯；② 相机控制的视频扩散能生成低重叠视角，却会扭曲几何或偏离目标相机；③ GS-Adapter 将输入视角的内部扩散特征提升到三维高斯，通过相机投影渲染新视角特征并自适应融合；④ 不同于输入级渲染图像注入，避开视角相关颜色噪声的直接传播，且可更换几何先验；⑤ 在多种室内外基准和两种扩散骨干上报告改善。

【机制解释】同一个三维高斯接收多个参考像素的可见性加权特征，随后投影到目标相机，使具有相同结构来源的特征具有跨视角联系；目标去噪特征仍保留扩散模型的信息，不要求严格复制高斯的 RGB 纹理。精炼步骤补偿提升与渲染的混合损失，门控根据局部可靠性调节残差（§3.2）。

### 数字主张与证据边界

【原文事实】摘要报告 9 scenes、18 settings，相对 SEVA 和 CameraCtrl 分别 11.3% 与 14.9% 改善，相机平移误差最高约减半，Chamfer Distance（CD）最高约降至七分之一。此处先保留作者报告措辞，不把最高改善当每个样本平均，也不把百分比写成 dB 的绝对增量。实验的口径将在 Table 1–4、6 和对应正文中分别核查。

【独立评价】摘要把几何先验可换和扩散骨干可集成并列，但具体训练设置 §4.2 包括不同骨干分别训练 LoRA 与适配模块，不能据摘要直接判定一个训练好的适配器可零样本迁移到任意扩散骨干。几何先验替换无需再训练的证据与跨骨干集成证据应分开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 贡献

### 三项贡献及继承技术

【原文事实】§1 p.3 列出三项：第一，GS-Adapter 在特征空间连接三维高斯与视频扩散，改善几何一致和相机控制；第二，模块化设计允许组合多种几何模型及扩散骨干；第三，在 9 scenes/18 settings 报告相对于既有生成式 NVS 的效果优势。

【背景知识】3D-GS 的 alpha compositing、跨注意力、残差更新、正弦编码、LoRA 和 DPT 式融合并非本文新发明。§2 p.4 对比静态语义特征蒸馏：传统每场景优化很难跟随每个去噪时刻不断改变的扩散特征分布。

【合理推断】贡献主要在连接方式及其任务适配：利用已有几何结构反复提升动态特征，通过参考视角重渲染的一致性损失训练精炼网络，再自适应地更新目标扩散特征；这是系统层面的机制创新，而不是重新定义高斯渲染。

### 创新的实际适用范围

【原文事实】§4.2 区分 SEVA 和 CameraCtrl 的训练配置；前者支持多个参考视角，后者受骨干限制只能用单参考。§4.6 Table 5 和 Fig.23 研究更换几何先验，Table 6 和 Fig.24 研究 CameraCtrl 集成。

【独立评价】“plug-and-play”最有直接支撑的解释是给定可用三维高斯表示，训练好的对应骨干适配器可接收不同来源的高斯。它并不消除相机尺度对齐、点云拟合、生成骨干接口适配及训练的必要性。对其他骨干不经训练即可迁移的普遍保证尚未证实。第三项是实验贡献，比较范围受数据划分和方法设置约束，不能外推到所有场景。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 结论

### 结论对应实验

【原文事实】§5 p.15 复述 GS-Adapter 的特征级几何调制、相对两个扩散骨干的性能改善与几何指标改善，报告非共视区域提升最高 2.08 dB、经 SfM（Structure from Motion，运动恢复结构）支持无位姿场景、剪枝降低推理开销约 2.2 倍。图8 对比单帧时间，表8 对比无位姿设置；具体数据须保留实验条件。

【合理推断】非共视像素并不一定具有可观测的真实特征。非共视区域改善可来自已观测邻近结构的约束和扩散先验的协同，不能据此断言真实不可见纹理可唯一恢复。

### 局限与未证实主张

【原文事实】作者明确指出：远离输入视角时生成能力降低，可能与特征渲染的不确定性上升有关；提升导致信息损失，细纹理可能模糊，即使结构较好。Fig.10 中整体 PSNR 较高的 Ours 仍有红框内细节模糊，Fig.28 补充薄结构和遮挡失败。

【独立评价】这解释了为何应将几何一致性、图像感知质量、相机轨迹精度分别评估。生成结果经几何模型重建后得到的 CD 和轨迹误差属于间接评估，受评估模型与对齐条件影响。本文未证明在先验完全错误、目标内容不可辨识或动态场景情况下仍能恢复真实结构。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 图表总览

### Figure 功能索引

【原文事实】图1–4 建立任务动机及模型机制；图5–9 支撑特征、感知、效率及几何实验；图10 暴露主文局限。附录图11–16 与 C.4 分解机制、超参数、网络结构和训练数据；图17–28 展开定性比较及失败案例。以下索引采用已核验 PDF 目录，不把每个子图独立计数。

| 图 | PDF 页 | 功能 |
|---|---:|---|
| Figure 1 | 1 | 双基线几何与相机控制示例 |
| Figure 2 | 3 | 三种几何条件注入路径 |
| Figure 3 | 5 | 多尺度网络与 GS-Adapter 信息流 |
| Figure 4 | 6 | 拼接融合与门控残差 |
| Figure 5 | 12 | 逐时间步特征调制与恢复 |
| Figure 6 | 14 | SEVA 定性对照 |
| Figure 7 | 14 | CameraCtrl 单参考定性对照 |
| Figure 8 | 14 | 剪枝与效率权衡 |
| Figure 9 | 15 | 长轨迹重建及相机轨迹 |
| Figure 10 | 15 | 细纹理失败 |
| Figure 11 | 18 | 输入级强度敏感性 |
| Figure 12 | 19 | top-k 提升 |
| Figure 13 | 19 | 特征精炼及潜变量优化 |
| Figure C.4 (unnumbered) | 20 | CFG 强度与生成质量 |
| Figure 14 | 21 | DPT 多尺度聚合 |
| Figure 15 | 21 | RefineNet 残差结构 |
| Figure 16 | 22 | 训练数据构造 |
| Figure 17 | 27 | 小视角定性组一 |
| Figure 18 | 28 | 小视角定性组二 |
| Figure 19 | 29 | 小视角定性组三 |
| Figure 20 | 30 | 小视角定性组四 |
| Figure 21 | 31 | 大视角定性组一 |
| Figure 22 | 32 | 大视角定性组二 |
| Figure 23 | 33 | 几何先验替换消融 |
| Figure 24 | 34 | CameraCtrl 模块消融 |
| Figure 25 | 35 | 长轨迹几何组一 |
| Figure 26 | 36 | 长轨迹几何组二 |
| Figure 27 | 37 | 输入级与特征级几何对照 |
| Figure 28 | 38 | 薄结构及遮挡失败 |

### Table 与 Algorithm 功能索引

【原文事实】表1–3、11–13 分别提供三类测试设置的 PSNR 及 SSIM/LPIPS；表4 为几何指标；表5–7 消融；表8 无位姿；表9 比较先验匹配；C.3 是特征精炼定量；表10 为符号。Algorithm 1 位于 p.23，给出训练数据视角选择。

| 表 | PDF 页 | 功能 |
|---|---:|---|
| Table 1 | 9 | 小视角 PSNR 全组 |
| Table 2 | 10 | 大视角 PSNR |
| Table 3 | 10 | 长轨迹 PSNR |
| Table 4 | 10 | 几何一致性指标 |
| Table 5 | 12 | SEVA 先验替换 |
| Table 6 | 12 | CameraCtrl 先验替换 |
| Table 7 | 13 | 组件消融 |
| Table 8 | 14 | 无位姿设置 |
| Table 9 | 18 | 数据集先验选择 |
| Table C.3 (unnumbered) | 19 | 精炼 PSNR 与 SSIM |
| Table 10 | 24 | 符号表 |
| Table 11 | 25 | 小视角 SSIM/LPIPS |
| Table 12 | 26 | 大视角 SSIM/LPIPS |
| Table 13 | 26 | 长轨迹 SSIM/LPIPS |

【边界】本索引只建立完整覆盖计划，不将目录存在等同于已精读。未编号对象独立列出，避免遗漏。后续逐图、逐表均需结合原始 PDF 及正文解释。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
## Figure 1 双基线几何与相机控制示例

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 1 We demonstrate GeoNVS, integrated into SEVA [90] and CameraCtrl [24], consistently enhancing geometric fidelity and camera controllability, while resolving photometric inconsistencies (right-top) across challenging large-scale indoor and outdoor scenes. Our approach consistently outperforms existing methods. Project page.

【原文证据】PDF p.1，Figure 1。题注译意：在 SEVA 和 CameraCtrl 中集成 GeoNVS，在室内外复杂大场景提升几何保真和相机控制，并缓解光度不一致。

![Figure 1](GeoNVS/Figures/Figure_01.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】左侧以输入及目标轨迹为条件，比较生成图像和三维重建；右上同场景雕像的红框突出光度及结构不一致，右下展示 CameraCtrl 条件下的对照。水果和室内场景示例将生成图像和轨迹重建共同呈现。

### 机制、数值与证据边界

【原文事实】Fig.1 是论文整体预览，三类目标分别对应感知、几何、控制，而非三个相互独立的方法。SEVA 与 CameraCtrl 是扩散骨干，不是本文另建的几何模型。

【合理推断】视角变化时，相同物体的几何位置应沿真实相机投影变化；若生成模型在相机旋转或平移时改变物体本身，则会同时损伤三维重建与相机轨迹估计。光度一致与几何一致并不相等：不同光照反射条件可能改变颜色却不改变结构。

【证据边界】示意案例不能代替全测试集统计，具体指标由表1–4、6 支撑；重建图属于生成结果的间接几何测量，不是从真值三维直接证明结构完全正确。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
## Figure 2 三种几何条件注入路径

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 2 Geometry-guided generative NVS. (a) Pure diffusion model produces view-inconsistent results. Given sparse input-view images, both (b) and (c) reconstruct 3D Gaussians from input views using a geometry prior. (b) Previous methods inject rasterized novel-view images from 3D-GS as input, causing artifacts from noisy rasterized colors. (c) Our method modulates internal diffusion features via a GS-Adapter conditioned on 3D-GS, achieving superior geometry consistency and visual quality.

【原文证据】PDF p.3，Figure 2。题注译意：(a) 纯扩散模型生成跨视角不一致结果；(b) 将 3D-GS 渲染的新视角 RGB 作为输入，颜色噪声带来伪影；(c) GS-Adapter 利用高斯条件调制扩散内部特征。

![Figure 2](GeoNVS/Figures/Figure_02.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】三行流程采用参考图像、目标潜变量与视频扩散的同类框架。上行没有显式三维中介；中行由参考图像构建高斯、渲染彩色图作为额外输入；下行通过 lifting 和 fusion 在内部构建 Feature 3D-GS。右端书架示例对比结构畸变与恢复。

### 机制、数值与证据边界

【机制】(a) 只由学习的相机条件与生成先验决定目标潜变量，目标像素没有强制关联到参考三维点；(b) 利用几何先验渲染图像，其颜色与高斯位置/可见性被共同编码，错误纹理会成为生成条件；(c) 固定结构权重而用当前去噪时刻的参考扩散特征替代 RGB 属性，目标相机投影形成几何条件，再由目标特征查询并融合。

这一区别并非“无图像输入”，参考图像仍是生成模型和几何模型的输入；区别在几何信息进入生成器的层级和载体。§4.2 的输入级实验将渲染潜变量用于噪声初始化，强度 s 控制自由度；C.1 扫描 s，表4 与图27 对照几何误差。

【独立评价】仅凭图中纹理失败不能证明所有输入级方法必然更差；实验支持本文具体实现与比较设定。特征融合也不能消除错误先验，图4、28 证明仍需可靠性判断。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
## Figure 3 多尺度网络与 GS-Adapter 信息流

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 3 GeoNVS architecture. (a) Overview of the integration with a video diffusion model. (b) The GS-Adapter pipeline for feature lifting, refinement, and fusion. All learnable modules (图中可训练模块图标，PDF文本提取未保留图标) are trained with LoRA [26]. During training, a consistency loss Lfeat is applied to preserve geometric detail lost during feature lifting. Please refer to the supplementary material for details of the multi-scale fusion module and RefineNet.

【原文证据】PDF p.5，Figure 3。题注译意：(a) GS-Adapter 与视频扩散模型的集成；(b) 特征提升、精炼、融合三步。训练用 LoRA，并以特征一致性损失保存提升中丢失的几何细节；多尺度融合及 RefineNet 见附录。

![Figure 3](GeoNVS/Figures/Figure_03.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】(a) U-Net 编码器四层经 DPT 式融合与通道压缩得到 F；GS-Adapter 返回修正目标特征，再卷积/反卷积变换到相应解码器尺度，通过跳连加入。(b) 参考和目标特征分开：参考特征经 lifting 赋给三维高斯，分别渲染 Gref/Gtar；共享 RefineNet 接收 GS-PE，输出精炼特征；参考输出与原始 Fref 计算余弦损失，目标输出经 Fusion 与 Ftar 合并。

### 机制、数值与证据边界

【机制】设 P 个参考、Q 个目标，VAE 缩放 n、特征通道 C。统一张量 F 具有 (P+Q)×h/n×w/n×C 形状。高斯几何来自外部模型，不直接产生目标 RGB 输出。每步 t，Eq.(4) 将参考 Fref 映射到每个高斯的 C 维属性；Eq.(3) 从参考/目标相机重渲染。参考分支的目的不是生成目标，而是提供可对齐的训练监督，保证通过同一几何路径后特征方向尽量接近原值。共享网络再用于目标分支。

Eq.(5) 给出基于最大贡献高斯位置和权重的编码；Eq.(6) 是参考特征一致性；Eq.(8)–(9) 是目标融合。Eq.(10) 将其和 latent loss 合并。去噪时每步重新提升，因为 Fref(t) 变化；几何结构本身无需每步重新估计。

【边界】网络结构并非只在扩散输入加一张几何图。论文说明训练 LoRA 注入注意力模块，同时训练适配器和多尺度模块；图中颜色标记不应解释成所有新模块仅由低秩矩阵替代。具体尺度见图14、15。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
## Figure 4 拼接融合与门控残差

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 4 Feature fusion module of GS-Adapter. Two fusion approaches are proposed to integrate the diffusion feature Ftar and the geometry-aware feature ˆGtar, producing the updated novel-view feature ˆFtar. We adopt adaptive fusion as it remains effective even when either the geometry prior or the generative model fails.

【原文证据】PDF p.6，Figure 4。题注译意：比较 naïve fusion 与 adaptive fusion，二者均融合目标扩散特征 Ftar 和精炼几何特征 Ĝtar；作者选择在任一先验失败时更有效的自适应方案。

![Figure 4](GeoNVS/Figures/Figure_04.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】左上拼接经 LayerNorm 和 MLP（2C→C）；左下跨注意力后拼接，Gate MLP（2C→1）预测 W，并残差更新。中间玩具车与建筑两组比较几何先验、生成基线、两种融合；右侧反光场景 Garden Spheres、Toy Car、Sedan 给出不确定性与门控权重的可视化。

### 机制、数值与证据边界

【机制】naïve：F̂tar=MLPs(Ftar⊕Ĝtar)。输出完全经学习映射，结构上没有原目标特征的恒等保留路径。adaptive：Q 来自 Ftar，K/V 来自 Ĝtar，得到 FA；对 Ftar⊕FA 做 MLP 后 Tanh，得到 W∈[-1,1]；F̂=F+W·FA。单个像素权重广播到 C 通道。W≈0 保持原特征，正负符号可增加或抵消注意力残差。不是 convex mixture，也不是概率。

【原文事实】§3.2 对三反光场景报告绝对门控值 |W| 与 3D-GS uncertainty 的 Pearson 相关约 −0.30，随去噪进展增强。用绝对值而非有符号值是因为研究残差幅度与不确定性关系。

【独立评价】这是中等偏弱负相关的实验支持，并非严格置信校准；W 的 MLP 输入不显式包含独立不确定性标签，门控不是直接使用不确定性图。图中的反例展示局部修正能力但不能保证错误先验永不传播。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
## Figure 5 逐时间步特征调制与恢复

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 5 Feature modulation by GS-Adapter. We visualize intermediate diffusion features during the denoising process. GS-Adapter consists of three stages: (1) lifting reference-view features $F^t_{ref}$ into 3D Gaussians, (2) refining the novel-view features $G_{tar}$ into $\hat G_{tar}$, and (3) fusing $\hat G_{tar}$ with $F^t_{tar}$ to generate geometry-corrected outputs.

【原文证据】PDF p.12，Figure 5。题注译意：去噪过程的中间扩散特征可视化。三步为参考特征提升、目标特征精炼、与目标扩散特征融合。

![Figure 5](GeoNVS/Figures/Figure_05.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】以参考图像与几何先验为起点，早期与晚期去噪分支展示彩色特征图；中间可对照渲染特征 Gtar 和 RefineNet 输出 Ĝtar，末端展示融合和最终生成。颜色先验的 RGB 渲染可见粗糙和伪影。

### 机制、数值与证据边界

【机制】不同 t 下扩散内部特征并非固定纹理，因此同一高斯位置上的特征值随 t 变化。提升 Eq.(4) 将多个像素加权平均，并 L2 归一化；再渲染 Eq.(3) 会混合多个高斯，可能损失高频和幅度信息；RefineNet 用 GS-PE 提供三维位置及权重恢复线索。目标融合保留生成特征供填补不可见区域。

【证据边界】可视化特征通常通过降维或颜色映射展示，图中颜色不等于特征每维或最终 RGB。图5 说明中间结果的结构相关性，不能独立证明全部特征质量；定量依赖表7及 C.3 的重渲染/精炼实验。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
## Figure 6 SEVA 定性对照

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 6 Qualitative results of GeoNVS with SEVA [90].

【原文证据】PDF p.14，Figure 6。题注译意：与 SEVA 集成的 GeoNVS 的定性结果。

![Figure 6](GeoNVS/Figures/Figure_06.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】两组参考与目标视角对照，列出几何/生成/组合方法及 GT。上组 VGGT 18.69、GenFusion 12.21、Difix3D+ 17.66、SEVA 13.28、Ours 20.08；下组 Pi3 15.90、MVSplat360 14.70、Difix3D+ 13.37、SEVA 7.89、Ours 17.67（dB）。

### 机制、数值与证据边界

【原文事实】示例中本文相对 SEVA 分别高 6.80 与 9.78 dB，且相对纯几何模型高 1.39 与 1.77 dB。目标是既维持几何对齐又补全视觉内容。纯几何模型在缺少纹理或遮挡时可能出现渲染空洞，生成模型则可能改变布局。

【背景知识】PSNR 依赖像素均方误差，数值越高表示与 GT 像素越近，不保证人眼细纹理更锐利。

【证据边界】图中数值是所选示例而不是全数据集均值；图6 不能单独验证统计显著性，需结合表1–3。模型先验及比较设置以 §4.2、附录B为准。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
## Figure 7 CameraCtrl 单参考定性对照

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 7 Qualitative results of GeoNVS with CameraCtrl [24]

【原文证据】PDF p.14，Figure 7。题注译意：与 CameraCtrl 集成的 GeoNVS 定性结果。

![Figure 7](GeoNVS/Figures/Figure_07.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】单张参考图下不同相机目标输出；DepthSplat 12.90、ViewCrafter 17.96、Difix3D+ 12.82、CameraCtrl 11.58、Ours 20.20 dB；GT 作为目标对照。

### 机制、数值与证据边界

【原文事实】该示例中 Ours 相对 CameraCtrl +8.62 dB、相对 ViewCrafter +2.24 dB。单参考限制来自 CameraCtrl 骨干（§4.2）。参考图像的深度与遮挡无法由多视角对应约束，几何先验预测的尺度和结构尤其重要。

【证据边界】图7 验证适配另一骨干的可行性而非证明同一组 SEVA 权重免训练直接迁移。训练分别使用不同分辨率、学习率和 LoRA rank，表6 给出整体测试。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
## Figure 8 剪枝与效率权衡

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 8 Runtime and memory analysis under Gaussian pruning. Inference time, peak GPU memory, and PSNR of GeoNVS versus Gaussian pruning rate across 10 DL3DV [46] scenes. Voxel-based pruning [32] yields a 2.22× speedup at 96.2% pruning. Peak memory drops to 12.93 GB, close to SEVA [90], while PSNR is preserved.

【原文证据】PDF p.14，Figure 8。题注译意：10 个 DL3DV 场景中随高斯剪枝率变化的推理时间、峰值 GPU 内存和 PSNR；96.2% 剪枝带来约 2.22× 加速，峰值 12.93 GB，PSNR 保持。

![Figure 8](GeoNVS/Figures/Figure_08.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】左图时间：SEVA 1.34 s/frame；GeoNVS 4.19 降到 1.89。右图峰值：SEVA 12.81 GB；GeoNVS 18.03 降到 12.93 GB；图中 PSNR SEVA 15.844、GeoNVS 18.108 基本不变（图标四舍五入）。

### 机制、数值与证据边界

【机制】高斯数量越大，反复特征提升和 rasterization 的候选关联、存储、计算通常更多。体素剪枝减少冗余几何粒子而保留主要空间覆盖，使适配器开销下降。4.19/1.89≈2.22，峰值减少 5.10 GB；剪枝后的模型仍比 SEVA 慢 0.55 s/frame，内存接近但非完全相等。

【边界】这是该 10 场景、硬件和所测剪枝规则下的结果；不能据此对所有高斯表示剪去 96.2%，尤其薄结构可能依赖少量高斯（图28）。论文并非宣称无额外计算。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
## Figure 9 长轨迹重建及相机轨迹

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 9 Qualitative results of GeoNVS. 3D reconstruction and camera trajectory estimated from the generated video, showing the improved geometric fidelity of GeoNVS.

【原文证据】PDF p.15，Figure 9。题注译意：对生成视频进行三维重建及相机轨迹估计，展示改善后的几何保真。

![Figure 9](GeoNVS/Figures/Figure_09.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】两个输入设置 P=3 和 P=9，显示参考、生成结果的三维结构及估计轨迹，与从 GT 新视角重建的 Oracle 对照；示例 CD 0.415→0.017、89.29→28.55。

### 机制、数值与证据边界

【原文事实】CD（Chamfer Distance）用于点云间距离；图中越小越好。上述两案例约 24.4 倍和 3.13 倍改善，均是选中案例而非总体指标。论文几何评估还包含平移和旋转误差（§4.4）。

【合理推断】若视频视角间结构漂移，重建器会估计不同位置的点或错误相机；更低 CD 与更接近 Oracle 的路径共同支持几何改善。

【边界】Oracle 是从 GT 目标视角经重建方法估计所得，不是完整真实三维场景；CD 同时受点云采样、尺度、重建器稳定性影响。附录图25存在个别 Ours CD 更高的案例，不能将改善当作每例保证。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN FIG-10 -->
## Figure 10 细纹理失败

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 10 Limitation: blurry textures. Our method preserves structure well but occasionally produces blurry textures in fine-detail regions (red boxes).

【原文证据】PDF p.15，Figure 10。题注译意：本文较好维持结构，但精细细节区域偶尔产生模糊纹理。

![Figure 10](GeoNVS/Figures/Figure_10.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】两组 SEVA/Ours/GT 对照，红框标示纹理。Ours 16.62 vs SEVA 15.65，以及 Ours 21.33 vs SEVA 19.80 dB，整体 PSNR 更高但局部纹理仍模糊。

### 机制、数值与证据边界

【机制】每高斯融合来自不同像素的特征再渲染，可能使高频细节平均化；RefineNet 只能学习补偿，无法保证信息无损。结构对齐能减少大尺度像素错位，从而提高整体 PSNR，即使局部边缘或纹理更软。

【原文事实】作者将模糊归因于 lifting 信息损失，并在 §5 说明远离输入视角时特征渲染不确定性升高。

【独立评价】这是几何先验与生成锐度之间的技术边界，不应因平均 PSNR 优势删除负面案例。遮挡区域无观测的纹理只能依赖生成先验，合理外观不等于真实。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-10 -->

<!-- PTH:BEGIN FIG-11 -->
## Figure 11 输入级强度敏感性

### 原图、题注与视觉布局

**Caption 原文**：PDF 未印出独立题注；对应静态源编号或正文引用已记录。不补造题注。

【原文证据】PDF p.18，Figure 11。§C.1 环绕图（PDF 未印出编号题注，正文引用 Fig.11）；分析 input-level inpaint strength。

![Figure 11](GeoNVS/Figures/Figure_11.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】左 DL3DV-10、右 MipNeRF-360；横轴 s=0.2 至1.0，纵轴 PSNR；Ours 固定水平线 18.11/16.57 dB，SEVA 基线在 s=1.0 附近标示。

### 机制、数值与证据边界

【原文事实】§4.2 将输入级渲染的目标潜变量 ztar 与噪声 x 混合初始化：xtar←x·σt0+ztar，t0=floor(Ts)。低 s 对渲染图约束强，高 s 更接近纯生成。在 DL3DV 输入级较小 s 表现更好，作者称随 s 增大单调下降；图中末端存在微小回升，不宜解读成严格数学单调。Mip360 随 s 波动，两个数据集均低于 GeoNVS。

【独立评价】本文的水平线代表不扫描该输入注入参数，而非在全部超参数上不敏感。对输入级最佳 s 的比较支持此实现的稳定性，不能代表其他训练过的图像条件模型。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-11 -->

<!-- PTH:BEGIN FIG-12 -->
## Figure 12 top-k 提升

### 原图、题注与视觉布局

**Caption 原文**：PDF 未印出独立题注；对应静态源编号或正文引用已记录。不补造题注。

【原文证据】PDF p.19，Figure 12。§C.2 环绕图（编号由静态源计数核验）；top-k Gaussian feature uplifting 对比。

![Figure 12](GeoNVS/Figures/Figure_12.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】两组曲线横轴 k=1,16,32,64,128,255，纵轴 PSNR。DL3DV-10 大致 18.068→18.11，Mip360 大致 16.472→16.57，存在平缓区间及轻微波动。

### 机制、数值与证据边界

【原文事实】k=1 仅保留每条射线上贡献最大的高斯，相当于将像素特征硬分配到一个代表点；k=255 为论文测试的完整软分配设置。Eq.(4) 权重来自 alpha compositing，每个高斯累积其所贡献的参考像素的特征。

【机制】一个像素可能由多个部分透明或重叠高斯贡献，单一最大权重丢弃其他贡献者；软分配将相同特征按相对可见性映射到多个高斯，重渲染时获得更平滑的空间场。但更平滑并不保证高频细节保存。

【独立评价】结果方向支持软分配，但绝对增益较小（约 0.04、0.10 dB），图形非严格逐 k 单调；没有显著性统计，不能将软分配等同于全部 3D-GS 优势。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-12 -->

<!-- PTH:BEGIN FIG-13 -->
## Figure 13 特征精炼及潜变量优化

### 原图、题注与视觉布局

**Caption 原文**：PDF 未印出独立题注；对应静态源编号或正文引用已记录。不补造题注。

【原文证据】PDF p.19，Figure 13。§C.3 未印出编号题注，Source 计数为 Fig.13；验证潜域特征渲染与精炼。

![Figure 13](GeoNVS/Figures/Figure_13.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】(a) VAE 编码参考特征，经 VGGT 高斯提升、目标渲染、VAE 解码；(b) 添加参考/目标共享 RefineNet 与参考余弦监督；下部表格给出 3/6/9 张输入 PSNR/SSIM。

### 机制、数值与证据边界

【原文事实】此处不是完整视频扩散模型的端到端实验，而是在 540 个 DL3DV 测试场景上对 VAE 编码的 latent 进行每场景优化比较。无精炼与有精炼的数值分别为 3 views 16.35/0.56→16.59/0.57；6 views 17.04/0.58→18.07/0.60；9 views 17.38/0.59→18.75/0.62。对应 PSNR 增量 0.24/1.03/1.37 dB。

【机制】RGB alpha 混合的线性可加性不必然适用于全部潜特征维度，多个高斯贡献可能造成特征偏移。RefineNet 通过参考视角的原始 latent 与重渲染 latent 的方向一致性补偿。

【证据边界】此受控实验支持“精炼恢复特征保真”的动机，并不证明所有 latent 线性可加，或参考视角监督能够唯一恢复遮挡区域。独立表 C.3 将保留全部数值。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-13 -->

<!-- PTH:BEGIN FIG-C4 -->
## Figure C.4 (unnumbered) CFG 强度与生成质量

### 原图、题注与视觉布局

**Caption 原文**：无独立编号题注，以 §C.4 段落与实际图标签定位。

【原文证据】PDF p.20，Figure C.4 (unnumbered)。§C.4 无编号 CFG 图；检验 classifier-free guidance scale 的敏感性。

![Figure C.4 (unnumbered)](GeoNVS/Figures/Figure_C4.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】顶部图像对照 CFG=1.0、1.5、2.0；下方 10 个 DL3DV-10K 场景曲线，SEVA 14.74→15.84（+1.10），Ours 17.69→18.16（+0.47）。

### 机制、数值与证据边界

【背景知识】CFG（classifier-free guidance）通过条件与无条件预测的差值调节条件强度，强度变化可能引起颜色和结构变化。此处图像观察显示 SEVA 在不同强度下出现伪影，GeoNVS 相对稳定。

【原文事实】作者解释显式几何条件稳定生成。图中 Ours 在所测范围 PSNR 更高且变化幅度更小。

【边界】仅 10 个场景及有限 CFG 值，不能推断任意 guidance scale 均稳定。CFG 不等同于 §C.1 的 inpaint strength，也不等同于 Eq.(8) 像素门控。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-C4 -->

<!-- PTH:BEGIN FIG-14 -->
## Figure 14 DPT 多尺度聚合

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 14 Multi-scale fusion module of GeoNVS.

【原文证据】PDF p.21，Figure 14。题注译意：GeoNVS 的多尺度融合模块。

![Figure 14](GeoNVS/Figures/Figure_14.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】四层 Encoder 输出三个不同空间尺度；标注 h/n,w/n,320；h/(2n),w/(2n),640；h/(4n),w/(4n),1280。Projection Convs 将通道统一 320；Upsample Layer 把空间尺寸统一至 h/n,w/n 并变成 64 通道；右侧 DPT Fusion 层级融合。

### 机制、数值与证据边界

【机制】对于 (P+Q) 个视角，先让不同 encoder 层的张量具备可融合的通道及空间分辨率，层级融合得 F∈R^((P+Q)×h/n×w/n×C)，图示 C=64。深层特征具有较大感受野，浅层具有更多局部结构；统一 F 使同一 GS-Adapter 不必在每个尺度重复提升。§3.3 再将修正目标特征降采样至各 decoder 尺度，通过 skip connections 注入。

【尚未证实】图14未给出每个卷积 kernel、stride、非线性及确切视角批维实现；不能据图重写完整网络代码。

<!-- PTH:END FIG-14 -->

<!-- PTH:BEGIN FIG-15 -->
## Figure 15 RefineNet 残差结构

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 15 RefineNet module (R) of GS-Adapter.

【原文证据】PDF p.21，Figure 15。题注译意：GS-Adapter 的 RefineNet 模块 R。

![Figure 15](GeoNVS/Figures/Figure_15.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】左侧渲染 Gref/Gtar 加 GS-PE，经过线性层及 ResNet Conv blocks 输出精炼 Ĝref/Ĝtar；右侧单个 block 示意 Conv2D C→2C、2C→2C、2C→C，GELU 与恒等残差。

### 机制、数值与证据边界

【机制】最大贡献高斯的归一化 xyz 与 w* 形成 GS-PE（Eq.5），增强渲染特征的三维定位信号。ResNet 保留输入的直接路径、学习残差校正；共享参考与目标参数，让参考可观测监督迁移至目标渲染特征。Eq.(6) 对参考特征方向施加约束，不直接在目标输入真实 feature 监督。

【边界】图示未声明每个残差分支可精确逆转 lifting；通道可扩大再压缩不是无损编码。C.3 给出定量增益而非理论可逆性。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-15 -->

<!-- PTH:BEGIN FIG-16 -->
## Figure 16 训练数据构造

### 原图、题注与视觉布局

**Caption 原文**（按 PDF 转写，换行/数学记号规范化）：Fig. 16 Pipeline to construct training dataset of GeoNVS.

【原文证据】PDF p.22，Figure 16。题注译意：构造 GeoNVS 训练数据集的流程。

![Figure 16](GeoNVS/Figures/Figure_16.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】由约300帧 multi-view video，经位姿距离图选择 anchor、K-Means 邻近采样、FOV IoU<40% 过滤，构造21张视图分组并形成150K clips。VGGT 在训练中包含目标图用于点云，InstantSplat 拟合排除目标图。

### 机制、数值与证据边界

【原文事实】§E / Algorithm 1：图边由相对平移及旋转距离确定，阈值保证至少21邻居；按图节点度采样3个独立 anchor；K-Means 选择参考 P∈{1,3,6,9,12}；其他候选按最小 CLIP cosine distance 分 easy（底部60%）与 hard，分别采样后用 frustum IoU≥0.4 过滤。总视图21。

【机制及边界】训练用目标视图协助 VGGT 点云估计，拟合高斯时只用参考 RGB；推理时不能使用目标真实图。必须区分几何初始化的信息与颜色优化的信息。训练先验可能比测试由稀疏参考预测的先验更完整，存在训练/测试分布差异；本文跨模型实验一定程度验证鲁棒性，但不证明不存在该差异。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-16 -->

<!-- PTH:BEGIN FIG-17 -->
## Figure 17 小视角定性组一

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 17 Qualitative results of small-viewpoint NVS.

【原文证据】PDF p.27，Figure 17。题注译意：small-viewpoint，单参考 NVS 定性结果；§G 将其对应到表1。

![Figure 17](GeoNVS/Figures/Figure_17.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】上下两组各四行；水果、走廊、房屋、长椅、雕像、客厅、办公室、建筑等场景。

### 机制、数值与证据边界

【原文事实】每行是一个参考条件与目标视图的定性对照，目标真值用于测试比较；不是训练样本演示。在单图深度歧义下比较几何渲染、纯生成与组合方法；观察结构、纹理和相机目标，而非要求所有遮挡真实恢复。

【独立评价】观察重点是边界位置、主要对象大小及相对布局是否随相机一致变化，另检查纹理和色彩是否接近 GT。所示图像为作者选择样例，不能据此断言全部方法或全部场景的排序；量化结果必须与对应表及数据划分结合。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-17 -->

<!-- PTH:BEGIN FIG-18 -->
## Figure 18 小视角定性组二

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 18 Qualitative results of small-viewpoint NVS.

【原文证据】PDF p.28，Figure 18。题注译意：small-viewpoint，1–3 参考 NVS 定性结果；§G 将其对应到表1。

![Figure 18](GeoNVS/Figures/Figure_18.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】粉色室内、外墙、图案板、木地板、碗、蓝色陶瓷、金属与水果等场景。

### 机制、数值与证据边界

【原文事实】每行是一个参考条件与目标视图的定性对照，目标真值用于测试比较；不是训练样本演示。平面边界和重复纹理便于发现结构漂移；反光物体的颜色变化又提示光度误差不总等价于几何误差。

【独立评价】观察重点是边界位置、主要对象大小及相对布局是否随相机一致变化，另检查纹理和色彩是否接近 GT。所示图像为作者选择样例，不能据此断言全部方法或全部场景的排序；量化结果必须与对应表及数据划分结合。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-18 -->

<!-- PTH:BEGIN FIG-19 -->
## Figure 19 小视角定性组三

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 19 Qualitative results of small-viewpoint NVS.

【原文证据】PDF p.29，Figure 19。题注译意：small-viewpoint，2/3/6 参考 NVS 定性结果；§G 将其对应到表1。

![Figure 19](GeoNVS/Figures/Figure_19.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】多张参考图和相机示意对应室内外目标图，同行对照几何、生成、组合方法与 GT。

### 机制、数值与证据边界

【原文事实】每行是一个参考条件与目标视图的定性对照，目标真值用于测试比较；不是训练样本演示。多参考提供更多共视约束，提升加权融合可能更可靠；不同场景及 P 混合展示不能单独分离 P 的因果效应。

【独立评价】观察重点是边界位置、主要对象大小及相对布局是否随相机一致变化，另检查纹理和色彩是否接近 GT。所示图像为作者选择样例，不能据此断言全部方法或全部场景的排序；量化结果必须与对应表及数据划分结合。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-19 -->

<!-- PTH:BEGIN FIG-20 -->
## Figure 20 小视角定性组四

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 20 Qualitative results of small-viewpoint NVS.

【原文证据】PDF p.30，Figure 20。题注译意：small-viewpoint，6/9 参考 NVS 定性结果；§G 将其对应到表1。

![Figure 20](GeoNVS/Figures/Figure_20.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】花、路锥、洗手间、建筑、柱子和室内等多视角实例。

### 机制、数值与证据边界

【原文事实】每行是一个参考条件与目标视图的定性对照，目标真值用于测试比较；不是训练样本演示。几何可见性和颜色恢复的失效具有空间差异；薄结构及重复纹理仍可能出错，不应只关注整体布局。

【独立评价】观察重点是边界位置、主要对象大小及相对布局是否随相机一致变化，另检查纹理和色彩是否接近 GT。所示图像为作者选择样例，不能据此断言全部方法或全部场景的排序；量化结果必须与对应表及数据划分结合。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-20 -->

<!-- PTH:BEGIN FIG-21 -->
## Figure 21 大视角定性组一

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 21 Qualitative results of large-viewpoint NVS.

【原文证据】PDF p.31，Figure 21。题注译意：large-viewpoint，单参考 NVS 定性结果；§G 将其对应到表2a。

![Figure 21](GeoNVS/Figures/Figure_21.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】黄色球、房间、店铺、花束、李子等物体与场景，同行对应较大视角变化。

### 机制、数值与证据边界

【原文事实】每行是一个参考条件与目标视图的定性对照，目标真值用于测试比较；不是训练样本演示。大视角下新暴露区域增多，纯几何容易缺覆盖，纯生成容易改变结构；显式特征约束与生成补全有互补作用。

【独立评价】观察重点是边界位置、主要对象大小及相对布局是否随相机一致变化，另检查纹理和色彩是否接近 GT。所示图像为作者选择样例，不能据此断言全部方法或全部场景的排序；量化结果必须与对应表及数据划分结合。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-21 -->

<!-- PTH:BEGIN FIG-22 -->
## Figure 22 大视角定性组二

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 22 Qualitative results of large-viewpoint NVS.

【原文证据】PDF p.32，Figure 22。题注译意：large-viewpoint，1–3 参考 NVS 定性结果；§G 将其对应到表2a。

![Figure 22](GeoNVS/Figures/Figure_22.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】城市建筑、绿植、商品店和工业物体；相机示意与参考图指示跨视角变化。

### 机制、数值与证据边界

【原文事实】每行是一个参考条件与目标视图的定性对照，目标真值用于测试比较；不是训练样本演示。较大角度导致共视下降与遮挡增加；参考观测提供的结构锚点并不能唯一确定目标不可见部分。

【独立评价】观察重点是边界位置、主要对象大小及相对布局是否随相机一致变化，另检查纹理和色彩是否接近 GT。所示图像为作者选择样例，不能据此断言全部方法或全部场景的排序；量化结果必须与对应表及数据划分结合。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-22 -->

<!-- PTH:BEGIN FIG-23 -->
## Figure 23 几何先验替换消融

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 23 Ablation results with various geometry prior in Tab. 5.

【原文证据】PDF p.33，Figure 23。题注译意：Table 5 的多种几何先验消融，分别比较 naïve 与 adaptive fusion。

![Figure 23](GeoNVS/Figures/Figure_23.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】两大组对照不同 geometry prior（DepthSplat、Pi3、VGGT、MVSplat 等）、目标图与生成结果。图示先验不佳时 naïve 输出结构/颜色异常，adaptive 通常更稳健。

### 机制、数值与证据边界

【机制】拼接 MLP 没有直接保留扩散目标特征的恒等路径，可能把错误几何特征无条件传播；adaptive 以目标扩散特征查询几何特征，通过 W 调节残差，低可靠区域可减少注入。

【原文事实】本图对应 Table 5 的三参考 zero-shot prior substitution，说明不必因替换几何模型重训对应适配器；并非去除几何估计过程。

【边界】每种先验模型的输入位姿及尺度条件不同，比较要求对齐并统一为高斯。个别样例仍可能变差，量化排序和例外见 Table 5。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-23 -->

<!-- PTH:BEGIN FIG-24 -->
## Figure 24 CameraCtrl 模块消融

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 24 Ablation results of GeoNVS with CameraCtrl in Tab. 6.

【原文证据】PDF p.34，Figure 24。题注译意：Table 6 对应的 CameraCtrl 消融。

![Figure 24](GeoNVS/Figures/Figure_24.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】两组各四行单输入图、相机条件和多种几何先验的结果；熊、网球、水果、苹果及玩具等示例。

### 机制、数值与证据边界

【原文事实】CameraCtrl 版本只支持单参考；先验可由不同几何模型产生，结构较准的先验可以改善相机条件下生成。

【机制】与 SEVA 共享思想，但输入视图和特征网络接口有差异，论文分别微调适配器及 LoRA。单参考使跨视角提升缺少多图平均的优势，但渲染仍提供显式三维对应。

【边界】本图支持第二骨干集成及先验替换可行性，不支持同一 checkpoint 任意骨干零样本迁移。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-24 -->

<!-- PTH:BEGIN FIG-25 -->
## Figure 25 长轨迹几何组一

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 25 Qualitative results of trajectory NVS.

【原文证据】PDF p.35，Figure 25。题注译意：轨迹 NVS 定性结果，通过生成视频重建三维及估计轨迹。

![Figure 25](GeoNVS/Figures/Figure_25.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】两组对比 SEVA/Ours/Oracle 重建及轨迹。图中 CD 分别 4.71→0.286、0.043→0.033、1.56→0.078、0.102→0.056、5.46→0.472，以及 0.167→0.329。

### 机制、数值与证据边界

【原文事实】前五组 CD 减少，最后一组反而增大。此处保留反例，说明作者整体结论不能解释成每例优势。CD 是点云几何距离，轨迹可视化用于补充估计相机是否偏离目标。

【独立评价】Oracle 从 GT 目标视角重建，不是真实物理点云。单个 CD 值可能受估计尺度、点云密度及重建失败影响；更低 CD 与相机轨迹更接近共同提供间接证据。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-25 -->

<!-- PTH:BEGIN FIG-26 -->
## Figure 26 长轨迹几何组二

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 26 Qualitative results of trajectory NVS.

【原文证据】PDF p.36，Figure 26。题注译意：附加轨迹 NVS 定性结果。

![Figure 26](GeoNVS/Figures/Figure_26.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】六组 CD 对照 0.03→0.02、0.054→0.029、3.25→1.46、2.01→1.71、0.366→0.216、0.315→0.264。

### 机制、数值与证据边界

【原文事实】本图所示六组均降低，但改善幅度不一；长轨迹中视角变化积累可能使纯生成的结构漂移更加明显。

【独立评价】不能对不同场景的 CD 直接平均或比较倍数而不核对论文的尺度对齐和统计口径。图25 的反例说明结果应与 Table 3b 等聚合数据共同理解。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-26 -->

<!-- PTH:BEGIN FIG-27 -->
## Figure 27 输入级与特征级几何对照

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 27 Qualitative results compared to input-level methods in Tab. 4.

【原文证据】PDF p.37，Figure 27。题注译意：与 Table 4 的 input-level methods 几何结果对照。

![Figure 27](GeoNVS/Figures/Figure_27.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】(a) 输入级对照三例 CD 0.901→0.029、2.04→0.150、8.67→0.070；(b) Difix3D+ 对照 0.222→0.096、0.917→0.096、167.85→0.018。

### 机制、数值与证据边界

【原文事实】作者展示由生成视角估计的几何与轨迹，相较输入级注入在所选样例具有较明显 CD 优势。

【机制】输入级渲染的颜色噪声会干扰生成模型的结构推断，特征级在相同几何结构上使用动态参考特征并经自适应门控，减少对错误 RGB 的依赖。

【边界】巨大 CD（如167.85）可能涉及重建不稳定，不能直接当作真实场景误差倍数。此图不提供对所有输入级方法的数学证明，整体证据需结合 §4.4 及 Table 4。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-27 -->

<!-- PTH:BEGIN FIG-28 -->
## Figure 28 薄结构及遮挡失败

### 原图、题注与视觉布局

**Caption 原文**（PDF 转写）：Fig. 28 Limitations and Failure Cases of GeoNVS. GeoNVS exhibits geometry inconsistency in thin structures (left), where sparse Gaussian representations struggle to capture fine-grained geometry, and blurry textures in occluded regions (right), where the geometry prior provides insufficient guidance for unobserved areas.

【原文证据】PDF p.38，Figure 28。题注译意：GeoNVS 的局限与失败。薄结构中稀疏高斯难以表达精细几何；遮挡区域缺少几何条件，产生模糊纹理。

![Figure 28](GeoNVS/Figures/Figure_28.jpg)

*PDF 区域展示副本（300 DPI JPEG）；客户端显示未验证。*

【视觉观察】左侧红框围绕杆、栅栏等细结构的几何不一致，右侧红框标记参考不可见区域的模糊纹理；图示两类不同失败机制。

### 机制、数值与证据边界

【原文事实】§H 将两类错误分别归因于稀疏 Gaussian 表示的精细结构缺失和所有参考都不可见的区域缺少指导。作者提出 uncertainty-aware Gaussian 或 inpainting-based prior 作为未来方向，并未在本文验证。

【机制】薄结构首先是先验结构质量问题，错位置/尺度会使投影权重与真实结构错位；遮挡则是观测缺失问题，lifting 无法为未知表面提供可靠原始特征。门控可降低错误信息的注入，却不能凭空恢复真实几何或纹理。

【独立评价】这两种机制不能仅靠提高生成锐度或调整 CFG 解决；应分别考察几何覆盖、置信校准及不可见区的不确定性。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-28 -->

<!-- PTH:BEGIN TABLE-01 -->
## Table 1 小视角 PSNR 全组

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 1 Perceptual metrics on small-viewpoint set NVS. The small-viewpoint set contains scenes with high view overlap between reference and target views. P denotes the number of reference images. For P=1, we sweep the unit length for camera normalization [90] to resolve scale ambiguity. All combined methods (generative + geometry) use the best feed-forward geometry model per dataset as the geometry prior.

【原文事实】PDF p.9，Table 1。题注译意：高重叠参考/目标集合，P 为参考数；P=1 时扫描相机归一化单位长度解决尺度歧义，组合模型按数据集采用最优先验。下列每列标签为 benchmark split/P，– 为未报告。

**(a) PSNR ↑ (dB)**

| 方法 | OO3D S/3 | RE D/1 | RE P/2 | RE R/1 | RE R/3 | LLFF R/1 | LLFF R/3 | DTU R/1 | DTU R/3 | CO3D V/1 | CO3D R/3 | WRGBD Se/3 | WRGBD Sh/6 | Mip360 R/6 | DL3DV S/6 | DL3DV L/9 | T&T O/2 | T&T O/3 | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| MVSplat | 21.82 | 10.68 | 25.56 | 11.26 | 23.35 | 5.69 | 13.68 | 7.47 | 11.35 | 10.12 | 12.04 | 12.89 | 12.38 | 13.45 | 13.89 | 14.07 | 12.52 | 13.45 | 13.65 |
| HiSplat | 21.98 | 12.38 | 25.50 | 13.89 | 25.81 | 5.75 | 15.16 | 7.11 | 12.37 | 10.59 | 12.55 | 13.49 | 12.48 | 13.90 | 14.04 | 14.20 | 12.38 | 15.57 | 14.40 |
| DepthSplat | 22.66 | 13.02 | 25.57 | 14.49 | 26.99 | 5.91 | 15.02 | 8.79 | 14.19 | 11.29 | 15.08 | 13.92 | 13.50 | 15.04 | 15.87 | 16.88 | 15.76 | 17.99 | 15.67 |
| VGGT | 22.12 | 13.73 | 23.98 | 13.95 | 27.48 | 10.89 | 20.08 | 7.05 | 17.15 | 10.48 | 15.39 | 11.09 | 12.56 | 15.49 | 17.09 | 18.31 | 20.92 | 23.96 | 16.76 |
| Pi3 | 18.48 | 10.56 | 24.35 | 11.00 | 27.66 | 8.80 | 15.54 | 8.82 | 7.81 | 12.08 | 12.60 | 10.54 | 10.46 | 13.16 | 16.03 | 17.23 | 14.46 | 16.98 | 14.25 |
| MotionCtrl | - | 13.10 | - | 13.86 | - | 11.40 | - | 8.41 | - | 13.64 | - | - | - | - | - | - | - | - | 12.08 |
| CameraCtrl | - | 13.08 | - | 14.09 | - | 11.88 | - | 8.72 | - | 13.70 | - | - | - | - | - | - | - | - | 12.29 |
| ViewCrafter | 18.56 | 14.37 | - | 17.44 | 15.83 | 11.79 | 11.39 | 9.22 | 8.49 | 15.87 | 9.89 | 9.23 | 9.37 | 10.81 | 10.51 | 9.75 | 10.74 | 11.13 | 12.02 |
| SEVA | 26.28 | 14.51 | 22.77 | 15.25 | 24.44 | 12.22 | 17.12 | 10.54 | 17.03 | 16.29 | 17.22 | 15.96 | 15.71 | 15.38 | 15.84 | 16.87 | 16.35 | 18.85 | 17.15 |
| MVSplat360 | 21.45 | 13.68 | 20.20 | 13.44 | 20.97 | 12.79 | 15.31 | 9.10 | 11.38 | 12.69 | 14.29 | 14.34 | 13.22 | 14.15 | 15.09 | 15.26 | 14.22 | 15.84 | 14.86 |
| GenFusion | 20.90 | 14.87 | 22.48 | 15.88 | 22.69 | 11.39 | 17.87 | 8.36 | 13.75 | 12.15 | 12.99 | 13.38 | 12.47 | 11.99 | 13.11 | 14.78 | 18.25 | 20.14 | 15.30 |
| Difix3D | 23.05 | 12.92 | 25.23 | 14.37 | 27.14 | 10.85 | 19.59 | 8.79 | 17.22 | 11.93 | 15.23 | 12.60 | 12.67 | 14.95 | 16.33 | 17.33 | 20.68 | 23.36 | 16.89 |
| GeoNVS | 26.58 | 16.66 | 24.78 | 17.62 | 27.11 | 14.10 | 18.29 | 12.12 | 19.18 | 17.14 | 19.26 | 18.23 | 17.12 | 16.57 | 18.11 | 19.66 | 18.76 | 22.24 | 19.09 |

**(b) SSIM ↑**

| 方法 | OO3D S/3 | RE D/1 | RE P/2 | RE R/1 | RE R/3 | LLFF R/1 | LLFF R/3 | DTU R/1 | DTU R/3 | CO3D V/1 | CO3D R/3 | WRGBD Se/3 | WRGBD Sh/6 | Mip360 R/6 | DL3DV S/6 | DL3DV L/9 | T&T O/2 | T&T O/3 | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| HiSplat | 0.877 | 0.444 | 0.872 | 0.540 | 0.902 | 0.01 | 0.384 | 0.149 | 0.467 | 0.286 | 0.434 | 0.434 | 0.419 | 0.290 | 0.419 | 0.457 | 0.360 | 0.468 | 0.456 |
| DepthSplat | 0.897 | 0.468 | 0.868 | 0.529 | 0.918 | 0.02 | 0.332 | 0.262 | 0.485 | 0.337 | 0.492 | 0.441 | 0.434 | 0.303 | 0.465 | 0.528 | 0.472 | 0.535 | 0.488 |
| VGGT | 0.892 | 0.499 | 0.798 | 0.561 | 0.914 | 0.234 | 0.672 | 0.215 | 0.640 | 0.297 | 0.524 | 0.409 | 0.432 | 0.362 | 0.518 | 0.571 | 0.716 | 0.807 | 0.559 |
| CameraCtrl | - | 0.491 | - | 0.578 | - | 0.294 | - | 0.400 | - | 0.451 | - | - | - | - | - | - | - | - | 0.443 |
| ViewCrafter | 0.861 | 0.551 | - | 0.693 | 0.647 | 0.283 | 0.253 | 0.370 | 0.274 | 0.520 | 0.300 | 0.233 | 0.215 | 0.194 | 0.241 | 0.230 | 0.286 | 0.300 | 0.379 |
| SEVA | 0.926 | 0.534 | 0.777 | 0.616 | 0.842 | 0.294 | 0.467 | 0.430 | 0.659 | 0.507 | 0.551 | 0.527 | 0.522 | 0.339 | 0.450 | 0.513 | 0.521 | 0.608 | 0.560 |
| GenFusion | 0.876 | 0.559 | 0.818 | 0.618 | 0.864 | 0.271 | 0.554 | 0.355 | 0.556 | 0.438 | 0.449 | 0.453 | 0.425 | 0.282 | 0.394 | 0.492 | 0.630 | 0.689 | 0.540 |
| Difix3D | 0.898 | 0.463 | 0.857 | 0.528 | 0.903 | 0.230 | 0.638 | 0.342 | 0.637 | 0.375 | 0.479 | 0.350 | 0.333 | 0.301 | 0.481 | 0.519 | 0.696 | 0.779 | 0.545 |
| GeoNVS | 0.904 | 0.612 | 0.826 | 0.680 | 0.895 | 0.346 | 0.514 | 0.480 | 0.691 | 0.550 | 0.627 | 0.607 | 0.577 | 0.383 | 0.562 | 0.635 | 0.603 | 0.695 | 0.622 |

**(c) LPIPS ↓**

| 方法 | OO3D S/3 | RE D/1 | RE P/2 | RE R/1 | RE R/3 | LLFF R/1 | LLFF R/3 | DTU R/1 | DTU R/3 | CO3D V/1 | CO3D R/3 | WRGBD Se/3 | WRGBD Sh/6 | Mip360 R/6 | DL3DV S/6 | DL3DV L/9 | T&T O/2 | T&T O/3 | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| HiSplat | 0.155 | 0.522 | 0.110 | 0.484 | 0.094 | 1.02 | 0.479 | 0.783 | 0.483 | 0.645 | 0.581 | 0.496 | 0.610 | 0.691 | 0.627 | 0.650 | 0.612 | 0.474 | 0.529 |
| DepthSplat | 0.134 | 0.444 | 0.100 | 0.414 | 0.074 | 0.979 | 0.351 | 0.550 | 0.362 | 0.553 | 0.498 | 0.431 | 0.493 | 0.552 | 0.487 | 0.477 | 0.332 | 0.235 | 0.415 |
| VGGT | 0.172 | 0.441 | 0.138 | 0.454 | 0.078 | 0.517 | 0.153 | 0.779 | 0.340 | 0.608 | 0.530 | 0.696 | 0.623 | 0.618 | 0.466 | 0.497 | 0.194 | 0.127 | 0.413 |
| CameraCtrl | - | 0.514 | - | 0.492 | - | 0.610 | - | 0.611 | - | 0.540 | - | - | - | - | - | - | - | - | 0.553 |
| ViewCrafter | 0.211 | 0.405 | - | 0.338 | 0.385 | 0.527 | 0.562 | 0.634 | 0.690 | 0.425 | 0.727 | 0.715 | 0.727 | 0.675 | 0.649 | 0.680 | 0.542 | 0.516 | 0.553 |
| SEVA | 0.062 | 0.400 | 0.131 | 0.402 | 0.095 | 0.469 | 0.212 | 0.524 | 0.210 | 0.399 | 0.333 | 0.339 | 0.331 | 0.367 | 0.305 | 0.288 | 0.238 | 0.164 | 0.293 |
| GenFusion | 0.158 | 0.408 | 0.183 | 0.384 | 0.159 | 0.554 | 0.301 | 0.702 | 0.430 | 0.621 | 0.616 | 0.507 | 0.577 | 0.655 | 0.537 | 0.520 | 0.278 | 0.223 | 0.434 |
| Difix3D | 0.106 | 0.447 | 0.097 | 0.422 | 0.083 | 0.520 | 0.153 | 0.648 | 0.240 | 0.544 | 0.456 | 0.508 | 0.549 | 0.425 | 0.313 | 0.326 | 0.158 | 0.101 | 0.339 |
| GeoNVS | 0.074 | 0.325 | 0.113 | 0.319 | 0.071 | 0.412 | 0.208 | 0.449 | 0.174 | 0.393 | 0.279 | 0.275 | 0.290 | 0.329 | 0.245 | 0.235 | 0.225 | 0.138 | 0.253 |

### 数值比较、设置与边界

【数值比较】18 设置平均 PSNR 19.09 对 SEVA 17.15（+1.94 dB，按数值比约11.3%），SSIM .622 对 .560，LPIPS .253 对 .293。Ours 相对 SEVA 的 PSNR 在所有18列更高；相对最佳纯几何则并非全胜，如 T&T O/3 VGGT 23.96 >22.24、LLFF R/3 VGGT 20.08 >18.29、RE P/2 DepthSplat 25.57 >24.78。OO3D 的 SSIM .904 低于 SEVA .926，LPIPS .074 高于 .062；故不可把平均改善称为每个指标全部领先。

【边界】单参考扫描尺度的协议、不同方法缺失列和不同数据集先验选择都影响公平性；不应把单视角方法的 Avg 与 18 设定平均等价。附录表11包含主表省略的其他 SSIM/LPIPS 行；不能用 HTML 编号替代。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
## Table 2 大视角 PSNR

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 2 Large-viewpoint set NVS. The large-viewpoint set contains scenes with low view overlap between reference and target views. P denotes the number of reference images. We use DepthSplat [80] as the geometry prior for both combined methods.

【原文事实】PDF p.10，Table 2。题注译意：参考与目标低重叠；P 为参考数，组合方法采用 DepthSplat 先验。PSNR ↑。

| 方法 | CO3D R/1 | WRGBD Sh/1 | Sh/3 | Mip360 R/1 | R/3 | DL3DV S/1 | S/3 | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| DepthSplat | 8.54 | 7.31 | 11.94 | 9.20 | 14.22 | 7.37 | 14.67 | 10.46 |
| MotionCtrl | 11.77 | 10.33 | - | 11.04 | - | 10.99 | - | 11.03 |
| CameraCtrl | 11.71 | 10.38 | - | 10.94 | - | 11.05 | - | 11.02 |
| ViewCrafter | 11.11 | - | 9.32 | 10.79 | 10.38 | 11.18 | 10.20 | 10.50 |
| SEVA | 12.27 | 11.27 | 13.72 | 11.19 | 14.30 | 11.48 | 14.27 | 12.64 |
| GenFusion | 10.21 | 9.60 | 11.37 | 9.84 | 11.88 | 10.11 | 13.33 | 10.91 |
| Difix3D | 8.56 | 7.33 | 11.81 | 9.30 | 13.83 | 7.48 | 14.24 | 10.36 |
| Ours | 14.66 | 12.72 | 15.08 | 12.34 | 15.47 | 13.07 | 16.09 | 14.20 |

### 数值比较、设置与边界

【数值】Ours 平均14.20 vs SEVA12.64 (+1.56 dB)，7个设置都更高。单参考纯 DepthSplat 在 WRGBD Sh/1、DL3DV S/1 仅7.31/7.37，而 SEVA11.27/11.48，说明低重叠时几何本身不足以覆盖目标。Ours 对应12.72/13.07。

【边界】分割由 CLIP 距离阈值0.11确定（§4.3），不等于固定视角角度；用相同先验有助于比较融合路径，但不能证明所有方法公平到每个超参数。SSIM/LPIPS见表12。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
## Table 3 长轨迹 PSNR

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 3 Long trajectory NVS. The long trajectory NVS measures the long-term generation ability of generative NVS. P denotes the number of reference images. We use VGGT [65] as the geometry prior for our method.

【原文事实】PDF p.10，Table 3。题注译意：长轨迹 NVS 衡量长期生成；P 为参考数，Ours 采用 VGGT 先验。

**(a) PSNR ↑**

| 方法 | Mip360/3 | /6 | /9 | DL3DV/3 | /6 | /9 | T&T/2 | /3 | /6 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| DepthSplat | 13.22 | 13.91 | 14.57 | 14.79 | 15.90 | 16.68 | 15.69 | 17.51 | 18.11 |
| VGGT | 12.16 | 14.45 | 15.80 | 15.43 | 16.99 | 18.11 | 20.99 | 24.06 | 26.84 |
| SEVA | 13.01 | 14.32 | 15.16 | 14.11 | 15.65 | 16.73 | 16.59 | 19.27 | 23.09 |
| Ours | 14.16 | 15.52 | 16.45 | 16.03 | 17.99 | 19.32 | 19.26 | 22.31 | 24.52 |

**(b) 几何指标 ↓（每组依次 Terr/cm, Rerr/deg, CD）**

| 方法 | RE10K/3 T | R | CD | DL3DV/3 T | R | CD | T&T/3 T | R | CD |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| SEVA | 1.62 | 5.84 | 1.35 | 35.0 | 6.44 | 1.03 | 6.08 | 2.87 | 1.42 |
| Ours | 0.76 | 4.83 | 0.19 | 28.3 | 5.13 | 0.62 | 3.56 | 1.52 | 0.92 |

### 数值比较、设置与边界

【数值】9 个长轨迹设置 PSNR 均超过 SEVA；DL3DV P=9 +2.59 dB，T&T P=3 +3.04 dB；但 T&T 的 VGGT 在 P=2/3/6 三列皆更高。RE10K 的 T 由1.62降0.76（约2.13倍），CD由1.35降0.19（约7.11倍），对应摘要最高约2倍、7倍。DL3DV 与T&T 改善幅度更小。

【边界】长轨迹采用 SEVA 的 two-pass sampling，不能把所有时序稳定性归功于新适配器。几何指标由 ViPE 重建估计，p.11 说明排除 GT 重建失败（T 或 R>50）的场景。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN TABLE-04 -->
## Table 4 几何一致性指标

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 4 Geometry Fidelity of GeoNVS. Terr, Rerr, and CD measure geometric fidelity. PSNRV and PSNRU denote synthesis quality on co-visible and non-co-visible regions w.r.t. the reference views. All the results are under the Trajectory NVS setting. “Input-level injection” denotes input-level integration of 3D Gaussian priors.

【原文事实】PDF p.10，Table 4。题注译意：轨迹 NVS 几何保真与共视/非共视区域合成质量，组合模型采用 VGGT 先验。T/R/CD 越低越好；PSNRV/U 越高越好。

| 方法 | Mip360 P3 T | R | CD | PSNRV | PSNRU | DL3DV P6 T | R | CD | PSNRV | PSNRU | WRGBD-Sh P3 T | R | CD | PSNRV | PSNRU |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| SEVA | 18.76 | 3.29 | 0.07 | 13.85 | 13.01 | 19.68 | 2.80 | 0.57 | 15.68 | 16.90 | 8.10 | 8.86 | 1.58 | 15.53 | 15.79 |
| Difix3D | 219.39 | 51.64 | 0.68 | 12.92 | 11.82 | 64.4 | 9.66 | 3.76 | 16.11 | 17.29 | 19.59 | 32.48 | 10.11 | 11.07 | 11.36 |
| SEVA+Input-level | 233.03 | 99.79 | 1.02 | 13.14 | 12.03 | 81.14 | 17.19 | 3.82 | 16.76 | 17.76 | 29.27 | 52.51 | 2.01 | 11.36 | 11.42 |
| Ours | 16.24 | 2.98 | 0.07 | 15.11 | 14.07 | 13.27 | 2.01 | 0.41 | 18.11 | 18.98 | 4.00 | 4.56 | 1.29 | 17.33 | 17.51 |

### 数值比较、设置与边界

【数值】Ours T/R 在三列低于 SEVA，CD 在 Mip360 同为0.07（按显示精度），DL3DV .57→.41、WRGBD 1.58→1.29。输入级 T/R 大幅退化，例如 Mip360 233.03/99.79；同时 DL3DV 输入级的 PSNRU 17.76 高于 SEVA16.90，说明像素质量与轨迹误差不可互相替代。

PSNRU 对 SEVA 分别+1.06/+2.08/+1.72 dB，PSNRV +1.26/+2.43/+1.80 dB。共视掩码由 VGGT 参考点云投影至目标构造（§4.4）。

【边界】非共视由估计点云而不是真实 visibility 标注定义；未证明不可见信息的唯一真实恢复。输入级对照仅特定方法及强度，且 ViPE 失败过滤影响评估样本。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-04 -->

<!-- PTH:BEGIN TABLE-05 -->
## Table 5 SEVA 先验替换

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 5 GeoNVS with SEVA [90]. We extend GeoNVS with various geometry priors in a zero-shot manner. All methods are evaluated with 3 reference images.

【原文事实】PDF p.12，Table 5。题注译意：SEVA 骨干上三参考的几何先验零样本替换。每组 PSNR/SSIM。

| 方法 | DTU P | S | CO3D P | S | WRGBD-Se P | S | T&T P | S |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| SEVA | 17.03 | .66 | 17.22 | .55 | 15.96 | .53 | 18.85 | .61 |
| MVSplat | 11.35 | .41 | 12.04 | .39 | 12.89 | .41 | 13.45 | .38 |
| DepthSplat | 14.19 | .49 | 15.08 | .49 | 13.92 | .44 | 17.99 | .54 |
| VGGT | 17.15 | .64 | 15.39 | .52 | 11.09 | .41 | 23.96 | .81 |
| Pi3 | 7.81 | .31 | 12.60 | .47 | 10.54 | .37 | 16.98 | .48 |
| Naïve+MVSplat | 17.04 | .65 | 18.79 | .62 | 17.62 | .59 | 21.18 | .68 |
| Naïve+DepthSplat | 17.94 | .68 | 19.00 | .63 | 17.97 | .61 | 22.05 | .71 |
| Naïve+VGGT | 18.89 | .69 | 18.60 | .60 | 17.60 | .59 | 22.22 | .70 |
| Naïve+Pi3 | 14.83 | .59 | 18.59 | .61 | 17.38 | .58 | 21.92 | .69 |
| Adaptive+MVSplat | 19.15 | .69 | 19.23 | .63 | 18.22 | .61 | 22.24 | .69 |
| Adaptive+DepthSplat | 19.14 | .69 | 19.24 | .63 | 18.22 | .61 | 22.24 | .69 |
| Adaptive+VGGT | 19.18 | .69 | 19.26 | .63 | 18.21 | .61 | 22.24 | .69 |
| Adaptive+Pi3 | 19.02 | .69 | 19.22 | .63 | 18.19 | .61 | 22.22 | .69 |

### 数值比较、设置与边界

【数值】DTU naïve+Pi3 14.83 <SEVA17.03；adaptive+Pi3 19.02，较 naïve +4.19 dB，先验替换的 PSNR 范围仅19.02–19.18。CO3D 19.22–19.26，WRGBD-Se 18.19–18.22，T&T 22.22–22.24。适应性融合降低了先验差异对输出的影响。但 T&T 纯 VGGT23.96/.81 高于所有组合；Adaptive 在 T&T SSIM .69 亦不超过部分 naïve .70/.71。

【边界】作者称两融合持续改善基线但正文自己承认 DTU 例外；据数值解读为大多数设置改善、adaptive 对 PSNR 更稳健，不声称所有指标统一领先。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-05 -->

<!-- PTH:BEGIN TABLE-06 -->
## Table 6 CameraCtrl 先验替换

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 6 PSNR ↑on GeoNVS with CameraCtrl [24]. GeoNVS built on CameraCtrl generalizes to various geometry priors in a zero-shot manner, without additional training. Following the original CameraCtrl setting, all methods are evaluated using a single reference image.

【原文事实】PDF p.12，Table 6。题注译意：CameraCtrl 骨干单参考，不需因替换先验额外训练，PSNR ↑。

| 方法 | CO3D V | CO3D R | DTU R | RE10K R | Mip360 R | WRGBD Sh | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| CameraCtrl | 13.23 | 11.71 | 8.72 | 14.09 | 10.94 | 10.38 | 11.14 |
| MVSplat | 10.12 | 9.14 | 7.37 | 11.26 | 8.55 | 6.48 | 8.82 |
| DepthSplat | 11.29 | 8.54 | 8.79 | 14.49 | 9.20 | 7.31 | 9.94 |
| VGGT | 10.48 | 8.43 | 7.04 | 13.94 | 8.89 | 6.27 | 8.67 |
| Pi3 | 12.08 | 11.80 | 8.82 | 11.00 | 10.60 | 8.86 | 10.16 |
| Naïve+MVSplat | 13.76 | 12.69 | 7.71 | 14.44 | 11.19 | 10.63 | 11.74 |
| Naïve+DepthSplat | 14.64 | 12.15 | 11.00 | 16.85 | 11.27 | 10.88 | 12.80 |
| Naïve+VGGT | 13.30 | 11.69 | 8.68 | 14.08 | 11.30 | 10.55 | 11.60 |
| Naïve+Pi3 | 13.60 | 12.99 | 8.99 | 13.35 | 11.30 | 10.97 | 11.87 |

### 数值比较、设置与边界

【数值】DepthSplat 组合均值12.80 vs11.14（+1.66 dB，约14.9%），其余先验均值也提高。但 MVSplat+Naïve DTU7.71低于基线8.72，VGGT在 CO3D R/DTU/RE10K 略低，Pi3在RE10K13.35低于14.09。纯几何也并非每列均低于 CameraCtrl（DepthSplat DTU、RE10K 等）。

【边界】表6只展示 naïve fusion，不应声称 CameraCtrl adaptive 的对应实验已完成。更换骨干需要 §4.2 所述独立训练。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-06 -->

<!-- PTH:BEGIN TABLE-07 -->
## Table 7 组件消融

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 7 Model ablation study of GeoNVS. We ablate the key components of GS-Adapter, including feature refinement, feature fusion strategy, and multi-scale fusion. All experiments use VGGT [65] as the geometry prior. Here, feats abbreviates features.

【原文事实】PDF p.13，Table 7。题注译意：VGGT 先验下精炼损失、GS-PE、融合策略及多尺度组件消融。

| 方法 | Lfeat | GS-PE | Naïve | Adaptive | Mip360 P/S | DL3DV P/S | T&T P/S |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| VGGT | - | - | - | - | 15.49/.362 | 17.09/.52 | 23.96/.81 |
| SEVA | - | - | - | - | 15.38/.339 | 15.84/.45 | 18.85/.61 |
| +Input-level | - | - | - | - | 15.40/.373 | 16.73/.52 | 23.05/.74 |
| +LoRA only | - | - | - | - | 15.55/.360 | 17.35/.54 | 21.41/.68 |
| Adapter 1,2,3 | ✓ | - | ✓ | - | 16.36/.378 | 17.80/.55 | 21.67/.69 |
| Adapter 1,2,3 | - | ✓ | ✓ | - | 16.47/.378 | 17.88/.55 | 22.11/.69 |
| Adapter 1,2,3 | ✓ | ✓ | ✓ | - | 16.48/.383 | 17.95/.56 | 22.22/.70 |
| Encoder1 | ✓ | ✓ | ✓ | - | 16.28/.376 | 17.49/.53 | 21.36/.68 |
| Encoder1,2 | ✓ | ✓ | ✓ | - | 16.31/.370 | 17.53/.53 | 21.73/.68 |
| Encoder1,2,3 | ✓ | ✓ | ✓ | - | 16.48/.383 | 17.95/.56 | 22.22/.70 |
| -RefineNet | - | - | - | ✓ | 16.23/.376 | 17.71/.45 | 21.91/.69 |
| Ours 1,2,3 | ✓ | ✓ | - | ✓ | 16.57/.383 | 18.11/.56 | 22.24/.69 |

### 数值比较、设置与边界

【数值】去 RefineNet 后相对 full PSNR 降 .34/.40/.33；DL3DV SSIM .45→.56，T&T SSIM 保持 .69。naïve 三尺度比单尺度 PSNR +.20/.46/.86；两尺度的 Mip360 SSIM .370 反低于单尺度 .376。Lfeat/GS-PE 共同启用的 PSNR 高于各单独启用。Adaptive 相对全 naïve PSNR +.09/.16/.02，T&T SSIM .69 vs .70；T&T input-level23.05 也高于 full22.24。

【边界】LoRA only 已显著改善，故不能把 full 相对原基线的全部增益归因于 adapter；应比较相同微调下模块增量。表中不是完整全因子设计，无方差，不证明每一组件在所有指标中必要。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-07 -->

<!-- PTH:BEGIN TABLE-08 -->
## Table 8 无位姿设置

### 题注、指标及数据

**Caption 原文**：PDF 未印出独立题注，按所在正文、源标签或未编号 tabular 定位。

【原文事实】PDF p.14，Table 8。PDF p.14 右表无打印题注，Source 静态标签 tab:pose_free；正文解释为 pose-free experiment。

| 位姿 | 方法 | DL3DV/6 T | R | PSNR | T&T/3 T | R | PSNR |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GT | SEVA | 19.68 | 2.80 | 15.64 | 6.08 | 2.87 | 18.63 |
| GT | Ours | 13.27 | 2.01 | 17.99 | 3.56 | 1.52 | 22.24 |
| ViPE | SEVA | 25.11 | 3.54 | 15.38 | 2.73 | 4.87 | 19.24 |
| ViPE | Ours | 20.64 | 2.98 | 17.87 | 1.05 | 2.40 | 22.31 |

### 数值比较、设置与边界

【数值】ViPE 下 DL3DV Ours PSNR17.87 vs GT17.99，T/R恶化；T&T 22.31 vs22.24略升，T 1.05 vs3.56下降，R2.40 vs1.52升高。作者解释可能因 COLMAP-derived poses 噪声；这是推测而非确认。

【机制与边界】ViPE 估计输入位姿，VGGT 点云按 Eq.(1) 对齐至 ViPE poses，再优化高斯；pose-free 不代表无需目标相机控制，也不是不依赖任何相机估计。表只覆盖两个基准。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-08 -->

<!-- PTH:BEGIN TABLE-09 -->
## Table 9 数据集先验选择

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 9 Geometry prior used for combined methods of small-viewpoint NVS in Tab. 1.

【原文事实】PDF p.18，Table 9。题注译意：Table 1 组合模型所用几何先验。

| benchmark split | P | 先验 |
| --- | ---: | ---: |
| OO3D-S | 3 | DepthSplat |
| RE10K-D | 1 | DepthSplat |
| RE10K-P | 2 | DepthSplat |
| RE10K-R | 1 | DepthSplat |
| RE10K-R | 3 | Pi3 |
| LLFF-R | 1,3 | VGGT |
| DTU-R | 1 | Pi3 |
| DTU-R | 3 | VGGT |
| CO3D-V | 1 | Pi3 |
| CO3D-R | 3 | VGGT |
| WRGBD-Se | 3 | DepthSplat |
| WRGBD-Sh | 6 | DepthSplat |
| Mip360-R | 6 | VGGT |
| DL3DV-S | 6 | VGGT |
| DL3DV-L | 9 | VGGT |
| T&T-O | 2,3 | VGGT |

### 数值比较、设置与边界

【原文事实】各数据集和参考数选择表现最佳先验；组合方法尽量采用相同先验，MVSplat360 设计限制为 MVSplat（§4.2）。VGGT/Pi3 先预测点云与相机再拟合高斯，DepthSplat 直接预测高斯（附录B.2）。

【独立评价】选择按基准适配的强先验是控制比较因素，但实际部署若未知最优先验需另考虑选择依据；表未证明一种模型在所有场景最优。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-09 -->

<!-- PTH:BEGIN TABLE-C3 -->
## Table C.3 (unnumbered) 精炼 PSNR 与 SSIM

### 题注、指标及数据

**Caption 原文**：PDF 未印出独立题注，按所在正文、源标签或未编号 tabular 定位。

【原文事实】PDF p.19，Table C.3 (unnumbered)。§C.3 未编号表，紧邻 Fig.13。540个随机 DL3DV 测试场景的潜域 lifting/rendering/refinement 受控实验。

| 方法 | 精炼 | 3视图 PSNR | SSIM | 6视图 PSNR | SSIM | 9视图 PSNR | SSIM |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| (a) | 否 | 16.35 | .56 | 17.04 | .58 | 17.38 | .59 |
| (b) | 是 | 16.59 | .57 | 18.07 | .60 | 18.75 | .62 |

### 数值比较、设置与边界

【数值】PSNR +0.24/+1.03/+1.37 dB，SSIM +.01/+.02/+.03。更多参考时提升较大，但这是固定测试协议下的相关观察，不是已证明的通用单调规律。

【机制】潜变量通道不一定与 opacity 线性对应，重渲染发生混合损失，参考监督精炼恢复部分信息。此表采用每场景潜域优化对照，不是主模型整体训练或推理时间。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-C3 -->

<!-- PTH:BEGIN TABLE-10 -->
## Table 10 符号表

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 10 Notation Table for GeoNVS.

【原文事实】PDF p.24，Table 10。题注译意：GeoNVS 符号表，按场景相机、高斯、扩散、损失和评价分类。

| 符号 | 定义 |
| --- | ---: |
| P,Q | 参考/目标视图数 |
| Iref,Itar | 参考/目标 RGB |
| πref,πtar | 参考/目标相机位姿 |
| s* | 相机尺度对齐 |
| Cgt,Cpred | GT/预测相机中心 |
| G,N | 高斯集合/粒子数 |
| μ,r,s,σ | 中心/四元数旋转/尺度/不透明度 |
| C(d,p),ci(d) | 像素渲染颜色/高斯视角相关颜色 |
| wi,αi | 渲染权重/不透明度乘投影密度 |
| fi | 每高斯特征 |
| n,C,t | VAE 降采样因子/通道数/去噪步 |
| Ft | 当前扩散特征 |
| Ftref,Fttar | 参考 P×h/n×w/n×C、目标 Q×h/n×w/n×C |
| F̂t | 几何增强扩散特征 |
| f̂i | L2归一化高斯特征 |
| Gtar | 渲染目标几何特征 |
|  i*(d,p) | 最大贡献高斯索引 |
| x̃(d,p),w* | 归一化中心/最大权重 |
| γ,D',ωk | 正弦编码/每变量 C/4 维/频率 |
| G' | 加位置编码的特征 |
| R,Ĝ | ResNet 精炼器/精炼结果 |
| Ĝref,Ĝtar | 精炼参考和目标特征 |
| FA, Wtar | 几何注意力特征/像素门控 [-1,1] |
| F̂tar | 最终几何增强目标特征 |
| Lfeat,Llatent | 特征一致性/潜空间对齐损失 |
| Terr,Rerr,CD | 平移 cm/旋转 deg/Chamfer 距离 |
| PSNRV,PSNRU | 共视/非共视 PSNR |

### 数值比较、设置与边界

【理解】C 在 C(d,p) 表颜色，在特征形状表示通道数；s 在 Eq.(1) 为全局相机尺度，在高斯参数为三维尺度，在输入级强度实验又为 s∈[0,1]；G 表高斯集合、粗体 Gtar 为渲染特征。i* 是最大渲染权重的高斯，并不一定等于最大 opacity。Wtar 是有符号门控而非概率。

【边界】本表是符号定义，不是独立推导，公式维度与成立条件见 MATH。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-10 -->

<!-- PTH:BEGIN TABLE-11 -->
## Table 11 小视角 SSIM/LPIPS

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 11 Small-viewpoint set NVS. We report SSIM and LPIPS metrics corresponding to Tab. 1 of the manuscript. P denotes the number of reference images. For P=1, we sweep the unit length for camera normalization following [90] to resolve scale ambiguity. For each dataset, all combined (generative + geometry) methods use the top-performing feed-forward geometry model by PSNR as the geometry prior.

【原文事实】PDF p.25，Table 11。题注译意：与表1对应的小视角 SSIM 与 LPIPS 完整结果，补齐主表省略行。

**(a) SSIM ↑**

| 方法 | OO3D S/3 | RE D/1 | RE P/2 | RE R/1 | RE R/3 | LLFF R/1 | LLFF R/3 | DTU R/1 | DTU R/3 | CO3D V/1 | CO3D R/3 | WRGBD Se/3 | WRGBD Sh/6 | Mip360 R/6 | DL3DV S/6 | DL3DV L/9 | T&T O/2 | T&T O/3 | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| MVSplat | 0.879 | 0.317 | 0.868 | 0.364 | 0.870 | 0.003 | 0.323 | 0.140 | 0.409 | 0.237 | 0.390 | 0.406 | 0.393 | 0.239 | 0.379 | 0.427 | 0.333 | 0.377 | 0.409 |
| HiSplat | 0.877 | 0.444 | 0.872 | 0.540 | 0.902 | 0.01 | 0.384 | 0.149 | 0.467 | 0.286 | 0.434 | 0.434 | 0.419 | 0.290 | 0.419 | 0.457 | 0.360 | 0.468 | 0.456 |
| DepthSplat | 0.897 | 0.468 | 0.868 | 0.529 | 0.918 | 0.02 | 0.332 | 0.262 | 0.485 | 0.337 | 0.492 | 0.441 | 0.434 | 0.303 | 0.465 | 0.528 | 0.472 | 0.535 | 0.488 |
| VGGT | 0.892 | 0.499 | 0.798 | 0.561 | 0.914 | 0.234 | 0.672 | 0.215 | 0.640 | 0.297 | 0.524 | 0.409 | 0.432 | 0.362 | 0.518 | 0.571 | 0.716 | 0.807 | 0.559 |
| Pi3 | 0.857 | 0.392 | 0.807 | 0.452 | 0.910 | 0.180 | 0.374 | 0.356 | 0.314 | 0.420 | 0.470 | 0.371 | 0.385 | 0.277 | 0.455 | 0.523 | 0.406 | 0.475 | 0.468 |
| MotionCtrl | - | 0.502 | - | 0.592 | - | 0.314 | - | 0.401 | - | 0.452 | - | - | - | - | - | - | - | - | 0.452 |
| CameraCtrl | - | 0.491 | - | 0.578 | - | 0.294 | - | 0.400 | - | 0.451 | - | - | - | - | - | - | - | - | 0.443 |
| ViewCrafter | 0.861 | 0.551 | - | 0.693 | 0.647 | 0.283 | 0.253 | 0.370 | 0.274 | 0.520 | 0.300 | 0.233 | 0.215 | 0.194 | 0.241 | 0.230 | 0.286 | 0.300 | 0.379 |
| SEVA | 0.926 | 0.534 | 0.777 | 0.616 | 0.842 | 0.294 | 0.467 | 0.430 | 0.659 | 0.507 | 0.551 | 0.527 | 0.522 | 0.339 | 0.450 | 0.513 | 0.521 | 0.608 | 0.560 |
| MVSplat360 | 0.884 | 0.518 | 0.756 | 0.574 | 0.817 | 0.314 | 0.419 | 0.366 | 0.517 | 0.459 | 0.555 | 0.514 | 0.483 | 0.327 | 0.471 | 0.524 | 0.467 | 0.535 | 0.528 |
| GenFusion | 0.876 | 0.559 | 0.818 | 0.618 | 0.864 | 0.271 | 0.554 | 0.355 | 0.556 | 0.438 | 0.449 | 0.453 | 0.425 | 0.282 | 0.394 | 0.492 | 0.630 | 0.689 | 0.540 |
| Difix3D | 0.898 | 0.463 | 0.857 | 0.528 | 0.903 | 0.230 | 0.638 | 0.342 | 0.637 | 0.375 | 0.479 | 0.350 | 0.333 | 0.301 | 0.481 | 0.519 | 0.696 | 0.779 | 0.545 |
| GeoNVS | 0.904 | 0.612 | 0.826 | 0.680 | 0.895 | 0.346 | 0.514 | 0.480 | 0.691 | 0.550 | 0.627 | 0.607 | 0.577 | 0.383 | 0.562 | 0.635 | 0.603 | 0.695 | 0.622 |

**(b) LPIPS ↓**

| 方法 | OO3D S/3 | RE D/1 | RE P/2 | RE R/1 | RE R/3 | LLFF R/1 | LLFF R/3 | DTU R/1 | DTU R/3 | CO3D V/1 | CO3D R/3 | WRGBD Se/3 | WRGBD Sh/6 | Mip360 R/6 | DL3DV S/6 | DL3DV L/9 | T&T O/2 | T&T O/3 | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| MVSplat | 0.152 | 0.643 | 0.098 | 0.615 | 0.134 | 1.04 | 0.514 | 0.816 | 0.516 | 0.662 | 0.592 | 0.494 | 0.615 | 0.673 | 0.614 | 0.630 | 0.566 | 0.533 | 0.550 |
| HiSplat | 0.155 | 0.522 | 0.110 | 0.484 | 0.094 | 1.02 | 0.479 | 0.783 | 0.483 | 0.645 | 0.581 | 0.496 | 0.610 | 0.691 | 0.627 | 0.650 | 0.612 | 0.474 | 0.529 |
| DepthSplat | 0.134 | 0.444 | 0.100 | 0.414 | 0.074 | 0.979 | 0.351 | 0.550 | 0.362 | 0.553 | 0.498 | 0.431 | 0.493 | 0.552 | 0.487 | 0.477 | 0.332 | 0.235 | 0.415 |
| VGGT | 0.172 | 0.441 | 0.138 | 0.454 | 0.078 | 0.517 | 0.153 | 0.779 | 0.340 | 0.608 | 0.530 | 0.696 | 0.623 | 0.618 | 0.466 | 0.497 | 0.194 | 0.127 | 0.413 |
| Pi3 | 0.294 | 0.701 | 0.136 | 0.684 | 0.087 | 0.691 | 0.328 | 0.666 | 0.727 | 0.593 | 0.643 | 0.652 | 0.675 | 0.660 | 0.467 | 0.503 | 0.406 | 0.311 | 0.512 |
| MotionCtrl | - | 0.502 | - | 0.478 | - | 0.738 | - | 0.664 | - | 0.518 | - | - | - | - | - | - | - | - | 0.580 |
| CameraCtrl | - | 0.514 | - | 0.492 | - | 0.610 | - | 0.611 | - | 0.540 | - | - | - | - | - | - | - | - | 0.553 |
| ViewCrafter | 0.211 | 0.405 | - | 0.338 | 0.385 | 0.527 | 0.562 | 0.634 | 0.690 | 0.425 | 0.727 | 0.715 | 0.727 | 0.675 | 0.649 | 0.680 | 0.542 | 0.516 | 0.553 |
| SEVA | 0.062 | 0.400 | 0.131 | 0.402 | 0.095 | 0.469 | 0.212 | 0.524 | 0.210 | 0.399 | 0.333 | 0.339 | 0.331 | 0.367 | 0.305 | 0.288 | 0.238 | 0.164 | 0.293 |
| MVSplat360 | 0.153 | 0.454 | 0.203 | 0.471 | 0.164 | 0.581 | 0.413 | 0.731 | 0.485 | 0.581 | 0.525 | 0.436 | 0.487 | 0.556 | 0.380 | 0.410 | 0.445 | 0.348 | 0.435 |
| GenFusion | 0.158 | 0.408 | 0.183 | 0.384 | 0.159 | 0.554 | 0.301 | 0.702 | 0.430 | 0.621 | 0.616 | 0.507 | 0.577 | 0.655 | 0.537 | 0.520 | 0.278 | 0.223 | 0.434 |
| Difix3D | 0.106 | 0.447 | 0.097 | 0.422 | 0.083 | 0.520 | 0.153 | 0.648 | 0.240 | 0.544 | 0.456 | 0.508 | 0.549 | 0.425 | 0.313 | 0.326 | 0.158 | 0.101 | 0.339 |
| GeoNVS | 0.074 | 0.325 | 0.113 | 0.319 | 0.071 | 0.412 | 0.208 | 0.449 | 0.174 | 0.393 | 0.279 | 0.275 | 0.290 | 0.329 | 0.245 | 0.235 | 0.225 | 0.138 | 0.253 |

### 数值比较、设置与边界

【数值】Ours 平均 SSIM .622，LPIPS .253，优于 SEVA .560/.293；相对 VGGT .559/.413 改善感知平均。但 T&T P=3 VGGT SSIM .807 >.695，Difix3D LPIPS .101 <.138；LLFF P=3 VGGT SSIM .672 >.514。

【边界】SSIM度量局部结构相似，LPIPS度量学习特征感知距离，均非三维几何真值；单图生成的锐利假细节也可能影响指标。缺失数据保留“-”。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-11 -->

<!-- PTH:BEGIN TABLE-12 -->
## Table 12 大视角 SSIM/LPIPS

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 12 Large-viewpoint set NVS. The large-viewpoint set contains scenes with low view overlap between reference and target views. P denotes the number of reference images. We use DepthSplat [80] as the geometry prior for both combined methods.

【原文事实】PDF p.26，Table 12。题注译意：与表2对应的大视角 SSIM/LPIPS，DepthSplat 先验。

**(a) SSIM ↑**

| 方法 | CO3D R/1 | WRGBD Sh/1 | Sh/3 | Mip360 R/1 | R/3 | DL3DV S/1 | S/3 | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| DepthSplat | .170 | .171 | .361 | .054 | .283 | .067 | .408 | .216 |
| MotionCtrl | .461 | .412 | - | .284 | - | .318 | - | .369 |
| CameraCtrl | .456 | .382 | - | .249 | - | .307 | - | .349 |
| ViewCrafter | .384 | - | .231 | .222 | .190 | .271 | .229 | .255 |
| SEVA | .428 | .382 | .455 | .235 | .318 | .543 | .390 | .393 |
| GenFusion | .346 | .302 | .370 | .190 | .253 | .262 | .390 | .302 |
| Difix3D | .182 | .170 | .337 | .077 | .248 | .089 | .388 | .213 |
| Ours | .534 | .442 | .507 | .276 | .352 | .370 | .480 | .430 |

**(b) LPIPS ↓**

| 方法 | CO3D R/1 | WRGBD Sh/1 | Sh/3 | Mip360 R/1 | R/3 | DL3DV S/1 | S/3 | Avg |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| DepthSplat | .797 | .756 | .577 | .807 | .609 | .828 | .538 | .702 |
| MotionCtrl | .616 | .723 | - | .817 | - | .720 | - | .719 |
| CameraCtrl | .643 | .676 | - | .701 | - | .648 | - | .667 |
| ViewCrafter | .659 | - | .729 | .647 | .676 | .602 | .645 | .660 |
| SEVA | .583 | .579 | .427 | .618 | .432 | .543 | .388 | .510 |
| GenFusion | .715 | .703 | .648 | .738 | .663 | .657 | .547 | .667 |
| Difix3D | .785 | .749 | .506 | .744 | .477 | .749 | .382 | .627 |
| Ours | .504 | .556 | .390 | .585 | .386 | .510 | .333 | .466 |

### 数值比较、设置与边界

【数值】Ours 平均 .430/.466 vs SEVA .393/.510；DL3DV S/1 SSIM .370 <SEVA .543，Mip360 R/1 .276 <MotionCtrl .284；LPIPS 相对 SEVA 七列都降低。说明几何与感知指标有各自失败样本，不能仅以 PSNR 判断。

【边界】大视角低重叠不是固定角度，结果依赖具体输入先验、目标场景及分割。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-12 -->

<!-- PTH:BEGIN TABLE-13 -->
## Table 13 长轨迹 SSIM/LPIPS

### 题注、指标及数据

**Caption 原文**（PDF 转写）：Table 13 Long trajectory. The long trajectory NVS measures the long-term generation ability of generative NVS. P denotes the number of reference images. We use VGGT [65] as the geometry prior for our method.

【原文事实】PDF p.26，Table 13。题注译意：长轨迹 NVS 的 SSIM/LPIPS，VGGT 先验。

**(a) SSIM ↑**

| 方法 | Mip360/3 | /6 | /9 | DL3DV/3 | /6 | /9 | T&T/2 | /3 | /6 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| DepthSplat | .277 | .292 | .310 | .415 | .464 | .490 | .474 | .530 | .552 |
| VGGT | .305 | .351 | .383 | .466 | .515 | .537 | .718 | .809 | .860 |
| SEVA | .302 | .326 | .343 | .384 | .442 | .479 | .534 | .614 | .721 |
| Ours | .341 | .372 | .395 | .485 | .559 | .607 | .628 | .712 | .763 |

**(b) LPIPS ↓**

| 方法 | Mip360/3 | /6 | /9 | DL3DV/3 | /6 | /9 | T&T/2 | /3 | /6 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| DepthSplat | .644 | .593 | .560 | .547 | .494 | .464 | .376 | .301 | .259 |
| VGGT | .659 | .633 | .645 | .466 | .469 | .488 | .193 | .126 | .124 |
| SEVA | .479 | .417 | .378 | .400 | .315 | .262 | .241 | .175 | .112 |
| Ours | .341 | .370 | .329 | .343 | .253 | .206 | .197 | .137 | .105 |

### 数值比较、设置与边界

【数值】Ours 对 SEVA 九列 SSIM 均升、LPIPS 均降；相对 VGGT，T&T 三列 SSIM 均低，LPIPS P=2 .197>.193、P=3 .137>.126。Mip360 P=6 Ours LPIPS .370 高于自身 P=3 .341，非所有指标随参考数严格单调。

【边界】轨迹 NVS 指标按生成帧比较，时序无闪烁须结合视频或专门时序指标；静态感知均值不能证明连续几何。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END TABLE-13 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 关系

### 方法图与公式映射

【原文事实】Fig.2 建立 input-level vs feature-level 差异，Fig.3 的三步对应 Eq.(4)、Eq.(3)/(5)/(6)、Eq.(7)–(9)，网络训练与 Eq.(10) 对应。Fig.4 展示两种融合，Fig.5 展示去噪过程中被调制的表示。Fig.14/15 是 Fig.3 内部模块细化；Fig.16 与 Algorithm 1 对应训练数据。

图11 的强度扫描对应 §4.2 输入注入初始化；图12 对应 Eq.(4) 的多高斯贡献，图13与表C.3 对应 Eq.(5)–(6) 信息损失补偿；C.4 检验 CFG 而非门控本身。

### 实验、消融与反例证据链

【证据映射】表1–3、11–13 与图6/17–22 衡量像素与感知；表4、表3b 与图9/25–27 衡量生成视频的估计几何；表5/6 对应图23/24 的几何先验替换及骨干集成；表7 对应图3–5、14/15 的模块消融；图8 衡量效率、表8 无位姿。

【独立评价】受控实验与定性图是互补证据：例如 Table 4 Mip360 CD 持平，图25反例CD升高；表7 T&T input-level PSNR 更高；图10/28 显示局部结构或纹理失败。必须保留这些反例，不能用平均改善代替逐项判断。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 问题

### 输入输出与任务约束

【原文事实】§3.1 p.4：输入 $\mathbf I_{\rm ref}\in\mathbb R^{P\times hw\times3}$、参考相机 $\boldsymbol\pi^{\rm ref}$、指定目标相机 $\boldsymbol\pi^{\rm tgt}$；输出 $\mathbf I_{\rm tgt}\in\mathbb R^{Q\times hw\times3}$。论文将指定相机下的目标图像生成作为任务，不以输出唯一三维模型为终点。几何模型生成的 $\mathcal G$ 是结构先验，中间的目标特征 $\hat{\mathbf F}_{\rm tar}$ 是去噪网络内部量，不是最终 RGB。

【背景知识】从像素射线 $\mathbf r(\lambda)=\mathbf o+\lambda\mathbf v$ 看，单图只能给出方向与外观，多个不同深度可在同像素投影，遮挡表面从输入完全不可见；多图对应增加约束但低重叠时匹配不足。给定目标相机使投影位置可定义，并不消除未知三维结构或未知纹理的歧义。

### 稀疏观测的歧义及论文任务边界

【原文事实】§1 提出矛盾：前馈几何擅长输入可见区域的一致结构，却难以合成低重叠的精细外观；纯生成模型利用学习先验补全内容，却可能改变物体布局或不遵循相机。本文核心问题是在生成器内如何传递并选择可信几何结构，而非从头求解稀疏重建。

【合理推断】对同一场景条件分布 $p(I_{tar}|I_{ref},\pi)$ 可包含多种合理图像；几何约束能缩小支持集，但对不可见区域不构成真实信息的充分条件。任务评价分像素/感知、估计几何、相机控制；不能把其中任一当成全部。CameraCtrl 单参考限制、pose-free 扩展仍需相机估计、静态场景假设及稀疏高斯薄结构失败均界定适用范围（§4.2/4.6/H）。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 动机

### 重建与生成各自失败机制

【原文事实】§1 p.2–3，NeRF/3D-GS per-scene optimization 在密集多图下有效，但稀疏图欠约束；前馈 3D 模型通过跨数据集学习的先验快速预测几何，而低重叠下真实细节或遮挡不能充分观测。生成模型在大量图像/视频学习外观与语义先验，在低重叠下可能生成看似真实却不遵循目标相机的结构。

【机制】重建类显式空间结构在不同目标视角使用同一组几何，故可见区域有共同几何来源；但几何错误也会稳定地传播。生成类相机参数是条件，不等同于硬投影约束；即使每帧合理，连续帧中也可能产生不一致物体尺度、形状、相对位置。Fig.1、2 示范此问题；Table 1–4 分别量化像素和间接几何。

### 为什么输入级颜色注入不充分

【原文事实】Fig.2(b) 的现有输入级做法把几何渲染的新视角图像或特征作为生成器输入；作者认为渲染颜色噪声掩盖结构。§4.2 对照把渲染 RGB 编成 latent 初始化去噪。若 s 小，生成器受该有噪 latent 约束强，难以改正；若 s 大，几何条件变弱。C.1 的扫描支持这个困境。

【合理推断】将输入级渲染视为图像条件，会同时传递几何和 appearance；几何位置正确不代表 SH 颜色估计正确。反之，如果直接使用每步的参考扩散特征，高斯起跨相机对应的载体作用，可减少颜色预测误差直传。但仍有几何偏差、线性特征混合损失，故需要精炼和门控。

【证据边界】这些是作者对具体基线失败的解释而非普遍定理；Table 7 T&T 输入级比 full PSNR 高，说明可靠颜色/几何下输入级仍可有效。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 相关工作

### 几何、生成与组合方法

【原文事实】§2 p.3–4 将相关工作分为三类。稀疏 NVS：NeRF [52]、3D-GS [35] 每场景优化；深度监督 [12,13,64,93] 和法线一致 [62,87] 增加约束。MVSplat [10]、HiSplat [63]、DepthSplat [80] 直接输出高斯；VGGT [65]、Pi3 [70] 等几何基础模型先预测点云/相机，再与 InstantSplat [17] 等结合。几何预测准确性决定目标渲染质量。

生成 NVS：MotionCtrl [71]、CameraCtrl [24]、SEVA [90] 等通过位姿、Plücker 坐标或轨迹注意力条件生成；前者与相机控制相关，SEVA 是本文主要基线。组合路线包括 MVSplat360 [11]、GenFusion [76]、Difix3D [74] 等，利用几何渲染与生成修复，具体测试配置见附录B.2。本文区分输入级条件和内部特征调制。

【边界】这些是本文对相关工作分类及实验配置的陈述；未独立阅读其他论文，不补写其结构或历史优先权。

### 动态特征提升与静态特征蒸馏区别

【原文事实】Feature field distillation：LangSplat 等 [42,54,91] 将静态语义特征蒸馏至高斯，通常每场景优化；Dr.Splat [33] 前馈特征蒸馏用于定位；LUDVIG [49] 将 DINOv2 特征结合场景几何用于分割。本文借鉴直接 lifting，不在每个去噪步启动新的特征场优化；区别在特征随 t 变化且需要像素级保真。

【合理推断】语义分割可接受同一实例的许多像素共享特征，而 NVS 的纹理、边缘、细节依赖局部高频差异。故仅保持类别语义不足以保证生成质量，C.3 的受控 latent 实验检验混合损失及 RefineNet 补偿。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-01 -->
## 几何先验

### 几何模型输出及相机对齐

【原文事实】§3.1 p.4、附录B.2 p.16–17：给定参考图及相机，几何先验最终为集合 $\mathcal G=\{(\mu_i,r_i,s_i,\sigma_i,c_i)\}_{i=1}^N$。MVSplat/HiSplat/DepthSplat 直接回归高斯，需要 posed images；VGGT/Pi3 输出点云与相机，先按 Eq.(1) 对齐尺度，再初始化高斯，InstantSplat 的共视冗余去除与仅参考图像优化用于构造结构。颜色参数存在于原高斯，但 adapter 不把高斯 RGB 作为目标扩散输入。

相机对齐以已对应的预测和目标 camera centers 为输入，只估计标量 $s^*$（详见 MATH-01），不是一般 Sim(3) 或任意外参优化。单参考需要基准中的 unit length sweep（Table 1）。pose-free 用 ViPE 提供估计位姿（Table 8）。

### 高斯拟合和固定结构的作用

【机制】高斯位置、旋转、尺度决定投影密度；opacity 与遮挡决定沿射线的可见贡献。固定这一结构便得到 reference 和 target 的渲染权重。每个 t 变化的是高斯上的扩散特征值，而不是重新拟合原几何位置。三维结构本身提供多个二维视角之间的中介。

【边界】输入位姿误差、尺度误差、高斯欠覆盖、薄结构或动态物体都会使结构先验失真。对 VGGT/Pi3 的高斯构建包含拟合，不能笼统说整个系统完全一次前馈或不需要 per-scene 优化。论文的 lifting 是每去噪步直接计算，与前置几何拟合是不同阶段。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END METHOD-01 -->

<!-- PTH:BEGIN METHOD-02 -->
## 提升与渲染

### 跨视角加权投影信息流

【原文事实】§3.2 p.5–6：输入 $\mathbf F^t_{ref}$、参考相机与高斯集合；目标是 $N$ 个 C 维特征 $\hat{\mathbf f}_i$，再以目标相机渲染 $\mathbf G_{tar}^t\in\mathbb R^{Q\times h/n\times w/n\times C}$。

1. 参考像素 $(d,p)$ 经高斯投影得到沿深度排序的 $w_i(d,p)$（Eq.2），对每个高斯建立贡献像素集合 $\mathcal I_i$。
2. 按 Eq.(4) 将该高斯从所有参考像素获得的特征加权累积，按其总权重归一化，再 L2 归一化。不是对单个像素的高斯权重总和归一化；分母固定高斯 i、遍历参考视图和像素。
3. 在参考和目标相机上以 Eq.(3) 重新投影，按各视角自己的权重合成 C 维特征。参考结果给监督，目标结果提供条件。

每步重算特征，而结构可复用；与逐场景优化特征的差别是没有每个 t 的额外迭代求解。

### 权重、可见性及信息损失

【机制例】构造一个高斯 i 在两参考像素的权重 0.8/0.2、特征 $(1,0),(0,1)$，则未 L2 归一化特征 $(0.8,0.2)$，归一化后约 $(0.970,0.243)$。在新视角该高斯的渲染权重可能为 0.6，与另一个高斯按 Eq.(3) 合成。reference 和 target 权重来自相机各自的投影与遮挡，并不相等。

【根因】软分配保留多个高斯贡献（Fig.12），但若两个不同表面错误落在同一 Gaussian 支持域，平均会混合细节；单位化消除幅度，重渲染也会发生线性混合。故从 $F_{ref}$ 到 $G_{tar}$ 不是可逆变换。未知/零可见高斯如何处理分母为零、数值 epsilon 等实现细节未明确，不能臆造。

【证据】Fig.5 表现中间特征，Fig.12 比较 k，C.3 检验混合损失。

<!-- PTH:END METHOD-02 -->

<!-- PTH:BEGIN METHOD-03 -->
## 精炼

### GS-PE 与 RefineNet 的输入输出

【原文事实】§3.2 p.6–7；Fig.15：输入重渲染 $\mathbf G_{ref}$ 和 $\mathbf G_{tar}$。为每个像素找 $i^*=\arg\max_i w_i(d,p)$；用 scene bounding box 归一化该高斯中心 $\tilde x,\tilde y,\tilde z$，另取其贡献 $w^*$，对四变量编码再相加（Eq.5）。C=64 时每变量 16 维、共 64 维，保证可与 $G$ 加法。位置编码指示主要结构来源而不是将每个目标像素的真实三维位置作为输入。

共享精炼网络 $\mathcal R$ 使用轻量 ResNet（Fig.15），输入 $G'$、输出 $\hat G$，保持参考/目标分支同一通道数。图示线性层与多层残差卷积（C→2C→2C→C，GELU）；未给出所有 kernel 与参数。

### 参考监督及不可见区域限制

【机制】训练时参考图在扩散内部已有可靠特征 Fref，重渲染后对其方向做 cosine alignment；这是通过同一几何映射往返后的自一致监督。目标图的精炼采用相同 R，但没有直接将目标真实图的特征传入推理分支。参考 loss 约束的是重渲染特征方向；latent loss 还训练最终生成任务。

【边界】原参考特征也来自网络，不是真实物理几何标签；cosine 对正比例幅值不敏感。精炼不能保证从混合后的特征唯一逆推原信息。图13/C.3 受控实验提供恢复保真的证据；图10/28 表明仍可能模糊。

<!-- PTH:END METHOD-03 -->

<!-- PTH:BEGIN METHOD-04 -->
## 自适应融合

### 交叉注意力与有符号门控

【原文事实】§3.2 p.7，Fig.4。目标特征 $F_{tar}$ 和精炼结构特征 $\hat G_{tar}$ 均为 $Q\times h/n\times w/n\times C$。Naïve 在通道维拼接，MLP 2C→C 直接输出 $\hat F_{tar}$；Adaptive 将 $F_{tar}$ 作为 query、$\hat G_{tar}$ 为 key/value，得到几何注意力特征 $F^A_{tar}$。再拼接 $F_{tar}$ 与 $F^A_{tar}$，gate MLP 2C→1，Tanh 输出每像素 W；广播乘 C 维残差后与原目标特征相加。详细注意力 token 范围/头数未在原文明确。

【设计原因】在几何可信时，注意力和残差帮助目标生成特征向三维对应结构调制；若几何错误，原目标特征提供独立生成路径。Naïve 的 MLP 理论上也能学会忽略输入，但没有显式恒等残差路径，经验上对先验质量更敏感。

### 残差路径、置信相关性及反例

【构造示例】某像素 $F=(2,1)$、$F^A=(1,-2)$，若 W=0，输出 (2,1)；W=.5 时 (2.5,0)；W=-.5 时 (1.5,2)。因此 Tanh 允许有符号修正；$|W|$ 更接近注入强度而非概率置信。幅值不仅取决 W，还取决 $\|F^A\|$，小 W 不严格代表小总残差。

【原文事实】§3.2 测三反光场景 Pearson r≈−0.30（绝对门控与高斯不确定性），后期更强。门控的直接输入是特征，不是已给定的不确定性图。Table 5 支持几何先验替换的鲁棒性；T&T SSIM 并非 adaptive 最优。不能称为严格 uncertainty calibration 或保证不注入错误。

<!-- PTH:END METHOD-04 -->

<!-- PTH:BEGIN METHOD-05 -->
## 网络集成

### DPT 聚合及解码器跳连

【原文事实】§3.3、Fig.3/14：视频扩散 encoder 中尺度特征经投影卷积统一至 C'=320 通道，再上采样、通道变换到 C=64，DPT 式层级融合为 $F\in\mathbb R^{(P+Q)\times h/n\times w/n\times C}$。将参考与目标分开后由 GS-Adapter 修正目标特征，修正后的特征再降采样回各 decoder 分辨率，通过 skip connections 加入。图14标注的三级空间大小为 $(h/n,w/n),(h/(2n),w/(2n)),(h/(4n),w/(4n))$。

【机制】浅层保留局部空间细节，深层感受野更大；聚合使同一空间高斯对应提供跨尺度条件。每步重算高斯 feature 使随时间变化的网络分布仍对齐同一结构，而非将固定 VAE 渲染 latent 反复作为同一输入。

### LoRA、训练与两种基线适用性

【原文事实】训练新多尺度模块、GS-Adapter、冻结视频扩散的 attention LoRA（§3.3）。LoRA 对冻结权重提供低秩增量；SEVA rank/alpha 16，CameraCtrl 4；训练配置分别为384×384 与320×576，初始学习率3e-5 与1e-4，8张 A6000、总 batch size 8、约80h（§4.2）。联合损失 Eq.(10)。

【边界】骨干的 feature interface、训练超参数不同，本文验证两种具体骨干，不是无需训练的任意网络插件。Table7 LoRA only 的改善需作为对照，不能全归给几何。

<!-- PTH:END METHOD-05 -->

<!-- PTH:BEGIN METHOD-06 -->
## 训练数据

### Algorithm 1 逐步分析

【原文事实】§E p.22–23，Fig.16 / Algorithm 1：原视频约300帧，预处理 resize 480×270 或960×540，w2c 与 normalized intrinsics；目标每 clip 21帧。用相对平移 $\mathbf t_{ij}$ 与旋转 $R_{ij}$ 的组合距离建立 graph，选阈值 δ 保证每节点至少21邻居。按 node degree 加权取3 anchor；每 anchor 加邻居形成候选组。对相机位置和方向向量拼接做 K-Means，选择 P 个参考（1,3,6,9,12）。余者按与参考最小 CLIP cosine distance 划分 easy（底部60%）/hard；easy 用 K-Means，hard 随机；frustum IoU≥0.4 保证一定重叠，形成参考/目标对。图16显示150K clips。

【几何解释】位姿距离兼顾相机中心位移和朝向；K-Means 降低参考集中于同一路径局部的冗余；CLIP距离作为视觉难度代理并不是几何视角角度。IoU过滤剔除完全分离的视锥，但不保证真实表面共视。

### 训练先验与测试先验的差异

【原文事实】训练 VGGT 点云构造可包含目标图，InstantSplat 颜色/高斯拟合只用参考图（Fig.16 的 Include novel-views to construct point cloud only in training data / Exclude novel-views）。训练中几何中介可更完整；实际测试用参考图像估计先验（§3.1、附录B.2）。

【独立评价】该训练条件使模型可学习更好的几何特征，但也存在 geometry prior 的训练/测试质量差异；Table5/6 检验跨模型替换并不等同消除该差异。训练时总21帧，推理 SEVA 可支持多参考与目标、长轨迹沿用两遍采样；不能据“arbitrary”推断无限帧计算或不受显存限制。

<!-- PTH:END METHOD-06 -->

<!-- PTH:BEGIN MATH-01 -->
## Eq.1

### 最小二乘尺度推导

【原文事实】Eq. (1)，p.4：

$$s^*=\arg\min_s\left\|\mathbf C^{\rm gt}-s\mathbf C^{\rm pred}\right\|_2^2.$$

$\mathbf C^{\rm gt},\mathbf C^{\rm pred}$ 是对应相机中心（可以按相同顺序堆成向量），$s$ 为单一全局尺度。

【补充推导】令 $\mathbf a=\mathbf C^{\rm gt},\mathbf b=\mathbf C^{\rm pred}$，展开二次目标：

$$E(s)=\mathbf a^T\mathbf a-2s\mathbf a^T\mathbf b+s^2\mathbf b^T\mathbf b.$$

$$\frac{dE}{ds}=-2\mathbf a^T\mathbf b+2s\mathbf b^T\mathbf b=0.$$

若 $\|\mathbf b\|^2>0$，二阶导为 $2\|\mathbf b\|^2>0$，唯一无约束最小值

$$s^*=\frac{\langle\mathbf C^{\rm pred},\mathbf C^{\rm gt}\rangle}{\|\mathbf C^{\rm pred}\|_2^2}.$$

若矩阵形式则以 Frobenius 内积解释；例如预测相机中心向量 (1,2)，目标 (2,4)，解为 (2+8)/(1+4)=2。此是补充推导，不是原文另编号公式。

### 成立条件与歧义

【成立条件】需要对应相机、共同原点及坐标朝向（或已处理相应刚体变换），只有尺度未知。若预测全部中心为零，目标函数与 s 无关，尺度不可辨。若限定物理正尺度而内积不正，需要额外约束处理；Eq.(1) 未写正尺度约束或 epsilon。单参考若中心在原点，单靠中心无法解尺度；Table1 的 unit length sweep 是另一基准处理。不能将 Eq.(1) 说成自动估计旋转平移的 Sim(3)。

【几何作用】尺度错误使同一目标相机平移相对高斯大小不正确，改变投影/遮挡与 feature correspondence。对齐是高斯先验可换的前提。Pose-free 中目标参照来自 ViPE 估计（§4.6），依然依赖相机框架。

<!-- PTH:END MATH-01 -->

<!-- PTH:BEGIN MATH-02 -->
## Eq.2–3

### alpha 合成与特征渲染逐项推导

【原文事实】Eq.(2)–(3)，p.5：

$$C(d,p)=\sum_{i=1}^N c_i(d)w_i(d,p),\qquad w_i(d,p)=\alpha_i(d,p)\prod_{j=1}^{i-1}(1-\alpha_j(d,p)).$$

$$\mathbf F(d,p)=\sum_{i=1}^N\mathbf f_iw_i(d,p).$$

$d$ 为视角，$p$ 为像素，$c_i(d)$ 是 SH 表达的视角相关 RGB，$\alpha_i=\sigma_i\mathcal N_i(p)$ 为 opacity 乘投影二维密度。$\mathbf f_i$ 为每高斯 C 维 feature。高斯的中心、旋转和尺度决定投影椭圆。原文用 $\mathbf F(d,p)$ 泛指渲染特征；适配器后续记为 $\mathbf G$。

【背景推导】按前到后排序，令透射 $T_i=\prod_{j<i}(1-\alpha_j)$，$T_1=1$，递推 $T_{i+1}=T_i(1-\alpha_i)$，当前粒子可见贡献 $w_i=T_i\alpha_i=T_i-T_{i+1}$。求和望远镜消去：

$$\sum_{i=1}^N w_i=1-T_{N+1}\le1.$$

若 $\alpha_i\in[0,1]$，权重非负，余项对应未被前景吸收的透射。Eq.(2) 未写背景项，不能自行宣称作者固定背景色。

### 遮挡顺序与颜色、特征区别

【构造例】两高斯 $\alpha_1=.6,\alpha_2=.5$，则 $w_1=.6,w_2=.4\cdot.5=.2$，剩余透射 .2；若交换顺序，个体权重改变，因而遮挡顺序重要。RGB 渲染使用 $c_i(d)$，特征渲染使用 $\mathbf f_i$，共享同一组结构权重 $w_i(d,p)$。若前景完全不透明，后方贡献为零；若都透明则输出为零（未写背景）。

【成立条件】排序、投影和 opacity 正确时该混合有几何可见性解释。特征不必遵循 RGB 的物理 alpha 线性关系；C.3 明确指出 latent 某些 channel 与 opacity 可能无对应。该公式是特征渲染假设，不是物理证明。颜色依赖视角，但本文每个去噪时刻的 $\hat f_i$ 为共享向量；不同视角通过权重产生差异。

<!-- PTH:END MATH-02 -->

<!-- PTH:BEGIN MATH-03 -->
## Eq.4

### 归一化加权提升及优化解释

【原文事实】Eq.(4)，p.6：

$$\hat{\mathbf f}_i=\frac{\mathbf f_i}{\|\mathbf f_i\|_2},\qquad \mathbf f_i=\sum_{(d,p)\in\mathcal I_i}\frac{w_i(d,p)}{\sum_{(d',p')\in\mathcal I_i}w_i(d',p')}\mathbf F_{\rm ref}(d,p).$$

$\mathcal I_i$ 固定第 i 个高斯、枚举所有参考视图中对它有贡献的像素。令 $S_i=\sum_{(d,p)\in\mathcal I_i}w_i(d,p)$。$S_i>0$ 时 $\beta_{i,d,p}=w_i/S_i$ 的和为1；加权平均得到 $\mathbf f_i\in\mathbb R^C$，再单位化得 $\hat{\mathbf f}_i$。

【补充推导】在固定结构权重且无正则时，该未单位化均值是如下加权最小二乘的最优解（仅是等价解释，非论文宣称额外优化）：

$$E_i(\mathbf z)=\sum_{(d,p)\in\mathcal I_i}w_i(d,p)\|\mathbf z-\mathbf F_{\rm ref}(d,p)\|^2.$$

$$\nabla_{\mathbf z} E_i=2S_i\mathbf z-2\sum w_i\mathbf F_{\rm ref}=0\quad\Rightarrow\quad\mathbf z=\mathbf f_i.$$

Hessian 为 $2S_iI_C$，$S_i>0$ 时唯一。L2 单位化是额外步骤，不是这个未约束最小二乘的最优值。

### L2 归一化与零分母限制

【两种归一化】Eq.(2) 的合成权重在固定像素/视角时遍历高斯；Eq.(4) 的分母固定高斯而遍历参考像素。这是从 3D→2D 的投影及 2D→3D 的累积两种方向。若将 reference 权重写成矩阵 A（像素数 M×高斯数 N），忽略 L2 单位化，lifting 为 $D^{-1}A^T F_{ref}$，其中 $D=\operatorname{diag}(A^T\mathbf 1)$；target rendering 则为 $A_{tar}D^{-1}A^TF_{ref}$。这不是求逆投影，多个像素信息会混合；单位化更消除幅值。

【边界及梯度】$S_i=0$ 或 $\|\mathbf f_i\|=0$ 时原公式未定义，实际 mask/epsilon 细节未证实。若固定权重，$\partial f_i/\partial F_{ref,p}=\beta_{i,p}I_C$；单位化 Jacobian 为 $(I-\hat f_i\hat f_i^T)/\|f_i\|$（非零时），所以梯度在径向分量被抑制。论文端到端训练与这种可微路径一致，但不据此宣称训练时更新先验 Gaussian 几何。Fig.12 说明 top-k 从1增至255，软分配收益较小但方向支持。

<!-- PTH:END MATH-03 -->

<!-- PTH:BEGIN MATH-04 -->
## Eq.5–6

### 四组位置编码及余弦监督推导

【原文事实】Eq.(5)，p.6：

$$\mathbf G'(d,p)=\mathbf G(d,p)+[\gamma(\tilde x)\|\gamma(\tilde y)\|\gamma(\tilde z)\|\gamma(w^*)],$$

$$\gamma(v)=[\sin(\omega_kv),\cos(\omega_kv)]_{k=0}^{D'/2-1},\quad \omega_k=\omega_0^{-2k/D'},\quad D'=C/4.$$

先找 $i^*(d,p)=\arg\max_i w_i(d,p)$，取其中心在 scene bbox 中归一化后的坐标 $\tilde{\mathbf x}$ 与该高斯权重 $w^*$。每变量 D' 维，四变量拼成 C 维，才能与 C 维渲染特征逐元素相加。C=64 时每变量16维，即8个频率的 sin/cos。原文未给 $\omega_0$ 的值，不能套用 Transformer 的 10000。坐标归一化确切区间和边长为零处理未明确。

【背景解释】正弦编码以多频率区分位置，权重编码提供可见性线索；若两个像素渲染特征相似但主要贡献 Gaussian 的位置不同，编码可使 R 识别不同结构来源。$\hat{\mathbf G}=\mathcal R(\mathbf G')$。位置编码只取最大贡献粒子，而 lifting 是多个粒子软分配，两者不要混淆。

### 参考约束与目标泛化边界

【原文事实】Eq.(6)，p.7：

$$\mathcal L_{\rm feat}=\left\|1-\frac{\hat{\mathbf G}_{\rm ref}\cdot\mathbf F_{\rm ref}}{\|\hat{\mathbf G}_{\rm ref}\|_2\|\mathbf F_{\rm ref}\|_2}\right\|_2.$$

【补充推导】对一对非零向量 a,b，$\cos\theta=a^Tb/(\|a\|\|b\|)$，由 Cauchy–Schwarz 落在[-1,1]；方向相同 loss 0，正交 loss 1，反向 loss 2。单位向量 $u=a/\|a\|,v=b/\|b\|$ 满足 $\|u-v\|^2=2-2u^Tv$，故 $1-\cos\theta=\frac12\|u-v\|^2$。外层 L2 的具体跨像素归约实现论文未明确，不改写成必然均值。若 b 固定，$\nabla_a(1-\cos\theta)=-(v-(u^Tv)u)/\|a\|$，说明方向对齐且正尺度不变；不能恢复绝对幅值。零向量需数值保护，论文未写。

【训练意义】参考视图的原扩散特征提供可观测监督；R 与目标共用，从参考路径学习恢复损失特征。目标不可见区域没有直接同类 reference 对应，不能保证泛化。Fig.13/Table C.3 支持恢复方向，Table7 验证模块增量。

<!-- PTH:END MATH-04 -->

<!-- PTH:BEGIN MATH-05 -->
## Eq.7–9

### 拼接、注意力和门控张量推导

【原文事实】Eq.(7)–(9)，p.7：

$$\hat{\mathbf F}_{\rm tar}=\mathrm{MLPs}(\mathbf F_{\rm tar}\oplus\hat{\mathbf G}_{\rm tar}).\tag{7}$$

$$\mathbf F_{\rm tar}^{A}=\mathrm{CrossAttn}(\mathbf F_{\rm tar},\hat{\mathbf G}_{\rm tar}),\qquad \mathbf W_{\rm tar}=\tanh(\mathrm{MLP}(\mathbf F_{\rm tar}\oplus\mathbf F^A_{\rm tar})).\tag{8}$$

$$\hat{\mathbf F}_{\rm tar}=\mathbf F_{\rm tar}+\mathbf W_{\rm tar}\cdot\mathbf F_{\rm tar}^{A}.\tag{9}$$

设目标 token 数 L=Q·h/n·w/n（是否全局 attention 未明确，仅为总数）。原图中拼接 2C→C，gate 2C→1。以一个 attention group 为例，标准 scaled dot product 的背景表达为 $Q_a=FW_Q,K_a=\hat GW_K,V_a=\hat GW_V$，$A=\operatorname{softmax}(Q_aK_a^T/\sqrt{d_k})V_a$，再输出映射到 C 维。这是解释 cross-attention 的一般形式，论文未给具体 heads、token grouping 或参数矩阵细节。目标 F 的 query 决定从 geometry-aware K/V 中读取何种信息。

### 门控符号、残差和相关性解释

【补充推导】对像素 j，输出为 $\hat F_j=F_j+w_j A_j$。$w_j=0$ 时恒等路径；$w_j=1$ 加完整 A；$w_j=-1$ 减 A。$\|\hat F_j-F_j\|=|w_j|\|A_j\|$，故门控幅度不是归一化概率或可靠度本身。Tanh 的导数 $1-\tanh^2z$ 在饱和处变小；在 $w=0$ 附近仍可学习，且原 F 有直接残差梯度（其他梯度通过 A,w）。Naïve MLP 理论也能学习恒等，但结构不保证，Fig.4/Table5显示经验差异。

Pearson $r=\mathrm{Cov}(|W|,U)/(\sigma_{|W|}\sigma_U)$（背景定义），论文三反光场景 r≈−.30，表示 U 大的区域倾向较小 |W|，不等同因果或严格校准；实际注入还取决于 A 的尺度。未证实门控能完全消除所有错误几何。

<!-- PTH:END MATH-05 -->

<!-- PTH:BEGIN MATH-06 -->
## Eq.10

### 联合损失与训练梯度路径

【原文事实】Eq.(10)，p.7：

$$\mathcal L=\mathcal L_{\rm latent}+0.05\mathcal L_{\rm feat}.$$

$\mathcal L_{\rm latent}$ 为论文引用 [3] 的 latent space alignment loss，具体 diffusion parameterization 或公式未在本文展开；$\mathcal L_{\rm feat}$ 是 Eq.(6) 参考 feature alignment。0.05 为训练超参数，不等于特征损失占总梯度5%。

【补充推导】对可训练参数 θ（适配器、DPT 模块、LoRA），有 $\nabla_\theta L=\nabla_\theta L_{latent}+0.05\nabla_\theta L_{feat}$。两项实际梯度大小依赖损失尺度及网络 Jacobian。参考分支损失经共享 R 及特征 lifting 路径影响训练，目标生成质量经 $\hat F_{tar}$ 回到 decoder 和 adapter；冻结 backbone 的原始权重不更新，LoRA 的低秩增量更新。

### 目标函数和权重解释

【机制】若仅最小化生成 latent loss，可能学到通过补偿外观绕过几何而非保存参考 feature；加入 Lfeat 直接约束重渲染后的参考表示。若 λ=0，只剩最终生成目标；λ 极大则可能过重保留参考特征而牺牲任务目标，这只是一般权衡，本文未扫描 λ。

【证据】Table7 分别改变 Lfeat 与 GS-PE，PSNR 有增益但没有 λ 灵敏度/随机种子方差，不能声称 0.05 全局最优。论文没有把参考 feature loss 当真实 3D 几何监督。

<!-- PTH:END MATH-06 -->

<!-- PTH:BEGIN MATH-07 -->
## 附录及指标

### 补充公式、指标与 Algorithm 1

【原文事实】Algorithm 1 p.23，位姿距离：

$$d(i,j)=\sqrt{\|\mathbf t_{ij}\|^2+2\left(1-\frac{\operatorname{tr}(\mathbf R_{ij})}{3}\right)}.$$

$\mathbf t_{ij},\mathbf R_{ij}$ 为相对位姿平移和旋转。补充推导：对合法三维旋转角 θ，$\operatorname{tr}R=1+2\cos\theta$，故旋转项为 $\frac43(1-\cos\theta)=\frac83\sin^2(\theta/2)$。当 θ=0 为0，θ=π 为8/3；小角度近似 $(2/3)\theta^2$。位移项与旋转项合并是用于邻近视角选择的设计量，依赖相机平移归一化尺度，并非物理统一量纲。算法按 δ 阈值建图、degree 加权采 anchor、K-Means 参考、CLIP easy/hard、frustum IoU 过滤（METHOD-06）。

【原文事实】§4.2 输入级注入：

$$\mathbf x_{tar}\leftarrow\mathbf x\,\sigma_{t_0}+\mathbf z_{tar},\qquad t_0=\lfloor Ts\rfloor,\quad s\in[0,1].$$

$z_{tar}$ 是高斯渲染图 VAE latent，x 为噪声，$\sigma_{t_0}$ 噪声尺度，s 越大给予扩散自由度越多。实验 s=.2，Fig.11 扫描；不能与 Eq.(1) 的尺度 s 混淆。

### 符号、估计误差和实验条件

【背景指标】PSNR（dB）在像素值范围 L 下为 $10\log_{10}(L^2/\mathrm{MSE})$，提高 Δ dB 对应 MSE 比 $10^{-\Delta/10}$（相同范围与计算条件）。例如 Table1 平均 dB +1.94 对应单次同口径 MSE 比约 .64；不能把平均 dB 的差直接当总体 MSE 减少率。SSIM 衡量局部结构/亮度/对比度，LPIPS 为学习感知距离。论文没有给具体 SSIM/LPIPS 实现参数。

§4.4：ViPE 重建生成及 GT 视频，平移误差 Terr [cm]、旋转 Rerr [deg]；CD 衡量估计点云差异。常见对称 CD 以双方最近邻距离的均值和定义（距离是否平方、归约细节原文未明确，不将通用定义冒充该实现）。Oracle 是从 GT 目标图重建，不是完美真值。排除 GT 重建 T 或 R>50 的场景。PSNRV/U 的掩码由 VGGT 输入点云投影；其遮挡判定及点密度影响区域划分。FOV IoU 与该像素共视掩码不同：前者用于训练视角过滤，后者用于测试区域误差。

【边界】各指标反映不同方面，不能从 PSNR 推断真实几何唯一性；不同数据划分、单位、pose normalization 会改变比较含义。

<!-- PTH:END MATH-07 -->

<!-- PTH:BEGIN EXP-01 -->
## 实验协议

### 数据、划分、训练及推理设置

【原文事实】训练 DL3DV-10K，VGGT/InstantSplat 构造 150K clips；每组21帧，(P,Q)=(1,20),(3,18),(6,15),(9,12),(12,9)，Fig.16 的目标图仅在训练点云阶段参与。训练 SEVA/Ctrl 的分辨率384×384/320×576、初始 LR 3e-5/1e-4、LoRA rank/alpha 16/4，8×A6000(48GB)、batch8、约80h。测试单 A6000。

§B 列九基准：OmniObject3D、RealEstate10K、LLFF、DTU、CO3D、WRGBD、Mip-NeRF360、DL3DV、Tanks and Temples。Split R=ReconFusion,D=4DiM,S=SEVA,P=pixelSplat,V=ViewCrafter,L=LongLRM,O=CF-3DGS 适配；WRGBD-Se/Sh 依原表标签保留。Small/Large Set NVS 由参考到目标 CLIP 距离0.11划分，任意顺序目标；trajectory NVS 用平滑相机轨迹、SEVA two-pass sampling。P=1 时基准扫描 unit length 处理 scale ambiguity。

### 对比公平性与输入限制

【原文事实】附录B.2 的实际比较设置：直接回归高斯 MVSplat/HiSplat/DepthSplat 需要 posed images；VGGT/Pi3 预测点云和相机，尺度对齐后用参考图优化高斯。ViewCrafter 原来探索式无目标位姿，本评估提供 GT poses 并使用 P reference +5 extra viewpoints 估计几何，再沿 GT trajectory 渲染后修复；作者认为更有利，不能理解为所有方法完全相同的输入。MotionCtrl 仅 CMCM、25步、576×1024→384×384；CameraCtrl 官方 SVD、25步、320×576→384×384；SEVA v1.0、50步、CFG2、384×384；MVSplat360 特定 MVSplat feature 输入 SVD；GenFusion 不执行完整原三维流程，改用 geometry prior RGBD render 再 refine，320×512→384×384；Difix3D 用目标渲染加按 camera-center distance 选最近参考。

【边界】组合模型尽可能使用按基准最优的同一先验（表9），MVSplat360 例外。不同分辨率、采样步数、模型适配方式与输入约束影响比较；结论针对论文实际测试协议，不是全面重跑其他论文。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END EXP-01 -->

<!-- PTH:BEGIN EXP-02 -->
## 小/大视角与轨迹

### Table 1–3 及 11–13 量化对比

【原文事实】Small viewpoint 18 个设定 PSNR 平均：GeoNVS19.09、SEVA17.15、VGGT16.76、Difix3D16.89；相对 SEVA 数值比例+11.3%，绝对+1.94 dB。SSIM .622 vs .560，LPIPS .253 vs .293（表1/11）。Large viewpoint 7设定 14.20 vs12.64，SSIM .430 vs .393，LPIPS .466 vs .510（表2/12）。长轨迹表3a/13：三数据集共9输入设置，Ours 对 SEVA 的 PSNR、SSIM 均升，LPIPS 均降。例 DL3DV/3,6,9：PSNR 16.03/17.99/19.32 vs14.11/15.65/16.73。

【机制】高重叠提供可利用的参考对应，显式三维 feature 可降低生成偏离。低重叠纯几何缺覆盖（表2 DepthSplat WRGBD1 7.31），生成器补全区域但可能漂移，adapter 约束已知结构。长轨迹使用一致几何先验和多步生成，减少结构累积偏差，但 two-pass sampling 继承自 SEVA。

### 失败与图像定性对应

【反例】T&T 纯 VGGT 在多个高重叠和轨迹设置的 PSNR/SSIM 高于 Ours；图6/7 只展示选中样本；表12 DL3DV 单图 SSIM .370 < SEVA .543。图10 细节模糊，图28 薄结构/遮挡失败。故应表述“相对同一生成基线平均及 PSNR 多组改善，优于许多对照”，不是“每一场景和每一指标 SOTA”。

【独立评价】表中不同方法缺失列使 Avg 不总可比，单视角方法只在单图子集比较。没有报告置信区间，不对统计显著性下结论。感知指标不等同严格几何；与表4独立验证结合。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END EXP-02 -->

<!-- PTH:BEGIN EXP-03 -->
## 几何

### Table 4、Figure 9/25–27 结果

【原文事实】§4.4 用 ViPE 估计生成/GT 的相机及点云。Table3b RE10K/3 T 1.62→.76 cm、R5.84→4.83 deg、CD1.35→.19；DL3DV/3 35→28.3、6.44→5.13、1.03→.62；T&T/3 6.08→3.56、2.87→1.52、1.42→.92。Table4 三场景同一 VGGT 先验，输入级/ Difix3D 的 T/R 大幅恶化；Ours T/R 优于 SEVA，Mip360 CD 同为.07（显示精度），其余降低。图9/25/26/27 展示间接重建及轨迹，图25有 CD .167→.329反例。

### 非共视、CD 与相机估计限制

【原文事实】PSNRV/U 用仅 input-view VGGT 点投影的可见掩码；非共视区域 Ours 对 SEVA +1.06/+2.08/+1.72 dB，最大 +2.08在DL3DV；共视+1.26/+2.43/+1.80。显式结构约束可影响生成网络更广泛区域，不仅目标渲染已覆盖像素。

【证据边界】GT 重建失败（Terr 或 Rerr>50）排除；oracle 重建来自 GT 视图，不是真实三维测量。尺度、点云采样及 ViPE 的估计误差可能影响 CD，巨大 CD 不宜视为物理距离直接比较。非共视掩码是估计覆盖，不证明不可见真实结构唯一恢复。因果上输入级颜色噪声是作者解释；实验比较完整路径，未完全隔离全部因素。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END EXP-03 -->

<!-- PTH:BEGIN EXP-04 -->
## 特征

### Figure 5/11–13/C4 及附录 C

【原文事实】§4.5 Fig.5 中早期 Ftar 空间无序，结构渲染仍清晰，精炼恢复部分细节，融合后逐步对齐。C.1 Fig.11 输入级 s 扫描，DL3DV/Mip360 的本文 PSNR18.11/16.57 均高于所测输入级；曲线局部波动不应写严格单调。C.2 Fig.12 k1→255 提升，两个数据集增量约.04/.10 dB；硬分配忽略部分贡献高斯，soft assignment 保持更平滑结构。

C.3 Fig.13/Table C.3 在540个 DL3DV 测试场景的 VAE latent 受控每场景优化：3/6/9输入的 refinement 增益 PSNR .24/1.03/1.37，SSIM .01/.02/.03；其目的是分离 feature splatting 的信息损失，不是完整生成模型。C.4 CFG=1,1.5,2 在10场景比较，Ours 17.69→18.16 vs SEVA14.74→15.84，变动较小。

### 逐步消融及指标解释

【机制与控制变量】输入级强度改变扩散起始噪声；top-k 改变 lifting 的贡献者集合；refinement 改变重渲染后的校正；CFG 改变生成条件强度。四者处于不同机制，不能用一个“置信度”概括。

【边界】C.3 结果证明有监督精炼在指定设置中提升重建 latent 解码指标，不证明潜域 alpha 合成严格物理有效或无损。C.1/C.4 只测有限参数/基准，k曲线缺误差条；Fig.5 的可视化不是逐维量化特征。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END EXP-04 -->

<!-- PTH:BEGIN EXP-05 -->
## 模块

### Table 5–7、Figure 23–24

【原文事实】Table5 固定 SEVA/3参考，替换 MVSplat、DepthSplat、VGGT、Pi3。naïve+Pi3 在 DTU14.83<SEVA17.03，adaptive+Pi3 19.02；adaptive 四先验 DTU19.02–19.18、CO3D19.22–19.26，较稳定。Table6 CameraCtrl 单参考只列 naïve，DepthSplat 组合 Avg12.80 vs11.14（+1.66 dB，14.9%）；其余先验亦提高平均但部分列下降。图23/24 说明局部先验错误和骨干差异。

【原文事实】Table7 固定 VGGT，原SEVA→LoRA only：Mip36015.38→15.55、DL3DV15.84→17.35、T&T18.85→21.41，表明训练本身贡献明显。三尺度 naïve 同时Lfeat、GS-PE 16.48/17.95/22.22，full adaptive16.57/18.11/22.24；去 RefineNet 16.23/17.71/21.91。

### 替换先验及不同基线的条件

【控制及限制】以相同几何先验、输入条件的不同 adapter 方案进行对比，能支持模块经验增量。不是全因子设计，无法完整分离交互效应；adaptive 的 T&T SSIM .69 不及 naïve .70。输入级 T&T PSNR23.05 > full22.24，但 Mip360/DL3DV full更高。不存在所有设置“几何先验越强、本文提升越大”的保证。

【独立评价】几何替换的 plug-and-play 证据较强，限这些四种输出可统一为 Gaussian 的模型；跨骨干是重新微调实现的兼容性，不是同一个权重免训练迁移。作者 r≈−0.30 的门控相关实验是三反光场景局部观察，不是校准评测。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END EXP-05 -->

<!-- PTH:BEGIN EXP-06 -->
## 效率与无位姿

### Figure 8、Table 8

【原文事实】Fig.8/§4.6 在10个 DL3DV 场景测 time、peak GPU memory、PSNR；高斯数量是主要 bottleneck。剪枝率96.2% 4.19→1.89 s/frame、18.03→12.93 GB，18.108 dB不变（图示精度）；SEVA1.34 s/frame、12.81GB、15.844 dB。此 2.22× 是 GeoNVS 剪枝前后，不是相对 SEVA 加速。

表8 pose-free 结合 ViPE input poses 与 VGGT point cloud，Eq.1 尺度对齐，仍需预测 intrinsics/extrinsics。DL3DV 6图 Ours GT17.99/ViPE17.87；T&T3图22.24/22.31。T&T ViPE 估计的姿态条件下，T降低但R升高；作者认为可能 COLMAP-derived pose 噪声，并未单独验证。

### 剪枝开销、SfM 条件和局限

【独立评价】减少粒子数会直接减少 lifting 与 rendering 负担，但剪枝是否保留薄结构依赖场景和体素规则。图8右侧内存横轴绘图范围与左侧不同，引用题注96.2%整体设置而不强行解释各点对应率。运行时间不一定包含完整前置几何估计，论文未逐阶段拆分，不据此报告端到端速度。

【边界】pose-free 不是无相机参数，而是由估计替代提供 GT；仅两个基准。若估计失败或尺度不准，显式约束可能放大错误；门控可缓和但非保证。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END EXP-06 -->

<!-- PTH:BEGIN SYNTHESIS-01 -->
## 综合

### 三项贡献及各自证据

【主张—证据矩阵】

| 主张 | 关键证据 | 支持及限制 |
|---|---|---|
| Feature-level geometry grounding 提升 NVS | §3 Eq.2–10、图2–5、表1–4/11–13 | 结构机制清楚，指定测试中相对 SEVA PSNR全面提高；非所有对照和指标领先 |
| 自适应 fusion 抑制错误几何 | 图4、表5/7、图23 | 换先验 PSNR变化小，r≈−.30 支持倾向；非概率校准、T&T SSIM例外 |
| RefineNet 缓解特征混合损失 | 表7、Fig.13/C.3 | 受控潜域+0.24/1.03/1.37dB；仍非可逆、细节模糊 |
| 更换几何先验和适配骨干 | 表5/6、Fig.23/24、§4.2 | 先验替换无需额外训练；两骨干各自微调，CameraCtrl单图 |
| 改善相机/几何 | 表3b/4、图9/25–27 | 估计T/R/CD整体改善；Oracle不是真实点云，CD有持平和反例 |
| 非共视/pose-free/效率 | 表4/8、图8 | 估计mask PSNRU最高+2.08dB、2数据集无GT位姿、剪枝前后2.22×；不能外推所有场景 |

【归纳】本文最核心创新是让固定三维结构充当动态扩散特征的跨视角中介，并以精炼及残差门控使错误和信息损失可被学习补偿。它不是提出新三维重建基础模型，也不是把渲染 RGB 直接送入扩散修复。

### 方法局限、失败及开放问题

【技术边界】① 先验错误仍可能传播，尤其薄结构；② 全遮挡/远离输入视角时信息缺失，生成只能推断；③ 加权平均、单位化、alpha mixing 有不可逆损失，可能模糊；④ pose/scale alignment 依赖估计质量，单图尺度有歧义；⑤ 适配器需要训练、特定 feature interface 和计算，不能称任意模型无需训练；⑥ 训练几何可见目标图，测试仅参考，存在分布差异；⑦ 静态可见性及颜色先验假设对其他域未经验证。

【尚未证实】未知区域的真实几何恢复、门控绝对可靠性、跨骨干直接权重迁移、任意场景剪枝保真以及视觉以外的物理准确性。作者 H 节提出 uncertainty-aware Gaussian/inpainting prior 未来方向，未实施。保留反例和数据条件，科学准确性比“全部领先”的表述重要。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END SYNTHESIS-01 -->

<!-- PTH:BEGIN SYNTHESIS-02 -->
## 终审

### 正文、附录及图表覆盖

【覆盖索引】正文 §1、2 在 PROBLEM/MOTIVATION/RELATED；§3.1–3.3 在 METHOD-01–05 与 MATH-01–06；§4.1–4.6 在 EXP-01–06；§5 在 CONCLUSION 与 SYNTHESIS-01。附录 A 在 Table10，B 在 EXP-01及Table9，C 在 Fig11–13/C4 与EXP04，D 在Fig14/15与METHOD03/05，E 在Fig16/Algorithm1/METHOD06/MATH07，F 在Table11–13，G 在Fig17–27，H 在Fig28及局限。29个图区域和14表均单元化；未编号C.3/C.4单列。参考文献用于定位本文引用，不独立阅读其他论文。

公式1–10及Algorithm1的距离、输入级初始化式均已对应数学单元，额外推导明确标为背景/补充。

### 来源、事实与推断的区分

【来源】PDF v2 为正式证据，HTML、Source 仅定位/核对公式与编号；同版 PDF 43页实际可读。记录图11/12边界文字碎片、19个带记录保真通过、HTML表号冲突、Source无独立版本戳、项目页访问失败。

【证据边界】作者声称的 SOTA、几何解释与 plug-and-play 不等同于普遍保证；已保留量化例外、训练测试条件和实现未知项。外部代码、模型权重仅核验身份，未运行或读代码作为科学依据；不声称客户端已显示、站点已部署、跨文献关系已建立。

本单元属于阅读定位或索引，不承担独立算法机制推导；相应方法将在专门单元展开。

本单元不承担独立公式推导；相关公式及成立条件在 MATH 单元完整分析。

<!-- PTH:END SYNTHESIS-02 -->



---

# 第二部分：学习与研究笔记

# GeoNVS 学习与研究笔记

> 精读报告 SHA-256：8c5239c940f2bb0b1ad9473e9bd5832640346d12c950185578da93d92f28bd42
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：关键概念内化

### 几何载体不是生成真值

【原文锚点】§3.1–3.2、Eq.(2)–(4)、Fig.2–3；精读报告 METHOD-01/02、MATH-02/03。

最有辨识度的思想是将 **空间支撑（geometry support）与随去噪步变化的信号（dynamic feature）分离**。前置高斯集合的中心、旋转、尺度、opacity 定义了视角相关的投影权重，却无需相信它预测的 RGB。把 reference features 放入同一组空间支撑后，目标视角仍用自己的投影权重读取。于是 3D-GS 可看成可见性加权的跨视角信息路由器，而不只是生成 RGB 的渲染器。

【构造示例】若两个相机都看见同一平面上的高斯，两处参考 latent 在正确对应下相似，平均特征可降噪；若投影错位到不同物体边缘，平均会把互不相关的信号合并。显式三维不能保证正确，错误空间支撑也会一致地传播错误。

### 提升与渲染构成非可逆映射

【补充推导】用 A 表示 reference 像素到高斯的 alpha 权重矩阵（M×N），B 表 target 的权重矩阵（L×N）。忽略 Eq.(4) 的第二步 L2 单位化，$D=\mathrm{diag}(A^T\mathbf1)$，有

$$Z_{tar}=BD^{-1}A^TX_{ref}.$$

假设所有使用的高斯列和非零、结构固定。该表达不是 $A^{-1}$ 或一般多视图最小二乘伪逆；其秩至多 $\min(M,N,L)$。若某个参考变化 $\Delta X$ 满足 $A^T\Delta X=0$，目标渲染完全无法区分它。加回单位化后更多幅值信息丢失。即便 A、B 几何精确，也不能保证原始 latent 保真。这是 RefineNet 的合理设计动机（C.3），而非作者已给出的可逆性理论。

反事实：若直接求解 $\min_Z\|AZ-X\|^2$，可用耦合高斯的正规方程 $A^TAZ=A^TX$，但每个去噪步计算昂贵且可能病态；若多层重叠则相关列导致歧义。Eq.(4) 是高效逐高斯独立平均。它保留了可见性权重而牺牲严格求逆，提示研究速度与保真的权衡。不能据此断言求逆一定生成更好，需测试。

### 门控不是可信概率

【原文锚点】Eq.(8)–(9)、Fig.4、Table5。$\hat F=F+wA$，$w\in[-1,1]$。原始特征残差路径的价值是允许几何注入被学习减少，不能解释成在两幅图之间做凸组合。更直接的注入幅值为 $\delta=|w|\|A\|$。若 A 取值很大，即使 $|w|$ 很小也可有较大改动。

【拓展分析】仅测 $\mathrm{corr}(|w|,U)$ 不能确定最终修正是否可靠。可额外观察与真实几何误差 E、参考共视覆盖 V、特征一致性残差 R 的条件相关；控制不同去噪时刻和场景。门控可能依据反光外观或模型纹理规律间接学习可靠性，不等于显式预测不确定性。

### 观测、先验和真实恢复

【原文锚点】§4.4、Fig.10/28。共视区域有部分可观测信息，非共视区域用生成先验补全。非共视 PSNR 更好是指定测试中对 GT 的统计改善，不是信息论保证。设两个场景在所有参考射线上产生完全相同观测而在遮挡后具有不同形状，任何只依赖参考的确定性算法都无法同时正确恢复两个场景；随机生成可以表达多解，但不自动给出真实解。

【边界】这些是数学解释与待验证拓展，不是 GeoNVS 作者已实现的理论或算法。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Observation & Prior（可见性与生成先验）
- **Keywords:** Sparse-view NVS; Co-visibility; Generative Prior; View Overlap
- **Highlight:** GeoNVS 将可见参考扩散特征经高斯结构投影到目标；未观测区域仍依赖生成先验，非共视 PSNR 提升不能证明真实恢复。
- **Evidence:** §3.1–3.2、§4.4、Table 4、Fig.28；精读 PROBLEM、EXP-03。
- **Connection Points:** 观测数量、共视覆盖、显式几何条件层级、不可见区生成。
- **Open Questions:** 不同观测稀疏度与覆盖模式怎样影响几何先验的有效区域？

### Topic: Scene Representation（动态特征高斯载体）
- **Keywords:** 3D Gaussian Splatting; Feature Lifting; Feature Rasterization; Alpha Compositing
- **Highlight:** 共享几何而每去噪步重新计算特征，使用渲染权重的跨视角归一化平均而非场景级反复优化。
- **Evidence:** Eq.(2)–(4)、Fig.3、Fig.12、§2；精读 MATH-02/03。
- **Connection Points:** 3D 支撑形式、动态/静态特征、hard/soft assignment、可微计算。
- **Open Questions:** 在固定几何下哪类传递算子最能保留边界、幅度和高频信息？

### Topic: Training & Supervision（参考重渲染监督）
- **Keywords:** Gaussian Positional Encoding; RefineNet; Cosine Similarity; Feature Consistency
- **Highlight:** GS-PE 给出主贡献高斯位置及权重，参考重渲染与原始参考特征方向对齐，目标分支共享精炼器。
- **Evidence:** Eq.(5)–(6)、Fig.13/15、Table 7/C.3；精读 METHOD-03。
- **Connection Points:** 特征自一致性、显式位置编码、监督域、混合信息损失。
- **Open Questions:** 参考方向一致性怎样影响目标不可见区域及特征幅值保真？

### Topic: Uncertainty（有符号几何注入）
- **Keywords:** Adaptive Fusion; Cross-Attention; Residual Gating; Geometry Uncertainty
- **Highlight:** W 在 [-1,1] 调节几何注意力残差，不是概率；三反光场景的 |W| 与不确定性相关约 -0.30。
- **Evidence:** Eq.(8)–(9)、Fig.4、Table 5；精读 MATH-05。
- **Connection Points:** 门控输入、置信估计、残差强度、校准及错误几何传播。
- **Open Questions:** 门控幅值和实际注入范数哪一个更能预测错误先验的影响？

### Topic: Generalization（先验替换与骨干兼容）
- **Keywords:** Plug-and-play; Zero-shot Geometry Prior Substitution; LoRA; Diffusion Backbone
- **Highlight:** 同一 SEVA adapter 接收多种高斯先验；CameraCtrl 另行微调且单参考，零样本先验替换不等于任意骨干迁移。
- **Evidence:** §4.2/4.6、Table 5/6、Fig.23/24；精读 EXP-05。
- **Connection Points:** 结构接口、尺度对齐、先验训练分布、骨干 feature interface。
- **Open Questions:** 如何量化 geometry prior 的域移位与不同 diffusion representation 的适配代价？

### Topic: Evaluation（生成视频的几何代理）
- **Keywords:** Camera Controllability; Chamfer Distance; ViPE; Oracle Reconstruction
- **Highlight:** 由生成和 GT 图像重建点云/相机，比较 Terr、Rerr、CD；GT 重建失败样本排除，指标不是直接真实几何测量。
- **Evidence:** §4.4、Table 3b/4、Fig.9/25–27；精读 EXP-03。
- **Connection Points:** 几何误差口径、相机估计器敏感性、失败样本处理、真实 GT 可用性。
- **Open Questions:** 代理几何指标与已知三维真值的误差是否一致？

### Topic: Efficiency & Scalability（粒子数与重复投影）
- **Keywords:** Voxel-based Pruning; Runtime; Peak Memory; Thin Structures
- **Highlight:** 在10个DL3DV场景剪枝前后4.19→1.89 s/frame，PSNR保持；薄结构是已知失败类型，不能泛化剪枝比例。
- **Evidence:** Fig.8/28、§4.6/H；精读 EXP-06。
- **Connection Points:** 场景支撑密度、投影代价、精度/效率权衡、结构保护。
- **Open Questions:** 能否以局部可观测性和几何贡献调节稀疏度而非统一剪枝？


## RESEARCH-DIRECTION：研究方向与可迁移边界

【定位】本文领域为稀疏观测三维视觉，Task 是 novel-view image synthesis，Task Setting 是指定相机、单/稀疏多图、不同重叠和长轨迹；Research Problem 是如何在生成补全时维持结构与相机对应；Method 是动态 3D-GS feature adapter。这五层不可等同。本文并非一般水下成像、介质建模或直接三维几何恢复论文。

### 对稀疏观测研究的普遍启发

当显式重建和生成先验各有可靠区域时，先构造有物理或几何意义的空间支撑，再让生成器读取该结构并控制错误信息传播，是值得比较的设计维度。真正科学问题不是“是否使用扩散”或“是否使用 3DGS”，而是哪些信号源在什么观测条件下可信，以及如何量化其约束的真实有效性。对应原文 Table4/5、Fig.4/28。

### 潜在领域拓展：Sparse-Observation Underwater Vision

【拓展假设，本文未验证】在水下稀疏视觉场景，观测外观还受介质传输影响；同一个三维点在不同相机位置可能存在与距离、光照、散射有关的外观变化。GeoNVS 避开几何先验 RGB 直接注入，但 reference diffusion feature 仍可能编码受介质影响的信号。若直接跨视角平均，不同观测距离产生的特征差异可能被误当成几何不一致或噪声。需要检验特征中哪些量在几何对齐后近似不变、哪些量应该随成像条件变化。

| 维度 | 本文已有机制 | 潜在领域障碍（未验证） |
|---|---|---|
| 几何支撑 | 3DGS alpha 权重 | 光度变化可能影响几何先验准确性；位姿、尺度误差耦合 |
| 特征传递 | 可见性加权 lifting | 特征未必在同一结构跨传播距离一致 |
| 可靠性 | 有符号残差门控 | 低相关可能来自介质外观或几何错误，原因难辨 |
| 生成补全 | 扩散先验 | 合理外观不等于真实几何或真实物理参数 |
| 评价 | PSNR/SSIM/LPIPS，ViPE代理 | 需要真实几何/位姿与成像条件控制才能分离误差 |

【边界】这里只提出可公开讨论的跨域技术假设，不设定具体用户未公开的课题、方法或结果，不声称已证明水下可行性。本文对一般 NVS 的证据优先于迁移设想。未来可先研究基础观测可靠性，不必预设最终任务是重建、恢复或生成。


## RESEARCH-PROBLEMS：可证伪的研究问题

以下仅是由本文证据派生的候选问题，未进行完整文献检索，不宣称原创或已经解决；优先选能区分机制而非直接迁移模型的实验。

### RP-A：几何传递中的错误归因与可靠性

**现象与证据。** Fig.4/Table5 中先验质量不同，门控减少部分失败；三反光场景 |W| 与 uncertainty 的 r≈−0.30。Fig.28 仍有薄结构错误。

**科学未知。** 可靠性信号能否区分几何错配、参考特征外观变化、遮挡，以及生成网络自己的不确定性？当前门控没有直接概率监督，相关性也不等于错误归因。

**可证伪假设。** 在固定真实几何和相机时，独立改变观测外观，会使基于 feature mismatch 的可靠性估计变差；引入可见性与几何误差的分离条件可减少这种混淆。若不同外观条件下原信号仍稳定准确地区分几何扰动，则该假设不成立。

**验证。** 用有 GT 几何/相机的静态合成场景，分别控制深度或点位扰动、相机扰动、遮挡率和外观变化；保持同一 diffusion backbone 与数据。基线：无门控、原门控、仅几何可见性、联合条件。评价 PSNR之外计算 GT depth/pose、geometry-error detection AUROC、calibration error（仅对定义为概率的预测）、实际注入范数与局部错误变化。成功需在独立测试场景同时改善错误识别与真实几何，不只是输出更漂亮。失败条件：门控完全忽略几何或在正确先验处误抑制。需核查已有 uncertainty calibration 与跨域几何引导工作。

### RP-B：几何支撑固定时特征提升的保真性

**现象与证据。** Eq.(4) 的平均+L2 和 Eq.(3) 的线性混合不可逆；C.3 的精炼在 9 views +1.37 dB，Fig.10 仍模糊。

**科学未知。** 几何错误与特征算子本身的信息损失各占多大比例？改善是否需要更多粒子、保留幅度/高频或区分局部特征混合？

**可证伪假设。** 在 oracle geometry 和已知 visibility 下，当前平均/单位化仍会损失高频特征；保留附加统计（如 feature norm、方差）或局部可逆性约束可提高目标重渲染精度。如果固定真实 geometry 后误差已近于零，或额外信息不能提升，则拒绝该机制假设。

**实验。** 控制同一组参考 feature 和 A/B 权重，比较 top-k、是否L2、独立平均、带正则最小二乘、精炼器；先用 VAE latent 重渲染隔离算子，再验证 diffusion feature 条件下生成。控制高斯密度、参考数、遮挡和噪声，评价 latent reconstruction error、边缘/高频保真、目标 PSNR/LPIPS、运行/内存。成功应在等算力或明确成本下有可复现收益；若仅增加模型参数或过拟合训练视角，则不能归因于算子。需检索特征 splatting、inverse rendering 和稀疏编码工作。

### RP-C：不可见区域的推断歧义与几何评价

**现象与证据。** Table4 非共视 PSNR 最高 +2.08 dB，但 Fig.28 遮挡失败；ViPE Oracle 不是直接真实三维。

**科学未知。** 由可见结构传播得到的生成补全能否表达多解的不确定性？感知质量和真实 geometry error 在遮挡区是否同向？

**可证伪假设。** 在固定参考图而改变遮挡后真实结构的配对场景中，单一确定性生成不能同时恢复真实后方结构；经合理不确定性估计的多样本生成应在未知区呈现更大分布方差，同时已观测区稳定。若未知区方差与真实可辨识性无关，则该不确定性方案无效。

**实验。** 构造具有相同已见区域、不同遮挡后几何的成对 GT 场景；控制相机轨迹、覆盖率及参考数。比较纯几何、生成、显式几何 feature 注入；多随机种子输出，按 GT visibility 和 proxy visibility 两种 mask 评估。观察未知区 geometry error、样本覆盖度、可信区稳定性、相机控制，不把样本多样性当正确性。失败是多样性仅为纹理随机性或可见区破坏。需查多解重建与不确定性文献。

### 优先级及边界

A 与论文门控机制最直接，可通过独立控制扰动分辨原因；B 最易在算子层建立受控验证，计算代价可测；C 对稀疏推断核心矛盾最重要但需要精心构造多解数据和真值评价。先 A/B 建立可控证据，再讨论 C 的完整生成式推断，不预设最终模型或论文创新。任何跨域拓展均需独立文献核查和实验验证。


## INTERNALIZATION：知识内化与理解自检

### 用自己的语言重述

先利用现有几何模型得到一个不必完美的显式三维支撑；在每个去噪时刻，将已经观测到的参考特征经可见性权重放进支撑，再按目标相机读取。由于这种跨视角传递丢失细节，先用结构位置编码和共享精炼器补偿；由于几何可能错，再让目标扩散 feature 决定读取并加上多少几何残差。训练时用参考重渲染一致性和最终 latent 任务共同训练。这就是从重建与生成各自失败推导出的设计，而不是单纯拼接渲染图。

### 概念地图

`参考 RGB+位姿 → 外部几何先验 G → 固定结构投影权重`；`每步扩散 encoder → reference/target features`；`reference features + 结构权重 → 高斯 feature → reference/target feature rendering → GS-PE + RefineNet`；`reference refined vs original → Lfeat`；`target refined + target original → cross-attention + gate → decoder → latent loss / denoise`。支撑先验与学习信号从两个来源汇合。训练可用更完整的 VGGT 点云，推理仅参考；骨干替换需各自微调。

### 易错理解自检

| 问题 | 正确理解 |
|---|---|
| 是否每步重新估计 3DGS？ | 每步重新赋特征；前置几何结构独立获得。 |
| lifting 分母在哪个维度？ | 固定高斯，汇总所有贡献参考像素，而非每像素对所有高斯。 |
| Eq.(3) 是否物理真值？ | 是以 RGB alpha 混合推广到 latent 的建模假设，可能失真。 |
| GS-PE 是否对每像素真实3D位置编码？ | 编码最大贡献高斯中心及权重，只是结构代理。 |
| W 是 sigmoid 概率吗？ | Tanh [-1,1] 的有符号残差门控。 |
| 参考监督能否保证 target 真实？ | 不保证；共享精炼用于学习补偿，未知区域无真实观测。 |
| CameraCtrl 使用 adaptive 吗？ | Table6报告 naïve；不要将 SEVA adaptive 消融外推。 |
| zero-shot 任意骨干？ | 先验替换与骨干兼容不同；骨干各自训练。 |
| CD是真实几何误差？ | ViPE 从生成和GT图像重建的代理距离。 |
| 是否所有结果领先？ | Table4 CD持平，Table5/7、Fig25、Fig28 等存在例外。 |

### 进一步阅读时的检验方式

对另一个生成 NVS 论文优先问：先验在输入、内部 feature、attention 或损失哪一层进入？结构支撑是否固定、是否需目标图或 GT pose、观测证据是否与生成先验区分？训练监督和推理输入是否一致？评估是否有独立真实几何？这些问题从本篇机制而来，仅是未来比较维度，不是已经完成的跨文献结论。


## GLOSSARY：技术术语

| English | 中文 | 本文含义与关系 |
|---|---|---|
| Novel View Synthesis (NVS) | 新视角合成 | 给参考图和相机，输出目标相机 RGB；§3.1 |
| Set NVS / Trajectory NVS | 集合/轨迹新视角合成 | 无序目标集合 vs 连续相机轨迹，§4.3 |
| Co-visibility | 共视性 | Table4 由参考估计点云投影形成目标掩码；与 FOV IoU 不同 |
| 3D Gaussian Splatting (3D-GS) | 三维高斯溅射 | 显式几何粒子，Eq.2/3 给可见性加权渲染 |
| Geometry prior | 几何先验 | 由输入估计的高斯结构，不是真实几何标签 |
| Feature lifting / uplifting | 特征提升 | 参考二维 latent feature→每高斯向量，Eq.4 |
| Feature rasterization | 特征光栅化 | 高斯 feature→目标或参考二维 feature，Eq.3 |
| Alpha compositing | Alpha 合成 | 透射乘局部 alpha 决定各粒子贡献，Eq.2 |
| Hard / soft assignment | 硬/软分配 | 每射线仅最大贡献高斯 vs 多高斯加权，Fig.12 |
| GS-Adapter | 高斯溅射特征适配器 | 动态 lifting、精炼、融合整体 |
| Gaussian Positional Encoding (GS-PE) | 高斯位置编码 | 最大贡献粒子归一化 xyz + 权重正弦编码，Eq.5 |
| RefineNet | 特征精炼网络 | 共享轻量 ResNet，参考 cosine loss，Fig.15 |
| Cosine similarity | 余弦相似度 | 比较向量方向，幅值不受正比例缩放影响，Eq.6 |
| Cross-attention | 交叉注意力 | target diffusion query，geometry feature key/value，Eq.8 |
| Adaptive fusion | 自适应融合 | 以有符号 W 调节几何注意力残差，Eq.9 |
| Naïve fusion | 直接拼接融合 | 2C→C MLP，无显式恒等残差，Eq.7 |
| DPT-style aggregation | DPT 式多尺度聚合 | 统一不同 encoder 尺度及通道，Fig.14 |
| Variational Autoencoder (VAE) | 变分自编码器 | latent 表示的编码/解码、空间 downsampling |
| Denoising timestep | 去噪时间步 | feature 随 t 改变，结构支撑仍可复用 |
| LoRA | 低秩适配 | 对冻结网络 attention 权重学习低秩增量 |
| Classifier-free guidance (CFG) | 无分类器引导 | 条件强度，C.4；不等于 geometry residual gate |
| Inpaint strength | 图像注入强度 | 起始噪声级控制，C.1；s 越大越接近自由生成 |
| Pose-free | 无 GT 位姿设置 | 由 ViPE 等估计输入位姿，非完全不使用相机参数 |
| Scale ambiguity | 尺度歧义 | Eq.1 对齐预测与目标相机中心；单图需额外条件 |
| Chamfer Distance (CD) | 倒角距离 | 点云最近邻距离度量，本文是重建代理 |
| Terr / Rerr | 平移/旋转误差 | cm/deg，估计轨迹与 GT 比较 |
| PSNR / SSIM / LPIPS | 峰值信噪比/结构相似度/学习感知距离 | 图像指标，分别 ↑/↑/↓；非真实3D准确性的充分证据 |
| Oracle reconstruction | 真值视角重建参照 | 由 GT 新视角经重建器估计，不是无误差真实点云 |
| Voxel-based pruning | 体素高斯剪枝 | 降低重复投影开销，图8条件下验证 |

【术语边界】将 geometry consistency、camera controllability、perceptual quality 分开，不能用生成图像观感替代真实空间精度。


## SOURCES：来源与证据边界

- 原论文：[GeoNVS arXiv:2603.14965v2](https://arxiv.org/abs/2603.14965v2)，2026-06-30；PDF 43页，正文§1–5、附录A–H、参考文献。PDF SHA-256 `52630797ac7c1a4ce100b7760b053fa038e19be594ca0e6912bc689ae953cc21`。
- 本笔记绑定已冻结完整精读报告，SHA-256 `8c5239c940f2bb0b1ad9473e9bd5832640346d12c950185578da93d92f28bd42`。正文对应 unit 名称仅作为可追溯定位；未改动报告。
- Source 与 HTML 为辅助，部分表号与 PDF 不同，以 PDF 为准；Source 无独立版本戳。29个已验收 Figure 展示副本中19个有保真差异记录；非像素级无损。
- 官方代码、模型权重链接仅用于身份核验，项目页访问未成功；未下载运行模型或读取其他论文全文。
- 潜在关联索引仅列本篇知识节点与开放问题，**跨文献实质比较延期**。未建立跨论文继承、改进、互补关系。
- 【边界】原文事实定位章节/公式/图表；数学背景与补充推导明示假设；跨域和研究候选未验证、不宣称创新或实验结果。未将私人研究材料纳入可发布笔记。



