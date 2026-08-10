# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目性质

这是基于 **Jekyll + Minimal Mistakes 主题**的学术个人主页，fork 自
[academicpages/academicpages.github.io](https://github.com/academicpages/academicpages.github.io)。
当前站点：https://steam009.github.io/

代码主干在 `master` 分支，包含 Markdown 源文件、SCSS 片段、模板等。

## 常用命令

### 本地预览
```bash
bundle exec jekyll serve -l -H localhost
# 然后浏览器打开 http://localhost:4000
```

### 纯 build（不需要服务器）
```bash
bundle exec jekyll build           # 输出到 _site/
bundle exec jekyll build --profile # 看每个文件的耗时，定位慢文件
```

### Clean rebuild（CSS 缓存导致样式不更新时用）
```bash
rm -rf _site _sass/.sass-cache && bundle exec jekyll build
```

### 部署
**当前部署链路已切换到 `gh-pages` 分支**（旁路 GitHub Actions，因为 2026/07 GH Pages Actions 路径出过排长队/失败故障）。
每次改完代码，**手动**走这两步：

```bash
rm -rf _site _sass/.sass-cache
bundle exec jekyll build
git -C _site add -A
git -C _site commit -m "Deploy: <short desc>"
git -C _site push origin gh-pages --force
```

如果想让 GitHub Actions 自动部署，重新切回 `actions/deploy-pages` 路径：在
Settings → Pages → Source 改回 "GitHub Actions"。但 7 月那次故障说明这条路径不稳定。

### Docker（如不想装 Ruby）
```bash
docker compose up --build
# 见 DOCKER_USAGE.md
```

### 其他
- `bundle check` — 检查 Gemfile 依赖是否装好（项目级 allow 已预批准 `bundle *` 命令）
- `touch _config.local.yml && bundle exec jekyll serve ...` — 强制 jekyll 重新加载配置

## 关键文件结构

| 路径 | 用途 |
|---|---|
| `_config.yml` | Jekyll 全站配置（author / sidebar / plugins / sass_dir） |
| `_config.local.yml` | 本地覆盖配置（gitignore） |
| `Gemfile` | 依赖（`github-pages` gem 锁定白名单插件） |
| `_pages/about.md` | **最常改的文件** —— 个人主页内容 |
| `_pages/*.md` | 各种固定页面（publications / talks / cv / teaching…） |
| `_posts/*.md` | 博客（日期前缀） |
| `_data/*.yml` | 站点元数据（authors / navigation / ui-text） |
| `_data/cv.json` | CV 数据，被 `cv-json.md` 渲染 |
| `_sass/` | SCSS 源码（**没有 `main.scss` 入口**） |
| `assets/css/main.scss` | **真正的 SCSS 入口**，用 `@import "..."` 把片段拼起来 |
| `_sass/layout/_sidebar.scss` | 头像、sidebar grid 宽度 |
| `_sass/layout/_page.scss` | 主内容区 grid 宽度 |
| `_sass/layout/_masthead.scss` | 顶部导航 |
| `_sass/_experience-row.scss` | 我新增的：`.experience-row` / `.experience-meta` / `.award-badge` |
| `_includes/` | Liquid include 片段（head / seo / sidebar / author-profile…） |
| `_includes/author-profile.html` | 头像渲染逻辑（也是改名字/bio 的入口） |
| `images/photo.png` | 侧栏头像（2688×1512，16:9 横屏） |
| `talkmap/` | Leaflet 地图 + JS 数据 |

## 架构关键点

### 1. SCSS 编译入口在 `assets/css/main.scss`
跟 Minimal Mistakes 默认仓库不一样——GitHub 标准的 minimal-mistakes 用 `assets/css/main.scss`
做入口，这里**没有删掉这个文件**，所以新增 SCSS 片段的正确姿势是：
1. 在 `_sass/` 下加新文件（如 `_my-feature.scss`）
2. 在 `assets/css/main.scss` 里加 `@import "my-feature"`（**不要带下划线前缀**）

### 2. 主题变量优先级
- `var(--global-text-color)` 等 CSS 变量定义在 `_sass/theme/_default.scss` 和 `_dark.scss`
- 跨主题可用的灰度颜色用 `var(--global-text-color-light)` 这种 CSS 变量，**不要用 SCSS 变量**（$gray / $dark-gray 等只在 _default/_dark 内部）
- 类型尺寸用 `$type-size-1` ~ `$type-size-8`（在 `_sass/_themes.scss` 全局定义）

### 3. responsive 设计用 susy + breakpoint
- `_page.scss` 里 `.page { @include span(10 of 12 last); }` —— 这是 susy 网格语法
- breakpoint 函数：`@include breakpoint($large) { ... }`（`$large = 925px`）

### 4. 头像布局算法（重要）
头像的"显示不全"经常是 sidebar 容器宽度限制 + `width: 100%` + 16:9 横屏照片组合导致的——
[`_sass/layout/_sidebar.scss`](_sass/layout/_sidebar.scss) 当前用：
- `width: 220px; height: 220px`（固定正方形）
- `object-fit: cover; object-position: center 25%`（让 16:9 横屏照自动裁切，人脸偏上对齐）

如果调整头像尺寸要注意 `object-position` 的 `25%` 配合你的实际照片位置。

### 5. 部署链路的临时变通
2026/07/02 GitHub Pages Actions 路径出过"slow and failing Pages deployments"故障，
本项目已**临时切到 `gh-pages` 分支部署**（手动 push `_site/`）。
再次启用 Actions 路径前，先看 https://www.githubstatus.com/ 确认 Pages operational。

### 6. 已知遗留
- `_config.yml` 里 `future: true` —— 会让 Jekyll 处理未来日期的 post，对 build 时间有小影响
- `bio: "Times New Roma"` —— 测试文本，需要替换为真实 bio（user 知情未改）

## 不要做的事

- **不要删 `assets/css/main.scss`**——这是 SCSS 入口，删了之后所有样式不打包
- **不要用 `_site/` 做源码修改**——它是 generated，重新 build 会被覆盖
- **不要把 `.env` / token / 私钥提交**—— `_gitignore` 已保护，但别手动覆盖
- **不要 git add `vendor/` / `_site/` / `.sass-cache` / `.bundle`**—— 已通过 `.gitignore` 排除
- **不要把 commit message / 注释里贴 token / API key** —— 用户全局规则

## 测试

此项目**没有**自动化测试。前端是静态站点，验证靠：
- 本地 `bundle exec jekyll serve` + 浏览器手动检查
- 切多个视口（Chrome DevTools → Toggle Device → iPhone/iPad）
- 检查关键路径：首页头像、Publications 渲染、TA 奖项徽章、visitor map include

## 关联资源

- 主题官方文档：https://mmistakes.github.io/minimal-mistakes/
- 上游模板：https://github.com/academicpages/academicpages.github.io
- Liquid 语法：https://shopify.github.io/liquid/
- Jekyll 文档：https://jekyllrb.com/docs/
