# AGENTS.md

> 在本仓（WCIS）干活时的记忆与入口。**先读本文件，再按需展开**：
> `llms.txt`（全仓索引）· `ARCHITECTURE.md`（结构）· `CONTRIBUTING.md`（写作与协作规范）。
>
> 本文件取代旧的 `agent.md`（住 `wiki/`）与主库根那份逐字节副本 `agents.md`——两份内容已并入本文件，那两份退役。此后 agent 记忆只有这一处，不再维护第二份副本。

安装所有东西前先确认前提：1. 机器上有木有已安装好的2. 有木有同类替代品3. 有木有所需环境4. 安装后是否确定可用。避免不必要的安装
每次用户输入都 检测。 是询问还是指令，不要把询问当 指令去执行。 用户 给出命令前，一定是 确认了后续方案 或明确进行尝试。 才开始。 要求。 执行。 不要用户要求推荐 一下就开始安装 
不要用本机的iGPU跑任务，用dgpu
本机默认浏览器Firefox，请使用 mozilla/firefox-devtools-mcp 作为mcp，win也自带edge，也安装了playwright-mcp可用。
任何模型条目的架构、构成、参数量必须落到 config 级实测证据，不能依赖目录名、周边已装模型或加载行为推断
加载失败可能是加载方式不匹配，再检查一次官方介绍并另行查明正确入口
电脑安装了Windows-MCP
优先使用 `rtk` 执行命令以精简输出，代码与文本搜索优先使用 `rg`、结构化代码搜索优先使用 `ast-grep`，Git 操作优先使用 RTK或dsh-ops 对应子命令；避免直接使用未经处理的冗长命令输出，以及使用 `grep -r`、`find` 等未过滤搜索方式扫描大型仓库，除非不支持、结果不完整或需要原始输出进行排查, 遇到这种情况，要提醒用户。
有中文或字符的情况下都指定utf-8

## 开工前：先读这五处（别跳过就干活）

**本项目最容易犯的错，就是没读规则直接开工。** 动手前按顺序读这五处：

| 在哪 | 读什么 |
|---|---|
| 主库根 `AGENTS.md`（本文件） | 干活纪律、frontmatter 标签聚合约定（`#SOP`／`#index`）、三台机与网络、脚本落点 |
| 主库根 `CONTRIBUTING.md` | 写作、校验、行尾、站点、推送规范 |
| 主库根 `ARCHITECTURE.md` | 仓库结构与作品系统架构 |
| 主库根 `llms.txt` | 全仓索引（**llm 优先读它**） |
| **`wiki/archive/开发管理/`** | **项目管理的全部真值**：状态源与决策记录、进度看板、风险与完成判据、清单（**只剩"现在做哪一件"＋流程图；原第 3／4 节已不存在，见§五**）、任务卡与台账规范、实施路线、验证与交付、pages建设过程 |

**`wiki/archive/开发管理/` 是承重目录**，不是归档垃圾堆：状态、决策、进度、风险、待验证项、任务卡口径全在里面。**任何涉及排期、架构、模块边界、组件选择、验证标准的动作，先读它；做完回写它**（见第六节）。**全库编号体系**（决策 `D-`、任务卡、台账行、风险、赛题事实、厂商问题单）的写法与取号规则在该页**第 8 节**；决策的取号规则与全量索引在第 4 节。新记录取**写作当刻的 UTC 秒**；**别用全局正则改这些短记号**（`A8`/`B8`/`C1` 同时是 URL 编码与 FPGA 管脚名，`M1` 是 ARM 核名，`Q4` 是量化档名）。

## 硬规矩：脚本与临时产物一律进 `workenv/`

