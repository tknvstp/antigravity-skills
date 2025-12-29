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

##### Scripts (`scripts/`)

需要确定性可靠性或重复重写的任务的可执行代码。

- **何时包含**：当代码被重复重写或需要确定性时
- **示例**：`scripts/rotate_pdf.py` 用于 PDF 旋转任务
- **优点**：Token 高效、确定性、甚至可以在不加载到上下文的情况下执行

##### References (`references/`)

旨在按需加载到上下文中以告知 Agent 过程和思维的文档和参考资料。

- **何时包含**：Agent 在工作时需要参考的文档
- **示例**：`references/finance.md` (财务架构), `references/api_docs.md` (API 规范)
- **最佳实践**：如果文件很大 (>10k words)，在 Workflow 中包含 grep 搜索模式。避免重复——信息应存在于 Workflow 或 Reference 中，而非两者都存。

##### Assets (`assets/`)

不打算加载到上下文中，而是用于 Agent 产生的输出中的文件。

- **何时包含**：当 Skill 需要用于最终输出的文件时
- **示例**：`assets/logo.png` (品牌资产), `assets/template.html` (前端模板)

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
