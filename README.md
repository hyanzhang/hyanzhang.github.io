# 个人 GitHub Pages 站点

本仓库是 [hyanzhang.github.io](https://hyanzhang.github.io) 的源码，基于 [al-folio](https://github.com/alshedivat/al-folio) 主题，使用 **Jekyll + Liquid + Markdown + Sass** 构建静态网站。没有后端服务，也不是通过 npm 启动的前端应用；Node.js 主要用于格式化和部署时的 CSS 清理。

目前保留首页、博客、论文、项目、GitHub 仓库、简历、教学和新闻等栏目入口，内容为空的栏目可逐步补充。

## 1. 页面组成与路由

| 页面        | 地址             | 源文件                                              | 布局与内容来源                                                                        |
| ----------- | ---------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------- |
| 首页 / 关于 | `/`              | [\_pages/about.md](./_pages/about.md)               | `about` 布局；正文、头像和个人信息来自本页，站点名称来自配置                          |
| 博客        | `/blog/`         | [\_pages/blog.md](./_pages/blog.md)                 | `default` 布局；读取 `_posts/`，置顶文章、列表、标签入口和分页均在本页生成，每页 5 篇 |
| 论文        | `/publications/` | [\_pages/publications.md](./_pages/publications.md) | `page` 布局；Jekyll Scholar 将 BibTeX 数据渲染为论文列表                              |
| 项目        | `/projects/`     | [\_pages/projects.md](./_pages/projects.md)         | `page` 布局；读取 `_projects/`，支持分类、纵向和横向卡片                              |
| GitHub 仓库 | `/repositories/` | [\_pages/repositories.md](./_pages/repositories.md) | `page` 布局；读取仓库数据，调用用户、仓库统计和奖杯组件                               |
| 简历        | `/cv/`           | [\_pages/cv.md](./_pages/cv.md)                     | `cv` 布局；当前读取 `_data/cv.yml`，可选 JSON Resume 和 PDF 下载                      |
| 教学        | `/teaching/`     | [\_pages/teaching.md](./_pages/teaching.md)         | `page` 布局；直接编辑 Markdown 正文                                                   |
| 新闻        | `/news/`         | [\_pages/news.md](./_pages/news.md)                 | `page` 布局；新闻组件读取 `_news/`，默认不在导航栏显示                                |
| 错误页      | `/404.html`      | [\_pages/404.md](./_pages/404.md)                   | `page` 布局；不存在的地址显示此页，3 秒后返回首页                                     |

文章和集合详情页不需要在 `_pages/` 中逐个创建：

- 博客文章：`_posts/YYYY-MM-DD-slug.md`，默认输出 `/blog/:year/:title/`。
- 项目详情：`_projects/slug.md`，输出 `/projects/:title/`。
- 新闻详情：`_news/slug.md`，输出 `/news/:title/`；`inline: true` 时新闻列表直接显示正文，`false` 时链接到详情页。
- 博客归档：有文章后，`jekyll-archives` 自动生成 `/blog/:year/`、`/blog/tag/:name/` 和 `/blog/category/:name/`。
- `jekyll-feed` 和 `jekyll-sitemap` 自动生成 `/feed.xml` 与 `/sitemap.xml`；[robots.txt](./robots.txt) 声明站点地图地址。

### 导航如何生成

[\_includes/header.liquid](./_includes/header.liquid) 始终提供首页链接，再遍历 `site.pages`，选取 `nav: true` 的页面，按 `nav_order` 排序。页面的 `title` 是导航文字，`permalink` 决定访问地址。

搜索由 [\_includes/scripts/search.liquid](./_includes/scripts/search.liquid) 和本地 Ninja Keys 脚本实现：构建时将导航、文章、集合内容和已配置的社交入口写入搜索数据，无需单独维护搜索索引。右侧还包含明暗主题切换按钮；搜索和主题按钮由站点配置控制。

## 2. 页面渲染与公共组件

普通页面的主要渲染关系为：

```text
Markdown / HTML 正文 + YAML Front Matter
    -> _layouts/about.liquid、page.liquid、post.liquid、cv.liquid 等
    -> _layouts/default.liquid
       -> head.liquid：元信息、字体、图标、样式和主题初始化
       -> header.liquid：导航、搜索按钮、主题切换、阅读进度条
       -> 页面正文：可选左右侧目录
       -> footer.liquid：版权、主题说明和可选更新时间
       -> scripts/*.liquid：按站点配置或页面开关加载脚本
    -> _site/ 中的 HTML、CSS、JavaScript 和静态资源
```

[\_layouts/](./_layouts/) 中的布局职责：

- `default`：普通页面共用的 HTML 外壳。
- `about`：首页标题、个人资料、头像，以及可选新闻、最新文章、精选论文和社交链接。
- `page`：常规内容页；`post`：文章日期、标签分类、目录、引用和评论。
- `cv`：将 YAML 简历或 JSON Resume 数据分发给对应的简历组件。
- `archive-year`、`archive-tag`、`archive-category`：博客归档列表。
- `bib`：单篇论文的作者、摘要、预览图、资源按钮和引用指标，不是独立访问的页面。
- `distill`：独立 HTML 外壳的长文布局，支持作者、脚注和文献。
- `profiles`：多人资料布局；`none`：不添加页面外壳，仅输出正文。当前没有使用这两个布局的栏目。

[\_includes/](./_includes/) 是可复用组件，不会自动生成独立页面：

| 组件                                                    | 用途                                                 |
| ------------------------------------------------------- | ---------------------------------------------------- |
| `metadata`、`head`、`header`、`footer`、`social`        | 页面元信息、公共头尾和社交链接                       |
| `news`、`latest_posts`、`selected_papers`               | 首页或独立页面中的内容列表                           |
| `projects`、`projects_horizontal`、`repository/`        | 项目卡片和 GitHub 展示组件                           |
| `cv/`、`resume/`                                        | 分别渲染 YAML 简历和 JSON Resume 各模块              |
| `figure`、`audio`、`video`                              | 图片、音频、视频嵌入                                 |
| `pagination`、`related_posts`、`citation`、`bib_search` | 分页、相关文章、文章引用和论文搜索                   |
| `giscus`、`disqus`、`scripts/`                          | 评论及数学公式、图表、地图、代码差异、图片缩放等扩展 |

主题能力仍然保留；删除示例内容不等于删除对应的布局、样式或功能。

## 3. 文件组织

```text
.
├── _config.yml          # 站点信息、集合、插件、功能开关和第三方库配置
├── _pages/              # 固定页面及栏目入口
├── _posts/              # 博客文章，当前仅保留 .gitkeep
├── _projects/           # 项目集合，当前仅保留 .gitkeep
├── _news/               # 新闻集合，当前仅保留 .gitkeep
├── _bibliography/       # 论文 BibTeX 数据
├── _data/               # 简历、仓库、作者和期刊等结构化数据
├── _layouts/            # 页面布局
├── _includes/           # 布局共享的 Liquid 组件
├── _plugins/            # 自定义 Ruby 插件、标签和过滤器
├── _sass/               # 主题样式、变量、布局和图标样式
├── assets/              # CSS 入口、JS、图片、字体、音视频等静态资源
├── .github/             # GitHub Actions、Issue 模板和仓库维护配置
├── .devcontainer/       # 开发容器配置
├── bin/                 # 构建、旧版部署和容器启动脚本
├── Gemfile              # Jekyll 与 Ruby 插件依赖
├── Gemfile.lock         # Ruby 依赖锁定文件（当前已被 Git 跟踪）
├── package.json         # Prettier 及 Liquid 格式化插件
├── package-lock.json    # Node.js 开发依赖锁定文件
├── requirements.txt     # Jupyter Notebook 转换所需的 nbconvert
├── Dockerfile           # 本地构建容器镜像
├── docker-compose*.yml  # 普通版和 slim 版开发容器
├── purgecss.config.js   # 部署时清理 _site/ 中未使用的 CSS
├── robots.txt           # 爬虫规则，构建时填充站点地图 URL
├── readme_preview/      # 上游主题文档预览图片，不参与站点发布
├── lighthouse_results/  # 性能报告徽章，不参与站点发布
└── _site/               # 本地构建产物，被 Git 忽略，不应手工编辑
```

其他说明文档：

- [INSTALL.md](./INSTALL.md)：上游安装和部署说明。
- [CUSTOMIZE.md](./CUSTOMIZE.md)：主题功能和定制说明。
- [FAQ.md](./FAQ.md)：常见问题。
- [CONTRIBUTING.md](./CONTRIBUTING.md)：上游贡献指南。
- [README_intro.md](./README_intro.md)：保留的上游主题介绍；社区示例网站目录统一链接到上游项目，具体站点状态以本文及配置为准。
- [LICENSE](./LICENSE)：主题许可证，保留原有授权和署名。

`_site/`、依赖目录和缓存不属于内容源；`_config.yml` 的 `exclude` 还排除了主要开发文件和说明文档。普通 Markdown 文件不会仅因存在就成为页面，站点内容需要 Front Matter 或集合生成规则。

### 数据与样式对应关系

| 文件                                                                                             | 消费位置                      | 当前状态                         |
| ------------------------------------------------------------------------------------------------ | ----------------------------- | -------------------------------- |
| [\_bibliography/papers.bib](./_bibliography/papers.bib)                                          | 论文页、精选论文、文内引用    | 空文献库                         |
| [\_data/coauthors.yml](./_data/coauthors.yml)                                                    | `bib` 布局中的合作者链接      | 空映射                           |
| [\_data/venues.yml](./_data/venues.yml)                                                          | 论文期刊简称、链接及颜色      | 空映射                           |
| [\_data/cv.yml](./_data/cv.yml)                                                                  | `cv` 布局及 `cv/` 组件        | 空列表                           |
| [\_data/repositories.yml](./_data/repositories.yml)                                              | 仓库页及 `repository/` 组件   | 用户和仓库列表均为空             |
| [assets/css/main.scss](./assets/css/main.scss)                                                   | 编译为 `/assets/css/main.css` | 引入 `_sass/` 中的主题和图标样式 |
| [\_sass/\_variables.scss](./_sass/_variables.scss)、[\_sass/\_themes.scss](./_sass/_themes.scss) | 全站样式                      | 基础变量及明暗主题               |
| [assets/js/](./assets/js/)                                                                       | 公共组件及按需功能            | 主题、搜索、交互和第三方脚本     |

简历默认使用 YAML。若需要 JSON Resume，可自行创建 `assets/json/resume.json`，在 `_config.yml` 的 `jekyll_get_json` 中配置 `data: resume` 和对应的 `json` 路径；加载后 `site.data.resume` 会优先于 YAML 简历。当前 `jekyll_get_json: []`，不会再加载 Einstein 的示例简历。

### 自定义插件

[\_plugins/](./_plugins/) 保留以下能力：

- `cache-bust.rb`：为静态资源生成缓存摘要。
- `details.rb`、`file-exists.rb`：提供折叠内容标签及文件存在检查。
- `external-posts.rb`：按 `external_sources` 抓取站外文章并纳入博客；当前来源列表为空。
- `download-3rd-party.rb`：可选下载第三方库并替换资源地址；当前 `third_party_libraries.download: false`，主要使用 CDN。
- `google-scholar-citations.rb`、`inspirehep-citations.rb`：论文引用次数查询。
- `hide-custom-bibtex.rb`、`remove-accents.rb`：过滤展示的 BibTeX 字段，规范化作者姓名。

## 4. 后续如何添加内容

### 站点与首页

先编辑 [\_config.yml](./_config.yml) 中的名称、描述、`url`、`baseurl`、社交账号及功能开关。当前站点部署在域名根目录，`baseurl` 为空。

首页正文和个人资料位于 [\_pages/about.md](./_pages/about.md)；`profile.image` 使用 `assets/img/` 中的图片。已有个人信息保持不变，示例介绍文字已移除。

首页动态区的显示条件：

- 新闻：本页 `news: true`，且站点 `announcements.enabled: true`。
- 最新文章：站点 `latest_posts.enabled: true`。
- 精选论文：本页 `selected_papers: true`，文献条目设置 `selected={true}`。
- 社交链接：本页 `social: true`。

上述动态区目前均关闭。

### 博客、项目与新闻

- 博客：向 [\_posts/](./_posts/) 添加符合日期命名规则的 Markdown 文件，Front Matter 至少填写 `layout: post`、`title` 和 `date`。列表、分页、归档和搜索会随构建更新。博客名称和副标题在 `_config.yml` 中设置。
- 项目：向 [\_projects/](./_projects/) 添加 Markdown 文件，使用 `layout: page`；`title`、`description`、`img`、`importance` 分别控制标题、简介、卡片图片和排序。需要分类时，再配置 `category` 和项目入口页的 `display_categories`；当前分类列表为空。
- 新闻：向 [\_news/](./_news/) 添加 Markdown 文件，设置 `layout: post`、`date`、`inline`；非行内新闻还应设置 `title`。
- 教学：直接在 [\_pages/teaching.md](./_pages/teaching.md) 的 Front Matter 下方编写正文。

### 论文、简历与仓库

- 论文：将真实条目写入文献库。作者突出显示需自行填写 `_config.yml` 中的 `scholar.first_name`、`scholar.last_name`；合作者和期刊数据按需补充。PDF、预览图等资源路径需要同时存在。
- 简历：编辑 YAML 简历数据，支持 `map`、`list`、`time_table`、`nested_list`、`list_groups`；PDF 可放入自行创建的 `assets/pdf/`，再设置简历入口页的 `cv_pdf` 文件名。
- GitHub：填写仓库数据中的 `github_users` 和 `github_repos`；统计卡片由外部服务提供，加载依赖网络。

新增栏目时，在 [\_pages/](./_pages/) 创建带 `layout`、`title`、`permalink` 的页面；需要加入导航时设置 `nav: true` 和 `nav_order`。

## 5. 本地开发与验证

建议使用与部署工作流一致的 Ruby 3.2.2、Bundler 2.5.7，并准备 ImageMagick。启用 Notebook 内容时还需要 Python、Jupyter / nbconvert。依赖以锁定文件为准。

```bash
gem install bundler -v 2.5.7
bundle install
bundle exec jekyll serve --host 0.0.0.0 --port 8081
```

访问 `http://localhost:8081`。修改 `_config.yml` 后需要重启原生 Jekyll 开发服务。

生产构建：

```bash
JEKYLL_ENV=production bundle exec jekyll build
```

输出写入 `_site/`。删除内容后应使用完整构建而非仅依赖增量构建，检查首页、导航、栏目、搜索及归档是否正确。

格式化使用现有 Prettier 配置；`package.json` 没有定义 `npm run dev/build/test`：

```bash
npm ci
npx prettier README.md _config.yml "_pages/**/*.md" "_data/**/*.yml" --check
```

容器方案见 [docker-compose.yml](./docker-compose.yml) 和 [docker-compose-slim.yml](./docker-compose-slim.yml)。注意：Compose 当前映射容器端口 `8080`，而本仓库 [bin/entry_point.sh](./bin/entry_point.sh) 启动的是 `8081`；若通过本地 Dockerfile 构建，需先统一映射与监听端口。不要把此差异误认为页面内容错误。

## 6. GitHub Pages 发布流程

主流程是 [.github/workflows/deploy.yml](./.github/workflows/deploy.yml)：

1. `main` / `master` 上相关文件的 push、pull request，或手动触发工作流。
2. 在 Ubuntu 22.04 runner 上检出源码，配置 Ruby 3.2.2 和 Bundler 依赖缓存。
3. 仅在工作流工作区中将 `giscus.repo` 设置为当前仓库，不提交该修改。
4. 安装 ImageMagick、Jupyter，使用生产环境执行 Jekyll 构建。
5. 根据 [purgecss.config.js](./purgecss.config.js) 扫描生成的 HTML / JS，清理未使用的 CSS。
6. 非 pull request 构建通过 GitHub Pages Deploy Action 发布 `_site/` 到发布分支（默认 `gh-pages`）；pull request 只验证构建。

部署、站内链接和 Axe 工作流固定使用 `ubuntu-22.04`，以匹配 Ruby 3.2.2 的官方预构建包，避免 `ubuntu-latest` 升级后找不到对应的 Ruby 安装包。

这套工作流发布的是已构建静态文件。GitHub 仓库 Pages 设置应与发布分支及根目录对应，不应依赖 GitHub Pages 的默认 Jekyll 构建来执行这些自定义插件。只修改本 README 不会自动触发部署。

其他工作流包括 Prettier 检查、源码链接与构建产物断链检查、Axe 无障碍检查、Lighthouse 性能徽章，以及 Docker 镜像构建与精简。部分主题维护流程依赖额外仓库权限或 Secrets，不属于页面正常渲染所必需的步骤。

[bin/cibuild](./bin/cibuild) 是简单的 Jekyll 构建入口。[bin/deploy](./bin/deploy) 是旧的手动部署脚本，会切换分支、清理文件并可能强制推送；日常发布优先使用 Actions，不要在有未保存工作的目录中直接运行它。

## 7. 本次示例清理范围

按“保留栏目入口、清空示例内容”的方式整理：

- 删除 29 篇功能演示文章、9 个示例项目、3 条示例新闻，保留集合目录以便继续写作。
- 删除重复的 Einstein 介绍文件、纯演示用下拉菜单，以及相对论和 Plotly 独立示例 HTML。
- 清空 Einstein 文献、简历、合作者、期刊及第三方 GitHub 展示数据；关闭示例 JSON 简历加载，移除示例 Notebook、表格数据、Distill 文献和 PDF。
- 清除首页与栏目中的占位介绍，以及示例博客标签、分类、作者匹配、Inspire HEP 账号和 Disqus 标识，避免出现演示身份或指向已删除归档的链接。
- 保留已有个人配置、栏目路由、主题布局、组件、样式、插件及其他静态素材。尚未填写内容的栏目为空，这是当前预期状态。
