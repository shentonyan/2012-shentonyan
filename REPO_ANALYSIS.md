# 仓库结构与内容全面分析

> 本文档对 `2012-shentonyan` WordPress 博客主题仓库进行通盘解读，帮助博主设计自己的博客。

---

## 一、仓库总览

| 属性 | 值 |
|------|-----|
| 主题名称 | 2012-shentonyan |
| 基础 | WordPress 官方默认主题 Twenty Twelve |
| 传承链 | Twenty Twelve → 2012_mtr（木头人版）→ 2012-huhexian → 2012-shentonyan |
| 版本 | 1.0.2.4 |
| PHP 最低要求 | 5.2.4 |
| WordPress 最低要求 | 3.5 |
| 测试至 | WordPress 6.1 |
| 许可证 | GNU GPL v2 |

---

## 二、文件结构树

```
2012-shentonyan/
│
├── 【主模板层 - WordPress 核心模板文件】
│   ├── index.php          首页文章列表
│   ├── single.php         单篇文章页
│   ├── page.php           独立页面（普通页）
│   ├── archive.php        分类 / 标签 / 时间归档
│   ├── search.php         搜索结果页
│   ├── 404.php            404 错误页
│   ├── image.php          图片附件页
│   └── comments.php       评论区模板
│
├── 【内容片段 - get_template_part() 调用】
│   ├── content.php        通用文章内容（列表 + 单篇）
│   ├── content-page.php   独立页面内容
│   └── content-none.php   无文章时显示
│
├── 【页面片段 - 头尾侧栏】
│   ├── header.php         页头（<head>、导航、Logo）
│   ├── footer.php         页脚（版权、快捷按钮、统计脚本）
│   └── sidebar.php        侧边栏（多区域动态小工具）
│
├── 【特殊功能页】
│   ├── r.php              外链跳转中转页（防采集 / 隐藏真实 URL）
│   └── searchform.php     搜索表单片段
│
├── 【主题逻辑 - PHP】
│   ├── functions.php      核心函数库（~2500 行，见下节详解）
│   ├── options.php        主题选项定义（OptionsFramework 配置项）
│   ├── postviews.php      文章浏览数统计
│   ├── quotes.php         随机格言功能
│   ├── widgets.php        自定义小工具类
│   ├── widget-websitestat.php  网站统计小工具
│   └── seo.php            SEO 元标签（title / description / og / twitter）
│
├── 【页面模板 page-templates/】
│   ├── template-archives.php   文章归档页（按年月折叠展示）
│   ├── template-links.php      友情链接页（从 WordPress Links Manager 读取）
│   └── template-readers.php    读者墙页（按评论数排名）
│
├── 【主题选项框架 options/】
│   ├── options-framework.php          入口
│   ├── includes/
│   │   ├── class-options-framework.php      核心类
│   │   ├── class-options-framework-admin.php 后台页面渲染
│   │   ├── class-options-interface.php       表单控件
│   │   ├── class-options-media-uploader.php  媒体上传
│   │   └── class-options-sanitization.php    数据净化
│   ├── css/  optionsframework.css / wp53.css
│   └── js/   media-uploader.js / options-custom.js
│
├── 【JavaScript js/】
│   ├── script.js          主脚本（滚动 / 折叠 / 目录 / 记住评论者 / 反镜像）
│   ├── ajax-comment.js    Ajax 异步提交评论
│   ├── fancybox.js        图片灯箱（FancyBox v3）
│   ├── clipboard.min.js   复制到剪贴板
│   └── navigation.js      移动端菜单展开/收起
│
├── 【字体 font/】
│   └── fontello.*（eot/svg/ttf/woff/woff2）  图标字体（含上/下箭头、目录、评论、暗黑模式等图标）
│
├── 【图片资源 images/】
│   ├── favicon.ico / favicon-*.png / apple-touch-icon.png  多尺寸图标
│   ├── android-chrome-*.png / mstile-150x150.png           平台图标
│   ├── default_first_img.png   文章无图时的默认 OG 封面
│   ├── post-secret-qr-120.png  加密内容公众号二维码占位图
│   ├── safari-pinned-tab.svg
│   ├── browserconfig.xml       Edge/IE 磁贴配置
│   └── site.webmanifest        PWA Manifest
│
├── style.css              主题注册信息 + 全部 CSS 样式（~77KB）
├── screenshot.png         主题预览截图
├── readme.txt             WordPress.org 标准自述
├── Readme.md              GitHub 用途说明
├── LICENSE                GPL v2 许可证
└── .gitattributes         Git 行尾换行配置
```

