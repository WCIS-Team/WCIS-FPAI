# CONTRIBUTING.md

> 在本仓写文档、改脚本、发站点的协作规范。范围：**文档库 `wiki/`（＝Obsidian vault）里的页面**，以及与之绑定的校验、行尾、站点、推送流程。
> 干活纪律与 `#SOP` 约定见 `AGENTS.md`；结构见 `ARCHITECTURE.md`；索引见 `llms.txt`。

skills/ste-word-gate 只在写用户手册时使用

## 一、页形（每页必须满足）

- **一页一个 H1，且与文件名一致**。唯一例外是首页 `wiki/README.md`（H1 是项目标题）。
- `## 相关页面` **可选**（2026-11-02 起）：写它是给读者一句"接着看哪几页"，不写不扣分、校验器也不报。**别当索引抄**——正文里已经链过一遍的页不必在页脚再列一次（2026-11-02 就是这么清掉 15 页纯重复页脚的）。
- 页内链接一律 `[[页面名]]`——**只用文件名，不带路径、不带 `.md` 后缀**。因此页面名在整个文档库内必须唯一（自 2026-10-18 起文档库只有一棵树 `wiki/`），重名会让双链和关系图失效。
- 文件用 **LF、UTF-8、不带 BOM**。

## 二、链接与标题

- **跨页链接要锚到小节，显示文字写成「页面名——小节名」**（D-1790553604）：
  `[[页面名#小节标题|页面名——小节标题]]`——不点也知道落在哪一页哪一节。
  - `D-nn` 那类小节只显编号（`状态源与决策记录——D-1790553603`），标题后半截的副标题塞进正文会把句子撑断。
  - 链接后面**原本又重复一遍的小节号**（`§3.3`、`第 5 节`）删掉；指到**另一节**或「某一条」的指针保留。
  - 唯一例外：D-1790553603 的凭据链接显示仍是 `登录凭证`／`那一处`，那几条显示文字本身就是凭据纪律的一部分。
  - 表格单元格里的分隔符必须转义成 `\|`，正文里用普通 `|`。
  - `check-md.py` 会拒绝任何指不到真实标题的锚点——改标题会立刻在这里暴露，不会留下静默失效的链接。
- **标题要能当锚点就得干净**：`##` 行里不写加粗、反引号，也不嵌套 `[[链接]]`；补充说明放到标题下面一行正文。
- **Mermaid 里做可点击节点**：节点标签必须**逐字等于**页面名，并把该节点 id 加进 `class ... internal-link`。`check-md.py` 会检查这两条。
- Mermaid 代码块**不写** `%%{init: ... "defaultRenderer": "elk" ...}%%`，排布交给默认渲染器；`classDef`／`class` 可以保留。
- 只有 Obsidian 里 Mermaid 节点可点击进入页面（节点文字＝页面名，由 `class <id> internal-link` 生效）；GitHub 与 VS Code 预览只画图不响应点击。

## 三、新增页面的流程

1. 先想清楚它属于哪一层（`PL`／`PS`／`SOP`／`man`／`archive`…），放进对应文件夹。
2. 在 `wiki/README.md` 的「目录」表里加一处——表负责跨端可达，Obsidian 的关系图自动覆盖。
3. 页形按第一、二节写（H1＝文件名、双链只写页名；`## 相关页面` 可选）。
4. 跑校验（第四节）。

## 四、校验

两个脚本都在 `wiki/format/`，根目录参数都有默认值——默认扫**实际存在**的文档树，所以不带参数就是全库检查；命令行显式给的根必须存在，写错路径直接 `rc=2` 报错。

```bash
python -X utf8 wiki/format/check-md.py            # 文档结构（默认扫 wiki/）
python -X utf8 wiki/format/check-md.py <目录> …    # 只查指定目录
python -X utf8 wiki/format/progress.py            # 由总账表回写进度汇总
python -X utf8 wiki/format/progress.py --check    # 只校验：数字与表格是否一致（CI 用）
python -X utf8 wiki/format/progress.py --list     # 列出还没完成的卡，适合发队友
```

