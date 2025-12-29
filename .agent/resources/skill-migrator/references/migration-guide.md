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

## 5. 资源类型使用规范

### Scripts 执行规范

| 脚本类型 | 执行命令格式 |
|---------|-------------|
| Python | `python {path}/scripts/xxx.py [args]` |
| Bash | `bash {path}/scripts/xxx.sh [args]` |
| Node.js | `node {path}/scripts/xxx.js [args]` |

**设计要求**：
- 通过命令行参数接收输入
- 输出清晰的状态/错误信息
- 返回有意义的退出码

### References 引用规范

| 场景 | Workflow 中的写法 |
|-----|-------------------|
| 必须阅读 | `**必须**使用 \`view_file\` 阅读：{path}` |
| 可选阅读 | `如需深入了解，可选阅读：{path}` |
| 大文件检索 | `使用 \`grep_search\` 在 {path} 中搜索 "关键词"` |

**最佳实践**：
- 避免将信息同时放在 Workflow Body 和 References 中
- 对于 >10k words 的文件，提供 grep 搜索模式

### Assets 使用规范

| 场景 | 操作命令 |
|-----|---------|
| 复制到输出 | `cp {path}/assets/file ./output/` |
| 批量复制 | `cp -r {path}/assets/* ./output/` |
| 模板替换 | 先复制，再用 `replace_file_content` 修改占位符 |

**常见 Assets 类型**：
- 模板文件：HTML/Markdown 骨架
- 品牌资产：Logo、图标、字体
- 配置骨架：JSON/YAML 配置模板
- 代码脚手架：项目初始化模板目录

