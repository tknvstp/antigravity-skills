# Skill Migration Guide

本指南详细描述了从 Claude Code Skill 迁移到 Antigravity Skill 的映射规则和最佳实践。

## 1. 格式映射表 (Anatomy Mapping)

| Claude Code (SKILL.md) | Antigravity 格式 | 说明 |
|------------------------|-----------------|------|
| `skill-name/SKILL.md` (Frontmatter) | `.agent/workflows/{skill}.md` (Frontmatter) | 定义触发条件（Trigger/Description） |
| `skill-name/SKILL.md` (Body) | `.agent/workflows/{skill}.md` (Body) | 主体指令，需适配文件引用方式 |
| `skill-name/scripts/` | `.agent/resources/{skill}/scripts/` | 保持为独立子目录 |
| `skill-name/references/` | `.agent/resources/{skill}/references/` | 保持为独立子目录 |
| `skill-name/assets/` | `.agent/resources/{skill}/assets/` | 保持为独立子目录 |

## 2. 能力增强映射

迁移是增强 Skill 的绝佳时机。利用 Antigravity 独有工具优化原流程：

| 原 Claude 能力 | Antigravity 增强方案 | 优化场景示例 |
|----------------|--------------------|-------------|
| 无网络能力 | `browser_subagent` | 添加"验证文档最新版本"、"搜索最佳实践"步骤 |
| 无视觉能力 | `generate_image` | 添加"生成架构图"、"设计 UI 草图"步骤 |
| 隐式流程 | `task_boundary` | 显式管理 PLANNING -> EXECUTION -> VERIFICATION 状态流转 |
| 文本交互 | `notify_user` | 在关键节点（如部署前）请求结构化确认 |
| 记忆缺失 | Artifacts | 使用 `task.md` 自动追踪复杂任务进度 |

## 3. 工具速查表

| 工具 | 用途 | 必须使用场景 |
|-----|------|-------------|
| `task_boundary` | 状态管理 | 任何复杂的多步骤任务 |
| `view_file` | 读取文件 | 替代原 Skill 中的 "Read x file" 指令 |
| `write_to_file` | 创建文件 | 生成 output 或新代码 |
| `browser_subagent` | 浏览器操作 | 原 Skill 无法实现的在线任务 |
| `notify_user` | 用户交互 | 需要中断流程等待用户确认时 |

## 4. 路径引用规范

在 Antigravity 中，Agent 读取文件需明确路径。

**推荐写法**：
> 详细信息请参考 `references/doc.md`（位于本 Skill 的资源目录 `.agent/resources/{skill-name}/references/`）。请使用 `view_file` 读取。

**绝对路径引用示例**：
```bash
# 假设工作区根目录为 /Users/user/project
view_file /Users/user/project/.agent/resources/my-skill/references/manual.md
```
