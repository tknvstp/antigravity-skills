---
name: skill-creator
description: 创建高质量 Antigravity Skill 的指南。当用户想要创建新 Skill（或更新现有 Skill）以使用专业知识、工作流或工具集成扩展 Agent 能力时使用此 Skill。
---

# Skill Creator

本 Skill 提供了创建高效 Antigravity Skills 的完整指南和规范。

## 关于 Skills

Skills 是模块化的、自包含的包，通过提供专业知识、工作流和工具来扩展 Agent 的能力。把它们看作是特定领域或任务的"入职指南"——它们将 Agent 从通用助手转变为具备专业程序性知识的专家。

### Skills 提供什么

1. **专业工作流** - 特定领域的多步骤程序
2. **工具集成** - 处理特定文件格式或 API 的指令
3. **领域专长** - 公司特定的知识、架构、业务逻辑
4. **打包资源** - 用于复杂重复任务的脚本、参考资料和资产

### 选择 Rule 还是 Workflow

在 Antigravity 中，你可以创建两种类型的 Skill：

| 维度 | Rule (`.agent/rules/`) | Workflow (`.agent/workflows/`) |
|-----|------------------------|-------------------------------|
| **触发方式** | 自动/手动/模型判断 | 用户显式调用 (`/命令名`) |
| **适用场景** | 代码规范、风格约束、自动检查 | 复杂多步骤任务、专业领域流程 |
| **资源依赖** | 通常无或很少 | 可能需要 scripts/assets/references |
| **示例** | 代码审查规则、命名规范 | PDF 处理、项目初始化、文档生成 |

**决策树**：

```
你的能力需要...
├── 在特定场景自动激活？ → Rule (trigger: model)
├── 始终生效？ → Rule (trigger: always)
├── 用户手动选择激活？ → Rule (trigger: manual) 或 Workflow
└── 用户通过 /命令 显式调用？ → Workflow
    └── 需要打包资源 (脚本/模板/参考文档)？ → Workflow + resources/
```

## 核心原则

### 简洁至上 (Concise is Key)

上下文窗口是公共资源。Skills 与其他所有内容共享它：系统提示、对话历史、其他 Skill 的元数据以及用户的实际请求。

**默认假设：Agent 已经非常聪明。** 只添加 Agent 尚未拥有的上下文。对每条信息进行挑战："Agent 真的需要这个解释吗？"以及"这段内容值得它的 token 成本吗？"

相比冗长的解释，更倾向于使用简洁的示例。

### 设定适当的自由度 (Set Appropriate Degrees of Freedom)

将特异性水平与任务的脆弱性和可变性相匹配：

- **高自由度 (文本指令)**：当多种方法有效、决策取决于上下文或由启发式指导时使用。
- **中自由度 (伪代码或带参数的脚本)**：当存在首选模式、可接受一定变化或配置影响行为时使用。
- **低自由度 (特定脚本、少量参数)**：当操作脆弱且易错、一致性至关重要或必须遵循特定顺序时使用。

把 Agent 想象成在探路：狭窄的桥梁需要特定的护栏（低自由度），而开阔的田野允许许多路线（高自由度）。

### Skill 解剖结构 (Anatomy of a Skill)

Antigravity Skill 由位于 `.agent/workflows/` 的主文件和位于 `.agent/resources/` 的可选资源组成：

```
.agent/
├── workflows/
│   └── skill-name.md        (必需，包含元数据和指令)
└── resources/
    └── skill-name/          (可选，打包资源)
        ├── scripts/         - 可执行代码 (Python/Bash 等)
        ├── references/      - 需按需加载到上下文的文档
        └── assets/          - 用于输出的文件 (模板、图标等)
```

#### Workflows 文件 (`.agent/workflows/*.md`)

每个 Workflow 文件包含：

- **Frontmatter** (YAML): 包含 `name` 和 `description` 字段。这是 Agent 决定何时使用该 Skill 的唯一依据，因此清晰全面地描述 Skill 的功能及适用场景至关重要。
- **Body** (Markdown): 使用 Skill 的指令和指南。仅在 Skill 触发后加载（如果触发）。

#### 打包资源 (`.agent/resources/skill-name/`)

资源目录包含三类文件，各有不同用途和使用方式：

##### Scripts (`scripts/`)

**定义**：需要确定性可靠性或避免重复重写的可执行代码。

**何时需要**：
- 复杂计算或转换（PDF 处理、数据解析）
- 需要外部依赖的操作（调用系统命令、库函数）
- 重复任务中追求一致性

**使用方式**：
在 Workflow Body 中指导 Agent 执行：
```markdown
## 步骤 2：处理 PDF
使用 `run_command` 执行：
```bash
python {工作区根目录}/.agent/resources/{skill-name}/scripts/rotate_pdf.py --input input.pdf --output output.pdf
```
```

**设计原则**：
- 使用命令行参数接收输入
- 输出清晰的状态信息
- 返回明确的退出码

---

##### References (`references/`)

**定义**：Agent 在工作过程中需要按需加载到上下文的知识文档。

