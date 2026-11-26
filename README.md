# 个人主页维护手册

站点：<https://weifan1226.github.io> · Jekyll 静态站 · push 到 `main` 后由 GitHub Pages 自动部署。

> 一句话：**内容只改 3 个文件**（`_config.yml`、`_pages/about.md`、`_data/navigation.yml`），其它都是主题引擎，不用碰。

---

## 1. 我要改 X → 去哪里改

| 想改的内容 | 文件 | 说明 |
| --- | --- | --- |
| 姓名、一句话简介、邮箱、GitHub / LinkedIn / Google Scholar 等链接 | `_config.yml` → `author` | 留空 = 不显示该图标 |
| 头像图片本身 | `images/avatar.png` | 512×512 PNG，圆形显示；见下方「换头像」 |
| 自我介绍、News、教育、实习、研究经历、论文、奖项 | `_pages/about.md` | 支持 Markdown + HTML 混写 |
| 顶部导航菜单 | `_data/navigation.yml` | 锚点必须对应 `about.md` 里的 `id` |
| 页面最后更新日期 | `_pages/about.md` 最后一行 | 每次更新顺手改 |
| 浏览器标签页图标 (favicon) | `images/favicon*.png` + `_includes/head/custom.html` | 用 favicon 生成工具整包替换 |
| 字体、配色、布局 | `_sass/`、`assets/` | 主题样式，一般不动 |
| 引用数自动抓取 | `.github/workflows/`、`google_scholar_crawler/` | 自动运行，不用改 |

---

## 2. 常见操作

### 换头像（推荐交给 Agent）

直接说：**「用这张图更新我的主页头像并发布」**，会调用 `homepage-update` skill 自动完成格式转换、裁剪、改配置和提交。

手动做也可以（macOS）：

```bash
# 从任意格式图片生成 512×512 方形 PNG，并同步 _config.yml 的路径
~/.workbuddy/skills/homepage-update/scripts/set_avatar.sh /path/to/photo.jpg
```

原图是 JPG/HEIC 都行，脚本会转 PNG、居中裁成正方形（页面用 `border-radius: 50%` 显示圆形，必须是方图）、缩到 512px。改完记得确认 `_config.yml` 里 `author.avatar` 指向 `images/avatar.png`。

### 加一条 News

编辑 `_pages/about.md` 中 `id="news"` 的 `<div>`，**在最上面插一行**（列表按时间倒序）：

```html
- 💼 <strong>[2026.11]</strong> 一句话写清楚发生了什么。<br>
```

### 加论文 / 奖项 / 经历

同一个文件里找到 `## 📄 Selected Publications`、`## 🏆 Selected Awards` 等章节，按现有格式加列表项即可。

### 改导航菜单

`_data/navigation.yml` 每一项是 `title` + `url`。`url` 里的 `#xxx` 要和 `_pages/about.md` 中对应章节的 `<a id="xxx"></a>` 一致；新增章节时两处都要加。首页固定为 `/`。

### 本地预览

```bash
gem install bundler -v 2.2.19 && bundle install   # 首次，需要 Ruby 环境
bash run_server.sh                                # 打开 http://127.0.0.1:4000
```

`_config.yml` 改动不会热重载，改完要重启服务；`about.md` 改动会自动刷新。

---

## 3. 发布

```bash
git add -A && git commit -m "update homepage" && git push origin main
```

push 后 GitHub Pages 自动构建，约 1–2 分钟生效。构建失败去仓库的 **Actions** 页看日志。
本机未安装对应版本的 bundler 时，也可以跳过本地预览直接 push，让 GitHub Pages 构建。

---

## 4. 目录说明（哪些不用动）

```
_config.yml              ← 改：站点信息 / 侧栏
_pages/about.md          ← 改：正文
_data/navigation.yml     ← 改：导航
images/                  ← 改：头像、favicon
_includes/ _layouts/ _sass/ assets/   ← 主题引擎，不动
.github/workflows/ google_scholar_crawler/  ← 自动抓引用数，不动
Gemfile Gemfile.lock run_server.sh          ← 本地构建依赖，不动
.workbuddy/              ← WorkBuddy 记忆，已被 exclude，不会发布
```

Google Scholar 引用数抓取由 Actions 每天 08:00 UTC 运行，结果写到 `google-scholar-stats` 分支；需要在仓库 Secrets 里配置 `GOOGLE_SCHOLAR_ID`。页面里用 `<span class='show_paper_citations' data='论文ID'></span>` 显示单篇引用数。

---

## 5. 用 Agent 快速更新

用户级 skill：`~/.workbuddy/skills/homepage-update/`

```
更新一下我的主页，加一条 news：……
把主页头像换成这张图
把我的实习经历改成……然后发布
```

skill 内置 `set_avatar.sh`（图片标准化）和 `verify.sh`（配置自检），会先校验再提交；**只有明确说了「发布 / push / 上线」才会推送到远端**。

---

## 6. 本次精简记录（可回滚）

删除了：旧头像 `images/me.png`（17.9 MB）、命名错误的重复图 `images/images:android-chrome-512x512.png.png`、未使用的占位图 `images/500x300.png`、上游模板宣传文档 `docs/`；`_config.yml` 中约 355 行注释掉的模板示例已移除。全部通过 `git rm` 删除：本次改动提交前用 `git restore --staged --worktree <路径>` 即可找回；提交之后用 `git restore --source=<提交号> <路径>` 从历史恢复。

主题来源：[AcadHomepage](https://github.com/RayeRen/acad-homepage.github.io)（MIT），基于 minimal-mistakes / academicpages。
