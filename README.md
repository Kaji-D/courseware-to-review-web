# courseware-to-review-web

把 PPT / PDF / 讲义 / 教材截图整理成**单文件、可离线、可交互**的复习网页的 Agent Skill。

它不是把 PPT 简单排版一遍，而是把课件重新组织成一条可学习、可自测、可复习的路径：

```text
资料提取 → 内容拆解 → 信息架构 → UI → 动画/交互 → 质检 → 发布
```

## 适合什么任务

- 把一章 / 一门课的 PPT、PDF 整理成复习网页
- 把一个计算题做成“4 步交互讲解 + 考场模板”的专题页
- 给已有的复习网页继续补新章节
- 让知识点逐步讲清楚“为什么这样算、怎么算”
- 把单文件网页发布成公网静态站

## 目录

| 路径 | 说明 |
|---|---|
| `SKILL.md` | 技能入口：工作流、硬约束、QA、常见坑 |
| `assets/page-template.html` | 可直接改的单文件网页骨架，含步骤切换、矩阵、考点模板 |
| `assets/content-model.json` | 示例结构化内容模型 |
| `references/method.md` | 完整方法论：提取、重构、UI、动画、质检、部署、bug 模式 |
| `references/checklist.md` | 一页纸开工/收工检查清单 |
| `references/prompts.md` | 提取、信息架构、UI、动画、专题页、质检、发布提示词 |

## 安装

把整个 `courseware-to-review-web` 目录放到 Agent 的 skills 目录：

```text
macOS / Linux:  ~/.dsh/skills/courseware-to-review-web
Windows:        %USERPROFILE%\.dsh\skills\courseware-to-review-web
```

重新加载技能后，对 Agent 说：

> 用 courseware-to-review-web 把这份课件整理成复习网页

## 核心原则

- **单文件**：CSS/JS/SVG 内联，零外部字体/CDN/图片依赖，离线可用。
- **内容优先**：章 → 节 → 知识点 → 例题/考点，先结构后视觉。
- **考试导向**：每个重点回答“是什么 / 为什么 / 怎么做 / 例子 / 易错 / 考试怎么写”。
- **动画克制**：只解释顺序、因果、定位、对比、状态；不做装饰。
- **移动可用**：320 / 360 / 390 / 768 / 1024 / 1440 无页面级横向溢出。
- **可验证**：公式、维数、行列约定、舍入、图示都要复核。

## 最小工作流

1. 用 `python-pptx` / PDF 工具提取文字、图片和页码。
2. 填一张 `chapter | section | KP | level | why | how | example | pitfall | slide` 表。
3. 复制 `assets/page-template.html`，按章节改内容。
4. 只加能解释内容的交互：步骤、点击、折叠、播放/暂停、静态总览。
5. 按 `references/checklist.md` 跑 320–1440、深色、减少动画、打印、无 JS。
6. 需要发布时，构建静态目录并用任意静态托管。

## 说明

- 模板与参考文档中的 `D:\课件` 等路径只是示例，使用时替换成你的工作区。
- 页面中的图示/演示应作为复习示意；解剖、公式、算法细节以课件和教材为准。
- 本仓库不包含任何课程原始 PPT/PDF，只包含方法、模板和提示词。
