# ats-mini-hub

Static OTA catalog + firmware images for the ATS Mini board (`Launcher/boards/ats-mini`).
Fetched at runtime by the Launcher firmware — see `ats-mini-ota.json`.

## Entries

| fid | name | source release (tag) | image_size | appSize (app slice) | data partition |
|---|---|---|---|---|---|
| `bruce-ats-mini` | Bruce ATS Mini | [`wendells01/bruce-ats-mini` `v1.17.0`](https://github.com/wendells01/bruce-ats-mini/releases/tag/v1.17.0) (upstream Bruce 1.17; `v1.0.0` kept as rollback) | 4171552 | 4106016 | spiffs 0x82, USE_REMAINING |
| `ats-mini-original` | ATS Mini Original | [`esp32-si4732/ats-mini` `v2.46`](https://github.com/esp32-si4732/ats-mini/releases/tag/v2.46) — merged image re-hosted as [this repo `ats-mini-original-v2.46`](https://github.com/wendells01/ats-mini-hub/releases/tag/ats-mini-original-v2.46) (upstream ships only split zips; `v2.45`, `v2.39` and `v2.38` releases retained as rollback) | 8388608 | 1800736 | littlefs 0x83, 0x70000 |
| `ats-mini-pv2ksm` | ATS Mini PV2KSM | [`PV2KSM/ats-mini-pv2ksm` `r30D`](https://github.com/PV2KSM/ats-mini-pv2ksm/releases/tag/r30D) OSPI OTA | 2681392 | 2681392 (offset 0) | littlefs + settings NVS |
| `ats-mini-english` | Chinese Firmware | [this repo `chinese-firmware`](https://github.com/wendells01/ats-mini-hub/releases/tag/chinese-firmware) (version = latest changelog entry 2025-09-22) | 3378864 | 3313328 | spiffs 0x82, 0x70000 |
| `ats-mini-marauder` | ATS-Mini Marauder | [this repo `marauder-firmware`](https://github.com/wendells01/ats-mini-hub/releases/tag/marauder-firmware) (version = in-binary build-date string) | 8388608 | 2022928 | littlefs 0x83, 0x70000 |
| `ats-mini-resonik` | Resonik ATS Mini | [`ep3mhb/resonik-ats-mini` `v2.45-r4`](https://github.com/ep3mhb/resonik-ats-mini/releases/tag/v2.45-r4), merged re-hosted as [this repo `ats-mini-resonik-v2.45-r4`](https://github.com/wendells01/ats-mini-hub/releases/tag/ats-mini-resonik-v2.45-r4) | 8388608 | 2052720 | littlefs 0x83, 0x70000 + settings 0x10000 |
| `ats-mini-salmer` | ATS Mini Salmer | [`wsalmi/ats-mini-salmer` `v3.01`](https://github.com/wsalmi/ats-mini-salmer/releases/tag/v3.01), merged re-hosted as [this repo `ats-mini-salmer-v3.01`](https://github.com/wendells01/ats-mini-hub/releases/tag/ats-mini-salmer-v3.01) | 8388608 | 1722624 | littlefs 0x83, 0x70000 + settings 0x10000 |

`image_size` = merged factory image length. `appSize` = measured ESP-image app payload
(magic `0xE9` at `0x10000`, segment walk, checksum byte at `align16(segments_end)-1`,
32 B SHA-256 at the next 16-aligned offset;
Bruce/English canonical ends land 16 B past EOF so the SD-path tail fallback
`file.size()-0x10000` applies — identical bytes). Launcher v1.0.4+ derives
Bruce's slice from `image_size` alone (the merged asset has no trailing padding),
so a catalog-only edit is enough for a Bruce bump. Data-partition
subtype/size measured from each merged's embedded partition table at `0x8000`;
sizing mirrors the Hub manifest branch (`LAUNCHER_DEFAULT_SPIFFS_THRESHOLD` =
0x500000 on ats-mini).

**Re-measure `appSize` on every upstream bump.** Upstream keeps `image_size`
identical (8388608) across releases while the app payload grows — v2.38 1698544,
v2.39 1698336, v2.41 1796208, v2.46 1800736. Launcher v1.0.3+ takes the app range
from the catalog, so a stale `appSize` flashes a truncated image and fails
validation; a value *larger* than the real payload is harmless (the extra bytes are
`0xFF` padding and `esp_image_verify` reads declared segments only). Launchers
before v1.0.3 fall back to their built-in table, which can still reject new sizes.

## Release checklist (orchestrator)

Each entry below is a live release on this repo; names/sizes must match the JSON:

- Tag `chinese-firmware`, asset `Chinese-Firmware`
  (3378864 B) — merged with
  `esptool --chip esp32s3 merge_bin --flash_mode dio --flash_freq 40m --flash_size 16MB`
  from `bootloader.bin` + `partitions.bin` + `firmware-english.bin`.
- Tag `marauder-firmware`, asset `Marauder-Firmware`
  (8388608 B).
- Tag `ats-mini-original-v2.46`, asset `ats-mini-original-v2.46-merged.bin`
  (8388608 B, md5 `75b6c0c2531e236852cf0abf2d3840c0`) — the upstream OSPI zip's
  own `ats-mini.ino.merged.bin`, re-hosted byte-for-byte (no appended padding).
- Tag `ats-mini-original-v2.45`, asset `ats-mini-original-v2.45-merged.bin`
  (8388609 B) — retained as rollback; it is upstream's merged plus one `0xFF`
  byte to tell it apart from v2.41 on Launchers that predate the catalog ranges.
- Tags `ats-mini-original-v2.41`, `v2.39` and `v2.38`, same asset pattern
  (8388608 B each) — historical; superseded by v2.46.
- Tag `ats-mini-resonik-v2.45-r4`, asset
  `resonik-ats-mini-v2.45-r4-merged.bin` (8388608 B, md5
  `d51a934f75da827ef770007b6b696cc6`) — upstream's own `ats-mini.ino.merged.bin`
  from the OSPI zip, re-hosted byte-for-byte.
- Tag `ats-mini-salmer-v3.01`, asset `ats-mini-salmer-v3.01-merged.bin`
  (8388608 B, md5 `4630dfd902118fe66e92dc6d7ebfa56d`) — the upstream OSPI release
  asset re-hosted byte-for-byte.

## Launcher versions

Original v2.46 ships upstream's merged image unmodified (8388608 B), the same size
as v2.41 — the exact size no longer distinguishes releases because Launcher
v1.0.3+ reads `app_offset`/`app_size` from the catalog. Older Launchers match the
stale table entry and fail image verification instead of activating a truncated
app. PV2KSM r30D uses the upstream OSPI OTA binary.

- v1.0.3 reads `app_offset`/`app_size` from the catalog, so it can install Bruce
  v1.17.0. It ignores `direct_file` and downloads from `file`.
- v1.0.4 adds `direct_file` raw-blob downloads and derives Bruce's app slice
  from `image_size` when the catalog omits the app fields.
- v1.0.5 reads data partitions from the catalog (`data_partitions` on a version),
  so a new firmware family needs no Launcher change. Entries without the field
  still use the built-in table.
- v1.0.0 ignores the app ranges entirely: it rejects Bruce v1.17.0
  (`Static image changed`). Do not use it for this catalog.

OTA and settings persistence still require hardware verification.

## Dynamic install metadata

Each version in `details` supplies `image_size` (download bytes), `app_offset`
(byte offset of the ESP app) and `app_size` (bytes to flash). Update all three
from the actual release artifact for each new version, and validate the app
image and board variant before publishing. The Launcher rejects invalid ranges
and verifies the ESP image before switching. Launcher v1.0.3+ reads these
fields; v1.0.0 falls back to its built-in table and fails closed on unknown
sizes.

An optional `data_partitions` array (`label`, `subtype`, `size`; up to 4 entries,
labels unique and ≤ 15 chars) describes the data partitions the Launcher must
create next to the app slot. Launcher v1.0.5+ reads it; entries without it use
the built-in table below.

`size` is only applied when the Launcher has to *create* the partition; an
existing label keeps its current size. So `0x70000` is the safe value for
littlefs: it is what the original family already creates, and the firmware
mounts by label and adapts to the real size. Declaring the firmware's own
`0x1D0000` would reserve 1.75 MB of free space for a partition that holds a few
kilobytes of settings.

Built-in fallback table (`atsMiniStaticDataPartitions`, Launcher ≤ v1.0.4),
using `LAUNCHER_DEFAULT_SPIFFS_SIZE` (0x70000) unless noted:

- `bruce-ats-mini` → spiffs 0x82, size = use remaining.
- `ats-mini-original`, `ats-mini-pv2ksm` → littlefs 0x83 + `settings` 0x10000.
- `ats-mini-english` → spiffs 0x82.
- `ats-mini-marauder` → littlefs 0x83.
- anything else → error before flashing.

## Direct GitHub API downloads

The catalog's optional `direct_file` points to an immutable Git blob under this
repository. The Launcher requests raw bytes from that API endpoint with HTTPS,
checks the advertised image and app ranges, and verifies the ESP image before
activating it. Original release URLs remain in `file` for older Launchers.
The Launcher caches the catalog for 60 seconds to reduce API requests; GitHub's
unauthenticated API rate limits still apply.

For a new compatible release, verify the source and board variant, place its
binary in `firmware-api/`, compute `git hash-object` and update `direct_file`,
`image_size`, `app_offset` and `app_size` together before publishing. This is
a manual update, not an automatic upstream polling service. A family whose data
layout differs from the built-in table ships `data_partitions`; only a change
the catalog cannot express (e.g. a different app partition table) still needs a
Launcher update.
