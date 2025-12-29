---
description: 将 Claude Code Skills 迁移到 Antigravity 格式。当用户想迁移/转换一个 Claude Code skill 到 Antigravity 的 rule 或 workflow 时使用。
---

# Skill Migrator

本 Workflow 旨在指导用户将 Claude Code 的 `.skill` / `SKILL.md` 格式转换为符合 Antigravity 最佳实践的 `.agent/` 目录结构。

> **核心原则**：迁移不仅是格式转换，更是利用 Antigravity 特性（如 browser_subagent, task_boundary）进行能力增强的机会。

---

## 背景知识与指南

在开始迁移前，**必须首先阅读**以下参考文档以建立必要的上下文：

1. **Antigravity 平台核心能力**（了解模式系统、Artifacts、工具）：
   使用 `view_file` 读取：`{此项目根目录}/.agent/resources/skill-migrator/references/antigravity-reference.md`

2. **迁移映射指南**（格式映射表、能力增强对照、工具速查）：
   使用 `view_file` 读取：`{此项目根目录}/.agent/resources/skill-migrator/references/migration-guide.md`

---

## 迁移决策

### 目标格式选择

```
原 Claude Code Skill 的用途是...
├── 代码规范、自动检查类 → 迁移为 Rule
│   └── 放置于 .agent/rules/{skill-name}.md
├── 复杂多步骤流程类 → 迁移为 Workflow
│   └── 放置于 .agent/workflows/{skill-name}.md
└── 不确定？ → 默认选择 Workflow（更灵活）
```

### 资源需求判断

```
原 Skill 包含...
├── 可执行脚本 (Python/Bash/Node) → 迁移到 scripts/
├── 背景文档/API说明/架构图 → 迁移到 references/
├── 模板/logo/配置骨架 → 迁移到 assets/
└── 仅 SKILL.md → 无需 resources/ 目录
```

---

## 迁移步骤

### 阶段 1：规划与分析 (Mode: PLANNING)

1. **分析原 Skill**：由 Agent 读取原 `SKILL.md`，理解其意图、输入输出及依赖资源。
2. **制定计划**：创建 `implementation_plan.md`，列出文件迁移清单和增强点（参考 `migration-guide.md` 中的增强映射）。

### 阶段 2：创建与迁移 (Mode: EXECUTION)

#### 1. 创建目录结构
// turbo
```bash
# 替换 {skill-name} 为实际名称
mkdir -p .agent/workflows
mkdir -p .agent/resources/{skill-name}/{scripts,references,assets}
```

#### 2. 迁移资源文件

根据资源类型分别处理：

**Scripts 迁移**：
// turbo
```bash
cp {原路径}/scripts/* .agent/resources/{skill-name}/scripts/
```

迁移后更新 Workflow 中的调用方式：
```diff
- 运行 scripts/process.py
+ 使用 `run_command` 执行：
+ python {工作区根目录}/.agent/resources/{skill-name}/scripts/process.py [参数]
```

**References 迁移**：
// turbo
```bash
cp {原路径}/references/* .agent/resources/{skill-name}/references/
```

迁移后更新 Workflow 中的引用方式：
```diff
- 参考 `references/api.md`
+ 请使用 `view_file` 阅读：`{工作区根目录}/.agent/resources/{skill-name}/references/api.md`
```

**Assets 迁移**：
// turbo
```bash
cp {原路径}/assets/* .agent/resources/{skill-name}/assets/
```

迁移后更新 Workflow 中的使用方式：
```diff
- 使用 assets/template.html 作为模板
+ 使用 `run_command` 复制模板到输出目录：
+ cp {工作区根目录}/.agent/resources/{skill-name}/assets/template.html ./output/
```

#### 3. 编写 Workflow 文件 (`.agent/workflows/{skill-name}.md`)

**Frontmatter**:
```yaml
---
name: {skill-name}
description: {详细描述，包含触发条件}
---
```

**Body (Markdown)**:
- 将原 `SKILL.md` 内容迁移至此。
- **关键修改**：所有对相对路径的引用必须参考 `migration-guide.md` 中的"路径引用规范"进行修改。

### 阶段 3：验证与交付 (Mode: VERIFICATION)

1. **路径检查**：确保 workflow 中提到的所有资源路径都真实存在。
2. **模拟运行**：在 Antigravity 中尝试触发该 Skill，验证流程。
3. **交付**：创建 `walkthrough.md` 记录迁移结果。

---

## 迁移示例：Skill-Creator

**原始结构**：
```
skill-creator/
├── SKILL.md
├── scripts/
│   └── init_skill.py
└── references/
    ├── workflows.md
    └── output-patterns.md
```

**迁移后结构 (Antigravity)**：
```
.agent/
├── workflows/
│   └── skill-creator.md
└── resources/
    └── skill-creator/
        ├── scripts/
        │   └── init_skill.py
        └── references/
            ├── workflows.md
            └── output-patterns.md
```
