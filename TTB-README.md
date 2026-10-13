# Team Turtlebots fork — changes from upstream

This is Team Turtlebots' fork of [nasa-jpl/open-source-rover](https://github.com/nasa-jpl/open-source-rover). It is used to plan and track our own rover build. This file lists every file we have added or changed relative to upstream, and what each one is for. Everything else in the repo is unchanged upstream content.

## Added files

| File | Purpose |
| --- | --- |
| [TTB-README.md](TTB-README.md) | This file. An index of the fork's changes. |
| [parts_list/extra_parts_shopping_list.csv](parts_list/extra_parts_shopping_list.csv) | A shopping list built from [extra_parts.md](parts_list/extra_parts.md). It has one row per purchase option, with vendor, SKU/part number, link and notes. Alternatives for the same item share an `Item #` and are lettered in the `Option` column. The `Purchased` column tracks what we've bought. |
| [parts_list/customized_parts.md](parts_list/customized_parts.md) | Our deviations from the official build. Entry 1 covers powering the rover from **Ryobi ONE+ 18V** tool batteries instead of a 4S LiPo: compatibility check, parts, wiring, build steps, Roboclaw and rover-software settings, and a first power-up checklist. Entries 2–5 cover the build workspace: a 24" x 80" silicone workbench mat, MotoFloor garage floor tiles, a solder fume extractor, and a 0–30V bench DC power supply. |

## Modified files

| File | Change | Purpose |
| --- | --- | --- |
| [parts_list/parts_list.csv](parts_list/parts_list.csv) | Added a `purchased` column | Track purchase progress for the mechanical (goBILDA) parts |
| [parts_list/digikey_bom.csv](parts_list/digikey_bom.csv) | Added a `Purchased` column. Values are no longer quoted, and prices are reformatted (e.g. `0.16400` → `0.164`). | Track purchase progress for the electrical (DigiKey) parts |

## Notes

- **The generated parts README is not updated.** [parts_list/README.md](parts_list/README.md) is generated from the two CSVs by [csv_to_md.py](parts_list/csv_to_md.py). The script reads columns by position, so the extra purchase columns are ignored. [customized_parts.md](parts_list/customized_parts.md) and the shopping list are not included in the generated README.
- **Keep the CSVs in UTF-8.** Editing `parts_list.csv` in a spreadsheet once turned its non-ASCII characters (`™`, `®`, and a full-width `２`) into `?`. This has been fixed. When editing in Excel, use *Save As > CSV UTF-8*.
- **Not intended for upstream.** The purchase columns and the shopping list are specific to our build. If anything here is worth contributing back, such as `customized_parts.md`, submit it as a separate pull request without the purchase-tracking changes.
- **Syncing with upstream.** Upstream edits to `parts_list.csv` and `digikey_bom.csv` will conflict with our added columns. Resolve the conflicts by keeping upstream's values and re-adding our purchase marks.