`check-md.py` 递归检查：未解析的双链、双链页面名重复、**双链里的 `#小节` 或 `#^块ID` 指不到目标页的真实标题／块**、被当成 HTML 标签吞掉的尖括号占位符、H1 与文件名不符、CRLF／BOM，以及 Mermaid `internal-link` 节点的标签是否真的对应一个页面。**`## 相关页面` 不再检查**（D-1793628864，2026-11-02 起该项改为可选）。

**一批目录按「原料」对待**（脚本里的 `RAW_DIRS`，按任意层级的目录名匹配）：`modellist`／`modrt`／`modtest`／`psrt`／`fpai-blog`／`repo2WCIS.repos`／`board_extract`／`man`／`schoolhardware`／`Procise手册Markdown`／`4compile`（2026-11-04 对齐 `wiki/format/check-md.py:23` 的实际清单，11 项）——里面是厂商转档、抓来的博客、取证提取件、word 转档和参考笔记，不做页面形状与正文检查；但它们的页名仍参与双链解析，别处链得动。**校验侧的 `RAW_DIRS` 与站点侧的那一份是两回事**（D-1792281605 解耦）：站点侧另有按文件名的 `RAW_FILES`，`*.docx.md` 那类转档靠它挡——别把整个目录塞进站点侧的 `RAW_DIRS`，目录一进就把别处指向里面的链接变成站点死链。

**基线读数不是零**，且会随页面增减而变，看的是趋势与 `no issues`：

| 日期 | `check-md.py` 读数 | 备注 |
|---|---|---|
| 2026-10-18 | `248 problems in 143 groups` | 旧树 |
| 2026-10-19 | `278／170` | `wiki/` 出库成子模块之后 |
| 2026-10-20 | `280 problems in 172 groups` | 差的 2 条／2 组来自一件未跟踪、正在写的新页 |
| 2026-11-02 | `211 problems in 141 groups` | **不是质量提升**：撤掉「缺 `## 相关页面`」这条判定（D-1793628864）少了 38 条，再清 15 页纯重复页脚少 7 条；未解析双链 70 条等既有问题一条没动。中间档 `256／179` 是撤规则前的读数 |

零问题时脚本打印 `OK: <N> pages, no issues`；**那个 N 只在零问题时出现，别把它当现值抄**。当前记录：`OK: 77 pages, no issues`（2026-10-18，剔除原料目录后的被校验页数；索引到的页数仍是 528）。

`progress.py` 检查看板总账（卡号唯一、分组／状态／路线取值合法、日期是 `YYYY-MM-DD`、三对注释不得紧贴表格），并强制三条纪律：**✅ 必须带 `E<级别> <指向>`**、**⛔ 必须出现在阻塞清单**（支持 `A1–A8` 这种范围写法）、**➖ 剔除必须在备注指到决策编号**。通过后重写 §3.2 的分组完成率与总进度。找看板那一页时要求 `<!-- ledger:begin/end -->` 与 `<!-- progress:begin/end -->` **四条 marker 齐全**、找清单页时要求 `<!-- checklist:begin/end -->` **成对**——只在正文里引用过其中某个字符串的页面不算看板或清单页。期望输出：

```
wiki\archive\开发管理\进度计算与实时看板.md: 0/20 = 0 %, 0 rule violation(s)
  summary block matches the ledger
```

两个脚本都不改正文其他内容；`progress.py` 只动 `<!-- progress:begin -->` 与 `<!-- progress:end -->` 之间的部分。

## 五、进度与状态的写法

