# AI Context for Search & Replace Block

## [2026-03-10] Session Log
### Original Requirements
1. 修复 `diff.html` 中 Monaco Editor Diff 视图左栏（Original Editor）不换行（一直水平滚动）的问题。
2. 修改代码生成的 Prompt，要求 AI 将针对同一个文件的所有 `<<<<<<< SEARCH` ... `>>>>>>> REPLACE` 块输出在同一个 Markdown 语法块中。

### Architectural Details & Notes
- **Diff Editor 换行修复**：Monaco Editor Diff 视图由于在隐藏的父容器（宽度为 0）中初始化，会触发底层的布局与回退策略 Bug，导致左栏的 `wordWrap` 配置永久失效。将 Tab 切换逻辑中隐藏页面的 CSS 从 `display: none` 修改为 `position: absolute; opacity: 0; pointer-events: none; z-index: -1;`，确保 DOM 始终渲染且能获取真实屏幕宽度，从而彻底修复此 Bug。并在配置中增加了 `useInlineViewWhenSpaceIsLimited: false` 作为双重防护。
- **Prompt 升级**：更新了模板字符串，添加了明确规则："CRITICAL: Output ALL search/replace blocks for a single file within the SAME code block. Do NOT close and reopen the ``` fences between changes in the same file." 以避免冗余的 Markdown 围栏。由于底层解析函数是全局正则匹配，此修改完全兼容现有逻辑。

### User Preferences & Corrections
- 页面切换倾向于平滑（通过 `opacity` 动画）而不是生硬的 DOM 销毁/隐藏（`display: none`），同时这兼顾了第三方库的渲染需求。

### Mistakes & Learnings (CRITICAL)
- **Mistake:** 最初尝试使用 `setTimeout` 延迟并调用 `updateOptions` 去强制更新处于隐藏或刚显示的 Monaco Editor 的配置。这不仅没有生效，反而加剧了 Monaco 官方已知的左栏换行失效 Bug（Issue #3346, #4701）。
- **Correction:** 凡是使用像 Monaco 这样高度依赖容器尺寸（DOM 宽度）进行复杂数学计算与重绘的组件，**千万不要在 `display: none` 的父容器内初始化它**。通过 CSS 的绝对定位和透明度隐藏 (`position: absolute; opacity: 0`) 来替代 `display: none`，是彻底绕过该类宽度为0导致初始化失败 Bug 的最干净、最稳妥方案。

### Modified Files
- `diff.html`

## [Session Log] Initial Git Setup
### Original Requirements
用户希望将当前目录初始化为Git仓库，并作为新的公开仓库上传至GitHub，以解决偶尔无tools情况下的不便。

### Architectural Details & Notes
- 初始化了本地仓库，将主分支设置为 `main`。
- 因为原始文件夹包含空格和特殊字符 (`Search & Replace Block`)，自动使用 GitHub CLI (gh) 规范地创建了名为 `Search-Replace-Block` 的公开远程仓库。
- 已自动关联远程并推送。

### User Preferences & Corrections
- 保持公开仓库以方便无环境时查看代码。

### Mistakes & Learnings (CRITICAL)
- PowerShell多行字符串的关闭标记 `'@` 必须独占一行且没有任何前导空格。前一次记录因语法闭合失败报错，已修正。

### Modified Files
- `AI_CONTEXT.md`

## [2026-03-11] Session Log
### Original Requirements
用户请求为新创建的仓库编写 README，并在 GitHub 上更新仓库描述。同时指出在使用 PowerShell 追加文件内容时需要明确指定 UTF8 编码。

### Architectural Details & Notes
- 创建了 README.md，包含功能特性、核心技术、使用说明以及工作原理。
- 使用 GitHub CLI gh repo edit 设定了项目的仓库简介。

### User Preferences & Corrections
- **编码规范：** 强调在 PowerShell 环境下使用 Add-Content 或写入命令时，必须显式指定 -Encoding UTF8 参数，以防止中文字符乱码。

### Mistakes & Learnings (CRITICAL)
- **Mistake:** PowerShell 默认的写入命令在 Windows 环境下可能默认使用 ANSI 等非 UTF-8 编码，这会破坏 Markdown 文件中的中文内容。
- **Correction:** 凡是在 PowerShell 下操作文本文件写入/追加，必须严格加上 -Encoding UTF8 参数。
- **Mistake:** PowerShell 的多行字符串 @" 或 '@ 的闭合标记必须完全顶格（无任何前导空格），否则会被视为普通字符串内容导致解析失败。

### Modified Files
- README.md
- AI_CONTEXT.md
## [2026-03-11] Session Log: Diff Editor Highlight Bug Fix
### Original Requirements
修复 Diff 页面中，差异高亮（修改的部分）有时会消失，但略缩图颜色依然存在的问题。

### Architectural Details & Notes
- 这个现象通常是因为 Monaco Editor 开启了 `wordWrap` 时，在调整窗口大小或在不同标签页之间切换（DOM 容器尺寸变动时），其内部使用的虚拟 DOM 对 `ViewZones`（负责绘制差异高亮的层）的视口计算出现了误差，导致本该显示的高亮被当作屏幕外元素剔除。
- **解决方案**：在每次切换回 Diff 视图时，除了调用 `diffEditor.layout()` 外，使用 `requestAnimationFrame` 确保浏览器已经完全重绘（应用了 `active` class 和正确的尺寸），然后执行 `diffEditor.setModel(diffEditor.getModel())`，通过强制重设 Model 来让 Monaco Editor 完全销毁并重建所有差异计算与 ViewZones。

### Mistakes & Learnings (CRITICAL)
- **Mistake:** 在工具调试过程中，尝试使用未全局安装的 `puppeteer` 时发生了模块未找到的报错。
- **Learning:** 运行一次性抓取脚本如果依赖特定的包，在不确定环境是否有对应模块时，应优先使用已知的安全方式（如 `curl`，或先本地初始化 npm 包），同时向用户清楚解释工具失败与用户问题本身无关。

### Modified Files
- `diff.html`
