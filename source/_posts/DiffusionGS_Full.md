---
title: "DiffusionGS：将三维高斯融入扩散去噪器的单阶段图像到三维生成与重建（完整精读与研究笔记）"
date: 2026-11-27 06:57:56
index_img: "/DiffusionGS/Figures/Figure_01.jpg"
categories: [Paper Reading]
hide: true
---

[返回论文概览](/Hexo/DiffusionGS/)

# 第一部分：论文精读报告

# DiffusionGS 论文完整精读报告

<!-- PAPER-METADATA BEGIN -->

**论文版本：** arXiv:2411.14384v6（2026-11-18）；核验时为最新版（2026-11-27）。

**官方资源：** [arXiv](https://arxiv.org/abs/2411.14384) | [PDF](https://arxiv.org/pdf/2411.14384v6) | [Project Page](https://caiyuanhao1998.github.io/project/DiffusionGS/) | [Code](https://github.com/caiyuanhao1998/Open-DiffusionGS) | [Model](https://huggingface.co/CaiYuanhao/DiffusionGS) | [Dataset](https://huggingface.co/datasets/CaiYuanhao/DiffusionGS) | [Demo 1](https://huggingface.co/spaces/CaiYuanhao/DiffusionGS-Object) | [Demo 2](https://huggingface.co/spaces/CaiYuanhao/DiffusionGS-Scene)

<!-- PAPER-METADATA END -->


> 论文：Baking Gaussian Splatting into Diffusion Denoiser for Fast and Scalable Single-stage Image-to-3D Generation and Reconstruction
> 来源：arXiv:2411.14384v6，用户上传同版资源 ZIP；PDF 11 页（p.1–8 正文、p.9–11 References）。
> PDF 是最终权威；HTML、Source 只作结构和 LaTeX 核对，原件保持字节不变。
> Figure 为源 PDF 区域保真 PDF 的独立展示副本；实际发布引用 JPEG，非原始素材。
> 证据类别：原文事实、背景知识、合理推断、独立评价和尚未证实。

<!-- PTH:BEGIN INIT -->
## 0. 自动初始化

### 论文身份与来源
【原文事实】PDF 首页题为 Baking Gaussian Splatting into Diffusion Denoiser for Fast and Scalable Single-stage Image-to-3D Generation and Reconstruction。作者为 Yuanhao Cai、He Zhang、Kai Zhang、Yixun Liang、Mengwei Ren、Fujun Luan、Qing Liu、Soo Ye Kim、Jianming Zhang、Zhifei Zhang、Yuqian Zhou、Yulun Zhang、Xiaokang Yang、Zhe Lin、Alan Yuille；所属 Johns Hopkins University、Adobe Research、HKUST、Shanghai Jiao Tong University。官方 arXiv 记录 ICCV 2025；PDF 印记 v6（2026-11-18）。

来源为收到的 arXiv 2411.14384v6 资源 ZIP；PDF SHA-256 `17a62eefba7fdfab4ab05da5d4856dacd0c4b3ad270475db0b230643d70139b5`，11 页完整可读。Source 已安全提取，主文档标题和作者匹配、无内部版本印记；HTML 身份匹配，部分引用未解析，二者仅作辅助。所有科学事实以 PDF 为准。官方开源仓库标注为重新实现和重新训练的版本，不作为论文原实验的等价依据。

### 章节结构
| PDF 页 | 内容 |
|---|---|
| 1–2 | Abstract；§1 Introduction 开始及动机 |
| 3 | §1 贡献列表；§2 Related Work（2.1 diffusion，2.2 GS）；§3 方法与 3.1 |
| 4–5 | §3.1 图像扩散、GS 预测与渲染；§3.2 混合训练及视角/RPPC |
| 6 | §3.2 双解码器、整体目标；§4 Dataset、Implementation |
| 7 | §4.1 对象与场景比较；Table 1、2 |
| 8 | §4.1 PhotoNVS、文本到场景；§4.2 消融；§5 Conclusion |
| 9–11 | References [1]–[91] |

### 图表目录
正文 9 Figure、2 Table；PDF 不含 Appendix 或独立编号的 Algorithm。未另行取得任何补充材料。Figure 均以相同的 PDF 权威区域提取，展示图为 300 DPI/Q90/4:4:4 JPEG；4 幅区域回渲染精确通过、5 幅带记录通过，实际观察未见明显缺失；不称全部像素级无损。

| 图/表 | 页 | 主题 |
|---|---:|---|
| Figure 1 | 1 | 对象、场景与文本生成串联 |
| Figure 2 | 2 | 对象三视图展示 |
| Figure 3 | 3 | 场景与渲染深度 |
| Figure 4 | 4 | 混训与单步去噪器 |
| Figure 5 | 5 | Plücker 和 RPPC |
| Figure 6 | 6 | 对象方法定性对照 |
| Table 1 | 7 | 用户研究、速度、对象与场景定量 |
| Figure 7 | 7 | 场景方法定性对照 |
| Table 2 | 7 | GSO 累加消融 |
| Figure 8 | 8 | PhotoNVS、相对深度与 RPPC |
| Figure 9 | 8 | 混训、多样性 |

本单元只建立身份与目录，不涉及机制推导。后续逐图引用同一展示资产；客户端网页显示尚未验证。公式编号 1–13 在方法部分逐组解释。

<!-- PTH:END INIT -->

<!-- PTH:BEGIN TITLE -->
## 1.1 标题（Title）

### 标题译文
将高斯泼溅融入扩散去噪器，实现快速、可扩展的单阶段图像到三维生成与重建。

Baking ... into ... 不是把二维结果预先烘焙成纹理，而是将 3D Gaussian Splatting（3DGS）作为去噪器每次预测的显式表示。Gaussian splatting 指将带颜色、透明度和各向异性形状的三维高斯投影并合成图像。Diffusion denoiser 是随噪声时间步变化的条件预测网络，本文使其直接输出高斯集合，而非只输出噪声或彼此独立的视图。

### Single-stage 的含义
单阶段指把扩散生成与显式三维构建放在同一个推理过程：网络预测三维，高斯渲染回图像，再用于下一次去噪。它不表示只执行一次网络前向，也不表示训练只有一个阶段：§3.1 写明使用 30-step DDIM，§4 则先混合训练，再按域微调。Fast 对应光栅化及避免事后拟合；Scalable 对应比固定三平面范围更灵活的点基表达和对象/场景混训。后两项是设计及实验主张，而非复杂度定理。

Image-to-3D 在这里涵盖两种使用场景：从物体照片生成完整可渲染物体，以及从室内外单图重建并补全场景。两者都需推测未观测内容，但对象生成通常容许多解；场景重建更强调与真实场景的目标视图对应。

### 标题不能确定的事项
标题没有给出确切输入相机参数、采样步数、深度参数化、监督来源或训练数据，也不能确定遮挡区真实几何唯一可恢复。准确几何与多视图可渲染性不是同一件事，需要后续公式和实验辨析。本标题单元不涉及公式推导。

<!-- PTH:END TITLE -->

<!-- PTH:BEGIN ABSTRACT -->
## 1.2 摘要（Abstract）

### 问题与困难
【原文事实】p.1 摘要讨论 feedforward image-to-3D。常见方案先用 2D 多视图扩散生成目标视图，再送入三维重建模块。即使每张图单独逼真，也可能把同一遮挡结构画成不同形状；如果没有同一个三维场景来解释它们，后处理只能折中拟合互相矛盾的观测。作者强调改变 prompt view 方向时易崩溃，且以对象为中心的数据限制模型对场景的适用性。

【背景知识】单张图像只给出某视线上的辐射/颜色，不给出唯一深度和不可见表面，扩散模型学习的条件分布为补全提供先验。跨视图渲染一致性有助于约束同一生成样本，但不解除单视图逆问题的欠定性。

### 方法与结果
摘要提出 DiffusionGS：把显式 3D Gaussian point cloud 置于扩散去噪器输出，每个时间步先预测共享 GS，再渲染多视图。借此避免先独立二维生成、后拟合三维的割裂。为扩充三维训练数据，场景和对象共同训练，以吸收不同视角分布、纹理和几何先验；后文 §3.2 通过相机筛选、RPPC、双解码器及训练目标实现。

原文摘要报告对象 PSNR/FID 改善 2.20 dB/23.25，场景 1.34 dB/19.16，速度约 6 秒（A100）、相对若干方法超过 5 倍，不使用单目深度估计器。Table 1c 具体对应 ABO 上 DMV3D 23.69→25.89 dB、32.28→9.03；Table 1e 对应 RealEstate10K 上 Flash3D 20.29→21.63 dB、35.03→15.87。对象上不是每个基线或每个数据集均提升同样幅度。速度表中 LGM 4.1 s 比本文 5.8 s 快。

### 证据边界
【独立评价】以上是摘要和正文报告的实验结果，不能由摘要独立证明三维点云几何真值精确、任意轨迹都可靠或无域偏差。没有外部深度估计器不表示没有几何先验；先验通过多视图监督、射线参数化和生成模型学习进入系统。

一句话：DiffusionGS 将三维高斯的预测和多视图图像扩散组织为同一逐步去噪闭环，通过混合域训练从一张图生成可重渲染的对象或场景。

<!-- PTH:END ABSTRACT -->

<!-- PTH:BEGIN CONTRIB -->
## 1.3 核心贡献（Contributions）

原文 §1 p.3 明确四项贡献，按作者主张而非独立新颖性判决解读。

### 贡献一：三维扩散框架（方法）
单图条件、目标相机及带噪多视图共同进入去噪器，像素对齐的 3DGS 作为每个时间步的输出。渲染产生干净图像预测并回到扩散采样状态；训练只用 2D 多视图渲染监督，无需高斯真值。对应 §3.1、Figure 4b、Eq. (1)–(8)。创新主张不是发明扩散或 GS，而是把它们组织成单阶段闭环。Table 2 中加入扩散相对于无时间控制的基线提高 2.94 dB。

### 贡献二：场景—对象混合训练（方法）
合用真实场景及对象数据，限制训练相机位置与朝向、采用射线参考点坐标和分域解码深度范围；用共享主干学习可迁移先验，再按域微调。对应 §3.2、Figure 4a、Eq. (9)–(13)、Figure 9a。其要解决的不仅是增加样本，还包括图像深度、背景、运动轨迹差别引起的非收敛风险。Table 2 混训增益是在已有扩散和点分布损失下测得。

### 贡献三：RPPC（几何参数化）
用射线相对世界原点的最近点替换传统 Plücker moment，与方向一起形成每像素六维相机条件。参考点几何意义直观，作者希望网络更容易学习相对深度。对应 Figure 5、Eq. (11)、Figure 8c/d、Table 2。此处是表示改造，不是新增真实深度观测。对单位方向两种表示信息可逆，详见数学分析。

### 贡献四：性能与速度（实验）
在 ABO、GSO、RealEstate10K 上比较单图重建或生成，并报告 A100 推理约 6 s、ABO 2.20 dB/23.25、场景 1.34 dB/19.16 的优势。对应 §4、Table 1 和 Figures 1–3、6–9。实验为系统设计的证据，不是理论贡献；没有覆盖每一组件的所有独立组合，也未证明场景生成几何唯一。

【独立评价】已有的 DDPM/DDIM、Transformer、pixel-aligned GS、可微渲染和 Plücker 坐标均是继承背景，本文的重点是系统机制组合及针对域差异的设计。是否在所有历史工作中首次采用该组合不在本篇阅读证据范围内。

<!-- PTH:END CONTRIB -->

<!-- PTH:BEGIN CONCLUSION -->
## 1.4 结论（Conclusion）

### 作者结论
【原文事实】§5 p.8 总结三点：一是单视图输入下对象生成和场景重建的统一框架；二是每时间步直接预测 3D 高斯点云，仅需 2D 渲染监督，以约束视图一致性；三是混合场景与对象训练和 RPPC 学习更普适的几何与纹理先验。作者报告相对强基线质量提升且推理约 6 秒。结论没有给出新的算法组件；证据来自 §4。

### 适用范围
所研究的是单张条件图加相对相机条件的生成或重建，不是无位姿的多张真实观测融合，也不是动态场景或水下成像。生成结果由同一组 GS 渲染，同一模型可生成不同种子的三维样本（Figure 9b），因此一致性与真实性必须分开。这里的真实场景比较为 RealEstate10K；不能据此认为任意真实场景、任意尺度和相机运动都可恢复。

### 未报告的限制与未来工作
§5 未单列作者自述局限、失败案例或未来工作。独立评价：没有直接几何真值的统计评估，图 3/8 的深度为生成表示的渲染或事后拟合结果；多视图图像指标不能独立识别遮挡区几何误差。对象与场景混训共享主干但最终分别微调，不能将其理解为同一最终参数无差别覆盖所有数据域。论文在扩散和点分布公式中也有需要区分原式与标准推导的细节，后文单独分析。本结论单元不新增数学推导。

<!-- PTH:END CONCLUSION -->

<!-- PTH:BEGIN FIG-OVERVIEW -->
## 2.1 图表功能总览

| 图表 | 页码 | 主要功能 |
|---|---:|---|
| Figure 1 | 1 | 整体能力展示：输入、生成多视图、点云，场景遮挡及文本串联 |
| Figure 2 | 2 | 15 组对象三视图，展示不同材质和视角 |
| Figure 3 | 3 | 三个场景的新视图与深度 |
| Figure 4 | 4 | 训练数据选择、单时间步 Transformer 和三维输出、渲染、损失 |
| Figure 5 | 5 | 传统 Plücker moment 与 RPPC 垂足的几何含义 |
| Figure 6 | 6 | 四类对象样例与五个基线定性比较 |
| Table 1 | 7 | (a) 人评、时间；(b) 2D扩散加GS拟合；(c–e) 对象和场景主指标 |
| Figure 7 | 7 | 两个真实场景的新视图与目标参考图对照 |
| Table 2 | 7 | GSO 上由无时间控制基线逐项加入组件 |
| Figure 8 | 8 | 2D生成+后拟合对照、相对深度图、RPPC 对照 |
| Figure 9 | 8 | 混训域收益、同输入不同种子样本 |

图 4、5 是方法机制依据；图 1–3 是能力展示，图 6–9 是定性或消融证据；表 1/2 提供定量对照。总览只用于阅读导航，不涉及公式推导或具体机制展开。

<!-- PTH:END FIG-OVERVIEW -->

<!-- PTH:BEGIN FIG-01 -->
## Figure 1：逐图精读（PDF p.1）

![Figure 1](DiffusionGS/Figures/Figure_01.jpg)

**Caption 原文**：Single-view object generation (upper) and scene reconstruction (lower) results of our method. For single-view object generation, the prompt views are shown in the left dashed box. The generated novel views and 3D Gaussian point clouds are depicted on the right. For single-view scene reconstruction, our model can handle hard cases with occlusion and rotation, as illustrated in the dashed boxes of the third row. The prompt views of object and scene text-to-3D demos are generated by stable diffusion [57] and Sora [4], respectively.

**Caption 中文**：本文方法的单视图物体生成（上）与场景重建（下）结果。物体生成的提示视图位于左侧虚线框，右侧展示生成的新视图和3D高斯点云。场景重建中，模型可以处理遮挡和旋转等困难情况，如第三行虚线框所示。物体与场景的文本到3D演示所用提示视图分别由Stable Diffusion[57]和Sora[4]生成。

**物体与场景**：上半部分左框覆盖GSO困难样例、野外图像、文本到图像和真实相机输入；右边各组交替展示 novel view 与 Gaussian 点云，表明目标不是单纯产生RGB。下半部分室内及湖边场景展示转动视角和遮挡揭示；最下方由文本提示经Sora生成场景图像，再由DiffusionGS生成3D。物体也可先由Stable Diffusion产生输入。

**证据边界**：p.1及§1、§4.1。展示跨输入域的可行性，而不是所有案例均有真实新视角；点云截图不能证明真实尺度或几何准确。文本到3D不是本文原生文本条件模型。

本图为定性结果展示，没有独立数学公式；涉及的机制在对应方法和公式单元展开。

<!-- PTH:END FIG-01 -->

<!-- PTH:BEGIN FIG-02 -->
## Figure 2：逐图精读（PDF p.2）

![Figure 2](DiffusionGS/Figures/Figure_02.jpg)

**Caption 原文**：Single-view object generation results of our method on GSO [13], wild images, and text-to-images prompted by stable diffusion or FLUX. Our DiffusionGS can robustly handle hard cases with furry appearance, shadow, flat illustration, complex geometry, and specularity.

**Caption 中文**：本文方法在GSO[13]、野外图像及Stable Diffusion或FLUX生成的文本到图像上的单视图物体生成结果。DiffusionGS能够稳健处理毛茸茸外观、阴影、平面插画、复杂几何和镜面反射等困难样例。

**15组样例**：图中五行，每行三组，各组由提示与生成的新视角构成。首行猫、怪兽、兔子；第二行蓝色玩偶、龟、动画角色；第三行金发人偶、蘑菇、玫瑰；第四行灰色人物、红色面具、玩具场景；第五行牛油果、手套、深色头发半身像。毛发、褶皱、凸起和大角度转向检验的不只是轮廓延伸，也包括不可见面纹理与形状生成。

**证据边界**：p.2及§4.1。图展示所选样例的视觉合理性，不能量化全数据分布、复杂反射的真实物理性质或背面准确率。

本图为定性结果展示，没有独立数学公式；涉及的机制在对应方法和公式单元展开。

<!-- PTH:END FIG-02 -->

<!-- PTH:BEGIN FIG-03 -->
## Figure 3：逐图精读（PDF p.3）

![Figure 3](DiffusionGS/Figures/Figure_03.jpg)

**Caption 原文**：Single-view scene reconstruction of our method on indoor and outdoor scenes. The depth maps are rendered by GS point clouds.

**Caption 中文**：本文方法在室内与室外场景上的单视图场景重建。深度图由GS点云渲染。

**深度来自GS**：三组分别是粉色室内空间、雪景住宅及树木、餐厅；每组并列输入RGB、深度和新视图。颜色编码深度展示近远结构，输出源于当前GS而非单独的深度网络；RGB与深度共享表示。但未标明深度合成定义与数值标尺，因此不能反推实际米制距离。

**证据边界**：p.3及§3.1、§4.1。没有GT深度、深度误差或标定验证，支持可渲染深度与相对结构的定性主张，不证明准确测距。

本图为定性结果展示，没有独立数学公式；涉及的机制在对应方法和公式单元展开。

<!-- PTH:END FIG-03 -->

<!-- PTH:BEGIN FIG-04 -->
## Figure 4：逐图精读（PDF p.4）

![Figure 4](DiffusionGS/Figures/Figure_04.jpg)

**Caption 原文**：Pipeline. (a) When selecting the data for our scene-object mixed training, we impose two angle constraints on the positions and orientations of viewpoint vectors to guarantee the training convergence. (b) The denoiser architecture of DiffusionGS in a single timestep.

**Caption 中文**：流程。(a) 为场景—物体混合训练选择数据时，对视点向量的位置和方向施加两种角度约束以保证训练收敛。(b) DiffusionGS在单一时间步内的去噪器结构。

**数据流**：(a) 场景级数据和物体级数据具有不同相机分布；从各自轨迹选择干净条件视角、噪声视角和监督新视角，通过位置及朝向约束限制采样。 (b) 采样时间t，将N张目标图加噪；与干净条件图、每像素6维相机条件一起patchify、MLP投影并加入位置嵌入。时间嵌入经MLP调制自适应层归一化；共享Transformer中自注意力跨token交互，MLP和残差继续更新。场景/物体解码器择一，预测像素对齐3D高斯集合，再光栅化为多个干净视图，送入t−1的采样过程。训练目标含去噪损失、新视角损失及点分布损失，监督渲染而非直接监督高斯。

**证据边界**：p.4、§3.1–3.2、Eq.1–13。图的t−1箭头是过程概略，不表示无噪声地直接回馈；具体由Eq.8给出。角度约束是作者经验设计，不能从图证明数学收敛。

本图的数学定义与推导在对应公式单元展开；本节明确图中变量及数据流关系。

<!-- PTH:END FIG-04 -->

<!-- PTH:BEGIN FIG-05 -->
## Figure 5：逐图精读（PDF p.5）

![Figure 5](DiffusionGS/Figures/Figure_05.jpg)

**Caption 原文**：Plücker ray vs. Reference-Point Plücker Coordinate.

**Caption 中文**：Plücker射线与参考点Plücker坐标的比较。

**垂足**：左图以原点O、相机位置o、方向d及力矩o×d刻画支撑直线，力矩垂直于o与d张成平面；右图以参考点p=o−(o·d)d替代力矩。对单位d，p·d=0，是原点到支撑直线的垂足，不必落在正向半射线上。m=p×d，p=d×m，因此两种坐标携带相同直线信息，但参考点具有更直观的空间位置意义。

**证据边界**：p.5、Eq.11。不是额外真实深度观测，也没有编码相机沿直线的位置；网络能否利用此归纳偏置由消融支持。

本图的数学定义与推导在对应公式单元展开；本节明确图中变量及数据流关系。

<!-- PTH:END FIG-05 -->

<!-- PTH:BEGIN FIG-06 -->
## Figure 6：逐图精读（PDF p.6）

![Figure 6](DiffusionGS/Figures/Figure_06.jpg)

**Caption 原文**：Visual comparison of single-view object generation on ABO, GSO, real-camera image, and text-to-image prompted by FLUX. Our method can generate more fine-grained details with accurate geometry from prompt views of any directions. Zoom in for a better view.

**Caption 中文**：在ABO、GSO、真实相机图像及FLUX生成的文本到图像上的单视图物体生成视觉对比。本文方法能够从任意方向的提示视角生成具有准确几何的更细致结果。放大查看效果更佳。

**四组对照**：每行依次为Prompt、Ours、DMV3D、LGM、CRM、12345++及DreamGaussian。四行灯笼笼架、乌龟底面、真人像与粉色玩具分别检验细条结构、大视角形状、真实图像和生成图输入。本文结果中笼架、四肢和边缘较清楚，其他方法出现模糊或形状变化，作者强调任意输入方向的适应性。

**证据边界**：p.6、§4.1与Table1。选图不能证明任意方向必然成功；对真实背面无参考的样例，仅能评价视觉合理性。

本图为定性结果展示，没有独立数学公式；涉及的机制在对应方法和公式单元展开。

<!-- PTH:END FIG-06 -->

<!-- PTH:BEGIN FIG-07 -->
## Figure 7：逐图精读（PDF p.7）

![Figure 7](DiffusionGS/Figures/Figure_07.jpg)

**Caption 原文**：Visual results of single-view scene reconstruction. We train the feedforward methods with the same scene data for fairness. Previous methods yield blurry images or introduce artifacts. In contrast, our method can robustly reconstruct scenes with occlusion.

**Caption 中文**：单视图场景重建视觉结果。为公平比较，作者使用相同场景数据训练前馈方法。以往方法产生模糊图像或伪影，相比之下本文方法可稳健重建有遮挡的场景。

**两组场景**：两行楼梯/走廊与住宅庭院，列为Prompt、Reference、Ours、Flash3D、VistaDream、Splatter-Image、PixelNeRF。Reference是真实目标视角，比较可见遮挡后区域是否合理补全及边缘是否清楚；PixelNeRF和Splatter-Image更模糊，作者指出Flash3D/VistaDream仍有黑块等伪影。

**证据边界**：p.7、§4.1、Table1(e)。训练数据公平说明仅针对所述前馈方法，不等同所有参数预算和推理条件完全一致。

本图为定性结果展示，没有独立数学公式；涉及的机制在对应方法和公式单元展开。

<!-- PTH:END FIG-07 -->

<!-- PTH:BEGIN FIG-08 -->
## Figure 8：逐图精读（PDF p.8）

![Figure 8](DiffusionGS/Figures/Figure_08.jpg)

**Caption 原文**：Visual comparison between the SOTA 2D method PhotoNVS [81] in (b) and our method in (c) on NVS and relative depth estimation. The depth map in (b) is predicted by the post-hoc 3DGS fitting the synthesized views. (d) shows the effect of our RPPC.

**Caption 中文**：在新视图合成与相对深度估计上，比较(b)中的先进二维方法PhotoNVS[81]与(c)中的本文方法。(b)的深度图由拟合合成视图的后处理3DGS预测。(d)展示本文RPPC的效果。

**深度颜色条**：(a)为输入；(b)左为PhotoNVS直接合成的新视图，右为后拟合高斯深度；(c)是完整模型；(d)是不使用RPPC的版本。蓝框标示视图不一致区域，房间、窗和地板结构可用于观察相对深度和几何连贯性。深度颜色条仅示意相对大小，没有数值刻度。后拟合无法自动修复输入视图互相冲突，完整模型借共享GS限制不一致。

**证据边界**：p.8、§4.1–4.2。不能把颜色差当作数值误差；没有深度GT或相同标尺的验证，不足以定量证明真实几何精度。

本图为定性结果展示，没有独立数学公式；涉及的机制在对应方法和公式单元展开。

<!-- PTH:END FIG-08 -->

<!-- PTH:BEGIN FIG-09 -->
## Figure 9：逐图精读（PDF p.8）

![Figure 9](DiffusionGS/Figures/Figure_09.jpg)

**Caption 原文**：Visual analysis. (a) studies the effect of using our mixed training strategy. (b) shows different generation samples of our method.

**Caption 中文**：视觉分析。(a)研究混合训练策略的作用。(b)展示本文方法的不同生成样本。

**混合训练与多样性**：(a)左为杯子输入和不混训/混训结果，背面纹理与伪影对比；右为房屋输入和两版本，比较建筑结构及伪影。作者采用相同训练迭代数。(b)同一玩具输入在三个随机种子下得到不同形状及纹理，表明条件分布可产生多个可能的3D结果，而非唯一确定答案。

**证据边界**：p.8、§4.2。混训比较没有独立控制所有样本和算力因素；随机种子演示不是后验置信度校准，也未证明样本覆盖真实条件分布。

本图为定性结果展示，没有独立数学公式；涉及的机制在对应方法和公式单元展开。

<!-- PTH:END FIG-09 -->

<!-- PTH:BEGIN TABLE-01 -->
## Table 1｜主要实验（PDF p.7）
**Caption 原文**：User study and main quantitative results of single-view image-to-3D task on ABO [11], GSO [13], and Realestate10K [90].

**Caption 中文**：ABO、GSO和RealEstate10K单视图图像到3D任务的用户研究与主要定量结果。

### (a) 用户研究
25人对14个对象的几何、纹理和提示图一致性进行1–6分评价；时间对应256×256。

|方法|得分↑|秒↓|
|---|---:|---:|
|DreamGaussian|1.94|120|
|LGM|3.04|4.1|
|DMV3D|3.16|31.4|
|CRM|2.69|10|
|12345++|3.81|60.0|
|DiffusionGS|4.88|5.8|

最高偏好，但LGM更快。相对于DMV3D约5.41倍、12345++约10.34倍速度。

### (b) PhotoNVS + 后拟合高斯
|方法|NVS秒|3D拟合秒|PSNR↑|SSIM↑|LPIPS↓|FID↓|
|---|---:|---:|---:|---:|---:|---:|
|PhotoNVS|61|2417|15.31|0.5215|0.4589|28.30|
|DiffusionGS|6|0|21.63|0.6787|0.2743|15.87|

总时间2478秒对6秒；PSNR提高6.32 dB，FID降低12.43。Figure 8展示后拟合深度和跨视图不一致问题。零拟合时间意指无需独立后拟合，不代表无需网络计算和多步渲染。

### (c) ABO
|方法|PSNR↑|SSIM↑|LPIPS↓|FID↓|
|---|---:|---:|---:|---:|
|LGM|16.01|0.7262|0.3255|86.32|
|GS-LRM|18.78|0.7974|0.2720|123.55|
|DMV3D|23.69|0.8634|0.1131|32.28|
|DiffusionGS|25.89|0.8880|0.0965|9.03|

### (d) GSO
|方法|PSNR↑|SSIM↑|LPIPS↓|FID↓|
|---|---:|---:|---:|---:|
|LGM|14.27|0.7183|0.3003|75.55|
|GS-LRM|17.70|0.7950|0.2411|112.96|
|DMV3D|20.82|0.8347|0.1289|33.48|
|DiffusionGS|22.07|0.8545|0.1115|11.52|

相对DMV3D，ABO/GSO分别+2.20/+1.25 dB，FID −23.25/−21.96。

### (e) RealEstate10K
|方法|PSNR↑|SSIM↑|LPIPS↓|FID↓|
|---|---:|---:|---:|---:|
|PixelNeRF|17.46|0.5713|0.5525|159.52|
|Splatter-Image|18.21|0.6115|0.4839|120.35|
|Flash3D|20.29|0.6483|0.3610|35.03|
|DiffusionGS|21.63|0.6787|0.2743|15.87|

相对Flash3D提高1.34 dB，FID降低19.16。

**证据边界**：PSNR测像素误差，SSIM测局部结构相似，LPIPS测感知特征距离，FID测图像分布距离。对不可见区域生成合理样本，未必与参考图逐像素一致；这四个指标均不是深度误差或几何真值度量。用户评分没有报告方差、置信区间；时间比较不等于全面资源成本分析。

<!-- PTH:END TABLE-01 -->

<!-- PTH:BEGIN TABLE-02 -->
## Table 2｜消融（PDF p.7）
**Caption 原文**：Ablation study. Results on the GSO [13] dataset are listed.

**Caption 中文**：消融研究。列出GSO数据集上的结果。

**累加消融**（从左至右逐项添加）：

|模型|PSNR↑|SSIM↑|LPIPS↓|FID↓|
|---|---:|---:|---:|---:|
|Baseline|17.63|0.7928|0.2452|118.31|
|+ Diffusion|20.57|0.8120|0.1417|47.86|
|+ Lpd|20.94|0.8423|0.1218|28.41|
|+ Mixed Training|21.73|0.8515|0.1196|17.79|
|+ RPPC|22.07|0.8545|0.1115|11.52|

Baseline为无时间步控制的去噪器，物体数据单视图输入，相同监督视图数量。四次增量PSNR为2.94、0.37、0.79、0.34 dB；FID下降70.45、19.45、10.62、6.27。最大单次增益来自扩散框架；Lpd、混训及RPPC在各自当前基线之上进一步改善结果。

**证据边界**：不是全因子实验，不宜将累加增益解释为各模块独立或顺序无关的因果效应。没有误差条，且公式12的数学疑点不能通过消融数据反推其真实代码。

<!-- PTH:END TABLE-02 -->

<!-- PTH:BEGIN FIG-TABLE-RELATIONS -->
## 图表关系综合
**机制链**：Figure 4(a)连接混合域视角筛选与Figure 4(b)时间条件去噪器，Figure 5解释输入RPPC的几何构造；共享GS使每步不同相机渲染从同一三维状态得到。Table 2对扩散、点分布损失、混训、RPPC顺序累加；Figure 9(a)局部对比混训，Figure 8(c,d)局部比较RPPC。

**验证链**：Figure 1–3展示物体、场景、点云和深度能力范围；Figure 6–7对应Table 1(c–e)的基线比较，Figure 8对应Table 1(b)二维生成后拟合的路线差异；Figure 9(b)补充同一提示多样性。Table 1(a)另含偏好与时间指标。两表均未给出深度真值误差，因此机制上的共同三维表示、渲染指标及定性几何结构必须区别于真实场景几何可辨识性的证明。

<!-- PTH:END FIG-TABLE-RELATIONS -->

<!-- PTH:BEGIN PROBLEM -->
## 研究问题
**观测与输出**：给定一张干净RGB条件图及其相机条件，训练利用同一实例的有位姿多视图；推理时选择目标相机条件并从N个噪声图像逐步生成，共享输出像素对齐高斯集合，可渲染任意指定相机（能力取决于训练与覆盖）。并非输入N张真实观测，更非无位姿重建。每步预测的GS是干净三维候选而非高斯参数加噪。

**欠定性**：单图仅记录沿可见射线的投影，许多深度、遮挡后结构与纹理可产生相同RGB。先验可以选择条件合理的三维解释，但不能从一张图必然恢复真实背面。二维多视图扩散虽合成多张图，却可能对应不同隐藏结构；后拟合得到共享GS时不得不折衷。本文用每步共享三维表示限制生成分布，但该一致性约束不等于唯一性。

**任务区分**：对象生成侧重中心对象、外部环视及形状纹理；场景重建侧重相机沿室内外轨迹运动、遮挡后新内容和宽深度范围。两域共享先验，采用不同深度参数化和解码器，不应理解为无需域选择的同一模型输出。

<!-- PTH:END PROBLEM -->

<!-- PTH:BEGIN MOTIVATION -->
## 研究动机
**二维先生成**：二维扩散各相机的条件概率受学习的跨视图关系约束，但不要求可由同一场景解释。后处理GS需拟合矛盾纹理，可能产生模糊、伪影和错误深度，并引入额外优化时间。Figure 8及Table1(b)给出实际对照；共享GS把这一约束移到每步生成中，而不是最后才施加。

**三维先验**：直接3D监督昂贵，作者指出以三平面NeRF为基础的三维渲染监督扩散受平面分辨率和体渲染速度限制；对象级大量合成数据纹理不一定真实，场景级约90K数据多是小幅运动。作者采用显式GS及可微光栅化，不需要真实GS标签。更快推理不自动意味着训练成本低。

**混合域**：环绕中心的对象具有背景空白、较小深度范围；连续运动场景具有密集背景、宽深度、大尺度变化。直接合并可能不收敛或产生伪影，作者提出视角位置/朝向筛选、RPPC和双解码器、对象点分布预热。Table2和Figure9支持性能提高，但没有独立收敛证明。

<!-- PTH:END MOTIVATION -->

<!-- PTH:BEGIN RELATED -->
## 相关工作及其边界（§2）
**四类扩散路线**：第一类对点云/网格等3D模型直接监督（Point-E、Shap-E等），训练3D资产成本限制数据规模；第二类用二维扩散的SDS蒸馏NeRF/GS等（DreamFusion、DreamGaussian），每资产优化慢；第三类给二维扩散加入相机位姿条件（Zero-1-to-3、MVDream及多视图生成方法），输出图像而非强制共享3D；第四类直接利用渲染监督3D扩散（RenderDiffusion、DMV3D），本文指出三平面分辨率和体渲染代价。分类是作者按主要机制的组织，不是互斥全集。

**GS相关路线**：3DGS用显式椭球与可微光栅化；Splatter-Image按像素预测高斯，GS-LRM/Flash3D等使用前馈高斯预测（Flash3D借助深度）；VistaDream通过扩散与优化重建场景。DiffusionGS继承像素对齐高斯与扩散渲染监督，但把每时间步输出改为GS，并把清晰渲染用于下一步图像采样。它不发明GS或DDPM，也不是简单将现成二维生成器接到3D拟合器。比较范围以论文§2、§4实际列出为限，并非对这些方法所有变体的统一评价。

<!-- PTH:END RELATED -->

<!-- PTH:BEGIN METHOD-01 -->
## 方法一：训练与推理数据流
**训练输入**（Fig4、§3.1）：一张条件图x_con及H×W×6相机条件v_con，N张目标清晰图X_0按同一时间t加入各自噪声得X_t，每张附对应6通道射线图V；另采M张监督新视图。前两组输入网络，M张监督图不是额外的真实推理观测。训练图和监督目标均是已知多视图数据。

**时间步**：在每步网络以(X_t,x_con,v_con,t,V)预测干净GS，光栅化出N张干净预测图及M张额外新视图。网络学习x0-prediction的目标通过渲染实现，不是直接预测高斯噪声，也不直接监督GT高斯。Eq1与Eq8的信号系数需按原式辨析（见数学单元）。

**推理输入**：单张条件RGB、相机条件和所选生成视角；初始化N张标准高斯噪声，t从T逐渐下降，渲染清晰图经Eq8用于下一时间步，论文明确使用30-step DDIM加速。最终输出GS与可渲染图。Single-stage指不依赖另一个后拟合三维优化阶段，并非single-step。

<!-- PTH:END METHOD-01 -->

<!-- PTH:BEGIN METHOD-02 -->
## 方法二：高斯去噪器
**Token化**：每个H×W×3图像与H×W×6相机条件拼接成9通道输入；patchify、线性映射，拼接位置嵌入成为Transformer token。相机射线图提供每个像素在世界系的方向与参考点，图像token携带外观。条件图没有添加目标噪声。

**Transformer**：共享L块主干，每块MSA、MLP、两次LN及残差；时间嵌入调制自适应层归一化，使主干与解码器适应不同噪声等级。作者称通过skip连接参考点信息贯穿块与解码器；未明确每处拼接和张量形状。跨视图token交互可帮助从局部条件推断共享结构。

**14通道**：Gaussian decoder线性映射并unpatchify，每组输出H×W×14参数图，共N+1组，合并(N+1)HW候选高斯。按通道提取深度权重w、旋转R、尺度S、不透明度α、RGB c；中心沿射线确定，协方差由R和S参数化。物体中心裁剪至[-1,1]^3。论文未逐通道列出14项，不自行假定四元数及冗余参数。对象与场景解码器分开，主干共享。

<!-- PTH:END METHOD-02 -->

<!-- PTH:BEGIN METHOD-03 -->
## 方法三：渲染监督
**共享三维表示**：Eq4对同一Gθ使用不同外参内参生成N张去噪图；Eq5从世界协方差经视图线性变换和投影雅可比得到相机投影协方差，取2×2子块；按投影覆盖tile分配高斯并按视空间深度排序，Eq6前向alpha混合颜色。不是不同视角各自拥有互相独立的最终场景。

**监督路径**：Eq7以像素L2和VGG-19感知损失比较清晰渲染与X0；在M张额外真实新视图上同样构造Lnv。可微渲染提供从图像误差到颜色、不透明度、位置、形状再到网络参数的梯度，避免需有GT GS参数。额外视角使仅拟合输入视图的几何较难通过训练监督。

**复杂度与限制**：高斯候选数随(N+1)HW线性增长，可能重叠且有效可见数量取决于opacity；Transformer注意力与光栅化也有开销，论文未给出完整复杂度。没有真实遮挡后监督时，推理只能使用训练先验；一致三维表示仍可包含浮点或错误形状。

<!-- PTH:END METHOD-03 -->

<!-- PTH:BEGIN METHOD-04 -->
## 方法四：混合训练的域适配
**视角筛选**：Eq9限制条件到噪声视图位置夹角及噪声到新视图位置夹角；Eq10限制条件朝向与两种目标朝向夹角。目的是选取一定重叠的图像、控制训练难度。位置角不等于一般场景欧氏基线，方向约束也不能保证遮挡或FOV重叠。

**RPPC**：以每条射线到世界原点的垂足p=o−(o·d)d代替Plücker力矩o×d，保留方向d。p与m在单位d下互相可逆；网络从参考点直接获取位置表达，属于表示归纳偏置而非增加独立观测或深度真值。Fig5解释其几何，Fig8和Table2给出效果。

**双解码器**：物体near/far=0.1/4.2，中心裁剪到[-1,1]^3；场景near/far=0/500。两MLP共享时间条件与主干；微调时只保留目标域头。对应深度范围按归一化坐标理解，不应未经尺度标定称为米。

<!-- PTH:END METHOD-04 -->

<!-- PTH:BEGIN METHOD-05 -->
## 方法五：训练目标与流程
**两阶段损失**：点分布Lpd用于对象预热；按Eq13，在iter≤iter0且object样本时仅Lpd，超过iter0才使用Lde+Lnv。Lde/Lnv都是L2+λ·VGG-19；Eq12希望使点分布集中，但按原式均值化会消掉标准化项（数学单元详述）。场景预热样本按字面总损失为0，未说明如何采样或处理。

**训练与微调**：Objaverse 730K、MVImgNet 220K、RealEstate10K约80K视频、DL3DV10K 10510视频。32 A100，40K iter，per-GPU BS16混训；64 A100分域微调80K（物体BS8）/54K（场景BS16）。PyTorch、Adam、BF16、sublinear memory及deferred GS rendering；学习率2K iter升至4e−4，再cosine衰减至0；256×256→512×512额外微调20K。

**未给出**：N/M、L、λ、四角阈值、iter0具体数值、Eq12实现及方差零保护、深度权重激活、DDIM的具体更新参数与噪声调度（论文明确使用30步）在当前PDF中均没有完整明确的说明。不能用开源重实现直接补为论文原始配置。

<!-- PTH:END METHOD-05 -->

<!-- PTH:BEGIN MATH-01 -->
## 公式 (1)、(8)：图像扩散与GS去噪
**Eq. (1) 原式**（p.4）：

$$x_t^{(i)}=\bar\alpha_t x_0^{(i)}+\sqrt{1-\bar\alpha_t}\epsilon_t^{(i)}.$$

$i$为噪声视角，$t$为时间，$\epsilon\sim\mathcal N(0,I)$。给定干净目标图，逐步增加噪声；条件图始终干净。注意p.3的DDPM分布均值为$\sqrt{\bar\alpha_t}x_0$，而正文Eq.1的信号系数没有平方根，两者不一致。

**Eq. (8) 原式**（p.5）：

$$x_{t-1}^{(i)}=\bar\alpha_{t-1}\hat x_{(0,t)}^{(i)}+\sqrt{1-\bar\alpha_{t-1}}\epsilon_{t-1}^{(i)}.$$

这里$\hat x_{(0,t)}$由本时间步共享高斯渲染，替代训练时的真实$x_0$。推理由标准正态噪声开始，使用30-step DDIM跳步，最终取$t=0$的GS。原文没有给出DDIM具体更新参数或完整调度，Eq.8不能直接视为标准确定性DDIM的完整实现。

**补充推导（标准DDPM，并非改写原式）**：设$\alpha_s=1-\beta_s$且$\bar\alpha_t=\prod_{s=1}^t\alpha_s$，递推$x_t=\sqrt{\alpha_t}x_{t-1}+\sqrt{\beta_t}z_t$，独立标准正态$z_t$。条件均值递推为$\sqrt{\bar\alpha_t}x_0$；方差递推$V_t=\alpha_t V_{t-1}+\beta_t I$，从$V_0=0$得$(1-\bar\alpha_t)I$，所以标准采样式信号系数是$\sqrt{\bar\alpha_t}$。若按Eq.1字面，条件均值为$\bar\alpha_t x_0$，在$0<\bar\alpha_t<1$与标准式不同。不能判定作者实际代码采用哪一种；开放重实现亦不是原始实现的证据。

标准无随机项DDIM的教学关系：$\hat\epsilon=(x_t-\sqrt{\bar\alpha_t}\hat x_0)/\sqrt{1-\bar\alpha_t}$，对于$t'<t$，$x_{t'}=\sqrt{\bar\alpha_{t'}}\hat x_0+\sqrt{1-\bar\alpha_{t'}}\hat\epsilon$。这说明确定性DDIM沿用由当前状态推得的噪声，区别于简单重新采独立噪声。本文的关键是$\hat x_0$来自GS渲染，而非逐图RGB直接预测。

