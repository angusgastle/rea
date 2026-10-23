# Project Spore — Evidence Ledger

Conventions: `PROJECT_BRIEF.md` §10. One entry per investigation; every substantive
conclusion cites evidence; executable identity is by SHA-256, not filename.

## Corpus registry

| ID         | File                                                                                    | SHA-256                                                            | Lineage                           |
| ---------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | --------------------------------- |
| M-DISC     | `spore_master/SPORE.cdr`                                                                | `2c2d5bce716175c0eb7ea451de1246b41d7f23c5a80c8cd7fa5eaa88f67fc773` | Retail disc (preservation master) |
| M-GOG-EXE  | `spore_master/SPORE.Collection.v3.1.0.29/setup_sporetm_collection_3.1.0.29_(77221).exe` | `9412ae86c313478f00982eee761c4faaac09da79ba71d9065b1f74e2b6686e0e` | GOG installer                     |
| M-GOG-BIN1 | `…_(77221)-1.bin`                                                                       | `d6a0771e0ec0c9e13031f6354762b29caf50bdca6e753a076dd2b285d94c66be` | GOG installer data                |
| M-GOG-BIN2 | `…_(77221)-2.bin`                                                                       | `19e751daceb665035b1b98eb7a39d71cf4e557e688d5ed30fe779c1de745e002` | GOG installer data                |
| A-GOG-EP1  | `spore_work/gog-3.1.0.29/SporebinEP1/SporeApp.exe` (FileVersion 3.1.0.29)               | `dc04aee5a3debc3f1ad4c1a937460e99a29b9bd3bc285008be83615dd5e59a37` | GOG — analysis candidate          |
| A-GOG-BASE | `spore_work/gog-3.1.0.29/SporeBin/SporeApp.exe` (FileVersion 1.3.0.29)                  | `25d42a7a5c4d438fb155233230f57d29e2849bfdff5c889a5d0847f0469d914e` | GOG — analysis candidate          |

Full per-file hashes: `manifest-disc.json`, `manifest-gog-3.1.0.29.json`.

---

## INV-000: Phase 0 baseline — preservation, inventory, and analysis-target selection

- Status: confirmed
- Confidence: observed (except where tagged)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: M-DISC; M-GOG-*; A-GOG-EP1; A-GOG-BASE
- Evidence: `file`/`hdiutil imageinfo` on M-DISC; ISO PVD at offset 0x8000; read-only
  mount + `ls -laR`; Python PE-header parse (sections, data directories, VS_VERSION_INFO,
  RSDS debug records); `shasum -a 256`; `innoextract 1.9 --info/--test/--gog`; manifests.
- Findings:
  - Observed — disc image:
    - UDF 1.5 + ISO 9660 bridge, label `SPORE`; volume space 2,022,736 × 2048 =
      4,142,563,328 bytes = file size (not truncated). Imaged offline by the operator with
      Disk Utility on a Mac with a built-in drive.
    - Hybrid Win/Mac: InstallShield 14 (`setup.ini` EngineVersion 14.0.0.162); Mac side is
      `installer/INSTALLDATA.tgz` (894 entries, owner `ciderbuild`, Wine-style dylibs,
      `transgaming/c_drive`), i.e. a TransGaming Cider wrapper.
    - `Spore/Data`: Audio1/Audio2/Content/Game/Graphics `.package`, 19 locales,
      `Config/ConfigManager.txt` (plaintext, author header "Andrew Willmott").
    - `Spore/SporeBin/SporeApp.exe`: PE32 i386, FileVersion 1.1.0.338, timestamp
      2008-08-05 04:33:29 UTC, linker 7.10, base 0x400000, Authenticode-signed.
      Sections `.text .rdata .data .tls CONST .rsrc Stext Sitext Srdata Sdata Sidata
      .securom`; entry RVA 0x1bdaa10 (in `Sitext`). RSDS records:
      `C:\source\securom_builds\SecuROM-DRM_7_37\SecuROM_DRM\Wrapper\wrapper.pdb` and
      `c:\BuildAgent\cm-spore012-spore\CMBuild\out\Win32\Release\SporeApp.pdb`.
    - `WINUI.dll` 1.0.0.2 PDB `…\GameLauncher\SecuRomPAUL\WINUI\…`; `paul.dll` 4.0.1.8;
      `DMCmdPortalClient.dll` 0.6.7.2; `msvcr71.dll` 7.10.3052.4.
  - Observed — GOG set:
    - Installer `.exe` carries an Authenticode certificate chain naming "GOG sp. z o.o."
      (DigiCert G4 code signing, Sectigo timestamp) — names read, signature not
      cryptographically validated. Inno Setup data 5.6.2; GOG game ID 1948823323;
      buildId 58035967057306769. `innoextract --test`: no checksum errors.
    - Provenance (operator statement): purchased on GOG on another laptop; AirDropped to
      this machine 2026-10-23 (quarantine agent `sharingd`).
    - A-GOG-EP1 and A-GOG-BASE: PE32 i386, linker 9.0, base 0x400000, DllCharacteristics
      0x8140 (DYNAMIC_BASE | NX_COMPAT | TERMINAL_SERVER_AWARE), `.reloc` present,
      signed, **no SecuROM sections or strings**. Timestamps 2024-09-30 23:05:52 (EP1) and
      2024-10-01 00:18:31 (base). Both PDB `c:\BuildAgent\max-spore001-spore\CMBuild\
      SporeEP1_RL\out\Win32\Release\SporeApp.pdb`. Same `.text` size (0xfcaf2a) but
      13.4 M bytes differ. `goggame-1948823323.dll` launch target is the EP1 exe.
      `steam_api.dll` (2015, unsigned) present in both bin dirs.
    - No shared `.package` matches the disc by hash or size (e.g. Spore_Game 18,918,920 →
      20,210,108 bytes). GOG adds PatchData, Spore_Pack_03, EP1 (5) and BoosterPack_01.
  - Strong inference:
    - `[SI]` The disc `.text` is encrypted/virtualized by the SecuROM wrapper (entry in
      wrapper section; standard 7.x layout); confirm in Phase 1 only if the disc binary
      is ever needed.
    - `[SI]` The two GOG executables come from one source tree built in two configurations
      (same PDB path, near-identical section layout).
  - Unresolved:
    - Whether patch 5.1 removes SecuROM from the retail executable.
    - Whether SecuROM activation for disc installs still works.
    - ModAPI symbol coverage for the 2024 GOG build (Risk 14).
    - The `steam_api.dll` role in the GOG build (stub vs functional).
    - GOG current-build discrepancy (3.1.0.29 installer vs 3.1.0.22 recorded in brief).
- Discriminating test: a second disc read with matching SHA-256 confirms the master;
  re-download from the GOG library with matching installer hashes confirms the GOG set.
- Follow-ups:
  - Make the masters read-only (`chmod 444`, `chflags uchg`); keep an offline copy.
  - §9.6 environment setup (Ghidra 12.1.x, arm64 JDK 21, REA, `rea doctor`).
  - INV-001 (Phase 1): REA/Ghidra import of the D8-selected executable; measure import
    time; ModAPI coverage check.
  - INV-002 (Phase 3 track, parallel): `.package` container format inventory on GOG data.

---

## INV-002: `.package` container format inventory (GOG 3.1.0.29 data)

- Status: confirmed (container layer only; resource payload formats are NOT yet decoded)
- Confidence: observed
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all 82 `*.package` under `spore_work/gog-3.1.0.29/` (13 top-level + 69 locale), manifest
  `ledger/manifest-gog-3.1.0.29.json`. Disc packages not analysed yet.
- Evidence: `tools/dbpf_inspect.py`, `tools/dbpf_verify_all.py`; per-package output in
  `ledger/inv002-gog/` (`summary.json` + one JSON per package); run time 16 s.
- Findings:
  - Observed: every package begins `DBPF`, version 3.0 (82/82).
  - Observed header fields (little-endian u32): `0x24` entry count, `0x2C` index byte size,
    `0x3C` index version (3), `0x40` index offset. Other header bytes (0x28, 0x30–0x38, 0x44–0x5F)
    were zero or unexplained and are not interpreted.
  - Observed index layout: u32 flags (values seen: 4, 6, 7), then one shared u32 per set low bit
    (bit0 type, bit1 group, bit2 instance_hi), then per entry the non-constant ones of
    type, group, instance_hi, then instance_lo, offset, size|0x80000000, mem_size (u32 each),
    u16 compression, u16 unknown (always 1). Test passed for 82/82: bytes consumed equal the
    header's index size, the index ends exactly at EOF, no entry overlaps or exceeds the file,
    no duplicate (type, group, instance) keys across 158,720 entries.
  - Observed compression: `0x0000` = stored (size == mem_size, checked on every sampled one);
    `0xFFFF` = RefPack/QFS. 5,125 sampled compressed entries decompressed with zero failures and
    every decompressed length equalled the entry's `mem_size`. Header variants seen: `10 FB`
    (bulk) and `50 FB` (80 entries in Spore_Game, all type `ee17c6ad`); the `0x80`/`0x01` flag
    bits of the generic format are implemented but not yet seen.
  - Observed: 3 packages (Spore_Game, Spore_EP1_Data, BoosterPack_01) contain a gap equal to the
    index size, i.e. a probable stale earlier index left after an in-place update (`[SI]`: the
    gap's start looks like an index header; not parsed further). Several locale/Text packages
    have small (6–7 KB) gaps; 4 East-Asian `UI.package` files have 84-byte gaps. Unexplained.
  - Observed content signatures (first bytes after decompression, sampled): type `2f7d0004`
    → PNG (306/306); `ee17c6ad` → ASCII `ANIM` (145/145); `2f4e681b` → `89 52 57 34`
    (‰RW4) (320/320); `02fac0b6` → UTF-8 BOM text; `022d2c83` → ASCII `EAPD`; `617715d9`
    and `024a0e52` mostly text. 90 distinct type IDs in total; the largest by bytes are
    `2f4e681c` (2.8 GB), `01a527db` (2.6 GB), `00e6bce5` (2.1 GB), `2f4e681b` (1.7 GB).
  - Strong inference `[SI]` (community docs, not derivable from these bytes): format is Maxis DBPF
    3.0; `00b1b104` = `.prop` property list, `2f7d0004` = PNG image, RW4 = RenderWare 4 resources.
    Type-ID names are NOT adopted in this repo until a content signature or code reference
    confirms them.
  - Unresolved: meaning of the u16 "unknown" (always 1) and of the 0x80000000 bit on size
    (always set); the remaining header fields; FNV/hash naming of group/instance IDs; whether
    the stale-index gaps carry deleted-resource data; payload formats for all types; whether
    the disc packages share the same layout (expected, untested); whether RefPack decompression
    of the full corpus (only ~3% sampled) is clean.
- Discriminating test: fully decompress every compressed entry (all 158,720) and compare length
  to `mem_size`; parse the disc packages with the same tool; cross-check the 3 stale-gap regions.
- Follow-ups:
  - INV-003: full-corpus decompression + type-signature table (sniff every entry, not a sample).
  - INV-004: disc-vs-GOG resource diff by (type, group, instance) key to see what the patches changed.
  - INV-005: pick one payload type to decode end to end; `.prop` (`00b1b104`, 30,867 entries,
    34 MB) is the smallest and highest-value, and is the creature/part configuration path.

---

## INV-004: disc vs GOG resource diff (retail 2008 disc → GOG 3.1.0.29)

- Status: confirmed (resource level); causes of change not investigated
- Confidence: observed
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: M-DISC (mounted read-only) vs `spore_work/gog-3.1.0.29`; 58 package paths exist on both sides
  (5 in `Data/`, 53 locale); the GOG tree adds 24 packages the disc lacks (PatchData, Spore_Pack_03,
  3 EP1, BoosterPack_01, 18 locale-only).
- Evidence: `tools/dbpf_diff.py`; output `ledger/inv004-disc-vs-gog/` (`SUMMARY.json` + one file per
  package); 10 s run. Resources paired by (type, group, instance_hi, instance_lo); classes:
  identical raw bytes / same content after decompression / changed content / added / removed.
- Findings:
  - Observed totals over the 58 paired packages (86,930 resource keys): identical 61,607;
    changed 24,671; added 644; removed 8; recompressed-only 0.
  - Observed: no resource differs only by recompression — differences are real content changes.
  - Observed by size/type: bulk payloads are largely unchanged. Of type `01a527db` (the 2.6 GB type),
    20,386 identical vs 17 changed; of RW4 `2f4e681b`, 10,378 identical vs 55 changed. The changes
    concentrate in `00b1b104` (9,946 changed + 424 added vs 1,878 identical), `02b9f662`
    (8,374 changed of 8,462), `ee17c6ad` ANIM (2,025 of 2,091), PNG `2f7d0004` (2,376 of 10,004),
    and `02fac0b6` text (1,226 of 3,526).
  - Observed by package: Spore_Content 1,162 changed / 0 added; Spore_Graphics 5,867 changed,
    122 added; Spore_Audio2 8,428 changed of 12,111; Spore_Audio1 38 changed; Spore_Game 7,950 changed
    of 11,529, 440 added.
  - Strong inference `[SI]`: the disc → GOG delta is dominated by configuration-like data (if
    `00b1b104` is `.prop`, per community docs) — i.e. gameplay/config tuning across the 1.1 →
    1.5.1 → 3.1 patch history, while the art/audio/model bulk is the original content.
  - Caveat — observed: GOG ships `Data/PatchData.package` (25,535 resources) that overlays the base
    packages; this diff compares base packages only, so "changed" does not mean "what the game
    loads". Effective-content comparison needs the engine's package-precedence rules `[NE]`.
  - **Correction by INV-005 (observed):** the `.prop` "changed" count is mostly noise. 9,160 of 10,008
    changed props in the five base packages differ only in 4 uninitialised-looking padding bytes
    (`4c039d00` on disc vs `e40ca000` on GOG); only 717 have real value changes. The `[SI]` above
    about "configuration tuning" is NOT supported by the `.prop` data as first stated. Non-`.prop`
    types were not re-checked for the same kind of noise.
  - Observation (INV-011): the per-package "changed" counts line up with overlaps against `PatchData.package`
    (e.g. Spore_Content: 1,162 changed here vs 1,162 keys shared with PatchData; Spore_Audio2: 8,428 changed vs
    8,386 shared). This is a lead, not an explanation: it was not checked whether the changed base resources
    equal the PatchData copies.
  - Unresolved: why Spore_Audio2 changed so heavily while Audio1 barely did; what the 8 removed
    keys are; per-key meaning of changes (content-level diff for a few `.prop` resources).
- Discriminating test: load order/precedence from the executable (INV-001 follow-up) and re-run
  the diff on the effective merged view; decode one changed `.prop` pair and read the actual delta.
- Follow-ups: INV-005 (.prop decode) should choose resources that appear in both lineages and
  differ, so the format and the delta can be learned together.

---

## INV-003: full-corpus decompression and signature table (GOG 3.1.0.29)

- Status: confirmed — upgrades INV-002's sampled decompression result to the whole corpus
- Confidence: observed
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all 82 `*.package` under `spore_work/gog-3.1.0.29/` (same files as INV-002)
- Evidence: `tools/dbpf_full.py` (8 parallel jobs, 71 s); `ledger/inv003-gog/` (one JSON per package +
  `TOTAL.json`). The decoder was rewritten for speed (slice copies) before this run and re-checked
  against all 9,314 compressed entries of Spore_Game (all matched).
- Findings:
  - Observed: 158,720 resources, 101,312 RefPack-compressed, **0 failures** — every decompressed
    length equals the index `mem_size`, every stored entry has size == mem_size. 10.45 GB
    decompressed in total; 90 distinct type IDs.
  - Observed first-4-byte signatures of full type populations (type → count, GB, signature):
    `2f4e681c` 5,931 / 2.81 GB / `01000000` (100%, 1 distinct); `01a527db` 33,781 / 2.57 GB /
    12 distinct, 73% `05005622`; `00e6bce5` 11,041 / 2.14 GB / `09000000` 62% + `08000000`;
    `2f4e681b` 13,498 / 1.67 GB / `89 52 57 34` (RW4) 100%; `ee17c6ad` 5,022 / 0.30 GB / `ANIM` 100%;
    `2f7d0004` 14,694 / 0.26 GB / PNG 100%; `00b1b104` 30,889 / 0.03 GB / 28 distinct signatures
    (small records, many layouts); `376840d7` 4 resources / 0.04 GB / `MVhd`.
  - Observed: the three types that hold 7.5 of the 10.45 GB (`2f4e681c`, `01a527db`, `00e6bce5`) have a
    uniform leading u32 (version/count-like) but no ASCII magic; their meaning is NOT established
    here.
  - Unresolved: what `2f4e681c`, `01a527db`, `00e6bce5` contain; `MVhd` (QuickTime-style movie header
    is a guess only — not adopted).
- Discriminating test: for a named type, decode one payload from the executable's own loader (INV-001
  follow-up) and check against the signature.
