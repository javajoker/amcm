# 参考文献与工程资源索引

> AI 自动生成 Minecraft 体素对象项目
> 版本 v0.1 · 2026-08 · 配套阅读：[可行性分析](01-feasibility.md) · [解决方案汇总](02-solutions.md)

每条目附「**→ 用在哪**」，标明它在本项目管线中的具体落点。
标记说明：**★** = 建议直接采用；**◆** = 值得深读的理论基础；**○** = 备选/参考。

---

## 目录

1. [Minecraft 专属：生成、数据集、竞赛](#1-minecraft-专属生成数据集竞赛)
2. [Minecraft 格式规范与工程库](#2-minecraft-格式规范与工程库)
3. [原生 3D 生成（文本/单图 → 3D）](#3-原生-3d-生成文本单图--3d)
4. [多视角重建与 SLAM](#4-多视角重建与-slam)
5. [辐射场与高斯泼溅](#5-辐射场与高斯泼溅)
6. [深度估计与全景处理](#6-深度估计与全景处理)
7. [分割与开放词表语义](#7-分割与开放词表语义)
8. [材质识别与内在图像分解](#8-材质识别与内在图像分解)
9. [3D 语义与稀疏体素骨干](#9-3d-语义与稀疏体素骨干)
10. [体素化与几何处理](#10-体素化与几何处理)
11. [离散优化与 MRF](#11-离散优化与-mrf)
12. [色彩科学、量化与抖动](#12-色彩科学量化与抖动)
13. [LLM 驱动的程序化生成](#13-llm-驱动的程序化生成)
14. [数据集](#14-数据集)
15. [评测方法](#15-评测方法)
16. [软件与工具链](#16-软件与工具链)

---

## 1. Minecraft 专属：生成、数据集、竞赛

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ 3D Building Generation in Minecraft via Large Language Models** (T2BM, IEEE CoG 2024) | [arXiv:2406.08751](https://arxiv.org/abs/2406.08751) · [IEEE](https://ieeexplore.ieee.org/abstract/document/10645539/) | 语言模态最直接的先行工作。**重点看它的"层间表示 + 修复"机制——那正是 LLM 直吐体素会失败的证据**，支撑我们改走 DSL 路线（方案 §5.B1） |
| **★ Generative Design in Minecraft (GDMC), Settlement Generation Competition** (Salge et al. 2018) | [arXiv:1803.09853](https://arxiv.org/abs/1803.09853) · [官网](http://gendesignmc.wikidot.com/) | 评测体系来源。四维人类评分（Adaptability/Functionality/Narrative/Aesthetic）直接改造为我们的评价维度（方案 §7.2） |
| **★ Comparing PCG metrics with Human Evaluation in Minecraft Settlement Generation** (2021) | [arXiv:2107.02457](https://arxiv.org/abs/2107.02457) | **重要警示**：自动 PCG 指标与人类评价相关性有限。论证了人类评价不可省 |
| **World-GAN: a Generative Model for Minecraft Worlds** (Awiszus, Schubert, Rosenhahn 2021) | [arXiv:2106.10155](https://arxiv.org/abs/2106.10155) · [GitHub](https://github.com/Mawiszus/World-GAN) | 早期 MC 体素生成基线。其 block2vec 嵌入思路可借鉴用于方块共现先验（方案 §2.5） |
| **★ Order-Aware Generative Modeling Using the 3D-Craft Dataset** (Chen et al. ICCV 2019) | [CVF PDF](https://openaccess.thecvf.com/content_ICCV_2019/papers/Chen_Order-Aware_Generative_Modeling_Using_the_3D-Craft_Dataset_ICCV_2019_paper.pdf) | ~2500 栋人建房屋 + **人类建造顺序**。用于 P7 放置顺序生成（方案 §2.7）与方块共现统计 |
| **CraftAssist: A Framework for Dialogue-enabled Interactive Agents** (Facebook 2019) | [arXiv:1907.08584](https://arxiv.org/abs/1907.08584) · [GitHub](https://github.com/facebookresearch/fairo) | 语言→建造的数据与框架基础；IGLU 的数据采集工具建立其上 |
| **★ IGLU: Interactive Grounded Language Understanding in a Collaborative Environment** (NeurIPS 2021 Competition) | [arXiv:2110.06536](https://arxiv.org/abs/2110.06536) | 语言→MC 结构的对齐数据集（Architect/Builder 多轮对话）。Phase 4 训练语料 |
| IGLU 2022 | [arXiv:2205.13771](https://arxiv.org/abs/2205.13771) · [AIcrowd](https://www.aicrowd.com/challenges/neurips-2022-iglu-challenge) | 同上，含澄清问题子任务 |
| IDAT: Multi-Modal Dataset and Toolkit for Interactive Task-Solving Agents (2024) | [arXiv:2407.08898](https://arxiv.org/html/2407.08898) | IGLU 数据的整理发布版，~9000 utterances |
| Learning to Execute Instructions in a Minecraft Dialogue (Narayan-Chen et al. ACL 2019) | [ACL Anthology](https://aclanthology.org/P19-1537/) | Minecraft 协作建造任务（MCBT）数据集 |
| Open-Ended Evolution for Minecraft Building Generation (2022) | [arXiv:2209.03108](https://arxiv.org/pdf/2209.03108) | 演化方法生成 MC 建筑，可作为 DSL 参数搜索的备选优化器 |
| Evocraft: A New Challenge for Open-Endedness (2020) | [arXiv:2012.04751](https://arxiv.org/abs/2012.04751) · [GitHub](https://github.com/real-itu/Evocraft-py) | MC 内程序化放置方块的 API，便于做实时演示 |
| ○ MineDojo (NeurIPS 2022) | [arXiv:2206.08853](https://arxiv.org/abs/2206.08853) · [官网](https://minedojo.org/) | MC 大规模多模态知识库（YouTube/Wiki/Reddit），可用于材质常识与风格描述挖掘 |
| ○ Voyager: An Open-Ended Embodied Agent with LLMs (2023) | [arXiv:2305.16291](https://arxiv.org/abs/2305.16291) · [官网](https://voyager.minedojo.org/) | **其迭代式代码自修复循环是方案 §5.B1 的 Verifier 闭环的直接范本** |
| Procedural Content Generation via Machine Learning (PCGML) 综述 | [arXiv:1702.00539](https://arxiv.org/abs/1702.00539) | PCG 领域背景综述 |
| WaveFunctionCollapse (Gumin) | [GitHub](https://github.com/mxgmn/WaveFunctionCollapse) | 约束式局部纹理传播。可作为 MRF 的轻量替代或用于细节填充 |
| WFC is Constraint Solving in the Wild (Karth & Smith 2017) | [ACM DL](https://dl.acm.org/doi/10.1145/3102071.3110566) | WFC 的理论分析（本质是约束满足） |
| MarkovJunior (Gumin) | [GitHub](https://github.com/mxgmn/MarkovJunior) | 概率重写规则生成，适合装饰细节的程序化补充 |

---

## 2. Minecraft 格式规范与工程库

| 资源 | 链接 | → 用在哪 |
|---|---|---|
| **★ Sponge Schematic Specification v3** | [GitHub](https://github.com/SpongePowered/Schematic-Specification/blob/master/versions/schematic-3.md) | `.schem` 导出的权威规范（方案 §2.8） |
| **★ NBT Format** | [Minecraft Wiki](https://minecraft.wiki/w/NBT_format) | 所有 MC 格式的底层二进制格式 |
| **★ Amulet-Core** | [GitHub](https://github.com/Amulet-Team/Amulet-Core) · [PyPI](https://pypi.org/project/amulet-core/) | Python 读写 MC 存档。直接写世界 / 批量处理 |
| **★ Amulet-NBT** | [GitHub](https://github.com/Amulet-Team/Amulet-NBT) | NBT 读写，Amulet-Core 的依赖 |
| **★ mcschematic** | [GitHub](https://github.com/Sloimayyy/mcschematic) | 代码生成 `.schem`，API 简洁，适合导出层 |
| nbtlib | [GitHub](https://github.com/vberlier/nbtlib) | 通用 NBT 读写，用于原版 `.nbt` 结构文件 |
| litemapy | [GitHub](https://github.com/SmylerMC/litemapy) | Python 读写 `.litematic`（大结构首选格式） |
| Litematica (Mod) | [GitHub](https://github.com/maruohon/litematica) | 客户端投影建造，是玩家实际使用输出的主要方式 |
| WorldEdit | [官网](https://enginehub.org/worldedit) | 服务端粘贴 `.schem`，注意 `//limit` 与分块 |
| Minecraft Wiki · Block states | [Wiki](https://minecraft.wiki/w/Block_states) | 楼梯/台阶/原木等 blockstate 的完整定义（亚体素拟合必读，方案 §2.6） |
| Minecraft Wiki · Map item format（地图色板） | [Wiki](https://minecraft.wiki/w/Map_item_format) | 地图色板（基色 × 4 明度）。**注意这只适用于 map art，不等于方块表面色板** |
| Minecraft Wiki · Model / Resource pack 格式 | [Wiki](https://minecraft.wiki/w/Model) | 解析 `models/block/*.json` 与继承链，调色板提取必读（方案 §2.1） |

### 已有的同类工具（竞品 / 可借鉴实现）

| 工具 | 链接 | 说明 |
|---|---|---|
| **★ ObjToSchematic** | [GitHub](https://github.com/LucasDower/ObjToSchematic) · [Web](https://objtoschematic.com/) | **最接近的开源先行工作**：网格 → MC 方块，含体素化算法选择与调色板匹配。**建议精读其源码**，尤其是方块颜色匹配与抖动部分。当前主要维护 Web 版 |
| MapartCraft | [Web](https://rebane2001.com/mapartcraft/) · [FAQ](https://rebane2001.com/mapartcraft/faq) | 2D map art 生成器，Floyd–Steinberg / 有序抖动的成熟实现。**注意其抖动策略不能直接搬到 3D**（方案 §2.3） |
| Mineways | [官网](http://www.realtimerendering.com/erich/minecraft/public/mineways/) | MC 世界 → 3D 模型（**反向**）。用于渲染验证回路 |
| Chunky | [官网](https://chunky-dev.github.io/docs/) | MC 路径追踪渲染器。用于自动评测的高质量渲染（方案 §7.1） |
| Goxel | [官网](https://goxel.xyz/) | 开源体素编辑器，支持导出多种格式，可作为中间编辑环节 |
| MagicaVoxel / `.vox` 格式 | [官网](https://ephtracy.github.io/) | 体素交换格式，社区生态大 |
| Arnis (OSM → Minecraft) | [GitHub](https://github.com/louis-e/arnis) | GIS 模态（方案 §6.D4）的成熟先例 |
| WorldPainter | [官网](https://www.worldpainter.net/) | 地形/DEM → MC，GIS 模态参考 |

---

## 3. 原生 3D 生成（文本/单图 → 3D）

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ TRELLIS: Structured 3D Latents for Scalable and Versatile 3D Generation** (CVPR 2025 Spotlight) | [arXiv:2412.01506](https://arxiv.org/abs/2412.01506) · [项目页](https://microsoft.github.io/TRELLIS/) · [GitHub](https://github.com/microsoft/TRELLIS) | **本项目最关键的外部依赖之一**。其 SLAT 表示 = 稀疏体素 + 潜向量，**可直接在其体素结构上做 L3，跳过 mesh↔voxel 往返**（方案 §4.C2 捷径） |
| **★ TRELLIS.2: Native and Compact Structured Latents for 3D Generation** | [GitHub](https://github.com/microsoft/TRELLIS.2) | TRELLIS 后继（4B 参数）。**O-Voxel「无场」稀疏体素结构 + 完整 PBR 材质输出**——PBR 材质对材质语义分类是极有价值的额外输入。**建议作为单图模态的首选** |
| Ultra3D: Efficient and High-Fidelity 3D Generation with Part Attention | [arXiv:2507.17745](https://arxiv.org/abs/2507.17745) | 稀疏体素 token 数爆炸的效率优化（>20K token 时注意力开销大）。高分辨率场景的备选 |
| UniLat3D: Geometry-Appearance Unified Latents for Single-Stage 3D Generation | [arXiv:2509.25079](https://arxiv.org/abs/2509.25079) | 几何-外观统一潜空间，单阶段生成。与我们「几何+外观一起进 L3」的思路契合 |
| OmniPart: Part-Aware 3D Generation with Semantic Decoupling | [arXiv:2507.06165](https://arxiv.org/abs/2507.06165) | **部件级语义解耦**——可直接提供 `instance_id`，对分材质上色极有用 |
| ○ XCube: Large-Scale 3D Generative Modeling using Sparse Voxel Hierarchies (CVPR 2024 Highlight) | [arXiv:2312.03806](https://arxiv.org/abs/2312.03806) | **稀疏体素层次扩散**。是 Phase 4「体素原生生成先验」最直接的架构参考（方案 §5.B4） |
| Hunyuan3D 2.x | [GitHub](https://github.com/Tencent-Hunyuan/Hunyuan3D-2) | 开源图生 3D 主力之一，含形状 + 纹理两阶段 |
| ◆ DreamFusion: Text-to-3D using 2D Diffusion (SDS 起源) | [arXiv:2209.14988](https://arxiv.org/abs/2209.14988) | SDS 优化范式的奠基工作。理解优化式 text-to-3D 的理论基础 |
| Magic3D | [arXiv:2211.10440](https://arxiv.org/abs/2211.10440) | 粗到细两阶段 SDS |
| Point-E | [arXiv:2212.08751](https://arxiv.org/abs/2212.08751) | 点云扩散，速度快质量低。仅作历史参考 |
| Shap-E | [arXiv:2305.02463](https://arxiv.org/abs/2305.02463) | 隐式函数参数生成 |
| MVDream / Zero123++ | [MVDream](https://arxiv.org/abs/2308.16512) · [Zero123++](https://arxiv.org/abs/2310.15110) | 多视角扩散。用于 D2 模态（三视图/概念图补全） |

---

## 4. 多视角重建与 SLAM

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ VGGT: Visual Geometry Grounded Transformer** (CVPR 2025 **Best Paper**) | [arXiv:2503.11651](https://arxiv.org/abs/2503.11651) · [GitHub](https://github.com/facebookresearch/vggt) · [CVF](https://openaccess.thecvf.com/content/CVPR2025/html/Wang_VGGT_Visual_Geometry_Grounded_Transformer_CVPR_2025_paper.html) | **照片模态的首选前端**。前馈、秒级、无需相机内参，一次输出位姿+深度+点图+轨迹。解决 COLMAP 在弱纹理/少视角下失败的痛点（方案 §4.C1） |
| **★ DUSt3R: Geometric 3D Vision Made Easy** (CVPR 2024) | [arXiv:2312.14132](https://arxiv.org/abs/2312.14132) · [GitHub](https://github.com/naver/dust3r) | 无标定成对稠密重建，VGGT 的前身 |
| **★ MASt3R: Grounding Image Matching in 3D** (ECCV 2024) | [arXiv:2406.09756](https://arxiv.org/abs/2406.09756) · [GitHub](https://github.com/naver/mast3r) | DUSt3R + 稠密匹配头，配准精度更高。**用于模态 A 的异源点云-图像配准兜底** |
| **★ COLMAP: Structure-from-Motion Revisited** (Schönberger & Frahm, CVPR 2016) | [官网](https://colmap.github.io/) · [论文](https://openaccess.thecvf.com/content_cvpr_2016/html/Schonberger_Structure-From-Motion_Revisited_CVPR_2016_paper.html) | 经典 SfM+MVS。慢但极稳健，作为 VGGT 的兜底路径 |
| ORB-SLAM3 | [arXiv:2007.11898](https://arxiv.org/abs/2007.11898) · [GitHub](https://github.com/UZ-SLAMLab/ORB_SLAM3) | 视觉-惯性 SLAM，实时采集场景 |
| ○ KinectFusion (ISMAR 2011) | [论文](https://www.microsoft.com/en-us/research/publication/kinectfusion-real-time-dense-surface-mapping-and-tracking/) | TSDF 融合的奠基工作。RGB-D 模态的理论基础 |
| ○ BundleFusion | [arXiv:1604.01093](https://arxiv.org/abs/1604.01093) | 全局一致的实时 RGB-D 重建 |
| RTAB-Map | [官网](http://introlab.github.io/rtabmap/) | 工程成熟的 RGB-D SLAM，适合手持扫描采集 |

---

## 5. 辐射场与高斯泼溅

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ 3D Gaussian Splatting for Real-Time Radiance Field Rendering** (SIGGRAPH 2023) | [arXiv:2308.04079](https://arxiv.org/abs/2308.04079) · [项目页](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/) · [GitHub](https://github.com/graphdeco-inria/gaussian-splatting) | 高质量外观场，用于逐面 albedo 烘焙的视图合成（方案 §4.C1 ②） |
| **★ 2D Gaussian Splatting for Geometrically Accurate Radiance Fields** (SIGGRAPH 2024) | [arXiv:2403.17888](https://arxiv.org/abs/2403.17888) · [GitHub](https://github.com/hbb1/2d-gaussian-splatting) | **表面对齐的面元表示，几何比 3DGS 准得多**。是「GS → 可体素化表面」的关键一步（方案 §4.C1 ③） |
| **★ SuGaR: Surface-Aligned Gaussian Splatting for Efficient 3D Mesh Reconstruction** (CVPR 2024) | [arXiv:2311.12775](https://arxiv.org/abs/2311.12775) · [GitHub](https://github.com/Anttwo/SuGaR) | GS → 网格提取，2DGS 的替代方案 |
| NeRF (ECCV 2020) | [arXiv:2003.08934](https://arxiv.org/abs/2003.08934) | 辐射场奠基。理解背景用 |
| Instant-NGP (SIGGRAPH 2022) | [arXiv:2201.05989](https://arxiv.org/abs/2201.05989) · [GitHub](https://github.com/NVlabs/instant-ngp) | 哈希编码加速 |
| **Neuralangelo** (CVPR 2023) | [arXiv:2306.03092](https://arxiv.org/abs/2306.03092) · [GitHub](https://github.com/NVlabs/neuralangelo) | **神经 SDF，几何质量最高但最慢**。对几何精度要求极高的场景备选 |
| SplaTAM (CVPR 2024) | [arXiv:2312.02126](https://arxiv.org/abs/2312.02126) · [GitHub](https://github.com/spla-tam/SplaTAM) | GS-SLAM，实时采集 |
| MonoGS: Gaussian Splatting SLAM (CVPR 2024) | [arXiv:2312.06741](https://arxiv.org/abs/2312.06741) · [GitHub](https://github.com/muskie82/MonoGS) | 单目 GS-SLAM，移动端采集路线 |
| nerfstudio | [官网](https://docs.nerf.studio/) · [GitHub](https://github.com/nerfstudio-project/nerfstudio) | 统一实验框架，快速对比多种重建方法 |

---

## 6. 深度估计与全景处理

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ Depth Anything V2** (NeurIPS 2024) | [arXiv:2406.09414](https://arxiv.org/abs/2406.09414) · [GitHub](https://github.com/DepthAnything/Depth-Anything-V2) | 单目深度先验。**metric 分支用于尺度标定**（方案 §4.C3） |
| Metric3D v2 | [arXiv:2404.15506](https://arxiv.org/abs/2404.15506) · [GitHub](https://github.com/YvanYin/Metric3D) | 零样本米制深度，尺度标定备选 |
| **★ Depth Any Panoramas (DAP)** (2025) | [arXiv:2512.16913](https://arxiv.org/abs/2512.16913) | **全景深度基础模型**，543K RGB-D 全景对训练。模态 A 中从全景补深度 |
| 360MonoDepth: High-Resolution 360° Monocular Depth Estimation (CVPR 2022) | [项目页](https://manurare.github.io/360monodepth/) | 切平面分块 + 融合的经典做法，工程上稳健 |
| UniFuse: Unidirectional Fusion for 360 Panorama Depth Estimation (RA-L 2021) | [GitHub](https://github.com/alibaba/UniFuse-Unidirectional-Fusion) | 等距柱状 + 立方体贴图双分支融合 |
| Cross360: 360° Monocular Depth via Cross Projections Across Scales | [arXiv:2601.17271](https://arxiv.org/html/2601.17271) | 较新的跨投影方法 |
| MVDiffusion (NeurIPS 2023) | [arXiv:2307.01097](https://arxiv.org/abs/2307.01097) | 全景/多视角一致生成。可用于全景补全缺失区域 |
| HorizonNet / HoHoNet | [HorizonNet](https://arxiv.org/abs/1901.03861) · [HoHoNet](https://arxiv.org/abs/2011.11498) | 全景室内布局估计。若目标是室内场景重建可作结构先验 |

> **全景处理要点**：ERP 两极畸变严重，**所有 2D 模型（SAM2、材质分类、去光照）必须先投影到 cubemap 或 tangent patches 再跑**，否则精度显著退化（方案 §3.A4）。

---

## 7. 分割与开放词表语义

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ Segment Anything (SAM)** (ICCV 2023) | [arXiv:2304.02643](https://arxiv.org/abs/2304.02643) · [GitHub](https://github.com/facebookresearch/segment-anything) | 区域分割基座 |
| **★ SAM 2: Segment Anything in Images and Videos** (2024) | [arXiv:2408.00714](https://arxiv.org/abs/2408.00714) · [GitHub](https://github.com/facebookresearch/sam2) | **视频/多视角一致分割**——照片序列模态的首选，天然给出跨帧一致的 mask |
| Grounded SAM: Assembling Open-World Models | [arXiv:2401.14159](https://arxiv.org/abs/2401.14159) · [GitHub](https://github.com/IDEA-Research/Grounded-Segment-Anything) | 文本提示 → 分割。用于「把'砖墙'这个词定位到像素」 |
| **★ CLIP: Learning Transferable Visual Models** (ICML 2021) | [arXiv:2103.00020](https://arxiv.org/abs/2103.00020) | 开放词表材质分类的骨干；也是文本符合度评测指标（CLIP-score）的基础 |
| SigLIP | [arXiv:2303.15343](https://arxiv.org/abs/2303.15343) | CLIP 的改进替代，分类精度更好 |
| LSeg / OpenSeg / ODISE | [LSeg](https://arxiv.org/abs/2201.03546) · [OpenSeg](https://arxiv.org/abs/2112.12143) · [ODISE](https://arxiv.org/abs/2303.04803) | 开放词表语义分割的几条主流路线 |

---

## 8. 材质识别与内在图像分解

> **这一节是本项目 L2 层的核心，也是最容易被低估的部分。** 见可行性分析 §2.3。

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ RGB↔X: Image decomposition and synthesis using material- and lighting-aware diffusion** (SIGGRAPH 2024) | [arXiv:2405.00666](https://arxiv.org/abs/2405.00666) · [项目页](https://zheng95z.github.io/publications/rgbx24) · [ACM DL](https://dl.acm.org/doi/10.1145/3641519.3657445) | **去光照首选**。一次输出 albedo / normal / roughness / metallic / irradiance。**这一步是「业余感」与「专业感」的分界线** |
| **★ Colorful Diffuse Intrinsic Image Decomposition in the Wild** (TOG 2024) | [ACM DL](https://dl.acm.org/doi/10.1145/3687984) | 野外场景的彩色漫反射分解，RGB↔X 的强力替代/互补 |
| **★ A Dense Material Segmentation Dataset for Indoor and Outdoor Scene Parsing (DMS)** (ECCV 2022) | [arXiv:2207.10614](https://arxiv.org/abs/2207.10614) · [GitHub](https://github.com/apple/ml-dms-dataset) | **3M 多边形标注 / 44K 图像 / 46 材质类**。材质分类模型的主训练集 |
| **★ MatSeg: Zero-Shot Material State Segmentation** | [arXiv:2403.03309](https://arxiv.org/abs/2403.03309) · [GitHub](https://github.com/sagieppel/MatSeg) · [数据集](https://github.com/sagieppel/MatSeg-Dataset) · [Zenodo](https://zenodo.org/records/11331618) | 零样本材质与材质状态分割（生锈、磨损、潮湿）。**材质"状态"对选方块很有用**——生锈的铁应该配氧化铜而非铁块 |
| Materials in Context Database (MINC) (CVPR 2015) | [arXiv:1412.0623](https://arxiv.org/abs/1412.0623) · [项目页](http://opensurfaces.cs.cornell.edu/publications/minc/) | 3M 点标注 / 23 材质类。经典材质识别基准 |
| OpenSurfaces (SIGGRAPH 2013) | [项目页](http://opensurfaces.cs.cornell.edu/) | 材质外观标注的奠基数据集 |
| Materialistic: Selecting Similar Materials in Images (SIGGRAPH 2023) | [项目页](https://prafullsharma.net/materialistic/) | 交互式材质区域选择。适合做「用户手动修正材质」的 UI |
| MatSynth (CVPR 2024) | [arXiv:2401.06056](https://arxiv.org/abs/2401.06056) | 大规模 PBR 材质数据集，用于材质属性回归 |
| ◆ SIRFS: Shape, Illumination, and Reflectance from Shading (Barron & Malik, PAMI 2015) | [论文](https://escholarship.org/uc/item/7g5316sr) | 内在图像分解的经典理论基础 |
| Intrinsic Image Diffusion (CVPR 2024) | [arXiv:2312.12274](https://arxiv.org/abs/2312.12274) | 扩散式内在分解，室内场景 |

---

## 9. 3D 语义与稀疏体素骨干

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ Point Transformer V3** (CVPR 2024) | [arXiv:2312.10035](https://arxiv.org/abs/2312.10035) · [GitHub](https://github.com/Pointcept/PointTransformerV3) | 点云/稀疏体素语义分割 SOTA 骨干。Phase 4 生成先验的骨干候选 |
| Minkowski Engine (CVPR 2019) | [arXiv:1904.08755](https://arxiv.org/abs/1904.08755) · [GitHub](https://github.com/NVIDIA/MinkowskiEngine) | 稀疏卷积框架 |
| spconv | [GitHub](https://github.com/traveller59/spconv) | 高性能稀疏卷积，TRELLIS 等广泛使用 |
| TorchSparse | [GitHub](https://github.com/mit-han-lab/torchsparse) | 稀疏卷积加速库 |
| O-CNN / OctFormer | [O-CNN](https://arxiv.org/abs/1712.01537) · [OctFormer](https://arxiv.org/abs/2305.03045) | 八叉树表示的 3D 网络，层次化体素的另一路线 |
| **★ Gaussian Grouping: Segment and Edit Anything in 3D Scenes** (ECCV 2024) | [arXiv:2312.00732](https://arxiv.org/abs/2312.00732) · [GitHub](https://github.com/lkeab/gaussian-grouping) | **多视角 SAM mask → 3D 一致实例**。方案 §4.C1 ⑥ 的实现 |
| Segment then Splat: Unified 3D Open-Vocabulary Segmentation | [arXiv:2503.22204](https://arxiv.org/html/2503.22204v2) | 先分割后重建，边界更干净 |
| OpenSplat3D: Open-Vocabulary 3D Instance Segmentation using GS (CVPRW 2025) | [CVF PDF](https://openaccess.thecvf.com/content/CVPR2025W/OpenSUN3D/papers/Piekenbrinck_OpenSplat3D_Open-Vocabulary_3D_Instance_Segmentation_using_Gaussian_Splatting_CVPRW_2025_paper.pdf) | 开放词表 3D 实例分割，文本驱动 |
| LangSplat (CVPR 2024) | [arXiv:2312.16084](https://arxiv.org/abs/2312.16084) · [项目页](https://langsplat.github.io/) | 3D 语言场，可用文本查询 3D 区域 |
| LERF: Language Embedded Radiance Fields (ICCV 2023) | [arXiv:2303.09553](https://arxiv.org/abs/2303.09553) | 语言嵌入辐射场，LangSplat 的前身 |
| OpenScene (CVPR 2023) | [arXiv:2211.15654](https://arxiv.org/abs/2211.15654) | 零样本 3D 场景理解 |
| OpenMask3D (NeurIPS 2023) | [arXiv:2306.13631](https://arxiv.org/abs/2306.13631) | 开放词表 3D 实例分割 |

---

## 10. 体素化与几何处理

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ Robust Inside-Outside Segmentation using Generalized Winding Numbers** (Jacobson et al. SIGGRAPH 2013) | [项目页](https://igl.ethz.ch/projects/winding-number/) · [论文 PDF](https://igl.ethz.ch/projects/winding-number/robust-inside-outside-segmentation-using-generalized-winding-numbers-siggraph-2013-jacobson-et-al.pdf) | **实体体素化的理论基础**。对非水密、带噪、自相交的网格/点云仍能正确判内外——这正是真实重建数据的常态（方案 §3.A2、§6.D1） |
| Fast Winding Numbers for Soups and Clouds (SIGGRAPH 2018) | [项目页](https://www.dgp.toronto.edu/projects/fast-winding-numbers/) | 上者的加速版，实用必备 |
| Simplification and Repair of Polygonal Models Using Volumetric Techniques (Nooruddin & Turk, TVCG 2003) | [论文](https://ieeexplore.ieee.org/document/1187112) | 经典体素化与模型修复（binvox 的算法基础） |
| binvox / viewvox | [官网](https://www.patrickmin.com/binvox/) | 经典体素化工具，快速基线 |
| **★ OpenVDB / NanoVDB** | [官网](https://www.openvdb.org/) | 工业级稀疏体素数据结构（VFX 标准）。大规模稀疏存储与形态学操作 |
| **★ libigl** | [官网](https://libigl.github.io/) · [Python](https://libigl.github.io/libigl-python-bindings/) | 广义缠绕数、网格布尔等几何算法的实现来源 |
| **★ Open3D** | [官网](https://www.open3d.org/) | 点云去噪、法向估计、Poisson 重建、体素下采样 |
| Poisson Surface Reconstruction (Kazhdan et al. SGP 2006) | [论文](https://hhoppe.com/poissonrecon.pdf) · [Screened 版](https://www.cs.jhu.edu/~misha/MyPapers/ToG13.pdf) | 点云 → 水密网格 |
| Marching Cubes (SIGGRAPH 1987) | [ACM DL](https://dl.acm.org/doi/10.1145/37402.37422) | 等值面提取，SDF → 网格 |
| trimesh | [GitHub](https://github.com/mikedh/trimesh) | Python 网格处理瑞士军刀，含体素化模块 |
| IfcOpenShell | [官网](https://ifcopenshell.org/) | IFC/BIM 解析（模态 D3） |

---

## 11. 离散优化与 MRF

> **这一节是 L3 核心算法的理论支撑。** 见可行性分析 §1.3、方案 §2.5。

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★◆ Fast Approximate Energy Minimization via Graph Cuts** (Boykov, Veksler, Zabih, PAMI 2001) | [论文](https://www.csd.uwo.ca/~yboykov/Papers/pami01.pdf) | **α-expansion 算法的原始文献**。本项目 MRF 求解的核心，度量条件下有 2 倍近似比保证 |
| **★◆ What Energy Functions Can Be Minimized via Graph Cuts?** (Kolmogorov & Zabih, PAMI 2004) | [论文](https://www.cs.cornell.edu/rdz/Papers/KZ-PAMI04.pdf) | **判定哪些能量函数可用图割精确/近似求解**。设计成对项 `ψ` 时的必读约束条件 |
| An Experimental Comparison of Min-Cut/Max-Flow Algorithms (Boykov & Kolmogorov, PAMI 2004) | [论文](https://www.csd.uwo.ca/~yboykov/Papers/pami04.pdf) | BK 最大流算法，PyMaxflow 的实现依据 |
| ◆ A Comparative Study of Energy Minimization Methods for MRFs (Szeliski et al. PAMI 2008) | [论文](https://vision.middlebury.edu/MRF/) | 图割 / BP / TRW 的系统对比，选型参考 |
| PyMaxflow | [GitHub](https://github.com/pmneila/PyMaxflow) | Python 图割实现，直接可用 |
| gco-wrapper | [GitHub](https://github.com/Borda/pyGCO) | α-expansion 的 Python 封装 |
| ◆ MaskGIT: Masked Generative Image Transformer (CVPR 2022) | [arXiv:2202.04200](https://arxiv.org/abs/2202.04200) | **离散 token 并行生成范式**。方块 ID 天然是离散 token，这是 Phase 4 生成先验的直接建模范本 |
| Structured Denoising Diffusion Models in Discrete State-Spaces (D3PM, NeurIPS 2021) | [arXiv:2107.03006](https://arxiv.org/abs/2107.03006) | 离散扩散的理论基础 |
| VQ-VAE (NeurIPS 2017) | [arXiv:1711.00937](https://arxiv.org/abs/1711.00937) | 离散潜表示的奠基工作 |

---

## 12. 色彩科学、量化与抖动

| 资源 | 链接 | → 用在哪 |
|---|---|---|
| **★ CIEDE2000 色差公式** | [Sharma et al. 补充说明](http://www2.ece.rochester.edu/~gsharma/ciede2000/) | **感知均匀色差的标准**。最终评测指标必须用它（方案 §2.2、§7.1） |
| **★ OKLab 色彩空间** (Björn Ottosson) | [文章](https://bottosson.github.io/posts/oklab/) | **感知均匀且计算极快（比 ΔE₀₀ 快约 20×）**。开发期内循环用 OKLab，最终评测用 ΔE₀₀ |
| colour-science | [官网](https://www.colour-science.org/) · [GitHub](https://github.com/colour-science/colour) | Python 色彩科学库，ΔE₀₀ 等的可靠实现 |
| **★ The Void-and-Cluster Method for Dither Array Generation** (Ulichney, SPIE 1993) | [论文 PDF](https://cv.ulichney.com/papers/1993-void-cluster.pdf) | **蓝噪声抖动矩阵生成**。方案 §2.3 的 3D 各向同性抖动的算法来源 |
| ◆ Digital Halftoning (Ulichney 1987) | [书](https://mitpress.mit.edu/9780262210096/digital-halftoning/) | 抖动与半调的经典专著，蓝噪声理论来源 |
| Floyd–Steinberg 误差扩散 | [Wikipedia](https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering) | 2D 标准方法。**记录在此是为了说明为什么不直接用于 3D**（方向性伪影，方案 §2.3） |
| Color Quantization / Median Cut (Heckbert, SIGGRAPH 1982) | [论文](https://dl.acm.org/doi/10.1145/965145.801294) | 调色板量化的经典算法 |
| Gamut Mapping 综述 (Morovič) | [书](https://www.wiley.com/en-us/Color+Gamut+Mapping-p-9780470030325) | **色域映射理论**。应对 MC 调色板色域缺口（可行性 §2.1）的方法论来源 |
| POT: Python Optimal Transport | [官网](https://pythonot.github.io/) | 最优传输，可用于色域映射的分布对齐实现 |

---

## 13. LLM 驱动的程序化生成

| 文献 | 链接 | → 用在哪 |
|---|---|---|
| **★ Code as Policies: Language Model Programs for Embodied Control** (ICRA 2023) | [arXiv:2209.07753](https://arxiv.org/abs/2209.07753) · [项目页](https://code-as-policies.github.io/) | **「LLM 输出程序而非直接输出动作」范式的奠基工作**。方案 §5.B1 DSL 路线的理论依据 |
| **★ 3D-GPT: Procedural 3D Modeling with Large Language Models** | [arXiv:2310.12945](https://arxiv.org/abs/2310.12945) · [项目页](https://chuny1.github.io/3DGPT/3dgpt.html) | LLM 驱动程序化建模，多智能体分解任务。DSL 设计的直接参考 |
| **★ SceneCraft: An LLM Agent for Synthesizing 3D Scenes as Blender Code** (ICML 2024) | [arXiv:2403.01248](https://arxiv.org/abs/2403.01248) | **LLM → Blender Python，含渲染反馈的自修复循环**。Verifier 闭环的直接范本 |
| **★ Voyager** (见 §1) | [arXiv:2305.16291](https://arxiv.org/abs/2305.16291) | MC 场景下的 LLM 迭代式代码自修复，最贴近本项目 |
| Infinigen: Infinite Photorealistic Worlds using Procedural Generation (CVPR 2023) | [arXiv:2306.09310](https://arxiv.org/abs/2306.09310) · [GitHub](https://github.com/princeton-vl/infinigen) | 纯程序化生成的规模化实践。DSL 原语库设计的丰富参考 |
| Holodeck: Language Guided Generation of 3D Embodied AI Environments (CVPR 2024) | [arXiv:2312.09067](https://arxiv.org/abs/2312.09067) | 语言 → 3D 场景布局，含约束求解 |
| Self-Refine / Reflexion | [Self-Refine](https://arxiv.org/abs/2303.17651) · [Reflexion](https://arxiv.org/abs/2303.11366) | LLM 自我修正循环的方法论 |

---

## 14. 数据集

| 数据集 | 链接 | → 用在哪 |
|---|---|---|
| **★ Objaverse** (CVPR 2023) | [arXiv:2212.08051](https://arxiv.org/abs/2212.08051) · [官网](https://objaverse.allenai.org/) | **800K+ 带纹理 3D 资产**。MVP 模态 D1 的输入来源 + 自动回归测试集构造（方案 §7.3） |
| Objaverse-XL (NeurIPS 2023) | [arXiv:2307.05663](https://arxiv.org/abs/2307.05663) | 10M+ 资产，规模化训练用 |
| ShapeNet | [arXiv:1512.03012](https://arxiv.org/abs/1512.03012) · [官网](https://shapenet.org/) | 经典 3D 形状库，分类清晰便于做受控实验 |
| ABO: Amazon Berkeley Objects (CVPR 2022) | [arXiv:2110.06199](https://arxiv.org/abs/2110.06199) | 带真实 PBR 材质的商品 3D 模型，材质分类训练很合适 |
| ScanNet / ScanNet++ | [ScanNet](http://www.scan-net.org/) · [ScanNet++](https://kaldir.vc.in.tum.de/scannetpp/) | 真实室内 RGB-D 扫描 + 语义标注。模态 A/D5 的测试数据 |
| Matterport3D | [官网](https://niessner.github.io/Matterport/) | **全景 + 点云同源采集**的标准数据集。**模态 A 的最佳测试数据来源** |
| Replica | [GitHub](https://github.com/facebookresearch/Replica-Dataset) | 高质量室内重建，含材质标注 |
| Structured3D | [GitHub](https://github.com/bertjiazheng/Structured3D) | 合成室内全景 + 结构标注 |
| 3D-Craft（见 §1） | [CVF PDF](https://openaccess.thecvf.com/content_ICCV_2019/papers/Chen_Order-Aware_Generative_Modeling_Using_the_3D-Craft_Dataset_ICCV_2019_paper.pdf) | MC 人建房屋 + 建造顺序 |
| Planet Minecraft / GrabCraft | [PMC](https://www.planetminecraft.com/) · [GrabCraft](https://www.grabcraft.com/) | schematic 抓取来源。**⚠️ 使用前必须逐个核实许可与 ToS** |

---

## 15. 评测方法

| 资源 | 链接 | → 用在哪 |
|---|---|---|
| LPIPS: The Unreasonable Effectiveness of Deep Features (CVPR 2018) | [arXiv:1801.03924](https://arxiv.org/abs/1801.03924) | 渲染图感知相似度 |
| DISTS | [arXiv:2004.07728](https://arxiv.org/abs/2004.07728) | 对纹理更鲁棒的感知指标，比 LPIPS 更适合评价方块纹理 |
| CLIP-R-Precision (DreamFields, CVPR 2022) | [arXiv:2112.01455](https://arxiv.org/abs/2112.01455) | 文本→3D 的符合度评测标准 |
| GDMC 人类评价体系（见 §1） | [arXiv:1803.09853](https://arxiv.org/abs/1803.09853) | 四维人类评分 |
| PCG 指标 vs 人类评价（见 §1） | [arXiv:2107.02457](https://arxiv.org/abs/2107.02457) | **自动指标的局限性证据** |

---

## 16. 软件与工具链

### 核心依赖速查

```bash
# 几何
pip install open3d trimesh libigl pymeshlab

# 稀疏体素
pip install spconv-cu120 torchsparse

# 色彩与优化
pip install colour-science PyMaxflow POT

# Minecraft IO
pip install amulet-core amulet-nbt mcschematic litemapy nbtlib

# 视觉
pip install torch torchvision transformers
# SAM2 / VGGT / TRELLIS 需从各自 GitHub 源码安装
```

### 渲染验证

| 工具 | 链接 | 用途 |
|---|---|---|
| Chunky | [文档](https://chunky-dev.github.io/docs/) · [GitHub](https://github.com/chunky-dev/chunky) | 路径追踪高质量渲染，用于评测 |
| Mineways | [官网](http://www.realtimerendering.com/erich/minecraft/public/mineways/) | MC → 3D 模型，反向验证 |
| Blender + MCprep | [MCprep](https://github.com/TheDuckCow/MCprep) | Blender 中渲染 MC 场景 |

---

## 附：建议的精读顺序

如果时间有限，按此顺序读前 8 篇即可覆盖项目 80% 的关键决策：

1. **[Boykov et al. PAMI 2001](https://www.csd.uwo.ca/~yboykov/Papers/pami01.pdf)** — α-expansion，L3 核心算法
2. **[Kolmogorov & Zabih PAMI 2004](https://www.cs.cornell.edu/rdz/Papers/KZ-PAMI04.pdf)** — 设计成对项的约束条件
3. **[TRELLIS arXiv:2412.01506](https://arxiv.org/abs/2412.01506)** + [TRELLIS.2](https://github.com/microsoft/TRELLIS.2) — 稀疏体素生成，以及那条跳过 mesh 往返的捷径
4. **[ObjToSchematic 源码](https://github.com/LucasDower/ObjToSchematic)** — 最接近的先行工程实现
5. **[RGB↔X arXiv:2405.00666](https://arxiv.org/abs/2405.00666)** — 去光照，质量分水岭
6. **[广义缠绕数 SIGGRAPH 2013](https://igl.ethz.ch/projects/winding-number/)** — 稳健实体体素化
7. **[Void-and-Cluster](https://cv.ulichney.com/papers/1993-void-cluster.pdf)** — 3D 蓝噪声抖动
8. **[T2BM arXiv:2406.08751](https://arxiv.org/abs/2406.08751)** — 语言模态的先行工作与其局限

---

## 索引维护约定

- 新增条目必须填写「→ 用在哪」，否则不予合并
- 已验证可用的实现，在链接后标注 `[已验证 YYYY-MM]`
- 被证伪或放弃的方案不删除，移入本文末尾的「已排除方案」并注明原因——**避免团队重复踩坑**

### 已排除方案

*（暂无。随项目推进补充。）*