---

## 三、核心模板逻辑解析

### 3.1 页面渲染流程

```
WordPress 路由
    │
    ├── 首页          → index.php    → get_header() / content.php × N / sidebar / footer
    ├── 单篇文章      → single.php   → content.php / nav-single / sidebar-single / comments
    ├── 独立页面      → page.php     → content-page.php / comments
    ├── 归档分类标签  → archive.php  → content.php 列表
    ├── 搜索          → search.php
    ├── 404           → 404.php
    └── 特殊页面模板  → page-templates/*.php（归档/友链/读者墙）
```

### 3.2 header.php 关键设计

- `<head>` 内引入 **LXGW WenKai Screen**（霞鹜文楷屏幕版）CDN 中文字体
- 引入 `seo.php`（动态生成所有 SEO 标签）
- 夜间模式 JS 内联注入（根据 `darkmode_time` 选项，默认 21:00—06:00 自动切换）
- 支持 Google Ads JS 代码在 `</head>` 前输出一次
- Logo 区：站点名称 + 站点描述 + 可配置的站点右侧短语（`site_title_quote`）
- 导航：`wp_nav_menu` 主导航 + 自定义 header 图片

### 3.3 footer.php 关键设计

- 版权信息行（动态年份）
- 网站地图链接
- 浮动快捷工具栏 `#scroll`：返回顶部、滚到底部、文章目录、跳到评论、夜间模式切换
- FastCGI 缓存模式下的 JS 异步更新浏览数
- `ClipboardJS` 初始化
- 反镜像脚本
- Google Analytics / 统计代码输出
- `instant.page` 预加载脚本

---

## 四、functions.php 功能模块一览（全文约 2500 行）

