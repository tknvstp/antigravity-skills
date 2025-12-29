# Antigravity Skills

为 [Antigravity](https://developers.google.com/idx/guides/get-started-antigravity) 平台设计的 Skills 和 Workflows，帮助扩展 AI Agent 能力。

本项目致力于实现两个核心目标：
1. **Skill 迁移**：将 Claude Code Skills 转换为符合 Antigravity 特性（如 task_boundary, browser_subagent）的 rule 或 workflow。
2. **Skill 创建**：支持基于自然语言创建符合规范的 rule 或 workflow，快速扩展 Agent 能力。

> **Antigravity** 是 Google 推出的 AI 辅助编程工具，基于 VS Code 构建，集成了强大的 AI Agent 功能。

---

## 📁 目录结构

```
.agent/
├── workflows/                      # 用户显式调用的工作流 (/命令名)
│   ├── skill-creator.md            # 创建新 skill 的指南
│   └── skill-migrator.md           # 从 Claude Code 迁移 skill 的指南
└── resources/                      # 辅助资源
    └── skill-creator/
        └── references/
            ├── antigravity-reference.md  # Antigravity 平台核心能力参考
            ├── workflows.md              # 工作流模式示例
            └── output-patterns.md        # 输出格式模式示例
```

---

## 🚀 快速开始

### 使用 Workflow

在 Antigravity 中直接使用斜线命令：

- `/skill-creator` - 创建新的 skill 或 workflow
- `/skill-migrator` - 将 Claude Code skill 迁移到 Antigravity 格式

### 安装

将 `.agent/` 目录复制到你的项目根目录：

```bash
cp -r .agent/ /your/project/root/
```

---

## 📖 Workflows 说明

### skill-creator

创建新的 skill 或 workflow，扩展 Agent 能力。

**适用场景**：
- 创建特定领域的专业知识包
- 封装常用工作流程
- 集成工具和脚本

**核心特性**：
- 渐进式披露设计（元数据 → 主体 → 资源）
- Antigravity 工具集成指导
- 模式切换建议（PLANNING / EXECUTION / VERIFICATION）

### skill-migrator

将已有的 Claude Code Skills 迁移到 Antigravity 格式。

**适用场景**：
- 迁移 Claude Code 的 `.skill` 或 `SKILL.md` 文件
- 利用 Antigravity 独有能力增强
- 保持 skill 功能的同时优化结构

**核心特性**：
- 格式映射表（Claude Code → Antigravity）
- 能力增强映射（新增 browser_subagent, generate_image 等）
- 迁移决策树和验证清单

---

## 🛠️ Antigravity 核心能力

这些 workflow 充分利用了 Antigravity 的独有能力：

| 能力 | 说明 |
|-----|------|
| `task_boundary` | 任务模式管理（PLANNING/EXECUTION/VERIFICATION） |
| `browser_subagent` | 浏览器子代理，支持网页研究和 UI 测试 |
| `notify_user` | 结构化用户通信，支持阻塞式确认 |
| `generate_image` | 图像生成和编辑 |
| Artifact System | task.md / implementation_plan.md / walkthrough.md |

详细说明请参考 [antigravity-reference.md](.agent/resources/skill-creator/references/antigravity-reference.md)。

---

## 📚 参考来源

本项目参考了以下优秀资源：

### Claude Code Skills

- **来源**：[Anthropic Courses - Skills](https://github.com/anthropics/courses/tree/master/prompt_engineering_interactive_tutorial)
- **内容**：Skill 设计原则、结构规范、最佳实践
- **许可**：遵循 Anthropic 课程许可

### Antigravity Prompts

- **来源**：[tfriedel/antigravity_prompts](https://github.com/tfriedel/antigravity_prompts)
- **内容**：Antigravity 系统 prompt 逆向工程分析
- **用途**：理解 Antigravity 平台机制和工具能力

---

## 📄 许可

本项目仅供学习和参考使用。

- Skill 设计理念来自 Anthropic 官方课程
- Antigravity 平台参考来自社区逆向工程项目
- 请遵循各来源项目的许可协议

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

如果你有：
- 新的 skill 想法
- 优化建议
- Bug 报告

请随时联系或提交贡献。
