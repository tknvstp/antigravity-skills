# Antigravity 平台参考

本文档提供 Antigravity 平台的核心能力参考，帮助创建和迁移 skill/workflow 时充分利用平台特性。

> 参考来源：[tfriedel/antigravity_prompts](https://github.com/tfriedel/antigravity_prompts)

---

## 核心工具清单

Antigravity 提供 25+ 工具，以下是创建/迁移 skill 时最常用的工具：

### 文件操作

| 工具 | 用途 | 适用场景 |
|-----|------|---------|
| `view_file` | 读取文件内容 | **必须使用绝对路径**，资源文件读取 |
| `view_file_outline` | 查看文件大纲 | 探索代码结构的首选 |
| `write_to_file` | 创建新文件 | 生成输出文件、模板 |
| `replace_file_content` | 编辑单个区块 | 文档局部修改 |
| `multi_replace_file_content` | 编辑多个区块 | 多处非连续修改 |

### 搜索定位

| 工具 | 用途 | 适用场景 |
|-----|------|---------|
| `find_by_name` | 按文件名搜索 | 定位资源文件、脚本 |
| `grep_search` | 在文件中搜索模式 | 代码/文档内容定位 |
| `view_code_item` | 查看代码节点 | 函数/类定义查看 |

### 命令执行

| 工具 | 用途 | 适用场景 |
|-----|------|---------|
| `run_command` | 执行终端命令 | 脚本执行、依赖安装、构建验证 |
| `command_status` | 检查命令状态 | 后台命令监控 |
| `send_command_input` | 发送命令输入 | 交互式命令、REPL |

### 浏览器与研究

| 工具 | 用途 | 适用场景 |
|-----|------|---------|
| `browser_subagent` | 委托浏览器任务 | **Claude Code 无此能力**：网页研究、在线验证、UI 测试 |
| `read_url_content` | 读取 URL 内容 | 静态文档抓取（无 JS 执行） |
| `search_web` | 网页搜索 | 获取最新信息 |

### 用户交互与任务管理

| 工具 | 用途 | 适用场景 |
|-----|------|---------|
| `task_boundary` | 任务模式切换 | 复杂任务的模式管理 |
| `notify_user` | 与用户通信 | 任务模式下的唯一通信方式 |
| `generate_image` | 生成/编辑图像 | UI 设计、资产生成 |

---

## 模式系统（Mode System）

Antigravity 使用三种模式管理复杂任务：

### PLANNING 模式

- **职责**：深度研究、理解需求、制定计划
- **产出**：`implementation_plan.md`
- **要点**：
  - 全面系统地研究，避免假设
  - 解决所有不确定性
  - **必须研究验证策略**：查找现有测试、构建命令

### EXECUTION 模式

- **职责**：执行实施计划
- **要点**：
  - 独立执行计划
  - 遇到未预料情况时**返回 PLANNING**
  - 避免盲目推进导致代码混乱

### VERIFICATION 模式

- **职责**：验证工作正确性
- **产出**：`walkthrough.md`
- **要点**：
  - 全面系统，不走捷径
  - 提供用户可理解的证明
  - 使用浏览器工具测试 Web 应用

---

## Artifact 系统

Antigravity 使用三个核心 artifact 管理任务：

### task.md - 任务追踪

**路径**：`{ArtifactDir}/task.md`

**格式**：
```markdown
- [ ] 未完成任务 <!-- id: 0 -->
- [/] 进行中任务 <!-- id: 1 -->
- [x] 已完成任务 <!-- id: 2 -->
```

**规则**：
- 用户请求变更时**立即更新**
- 任务完成时**立即标记**
- 可添加子任务缩进列表
- ID 必须保持稳定

### implementation_plan.md - 实施计划

**路径**：`{ArtifactDir}/implementation_plan.md`

**结构**：
```markdown
# [目标描述]

## User Review Required
[需要用户审核的关键决策]

## Proposed Changes
### [组件名称]
#### [MODIFY] / [NEW] / [DELETE] [文件名](file:///path)
[具体变更说明]

## Verification Plan
### Automated Tests
### Manual Verification
```

### walkthrough.md - 完成摘要

**路径**：`{ArtifactDir}/walkthrough.md`

**用途**：
- 文档化所完成的工作
- 包含验证结果
- 嵌入截图/录屏演示

---

## Workflow 机制

### 目录结构

```
.agent/
├── workflows/          # 用户显式调用 (/命令名)
│   └── skill-name.md
├── rules/              # 自动/手动触发
│   └── skill-name.md
└── resources/          # 辅助资源
    └── skill-name/
        ├── scripts/
        └── ...
```

### Frontmatter 格式

**Workflow**：
```yaml
---
description: 功能描述。何时使用。
---
```

**Rule**：
```yaml
---
trigger: model  # 或 always, manual
description: 功能描述。激活条件。
globs:          # 可选文件过滤
---
```

### Turbo 注解

- `// turbo`：标记单个步骤可自动执行
- `// turbo-all`：标记整个 workflow 可自动执行

```markdown
2. 创建目录
// turbo
3. 复制文件
```

---

## 能力增强映射

从 Claude Code 迁移时，考虑以下增强：

| Claude Code 能力 | Antigravity 增强 |
|-----------------|-----------------|
| 无浏览器能力 | `browser_subagent` 网页研究、UI 测试 |
| 无图像生成 | `generate_image` 资产生成 |
| 手动任务追踪 | `task.md` artifact 自动追踪 |
| 手动模式切换心智 | `task_boundary` 显式模式管理 |
| 用户交互混合在回复中 | `notify_user` 结构化用户通信 |

---

## 关键规则

1. **绝对路径**：所有文件操作必须使用绝对路径
2. **资源路径格式**：`{工作区根目录}/.agent/resources/{skill-name}/{文件名}`
3. **任务持续更新**：task.md 必须实时反映进度
4. **模式明确切换**：复杂任务必须正确使用 task_boundary
5. **验证优先研究**：PLANNING 阶段必须研究验证策略
