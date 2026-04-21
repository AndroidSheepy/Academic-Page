# 个人学术主页 SOP

> 本地 Docker 开发 + 内容修改速查手册。本项目基于 al-folio 模板，部署目标为 Cloudflare Pages。

---

## 目录

- [一、环境要求](#一环境要求)
- [二、启动 / 停止容器](#二启动--停止容器)
- [三、访问本地站点](#三访问本地站点)
- [四、修改站点内容](#四修改站点内容)
  - [4.1 全站信息（姓名、标题、域名）](#41-全站信息姓名标题域名)
  - [4.2 头像](#42-头像)
  - [4.3 About 页（首页）](#43-about-页首页)
  - [4.4 社交链接](#44-社交链接)
  - [4.5 CV / 简历](#45-cv--简历)
  - [4.6 发表论文（Publications）](#46-发表论文publications)
  - [4.7 博客文章（Blog）](#47-博客文章blog)
  - [4.8 News（动态）](#48-news动态)
  - [4.9 Projects（项目）](#49-projects项目)
  - [4.10 Teaching（教学）](#410-teaching教学)
  - [4.11 导航栏显隐](#411-导航栏显隐)
- [五、热重载原理与注意事项](#五热重载原理与注意事项)
- [六、常见问题排查](#六常见问题排查)
- [七、命令速查](#七命令速查)

---

## 一、环境要求

| 项                | 要求                                                          |
| ----------------- | ------------------------------------------------------------- |
| Docker Desktop    | 已安装并运行（`docker info` 无报错）                          |
| 端口 8080 / 35729 | 未被其他服务 **LISTEN** 占用                                  |
| 项目路径          | `~/Library/CloudStorage/OneDrive-Personal/code/Academic-Page` |

> ⚠️ OneDrive 同步目录下开发 Jekyll 有时会因为 OneDrive 重命名临时文件触发多余重建，如遇构建异常可考虑把项目 clone 到本地非同步路径。

---

## 二、启动 / 停止容器

所有命令在项目根目录执行：

```bash
cd "/Users/androidsheep/Library/CloudStorage/OneDrive-Personal/code/Academic-Page"
```

### 首次启动（或镜像更新后）

```bash
docker compose pull       # 拉取 amirpourmand/al-folio:v0.16.3
docker compose up -d      # 后台启动容器
```

### 日常启动

```bash
docker compose up -d
```

### 查看实时日志

```bash
docker compose logs -f
# Ctrl+C 仅退出日志查看，不会停止容器
```

### 暂停 / 恢复

```bash
docker compose stop       # 暂停（保留容器状态）
docker compose start      # 恢复
```

### 彻底停止并清理容器

```bash
docker compose down
# 加 -v 会删除卷；一般不需要
```

### 强制重建（改了 Dockerfile 或 Gemfile 后）

```bash
docker compose up -d --build --force-recreate
```

---

## 三、访问本地站点

浏览器打开：

> **http://localhost:8080/al-folio/**

⚠️ 末尾的 `/al-folio/` 不能省，因为 [\_config.yml](_config.yml) 的 `baseurl: /al-folio` 仍是模板默认值。后续部署到 Cloudflare 自定义域名时 `baseurl` 置空，届时本地访问也会变成 `http://localhost:8080/`。

---

## 四、修改站点内容

**核心心智模型**：改项目文件 → 挂载进容器 → Jekyll 自动检测 → 增量重建 → 浏览器刷新即可见。**除极少数例外（见 §5），都不需要重启容器。**

### 4.1 全站信息（姓名、标题、域名）

**文件**：[\_config.yml](_config.yml)

关键字段：

```yaml
title: "" # 留空则用 first+last name
first_name: Jiaqi
middle_name:
last_name: Ruan
description: >
  PhD Student at MBZUAI. Research interests: ...
keywords: mbzuai, machine learning, ... # SEO 关键词
url: https://jiaqiruan.com # 部署后的根域名
baseurl: # Cloudflare 自定义域名 → 留空（不要删此行）

icon: 🎓 # 浏览器标签页 favicon（emoji 或 assets/img/ 下图片名）
```

⚠️ 改 `_config.yml` 会**自动重启 Jekyll**（见 [bin/entry_point.sh](bin/entry_point.sh) 的 inotify 监听），大约 30 秒后生效。

### 4.2 头像

替换文件：[assets/img/prof_pic.jpg](assets/img/prof_pic.jpg)

- 建议分辨率 ≥ 800×800
- 同名替换即可，Jekyll 会自动生成 480/800/1400 三种 WebP 尺寸
- 生成过程可能触发一次较长的重建（ImageMagick 处理）

### 4.3 About 页（首页）

**文件**：[\_pages/about.md](_pages/about.md)

结构：

- 顶部 front matter 控制页面元数据（标题、子标题、profile 组件、news/社交/最新文章开关）
- 正文是 Markdown，支持 HTML 片段

常见定制：

```yaml
subtitle: <a href='https://mbzuai.ac.ae/'>MBZUAI</a>. Abu Dhabi, UAE.

profile:
  align: right
  image: prof_pic.jpg
  image_circular: false
  more_info: >
    <p>Office xxx</p>
    <p>MBZUAI, Masdar City</p>
    <p>Abu Dhabi, UAE</p>

selected_papers: true # 首页显示标记为 selected 的论文
latest_posts:
  enabled: true
social: true
announcements:
  enabled: true
```

### 4.4 社交链接

**文件**：[\_data/socials.yml](_data/socials.yml)

取消注释对应字段并填值即可，例如：

```yaml
email: jiaqi.ruan@mbzuai.ac.ae
github_username: your-github
scholar_userid: xxxxxxxxx # Google Scholar 主页 URL 中 user= 后那一段
orcid_id: 0000-0000-0000-0000
linkedin_username: jiaqi-ruan
```

### 4.5 CV / 简历

两种方式**二选一**：

**方式 A：YAML 格式（推荐）** — 编辑 [\_data/cv.yml](_data/cv.yml)，按现有结构加条目。

**方式 B：JSON Resume** — 用 `assets/json/resume.json`，符合 [JSON Resume Schema](https://jsonresume.org/)。

页面入口：[\_pages/cv.md](_pages/cv.md)（一般不用改）。

### 4.6 发表论文（Publications）

**文件**：[\_bibliography/papers.bib](_bibliography/papers.bib)

标准 BibTeX 格式，al-folio 扩展了若干自定义字段：

```bibtex
@article{ruan2026example,
  title     = {Your Paper Title},
  author    = {Ruan, Jiaqi and Coauthor, Name},
  journal   = {NeurIPS},
  year      = {2026},
  selected  = {true},                       # 首页 selected_papers 会显示
  abbr      = {NeurIPS},                    # 左侧彩色标签
  abstract  = {...},
  pdf       = {https://arxiv.org/pdf/xxxx}, # 或放到 assets/pdf/ 下并写文件名
  arxiv     = {2604.12345},
  code      = {https://github.com/...},
  bibtex_show = {true},                     # 显示 BibTeX 按钮
  preview   = {paper_preview.png}           # 放在 assets/img/publication_preview/
}
```

完整字段见 [\_config.yml](_config.yml) 的 `filtered_bibtex_keywords` 列表。

页面入口：[\_pages/publications.md](_pages/publications.md)。

### 4.7 博客文章（Blog）

**目录**：[\_posts/](_posts/)

文件名格式 **必须** 为 `YYYY-MM-DD-slug.md`，例如：

```
_posts/2026-04-21-first-post.md
```

front matter 示例：

```yaml
---
layout: post
title: My First Post
date: 2026-04-21 10:00:00
description: 简短描述
tags: research deep-learning
categories: notes
featured: true # 首页 latest_posts 会突出显示
---
正文 Markdown...
```

### 4.8 News（动态）

**目录**：[\_news/](_news/)

每条一个文件，文件名同样是 `YYYY-MM-DD-slug.md`，内容很短：

```yaml
---
layout: post
date: 2026-04-21
inline: true
related_posts: false
---
Paper accepted at **NeurIPS 2026**! 🎉
```

`inline: true` 会直接把内容显示在 about 页的 news 模块里。

### 4.9 Projects（项目）

**目录**：[\_projects/](_projects/)

每个项目一个 Markdown 文件，front matter：

```yaml
---
layout: page
title: Project Name
description: 一句话介绍
img: assets/img/12.jpg
importance: 1 # 排序，数字越小越靠前
category: research # 分类标签
related_publications: true # 显示相关论文
---
```

页面入口：[\_pages/projects.md](_pages/projects.md)。

### 4.10 Teaching（教学）

**目录**：[\_teachings/](_teachings/) · 入口：[\_pages/teaching.md](_pages/teaching.md)

结构与 projects 类似，按文件建立课程条目即可。

### 4.11 导航栏显隐

在 [\_pages/](_pages/) 下每个 `.md` 的 front matter 中：

```yaml
nav: true # 是否显示在导航栏
nav_order: 2 # 排序
```

把 `nav: true` 改成 `nav: false` 即可隐藏该栏目。

---

## 五、热重载原理与注意事项

al-folio 的 Docker 启动脚本（[bin/entry_point.sh](bin/entry_point.sh)）做了两件事：

1. `jekyll serve --watch --livereload` 监听所有 Jekyll 可感知的文件变化 → 增量重建
2. `inotifywait` **额外**监听 `_config.yml` → 变化时杀掉 Jekyll 并重启

### 哪些改动会自动生效

- `_pages/`、`_posts/`、`_projects/`、`_news/`、`_teachings/`、`_books/` 下的 Markdown
- `_data/` 下的 YAML
- `_bibliography/papers.bib`
- `_sass/` 样式
- `assets/` 下的图片、JS、CSS
- `_includes/`、`_layouts/` 模板

### 哪些改动需要重启容器

| 改动                       | 是否需重启                                         |
| -------------------------- | -------------------------------------------------- |
| `_config.yml`              | **自动重启**（≈30秒）                              |
| `Gemfile` / `Gemfile.lock` | 需 `docker compose up -d --build`                  |
| `Dockerfile`               | 需 `docker compose up -d --build --force-recreate` |
| `docker-compose.yml`       | 需 `docker compose down && docker compose up -d`   |

### 小坑

- 改完后浏览器**硬刷新**（⌘+Shift+R）以绕过缓存
- 图片放进 `assets/img/` 后首次构建慢，后续会缓存为 WebP
- Liquid 模板语法错误会让 Jekyll 终止，看 `docker compose logs` 定位

---

## 六、常见问题排查

### Q1：浏览器打开 404

- 确认 URL 带了 `/al-folio/` 前缀
- 确认容器在跑：`docker ps | grep academic-page`

### Q2：改完文件没生效

```bash
docker compose logs --tail=50
# 若看到 "regenerating..." 说明 Jekyll 正在重建；等几秒后硬刷新
# 若看到 Liquid/YAML 报错，修正对应文件
```

### Q3：端口被占用

```bash
lsof -iTCP:8080 -sTCP:LISTEN
# 找到占用进程后 kill，或改 docker-compose.yml 的端口映射为 "8081:8080"
```

### Q4：Gemfile.lock 冲突 / 依赖问题

进入容器调试：

```bash
docker compose exec jekyll /bin/bash
bundle install
./bin/entry_point.sh
```

### Q5：ImageMagick 相关错误

临时关闭响应式图片（[\_config.yml](_config.yml)）：

```yaml
imagemagick:
  enabled: false
```

### Q6：想重置到干净状态

```bash
docker compose down
rm -rf _site .jekyll-cache
docker compose up -d
```

---

## 七、命令速查

| 目的         | 命令                                            |
| ------------ | ----------------------------------------------- |
| 启动         | `docker compose up -d`                          |
| 看日志       | `docker compose logs -f`                        |
| 暂停         | `docker compose stop`                           |
| 恢复         | `docker compose start`                          |
| 停止+删容器  | `docker compose down`                           |
| 进容器 shell | `docker compose exec jekyll /bin/bash`          |
| 重建镜像     | `docker compose up -d --build`                  |
| 强制全新启动 | `docker compose up -d --build --force-recreate` |
| 看容器状态   | `docker ps --filter name=academic-page`         |
| 清构建缓存   | `rm -rf _site .jekyll-cache`                    |
| 格式化代码   | `npx prettier . --write`                        |

---

## 八、下一步（待办）

- [ ] 替换 `_config.yml` 中的作者信息与 `url`
- [ ] 替换 `assets/img/prof_pic.jpg` 头像
- [ ] 完善 `_data/socials.yml` 社交链接
- [ ] 录入首批 publications 到 `_bibliography/papers.bib`
- [ ] 删除 al-folio 自带的 `.github/workflows/deploy.yml`（改用 Cloudflare Pages）
- [ ] Cloudflare Pages 接入与自定义域名配置
