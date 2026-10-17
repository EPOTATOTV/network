# POTATOTV 赛事官网

POTATOTV 起床战争赛事社区官方网站，纯静态站点。

- 线上地址：<https://www.potatotv.asia>
- 代码仓库：<https://github.com/EPOTATOTV/network>

## 简介

本站用于发布 POTATOTV 起床战争赛事的相关信息：赛事守则、工作室成员、办赛成果、赞助渠道与反作弊客户端下载等。

站点为纯静态 HTML / CSS / JavaScript，**无需任何构建步骤**，克隆仓库后即可直接部署。

## 主要特性

- 首页横幅使用本地 MP4 视频背景，叠加内联 SVG 渐变遮罩与网格纹理
- 顶部导航栏与弹出菜单使用玻璃拟态（`backdrop-filter` 毛玻璃）与蓝色强调色
- 响应式布局，覆盖桌面至手机各档断点
- Source Sans Pro 字体本地自托管，不依赖境外 CDN

## 页面

| 页面 | 文件 | 说明 |
| --- | --- | --- |
| 首页 | `index.html` | 横幅视频、赛事入口卡片、赛事特色 |
| 工作室 | `team.html` | POTATOTV 工作室成员介绍 |
| 加入我们 | `joinus.html` | 联系方式与合作入口 |
| 办赛成果 | `pic.html` | 历届赛事图片 |
| 错误页 | `404.html` | 404 页面 |

## 技术栈

- HTML5 + CSS3（原生，无框架、无构建工具）
- JavaScript：jQuery（本地引入）+ 站点脚本 `assets/js/main.js`
- 图标：Font Awesome 5 Free（本地）
- 页面模板：基于 HTML5 UP 的 **Forty**

## 本地预览

不要直接双击打开 `index.html`——在 `file://` 协议下浏览器可能拒绝加载视频与字体。请启动一个本地 HTTP 服务：

```bash
python -m http.server 8080
```

然后访问 <http://127.0.0.1:8080/>。

## 目录结构

```
.
├── index.html / team.html / joinus.html / pic.html / 404.html
├── assets
│   ├── css/        main.css（生效样式）、noscript.css、fontawesome-all.min.css
│   ├── fonts/      Source Sans Pro 本地字体
│   ├── js/         jQuery 与站点脚本（main.js 为核心）
│   ├── sass/       上游模板的 SCSS 源码（未同步，见开发文档）
│   └── webfonts/   Font Awesome 字体文件
├── images/         图片与背景视频
└── .gitignore
```

## 部署

站点为纯静态资源，将其内容放入任意 Web 服务器的站点根目录即可。

### 1. 获取代码

```bash
git clone https://github.com/EPOTATOTV/network.git
```

### 2. 补齐背景视频

`images/banner.mp4`（首页横幅视频）因体积原因**未纳入 Git 仓库**（见 `.gitignore`）。部署前须单独获取该文件并放入 `images/` 目录，否则首页横幅只会显示静态封面图 `banner.jpg`。

该视频的规格与重新生成方法见 [DEVELOP.md](./DEVELOP.md#6-首页视频背景实现)。

### 3. 上传到服务器

```bash
rsync -av --delete ./ user@<服务器地址>:/var/www/potatotv/
```

### 4. 配置 nginx

```nginx
server {
    listen 80;
    server_name www.example.com;

    root  /var/www/potatotv;
    index index.html;
    charset utf-8;

    gzip on;
    gzip_types text/css application/javascript image/svg+xml;

    # 静态资源：长缓存
    location ~* \.(?:png|jpe?g|gif|webp|ico|svg|woff2?|ttf|eot|mp4|webm)$ {
        expires 30d;
        access_log off;
    }

    # HTML：不缓存，便于更新后立即生效
    location ~* \.html?$ {
        expires -1;
    }

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
}
```

### 5. 启用 HTTPS

域名解析到服务器后，使用 Let's Encrypt 免费证书：

```bash
certbot --nginx -d www.example.com
```

`certbot` 会写入证书配置并自动设置续期。

> 服务器地址、账号、密钥等部署凭据不保存在本仓库中，请向维护者索取。

## 浏览器兼容

需要支持以下特性的现代浏览器：`backdrop-filter`、CSS 自定义属性、`object-fit`、HTML5 `<video>` 自动播放。

- Chrome / Edge 76+
- Firefox 103+
- Safari 12+

不支持 `backdrop-filter` 的旧浏览器上，导航栏会降级为半透明纯色背景（内容仍可正常阅读）。

## 更多文档

开发、样式改造与维护细节见 [DEVELOP.md](./DEVELOP.md)。

## 许可与版权

- 页面模板基于 HTML5 UP 的 **Forty**，依 CCA 3.0 许可使用
- 图标字体为 Font Awesome 5 Free
- 站点内容、图片与代码：POTATOTV 工作室群 版权所有