<!-- PTH:END MATH-01 -->

<!-- PTH:BEGIN MATH-02 -->
## 公式 (2)、(3)：像素对齐高斯
**Eq. (2)**（p.4）：

$$\mathcal G_\theta(\mathcal X_t|x_{con},v_{con},t,\mathcal V)=\{G_t^{(k)}(\mu_t^{(k)},\Sigma_t^{(k)},\alpha_t^{(k)},c_t^{(k)})\}_{k=1}^{N_g}.$$

$N_g=(N+1)HW$；$\mu\in\mathbb R^3$中心，$\Sigma\in\mathbb R^{3\times3}$协方差，$\alpha\in\mathbb R$不透明度参数，$c\in\mathbb R^3$颜色。索引$k$跨条件和N张噪声图像的所有像素。通过中心$\mu_t^{(k)}=o^{(k)}+u_t^{(k)}d^{(k)}$把2D像素变为沿其相机射线的3D点。$o,d$为相机中心和射线方向；单位$d$时$u$为有符号射线距离，并非一般相机z深度。

**Eq. (3)**：

$$u_t^{(k)}=w_t^{(k)}u_{near}+(1-w_t^{(k)})u_{far}.$$

$w$越大越靠near（若near<far）；$w=0$对应far，$w=1$对应near。论文仅写$w\in\mathbb R$，没有明确激活方式，所以只有在$w\in[0,1]$时才保证插值范围；物体中心还会裁剪到[-1,1]³。场景范围0–500，对象0.1–4.2。R、S用于协方差，原文未展开分解；标准GS可写$\Sigma=RSS^TR^T$，此为教学补充，尺度非负时得到半正定矩阵。

