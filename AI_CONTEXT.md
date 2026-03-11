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
