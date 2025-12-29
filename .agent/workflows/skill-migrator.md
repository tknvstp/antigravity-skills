---
description: 将 Claude Code Skills 迁移到 Antigravity 格式。当用户想迁移/转换一个 Claude Code skill 到 Antigravity 的 rule 或 workflow 时使用。
---

# Skill Migrator

本 Workflow 旨在指导用户将 Claude Code 的 `.skill` / `SKILL.md` 格式转换为符合 Antigravity 最佳实践的 `.agent/` 目录结构。

> **核心原则**：迁移不仅是格式转换，更是利用 Antigravity 特性（如 browser_subagent, task_boundary）进行能力增强的机会。

---

## 背景知识

**必须首先阅读**：Antigravity 平台核心能力与规范：
使用 `view_file` 读取：`{此项目根目录}/.agent/resources/skill-creator/references/antigravity-reference.md`

该文档包含：
- Mode System (PLANNING / EXECUTION / VERIFICATION)
- Artifact System (task.md / implementation_plan.md)
- 工具能力 (Browser, Image Generation)

---

## 格式映射表 (Anatomy Mapping)

| Claude Code (SKILL.md) | Antigravity 格式 | 说明 |
|------------------------|-----------------|------|
| `skill-name/SKILL.md` (Frontmatter) | `.agent/workflows/{skill}.md` (Frontmatter) | 定义触发条件（Trigger/Description） |
| `skill-name/SKILL.md` (Body) | `.agent/workflows/{skill}.md` (Body) | 主体指令，需适配文件引用方式 |
| `skill-name/scripts/` | `.agent/resources/{skill}/scripts/` | 保持为独立子目录 |
| `skill-name/references/` | `.agent/resources/{skill}/references/` | 保持为独立子目录 |
| `skill-name/assets/` | `.agent/resources/{skill}/assets/` | 保持为独立子目录 |

---

## 能力增强映射

迁移是增强 Skill 的绝佳时机。利用 Antigravity 独有工具优化原流程：

| 原 Claude 能力 | Antigravity 增强方案 | 优化场景示例 |
|----------------|--------------------|-------------|
| 无网络能力 | `browser_subagent` | 添加"验证文档最新版本"、"搜索最佳实践"步骤 |
| 无视觉能力 | `generate_image` | 添加"生成架构图"、"设计 UI 草图"步骤 |
| 隐式流程 | `task_boundary` | 显式管理 PLANNING -> EXECUTION -> VERIFICATION 状态流转 |
| 文本交互 | `notify_user` | 在关键节点（如部署前）请求结构化确认 |
| 记忆缺失 | Artifacts | 使用 `task.md` 自动追踪复杂任务进度 |

---

## 迁移步骤

### 阶段 1：规划与分析 (Mode: PLANNING)

1. **分析原 Skill**：由 Agent 读取原 `SKILL.md`，理解其意图、输入输出及依赖资源。
2. **制定计划**：创建 `implementation_plan.md`，列出文件迁移清单和增强点。

### 阶段 2：创建与迁移 (Mode: EXECUTION)

#### 1. 创建目录结构
// turbo
```bash
# 替换 {skill-name} 为实际名称
mkdir -p .agent/workflows
mkdir -p .agent/resources/{skill-name}/{scripts,references,assets}
```

#### 2. 迁移资源文件
// turbo
```bash
# 复制脚本
cp {原路径}/scripts/* .agent/resources/{skill-name}/scripts/
# 复制参考文档
cp {原路径}/references/* .agent/resources/{skill-name}/references/
# 复制资产
cp {原路径}/assets/* .agent/resources/{skill-name}/assets/
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
- **关键修改**：所有对相对路径的引用（如 `[ref](referenecs/doc.md)`）必须改为 Antigravity 兼容的引用方式。

**引用规范**：
Antigravity 中 Agent 读取文件需明确路径。
推荐写法：
> 详细信息请参考 `references/doc.md`（位于本 Skill 的资源目录 `.agent/resources/{skill-name}/references/`）。请使用 `view_file` 读取。

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
        │   └── init_skill.py  (或废弃，改用 Agent 指令)
        └── references/
            ├── workflows.md
            └── output-patterns.md
```

---

## 工具速查表

| 工具 | 用途 | 必须使用场景 |
|-----|------|-------------|
| `task_boundary` | 状态管理 | 任何复杂的多步骤任务 |
| `view_file` | 读取文件 | 替代原 Skill 中的 "Read x file" 指令 |
| `write_to_file` | 创建文件 | 生成 output 或新代码 |
| `browser_subagent` | 浏览器操作 | 原 Skill 无法实现的在线任务 |