**补充推导**：$u=u_{far}+w(u_{near}-u_{far})$，因此$\partial\mu/\partial w=(u_{near}-u_{far})d$。同样权重变化，在场景范围内位移敏感度远大于物体域；双解码器避免共享输出头直接适应极不相同尺度。按相同像素输出一个候选使输出尺寸固定，但不代表每个像素唯一真实表面，也不保证同一表面不同视图预测的高斯重合。

<!-- PTH:END MATH-02 -->

<!-- PTH:BEGIN MATH-03 -->
## 公式 (4)–(7)：共享高斯渲染与监督
**Eq. (4)**（p.4）：

$$\hat x_{(0,t)}^{(i)}=F_r(M_{ext}^{(i)},M_{int}^{(i)},\mathcal G_\theta(\mathcal X_t|x_{con},v_{con},t,\mathcal V)).$$

$F_r$可微光栅化，内外参改变而G不变；N张图同时被同一3D集合解释，减少独立生成视图几何不一致。监督仍是2D。

**Eq. (5)**：

$$\Sigma_t'^{(k,i)}=J_t^{(i)}W_t^{(i)}\Sigma_t^{(k)}(W_t^{(i)})^T(J_t^{(i)})^T.$$

$W$为视图变换线性部分，$J$为投影局部仿射近似的雅可比；原文用3×3，去除第三行列得到2×2投影协方差。补充推导：随机偏移$\delta X$零均值协方差$\Sigma$，线性近似$\delta y\approx JW\delta X$，于是$\operatorname{Cov}(\delta y)=JW\Sigma W^TJ^T$。非线性投影只在局部近似有效。

