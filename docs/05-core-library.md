# 公共库设计 · mcvox

> 所有输入路线、所有交付形态共用的核心库
> 版本 v0.1 · 2026-08 · 前置：[技术设计总纲](04-tech-design.md)

> **包名 `mcvox` 为工作名**，Phase 0 需核实 PyPI 可用性（[PRD Q4](00-PRD.md#9-风险与未决问题)）。备选：`voxcraft`、`blockify`、`mcforge`。

---

## 1. 设计目标

这个库要同时服务四类调用方，它们的诉求并不一致：

| 调用方 | 诉求 | 对设计的要求 |
|---|---|---|
| 8 条输入路线 | 复用几何/色彩/导出能力，不重复造轮子 | 模块细粒度可单独引用 |
| 4 种交付形态 | 同一套算法，不同的编排与呈现 | 无 I/O 副作用，配置显式传入 |
| 第三方开发者 | 在其上构建自己的路线 | API 稳定、文档完整、安装轻量 |
| 回归测试 / 研究实验 | 单独跑某一个 stage，可复现 | stage 可独立调用，随机性可控 |

**统一到三条设计原则：**

1. **契约优先** —— `VoxelIR` 与 `Stage` 协议先于任何实现确定，且版本化
2. **重依赖可选** —— 核心包安装体积控制在几十 MB，重模型走适配器
3. **纯函数化** —— stage 是 `(IR, Config) → IR`，无隐式全局状态，随机性由显式种子驱动

---

## 2. 模块划分

```
mcvox/
├── core/                    # 零重依赖的基础类型与协议 —— 其他模块只依赖它
│   ├── types.py             #   VoxelIR, BlockRef, MaterialClass, Provenance
│   ├── grid.py              #   SparseVoxelGrid 稀疏体素容器
│   ├── frame.py             #   坐标系与尺度约定 (voxels_per_meter, Y-up)
│   ├── protocols.py         #   Stage / *Adapter 协议定义
│   ├── config.py            #   分层配置（默认 → 文件 → 环境 → 参数）
│   ├── cache.py             #   内容哈希缓存 (ADR-09)
│   └── report.py            #   run_report.json 结构化运行报告
│
├── palette/                 # ★ 方块调色板 —— L3 的基石，所有路线共用
│   ├── extract.py           #   从 .jar / 资源包提取纹理（仅本机，不落库）
│   ├── resolver.py          #   blockstates / models JSON 继承链解析
│   ├── stats.py             #   Lab 均值·方差·频率能量（线性空间平均）
│   ├── attrs.py             #   方块属性：重力/支撑/亚体素形状/tier/群系着色
│   ├── registry.py          #   PaletteRegistry：按 MC 版本 + 资源包缓存
│   └── gamut.py             #   ★ E1 色域分析：最近邻距离场、覆盖率、热力图
│
├── color/                   # 色彩科学 —— 纯函数，零重依赖
│   ├── spaces.py            #   sRGB ↔ linear ↔ XYZ ↔ Lab ↔ OKLab
│   ├── diff.py              #   ΔE₀₀ / OKLab 欧氏 / 逐面加权色差
│   ├── gamut_map.py         #   色域映射（保序压缩进 MC 可达域）
│   └── dither.py            #   ★ 3D 蓝噪声 void-and-cluster 阈值场
│
├── geom/                    # 几何处理
│   ├── voxelize.py          #   ★ 实体体素化（广义缠绕数）/ 表面体素化
│   ├── normals.py           #   法向估计、面可见性判定
│   ├── residual.py          #   ★ 亚体素残差计算（ADR-03）
│   ├── morph.py             #   3D 形态学：闭运算填缝、开运算去噪
│   └── topology.py          #   连通分量、悬空检测、支撑图
│
├── appearance/              # L2 外观
│   ├── bake.py              #   ★ 逐面投影烘焙（加权中位数抗高光）
│   ├── panorama.py          #   ERP ↔ cubemap ↔ tangent patches 投影
│   ├── delight.py           #   去光照（经 DelightAdapter）
│   └── inpaint.py           #   未观测面的颜色补全（邻域继承 → 材质先验）
│
├── semantic/                # L2 语义
│   ├── taxonomy.py          #   材质分类体系（M≈30 类）+ 别名归一化
│   ├── segment.py           #   分割（经 SegmentationAdapter）
│   ├── classify.py          #   开放词表材质分类
│   └── fuse3d.py            #   多视角语义投票融合到体素
│
├── blockify/                # ★★ L3 核心 —— 全项目心脏
│   ├── unary.py             #   一元代价矩阵（GPU 批量）
│   ├── pairwise.py          #   共现先验 + 成对项构建
│   ├── solver.py            #   求解后端：alpha_expansion / icm / greedy
│   ├── subvoxel.py          #   ★ 亚体素几何拟合 + 形状模板库
│   ├── repair.py            #   物理合法性修复 pass 链
│   ├── budget.py            #   材料预算约束与降级替换
│   ├── mapping.py           #   材质类 → 候选方块映射表加载
│   └── pipeline.py          #   L3 标准编排（对外主入口）
│
├── io/                      # MC 格式读写
│   ├── litematic.py         #   .litematic（默认导出）
│   ├── schem.py             #   Sponge Schematic v3
│   ├── structure_nbt.py     #   原版结构方块（48³ 上限校验）
│   ├── mcstructure.py       #   基岩版（小端 NBT）
│   ├── vox.py               #   MagicaVoxel 交换
│   ├── ir_io.py             #   ★ VoxelIR 序列化（.mcir）
│   └── manifest.py          #   材料清单 + provenance
│
├── eval/                    # 评测
│   ├── metrics.py           #   IoU / Chamfer / ΔE₀₀ 分布 / 法向一致性
│   ├── render.py            #   渲染适配（Chunky / 内置轻量渲染器）
│   ├── perceptual.py        #   LPIPS / DISTS
│   └── suite.py             #   回归测试套件运行器
│
├── viz/                     # 可视化
│   ├── preview.py           #   快速体素预览（纯色块，无纹理依赖）
│   └── report.py            #   HTML 质量报告生成
│
└── cli/
    └── main.py              #   mcvox 命令行入口
```

### 依赖方向（严格单向，CI 强制）

```
cli  →  blockify  →  {palette, color, geom, appearance, semantic, io}  →  core
                                        ↓
                              adapters（可选，经协议）
```

**规则：`core` 不依赖任何其他 mcvox 模块；`color` 与 `geom` 不依赖 `palette`；任何模块不得反向依赖 `cli`。** 违反即 CI 失败。

---

## 3. VoxelIR — 核心契约

```python
# mcvox/core/types.py
from dataclasses import dataclass, field
from typing import Literal
import numpy as np

IR_VERSION = "1.0"

@dataclass
class VoxelIR:
    """所有输入路线的唯一产出，核心层的唯一输入。

    坐标约定：右手系，Y 轴向上（与 MC 一致），整数体素坐标，
    原点在包围盒最小角。颜色统一在 OKLab 空间。
    """

    # ---- 必填 ----
    ir_version: str = IR_VERSION
    coords: np.ndarray = None          # (Nv, 3) int32   占用体素坐标
    voxels_per_meter: float = None     # 尺度锚点，必填（对应可行性 §2.2）
    provenance: "Provenance" = None    # 溯源，必填（对应 G5 可复现性）

    # ---- 几何 ----
    normal: np.ndarray | None = None       # (Nv, 3) f32    表面外法向
    residual: np.ndarray | None = None     # (Nv,)  f32     ∈[-0.5,0.5] 亚体素残差
    face_vis: np.ndarray | None = None     # (Nv, 6) bool   六面是否暴露

    # ---- 外观（OKLab，逐面）----
    albedo: np.ndarray | None = None       # (Nv, 6, 3) f32
    albedo_conf: np.ndarray | None = None  # (Nv, 6) f32    0 = 未被任何视角观测
    freq_energy: np.ndarray | None = None  # (Nv, 6) f32    纹理频率能量
    roughness: np.ndarray | None = None    # (Nv, 6) f32    可选

    # ---- 语义 ----
    mat_logits: np.ndarray | None = None   # (Nv, M) f32
    mat_taxonomy: list[str] = field(default_factory=list)   # 长度 M
    instance_id: np.ndarray | None = None  # (Nv,) int32    部件分组

    # ---- 约定 ----
    up_axis: Literal["Y"] = "Y"
    color_space: Literal["oklab"] = "oklab"

    # ---- 查询与校验 ----
    @property
    def n_voxels(self) -> int: ...
    def has(self, field_name: str) -> bool: ...
    def validate(self) -> list["IRIssue"]:
        """结构性校验。返回问题列表而非抛异常，
        让调用方决定哪些问题是致命的。"""
    def subset(self, mask: np.ndarray) -> "VoxelIR": ...
    def bbox(self) -> tuple[np.ndarray, np.ndarray]: ...
```

### 3.1 缺字段的降级行为（必须实现并测试）

这是集成故障最常见的来源，所以在契约层就规定死：

| 缺失字段 | 降级行为 | 质量影响 |
|---|---|---|
| `normal` | 从占用场用 Sobel 估计 | 轻微 |
| `residual` | **禁用亚体素拟合**，全用全立方体 | 中等（屋顶/曲面变阶梯状） |
| `face_vis` | 从占用场邻域计算 | 无（本就可推导） |
| `albedo` | 全部用材质类先验色 | **严重**，仅适用于纯语义输入（如 CAD） |
| `albedo_conf` | 视为全 1.0 | 轻微（未观测面不会被特殊处理） |
| `freq_energy` | 禁用纹理频率项 | 轻微（可能把平墙配成花岗岩） |
| `mat_logits` | **退化为纯颜色匹配** | 中等（材质语义丢失） |
| `instance_id` | 视为单一实例 | 轻微 |

**降级必须记入 `run_report.json`**，让用户知道结果为什么不如预期，而不是默默给出劣化结果。

### 3.2 序列化

```python
# mcvox/io/ir_io.py
def save_ir(ir: VoxelIR, path: Path) -> None: ...
def load_ir(path: Path) -> VoxelIR: ...
```

`.mcir` = zip 容器（`arrays.npz` + `meta.json`）。核心库拒绝加载不兼容 major 版本并给出明确升级提示。

**IR 落盘是架构红利的兑现点**：让 L3 团队用固定样本独立开发、让 GPU/CPU worker 分离调度、让回归测试排除上游随机性——三件事都依赖它。

---

## 4. Stage 协议与管线

```python
# mcvox/core/protocols.py
from typing import Protocol, runtime_checkable

@runtime_checkable
class Stage(Protocol):
    name: str
    version: str
    def requires(self) -> set[str]: ...
    def provides(self) -> set[str]: ...
    def run(self, ir: VoxelIR, cfg: Config) -> VoxelIR: ...
```

```python
# mcvox/core/pipeline.py
class Pipeline:
    def __init__(self, stages: list[Stage], cache: Cache | None = None): ...

    def validate(self) -> list[str]:
        """★ 静态校验：逐 stage 累积 provides，检查每个 requires 是否已满足。
        在跑之前就报出「亚体素拟合排在了残差计算之前」这类编排错误。"""

    def run(self, ir: VoxelIR, cfg: Config) -> tuple[VoxelIR, RunReport]: ...
```

`requires`/`provides` 的价值在于**把编排错误从运行时前移到编排时**。八条路线各自组装管线，没有静态校验的话，错误会以"跑了 10 分钟后在某个 stage 里 KeyError"的形式出现。

---

## 5. 适配器机制

### 5.1 协议

```python
# mcvox/core/protocols.py
class ReconstructionAdapter(Protocol):
    def reconstruct(self, images: list[Path], cfg: Config) -> ReconResult: ...

class SegmentationAdapter(Protocol):
    def segment(self, image: NDArray, prompts: list[str] | None) -> list[Mask]: ...

class DelightAdapter(Protocol):
    def decompose(self, image: NDArray) -> IntrinsicChannels: ...

class Generative3DAdapter(Protocol):
    def generate(self, cond: TextOrImage, cfg: Config) -> Generated3D: ...
    def native_sparse_voxels(self) -> SparseVoxelGrid | None:
        """★ 若模型内部即稀疏体素（TRELLIS 的 SLAT / TRELLIS.2 的 O-Voxel），
        直接暴露，让单图路线跳过 mesh↔voxel 往返，省掉两次量化误差。
        返回 None 表示该适配器不支持此捷径。"""
```

### 5.2 注册与发现

```python
# 独立包 mcvox-adapters-trellis 的 pyproject.toml
[project.entry-points."mcvox.adapters"]
trellis2 = "mcvox_adapters_trellis:Trellis2Adapter"
```

```python
from mcvox.core.registry import get_adapter

adapter = get_adapter("generative3d", "trellis2")   # 未安装则抛出带安装指引的异常
```

### 5.3 缺失时的行为

```
$ mcvox convert photo.jpg --out house.litematic

✗ 单图路线需要 3D 生成适配器，但未找到可用实现。

  安装其一：
    pip install mcvox-adapters-trellis     # 推荐，支持稀疏体素捷径
    pip install mcvox-adapters-hunyuan3d

  或改用已安装的路线：
    mcvox convert model.glb ...            # 资产路线（无需额外适配器）
```

**错误信息必须给出可执行的下一步**，而不是 `ModuleNotFoundError`。这直接关系到"第三方开发者"这个 P0 角色的体验。

---

## 6. 关键模块 API

### 6.1 palette — 调色板

```python
from mcvox.palette import PaletteRegistry, analyze_gamut

reg = PaletteRegistry.build(
    mc_version="1.21",
    jar_path=Path("~/.minecraft/versions/1.21/1.21.jar"),
    resource_packs=[],
    biome="plains",              # 群系着色参数（草/叶/水）
)
reg.save(Path("data/palette/1.21_vanilla.parquet"))   # 只存派生数值，绝不存纹理

entry = reg.get("minecraft:grass_block")
entry.face_lab["up"]      # OKLab 均值
entry.face_std["up"]      # 色彩方差
entry.freq_energy["up"]   # 纹理频率能量
entry.attrs.gravity       # False
entry.attrs.tier          # 0

# ★ E1 实验入口
report = analyze_gamut(reg, sample_density=32)
report.max_nn_distance_field    # sRGB 立方体上的最近邻 ΔE₀₀ 场
report.coverage_ratio(threshold=5.0)
report.worst_hues()             # 最难表达的色相区间
report.to_html(Path("reports/E1.html"))
```

### 6.2 color — 色彩

```python
from mcvox.color import srgb_to_oklab, delta_e2000, blue_noise_3d, gamut_map

lab   = srgb_to_oklab(rgb, assume_linear=False)   # 内部先转线性再运算
d     = delta_e2000(lab_a, lab_b)                 # 评测用
mask  = blue_noise_3d(size=64, seed=0)            # void-and-cluster，预计算并固定
mapped = gamut_map(lab, reachable=reg.gamut_hull, mode="perceptual")
```

> `blue_noise_3d` 的结果**随版本固定并落盘**，不在运行时生成——这是 G5 可复现性的要求（[技术设计 §5.2](04-tech-design.md#52-可复现性g5硬门禁)）。

### 6.3 geom — 几何

```python
from mcvox.geom import voxelize_solid, compute_residual, estimate_normals, find_floating

occ, residual = voxelize_solid(
    mesh,
    voxels_per_meter=4.0,
    method="winding_number",     # 对非水密/自相交网格稳健
    close_gaps=True,             # 形态学闭运算补裂缝
)
normals  = estimate_normals(occ)
floating = find_floating(occ, anchor="ground")
```

### 6.4 blockify — L3 核心

```python
from mcvox.blockify import BlockifyPipeline, BlockifyConfig

cfg = BlockifyConfig(
    palette=reg,
    mapping="medieval",                 # 材质映射表
    weights=dict(color=1.0, material=0.6, texture=0.3, geom=0.4, rarity=0.1),
    solver="alpha_expansion",           # 或 "icm" / "greedy"（快速预览）
    dither="blue_noise_3d",
    subvoxel=True,                      # 亚体素几何拟合
    repair=["floating", "gravity", "support", "subvoxel_integrity"],
    budget=None,                        # 或 MaterialBudget(...)
    seed=0,
)

result = BlockifyPipeline(cfg).run(ir)
result.blocks        # (Nb, 3) 坐标 + block id + blockstate
result.manifest      # 材料清单
result.report        # 质量指标 + 降级记录
```

**这是全库最重要的一个入口。** 八条路线全部收敛到这一行调用。

### 6.5 io — 导出

```python
from mcvox.io import export

export(result, Path("house.litematic"))                    # 按扩展名分派
export(result, Path("house.nbt"), split_48=True)           # 原版 48³ 自动切分
export(result, Path("house.schem"), mc_version="1.21")
```

---

## 7. 路线层如何使用

路线是**薄适配层**，唯一职责是产出 IR。以资产路线（MVP）为例：

```python
# routes/asset/pipeline.py
from mcvox.core import VoxelIR, Provenance
from mcvox.geom import voxelize_solid, estimate_normals
from mcvox.color import srgb_to_oklab
from mcvox.semantic import taxonomy

def asset_to_ir(path: Path, cfg) -> VoxelIR:
    mesh = load_mesh(path)                                    # trimesh
    occ, residual = voxelize_solid(mesh, cfg.voxels_per_meter)
    albedo = sample_uv_per_face(mesh, occ)                    # glTF baseColor ≈ albedo
    mat    = taxonomy.from_material_names(mesh.materials)     # 材质名 → 类别

    return VoxelIR(
        coords=occ.coords, residual=residual,
        normal=estimate_normals(occ),
        albedo=srgb_to_oklab(albedo),
        albedo_conf=np.ones(albedo.shape[:2]),
        mat_logits=mat.logits, mat_taxonomy=taxonomy.CLASSES,
        voxels_per_meter=cfg.voxels_per_meter,
        provenance=Provenance.capture(route="asset", inputs=[path], config=cfg),
    )
```

之后所有路线共用同一段：

```python
ir     = <某条路线>_to_ir(...)
result = BlockifyPipeline(blockify_cfg).run(ir)
export(result, out_path)
```

> **这段三行代码就是 AR-1 架构规则的全部意义**：新增一条输入路线的成本，等于写一个 `*_to_ir` 函数。

---

## 8. 配置系统

分层覆盖，后者优先：

```
内置默认  →  ~/.mcvox/config.toml  →  项目 mcvox.toml  →  环境变量  →  CLI 参数 / API 入参
```

```toml
# mcvox.toml
[palette]
mc_version = "1.21"
biome = "plains"

[blockify]
mapping = "medieval"
solver = "alpha_expansion"
subvoxel = true
seed = 0

[blockify.weights]
color = 1.0
material = 0.6
texture = 0.3

[compute]
max_vram_gb = 22
mrf_workers = 16
cache_dir = "~/.mcvox/cache"
```

所有配置项**必须可序列化并进入 provenance**——否则可复现性无法成立。

---

## 9. 测试策略

| 层级 | 内容 | 要求 |
|---|---|---|
| 单元测试 | 色彩转换、几何算法、格式读写 | 覆盖率 ≥85%；色彩转换对已知参考值 |
| 契约测试 | IR 校验、**每个缺字段降级路径** | §3.1 表格每行一个测试，不可省 |
| 管线测试 | `Pipeline.validate()` 的编排错误检测 | 正反例都要有 |
| 黄金测试 | 固定 IR 样本 → 固定输出 | **逐字节比对**，守护 G5 |
| 往返测试 | 导出 → 读回 → 逐方块比对 | 每种格式 |
| **游戏内测试** | 导出文件在真实客户端加载放置 | **半自动，但不可跳过**（F-06 硬门禁） |
| 回归套件 | Objaverse 200 对象跑分 | 每次提交跑；指标入库可对比历史 |
| 性能基准 | 各 stage 耗时与显存 | 防止性能悄悄劣化 |

> 「游戏内测试」无法完全自动化，但它是唯一能验证 G1/G2 的手段。建议做成"每次发布前的人工 checklist + 录屏存档"，而不是因为难自动化就省略。

---

## 10. 版本与兼容

| 对象 | 策略 |
|---|---|
| 库版本 | 语义化版本。**v1.0 之前 API 可破坏性变更，之后严格遵守** |
| `IR_VERSION` | 独立于库版本。major 不兼容则拒绝加载并提示迁移 |
| 调色板数据 | 按 MC 版本 + 资源包哈希，多版本共存 |
| 材质映射表 | 语义化版本，声明兼容的 MC 版本范围 |
| 适配器 | 声明兼容的 mcvox 版本范围 |

**API 稳定性承诺分级**：`core` 与 `blockify.pipeline` 是稳定公开 API；各模块内部函数标记为 `_internal` 或在文档中注明不保证稳定。第三方只应依赖前者。

---

## 11. 实施顺序

公共库不是一次建成的。按下游依赖倒推：

| 顺序 | 模块 | 阻塞谁 |
|---|---|---|
| 1 | `core.types` (VoxelIR) + `io.ir_io` | **阻塞所有人**，最优先 |
| 2 | `palette` + `color` | 阻塞 E1 实验与全部 L3 |
| 3 | `io`（导出格式） | 阻塞 Phase 0 的链路验证 |
| 4 | `geom.voxelize` + `geom.residual` | 阻塞资产路线 MVP |
| 5 | `blockify.unary` + `solver` | 阻塞 MVP 出结果 |
| 6 | `eval` + `viz` | 阻塞迭代（没有度量就无法改进） |
| 7 | `blockify.subvoxel` + `repair` | Phase 2 |
| 8 | `appearance` + `semantic` | Phase 2–3，阻塞照片/点云路线 |

前三项是**全项目的关键路径起点**，任何人力都应优先投在这里。
