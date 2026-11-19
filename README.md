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

**Re-measure `appSize` on every upstream bump of the *tabulated* fids** (Original,
PV2KSM, English, Marauder). Upstream keeps `image_size` identical
(8388608) across releases while the app payload grows — v2.38 1698544, v2.39 1698336,
v2.41 1796208. The Launcher's exact-match guard keys on `image_size` only, so it cannot
detect the growth; a stale `appSize` flashes a truncated image and fails validation.
A stale value is harmless only while it is *larger* than the real payload (the extra
bytes are `0xFF` padding and `esp_image_verify` reads declared segments only).

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
sizes. Firmware families and data partitions remain defined in the Launcher;
changes to those can still require a Launcher update.

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
a manual update, not an automatic upstream polling service. Changes in the
firmware's partition needs may still require a Launcher update.