**Eq. (6)**（p.5）：

$$\hat x_{(0,t)}^{(i)}(m,n)=\sum_{j\in\mathcal N}c_t^{(j)}\sigma_t^{(j)}\prod_{l=1}^{j-1}(1-\sigma_t^{(l)}).$$

原文$\sigma=\alpha P(z|\mu,\Sigma)$，$z$为交点，$P$表述为高斯概率；标准splat的有效alpha通常由投影高斯核乘opacity得到，不应把归一化连续概率密度直接当作有界alpha。$\mathcal N$为覆盖像素的深度排序高斯，乘积是前方剩余透射率。两点例：$C=c_1\sigma_1+c_2\sigma_2(1-\sigma_1)$；若第一点不透明则第二点被遮挡。背景项在原式未显式写出；若$0\le\sigma\le1$，权重和$1-\prod_j(1-\sigma_j)$至多1。深度排序与覆盖tile影响渲染效率和可见性。

**Eq. (7)**：

$$\mathcal L_{de}=\mathcal L_2(\hat{\mathcal X}_{(0,t)},\mathcal X_0)+\lambda\mathcal L_{VGG}(\hat{\mathcal X}_{(0,t)},\mathcal X_0).$$

L2使渲染接近像素参考，VGG-19感知特征补充外观相似性。M个额外监督相机上构造同类$\mathcal L_{nv}$。补充链式关系$\partial L/\partial\theta=(\partial L/\partial\hat X)(\partial\hat X/\partial G)(\partial G/\partial\theta)$说明如何由RGB误差训练高斯去噪器；但被完全遮挡的点可能没有直接梯度，几何不唯一。λ数值未明确给出。

