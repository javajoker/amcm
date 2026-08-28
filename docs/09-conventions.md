# 工程约定

> 代码、测试、文档、版本、许可规范
> 版本 v0.1 · 2026-08

---

## 1. 技术栈与环境

| 项 | 选择 |
|---|---|
| 语言 | Python 3.11+ |
| 依赖管理 | `uv`（或 Poetry），锁文件入库 |
| 类型检查 | `mypy --strict`（`core` 与 `blockify` 强制，其余渐进） |
| 格式化 / lint | `ruff` |
| 测试 | `pytest` + `pytest-benchmark` |
| 数组 | NumPy 为主；GPU 部分用 PyTorch |
| 文档 | Markdown + `mkdocs-material` |

### 依赖分层

```toml
[project]
dependencies = [           # 核心，必须保持轻量（目标 < 50 MB）
  "numpy", "scipy", "pyyaml", "pydantic", "typer",
  "amulet-nbt", "trimesh", "colour-science",
]

[project.optional-dependencies]
geom  = ["open3d", "libigl", "pymeshlab"]
solve = ["PyMaxflow"]
gpu   = ["torch"]
eval  = ["lpips", "scikit-image"]
all   = ["mcvox[geom,solve,gpu,eval]"]
```

**重模型（VGGT / TRELLIS / SAM2 / RGB↔X）一律不进 `optional-dependencies`**，而是独立的 `mcvox-adapters-*` 包（[技术设计 ADR-07](04-tech-design.md#adr-07--重模型通过适配器协议注入全部为可选依赖)）。

---

## 2. 代码规范

### 2.1 依赖方向（CI 强制）

```
cli → blockify → {palette, color, geom, appearance, semantic, io} → core
```

- `core` 不依赖任何其他 mcvox 模块
- `color` 与 `geom` 不依赖 `palette`
- 任何模块不得反向依赖 `cli`
- 路线包（`routes/*`）不得 import `mcvox.blockify` 的**内部**模块，只能用 `blockify.pipeline` 的公开入口

CI 用 `import-linter` 或自写脚本校验。**违反即失败，不接受人工豁免**——[技术设计 AR-1](04-tech-design.md#三条不可违反的架构规则) 是最容易在赶工期时被侵蚀的规则。

### 2.2 Stage 编写规范

```python
class MyStage:
    name = "my_stage"
    version = "1.0"

    def requires(self) -> set[str]:
        return {"coords", "albedo"}          # 必须如实声明

    def provides(self) -> set[str]:
        return {"mat_logits"}

    def run(self, ir: VoxelIR, cfg: Config) -> VoxelIR:
        ...                                   # 纯函数：不改入参，返回新 IR
```

- **不得原地修改入参 IR**——返回新对象。这是缓存与可复现性的前提
- 随机性只能来自 `cfg.seed`，不得使用全局 `np.random`
- 不做 I/O（读写文件由调用方负责），除非本身就是 I/O stage
- `requires`/`provides` 必须如实声明，否则静态校验失效

### 2.3 数值约定

| 项 | 约定 |
|---|---|
| 颜色空间 | IR 内统一 **OKLab**；导出/展示时才转 sRGB |
| 色彩平均 | **必须在线性空间**计算，绝不在 gamma 空间平均 |
| 坐标系 | 右手系，**Y 轴向上**，整数体素坐标，原点在包围盒最小角 |
| 浮点 | 数组默认 `float32`；**MRF 代价在求解前量化为 int32** |
| 角度 | 一律弧度，函数名不加 `_rad` 后缀 |

> 「线性空间平均」和「代价量化为整数」是两个具体但影响面很大的约定，各配一个单元测试守护。前者错了颜色会系统性偏暗，后者错了可复现性直接失效。

### 2.4 命名

- 模块/函数 `snake_case`，类 `PascalCase`，常量 `UPPER_SNAKE`
- 内部 API 前缀 `_`，或在 docstring 标注"不保证稳定"
- MC 方块 ID 一律用完整命名空间：`minecraft:oak_planks`
- 避免缩写，除非是领域通用（`ir`、`lab`、`sdf`、`mrf`、`erp`）

---

## 3. 测试规范

### 3.1 分层

| 层级 | 位置 | 何时跑 |
|---|---|---|
| 单元测试 | `tests/unit/` | 每次提交 |
| 契约测试 | `tests/contract/` | 每次提交 |
| 黄金测试 | `tests/golden/` | 每次提交 |
| 往返测试 | `tests/roundtrip/` | 每次提交 |
| 回归套件 | `tests/regression/` | 每次提交（小集）/ 每周（全集） |
| 性能基准 | `tests/bench/` | 每周 |
| 游戏内验证 | 人工 + `docs/verification/` | 每阶段出口 |

### 3.2 必须存在的测试

- [ ] **每个 IR 缺字段降级路径一个测试**（[公共库 §3.1](05-core-library.md) 表格逐行）
- [ ] 色彩转换对已知参考值（sRGB ↔ Lab ↔ OKLab 往返）
- [ ] 线性空间平均的正确性
- [ ] 可复现性：相同输入跑两遍逐字节相同
- [ ] `Pipeline.validate()` 的编排错误检测（正反例都要）
- [ ] 每种导出格式的往返一致性
- [ ] 48³ 上限的边界行为

### 3.3 游戏内验证

无法自动化，但是唯一能验证 G1/G2 目标的手段。规程：

1. 每阶段出口时执行
2. 覆盖：全立方体 / 楼梯 / 台阶 / 重力方块 / 需支撑方块 / 超 48³ 切分
3. **录屏存档**到 `docs/verification/<date>-<phase>/`
4. 结果记入检查清单

**不接受"因为难自动化所以跳过"。**

---

## 4. 数据资产规范

### 4.1 🔒 红线

**仓库中不出现任何游戏纹理像素。** CI 扫描图片文件并在发现时失败。

允许提交的是**派生数值**：`(方块,面) → Lab 均值 / 方差 / 频率能量`。这些是统计事实而非美术资产。

### 4.2 存放

| 资产 | 路径 | 版本化 |
|---|---|---|
| 调色板统计 | `data/palette/<mcver>_<packhash>.parquet` | 随 MC 版本 |
| 材质映射表 | `data/mappings/<style>.yaml` | 语义化版本 |
| 方块属性 | `data/blocks/<mcver>.yaml` | 随 MC 版本 |
| 共现统计 | `data/priors/cooccur_<corpus>.npz` | 随语料 |
| 亚体素模板 | `data/subvoxel/templates.json` | 语义化版本 |
| 大文件 | Git LFS 或外部存储 + 校验和 | — |

### 4.3 语料溯源

任何进入统计表或模型权重的语料，必须在 `data/priors/SOURCES.md` 记录：来源 URL、许可、获取日期、是否允许再分发。**许可不明的数据不得进入发布产物。**

---

## 5. 版本与发布

### 5.1 版本策略

| 对象 | 策略 |
|---|---|
| 库版本 | 语义化。**v1.0 前 API 可破坏性变更，之后严格遵守** |
| `IR_VERSION` | 独立编号。major 不兼容则拒绝加载并提示迁移路径 |
| 调色板数据 | MC 版本 + 资源包哈希，多版本共存 |
| 映射表 | 语义化，声明兼容的 MC 版本范围 |
| 适配器 | 声明兼容的 mcvox 版本范围 |

### 5.2 API 稳定性分级

| 级别 | 范围 | 承诺 |
|---|---|---|
| **稳定** | `mcvox.core.*`、`blockify.pipeline`、`io.export`、CLI 命令 | v1.0 后遵守语义化版本 |
| **实验** | 各模块内部函数、新增未定型的 API | 可随时变更，docstring 标注 |
| **内部** | `_` 前缀 | 无承诺 |

第三方只应依赖"稳定"级。文档需明确标注每个公开 API 的级别。

### 5.3 变更记录

`CHANGELOG.md` 遵循 Keep a Changelog。**IR schema 的任何变更必须单列一节**并说明迁移方式——它是八条路线共用的契约，改动影响面最大。

---

## 6. 文档规范

### 6.1 文档即契约

以下变更**必须同步更新文档，否则 PR 不予合并**：

- IR schema 变更 → [公共库设计 §3](05-core-library.md)
- 新增/移除架构决策 → [技术设计 §3 ADR](04-tech-design.md)
- 新增输入路线 → [分路线计划](07-route-plans.md)
- 质量阈值变更 → [PRD §6](00-PRD.md) + [检查清单](08-CHECKLIST.md)

### 6.2 参考文献索引

新增条目**必须填写「→ 用在哪」**，说明它在管线中的具体落点，否则不予合并。

被证伪或放弃的方案**不删除**，移入[参考文献索引](03-references.md)末尾的「已排除方案」并注明原因——避免团队重复踩坑。

### 6.3 已验证标记

实际集成并验证可用的实现，在链接后标注 `[已验证 YYYY-MM]`。

---

## 7. 提交与评审

### 7.1 提交信息

```
<type>(<scope>): <简述>

<正文：为什么这么改，不是改了什么>
```

`type`：`feat` / `fix` / `perf` / `refactor` / `test` / `docs` / `data` / `chore`
`scope`：`core` / `palette` / `color` / `geom` / `blockify` / `io` / `eval` / `route-<name>` / `cli`

### 7.2 评审重点

按本项目的实际风险排序：

1. **是否违反依赖方向**（AR-1/AR-2）——最高优先级，最难事后补救
2. **是否引入了不由 seed 控制的随机性**——破坏可复现性硬门禁
3. **是否原地修改了 IR**——破坏缓存正确性
4. **新 stage 的 `requires`/`provides` 是否如实**——影响静态校验
5. 是否有对应测试，尤其是降级路径
6. 是否需要同步文档

---

## 8. 许可

| 组件 | 许可 |
|---|---|
| `mcvox` 核心库、CLI、回归框架 | Apache-2.0（建议） |
| 基础材质映射表（vanilla + 1 套风格） | Apache-2.0 |
| 精修风格映射表包 | 商业 |
| 托管服务 | 商业 |
| 模型权重 | 待定，取决于训练数据许可 |

**每个源文件不加许可头**（用根目录 LICENSE + `pyproject.toml` 声明即可），但 `data/` 下每个资产目录需有 `LICENSE` 或 `SOURCES.md` 说明来源与许可。

### 商标与合规

- Minecraft 是 Mojang/Microsoft 的商标。README 与产品页需有非隶属声明
- 遵守 Mojang 的品牌与内容使用准则
- 不再分发任何游戏美术资源（§4.1 红线）
