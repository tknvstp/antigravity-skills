---
description: 将 Claude Code Skills 迁移到 Antigravity 格式。当用户想迁移/转换一个 Claude Code skill 到 Antigravity 的 rule 或 workflow 时使用。
---

# Skill Migrator

将 Claude Code 的 `.skill` / `SKILL.md` 格式转换为 Antigravity 的 `.agent/` 目录结构。

---

## 格式映射表

| Claude Code (SKILL.md) | Antigravity 格式 | 说明 |
|------------------------|-----------------|------|
| YAML `name` | 文件名 | 不需要重复定义 |
| YAML `description` | YAML `description` | 必须包含「何时使用」信息 |
| YAML `license` | 不迁移 | Antigravity 不需要 |
| Body 中「When to use」 | 合并到 `description` | 触发依据必须在 frontmatter |
| Body 工作流程 | 保留为主体内容 | 简化并本地化 |
| 相对路径引用 | 绝对路径 + `view_file` 指导 | **关键改造点** |
| `scripts/` | `.agent/resources/{skill}/scripts/` | 保持结构 |
| `references/` | `.agent/resources/{skill}/` | 扁平化或保留 |
| `assets/` | `.agent/resources/{skill}/assets/` | 保持结构 |

---

## Antigravity 核心工具参考

迁移时应充分利用 Antigravity 的工具能力，不仅是格式转换，更是能力增强。

### 工具速查表

| 工具 | 用途 | 迁移优化场景 |
|-----|------|-------------|
| `task_boundary` | 任务模式切换 (PLANNING/EXECUTION/VERIFICATION) | 为复杂 workflow 添加模式切换指导 |
| `notify_user` | 任务模式下与用户通信 | 需要用户确认的关键检查点 |
| `view_file` | 读取文件内容（必须用绝对路径） | **替代所有相对路径引用** |
| `write_to_file` | 创建新文件 | 生成输出文件 |
| `replace_file_content` | 编辑现有文件 | 文档修改类任务 |
| `run_command` | 执行终端命令 | 脚本执行、依赖安装 |
| `browser_subagent` | 委托浏览器任务给子代理 | **新增：网页研究、在线验证** |
| `find_by_name` | 按名称模式搜索文件 | 定位资源文件 |
| `grep_search` | 在文件中搜索文本模式 | 代码/文档内容定位 |
| `generate_image` | 生成或编辑图像 | UI 设计类 skill |

### 能力增强建议

将 Claude Code skill 迁移到 Antigravity 时，考虑以下能力增强：

#### 1. 模式切换（task_boundary）

在 workflow 中明确指导何时切换模式：

```markdown
### 模式切换
- **信息收集阶段**：PLANNING 模式，理解需求、收集素材
- **实施阶段**：EXECUTION 模式，执行具体操作
- **验证阶段**：VERIFICATION 模式，检查结果、确认质量
```

#### 2. 浏览器子代理（browser_subagent）

原始 Claude Code 无此能力。迁移时可增强：

```markdown
### 在线研究（可选）
如需最新信息或验证，使用 `browser_subagent` 工具：
- 查询 API 文档最新版本
- 验证外部链接有效性
- 收集行业最佳实践
```

#### 3. 结构化任务追踪（task.md artifact）

复杂 workflow 可利用 artifact 系统：

```markdown
### 任务追踪
创建 artifact `task.md` 追踪进度：
- [ ] 步骤 1: ...
- [ ] 步骤 2: ...
```

#### 4. 用户交互检查点（notify_user）

需要用户决策时明确指出：

```markdown
### 用户确认点
使用 `notify_user` 工具在以下节点请求确认：
- 方案设计完成后
- 关键变更执行前
```

---

## 迁移决策树

### 判断目标类型

1. **Workflow**（用户显式调用）
   - 用户通过 `/命令` 调用
   - 多步骤流程导向任务
   - 例：skill-creator、scqa-writer

2. **Rule (trigger: model)**（自动触发）
   - 模型根据描述自动激活
   - 工具类、格式处理类
   - 例：docx、pdf、xlsx

3. **Rule (trigger: always)**（始终生效）
   - 全局约束、编码规范
   - 例：brand-guidelines

