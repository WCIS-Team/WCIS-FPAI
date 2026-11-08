# ARCHITECTURE.md

> 这份文档说两层结构：**一、仓库结构**（三库怎么分、文档库怎么分层、数据怎么流到站点）；**二、作品系统架构**（FMQL30TAI 上 FPGA＋NPU＋CPU 的分工与链路）。
> 索引见 `llms.txt`，干活纪律见 `AGENTS.md`，写作与协作规范见 `CONTRIBUTING.md`。

---

# 一、仓库结构

## 1.1 三个仓库

| 仓库 | 分支 | 在本地 | 职责 |
|---|---|---|---|
| `WCIS-FPAI` | `main` | 主库根 `S:\work\WCIS` | 只记两个子模块指针 ＋ 根级文档（`AGENTS.md`／`ARCHITECTURE.md`／`CONTRIBUTING.md`／`llms.txt`／`.gitmodules`／`.gitattributes`／`.gitignore`） |
| `WCIS-FPAI-wiki` | `main` | `wiki/` | 文档库：本项目全部人类可读 Markdown，**同时就是 Obsidian vault** |
| `WCIS-FPAI-repos` | `master` | `repos/` | 代码库：我方代码 ＋ 厂商 FPGA demo |

三库的远端**全部在 GitHub 组织 [`WCIS-Team`](https://github.com/orgs/WCIS-Team/repositories) 下**（2026-11-06 以当时文件快照重建为无父提交的单提交根历史，D-1793967893；旧 `takomitkm/*` 地址只有 301 跳转）。组织里另有五个工程仓：`HA4FPAI`、`fmql30tai-sdi-npu-face-recognition`（原 `repos/yuanma/` 提交快照的公开形状）、`ERNIE-4.5-0.3B-PT4icraft430tai`、`Qwen3-0.6B4icraft430tai`、`Qwen2_0.5B_zg3.36.1_axi_demo`；组织外唯一例外是 `takomitkm/ekho-Linux`。仓库↔本地目录的逐条对应正本＝`wiki/repo2WCIS.repos/repo项目索引.md` 第一节，本页不复制那张表。

两个子模块各自**自带** `.gitattributes` 与 `.gitignore`——主库那两份不跨子模块生效（D-1792368000）。`repos/HA4FPAI/` 另带自有 Git 历史与远端，现势目录包含 `klar/`、`boards/`、`lang/`、`control/`、`web/`；语言链已整体收进 `HA4FPAI/lang/`，旧 `repos/lang/` **盘上已不存在**，不再是任何入口。`repos/` 顶层另有一批**只在本机、2026-11-05 起不入库**的目录（`FPAI30TAI图像FPGA/`、`official/`、`unofficial/`、`ha-small/`），判据与替代真身见 [[repo项目索引#四、本机保留不入库的目录|repo项目索引——第四节]]。

## 1.2 主库目录

```
S:\work\WCIS\
├── .gitmodules / .gitattributes / .gitignore
├── AGENTS.md  ARCHITECTURE.md  CONTRIBUTING.md  llms.txt
├── repos/        → WCIS-FPAI-repos（子模块）
├── wiki/         → WCIS-FPAI-wiki（子模块，Obsidian 入口）
├── skills/       运行中按需加载的技能（429 件／约 3.2 MB，2026-10-25 起入库）
├── source/       已提取出 Markdown 放进 wiki 的原文件      ┐
├── workenv/      临时脚本与中途产物                        ├ 只在本机一份
└── largefiles/    一次性大文件（如 JFM_Kits/ip_patch）      ┘ 永久排除、不建远程
```

`source/`、`workenv/`、`largefiles/` 由主库 `.gitignore` 永久排除，不做子模块；需要单件时直接给路径拷给别人，不要 `git add`。

## 1.3 文档库（`wiki/`）分层

```
wiki/
├── README.md          首页：项目简介、工程路线四步、目录表、阅读方式
├── dict.md  闲话.md     术语表 / 零散记录（notes.md 已删）
├── SOP/               可照抄的操作规程，一页一条链路的「怎么跑＋怎么判过了」
│   ├── MOD/           模型与工具链类（＋ test/ 子目录＝测试类；NPU调用链 从 man/ 移来）
│   └── wikimgr/       文档库与站点自身（Quartz→Pages、num-audit、同步、录音筛选、skill 检索）
├── ops/               Windows 主机与校机运维（交叉编译路由、无容器/无 WSL、Klar 上板、代理与登录、免 UAC、提权与通知）
├── PL/                可编程逻辑（pldev／plvid／plnet 三页总纲）
│   ├── FPGA/          自著页：位流手术、Verilator 长跑仿真、PL 网口 bring-up、sdi-psddr、sdi2hdmi、SDI→PS、SDI→HDMI
│   └── MOD/           NPU 模型自著页（＋ llm/ 子目录＝LLM 迁移线）
├── PS/                处理系统（Processing System）
│   ├── 板端我方代码与产物台账.md   板端落点唯一台账（#nnn 编号＋删件墓碑行）
│   ├── psrt&dev/      os.md（OS 层改动唯一归口）· psdev · 板端内存上限与心跳日志
│   │   ├── hardware/  板端设备事实页 ＋ 硬件与基础运行环境档案 ＋ raw_logs ＋ emmc/（七篇）
│   │   └── board_extract/  板载系统镜像提取件（⚠ 现被 `wiki/.gitignore` 整目录挡住，只在本机）
│   ├── oldmainchain/  HA4FPAI 统一语音链之前的主链记录：Backend（mods/audio＝全部 TTS 页）· OrchestrationLayer · WebFrontend
│   └── ha/            HA4FPAI-CI开发记录 ＋ ci-records/2026-11-05 原始证据（保留原始字节，不作普通页改写）
├── mainchain/         HA4FPAI 与视频联合规划五篇 ＋ 阶段01–05 ＋ 重构剩余工作详细清单（2026-11-07 统一归口这里）；cloud.md·s2s.md＝历史链路草图
├── repo2WCIS.repos/   与 repos/ 对应的说明页；入口 repo项目索引；FPAI30TAI图像FPGA/ 组件设计系列
├── official/          外部资料集中区（厂商手册/教程、ModelScope 模型卡、CSDN/知乎/eeworld 转载、飞书导出、Procise 手册、demo 工程转档）——⚠ 整目录不入库、站点不发布，只有 INDEX.md 入库
├── archive/           初稿与被撤销页面；承重页也住这里（开发管理/）
├── schoolhardware/    校机（Windows 宿主）SOP 2 归档：机器产物 JSON ＋ raw_logs ＋ 生成页
├── format/            校验与行尾脚本（久未更新，见 §1.5）
├── site/              Quartz 站点：make-content.py · check-plugins.py · harden-site.py · quartz.config.yaml · quartz.lock.json · robots.txt
├── todo/              待办与待验证清单（better／dev／路线要更新）
└── .github/workflows/deploy.yml  站点工作流正本
```

**2026-11-06 提交 `b51ca96` 做过一次大重构，本节这棵树是按它之后的盘面重画的**：`PS/Backend|OrchestrationLayer|WebFrontend` → `PS/oldmainchain/` 下；`PS/psrt/rt/os.md` → `PS/psrt&dev/os.md`；`PS/hardware`·`PS/emmc`·`PS/board_extract` → `PS/psrt&dev/` 下；`man/` → `SOP/MOD/` 与 `official/`；`PL/MOD/modellist`·`PL/MOD/modrt`·`PL/FPGA/FPGArt`（含 Procise 手册转档）·`fpai-blog`·厂商 `psrt` 转档 → `wiki/official/`（整目录改为不入库）；`PL/FPGA/4compile/` → `largefiles/4compile/`；`SOP/` 拆出 `MOD/test/` 与 `wikimgr/`，交叉编译类移入 `ops/`；`agentpreview/`·`完整复现/`·`notes.md`·`llms.md`·`agent.md` 删除。

分层规则：**内容按「PL／PS／SOP／archive／official」分，而不是按人分**。厂商转档、抓来的博客、取证提取件进 `RAW_DIRS`，不做页面形状与正文校验，但页名仍参与双链解析。⚠ **`RAW_DIRS` 的准头在代码里、不在本页**：`wiki/format/check-md.py` 与 `wiki/site/make-content.py` 各有一份，两份本就长期不一致（`make-content.py` 那份另与 `wiki/.gitignore` 的 `/official/*` 对齐，并靠 `git_visible()` 把不入库内容挡在站点外）；**11-06 重构后两份里的旧目录名都还没跟着改**，所以现在这两套豁免规则对 `man/`·`PL/MOD/modrt`·`PL/MOD/modellist`·`PL/FPGA/FPGArt`·`PL/FPGA/4compile` 的条目已经指不到东西。**改 `RAW_DIRS` 必须两处同改**——这条缺陷仍是登记状态，本轮未修（`wiki/format/` 你已声明久未更新，不作为规范源）。

## 1.4 承重页与数据流

三页承重，且只有这三处是「真值」：

| 页 | 管什么 |
|---|---|
| `wiki/archive/开发管理/状态源与决策记录.md` | 状态与决策唯一可信源，新增一条 `D-nn` |
| `wiki/archive/开发管理/进度计算与实时看板.md` | 进度唯一总账（§3.1 一行一卡；§3.2 由脚本回写，不手改） |
| `wiki/archive/开发管理/风险与完成判据.md` | 风险与完成判据 |

数据流：

```
状态源与决策记录 (D-nn)  ──►  进度看板 §3.1 总账表  ──►  format/progress.py  ──►  §3.2 分组完成率与总进度
                                        │
                                        └──►  site/make-content.py  ──►  Quartz  ──►  GitHub Pages
```

「一个事实只写一处」（D-1790467200）——**但"找什么去哪"不再有独立分流表**（2026-11-01 定）：`wiki/archive/开发管理/清单.md` 原有的第 3／4 节已经不存在，全库也不该再建第二张。这条查法改由**四处合起来**承担，按顺序用：

| 序 | 在哪 | 管什么 |
|---|---|---|
| ① | `AGENTS.md` §五「目录与信息放哪」 | 规矩、三库分工、承重页指向、优先级口径 |
| ② | `llms.txt`「入口文档」＋「目录」 | llm 优先读的入口清单、主库与文档库逐目录一句话 |
| ③ | `wiki/README.md`「目录」表 | **逐目录"这里放什么"＋代表页** —— 这四条里最接近原分流表的一张 |
| ④ | 本节（`ARCHITECTURE.md` §1.4） | 三处真值与数据流 |

**按卡号找判据页**不问这四处，直接看 `进度计算与实时看板` §3.1 总账表末列「详情」（列规则要求它必须是一个 `[[页面名]]`）。

## 1.5 校验与工具脚本（`wiki/format/`）

| 脚本 | 作用 |
|---|---|
| `check-md.py` | 文档结构校验：未解析双链、页名重复、锚点指不到真实标题／块、H1 与文件名不符、CRLF／BOM、Mermaid `internal-link` 节点标签（`## 相关页面` 自 D-1793628864 起不查） |
| `progress.py` | 由 §3.1 总账表推导并回写 §3.2；`--check` 只校验（CI 用），`--list` 列未完成卡，`--tick` 改一行 |
| `fix-eol.py` | 一次性把散落文件的行尾／BOM 转回来 |

⚠ **2026-11-07 更正**：本节原来还列 `lf.py`、`allconvert2md.py`、`rename/` 三项——盘上 `wiki/format/` 现在**只有上面这三份**，那三项已不在。她 2026-11-07 明确指示 `wiki/format/` 久未更新，**它的规则不当代写作规范源**：页形与协作口径看 `CONTRIBUTING.md` 与 `AGENTS.md`，这三份脚本只当读数工具。当前 `check-md.py` 全树实测基线＝236 problems／152 groups（2026-11-07），其中大头是 11-06 重构删掉的页留下的 unresolved wikilink。

两个校验脚本都按目录读，不读 git；`os.walk` 里剪掉 `site`／`qbuild`／`public`／`node_modules`／`.quartz-cache`（`BUILD_DIRS`），否则会把装配器自己的产物当原料吃掉。

## 1.6 站点链路

- **当前状态（2026-11-06 恢复，D-1793969383 推翻 D-1793566627 的停用）**：Pages 来源＝GitHub Actions，站点 **<https://wcis-team.github.io/WCIS-FPAI-wiki/>**；工作流 `Build and publish wiki site`（id `390042843`）已重新启用，仓库变量 `PUBLISH` 已打开；文档库同时被改为 **public**。站点 baseUrl 与 GitHub 导航都已改到 `WCIS-Team`。`wiki/.github/workflows/deploy.yml` 文件头保留了"停用已被取代"的说明。
- **触发定义**：push 到 `wiki` 库 `main`（不带 `paths` 过滤）或 `workflow_dispatch`。**新历史只有重建后的提交，页面历史日期因此以新历史为限**（D-1793969383 内容与边界）。
- **链路**：`actions/checkout`（`fetch-depth: 0`，日期要全史）→ 取 Quartz `v5.0.0` 到 `qbuild/` → 恢复插件缓存（key `quartz-plugins-v2-…`，命名空间已升 v2）→ **`site/check-plugins.py prepare`**（为版本不符的缓存插件**预取精确锁定 SHA**，避免浅克隆里没有锁定提交导致插件更新失败而 Quartz 仍返回成功，D-1793971282）→ `npx quartz plugin install` → **`check-plugins.py verify`**（逐插件核 HEAD 与 dist 入口）→ `plugin resolve` → `make-content.py` 装配内容 → `npx quartz build` → **`check-plugins.py output`**（检查 contentIndex 所有标题不带 `Folder:`）→ `site/harden-site.py` 加固 → 上传制品 → `deploy-pages`。
- **目录页显示**：`folder-page` 插件固定到支持 `prefixFolders` 的上游提交 `6e740cc…` 并显式 `prefixFolders=false`（去 `Folder:` 前缀）；文件夹页不再清空右栏，继承关系图与反向链接（D-1793970553）。关系图组件标题由 `harden-site.py` 就地替换为 `Graph View (Ctrl+G)`，找不到任何图标题即构建失败（D-1793972954）。
- 装配口径：**库根即内容树**（`TREES = ("",)`、`HOME_PAGE = "README.md"` 落站点根 `index.md`），所以站点页面路径＝库内路径；不入库的内容（`official/`、`board_extract/`、体积类与凭据具名件）由 `RAW_DIRS` 与 `git_visible()` 两道闸挡在发布集合外。
- 开关：仓库变量 `PUBLISH`——不存在＝照常发布，建成 `false` 只跳过部署（站点冻在最后一版，URL 不 404）；要停构建另需 Disable workflow。
- 爬虫：全站 deny-all（`site/robots.txt` ＋ 每页 `noindex` ＋ 删 `sitemap.xml`），由 `harden-site.py` 在构建后注入，注不进去当场失败。

---

# 二、作品系统架构

## 2.1 定位与形态

**CPU＋FPGA＋NPU 异构多模态边缘智能终端**。以下为 2026-11-04 源码与既有验证记录对齐后的结构，区分已实现接口、历史板端测试和待集成能力；不代表本轮重新探测板端在线状态。

- **视觉工程**：开发树 `repos/FPAI30TAI图像FPGA/` 包含 SDI 采集、FAC1 几何预处理、NPU 人脸身份与 PAD、VPU 硬编码、预览和 eMMC 录像（该目录 2026-11-05 起**只在本机、不入库**）。对外发布的交付形状是 [WCIS-Team/fmql30tai-sdi-npu-face-recognition](https://github.com/WCIS-Team/fmql30tai-sdi-npu-face-recognition)＝`repos/fmql30tai-sdi-npu-face-recognition/`（`源码/fpga_source`·`ps_source`·`board_runtime`·`fpga_restore`·`logpack`＋`FMQL30TAI-Install/`＋带原许可的 `models/`），厂商 Confidential 件按其 `.gitignore` 排除。详见 `wiki/repo2WCIS.repos/FPAI30TAI图像FPGA/FPAI30TAI图像FPGA索引.md` 与 [[repo项目索引#七、视频交付仓的目录形状|repo项目索引——第七节]]。
- **系统软件**：`repos/HA4FPAI/` 集中上层控制、语言链及交互界面。`lang/server/` 编排语音组件，`lang/npuserv/` 做 NPU 槽位监督（**归 Qoder 会话**），`klar/` 负责 NLU 集成和执行器，`boards/` 提供控制台与设备验证框架，`control/` 是统一控制与资料层骨架（SQLite 业务主库＋版本化 HTTP＋持久 outbox，见 [[HA4FPAI与视频联合总规划]]），`web/` 提供试验台和原生 HMI；模型交付线以子模块挂在 `lang/llm/models/` 与 `lang/tts/ekho-linux`。
- **另一条独立线**：`repos/ha-small/` 是 Home Assistant Core 2026.10.4 的删除式裁剪产物，**只在本机暂存、无远端**，归属定为 `HA4FPAI` 的 `ha-small` 分支材料（D-1794005260 修正 D-1793914150）；它不替代上面的 HA4FPAI 重构线。
- **已有板端语音验证**：重烧后以在线 ASR、云端 LLM 和 Edge TTS 恢复服务；HTTP、Wyoming、文件模拟对话和严格文本切块分别留下记录。真实麦克风、扬声器以及最新十级链整链验收仍待完成。
- **后续集成**：离线 LLM 问答、语音触发看图/OCR、融合身份授权与真实门锁动作均不能由早期规划推定为已交付。

状态与验证窗口以 `wiki/archive/开发管理/状态源与决策记录.md` 最新记录和 `wiki/PS/板端我方代码与产物台账.md` 为准；看板任务卡计数仍按其原有验收规则，不用专项测试替换整机完成率。

## 2.2 两条感知链路

**视觉链路**：SDI → PL 采集 → PS DDR 原帧 → 原帧上传 PL DDR → FAC1 检测补边/逐脸五点对齐 → NPU YuNet／MobileFaceNet／MiniFASNet → PS 坐标回算、身份比对及逐脸 PAD 状态。媒体分支由 VPU 产生一路 H.264，供网页预览及独立录像服务消费。PAD 模型输出与安全阈值验收分开，不能据此承诺安全开锁。

**语言链当前源码**：

```text
sar → rms → kws → kws_route → vad → asr → router → llm → tokenizer → tts
前处理  能量门  唤醒  关键词路由  切段  转写  意图路由  对话  文本切块  合成
                                                        ↓
                                      逐块音频拼接 → 输出重采样 → WAV
```

`sar` 是 CPU 音频前处理/重采样；VPU 归属视频编码。`rms` 可短路静音，`kws_route` 决定继续、停止或固定回应；ASR 后的 `router` 可选 klar，验证分支 `verify` 仍留空。当前十级实现与自检位于 `repos/HA4FPAI/lang/server/`；最新十级专项记录是主机测试，不能直接视为全链上板。板端已验证的在线链与 `tts_split` 换档分别见 D-1793767689、D-1793768465、D-1793769625；十级变化见 D-1793773995。后续 D-1793775864 与 `workenv/lang-deploy/final-online-verification.json` 已留存十级选档回显、模型文件就绪和服务健康检查，未运行真实推理请求，仍不能补成新版整链功能回归。

## 2.3 意图路由与输出

| 请求类型 | 实现与验证边界 |
|---|---|
| 固定设备指令 | `klar/klar_executor.py` 先匹配命令表，再调用视觉 REST；历史 8/8 是文本/API 层验证 |
| 通用意图 | 引入的 Rust klar 引擎解析意图与槽位，自建执行器映射动作；新 `lang/server` klar 路由与实际引擎协议还需联调 |
| 语音问答 | 已记录板端在线 ASR → 云 LLM → Edge TTS；文件模拟输入完成一轮状态迁移 |
| 本地模型 | sherpa 档可替换；本地 LLM 为可选实验，`npuserv` 的文本/ids 桥未接，当前不承诺断网问答 |
| 抓拍/OCR 后问答 | 保留集成方向，不能把视觉接口存在写成语音触发已通过 |
| 身份授权 | 人脸/PAD 输出、声纹组件及验证框架各自留证；`verify` 未接、PAD 未安全标定、真实门锁未接 |
| 回答输出 | 文本按严格上限切块 → TTS → 音频拼接/重采样；板端有 WAV 结果，真实扬声器播放待验 |

## 2.4 计算资源分工

| 单元 | 当前职责与边界 |
|---|---|
| FPGA／PL | SDI 数据通路与 FAC1 几何预处理；早期统一音频前端、OCR 共享前端仍按各自证据验收 |
| NPU | 已有视觉模型与声码器/LLM专项实验；不把这些实验概括为 ASR、OCR、LLM 全部并行部署 |
| PS／CPU | Linux 采集、SDK 所有权、身份比对、音频前处理、文本切块、配置、调度、REST 与界面 |
| VPU | H.264 硬件编码，预览和录像复用同一编码流 |
| 云端 | 当前板端在线档的 ASR、LLM 与 Edge TTS；断网不能完成该档问答 |

## 2.5 服务边界与交互

| 模块 | 职责与约束 |
|---|---|
| 视觉 owner／交接控制 | 单一设备所有者持有 SDK 与采集资源；需占用同一硬件的任务按关闭回执与交接事务切换。在线语音服务存在不等于它已经接入 NPU 交接 |
| `lang/server` | HTTP、Wyoming、对话循环三个入口；组件按配置装配和缓存；同一进程内 sherpa 档共享运行时，独立服务进程不共享模型内存 |
| `lang/config` | 一组件一 YAML；服务启动时读取，修改后需重启对应进程 |
| `lang/npuserv` | 独立 worker 的 NPU 槽位监督，避免把 CMA 生命周期混入语音服务；文本推理桥仍待接 |
| `klar`／`boards` | 确定性 NLU、双通道执行器、控制台和设备验证框架；外部 HA 接口与板端自行实现的职责分开 |
| `web/ggradio` | 进程内逐段试验、配置面板和产物下载；另托管 HTTP 子进程，保存、重载试验链、重启服务是不同动作 |
| `web/offlineHMI` | 可断网本机操作的 kiosk 自助终端，直接接 HDMI 显示器与 USB 鼠标；以回环接口访问本地视觉/录像服务，在线语音另需网络。PNG 预演与 udmabuf 两后端、evdev 输入、四功能页；已有 PC/校机预演，真 HDMI 与鼠标联合验收待做 |

## 2.6 工程实施路线（四步）

一句话：**先让现成的东西全部跑起来，再一个一个换掉它。**

| 步骤 | 内容 | 判据 |
|---|---|---|
| ① NPU 直接部署 | 现成模型直接部署到 NPU | 跑不通即换模型，只求跑通 |
| ② CPU 可用链 | 用现成软件在 CPU 上跑通全链路 | 冻结为黄金基线 `baseline-v1` |
| ③ FPGA 组件下沉 | 适合的组件逐个从 CPU 下沉到 FPGA 重写 | 每换一处即与基线比对，确保不回退 |
| ④ 自定义算子补缺 | NPU 缺失算子的模型用 FPGA 自定义硬算子补齐 | **条件项**，只在实测发现值得硬件化的瓶颈后才评估 |

这是实施与验收方法，不能据表格判断四步已完成。卡号、冻结基线和整机回归仍看 `wiki/archive/开发管理/进度计算与实时看板.md`；该总账最后刷新日期为 2026-09-28，专项进展另有后续记录，本轮不据此重算总进度。链路页：`wiki/archive/开发管理/实施路线/`。

## 2.7 目标平台

复旦微 FMQL30TAI／FPAI30TAI 悟净板：PS 为 4 核 Cortex-A53（最高 1 GHz）＋PL（125K 逻辑、78 600 LUT）＋NPU（ZG330 后端档）。板上内存 993 MB，CMA 池 256 MB。术语见 `wiki/dict.md`。

---

## 相关文件

- 全仓索引：`llms.txt`
- 干活纪律与 `#SOP` 约定：`AGENTS.md`
- 写作、校验、行尾、站点、推送规范：`CONTRIBUTING.md`
- 文档库首页：`wiki/README.md`；系统架构旧稿：`wiki/archive/02-系统架构.md`
