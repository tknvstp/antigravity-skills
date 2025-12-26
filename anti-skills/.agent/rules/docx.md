---
trigger: model
description: DOCX 文档创建、编辑和分析。当需要处理 Word 文档（.docx 文件）时激活：创建新文档、修改内容、跟踪修改、添加批注、提取文本等任务。
globs:
---

# DOCX 文档处理

## 概述

用户可能要求创建、编辑或分析 .docx 文件。.docx 文件本质上是一个包含 XML 文件和其他资源的 ZIP 压缩包。根据不同任务，有不同的工具和工作流程可用。

## 工作流决策树

### 读取/分析内容
使用下方「文本提取」或「原始 XML 访问」

### 创建新文档
使用「创建新 Word 文档」工作流

### 编辑现有文档
- **自己的文档 + 简单修改** → 使用「基础 OOXML 编辑」
- **别人的文档** → 使用「红线审阅」工作流（推荐默认）
- **法律/学术/商业/政府文档** → 使用「红线审阅」工作流（必需）

---

## 读取和分析内容

### 文本提取

使用 pandoc 将文档转换为 markdown：

```bash
# 转换文档为 markdown，保留跟踪修改
pandoc --track-changes=all path-to-file.docx -o output.md
# 选项: --track-changes=accept/reject/all
```

### 原始 XML 访问

需要原始 XML 访问的场景：批注、复杂格式、文档结构、嵌入媒体、元数据。

**解包文件**：
```bash
python {工作区根目录}/.agent/resources/docx/ooxml/scripts/unpack.py <office_file> <output_directory>
```

**关键文件结构**：
- `word/document.xml` - 主要文档内容
- `word/comments.xml` - document.xml 中引用的批注
- `word/media/` - 嵌入的图像和媒体文件
- 跟踪修改使用 `<w:ins>`（插入）和 `<w:del>`（删除）标签

---

## 创建新 Word 文档

使用 **docx-js**（JavaScript/TypeScript 库）从零创建 Word 文档。

### 工作流

1. **必须 - 阅读完整文件**：使用 `view_file` 工具读取 `{工作区根目录}/.agent/resources/docx/docx-js.md`（约 350 行），获取详细语法和最佳实践
2. 使用 Document、Paragraph、TextRun 组件创建 JavaScript/TypeScript 文件
3. 使用 `Packer.toBuffer()` 导出为 .docx

---

## 编辑现有 Word 文档

使用 **Document library**（Python OOXML 操作库）编辑现有文档。

### 工作流

1. **必须 - 阅读完整文件**：使用 `view_file` 工具读取 `{工作区根目录}/.agent/resources/docx/ooxml.md`（约 610 行），获取 Document library API 和 XML 模式
2. 解包文档：`python {工作区根目录}/.agent/resources/docx/ooxml/scripts/unpack.py <office_file> <output_directory>`
3. 创建并运行 Python 脚本（参见 ooxml.md 中的"Document Library"部分）
4. 打包最终文档：`python {工作区根目录}/.agent/resources/docx/ooxml/scripts/pack.py <input_directory> <office_file>`

---

## 红线审阅工作流

此工作流允许在 OOXML 中实现全面的跟踪修改。

**批处理策略**：将相关修改分组为 3-10 个修改一批。

**原则：最小化、精确编辑**  
实现跟踪修改时，只标记实际更改的文本。

### 跟踪修改工作流

1. **获取 markdown 表示**：
   ```bash
   pandoc --track-changes=all path-to-file.docx -o current.md
   ```

2. **识别并分组修改**：审查文档，识别所有需要的修改，按逻辑分批组织

3. **阅读文档并解包**：
   - **必须**：使用 `view_file` 读取 `{工作区根目录}/.agent/resources/docx/ooxml.md`
   - **解包文档**：`python {工作区根目录}/.agent/resources/docx/ooxml/scripts/unpack.py <file.docx> <dir>`
   - **记录建议的 RSID**：解包脚本会建议一个 RSID 用于跟踪修改

4. **分批实现修改**：使用 Document library 的 `get_node` 查找节点，实现修改，然后 `doc.save()`

5. **打包文档**：
   ```bash
   python {工作区根目录}/.agent/resources/docx/ooxml/scripts/pack.py unpacked reviewed-document.docx
   ```

6. **最终验证**：
   ```bash
   pandoc --track-changes=all reviewed-document.docx -o verification.md
   grep "原始短语" verification.md  # 不应找到
   grep "替换短语" verification.md  # 应该找到
   ```

---

## 转换文档为图像

1. **DOCX 转 PDF**：
   ```bash
   soffice --headless --convert-to pdf document.docx
   ```

2. **PDF 转 JPEG**：
   ```bash
   pdftoppm -jpeg -r 150 document.pdf page
   # 创建 page-1.jpg, page-2.jpg 等
   ```

---

## 依赖项

如未安装，请安装以下依赖：

- **pandoc**：`brew install pandoc`（文本提取）
- **docx**：`npm install -g docx`（创建新文档）
- **LibreOffice**：`brew install --cask libreoffice`（PDF 转换）
- **Poppler**：`brew install poppler`（pdftoppm）
- **defusedxml**：`pip install defusedxml`（安全 XML 解析）
- **lxml**：`pip install lxml`（验证脚本需要）

---

## 关键词

Word 文档、docx、创建文档、编辑文档、跟踪修改、红线审阅、批注、文档转换