**所有一次性脚本、一次性工具、中途产物、日志、临时 `json`／`csv` 一律放 `S:\work\WCIS\workenv\`**（按主题建子目录）。**不许**散落在主库根、`wiki/`、`repos/` 或任何入库路径里——这是硬规矩，不是偏好。

**判据只有"不许乱丢"这一条；处置动作是移动到 `workenv/`，不是删除**（2026-11-03 她裁）。发现一次性脚本或临时产物落在了别处，就把它挪进 `workenv/` 的对应主题子目录——别就地留着，也别顺手清掉（脚本本身是下次复跑与取证的入口）。别处已有的同类落点不判违规，**不要为了"统一"去搬动或删改不属于本仓的东西**。

入库路径里只留三类东西：

| 允许留在库里 | 说明 |
|---|---|
| 页面（`.md`） | 文档库 `wiki/` 的正文 |
| 配置 | `site/quartz.config.yaml`、`.gitattributes`、`.gitignore`、`skills/`（运行中按需加载的技能，2026-10-25 起按配置类入库，含其自带脚本） 等 |
| **登记过的工具链脚本** | 只有 `wiki/format/`（校验与行尾）、`wiki/site/`（站点装配与加固）、`.github/workflows/`（CI）三处；它们是项目工具，有家、有正本 |

其余一律进 `workenv/`：`workenv/` 已被 `.gitignore` 永久排除，**不进库、不建远程、不做子模块**。要重跑的规程写成 SOP 进 `wiki/SOP/`，脚本本身仍留在 `workenv/`。

**`workenv/` 里的东西一律不清理**（2026-11-02 她裁："所有的临时产物和一次性脚本都放在 workenv，不用清理"）：不删、不挪、不压缩、不打包，**也不再在 SOP 收尾时问"要不要删临时产物"**（第二节第 5 条已按此作废）。理由＝这些一次性脚本是下一次复跑与取证的入口，删了就得重新发明；`workenv/` 不入库、不进任何交付面，占的只是本机盘。需要复用的脚本就地留着，别为了"整齐"往别处搬。

### 正式开发证据归档例外

2026-11-05 用户明确指定：HA4FPAI 的已完成运行日志、性能图表、原始采样及验证结果可以作为正式开发记录归档到 `wiki/PS/ha/`。此类记录须有来源、版本/时间、结论边界和逐文件校验清单；原始证据放 `wiki/PS/ha/ci-records/`，保留原始字节，不作为普通 Markdown 页面改写。原始 Markdown 报表可用 `.md.txt` 存证，避免重复页名。

该例外只覆盖已登记的正式开发证据，不把运行中的临时输出、一整棵源码副本、依赖缓存或安装二进制带入 Wiki。一次性操作脚本仍留在 `workenv/`，Wiki 只引用其位置和复现方法；`workenv/` 原件只复制、不移动或清理。归档不代表授权执行发布脚本或推送仓库。

**落库前自查**：`git status` 里不该出现散落的新增 `*.py`／`*.sh`／`*.ps1`／`*.bat`／`*.log`／临时 `*.json`／`*.csv`——出现先按本节归位；上面已登记的正式开发证据除外。

## 一、每次新任务的信息来源（整节作废）

- **2026-11-04 她裁**：旧内网信息站已作废（地址按她 2026-11-07 指令不再记于此），本节整节撤销，不再作为任何任务的开工信息来源。开工信息以本地 `llms.txt`、`wiki/README.md`、`ARCHITECTURE.md` 为准。节号保留占位，免得后面各节重号。

## 二、干活纪律

1. 先检查问题的错误、前提逻辑跳跃和信息缺失，独立判断，不迎合。区分事实、推测和主观观点；涉及数字、人物和结论时尽量核实来源。不同意时直接指出并给出依据，主动提醒遗漏的变量、成本、风险、替代解释和偏差。
2. 把这次实际上的最后操作记录下来，做成一个**可复现的 SOP**。
3. **一定不要在 113 上编译**；优先校机，其次本机；需要交叉编译且校机无法完成时再走 `ssh takomi@192.168.1.107`（密码 `<见凭据管理器>`，podman 里有可用容器）。
4. ~~每一个 SOP 完成后（即使还有需要用户判断的问题）都**询问是否删除临时产物**~~ ⇒ **2026-11-02 她裁作废**：`workenv/` 下的临时产物与一次性脚本一律留着，不问也不删（见「硬规矩」那节）。SOP 收尾该问的只剩页面与决策回写。
5. 对博客主题的操作同步到博客上，但不要丢失个人配置。
6. clone 若坚持直连，必须加超时保护或改走 gitee／ghproxy 镜像——原脚本无超时会再次死等。
7. 模型下载一定用模型提供网站的**官方工具**，不要用 `curl`／`wget`；连不上抱脸时先换国内同款站点找同款模型，无果再检索本机有无 mihomo 在运行，使用其代理。
8. 临时脚本、工作时产生的中途产物一律放 `workenv/`。
9. 每次会话在没有任何悬而未决的问题和汇报项之后，才询问用户是否总结成一个 Markdown 单文件（要输出名字）放入 wiki 对应某目录（要输出推荐目录，可多选）。悬而未决＝问出去还没得到回答、或留给她拍板还没结论的事；报告里还挂着这类尾巴时不得触发。

## 三、用 Obsidian 的 frontmatter 标签聚合

**通用约定（对任意 tag 都成立）**：

- 页面级归类标签写在 frontmatter（文件开头的三横线块内）：`tags: [标签名]`；一页可多标（`tags: [完整复现, index]`）。Obsidian 即成 `#标签`，标签面板与搜索都能聚合。
- 命令行检索模板：`rg -l '^tags:.*\b<标签>\b' wiki --glob '*.md'`；任何清单都以检索实测为准，正文里的计数会过期，别当真值。
- **每新立一个 tag，必须在本节下挂一个小节**（判据＋写法＋检索），需要全量页单的（如 `#index`）同时把清单登记进 `llms.txt`，打标／除名时随手维护。
- **现在有三个 tag**：`#SOP`（操作规程页）、`#index`（索引页）、`#error`（故障/错误诊断页），约定分见下面各小节。

