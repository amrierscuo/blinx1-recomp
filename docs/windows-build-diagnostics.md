# Windows linker diagnostics

If the Windows build stops at `ld.lld: error: undefined symbol` for a generated body, keep
the failed build folder. The symbol in the error and the fingerprint of the
executable used for that build are enough to start the investigation.

## Identify the build and disc

In PowerShell, open the source checkout (the wizard puts it in the build
folder's `src` directory). With the current CLI, run:

```powershell
.\blinx1.cmd fingerprint .\game_files\default.xbe
```

For older releases without `fingerprint`, get the SHA-256 directly:

```powershell
(Get-FileHash -LiteralPath '.\game_files\default.xbe' -Algorithm SHA256).Hash.ToLowerInvariant()
```

Record the wizard release or, for a Git checkout, the output of `git rev-parse HEAD`.
The supported NTSC-U v6 fingerprint is in [game.toml](../game.toml). At
revision `67387b8`, the PAL v3 executable listed in the
[PAL symbol map](../config/releases/pal-v3.toml) reports both `code` and
`another release: PAL, version 3`. The first describes code section differences
from the reference. The second identifies a different release. The map
supplies addresses for named symbols in that PAL executable. The project's
[supported release](../README.md) is NTSC-U v6.

## Trace the unresolved symbol

[Issue #1](https://github.com/apexJCL/blinx1-recomp/issues/1) reported three
unresolved bodies with wizard v2026.11.1. For checkouts using
`src/recomp/gen/recomp_names.h`, the same wrapper can request a different
body on each disc:

| Wrapper name | NTSC-U v6 body | PAL v3 body |
| --- | --- | --- |
| `gfx_sphere_visible` | `sub_0009AD80_gen` | `sub_00034F90_gen` |
| `gfx_lens_flare` | `sub_00096BB0_gen` | `sub_00030B90_gen` |
| `pad_poll_frame` | `sub_000A2DE0_gen` | `sub_0009EBC0_gen` |

If recompilation itself reports `is not a function in this XBE's analysis`,
keep that diagnostic and the address it gives. This check can stop the build
before it produces C files or runs the linker.

For example, inspect the selected name and search the generated C files.
Replace the example body with the exact symbol from the failing link:

```powershell
Select-String -LiteralPath '.\src\recomp\gen\recomp_names.h' -SimpleMatch 'gfx_sphere_visible'
Get-ChildItem -LiteralPath '.\src\recomp\gen' -Filter '*.c' -File |
    Select-String -SimpleMatch 'sub_00034F90_gen'
```

- If the error uses the NTSC address while the current header selects the PAL
  one, check which source checkout, generated files and object directory the
  failed build used. Older releases can use literal addresses in
  [recomp_manual.c](../src/recomp_manual.c).
- If there is no function **definition** for the requested body, inspect the
  analysis and recompilation logs for that disc. A declaration or a call can
  also appear in the search results. A definition has a body in braces.
- If the definition exists, use its C filename to locate its object in the
  failed build and inspect the final link command or its response file.
  [CMakeLists.txt](../CMakeLists.txt) collects the generated `.c` files under
  `src/recomp/gen/`.

In the [bug report](https://github.com/apexJCL/blinx1-recomp/issues/new?template=bug_report.yml),
include the fingerprint, build version, exact missing symbol and failing build
log named under `src/build-logs/`. Review personal paths before posting.
Keep the XBE, generated game code, saves and built executable on your machine.
