# 个人主页维护手册

站点：<https://weifan1226.github.io> · Jekyll 静态站 · push 到 `main` 后由 GitHub Pages 自动部署。

> 一句话：**内容只改 3 个文件**（`_config.yml`、`_pages/about.md`、`_data/navigation.yml`），其它都是主题引擎，不用碰。

---

## 1. 我要改 X → 去哪里改

| 想改的内容 | 文件 | 说明 |
| --- | --- | --- |
| 姓名、一句话简介、邮箱、GitHub / LinkedIn / Google Scholar 等链接 | `_config.yml` → `author` | 留空 = 不显示该图标 |
| 头像图片本身 | `images/avatar.png` | 512×512 PNG，圆形显示；见下方「换头像」 |
| 自我介绍、News、教育、实习、研究经历、论文、奖项 | `_pages/about.md` | Markdown + HTML 混写；论文是卡片式 HTML |
| 论文卡片的样式 | `_includes/head/custom.html` 末尾 `<style>` 块 | `.pub-item` 等样式，非必要不动 |
| 顶部导航菜单 | `_data/navigation.yml` | 当前只保留 About Me；新增项时锚点要对应正文 |
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

原图是 JPG/HEIC 都行，脚本会转 PNG、居中裁成正方形（页面用 `border-radius: 50%` 显示圆形，必须是方图）、缩到 512px。换完后仓库里只保留一个头像文件 `images/avatar.png`，`_config.yml` 的 `author.avatar` 也指向它。

### 加一条 News

在 `_pages/about.md` 的 **`## 📰 News` 标题下方那个可滚动 `<div>` 列表**最上面插一行（时间倒序）：

```html
- 💼 <strong>[2026.11]</strong> 一句话写清楚发生了什么。<br>
```

### 加论文 / 奖项 / 经历

- 论文区是卡片：复制一整个 `<div class="pub-item">` 块，替换标题、作者、会议徽标、Paper/Code 链接；配图放到 `images/` 并在 `<img src="images/xxx.png">` 引用。样式定义在 `_includes/head/custom.html` 末尾。
- 奖项 / 教育 / 实习 / 研究经历是普通 Markdown 列表，照现有格式加一行即可。
- 章节标题下面有 `{: #news }`、`{: #publications }` 这类 kramdown 锚点，新增章节时照着加一个，供导航使用。

### 改导航菜单

`_data/navigation.yml` 每一项是 `title` + `url`，目前只有 About Me。新增时 `url` 里的 `#xxx` 要和 `_pages/about.md` 中对应的 `<div id="xxx"></div>` 或标题下的 `{: #xxx }` 一致，两处都要改。

### 本地预览

```bash
gem install bundler -v 2.2.19 && bundle install   # 首次，需要 Ruby 环境
bash run_server.sh                                # 打开 http://127.0.0.1:4000
```

`_config.yml` 改动不会热重载，改完要重启服务；`about.md` 改动会自动刷新。

---

## 3. 发布

**先同步远端再提交** —— 本地克隆可能落后 GitHub 好几个月（网页端或别的机器改过），直接 push 会被拒绝：

```bash
git fetch origin main          # 看一眼 origin/main 有没有新提交
git status
git add -A && git commit -m "update homepage"
git push origin main
```

若提示 `fetch first`，用 `git merge origin/main` 合并后再 push，**不要强推、不要用本地旧版本覆盖线上内容**。push 后 GitHub Pages 自动构建，约 1–2 分钟生效；失败去仓库 **Actions** 页看日志。本机缺 bundler 时可以跳过本地预览，直接交给云端构建。

---

## 4. 目录说明（哪些不用动）

```
_config.yml              ← 改：站点信息 / 侧栏
_pages/about.md          ← 改：正文（论文卡片也在这里）
_data/navigation.yml     ← 改：导航
images/                  ← 改：avatar.png、jailbreaklora.png、favicon
_includes/ _layouts/ _sass/ assets/   ← 主题引擎 + 论文卡片 CSS，不动
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

skill 内置 `set_avatar.sh`（图片标准化）和 `verify.py`（提交前自检），会先校验再提交；**只有明确说了「发布 / push / 上线」才会推送到远端**。

---

## 6. 本次整理记录（可回滚）

- 换头像：旧的 `images/me512.png`（白天树荫照）已删除，改为 `images/avatar.png`（维港夜景，512×512 PNG）；`_config.yml` 中原本指向不存在的 `images/me512.png` 的配置已修正。
- 精简：删除旧 `images/me.png`（17.9 MB）、命名错误的重复图、未使用的占位图、上游模板宣传文档 `docs/`；`_config.yml` 520 → 122 行（约 355 行注释示例移除）；仓库 42 MB → 25 MB。
- 合并：远端 2026-06 的改版（论文卡片 + 配套 CSS + News 标题 + IAL 锚点 + 简化导航）全部保留，只叠加头像替换与结构性精简。
- 修复：清掉两个指向不存在文件的 meta，修正 favicon manifest 图标路径，`.gitignore` 与 Jekyll exclude 都排除 `.workbuddy/`。
- 所有删除都通过 `git rm`：本次改动提交前用 `git restore --staged --worktree <路径>` 找回；提交之后用 `git restore --source=<提交号> <路径>` 从历史恢复。

主题来源：[AcadHomepage](https://github.com/RayeRen/acad-homepage.github.io)（MIT），基于 minimal-mistakes / academicpages。