4. **Rule (trigger: manual)**（按需引用）
   - 用户通过 `@文件名` 引用
   - 按需参考的指南

---

## 迁移步骤

### 步骤 1：分析原始 Skill

使用 `view_file` 读取原始 `SKILL.md`，识别：

1. **功能类型** → 决定 workflow 还是 rule
2. **触发条件** → 合并到 description
3. **外部资源** → 需迁移到 `.agent/resources/`

**提问清单**（如信息不明确）：
- 这个技能应该自动触发还是用户显式调用？
- 有哪些资源文件需要保留？

### 步骤 2：创建目录结构

```bash
# Workflow（无资源）
mkdir -p .agent/workflows

# Workflow/Rule（有资源）
mkdir -p .agent/workflows
mkdir -p .agent/resources/{skill-name}
# 可选子目录
mkdir -p .agent/resources/{skill-name}/scripts
mkdir -p .agent/resources/{skill-name}/assets
```

### 步骤 3：迁移资源文件

// turbo
```bash
# 复制所有资源
cp -r {原路径}/scripts .agent/resources/{skill-name}/
cp -r {原路径}/references/* .agent/resources/{skill-name}/
cp -r {原路径}/assets .agent/resources/{skill-name}/
```

处理要点：
- 更新脚本中的相对路径引用为绝对路径
- 检查 Python/Bash 脚本的路径调用

### 步骤 4：编写主文件

#### Workflow 格式

```yaml
---
description: 简短描述功能。包含触发场景。
---

# 标题

## 概述
...

## 工作流程
...
```

#### Rule 格式

```yaml
---
trigger: model
description: 详细描述功能和所有触发场景。当用户需要...时激活。
globs:
---

# 标题

## 适用场景
...

## 工作流程
...
```

### 步骤 5：改写文件引用（关键）

**原始写法（Claude Code）**：
```markdown
Read [`docx-js.md`](docx-js.md) for details.
```

**Antigravity 正确写法**：
```markdown
使用 `view_file` 工具读取参考文档。路径格式：
`{工作区根目录}/.agent/resources/{skill-name}/{文件名}`

| 内容 | 文件名 |
|-----|--------|
| DOCX-JS 语法指南 | `docx-js.md` |

**操作步骤**：
1. 确定工作区根目录（用户项目的根路径）
2. 拼接完整绝对路径
3. 调用 view_file 读取文件内容
4. 不要使用搜索，直接读取指定路径
```

### 步骤 6：验证

#### 语法检查
- [ ] YAML frontmatter 格式正确
- [ ] `description` 包含完整触发条件
- [ ] 主体内容 <500 行

#### 资源检查
- [ ] 所有资源文件已复制到正确位置
- [ ] 路径引用使用 `{工作区根目录}` 占位符

#### 功能测试
1. 将 `.agent/` 复制到测试项目
2. 在 Antigravity 新对话中输入触发关键词
3. 验证 skill 正确激活并执行

---

## 快速参考：三种 Trigger 对比

| Trigger | 激活方式 | frontmatter 格式 |
|---------|---------|-----------------|
| model | 模型判断 description 匹配 | `trigger: model` |
| always | 始终生效 | `trigger: always` |
| manual | 用户 `@文件名` | `trigger: manual` |

---

## 迁移示例

### 简单 Rule 迁移（如 internal-comms）

**原始**：
- 无脚本/资产
- 只有示例文件

**迁移结果**：
```
.agent/
├── rules/internal-comms.md
└── resources/internal-comms/examples/
    ├── 3p-updates.md
    └── ...
```

### 复杂 Rule 迁移（如 docx）

**原始**：
- 有 scripts/（ooxml 操作脚本）
- 有 references/（详细文档）

**迁移结果**：
```
.agent/
├── rules/docx.md
└── resources/docx/
    ├── docx-js.md
    ├── ooxml.md
    └── ooxml/scripts/
        ├── pack.py
        └── unpack.py
```

### Workflow 迁移（如 skill-creator）

**原始**：
- 多步骤创建流程
- 需用户显式调用

**迁移结果**：
```
.agent/
├── workflows/skill-creator.md
└── resources/skill-creator/
    ├── workflows.md
    └── output-patterns.md
```

---

## 关键词

skill 迁移、Claude Code、Antigravity、rule、workflow、格式转换
