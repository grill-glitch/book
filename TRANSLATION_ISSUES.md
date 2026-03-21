# 翻译与构建潜在问题

## 已完成的翻译
- `src/ssl-and-tls.rst` → 已翻译为简体中文

## 发现的问题

### 1. Sphinx 语言配置
**问题**：`conf.py` 未设置 `language` 参数，默认使用英文。
**影响**：直接构建会使用英文主题和静态文本（如"Figure 1"、"Chapter 1"等）。
**解决方案**：
- 临时：`make latexpdf O="-D language=zh_CN"`
- 永久：修改 `conf.py`，添加 `language = "zh_CN"`

### 2. 中文字体支持（PDF/LaTeX）
**问题**：LaTeX 渲染需要中文字体配置。当前 `latex/fontpkg` 仅设置西文字体。
**影响**：PDF 生成时中文会显示为方框或编译错误。
**解决方案**：在 `src/latex/preamble.tex` 中添加 XeLaTeX 中文字体配置，如：
```
\usepackage{xeCJK}
\setCJKmainfont{Noto Serif CJK SC}
\setCJKsansfont{Noto Sans CJK SC}
\setCJKmonofont{Noto Sans Mono CJK SC}
```

### 3. gettext 翻译模板
**问题**：未生成中文翻译模板（.pot 文件），也未创建 `locale/zh_CN/LC_MESSAGES/`。
**影响**：如果其他用户想贡献翻译或使用不同语言，无法通过 gettext 工作流。
**解决方案**：运行 `make gettext` 生成模板文件。

### 4. 插图文本
**问题**：部分插图（.svg 中的 .tex 文件）包含英文标签。
**影响**：插图内的文字未翻译。
**解决方案**：修改 `src/Illustrations/**/*.tex` 文件中的文本为中文。

### 5. 术语一致性
**问题**：翻译术语需要统一（如"handshake"译为"握手"而非"联络"）。
**建议**：创建 `glossary.rst` 的翻译对照表或术语词典。

### 6. 发布流程
**问题**：`Makefile` 中的 `deploy` 目标使用 rsync 到特定服务器，当前配置可能不适用于你的环境。
**影响**：HTML 无法自动部署到网站。
**解决方案**：修改 `deploy` 目标或创建新的发布脚本。

### 7. GitHub Release 自动化
**问题**：当前 Makefile 没有构建后自动创建 Release 并上传附件的目标。
**解决方案**：添加自定义发布脚本，使用 `gh release create` 上传构建产物。

---

## 建议的下一步

1. **等待指示**：先确认上述问题的处理方式。
2. **试点翻译**：仅修改 `src/ssl-and-tls.rst` 并尝试构建中文 PDF 以验证中文字体配置。
3. **批量翻译**：如果单文件验证成功，继续翻译其他章节。

请指示如何处理这些问题？🐱