### `#SOP`：操作规程页

**现状（2026-11-07 按约定检索命令实测）：已打标 29 篇**（2026-11-04 那轮 26 篇之后，目录重构与新增页使清单变长；当年「待办」17 篇清单已清零）。`bitproc.md`／`modproc.md` 两个残缺片段仍不打标（见待办）。约定从本文件起生效。**以标签清单为准，别信这句文字**：`rg -l '^tags:.*\bSOP\b' wiki --glob '*.md'`。

#### 约定

- SOP 页的 frontmatter 写 `tags: [SOP]`（YAML，文件开头的三横线块内）。Obsidian 里即成 `#SOP` 标签，标签面板与搜索都能聚合。
- 位置：SOP 页放 `wiki/SOP/`，模型／工具链类放 `wiki/SOP/MOD/`（测试类在 `wiki/SOP/MOD/test/`，文档库与站点自身的在 `wiki/SOP/wikimgr/`），Windows 主机与校机运维类放 `wiki/ops/`；与某条具体链路强绑定的也可以就放在那条链路旁边（如 `wiki/PS/oldmainchain/OrchestrationLayer/VPU串流直播SOP.md`）。
- 一页一条链路：写清「怎么跑 ＋ 怎么判过了」。判据要可复现，读数要带来源。

#### 新建 SOP 时

1. 页形照 vault 规矩：`H1` 与文件名一致、LF／UTF-8／无 BOM（`## 相关页面` 自 2026-11-02 起可选，详见 `CONTRIBUTING.md`）。
2. frontmatter 至少三行：

   ```yaml
   ---
   tags: [SOP]
   ---
   ```

   可另加 `updated: YYYY-MM-DD`。
3. 写完在 `wiki/README.md` 的「目录」表里登记一处（表负责跨端可达）。
4. 跑校验：`python -X utf8 wiki/format/check-md.py`。

#### 检索

- Obsidian：搜索框输入 `tag:#SOP`，或右侧标签面板点 `SOP`。
- 命令行：`rg -l '^tags:.*\bSOP\b' wiki --glob '*.md'`（frontmatter 里出现即命中）。
- **全库 SOP 页清单以标签为准，本文件不维护第二张表**（一个事实只写一处）。

#### 待办（2026-11-04 已清零）

当年那张 17 篇清单处理完毕：**15 篇带上了 `tags: [SOP]`**；`bitproc.md`（16 行草稿碎片，页内自述"处理完再放回 wiki"）与 `modproc.md`（8 行残缺表片段，各行正本都在别页）**不是成形的规程，暂不打标**——补成真页或挪去 `todo/`，等她发落。同轮另说"把 `ops/` 三篇 Windows 运维规程一并打标"——**这句按 2026-11-07 实测更正**：那三篇里 `Windows免提权弹通知与SSH登录告警`（当时指的名字）**从来没有 frontmatter、也就从没带过标**，且该页已于 2026-11-06 目录重构 `b51ca96` 删除；现役 `ops/` 里带 `tags: [SOP]` 的是 **7 篇**（`win无wsl交叉编译`、`交叉编译主机路由SOP`、`校机交叉编译环境搭建SOP`、`校机代理与GitHub登录SOP`、`校机无UAC远程操作与零窗口SOP`、`校机无容器交叉编译`、`校机构建Klar前端并部署上板SOP`），唯一不带标的是 [[Windows提权与通知失败记录]]（它按定位是**失败史**、不是可照抄的规程，不打标是对的）。**此后不再维护纸质待办清单**：新 SOP 页落笔即带 `tags: [SOP]`，清单以检索命令为准。

