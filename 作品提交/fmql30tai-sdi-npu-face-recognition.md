# fmql30tai-sdi-npu-face-recognition

WCIS 的视频交付工程公开仓——SDI 采集 + FPGA 视觉 + NPU 人脸身份/PAD 的对外发布形状（原 `repos/yuanma/` 提交快照的公开形态）。

- 仓库：`https://github.com/WCIS-Team/fmql30tai-sdi-npu-face-recognition`（分支 `main`）
- 内容：`源码/fpga_source`·`ps_source`·`board_runtime`·`fpga_restore`·`logpack`＋`FMQL30TAI-Install/`＋带原许可的 `models/`；厂商 Confidential 件按该仓 `.gitignore` 排除。
- 本地子模块位置：`repos/fmql30tai-sdi-npu-face-recognition/`（pin `c083ec7`）。
- 该工程在整体交付中的角色与架构：见 [ARCHITECTURE.md](../ARCHITECTURE.md) 与 [repo项目索引](../../wiki/repo2WCIS.repos/repo项目索引.md)。

## 来源与署名

- **原作者**：`anxianji11-pixel`（<https://github.com/anxianji11-pixel>），WCIS-FPAI 的 contributor。视频链路（SDI 采集＋FPGA 视觉＋NPU 人脸身份/PAD）的原创部分归其所有。
- **代上传**：本公开仓由 `takomitkm` 代其建仓并上传；署名正本见该仓 [`NOTICE.md`](https://github.com/WCIS-Team/fmql30tai-sdi-npu-face-recognition/blob/main/NOTICE.md) 与 [README「来源与署名」](https://github.com/WCIS-Team/fmql30tai-sdi-npu-face-recognition/blob/main/README.md#来源与署名)，第三方许可与例外见 `THIRD_PARTY_NOTICES.md`／`LICENSE_SCOPE.md`。
- **历史边界**：2026-11-06 三个组织库按文件快照重建根历史（D-1793967893），旧提交祖先被丢弃，故 `WCIS-FPAI`／`WCIS-FPAI-repos`／`WCIS-FPAI-wiki` 的贡献视图里不含其对视频链路的旧提交；**视频子模块仓 `WCIS-Team/fmql30tai-sdi-npu-face-recognition` 自身历史不在重建范围、署名完好**（根提交 `449bcf9` 的 author 即 `anxianji11-pixel`）。本轮以带 `Co-authored-by` 尾注的新提交补记署名，不改写历史、不恢复旧提交；口径见 D-1794150604。

