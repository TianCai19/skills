请先创建一个专门的「HTML 笔记生成子 agent」来完成本任务；主会话只负责下发需求、接收结果、做最终验收与打开预览，不要把生成、修改、验证都集中在同一个主终端流程里完成。

子 agent 任务说明：

1. 使用 HTML5 语义标签 + CSS Custom Properties + BEM + CSS Grid/Flexbox + Prism.js + KaTeX auto-render + 内联 SVG animateMotion + IntersectionObserver 懒渲染 + mark.js 全文高亮 + localStorage 批注持久化，生成一个单文件在线笔记页面。
2. 页面必须具备：自动 TOC、折叠推导块、数学公式展示、代码高亮、全文搜索/高亮、批注持久化、响应式阅读体验。
3. 如果当前目录存在 `experiences.md`，子 agent 必须先阅读并吸收其中可复用建议；注意文件名是 `experiences.md`。
4. 子 agent 需要独立完成实现与基础自检，并在结束时返回：
   - 创建或修改的文件路径；
   - 已完成的核心功能清单；
   - 自检方式与结果；
   - 需要主会话最终确认或打开预览的事项。

子 agent 完成后，主会话再打开生成的页面进行预览，并汇总结果。

请把下面的主题交给该子 agent 完成：