- Follow-ups: INV-005 `.prop` decode; use the signature table to pick the next payload type.

---

## INV-001: REA/Ghidra import of the GOG EP1 executable (Phase 1 gate, first half)

- Status: partially confirmed — toolchain gate passed on the large PE; ModAPI coverage check and
  function-level work not started
- Confidence: observed (REA tags the evidence `derived`)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: A-GOG-EP1 `SporebinEP1/SporeApp.exe`, SHA-256
  `dc04aee5a3debc3f1ad4c1a937460e99a29b9bd3bc285008be83615dd5e59a37`
- Evidence: `spore_work/inv001/inspect-ep1.json` (REA `inspect --provider ghidra`, evidence id
  `ev_…` inside), `inspect-ep1.time`; Ghidra 12.1.4, JDK 21.0.13 arm64, native decompiler built for
  arm64; env in `env/README.md`, `GHIDRA_HEADLESS_MAXMEM=12G`.
- Findings:
  - Observed: first run (11:16) was cancelled when the earlier Claude Code process exited
    (`inspect-ep1.cancelled-run1.json`, `code: cancelled`, cleanup complete); second run finished.
  - Observed: import + auto-analysis + inventory in **481 s wall-clock** (574 s user), peak RSS
    **7.2 GB** (`time -l`, includes child processes), exit 0. Measures Risk 4: each fresh session
    pays about 8 minutes; the REA MCP session keeps an opened target across turns.
  - Observed: image base `0x400000`, 9 segments (`Headers`, `.text` 0x401000–0x13cc000 [16.5 MB],
    `.rdata`, `.data`, `.tls`, `CONST`, …), **58,107 procedures** discovered, **16,818 strings**.
  - Observed: Ghidra decoded the x86-32 PE without errors; no SecuROM sections (INV-000).
  - Unresolved: no function has been decompiled yet on this target; procedure count is Ghidra's
    discovery, not a ground-truth function count `[NE]`; ModAPI symbol coverage for this build (Risk 14).
- Discriminating test: decompile 3 functions of known role (e.g. one found via a known string) and
  check pseudocode against string/xref evidence; import the 1.3.0.29 executable and compare coverage (D8).
- Follow-ups: INV-006 (Phase 1): string/xref sweep for package loading (`.package`, `DBPF`, `RefPack`
  style routines) to tie INV-002/003 to code; ModAPI coverage check; decide D8.

---

## INV-005: `.prop` (type `00b1b104`) structure decode and disc→GOG delta

- Status: confirmed — 30,889/30,889 resources parse exactly (data-derived rules, then code-confirmed in INV-006)
- Confidence: observed (structure derived from bytes; no property names adopted)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all 30,889 `00b1b104` resources in the GOG tree; disc comparison on the five base packages
- Evidence: `tools/prop_parse.py`, `tools/prop_corpus.py`, `tools/prop_delta.py`; 24,700-file sample
  used to derive sizes, then full-corpus test.
- Findings:
  - Observed layout (big-endian): u32 entry count; per entry u32 id, u16 type, u16 flags, value.
    Test: **30,730 of 30,889 resources parse and consume exactly their bytes** (count entries, no
    trailing bytes, no overruns).
  - Observed value rules: flags with bit `0x10` set (seen `0x1c`, `0x9c`) → array: u32 count, u32
    element size, count×size bytes. Scalars (flags `0x0000`): type `0x01` 1 byte; `0x09`, `0x0a`, `0x0d`
    4 bytes; `0x20` and `0x30`–`0x34` 16 bytes. Strings types `0x12`/`0x13`: u32 char count + chars
    (UTF-16LE seen for `0x13`, flags `0x000d`). Type numbers' meanings (int/float/key/vector…) are
    NOT asserted; only sizes are.
  - Observed: 16-byte `0x20` values are three u32 (instance, type `04b1b100` = `00b1b104` byte-reversed,
    group — the little-endian IDs match the owning resource's own group) plus 4 trailing bytes. The
    trailing 4 bytes are a build-specific constant: `4c039d00` (disc, 36,080×), `e40ca000` (GOG,
    36,144×), also two stack-address-like values (`40c41200` vs `3cc41200`).
    `[SI]` uninitialised serializer padding. Type `0x34` ends in `0000803f` (1.0f) so it carries
    real data in all 16 bytes.
  - Observed delta: of 11,807 resources present in both lineages' five base packages: 1,861
    byte-identical; 9,160 differ only in the 4 padding bytes; **717 real changes**; 69 unparsed.
    424 props only in GOG, 2 only on the disc.
  - Update (INV-006): the executable's own reader gives the remaining rules (type sizes 1/2/4/8 bytes
    for types 1–0xE, raw 16-byte default branch, string arrays read element by element). With them
    **30,889 of 30,889 resources consume exactly** (the earlier 159 failures were string arrays and
    rare small types). The "99.5%" figure above is superseded.
  - Unresolved: numeric meaning of each type code (sizes known, semantics not);
    property-ID → name mapping (IDs look like 32-bit hashes; not tested); the 717 real changes not
    yet read at the value level.
- Discriminating test: check ID hashing against known property names; read the 717 real changes and
  tie one to a code path in the executable (INV-006).
- Follow-ups: INV-006 (code side of the loader, links INV-002/005 to INV-001); decode the 159 failures.

---

## INV-006: package/property loader in the GOG EP1 executable; ModAPI coverage

- Status: partially confirmed — two loader functions matched to the data formats; D8 pending base-exe comparison
- Confidence: observed (REA/Ghidra decompilation of the shipped binary; pseudocode, not source)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: A-GOG-EP1 (`dc04aee5…d914e`); VAs at image base `0x400000`
- Evidence: REA MCP `open_binary`/`search_strings`/`xrefs`/`procedure_pseudo_code`, evidence ids
  `ev_94d0…` (strings), `ev_fcdc…` (0x8dbcc0), `ev_239e…` (0x6a2f60), `ev_7d08…` (0x694410),
  `ev_fc61…` (0x93a6f0), `ev_f7cf…` (0x93a630), `ev_8207…` (0x93a770), `ev_b2f9…` (0x93a670);
  `spore_work/inv006/procedures-ep1.json` (57,626 internal + 481 external procedures);
  ModAPI reference clone `spore_work/ref/Spore-ModAPI` (HEAD `69b27b5`, 2026-10-16).
- Findings:
  - Observed strings: `*.package` (0x1403028, referenced from 0x68731f, 0x697037, 0x6972a0),
    `Preload.package`, `GraphicsCache.package`, `Pollination.package`, `ServerCache.package`,
    `Editors_BlockData.package`, `TempDBPF_` (0x14369e8, referenced from 0x8da524 in function
    0x8da460, 599 bytes).
  - Observed — package index reader `0x8dbcc0` (FUN_008dbcc0; ModAPI march2017 names this address
    `Resource::PFIndexModifiable::Read`): requires flags bit 2; bit 0 → shared type, bit 1 → shared
    group; requires the shared `instance_hi` word to be 0; per entry reads four u32 (instance_lo,
    offset, size, mem_size) after the non-shared fields; if size bit 31 is set it clears it and reads one
    extra u32 (compression word), otherwise sets compression to 0 when size == mem_size, else `0xFFFF`.
    This **confirms INV-002's index layout from code** (flags 4/6/7, constant instance_hi = 0, size
    bit 31 always set in the data) and adds a compact entry form the shipped data never uses.
  - Observed — property reader chain: `0x6a2f60` (ModAPI: `App::PropertyList::Read`) reads a count
    (top bit = a parent key follows, three u32), sizes a 24-byte-per-entry array, then per entry an id
    and a value via `0x694410`. `0x694410` reads type (u16) then flags (u16); `flags & 0x30` selects
    the array path (count then element size, then elements; strings per element), else a type switch.
    `0x93a6f0` reads N×u32 big-endian (byte swap); `0x93a670` N×u16 big-endian; `0x93a770` N×8
    bytes; `0x93a630` N raw bytes. Types 2/5/6 → 1 byte, 3/7/8 → 2, 9/0xA/0xD → 4, 0xB/0xC/0xE → 8,
    0x11 → none, 0x12/0x13 → strings, any other type → raw 16-byte read. This confirms INV-005 from
    code. The parent key is never encoded as a count top bit in the data; it appears as an ordinary
    key property (`00b2cccb`) `[SI]`.
  - Observed — the raw 16-byte read explains INV-005's padding noise: the writer's in-memory 16-byte
    struct (12 data bytes + 4 padding) is stored verbatim `[SI]` (the writer was not examined).
  - Observed — ModAPI coverage: **1,203 of 1,670 (72.0%)** ModAPI march2017 function addresses are Ghidra
    function entries in GOG EP1; only **145 of 1,670 (8.7%)** of the `disk` addresses are. A random
    16-aligned `.text` address is an entry 5.4% of the time. Two matches were checked by behaviour
    (0x8dbcc0, 0x6a2f60): both fit their ModAPI names. Entry-point coincidence is not proof of
    identity for the other ~1,200.
  - Strong inference `[SI]`: the 2024 GOG build is a recompile close to the 2017 Steam/Origin layout
    (march2017), not the 2008 disc layout; march2017 symbol data is a usable seed for naming.
  - Update (INV-011): the 72.0% figure is a LOWER BOUND. Of the 467 misses, 2 of 2 checked
    (`RegisterDatabase` 0x8e1000, `FindDatabase` 0x8de620) are real functions at the ModAPI address with the
    right behaviour; Ghidra just never defined them (vtable-only targets). Most misses sit in gaps with no defined
    function. Real coverage is unknown but higher.
  - Unresolved: how many of the 1,203 are the _same_ function (needs behavioural sampling); the 467
    misses (moved or removed?); package precedence/load order; where `PatchData.package` is loaded
    relative to base packages; the reader of `ee17c6ad`/RW4/`01a527db`/`00e6bce5`.
- Discriminating test: decompile 20 randomly chosen ModAPI-named functions with distinctive strings or
  constants and check each against its ModAPI name; compare coverage on the 1.3.0.29 base executable
  (D8, in progress).
