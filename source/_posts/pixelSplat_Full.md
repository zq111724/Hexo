---
title: "pixelSplat：从双视图前馈预测三维高斯（完整精读与研究笔记）"
date: 2026-11-26 00:00:03
index_img: "/pixelSplat/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/pixelSplat/)

# 第一部分：论文精读报告

# pixelSplat 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2312.12337v4（2024-04-04）；缓存记录 v4（2024-04-04）；最新版未能确认。核验日期：2026-11-26。

**官方资源：** [arXiv](https://arxiv.org/abs/2312.12337) | [PDF](https://arxiv.org/pdf/2312.12337v4) | [Project Page](https://dcharatan.github.io/pixelsplat/) | [Code](https://github.com/dcharatan/pixelsplat)

<!-- PAPER-METADATA END -->

> 论文：pixelSplat: 3D Gaussian Splats from Image Pairs for Scalable Generalizable 3D Reconstruction
>
> 作者：David Charatan、Sizhe Lester Li、Andrea Tagliasacchi、Vincent Sitzmann；CVPR 2024。
>
> 版本及来源：用户上传 arxiv_2312.12337.zip，实际 PDF arXiv:2312.12337v4（2024-04-04）。PDF 16页，正文1–8、参考文献9–11、附录12–16。
>
> 阅读方式：Paper-to-Hexo v0.5.8，Markdown-first 自动精读。PDF 权威，HTML 辅助；全部11幅原图经PDF区域提取与展示副本验收。
>
> 证据：明确区分【原文事实】【背景知识】【合理推断】【独立评价】【尚未证实】。本报告不使用其他论文全文或代码作为科学证据。

<!-- PTH:BEGIN INIT -->
# 0. 自动初始化

## 资源与版本

【原文事实】本次论文为 *pixelSplat: 3D Gaussian Splats from Image Pairs for Scalable Generalizable 3D Reconstruction*。PDF 首页署名 David Charatan、Sizhe Lester Li、Andrea Tagliasacchi、Vincent Sitzmann，arXiv 印记 `2312.12337v4 [cs.CV] 4 Apr 2024`。PDF 16 页，实际读取并预览每页；HTML 主文件身份、作者与 v4 链接一致。本地资源来自 `arxiv_2312.12337.zip`，不混合其他版本。

## 章节与目录

| PDF 页 | 内容 |
|---|---|
| 1–2 | Abstract、1 Introduction；2 Related Work |
| 2–3 | 3 Background: 3D Gaussian Splatting |
| 3–6 | 4 Image-conditioned 3D Gaussian Inference；4.1 Resolving Scale Ambiguity；4.2 Gaussian Parameter Prediction |
| 6–8 | 5 Experiments；5.1 Experimental Setup、5.2 Results、5.3 Ablations and Analysis；6 Conclusion |
| 9–11 | References |
| 12–13 | Supplement A Training Details（A.1–A.4）、B Using More Reference Views、C Limitations、D Additional Results |
| 14–16 | 补充图9–11 |

|编号|PDF页|内容|范围|
|---|---:|---|---|
|Figure 1|1|总体示意|正文|
|Figure 2|3|SfM尺度与极线|正文|
|Figure 3|4|概率高斯预测|正文|
|Figure 4|5|点云与深度|正文|
|Figure 5|7|定性对比|正文|
|Figure 6|8|注意力|正文|
|Figure 7|8|消融图|正文|
|Figure 8|13|架构|附录|
|Figure 9|14|两视图/三视图|附录|
|Figure 10|15|RealEstate10k更多结果|附录|
|Figure 11|16|ACID更多结果|附录|
|Table 1|6|主定量比较|正文|
|Table 2|8|消融|正文|
|Table 3|13|更多参考图像|附录|
|Algorithm 1|4|概率预测单个像素对齐高斯|正文|

合计 11 Figure（正文7，附录4）、3 Table（正文2，附录1）、1 Algorithm。式（1）–（8）正文，式（9）–（10）附录A.1。图表均可从完整 PDF 访问。

## 证据与保真边界

22 个原始资源文件的字节哈希复核通过，HTML 七项未解析引用属于网页装饰/样式/脚本/字体，并非核心图像。PDF 是事实权威。十一幅图已按完整页面区域提取，包含图中标签、箭头、子图；图8初次边框裁切已修正，最终五幅精确通过、六幅带非精确回渲染记录通过。JPEG 为 300 DPI/Q90/4:4:4，不能作为原始像素的定量测量依据。保存原件、区域 PDF、图映射和审计。实时查新无法确认，官方 abs 缓存显示截至 v4；因此最新版本状态未验证。项目页和代码仓库仅核对身份，不使用其科学内容补全原文。外部参考文献未阅读，不以论文对其描述代替独立验证。

本单元为来源与目录核验，不涉及方法机制。

本单元只登记公式编号，不展开数学推导。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
# 1. 全局认识

## 1.1 Title：标题翻译

**pixelSplat: 3D Gaussian Splats from Image Pairs for Scalable Generalizable 3D Reconstruction**

**pixelSplat：从图像对生成三维高斯基元，实现可扩展的泛化三维重建。**

- *3D Gaussian Splats*：指由三维中心、协方差、不透明度及视角相关颜色参数定义的原语。Splat 是将原语投影至目标图像平面并合成的操作；标题强调用这些显式原语构建辐射场，而非仅输出一张深度图。
- *Image Pairs*：两个参考视角，不是随意两张无关联图像。§4明确还需要二者对应相机参数。标题本身未说明姿态已知，不能仅凭标题断言免姿态。
- *Scalable*：在本文语境下主要指计算/显存开销降低及可在多场景数据上训练。不是已证明能随视图数无限扩张，附录C反而指出注意力的显存瓶颈。
- *Generalizable*：跨场景学习共享模型，推理时从新的参考图像直接生成新场景的高斯集合，不等于对任何域、任意相机分布或未见区域保证正确。
- *3D Reconstruction*：输出明确的三维辐射场原语，其性能主要通过新视角图像检验；不是以公制深度、网格误差或测绘精度作为本篇主要评价标准。

### 任务与非任务

【原文事实】§4给定一对图像及其相机参数，推断可渲染的三维高斯表示。其特点是前馈推理，不要求为每个新场景迭代拟合完整场景；输出可供后续多帧目标视图复用。

【背景知识】传统逐场景优化中，每个场景的原语参数通过该场景多图优化得到；泛化方法将主要学习开销移到跨场景训练，网络参数在测试场景保持共享。两者优化的对象和推理过程不同。

【证据边界】标题提示一种输出表示和应用目标，但未说明相机尺度如何处理、概率分布如何监督、是否直接预测全部属性、哪些实验建立效率优势。这些均需阅读§3–5及附录。本文处理的是给定姿态坐标系中的几何一致性，不是从两张图凭空恢复绝对尺度；也不等同于无姿态 SfM 或生成式填补任意不可见区域。

标题单元不需要具体公式；公式与尺度推导将在后续数学单元展开。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 Abstract：摘要的技术逻辑

### 背景与目标

【原文事实】摘要首先提出从图像对前馈重建用三维高斯原语参数化的辐射场。作者强调实时、显存友好的渲染，可扩展训练及快速推断。其逻辑与引言相同：隐式场往往需对每条目标射线反复查询网络，而显式高斯经栅格化可高效合成图像。光场Transformer虽然减少了体采样，但仍需每条目标射线做特征交互，且不直接输出可编辑三维场景。

因此目标不是只让一次新视图生成更快，而是令网络一次获得可复用的场景表示：一次编码生成原语，随后以目标相机多次渲染。若场景要输出100帧视频，编码开销在这些帧间摊销，这是区别于逐射线网络的重要计算结构。

### 技术机制

【原文事实】作者识别稀疏且局部支撑的原语拟合易陷入局部极小。一个被放错位置的小高斯可能离真实表面过远，移动中心得到的梯度很弱；即便可移动，跨过空空间时遮挡其他物体还会使损失暂时增加。原3DGS可用非可微的增密/裁剪启发式，而端到端网络预测需要可回传的训练信号。

本文改为预测致密概率分布，在每次前向中采样高斯中心，而不是固定单个位置。沿像素射线的深度分桶构成离散候选，网络可以为不同候选赋予概率。若采中的位置有利于渲染，增加该高斯的 opacity 有望降低误差；通过令 opacity 等于对应桶概率，渲染误差可以回传到概率。下一次前向更可能在该位置采样。摘要称这种耦合为重参数化技巧。实际算法中的离散采样仍非普通路径可微：梯度是经不透明度这一连续分支进入分布，并非对采样索引直接求导（详见式7及数学单元）。

摘要未详细写出引言中另一关键设计：SfM的每场景任意尺度。本文利用双视图极线注意力将候选对应和在给定姿态单位中的深度相关联，使预测中心符合这些姿态所采用的尺度。

### 结果与证据边界

【原文事实】作者在真实场景 RealEstate10k（室内视频为主）与 ACID（航拍景观）上开展宽基线新视角合成，对比光场Transformer等方法，声称提高新视角质量、加速渲染约2.5个数量级，并输出可解释和可编辑的辐射场。具体速度应查看Table 1及§5.2的不同时间口径，不把摘要的数量级误读成固定所有硬件上的端到端倍数。

【尚未证实】摘要不是严格证明：没有从中获得绝对公制尺度、真实三维几何误差的独立保证，也没有证明采样方案不存在局部极小或给出无偏梯度估计。两个数据集的结果不能直接推广至所有域。

**一句话概括：** pixelSplat 将具有双视图尺度信息的像素特征转成沿射线的概率原语预测，使用高斯不透明度提供分布的训练梯度，最终一次前馈得到可快速复用渲染的显式3D高斯场。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 Contributions：贡献主张

【原文事实】引言（pp.1–2）没有正式编号的贡献列表。以下是依据作者在引言、§4和结论所述归纳的三条主线，而非冒充作者原列表。

### 尺度感知编码

**问题发现/方法设计。** 真实视频SfM姿态只确定到每场景独立尺度，单张图像不能确定该任意尺度。几何预测与给定相机平移的尺度不一致时，即使预测视觉上合理深度，也无法生成与目标相机正确匹配的视差。作者利用双视图在极线上搜索对应，将按相机姿态三角化的深度进行位置编码并拼入注意力键值，继而自注意力传播。§4.1、图2、图6和Table 2的 No Epipolar Encoder、No Depth Encoding 是证据主线。极线几何与注意力本身不是本文首创；创新重点在为前馈高斯提供相机尺度一致的特征。

### 概率原语预测

**核心方法贡献。** 对于局部支撑的原语，直接回归高斯中心存在弱梯度和错误遮挡产生的能量障碍。作者不对一个位置做连续移动，而是从离散深度分布抽取位置；把不透明度与被采桶概率绑定，使渲染误差能直接影响将来的采样频率。依据§3、§4.2式5–8及Algorithm 1、Figure 3、Table 2。不是直接预测稠密真实体密度，也不是传统3DGS启发式的原样复制。作者的重参数化说法需要从连续代理梯度而非严格类别采样微分理解。

### 显式前馈重建与实验

**系统/实验贡献。** 将逐像素高斯预测与3DGS可微栅格化组合，使从两张已知姿态图像一次前馈生成显式场景成为可训练的任务。§4与图1、图8说明结构，§5的Table 1、Figure 4/5展示质量、效率及显式表示。与使用每个目标射线query的光场渲染不同，编码一次后每个目标视角直接投影原语。作者声称首次实现此类从图像对前馈预测3DGS；本报告只能确认其原文主张，不通过外部查新独立证明优先权。原始高斯原语、球谐和栅格化继承自既有方法。

|主线|类型|主要论证|待后续核查|
|---|---|---|---|
|相机尺度一致的极线编码|问题与设计|Fig.2、6；Tab.2|对应是否正确、深度特征是否足够|
|概率分桶＋采样 opacity|方法/优化机制|Fig.3、Alg.1、Tab.2|梯度的真实路径和理论边界|
|显式、可复用的前馈3DGS|系统和实验|Fig.1、4、5、8；Tab.1|质量/效率口径、几何可靠性|

三条主线有因果依赖：尺度编码使深度候选具有相机坐标意义；概率预测缓解该位置的直接回归问题；可微渲染最终提供监督和实际效率。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 Conclusion：结论与原文边界

### 作者结论

【原文事实】§6（p8）回顾了从两图重建基元化辐射场、比先前泛化NVS工作更快的推断、显式场景表示，以及使用致密概率分布参数化原语位置、以不透明度耦合将梯度回传到分布参数的设计。作者认为这类原语参数化思路可推广至其他应用中的先验驱动原语推断。这是研究方向建议，不是已经在其他任务验证的结论。

作者提到两条未来路线：与扩散模型组合实现生成建模；移除相机姿态要求，利于扩大训练规模。两条都不是当前pixelSplat已解决的能力。它们分别对应未观测区域的信息不足与现有姿态输入要求。

### 适用范围与局限

【原文事实】附录C（p13）明确列出三项局限：

1. 双视图高斯只是取并集，未融合或去重。两个参考像素可能观测同一个真实表面，但生成多个三维原语；方法未建立全局唯一表面对应。
2. 未解决不可见部分的生成式建模。图像内推/基于观察的补全与生成完全未见内容不是同一任务。
3. 多参考视图时极线注意力显存成本过高。附录B虽示范3视图，却不能据此推断视图数量可无成本增加。

§5.2（p7）进一步说明，使用深度正则微调生成图4的点云；在训练分布以外相机视角，反射表面常表现透明，高斯常具有 billboard（如面向参考相机的薄片）观感。可见的3D点云不意味着精确几何或无视角依赖伪影。

【独立评价】作者展示的证据足以使两图显式场成为有意义的技术路线，但图像合成指标和主观图像不能保证原语位置是真实表面、物理尺度正确或遮挡后区域唯一确定。所谓泛化应限定到论文所用训练/测试划分与输入姿态条件；生成式多假设、跨域、免姿态均需额外实验。

结论单元不引入额外数学推导。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
# 2. 图表精读

## 2.1 正文图表功能

|编号|名称|核心功能|
|---|---|---|
|Figure 1|方法总览|从输入图像对推断3DGS，再合成新视图，强调显式且可复用的表示。|
|Figure 2|尺度歧义|通过两个不同尺度树和极线示意说明匹配与姿态尺度深度的关系。|
|Figure 3|概率预测|通过三条射线说明概率分桶、采样高斯位置、opacity绑定及输出属性。|
|Figure 4|点云、深度|展示5个场景的显式三维高斯和每个场景两个参考视点的渲染深度。|
|Figure 5|新视角定性对比|三组室内、两组户外，比较同目标视角的纹理、结构和伪影。|
|Figure 6|极线注意力|在两视图示例上标出五个像素及其极线注意力位置。|
|Figure 7|消融图|比较完整模型、无极线、无采样、加深度正则的视觉结果。|
|Table 1|主要实验|两个数据集的PSNR、SSIM、LPIPS和推断时间、显存。|
|Table 2|消融|评估极线、深度编码、概率采样与深度正则。|
|Algorithm 1|高斯预测|五步定义单像素生成一高斯的概率过程。|

## 2.2 附录图表功能

|编号|名称|核心功能|
|---|---|---|
|Figure 8|架构图|补足单图编码器、双向极线采样、两轮注意力与高斯预测的连接。|
|Figure 9|三视图对比|固定第三参考图像位于两参考视图之间，展示更多观测的定性收益。|
|Table 3|三视图数值|量化同一设置下的额外观测收益。|
|Figure 10|RealEstate10k更多结果|增加7组室内外例子，展示方法相对基线的可见差异。|
|Figure 11|ACID更多结果|增加7组航拍自然/建筑例子，拓宽定性观察范围。|

【证据层级】图1–3/8为总体/设计示意，不是实验本身；图4–7/9–11是定性结果或内部可视化，Table 1–3为定量评价。图3与Algorithm 1对应实际数学步骤，图6及7连接编码、采样的消融；图4的几何观感是显式表示的展示而非定量真实几何评估。

本单元仅建立图表导航，不展开图中具体机制。

图表功能总览不推导公式。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
## Figure 1：从图像对到可复用的场景

![Figure 1](pixelSplat/Figures/Figure_01.jpg)

*来源：PDF 第1页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 1. Overview. Given a pair of input images, pixelSplat reconstructs a 3D radiance field parameterized via 3D Gaussian primitives. This yields an explicit 3D representation that is renderable in real time, remains editable, and is cheap to train.

**中文：** 概览。给定一对输入图像，pixelSplat 重建由三维高斯原语参数化的三维辐射场。这种显式三维表示能够实时渲染、保持可编辑性，并且训练成本较低。


### 输入—原语—渲染

**视觉观察。** 左列 Input Views 为卧室的两张参考图像（同一张床、墙面和窗口，相机位置不同）；中间是从更远视角观看的三维高斯组成的房间局部，蓝色相机线框表示视点；右列 Novel Views 是两张新视角图像。两段箭头分别标记 Inference 和 Splatting。中央不是第三张输入，而是由原语组成的空间表示；左侧到中间是一次场景推断，中间到右侧是对场景进行不同投影。

**技术含义。** 给定参考图像 $I_1,I_2$ 及相机内外参，网络 $E_\theta$ 生成高斯集合 $G$。以相机 $P_t$ 查询时，通过可微栅格化 $\mathcal R(G,P_t)$ 获得目标图像。网络的学习对象是跨场景共享的 $\theta$，而不是为某一输入对单独拟合网络权重。输出集合含每个高斯中心、协方差、不透明度与球谐系数；可直接查看、变换或选择原语，在理论上可以为不同目标相机重复渲染。相反，逐目标射线做特征匹配的光场方法每生成新帧都重复大量查询。

**为什么强调中间表示？** 一张预测图像本身没有显式的三维可编辑原语；中间集合可用于跨帧复用。它既承担多视角一致性的结构约束，又允许采用已有的3DGS光栅渲染。若一场景只渲一张图，编码开销不可忽略；若连续渲染许多视角，编码成本被摊销。训练时同一中间表示受多个目标图像监督（附录A.1）。

### 正文关联与证据边界

§4给出双视图编码和高斯预测，§5.2及Table 1量化速度与图像质量，图8给出具体架构。图1用于直观说明输出形式，而非严谨证明实时速度或训练资源需求。中央高斯形状的可视化不能证明真实物体表面、绝对尺度或未观察区域被重建；实时渲染速度也不等于前馈编码速度。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
## Figure 2：任意尺度与双视图深度

![Figure 2](pixelSplat/Figures/Figure_02.jpg)

*来源：PDF 第3页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 2. Scale ambiguity. SfM does not reconstruct camera poses in real-world, metric scale—poses are scaled by an arbitrary scale factor that is different for each scene. To render correct views, our model’s 3D reconstruction needs to be consistent with this arbitrary scale. We illustrate how our epipolar encoder solves this problem. Features belonging to the ray’s corresponding pixel on the left are compared with features sampled along the epipolar line on the right. Epipolar samples are augmented with their positionally-encoded depths along the ray, which allows our encoder to record correct depths. Recorded depths are later used for depth prediction.

**中文：** 尺度歧义。SfM 无法按现实世界的公制尺度重建相机姿态，而是以每场景不同的任意尺度因子缩放姿态。为了渲染正确的新视图，三维重建必须与此尺度一致。作者用图示说明极线编码器的处理方式：左侧射线对应像素的特征与右侧极线上的采样特征比较，极线样本附带沿射线的位置编码深度，以便编码器记住正确深度并用于后续预测。


### 尺度匹配几何

**视觉观察。** 图中有小而深色的树（Arbitrary SfM Scale）和大而浅色的树（Real-world (Metric) Scale）。左右相机及图像平面以线框呈现，蓝色射线从左相机经过左图树的位置穿过场景；在右相机图像平面上的投影形成标记的 Epipolar Line。图中同一类物体的不同尺度直观表示了单图对尺度不敏感的现象。这里的两棵树是概念示意，而非两个实际同时存在的物体。

**机制。** 若所有场景点与相机平移共同乘以正尺度 $s_i$，针孔投影中的坐标比值不变，图像本身无法告诉网络是现实尺度还是SfM任意尺度。给定的SfM基线长度却能在任意单位中提供相对深度约束：固定左像素后，候选3D点只能在对应射线上；投影到右图即一条极线。匹配右图的不同候选位置相当于选择不同的三角化深度。若基线单位放大，三角化深度也同比放大。网络不必先求出一个未知的公制缩放因子，它只需要预测与现有相机参数相容的深度。

式（1）将右图候选特征和三角化深度的频率位置编码拼接成 $\mathbf s_l$，式（2）把左像素特征投影为query、候选特征投影为key/value；式（3）的softmax注意力将高权候选信息写回左图像素特征。反向也做对应过程。图示中没有显示所有候选、权重或数值，这些过程由§4.1给出。

对遮挡区域或右图视野外像素，没有可靠匹配；作者用逐图self-attention传播尺度与上下文信息，而不是宣称三角化了这些不可匹配点。

### 正文关联与证据边界

§4.1（pp3–4），式1–4，Figure 6的注意力可视化及Table 2的两种编码消融共同对应此图。图2不证明模型精确找到每个对应，也不证明恢复了绝对公制尺度。图上树的大小无定量轴，不应读成具体比例。只要姿态不可靠、基线过小、纹理重复或遮挡，匹配及三角化本身就可能不稳定。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
## Figure 3：像素对齐高斯的概率预测

![Figure 3](pixelSplat/Figures/Figure_03.jpg)

*来源：PDF 第4页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 3. Proposed probabilistic prediction of pixel-aligned Gaussians. For every pixel feature F[u] in the input feature map, a neural network f predicts Gaussian primitive parameters Σ and S. Gaussian locations µ and opacities α are not predicted directly, which would lead to local minima. Instead, f predicts per-pixel discrete probability distributions over depths pϕ(z), parameterized by ϕ. Sampling then yields the locations of Gaussian primitives. The opacity of each Gaussian is set to the probability of the sampled depth bucket. The final set of Gaussian primitives can then be rendered from novel views using the splatting algorithm proposed by Kerbl et al. [19]. Note that for brevity, we use h to represent the function that computes depths from bucket indices (see equations 6 and 7).

**中文：** 所提出的像素对齐高斯概率预测。对于输入特征图中的每个像素特征 F[u]，网络 f 预测高斯参数 Σ 和 S。不直接预测位置 μ 与不透明度 α，以避免直接预测造成的局部极小。f 改为预测由 ϕ 参数化的逐像素离散深度概率分布 pϕ(z)，通过采样获得原语位置。每个高斯的不透明度等于所采深度桶的概率。最终高斯集合通过 Kerbl 等人的 splatting 算法渲染新视图。图中以 h 简记由桶索引求深度的函数（式6–7）。


### 概率采样与不透明度

**视觉观察。** 从左到右：左侧网格特征图 F 中选红、蓝、绿三个像素 a/b/c，其射线以对应颜色绘出；$F[a],F[b],F[c]$ 输入同一个神经网络 $f$。中间上框是三行不同颜色的离散深度概率直方图（红色峰在中部偏右、蓝色峰较靠前、绿色峰偏后）；箭头标明从各自分布采样 $z\sim p_{\phi}(z)$。图中中间下框给出每个样本的中心 $\mu_a=h(z_a)$ 等和不透明度 $\alpha_a=p_{\phi_a}(z_a)$ 等。协方差 $\Sigma$ 与球谐系数 $S$ 由网络另一路输出。右端三条不同颜色的三维射线上布置对应颜色的椭球，标签列出三个高斯完整属性。图中的颜色用于标记不同像素的处理，不代表最终物体必为红绿蓝或实际深度值。

**数据流。** 假设单射线只生成一个原语（正文简化 $M=1$）。每个像素对应 $Z$ 个候选深度桶，网络输出归一化的概率 $\phi\in\mathbb R^Z$、每桶偏移 $\delta$ 和 $\Sigma,S$。从分类分布抽到索引 $z$，由式（6）的 $b_z$ 和式（7）的 $\delta_z$ 得到射线位置 $\mu=o+(b_z+\delta_z)d_u$，并设 $\alpha=\phi_z$。图里 h 表示这些深度/位置计算的简化记号，并不等于网络直接回归整个位置。网络仍回归协方差、球谐和偏移，并未对所有参数做随机采样。

**Why：** 一个直接回归的高斯若初始位置错误，其小支撑范围使位置梯度可能接近零。离散分布的多个非零候选为位置探索提供跳跃路径：不必在每一步沿中心坐标连续穿过空区域。与标准3DGS的剪枝/分裂不同，候选概率在学习中改变，而不是手工修改不可微的原语列表。

**训练路径。** 设本次抽中索引 z 为固定，渲染损失对 $\alpha$ 有梯度，通过取值 $\alpha=\phi_z$，被选概率可收到相应梯度。对概率logits再通过softmax反传，会联动各桶。若当前位置的颜色和遮挡合理，提高其不透明度可能降低图像损失；训练趋向提高该桶概率。若错误位置产生遮挡或颜色误差，梯度可能压低该桶。下一次抽样频率随概率变动。但这个逻辑依赖损失对不透明度的实际方向；颜色、协方差、其他高斯可能提供替代解释。不可把离散索引采样直接称为普通可微的坐标变换。

**正文简化与实际区别。** Algorithm 1 描述一个高斯；附录A.1训练时每射线独立采样3次，且 $\alpha=\phi_z/3$。这让每次前向可在多个深度获得梯度，并控制合成不透明度。图3画的每射线一个高斯只是简化。不同次采样可以抽到同桶，不保证有三个不同位置。

### 正文关联与证据边界

§3局部极小，§4.2式5–8、Algorithm 1，附录A.1以及Figure 7/Table 2 No Probabilistic Sampling。图示不是严格的无偏梯度证明或全局最优证明。桶概率不是经过真实深度标签校准的后验概率；它是通过渲染目标学习的位置分布。后续数学单元将明确该代理梯度与完整期望梯度的差别。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
## Figure 4：显式高斯和深度图

![Figure 4](pixelSplat/Figures/Figure_04.jpg)

*来源：PDF 第5页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 4. 3D Gaussians (top) and corresponding depth maps (bottom) predicted by our method. In contrast to light field rendering methods like GPNR [47] and that of Du et al. [10], our method produces an explicit 3D representation. Here, we show zoomed-out views of the Gaussians our method produces along with rendered depth maps, as viewed from the two reference viewpoints.

**中文：** 作者方法预测的三维高斯（上）与对应深度图（下）。不同于 GPNR [47] 和 Du 等 [10] 的光场渲染方法，本文生成显式三维表示。这里展示作者方法输出高斯的缩远视图，以及两个参考视点的渲染深度图。


### 点云与深度

**视觉观察。** 从左到右是五个场景，上部为缩远查看的高斯集合，下部每个场景对应两张彩色深度图，共10张。第一组房间的橙红墙、床和黑色相机框可辨，底部深度呈前后明显变化；第二组有沙发及墙上挂画，边缘处有悬浮/不连续感；第三组黄色室内墙面、窗及桌椅，深度图中近处与窗口区域有明显颜色分区；第四组为户外树和岩石/建筑状结构，场景轮廓较为分散；第五组可见室内桌椅和较大片稀疏灰白、黑色原语。各组下方深度图呈不同颜色分布，颜色没有公制刻度或统一colorbar，不能从色值反推米数或比较不同场景绝对深度。

**机制。** 每个参考视图通过像素对齐原语预测生成中心及属性。两视图的高斯被转换至同一个世界坐标系并求并集，故可以从远离两个参考相机的观察点可视化，不必为新的相机执行逐射线特征推断。深度图由预测表示渲染得到，不是独立传感器测得的深度真值。作者在§5.2说明，图4使用附录A.1的深度正则额外微调50,000步的版本，不能把图中点云无条件归因于未加正则的主模型。

【背景知识】高斯是辐射场的一种软支撑原语，颜色和不透明度受图像监督，原语中心不必恰好落在真实实体表面。图中薄片、缺口、漂浮原语可能来自稀疏视角下的几何歧义；这与二维新视图可能看起来正确并不矛盾。

### 正文关联与证据边界

§5.2（pp6–7）明确报告分布外视角时反射面透明、billboard状原语；附录A.1给出深度正则，Table 2显示该变体图像质量略下降。Figure 4展示显式表示可查看和深度可渲染的性质，不能给出真实三维几何精度、深度误差或可完整重建的保证。题注 GPNR 引用编号为[47]，表1及§5.1为[46]；按原文记录编号差异，不自行改写被引文献。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
## Figure 5：宽基线新视角合成定性结果

![Figure 5](pixelSplat/Figures/Figure_05.jpg)

*来源：PDF 第7页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 5. Qualitative comparison of novel views on the RealEstate10k (top) and ACID (bottom) test sets. Compared to the baselines, our approach not only produces more accurate and perceptually appealing images, but also generalizes better to out-of-distribution examples like the creek in the bottom row.

**中文：** 在 RealEstate10k（上）与 ACID（下）测试集上的新视角定性比较。与基线相比，作者认为其方法生成更准确、更具感知吸引力的图像，并更好地泛化到分布外例子，例如最下行的溪流。


### 五组定性结果

**布局。** 第一列 Ref. 每组上下两张小图是参考视角，其余五列分别为真实 Target View、Ours、Du et al. [10]、GPNR [46]、pixelNeRF [58]。每行使用同一场景与目标视角，前3行为RealEstate10k室内，后2行为ACID户外。

|行|目标场景与可见比较|
|---|---|
|1|餐桌、椅子、后方高亮窗户与窗帘。Ours保留较清晰桌椅边缘及背景形状，Du接近但局部更模糊，GPNR的结构细节较弱，pixelNeRF出现明显的模糊/重影。|
|2|玻璃桌、椅子和玻璃门。Ours较清楚地保留椅背和门框，Du有较多柔化，GPNR与pixelNeRF的椅子/反光区域出现更明显模糊。玻璃是困难区域，即使Ours也非完全一致。|
|3|客厅、沙发、墙画、窗。Ours保留沙发、窗及边缘轮廓，其他方法逐渐出现细节缺失或模糊；pixelNeRF尤明显。|
|4|蓝天、海岸及红色屋顶、绿植。Ours与Du能保持主要地平线和建筑布局，GPNR/ pixelNeRF有明显的植被/建筑边缘模糊。|
|5|狭窄溪流/峡谷与斜向树干。Ours的近处树干及溪流纹理较清晰，Du仍有合理结构，GPNR和pixelNeRF出现形变或大面积模糊。作者特别强调此例的分布外性质。|

**解读。** 这是相同目标视图的视觉保真比较，不是三维结构直接测量。具有正确尺度的跨视图对应能减少投影错位；在仅一个参考视图观察到的区域，网络还依赖上下文及训练先验。若几何对齐较好，同一个显式场可在相机运动中产生一致投影。不过图中不能单独分辨“几何更正确”与“网络学习了更好的纹理先验”各自贡献。

### 正文关联与证据边界

§5.1使用原训练/测试划分、相同课程与256×256分辨率，目标为两参考帧之间的视频帧。Table 1给出跨测试集的平均指标，图5是少量实例补充。最末行作者称 OOD，但未在此图给出预先定义的OOD划分或专门统计；不能以此证明对所有陌生场景强泛化。没有真实深度标注对比，因此可见清晰度不等价于真实三维准确性。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
## Figure 6：极线交叉注意力

![Figure 6](pixelSplat/Figures/Figure_06.jpg)

*来源：PDF 第8页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 6. Attention visualization. We visualize the epipolar cross-attention weights between the rays on the left and the corresponding epipolar lines on the right to confirm that our model learns to find the correct correspondence along each ray.

**中文：** 注意力可视化。作者将左图射线与右图对应极线之间的极线交叉注意力权重可视化，以确认模型学会沿每条射线寻找正确对应。


### 注意力与对应

**视觉观察。** 两张客厅图像显示同一面灰墙、挂画、沙发、窗与地面。左图在不同部位标出绿、橙、红、黄、蓝五个彩色点；右图上有对应五条粗黑极线，靠近匹配位置有同色短条：例如绿点在墙画、橙点在旁边墙面、红点在沙发、黄点在边桌、蓝点在地板附近。极线的斜率及位置随像素与相机几何而不同。颜色是跨两幅图的关联标识；图没有色条、精确attention数值或深度刻度，不能从黑线宽度反推数值权重。

**为何看极线而非全图？** 给定左像素和相机姿态，该射线在右图投影的极线包含可能对应的像素。只沿极线采样，搜索空间由二维邻图变成几何约束的候选序列。式（1）拼接三角化深度位置编码，式（2）构成Q/K/V，式（3）用查询与候选键匹配的softmax权重汇总值。这里权重较大的位置若与左像素对应同一表面，值中的深度编码就提供与姿态一致的深度线索。

这不是把颜色点自身当作监督。训练目标仍是新视角重建损失；注意力可视化揭示网络学到的内部行为。逐图 self-attention 让缺乏直接对应的像素从已匹配区域获取信息，但这种传播不保证无歧义深度。

### 正文关联与证据边界

§4.1与§5.3 Q1a/Q1b，Table 2 No Epipolar Encoder 相对完整模型下降6.20 dB，No Depth Encoding下降1.12 dB。图6支持“该示例中学到合理对应”这一机制解释；不提供像素级对应误差统计，也没有直接评估真实公制尺度。单次可视化不能证明每个attention head或每个像素都专门学习了几何。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
## Figure 7：消融结果的可视化

![Figure 7](pixelSplat/Figures/Figure_07.jpg)

*来源：PDF 第8页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 7. Ablations. Without the epipolar transformer, our model is unable to resolve scale ambiguity, leading to ghosting artifacts. Without our sampling approach, our model falls into local minima that manifest themselves as speckling artifacts. Regularizing our model’s predicted depths minimally affects rendering quality.

**中文：** 消融实验。不使用极线 Transformer 时，模型无法解决尺度歧义，产生重影。不使用采样方案时，模型陷入表现为斑点伪影的局部极小值。对预测深度施加正则对渲染质量影响很小。

### 四组消融图
图为三组场景，每行从左到右为完整模型、No Epipolar Encoder、No Probabilistic Sampling、Plus Depth Regularization。完整模型是比较基准；第二列出现物体边缘的重影及模糊：两幅参考图各自预测的高斯可能在同一物理表面附近产生不同深度，因此从目标相机投影时无法对齐。第三列存在离散的噪点和斑驳外观，作者将其解释为直接预测位置引起的局部极小值；其现象与第二列不同，不能将所有伪影都归因于尺度。最后一列整体接近完整模型，说明在这些示例中深度正则没有明显破坏外观。

### 正文关联与证据边界
§5.3、Table 2 提供量化支撑：去极线编码器 PSNR 下降 6.20 dB，去概率采样下降 1.47 dB，加入深度正则下降 0.15 dB。表中还有 No Depth Encoding（24.97 dB），本图没有这一列。图显示与机制假设一致的输出变化，但没有独立测量尺度误差、优化轨迹或深度 GT，故不能单独证明重影或斑点的唯一原因。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
## Figure 8：编码器完整架构

![Figure 8](pixelSplat/Figures/Figure_08.jpg)

*来源：PDF 第13页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 8. Architecture diagram.

**中文：** 网络架构图。

### 四模块架构
- Overview：两张参考图各自送入同权重的 Per-Image Encoder，随后以双向 Epipolar Attention 交互，分别输出特征图。图中两条支路对应两个参考相机，不表示有两套独立训练参数。
- Per-Image Encoder：ResNet-50 路径提供局部特征，DINO 预训练 ViT-B/8 路径提供全局上下文，经层级投影及上采样对齐到像素网格，逐像素相加形成特征。局部特征有助于识别边界/纹理，全局特征有助于消解重复纹理；后者为机制解释而非单独消融结论。
- Epipolar Sampling：参考像素确定一条空间射线；在另一图沿对应极线取样，位置编码来自候选深度，特征与编码拼接。两方向均执行，候选是几何限定的匹配集合而非全图任意点。
- Epipolar Attention：目标像素特征作 query，候选拼接特征作 key/value，交叉注意力残差更新后做图内自注意力。该模块重复两轮，使跨图信息在图内扩散，再参与下一轮跨图匹配。

### 正文关联与证据边界
§4.1 式(1)–(4)、附录A给出编码器构成。图未画出所有训练超参数，也没有给出完整通道维数或精确的投影实现；不能根据框图反推未披露的卷积核/层数。图8与图3衔接：输出的逐像素特征才进入高斯属性预测网络。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
## Figure 9：额外参考观测

![Figure 9](pixelSplat/Figures/Figure_09.jpg)

*来源：PDF 第14页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 9. Qualitative comparison of novel views given 2 and 3 reference views. Our method can be extended to use an arbitrary, fixed number of reference views as input. Here, we compare the results from a 3-view model with those from the original 2-view model. The 2-view model uses the top and bottom reference views as input, while the 3-view model uses all three.

**中文：** 使用两个和三个参考视图时的新视图定性比较。方法可以扩展到任意固定数量的参考视图输入。比较三视图模型与原始双视图模型；双视图使用最上和最下的参考图，三视图使用全部三张。

### 双视图与三视图
三行对应三组室内场景，与 Figure 5 中的室内示例相关。每行左侧三张参考图，随后为目标真值、双视图和三视图输出。比较应固定同一目标：第三张参考图处在原两参考视图之间，新增观测提供此前遮挡、弱纹理或长基线匹配困难区域的补充约束。可观察到三视图结果在部分边缘、纹理和结构处更接近目标图，但不能从这几行读出精确深度误差。

### 正文关联与证据边界
附录B把极线候选扩展为来自其余参考图的样本并集，并附加视图嵌入；每个参考视图仍生成自己的高斯，最后并集渲染。Table 3 的 RealEstate10k 结果从 26.09/0.863/0.136 变为 28.31/0.908/0.100（PSNR/SSIM/LPIPS）。比较对象是分别训练的固定输入数量模型，不是证明同一已训练双图模型无需修改即可接受任意数量，也不是任意第三视角位置的受控试验。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN FIG-10 -->
## Figure 10：室内外补充结果

![Figure 10](pixelSplat/Figures/Figure_10.jpg)

*来源：PDF 第15页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 10. More results on the Real Estate 10k dataset.

**中文：** RealEstate10k 数据集上的更多结果。

### 七组室内外补充案例
六列为参考图对、目标图、Ours、Du et al.、GPNR、pixelNeRF。第一行露台遮阳伞，注意伞沿与远处建筑；第二行厨房冰箱，注意柜体线条与冰箱边缘；第三行洗衣设备，注意圆形门与墙体轮廓；第四行卧室，注意床与墙面交界；第五行餐桌及吊灯，注意细杆状结构；第六行走廊植物，注意叶片与背景遮挡；第七行客厅窗户，注意窗框及明暗反差。各行目标图是同场景新视图，非输入参考图。作者方法在多个例子的轮廓与纹理上更接近目标，基线中可见不同程度的模糊、重影或几何错位。不要把这种观察写成所有像素都更准确。

### 正文关联与证据边界
附录D补充§5.2/Figure 5；与 Table 1 的 RealEstate10k 整体质量比较相互支持。图没有逐图指标、误差图或额外训练设置；不能由它确定每个场景的排名、测量绝对几何或判断置信度。

 此图为补充定性对比，无独立数学量。

<!-- PTH:END FIG-10 -->

<!-- PTH:BEGIN FIG-11 -->
## Figure 11：航拍补充结果

![Figure 11](pixelSplat/Figures/Figure_11.jpg)

*来源：PDF 第16页，已验收的 PDF 区域渲染展示副本（JPEG），不是原始矢量或位图文件。*

### Caption 原文与译文

> Figure 11. More results on the ACID dataset.

**中文：** ACID 数据集上的更多结果。

### 七组航拍补充案例
列布局同 Figure 10，第一列为上下堆叠的两参考视图。七行依次可见山地云雾与植被、海滨城镇、较大酒店/建筑群、绿色丘陵路径、海岸沙滩、红色屋顶村镇、城市公寓航拍。比较重点分别是远景层次与云雾、岸线和屋顶、重复建筑纹理、地形路径连续性、陆海边界、密集屋顶轮廓、成片建筑线条。完整模型在多个场景中保留较清晰的结构与纹理；基线在弱纹理、远景和重复结构中出现模糊或不一致。ACID 的航拍尺度和视角变化与 RealEstate10k 不同，这正是第二数据集的补充价值。

### 正文关联与证据边界
附录D与 Table 1 ACID 列。表中完整模型为 28.27 dB、0.843、0.146。图仅展示选定案例，不能证明无界场景、任意航拍高度、动态水面或极端遮挡的泛化；也不能从清晰的地形外观判断地形高程精度。

 此图不包含独立公式或逐图量化。

<!-- PTH:END FIG-11 -->

<!-- PTH:BEGIN TABLE-01 -->
## Table 1（PDF 第6页）

### Caption 原文与译文

> Table 1. Quantitative comparisons. We outperform all baseline methods in terms PSNR, LPIPS, and SSIM for novel view synthesis on the real-world RealEstate10k and ACID datasets. In addition, our method requires less memory during both inference and training and renders images about 650 times faster than the next-fastest baseline. In the memory column, we report memory usage for a single scene and 256 × 256 rays, extrapolating from the smaller number of rays per batch used to train the baselines where necessary. Note that we report GPNR’s encoding time as N/A because it has no encoder. We bold first-place results and underline second-place results in each column.

**中文：** 定量比较。我们在真实 RealEstate10k 与 ACID 数据集的新视角合成中，PSNR、LPIPS、SSIM 优于所有基线，推理和训练所需内存较低，渲染速度约为次快基线650倍。内存列统计单场景与256×256条射线，必要时由基线训练使用的较少射线数外推。GPNR没有编码器，因此编码时间为N/A。加粗第一名、下划线第二名。

### 完整定量表
| 方法 | ACID PSNR↑ | SSIM↑ | LPIPS↓ | RE10k PSNR↑ | SSIM↑ | LPIPS↓ | 编码(s)↓ | 渲染(s)↓ | 训练(GB)↓ | 推理(GB)↓ |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| pixelSplat | 28.27 | .843 | .146 | 26.09 | .863 | .136 | .102 | .002 | 14.4 | 3.002 |
| Du et al. | 26.88 | .799 | .218 | 24.78 | .820 | .213 | .016 | 1.309 | 314.3 | 19.604 |
| GPNR | 25.28 | .764 | .332 | 24.11 | .793 | .255 | N/A | 13.340 | 3789.9 | 19.441 |
| pixelNeRF | 20.97 | .547 | .533 | 20.43 | .589 | .550 | .005 | 5.294 | 436.7 | 3.962 |

### 比较与证据边界
Du 为两数据集全部质量指标的次优：ACID +1.39 dB、+.044、−.072；RE10k +1.31 dB、+.043、−.077。渲染单帧 1.309/.002=654.5 倍，注意编码本方法反而慢于 Du 和 pixelNeRF；新视图多次渲染时才摊销显式表示的编码成本。假设表值线性外推100帧，Du为130.916秒、ours为.302秒，约433.5倍，不能把正文§5.2关于100帧约650倍的表述直接等同于此表可复算的总耗时。训练内存是按单场景65536射线的比较口径，不是各训练运行实际峰值（见附录A）。PSNR/SSIM/LPIPS测量目标视图外观，不是几何GT。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
## Table 2（PDF 第8页）

### Caption 原文与译文

> Table 2. Ablations. Both our epipolar encoder and our probabilistic sampling scheme are essential for high-quality novel view synthesis. Depth regularization slightly impacts rendering quality.

**中文：** 消融实验。极线编码器和概率采样方案均为高质量新视角合成的关键。深度正则略微影响渲染质量。

### 完整消融表
| 方法 | PSNR↑ | SSIM↑ | LPIPS↓ | PSNR相对完整模型 |
|---|---:|---:|---:|---:|
| Ours | 26.09 | .863 | .136 | 0 |
| No Epipolar Encoder | 19.89 | .639 | .295 | −6.20 |
| No Depth Encoding | 24.97 | .830 | .158 | −1.12 |
| No Probabilistic Sampling | 24.62 | .828 | .171 | −1.47 |
| Plus Depth Regularization | 25.94 | .862 | .137 | −.15 |

### 比较与证据边界
去极线编码器的SSIM下降.224、LPIPS增.159，表示整个跨图匹配机制价值；去深度编码仍保留极线匹配，SSIM下降.033、LPIPS增.022，支持显式尺度信息的额外作用。去采样SSIM下降.035、LPIPS增.035，支持概率机制而非确定性位置回归。深度正则仅−.001 SSIM、+.001 LPIPS，并非提高这些外观指标；它被用于图4的深度/点云可视化。所有消融在RealEstate10k的相应条件下，不提供跨随机种子方差或度量深度准确率；不能将各差值相加解释成独立因果效应。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN TABLE-03 -->
## Table 3（PDF 第13页）

### Caption 原文与译文

> Table 3. Quantitative comparison of 2 and 3 view Real Estate 10k results. Given a third reference view located halfway between the two existing reference views, a 3-view variant of pixelSplat produces slightly better results.

**中文：** 两视图与三视图 RealEstate10k 结果的定量比较。给定位于原两个参考视图中间的第三个参考视图，三视图 pixelSplat 变体产生稍好的结果。

### 完整三视图对照
| 参考视图数 | PSNR↑ | SSIM↑ | LPIPS↓ |
|---|---:|---:|---:|
| 2 | 26.09 | .863 | .136 |
| 3 | 28.31 | .908 | .100 |

### 比较与证据边界
差值为 +2.22 dB、+.045、−.036。附录B中为固定数量输入而训练的三视图变体；新增的是原两视图中间的观测。该结果同时反映新增信息和模型输入结构变化，不单独隔离其贡献；不能外推到任意三视角布局或多视图规模。

<!-- PTH:END TABLE-03 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 图表之间的逻辑关系
### 问题—机制—实验链
Figure 1 定义输入、显式表示和输出；Figure 2 解释场景尺度问题，Figure 8 展开编码实现，Figure 6 查看匹配注意力，Table 2 的 No Epipolar Encoder 与 No Depth Encoding 分别测试整个模块和深度编码。Figure 3/Algorithm 1 解释概率采样如何借助不透明度反传，Figure 7 的 No Probabilistic Sampling 及 Table 2 验证其外观收益。Figure 4 显示显式原语可视化及深度（经正则微调）；Figure 5、10、11 展示两数据集输出，Table 1 汇总整体质量、耗时、内存。Figure 9/Table 3 是输入扩展的定性/定量配对。图7的深度正则列与表2最后一行也限定图4的条件。
### 证据层级
图1/2/3/8是机制示意，不直接测量效果；图4/5/6/7/9/10/11是选例定性证据，尤其图6权重聚集不是全部对应点的准确率；表2提供受控变体但未给随机种子误差；表1提供同协议的综合外观与效率结果；表3提供固定中间视角条件下的扩展证据。不能从渲染PSNR反推出绝对几何准确率，也不能从注意力热图证明采样期望梯度正确。

 本单元为证据组织，相关公式在数学单元。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 研究问题与任务定义
### 任务的形式化
【原文事实】§4 输入两张参考图及相机参数。设 $I_1,I_2\in\mathbb R^{H\times W\times3}$，相机内参 $K_i$、相机到世界外参 $T_i$。共享参数网络 $E_\theta$ 从图像对与相机预测 $G=\{(\mu_k,\Sigma_k,\alpha_k,S_k)\}$，新目标相机 $C_t=(K_t,T_t)$ 用固定的可微 Gaussian splatting renderer $R$ 得到 $\hat I_t=R(G,C_t)$。训练时跨大量场景最小化目标图像误差，目标 RGB 是监督而非输入；推理时未知新场景只需一次编码，无场景内迭代优化。两图每像素预测M个原语，若分辨率相同，合并前数目为 $2HWM$，实际M=3。显式原语可以在不同目标相机复用，且保留中心、形状、透明度、方向外观等可编辑属性。
### 科学问题与边界
问题不仅是如何渲染稀疏图像，而是如何将两幅有限投影中的几何与外观信息直接映射为一个可高效重用的显式辐射场。稀疏观测在遮挡、低纹理、重复纹理、反射区域产生不可唯一求解的几何与外观；跨场景学习提供先验，但不消除所有歧义。作者具体聚焦 SfM 姿态的任意尺度与局部高斯的优化困难。相机已知，不是 SfM/位姿估计任务；坐标与给定姿态一致，不是绝对米制恢复；图像监督能够约束可见辐射场，却不能唯一证明底层密度或真实表面。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 研究动机与根因
### 计算成本与表示
【背景知识】NeRF式渲染对一条射线多个位置查询密度/颜色，再积分或离散合成；若每像素采样N次，H×W视图需约HWN次场查询，训练及渲染代价大。光场方法直接根据目标射线聚合参考特征，减少显式3D场的需求，却仍需对新目标射线做网络运算。3DGS先建立有限个具有空间支撑的原语，可投影到屏幕空间并栅格化，场景编码与后续渲染分开。原版单场景3DGS依赖多图逐场景优化，不能直接从两图可靠恢复。pixelSplat的目标是同时获得前馈泛化、显式场景与快速渲染。
### 尺度歧义根因
SfM仅能从图像确定相似变换意义下的场景：若所有点和相机平移乘同一正数s，投影比值不变。若网络只看一张图，即便认出一把椅子的典型物理大小，也不知道 SfM 采用哪个任意单位；因此在两相机坐标中预测的同一物体深度可能不符合给定基线。另一参考相机的已知基线在SfM单位下提供匹配点三角化的深度。作者没有试图预测真实世界的s，而是把三角化候选深度编码给网络，使输出适配该坐标系。§4.1及图2、表2支持这一设计。
### 原语局部极小
Gaussian虽然数学上不具紧支撑，但远离中心若干标准差后权重与梯度迅速衰减。即使存在有用梯度，移动一个不透明原语到正确位置可能经过空域并遮挡背景，使沿途光度误差先升后降，简单局部优化难以跨越。原版3DGS以分裂、增殖、剪枝作自适应密度控制，这些离散操作不能直接作为预测网络常规可微输出路径。作者提出沿像素射线采样候选位置，同时绑定opacity与采样概率，从而无需每个原语沿空间连续移动即可探索新位置。它改善而非数学保证消除局部极小，训练依赖被采到的候选。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 相关工作定位
### 单场景方法
§2第一类涵盖体素与学习渲染、神经场与体渲染（NeRF [29] 等）、离散加速结构、3DGS [19]。共同点是依据场景的多幅已知相机图像，对表示做光度反传优化。NeRF 以坐标查询连续场，3DGS以高斯的显式属性和栅格化降低渲染代价；这并未自动解决少视角下的欠约束。本方法直接预测3DGS属性，训练成本跨场景摊销，推理不再逐场景优化。
### 先验驱动与光场
§2第二类包括使用代理深度的图像渲染、适合小基线的多平面图像（MPI）、对象级潜码的神经场、pixelNeRF [58] 等像素对齐特征、受MVS启发的cost volume以及GPNR和Du et al.光场Transformer。像素对齐保留特征空间局部性，避免单个全局潜码压缩整个场景；cost volume/极线匹配利用跨视角几何。光场Transformer根据目标射线产生输出，作者认为牺牲显式可解释3D以换取相较传统场方法的效率。pixelSplat借鉴像素对齐、极线注意力和3DGS，不以光场直接预测目标RGB，而预测可重用原语。注意本文对前作的描述来自§2，不是本次独立复现或完整前作精读。
### 尺度处理
单目深度方法以尺度不变损失评估/监督相对深度；单图3D扩散工作按深度统计重新缩放场景并条件化编码器。本文采用已知跨视图相机来标注候选深度，解决的是与输入SfM坐标一致的问题。既不依赖外部真实米制基准，也不是只训练一个尺度不变的深度输出。本文的创新侧重将几何尺度信息与显式原语预测衔接，而非首次提出极线、三角化或注意力。

 相关工作定位不独立推导前作公式。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-01 -->
## 方法（一）：数据流与图像特征
### 总体数据流
1. 输入参考图对与已知相机，分别单图编码 $F_1,F_2$。
2. 依赖相机几何为每个像素在另一图构造候选极线样本及三角化深度，双向交叉注意力更新，再图内自注意力；重复两轮（图8）。
3. 每个参考像素特征输入 f，预测深度概率、偏移、协方差和球谐。采样中心并设定不透明度（图3/Algorithm 1）。
4. 原语从相机系转换到共同世界系，取两图预测集合并集，不执行显式匹配去重。
5. 用目标相机的高斯栅格化生成RGB；训练误差反传到网络；推理阶段已生成的原语不因目标相机改变而重新编码。
### 单图特征编码
图8单图分支以 ResNet-50 与 DINO预训练 ViT-B/8 获取特征；图示局部tokens、全局token以及layer0至layer3输出、Project/Upsample。各路径在像素空间对齐再形成融合特征。ResNet卷积具有局部空间归纳偏置，ViT利用长程上下文；二者的互补是合理解释，论文未单独给出各路径消融。准确通道数、融合前投影与上采样实现不从图示猜测。
### 训练与推理接口
同一预测网络在大量场景上共享参数，训练目标视图与参考视图区分。目标监督可从多个视图惩罚位置与外观的不一致，但没有直接深度监督。附录A每个场景渲染4个目标，先MSE，后加LPIPS；原语本身是中间显式表示，不是另一个逐场景优化变量集合。推理目标相机只进入渲染器；编码成本每场景发生一次（Table 1），若切换参考图则需重新编码。

<!-- PTH:END METHOD-01 -->

<!-- PTH:BEGIN METHOD-02 -->
## 方法（二）：尺度感知的极线编码
### 极线候选与三角化
给定 I 中像素 u，其三维射线在另一图的投影构成极线。沿极线选取若干像素 $\tilde u_l$，每对候选像素与相机中心决定两条射线；利用给定姿态三角化得到参考相机深度 $\tilde d_{\tilde u_l}$。拼接另一图特征与其深度位置编码，再将本图特征映射到query，将候选映射到key/value（式1–2）。这种设计的关键不是先输出一个硬匹配深度，而是让跨图候选携带几何单位：若图像对应可靠，注意力即可将相应深度编码传入特征。
### 交叉注意力与自注意力
式3的softmax注意力对候选value做加权和，残差加回F[u]。它保留本图原有语义特征，并选择对同一三维点更相关的匹配与深度信息。另一方向同理。式4图内自注意力传播信息：遮挡、超出对方视野的像素不能直接可靠三角化，但可利用同一图中相关区域的线索。图8显示交叉与自注意力重复两轮。此处self-attention的“in-paint”是特征补全，不是生成所有遮挡区域的真实几何。
### 信息与计算边界
几何位置编码由相机和候选像素计算，不是输入真值深度。注意力高权重是模型内部匹配依据，不保证概率校准或对应必然正确。近纯旋转、短基线、重复纹理、非Lambertian与低纹理会弱化约束。设每像素L个候选、特征维C，交叉注意力候选内积约与HWLC成正比；不是全图HW×HW匹配，图内自注意力的具体效率取决于实现。更多参考图的候选并集会增大内存（附录B/C）；不能仅凭公式宣称任意输入规模线性总成本。

<!-- PTH:END METHOD-02 -->

<!-- PTH:BEGIN METHOD-03 -->
## 方法（三）：概率化的像素对齐高斯
### 像素对齐概率预测
基准式5直接回归深度，将中心约束在参考像素射线上。不同于全局自由3D原语，这利用每个输入像素的局部视觉信息，但仍有深度回归的局部极小问题。本文将近远平面间深度按逆深度分为Z个候选b；网络从F[u]产生经softmax归一化的离散概率φ，以及每桶偏移δ（式6–7）。另预测协方差Σ与球谐S（式8），每像素共享一组Σ、S。抽样得到z后通过深度与射线方向恢复3D均值。δ在论文中标为[0,1]，式7直接加到b；不能擅自补入未写出的桶宽乘法，见数学单元。
### Algorithm 1
输入：深度桶b，特征F[u]，参考相机中心o，射线方向d_u。按原算法五步重述：
```text
1. (φ, δ, Σ, S) = f(F[u])
2. z ~ Categorical(φ)
3. μ = o + (b[z] + δ[z]) d_u
4. α = φ[z]
5. return (μ, Σ, α, S)
```
第一步输出所有候选概率与偏移、协方差、球谐；第二步真正离散采样而非argmax或期望深度；第三步得到采样位置；第四步绑定概率与不透明度；第五步传给后续渲染。正文为M=1说明，训练实际独立抽样3次并使用α=φ_z/3（附录A.1），同像素可在同一桶重复采样。两参考图各自产生原语并合并。
### 局部极小与梯度边界
若采中位置适合解释目标图，增加不透明度可能降低渲染误差；由于α由φ选中分量决定，反传增加对应logit可提升下次采样概率。若位置导致遮挡或错误外观，损失可能驱动α下降。这样，候选位置变化靠重新采样而非沿连续空间搬动原语，探索机制与优化参数解耦。但 categorical采样本身不可微，绑定opacity提供条件于已采样索引的梯度路径，不是对离散位置严格可微重参数化，也非完整期望梯度的无偏估计证明。详细导数见MATH-04。

<!-- PTH:END METHOD-03 -->

<!-- PTH:BEGIN METHOD-04 -->
## 方法（四）：原语变换、可微渲染与扩展
### 原语属性和世界变换
Gaussian属性含均值μ（位置）、协方差Σ（方向与空间支撑）、不透明度α、球谐系数S（视角依赖颜色）。网络预测相机系属性，附录A式10通过相机到世界T、旋转R将均值与协方差变换到统一坐标，α不变，球谐系数使用旋转的Wigner D矩阵。两图原语取并集，无显式融合或去重（附录C）。注意式5/7已有以相机中心和射线表示的坐标写法，实现时不能将同一次坐标变换重复施加。
### 可微渲染与监督
【背景知识，非论文新公式】三维高斯经投影局部Jacobian近似得到二维椭圆；可见原语按深度顺序做前向透明度合成。设屏幕像素的第i个有效不透明度为 $a_i=\alpha_i G_i(u)$、颜色为 $c_i$，则 $\hat C(u)=\sum_i c_i a_i\prod_{j<i}(1-a_j)$（忽略背景项）。因此预测中心、协方差、opacity及SH均会影响渲染误差；opacity的梯度沿式7/Algorithm1的绑定传给φ。论文沿用Kerbl等[19]的可微栅格化，不把这里的背景公式冒称为作者式(1)–(10)。训练前150k步MSE，之后加权0.05的LPIPS；目标视角4张。MSE提供像素颜色约束，LPIPS提供学习特征上的感知约束。
### 多视图扩展
附录B：对于某参考像素，采样其余参考视图的极线，取候选集合并集；给不同来源样本加可学习的per-image embedding，帮助注意力区分视图。每视图仍独立产原语，合并渲染。固定输入数量可重新训练（3视图实验证据见表3）。附录C明确重复原语、不生成未见区域、编码内存随视图数增长。更多观测增加约束，但不是无条件地保证更准，且不证明双视图权重直接支持任意数量。

<!-- PTH:END METHOD-04 -->

<!-- PTH:BEGIN MATH-01 -->
## 数学分析（一）：相机尺度、投影和三角化
### 投影与相似尺度不变性
【背景推导】令相机中心为 $o_i$，世界到相机旋转为 $R_i^T$（$R_i$ 为相机到世界旋转），点为 $X$。像素齐次坐标满足 $\lambda_i\bar u_i=K_iR_i^T(X-o_i)$，其中 $\bar u_i=(u_x,u_y,1)^T$，$\lambda_i$ 为齐次尺度（标准针孔中为相机z深度）。令任意正数 $s$，$X'=sX$，$o_i'=so_i$，则 $K_iR_i^T(X'-o_i')=sK_iR_i^T(X-o_i)$。除以第三分量时s消去，所有图像像素不变。因此单图无法知道SfM的任意s；即使两图可以三角化，相对尺度仍由给定相机基线决定。该尺度是坐标系约定，不是现实世界绝对尺度。
### 极线三角化推导
参考射线 $X=o+td$，另一射线 $X=\tilde o+q\tilde d$，方向由内外参计算。设 $A=[d,-\tilde d]$，$b=\tilde o-o$。若理想对应，$A[t,q]^T=b$，两射线共面相交；噪声下可求最小二乘 $(A^TA)^{-1}A^Tb$（满列秩时）。若方向单位化，t为沿射线距离；未单位化时是射线参数。两方向夹角θ，$\det(A^TA)=\|d\|^2\|\tilde d\|^2\sin^2\theta$，近平行时退化。将基线b缩放s而方向不变，解t,q同乘s。因此候选三角化深度正好继承输入姿态尺度。极线由第二相机投影第一射线而得，所有理想对应满足共面约束；沿线候选尚非真实对应。纯旋转没有足够平移基线，低视差/遮挡/噪声使三角化不稳定。
### 证据边界
以上为理解§4.1、Figure 2和式1–2的针孔几何推导，非作者额外公式。作者以注意力从带候选深度的特征中选择对应信息，不是强制显式输出上述最小二乘解。故证明的是深度参数随SfM基线缩放，而非网络必然正确匹配、消除绝对尺度歧义。

<!-- PTH:END MATH-01 -->

<!-- PTH:BEGIN MATH-02 -->
## 数学分析（二）：式(1)–(4) 极线注意力
### 公式 1–2
原式(1)（PDF p3）：

$$\mathbf s=\tilde F[\tilde u_l]\oplus\gamma(\tilde d_{\tilde u_l})$$

$\tilde F[\tilde u_l]$ 为另一图候选l的特征向量，$\gamma$ 为位置编码，$\oplus$ 为通道拼接，$\tilde d$ 是由姿态与两个像素计算的深度。若特征C维、编码P维，s是C+P维。原式(2)：

$$\mathbf q=QF[u],\quad \mathbf k_l=K\mathbf s,\quad \mathbf v_l=V\mathbf s$$

这里Q,K,V是学习的线性嵌入矩阵（K与相机内参同符号，语境不同）；q、k须同维d_k，v维度为d_v。位置编码不是预测真值，而把候选尺度作为可学习信号；投影矩阵可融合特征相似性与深度线索。
### 公式 3–4
原式(3)（PDF p4）：

$$F[u]\mathrel{+}=\mathrm{Att}(\mathbf q,\{\mathbf k_l\},\{\mathbf v_l\})$$

【背景展开】标准缩放点积注意力 $w_l=\exp(q^Tk_l/\sqrt{d_k})/\sum_m\exp(q^Tk_m/\sqrt{d_k})$，输出 $\sum_lw_lv_l$，多头时再拼接与投影。残差将该输出加到原像素特征，避免丢掉单图信息。由于v由拼接向量线性映射，若 $V=[V_F,V_\gamma]$，则输出为 $V_F\sum_l w_l\tilde F_l+V_\gamma\sum_lw_l\gamma(\tilde d_l)$。这说明深度编码信息可跨图传播，但加权编码一般不等于编码后的加权深度。梯度经w与v传入Q/K/V及两图编码器，不要求离散选择正确匹配。原式(4)：

$$F\mathrel{+}=\mathrm{SelfAttention}(F)$$

图内自注意力通过其他像素的value传播上下文，因此不能跨图三角化的区域也可能获得与场景尺度一致的深度线索。图8两轮交叉/自注意力可重复传播。
### 条件与局限
论文未披露位置编码的频率、确切通道和所有注意力细节，本节标准点积展开作为教学说明。Figure 6的少量匹配示例和Table 2的去编码消融支持该机制，但并非深度分布校准或匹配准确率统计。遮挡、反射、极短基线等仍限制可靠性。

<!-- PTH:END MATH-02 -->

<!-- PTH:BEGIN MATH-03 -->
## 数学分析（三）：式(5)–(8) 位置及属性参数化
### 公式 5
PDF p5 的确定性基准：

$$\mu=o+d_u\,\mathbf d,\quad d_u=g(F[u]),\quad \mathbf d=TK^{-1}[u,1]^T.$$

o为参考相机中心，g回归正深度，T为外参、K为内参。论文把射线方向写作T作用于反投影向量。作为方向应用时应只使用旋转部分（齐次方向末分量0），而不是再加相机平移；若采用单位化方向d则深度是距离，若采用未单位化标准针孔反投影则参数为z深度。原文称distance，未明确式5归一化细节，本报告保留其符号并区分两种几何约定。中心被限制在参考射线上，减少三维自由度但仍需跨空间优化深度。
### 公式 6–7
原式(6)：

$$b_z=\left[(1-z/Z)(1/d_{\rm near}-1/d_{\rm far})+1/d_{\rm far}\right]^{-1}.$$

$0<d_{\rm near}<d_{\rm far}$，Z为桶数。记 $a=1/d_{\rm near}-1/d_{\rm far}>0$，$b_z=(1/d_{\rm near}-az/Z)^{-1}$。连续延拓导数 $db/dz=(a/Z)b_z^2>0$，故桶在深度空间递增，远处分辨率更粗；在逆深度空间等间距。z=0给near、z=Z给far，但长度Z的离散索引不可能同时包含0与Z；论文未明确索引端点，不能猜测具体实现。例：near=1、far=10、Z=4时z=0,1,2,3,4的连续延拓为1、1.290、1.818、3.077、10；这只是数学示例，并非本文配置。

原式(7)：

$$\mu=o+(b_z+\delta_z)\mathbf d_u,\quad z\sim p_\phi(z),\quad (\phi,\delta)=f(F[u]).$$

φ为Z维概率向量，$p_\phi(z)=\phi_z$，各分量非负和为1；δ为[0,1]^Z的桶内调整。若预测logitℓ，则 $\phi_j=e^{\ell_j}/\sum_k e^{\ell_k}$。原文称δ调整桶内位置，但式7直接把δ加到b_z，未显示与不等宽桶宽的乘积，按字面无法保证每桶不越界；该单位/实现细节应留作复现核查，不擅自改为插值公式。抽样z离散，不能把其替换为深度期望或argmax后仍称相同算法。
### 公式 8
PDF p6：

$$(\phi,\delta,\Sigma,S)=f(F[u]).$$

每像素预测深度分布、偏移、一个协方差及一套SH。Σ为3×3协方差，按定义应半正定（有效椭球通常正定），S为方向相关颜色的球谐系数；本文未给出所有形状参数化、SH阶数。Algorithm1采样后以φ_z设α，输出(μ,Σ,α,S)。同像素多样本共享Σ/S而均值和α依抽样结果变化。

<!-- PTH:END MATH-03 -->

<!-- PTH:BEGIN MATH-04 -->
## 数学分析（四）：概率采样梯度的严谨边界
### 采样与梯度
【原文事实】§4.2 将抽样得到的桶z的opacity绑定到φ_z：$\alpha=\phi_z$，作者用 $\nabla_\phi L=\nabla_\alpha L$ 说明反馈路径。严格地对固定已采样索引z、暂不考虑共享网络的其他输出路径，有 $\partial\alpha/\partial\phi_j=\mathbf1[j=z]$，$\partial L/\partial\phi_j=(\partial L/\partial\alpha)\mathbf1[j=z]$。因此并不是整个概率向量梯度等于标量opacity梯度，而是选中分量获得直接梯度。若φ由logit ℓ softmax，$\partial\phi_z/\partial\ell_j=\phi_z(\mathbf1[j=z]-\phi_j)$。链式法则得

$$\frac{\partial L}{\partial\ell_j}=\frac{\partial L}{\partial\alpha}\phi_z(\mathbf1[j=z]-\phi_j).$$

若$\partial L/\partial\alpha<0$，梯度下降在该分支增加选中logit相对其他logit，提高未来采样概率；反之降低。这是作者直观论证的局部成立条件。梯度还可能通过δ、Σ、S及共享网络参数影响结果，不能断言每一步正确深度概率必增。
### 期望梯度的边界
【独立推导】设单抽样目标 $J(\theta)=\sum_z p_\theta(z)L(z,\theta)$，其中L依赖opacity与连续属性。直接求导：

$$\nabla_\theta J=\sum_z p_\theta(z)\nabla_\theta L(z,\theta)+\sum_z L(z,\theta)\nabla_\theta p_\theta(z)$$

$$=\mathbb E_z[\nabla_\theta L(z,\theta)+L(z,\theta)\nabla_\theta\log p_\theta(z)].$$

第一项是固定z的连续梯度；第二项来自抽样分布本身变化。作者通过α=φ_z提供第一项中的概率相关路径，未给出对第二项的估计或证明其恒为零。多独立抽样时同理，score项由各样本log概率求和。故“making sampling differentiable”应理解为可训练的代理路径，不等价于严格对categorical位置可微，也不保证对上述期望目标的无偏梯度。采样概率很低时正确候选仍可能长时间未被抽到。Table2验证外观收益，不是数学收敛证明。
### 多样本不透明度
附录A.1实际M=3：独立 $z_m\sim p_\phi$，$\alpha_m=\phi_{z_m}/3$；固定样本时上式梯度乘1/3，再对三个样本分支求和。对任意M，$\sum_m\alpha_m\le1$，期望为 $\sum_z\phi_z^2$（因 $E[\phi_z]=\sum_z\phi_z^2$），只有概率高度集中才接近1。实际前向合成还非线性：若三原语在像素处footprint均为1且α=1/3，则总有效遮挡 $1-(1-1/3)^3=19/27\approx0.704$，不是1。作者“roughly 1”是直观尺度说明，不是任意分布及实际合成的严格等式。这个分析有助于理解概率与opacity的双重作用，同时不可据此推断实际代码未披露的处理。

<!-- PTH:END MATH-04 -->

<!-- PTH:BEGIN MATH-05 -->
## 数学分析（五）：式(9)–(10) 深度正则与世界坐标
### 公式 9
附录A.1 p12原式，以像素u=(x,y)、渲染深度图D和图像I表示：

$$D_x^\Delta[u]=D[x-1,y]-2D[x,y]+D[x+1,y]$$
$$D_y^\Delta[u]=D[x,y-1]-2D[x,y]+D[x,y+1]$$
$$I_x^\Delta=\max(|I[x-1,y]-I[x,y]|,|I[x+1,y]-I[x,y]|)$$
$$I_y^\Delta=\max(|I[x,y-1]-I[x,y]|,|I[x,y+1]-I[x,y]|)$$
$$\mathcal L_{\rm reg.depth}[u]=D_x^\Delta[u]\exp(8I_x^\Delta[u])+D_y^\Delta[u]\exp(8I_y^\Delta[u]).$$

D的两项是有符号二阶有限差分，线性深度坡面内为零；I项为两侧最大绝对色差，常数8控制权重变化速率。特别核对实际PDF：指数是正号，D二阶差分未取绝对值。按字面正指数在大色差时增大权重，符号差分也可能相消或为负，与常见非负、边缘抑制型TV表达不一致。作者称edge-aware TV，但本文不擅自更改为负指数、绝对值或平方，也不声称实际实现采用何者。边界像素、RGB通道归约与全图损失聚合未明示。作者使用50k步微调得到图4的点云，表2外观略降。
### 公式 10
附录A.1 p12原式：

$$\mu_{\rm world}=T\mu_{\rm cam},\qquad \Sigma_{\rm world}=R\Sigma_{\rm cam}R^T,$$
$$\alpha_{\rm world}=\alpha_{\rm cam},\qquad s_{\rm world}=D_Rs_{\rm cam}.$$

T为4×4相机到世界矩阵，R为3×3旋转，t为平移。第一式采用齐次坐标，即非齐次为$R\mu+t$。设随机向量$X\sim\mathcal N(\mu,\Sigma)$，$Y=RX+t$，则$E[Y]=R\mu+t$，$\mathrm{Cov}(Y)=E[R(X-\mu)(X-\mu)^TR^T]=R\Sigma R^T$。不透明度是标量，不因坐标旋转变换；SH颜色在旋转后的观察方向上应保持同一辐射函数。对每个阶l，球谐基在旋转下由$(2l+1)\times(2l+1)$的Wigner D表示变换；全阶拼成块对角$D_R$，RGB通道分别作用。作者采用e3nn库并调整原3DGS代码以符合其约定。具体实/复基、主动/被动旋转及系数排序需一致；本节不能从矩阵符号推断代码实现。
### 解释与复现边界
式9按原文照录疑点，不将常识替代证据；式10是两图原语在公共坐标系合并与视角颜色保持的数学基础。它不自动校正错误相机尺度或消除重复预测。

<!-- PTH:END MATH-05 -->

<!-- PTH:BEGIN EXP-01 -->
## 实验（一）：数据集、训练协议与评价
### 数据与协议
RealEstate10k为网络住宅漫游视频，ACID为航拍景观视频；均使用SfM姿态与官方训练测试划分。作者为兼容Du基线的限制，所有模型训练/评价均用256×256。测试目标是所选两参考帧之间的视频帧，即插值型宽基线NVS，不能视为任意外插相机的测试。各基线以公开代码、相同数据集与本方法加载器重训；训练课程前150k步将参考帧间隔线性从25提高到45。作者未在正文提供全部场景抽样及每个split样本量，故不猜测。
### 训练设置
| 模型 | 训练步数 | 批量 | GPU及实际显存 | 主要设置 |
|---|---:|---:|---|---|
| pixelSplat | 300k | 7 | 单卡约80GB | Adam；150k步前MSE，之后加0.05 LPIPS；每场景4目标；每像素3高斯 |
| Du et al. | 300k | 32 | 4卡各约44GB | 作者默认超参；150k后LPIPS |
| pixelNeRF | 500k | 12 | 单卡约20GB | NMR默认超参，禁首池化；near=.1/far=10（仅此基线配置说明） |
| GPNR | 250k | 4098 | 6卡各约67GB | 默认超参，LR从3e-4改为1e-4（原LR多次训练崩溃） |
原文§5.1明确ResNet-50、ViT-B/8均DINO预训练，逐像素输出相加。两轮极线cross-attention。附录A.1三抽样与α=φ_z/3，深度正则变体额外50k步微调。不同批量单位未统一为场景数（GPNR批量尤其不能直接与ours场景数相除）。正文没有披露全部超参，不能据本表复现所有训练。
### 指标与可比性
PSNR为像素误差的对数度量（背景知识：10log10(MAX²/MSE)，越大越好），SSIM度量结构相似性（越大越好），LPIPS学习特征感知距离（越小越好）。目标GT是视频RGB，并不是三维真值。编码耗时每个场景一次，渲染/解码耗时每目标帧一次。Table1训练及推理内存为单场景256²射线统一口径，必要时外推基线较少训练射线数。该表与实际运行的卡数、batch、峰值显存不同；因此可比较单位场景/射线开销，而不能把14.4GB当作作者完整训练只需14.4GB。

<!-- PTH:END EXP-01 -->

<!-- PTH:BEGIN EXP-02 -->
## 实验（二）：主结果、效率与显式表示
### 主实验结果
Table 1的完整原数值已在表单元列出。对ACID，pixelSplat相对Du/GPNR/pixelNeRF的PSNR分别+1.39/+2.99/+7.30，SSIM+.044/+.079/+.296，LPIPS−.072/−.186/−.387。对RE10k，PSNR+1.31/+1.98/+5.66，SSIM+.043/+.070/+.274，LPIPS−.077/−.119/−.414。三个指标均在两个数据集优于所有基线；感知指标的下降与图5边界、纹理更清晰相符。没有误差条，不能判断每个样本都赢或统计显著性。
### 效率与成本
编码.102s比Du .016及pixelNeRF .005更慢；GPNR无encoder。渲染.002s约等于500fps的表中倒数（非完整实时系统帧率），比Du 1.309s快654.5倍。以N帧总成本T_enc+N T_render估算，N=100时ours .302s、Du130.916s，约433.5倍；原文正文称100帧约650倍，与表中的单帧倍率一致而与该表的总时长计算不完全一致，保留原文并注明口径，不猜测未披露计时。训练比较口径14.4GB vs 314.3/3789.9/436.7GB，推理3.002GB vs 19.604/19.441/3.962GB，均低于基线。数值为单场景65536条射线的口径和可能外推，不是训练机配置（附录A实际约80GB）。
### 定性与几何边界
Figure5及10/11示例支持细节与仅一幅参考可见区域的外观恢复，溪流是作者标注的分布外样例，但无OOD专门划分定量。Figure4经深度正则微调50k后，以远离训练视角的观察位置展示高斯与两参考深度图，说明模型具有显式3D表示。作者同时承认反射表面常透明、极外推视角出现billboard。清晰NVS与深度连续并非绝对深度准确证据，尤其图4与主表模型不同，不能把图4的深度视觉质量当作完整模型的独立几何评测。

<!-- PTH:END EXP-02 -->

<!-- PTH:BEGIN EXP-03 -->
## 实验（三）：消融设计与解释
### 消融问题与控制
§5.3在RealEstate10k回答三问：Q1a极线encoder是否负责处理尺度（两图独立编码的变体）；Q1b显式深度编码是否是其有效机制（保留极线匹配，移除深度位置编码）；Q2概率预测是否缓解直接回归局部极小（改为确定性深度回归）。另有深度正则微调变体，展示可调几何/外观权衡。Q1a与Q1b嵌套，不能把两者差异视为完全独立因素。
### 数值与定性
完整模型26.09/.863/.136；去极线19.89/.639/.295（−6.20/−.224/+.159）；去深度24.97/.830/.158（−1.12/−.033/+.022）；去概率24.62/.828/.171（−1.47/−.035/+.035）；正则25.94/.862/.137（−.15/−.001/+.001）。图6的极线注意力峰值在少量正确候选处，说明跨图信息可能确实与对应有关；图7去极线重影，去采样斑点，正则外观接近基准。作者用尺度歧义、局部极小解释这些结果。
### 因果解释边界
去整个encoder损失最大，但同时失去跨图外观和几何聚合，不能精确归因于尺度独自贡献。Q1b在保留匹配的基础上提供更直接的深度编码证据，仍未测量尺度误差。去概率预测验证外观指标的实际价值，但没有观察优化地形、抽样命中率或收敛至全局最优。正则微调未显示图像指标收益，也无独立几何真值支持几何精度提高。未提供多seed方差、不同桶数和M对比；这些是本次独立阅读的未验证问题，并非作者报告的实验。

<!-- PTH:END EXP-03 -->

<!-- PTH:BEGIN EXP-04 -->
## 实验（四）：三视图扩展与补充结果
### 更多参考视图
附录B中从双图扩展为固定三图模型：对给定参考像素从其余两图取极线样本，使用并集及view embedding聚合，再将三图原语并集用于渲染。新增参考位于原两个的中间，图9固定目标真值比较两/三视图。Table3：2视图26.09/.863/.136，3视图28.31/.908/.100，即+2.22/+.045/−.036。观测增加有助于遮挡补全及视差匹配，但同时模型输入结构不同，不能从该实验独立量化各因素。
### 附录补充实验
Figure10 七组RealEstate10k的室内外生活场景（露台、厨房、洗衣、卧室、餐桌、走廊、窗户）和Figure11 七组ACID航拍场景（山地、海岸、建筑、丘陵、沙滩、屋顶、城市），均按Ref/Target/Ours/Du/GPNR/pixelNeRF六列排布。与图5的五个主文样例一起，提供定性覆盖而不另设新评价协议。多个案例中ours边缘纹理更完整，不能以这14组补充图代替全测试集统计。
### 条件与限制
只在RealEstate10k测试该中点第三视角扩展，未证明任意布局、更多视角或不固定数量的训练/推理可用性。附录C特别指出多视图候选导致极线注意力内存成本过高、每图高斯并集造成重复、不生成未见区域。该实验只支持更多可用观测在所测设定下有收益。

<!-- PTH:END EXP-04 -->

<!-- PTH:BEGIN SYNTHESIS-01 -->
## 综合评价（一）：贡献与证据强度
### 主张—证据矩阵
| 主张 | 机制/数学 | 实验依据 | 能支持的结论与不足 |
|---|---|---|---|
| 前馈显式重建 | Fig1、4.2、Alg1、式10 | Table1、Fig4 | 单次预测显式原语、外观高质量；不证明metric几何 |
| 与SfM尺度一致 | Fig2、式1–4 | Fig6/7、Table2 Q1a/Q1b | 匹配与深度编码改善NVS；无尺度误差直接测量 |
| 缓解原语局部极小 | Fig3、式5–8、α绑定 | Fig7、Table2 Q2 | 概率版优于点回归；无收敛/无偏梯度证明 |
| 高效重用渲染 | 3DGS显式栅格化 | Table1 | 单帧约654.5倍；100帧总成本口径见EXP-02 |
| 更强新视图外观 | 端到端跨场景学习 | Table1、Fig5/10/11 | 所测插值视频测试优于基线；未证明任意OOD |
| 输入扩展 | 附录B | Fig9/Table3 | 中点额外观测有收益；固定3图，非任意输入规模 |
### 创新与继承
原始高斯属性和可微栅格化源自3DGS；像素对齐、极线匹配、位置编码、注意力、SH不是本文首创。本文贡献是让图像对条件下的前馈网络预测可重用3DGS：将姿态尺度的候选深度编码进特征，并以离散深度采样与opacity耦合实现原语位置探索和梯度传递。模型集成和两数据集成本/质量比较具有实验贡献。不能将作者关于首次单次前馈预测的主张写成经过本次外部文献检索证明的优先权结论。

<!-- PTH:END SYNTHESIS-01 -->

<!-- PTH:BEGIN SYNTHESIS-02 -->
## 综合评价（二）：局限、复现问题与覆盖审计
### 论文明确局限
附录C：各参考图高斯简单并集，重复预测没有去重；不处理未见部分的生成建模；更多参考图使极线注意力内存开销难以接受。§5.2：反射表面往往透明，极外推视角下出现billboard状高斯，类似原版3DGS失败模式。结论提出未来与扩散模型结合并研究无相机位姿输入，不能写成本论文已解决。
### 未证实问题
【独立评价】①RGB插值外观不唯一约束几何，无绝对尺度、GT深度/表面指标；②已有相机姿态误差、弱纹理、遮挡与非Lambertian匹配可靠性没有系统检验；③离散采样的opacity反馈并不严格包含完整期望梯度，低概率模式可能仍难探索；④概率是否校准、每像素3个高斯是否最佳、桶数/深度区间敏感性未提供系统实验；⑤式7偏移与桶宽关系、式9正指数与有符号差分、表1与100帧效率口径存在需进一步实现核查的细节。以上不是否认论文效果，而是区分可复现事实与待检验解释。
### 覆盖与来源审计
基于本地arXiv:2312.12337v4 PDF（16页）与同版本HTML：标题/摘要/引言/相关工作/背景/方法4.1–4.2/实验5.1–5.3/结论及附录A–D均已进入对应单元；参考文献用作本文内部定位，不声称阅读其他论文全文。Figure1–11均引用B2验收展示副本并包含原caption与译文及图文关联；Table1–3完整原数值及caption和翻译；Algorithm1五步；式1–10及必要数学展开。报告区分原文事实、背景推导、独立评价和未证实事项。当前最新arXiv版本实时不可核实，CVF网页403；本地v4身份已确认。HTML非科学资源缺项记录在B1。原图JPEG是PDF区域渲染副本而非原始矢量。没有使用外部图片搜索、没有检查代码或实际重训；最终论文复现尚未执行。

<!-- PTH:END SYNTHESIS-02 -->



---

# 第二部分：学习与研究笔记

# pixelSplat 学习与研究笔记

> 精读报告 SHA-256：ca86f5f98fb415c46846dbe5eb5259f82211c3117b23b9540bad9d20e1bfd885
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## CONCEPTS：关键概念的内化与拓展
### 1. 尺度一致性与度量尺度不是同一问题
锚点：精读 MATH-01、§4.1/Fig2。相机中心及点整体乘s不会改变投影。若网络预测深度10而基线在数据坐标中为0.1，预测视差约$fB/Z$，例如f=200像素，结果2像素；真实场景在同一坐标中若Z=1，则视差20像素。单图可以猜测相对形状却不知道这种任意坐标单位。已知基线和匹配视差给出 $Z=fB/\Delta u$（仅平行校正双目）。这个约束不需要知道B的真实米数。若B同乘s，则Z同乘s。若误差为σ像素，小扰动下 $\sigma_Z\approx Z^2\sigma/(fB)$：远景、小基线的深度误差被放大。非校正相机应用通用射线几何，不照搬这一公式。
反事实：去掉相机基线信息的注意力仍可能学到纹理对应，却难以知道输出在什么坐标单位中；去掉所有跨视图编码还会损失外观聚合。因此表2两项消融并不完全等价。
### 2. 候选分布与概率校准
锚点：精读 MATH-03/04、§4.2/Fig3。输出概率是“在哪里采样原语”的参数，不一定是正确表面深度的后验概率。考虑两候选深度1和5，概率各0.5：均值3可能落在空域，而随机采样会落在两种假设之一。若真实候选当前概率p，每次独立M抽样至少命中一次的概率为 $1-(1-p)^M$。p=.02、M=3仅约5.88%；增加M虽扩大探索机会，也提高原语数和成本。此为独立数学例子，不是论文统计。
反事实：直接argmax更稳定但阻断索引梯度及探索；概率期望可微却可能平均互斥模式；离散采样加opacity梯度能探索但不能完整表示期望目标梯度。若采用显式score-function梯度可理论上补全分布变化项，却可能高方差、需设计基线；若用连续松弛，软位置可能在空域。均为待比较方案。
### 3. opacity与存在概率的语义冲突
锚点：精读 MATH-04。高斯的α同时决定投影贡献和未来采样机会。渲染误差对α的偏好不必等于几何正确概率：例如一个错误深度的原语如果颜色恰巧解释训练目标，仍可能被强化。对于两互相遮挡的原语，前方有效α改变后方透射率，梯度不只是局部颜色误差。建议将候选置信度、物理/表示不透明度、可见性作为不同变量理解。独立假设：可分离这些变量并以匹配一致性或目标残差约束概率，能否提升几何准确性？需要已知几何及严格同成本对照，不能凭变量分离就认定优越。
### 4. 显式可重用表示与可观测性
锚点：§3、§5.2/附录C、精读 METHOD-04。显式原语使视点切换成本主要变成栅格化，但显式不等于正确；多套不同的密度与SH可以产生相似RGB。若两个相机仅看见同一不透明物体正面，其背部颜色无法由这两图唯一确定。对背面作合理填充属于先验推断而非观测重建。有限参考的可观测性、像素颜色的可辨识性、跨场景泛化和目标视图外观质量是四个不同概念。可通过观测覆盖分区、可见区域与不可见区域分别评估避免混淆。


## CROSS-PAPER：潜在跨文献关联索引
### Topic: Geometry Inference（姿态尺度一致性）
- **Keywords:** Epipolar Attention; Triangulation; SfM Scale Ambiguity; Depth Positional Encoding
- **Highlight:** 利用相机姿态计算极线候选深度并拼接其位置编码，让跨视图注意力将SfM单位内的深度线索写入像素特征，而非恢复绝对米制尺度。
- **Evidence:** §4.1、Fig2/6/8、式1–4、Table2 Q1a/Q1b。
- **Connection Points:** 候选构造；相机尺度；深度编码；匹配置信度；遮挡。
- **Open Questions:** 位姿误差、弱基线、重复纹理如何影响深度特征一致性？
### Topic: Inference & Optimization（原语位置的随机探索）
- **Keywords:** Categorical Depth Sampling; Pixel-aligned Gaussians; Opacity Coupling; Local Minima
- **Highlight:** 预测离散候选概率，通过抽样确定中心并绑定概率与opacity，绕开对单个中心的连续移动；固定抽样的连续梯度不等于完整期望梯度。
- **Evidence:** §3、§4.2、Fig3、Alg1、式5–8、Table2、附录A.1。
- **Connection Points:** 探索机制；梯度估计；概率校准；原语预算；遮挡梯度。
- **Open Questions:** 在相同原语数下，哪些梯度和采样机制能降低几何失败率？
### Topic: Scene Representation（显式前馈与复用渲染）
- **Keywords:** Feed-forward Reconstruction; 3D Gaussian Splatting; Spherical Harmonics; World-space Transformation
- **Highlight:** 双图独立产生像素对齐原语并统一到世界系取并集，编码后可重用栅格化；没有显式去重。
- **Evidence:** §4、Fig1/4、式10、Table1、附录C。
- **Connection Points:** 显式/隐式表示；共享场景表示；重复原语；摊销计算。
- **Open Questions:** 如何在保持外观与速度时减少重复原语并加强跨视图几何一致？
### Topic: Observability & Identifiability（有限观测和不可见区域）
- **Keywords:** Sparse-view NVS; Occlusion; Unseen Regions; Billboard Artifacts; View-dependent Appearance
- **Highlight:** 输入为已知姿态的图像对，主测试为两参考间视频帧；作者明确未建模未见部分，并观察反射透明与外推billboard。
- **Evidence:** §5.1/5.2、Fig4/5、附录C。
- **Connection Points:** 观测覆盖；先验与证据；插值/外插；几何与外观指标。
- **Open Questions:** 如何区分观测内几何可靠性与不可观测区域的生成合理性？
### Topic: Efficiency & Scalability（固定多视图扩展）
- **Keywords:** Multi-view Attention; View Embedding; Memory Scaling; Amortized Rendering
- **Highlight:** 多图候选并集和可学习视图嵌入支持固定数量输入；新增中点图像改善结果，但多图注意力内存及原语冗余仍受限。
- **Evidence:** Table1/3、Fig9、附录B/C。
- **Connection Points:** 参考图数；视角布局；内存；候选剪枝；多视图融合。
- **Open Questions:** 能否在固定计算预算下选择最有几何信息的候选观测？


## RESEARCH-DIRECTION：一般研究方向与技术迁移
本次按论文公开问题展开一般研究分析，不把未明确要求发布的个人科研设想写入隐藏完整版。以下是公开可讨论的研究线索，未开展跨文献独立查新。
### 层级区分
| 层级 | 本篇定位 | 不等价概念 |
|---|---|---|
| Field | 三维视觉、神经渲染 | 不是仅3DGS优化 |
| Task | 从参考图构建显式辐射场并合成新视图 | 不是无姿态重建或绝对尺度测量 |
| Task Setting | 已知SfM相机，双图/固定多图，跨场景学习 | 不是任意视角、任意域 |
| Research Problem | 有限视角下的尺度一致、原语推断与效率 | 不等同于采用某一网络 |
| Method | 极线深度编码、随机采样、opacity耦合与splatting | 方法是候选解，不是研究问题本身 |
### 可迁移机制及条件
第一，候选几何与匹配证据联动：已知相机才能构造可靠极线；当姿态或内参有误，真对应可能离开候选集合。可设计软带搜索或联合估计，但会扩大搜索复杂度，需要额外监督/正则。第二，概率假设替代单一深度：适合重复纹理、部分遮挡等多解情形；若概率仅由光度损失强化，不能把它解释为校准置信度。第三，编码一次并渲染多帧：适用于多目标复用，不意味着单帧端到端最快；如果编码时频繁变更参考图或目标依赖的细化，摊销优势会变化。第四，多图融合：附录B扩展候选，C提出冗余与内存。候选稀疏化、视图选择、原语合并是不同的计算路径，应以相同预算和相同观测比较。
### 观测—先验冲突及适用范围
对于几何与外观同时变化的场景，匹配相似度不总能代表深度正确。动态物体、镜面反射、曝光、参与介质引起视角变化，都可能让正确对应呈现不同外观。把低匹配分数直接归因于错深度会产生偏差；简单忽略特征也会损失几何信息。可检验是否在有可靠多视图几何约束时优先信任观测，在遮挡或冲突时采用学习先验，并单独校准误差。此为一般迁移假设，不是pixelSplat已经建模或验证这些条件。明确不同场景的成像机制，才能决定其几何假设是否成立。


## RESEARCH-PROBLEMS：可证伪问题候选
以下均为由冻结报告与论文开放边界提出的候选，不代表原创性已经确认。优先级按现有证据、可控性与验证成本排序，而非论文影响力预测。
### RP-A：概率预测是否表达几何可靠性，而不只是渲染有用性？（优先）
现象：§4.2概率与opacity耦合、Table2显示外观提高，但作者未提供概率校准或GT深度。科学未知：在外观可由错误几何补偿时，概率是否会集中于错误位置？假设：在重复纹理、遮挡或视角相关外观中，单纯渲染反馈使概率置信度与几何误差失配；加入几何一致性信号或分离opacity与候选置信度能减少失配。验证：构造GT相机/深度合成场景，控制基线、纹理、遮挡、BRDF；使用同一编码器、桶数、原语数、训练量，比较原机制、直接深度回归、置信度与opacity分离、带几何一致性分支。评价深度相对误差、阈值内准确率、概率质量落在真值邻域的比例、可靠性曲线/Brier或NLL（需定义GT桶）、NVS PSNR/LPIPS、显存。成功需在多个难度下同时降低几何误差及校准误差，并报告外观与成本权衡；若原机制校准同等良好或改进仅靠新增监督，则反驳假设或限制其范围。可行性：有GT的数据可控；真实域无深度真值，不能以RGB替代。需后续查不确定性估计、概率深度、多视图一致性相关工作。
### RP-B：候选匹配对成像差异与位姿误差的敏感性是否可分离？（优先）
现象：图6与表2支持极线深度编码，但未系统改变姿态、成像条件。未知：在几何对应正确而外观不一致，或外观相似但相机错误时，注意力是否产生相同失败？假设：候选几何误差和外观差异会产生不同的注意力响应模式，基于可见性/几何一致性的可靠性估计能提升候选选择。验证：同一场景单独改变相机扰动、颜色/曝光变化、反射、遮挡，再做两因素交叉；保留已知真对应。基线原模型、去深度编码、扩大候选带、使用几何置信度，固定采样数及计算预算。评价真对应召回率、深度误差、校准、NVS、对噪声强度的退化曲线。若无法区分两种干扰或性能只在训练干扰分布有效，则假设不成立/不具泛化性。复现时先验证纯旋转等不可观测条件，不能要求网络凭空恢复。后续查鲁棒匹配、联合姿态和深度估计。
### RP-C：可重用显式原语的冗余能否随观测增加而受控？（次优）
现象：附录C指出每视图高斯并集有重复、更多图导致注意力内存开销；Table3仅验证固定3视图。未知：冗余来自多图观察同一表面还是概率采样重复，以及何时冗余有益于外观表达。假设：在共享几何一致区域适当融合原语，并按基线/可见性筛选候选，可在固定外观质量下减小原语与注意力开销；遮挡边缘及非Lambertian处可能不宜强制融合。验证：固定场景及目标集合，改变参考数2/3/5/8及布局（中点、分散、聚集），比较朴素并集、仅候选筛选、仅原语融合、二者组合；控制训练集、目标采样、总预算。评价单位场景原语数、内存峰值、编码和N帧总时间、几何指标及可见/遮挡区外观。成功标准为同质量成本明显降低，或同预算几何/外观显著提升；若合并损坏遮挡或视角相关外观，需限制适用区域而非宣称统一去重。需要查多视图融合与自适应原语分配。该问题工程验证较容易，但科学价值取决于是否能建立冗余与信息增益的可解释关系。
### RP-D：可见证据与不可见生成的边界如何量化？（探索性）
现象：附录C未处理未见区域、§5.2外推billboard。假设：对已观测区域和不可观测区域分别建模/评价，可降低生成先验污染可见几何的风险。验证：GT可见性掩码控制观测覆盖，比较仅观测重建与引入生成先验的方案；在已见区域测几何及重投影一致性，在不可见区域测合理性、多样性与不确定性，并报告随机种子。不要求不可见区域唯一正确；若生成增强破坏已观测几何即失败。成本与指标定义较难，因此暂列探索性。需要检索生成式NVS、观测一致约束、可见性评价。


## INTERNALIZATION：知识内化与自检
### 自己的语言解释
pixelSplat 把“给一条目标射线算颜色”的工作改为“先从参考像素猜测哪些三维原语存在，再用这些原语回答多个目标视角”。首先借助给定相机几何把跨图匹配转换为同一尺度的候选深度线索，再让每个像素输出沿射线的多个位置假设。原语形状和视角外观仍由网络回归，位置采样概率同时决定不透明度，渲染损失由此可反馈给采样参数。训练得到跨场景共享的预测器，推理时不需要为每个新场景独立优化。学习先验并没有消除稀疏观测的不确定性；这解释了为何外观指标优秀却仍有反射、远离参考视角及未知区域的局限。
### 技术与概念地图
```text
已知姿态与两幅参考图
  -> 单图局部/全局特征
  -> 极线候选匹配 + 三角化深度编码（姿态尺度）
  -> 交叉注意力 + 图内信息传播（遮挡区域线索）
  -> 深度离散分布 + 偏移 + 形状 + SH
  -> 分类采样中心 + opacity 耦合（探索及连续代理梯度）
  -> 公共坐标变换、原语并集
  -> 可微栅格化 -> 目标视图误差
  -> 训练反馈至预测器；推理时重用原语
```
关键知识：跨场景前馈预测和单场景优化的区别；相似尺度不变性；极线候选不是对应真值；多峰分布不宜总用期望点；随机探索与梯度估计须区分；显式表示允许复用但不保证几何可辨识。
### 易错理解自检
1. 若两个SfM相机的基线同时缩放2倍且像素不变，正确三角化深度如何变化？答：同乘2，米制尺度仍未知。若深度保持不变，视差预测会不一致。
2. 为什么不能直接把注意力值当深度？答：value包含经过投影的图像特征与位置编码，注意力输出是加权特征，不是显式标量深度，且匹配可能错误。
3. 三个深度样本都采中正确桶且φ=1，是否必然全不透明？答：alpha和为1，但同一像素前向合成在足迹均1时为19/27，另有跨射线贡献，故不能据此断言实际全不透明。
4. 概率分布具有可导softmax是否意味着采样位置可导？答：否；固定抽样下经α获得梯度，并非抽样索引路径可导；期望梯度还有score项。
5. 何时650倍速度表述合适？答：表1单帧渲染比约654.5，包含编码的100帧总成本按表约433.5，不能混用。
6. 为什么图4深度图不能证明主模型几何更准？答：该图为正则微调模型的渲染深度，并非GT误差比较；也没有公制颜色刻度。
### 研究启发和未解决问题
针对稀疏观测，可优先识别哪些信息由姿态和对应提供、哪些由学习先验补全、哪些根本不可观测，然后为不同区域定义独立评价。训练中的概率可以用于资源分配及主动候选探索，但若要用于置信决策须重新校准。尚未解决的是概率与真实深度误差的关系、极线候选对位姿和外观变化的鲁棒性、固定预算下多视图冗余、未见区域生成的评价；这些对应前述RP-A–D，而非已完成成果。


## GLOSSARY：术语及概念关系
| 英文 | 中文 | 本文技术含义与相近概念区别 |
|---|---|---|
| Feed-forward inference | 前馈推断 | 固定权重从新场景图像直接预测表示，而非逐场景梯度优化。§4 |
| Generalizable novel view synthesis (NVS) | 泛化新视角合成 | 在训练场景外的测试场景预测目标视图；不等于任意域泛化。§5.1 |
| Radiance field | 辐射场 | 空间位置与视角相关辐射的表示；本文以高斯集合参数化而非坐标网络。§3 |
| 3D Gaussian Splatting (3DGS) | 三维高斯溅射 | 由均值、协方差、opacity和SH定义原语，再投影栅格化合成。§3 |
| Pixel-aligned Gaussians | 像素对齐高斯 | 原语中心从参考像素射线确定；不等同于每个像素只有一个真实表面。§4.2 |
| SfM | 运动恢复结构 | 从多图估计相机/结构，通常只到相似尺度。本文用其姿态，不重新估计。§4.1 |
| Scale ambiguity | 尺度歧义 | 场景与相机中心共同缩放投影不变；相机坐标一致不等于绝对公制恢复。MATH-01 |
| Epipolar line | 极线 | 参考射线在另一图的投影，约束候选对应，不保证线上任意点是正确匹配。Fig2 |
| Triangulation | 三角化 | 用两视图射线与基线估计空间点，弱视差时退化。MATH-01 |
| Positional encoding | 位置编码 | 将候选深度变换为可学习特征，与视觉特征拼接；不是GT深度标签。式1 |
| Query/Key/Value | 查询/键/值 | 通过q-k关系分配权重并聚合v；不直接输出深度。式2–3 |
| Cross-/Self-attention | 交叉/自注意力 | 跨图候选信息传递及图内信息传播。式3–4 |
| Disparity-space bins | 逆深度分桶 | 在逆深度空间均匀排列的候选，远处深度桶更宽。式6 |
| Categorical distribution | 分类分布 | 在离散深度索引上的概率，抽样与连续softmax不同。式7 |
| Opacity | 不透明度 | 原语渲染贡献参数，本文与被抽中概率绑定；不天然是几何置信度。MATH-04 |
| Reparameterization | 重参数化 | 作者称为重参数化的opacity耦合提供连续代理梯度，不是严格categorical路径导数。MATH-04 |
| Local minimum | 局部极小 | 邻域内难以进一步降低目标的参数状态；随机探索缓解但不保证全局最优。§3 |
| Adaptive Density Control | 自适应密度控制 | 原版3DGS通过离散增殖/剪枝调整原语；与直接网络预测梯度路径有差异。§3 |
| Spherical harmonics (SH) | 球谐函数 | 方向辐射的基函数展开；系数坐标变换使用Wigner D。式10 |
| Wigner D matrix | Wigner D矩阵 | 旋转对固定球谐阶子空间的线性表示，不是相机的普通3×3旋转直接作用于所有系数。式10 |
| Amortized rendering | 摊销渲染 | 每场景编码成本由多次渲染分摊；单帧与总耗时倍率不同。Table1 |
| LPIPS | 学习感知图像块相似度 | 基于学习特征的距离，越低越好；不直接衡量几何。§5.1 |
| Billboard artifact | 广告牌状伪影 | 原语从极外推视点看呈面片状的失败模式；与高质量参考附近渲染可共存。§5.2 |
| Observability / Identifiability | 可观测性 / 可辨识性 | 前者问观测能否感知参数变化，后者问能否唯一确定；属于拓展分析而非作者新定义。 |


## SOURCES：来源、版本与证据边界
- 主要来源：David Charatan、Sizhe Lester Li、Andrea Tagliasacchi、Vincent Sitzmann，pixelSplat: 3D Gaussian Splats from Image Pairs for Scalable Generalizable 3D Reconstruction，arXiv:2312.12337v4，2024-04-04，16页，CVPR 2024。以用户提供本地同版本PDF为最终依据，HTML辅助。
- PDF SHA-256：606933b1f086ad92b5863b1b8d21a8af565bfb02b71ac47fb8c8f850d987f5b5。
- 依赖冻结报告 SHA-256：ca86f5f98fb415c46846dbe5eb5259f82211c3117b23b9540bad9d20e1bfd885。
- 官方：[arXiv](https://arxiv.org/abs/2312.12337)、[PDF v4](https://arxiv.org/pdf/2312.12337v4)、[Project Page](https://dcharatan.github.io/pixelsplat/)、[Code](https://github.com/dcharatan/pixelsplat)。项目和代码链接仅确认身份，未将代码作为研究结果证据；arXiv实时最新版未核实，缓存v4。
- 本轮为单文献精读及潜在关联索引，未阅读其他论文全文、未进行独立跨文献排名或创新性查新。跨论文比较延期；RP候选均需后续真实文献查新、实验与反证。
- 原文事实以节/图表/公式定位，补充推导与反事实是学习分析；研究问题为未验证假设。未证实项包括概率校准、真实几何精度、任意外推、域迁移、式7/9实现细节和表1总耗时口径。未运行代码或重训。
- 发布范围：笔记采用一般公开科研分析，不包含未明确授权发布的个人研究设想。隐藏完整版不是访问权限保护；源文件与冻结报告须保持不变，后续讨论在新笔记版本增量追加。