- 状态与决策以 `wiki/archive/开发管理/状态源与决策记录.md` 为准，**冲突时它赢**。
- 进度、阻塞、风险只改 `wiki/archive/开发管理/进度计算与实时看板.md` 与 `wiki/archive/开发管理/风险与完成判据.md`。
- **进度只记在看板 §3.1 那张总账表里**，一行一卡、列序固定；分组完成率与总进度由 `wiki/format/progress.py` 从这张表推导并回写 §3.2——**不要手改 §3.2**，也不要删表外的 `<!-- ledger:begin/end -->`、`<!-- blockers:begin/end -->`、`<!-- progress:begin/end -->` 三对注释。注释必须和表格之间隔一个空行——紧贴的话 Obsidian 不会把它排版成表格，脚本也会直接报错。
- **任何影响项目重点、架构、模块边界、组件选择、验证标准或排期的决定，先回写状态源（新增一条 `D-nn`），再视为长期决定。**
- **同一数据只写一处**：误差容限、黄金参考、运行时开关在台账（`任务卡与台账规范` 4.2），不在看板重复。反复引用的数值一律按 D-1790553603 的「单点归属登记表」办——一个数只有一个家，别处写名字＋链接（口令、借用金额、可用内存、串口号与波特率、备用网段、icraft 版本各自的「家」都列在那张表里）；文件名、镜像原文引用和图里的标注不受这条限制，它们是指称不是复述。
- 规则只有一条（D-1790467200）：**一个事实只在一处写，其余页面一律用 `[[链接]]` 指过去，不复制表格。** “找什么去哪”按 `AGENTS.md` §五、`llms.txt` 目录、`wiki/README.md` 目录表与 `ARCHITECTURE.md` §1.4 联合查找；旧 `清单.md` §4 已不存在，不再引用。

## 六、行尾与编码

Git for Windows 安装时把 `core.autocrlf` 写进了 system 配置（`true`：检出转 CRLF、提交转 LF），所以本机 Markdown 曾是 CRLF。现已用 `git config --global core.autocrlf input` 覆盖（global 优先级高于 system，且不动需要管理员权限的 system 文件），仓库内的 `.gitattributes` 再把 `*.md`／`*.json`／`*.mmd`／`*.txt`／`*.csv` 钉成 `text eol=lf encoding=utf-8`。队友那台机器即使 `autocrlf=true`，`.gitattributes` 也照样生效。

`autocrlf=input` 的副作用是 `.bat` 也会变 LF，而 cmd.exe 对纯 LF 批处理不稳。以后往仓库里加 `.bat`，需在 `.gitattributes` 单独写 `*.bat text eol=crlf`。

一次性把散落文件转回来：

```bash
python -X utf8 wiki/format/fix-eol.py --check wiki   # 只报告
python -X utf8 wiki/format/fix-eol.py wiki           # 改写
```

参数给 `wiki` 而不是给 `.`——`.` 会走进永久排除的 `largefiles/`、`source/`、`workenv/`。

**`wiki/` 出库成子模块之后（D-1792368000），它自己那份 `.gitattributes` 与 `.gitignore` 必须自带**——主库的两份都不跨子模块生效，缺了 `.gitattributes` 时 `autocrlf` 会在新库里换行、缺了 `.gitignore` 时那三件凭据载体与 `.bit` 就从「被挡住」变成「可提交」。两份都照主库的口径复制，`repos/` 早就是这么办的（它那份是 `* -text`，因为台账判据是「板上 md5＝本地 md5」）。

## 七、站点

站点的正本只有四处，全部住在文档库根：`site/quartz.config.yaml`（Quartz v5.0.0 配置，插件写 `github:` 源形式）、`site/quartz.lock.json`（该 tag 自己 lockfile 的裁剪副本，只留配置启用的 34 件）、`site/make-content.py`（内容装配）、`.github/workflows/deploy.yml`。

```bash
python -X utf8 site/make-content.py                        # 在本库根：策展集 -> site/content
python -X utf8 site/make-content.py --all                  # 连原料目录一起发
python -X utf8 site/make-content.py --dest <目录>/content  # CI 写进 qbuild/content
python -X utf8 site/make-content.py --audit-only           # 只报数不写
```