| 行号范围 | 功能模块 |
|----------|---------|
| 1–39 | OptionsFramework 引入；子文件 require |
| 48–130 | `twentytwelve_setup()`：主题支持特性注册；侧边栏 5 个区域注册 |
| 132–173 | WP Cache 封装（区分登录/访客） |
| 175–218 | 去 `<br>` 保留 `<p>`；禁止恶意 UA（BOT/PHP 爬虫）|
| 219–263 | 脚本样式加载：jQuery / style.css / navigation.js / clipboard / script.js / ajax-comment.js / fancybox |
| 265–298 | 提取文章第一张图片；裸域提取；Gravatar 镜像替换 |
| 300–449 | 面包屑导航 `the_crumbs()`（支持文章/页面/分类/标签/日期/作者归档）|
| 451–502 | 短代码：`[s]`折叠展开、`[p]`段落、`[login]`登录可见、`[password]`加密内容 |
| 503–518 | 评论图片地址自动转 img 标签 |
| 520–572 | 垃圾评论过滤：名称黑名单、Email 黑名单、必须含中文、禁止日文 |
| 574–655 | 评论链接逻辑：管理员/作者/注册用户/外链区分；评论等级 LV1–LV7；友链认证徽章 |
| 657–713 | 文章/评论外链跳转中转（通过 `r.php`）|
| 717–760 | 后台快速按钮工具条（链接/代码/颜色/折叠/加密/灯箱/相册等 20+ 按钮）|
| 764–797 | FancyBox 图片灯箱自动包装；图片 alt/title 自动填充 |
| 811–864 | 文章归档函数 `zww_archives_list()`（按年月折叠，支持评论数/浏览数）|
| 871–921 | 读者墙 `allreaders_cy()`（按评论总数排名）|
| 943–997 | 最近活跃访客 `get_active_friends()`（近 30 天头像墙）|
| 1000–1048 | 最近更新文章 `recently_updated_posts()`（更新与发布相差 N 天）|
| 1050–1132 | Ajax 评论回调（`fa_ajax_comment_callback`）|
| 1135–1180 | 短代码：`[theme_insert_content_block]` 插入预设段落；`[tags_posts]` 指定标签文章列表 |
| 1182–1234 | 新文章自动用 ID 作 slug；禁用自动保存；禁止修订；文章 ID 连续性维护 |
| 1236–1274 | 自动勾选"记住我"；延长登录 Cookie 至 30 天；评论回复邮件通知（HTML 格式）|
| 1276–1288 | SMTP 邮件发送配置 |
| 1290–1301 | 评论链接去 replytocom 参数；评论嵌套层数扩展至 444 层 |
| 1305–1398 | 评论回调 `twentytwelve_comment()`（楼层数、头像、时间、回复按钮）|
| 1400–1470 | 相关文章 `Theme_Related_Posts()`（同分类→同标签补全）；友情链接 flex 布局输出 |
| 1472–1495 | 系统信息输出：数据库查询次数/耗时/内存/VPS 运行时长 |
| 1496–1572 | **文章目录 TOC**（自动解析 h2–h4，生成浮动目录面板）|
| 1574–1592 | 评论跳转链接 nofollow；后台按分类 ID 列表 |
| 1596–1647 | 搜索/归档页隐藏私密文章；后台管理工具栏自定义菜单 |
| 1648–1700 | 后台/前台分离 favicon；标签云字体大小；标签新标签页打开；归档标题前缀 |
| 1703–1743 | 彻底禁用 Feed（可选）；禁用 Google Open Sans；sitemap lastmod/changefreq/priority 注入 |
| 1746–1760 | 后台仪表盘小工具（IP/最近更新/Site 搜索快捷链接）|
| 1762–1825 | 去除静态资源版本号；禁用 Emoji；禁用管理员工具条（非管理员）；后台 CSS |
| 1837–1904 | 清理 `<head>` 冗余代码；禁止 pingback；禁止评论超链接；禁用 s.w.org DNS 预取 |
| 1906–1966 | 彻底关闭 WordPress 自动更新及后台更新检查；禁用 oEmbed |
| 1968–1999 | 禁用 Gutenberg；移除 global-styles；禁用 Site Health 检测项 |
| 2022–2100 | AMP 图片 JSON-LD 修正；Google 广告插入（第1段后/最后一段前）|
| 2100+ | 全站字数统计 `allwords()`（匹配书籍等价）；`zm_count_words()` 文章字数；IP 获取 |

---

## 五、主题选项配置项（options.php）

主题提供**后台"主题选项"页**（依赖 OptionsFramework），分两大组：

### 5.1 基本设置