**何时需要**：
- 领域知识过于庞大无法全部放入 Workflow Body（>500 行）
- 信息可能按需选择性使用
- 内容需要独立维护更新

**使用方式**：
在 Workflow Body 中提供加载引导：
```markdown
## 背景知识
在开始前，**必须**使用 `view_file` 阅读：
- API 规范：`{工作区根目录}/.agent/resources/{skill-name}/references/api.md`

如需深入了解架构，可选阅读：
- 架构文档：`{工作区根目录}/.agent/resources/{skill-name}/references/architecture.md`
```

**与 Workflow Body 的关系**：
| 放在 Workflow Body | 放在 References |
|-------------------|----------------|
| 核心指令和步骤 | 背景知识和详细说明 |
| 始终需要的信息 | 按需加载的信息 |
| <500 行 | 可以很大 |

---

##### Assets (`assets/`)

**定义**：不加载到上下文，而是用于 Agent 产出物的文件。

**何时需要**：
- 输出需要固定的模板结构
- 需要嵌入品牌资产（logo、图标）
- 提供项目骨架或配置模板

**使用方式**：
在 Workflow Body 中指导复制或引用：
```markdown
## 步骤 3：生成报告
1. 复制模板到输出目录：
   ```bash
   cp {工作区根目录}/.agent/resources/{skill-name}/assets/template.html ./output/
   ```
2. 将 logo 复制到输出目录：
   ```bash
   cp {工作区根目录}/.agent/resources/{skill-name}/assets/logo.png ./output/
   ```
```

**常见类型**：
| 类型 | 示例 | 使用场景 |
|-----|------|--------|
| 模板文件 | `template.html`, `report.md` | 报告生成、页面输出 |
| 品牌资产 | `logo.png`, `icon.svg` | 需要一致品牌的输出 |
| 配置骨架 | `config.json`, `.env.example` | 项目初始化 |
| 代码骨架 | `hello-world/` | Webapp 脚手架 |

### 渐进式披露设计原则 (Progressive Disclosure)

Skills 使用三级加载系统高效管理上下文：

1. **Metadata (name + description)** - 始终在上下文中 (~100 words)
2. **Workflow Body** - 当 Skill 触发时 (<5k words)
3. **Bundled resources** - 按需加载

#### 渐进式披露模式

保持 Workflow Body 精简（<500行）。

**模式 1：带引用的高级指南**

```markdown
# PDF Processing

## 高级功能

- **表单填充**: 详见 [form-guide](file:///abs/path/to/.agent/resources/pdf/references/forms.md)
- **API 参考**: 详见 [api-ref](file:///abs/path/to/.agent/resources/pdf/references/reference.md)
```

**模式 2：特定领域组织**

对于多领域 Skill，按领域组织内容以避免加载无关上下文。

**模式 3：条件细节**

展示基本内容，链接到高级内容。

**重要准则**：
- **绝对路径**：在 Antigravity 中引用资源文件时，必须使用绝对路径或基于 workspace 的明确路径。

## Skill 创建流程

创建 Skill 涉及以下步骤：

1. **理解**：通过具体示例理解 Skill
2. **规划**：规划可复用的 Skill 内容 (scripts, references, assets)
3. **初始化**：创建目录结构
4. **编辑**：编写资源和 Workflow 文件
5. **验证**：测试并迭代

### 步骤 1：理解 Skill

明确 Skill 将如何被使用。例如："用户会说'帮我去掉这张图的红眼'或'旋转这张图'。"

### 步骤 2：规划内容

分析示例以确定需要哪些资源：
- 旋转 PDF -> 需要 `scripts/rotate_pdf.py`
- 构建 Webapp -> 需要 `assets/hello-world/` 模板
- 查询数据库 -> 需要 `references/schema.md`

### 步骤 3：初始化结构

在 Antigravity 中，你需要创建正确的文件结构。你可以使用 Agent 指令来自动创建。

推荐结构：
```bash
mkdir -p .agent/workflows
mkdir -p .agent/resources/<skill-name>/{scripts,references,assets}
```

### 步骤 4：编辑 Skill

编写 `.agent/workflows/<skill-name>.md` 和相关资源。

#### 学习经过验证的设计模式

参考以下资源（位于 `.agent/resources/skill-creator/references/`）：

- **多步骤流程**: 见 `workflows.md` 了解顺序工作流和条件逻辑
- **特定输出格式**: 见 `output-patterns.md` 了解模板和示例模式
- **平台能力**: 见 `antigravity-reference.md` 了解如何利用 Antigravity 工具

#### 编写 Workflow 文件

**Frontmatter**:
```yaml
---
name: my-skill
description: Comprehensive guide for [task]. Use when [specific context].
---
```

**Body**:
编写清晰的指令。

### 步骤 5：验证与迭代

在实际任务中使用 Skill，观察效果，并根据反馈调整。

---
**注意**：在 Antigravity 中，我们不使用 `.skill` 压缩包格式，而是直接以文件形式存在于 `.agent` 目录中。