- 目标目录必须叫 `content`／`content-test`，且只允许覆盖上一次自己生成的目录（靠里面的 `.generated-by-make-content` 哨兵判定）。
- 装配改为「库根即内容树」：`TREES = ("",)`、`HOME_PAGE = "README.md"`（落站点根 `index.md`），**站点页面路径＝库内路径**，不再套一层 `wiki/`；`SKIP_DIRS` 必须含 `site`／`qbuild`／`public`／`__pycache__`，否则会把生成副本当原料吃掉、满屏 `DUP`。
- 对**生成的副本**做三处改写，笔记本体不动：① 注入 `created`／`modified`；② 把媒体嵌入目标写成全路径（Obsidian 按文件名在全库解析图片，Quartz 只按页面自己那层目录拼）；③ 日期走全史、由 `rename_lookup()` 把旧路径折回现名。
- **最近确认的运行状态**：D-1793566627（2026-11-02）已停用站点工作流，`deploy.yml` 文件头也记录 `disabled_manually`；本轮未联网复核远端设置。按此记录，push 与手动 dispatch 都不执行构建。**启用后的触发定义**仍为 push 到 `wiki` 库 `main`（无 `paths` 过滤）或 `workflow_dispatch`，须先显式恢复启用。
- **发布开关**是仓库变量 `PUBLISH`：不存在＝照常发布；建成 `false` 只跳过部署（站点冻在最后那版、现有 URL 一个都不 404、整笔作业仍绿）。要停构建与分钟就 Settings → Actions 禁用工作流；要把 URL 拿走只有 Settings → Pages → Unpublish。
- **全站拒绝爬虫**（D-1792454402）：`site/robots.txt` 是 deny-all 再点名 40 个搜索引擎与 AI 抓取器，构建后由 `site/harden-site.py` 写进产物、给每个 HTML 注入 `noindex` 的 meta、并删掉 `sitemap.xml`——这一步是硬门禁，注不进去就当场失败不发布。robots 与 meta 都是**请求不是强制**，GitHub Pages 又设不了 `X-Robots-Tag`，所以真要把某页从公开面拿走只有把它剔出内容集（`ignorePatterns`）。
- 耗时参考（本库实测）：冷 `Build site` 18 分 09 秒、整笔约 20 分钟；缓存命中时整笔 3 分 21 秒＋部署 11 秒。

## 八、推送到 GitHub

远端是**三个仓库**：主库 `WCIS-FPAI`（`main`）＋两个子模块 `repos/`（`master`）、`wiki/`（`main`）。登录凭证存在 Windows 凭据管理器里，`git push` 会直接用，不需要再输密码／token。

这台机器直连 `github.com:443` 会被重置，只有走本机代理 `127.0.0.1:7897` 才通；反过来 `api.github.com` 必须**直连**（代理给 403）。所以推送要带代理参数，查元数据不要带：

```bash
git status                 # 先看要提交什么，确认没混进凭据（口令／私钥／token）
git add -A                 # 慎用：这条会连子模块指针一起 stage，正在干活的那个子库要单独摘出来
git commit -m "docs: ..."
git -c http.proxy=http://127.0.0.1:7897 -c https.proxy=http://127.0.0.1:7897 push
```

改过 `repos/` 或 `wiki/` 里的东西时，顺序固定：先在子库里 commit＋push，再回主库用 `git commit --only repos wiki` 把指针 bump 上去（**只动这两个 gitlink**），最后推主库。只推主库不会把子库的内容带上去，只会留一个指不到提交的指针。**写页面的提交现在落在 `wiki/` 库里**（`git -C wiki push` 直接可用），主库那边只出现一行 `wiki` 指针变化。**站点的触发定义跟文档子库，不跟主库指针**（D-1792454400）；但自动部署已按 D-1793566627 停用，当前不能承诺改文档就重发。主库指针只影响「别人 clone 主库时拿到哪个版本的文档」。

