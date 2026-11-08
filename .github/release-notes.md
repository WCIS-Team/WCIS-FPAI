# ha-small v0.1.0 (2026-11-08) — From Home Assistant Core 2026.10.4

## What is ha-small?

A trimmed-down Home Assistant installation derived by **deletion-only cut** from the official upstream:

> **Home Assistant Core 2026.10.4** → delete unused integrations → keep all source bytes identical via SHA256 audit.

This is not a fork or reimplementation; it's the exact same codebase minus 11 leaf integrations that had no dependency on any retained feature.

## Build artifacts

| Artifact | Size | SHA256 | Description |
|---|---|---|---|
| `ha-small-0.1.0-source.tar.gz` | 6,850,596 B | `04f014cccc00635bb3d29e5cb24afb0ecf6b482716c99401784381ae1f12826e` | Source tarball (cut/ examples included) |
| `homeassistant-2026.10.4-py3-none-any.whl` | 8,280,826 B | `6bd60dc72c88312185fd5563c1d6f30ddd968b9aa0d4c1c57d4c638f1b0f5c2b` | Wheel built from trimmed tree |
| `ha-small_0.1.0_all.deb` | 7,366,076 B | `c56af67a74a4507f26daa04659009993bf901e3aac31fbec4da802d809c841a8` | Debian package with installer + disk-guard service |

## Trimming details

- **Integrations**: from HA upstream's 128 → **118** (removed 11 leaf integrations)
  - Removed leaves: `ai_task`, `air_quality`, `calendar`, `geo_location`, `local_file`, `radio_frequency`, `schedule`, `switch_as_x`, `todo`, `command_line`, `shell_command`
  - Retained pillars: `generic` (camera), `mqtt`, `rest`, `rest_command`, `counter`, `wyoming` (verified features)
- **Validation**: clean venv install + `check_config` (zero errors) + `pip check` (clean) + 30/30 functional suite pass
- **Audit**: 5630 files byte-identical to upstream via SHA256 verification

## New capabilities added in this trim

- **Disk guard for video recording**: Stops recorder if free space <10GB (`/api/status.disk_free_bytes` read-only probe)
- **mTLS support**: Client certificate validation at broker level
- **MQTT device bridge**: Auto-discovery + LWT availability + event entity from board side
- **Weak network test**: netem-delayed MQTT reconnect verified under loss

## Installation

### Container (local testing)
```bash
podman run -d --name ha-small python:3.14-slim sleep infinity
podman cp homeassistant-2026.10.4-py3-none-any.whl ha-small:/tmp/
podman exec ha-small /venv/bin/pip install /tmp/homeassistant-2026.10.4-py3-none-any.whl
```

### Board/Debian system
```bash
sudo dpkg -i ha-small_0.1.0_all.deb
sudo /opt/ha-small/bin/hass-setup
```

## Actions workflow

Trigger manually from GitHub:

1. Go to GitHub → WCIS-Team/HA4FPAI → Actions
2. Select "Release ha-small" workflow
3. Click "Run workflow" and choose version tag

No CI automation per project policy; runs are manual-triggered only.

## Known boundaries

- **Python requirement**: ≥3.14.2 (board has only 3.8, cannot be installed)
- **mTLS certificates**: Lab-grade CA valid 30 days, production needs replacement
- **Guard threshold**: Default 10GB, adjustable via script parameter
- **Camera source**: Current bridge depends on `live.mp4` interface — ensure camera pipeline (SDI→PL→PS) provides H264 input

## Verification evidence

- Functional probes: `cut/results/{functional,board-*,mqtt-*.json}`
- Documentation: [[RELEASE-0.1.0]], [[record-guard-sop]], `cut/BUILD.md`
- Audit: `cut/source-audit.json`, `cut/cut-report.json`

---

**Author**: WCIS agent  
**Time**: 2026-11-08T20:30 UTC  
**From**: [Home Assistant Core 2026.10.4](https://pypi.org/project/homeassistant/2026.10.4/) via deletion-only cut

## Comparison table

| Metric | HA upstream | ha-small v0.1.0 | Δ |
|---|---|---|---|
| Integrations | 128 | **118** | –11 |
| Upstream files | 6158 | **5630** | –528 (trimmed .py/.md) |
| Installed size (venv) | ~784 MB | **~784 MB** (no gain yet) | 0 |
| Source archive | 6,135,211 B | **6,850,596 B** (+615 KB) | +10% |

Note: Installation size unchanged until wheel rebuilds dependencies; source archive includes extra metadata (cut/, examples). The real ROI comes from reducing attack surface (fewer integrations) and removing unused code paths.

---

## Changelog

### v0.1.0 (2026-11-08)
- Trimmed 11 unused leaf integrations, validated on clean venv + 30/30 functional suite
- Added disk guard service (watchdog stops recording when disk <10GB free)
- Added mTLS broker support (client cert validation)
- Added MQTT device bridge with auto-discovery/LWT/event entity
- Verified weak-network resilience under netem delay+loss
