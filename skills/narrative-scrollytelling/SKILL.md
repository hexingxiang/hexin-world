---
name: narrative-scrollytelling
description: 制作「单文件电影级叙事主页」：CDN 依赖 + 三幕/六幕滚动编舞 + three.js 体素背景 + Web Audio 全合成 BGM + 游戏 HUD 导航 + 跨章伏笔交互，零构建单 HTML 直接挂 GitHub Pages。当用户说"做一个 hexin-world 那样的主页""电影级滚动叙事页""个人主页 scrollytelling""带开门动画的主页"时使用。
---

# 单文件电影级叙事主页 · 制作手册

范式来源：本仓库 `index.html`（约 3900 行单文件，Minecraft 皮肤）。在线成品：https://hexingxiang.github.io/hexin-world/

一句话方法论：**把主题切成 5-6 幕剧本，每一幕 = 一个 `<section>` + 一句金句 + 一个可交互 widget + 一种 BGM/色彩情绪；全部塞进一个零构建的单 HTML，双击就能看。** 皮肤可换（像素风/赛博/水墨），结构模式直接复用。

## 0. 硬约束（每页不变）

- 单文件 HTML，无构建、无本地资源引用：依赖全部 CDN
  - Tailwind Play CDN `<script src="https://cdn.tailwindcss.com/3.4.17">`
  - three.js `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`
  - Google Fonts（像素字 Press Start 2P + 衬线 Cormorant Garamond / Noto Serif SC + 等宽 JetBrains Mono）
- 体积控制在 ~200KB 量级；任何 CDN 挂掉都要有系统字体/合成音兜底，页面不能白屏
- 所有 `<audio>`/`<canvas>`/`src` 只允许绝对 URL 或程序生成，禁止相对路径文件

## 1. 骨架：叙事结构

- `Prologue` 开场屏 + N 个 `<section class="chapter-section" data-chapter="I · 标题">`，每幕一个人生阶段/主题段落
- 幕内编舞用 `.story-beat`：初始 `opacity .04 + blur(8px) + translateY(44px) scale(.975)`，进入视口加 `.in-view`，`cubic-bezier(0.16,1,0.3,1)` 1.15s 过渡——"话剧逐句亮相"感
- 双层 IntersectionObserver：
  - `.story-beat` 级 threshold 0.14（只加类，不移除，滚动不回退）
  - `.chapter-section` 级 threshold 0.32，统一驱动：顶栏徽章、导航胶囊/hotbar 高亮、BGM 情绪、背景世界变形
- 章头放巨型水印字：`-webkit-text-stroke: 1px rgba(255,255,255,.055)` + `color: transparent`
- 金句排版：中文大字 blockquote + 英文斜体 editorial 系字体上下呼应

## 2. 开场门（Gate Overlay）

全屏 `fixed inset-0 z-[60]` overlay 先演一段"仪式"（本例：2010 破台式机点 POWER → BIOS 文本逐行 → Loading terrain 进度条 → `scale(1.38)` 淡出进正片）。三条铁律：

1. 必须给「直接进入世界」跳过按钮（右上角小字）
2. 必须给「重温开机」入口（Prologue 里放按钮）
3. overlay 淡出用 `opacity + transform + pointer-events:none` 三件套，别 display:none 硬切

## 3. 背景：three.js 体素世界

- `<canvas>` `fixed inset-0 z-0 pointer-events-none` 全程铺底，正文内容 `relative z-10` 浮在上层
- 程序化像素纹理：offscreen canvas 手画 16×16 网格（边框一圈 border 色 → 左上内圈高光 bevel → 右下内圈暗部 → 中间放矿物/斑纹）→ `THREE.CanvasTexture` + `magFilter/minFilter = NearestFilter` 锁死像素感
- `FogExp2` 出氛围；`worldGroup` 挂体素浮块，随 `activeChapterId` 切换世界形态（色调/密度/相机）
- 性能：`setPixelRatio(Math.min(dpr, 2))`，不开 shadowMap，`requestAnimationFrame` 单循环

## 4. Web Audio 全合成 BGM（零版权音频文件）

- 主链路：合成音色 → 音色 lowpass → `StereoPanner`（按音高定声像）→ masterFilter（全局 low-pass，情绪阀门）→ DynamicsCompressor → 自制卷积混响（生成 impulse buffer：白噪声 × 衰减包络）+ 立体声延迟并联
- "毛毡钢琴" = osc + 快 attack(0.004s) 慢 release(2.8s) 增益包络；暖 Pad = 双 osc detune ±6 cents + lowpass swell
- 音序器：`TRACKS` 数组存每轨 steps（主旋律/和弦/低音），`setTimeout` 链逐 step 调度，加 rubato 随机错位（`idx*0.14 + rand*0.02`s）去机器感
- 双保险：官方原声热链（如 archive.org）能播就播，`error/stalled` 自动回落合成器
- **章节情绪联动**是灵魂：滚到低潮章 masterFilter 降到 480Hz + BGM 变慢 1.25×（水下闷感），离开恢复 3200Hz
- ⚠️ AudioContext 必须用户手势后 resume——首次任意点击即起音乐，页面静音状态也要可手动开关