> **推模块子仓库时，顺带更新主库指针——但主库其他部分一律不碰**（2026-11-02 她改判并明确范围；旧 AGENTS.md 那条「默认不做，要她当次明说」已作废——两份文件此前互相矛盾，现已对齐）。
>
> **范围铁律：只动 `repos`、`wiki` 这两个 gitlink，主库别的什么都不碰。** 实测依据：2026-11-02 主库工作区有 **497 个已跟踪改动**（`.gitignore`／`agents.md`／`bench_cpu.c`／`hardware/**` 等大量删除与修改，都不是我的）＋ **29 个未跟踪条目**，一个都不能被扫进 bump 提交。
>
> **写法（已 dry-run 验证）**：`git commit --only repos wiki` —— dry-run 确认 `Changes to be committed` 只有这两个 gitlink，497 个改动全部留在 `Changes not staged for commit`。**绝不**用 `git add <路径> && git commit`（不带路径），也绝不用 `git add -A`／`git commit -a`。
>
> **但别自我安慰说它「没有副作用」：bump 就是一次主库 commit。** gitlink 存在 tree 里、tree 必须属于 commit，**要改指针就必须提交**，没有第二条路。所以 bump 会推进主库 HEAD、在主库历史里留一笔、要一次 push 才对别人生效。唯一真的无害之处很窄：那次 commit **不改主库任何文件内容**，不会跟别人的文字改动冲突、不会弄坏别人 clone（前提是子库 commit 已推到远端）。
>
> **三种状态别混**：① 干净 → ② 工作区脏（`M repos`／`M wiki`，未提交，**别人一句 `git add -A` 就扫走**）→ ③ 已提交（推了才生效）。**子库没推之前应停在 ②，不要提前进 ③。**
>
> **推主库前必须过一道护栏：**
>
> ```bash
> git rev-list --left-right --count origin/main...HEAD   # 右=本地独有
> git log --oneline origin/main..HEAD                    # 本地独有的到底是哪几笔
> ```
>
> 只有 bump 那一笔（或全是本次要推的）才推；**还夹着别的提交就停下报告**——一笔主库 push 会把它一起带上去，10-30 那次就这么推上了 `skills/` 429 件／53,684 行。
>
> 另一个前提：子库那个 commit **必须已推到子库远端**，否则别人 `submodule update` 拉不到。**顺序永远是：先推子库 → 再 bump 主库 → 最后推主库。**

## 九、体积与内容范围

**体积是硬闸，不是风格问题**：GitHub 对单文件 >100 MiB 直接拒绝整个 push（按 blob 判，跟打包后大小无关），50 MiB 起警告。所以 `.gitignore` 里那些体积规则是承重的：`*.lib *.a *.o`、`*.bit`、压缩包、模型与轮子都在库外。顶层三目录永久排除、只在本机一份：`source/`、`workenv/`、`largefiles/`（2026-10-25 更正：这里旧写作 `大文件/`，盘上真名是 `largefiles/`，`.gitignore` 里那条 `/大文件/` 挡不住任何整树扫描，已另补 `/largefiles/`）。要单件时用 `git add -f <路径>` 强制加入，但先确认它不超过 100 MiB，否则整笔 push 会被拒。

**内容范围**：不设保密限制，赛题资料与原始表格都可以提交（D-1790467201）。`*.png`／`*.xlsx`／`*.xls`／`*.pdf` 四类已解禁、按普通文件提交（D-1791158404／D-1791158405／D-1791158406；`*.img` 在 D-1791158406 里被重新挡回）。`board_extract/` 自 D-1791158407 起整目录入库，规则是任意层级匹配 `!**/board_extract/**`；**始终留在仓库外的只有三件凭据载体**：`10_etc.tar.gz`、`11_root_home_vendor.tar.gz`、`53_rc_local_and_login.txt`。对外分发时按 `wiki/archive/开发管理/验证与交付.md` 的交付包清单口径剔除。

## 十、编程约定

- **脚本与配置文件**（`.sh`／`.bat`／`.cmake`／`.yml`）内容一律英文，**不写中文注释**，避免 Windows codepage 与 CI 编码问题。
- **工程分层**：ModelBackend、OrchestrationLayer、WebFrontend 分别维护处理能力、业务调度与交互接口。早期「只做 ModelBackend、暂不写另两层」是当时原型范围，不再描述当前实现；现有语言编排、执行器、网页试验台与原生 HMI 已有源码和分项验证，当前边界见 `ARCHITECTURE.md` 第二部分及状态源最新记录。复用成熟组件时明确来源、接口和实测范围，不把规划或样例直接记为整机交付。
- **脚本与临时产物的落点（硬规矩）见 `AGENTS.md` 的「硬规矩：脚本与临时产物一律进 `workenv/`」一节**：临时脚本、一次性工具和运行中的中途输出放 `S:\work\WCIS\workenv\`；已完成开发证据仅按该节登记的用户指定归档例外入库，操作脚本本身仍留在 `workenv/`。
- 凭证纪律：不把密码、Cookie、令牌等凭证写入项目、记忆、日志或回复（D-1790553603 单点记账）。

## 相关文件

- 全仓索引：`llms.txt`
- 干活纪律与 `#SOP` 约定：`AGENTS.md`
- 仓库与系统结构：`ARCHITECTURE.md`
- 文档库首页与目录表：`wiki/README.md`