- Follow-ups: INV-007 behavioural sampling of the 1,203 matches; INV-008 find the package mount/
  precedence code (callers of 0x8dbcc0's owner, `*.package` xrefs 0x68731f/0x697037/0x6972a0);
  seed REA names from ModAPI for the matched set.

---

## INV-007: behavioural check of ModAPI-named functions in GOG EP1 (sample of 21)

- Status: confirmed for the sample — consistent in all 21 cases checked, none contradicted
- Confidence: observed (decompiled code / raw bytes of the shipped executable)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: A-GOG-EP1; ModAPI march2017 names (INV-006 coverage set)
- Evidence: decompilations `ev_b560…` (0x407190), `ev_fbc7…` (0x6a3770), `ev_b737…` (0x6a3860),
  `ev_3f47…` (0x6a1110), `ev_52b6…` (0x8d8640); byte reads `ev_7431…` (0x6a36e0, 480 B),
  `ev_d2f0…` (0x6a0f70, 340 B). Getter bodies decoded offline from the `0f b7 41 12 66 83 f8 NN`
  pattern (`movzx eax,[ecx+0x12]; cmp ax,NN`), one function per 0x30 bytes at the ModAPI addresses.
- Findings (each getter's name vs the one type code it tests):
  - Observed, consistent with name: `GetBool`→1; `GetValueChar`→2; `GetValueWChar`→3; `GetValueInt8`→5;
    `GetValueUInt8`→6; `GetValueInt16`→7; `GetValueUInt16`→8; `GetValueInt64`→0xB; `GetValueUInt64`→0xC;
    `GetValueDouble`→0xE; `GetValueKey`→0x20; `GetValueFlags`→0x21; `GetValueText`→0x22;
    `GetValueVector2`→0x30; `GetVector3`→0x31 (copies exactly 12 bytes); `GetValueColorRGB`→0x32;
    `GetValueVector4`→0x33; `GetValueColorRGBA`→0x34; `GetValueTransform`→0x38.
  - Observed, independent check: for the nine getters whose type has a size in INV-006's reader table
    (Char/Int8/UInt8 = 1 byte, WChar/Int16/UInt16 = 2, Int64/UInt64/Double = 8), the size implied by the name
    equals the size derived from the reader helpers. 9/9.
  - Observed: `GetVector3` copies 12 bytes of a 16-byte stored value — direct evidence that the last 4
    bytes of `0x30`–`0x34` values are not data for Vector3 (INV-005 padding finding).
  - Inconclusive: `DatabasePackedFile::ReadHeaders` (0x8d8640) is a thin virtual-call wrapper; consistent with
    the name but not distinguishing.
  - Unresolved: Int32/UInt32/Float codes (9/0xA/0xD) were not read (assumed from sizes `[SI]`);
    BBox likely 0x39 (24-byte elements, 6 floats) `[SI]`, not read; type 4 and 0x10 (variant) meanings.
  - Limits: 21 of 1,203 address matches checked, chosen for checkability, not at random. This supports
    the march2017 seed for the Property API cluster; other clusters are untested.
- Discriminating test: random sample across namespaces (Simulator, UTFWin, IO) with a distinctive
  constant or string per function.
- Follow-ups: INV-009 sample the Simulator cluster; adopt Property type names into `prop_parse` output.

---

## INV-008: package mounting code (what loads `*.package` and in what way)

- Status: partially confirmed — mounting is directory-driven; precedence rule NOT yet established
- Confidence: observed for control flow; priority interpretation is an inference
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: A-GOG-EP1 functions `0x6872b0` (670 B), `0x696c80` (2,485 B), `0x6866b0`, plus `0x8da460`
- Evidence: `ev_9c21…` (0x6872b0), `ev_4746…` (0x696c80), `ev_9fc2…` (0x6866b0).
- Findings:
  - Observed: `0x6872b0(owner, dir, filter, ctx)` enumerates `L"*.package"` in a directory with a recursive/
    flag argument (`0x100000`), optionally filters each path through a callback, allocates an
    `App/DatabasePackedFile` (0x388 bytes, constructor 0x8da2a0), sets its path (vtable +0x2c), opens it
    (vtable +0x18, args `1,6,0`), and on success passes it to `0x6866b0`.
  - Observed: `0x696c80(list, flag)` is the discovery driver: resolves directory paths by hashed ID
    (`0x45962cc`, `0x6cbed3a` via `0x6887d0`), runs the same enumerate/create/open sequence for each, and
    appends opened databases to one of two owner-side linked lists (offsets `+0x50` / `+0x60`, chosen by `flag`);
    when a lookup fails it calls itself with `flag = 1`. Hashing uses FNV-1 32-bit (offset basis
    `0x811C9DC5` seen in `0x932f00` calls).
  - Observed: there is no fixed list of package names in this code path — any `*.package` in the
    configured directories is mounted; order follows the enumerator's output order `[NE: whether it sorts]`.
  - Observed: `0x6866b0` validates a package by reading a `.prop` resource (type `0x00b1b104`, key built
    from hashed names), tests properties `0x6ef59e5`–`0x6ef59eb`, hashes the name "Bless", compares a
    computed value against a Windows registry value `BlessID` (HKLM) — an authorisation/"bless" check for
    some packages — and at the end calls the resource manager (vtable +0x50) with the database and a
    value that comes from property `0x6ef59e5` when it is an int32 (type 9).
  - Strong inference `[SI]`: `0x6ef59e5` carries a per-package priority/order consumed by the manager;
    decompiler output left that argument as an unnamed register, so this is not confirmed.
  - Unresolved: the manager's lookup order (which database answers a key when several define it); whether
    `PatchData.package` wins by name, by priority or by enumeration order; what "Bless" gates; sorting of
    enumeration results; which directories `0x45962cc` / `0x6cbed3a` name.
- Discriminating test: read the manager's vtable +0x50 target and its key lookup (find callers of
  `0x6866b0`'s manager), or read the `0x6ef59e5` value in each shipped package's own prop.
- Follow-ups: INV-010 inspect packages' own prop resource for `0x6ef59e5` (data side, no Ghidra);
  INV-011 manager lookup order (code side).

---

## INV-010: per-package info resource and the `0x06ef59e5` property (data side)

- Status: confirmed (data); meaning of the values inferred from INV-008 code
- Confidence: observed (values); strong inference (priority meaning)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all `00b1b104` resources in the GOG tree, scanning for property ids `0x06ef59e5`–`0x06ef59ef`
  (those ids are the ones `0x6866b0` reads, INV-008)
- Evidence: scan with `tools/prop_parse.py` over all 30,889 props (this session; results below).
- Findings:
  - Observed: only 5 of 82 packages hold these ids, each in a resource of group `0x40404000`:
    `Data/PatchData.package` (instance `00000001`): `0x06ef59e5` = int32 **2**; `0x06ef59e6` = uint32 array `[06f4a0e6]`.
    `Data/Spore_Game.package`: `0x06ef59e6` = `[06f4a0e6]` only; no `0x06ef59e5`.
    `Data/Spore_Pack_03.package` (instance `e738351a`): `0x06ef59e6` = `[06f4a0e6]` only.
    `DataEP1/Spore_EP1_Data.package` (instance `2c24ab68`): `0x06ef59e5` = int32 **101**; `0x06ef59e6` =
    `[06f4a0ef]`; `0x06ef59e7` = key (raw16).
    `bp1content/BoosterPack_01.package` (instance `c8dc9291`): `0x06ef59e5` = int32 **1**; `0x06ef59e6` =
    `[06f4a0e6]`; `0x06ef59e7` = key (raw16).
  - Observed: the property the code reads as an int32 (type 9) and passes to the resource manager is
    present with values 1, 2 and 101 on three packages and absent on the other two with an info resource.
  - Strong inference `[SI]`: `0x06ef59e5` is a load priority — the ordering PatchData (2) > BoosterPack_01 (1) and
    EP1 data (101) highest matches "patch/expansion content overrides base"; packages without it use a
    default (value unknown). Not confirmed against the manager's lookup (INV-011).
  - Unresolved: default value when absent; what `0x06ef59e6` (a single hashed id per package;
    `06f4a0e6` vs `06f4a0ef`) and `0x06ef59e7` mean; why the other 77 packages (Content, Graphics, Audio,
    locale, EP1 content) carry no info resource; whether the lookup treats larger values as winning.
- Discriminating test: find the manager lookup that iterates databases (INV-011) and check ordering
  against these values; or construct a test where two packages define the same key.
- Follow-ups: INV-011 (code side), using the vtable +0x50 registration path from INV-008.

---

## INV-011: resource manager — database registration and lookup order

- Status: confirmed for the ordering rule; default priority of un-annotated packages NOT established
- Confidence: observed (code + data); defaults unresolved
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: A-GOG-EP1 `0x8e1000` (RegisterDatabase), `0x8dec10` (sorted insert), `0x8de620` (FindDatabase)
- Evidence: `llvm-objdump -d --start-address=0x8e1000` of the on-disk executable; byte reads `ev_d9b4…`, `ev_a2fe…`;
  decompilation `ev_46f0…` (0x8dec10); ModAPI headers `IResourceManager.h` (RegisterDatabase at vtable +0x50,
  `priority = 1000` as the _header's_ default argument, FindDatabase at +0x58, "descending order of priority").
- Findings:
  - Observed: `0x8e1000(add, db, priority)` — when `add`, if `IsDatabaseRegistered(db)` (vtable +0x54) is false it
    calls `0x8dec10(db, priority)` on the container at `manager+0x2c`, then `db->vtable[+0x44](1, manager, 1)`;
    when removing, it walks the same list for `db`. Matches the ModAPI signature.
  - Observed: `0x8dec10` builds a node (next, db, priority) and inserts it **before the first node whose priority is
    strictly less than the new one** (it advances while `node.priority >= new`). The list is therefore sorted by
    descending priority; for equal priorities the earlier-registered database stays first.
  - Observed: `0x8de620` (FindDatabase) walks the list from the head and returns the first database whose
    vtable +0x34 reports it contains the key. **Higher priority wins; ties go to the earlier-registered.**
  - Observed (with INV-010): `0x06ef59e5` values 101 (EP1 data), 2 (PatchData), 1 (BoosterPack_01) are passed
    as that priority (INV-008 call `RegisterDatabase(1, db, value)`), so for duplicate keys EP1 data beats
    PatchData beats BoosterPack_01. In `0x696c80` the second-pass directory registers with priority
    `0xfffffc18` = **-1000**; the first pass takes its default from a field at `+0xc` (value not read).
  - Observed — overlap in the shipped GOG data (all 82 packages): 158,720 entries → **123,796 unique keys**;
    **29,797 keys occur in more than one package; 34,924 copies (22%) are shadowed**; no key repeats inside a
    single package. Largest conflicts: PatchData×Spore_Game 10,869; PatchData×Spore_Audio2 8,386;
    EP1_Data×BoosterPack_01 4,474; PatchData×Spore_Graphics 3,566; PatchData×Spore_Content 1,162;
    PatchData×Spore_Game×EP1_Data 647; 194 keys across 23 packages (locale-style).
  - Strong inference `[SI]`: base packages without a priority property sit below PatchData (the shipped game applies
    patches over base), i.e. default < 2; the actual default and its source were not read.
  - Unresolved: first-pass default priority (field `+0xc`); whether the per-package priority is read before or after
    `RegisterDatabase` for packages without the property; locale packages' priorities; behaviour of
    `GetDatabaseList`/enumeration (not the lookup path); disc build ordering (not analysed).
- Discriminating test: read the field at `+0xc`'s initialiser; or build two throwaway packages with one shared key
  and priorities 1 and 2 and ask the game which value it returns.
- Follow-ups: INV-012 effective merged view — resolve every key by this rule and rebuild the INV-004 disc↔GOG
  comparison on the merged view; use it for fixtures.

---

## INV-006 addendum — D8: base-game executable vs EP1 (ModAPI coverage)

- Status: confirmed (address level)
- Confidence: observed
- Target: A-GOG-BASE `SporeBin/SporeApp.exe` (`25d42a7a…914e`) vs A-GOG-EP1; `spore_work/inv006/procedures-base.json`
- Findings:
  - Observed: base executable has 57,920 internal + 482 external procedures (EP1: 57,626 + 481). Only
    17,642 function-entry addresses are common to both builds.
  - Observed: ModAPI march2017 addresses that are function entries: **base 325/1,670 (19.5%)** vs
    **EP1 1,203/1,670 (72.0%)**; `disk` addresses: base 172 (10.3%), EP1 145 (8.7%).
  - Observed: of five key addresses (0x8dbcc0, 0x6a2f60, 0x6a3770, 0x6a1110, 0x407190) four are function
    entries in the base build; 0x8dbcc0 (package index reader) and 0x8dec10 (sorted insert) are not.
  - Decision: D8 resolved — EP1 is primary. Limits: coverage counts Ghidra-defined entries only (a lower bound,
    INV-011), and the base build was not decompiled.

---

## INV-012: effective merged resource view and disc→GOG comparison on it

- Status: confirmed under stated assumptions (below)
- Confidence: observed (hashes, overlaps); assumptions flagged
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: GOG tree (16 packages in the en-us view) vs disc tree (all mountable packages, en-us locale)
- Evidence: `tools/dbpf_merged.py`, `tools/dbpf_hashes.py`; SHA-256 of decompressed content for 153,968 (GOG) and
  82,758 (disc) resources (`spore_work/inv012/*.tsv`, local only); `ledger/inv012/merged-summary.json`.
- Assumptions: (A1) priority rule from INV-011; GOG priorities EP1_Data 101, PatchData 2, BoosterPack_01 1;
  (A2) packages without a priority property take default 0; (A3) only the `en-us` locale folder is mounted;
  (A4) ties go to the alphabetically earlier path; (A5) disc packages all default priority.
- Findings:
  - Observed — sensitivity to A2: the merged view is **identical for every default priority ≤ 0**; defaults 1 or 2
    change one key; a default ≥ 3 would make base packages beat PatchData for ~24,100 keys (contradicts patches taking
    effect). Ties under A2: 19 keys (18 Spore_Content×EP1_Content_01, 1 Audio1×Audio2).
  - Observed — GOG effective view: 123,568 unique keys, 29,556 of them defined in more than one package. Winners by
    package: PatchData 24,725; EP1_Data 22,204; Spore_Graphics 20,977; Spore_Audio1 17,540; Spore_Content 15,957;
    EP1_Content_02 10,801; EP1_Content_01 5,789; Spore_Audio2 3,637.
  - Observed — shadowed copies (en-us view): 30,400, of which **8,473 (28%) are byte-identical to the winner**
    (redundant) and 21,927 differ (real overrides). Largest: PatchData over Audio2 8,370 differ; over Game 7,947
    differ + 2,922 same; over Graphics 3,371 differ; over Content 1,162 differ; EP1_Data over BoosterPack_01 4,283
    same + 193 differ.
  - Observed — effective disc → GOG: identical 58,918; changed 23,831; only in GOG 40,819; only on disc 8.
    Only-in-GOG winners: EP1_Data 21,310, EP1_Content_02 10,801, EP1_Content_01 5,789, BoosterPack_01 1,500,
    Pack_03 210, PatchData 1,203. Changed winners: PatchData 20,874, Spore_Graphics 2,377, EP1_Data 546, Text 34.
    Spore_Content, Spore_Audio1 and Spore_Audio2 have **no** changed effective winners.
  - Observed — by type, changed in the effective view (raw hash difference, before padding correction): `00b1b104` 10,070; `02b9f662` 8,379; PNG `2f7d0004` 2,384;
    ANIM `ee17c6ad` 2,085; RW4 `2f4e681b` 77; `01a527db` 71.
  - Observed — three-way check (disc / GOG base / PatchData) for keys PatchData shadows: for ~20,600 keys
    (Audio2 8,414; Game 7,654; Graphics 3,344; Content 1,162) all three versions differ; ~2,974 Game keys are
    identical in all three; some keys have patch == base ≠ disc (296 Game, 144 Graphics). I.e. GOG's base
    packages are not the disc data; they were rewritten for exactly the keys PatchData overrides. This
    resolves INV-004's lead: the base-package diff there measured those rewritten copies, which the game never
    loads when PatchData is mounted.
  - **CORRECTED (INV-013): the claim that all 10,070 effective changed props are real changes was a bug.** The
    padding normaliser stopped matching after INV-006 renamed 16-byte entries from kind `scalar` to `raw16`, so it
    did nothing. With the fix (and a self-test), of the 10,070: 7,994 are padding-only and 2,076 are real changes.
    The statement that padding differences occur only in shadowed base copies is also withdrawn.
  - Unresolved: whether `02b9f662`, ANIM and PNG "changes" are similarly noisy (not normalised); the 3-way
    pattern's cause (why base copies differ from disc); real enumeration order for ties; non-en-us locales;
    the 8 disc-only keys; whether the disc's own packages carried priority info resources (not scanned).
- Discriminating test: read the field at `+0xc` for the first-pass default; decode `02b9f662` and ANIM to see
  what the changes are; test padding hypothesis on those types.
- Follow-ups: INV-013 decode `02b9f662` (8,379 changed, only 2 MB total) as a next payload type; build fixtures
  from the effective view (pick unchanged and changed prop pairs); creature-focused resource search.

---

## INV-013: noise vs real change in the effective disc→GOG delta; prop-format types; creature animation

- Status: confirmed for the sampled/complete sets stated
- Confidence: observed
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: effective-view keys whose content hash differs between disc and GOG (INV-012), types `00b1b104`, `02b9f662`,
  `03f51892`, `02c9eff2`, `2f7d0004`, `ee17c6ad`
- Evidence: `tools/effective.py`, `tools/prop_delta.py` (fixed, self-tested), `tools/png_pixels.py` (validated against
  macOS `sips` on 12 pairs: both say "differ" for all 12 — agreement on the verdict only), disc mounted read-only.
- Findings:
  - Observed: three more types use the prop layout (25/25, 25/25, 18/18 samples parse exactly): `02b9f662`
    (11,005 effective keys), `03f51892` (50), `02c9eff2` (18). `02b9f662` is the second most common prop-format type.
  - Observed — padding vs real change among the effective changed resources (all of them, not a sample):
    `00b1b104` 10,070 changed → **7,994 padding-only, 2,076 real**; `02b9f662` 8,379 → **8,270 padding-only,
    109 real**; `03f51892` 44 → 41 / 3; `02c9eff2` 18 → 18 / 0. Total real prop-format changes: 2,188 of 18,511 raw.
    Padding bytes: three differing bytes of a 4-byte tail (`40c412` on the disc vs `010000` in GOG for `02b9f662`).
  - Observed — PNG (`2f7d0004`, 2,384 changed): chunk-type lists match in 194/200 sampled (6 differ in IDAT count only);
    scanlines are re-encoded in all 200. Decoded pixels differ in 2,372 of 2,384, dimensions/format differ in 12. On a
    297-pair sample with equal dimensions: median 2.2% of bytes differ (p90 13%), median mean-absolute-difference
    0.03/255 (p90 0.63) — small edits, plausibly re-encoding and alpha/colour clean-up; 3 sampled pairs changed size
    (e.g. 18×18 → 24×24, 32×64 → 64×64). The cause of the small differences was not determined.
  - Observed — ANIM (`ee17c6ad`, 2,085 changed): every sampled pair differs in size; the GOG copy is smaller in 2,084
    of 2,085 (total 180.2 MB disc → 85.5 MB GOG). Each resource starts with ASCII `ANIM`, a u32 equal to the
    resource length, a u32 (disc 0x15, GOG 0x19 in the example), then an embedded source path such as
    `C:\Spore_P4_Cronus\Data\Creatures\Animations\_final\creature\tree_idle.animation` (a "Spore_P4_Cronus" build
    tree). 1,877 disc and 1,628 GOG ANIM resources contain a `C:\` path. These are creature animation clips — the
    resource class behind the motion system in the brief (§3.2) `[SI]` (name and path only; structure not decoded).
  - Strong inference `[SI]`: the PNG and padding differences are build/export noise, the ANIM size drop is a content
    re-export or stripping; neither is confirmed.
  - Process note: the INV-012 error above came from renaming an entry kind without re-running dependent analyses.
    `prop_delta.py` now self-tests the normaliser; any future change to `prop_parse` kinds must re-run INV-005/012/013.
  - Unresolved: ANIM structure and what the extra data in disc ANIM files is; whether GOG ANIM data is a lossy
    reduction (affects fidelity fixtures); names of properties (IDs are hashes); meaning of the 2,188 real prop changes.
- Discriminating test: decode the ANIM header fields and track counts on a pair; compute FNV-1 hashes of candidate
  property names (INV-014) to label the 2,188 real changes.
- Follow-ups: INV-014 property-ID naming via ModAPI strings + FNV test; INV-015 locate creature part resources
  (RW4 models + prop part definitions) for the Phase 4 slice.

---

## INV-014: naming property and group IDs (hash test); audio property lists

- Status: confirmed for the hash algorithm and 499 property / 33 group names; most IDs remain unnamed
- Confidence: observed (exact hash matches); semantics of types `[SI]`
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all prop-format resources (types `00b1b104`, `02b9f662`, `03f51892`, `02c9eff2`) in the GOG tree
- Evidence: `tools/prop_names.py` (FNV-1, 32-bit, lowercased input, basis `0x811C9DC5`, prime `0x1000193`);
  candidate tokens from the executable's bytes (ASCII + UTF-16), ModAPI sources (`id("…")` literals and identifiers),
  string values inside the props; names JSON `spore_work/inv014-names.json` (local).
- Findings:
  - Observed: the game's ID hash is **FNV-1 32-bit over the lowercased string** (ModAPI `Hash.h` agrees; the executable
    uses basis `0x811C9DC5`, INV-008). A name is accepted only when its hash equals an ID occurring in the data.
  - Observed — properties: 5,882 distinct property IDs in 369,452 entries; **499 IDs named (8.5% of distinct, 17.7%
    of occurrences)** — 424 from the executable, 72 from ModAPI, 3 from prop strings. Expected false hits with this
    candidate set are well below 1. Examples: `5f6317d5` = `parent` (the key-typed parent link, 19,612×),
    `701ed91e` = `Samples` (19,037×), `25df0108` = `gain`, `6dd08218` = `islooped`, `12d2a4d4` = `is3d`.
  - Observed: the most frequent IDs are unnamed (`00b2cccb` 23,390×, `00f9efbb` 11,361×, `00b2ccca` 7,106×); several come
    in consecutive runs (`00f9efb9`…`00f9efbe`), so some property IDs are hand-assigned constants, not name hashes.
  - Observed — groups: 605 distinct group IDs; 33 named (6,524 of 153,968 resources) incl. `CreatureAbilities`
    (314 props), `AbilityTemplates` (89), `CreatureGame` (22), `TribeAbilities` (48), `ConsequenceTraits` (54),
    `PaletteItems` (940), `PlannerThumbnails` (965 PNG), `AssetBrowserGraphics` (899 PNG), `Havok` (18, type `448ae7e2`).
    Largest groups are unnamed: `021407ee` (54,179: 33,781 × `01a527db` + 19,630 × `02b9f662`), `00000000` (13,214:
    5,727 RW4 + 5,022 ANIM), and patterned IDs `40606000`/`40626200`/`40636200`/`40646000`… (editor-part-like mixes of
    props, PNG, RW4 and other types) — patterned, not hashed names.
  - Observed — reference test: 3,003 `02b9f662` resources contain a `Samples` key array (element 12 bytes, little-endian
    words: instance, type, group; type and group are zero = wildcard). **40,419 references: 40,330 resolve by instance
    (99.8%); 40,234 (99.5%) resolve to a type-`01a527db` resource in the same group as the referencing resource.**
    So `Samples` links sound descriptions to the 2.6 GB type. `[SI]` `02b9f662` = sound/effect property lists and
    `01a527db` = audio sample data (names from the property names only; sample data not decoded).
  - Unresolved: names for the top unnamed IDs (need an external list or runtime string capture); meaning of groups
    `40xx6x00`; what `00e6bce5` (2.1 GB) and `2f4e681c` (2.8 GB) hold; whether key words are little-endian for scalar keys
    in all contexts (arrays of 12-byte keys: yes here).
- Discriminating test: for a named property cluster (e.g. `CreatureAbilities`), check the values make sense by reading a
  few props; locate the creature part definitions via keys that reference RW4 models (`2f4e681b`).
- Follow-ups: INV-015 creature-part discovery (RW4 models + the prop lists that reference them).

---

## INV-015: type-ID names; creature definition chain (CRT → block → RW4)

- Status: confirmed for the chain on the data; semantics `[SI]`; RW4 internals and `creatureData` not decoded
- Confidence: observed (hash matches, XML text, key resolution)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: type IDs of all 153,968 resources in the GOG tree; types `2b978c46`, `0f43029a`, `25df0112`, `5caa5e28`, `2f4e681b`
- Evidence: FNV-1 lowercase hash of tokens from the executable, ModAPI sources and a short list of file-type words
  vs the 90 type IDs; package scans in `Spore_Game`, `PatchData`, `Spore_Graphics`, `Spore_Content`, `Spore_EP1_Content_01`.
- Findings:
  - Observed — **type IDs are name hashes too: 40 of 90 type IDs equal the FNV-1 lowercase hash of a string** (expected
    coincidences under 1). Matches: `ee17c6ad`=`animation` (5,022), `5caa5e28`=`blockdata` (3,260), `1a99b06b`=`bem` (2,621),
    `0f43029a`=`creatureData` (1,365), `250fe9a2`=`spui` (748), `dfad9f51`=`Cell` (489), `4b9ef6dc`=`Structure` (376),
    `617715d9`=`pD` (318), `497767b9`=`pfx` (126), `d92af091`=`lootTable` (115), `2b978c46`=`CRT` (94),
    `da141c1b`=`populate` (64), `25df0112`=`gait` (39). The biggest types stay unnamed (`01a527db`, `00b1b104`, `2f4e681b`…).
  - Observed — `CRT` (`2b978c46`): 94 resources (47 distinct, present identically in `Spore_Game` and `PatchData`), all in
    group `40626200`, **plain UTF-8 XML**: root `<sporemodel>`, `<formatversion>16</formatversion>`, `<properties>`
    (`modeltype`, `skineffect1-3` ids, `skineffectseed1-3`, `skincolor1-3` as RGB floats, `requiredpack`, `zcorpscore`) and
    `<blocks count="N">` of `<blockref>` entries (`blockid` = `0xGROUP, 0xINSTANCE`, `transform` with `scale`, `position`,
    `triangledirection`, `trianglepickorigin`, 3×3 `orientation`, `snapped`, optional `childlist` of `childid`).
    N = 3–20 (median 10); 972 block references in total; tags counted: blockref 972, transform 972, childid 878.
  - Observed — block resolution: the 972 references use only **5 distinct block keys, all in group `40626000`**; each resolves
    to a `.prop` (810 B in the example) in `Spore_Graphics`/`PatchData` plus a 36-byte `blockdata` (`5caa5e28`) in
    `Spore_Game`/`PatchData`.
  - Observed — block prop `40626000/d05c53a3` (parses exactly): string property `02784061` = `creature_vertebrae`;
    `00f9efbb` and `00f9efc0` = keys to RW4 models `40606000/06b00d54` (7,580 B) and `40606000/dcfbbb49` (2,356 B),
    both found in `Spore_Graphics` with header `89 52 57 34 77 33 32 00 0d 0a 1a 0a` (`‰RW4w32`); other entries: bools,
    ints, floats 0.28 and 2.2, a parent key (`00b2cccb`), a Text array (`8f6fc401`, element 520 bytes, UTF-16 text).
    The 36-byte `blockdata` = nine words, the first six read as floats (−0.0165, −0.0576, −0.0092, 0.0165, 0.0576,
    0.0092 pattern) `[SI]` a bounding box.
  - Strong inference `[SI]`: `CRT` files are premade creatures built as spines of "vertebrae" blocks (name string +
    block structure), i.e. the creature-definition format of the editor family; `00f9efbb`/`00f9efc0` are model-LOD-like
    properties (names not known).
  - Observed — other creature types: `creatureData` (`0f43029a`): 1,365 resources (7.7 MB), binary, magic
    `b7 55 b4 ab`, then a u32 version — 9 in `Spore_Content` (1,022 resources; groups `40626200` 758, `40666200` 153,
    `406b6200` 67), 10 in `Spore_EP1_Content_01` (343 resources; groups `40626200`, `40666200`); size 386–16,655 B.
    `gait` (`25df0112`): 39 resources, magic ASCII `GAIT` + a u32 length (equals the resource size in the sample) + small
    float tables (one example with 11 entries of paired floats), group `00000000`, in `Spore_Game`, `PatchData`, `Spore_EP1_Data`.
  - Unresolved: the 36-byte `blockdata` layout; `creatureData` and `gait` field layouts; RW4 internals (sections,
    vertex/index buffers, skeleton); names of the model-key properties; how the 2,262 non-CRT creature-looking props and
    `creatureData` tie to the CRT chain; meaning of the 3 ids in `modeltype`/`skineffect*`.
- Discriminating test: parse the RW4 container of `40606000/06b00d54` (section table) and render or export its geometry;
  compare the vertebra block's `scale/position/orientation` placement against a rendered spine.
- Follow-ups: INV-016 RW4 container parser and mesh extraction (for the first vertical-slice render);
  INV-017 `blockdata` / `creatureData` layouts with the code side (ModAPI names `Anim`/`Editors` clusters).

---

## INV-016: RW4 container parser, mesh extraction, first assembled creature (Phase 4 slice, first result)

- Status: confirmed for the container (100%) and for the mesh path on 4,243 models; 672 skinned/variant models and all textures open
- Confidence: observed (parsing, corpus test, numeric convention test); renderer correctness judged visually
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all 13,498 type-`2f4e681b` (RW4) resources in the GOG tree; CRT creatures (INV-015)
- Evidence: `tools/rw4.py`, `tools/rw4_corpus.py` (22 s over the whole corpus), `tools/crt_assemble.py`;
  `spore_work/inv016-rw4-corpus.json`; ModAPI `RWHeader.h`, `RWSectionInfo.h`, `Mesh.h`, `VertexBuffer.h`, `IndexBuffer.h`,
  `VertexDescription.h` used as hypotheses and checked against the bytes. Meshes/renders are local only (derived from game assets).
- Findings:
  - Observed container layout: 12-byte magic `89 52 57 34 77 33 32 00 0d 0a 1a 0a`; u32 at 0x1c (=1), 0x24 section count,
    0x28 kind (0x10 model, 4 texture), 0x30 offset of the section-info table, 0x44/0x4c offset/size of a trailing buffer
    area; section info = 24 bytes `{data_off, f04, size, align, type_index, type_code}`; a type-code table follows the header
    (type_index is its index). Check: sections end exactly at the info table, the buffer area ends at EOF
    (e.g. 0x7a4 + 0x190 = 0x934 = file size). Data offsets are absolute file offsets except type `0x10030`–`0x10032`
    sections, which are relative to the buffer area. Pointer-like fields inside structs hold **section indices**.
  - Observed — corpus test: **13,498 / 13,498 containers parse** (7,455 models kind 0x10; 6,043 textures kind 4, matching
    ModAPI's comment). Section type counts: `0x10030` 24,774; `0x20003` 15,509; `0x2000b` 13,764; `0x70002` 9,966;
    `0x20009` (Mesh) 8,632; `0x2001a` 8,632; `0x70001` 7,803; `0x80003` 6,850; `0x20007` (IndexBuffer) 5,020; `0x80005` 4,917;
    `0x20004` (VertexDescription) 4,917; `0x20005` (VertexBuffer) 4,245; `0xff0000` 3,724; `0xff0001` 2,260; `0x200af` 672.
  - Observed — mesh path (ModAPI struct layouts hold): 4,243 models contain Mesh sections; **7,158 meshes extracted with
    4,461,106 triangles and 28,407,995 vertices; 0 index-out-of-range, 0 non-finite/huge positions.** Index formats 101
    (16-bit) and 102 (32-bit) handled; primitive types 4 (list) and 5 (strip); positions Float3; normals UBYTE4 mapped
    byte/127.5−1 (assumed). Example: model `40606000/dcfbbb49` = 16 vertices / 24 triangles, stride 16 (Float3 position at 0,
    UBYTE4 normal at 12); model `40606000/06b00d54` = 90 vertices / 64 triangles with extra blend indices (UBYTE4) and weights (UBYTE4N).
  - Update (INV-017): the 672 failures below are RESOLVED — all decode, see INV-017.
  - Observed — failures: **672 models** (exactly those with a `0x200af` section) fail because a Mesh's vertex-buffer
    reference is not a plain section index (example value `0x400000`, `Spore_Graphics 40606000/3332f07f`, 59,100 B, 32
    sections incl. three Mesh and a 0x93e8-byte `0x200af`). Not decoded. 3,212 other kind-0x10 models hold no Mesh section
    (not examined). Texture sections (kind 4) not decoded.
  - Observed — assembling a CRT: block placement uses `scale`, `position` and the three `orientation` rows. **The rows are
    the world-space directions of the block's local axes (v' = Σ v_i·row_i).** Test over all 47 distinct creatures
    (439 parent→child links): with this convention the block's long local axis (Y of the vertebra mesh) aligns with the
    link direction at mean |cos| = **0.983** (95.4% of links > 0.95, 0.5% < 0.5); the transposed convention gives mean 0.669
    (24.6% > 0.95).
  - Observed — result: the 20-block creature `40626200/9bd7a03a` assembles to 1,800 vertices / 1,280 triangles; the wireframe
    side view is a single smooth curved spine with vertebra size following the XML scales. Placing only the vertebra mesh
    (model A, 90 verts); model B (16-vertex box, ±0.207 × ±0.053 × −0.336…0.022) overlaps neighbours and is not the visible
    vertebra `[SI]` (proxy/handle volume; name of the `00f9efc0` property unknown).
  - Unresolved: the 672-model variant and `0x200af`; skeleton/rig data and how the blend indices bind to bones; textures and
    materials (kinds 0x2000b, 0x70001…); UV coordinates; skin generation (CRT `skineffect*`, `skincolor*`); true normal
    decoding; why the CRT blocks list a second model (`00f9efbb` vs `00f9efc0`); how limbs/other part types look (only
    one part type — vertebra — appears in the 47 CRT creatures).
- Discriminating test: render a creature with a part type other than vertebra (needs other blockrefs / part resources, e.g.
  `creatureData` and the 2,262 other creature props); compare the vertebra mesh dimensions with a Spore-side screenshot or
  with the `blockdata` box.
- Follow-ups: INV-017 decode the `0x200af` variant and skeleton sections; INV-018 `blockdata` and `creatureData` layouts;
  INV-019 skin colour/effect application (flat colour first).

---

## INV-017: skinned/morph RW4 variant (`0x200af`) and RW4 section roles

- Status: confirmed — all 672 variant models decode; every one of the 8,632 Mesh sections in the corpus now yields geometry
- Confidence: observed (validated against a stored bounding box in 672/672 models)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: the 672 type-`2f4e681b` resources with a `0x200af` section (659 in group `40606000`, 12 in `40606100`, 1 in `00000000`)
- Evidence: `tools/rw4.py` (`skinned_blob`, `extract_skinned`), `tools/rw4_corpus.py` rerun (22 s); ModAPI names for section types.
- Findings:
  - Observed — section roles (ModAPI `RW_TYPE` constants; each also appears in the data): `0x20003` Raster (texture),
    `0x20004` VertexDescription, `0x20005` VertexBuffer, `0x20007` IndexBuffer, `0x20009` Mesh, `0x2000b` CompiledState,
    `0x2001a` MeshCompiledStateLink, `0x70001` KeyframeAnim, `0x70002` Skeleton, `0x70003` AnimationSkin, `0x7000b`
    SkeletonsInK, `0x7000c` SkinsInK, `0x7000f` SkinMatrixBuffer, `0xff0000` MorphHandle, `0xff0001` Animations.
    Not named by ModAPI: `0x80003`, `0x80005`, `0x200af`, `0xff0002`. Role split of the 13,498 RW4 resources: **6,043 textures,
    4,243 plain-mesh models, 672 skinned-variant models, 2,540 animation/skeleton-only models.** The `40606000` part-model group
    has 1,419 plain + 659 skinned (32% skinned); `40606100`: 684 + 12.
  - Observed — where the data is: in a skinned model the Mesh sections are draw ranges (first index, index count) over one
    shared IndexBuffer (16-bit triangle list); their `pVertexBuffers[0]` is not a section index (e.g. `0x400000`). Vertex data is
    in the `0x200af` blob: header `u32[0]=1`, `[1..4]` = offsets of the position / normal / tangent / UV streams, `[13]` =
    vertex count; **position, normal and tangent streams each hold `S` consecutive arrays of float4 (xyz + 0), one per
    "state"; UV is one float4 array (u, v, 0, 0)**. `S = (normal_offset − position_offset) / (16 × count)`; header word 15
    is not the state count (0 in many files). Further per-vertex tables follow: 8 bytes/vertex (pairs such as `0x60000, 0xc`)
    and 12 bytes/vertex (float triplets) `[SI]` (blend/morph data, not decoded).
  - Observed — validation: all **672 / 672** models parse; **state 0's bounding box equals the stored bounding box** (section
    `0x80005`: min xyz at 0, max xyz at 16) in 672/672; no index out of range; state counts: 2 (43), 3 (196), 4 (170),
    5 (161), 6 (55), 7 (27), 8 (9), 9 (10), 10 (1). Example `40606000/3332f07f`: 210 vertices, 3 states — state 0 box
    ±2.61 × ±3.19 × −0.05…2.34 (= stored), state 1 flattened (z −0.09…0), state 2 enlarged (±5.46 × ±6.68); vertex stride
    in the description 44 (position, normal, tangent float3 + UV float2). Triangle-edge test picked the layout (mean edge 0.97 / 0.62 / 1.79
    for the three arrays vs 4.28 for random triples).
  - Observed — corpus totals now: **8,632 meshes, 5,039,797 triangles, 29,644,864 vertices, 0 errors.**
  - Strong inference `[SI]`: the states are morph/deformation extremes of a deformable part (state 0 = rest); textures are
    embedded as DXT1 rasters (e.g. 64×64, 0xab8 bytes with mips) next to the mesh.
  - Unresolved: what each state means; the skeleton (`0x70002`) and skin-binding data (`0x70003`, `0x7000c`, `0x7000f`) layouts; the
    8-byte and 12-byte per-vertex tables; how to pick the state for a given creature/handle setting; textures and
    CompiledState (material) contents; `0x80003` and `0xff0002`.
- Discriminating test: apply a part's handle/morph setting from a creature definition and compare the deformed shape with the
  stored geometry; decode a Skeleton and bind the blend indices of a plain mesh (vertebra A has UBYTE4 blend data).
- Follow-ups: INV-019 skeleton + skin binding; texture (DXT1) export for the first textured render.

---

## INV-018: `blockdata` and `creatureData` (partial)

- Status: partially confirmed — structure hints only; layouts NOT decoded
- Confidence: observed counts/relations; semantics `[SI]`
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: `blockdata` (`5caa5e28`, 3,260 resources); `creatureData` (`0f43029a`, 1,365 resources)
- Evidence: `tools/blockdata_test.py`; size fits and key-sibling counts from the GOG tree (this session).
- Findings:
  - Observed — `blockdata`: 1,439 of 3,260 resources are not 36 bytes (variable-length format). The 36-byte form read as
    nine little-endian words/floats: six floats followed by three words. For the 245 whose block prop resolves to an RW4 model,
    the model's state-0 bounding box equals the first six floats exactly in 45 cases (18%); the extent ratio (model/blockdata)
    is `(1,1,1)` in ~25%, `(1.11,1.11,1.11)` in a few, and other patterns. So it behaves like a bounding box of the model but
    is not equal to it in general; no rule found. 27 had no resolvable model.
  - Observed — `creatureData`: magic bytes `b7 55 b4 ab`, then a u32 version (9 in `Spore_Content`, 10 in `Spore_EP1_Content_01`), a
    u32 `N` at offset 8 and two more small counts. Size fits `148 + 163·N` (v9, mean abs residual 162) and `−132 + 168·N`
    (v10, residual 179), i.e. a header plus about 165-byte records. `N` ranges up to ~41 (most common 26, 23, 37, 28). Sizes
    386–16,655 bytes.
  - Observed — sibling sets: all 1,365 `creatureData` share group and instance with a PNG thumbnail (`2f7d0004`), `1a99b06b` (bem),
    `030bdee3` and `02d5c9af`; 635 also have a prop (`00b1b104`); 67 share with `00e6bce5` instead. 937 such sets are in group
    `40626200`, 317 in `40666200`. `creatureData` resources are NOT paired with the 47 CRT files (0 of 47 share a key).
  - Strong inference `[SI]`: each set is a premade creature (the game ships hundreds): thumbnail + creature data (N parts of
    ~165 B) + `bem` and sibling resources; the 47 CRT creatures are a different, simpler set (vertebra spines only).
  - Update (INV-020): the `creatureData` layout is now fully decoded and validated — see INV-020. `N` = number of rigblocks.
  - Unresolved: field layouts of `blockdata`; what `bem` (`1a99b06b`, 2,621 resources, 11.8 MB), `030bdee3`, `02d5c9af` contain.
- Discriminating test: decode one small `creatureData` (N=2, 386 B) completely and check its fields against the sibling `bem`
  and the thumbnail; or resolve a record's key words against the block/model key tables.
- Follow-ups: INV-019 skeleton + skin; INV-020 decode a small `creatureData` + `bem` pair end to end.

---

## INV-020: `creatureData` decoded; first assembled premade creature

- Status: confirmed — 1,365/1,365 parse exactly; assembly agrees qualitatively with the game's own thumbnail
- Confidence: observed (code-derived layout, exact-size and key-resolution tests); visual comparison is qualitative
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all 1,365 type-`0f43029a` resources; the premade creature `40626200/06538e73` (Spore_Content) as the worked example
- Evidence: executable functions `0x4bf970` (reader, ModAPI march2017 address of `cCreatureDataResource::Read`) and `0x4bf7e0` (RLE
  decoder), evidence ids `ev_f069…`, `ev_1667…`; ModAPI `cCreatureDataResource.h`, `cEditorResourceProperties.h`; `tools/creature_data.py`,
  `tools/creature_assemble.py`; sibling thumbnail `Spore_Content 40626200/06538e73` (PNG 128×128, local only).
- Findings:
  - Observed — file layout (from the reader, confirmed by byte arithmetic on a 386-byte sample: 20 + 72 + 2·140 + 8 + 2 + 4 = 386):
    `u32 0xABB455B7`, `u32 version` (9 or 10), `u32 n_rigblocks`, `u32 n_capabilities`, `u32 n_animations`, [v10: `u32 n_effects`],
    then the **properties block** (RLE: chunks of `u16 length` + bytes; byte `0xFE 00` = literal FE, `0xFE c` = repeat previous byte
    `c` times; expands to 128 bytes; 68 if version < 8), then `n_rigblocks × 0x8C` bytes, `n_cap × 4` capability tag bytes
    (ASCII like `spin`), `n_cap` level bytes, `n_anim` u32 ids, `n_anim` f32 values, `n_anim` f32 weights, one u32, and in v10
    `n_effects × 0x38` bytes. Limits in the reader: rigblocks ≤ 256, capabilities ≤ 4096, animations ≤ 1024.
  - Observed — properties block (`cEditorResourceProperties`, 0x80 bytes): `u32 model_type`, `i32 zcorp_score`, 3×`u32` skin-effect ids, 3×`u32`
    seeds, 3× RGB float skin colours, padding. **These are exactly the fields of the CRT XML `<properties>`**, so CRT XML and
    `creatureData`/`bem` carry the same creature data in different encodings (`bem` shares the properties bytes: INV-018).
  - Observed — rigblock record (0x8C bytes, little-endian; ModAPI field names): i16 index, i16 field_2, i16 parent, i16 symmetric, i16 flags,
    i8 field_A, i8 type, i16 capability index, i16 morphs index, i8 n_caps, bbox 6×f32 @0x14, scale f32 @0x2c, orientation 3×3 f32 @0x30,
    position 3×f32 @0x54, effect offset 3×f32 @0x60, scale_rel/socket/muscle/base_muscle/field_7c f32 @0x6c–0x7c, u32 foot/weapon/mouth
    type @0x80, **u32 group @0x84, u32 instance @0x88** (the block's prop key).
  - Observed — validation over all 1,365 resources (1,022 v9, 343 v10): **100% parse and consume the exact file size**; 46,418 rigblocks;
    **0 invalid parent indices; 46,418/46,418 block keys resolve to existing props**; 46,416/46,418 orientation matrices are rotations
    (|det| within 0.01 of 1). Block types: Standard 14,853, Unk3 12,365, Vertebra 11,020, Unk1 7,839, PlantRoot 341.
  - Observed — block models: every model key found in the blocks' props points to an RW4 (no `gmdl` references); main model property
    `00f9efbb` is on 46,356 of 46,418 blocks (resolves in the data for most; 8,002 of 124,790 model references overall do not resolve),
    secondary `00f9efc0` on 28,006, `02424655/6/7` on ~16,800 each. 855 of 5,042 blocks in a 150-creature sample have no resolvable main
    model. The rigblock `bbox` equals the model's state-0 bounding box in only 47 of 4,187 cases — it is some other quantity.
  - Observed — assembly of `40626200/06538e73` (34 blocks: 14 Vertebra, 12 Unk3, 8 Standard; 5,520 vertices, 7,090 faces): positions are absolute;
    the vertebra chain has parent distances ~0.10–0.11 (scale 0.4–1.25) and forms a continuous S-curved spine; Unk3 blocks form
    limb chains (parents 9→14→15→16→foot 30 etc.) with segment distances 0.15–0.33 vs model extent 0.29; Standard blocks 26–29 sit
    ~0.4 from parent vertebrae (head/jaw cluster, with a visible gap the skin would fill `[SI]`); symmetric partner indices are
    consistent (e.g. 14↔17, 15↔18). Same orientation convention as the CRT rows (local axes in world, INV-016).
  - Observed — the creature's own thumbnail (128×128 PNG): a yellow creature with a curved body, beaked face at upper left, four legs
    with clawed feet at the bottom. The reconstruction shows the same pose (S-curve, head cluster left, limbs down). Qualitative only:
    the thumbnail shows skinned geometry, the reconstruction is the block skeleton.
  - Unresolved: the skin surface (generated procedurally from blocks; not reconstructed); morphs (`morph_index`) and handle poses; flags
    meaning beyond ModAPI names; block types Unk1/Unk3; capability tags and animation id/value/weight meaning; v10 effect records
    (0x38 bytes); textures/materials/colours on the render; 17% of blocks without a resolvable main model; `field_2`/`field_A`.
- Discriminating test: project the reconstruction with the thumbnail's camera (needs the view transform) and compare silhouettes;
  render a second premade creature and check against its thumbnail; reconstruct a skin surface and compare.
- Follow-ups: INV-021 unresolved-model audit (which blocks lack models and why); INV-022 apply skin colours/flat shading;
  INV-023 skeleton (`0x70002`) + animation (`ANIM`, gait) for a first posed animation.

---

## INV-021: audit of rigblocks without a main model; texture references; mirror symmetry; full assembly

- Status: confirmed
- Confidence: observed (counts over all 1,365 creatures / 46,418 rigblocks)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all `creatureData` rigblocks and their block props; model properties `00f9efbb`, `00f9efc0`, `02424655`–`02424657`
- Evidence: `tools/creature_assemble.py` (fallback added); audit scripts run this session; retail-disc hash table.
- Findings:
  - Observed — outcome per rigblock: **38,731 (83.4%)** resolve through `00f9efbb`; **7,625** have a `00f9efbb` key whose RW4 does not exist
    (all in group `40606000`; **only 65 distinct models**; 0 of the 65 exist on the retail disc either); **62** Unk1 blocks have no
    `00f9efbb` key. By type: Vertebra 11,020/11,020 ok; Standard 10,332 ok + 4,521 missing; Unk3 9,602 ok + 2,763 missing;
    PlantRoot 0 ok + 341 missing; Unk1 7,777 ok + 62 no key. The same instance IDs exist elsewhere only as props/blockdata/PNG, never as RW4.
  - **Correction (observed):** properties `02424655`, `02424656`, `02424657` are **texture references** — keys of type `2f4e681b` in group
    `406a2100` (the RW4 texture group), not alternate meshes. An earlier test in this session counted them as "resolvable alternatives";
    that reading is withdrawn.
  - Observed — fallback: of the 7,625 blocks with a missing main model, **2,508 (all Unk3) have `00f9efc0` resolving to a decodable mesh**;
    **5,115 have no mesh but three texture references** (PlantRoot 341, Standard 4,521, Unk3 255 — 3 textures each); 2 have nothing.
  - Observed — creatures: of 1,365 `creatureData`, **341 are plants** (each contains one PlantRoot block; all have mesh-less blocks);
    **1,001 non-plant creatures have every block drawable** (main model or `00f9efc0` fallback); **23 non-plant creatures have 1–6
    mesh-less Unk3 'limb' blocks**. Capability tags on mesh-less blocks: `frut` (1,749 tag entries), `spin` (341), `limb` (255).
  - Strong inference `[SI]`: plant parts are generated procedurally from textures (no stored mesh), matching the PlantRoot/`frut` pattern.
  - Observed — symmetry check on all creatures: of 11,621 blocks with a later symmetric partner, **11,482 (98.8%)** have positions mirrored
    in x within 1e-3 (y, z equal); the other 139 differ by ~0.006 (e.g. creature `40616200/00d63aa9`); scales are equal in 11,621/11,621.
  - Observed — assembly: with the fallback, **all 1,001 fully drawable non-plant creatures assemble** (0 non-finite or >100-unit vertices);
    22 of the 23 others assemble with the missing blocks skipped, 1 with two missing, and 10 with three or more.
  - Process note: a tuple-order slip in the key lookup (type,instance,group vs type,group,instance) briefly produced a wrong 13,871-block
    "mesh-less" count; fixed by defining the key order in one place (`ENT` key = (type, group, instance); raw key words are (instance, type, group)).
  - Unresolved: what generates plant geometry and whether it is derivable from the 3 textures; the 255 Unk3 limb blocks without any mesh;
    the cause of the 139 slightly asymmetric pairs; `00f9efc0`'s role (fallback mesh vs. variant) beyond the 2,508 blocks where it is the only model.
- Discriminating test: for creatures with both `00f9efbb` and `00f9efc0`, compare the two meshes (LOD or variant?); render a plant from its textures.
- Follow-ups: INV-022 colour (skin colours vs the thumbnails); INV-023 skeleton + animation; INV-024 texture (DXT) export for plant parts.

---

## INV-022: skin colour decoding checked against thumbnails; first shaded render

- Status: confirmed (statistical, plus one creature checked by eye)
- Confidence: observed; thumbnails include non-skin colours so a perfect match is not expected
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: `creatureData` properties block (3× RGB float skin colours) and each creature's sibling PNG thumbnail
- Evidence: `tools/render.py` (stdlib z-buffer renderer, flat Lambert shading), `tools/png_pixels.py`; comparison run over all creatures with a
  thumbnail; thumbnails and renders stay local.
- Findings:
  - Observed — 1,296 of 1,365 creatures have a thumbnail with enough saturated opaque pixels (alpha > 200, S > 0.25, V > 0.25). Hue distance between
    the thumbnail's mean saturated colour and the nearest of the creature's three skin colours: **median 0.025** (own) vs **0.072** (another random
    creature's skin colours); **73.9% within 0.05 hue (own) vs 36.8% (random)**; own colours closer than a random pairing in 73.7% of comparisons.
    So the three float triples are real RGB skin colours (0–1) that show up in the game's render.
  - Observed — worked example `40626200/06538e73`: skin colours `(1.00, 0.87, 0.16)` yellow, `(1,1,1)` white, `(0.64, 0.31, 0.15)` orange-brown; the
    thumbnail shows a yellow body with orange claws/feet.
  - Observed — shaded render of the assembled block skeleton in colour 1 (two viewing angles): reads as the same body plan as the thumbnail (long
    curved neck/body, four jointed limbs with feet, head cluster at the top). It is parts only: no skin surface, no textures, no eyes/mouth detail,
    no per-part colour, and the view does not reproduce the thumbnail's camera.
  - Unresolved: how the skin colours/effects map onto parts (which colour on which block; `skineffect*` ids); the generated skin surface; part textures
    (the three texture RW4s per block); lighting/camera used by the game.
- Discriminating test: compare silhouettes under a fitted camera; check that colour 3 (orange-brown) appears on the same parts as in the thumbnail
  (feet/claws).
- Follow-ups: INV-023 skeleton + animation; INV-024 DXT texture export and application to parts; INV-025 skin surface (blocks → generated surface).

---

## INV-023: animation data — ANIM container decoded; channel bodies and sampler not yet

- Status: partially confirmed — container, header, channel table, relocation table and names decoded and validated on all 5,022 resources; channel body (keyframes) NOT decoded
- Confidence: observed (container/header/names/version delta); runtime architecture from decompilation; channel semantics open
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all 5,022 type-`ee17c6ad` (`animation`) resources; executable functions `0x99e020`, `0x9b0d60`; RW4 skeleton/keyframe section types
- Evidence: `tools/anim_parse.py`; ModAPI `Anim/SPAnimation.h`, `RenderWare/Skeleton.h`, `KeyframeAnim.h`, `AnimationSkin.h`; decompilation evidence
  ids `ev_2f7c…` (0x99e020), `ev_0c22…` (0x9b0d60); disc↔GOG pairs via the INV-012 effective views.
- Findings:
  - Observed — ANIM header (ModAPI `SPAnimation` agrees with the bytes): `0x00 'ANIM'`, `0x04` u32 file size (= resource size, 5,022/5,022), `0x08` u32
    version, `0x0C` char path[256] (source path), `0x110` u32 id (**equals the resource's instance ID in 5,022/5,022**), `0x11C` f32 frame length,
    `0x120` f32 length in frames, `0x13C` u32 vfx count, `0x140` vfx data offset, `0x144` u32 channel count, `0x148` offset of a table of channel
    offsets, `0x14C` u32 relocation count, `0x150` offset of the relocation table (u32 file offsets of the stored pointer fields, e.g. `0x140`, `0x148`,
    each channel-table entry and each `CHAN` back-pointer). Each channel starts at its offset with `'CHAN'`, a u32 back-pointer, `char name[0x80]`, then
    its body from `+0x88`.
  - Observed — validation: **5,022 / 5,022 parse** (size field, relocation table in range, every channel offset has `CHAN`). Versions: **25 (2,906) and 21
    (2,116)**. Frame length `0.0333` (1/30) in 4,771, `0.0278` (1/36) in 82, `0.0222` (1/45) in 57. Length median 137 frames (1–2,060). 6–13 channels per
    animation typically (max 21).
  - Observed — **channel names are body roles, not bones**: 487 distinct names; top: `root` 4,509, `ears` 4,296, `L foot` 3,820, `R foot` 3,812, `eyes`
    3,349, `head` 3,276, `R hand` 2,935, `L hand` 2,810, `MouthMorphs` 2,008, `Head` 1,414, `Eyes` 1,333, `shoulders` 695, `R grasper` 619, `L grasper` 596,
    `Clavicle` 415, `creature Frm` 338. This is the body-independent authoring scheme of brief §3.2, visible in the data. Casing/spacing vary (`root`/`Root`).
  - Observed — disc vs GOG: all **2,085** animations present in both effective views are **v21 on the disc and v25 in GOG**; length and frame length are
    identical in 2,077 (99.6%); GOG files are about half the size (median ratio 0.49). Channel names identical in 1,029; identical modulo case/space/order in 10;
    **different sets in 1,046** — nearly always **channels removed in v25** (−1: 656, −2: 192, −3: 77, −4: 42, −5: 19, −9: 18), most often `shoulders` (602),
    then `mouthMorphs` (70), `clavicle` (57), hand and eye channels; very few added. So animation content differs between the lineages; fixtures must record
    the lineage and version.
  - Observed — channel body: DECODED in INV-024 (below).
  - Observed — runtime code: `0x99e020` is an animation **blend/accumulate** routine (not the file loader; its apparent `ANIM` constants were
    coincidental bytes): it loops over active animation instances (16-byte records), evaluates each channel through `0x9b0d60`, and adds weighted position (xyz)
    and rotation (4-float quaternion-style) into per-role accumulators in the creature object, with separate paths for root/frame motion and for hand/foot/head
    roles. `0x9b0d60` evaluates one channel's pose: inputs are flag words for the position and rotation parts (bit `0x10` = present; `0x20/0x40/0x60` = rotation
    modes; `0x100/0x120/0x200` = reference-frame modes; header flag bit 1/2/8/0x10 selecting IK-like, scaled and absolute variants), helper calls that scale by the
    creature's morphology, and quaternion products; outputs a position and a rotation. `[SI]` this is Hecker's retargeting step (§3.2): pose constraints per role
    adapted to the actual body.
  - Observed — RW4 side: model files also hold animation sections (ModAPI: `0x70001` KeyframeAnim with channel names/count/skeleton id/length, `0x70002`
    Skeleton with bone ids/flags/parents, `0x70003` AnimationSkin, `0x7000b/c/f`); 2,540 RW4 models contain only such sections (groups `40636200`,
    `41606100`, `40637100`, `40667100`, `40646200`). A second animation path (skeletal keyframes). Not decoded here.
  - Unresolved: the channel body layout and the keyframe sampler/loader; semantics of the flag bits; how morphology-dependent scaling is derived; `gait`
    tables (`GAIT`, 39 resources) and how they pair with animations; the meaning of `INFO` sub-blocks; `0x80003`/`0xff0002` sections; whether v21 and v25 differ
    in channel encoding as well as channel set.
- Discriminating test: find the keyframe sampler (callers of `0x9b0d60` that fill its position/rotation inputs; `CHAN`/`INFO` tag comparisons in code), then
  decode one small channel and compare against its channel name's expected motion (e.g. an idle `root` bob); check value ranges across the corpus.
- Follow-ups: INV-024 sampler and channel body decode (start from `0x9b0d60`'s callers); INV-025 DXT textures; INV-026 skin surface.

---

## INV-024: ANIM channel body — keyframes decoded; walk cycles recovered

- Status: confirmed — 48,700 / 48,700 channels decode exactly; semantic checks pass
- Confidence: observed (structure validated at scale; semantics checked against gait behaviour)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all channels of all 5,022 `ANIM` resources (both versions)
- Evidence: `tools/anim_parse.py` (`decode_channel`); analyses in this session; plot of a walk animation's foot curves (local only).
- Findings:
  - Observed — channel body layout: at `CHAN + 0x88 + 0x4c`: `u32 n_keys`, `u32 keys_offset`, `u32 n_info`, `u32 info_offset`. The **INFO table** has `n_info`
    entries of 0x20 bytes `{u32 flags, u32 tag, u32 offset_in_key, u32 key_size, …}`; entry 0 has tag `'INFO'`; other tags are `'POS\0'` (0x00534f50), `'ROT\0'`
    (0x00544f52) and small integers 1…6 (scalar tracks). The keys are `n_keys` records of `key_size` bytes, back to back, **ending exactly at the end of the channel**.
    Each key: u32 frame number at +0; the `POS` track (at its INFO offset) = x, y, z, weight(1.0) + further floats; the `ROT` track = quaternion x, y, z, w + weight + further floats;
    scalar tracks follow at their INFO offsets (e.g. +0xc8, +0xf0, +0x118, +0x140, +0x168, +0x190 in the 0x1b8-byte layout). The INFO `flags` word per track holds the mode bits
    the runtime evaluator tests (`0x10` present, others select absolute/relative/scaled modes; e.g. POS flags 0x31/0x71/0x91, ROT 0x12/0x22/0x32) `[SI]`.
  - Observed — validation: **48,700 of 48,700 channels** (v21: 21,572; v25: 27,128) satisfy `keys_offset + n_keys × key_size == channel end`; **878,791 keyframes**; frame numbers
    non-decreasing in 100% of channels and ≤ animation length + 1 in 99.4%; **731,383 ROT quaternions have median length 1.0000 and 99.87% within 0.01 of 1**; keys per channel
    median 13, max 511.
  - Observed — **version difference explained**: every v21 (disc) channel uses a **440-byte key** with a full track set (POS, ROT, six scalar tracks); v25 (GOG) uses **compact keys**
    of 60, 140, 200, 240 or 280 bytes (fewer tracks per key). Together with removed channels (INV-023) this accounts for the ~0.49 size ratio.
  - Observed — semantics: root keyframes vary smoothly (e.g. POS (0.005, 0, 0.02) → (−0.031, 0, 0) → (−0.064, −0.0, 0.029); ROT (−0.0, −0.046, 0.079, 0.996)). **Left/right foot position
    tracks are anti-phased**: over animations with usable foot tracks, the most negative per-axis correlation between L foot and R foot of the same animation has median −0.95 for
    walk/run-named animations (n=23; 87% below −0.8), −0.69 for the other 2,256 (38.4% below −0.8), versus median −0.23 (3.2% below −0.8) for a left foot paired with another animation's
    right foot (n=986). Plot of `soc_gift_walk_1_noG.animation` (133 frames, 26 keys per foot): three repeated lift–swing–plant cycles per foot, right foot offset by about half a period from
    the left, then a held pose.
  - Unresolved: the extra floats after xyz/quaternion in POS/ROT (tangents/ease/weights?); the six scalar tracks' meaning (blend/morph/weight values?); the interpolation law used at runtime
    (linear/slerp assumed in my checks); how POS/ROT are expressed (body-relative units? reference frame per flags); VFX event table (4 entries of 0x60 bytes, names stored as text after the relocation
    table) not decoded beyond names; the RW4 skeletal-keyframe path; `gait` resources.
- Discriminating test: apply decoded POS/ROT of the feet/hand/head/root channels to an assembled creature (per-role retargeting) and compare against a game video/screenshot of the same
  animation, or compare the stride length of a walk animation with the leg length of creatures that use it.
- Follow-ups: INV-025 DXT textures + plants; INV-026 skin surface; INV-027 role-to-block mapping (which blocks a channel such as `L foot` drives) and first retargeted pose.

---

## INV-027: mapping animation roles to creature blocks (partial)

- Status: partially confirmed — role vocabulary and mode flags established; units and retargeting NOT established; no posed creature yet
- Confidence: observed for the tag/flag relations and the flag table; units hypotheses inconclusive
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: capability tags/flags of all 46,418 rigblocks; INFO track flags of every role-named channel in all `ANIM` resources
- Evidence: this session's scans; `tools/anim_parse.py`; decompiled evaluator `0x9b0d60` (INV-023).
- Findings:
  - Observed — block roles are carried by the capability tags (4-char ids; ModAPI calls them PCTP identifiers). Tag vocabulary (entries): `limb` 12,365 (all Unk3), `spin`
    11,361 (11,020 Vertebra + 341 PlantRoot), `cute` 8,210, `socl` 5,133, `mean` 4,538, `heal` 2,926, `chrm` 2,679, `cspd` 2,499, **`foot` 2,411**, **`eye` 2,117**, `sens` 2,117,
    `crge` 1,776, `frut` 1,749, `dvis` 1,565, `danc` 1,542, `spnt` 1,463, **`grsp` 1,280**, `jump` 1,022, `poke` 1,008, `strk` 990, **`mout`/`bite`/`voca`/`call` 972 each**,
    `stel` 967, `post` 801, `tarm` 769, `carn` 720, `herb` 640, **`ear` 624**, `tatk` 571, `tsoc` 394, `eycl` 311, `mvcl` 308, `wing` 293, `fin` 215, `spit` 137 (names are the ids; meanings of
    most are inferred from the letters only).
  - Observed — **block flag `0x20` (ModAPI `kFlagIsFoot`) is set on exactly the blocks tagged `foot`: 2,411 of 2,411, and on no other block** (no block with the tag lacks the flag; none with the flag
    lacks the tag). `Standard` blocks carry the foot/grasper/eye/mouth tags; no block has the weapon flag `0x80`.
  - Observed — animation channel roles (normalised names) and their track mode flags (POS / ROT `INFO` flags): `root` POS 0x71 (3,528 of 5,047), 0x31 (933), 0xb1 (531); ROT 0x32 (5,006).
    `head` POS 0x71 (3,631), 0x31 (1,049), 0x271 (134); ROT 0x32 (4,651), 0x52 (181), 0x12 (156). `L foot` POS 0x71 (2,842 of 4,269), 0xb1 (753), 0x31 (665); ROT 0x32 (4,266); `R foot` the same
    pattern (0x71 2,813 / 0xb1 764 / 0x31 699). `L hand` POS 0x71 / 0xb1 / 0x31 and ROT 0x32 / 0x12 / 0x22 (3,730 channels; 256 without POS); `R hand` likewise. `creatureFrm`: POS 0x11 (776),
    0x51 (420), 0x91 (193), ROT 0x12 (1,401). `eyes`, `ears`, `MouthMorphs`, `shoulders`: mostly POS 0x61/0x21 and ROT 0x22, and **58%/58%/54% of eyes/ears/MouthMorphs channels have no POS
    track at all** (scalar tracks only). Low nibble of a track flag = track type (1 POS, 2 ROT); higher bits select modes; the runtime evaluator tests `0x10`, `0x20`, `0x40`, `0x60`, `0x100`, `0x120`,
    `0x200` and, for the header word, bits 1/2/8/0x10 (INV-023). The exact mode meanings are not established.
  - Observed — walk example (`soc_gift_walk_1_noG.animation`, v25): `creatureFrm` POS flags 0x11 with z hop to 0.84–0.88 and ROT 0x12; `root` POS 0x71, ROT 0x32 with z moving 0.33 → −0.24 → −0.05; `head` POS 0x71,
    ROT 0x32; first keys are near zero or small offsets.
  - **Inconclusive (test design flaw acknowledged):** a check of candidate POS units against leg lengths over 903 creatures × 13 walk animations was circular for the "× leg chain length" hypothesis
    (coefficient of variation 0.00 by construction) and therefore is not evidence. The only non-circular figure: with raw animation units, a foot's maximum excursion is a median 0.47 of its own leg-chain length
    (CV 0.39 across creatures); animations' max foot offsets range 0.0 (no foot motion) to ~0.5. Consistent with raw units being comparable to creature units `[SI]`; unproven.
  - Unresolved: the unit/reference frame of POS (the executable's morphology helpers `0x9b05f0`, `0x9b0b80`, `0x9b06a0`, `0x9b0c50` compute body-relative scaling); which block(s) each role drives (L/R
    assignment convention; quadrupeds with four feet vs two animation foot channels); IK for limb chains; how `creatureFrm`/`root`/`head` combine; eyes/mouth scalar tracks as morph weights.
- Discriminating test: decompile `0x9b05f0` / `0x9b0b80` to get the body-relative frame and scale, then pose one creature with a walk and check that feet stay within reach of their chains and that stride
  length scales with leg length.
- Follow-ups: INV-028 read the morphology helpers and implement retargeting for foot/root/head; INV-029 DXT textures and plants; INV-030 skin surface.

---

## INV-028: morphology helpers of the animation evaluator (retargeting structure; units still open)

- Status: partially confirmed — structure of the body-relative scaling read from code; unit of POS not resolved; no retargeted pose yet
- Confidence: observed (decompiled code, data tables); units inconclusive
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: executable helpers `0x9b05f0`, `0x9b0b80` (called by the channel evaluator `0x9b0d60`); ModAPI `anim_csd`; channel body headers
- Evidence: decompilation `ev_2e0c…` (0x9b05f0), `ev_4116…` (0x9b0b80); ModAPI `Anim/anim_csd.h`; per-role tables from all 5,022 `ANIM` files; foot-excursion tests over 903 creatures × 11 walk animations.
- Findings:
  - Observed — `0x9b05f0(out, creature, channel_flags, default_pos)`: if the channel descriptor's word 5 `& 0x100003 == 2` (body-part-bound mode) it asks the creature for the bound part's position (`0x9cd4b0`), subtracts the creature
    origin (`creature+0x18..0x20`), and rotates it into the creature frame (`0x99c220` with the matrix at `creature+0x3c`); otherwise it returns the default position unchanged. i.e. the reference point of a role channel is the
    **creature's own part position** — the morphology dependence.
  - Observed — `0x9b0b80(vec, flags, _, creature, anim)` returns a scale: if `(flags & 0xc0) == 0x40` → `Σ vec_i · extent_i · s` with `extent` = three floats at `csd+0x364/0x368/0x36c` and `s` = creature scale (`creature+0x70`) (+1e-8);
    if `(flags & 0xc0) == 0x80` → a single size scalar (`csd+0x374`, or a value at `anim+0x434`, selected by a bit at `anim+0x154`) × `s` (1.0 if ≤ 1e-6); else 1. So a POS flag with `0x40` (the common `0x71`) means the components are
    **fractions of per-axis creature extents**; `0xb1`/`0x91` (`0x80`) scale by a single size; `0x31`/`0x11` apply no morphology scale `[SI]` (the mapping from file flag bits to these tests is read from the evaluator's use of the same bit masks).
  - Observed — ModAPI's `anim_csd` (creature static animation data; vector of `CreatureBlock` at `+0x384`, `isPlant` at `+0x3f0`) has an unnamed `Vector3 field_364` (the extents) and `float field_374` (the size scalar), plus Vector3s at
    `0x324/0x330/0x33c/0x358`. The code that fills them from creature data was not found.
  - Observed — channel kinds from the first words of each channel body (`CHAN+0x88`): word1 `& 0x100003`: `creatureFrm` 0x100000 (1,355 of 1,406) or 0x100001 (51); `root` 0x1 (5,047 of 5,047); every other role 0x0 (heads 5,035,
    eyes 4,765, ears 4,712, MouthMorphs 2,751, shoulders 922, L/R hands 3,730/4,270, L/R feet 4,269/4,285). Word0: mostly 0 or 0x10 (root 0x10 in 3,433; creatureFrm 0 in 1,097, 0x10 in 301; head 0x2 in 529; L hand 0x20 in 334).
  - Inconclusive: tests of candidate extents — foot excursion × (candidate extents) / own leg-chain length over 903 creatures: raw (1,1,1) CV 0.39 (median 0.47); block-position bbox dims CV 0.37 (0.43); block-bbox-inclusive dims CV 0.36 (0.59);
    uniform max position dim CV 0.31 (0.63); uniform height CV 0.35 (0.56). No hypothesis is clearly tighter than raw, so the data cannot discriminate them (a "× leg length" variant is circular and excluded). Walks differ in how
    much of the leg they use, which may dominate the spread.
  - Unresolved: where `csd+0x364`/`+0x374` are computed; what `creature+0x70` scale is; mapping of file flag bits to code masks beyond the inference; IK for limb chains; L/R convention; quadruped foot assignment; how `creatureFrm`/`root`/`head` compose.
- Discriminating test: find the `anim_csd` builder (writers of `csd+0x364`, e.g. via references to the `cal3DRootBlockIndex` initialisation or `isPlant`), compute the extents, then pose one creature and check foot reach and stride/leg ratio.
- Follow-ups: INV-029 DXT textures and plants (visible progress, independent of retargeting); INV-030 skin surface; INV-031 `anim_csd` builder and retargeted walk.

---

## INV-029: RW4 textures (Raster) decoded; skin-paint brush role confirmed

- Status: confirmed
- Confidence: observed (exact size checks on every texture; visual coherence); role from ModAPI's reconstructed source
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: all 6,043 RW4 resources with kind 4; the three texture properties of rigblock props
- Evidence: `tools/rw4_raster.py` (Raster reader, DXT1/3/5 + A8R8G8B8/L8/A8/R5G6B5 decoders, PNG writer); ModAPI `Raster.h` and `Documentation/SourceCodeSkinner.md`.
- Findings:
  - Observed — Raster struct (ModAPI layout holds): +0 u32 format (FourCC `DXT1/3/5` or D3DFORMAT number), +4 u16 flags (0x08 texture, 0x1000 cube), +0xC u16 width, +0xE u16 height, +0x11 u8 levels, +0x1C data
    pointer = section index of a type-`0x10030` buffer. **Validation: for all 6,043 texture files (one raster each) the mip-chain size (largest level first, down to 1×1, ×6 faces for cube maps) equals the data buffer size exactly**
    (an initial 31 mismatches were the cube maps, resolved by the six-face rule).
  - Observed — formats: DXT5 3,756; A8R8G8B8 1,502; DXT1 549; L8 234; DXT3 2. Sizes: 128×128 3,848; 256×256 834; 64×64 560; 512×512 286; 32×32 202; 256×512 50; 64×512 44; 1024×1024 32. Levels 6–11 (8: 3,880; 9: 915; 7: 578; 10: 393).
  - Observed — a decoded plant-part set (block `40666000/6f9bbc38`, textures `406a2100/{83db526d, b4904e9c, 1dbc36e6}`): the colour map (DXT5 128×128) shows a scalloped leaf fringe along the top and a spoked circular shape bottom-right; the
    alpha map (DXT5) outlines the same two shapes; the third (A8R8G8B8) is almost blank. Colour and mask agree spatially.
  - Observed (external confirmation) — ModAPI's reconstructed `cSkinPainterJobPaintParts` reads rigblock property **`0x2424655` as `skinpaintDiffuseTexture`** (with the following ids as the tint mask and spec/bump textures) and draws it into the creature's skin texture
    using per-rigblock UVs; property names `skinpaintBumpHeight` (`0c19db33`) and `skinpaintPartBumpScale` (`1c76d9b5`) are among the 499 hash-named ids (INV-014). So the three per-block textures are **skin-paint brushes**, not the part mesh's own textures;
    mesh-less blocks (INV-021) are paint-only parts. This corrects the earlier `[SI]` that plant geometry was generated from them.
  - Unresolved: tint-mask and spec/bump channel semantics; the UV layout per rigblock; the painter's pipeline and shaders; the property→role mapping for `0x2424656/7` (assumed from ModAPI's code order).
- Discriminating test: reproduce a creature's skin texture by painting the decoded brushes at the rigblock UVs and compare to the game's skin; requires the skin mesh UV generation.
- Follow-ups: INV-030 skin proxy; INV-031 animation extents/retarget.

---

## INV-030: skin proxy and thumbnail-silhouette test of the assembled geometry (NOT the real skin generator)

- Status: confirmed as a test of placement; real skin generation NOT reconstructed
- Confidence: observed (paired comparison); the proxy is crude by design
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: 150 non-plant creatures with all blocks drawable and a thumbnail PNG; skin geometry generator (not reconstructed)
- Evidence: `tools/skin_proxy.py` (union of discs along parent→child block segments, radius 0.6 × scale × median model extent; orthographic projection; camera grid of 18 yaws × 5 pitches; scale/offset fitted to the thumbnail's alpha bounding box; IoU at 64×64).
  ModAPI documents only the skin _painter_ (`Skinner` painting jobs), not the mesh generator.
- Findings:
  - Observed — over 150 creatures × 2 random partners (300 pairs): best silhouette IoU with the creature's own thumbnail has **median 0.578**; with a random other creature's thumbnail **median 0.486**; paired difference median +0.085 (p10 −0.023, p90 +0.217).
    **The own thumbnail fits better in 250 of 300 pairs (83.3%).** (An earlier run printed a "120 of 120" by comparing sorted lists; that figure was invalid and is withdrawn — the paired result above is the real one.)
  - Strong inference `[SI]`: the assembled block layout reproduces creature proportions well enough to match thumbnails better than chance in most cases; the gap is modest because bounding-box fitting makes any blob overlap strongly and the proxy omits the real skin's shape.
  - Unresolved: the real skin generator (metaball/surface algorithm), skin UVs, how morph handles change the surface, the thumbnails' real camera and lighting, why 17% of pairs do not favour the own thumbnail (crude proxy, extreme poses, or limb sizes).
- Discriminating test: locate the skin-mesh builder in the executable (callers of the painter, `cSkinMesh`/`mpCreatureData` users) and compare its vertices with the proxy; or fit the thumbnail camera more precisely and test per-part overlays.
- Follow-ups: INV-031 anim_csd extents / retargeted walk; later: skin mesh generator.

---

## INV-031: the animation extents (`anim_csd` +0x364) — NOT found; editor→creatureData converter decoded as a by-product

- Status: negative result for the stated goal (extents builder not located; no retargeted walk); positive by-product (semantics of creatureData fields from `0x4c7460`)
- Confidence: observed (decompiled code); extents unresolved
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: writers of `anim_csd` fields `+0x324/0x330/0x33c/0x358/0x364/0x368/0x36c/0x374/0x378/0x3f0` (ModAPI); `Editors::cEditorSkinMeshBase::FromRigblocks` (`0x4c7460`)
- Evidence: full-executable linear disassembly (`llvm-objdump`, 5.5 M lines, local), store-pattern searches, decompilations `ev_bf01…` (0x4c7460), `ev_5c3b…` (0x4e1d20); ModAPI `anim_csd.h`.
- Findings (goal):
  - Observed — the executable contains 87/78/54/53 stores to offsets `0x364/0x368/0x36c/0x374` across unrelated classes; restricted to float stores forming a +0x364/+0x368/+0x36c triple there are 7 sites
    (`0x97fd86`, `0x9b31f4`, `0x9b35de`, `0x9b3f78`, `0x9c1f30`, `0x9c4a08`, `0x9c8f01`). `0x4e1d20` (the 0x4e2bb1 site) is a large copy-constructor of a 0x440-byte object, and `0x9b3d10` (the 0x9b3f78 site) is a 925-byte
    single-block field copier; a constructor-like region at `0x9c1d62` writes all ten `anim_csd` offsets in ModAPI order. None was confirmed as the code that computes the extents from creature data.
  - Not done: pose a creature with a decoded walk; determine the POS unit; IK. The only non-circular unit evidence remains INV-028's (inconclusive).
- Findings (by-product) — `0x4c7460(editor_rigblock_array, count, out)` builds the `creatureData` rigblock list from the editor's live rigblock objects and thus defines how the file fields arise:
  - Observed — type byte: editor object flags (`obj+0xdc8`) `& 0x80` → **5 Vertebra**; `& 0x100` → **4 PlantRoot**; `& 0x800` → **3 (Unk3, the limb blocks)**; `& 0x100000` → **0 NullBlock**; otherwise **2 Standard** when the index is below the block count and **1 Unk1**
    otherwise.
  - Observed — block flag bits come from: `obj+0x5f0 > 0` (or a model-collision test) → `0x8` (baked); `obj flag 0x400` → `0x1` (no skin); `obj flag 0x80000000` → `0x4`; `obj+0x1cc == 1` → `0x10`; property `0xb6f17b6c` false → `0x200` (jiggle); property `0xa64e0669` true → `0x800` (extra jiggly);
    a capability found for id `0xdd94fb65` → `0x20` (foot); for id `0x2dc2a89d` → `0x80` (weapon); property `0xf48eb09` (a key whose instance id equals the block's own) → `0x100`; `obj+0xdcc & 0x2000000` → `0x1000` (asymmetric).
  - Observed — other record fields: `parent` and `symmetric` indices are remapped through the editor's object links (−1 if the partner is not in the list); `bbox` from the model query (vtable +0x54); scale from `obj+0x1d8`; orientation 9 floats copied from `obj+0x60`; position from `obj+0x48..0x50`;
    `field_7C`-like values from `obj+0x1d0`; props `0x68de7a9` (a Vector3, with `0xfba610/611/613` overrides) set the effect offset; `0x68de7aa/ab` interpolate between the bbox corners.
  - Observed — **v10 effect records (0x38 bytes) come from block props**: key array `0x32f4549` (`GetArrayKey`) paired with a `GetArrayTransform` array `0x2a907b6` (56-byte transforms): each effect record = (rigblock index, key instance, the transform). Both ids are among the frequent unnamed property ids of INV-014
    (`02a907b6` 3,201×, `02a907b5` 4,003×), consistent.
  - Observed — the type/flag semantics confirm and refine INV-020 (Unk3 = limb blocks, Unk1 = extra/auxiliary blocks beyond the count) and INV-027 (`foot` flag/tag).
  - Unresolved: the extents; units; the semantics of `Unk1` blocks; the remaining flag meanings (`0x100`, `0x4`, `0x10`); the real skin mesh generator (the `FromRigblocks` result is the input to the skin build; not followed).
- Discriminating test: find the creature→`anim_csd` constructor by following callers of the evaluator's setup (e.g. `cEditorAnimWorld::GetAnimatedCreature` `0x59ca70` and the function creating the animated creature), then read how `csd+0x364` is derived (compare with the block bboxes/positions).
- Follow-ups: INV-032 follow `cEditorAnimWorld`/`GetAnimatedCreature` to the `anim_csd` builder; INV-033 skin mesh generator from `cEditorSkinMeshBase` users; Phase 5 planning.

---

## INV-032: the `anim_csd` builder located and the csd-level extents (`+0x364`, `+0x374`) derived from code (retarget test not yet run)

- Status: partial — builder and extent formulas decoded; no posed creature / retargeted walk yet
- Confidence: observed (decompiled code); the interpretation of `+0x374` is strong inference `[SI]`
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: GOG EP1 `SporeApp.exe` 3.1.0.29: `Anim_LoadAnimatedCreature` (`0xa0b1c0`), csd builder `0x9cbe10`, finaliser `0x9c4950`, `0x9b69e0`, `0x9b5230`, `0x9b50d0`, evaluator helper `0x9b0b80`
- Evidence: decompilations of those functions plus `0x9b0d60`, `0x9b52c0`, `0x9cbce0` (REA/Ghidra, local); the linear disassembly of `0x9cbe10`; `procedure_callers(0xa05ca0)` = only `0xa0b1c0`; `procedure_callers(0x9b0b80)` = only `0x9b0d60`.
- Findings:
  - Observed — `0xa0b1c0` allocates `Anim/World/new_csd` (0x410 B), loads the creatureData and calls `0x9cbe10(csd, creatureData)`, then allocates `new_cid`, `new_cmid`, `new_ac` (`0xa05ca0`), `new_qb`.
  - Observed — `0x9cbe10` fills a vector of **0x468-byte per-block entries** at csd `+0x384` (end pointer `+0x388`): `+0x10c` position, `+0x118` quaternion (from the 3x3 orientation), `+0x138` half-extents `(bbox size x scale)/2` (min 0.1), `+0x144` bbox offset, `+0x150/+0x154` flags, `+0x1f8/+0x1fc` joint chain indices, `+0x344` a block vector, `+0x35c..+0x364` the `0x9b52c0` offset vector, `+0x368` its length, `+0x36c` = FLT_MAX. It copies the skin colours from `creatureData+0x40` and looks up "Pollen Metadata" (`030bdee3`). **Correction of the earlier draft of this entry:** the per-block `+0x364..+0x36c` fields are not what the evaluator reads (see next).
  - Observed — in `0x9b0b80` (called only from `0x9b0d60` with `param_4 = param_7`, `param_5 = param_8`), `*param_4` is the **csd** and `*param_5` a block entry. So the csd-level fields `+0x364/+0x368/+0x36c` (extents Vector3) and `+0x374` (size) are the ones the evaluator uses. INV-027/028/031's premise (csd-level extents) was right; INV-032's first draft wrongly retracted it.
  - Observed — the csd extents are written by the finaliser `0x9c4950`: `0x9b69e0` computes the axis-aligned min/max over the 8 corners of every block's oriented box (`position +/- R(quaternion) x half-extent`, blocks at `+0x10c/+0x118/+0x138`); then `+0x358..+0x360` = min, **`+0x364..+0x36c` = max - min (box size)**, `+0x370` = its length. Strong inference `[SI]` for the exact corner set (decompiled as eight rotated half-extent combinations).
  - Observed — **`+0x374` = `0x9b5230`: the mean, over blocks whose `+0x154` has bit `0x4`, of `0x9b50d0(block)`**, which walks the block's joint chain (`+0x1f8` start, `+0x1fc` next, until `+0x430`) and sums `|+0x344|` of the first entry, then `|child.+0x344 - parent.+0x350|` per link, then `|+0x350|` of the last; clamped to at least 0.01. This is a **limb-chain length** (per block flagged 0x4, averaged). `[SI]`: flag `0x4` is the foot/limb-end marker; its builder store is not yet pinned.
  - Observed — the evaluator (`0x9b0b80`): POS flag `0x40` -> `dot(direction, (+0x364,+0x36c,+0x368)) x scale`; flag `0x80` -> `(+0x374, or +0x434 of the target block) x scale`, where `scale = param_4[0x1c]` (instance scale).
  - Not done: the test below; pinning flag `0x4`; IK; the L/R convention.
- Strong inference `[SI]`: stride/foot POS tracks are expressed in fractions of the creature's own box size (0x40) and limb-chain length (0x80), which explains why INV-028's global candidates never separated.
- Discriminating test (next): from creatureData compute the oriented-box AABB size and the limb-chain length for many creatures; multiply the foot POS track by the matching term and check that foot excursion (in world units) scales with leg length across creatures (CV should fall well below 0.31).
- Follow-ups: INV-032b run that test and pose one walk; INV-033 skin mesh generator; Phase 5 roadmap.

---

## INV-032b: retarget scaling test with the decoded csd extents — inconclusive (prediction not met)

- Status: negative/inconclusive for "extents make foot excursion scale with leg length"; no posed creature
- Confidence: observed (numbers); explanation open
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: 918 non-plant creatures x 11 walk `ANIM`s whose foot channels have POS flag `0x71` (the 0x40 "fraction of extents" mode)
- Evidence: `tools/retarget_test.py`, results `ledger/inv032b-retarget.json`. Extents = oriented-box AABB size per INV-032 (block centre taken as position + R x bbox-centre x scale; assumed, not read from the builder); leg length = mean over foot-flagged blocks of the parent-link distance sum up to the Vertebra; CV of (foot excursion in world units / leg length) across creatures, averaged over walks.
- Findings:
  - Observed — raw excursion (no extents) CV **0.381**; extents with the 6 axis mappings 0.34 (two mappings: perm (0,2,1) and (1,2,0)) to 0.42; uniform scalar (max or mean extent) **0.33**. Pre-stated expectation was "well below 0.31"; **it was not met**. Best result is a ~13% reduction, which is within what the crude leg proxy and walk differences could produce.
  - Not established — which axis mapping is right (two share the lowest per-axis CV, both put the anim y component on the extents' z axis), whether the foot excursion is stride-limited rather than size-limited, and whether my block-centre and leg-chain proxies match `0x9cbe10`/`0x9b50d0` (the latter uses `+0x344/+0x350` joint vectors, not block positions).
  - Strong inference `[SI]` unchanged: the code reads extents as in INV-032; the data do not yet confirm the units. Extents from code and from this test agree only weakly.
- Discriminating test: reproduce `+0x344/+0x350` (joint vectors) and flag `0x4` from `0x9cbe10` so the limb-chain length matches `0x9b50d0` exactly, then pose one creature (foot POS x extents in its frame) and check that the foot stays within the chain's reach.
- Follow-ups: INV-032c joint-vector reproduction; INV-033 skin generator; Phase 5 roadmap.

---

## INV-032c: csd block entries reproduced from `0x9cbe10`; exact limb-chain length; retarget test re-run (improved, still not decisive)

- Status: partial — entry construction, foot flag and chain stop rule decoded and implemented; no posed creature
- Confidence: observed (decompiled code, 918-creature test); axis interpretation open
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: `0x9cbe10`, `0x9fc080`, `0x9fc160`, `0x9b2ee0`, `0x9b2e60`, `0x9b4bb0`, `0x9b50d0`, `0x9b5230`
- Evidence: decompilations (REA/Ghidra, local); `tools/retarget_test.py --exact`; results `ledger/inv032c-retarget-exact.json` (the INV-032b run is `inv032b-retarget.json`).
- Findings:
  - Observed — **entry order**: the block that is the parent of rigblock 0 (if any) becomes entry 0; indices below it shift +1 (`m(i)=i+1` if `i<r`, `0` if `i==r`, else `i`); parents map the same way, `-1` stays.
  - Observed — **block centre** = `P + Σ c_i·row_i` with `c = (bbox_lo+bbox_hi)/2 × scale` (orientation rows = world directions of the local axes, so this equals the earlier `R·c`); half-extents `(bbox_hi−bbox_lo)×scale/2` (min 0.1); `+0x144` = −c (block origin in the box-centre frame).
  - Observed — **joint vectors** (second loop, entries with a parent): joint point `J = P`, except for a Vertebra (type 5) with a parent, where `J` = midpoint of its `P` and its parent's `P`. `+0x344 = Rc⁻¹(J − centre_child)`, `+0x350 = Rp⁻¹(J − centre_parent)`; `+0x35c.. = +0x344 − (+0x144)` when flag `8`, length at `+0x368`.
  - Observed — **per-block capability bits** (`0x9b2e60`, from the 4-char ids as little-endian u32): `root` 1, `limb` 2, **`foot` 4**, `spin` 8, `psft` 0x10, `mout` 0x20, `grsp` 0x40, `nstr` 0x80. Word `+0x150` ORs all of the block's capabilities (after expansion through a global implication table `DAT_0166c080`, not decoded); word `+0x154` ORs only those with a non-zero level. **So the INV-032 "flag 0x4" is the foot capability with a non-zero level** (resolves that open item); `+0x3c8` counts such blocks.
  - Observed — **chain stop** (`0x9b4bb0`): for each entry, `+0x430` = the first ancestor (following `+0x1fc`) that is entry 0 or has `spin` set; for a `spin` block it is itself. `0x9b50d0(foot)` = `|j344(foot)| + Σ |j344(a) − j350(prev)|` over ancestors up to (not including) the stop, `+ |j350(last)|`, floored at 0.01; `+0x374` = mean over foot blocks.
  - Observed — re-run of the INV-032b test with that **exact** leg length (918 creatures × 11 walks): raw CV **0.320** (was 0.381); extents with the identity axis mapping **0.344** (worse than raw); mappings (0,2,1)/(1,2,0) **0.288/0.289**; uniform max extent 0.277; **uniform mean extent 0.258** (best, ~19% below raw). The pre-stated "well below 0.31" bar is met only by the uniform-scalar variants, narrowly.
  - Strong inference `[SI]`: the animation's axes are not the block frame's axes (the identity mapping the code's `+0x364/0x368/0x36c` order suggests is the worst; a y↔z swap is better), consistent with `0x9b0d60` building its basis from a rotated frame; not tested directly.
  - Not done: posing a creature; the capability implication table; the `0x9c31e0` pre-pass of the finaliser (may alter blocks before the extents are computed); L/R convention; the 0x80-mode (`+0x434`) scale.
- Discriminating test (next): decode the creature frame (`0x9b0210` / `0x9b05f0`) to fix the axis mapping, then pose one creature and check that the feet stay within reach of their chains.
- Follow-ups: INV-032d axis frame + pose one walk; INV-033 skin generator; Phase 5 roadmap.

---

## INV-032d: the evaluator's exact axis rule, tested; the finaliser's re-rooting pre-pass found (retarget units still unconfirmed)

- Status: partial — evaluator scaling rule decoded and tested; data do not confirm it; creature not posed
- Confidence: observed (decompiled code, 918-creature test); explanation open
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: `0x9b0c50`, `0x9b0210`, `0x9b05f0`, `0x9c31e0` (called first by `0x9c4950`)
- Evidence: decompilations (REA/Ghidra, local); `tools/retarget_test.py --exact` (now reads each foot channel's word0); results `ledger/inv032d-retarget-code-rule.json`.
- Findings:
  - Observed — `0x9b0c50(mode, word0, csd_holder, block_holder, vec)` is the scaling for channels that are **not** body-bound (channel word0 bit 1 clear): POS mode `0x40` -> `x*ext_x*s`, `y*ext_y*s`, and `z*ext_z*s` **unless channel word0 bit `0x10` is set (z left unscaled)**, with `ext = (csd+0x364, +0x368, +0x36c)` (identity axis order) and `s` = instance scale; mode `0x80` -> all components `x size*s`, where size = `csd+0x374` for blocks with the `spin` bit (`+0x154 & 8`), else the block's own `+0x434`. Body-bound channels (`0x9b05f0` path, word0 bit 1) instead build a look-at basis (`0x9b0210`: forward = target − origin, up = +z, forward on axis 0, up on axis 2) and scale each basis axis by `0x9b0b80` (extents dotted with that axis).
  - Observed — result of applying the exact non-bound rule to the foot POS tracks (flag `0x71`, 918 creatures × 11 walks, exact leg length from INV-032c): CV with the rule **0.311** (norm) / 0.354 (horizontal) versus **0.320** unscaled — **no improvement**. The best variant stays the uniform mean-extent scalar (0.258, INV-032c).
  - Observed — `0x9c31e0` (called at the start of the finaliser `0x9c4950`) is a large **re-rooting/reordering pass**: it chooses a root block (the `root`-capability block, else the spin block nearest the origin, else heuristics on spin-block children), reverses the parent links along the path to the old root (swapping `+0x344`/`+0x350` joint vectors on the way), re-sorts the entries topologically (`0x9c3070`), rebuilds parent/child index tables (`+0x1fc`, `+0x204/+0x208`, `+0x214/+0x218`), fills per-block cumulative lengths at `+0x36c` (parent's value + 1.0-ish × distance to the parent centre; root = 3.0), and stores per-entry bind matrices (0x30 bytes) at `csd+0x3f8`. INV-032b/c's entries do **not** include this pass; the extents (order-independent) are unaffected, the chain lengths are affected only where a foot chain crosses the re-rooted path `[SI]`.
  - Strong inference `[SI]`: the data and the rule disagree because (a) stride is not proportional to leg-chain length across body plans (quadrupeds/multi-legged creatures use different stride patterns), (b) the re-rooted chain lengths and `+0x434` differ from my reproduction, or (c) a walk's excursion is limited by the animation, not scaled by size. Not discriminated.
  - Not done: the `0x9c31e0` re-rooting reproduction; posing; `+0x434` (block length) derivation; body-bound channels.
- Discriminating test: reproduce `0x9c31e0` and `0x9b5e60`/`0x9b5b20` (bind matrices) well enough to build the final skeleton, then pose one biped walk and compare the foot path with the creature's foot reach. Cost is high relative to progress; low priority versus the skin generator.
- Follow-ups: INV-033 skin mesh generator (higher visible value); Phase 5 roadmap; INV-032e retarget only if posing is required.

---

## INV-033: Milestone C — first interactive native viewer of real creatures (assembled parts; proxy spine skin; NOT the game's skin generator)

- Status: delivered as a working vertical slice; the skin surface, textures, rig and animation are not reconstructed
- Confidence: observed (offscreen renders inspected; window launch smoke-tested)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: `tools/export_creatures.py` (assembly per INV-020/021 → `spore_work/viewer/scene.bin`, local, git-ignored); `viewer/` (SwiftPM, Metal, AppKit)
- Evidence: `viewer/Sources/SporeView/main.swift`; snapshots `spore_work/viewer/snap/v*.png` (local); 400 creatures exported in ~1 minute (178 MB scene, 12k-vertex typical creature).
- Findings:
  - Observed — the viewer opens a 1280x800 window (Retina-aware drawable), Metal pipeline compiled at runtime, orbit/zoom/next-previous/wireframe/auto-rotate controls, and an offscreen `--snapshot` mode. Real creatureData-assembled part meshes appear in the correct layout in the creature's own skin colour (`skin_colors[0]`).
  - Observed — a **proxy** skin (tapered 16-sided tubes along Vertebra parent→child links, radius `0.1 × scale × max bbox extent`) joins the spine; it is a stand-in and does not reflect the game's surface generator (INV-030).
  - Not done: the game's skin surface (generator not located), paint textures (the `02424655/6/7` brush textures, INV-021/029), rig + animation playback (INV-025..032 give decoders and partial retargeting), plants, lighting parity with the game, audio, gameplay.
  - **Scope statement:** this is Milestone C (first reconstructed creature on the new pipeline), not a playable game. The retail Spore itself still needs Windows (or a compatibility layer) to run.
- Discriminating test: side-by-side with the game's creature thumbnails (INV-030 method) once the real skin generator exists.
- Follow-ups: skin generator (REA: `cEditorSkinMeshBase` users); animation playback in the viewer using the decoded ANIM channels; texture the proxy with paint brushes; Phase 5 roadmap.

---

## INV-034: walk-animation preview in the viewer from decoded ANIM channels (approximate)

- Status: delivered as an approximate preview; correctness of axes/units/limb assignment NOT established (INV-032 open)
- Confidence: observed (frames differ between cycle phases in offscreen renders); fidelity unknown
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: `tools/export_creatures.py` (`pick_walk`, `bake`), `viewer/Sources/SporeView/main.swift` (scene format `SPC2`, per-block matrices in the vertex shader)
- Evidence: renders at cycle 0 / 0.25 / 0.5 / 0.75 of creature 4 (`spore_work/viewer/snap/w_*.png`, local); the walk chosen is the first `*walk*` ANIM with decodable root/head/L foot/R foot POS+ROT channels.
- Findings:
  - Observed — each creature gets 60 baked frames: root POS/ROT (and `creatureFrm` POS) move/rotate every block about the creature centre; head ROT rotates blocks tagged `mout`/`eye`/`ear` about their own position; foot POS moves blocks with the foot flag (`0x20`), left/right chosen by the sign of block x (the mirror axis, INV-020). POS values are scaled by the INV-032 extents (identity axis mapping; z unscaled when channel word0 bit `0x10`).
  - Observed — the motion is visible but subtle (leg tips shift between phases).
  - Not established — the POS units and axes (INV-032b–d inconclusive), the L/R convention, which blocks the head channel drives, limb IK, body-bound channels, blending; this is **not** the game's animation system.
- Discriminating test: compare against the game's own walk (needs the original running, or the evaluator reproduced: `0x9b0d60`).
- Follow-ups: reproduce the evaluator and `0x9c31e0` to get a faithful pose; textures; the real skin generator.

---

## INV-035: the original game (GOG 3.1.0.29, EP1/Galactic Adventures build) launched on macOS under CrossOver

- Status: confirmed — the game reaches its main menu on this Mac
- Confidence: observed (live window captured: Spore logo, galaxy, Play / Create / Share / Galactic Adventures menu)
- Claimed by: Angus Gastle + Claude Code session — 2026-10-23
- Target: `SporebinEP1/SporeApp.exe` (user's own GOG purchase, extracted to `spore_work/gog-3.1.0.29`), CrossOver 26.3.0 (Homebrew cask `crossover`, installed this session; 14-day trial), bottle `spore` (win10_64)
- Evidence: `tools/launch_spore_crossover.sh`; Wine `+file`/`+reg` traces (`/tmp`, local); window captures (local, not committed).
- Findings:
  - Observed — the Homebrew `wine-stable`/`wine@devel` casks are disabled (fail Gatekeeper); no override was applied. CrossOver is signed/notarised and installed normally.
  - Observed — start-up failures, in order: `Configuration Script Failed [2000]` (registry keys absent: the extracted tree never ran the installer) → `Could not find the default preferences… [1004]` (keys wrong: the installer writes `appdir` = the folder name `Spore`, not a path, plus `installloc`, `datadir`, `playerdir`="My Spore Creations", `locale`, and per-expansion `spore_EP1`/`Creepy and Cute Parts Pack` `DataDir`s). The exact values come from `goggame-1948823323.script` (`setRegistry` actions), which also installs `__support/add/login.prop` to `%APPDATA%\Spore\Preferences\login.prop` (skips the EA registration screen).
  - Observed — with those values the game reads `Data/Config/*`, mounts packages and shows the main menu. Rendering fills only the upper-left ~78% of the window (a HiDPI scaling quirk of this bottle, not investigated).
  - Not done — gameplay stages, saves, audio check, performance, resolution/scaling fix, a clean second-launch test of the script, and any comparison against the native reconstruction.
  - Rights/security — nothing was bypassed; the game files are the user's own and stay local; CrossOver is commercial trial software (`brew uninstall --cask crossover` removes it).
- Follow-ups: use the running game as the behavioural reference (screenshots/traces) for INV-032 animation and the skin generator; fix window scaling; Phase 5 roadmap.