| 选项 ID | 类型 | 说明 |
|---------|------|------|
| `home_description` | textarea | 首页 SEO 描述 |
| `home_keywords` | textarea | 首页 SEO 关键词 |
| `darkmode` | checkbox | 开启夜间模式 |
| `darkmode_time` | text | 夜间模式时段，默认 `21,6` |
| `post-secret-code` | text | 加密内容统一密码 |
| `post-secret-name` | text | 公众号名称（密码发放渠道）|
| `post-secret-qrcode` | text | 公众号二维码图片地址 |
| `site_title_quote` | textarea | 站点名称右侧短语（约 20 字）|
| `content_block_text` | textarea | 预设内容段落（`\|` 分隔，短代码调用）|
| `home_exclude_cat` | textarea | 首页排除分类 ID（逗号分隔）|
| `home_exclude_tag` | textarea | 首页排除标签 ID |
| `home_set_cat` | text | 首页第1篇文章下插播分类 ID |
| `post_views_fastcgi_cache` | checkbox | FastCGI 缓存模式下 JS 更新浏览数 |
| `post_views_guest_off` | checkbox | 访客不显示浏览数 |
| `feed_rss_enable` | checkbox | 禁用 RSS Feed |
| `custom_favicon` | textarea | 自定义 Favicon HTML 代码 |
| `custom_favicon_admin` | textarea | 后台专用 Favicon |
| `custom_site_icon_hook` | textarea | WordPress 站点图标 URL |
| `analyticscode` | textarea | 统计代码（Google Analytics 等）|
| `copyright` | text | 建站日期，用于版权行 |
| `footerinfo_first` | editor | 页脚第一行附加信息 |
| `footerinfo` | editor | 页脚第二行信息 |
| `article_info_head` | editor | 文章第1段后插入内容 |
| `article_info_foot` | editor | 文章末尾插入内容 |
| `cn_avatar_url` | textarea | Gravatar 镜像源（默认 `cravatar.cn`）|
| `comments_name_blacklist` | textarea | 昵称黑名单 |
| `comments_email_blacklist` | textarea | Email 黑名单 |
| `text_ctfile_replace` | textarea | 文章内容关键词替换（`旧->新`）|
| `text_content_replace` | textarea | 评论内容关键词替换 |

### 5.2 广告管理

| 选项 ID | 说明 |
|---------|------|
| `google_ads_enable` | 启用/禁用 Google 广告 |
| `google_ads_js_code` | Google AdSense JS 代码 |
| `google_ec_post_page_ids` | 排除广告的文章/页面 ID |
| `google_ad_info_index` | 首页第1篇文章下广告 |
| `google_ad_info_index_paged` | 首页翻页后文章列表前广告 |
| `google_ads_single_first` | 文章第1段后广告 |
| `google_ads_single_last` | 文章最后一段前广告 |
| `archive_ad_info` | 分类/标签页文章前广告 |
| `commentform_ad_info` | 评论框广告 |

---

## 六、侧边栏区域（Sidebar）

主题注册了 **5 个独立侧边栏区域**，分工明确：

| 区域 ID | 显示位置 | 用途建议 |
|---------|----------|---------|
| `sidebar-top` | 所有页面顶部 | 全局公告 / 搜索框 |
| `sidebar-home-t` | 仅首页 | 推荐文章 / 近期热门 |
| `sidebar-single-t` | 仅文章/页面 | 文章目录 / 相关推荐 |
| `sidebar-all` | 所有页面底部 | 标签云 / 归档 / 友链 |
| `sidebar-single` | 正文底部 | 打赏 / 关注 / 相关文章 |

---

## 七、自定义小工具（widgets.php）

| 小工具类 | 名称 | 功能 |
|---------|------|------|
| `recently_updated_posts` | 最近更新文章·可指定分类 | 显示距发布满 N 天后被更新的文章（去旧闻） |
| `related_post` | 相关文章 | 同分类+同标签取相关 |
| `readers` | 读者墙 | 最近 N 天活跃访客头像（Gravatar）|
| `active_friends` | 最近活跃读者 | 按评论数排序的头像墙 |
| *(websitestat)* | 网站统计 | 文章数/分类数/评论数/浏览数汇总 |

---

## 八、页面模板（page-templates/）

### template-archives.php（文章归档）
- 按年 → 月折叠展示全部文章
- 显示站点统计（文章数/分类/标签/评论/浏览/最近更新时间）
- 月份标题可点击展开/折叠，顶部有"全部展开/收缩"按钮

### template-links.php（友情链接）
- 从 WordPress **Links Manager** 读取链接
- 支持两种展示：有 `link_image` 则显示自定义图片；否则显示 Gravatar（以 `link_notes` 字段中的 Email 取头像）
- 随机顺序输出

### template-readers.php（读者墙）
- 调用 `allreaders_cy()` 显示按评论总数排名的所有读者名单（文字列表版）

---

## 九、JavaScript 功能细节（js/script.js 约 83KB）

