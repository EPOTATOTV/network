# 开发文档

面向具备基础前端知识（HTML / CSS / JavaScript）的维护者，说明本站的结构、改造点与日常维护方式。

## 目录

1. [项目概览](#1-项目概览)
2. [目录与职责](#2-目录与职责)
3. [页面公共结构](#3-页面公共结构)
4. [脚本 main.js 做了什么](#4-脚本-mainjs-做了什么)
5. [样式体系：CSS 与 SCSS 的关系（务必先读）](#5-样式体系css-与-scss-的关系务必先读)
6. [首页视频背景实现](#6-首页视频背景实现)
7. [字体本地自托管](#7-字体本地自托管)
8. [常见改动怎么做](#8-常见改动怎么做)
9. [大体积媒体与 Git](#9-大体积媒体与-git)
10. [部署](#10-部署)
11. [排查手册](#11-排查手册)
12. [已知限制](#12-已知限制)

---

## 1. 项目概览

- **类型**：纯静态站点，无构建步骤、无包管理器、无 CI
- **模板来源**：HTML5 UP 的 *Forty*（CCA 3.0 许可）
- **生效样式**：`assets/css/main.css`（唯一，手工维护）
- **核心脚本**：`assets/js/main.js`
- **相对上游的改造**：
  1. 首页横幅由静态图片改为**本地 MP4 视频背景 + 内联 SVG 遮罩层**
  2. 导航栏与弹出菜单改为**玻璃拟态**，强调色统一为 `#2b8cff`
  3. 字体由 Google Fonts 改为**本地自托管**

改样式请直接改 `assets/css/main.css`。第 5 节解释了为什么**不要**去编译 `assets/sass/`。

---

## 2. 目录与职责

| 路径 | 职责 |
| --- | --- |
| `index.html` | 首页。横幅视频、tiles 入口卡片、赛事特色、页脚 |
| `team.html` | 工作室成员页 |
| `joinus.html` | 联系方式与合作页 |
| `pic.html` | 办赛成果图片页 |
| `404.html` | 404 错误页 |
| `assets/css/main.css` | **站点全部生效样式** |
| `assets/css/noscript.css` | 禁用 JavaScript 时的降级样式 |
| `assets/css/fontawesome-all.min.css` | Font Awesome 5 Free（第三方） |
| `assets/fonts/*.woff2` | Source Sans Pro 本地字体（4 个） |
| `assets/js/main.js` | 站点行为脚本（未压缩，可读，是主要改动入口） |
| `assets/js/jquery*.js`、`browser.min.js`、`breakpoints.min.js`、`util.js` | 第三方与工具脚本（压缩版，一般不修改） |
| `assets/sass/**` | 上游模板的 SCSS 源码。**已与 main.css 不同步**，见第 5 节 |
| `assets/webfonts/**` | Font Awesome 字体文件（`.eot/.svg/.ttf/.woff/.woff2`） |
| `images/**` | 图片与背景视频。其中 `banner.mp4` 与母带 MP4 未纳入 Git |

---

## 3. 页面公共结构

所有页面共享同一套骨架：

```html
<body class="is-preload">
  <div id="wrapper">

    <!-- 顶栏：alt 表示处于横幅上方的透明态 -->
    <header id="header" class="alt">
      <a href="index.html" class="logo"><strong>POTATOTV</strong> <span>…</span></a>
      <nav><a href="#menu">菜单</a></nav>
    </header>

    <!-- 弹出菜单（内容在运行时被 main.js 包裹进 .inner） -->
    <nav id="menu">
      <ul class="links">…</ul>
      <ul class="actions stacked">…</ul>
    </nav>

    <!-- 横幅 -->
    <section id="banner" class="major">
      …
    </section>

    <!-- 正文 -->
    <div id="main">
      <section id="one" class="tiles">…</section>
      <section id="two">…</section>
    </div>

    <footer id="footer">…</footer>
  </div>

  <script src="assets/js/jquery.min.js"></script>
  <script src="assets/js/jquery.scrolly.min.js"></script>
  <script src="assets/js/jquery.scrollex.min.js"></script>
  <script src="assets/js/browser.min.js"></script>
  <script src="assets/js/breakpoints.min.js"></script>
  <script src="assets/js/util.js"></script>
  <script src="assets/js/main.js"></script>
</body>
```

脚本的加载顺序**不可调整**：`main.js` 依赖前面的 jQuery 与工具库。

### `body.is-preload` 的作用

初始 `is-preload` 类会让横幅内容保持模糊、位移与透明状态，避免首屏出现"跳变"。`main.js` 在 `window` 的 `load` 事件后 100ms 移除该类，从而触发 CSS 过渡形成淡入效果。若禁用 JS，则由 `noscript.css` 处理降级。

### 首页的日期计数脚本

`index.html` 的横幅下方有一段内联脚本，每秒刷新一次，在 `#h1` 元素里显示"PotatoTV赛事已举办 N 天"：

```javascript
time(h1, '2023/1/13');   // 起始日期在此处硬编码
```

如需修改起始日期，改这一个参数即可。

---

## 4. 脚本 main.js 做了什么

`main.js` 已保留 HTML5 UP 原版结构，主要职责：

| 功能 | 说明 |
| --- | --- |
| 断点系统 | 注册 `xlarge / large / medium / small / xsmall / xxsmall` 六档断点，供 CSS 与其它模块联动 |
| 首屏动画 | `window.load` 后移除 `body.is-preload` |
| 锚点平滑滚动 | 对 `.scrolly` 元素启用 `scrolly`，偏移量为"顶栏高度 − 2px" |
| **tiles 卡片** | 遍历 `.tiles > article`：把内部 `<img>` 的地址设为其父元素的 `background-image`，随后隐藏原 `<img>`；并为 `.link` 复制一个 `.primary` 覆盖整块的链接 |
| 顶栏状态 | 横幅滚出视口时，`#header` 由 `alt` 切换为 `reveal` |
| 横幅视差 | 对 `#banner` 启用 `_parallax(0.275)`（仅桌面端） |
| 弹出菜单 | 包裹 `.inner`、加锁防抖、开合切换、`Esc` 关闭、点击外部关闭、注入 `Close` 按钮 |

理解 tiles 的行为对新增卡片很重要，见第 8.1 节。

---

## 5. 样式体系：CSS 与 SCSS 的关系（务必先读）

### 5.1 现状

`assets/sass/` 是 **上游 Forty 模板的原始 SCSS 源码**，它**不包含**本站的任何改造。`assets/css/main.css` 才是实际生效的样式，其结构为：

```
assets/css/main.css
├── 1 行          @import fontawesome-all.min.css
├── 6–36 行       本站新增的 4 个 @font-face（本地字体）
├── 约 37–4043 行  上游 Forty 的编译产物（banner / tiles / header / menu 等）
└── 约 4044 行起   本站新增的 "Theme Refresh" 主题层（文件末尾）
```

"Theme Refresh" 主题层包含：

- `:root` 中的 CSS 变量：
  - `--ptv-accent: #2b8cff`（强调色，蓝）
  - `--ptv-accent-deep: #0b62d6`（按下态）
  - `--ptv-ink: #060b16`（深色底）
  - `--ptv-glass-line: rgba(255,255,255,0.12)`（玻璃描边）
- `#header`、`#header.alt`、`#menu` 的玻璃拟态（`backdrop-filter`）
- `#header .logo strong` 及其 hover / active 配色
- `#header nav a[href="#menu"]:after` 与 `#menu .close:after` 的蓝色图标（内联 SVG 的 `stroke` 改为 `%232b8cff`）
- `.button.primary` 及其 hover / active 配色
- `#banner h1` 的静态文字阴影

主题层写在文件**末尾**，是为了利用 CSS 层叠顺序覆盖上游规则，而不必改动上游代码。

### 5.2 重要警告

> **不要直接编译 `assets/sass/main.scss`。**
>
> `main.scss` 第 8 行仍然是 `@import url('https://fonts.googleapis.com/...')`，且 `sass/libs/_vars.scss` 里的调色板仍是上游原值（`bg: #242943`、`highlight: #9bf1ff`）。用它编译出来的 CSS 会：
>
> 1. **丢失**文件末尾的整个主题层（玻璃拟态与蓝色强调色全部消失）
> 2. **覆盖**本地字体声明，回退为 Google Fonts 引用（国内不可达，字体将退回系统默认）
>
> 也就是说，编译产物会直接把官网外观"打回原形"。

因此当前工作流是：**直接编辑 `assets/css/main.css`**。

若将来希望恢复 SCSS 工作流，正确做法是**先把 `main.css` 的三处改造回填进 SCSS**，再启用编译：

1. 在 `main.scss` 中用 `@font-face` 替换 Google Fonts 的 `@import`
2. 把主题层的规则写进对应的局部文件（如 `layout/_header.scss`、`layout/_menu.scss`、`components/_button.scss`）
3. 把 `_vars.scss` 的 `bg` / `highlight` 等替换为本站配色
4. 编译后与当前 `main.css` 做视觉对比确认无回归

---

## 6. 首页视频背景实现

### 6.1 HTML 结构

`index.html` 的横幅由三层组成，自下而上依次是 **视频 → SVG 遮罩 → 文字内容**：

```html
<section id="banner" class="major video-banner">
  <!-- 第 1 层：背景视频 -->
  <video class="banner-video" autoplay muted loop playsinline preload="auto"
         poster="images/banner.jpg">
    <source src="images/banner.mp4" type="video/mp4" />
  </video>

  <!-- 第 2 层：SVG 遮罩（渐变 + 网格） -->
  <svg class="banner-svg" xmlns="http://www.w3.org/2000/svg"
       preserveAspectRatio="none" aria-hidden="true" focusable="false">
    <defs>
      <linearGradient id="bannerScrim" x1="0" y1="0" x2="0" y2="1">
        <stop offset="0%"   stop-color="#060b16" stop-opacity="0.72" />
        <stop offset="45%"  stop-color="#060b16" stop-opacity="0.42" />
        <stop offset="100%" stop-color="#060b16" stop-opacity="0.92" />
      </linearGradient>
      <pattern id="bannerGrid" width="48" height="48" patternUnits="userSpaceOnUse">
        <path d="M48 0H0V48" fill="none" stroke="#2b8cff"
              stroke-opacity="0.14" stroke-width="1" />
      </pattern>
    </defs>
    <rect width="100%" height="100%" fill="url(#bannerScrim)" />
    <rect width="100%" height="100%" fill="url(#bannerGrid)" />
  </svg>

  <!-- 第 3 层：文字内容 -->
  <div class="inner">…</div>
</section>
```

使用内联 SVG 而非位图，好处是：矢量网格任意分辨率都锐利、可随主题色改一个变量就换色、体积极小且不产生额外请求。

### 6.2 CSS 要点

```css
#banner.video-banner { overflow: hidden; background-image: none; }

#banner .banner-video {
  position: absolute; top: 50%; left: 50%;
  width: 100%; height: 100%; min-width: 100%; min-height: 100%;
  transform: translate(-50%, -50%);
  object-fit: cover;            /* 任意屏幕比例下铺满且无黑边 */
  z-index: 0;
  pointer-events: none;
  filter: saturate(1.15) contrast(1.05) brightness(0.9);
}

#banner .banner-svg {
  position: absolute; top: 0; left: 0; width: 100%; height: 100%;
  z-index: 1; pointer-events: none; display: block;
}

/* 原模板的深色重遮罩改为淡蓝渐变，让视频透出 */
#banner.video-banner:after {
  background-color: transparent;
  background-image: linear-gradient(to bottom, rgba(43,140,255,0.16), rgba(43,140,255,0));
  opacity: 1;
}
```

最内层的文字 `.inner` 由上游规则设为 `z-index: 2`，因此位于视频与 SVG 之上。

移动端（`max-width: 736px`）横幅高度会变为 `auto`，此时绝对定位的视频无法计算百分比高度，故补了一条 `#banner.video-banner { min-height: 28em; }` 兜底。

### 6.3 视频规格

| 项 | 值 |
| --- | --- |
| 文件名 | `images/banner.mp4` |
| 分辨率 | 1280×720（720p） |
| 帧率 | 30 fps |
| 编码 | H.264（`libx264`），`yuv420p` |
| 码率 | 约 1.38 Mbps |
| 音轨 | 无（`-an`，背景视频本就静音） |
| 时长 | 4 分 11 秒 |
| 体积 | 约 41.5 MB |
| 封装 | MP4，`+faststart`（`moov` 前置，可边下边播） |

母带为 1920×1080 / 60fps / 18 Mbps / 540 MB。

### 6.4 重新生成视频

```bash
ffmpeg -i master.mp4 \
  -vf "scale=-2:720,fps=30" \
  -c:v libx264 -preset veryfast -crf 33 \
  -an -pix_fmt yuv420p \
  -movflags +faststart \
  banner.mp4
```

参数说明：

- `scale=-2:720`：高度固定 720，宽度按比例自动计算并对齐到偶数（编码器要求）
- `fps=30`：降到 30 帧。素材本身 60fps，降帧是**最有效的体积削减手段**（画面本身仍足够流畅）
- `-crf 33`：质量/体积平衡点。数值越大体积越小；`28` 约 59MB，`31` 约 45MB，`33` 约 41MB（对 Minecraft 这类高帧率、纹理密集的画面，压缩率明显低于普通实拍素材）
- `-preset veryfast`：编码速度与压缩率的折中。换 `medium` 仅省约 1MB，耗时却翻倍，不值得
- `-an`：去掉音轨（浏览器端本来就 `muted`）
- `-pix_fmt yuv420p`：保证各类浏览器与设备的兼容性
- `-movflags +faststart`：把索引前置，使视频无需下完即可播放

**调参建议**：先用 10 秒片段试算，避免整片反复转码：

```bash
ffmpeg -ss 90 -t 10 -i master.mp4 -vf "scale=-2:720,fps=30" \
  -c:v libx264 -preset veryfast -crf 33 -an -movflags +faststart probe.mp4
```

用 `probe.mp4` 的体积乘以（总时长 ÷ 10）即得全片估算值。

### 6.5 为什么必须写 `muted` 和 `playsinline`

浏览器普遍禁止**带声音**的媒体自动播放。`autoplay` 只有在同时声明 `muted` 时才被允许；`playsinline` 则避免 iOS Safari 将视频强制全屏接管。这两者与 `autoplay` 缺一不可。

### 6.6 更换视频时的缓存问题

线上静态资源被配置为 30 天强缓存（见 README 的 nginx 片段）。若**用同名文件覆盖**更新视频，老访客可能长时间仍播放旧视频。稳妥做法是：

- 换文件名（如 `banner-v2.mp4`）并同步更新 `<source src>`，或
- 在地址后加版本查询串（如 `banner.mp4?v=2`）

---

## 7. 字体本地自托管

`assets/css/main.css` 开头声明了 4 个 `@font-face`，对应 Source Sans Pro 的 300 / 600 字重（含斜体）：

```css
@font-face {
  font-family: 'Source Sans Pro';
  font-style: normal;
  font-weight: 300;
  font-display: swap;
  src: url("../fonts/source-sans-pro-latin-300-normal.woff2") format("woff2");
}
/* 另有 300-italic、600-normal、600-italic 共 3 个 */
```

- 文件位于 `assets/fonts/`，路径是**相对于 main.css** 的 `../fonts/`
- `font-display: swap` 让文字先用系统字体渲染，字体加载完成后替换，避免首屏空白
- 字体族名 `'Source Sans Pro'` 必须与 `main.css` 第 154 行附近的 `font-family` 声明完全一致

### 为什么不用 Google Fonts

`https://fonts.googleapis.com` 在境内不可达（表现为 `ERR_TOO_MANY_REDIRECTS`），字体会静默回退为系统字体，观感与设计稿不符且不产生明显报错，排查成本高。改为自托管后既消除了这次外部请求，也保证了国内访问的一致性。

### 新增字重

1. 将对应的 `.woff2` 放入 `assets/fonts/`
2. 在 `main.css` 开头的 `@font-face` 区块追加一条声明
3. 若需在版式中使用，同步检查第 154 行的 `font-family` 所在规则

---

## 8. 常见改动怎么做

### 8.1 新增 / 修改首页入口卡片（tiles）

首页卡片位于 `index.html` 的 `<section id="one" class="tiles">` 中，每张卡片是一个 `<article>`：

```html
<article>
  <span class="image">
    <img src="images/foo.png" alt="" />
  </span>
  <header class="major">
    <h3><a href="目标地址" class="link">卡片标题</a></h3>
    <p>Card subtitle</p>
  </header>
</article>
```

要点：

- `main.js` 会把这个 `<img>` 的地址设成整块卡片的背景图，并隐藏原 `<img>`，因此**图片路径必须正确**，否则卡片会是空白块
- 标题链接必须带 `class="link"`，`main.js` 才会生成覆盖整卡的点击区域
- 卡片在桌面端按两列交错排布，宽度与高度由 CSS 控制，通常无需写内联样式
- 图片建议预先压缩（单张控制在数百 KB 内），这些图不参与自动优化

### 8.2 修改导航菜单项

菜单项在 `nav#menu` 的 `<ul class="links">` 中，直接增删 `<li><a href="…">…</a></li>` 即可。注意各页面的菜单项并不完全一致（`joinus.html`、`team.html` 比首页多几项），如需全站统一需逐页修改。

### 8.3 新增一个页面

1. 复制任一现有页面（推荐 `team.html`，结构最简）作为模板
2. 修改 `<title>`、`.logo` 的 `<span>`、横幅 `<h1>` 与正文
3. 保留骨架部分：`#header`、`#menu`、`#footer` 以及底部全部 `<script>` 标签
4. 若该页需要出现在菜单中，在所有页面的 `nav#menu` 里补上链接

### 8.4 调整主题强调色

主题层集中在 `main.css` 末尾的 `:root`：

```css
:root {
  --ptv-accent: #2b8cff;
  --ptv-accent-deep: #0b62d6;
  --ptv-ink: #060b16;
  --ptv-glass-line: rgba(255, 255, 255, 0.12);
}
```

改 `--ptv-accent` 即可同步生效于按钮、菜单 hover 等位置。**但有例外**：以下三处的颜色是硬编码的，不跟随变量，需一并手工替换：

- `#header nav a[href="#menu"]:after` 的内联 SVG（`stroke: %232b8cff`）
- `#menu .close:after` 的内联 SVG（同上）
- `#banner h1` 的 `text-shadow`（`rgba(43, 140, 255, …)`）

另外，首页横幅 SVG 的网格线颜色写死在 `index.html` 的 `<pattern>` 里（`stroke="#2b8cff"`），改主题色时也需同步。

> 内联 SVG 里的 `#` 必须写成 URL 编码的 `%23`，否则会被当作片段标识符而失效。

### 8.5 修改横幅标题与副标题

在对应页面的 `<section id="banner">` 内修改 `<h1>` 与 `.content` 中的文字。注意首页横幅的 `.content` 里有一个 `<p id="h1">`，其内容是第 3 节的日期脚本动态写入的，**不要删掉该元素**。

---

## 9. 大体积媒体与 Git

`.gitignore` 排除了两个视频文件：

```
images/43508f6a-2a5e-4608-bf8e-602aa098ac0a.MP4   # 母带，540MB
images/banner.mp4                                  # 线上使用的 720p30 转码版，41.5MB
```

原因：

- GitHub 对**单个文件**硬上限为 100MB，母带 540MB 无法推送
- 41.5MB 的转码版虽可提交，但每次克隆都会拉取，且它属于"可再生成的产物"，因此同样排除

**影响**：新克隆的仓库中**没有** `banner.mp4`，直接打开首页只会显示封面图 `banner.jpg`。部署或本地预览前需按第 6.4 节自行生成，或从维护者处获取。

若希望仓库自包含（clone 后即可看到完整效果），可删除 `.gitignore` 中 `images/banner.mp4` 一行再提交——但需接受约 41.5MB 的仓库体积增长。

---

## 10. 部署

站点为纯静态资源，部署即"上传文件"。完整步骤（含 nginx 与 HTTPS 配置）见 [README.md](./README.md#部署)。

开发侧需要留意的两点：

1. **必须连同 `banner.mp4` 一起上传**，该文件不在 Git 中，容易遗漏
2. 上传后若发现样式或脚本仍是旧版，先排查浏览器缓存与 CDN 缓存，再怀疑部署本身（参见第 11 节）

---

## 11. 排查手册

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| 首页横幅只有静态封面图，没有视频 | `images/banner.mp4` 缺失 | 该文件不在 Git 中，需单独获取/生成并上传 |
| 视频存在但不自动播放 | 缺少 `muted` 或 `playsinline` | 补回 `<video>` 上的这两个属性 |
| 视频加载慢或播放卡顿 | 体积过大或码率过高 | 按第 6.4 节重新转码；确认服务器开启了 `Accept-Ranges`（支持分段请求） |
| 中文与标题字体不对 | 本地字体未加载 | 检查 `assets/fonts/*.woff2` 是否存在，以及 `main.css` 中路径是否为 `../fonts/…` |
| 字体位置对但仍是系统字体 | `@font-face` 的 `font-family` 名与使用处不一致 | 两者必须都是 `'Source Sans Pro'` |
| 改了样式但页面没变化 | 浏览器缓存（静态资源 30 天强缓存） | 强制刷新（Ctrl+F5），或给资源地址加版本查询串 |
| 卡片显示为空白块 | `<img>` 路径错误 | tiles 的背景图取自该 `<img>` 的 `src`，路径必须有效 |
| 玻璃拟态看不到模糊效果 | 浏览器不支持 `backdrop-filter` | 属降级行为，导航栏会显示为半透明纯色 |
| 菜单点击无反应 | `main.js` 未加载或脚本顺序被打乱 | 确认底部脚本顺序与第 3 节一致 |
| HTTPS 证书到期报错 | 自动续期未生效 | 检查服务器上的 certbot 续期定时任务与日志 |

---

## 12. 已知限制

- **SCSS 源码已不同步**：`assets/sass/` 不反映本站改造，编译会覆盖主题层与本地字体（见第 5 节）
- **无构建与压缩流程**：`main.css` 为手工维护，改动后不会自动生成压缩版或添加浏览器前缀
- **视频未纳入版本控制**：换机器或重新克隆后需重新获取 `banner.mp4`
- **菜单项在各页面间不完全一致**：修改导航需逐页检查
- **部分颜色硬编码**：主题色的三处内联 SVG 与横幅标题阴影不跟随 CSS 变量（见第 8.4 节）
- **无自动化测试**：改动后建议人工核对首页、工作室、加入我们、办赛成果四个页面在桌面与手机宽度下的表现