<!-- PTH:END MATH-03 -->

<!-- PTH:BEGIN MATH-04 -->
## 公式 (9)、(10)：视角筛选
**Eq. (9)**（p.5）：

$$\theta_{cd}^{(i)}\le\theta_1,\quad \theta_{dn}^{(i,j)}\le\theta_2.$$

$c$条件、$d$噪声、$n$额外监督视角。位置向量来自c2w平移；角为相对世界原点的向量夹角。对位置$a,b$，$\cos\theta=a\cdot b/(\|a\|\|b\|)$。若两者在同半径$r$球面，则距离$\|a-b\|=2r\sin(\theta/2)$；不同半径时一般式$\|a-b\|^2=\|a\|^2+\|b\|^2-2\|a\|\|b\|\cos\theta$。因此位置角并非一般欧氏距离，零向量时角未定义。

**Eq. (10)**：

$$\frac{z_{con}\cdot z_{noise}^{(i)}}{\|z_{con}\|\|z_{noise}^{(i)}\|}\ge\cos\varphi_1,\quad \frac{z_{con}\cdot z_{nv}^{(j)}}{\|z_{con}\|\|z_{nv}^{(j)}\|}\ge\cos\varphi_2.$$

$z$为c2w相机前向方向；朝向差限制噪声及额外监督与条件视角的角度。阈值由超参控制，本文未提供数值。

