# agent.md — Powersim 开发说明

## 项目简介
单文件 Web 应用「马力机模拟器」。输入或生成发动机功率数据点，绘制马力 PS 与扭矩 N·m 双纵轴平滑曲线；支持多工作文件、拖拽/增删数据点、峰值线与长按读数、提示词生成、日间/夜间主题、可拖拽图表高度。

## 目录结构
- `index.html` — 唯一源文件（内联 CSS/JS；图标以 base64 PNG 内联为 CSS 遮罩）
- `agent.md` — 本文件
- `.nojekyll` — 空文件，关闭 GitHub Pages 的 Jekyll 处理
- `CNAME` — 自定义域名 `powersim.caracter.cn`（GitHub Pages 读取，勿删）
- `.git/` — 远程 `origin` = https://github.com/Productionl/powersim.git，分支 `main`

## 运行与调试
- 直接用浏览器打开 `index.html`（`file://` 可用），无需构建、无第三方依赖。
- 数据存于 localStorage，测试持久化时不要用无痕/隐私模式。
- 线上地址：https://powersim.caracter.cn/（GitHub Pages 默认地址 https://productionl.github.io/powersim/ 会 301 到自定义域名）
- 自定义域名：`CNAME` 文件内容一行 `powersim.caracter.cn`；DNS 需配置 `powersim` 的 CNAME → `productionl.github.io`（子域不要加 A 记录）。Settings → Pages 中填同一个域名并勾选 Enforce HTTPS。

## 部署（GitHub Pages）
- 站点源为 `main` 分支根目录（仓库公开）；push 到 `main` 后 1–2 分钟自动发布。
- 首次启用：仓库 Settings → Pages → Source 选 “Deploy from a branch”，Branch 选 `main` / `(root)`，Save。
- 页面自包含、无外部资源与绝对路径，因此在 `/<repo>/` 子路径下可直接运行。
- 根目录 `.nojekyll` 防止 Jekyll 误解析内联脚本中的模板字符串；不要删除。
- 发布后浏览器打开的是新 origin，localStorage 与本地 `file://` 数据不互通（键名相同、互不影响）。

## 关键常量与公式
- `PS_W = 735.49875`（1 公制马力）
- `K = PS_W * 60 / (2π) ≈ 7023.4957`
- 扭矩 `T = K * PS / rpm`；功率 `PS = T * rpm / K`

## localStorage
- `power-torque-v2`：`{ activeId, files:[{id,name,points:[{rpm,ps}],psMin,psMax,rpmMin,rpmMax}], ui:{snap,snapStep,chartH} }`
- `power-torque-points-v1`：旧版单文件数据，仅用于迁移
- `powersim-theme`：`"light" | "dark"`

## 代码约定与易错点
- 保持单文件、零依赖、可直接 `file://` 打开。新增图标请从 material-design-icons 仓库取 outlined PNG，base64 内联为 CSS `mask-image`（参考现有 `.mi-*`）。
- `render()` 每次用 `chart.innerHTML = ...` 重建内容。任何需要长期存在的 DOM 都不能放进 `#chart`；例如图表高度手柄 `#chartResizer` 放在其外层 `.chart-wrap`。
- 绘图区底色是 `render()` 内联的 `<rect fill="#ffffff">`；夜间由 `html[data-theme="dark"] .plot-bg` 覆盖（CSS 规则优先于 SVG 呈现属性）。
- `buildPrompt()` 返回模板字符串，含 `${cfg.join("\n")}`、`${powerWord}`、`${powerRule}`、`${torqueCheckFormula}`、`${wheel ? ... : ...}` 插值。修改提示词时只改正文，勿破坏插值、函数签名或 `copyPrompt()`。
- 主题：浅色为基础规则，深色为文件末尾 `html[data-theme="dark"] ...` 覆盖块。新增可见元素时需同时补夜间配色。
- 交互基于 Pointer Events。图表高度自定义手柄须 `touch-action:none`（iOS Safari 不支持普通元素的 CSS `resize`）。
- UI 文案使用中文；数值统一用 `fmt()` 格式化。

## 修改前自检
- 提取 `<script>` 块做 `node --check`。
- 检查 CSS 花括号配对、`getElementById` 引用的 id 均存在。
- 手动回归：桌面与窄屏、日/夜主题、多文件切换、拖点/双击删点/点击曲线加点/长按纵线、图表高度拖拽、提示词复制与追加。

## Git
- 仅当用户明确要求时 commit / push；提交信息用简洁英文（与现有历史一致）。