| 功能 | 说明 |
|------|------|
| 滚动导航 | 返回顶部 / 滚到底部 / 滚到评论区 |
| 文章目录浮动面板 | 联动 PHP 端生成的 TOC，支持展开/收起 |
| 文字折叠展开 | `[s][p]...[/p]` 短代码对应的 JS 展开效果 |
| 记住评论者信息 | 勾选"记住我"后将昵称/邮箱/网址写入 localStorage，下次自动填入 |
| 反镜像检测 | 检测域名是否匹配，不匹配则跳回真实站点 |
| 图片灯箱 | FancyBox 初始化，点击文章图片放大 |
| 复制到剪贴板 | 优惠码/代码一键复制 |
| 移动端菜单 | 汉堡菜单展开收起 |

---

## 十、SEO 设计（seo.php）

| 场景 | 输出的 Meta 标签 |
|------|----------------|
| 首页 | description / og:description / keywords / og:url / og:image / twitter:card/site/creator/image |
| 文章/页面 | description（摘要→正文第一段→前138字）/ keywords（标签）/ og:title / og:url / og:image（第一张图）/ twitter:card / bytedance 时间戳 |
| 分类页 | description / og:description / keywords |
| 标签页 | description / og:description / keywords |
| 评论分页 | robots noindex,nofollow |
| 所有页 | og:site_name / og:type / og:author / Favicon 多尺寸 |

---

## 十一、安全与防采集机制

| 机制 | 实现方式 |
|------|---------|
| 禁止恶意 UA | `deny_mirrored_request()`：匹配 BOT/PHP 等 UA 直接 die |
| 垃圾评论过滤 | 必须含中文、禁止日文、昵称/Email 黑名单、WordPress 内置黑名单 |
| 外链跳转 | 评论外链经 `r.php` 中转（Base64URL 编码），防止直接暴露真实 URL |
| 反镜像 | 文末注入隐藏 img onerror JS，检测当前域名，非白名单则跳回 |
| 文末版权声明 | 文章内容末尾自动追加"本文首发于：文章标题-站点名"链接 |
| 禁用 pingback | 通过 `xmlrpc_methods` filter 返回 false |
| 禁用自动更新 | 全面 remove_action 去掉 WordPress 版本/插件/主题检查 |
| 禁用 Gutenberg | 防止 Block Editor 引入额外 JS/CSS |
| 禁用 oEmbed | 防止 iframe 嵌入注入 |

---

## 十二、夜间模式设计

- 在 `<head>` 内联一段 JS（最小化）
- 读取 `localStorage` 中用户手动切换的记录及时间戳
- 与服务器端配置的起止时间比较，决定当前应用 `light` 还是 `dark` 类名到 `<html>` 元素
- CSS 通过 `:root` 和 `.dark` 两套 CSS 变量切换颜色方案
- 页脚浮动按钮（太阳/月亮图标）触发 `toggleCustomDarkMode()` 手动切换，并写入 localStorage

---

## 十三、字体与图标

| 资源 | 来源 | 用途 |
|------|------|------|
| LXGW WenKai Screen | npm.elemecdn.com CDN | 全站中文正文字体 |
| fontello | 本地 font/ 目录 | 图标字体（上下箭头、列表、评论、剪刀等）|

---

## 十四、性能优化手段

| 优化 | 说明 |
|------|------|
| WP Object Cache 封装 | 区分登录/访客，相关文章/归档/读者墙均缓存 1 天–6 小时 |
| FastCGI Cache 支持 | 开启后浏览数通过前端 Ajax + Cookie 更新，不破坏页面缓存 |
| 去除静态资源版本号 | `remove_cssjs_ver()` |
| 禁用 Emoji | 移除 WordPress 内置 Emoji JS/CSS |
| 禁用 Google Open Sans | 后台禁用，加速 Dashboard 加载 |
| 禁用 oEmbed | 去掉 iframe 嵌入支持 |
| instant.page | 页脚引入鼠标悬停预加载脚本 |
| CSS filemtime 缓存破坏 | 每次修改 style.css 自动更新版本 |