**补充示例**：同半径相机相隔60°，欧氏基线为r；半径分别r与3r且共方向，位置角为0但基线为2r。两个朝向完全相同的相机也可能看见不重叠物体。实际重叠还取决于内参、视场、场景深度和遮挡。两类筛选只是数据分布启发式，不构成充分重叠或训练收敛的数学证明。

<!-- PTH:END MATH-04 -->

<!-- PTH:BEGIN MATH-05 -->
## 公式 (11)：RPPC
**Eq. (11)**（p.5）：

$$r=(o-(o\cdot d)d,d).$$

设单位方向$d$，直线$L(s)=o+sd$。求到原点最近点：最小化$\|o+sd\|^2$，导数$2(o\cdot d+s)=0$，得$s^*=-o\cdot d$，参考点$p=o-(o\cdot d)d$，并有$p\cdot d=0$。若方向未归一化，公式应除以$\|d\|^2$；原式对应单位方向。垂足是支撑直线的最近点，可能不在相机前方半射线。

**可逆性**：标准Plücker力矩$m=o\times d$。因$(o-p)$平行$d$，$m=p\times d$；由向量三重积$d\times(o\times d)=o(d\cdot d)-d(d\cdot o)=p$，故$m$与$p$可逆。若将$o$沿$d$平移$o'=o+ad$，参考点不变，符合沿同一射线的光场参数化；任意世界原点改变则一般会改变p，不能称其对任意坐标平移不变。

**证据边界**：RPPC把力矩变为具有直观位置意义的向量，可能降低网络学习坐标变换的负担；没有新增几何观测、沿射线相机位置或真实深度。Figure 8和Table 2支持特定设置中的效果，而不是信息论上比Plücker多携带信息。

<!-- PTH:END MATH-05 -->

<!-- PTH:BEGIN MATH-06 -->
## 公式 (12)、(13)：点分布与训练门控
**Eq. (12) 原式**（p.6）：

$$\mathcal L_{pd}=\mathbb E_k\left[l_t^{(k)}-\left(\frac{l_t^{(k)}-\mathbb E_k[l_t^{(k)}]}{\sqrt{\operatorname{Var}(l_t^{(k)})}}\sigma_0+\mathbb E_k[|o^{(k)}|]\right)\right].$$

$l_t^{(k)}=|u_t^{(k)}d^{(k)}|$，$\sigma_0=0.5$目标标准差。作者意图鼓励物体点云集中。独立数学核查：设$\mu_l=\mathbb E_k[l]$、$s_l=\sqrt{\operatorname{Var}_k(l)}>0$，若内外均值取同一集合，$\mathbb E_k[(l-\mu_l)/s_l]=0$。因此原式等价于$\mu_l-\mathbb E_k[|o^{(k)}|]$；$\sigma_0$项抵消，字面上没有显式方差匹配目标。如果方差为零，原式还出现除零。原文未写绝对值或平方作用于整个差，也未写detach及数值保护，不应擅自补上。消融结果支持作者实际方法的效果，但无法反推出准确实现。

**Eq. (13) 原式**：

$$\mathcal L=(\mathcal L_{de}+\mathcal L_{nv})\mathbf1_{iter>iter_0}+\mathcal L_{pd}\mathbf1_{iter\le iter_0}\mathbf1_{object}.$$

逐情况：训练超过阈值，两域均使用图像损失；阈值内物体仅使用Lpd；阈值内场景按字面为0。$iter_0$与学习率2K预热并非必然同一阈值。没有说明场景样本是否在这一阶段被跳过。

**证据边界**：两式均照录PDF，不将推导疑点定性为代码错误。Lpd真实实现、非零方差保护和预热采样机制待作者原始代码或补充说明核验；公开重实现不可作为原始实现的直接证据。

<!-- PTH:END MATH-06 -->

<!-- PTH:BEGIN EXP-01 -->
## 实验一：数据、监督、训练开销
**数据与协议**（p.6）：Objaverse 730K样本，归一化对象到[-1,1]³，每个随机渲染32张50° FOV图；MVImgNet 220K样本，裁剪对象/去背景/归一相机与尺度。对象评估ABO、GSO。场景训练RealEstate10K（约80K视频片段，标准划分）和DL3DV10K（10510视频、96类真实场景）；评估RealEstate10K。全部评价每实例1张输入、10张测试视图。

**训练开销**：PyTorch、Adam、BF16混精、sublinear memory和deferred GS rendering。32 A100、40K iter、每GPU BS16混训；64 A100分别80K与54K iter、BS8/16微调；2K迭代将学习率升到4e−4，余弦衰减到0，256²到512²额外20K微调。作者报告约6秒A100推理不含巨大离线训练成本；30步DDIM而非单次前向。没有报告全部GPU小时、显存峰值、总参数量。

**指标**：PSNR衡量参考图像像素均方误差，教学关系PSNR=10log10(MAX²/MSE)；SSIM结构相似，LPIPS感知距离，FID图像特征分布距离。高PSNR/SSIM、低LPIPS/FID代表这些图像度量改善；无深度真值、相机误差或模型表面误差评估。

<!-- PTH:END EXP-01 -->

<!-- PTH:BEGIN EXP-02 -->
## 实验二：对象
**对象生成**：ABO上25.89/0.8880/0.0965/9.03，GSO上22.07/0.8545/0.1115/11.52（PSNR/SSIM/LPIPS/FID）。相对DMV3D，ABO +2.20 dB、−23.25 FID；GSO +1.25 dB、−21.96 FID。Figure 6的笼架、乌龟、玩偶等展示细节及输入方向适应性。25人对14对象评分4.88，优于比较方法最高3.81。

**效率**：256²对象5.8秒；DreamGaussian 120、DMV3D 31.4、CRM 10、12345++ 60.0、LGM 4.1秒。相对DMV3D/12345++约5.4/10.3倍；并非最快。避免后拟合与光栅化是设计理由，单独归因需更严格控制实验。

**证据边界**：在有参考图的数据集，图像指标衡量相应测试视图；真实拍摄和文本生成输入的无参考案例不能验证隐藏面真实几何，用户评分为视觉偏好而非客观几何精度。

<!-- PTH:END EXP-02 -->

<!-- PTH:BEGIN EXP-03 -->
## 实验三：场景及二维先生成基线
**场景重建**：RealEstate10K，PixelNeRF 17.46/0.5713/0.5525/159.52，Splatter-Image 18.21/0.6115/0.4839/120.35，Flash3D 20.29/0.6483/0.3610/35.03，本文21.63/0.6787/0.2743/15.87（四指标顺序同前）。相对Flash3D +1.34 dB、−19.16 FID。Figure7比较两场景与真实参考，VistaDream仅出现在图示对比。

**PhotoNVS对照**：Table1(b) 15.31/0.5215/0.4589/28.30，生成61秒+后拟合2417秒；DiffusionGS 21.63/0.6787/0.2743/15.87，6秒+0秒；提高6.32 dB、FID降12.43。Figure8显示二维先合成的新图可能无法保持共享窗、地板和房间结构，后拟合深度也不稳定。共享GS在每步约束输出可渲染性，但不保证深度等于真实。

**证据边界**：前馈基线使用相同场景数据训练，未报告全部超参/预算完全对齐；PhotoNVS的后拟合配置不够详细。相对深度图没有GT，不能声称获得定量几何精度。

<!-- PTH:END EXP-03 -->

<!-- PTH:BEGIN EXP-04 -->
## 实验四：各组件与生成多样性
**累加消融**：GSO从无时间控制Baseline开始，逐步增加扩散、Lpd、混训、RPPC，PSNR 17.63→20.57→20.94→21.73→22.07；FID 118.31→47.86→28.41→17.79→11.52；SSIM从0.7928增至0.8545、LPIPS从0.2452降至0.1115。各项增量+2.94/+0.37/+0.79/+0.34 dB，与表2一致。

**混合训练**：Figure9(a)以相同迭代数比较；杯背纹理和房屋结构较好，RealEstate10K +0.61 dB、−10.53 FID（§4.2）。这支持跨域先验可提升当前模型，不证明所有域的数据混合都有效。

**RPPC**：Figure8(c,d)房间结构与相对深度，场景+0.28 dB、−7.09 FID。可逆重参数化的效果属于网络归纳偏置的实验支持，不是信息量增加。

**随机种子**：Figure9(b)同一条件图改变种子生成不同形状和纹理，证明生成输出可以多样；样本频率不能直接解释为已校准的几何置信概率。累计消融也缺少独立全因子与统计显著性分析。

<!-- PTH:END EXP-04 -->

<!-- PTH:BEGIN SYNTHESIS-01 -->
## 贡献—证据矩阵
|主张|机制与证据|不能推出|
|---|---|---|
|每步显式3D使生成更一致|Eq2/4/8、Fig4/8；表1(b)及表2扩散增量|真实几何唯一性、严格跨视角一致度指标|
|混合训练学习通用先验|Eq9/10、双头、Fig9(a)、表2及场景增益|独立模块因果贡献、全部域泛化|
|RPPC改善几何感知|Eq11、Fig5/8、表2及场景增益|新增射线信息、真实深度准确率|
|速度和质量改善|表1全部子表；约6秒A100、无需后拟合|所有方法最快、低训练成本、任意分辨率可扩展|