## 5. 游戏 HUD 化导航

- 顶部 fixed：徽标 + 章节徽章 + 胶囊导航 + 唱片机（EQ 跳动条 + 换碟 ⏭）/ 自动放映 / 语言 / 音效开关
- 底部 fixed：**经验条 = 阅读进度条**（LVL 数值随滚动从 9 涨到 25，暗合年龄线）+ hotbar 格子 = 各章传送门 + 唱机快捷键
- 成就弹窗：右上角滑入 "ACHIEVEMENT UNLOCKED"，奖励每次关键交互（点击开门、点亮 180 天等）
- 滚动监听一律 `{ passive: true }`，只改 width/transform，不触发重排

## 6. 每幕一个交互 widget（页面的灵魂）

| 幕 | 主题 | 交互 widget |
|---|---|---|
| I | 起点 2010 | 可点击体素条 → 生成地形 + 每格弹一个钢琴音阶 |
| II | 学 Java | 16×16 草方块 canvas 像素画 ↔ 十六进制字节码视图切换；IDE 窗口 ▶ Run 让 `curiosity++` 计数器狂涨 |
| III | 低谷暂停 | `▐▌ paused` 状态栏 + 「尝试解除暂停」按钮（永远失败，但每按一次给一句鸡汤） |
| IV | 重生 | 180 天 Git 贡献格（按住鼠标划过连续点亮，Git 四级绿）+ `comeback.py` 拆 Offer 信封 |
| V | 现在 | 30㎡ 等距房间 canvas（点家具换"正在做 X"状态）+ `while alive: learn("AI")` ⚡加速按钮（Epoch +1） |
| VI | 门 | CSS 3D 像素木门 `rotateY(-112deg)` 开门 → 门后露出蓝天草地 canvas（晴空/晚霞/星夜三态切换） |

widget 设计原则：每个都对应主题隐喻（Git 格子=坚持、信封=Offer、门=邀请），而不是随便塞按钮。

## 7. 跨章伏笔（高级技巧）

低潮之后埋"敲门声"：IV 章按钮敲 3 下（Web Audio 合成 knock，音量渐弱）才解锁线索；VI 章开门前必须先听到同样的敲门声——**跨 2000 行的因果链让页面像游戏而不是 PPT**。做新页时至少设计一条这种前后呼应。

## 8. 双语三态

- `body[data-lang-mode="zh|en"]` + CSS 规则隐藏 `.lang-en-only` / `.lang-zh-only`（静态文本零 JS）
- 动态文本用 `data-i18n-zh` / `data-i18n-en` 属性，切语言时遍历替换
- 语言按钮循环：中英双语 → 纯中 → 纯 EN

## 9. 视觉细节清单（廉价但高级感拉满）

- 光标追光：`--mouse-x/--mouse-y` CSS 变量 + 800px radial-gradient 跟随
- 影院暗角：`inset 0 0 140px rgba(0,0,0,.85)` box-shadow
- 毛玻璃卡片：`backdrop-filter: blur(24px)` + 顶部 1px 内描边高光 + mousemove 3D tilt
- 像素字只做点睛（标签/按钮），正文用衬线/无衬线，别满屏 Press Start 2P
- 自定义 5px 滚动条，hover 变主题色
- 低潮章全局 `filter: saturate(0.15)` + 扫描线 overlay，离开恢复

## 10. 部署

单文件零构建：GitHub 仓库放 `index.html` → Settings 开 Pages（main 分支根目录）即上线。本机推送要挂 Clash 代理（`HTTPS_PROXY=http://127.0.0.1:7897`）；大陆观众访问 github.io 需自备代理——介意的话后续换 Cloudflare Pages + 自有域名。

## 制作流程（新开一页时）

1. **先写剧本**：主题切成 5-6 幕，每幕定：一句金句 / 一个交互创意 / 情绪三件套（主色、BGM 状态、背景世界形态）
2. **搭骨架**：CDN `<head>` + Prologue + 全部空 section + 顶/底 HUD + 背景 canvas
3. **逐幕填充**：HTML 结构 → 挂 `.story-beat` → 写 widget 交互 → 接章节 observer
4. **最后加糖**：开场门、跨章伏笔、成就系统、自动放映、双语
5. 全程本地双击 `index.html` 预览（Chrome 即可，无需服务器），改完 push
