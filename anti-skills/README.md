# Anti-Skills：Claude Code Skills → Antigravity 迁移

将 Claude Code 的 Skills 迁移到 Antigravity 平台使用。

## 目录结构

```
anti-skills/
├── README.md                    # 本文件（迁移指南）
└── .agent/                      # 直接复制到项目根目录即可使用
    ├── rules/                   # 自动/手动激活的规则
    │   └── internal-comms.md
    ├── workflows/               # 斜线命令触发的工作流
    └── resources/               # 辅助资源（示例、参考）
        └── internal-comms/
            └── examples/
```

## 使用方法

```bash
# 复制 .agent 目录到你的项目
cp -r anti-skills/.agent /path/to/your-project/
```

---

# 迁移规范

## Rules 文件格式

```yaml
---
trigger: model
description: 规则描述，用于模型判断是否激活
globs:
---

# 规则标题

（规则内容）
```

### trigger 取值

| 值 | 说明 |
|---|------|
| `model` | 模型根据 description 自动决定是否激活 |
| `always` | 始终生效 |
| `manual` | 需要用户通过 `@文件名` 手动引用 |

---

## 引用外部文件的正确方式

**关键发现**：Antigravity Agent 需要明确指导才能正确读取外部文件。

### ❌ 错误写法（会导致搜索）

```markdown
加载对应指南：
- `resources/internal-comms/examples/3p-updates.md`
```

### ✅ 正确写法（明确指导工具使用）

```markdown
使用 `view_file` 工具读取示例文件。路径格式为 `{工作区根目录}/.agent/resources/internal-comms/examples/{文件名}`：

| 沟通类型 | 文件名 |
|---------|--------|
| 3P 进度汇报 | `3p-updates.md` |
| 公司通讯 | `company-newsletter.md` |

**操作步骤**：
1. 确定工作区根目录（通常是用户当前项目的根路径）
2. 拼接完整路径：`{根目录}/.agent/resources/internal-comms/examples/3p-updates.md`
3. 调用 view_file 读取文件内容
4. 不要使用搜索，直接读取指定路径
```

### 为什么需要这样写？

1. **Antigravity 要求绝对路径**：所有工具调用都需要绝对路径
2. **没有自动 include 机制**：Rules 不能自动包含其他文件
3. **明确指令优于隐式行为**：告诉 Agent "使用 view_file" 而非让它自己猜测

---

## 模式切换（Mode）

Antigravity 的 `task_boundary` 工具支持三种模式：

| Mode | 用途 | 典型活动 |
|------|------|---------|
| `PLANNING` | 理解需求、设计方案 | 收集信息、创建计划 |
| `EXECUTION` | 实施设计 | 写代码、生成内容 |
| `VERIFICATION` | 验证正确性 | 检查格式、审核结果 |

### 在 Rule 中引导模式使用

```markdown
### 模式切换（Mode）
- **信息收集阶段**：PLANNING 模式，了解团队、时间范围、受众
- **撰写阶段**：EXECUTION 模式，按模板生成内容
- **审核阶段**：VERIFICATION 模式，检查是否符合格式要求
```

---

## 迁移进度

| 原 Claude Skill | 状态 | 类型 | 目标位置 |
|----------------|------|------|---------|
| `internal-comms` | ✅ 完成 | Rules | `.agent/rules/internal-comms.md` |
| `docx` | ✅ 完成 | Rules | `.agent/rules/docx.md` + `.agent/resources/docx/` |
| `scqa-writer` | ⏳ 待迁移 | Workflow | `.agent/workflows/scqa-writer.md` |
| `doc-coauthoring` | ⏳ 待迁移 | Workflow | - |
| `skill-creator` | ✅ 完成 | Workflow | `.agent/workflows/skill-creator.md` + `.agent/resources/skill-creator/` |
| `skill-migrator` | ✅ 完成 | Workflow | `.agent/workflows/skill-migrator.md`（迁移工具本身） |
| 其他... | ⏳ 待迁移 | - | - |

---

## 参考

- [Antigravity 官方文档](https://antigravity.google/docs/get-started)
- [Claude Code Skills 仓库](https://github.com/anthropics/skills)