**结论强度**：共享表示是明确的架构约束；生成图像质量、偏好和已报告速度有定量支持；更合理的几何有定性图示支持，但未测量真值深度/表面。扩展性主要由像素高斯、域适配与512分辨率训练支持，不是无限尺度或资源复杂度证明。若需要确定真实遮挡背面，应额外采集观测并检验多解是否被消除。

<!-- PTH:END SYNTHESIS-01 -->

<!-- PTH:BEGIN SYNTHESIS-02 -->
## 技术边界与覆盖审核
**技术边界**：单张条件RGB+相对相机条件不是未知位姿的多图重建；采样多视图不是新增实测观测。不同域需选择对应解码器及深度范围；复杂度随视图/分辨率变化，训练资源庞大。Eq1/8的信号系数与标准DDPM不一致，Eq12字面抵消标准化项，Eq13预热场景门控不明。论文未提供完整实现，不能由重实现替代。所有Figure来自同版本PDF区域，JPEG为展示副本；没有附录或深度GT。

**未解决问题**：隐藏结构真实性、米制尺度、预测不确定性校准、域外成像（如参与介质）、相机误差敏感性、不同目标轨迹和高分辨率的资源增长。上述为独立研究边界，而非作者已经验证的局限实验。

**覆盖审计**：本报告逐项解读Fig1–9、Table1(a–e)及Table2；Eq1–13全部原式和计算含义分别列于六个数学单元；§1–5的任务、相关工作、机制、训练与实验协议均已覆盖。主证据为PDF p.1–8，p.9–11为参考文献；HTML有未解析引用、Source内版本未独立确认，故两者不补充超出PDF的科学内容。图像质量、几何真实、生成多样性及效率分别评价。

<!-- PTH:END SYNTHESIS-02 -->



---

# 第二部分：学习与研究笔记

# DiffusionGS 学习与研究笔记

> 精读报告 SHA-256：2ffb198fb372cc2e77b9c34fd5e239d3dc80c932765c134a69c488b164ff3fc8
> 原文事实、背景知识、推断、独立评价与未证实内容分开；后续讨论按单元追加。

## 核心概念内化

### 从像素域扩散到三维候选的闭环
依据§3.1与Figure 4、Eq. (2)、(4)、(8)。一个容易误解的地方是“三维扩散”并非给高斯中心、尺度及颜色逐项加高斯噪声；扩散状态仍是N张RGB。网络执行映射 $g_\theta:(X_t,x_c,C,t)\to G_t$，渲染器 $R_C:G_t\to\hat X_0$，采样器再把$\hat X_0$变为下一状态。三者构成条件生成闭环，真正显式约束的是每次清晰图像候选处在可由该三维表示渲染的集合中。若直接预测RGB，训练中的跨视图注意力也可能使图片一致，但没有这一明确共同表示。若只在末步拟合GS，过程中已经生成的互相矛盾的遮挡内容很难修复。将共享表示放在每一步，相当于逐步约束候选空间；这并不是严格数学投影（渲染不具唯一逆，网络也非精确最小二乘投影）。

### 一致性与真实几何
单像素已知颜色c，设两种场景在该射线上分别将不透明同色表面放在距离1和2，并令其他可见射线同样形成原图，两者都解释条件图；背面可赋予任意纹理。若额外真实视角横向移动，视差可提供独立约束。以一维针孔教学模型，$u=fX/Z$，横移基线b后$u'=f(X-b)/Z$，故视差$\Delta u=fb/Z$。单图无法独立观测该视差；生成的目标视图虽然可与某个Z一致，但不提供独立测量。共享GS的约束可使每个样本自洽，却不消除条件后验多峰。Fig9(b)的不同种子形状纹理印证输出多样性；没有后验校准或真实隐藏面度量。

### 像素对齐的表示取舍
Eq.2–3固定候选数，将中心搜索限制在相机射线上（物体另有裁剪）。这用已知投影关系减轻从全空间直接回归位置的负担，但所有射线候选合并可能重复、错位或漂浮；透明度和形状参数可以掩盖几何不准。分辨率倍增时(N+1)HW候选数变为四倍，模型内存与渲染复杂度未必线性，因为投影覆盖与token注意力也变化。替代设计可用稀疏共享点、分层候选或按不确定性增密；这些是可能的后续研究，而非本文已验证设计。

### RPPC 是坐标选择
Eq.11的垂足 p 与传统力矩 m 满足 p=d×m、m=p×d（单位方向）。与其称“更多信息”，更准确是同一射线信息在网络中的可学习性改变。如果特征容量有限、网络偏好局部平滑位置坐标，显式p可能更易建立几何关系。反事实：若模型准确学习可逆映射且训练无限充分，两者信息上等价；有限网络的性能仍可不同。可比较固定变换后共享网络、同样参数量与训练预算，并测深度误差而非仅FID，验证归纳偏置。改变世界原点会改变参考点，沿直线移动相机中心则不改变p，不能混淆这两种平移。

### 点分布损失的理论检查
Eq.12对同一索引集合的标准化差求有符号均值，因此标准差项抵消；真实实现未报告。如研究目标确为使径向分布标准差接近0.5，候选形式可用 $(\mathrm{std}(l)-0.5)^2$，或比较实际点云相对物体中心的半径分布；前者仅控制沿射线距离分布，仍未必等价空间点云紧致。方差为零时需正则化分母。此处只是设计反事实，不能替换原式或断言作者采用了它。


## CROSS-PAPER：潜在跨文献关联索引

### Topic: Inference & Optimization（生成状态与显式表示）
- **Keywords:** Multi-view Diffusion; x0-prediction; 3D Gaussian Splatting; Differentiable Rasterization
- **Highlight:** 扩散状态为多视图图像，每步预测干净像素对齐GS并渲染回图像状态，而非先生成再拟合；推理30-step DDIM。
- **Evidence:** §3.1、Eq. (1)–(8)、Figure 4(b)、Table 1(b)。
- **Connection Points:** 扩散空间；显式或隐式三维中间表示；跨视图一致性约束施加位置；后拟合需求。
- **Open Questions:** 逐步施加共同三维约束相对仅终步重建的收益如何随噪声等级和目标视角变化？

### Topic: Geometry Inference（像素对齐与射线参数化）
- **Keywords:** Pixel-aligned Gaussians; Ray Conditioning; Plücker Coordinates; Reference-Point Plücker Coordinate
- **Highlight:** 高斯中心沿每个像素射线预测距离，候选数固定；RPPC以垂足代替力矩，两者在单位方向条件下可逆。
- **Evidence:** §3.1、Eq. (2)–(3)、§3.2 Eq. (11)、Figure 5、Figure 8。
- **Connection Points:** 深度参数化；坐标归纳偏置；相机输入条件；尺度范围和重复候选。
- **Open Questions:** 不同相机尺度与原点规范下的坐标表达，是否同样改善几何误差及收敛？

### Topic: Generalization（对象与场景域混合）
- **Keywords:** Scene-object Mixed Training; Viewpoint Selection; Dual Gaussian Decoder; Domain Adaptation
- **Highlight:** 相机位置及朝向角筛选、分域深度范围和两解码头支持共享主干；微调时只保留对应头，混训提高当前实验图像指标。
- **Evidence:** §3.2、Eq. (9)–(13)、Figure 4(a)、Figure 9(a)、Table 2、§4.2。
- **Connection Points:** 数据分布差异；覆盖约束；共享先验与域特有参数；负迁移。
- **Open Questions:** 在固定样本与计算预算下，哪些域特有统计量会诱发混合训练负迁移？

### Topic: Observability & Identifiability（生成多样性与真实性）
- **Keywords:** Single-view Reconstruction; Occlusion; Conditional Generative Prior; Uncertainty; View Consistency
- **Highlight:** 同一输入可由随机种子产生不同形状与纹理；显式三维一致性和图像指标不等同真实隐藏面或标定深度。
- **Evidence:** Figure 3、Figure 8、Figure 9(b)、Table 1、§4.2。
- **Connection Points:** 观测数量；可见性；几何真值与图像指标；后验样本及置信度校准。
- **Open Questions:** 生成多样性是否能反映未观测区域真实误差和新增真实视角的信息价值？


## 研究方向联系与适用条件
本节采用公开、一般性的稀疏观测视觉问题作为研究分析框架，不将任何私人未发表研究设想写入发布笔记。

- Field：三维视觉与生成式视觉推断。
- Task：单视图图像到三维（对象生成、场景重建），可渲染新视角。
- Task Setting：一张RGB及相对相机条件，训练使用同实例已知相机多视图，推理使用生成而非真实新增视角。
- Research Problem：在欠定观测下如何协调学习先验、几何一致性和实际证据，避免把可渲染但不真实的场景当成已测量的结果。
- Method：图像扩散状态＋每步显式共享GS、可微渲染、分域适配及相机坐标参数化。

### 可迁移的技术机制
逐步三维约束适合任何输出需要多个相机共享空间表示的条件生成问题。额外真实视角若可得，应作为观测证据，与生成视角分开建模；仅增加生成视角数量并没有增加独立测量。混训策略展示如何把共享主干与深度域差异解耦，但不是所有域共享先验的证明。

### 参与介质成像下的边界（独立拓展）
水下或雾中可考虑同样的稀疏观测问题，但RGB成像还依赖光传播。教学简化式 $I_c=J_c e^{-\beta_c z}+B_c(1-e^{-\beta_c z})$ 描述均匀介质、单一有效系数和背景光条件下的直达与后向散射。未知$J_c,\beta_c,B_c,z$时，固定$I_c$通常对应多组解：即使相机准确、生成视图一致，几何变化、介质变化、表面颜色变化仍可互相补偿。本文没有介质传输模型，也没有水下训练或评估，因此不能把干净RGB上的增益直接外推到这些场景。

一种技术迁移是把共享GS渲染器扩展为含物理成像过程的可微前向模型，再用实测图像残差约束生成候选；这需要介质参数化、尺度标定和不同条件下的真实多视图/合成真值验证。仅将受介质影响图像输入本文现成模型可能得到视觉合理而物理不一致的3D。研究重点应检验先验对欠定性的作用与证据冲突，而不是预设扩散必优或只追求新视图好看。


