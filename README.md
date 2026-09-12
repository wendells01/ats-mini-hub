# ats-mini-hub

Static OTA catalog + firmware images for the ATS Mini board (`Launcher/boards/ats-mini`).
Fetched at runtime by the Launcher firmware — see `ats-mini-ota.json`.

## Entries

| fid | name | source release (tag) | image_size | appSize (app slice) | data partition |
|---|---|---|---|---|---|
| `bruce-ats-mini` | Bruce ATS Mini | [`wendells01/bruce-ats-mini` `v1.0.0`](https://github.com/wendells01/bruce-ats-mini/releases/tag/v1.0.0) | 3786496 | 3720960 | spiffs 0x82, USE_REMAINING |
| `ats-mini-original` | ATS Mini Original | [`esp32-si4732/ats-mini` `v2.38`](https://github.com/esp32-si4732/ats-mini/releases/tag/v2.38) — merged image re-hosted as [this repo `ats-mini-original-v2.38`](https://github.com/wendells01/ats-mini-hub/releases/tag/ats-mini-original-v2.38) (upstream ships only split zips) | 8388608 | 1698544 | littlefs 0x83, 0x70000 |
| `ats-mini-english` | Chinese Firmware | [this repo `chinese-firmware`](https://github.com/wendells01/ats-mini-hub/releases/tag/chinese-firmware) (version = latest changelog entry 2025-09-22) | 3378864 | 3313328 | spiffs 0x82, 0x70000 |
| `ats-mini-marauder` | ATS-Mini Marauder | [this repo `marauder-firmware`](https://github.com/wendells01/ats-mini-hub/releases/tag/marauder-firmware) (version = in-binary build-date string) | 8388608 | 2022928 | littlefs 0x83, 0x70000 |

`image_size` = merged factory image length. `appSize` = measured ESP-image app payload
(magic `0xE9` at `0x10000`, segment walk, `align16+1(checksum)+32(hash)+align16`;
Bruce/English canonical ends land 16 B past EOF so the SD-path tail fallback
`file.size()-0x10000` applies — identical bytes). Data-partition subtype/size measured
from each merged's embedded partition table at `0x8000`; sizing mirrors the Hub
manifest branch (`LAUNCHER_DEFAULT_SPIFFS_THRESHOLD` = 0x500000 on ats-mini).

## Release checklist (orchestrator)

Each entry below is a live release on this repo; names/sizes must match the JSON:

- Tag `chinese-firmware`, asset `Chinese-Firmware`
  (3378864 B) — merged with
  `esptool --chip esp32s3 merge_bin --flash_mode dio --flash_freq 40m --flash_size 16MB`
  from `bootloader.bin` + `partitions.bin` + `firmware-english.bin`.
- Tag `marauder-firmware`, asset `Marauder-Firmware`
  (8388608 B).
- Tag `ats-mini-original-v2.38`, asset `ats-mini-original-v2.38-merged.bin`
  (8388608 B) — merged from the upstream OSPI zip (upstream ships only split zips).
