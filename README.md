# ats-mini-hub

Static OTA catalog + firmware images for the ATS Mini board (`Launcher/boards/ats-mini`).
Fetched at runtime by the Launcher firmware — see `ats-mini-ota.json`.

## Entries

| fid | name | source release (tag) | image_size | appSize (app slice) | data partition |
|---|---|---|---|---|---|
| `bruce-ats-mini` | Bruce ATS Mini | [`wendells01/bruce-ats-mini` `v1.0.0`](https://github.com/wendells01/bruce-ats-mini/releases/tag/v1.0.0) | 3786496 | 3720960 | spiffs 0x82, USE_REMAINING |
| `ats-mini-original` | ATS Mini Original | [`esp32-si4732/ats-mini` `v2.38`](https://github.com/esp32-si4732/ats-mini/releases/tag/v2.38) — merged image re-hosted as [`wendells01/Launcher` `ats-mini-original-v2.38`](https://github.com/wendells01/Launcher/releases/tag/ats-mini-original-v2.38) (upstream ships only split zips) | 8388608 | 1698544 | littlefs 0x83, 0x70000 |
| `ats-mini-english` | Chinese Firmware | this repo, `ats-mini-english-v2025.09.22` (version = latest changelog entry 2025-09-22) | 3378864 | 3313328 | spiffs 0x82, 0x70000 |
| `ats-mini-marauder` | ATS-Mini Marauder | this repo, `ats-mini-marauder-v2026.09.09` (version = in-binary build-date string) | 8388608 | 2022928 | littlefs 0x83, 0x70000 |

`image_size` = merged factory image length. `appSize` = measured ESP-image app payload
(magic `0xE9` at `0x10000`, segment walk, `align16+1(checksum)+32(hash)+align16`;
Bruce/English canonical ends land 16 B past EOF so the SD-path tail fallback
`file.size()-0x10000` applies — identical bytes). Data-partition subtype/size measured
from each merged's embedded partition table at `0x8000`; sizing mirrors the Hub
manifest branch (`LAUNCHER_DEFAULT_SPIFFS_THRESHOLD` = 0x500000 on ats-mini).

## Phase 2 release checklist (orchestrator)

Publish these two release assets exactly (names/sizes must match the JSON):

- Tag `ats-mini-english-v2025.09.22`, asset `ats-mini-english-v2025.09.22-merged.bin`
  (3378864 B, md5 `dcb1d356ade3c616dd48f29bcca89f2c`) — merged with
  `esptool --chip esp32s3 merge_bin --flash_mode dio --flash_freq 40m --flash_size 16MB`
  from `bootloader.bin` + `partitions.bin` + `firmware-english.bin`.
- Tag `ats-mini-marauder-v2026.09.09`, asset `ats-mini-marauder-v2026.09.09-merged.bin`
  (8388608 B, md5 `44d81a3c10d73a7aa51c6102ebf148d3`).