## 可证伪的研究问题候选
以下是依据本篇证据形成的待验证问题，尚未进行独立文献新颖性审查，不宣称原创或已验证。

### RP-A：逐步共享几何是否减少真实结构错误，还是只减少渲染不一致？（优先）
**现象与未知。** Table1(b)及Figure8显示每步共享GS优于PhotoNVS后拟合，但不同生成器、训练与拟合过程同时变化，且没有深度真值指标。未知是结构约束的施加时机对可见区和隐藏区真值的不同影响。

**可证伪假设。** 在同一训练数据、预算和模型主干下，每步GS渲染约束相比末步GS拟合降低已观测区几何误差与跨目标视角重投影误差；对完全未观测面，收益受先验适配程度限制。

**实验设计。** 使用具有相机/几何/无遮挡参考的合成或扫描数据，控制目标轨迹、观测数量与基线、同一噪声种子和总推理预算；比较二维去噪+事后GS、仅末步GS、每步GS。以真实可见性分割可见/部分遮挡/全遮挡区域，测深度绝对/相对误差、表面距离、真实参考重投影及图像指标。用额外真实视图作独立验收，不把生成视图当GT。重复多个种子及场景，报告分布而非单例。若只改善FID而几何误差无改善，则反驳几何收益主张；若控制预算后优势消失，则要重新分析成本因素。优先因为与本文核心架构直接对应，现成数据可验证；代价是需要一致训练框架和算力，结果不覆盖全部真实场景。

### RP-B：生成多样性是否能用于稀疏观测下的误差识别？
**现象与未知。** Fig9(b)产生不同隐藏形状，但未测量多样性与误差的对应关系。

**可证伪假设。** 在固定先验及观测条件下，跨种子几何分歧与未观测区真实误差存在可校准关系；新增真实视角后，分歧和误差应共同下降。反例是所有种子共享同一训练偏差，低分歧而高误差。

**验证。** 固定相机尺度、场景、采样步数，每实例多次生成，将各样本渲染到同一真实评价相机得到深度分布；分可见/遮挡区域测方差与误差相关、误差检测AUROC及预测区间覆盖率；对照单样本置信度、与无观测掩码启发式。逐次加入真实相机需使用支持多观测的改造模型或额外一致性优化，不能声称原版支持。若分歧不比可见性掩码预测误差更好或区间严重失校准，假设不成立。评价成本随种子数增长，不能仅凭方差宣称不确定性量化。优先级次于A，需可靠跨样本对齐与校准数据。

### RP-C：域迁移时如何分离几何与介质观测冲突？（探索）
**现象与未知。** 本文跨对象/场景训练有收益，但水下或雾中的投影颜色会同时受几何距离与传输系数影响。未知显式物理渲染约束是否能减轻先验把介质伪影当结构的错误。

**可证伪假设。** 在同一真实观测数量及相机精度下，加入正确受限传输模型并明确观测/生成视角的权重，能在介质变化时比纯RGB扩散重建降低几何偏差；如果介质参数不可辨识或模型错设，效果可能反转。

**验证。** 生成同几何、同相机、不同介质参数的成对样本；对比清晰域训练模型、仅域微调、显式物理前向模型及oracle介质参数。控制观测数、尺度、光照，分别报告真实深度、传输/清晰外观误差、novel-view PSNR及跨域泛化；在相同图像质量下看几何是否仍有差异。若仅PSNR提高或误差被介质参数吸收，则不足以支持假设；未知介质参数时须作灵敏度和不确定性分析。可先做小规模合成检验，真实验证需标定和水下数据。优先级较低因为本文无直接介质证据，需要独立调研该领域建模与已有工作。


## 知识内化与自测

如果要向未读论文的同学解释：一张图没有告诉我们背面是什么。一般二维扩散可以猜出多个视角，但可能猜成几套互相矛盾的场景。DiffusionGS每次都先猜一套三维高斯，再从多个相机拍这套高斯，所得图片进入下一次去噪。相当于让每次“猜测”在三维中统一，而非最后修补互相矛盾的图像。它仍然在猜，没有真实背面观测就不能保证正确。相机射线限制高斯中心位置，参考点坐标改善表示；不同对象/场景的深度范围由独立头处理，多视图渲染监督则把图像误差传回网络。训练成本大，但推理省去后拟合。

### 概念地图
观测与相机 → 条件生成先验 → 多视图带噪状态 → Transformer跨视图交互 → 像素对齐GS → 同一场景渲染 → 图像去噪反馈；训练时真实多视图 → Lde/Lnv → 可微渲染梯度；数据域差异 → 角度筛选/RPPC/分域头/对象预热 → 共享主干及分域微调。实证链为Fig4/5机制、Table1性能与Table2增量、Fig8/9定性边界。

### 易错理解自检
**是否直接对GS加噪？** 否。Eq1/8是RGB状态，Eq2是每步GS输出。

**是否需要多张真实输入？** 否，推理单图及相对相机条件；N张噪声图不是真实观测。

**是否一轮前向？** 否，30步DDIM，单阶段指无需另行3D优化。

**RPPC提供新深度吗？** 否，单位方向下与Plücker力矩可逆，改变表示。

**图3/8是否验证真实深度？** 否，无深度GT定量；共享GS渲染深度只说明可提取表示的深度。

**Eq12能否直接证明点云方差收缩？** 否，按字面标准化项抵消；实际实现不明。

**所有方法均比本文慢吗？** 否，Table1(a) LGM 4.1秒，本文5.8秒。

### 仍需理解的问题
如何让生成先验服从真实稀疏观测而非反向替代证据？不同种子背面一致是否只是训练偏差？生成目标轨迹如何影响可见/不可见区域的误差？更强的物理或几何先验在域移位时可能是帮助，也可能制造系统偏差，需用真实观测和分区真值实验检验。这些不是原文已经回答的问题。


## 术语表
|英文|中文与技术含义|相近概念区别|
|---|---|---|
|Image-to-3D|图像到三维：从图像生成可渲染三维表示|不等于只输出NVS图片|
|Novel View Synthesis (NVS)|新视图合成：给定相机生成未观测视角图像|图像质量不等于几何精度|
|3D Gaussian Splatting (3DGS)|三维高斯泼溅：显式有颜色、不透明度、位置及协方差的高斯集合|本文每步输出，不是常规每场景独立拟合|
|Pixel-aligned Gaussian|像素对齐高斯：按像素射线回归中心与属性，Eq2/3|固定候选数量不等于唯一表面|
|DDPM|去噪扩散概率模型，图像逐步加噪与逆向预测|标准信号系数为累计alpha的平方根；本文Eq1/8须区分|
|DDIM|去噪扩散隐式模型，允许跳时间步采样|本文30步，并非一次前向|
|x0-prediction|预测干净数据：本文经共享GS渲染得到|区别直接预测噪声epsilon|
|Condition view|条件视图，唯一真实推理图|区别噪声视图、额外真实监督视图|
|RPPC|参考点Plücker坐标，以垂足和方向表射线|与力矩表示可逆，非深度标签|
|Camera-to-world (c2w)|相机到世界变换，用其平移和前向取角约束|位置角不一定等于欧氏基线|
|AdaLN|自适应层归一化，以时间条件调制|不是独立三维监督|
|Rasterization|光栅化，高斯投影与深度排序alpha混合|区别NeRF体渲染积分实现|
|Denoising loss|去噪损失，Eq7 L2+VGG|额外相机另构造Lnv|
|Point distribution loss|点分布预热损失，Eq12|作者意图集中点云，字面式不能确认方差约束|
|Dual Gaussian decoder|对象/场景双高斯解码器|共享主干；微调仅保留相应头|
|SDS|分数蒸馏采样损失，从扩散先验优化3D资产|本文不需独立每资产优化|
|Observability|可观测性，实际观测能约束哪些信息|生成图不是额外测量|
|Identifiability|可辨识性，观测是否确定唯一或有限参数解|一致性不保证唯一真值|
|PSNR/SSIM|像素/结构图像度量|非直接深度误差|
|LPIPS/FID|感知距离/特征分布距离|不能替代几何真值或置信度校准|


## 来源、证据与研究边界

- 原论文：Baking Gaussian Splatting into Diffusion Denoiser for Fast and Scalable Single-stage Image-to-3D Generation and Reconstruction，arXiv:2411.14384v6，2026-11-18；PDF共11页，正文p.1–8、参考文献p.9–11。PDF SHA-256：`17a62eefba7fdfab4ab05da5d4856dacd0c4b3ad270475db0b230643d70139b5`。
- 绑定精读报告 SHA-256：`2ffb198fb372cc2e77b9c34fd5e239d3dc80c932765c134a69c488b164ff3fc8`。上文数学推导和图表证据可定位报告的MATH、Figure及实验对应章节；原始PDF优先。
- 官方身份/资源核验：https://arxiv.org/abs/2411.14384 及 https://caiyuanhao1998.github.io/project/DiffusionGS/ 。https://github.com/caiyuanhao1998/Open-DiffusionGS 声明为重新实现及训练；不作为原实验代码依据。
- 用户上传ZIP含PDF、HTML及Source，HTML部分资源引用未解析，Source无独立内部版本印记；未用二者填补PDF未报告的技术事实。图像使用已验收展示副本。

**证据分类**：论文报告的具体指标及结构为原文事实；针孔视差、DDPM标准推导、协方差、RPPC可逆性等为补充教学推导；介质方程及研究实验为独立拓展；真实深度、域迁移、置信度校准和原始代码实现均尚未证实。Eq1/8、Eq12/13疑点保持原文与推导分离。

**跨论文分析延期**：索引仅抽取本篇有证据的机制和问题，没有独立读取其他论文建立已验证比较。研究候选尚需文献调研、可复现实验及独立验证。公开笔记不含未授权的私人研究资料。