> **2026-10-31 新增 [[板端磁盘清理SOP]]**（`wiki/SOP/板端磁盘清理SOP.md`，frontmatter 带 `tags: [SOP]`）。它管"板子快满怎么安全清"：第 0 步 `find -newermt "-15 minutes"` 查有没有别的会话在写（**只删自己确认没人用的**）→ 分级（缓存／已装包残留／测试期产物／模型权重，**第四级必须她点头**）→ 只做到第一级也能回收 800M–1G → 删完 `df` 复读＋8 项服务复查 → 回写台账墓碑行与 [[06-磁盘清理清单]]。判据正本在 [[06-磁盘清理清单]]（2026-10-31 已全量重刷），删除记录在 [[板端我方代码与产物台账]] §廿四。
>
> **顺带更正上面那条"现状：一篇都还没打标"**——按约定的检索命令 `rg -l '^tags:.*\bSOP\b' wiki --glob '*.md'` 实测，**已经打了标的有 4 篇**：本页新增的 `板端磁盘清理SOP`、`交付物数字审计num-audit`（10-31）、`MOD/icraft数值验证与Podman环境SOP`（10-31）、`MOD/交付件就绪状态盘点SOP`（10-30）。⇒ 下面那张 17 篇待办清单**实际还剩 16 篇没打**（`交付物数字审计num-audit` 不在原清单里，是后来加的）。
>
> **同一轮落地的 OS 层改动（归口在 [[os]] 第七节）**：journald 上限走 drop-in `/etc/systemd/journald.conf.d/99-wcis-size.conf`，`SystemMaxUse=256M`／`RuntimeMaxUse=32M`／`MaxRetentionSec=1month`，**出厂主文件 `/etc/systemd/journald.conf`（1042 B）一个字节没动**，回滚＝删 drop-in＋重启 journald。依据：实测增速 **3.0 MiB/h 原始文本（约 72 MiB/天，journal 二进制再乘 1.3–1.6）**，256M ≈ 3.5 天；出厂默认是 `min(盘 10%, 留 SystemKeepFree)`＝在 26G 的 SD 卡上放行 2.6G，这就是它反复回涨的根因。**判生效的唯一可靠证据是 journald 自己启动打印的 `is … max 256.0M … free` 那行**——`systemctl show systemd-journald` 在 systemd 245 不暴露这些属性，`systemd-analyze verify <某.conf>` 也不支持 .conf（它只验 unit，报 `Invalid argument` 是用法错不是语法错）。

### `#index`：索引页（2026-11-04 她定）

- **什么页打**：以"指向别处"为职的页面——索引页、总纲/总览、清单、台账、阅读地图、目录占位 stub（`PS/oldmainchain/WebFrontend/webapp.md` 那类）。判据＝**页面的价值在导航而不在事实**；事实页不打（反例：`dict.md` 的词条是定义本体、`00_失败与限制` 登记的是事实，都不打）。
- **怎么写**：frontmatter 三行，与 SOP 同款——`tags: [index]`；可与项目自有标签并存（多标的例子见 `llms.txt`「索引页（#index）」一节）。⚠ 原来举的 `完整复现索引.md` 那页已不在盘上（`wiki/完整复现/` 整个目录 2026-11-06 起不存在，git 历史里也没有）。
- **检索**：`rg -l '^tags:.*\bindex\b' wiki --glob '*.md'`；Obsidian 标签面板点 `index`。
- **全量清单的正本在 `llms.txt`「索引页（#index）」一节**——本节只管规则，不维护清单；打标／除名时去 llms.txt 登记一行，实测以检索命令为准。

### `#error`：故障/错误诊断页（2026-11-08 她定）

- **什么页打**：记录**已发生的故障/错误**的页面——现象、根因（带证据）、修法、边界条件。不打 `#SOP`（那是“怎么跑＋怎么判过”的规程）；一页既给规程又做诊断时可多标（`tags: [SOP, error]`），但默认分开。
- **怎么写**：frontmatter 三行——`tags: [error]`；正文要有**证据来源**（日志/读数/时间）和**机制**（别只写“重启就好了”）。修法落在页内；若修法已固化成规程，规程单独成页打 `#SOP`，本页只链接过去。
- **检索**：`rg -l '^tags:.*\berror\b' wiki --glob '*.md'`；Obsidian 标签面板点 `error`。分类 tag，不维护全量页单。

## 四、三台机与网络