---

## 十五、评论系统

| 功能 | 实现 |
|------|------|
| Ajax 无刷新提交 | `ajax-comment.js` + `fa_ajax_comment_callback()` |
| Ctrl+Enter 提交 | `comments.php` 内联 JS |
| 楼层编号 | `twentytwelve_comment()` 回调中维护全局计数器 |
| 头像 | Gravatar + 镜像（默认 `cravatar.cn`）|
| 评论等级徽章 | 累计评论数 10/20/40/80/160/320+ 对应 LV1–LV6 |
| 友链认证徽章 | 评论者 URL 在 Links Manager 指定分类（ID 480）中则显示❤图标 |
| 回复邮件通知 | HTML 格式邮件，通过 SMTP 发送（需配置账号密码）|
| 评论回复 | 嵌套最深 444 层（实际受主题 CSS 约束）|

---

## 十六、短代码速查

| 短代码 | 用途 | 示例 |
|--------|------|------|
| `[s]` | 折叠展开入口 | `[s][p]内容[/p]` |
| `[p]...[/p]` | 折叠段落内容 | 配合 `[s]` 使用 |
| `[login]...[/login]` | 登录可见 | 仅登录用户显示 |
| `[password key="xxx" tips="关键词"]...[/password]` | 密码查看 | 输入密码或通过公众号获取 |
| `[addbr]...[/addbr]` | 引用块+换行 | blockquote 包裹 |
| `[tags_posts tags=1,2]` | 指定标签文章列表 | 输出带链接的 `<ul>` |
| `[theme_insert_content_block ids=1]` | 插入预设段落 | 从主题选项中取第 N 段 |

---

## 十七、如何定制你的博客——设计建议

基于以上分析，以下是针对个人博客设计的关键建议：

### ✅ 必改项（影响展示效果）

1. **头像/Logo**：`header.php` 中可放自己的头像图片（目前已移除原硬编码图片，可按需添加）
2. **字体**：`header.php` 末尾修改 CDN 字体 URL，或替换为本地字体
3. **主题选项**：登录 WordPress 后台 →「主题选项」，填写首页描述、站点短语、Gravatar 镜像等
4. **侧边栏**：后台「外观→小工具」配置 5 个区域的内容

### ✅ 推荐配置项

| 选项 | 建议值 |
|------|--------|
| `darkmode` | 开启，时段 `21,6` |
| `cn_avatar_url` | `cravatar.cn`（国内访问更快）|
| `post_views_fastcgi_cache` | 使用 Nginx FastCGI 缓存时开启 |
| `feed_rss_enable` | 根据需求决定是否保留 RSS |
| `analyticscode` | 填入统计代码（Umami / Google Analytics / 51La 等）|
| SMTP 配置（functions.php 约 1278 行）| 填写真实邮箱账号密码，启用评论回复通知 |

### ✅ 可选扩展页面

通过「页面→新增」并选择以下模板创建特色页面：

- **文章归档**：`template-archives.php`  → 按年月折叠的全站文章目录
- **友情链接**：`template-links.php` → 从 WordPress Links 管理器自动读取
- **读者墙**：`template-readers.php` → 展示历史评论者排名

### ✅ 安全配置建议

- 修改 `functions.php` L1279 处的 SMTP 密码（不要提交到 Git）
- 根据自己的域名更新 `theme_deny_mirrored_websites()` 中的白名单域名
- 定期更新 `comments_name_blacklist` 和 `comments_email_blacklist` 黑名单

---

## 十八、上游传承关系参考

```
Twenty Twelve（WordPress 官方）
    ↓ xuv.cc 修改
2012_mtr（木头人版）
    ↓ huhexian 修改
2012-huhexian（印记版，https://github.com/huhexian/2012-huhexian）
    ↓ 本仓库修改
2012-shentonyan（当前版本）
```

---

*文档生成时间：2026-03-04*
