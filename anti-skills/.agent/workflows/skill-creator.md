---
description: 创建新的 skill 或 workflow，扩展 Agent 能力。当用户想创建/更新一个扩展 Claude 能力的 skill（包含专业知识、工作流或工具集成）时使用。
---

# Skill Creator

本 workflow 指导如何创建有效的 skills。

## 关于 Skills

Skills 是模块化、自包含的包，通过提供专业知识、工作流和工具来扩展 Agent 能力。可将其视为特定领域或任务的"入职指南"——将通用 Agent 转变为具备程序性知识的专业 Agent。

### Skills 提供什么

1. **专业工作流** - 特定领域的多步骤程序
2. **工具集成** - 处理特定文件格式或 API 的指令
3. **领域专业知识** - 公司特定知识、数据模式、业务逻辑
4. **资源打包** - 脚本、参考资料和复杂重复任务的资产

---

## 核心原则

### 简洁是关键

上下文窗口是公共资源。Skills 与系统提示、对话历史、其他 Skills 元数据和实际用户请求共享上下文窗口。

**默认假设：Agent 已经非常智能。** 只添加 Agent 尚不具备的上下文。质疑每条信息："Agent 真的需要这个解释吗？" "这段内容值得占用 token 吗？"

优先使用简洁示例而非冗长解释。

### 设置适当的自由度

根据任务的脆弱性和可变性匹配具体程度：

| 自由度 | 适用场景 | 实现方式 |
|-------|---------|---------|
| 高 | 多种方法都有效，决策依赖上下文 | 文本指令 |
| 中 | 存在首选模式，可接受一定变化 | 伪代码或带参数的脚本 |
| 低 | 操作脆弱易错，一致性关键 | 具体脚本，少量参数 |

---

## Skill 结构（Antigravity 格式）

在 Antigravity 中，skills 以 `.agent/` 目录结构组织：

```
.agent/
├── workflows/              # 斜线命令触发的工作流 (/命令名)
│   └── skill-name.md
├── rules/                  # 自动/手动激活的规则
│   └── skill-name.md
└── resources/              # 辅助资源
    └── skill-name/
        ├── scripts/        # 可执行脚本
        ├── references/     # 参考文档（可按需读取）
        └── assets/         # 输出资源（模板、图标等）
```

### Workflow vs Rule

| 类型 | 触发方式 | 适用场景 |
|-----|---------|---------|
| Workflow | 用户主动调用 `/命令` | 流程导向、多步骤任务 |
| Rule (trigger: model) | 模型根据描述自动激活 | 工具类、格式处理类 |
| Rule (trigger: always) | 始终生效 | 全局约束、编码规范 |
| Rule (trigger: manual) | 用户 `@文件名` 引用 | 按需参考的指南 |

---

## 渐进式披露设计

Skills 使用三级加载系统高效管理上下文：

1. **元数据（description）** - 始终在上下文中（~100 词）
2. **主体内容** - 触发后加载（<5k 词）
3. **资源文件** - 按需由 Agent 读取（无限制）

### 引用外部文件的正确方式

**关键**：Antigravity 需要明确指导才能正确读取外部文件。

```markdown
使用 `view_file` 工具读取参考文档。路径格式：
`{工作区根目录}/.agent/resources/skill-creator/{文件名}`

| 内容 | 文件名 |
|-----|--------|
| 工作流模式 | `workflows.md` |
| 输出格式模式 | `output-patterns.md` |

**操作步骤**：
1. 确定工作区根目录
2. 拼接完整绝对路径
3. 调用 view_file 读取
```

---

## Skill 创建流程

1. **理解 skill 的具体用例**
2. **规划可复用内容**（脚本、参考资料、资产）
3. **创建目录结构**
4. **编写主文件**
5. **验证和迭代**

### 步骤 1：理解具体用例

通过提问收集信息：
- "这个 skill 应该支持什么功能？"
- "能给出一些使用示例吗？"
- "什么情况下应该触发这个 skill？"

### 步骤 2：规划可复用内容

分析每个用例：
1. 考虑如何从零执行
2. 识别重复执行时有帮助的脚本、参考资料和资产

### 步骤 3：创建目录结构

```bash
# 在项目根目录创建 skill 目录
mkdir -p .agent/workflows
mkdir -p .agent/resources/skill-name/scripts
mkdir -p .agent/resources/skill-name/references
mkdir -p .agent/resources/skill-name/assets
```

### 步骤 4：编写主文件

**YAML Frontmatter**：

```yaml
---
trigger: model  # 或 always, manual
description: 详细描述 skill 功能和触发场景。包含所有"何时使用"信息。
globs:          # 可选，文件类型过滤
---
```

**主体内容**：
- 使用祈使语气
- 保持简洁（<500 行）
- 将详细内容拆分到 resources 文件

### 步骤 5：验证和迭代

1. 在真实任务中使用
2. 注意困难或低效之处
3. 更新内容并再次测试

---

## 设计模式参考

需要了解更多模式时，使用 `view_file` 读取：

- **多步骤流程**：`{根目录}/.agent/resources/skill-creator/workflows.md`
- **输出格式/质量标准**：`{根目录}/.agent/resources/skill-creator/output-patterns.md`

---

## 关键词

skill, workflow, rule, 创建技能, 扩展能力, agent 定制