| 机器 | 地址 | 账号 | 能做什么／不能做什么 |
|---|---|---|---|
| 本机 | `192.168.1.107` tkmnopc | `takomi` / `<见凭据管理器>` | Windows 主力工作站，可 ssh；`S:\work\WCIS` 在这里，大部分信息都在这台机器上；有时可经路由器 `18389` 端口 RDP。直连 `github.com:443` 会被重置，推送要走本机代理 `127.0.0.1:7897`；`api.github.com` 反过来必须直连（代理给 403） |
| 校机 | `ssh school-guest`（另有 `-lan`／`-pc2` 两个别名，三条都落同一台，实测 `hostname` 全是 `Win11-HV1`） | `YiZhidi` | **2026-10-31 实测更正：ssh 进去的是台 Windows 11 虚拟机**（`Win11 Pro for Workstations`／build 26200，`whoami`＝`win11-hv1\yizhidi`，`Win32_ComputerSystem`＝Microsoft Corporation／Virtual Machine、`HypervisorPresent`＝True），硬件是 AMD EPYC 9654、24 物理核／24 逻辑核、125 GB——即那台主机的子机，**不是"仅 CPU 的 Linux 机"**。该机**没有 iCraft、没有 WSL、没有任何容器运行时**（`where icraft` 空、`C:\Icraft` 不存在、`wsl --status`／`wsl --version` RC＝50／1 报未安装、PATH 里无 docker／podman／nerdctl／containerd、服务表只有 `HvHost`（Stopped／Manual）、Program Files 无容器类目录、无 Linux 类 Appx），**我也补不回来**：`whoami /groups`＝`Medium Mandatory Level`（S-1-16-8192）、组只有 Users＋Remote Desktop Users（administrators 里只有 `Administrator` 与 `SuRongbin`），`Get-WindowsOptionalFeature` 直接报 requires elevation；`SecondLevelAddressTranslationExtensions`＝False ⇒ 宿主没给这台 VM 透传嵌套虚拟化，WSL2／容器要的那个轻量 VM 起不来。**但"没容器"不等于"不能交叉编译"（同一天第二次更正：我第一遍只查了 C 盘与 PATH 就写"跑不了"）**——板端 aarch64 的交叉编译环境整套在 **D 盘**且实跑通：`aarch64-none-linux-gnu-gcc.exe --version` RC＝0 出 `10.3.1 20210621 (arm-10.29)`（`D:\yzd\tools\cross\gcc-arm-10.3-2021.07-mingw-w64-i686-aarch64-none-linux-gnu\bin\`），`D:\yzd\MSYS2\usr\bin\bash.exe`／`mingw64\bin\cmake.exe`／`mingw64\bin\ninja.exe` 都在，`D:\yzd\WCIS\crosswin` 根在（`kit` 948 件／`pyroot` 209／`src` 45／`donor` 10／`logs` 31，产物 `build\yolov8n` 6 408 568 B＝2026-10-16 06:33、`build-py` 五件），D 盘可用 1 571.9 GB（C 盘只剩 19.5 GB）⇒ **交叉编译照旧走校机，正本＝[[交叉编译主机路由SOP]] 第一节＋[[校机无容器交叉编译]]**（那条路本来就是"无容器"设计，用的是 MSYS2＋Arm GNU，不是容器也不是 WSL）。要 **iCraft 模型编译**才得换主机：现役 CLI 在本机 Windows（`C:\Icraft\CLI\bin\icraft.exe`，`icraft --version` 直读 `Icraft 版本: 3.39.0`／`CLI 版本: 3.39.0.0-431be90(2608211646)`；`C:\Icraft\CLI v3.39.0` 那份额外副本的 exe FileVersion／ProductVersion 与 build 号完全相同＝同一颗包，MACHINE 级 PATH 只挂前一个），且 **ZG 档 imodel 只能本机 Windows CLI 出**（Linux x86_64 那份 amd64 包的 codegen 只带布衣，证据见 [[NPU模型上板出指标SOP]] 第九节坑 4）。ssh 定义与代理口径的正本仍是 [[校机代理与GitHub登录SOP]] |
| 开发板 | `192.168.1.113` FMQL30TAI676H 悟净（完整版） | `root` / `<厂商默认口令，见交付件>` | WCIS 项目目标平台、边缘设备，**不可用于编译**（993 MB 内存、CMA 池 256 MB，编译会挤死现役链）；arm 内核、Ubuntu 20.04、分 PS 和 PL 端、有 NPU、可执行 FPGA。可 ssh；网络不可用时可通过 107 电脑走 putty 连 COM 端口（速率 `115200`）访问内核 |
| 中转机 | `192.168.1.102` debian-h61 | `user` / `<见凭据管理器>` | Debian 13，可 ssh；可中转 `ssh -J user@10.101.54.192:18222 root@192.168.1.113` 连接 113。旧内网信息站**已于 2026-11-04 作废（§一）**，站点现行地址是 `https://wcis-team.github.io/WCIS-FPAI-wiki/`（D-1793969383）；那台 `wcis-relay` 没有 systemd unit 也没有开机自启，它吐的 200 只代表自己磁盘里那份旧副本 |

- 路由器 `10.101.54.192`；内网 `192.168.1.0/24`。

## 五、目录与「信息放哪」

- 三个仓库（**远端全在 GitHub 组织 `WCIS-Team` 下**，2026-11-06 重建根历史，旧 `takomitkm/*` 只有跳转）：主库 [WCIS-FPAI](https://github.com/WCIS-Team/WCIS-FPAI)、文档库 [WCIS-FPAI-wiki](https://github.com/WCIS-Team/WCIS-FPAI-wiki)（＝`wiki/`，Obsidian 入口）、代码库 [WCIS-FPAI-repos](https://github.com/WCIS-Team/WCIS-FPAI-repos)（＝`repos/`）。仓库↔目录的逐条对应正本＝`wiki/repo2WCIS.repos/repo项目索引.md` 第一节。
- 主库根：`repos/`、`wiki/` 两个子模块，加根级这几份文档；`source/`、`workenv/`、`largefiles/` 只在本机一份、永久排除（`largefiles/` 曾写作 `大文件/`，`.gitignore` 里那条同名规则是死规则，2026-10-25 已补真名）。
- 我方产物的存放规则：FPGA → `wiki/PL/FPGA`；NPU 模型 → `wiki/PL/MOD`；Linux 端 → `wiki/PS`；两边都有的 → `wiki/repo2WCIS.repos`（**目录真名；旧文写作 `repo对应WCIS.wiki.repo` 是错的**，2026-11-01 更正）。
- 规则只有一条（D-1790467200）：**一个事实只在一处写，其余页面用 `[[链接]]` 指过去，不复制表格。**
- **「找什么去哪」怎么查（2026-11-01 定：不再有独立分流表）**。原来那句"查 `清单.md` §4 分流表"已作废——`清单.md` 的第 3／4 节在某一刻就没了（2026-11-01 核实时全页只剩第 1、2 节），但全库还有 7 处在指它。**改由下面四处合起来承担，按顺序用**：

  | 序 | 在哪 | 管什么 |
  |---|---|---|
  | ① | **本节**（AGENTS §五） | 规矩、三库分工、承重页指向、组件优先级口径 |
  | ② | `llms.txt`「入口文档」＋「目录」 | llm 优先读的入口清单；主库／文档库逐目录一句话 |
  | ③ | **`wiki/README.md`「目录」表** | **逐目录"这里放什么"＋代表页** —— 四者里最接近原分流表的一张，**新页先登记到这里** |
  | ④ | `ARCHITECTURE.md` §1.4 | 三处真值（状态源／看板／风险判据）与数据流 |

  另外三条高频查询不必走上面这张表：**按卡号找判据页** → `进度计算与实时看板` §3.1 总账表末列「详情」；**找板端某个文件落在哪** → `板端我方代码与产物台账`（按 `#nnn` 编号，含已删件的墓碑行）；**工具／任务在哪台机跑** → `交叉编译主机路由SOP` §二路由表（Vivado／综合布线只在本机，吃性能的仿真在校机；校机不行、只能容器交叉编译时，本机 podman 有现成镜像 `localhost/fpai-ub2004:arm64-icraft3.39.0`）。
- **板端 OS 层改动只有一个归口**：`wiki/PS/psrt&dev/os.md` 第七节（发行版／内核／模块／分区／包／时间与日志／启动与服务 ＋ **我方对 OS 的全部改动**，如 swap、`swappiness`、`rc.local`、`/usr/local/bin` 软链、apt 增量、journald 上限）。判据＝**动到 OS 层才算**；纯我方脚本落 `/root/` 下的不进那页、进台账。**这条以前没有落点**（2026-11-01 补）。⚠ 路径已于 2026-11-06 目录重构从 `wiki/PS/psrt/rt/os.md` 变为 `wiki/PS/psrt&dev/os.md`（`psrt/优化/` 那篇也升到了 `psrt&dev/` 直下）；全库还有指旧路径的页面在按批次改，**新增引用一律写新路径**。
- 状态与决策以 `wiki/archive/开发管理/状态源与决策记录.md` 为准，冲突时它赢；进度只看 `wiki/archive/开发管理/进度计算与实时看板.md` §3.1 那张总账表。
- **组件选型的优先级口径**：P0＝当前必须；P1＝计划内后置；P2＝可选；条件 P0＝由实测决定是否做。**优先级不等于进度**，进度只看看板。选型表 `组件.xlsx` **已删、不再维护**（2026-11-04 她裁；2026-10-18 布局重组 `452a586` 删除，旧版可 `git show '452a586^:组件.xlsx'` 取回）。
- 缺件与一次性大文件：见 `llms.txt` 的「大文件与外部依赖」（`largefiles/JFM_Kits/ip_patch.zip`＋`ip_patch_filelist.txt`；那 384 颗 `.bit` 在 zip 内的 `ip_patch/tools/bitfile/`，盘上没有解出来的目录，要用得先解包——已解包的那份 `largefiles/4compile/JFM_Kits/ip_patch/` 里 `tools/` 下确实没有 `bitfile/`，2026-11-07 复核仍成立）。⚠ 位流编译工作区已从 `wiki/PL/FPGA/4compile/` 移到 **`largefiles/4compile/`**（2026-11-06 重构），`wiki/.gitignore` 里 `/PL/FPGA/4compile/` 那条现在是死规则。
- 详细目录树见 `ARCHITECTURE.md`。

## 六、决策与状态回写

任何影响项目重点、架构、模块边界、组件选择、验证标准或排期的决定，**先回写 `wiki/archive/开发管理/状态源与决策记录.md`（新增一条 `D-nn`）**，再视为长期决定。状态源只有这一页。

## 七、提交与推送（细节见 CONTRIBUTING.md）

- **内容只进它自己的子库**：页面进 `wiki/` 库（`git -C wiki push`，站点跟这一笔走），代码进 `repos/` 库（`git -C repos push`）。只推主库不会把子库内容带上去，只会留一个指不到提交的指针。
- **推模块子仓库时，顺带更新主库指针——但主库其他部分一律不碰**（2026-11-02 她改判并明确范围）。
  - **触发时机**：推 `repos/` 或 `wiki/`（＝主库的两个模块子仓库）时，顺手把主库对应 gitlink 一起更新。不用再逐次问她。
  - **范围铁律**：**只动 `repos`、`wiki` 这两个 gitlink 条目，主库别的什么都不碰。** 实测依据：2026-11-02 主库工作区里有 **497 个已跟踪改动**（`.gitignore`／`agents.md`／`bench_cpu.c`／`hardware/**` 等大量删除与修改，都**不是我的**）＋ **29 个未跟踪条目**；这些一个都不能被扫进 bump 提交。
  - **写法（已 dry-run 验证）**：
    ```bash
    git commit --only repos wiki
    ```
    dry-run 输出确认：`Changes to be committed` 只有 `modified: repos` 和 `modified: wiki`，那 497 个改动全部留在 `Changes not staged for commit`。这是唯一正确的写法。
  - **绝不**用 `git add <路径> && git commit`（**不带路径**）——那会把 index 里所有 staged 的东西一起提交，正是 2026-11-02 我吞掉别人 3 个 audio 页的翻车姿势；也绝不用 `git add -A`／`git commit -a`。
  - **得承认：bump 就是一次主库 commit。** gitlink 存在 tree 里、tree 必须属于 commit，**要改指针就必须提交**，没有第二条路。所以它会推进主库 HEAD、在主库历史里留一笔、要一次 push 才对别人生效。唯一真的无害之处很窄：那次 commit **不改主库任何文件内容**，不会跟别人的文字改动冲突、不会弄坏别人 clone（前提是子库 commit 已到远端）。
  - **三种状态别混**：① 干净（指针=旧值）→ ② 工作区脏（`M repos`／`M wiki`，未提交，**别人一句 `git add -A` 就扫走**）→ ③ 已提交（HEAD 前进，推了才生效）。**子库没推之前应停在 ②，不要提前进 ③。**
  - **技术前提**：子库那个 commit **必须已在子库远端**（`git -C <子库> ls-remote origin refs/heads/<分支>` 与本地 HEAD 一致）。否则主库指针指向别人拉不到的提交，`submodule update` 直接失败。**顺序永远是：先推子库 → 再 bump 主库。**
  - **推主库前过一道护栏**（护栏在「推」，不在 bump）：
    ```bash
    git rev-list --left-right --count origin/main...HEAD   # 右=本地独有
    git log --oneline origin/main..HEAD                    # 本地独有到底是哪几笔
    ```
    - 只有 bump 那一笔（或全是本次要推的）⇒ 推。
    - **还夹着别的提交**（别的会话攒的、她没叫推的）⇒ **停下报告，不推**，把清单给她看。这是 2026-10-30 事故的形状（一笔主库 push 带上 `skills/` 429 件／53,684 行，撤回＝`dfc867c`），别重演。
  - **旧规矩作废**：2026-10-30 那条「默认不做，要她当次明说」已废。
- **push 被拒（non-fast-forward）就是被拒，不要自己合并**：不 stash、不替另一条会话提交、**不 `reset` 本地分支头**。本地三个分支头常年落后于远端是**故意的**——挪头会让共享工作树里凭空多出几百个"已删除／已修改"，正对着对方那波未提交的文件，一句 `git add -A` 就能把内容真删掉。
- 她当次授权了要推，就在临时工作树里做，别碰共享工作树：`git -C <库> worktree add --detach S:/work/WCIS/workenv/tmp/<名> origin/<分支>` → 在里面 commit／merge → `git -C <库> push origin HEAD:<分支>` → `git -C <库> worktree remove <同一路径>`。**分支名各库不同：`wiki`＝`main`、`repos`＝`master`、主库＝`main`。**
- **同一文件里另一条会话有未提交改动 ⇒ `git add <文件>` 会把它的那几行一起吞进我这一笔**（2026-11-01 实测：台账 §廿七 那页她正在写 §廿八，`git status` 只显示一个 `M`，看不出里面有别人的行）。这种时候**不要**在共享工作树里 add，走上面临时工作树：从 `origin/<分支>` 起 → 只在自己的那一段（按节标题定位行号）改 → commit → `push origin HEAD:<分支>` → `worktree remove --force`（工作树里只要留着任何未跟踪或改过的文件，不加 `--force` 就报 `contains modified or untracked files` 撤不掉，2026-11-02 实测；用完再 `git worktree prune`，别让它挂在 `worktree list` 上）→ **再把同一段文字镜像回共享工作树里那页**，否则本地那页比远端旧，下一次别人在这页改就制造冲突。镜像用脚本按行号切，落笔前后各核一次"这一段逐字节相同"＋无 BOM／无 CRLF。
- **子模块指针冲突 git 自己合不了**（`CONFLICT (submodule)`，"not checked out" 是正常提示）：记进去的值必须**同时包含两边**的提交，用 `git update-index --cacheinfo 160000,<40 位哈希>,<路径>` 写；写完对每个方向跑 `git merge-base --is-ancestor` 全过再推——记本地头等于悄悄撤掉对方刚推的那次 bump。
- **子模块指针核验禁用 9 位缩写**：`ls-tree`／日志里的 9 位 SHA 会因同仓不同提交前缀相撞而误判（2026-11-08 实测：HA4FPAI 记的 ERNIE 指针 `790afc5043cea45…` 与真实 master `790afc5043aa…` 前 10 位相同、ERNIE 仓里根本没有前者，9 位缩写却"通过"了，结果云端链路断开、clone 后 `submodule update` 失败）。核验指针必须取**整 40 位**，并用 `git -C <子模块> cat-file -t <40位SHA>` 确认该提交**真实存在于子模块仓**（返回 `commit` 才算数）；bump 完指针后**必须把共享工作树对应子模块 ff 到该提交**（`git -C <子模块> merge --ff-only <SHA>`），否则共享树会一直显示 `M 子模块`／落后云端。
- 推送带代理参数，查元数据不带：`git -c http.proxy=http://127.0.0.1:7897 -c https.proxy=http://127.0.0.1:7897 push`；报 403 时加 `-c credential.helper= -c credential.helper=manager`（本机 `~/.git-credentials` 里那颗 PAT 会挡路）。
- 提交前 `git status` 确认没混进凭据（口令／私钥／token）；命中就把那件用 `.gitignore` **具名**排除并在注释写清原因，**不要为了推上去去改别人的代码**。
- 体积是硬闸：单文件 >100 MiB 直接拒整笔 push。

## 与旧 agent.md 的取舍

- 已并入：9 条干活纪律、`组件选型与复用边界` 里跨页通用的两条口径（优先级 P0/P1/P2/条件 P0、`组件.xlsx` 的位置）、`项目约定`、`每次新任务的信息来源`、`PL端编程约定`（→ `CONTRIBUTING.md` §十），以及那页里的写作／校验／行尾／站点／推送约定（→ `CONTRIBUTING.md`）。「每次完成任务后询问是否总结成单文件」那条（旧文件第 2 行）当初合并时漏搬，2026-11-04 按她原话补回第二节第 9 条；同日她指正我触发过早（报告里还挂着两个待拍板项就问了），触发条件据此写实：悬而未决＝问出去没得到回答、留给她拍板没结论，尾巴没清零不得触发。
- **未搬运**：旧文件里那段未整理的粘贴件正文（`组件选型与复用边界` 的表格与行文本身）。要留档就把它还原成 `wiki/` 里的一页，别塞进 AGENTS.md。

## 相关文件

- 全仓索引：`llms.txt`
- 仓库与系统结构：`ARCHITECTURE.md`
- 写作、校验、行尾、站点、推送规范：`CONTRIBUTING.md`
- 文档库首页：`wiki/README.md`
