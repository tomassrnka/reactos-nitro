# reactos-nitro changelog

reactos-nitro is an unofficial patch set on top of upstream ReactOS. It is not an official ReactOS release and is not affiliated with the ReactOS project. Newest entry first. All times UTC.

## reactos-nitro 2026.10-4

**Tagged `nitro-2026.10-4`** (2026-10-24) at fork `main` [`be9d31b1b9a`](https://github.com/tomassrnka/reactos/commit/be9d31b1b9a4dcdc3123026d15a3bab5c44c35ea). The app-compatibility release: **Claude Desktop (Electron/Chromium) now runs, renders, and connects to the live service on reactos-nitro**, on the same upstream base as 2026.10-2/-3, plus Firefox 32+64 side-by-side rendering and a kernel security/stability batch.

| Item | Value |
|---|---|
| Release tag | `nitro-2026.10-4` → [`be9d31b1b9a`](https://github.com/tomassrnka/reactos/commit/be9d31b1b9a4dcdc3123026d15a3bab5c44c35ea) |
| ReactOS base | upstream [`f53b1d15601`](https://github.com/reactos/reactos/commit/f53b1d1560120c529763b18963fdee68f39118af) — unchanged from 2026.10-2/-3 |
| Patch set | Cumulative over-base patch `nitro-2026.10-4-over-f53b1d15601.cumulative.patch` (verified: apply on `f53b1d15601` → tree == REL_SHA, diff empty). 3232 commits (2440 non-merge) over upstream, same base as 2026.10-2/-3. Direct-main cut (no replayed topic stack this release). |
| Artifacts | amd64-with-WoW64 ISO, i386 ISO, source tarball, patch (sha256 below) |

| Artifact | Size | sha256 |
|---|---|---|
| reactos-nitro-2026.10-4-amd64-wow64-Release.iso | 550502400 | `dd47c83e75cedaf198c08a55bbea4fcaf2b4b45a5ce41bb1d25e88fa40363d36` |
| reactos-nitro-2026.10-4-i386-Release.iso | 500170752 | `3cf90c74d81509a715bec45aec7d6333218b536ed1955c781c3b62f3886b6aa7` |
| reactos-nitro-2026.10-4-src.tar.gz | 113864152 | `72d72c0a369966113224773e65df1cf58401e4534a082ffd1be64aa8509cc928` |
| nitro-2026.10-4-over-f53b1d15601.cumulative.patch | 44891396 | `14c141906570d7cb54731b4fb5e72e422cabc56ddddc0414c28fec045eb386e4` |

### Highlights
**Claude Desktop runs and renders on reactos-nitro** — the modern Electron/Chromium app class is unlocked. Stock Electron 37.6.0 / Chrome 138 installs (via the extracted app payload), launches, creates its GUI window, and paints its UI through Chromium's SkiaRenderer/GPU compositor over ANGLE→SwiftShader (pure-CPU). Reaching a painted window required a chain of Windows-correctness fixes across win32k, user32, ntdll and the Object Manager — each an app-agnostic multiplier that benefits the whole Electron/Chromium (and GL/D3D) class, not just Claude. Claude Desktop also **connects to the live claude.ai** — the login UI loads, Cloudflare challenge passes, HTTP/2 — over the guest's standard networking (Chromium's own TLS stack; no new OS crypto code required). Also in this release: **Firefox 158** UI + content sandbox (level 1), and **Firefox-ESR 32-bit and 64-bit rendering content side-by-side on a single amd64+WoW64 system** (T104: the WoW64 thread-handle regression that blanked the 32-bit broker is fixed), and a kernel **security batch** (a tcpip type-confusion/UAF regression fix and FsRtl change-notify use-after-free fixes). Gate: 4-config builds rc=0, both ISOs boot to desktop, reproducibility verified (rebased stack tree == main), T0 + targeted suites green. **Correction (post-release):** the full regression suite completed after this tag and found that one change in this release — the `ObDuplicateObject` handle-access change (T100) — is Windows-incorrect; see Limitations.

### What reactos-nitro adds on top of ReactOS
- **Modern Electron/Chromium rendering (Claude Desktop):** `GetDC(NULL)` screen DC now associates with the desktop window (ANGLE EGL display bring-up); Win8 pointer-input `user32` exports (Chromium delay-load); app-manifest OS-version honored in `GetVersion`/`RtlGetVersion` per-process (Crashpad handler spawn); `advapi32` `BuildSecurityDescriptorW/A` implemented + unprivileged mandatory-label set (Crashpad named-pipe SD); `ntdll` `RtlGetUnloadEventTraceEx` implemented + `RTL_UNLOAD_EVENT_TRACE` pointer-width (crashpad dump capture). (An `ObDuplicateObject` change for Chromium read-only shared-memory also shipped but is superseded — see Limitations.) Per-app config (SwiftShader ICD + `no3d`) ships with the Claude app. See the render recipe in the harness docs.
- **32-bit COM / MSXML (WoW64):** 32-bit `msxml3` shipped + registered in SysWOW64 so x86 `CoCreateInstance(DOMDocument30)` works on amd64 — unblocks a broad class of 32-bit installers/apps.
- **Firefox 158**: Win10 per-app identity + Win10 API surface + content-sandbox level 1 (64-bit UI renders). WoW64 Firefox-ESR 32-bit + 64-bit both launch with browser chrome + navigation; side-by-side *content* rendering is limited (broker/sandbox) and tracked under T86.
- **Security hardening:** tcpip `SET_INFORMATION_EX` entity type-check (fixes a non-admin kernel type-confusion/UAF regression); FsRtl change-notify cancel/cleanup races, lost-wakeup and a reuse-UAF closed; GDI `CreateDIBSection` size-math overflow guards. (The `ObDuplicateObject` handle-access change originally listed here is **withdrawn as a security fix** — it was over-broad and Windows-incorrect; see Limitations.)
- **Stability:** process-quota block-switch race (prevents a QUOTA_UNDERFLOW bugcheck); ramdisk AddDevice flag-mask fix; OOM test-hook stripped from Release kernels.

## Bug fixes

<details><summary>App compatibility — Electron/Chromium (Claude Desktop) (8 fixes)</summary>

- WIN32SS/NTUSER: `IntWindowFromDC` returns the desktop window for the windowless screen DC so ANGLE's `eglGetPlatformDisplay(GetDC(NULL))` succeeds — T96 ([437c7de7424](https://github.com/tomassrnka/reactos/commit/437c7de7424))
- WIN32SS/USER32: export Win8 pointer-input APIs (`GetPointerType`/`GetPointerDevice`/`GetPointerPenInfo`/`RegisterPointerDeviceNotifications`) so Chromium delay-load resolves — T100 ([44ce99c2dfd](https://github.com/tomassrnka/reactos/commit/44ce99c2dfd))
- NDK/RTL: `RTL_UNLOAD_EVENT_TRACE.SizeOfImage` pointer-wide (x64) so crashpad records the unloaded-module list — T100 ([64cea71224c](https://github.com/tomassrnka/reactos/commit/64cea71224c))
- NTOS:OB: `ObDuplicateObject` change for Chromium read-only shared-memory — ⚠️ **SUPERSEDED/INCORRECT:** this blanket-blocked user-mode handle-access widening, which is Windows-incorrect (Windows re-evaluates the object's security descriptor and grants when the DACL allows). It over-broadly broke legitimate `DuplicateHandle` elevation and is being corrected next release — T100 ([77bb7949b8d](https://github.com/tomassrnka/reactos/commit/77bb7949b8d)). See Limitations.
- KERNEL32/NTDLL: honor the app-manifest compatibility version in `GetVersion`/`RtlGetVersion` (per-process) so Crashpad's handler spawn succeeds — T94 ([58a90fb603b](https://github.com/tomassrnka/reactos/commit/58a90fb603b))
- ADVAPI32/NTOS:OB: implement `BuildSecurityDescriptorW/A` + allow an unprivileged process to set its own mandatory label, fixing Crashpad's named-pipe security descriptor — T90 ([c092b8f5038](https://github.com/tomassrnka/reactos/commit/c092b8f5038))
- NTDLL/RTL: implement `RtlGetUnloadEventTraceEx` (was a stub returning success without writing its out-pointers → AV in Vista+ callers) — T97 ([d315f811604](https://github.com/tomassrnka/reactos/commit/d315f811604))
- SYSWOW64/SYSSETUP: ship + register 32-bit `msxml3`/`libxslt` so x86 `CoCreateInstance(DOMDocument30)` works on amd64 — T93 ([0c5d91b1606](https://github.com/tomassrnka/reactos/commit/0c5d91b1606))
</details>

<details><summary>Firefox 158 / WoW64 (4 fixes)</summary>

- WOW64: `NtCreateThread` thunk clamps the returned guest handle's access to what the thread object grants (via `NtQueryObject`) instead of requesting `THREAD_ALL_ACCESS` verbatim — fixes WoW64 `CreateThread` failing (the thread generic mapping grants `0x1f0fff`, not `0x1fffff`, so the over-request was rejected), which had blanked 32-bit Firefox's Chromium broker init under WoW64. Enables FF32+FF64 side-by-side content render — T104 ([cd739717eaf](https://github.com/tomassrnka/reactos/commit/cd739717eaf))
- Shim-engine PAGE_READWRITE fix + Firefox 116+ per-app Win10 identity + `QueryUnbiasedInterruptTimePrecise`/`GetSystemCpuSetInformation`/`CM_Register_Notification` ([03f77e1dc84](https://github.com/tomassrnka/reactos/commit/03f77e1dc84))
- KERNEL32: validate+accept `CHILD_PROCESS_POLICY`/`COMPONENT_FILTER`/`JOB_LIST` proc-thread attributes ([41be7cf42c0](https://github.com/tomassrnka/reactos/commit/41be7cf42c0))
- NTOS:MM: `NtReadVirtualMemory` Windows partial-copy semantics (`STATUS_PARTIAL_COPY`) ([9b08b829477](https://github.com/tomassrnka/reactos/commit/9b08b829477))
</details>

<details><summary>Filesystems — FsRtl / ntfsng change-notify (3 fixes)</summary>

- ntfsng: change-notify torn-read race across rename (over-read into watcher output) + drop 64KB/watch alloc — T87 ([efdb9109733](https://github.com/tomassrnka/reactos/commit/efdb9109733))
- NTOS:FSRTL: change-notify cancel/cleanup unlink race + completed-IRP requeue + report-size cluster (bugcheck 0x139 negative control) — T88 ([0715ca13dce](https://github.com/tomassrnka/reactos/commit/0715ca13dce))
- NTOS:FSRTL: change-notify lost-wakeup + reuse-edge use-after-free — T98 ([424e479fc37](https://github.com/tomassrnka/reactos/commit/424e479fc37))
</details>

<details><summary>Networking (1 fix, security)</summary>

- tcpip: `SET/QUERY_INFORMATION_EX` ADDRESS_OBJECT/CONNECTION entity type-check + live reference — fixes a non-admin kernel type-confusion/pool-corruption/UAF regression — T89 ([34fb956b444](https://github.com/tomassrnka/reactos/commit/34fb956b444))
</details>

<details><summary>win32k GDI (1 fix)</summary>

- `DIB_CreateDIBSection` size-math made overflow-safe (64-bit pitch, checked alloc, reject bad dimensions) — T62 ([7016ac5f357](https://github.com/tomassrnka/reactos/commit/7016ac5f357))
</details>

<details><summary>Kernel — Ps / Mm (2 fixes)</summary>

- NTOS:PS: quota block-switch race vs lockless charges (epoch-quiesce; prevents a QUOTA_UNDERFLOW 0x21 bugcheck, Release-affecting) — T57 ([f83de5692f5](https://github.com/tomassrnka/reactos/commit/f83de5692f5))
- NTOS:MM: compile the `MmDbgTestOomRetryBound` test hook out of Release kernels (release-prep) — T61 ([d1b1077cba6](https://github.com/tomassrnka/reactos/commit/d1b1077cba6))
</details>

<details><summary>Drivers & toolchain (2 fixes)</summary>

- ramdisk: `RamdiskAddDevice` clears only `DO_DEVICE_INITIALIZING` (was wiping `DO_DIRECT_IO`/`DO_POWER_PAGABLE`) — T75 ([0e439e7d3e7](https://github.com/tomassrnka/reactos/commit/0e439e7d3e7))
- ADVAPI32: link at DLL 0x502 (guard the `RtlAddMandatoryAce` Vista+ dependency) — T95 ([51a81438119](https://github.com/tomassrnka/reactos/commit/51a81438119))
</details>

### Limitations
- **Claude render requires the per-app config** (SwiftShader ICD + `no3d` + ANGLE launch flags) that ships with the Claude app packaging; it is pure-CPU rendering (no GPU acceleration).
- **Firefox 32+64 side-by-side content render is verified via `file://`**, exercising the full broker → sandbox-target → content-process → renderer → JS pipeline on both browsers simultaneously. End-to-end `http` page fetch inside the test VMs is a current harness/network-profile limitation (not a product gap in the WoW64 broker fix) — live web browsing depends on the same guest networking that the `-5` runtime/crypto work addresses.
- .NET Framework 4.5 enablement is in progress (MSXML/32-bit-COM installer wall cleared this release; the CLR itself is not yet supported — tracked for `-5`).
- ⚠️ **Known regression — `ObDuplicateObject` (T100):** this release's `ObDuplicateObject` change blanket-blocks user-mode `DuplicateHandle` requests that widen access beyond the source handle. That is **Windows-incorrect** — Windows re-evaluates the object's security descriptor and grants the wider access when the DACL permits. As a result, legitimate DACL-granted handle-access elevation is broken in `-4` (surfaced by `NtOpenProcess`/token/handle-duplication tests, including Windows-validated ones). Headline apps (Claude Desktop, Firefox) are unaffected. The correct fix — re-evaluating the security descriptor Windows-style, while still preventing write-elevation of Chromium's restrictive-SD read-only sections — lands in `-5`. (Originally mis-described here as an elevation-of-privilege *fix*; it is not.)

### Changes since reactos-nitro 2026.10-3
Same upstream base. This release adds the Claude Desktop / Electron render chain (win32k/user32/ntdll/Ob), 32-bit MSXML COM, Firefox 158 + sandbox-L1, and a kernel security/stability batch (tcpip type-confusion regression, FsRtl notify UAFs, quota-switch bugcheck, GDI size-math, ramdisk flags; note the `ObDuplicateObject` change is superseded — see Limitations). Shipped as a cumulative over-base patch (3225 commits / 2436 non-merge over f53b1d15601) rather than a replayed topic stack (this window landed direct-to-main).

## reactos-nitro 2026.10-3

**Tagged `nitro-2026.10-3`** (2026-10-22) at fork `main` [`92f4586f849`](https://github.com/tomassrnka/reactos/commit/92f4586f849d3e5bdc2538ff1e9374136b865721). A large security, stability and SMP point release on the same upstream base as 2026.10-2.

| Item | Value |
|---|---|
| Release tag | `nitro-2026.10-3` → [`92f4586f849`](https://github.com/tomassrnka/reactos/commit/92f4586f849d3e5bdc2538ff1e9374136b865721) |
| ReactOS base | upstream [`f53b1d15601`](https://github.com/reactos/reactos/commit/f53b1d1560120c529763b18963fdee68f39118af) (sync #3, 2026-10-20) — unchanged from 2026.10-2 |
| Patch series | 47 topic patches on `f53b1d15601` (`main-rebased-20261022` → `9eeefc87b93`); applied tree `63dc72d7479` (== main) |
| Artifacts | amd64-with-WoW64 ISO, i386 ISO, source tarball, patch series (sha256 below) |

| Artifact | sha256 |
|---|---|
| reactos-nitro-2026.10-3-amd64-wow64.iso | `98f19a091d271e7cd5689c162dd120fe94b3d7fffd6b5583659f8cf6839a572c` |
| reactos-nitro-2026.10-3-i386.iso | `afe05a0ed86c0fb965c165ef88a1a2f0a4e356c16a4c8a1ad4713a664d3fe65f` |
| reactos-nitro-2026.10-3-src.tar.xz | `5b82c65a880ac4744ed20ddba56346c35cee41055dd84c93f2ba1da4b0d7d29b` |
| reactos-nitro-2026.10-3-patch-series.tar.xz | `e6e46c5c02aeff4122bd0d4c85ab180bc06b34d74621d544033d77eb610f69f1` |

### Highlights
A broad security + stability + SMP release on the same base as 2026.10-2. Headline areas: a deep kernel/win32k security sweep (verified attack-surface seams in win32k GDI/USER capture, kernel streaming, AFD, mountmgr, keyboard/mouse class, dbgk/kd64, WoW64), non-admin TCP/UDP sockets, and two SMP throughput levers (per-CPU object-handle cache and per-CPU nonpaged pool). Gate: 4-config builds rc=0, both ISOs boot to desktop, reproducibility verified (rebased stack tree == main), and the T1 regression suite showed only known-flaky entries (0 attributable to this release).

### What reactos-nitro adds on top of ReactOS
- **Security sweep (Astra#2/#3 verified seams):** win32k GDI/USER user-pointer capture and size-math (`ExtEscape`, `CreateDIBSection` ppvBits, clipboard CF_DIB, menu/atom/icon/hook handlers); kernel-streaming user-buffer capture; AFD use-after-free / enum-events overflow / METHOD_NEITHER arbitrary-write / transport-address leak; mountmgr IOCTL validation; keyboard/mouse class pending-IRP handling; dbgk fault-under-mutex + kd64 physical R/W; WoW64 residual cross-process queries.
- **Non-admin networking (T66):** standard users can open `\Device\Tcp` and create/bind/listen/connect TCP plus UDP; privileged control (ARP/route/IP-address writes) stays admin-gated behind an explicit access check.
- **SMP:** per-CPU object-handle FirstFree cache (ABA-safe, kill-switch); per-CPU nonpaged pool descriptors; building on 2026.10-2's pool scaling.
- **Stability/robustness:** ramdisk I/O-range OOB guard; process-quota underflow and limited-quota fixes; bounded OOM fault retry; device query-remove/eject; a real-hardware FreeLdr video-mode fallback (boots where a passed-through/real VBIOS disagrees with INT 10h).

## Bug fixes

<details><summary>Kernel: Ke, HAL and amd64 (2 fixes)</summary>

- NTOS:KE/x64: do not copy undeclared exception parameters to user mode ([c2855982a39](https://github.com/tomassrnka/reactos/commit/c2855982a39))
- SDK/RTL: do not leave the user pointer in `Dest` when `ProbeAndCaptureUnicodeString` fails to allocate ([fa8a0c7a7db](https://github.com/tomassrnka/reactos/commit/fa8a0c7a7db))
</details>

<details><summary>Kernel: Mm and Cc (10 fixes)</summary>

- NTOS:MM: bound the `MmAccessFault` OOM retry taken under a held lock, so a low-memory fault can no longer spin forever ([fix/mm-mdl-oom-bounded](https://github.com/tomassrnka/reactos/commit/8267776f1825dcc46e362795c100c969176e34bf))
- NTOS:MM: enforce a copy-on-write-aware maximum protection for image views ([6c3f58ed7ed](https://github.com/tomassrnka/reactos/commit/6c3f58ed7ed))
- NTOS:MM: enforce the section maximum protection in `MmProtectSectionView` ([ef493c744eb](https://github.com/tomassrnka/reactos/commit/ef493c744eb))
- NTOS:MM: do not trust the user buffer header in `MemorySectionName` ([efc948abe40](https://github.com/tomassrnka/reactos/commit/efc948abe40))
- NTOS:MM: preserve both boundary pages in `MmZeroSegmentRange` (`CcZeroData`) ([059d9068f2f](https://github.com/tomassrnka/reactos/commit/059d9068f2f))
- NTOS:EX/MM: true per-processor nonpaged small-block pool lookaside lists ([8cbd43795e2](https://github.com/tomassrnka/reactos/commit/8cbd43795e2))
- NTOS:MM: per-descriptor nonpaged pool spinlock + descriptor array (slice 1/4) ([fc189f7bc18](https://github.com/tomassrnka/reactos/commit/fc189f7bc18))
- NTOS:MM: wire nonpaged pool per-descriptor selection + header `PoolIndex` (slices 2+3/4) ([a1d5be35d5b](https://github.com/tomassrnka/reactos/commit/a1d5be35d5b))
- NTOS:MM: activate per-CPU nonpaged pool descriptors (slice 4/4) ([aab06648164](https://github.com/tomassrnka/reactos/commit/aab06648164))
- NTOS:EX/MM: restore lookaside depth adjustment, make the page-walk check DBG-only ([c01821e7e04](https://github.com/tomassrnka/reactos/commit/c01821e7e04))
</details>

<details><summary>Kernel: Ob, Se, Ps, Ex and registry (14 fixes)</summary>

- NTOS:OB/EX: per-CPU object-handle `FirstFree` cache, ABA-safe, behind a kill-switch (SMP lever) ([fix/handle-percpu-cache](https://github.com/tomassrnka/reactos/commit/44b34bde1cb4eff496f71f35c14a76aaf9aaa3be))
- NTOS:PS: fix a process-quota underflow on a self-block switch ([fix/ps-quota-selfblock-underflow](https://github.com/tomassrnka/reactos/commit/bfe5e8b940bc3e9fb33f72ea5072272e9499f951))
- NTOS:OB: remove now-unused `Total` locals (fix i386 Debug `-Werror`) ([d461572b669](https://github.com/tomassrnka/reactos/commit/d461572b669))
- NTOS:OB: per-processor cache-line-padded object-type counters ([475b1853035](https://github.com/tomassrnka/reactos/commit/475b1853035))
- NTOS:PS: do not drop the process reference on quota-block allocation failure ([6f285d389ab](https://github.com/tomassrnka/reactos/commit/6f285d389ab))
- NTOS:PS: honor requested pool limits in `PspSetQuotaLimits` ([9b8fd94d56e](https://github.com/tomassrnka/reactos/commit/9b8fd94d56e))
- NTOS:PS: use the matching return threshold for paged and nonpaged pool quota ([e78f377052c](https://github.com/tomassrnka/reactos/commit/e78f377052c))
- NTOS:PS: fix a wild write and uninitialized quota block in the limited-quota path ([a5d8172e23c](https://github.com/tomassrnka/reactos/commit/a5d8172e23c))
- NTOS:PS: charge and return default-block pool quota without `PspQuotaLock` ([cc478a6962d](https://github.com/tomassrnka/reactos/commit/cc478a6962d))
- NTOS:EX: do not trust the output buffer in `SystemProcessInformation` ([a3a0524b2be](https://github.com/tomassrnka/reactos/commit/a3a0524b2be))
- NTOS:PS: reject a cross-process thread in `NtGetNextThread` ([a72b2cb8ee1](https://github.com/tomassrnka/reactos/commit/a72b2cb8ee1))
- NTOS:PS: probe the user TLS slot in `ThreadZeroTlsCell` ([32a31cbba77](https://github.com/tomassrnka/reactos/commit/32a31cbba77))
- NTOS:dbgk/kd64: fix a `SetInformation` fault taken under a mutex + the kd64 physical read/write command ([fix/kd64-dbgk-capture](https://github.com/tomassrnka/reactos/commit/8be3bf4b6a7e761bbccac47b98d614e769ca5b15))
- SDK:CMLIB: reject undersized/unaligned hive cells before linking them (registry) ([cfe545a521e](https://github.com/tomassrnka/reactos/commit/cfe545a521e))
</details>

<details><summary>I/O manager and file-system libraries (3 fixes)</summary>

- NTOS:IO: serialize PnP property-change events with the queue lock ([745246dd841](https://github.com/tomassrnka/reactos/commit/745246dd841))
- MSFS: probe and capture `ReadTimeout` in `FileMailslotSetInformation` ([214bd796cb1](https://github.com/tomassrnka/reactos/commit/214bd796cb1))
- NPFS: fix an output overflow and a PEPROCESS leak in `FSCTL_PIPE_QUERY_CLIENT_PROCESS` ([8bab9c5b36d](https://github.com/tomassrnka/reactos/commit/8bab9c5b36d))
</details>

<details><summary>win32k, GDI and USER (8 fixes)</summary>

- WIN32K:GDI: user-pointer capture and size-math hardening (Astra#2) ([fix/win32k-gdi-usercapture](https://github.com/tomassrnka/reactos/commit/0bc42fa3afce25319b05be838a47bc1766c856e0))
- WIN32K:GDI: `NtGdiCreateDIBSection` capture + `ppvBits` arbitrary-write fix ([fix/win32k-dibsection-capture](https://github.com/tomassrnka/reactos/commit/90f6d75990b47cc67c54afa317cba03e5f766a3b))
- WIN32K: clipboard CF_DIB size check + `FamInfo` NULL-init ([fix/win32k-clipboard-faminfo](https://github.com/tomassrnka/reactos/commit/8112aa606f4c6a9476b58a1f4f80277180ec7db1))
- WIN32K:USER: `NtUserGetAtomName` double-fetch ([fix/useratom-doublefetch](https://github.com/tomassrnka/reactos/commit/499b8487bd7602c03690c80226a8a18b137b69ca))
- WIN32K:USER: `NtUserGetIconInfo` probe + leak; `SetWindowsHookEx` `ModuleName.Length` OOB ([fix/ntuser-geticon-hook](https://github.com/tomassrnka/reactos/commit/0938fcc82440a5196104ae973002605a76567ec4))
- WIN32K:USER: `IntSetMenuInfo` kernel-pointer type-confusion EoP ([fix/menu-typeconfusion](https://github.com/tomassrnka/reactos/commit/b9ac8118a69d10b584966de921c9b6bfd3a1bfae))
- WIN32SS:NTUSER: `NtUserSetWindowWord` OOB write when `cbwndExtra < 2` (unsigned underflow) ([0aeb445daec](https://github.com/tomassrnka/reactos/commit/0aeb445daec))
- WIN32SS:ENG: reject an overflowing bitmap size in `SURFACE_AllocSurface` ([258179096ed](https://github.com/tomassrnka/reactos/commit/258179096ed))
</details>

<details><summary>WoW64 (2 fixes)</summary>

- WoW64: amd64 fibers + `NtQuerySystemInformationEx` + wow64 test crashes ([fix/wow64-fiber-threadexit](https://github.com/tomassrnka/reactos/commit/cfcc82290cb59c031c40a09be468391b493a88e8))
- WoW64: residual cross-process queries — `ProcessImageInformation`, `ThreadDescriptorTableEntry` ReturnLength, amd64 `Win32StartAddress` ([fix/wow64-residual-queries](https://github.com/tomassrnka/reactos/commit/ae88a070738a704844a15edb3c01a1e4a29a13cd))
</details>

<details><summary>Networking (4 fixes)</summary>

- AFD: window-size SET_INFO use-after-free + enum-events overflow + METHOD_NEITHER arbitrary-write + transport-address leak ([fix/afd-cluster](https://github.com/tomassrnka/reactos/commit/09c310fbc8bc2e5292dccada6adc1490c157c1ad))
- AFD: `AfdAccept` length + requestor-mode checks, `LockBuffers` probe, `AfdCreateSocket` EA over-read ([fix/afd-input-validation](https://github.com/tomassrnka/reactos/commit/52c66667ac3776306e261fc7400151ea69a30e98))
- TCP/IP: non-admin TCP/UDP via an Everyone ACE + an explicit control-access gate for privileged requests ([fix/tcp-nonadmin-dacl](https://github.com/tomassrnka/reactos/commit/6d1cf318108fddd8f11a6162b304294f98e0b86f))
- Network: time-bound the receive DPC so one pass cannot hold DISPATCH_LEVEL for seconds ([47b12cdea20](https://github.com/tomassrnka/reactos/commit/47b12cdea20))
</details>

<details><summary>Audio and multimedia (1 fix)</summary>

- KS (kernel streaming): user-buffer capture — method/property/event double-fetch + type-confusion ([fix/ks-userbuffer-capture](https://github.com/tomassrnka/reactos/commit/53adfb2fadadf0e9d1a67a77c736cff5b01034a6))
</details>

<details><summary>Storage, USB, PnP and drivers (4 fixes)</summary>

- mountmgr: IOCTL input validation ([fix/mountmgr-p1-ioctl-validation](https://github.com/tomassrnka/reactos/commit/701bb72d90524a2d0fa546f12dafebcf7284f22c))
- ramdisk: `RamdiskReadWrite` OOB guard + NULL-MDL / uninit-relations / self-deadlock / loader-block / PnP-IRP fixes ([fix/ramdisk-native-bundle](https://github.com/tomassrnka/reactos/commit/91a039cb373ff4190693db851f468d2a2b99cf93))
- kbdclass/mouclass: a second read no longer orphans a pending IRP ([fix/kbdmou-pendingirp](https://github.com/tomassrnka/reactos/commit/3732b730437d1958aaf6c6b0e58f1844c24cb73a))
- PnP: device query-remove / eject / disable / uninstall, with correct veto (`CR_REMOVE_VETOED`) ([unit/queryremove](https://github.com/tomassrnka/reactos/commit/34d861c96fb6a1aa29c749a08543c1e495a95611))
</details>

<details><summary>Setup and boot (1 fix)</summary>

- FreeLdr: do not assert on a BDA-vs-INT-10h video-mode mismatch, so it boots on real hardware / a passed-through GPU VBIOS ([fix/freeldr-pcvideo-bda](https://github.com/tomassrnka/reactos/commit/743b3ae0f9b2453c0d9dc55e34a8ed8375c3185c))
</details>

<details><summary>Browser, COM and Wine imports (1 fix)</summary>

- MSHTML: do not re-set an unchanged link media, fixing an IE hang and runaway memory on `loadCSS` pages ([b8fc159f5ca](https://github.com/tomassrnka/reactos/commit/b8fc159f5ca))
</details>

<details><summary>Tests only (6 fixes)</summary>

- kmtest: fix a `PsQuotaLimitedTrim` double-eval that tripped a teardown ASSERT (test-only) ([fix/quota-trim-bugcheck](https://github.com/tomassrnka/reactos/commit/1f283a97d74648cfcddf38729052a53897f56073))
- KMTESTS: `PsQuotaLimited` tests for private quota blocks ([a80df7f7dc4](https://github.com/tomassrnka/reactos/commit/a80df7f7dc4))
- MSFS_APITEST: `FileMailslotSetInformation` contract test ([4d9acd1d170](https://github.com/tomassrnka/reactos/commit/4d9acd1d170))
- NPFS_APITEST: `FSCTL_PIPE_QUERY_CLIENT_PROCESS` contract test ([e85e6df6324](https://github.com/tomassrnka/reactos/commit/e85e6df6324))
- NTDLL_APITEST: `SystemProcessInformation` image-name and fault tests ([ee26d6920f0](https://github.com/tomassrnka/reactos/commit/ee26d6920f0))
- NTDLL_APITEST: `NtQueryVirtualMemory` `MemorySectionName` tests ([9a261367d2a](https://github.com/tomassrnka/reactos/commit/9a261367d2a))
</details>

### Limitations
- 32-bit Firefox renders blank at the default sandbox (WoW64 sandbox limitation).
- APIC HAL: single I/O-APIC, active-high; MADT polarity overrides not honored (real-hardware risk; QEMU-tolerated).
- `accept()` returns `WSAEACCES` for all users — no inbound TCP servers yet; outbound/client TCP works.
- Cross-process `WriteProcessMemory`/`VirtualProtectEx` into a remote process's image data sections fails — a side-effect of the W^X image-protection hardening in this release (breaks sandboxed Chromium/Electron children and code injectors; the hardening is kept, the remote-write path is fixed in a follow-up).
- OS identity is `5.2.3790` (Server 2003); applications that require Windows 7 (e.g. current Firefox ESR) will refuse to run.

### Changes since the previous Nitro
On the same upstream base as 2026.10-2 (no new upstream sync). 56 changes on `main`'s first-parent spine since the `nitro-2026.10-2` tag: 22 reviewed fix-branch merges plus 34 direct commits — a kernel/win32k security sweep, SMP pool and handle-cache work, quota and Mm hardening, and storage/networking fixes (grouped above).

## reactos-nitro 2026.10-2

**Tagged `nitro-2026.10-2`** (2026-10-20) at fork `main` [`054e4af6a88`](https://github.com/tomassrnka/reactos/commit/054e4af6a88f283b3e149103ded67ac211562ba2). A security + upstream-sync point release on top of 2026.10-1.

| Item | Value |
|---|---|
| Release tag | `nitro-2026.10-2` → [`054e4af6a88`](https://github.com/tomassrnka/reactos/commit/054e4af6a88f283b3e149103ded67ac211562ba2) |
| ReactOS base | upstream [`f53b1d15601`](https://github.com/reactos/reactos/commit/f53b1d1560120c529763b18963fdee68f39118af) (sync #3, 2026-10-19) |
| Patch series | 52 cumulative topic patches on `f53b1d15601` (2026.10-1's 48 + the 4 new this release, listed under Bug fixes); applied tree `e0cf06866f46` (== main) |
| Artifacts | amd64-with-WoW64 ISO, i386 ISO, source tarball, patch series |

### Highlights
4 security fixes plus upstream sync #3 (Wine-10.0 DCOMP/TBS/WEVTAPI/BCP47LANGS import + a basic IShellItem2). Gate 8/8, zero T1 regressions over the identical base.

### What reactos-nitro adds on top of ReactOS
#### Upstream sync (#3)
Merged ReactOS master up to `f53b1d15601`: Wine-10.0 import of DCOMP, TBS, WEVTAPI and BCP47LANGS, plus a basic IShellItem2. The fork kernel is kept byte-identical to the previous release (one upstream KE change set aside to preserve the fork's cross-processor lock handling).

## Bug fixes

<details><summary>Kernel: Ke, HAL and amd64 (2 fixes)</summary>

- NTOS:KE amd64: mask user-supplied MXCSR to the processor-supported bits before load, so a reserved-bit thread context can no longer GP-fault the kernel on `NtContinue`/`SetThreadContext` return-to-user ([fix/amd64-mxcsr-sanitize](https://github.com/tomassrnka/reactos/commit/5c8a2ac7e054fea52e646251c8416c38be9a695b))
- NTOS:KE/EX: bounds-check the kernel profiling object — reject the exclusive endpoint and compute the bucket count as SIZE_T, preventing an out-of-bounds counter write ([fix/ke-profile-bounds](https://github.com/tomassrnka/reactos/commit/77dbdcff4eba391b844a8359caeb8db3cf62baad))
</details>

<details><summary>Kernel: Ob, Se, Ps, Ex and registry (1 fix)</summary>

- NTOS:SE/OB: return a mandatory-label SACL on process and thread objects, as Windows does; also fixes an object-manager use-after-free in the insert-cleanup path and removes a stopgap that failed every label query ([fix/se-process-thread-integrity-label](https://github.com/tomassrnka/reactos/commit/1b6aa10b6e09d59b2d2789ea0f173898da88f0c0))
</details>

<details><summary>win32k, GDI and USER (1 fix)</summary>

- WIN32K:GDI: initialize the `ExtEscape` result so the failure paths cannot return uninitialized kernel stack bytes to user mode ([fix/gdi-extescape-result-init](https://github.com/tomassrnka/reactos/commit/4748c129fd8b3e69f8317cba6ac7e89a41c7dfef))
</details>

### Limitations
Unchanged from 2026.10-1 — including that 32-bit Firefox under WoW64 renders only with its content sandbox disabled.

### Changes since the previous Nitro
This is a point release: the section above lists only what's **new in 2026.10-2** (upstream sync #3 + 4 security fixes). **2026.10-2 includes everything from 2026.10-1** — the full ~554 fixes and all feature tracks are in the [2026.10-1 section below](#reactos-nitro-20261). Gate 8/8; T1 (154 shards) zero batch-introduced regressions.

## reactos-nitro 2026.10-1

**Tagged `nitro-2026.10-1` (private) on 2026-10-19**, annotated tag at fork `main` [`c203b29276c`](https://github.com/tomassrnka/reactos/commit/c203b29276c5e7d4fc54177d8d999c54d5cb2a45). Gated green (both ISOs boot-tested; TC 32/64, WoW64-GUI, Firefox 64-bit, NTFS 4h soak, licence-clean all PASS). **Not yet public** — publication is held pending the FF32-under-WoW64 render fix (see Known limitations), a trademark check of the "reactos-nitro" name, and the NTFS driver licence-statement clean-up.

| Item | Value |
|---|---|
| Release tag | `nitro-2026.10-1` → [`c203b29276c`](https://github.com/tomassrnka/reactos/commit/c203b29276c5e7d4fc54177d8d999c54d5cb2a45) (private fork) |
| ReactOS base | upstream [`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e30438dedbc1a2216db8ca8682f5fc871f) "[DISKPART] Sort the volumes list", committed 2026-10-18 11:12 UTC |
| Patch series | 48 topic patches on `ac1ab9e3043`, series head [`926768aba66`](https://github.com/tomassrnka/reactos/commit/926768aba66); applied tree byte-identical to the tag (`94e948f52aad`) |
| Artifacts | amd64-with-WoW64 ISO (444,596,224 B), i386 ISO (391,118,848 B), source tarball, per-topic patch series — rebuilt from the series at release time (see RELEASE-PROCESS) |

### Highlights

547 fix branches and several feature stacks, 2,316 non-merge commits on top of upstream ReactOS `ac1ab9e3043`.

- Run 32-bit Windows programs on the 64-bit build (WoW64); Total Commander and Firefox ESR, each as 32-bit and 64-bit side by side, are the proof. Upstream has no WoW64.
- A new NTFS driver, built on the Linux kernel's `fs/ntfs` with our own journal and power-cut recovery; NTFS is the default file system.
- Btrfs: WinBtrfs 1.10 (upstream ReactOS ships 1.8.1).
- Multi-processor support on x86 and amd64, installed by default; upstream's amd64 kernel does not start extra processors.
- Smooth window dragging on AMD hosts (shadow framebuffer, USB polling and pointer fixes).
- Faster and sturdier networking: a reworked receive path and 57 networking fixes.
- Setup and profiles: setup preselects NTFS, upstream's C:\Users layout with Windows-style legacy folder links, working profile deletion.
- Memory manager and cache hardening, including commit charging with a commit limit.
- Desktop and tools: LCD text rendering, bundled open fonts, crash reports with a Windows 7-style dialog, Disk Management.
- Not yet: the Firefox sandbox starts on 64-bit but does not restrict anything (see Limitations).

### What reactos-nitro adds on top of ReactOS

For each area: what upstream ReactOS has at the base commit, then what reactos-nitro adds.

#### WoW64: 32-bit programs on 64-bit ReactOS

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- Not supported: there is no `dll/wow64`, and process creation reports "WOW64 is not supported yet" ([CORE-17655](https://jira.reactos.org/browse/CORE-17655)).

**reactos-nitro 2026.10-1:**
- 32-bit Windows programs run on the 64-bit build (amd64 only; on in the WoW64 ISO).
- SysWOW64, system-call and win32k thunks, file-system and registry redirection, and 32-bit consoles, windows, threads and process creation.
- Total Commander 11.58 and Firefox ESR 128, each 32-bit and 64-bit, run side by side in one session.

#### NTFS

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- The original ReactOS NTFS driver, with install support and write enabled by default on a normal install. It is still labelled experimental (the enabling registry value is named `MyDataDoesNotMatterSoEnableExperimentalWriteSupportForEveryNTFSVolume`), partial (several operations still return STATUS_NOT_IMPLEMENTED), and has no journaling or power-cut recovery.

**reactos-nitro 2026.10-1:**
- `ntfsng`, a read/write NTFS driver built on the Linux kernel's `fs/ntfs` ([Linux v7.3-rc6](https://github.com/tomassrnka/reactos/commit/77c12ef7d52bc291a0243ef5aeac1fa79c8b69cd), imported unmodified; GPL-2.0-or-later except two MIT files), running through a compatibility layer.
- Journaling with replay after a power cut, and a boot-time consistency check.
- NTFS is the default system file system, and setup preselects it.

#### Btrfs

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- WinBtrfs 1.8.1.

**reactos-nitro 2026.10-1:**
- [WinBtrfs 1.10](https://github.com/maharmstone/btrfs#changelog), which brings in 1.8.2, 1.9 and 1.10:
-   - the block group tree from Linux 6.1;
-   - writing free-space bitmaps and compressed inline files;
-   - better handling of removable devices and of out-of-memory/out-of-space errors;
-   - fixes for crashes when moving files across subvolumes or creating many files at once;
-   - fixes for snapshot extent-tree corruption, a hard-link deadlock and a hang under heavy load.
- Our own fixes on top ([`fix/btrfs-sync-1.10-main`](https://github.com/tomassrnka/reactos/commit/54e35c388a2617297197655bdbae650d6e0092f5)): volumes from current btrfs-progs mount, `format /FS:BTRFS` works, and drive letters stick.

#### Display

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- The VBE/Bochs `framebuf` driver draws straight into video memory.

**reactos-nitro 2026.10-1:**
- A shadow framebuffer: drawing goes to a cached surface and only changed areas are copied to video memory, which makes window dragging smooth and cheap.
- USB UHCI polling and software-pointer fixes, so pointer movement doesn't flicker.

#### SMP and kernel

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- amd64: the processor start-up routines (`KeStartAllProcessors`, `HalStartNextProcessor`) are stubs, so amd64 runs on one processor.
- i386: an MP kernel and HAL are built, but setup selects the single-processor kernel.

**reactos-nitro 2026.10-1:**
- Multi-processor kernels on x86 and amd64; setup installs the MP kernel and HAL on ACPI machines.
- Tested from 1 to 16 vCPUs, with long soaks on AMD and Intel hosts.
- Much faster cross-processor thread hand-off and wake-up.
- A fix for the root cause of an Intel-only STOP 0x7E.
- Fixes for x87/FPU state handling, cross-processor TLB flushes (batched and targeted), and amd64 interrupt, NMI and exception delivery.
- 32 kernel fixes in the list below.

#### Networking

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- The lwIP-based TCP/IP driver and AFD.

**reactos-nitro 2026.10-1:**
- A reworked receive path: lwIP core lock, in-order TCP receive per adapter, fewer thread hops and reassembly locking.
- 57 networking fixes: AFD connect, accept and cancel paths, TCP/IP, Winsock and iphlpapi.

#### Setup and user profiles

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- NTFS install support (formatter and boot sector) and the C:\Users profile layout.

**reactos-nitro 2026.10-1:**
- The graphical setup preselects NTFS.
- The C:\Users layout, with fixes for shell calls that crash in upstream's stubs, and a read-only Downloads folder.
- Windows-style legacy links (`Documents and Settings`, `Default`, `ProgramData`).
- `DeleteProfile` works and refuses to delete a loaded profile. Upstream's version cannot succeed.

#### Disk Management

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- Only the `diskmgmt.msc` console file; there is no Disk Management implementation.

**reactos-nitro 2026.10-1:**
- Disk Management (`diskmgmt.msc`) as a standalone application with a Windows-like disk list and graphical layout view. It isn't built on ReactOS's in-progress MMC; porting it to MMC is planned.
- Covers basic, extended and GPT disks, SAS disks and optical drives.
- Disk operations:
  - initialize a disk (MBR or GPT);
  - create, delete and format volumes;
  - shrink and extend volumes in place;
  - change drive letters and mount points.
- Windows 7-style dialogs and wizards.
- Write safety: changes are planned against the disk layout the user saw, and are re-checked before writing.
- `SetVolumeMountPointW` and `DeleteVolumeMountPointW` implemented in kernel32.

#### Internet Explorer and the web stack

**ReactOS master ([`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043)):**
- Wine-based `iexplore` with mshtml/urlmon/wininet. Wine Gecko is not provided on amd64.

**reactos-nitro 2026.10-1:**
- Wine Gecko is provided on amd64, and `iexplore` offers to install Gecko when it's missing.
- File downloads work: download dialogs and engine in ieframe, `DoFileDownload` from shdocvw, and content the page can't show is handed to the download path.
- Fixes:
  - urlmon tasks queued during a read are now run;
  - wininet fails stalled reads after the timeout and reports damaged gzip bodies like Windows;
  - raw HTTP headers are read as ANSI;
  - saving an Internet shortcut whose properties are open works.

#### Other

**reactos-nitro 2026.10-1:**
- ARM3 commit charging with a commit limit, so over-commit fails with STATUS_COMMITMENT_LIMIT, plus special-pool debugging.
- LCD text rendering and bundled open fonts.
- Crash dumps, Application Error event 1000 and a Windows 7-style crash dialog.
- Wine syncs (Media Foundation, kernel32 profile, setupapi) and upstream sync #2.

### Fixed upstream ReactOS JIRA reports

These reactos-nitro fixes address bugs reported in the [ReactOS JIRA](https://jira.reactos.org). They're listed here to make porting easier. The list includes only reports that clearly describe the same bug. Some reports were already closed upstream, for example as *Won't Fix*. Fixes with possible but unconfirmed matches are not listed.

| Upstream report | Summary (status at lookup) | Fixed in reactos-nitro by |
|---|---|---|
| [CORE-8717](https://jira.reactos.org/browse/CORE-8717) | ConvertStringSidToSid succeeds for empty string and other invalid SIDs / Reopened / Unresolved | [fix/advapi32-convert-string-sid](https://github.com/tomassrnka/reactos/commit/9bb9005936e7f4f25b4672c3eeaf7e4200f30a2c) |
| [CORE-12498](https://jira.reactos.org/browse/CORE-12498) | telnet is looking for telnet.ini which is not distributed with ReactOS; Reopened, no resolution | [fix/telnet-install-config](https://github.com/tomassrnka/reactos/commit/b5e19db1e584c47709cc4a7b25b1e65b1ead7206) |
| [CORE-12604](https://jira.reactos.org/browse/CORE-12604) | ReactOS executes extensionless files in PATH / Open / Unresolved | [fix/cmd-search-extensionless](https://github.com/tomassrnka/reactos/commit/a2cf87a2385c45912696d67519e596a4174dca56) |
| [CORE-14795](https://jira.reactos.org/browse/CORE-14795) | CMD ECHO - incorrect text output to file / Untriaged / Unresolved | [fix/cmd-redirect-ansi](https://github.com/tomassrnka/reactos/commit/25a3be3914f44d06b7225f2e21cdac91bb22e5a6) |
| [CORE-15934](https://jira.reactos.org/browse/CORE-15934) | REGRESSION: Resizing a themed window causes a classic frame to be drawn temporarily and makes the caption blink (Resolved/Fixed) | [fix/win32k-framechanged-screen-region](https://github.com/tomassrnka/reactos/commit/65fdcb4306f4ab6641b71bf3e03df3dffcd2a6bd) |
| [CORE-16061](https://jira.reactos.org/browse/CORE-16061) | Vista Paint and Minesweeper cannot locate RegisterApplicationRestart in KERNEL32.DLL - Resolved/Postponed | [fix/kernel32-application-restart](https://github.com/tomassrnka/reactos/commit/632cc8034617b0b37c1363e672e32208f73efb34) |
| [CORE-16273](https://jira.reactos.org/browse/CORE-16273) | IniFileMapping is not implemented - Untriaged | [fix/kernel32-profile-sync](https://github.com/tomassrnka/reactos/commit/9b644b68fd6cd4cde766f50f9efd43dd874e9690) |
| [CORE-16444](https://jira.reactos.org/browse/CORE-16444) | Shell Find Fails on some Folders using Shell Extensions (RECYCLED or fonts) (Open) | [fix/shellfind-font-folder-results-v2](https://github.com/tomassrnka/reactos/commit/c56e3d5307a30e75516230022181244aa92c9468) |
| [CORE-17317](https://jira.reactos.org/browse/CORE-17317) | [Kernel32] Implement missing I64 flag in format_message / Untriaged / Unresolved | [fix/kernel32-formatmessage-int64](https://github.com/tomassrnka/reactos/commit/a96335568d79b05b0ee8277326c64ab540f7d2ac) |
| [CORE-18390](https://jira.reactos.org/browse/CORE-18390) | SPI_GETWHEELSCROLLCHARS incorrectly reported as invalid - Resolved/Postponed | [fix/win32k-wheel-scroll-chars](https://github.com/tomassrnka/reactos/commit/1e51c969ad44e733c335feea6ef3a0cd9b95120b) |
| [CORE-18789](https://jira.reactos.org/browse/CORE-18789) | FILE_APPEND_DATA not handled correctly in NtCreateFile & friends; Untriaged/None | [fix/append-write](https://github.com/tomassrnka/reactos/commit/678833d1e2cd9a26c1b099354419977cf393455e) |
| [CORE-19707](https://jira.reactos.org/browse/CORE-19707) | Inno Setup 6 based installers do not work / Untriaged / Unresolved | [fix/ntdll-actctx-loadfrom](https://github.com/tomassrnka/reactos/commit/53e2c8a0ee496debe89db83ff5ae52e794d8a368) |
| [CORE-19855](https://jira.reactos.org/browse/CORE-19855) | Shortcut's 'Start in' field treated incorrectly - Open | [fix/shell32-exec-context-menu-directory](https://github.com/tomassrnka/reactos/commit/8d089d7697b4e47b9b0937297caf9c76e75397f4) |
| [CORE-20067](https://jira.reactos.org/browse/CORE-20067) | [MAGNIFIER] Help button should be grayed out (Resolved, Won't Fix) | [fix/help-unavailable-disabled](https://github.com/tomassrnka/reactos/commit/5a03092962cdd75111e756a8604d5c4da784d3ec) |
| [CORE-20129](https://jira.reactos.org/browse/CORE-20129) | Broken file operations when using folder starting by '...' - Open | [fix/shlwapi-path-dotdot-prefix](https://github.com/tomassrnka/reactos/commit/2264a530e0fc1ef84956ed1c9f945407adbb4670) |
| [CORE-20687](https://jira.reactos.org/browse/CORE-20687) | [COMCTL32] Bugs when setting the title of property sheets or wizards - Untriaged | [fix/comctl32-propsheet-title](https://github.com/tomassrnka/reactos/commit/d99b70541d359a58f062d3fac4840812991db9c6) |
| [CORE-20756](https://jira.reactos.org/browse/CORE-20756) | Notepad > Save as 'All files' adds the extension anyway - Untriaged | [fix/comdlg32-defext-keep-extension](https://github.com/tomassrnka/reactos/commit/a14b21ad5a83a3be5ef1e0b441a10b1ebb6343a2) |

## Bug fixes

554 fixes: one line for each of the 547 fix branches merged into Nitro main and each of 7 small fixes committed directly on main, grouped by area (click an area to open it). The wording comes from the fix's own commit message; security-related fixes are described only generically. Each branch name links to its last commit on GitHub, and a ReactOS JIRA ticket is named where we found a clear match.

<details><summary>Kernel: Ke, HAL and amd64 (32 fixes)</summary>

- HAL: Make the DMA lock exclude other threads ([fix/hal-dma-32bit-bounce](https://github.com/tomassrnka/reactos/commit/10bd01a90bd42528682d7451609ff5ebb1e42eb5))
- HALX86: Refuse ports outside the PCI configuration range in the AMLI handler ([fix/hal-amli-pci-handler](https://github.com/tomassrnka/reactos/commit/c75f7702b8e0e45d98753652c250de2cee294a3d))
- HALX86: Send the APC interrupt EOI before enabling interrupts ([fix/hal-apc-eoi](https://github.com/tomassrnka/reactos/commit/239f92ea5cc37d6c58a49d779482f80a130c9bb3))
- HALX86: Count lost RTC periods in the APIC HAL clock interrupt ([fix/hal-clock-lost-ticks](https://github.com/tomassrnka/reactos/commit/f7a2c6672ae31354db510517a45fd6e10a201b90))
- HALX86: Reset the map register index for each queued channel grant ([fix/hal-dma-queued-channel](https://github.com/tomassrnka/reactos/commit/f6989394c945c684ed0d92c4db210d1af3b8a8f3))
- HALX86: Write a redirection entry's destination before unmasking it ([fix/hal-ioapic-destination-order](https://github.com/tomassrnka/reactos/commit/2c1e0e009e5e1db870445b4ae820b2aefeba1092))
- HALX86: Do not read the I/O APIC when deferring an interrupt ([fix/hal-ioapic-vector-atomic](https://github.com/tomassrnka/reactos/commit/e0321c602a08173391a14aeb539e51ca8caa765b))
- HALX86: Take the CMOS lock to acknowledge the RTC clock interrupt ([fix/hal-rtc-ack-lock](https://github.com/tomassrnka/reactos/commit/8f6ce562564491926c2f65ee60224ea7e4a0081c))
- HALX86: Read RTC register C after enabling the clock interrupt ([fix/hal-rtc-register-c](https://github.com/tomassrnka/reactos/commit/f170a6d4208a954456463e901130db12648db1de))
- HALX86: Do not reset the TSC during calibration ([fix/hal-tsc-no-reset](https://github.com/tomassrnka/reactos/commit/1e0ba781a47ffc3c5be81ca8435b550b1148e336))
- HALX86:APIC: Lower the lazy IRQL with interrupts disabled ([fix/hal-apic-lazy-irql-lower](https://github.com/tomassrnka/reactos/commit/8e53fe18c4c6e656a25bdb13c95de384041b6c0d))
- HALX86:APIC: fix a security-related bug ([fix/hal-apic-profile-trap](https://github.com/tomassrnka/reactos/commit/dc551055776e7dd297683dce8e9e0f5f0d295a32))
- NTOS:EX: Report a keyed event pairing to the side whose wait just timed out ([fix/ke-keyed-event-timeout-race-v2](https://github.com/tomassrnka/reactos/commit/d1e415ade66f7d20f31aa0efe45289e82c847b1c))
- NTOS:EX: Return the cycle time of each process in SystemProcessInformation ([fix/ntos-cpu-usage-info-v2](https://github.com/tomassrnka/reactos/commit/07c6e88531a0cf1337c30db1178c7d442fc10c77))
- NTOS:KE: Let one processor report an unclaimed NMI on i386 ([fix/amd64-nmi](https://github.com/tomassrnka/reactos/commit/778999c6b4582c46c6cdced532c253577b4cccb3))
- NTOS:KE: Return STATUS_NOT_IMPLEMENTED from NtSetLdtEntries on amd64 ([fix/amd64-setldtentries-stub](https://github.com/tomassrnka/reactos/commit/3efcaaa6e695173100d8f3d3ed66da55c28864ab))
- NTOS:KE: Set the Inserted state in KeInsertByKeyDeviceQueue ([fix/devqueue-insert-by-key](https://github.com/tomassrnka/reactos/commit/da9c28ce8cdb5046318a58678487b0e8056ed565))
- NTOS:KE: Deliver a user mode fast fail as a second chance exception ([fix/fastfail-second-chance](https://github.com/tomassrnka/reactos/commit/c15cf6d6ae9b294cb43d8baa6b9ff4531f529048))
- NTOS:KE: Read the thread's queue under the dispatcher lock in KeWaitForGate ([fix/ke-gate-wait-queue](https://github.com/tomassrnka/reactos/commit/8bd04dc388dc7686bb142d830e565ad35bd7ff94))
- NTOS:KE: Clear a processor from the idle summary when it gets a thread ([fix/ke-idle-summary](https://github.com/tomassrnka/reactos/commit/123d2b8c4874a5dc41f9b6dff1696d7c842d0635))
- NTOS:KE: Read the thread state once in KiSetPriorityThread ([fix/ke-setpriority-state](https://github.com/tomassrnka/reactos/commit/61be28cac6e6a307f0d9e7c6c51e84764680a1da))
- NTOS:KE: Place deferred ready threads before choosing the next thread in KiSwapThread ([fix/ke-swap-deferred-ready](https://github.com/tomassrnka/reactos/commit/1ac8c3ce0f019e673cc2478f227737f7712e815c))
- NTOS:KE/NDK: fix a security-related bug ([fix/amd64-syscall-17-args](https://github.com/tomassrnka/reactos/commit/03fbea61318d2ba9aee3220f4761a584ce980444))
- NTOS:KE/x64: Deliver a user-mode stack fault as an exception ([fix/amd64-stack-fault-v2](https://github.com/tomassrnka/reactos/commit/ba212840961bd6f404d0d9f56601ab84e3ac2cd9))
- NTOS:KE/x64: Enable interrupts in trap handlers that can wait ([fix/amd64-trap-interrupts-main](https://github.com/tomassrnka/reactos/commit/95fe0f3940edcdf560bdddcbb3ca20f30442b090))
- NTOS:KE/x64: Send an EOI for an unexpected interrupt only when it is in service ([fix/amd64-unexpected-int-eoi](https://github.com/tomassrnka/reactos/commit/6abb8f88e5a1cb053c4ebeea72a2515a845f6cfb))
- NTOS:KE/x64: amd64 kernel delivers STATUS_STACK_OVERFLOW to the program instead of killing it when its stack runs out ([fix/ke-amd64-dispatch-stack-overflow-retry](https://github.com/tomassrnka/reactos/commit/f5dad9688000af58cf9ebe7321eb72c656ed7853))
- NTOS:KE/x64: Handle __fastfail (INT 29h) ([fix/ke-amd64-fastfail](https://github.com/tomassrnka/reactos/commit/fe5c75384c984a9a190fbd785bd669b95e51dca7))
- NTOS:MM: Size the amd64 paged pool like the x86 one ([fix/amd64-paged-pool-size](https://github.com/tomassrnka/reactos/commit/5e10b13e133f998898b7762bf6da96a5af939345))
- PSEH: Preserve amd64 funclet call frames and run scope cleanup ([fix/amd64-pseh-flags](https://github.com/tomassrnka/reactos/commit/61f1d6da5c0d159442ca859e3d6a5ffc0d061d41))
- PSEH: Keep x64 GCC handlers of try blocks that never complete ([fix/amd64-pseh-noreturn-try](https://github.com/tomassrnka/reactos/commit/c040901ca922f7ccf573f84e5b0436a4d29ec400))
- SDK:ASM: Lay out the amd64 user-mode system call stubs as Windows does ([fix/amd64-ntdll-stub-layout](https://github.com/tomassrnka/reactos/commit/c08bfc06aa9c9bdd4f98ccb0c075a9a52997bdb6))

</details>

<details><summary>Kernel: Mm and Cc (48 fixes)</summary>

- NTDLL:LDR: Do not refuse a relocated EXE that has a start context ([fix/mm-image-high-base](https://github.com/tomassrnka/reactos/commit/f2f40deb153c4367b1182deafe172017e53e41ce))
- NTOS:CC: Reference the shared cache map while CcFlushCache uses it ([fix/cc-flush-reference](https://github.com/tomassrnka/reactos/commit/f299f728af2d63032c54535724cdbb3a292f2d47))
- NTOS:CC: Fix the VACB range test in CcPurgeCacheSection ([fix/cc-purge-overlap](https://github.com/tomassrnka/reactos/commit/c3d34d7a2811036a117647be7d459a986ebd35ad))
- NTOS:CC/NTOS:MM: CcZeroData keeps mapped data when a file grows, OOM-safe [pick] ([main/uf23-cc-zero](https://github.com/tomassrnka/reactos/commit/435ad17941d59f1a42a615a3f8e5afcb7bc9289a))
- NTOS:CM: Read the special pool tag from the registry ([fix/mm-special-pool-tag-registry](https://github.com/tomassrnka/reactos/commit/af197ebd6a60c2bcce5b589d35df6dc63108d1c2))
- NTOS:KE: Retire DPCs on the DPC stack in the x86 idle loop ([fix/mm-idle-dpc-stack](https://github.com/tomassrnka/reactos/commit/8b9008304f7bf4694611f395ea760a0da49af8cc))
- NTOS:MM: Compute the next PTE from the PXE when skipping an empty PXE ([fix/mm-amd64-empty-pxe](https://github.com/tomassrnka/reactos/commit/b4c7fdc1b4b28e73f40baf1425be824a3ce8ec7d))
- NTOS:MM: Free the amd64 hyperspace page tables of a deleted process ([fix/mm-amd64-hyperspace-leak](https://github.com/tomassrnka/reactos/commit/37179bd152b5643e0b6ce1afffa3eea7510c9d06))
- NTOS:MM: Drop the LRU walk's page reference when the trim wraps around ([fix/mm-balancer-lru-reference](https://github.com/tomassrnka/reactos/commit/0d8c5d4b050da907fb633eb5ddb591075ed88244))
- NTOS:MM: Retry for free pages when the cache copies file data ([fix/mm-cache-page-in-wait](https://github.com/tomassrnka/reactos/commit/f66f6259db6a9beeac6fe45362f5e68bdca10e01))
- NTOS:MM: Throttle page requests only when pages or the cache are under pressure ([fix/mm-consumer-throttle](https://github.com/tomassrnka/reactos/commit/96bd31303c8949ddf2488e61d35ddf9392b11ea9))
- NTOS:MM: Make the private copy of a copy-on-write page writable on SMP ([fix/mm-cow-smp-write](https://github.com/tomassrnka/reactos/commit/b346641a808bc71112ef7268ca8a4459dd79d7b4))
- NTOS:MM: Make a copy-on-write view page writable once it is private ([fix/mm-cow-view-region](https://github.com/tomassrnka/reactos/commit/5ca9a1f960edde695f9c485295511848ccb2503c))
- NTOS:MM: Free pages in transition when decommitting them ([fix/mm-decommit-transition](https://github.com/tomassrnka/reactos/commit/969a8da8d03eead29931d3d1647cac40b096d2cc))
- NTOS:MM: Flush the TB once per page-table run when deleting virtual addresses ([fix/mm-delete-va-tb-batch](https://github.com/tomassrnka/reactos/commit/e5a7a55f29eabf0e038c0c1b79a81b33e0856465))
- NTOS:MM: Refuse a data section view with nothing to map ([fix/mm-extend-section-rounding-v3](https://github.com/tomassrnka/reactos/commit/a19716f2f52a038694ea0b8495efa5bbe951657f))
- NTOS:MM: Wait for TB flushes without holding the PFN lock ([fix/mm-flush-outside-pfn-lock](https://github.com/tomassrnka/reactos/commit/d3b23c5fd84e248bc669d5bb01167b5e46140427))
- NTOS:MM: Write adjacent dirty pages of a file with one paging write ([fix/mm-flush-write-clusters](https://github.com/tomassrnka/reactos/commit/05d888150c56363b55e075f0113774abcdfa814b))
- NTOS:MM: Return STATUS_IMAGE_MACHINE_TYPE_MISMATCH for a foreign image view ([fix/mm-image-machine-mismatch](https://github.com/tomassrnka/reactos/commit/65bd363f23ce52e924fa9b6c2cb876ef2aec4fcf))
- NTOS:MM: fix a memory-safety bug ([fix/mm-image-unmap-uaf](https://github.com/tomassrnka/reactos/commit/b397b92d605d3b82c37a2cd2ccdba0a614e460ec))
- NTOS:MM: fix a security-related bug ([fix/mm-image-validation](https://github.com/tomassrnka/reactos/commit/cbc5dbd60675518a336f27cd07d61e0231b4eb2a))
- NTOS:MM: Tear down user mappings of locked MDLs at process exit ([fix/mm-mdl-view-exit](https://github.com/tomassrnka/reactos/commit/98fec594f131c454da9b42180ca2ff073329c5c3))
- NTOS:MM: Fail a paged pool expansion when too few pages are available ([fix/mm-pagedpool-expand](https://github.com/tomassrnka/reactos/commit/eb34142a6af68925547659c232a43de9894ea5d7))
- NTOS:MM: fix a security-related bug ([fix/mm-pagingfile-map-free](https://github.com/tomassrnka/reactos/commit/ab29b388276b4e1dd15a69909a44afe303b7a7fc))
- NTOS:MM: Fail a prototype PTE fault with STATUS_NO_MEMORY ([fix/mm-proto-fault-no-memory](https://github.com/tomassrnka/reactos/commit/21cb2fadbc338ef078cd984fc84bd13914f34b1e))
- NTOS:MM: Keep MEM_MAPPED for ARM3 section views in NtQueryVirtualMemory ([fix/mm-queryvm-section-view](https://github.com/tomassrnka/reactos/commit/69ddbf5f1130d357f33395cf039d94fa34d10be5))
- NTOS:MM: Map ordinary RAM cached: give its PFNs the MiCached attribute ([fix/mm-ram-cache-attribute-v2](https://github.com/tomassrnka/reactos/commit/43745bdaf7f5ce79dc35a04db58047925a9d62bd))
- NTOS:MM: Do not wait for the balancer while holding address space locks ([fix/mm-rebalance-wait-locks](https://github.com/tomassrnka/reactos/commit/2fa185eb5d883de51edee21cc92f1a305bd05213))
- NTOS:MM: Charge commitment once per committed page ([fix/mm-recommit-charge](https://github.com/tomassrnka/reactos/commit/bf7ae60bee7ac3f2b9347c4f2b6a5c35bb2fe5b3))
- NTOS:MM: Take the rmap entry of a section fault before mapping the page ([fix/mm-rmap-alloc-wait](https://github.com/tomassrnka/reactos/commit/b15617905a77e90d9b22ae3642dff7dcc370ac94))
- NTOS:MM: Return STATUS_SECTION_TOO_BIG for sections larger than they may be ([fix/mm-section-too-big](https://github.com/tomassrnka/reactos/commit/330e4bcdb6e88d3a66f4b12c2facd3a8f9e7baa7))
- NTOS:MM: Purge and flush a data section up to its highest page table ([fix/mm-segment-range](https://github.com/tomassrnka/reactos/commit/f6cfb1956dea647792246cb02bc023486b301c6c))
- NTOS:MM: Check for a single-entry import list before reading its count ([fix/mm-single-import](https://github.com/tomassrnka/reactos/commit/683e988267e16659bb4908f34be786aba8a889ab))
- NTOS:MM: fix a memory-safety bug ([fix/mm-stack-guarantee](https://github.com/tomassrnka/reactos/commit/488a6e8833112b240821f61152bae3dc20837c46))
- NTOS:MM: Forget the saved swap entry of a page whose write failed ([fix/mm-swap-bitmap](https://github.com/tomassrnka/reactos/commit/246e8084dfbe6b8195801745e4ff14da8a5f2e9d))
- NTOS:MM: Bring a locked page back from transition without unlinking it ([fix/mm-transition-original-pte](https://github.com/tomassrnka/reactos/commit/4ab6a69d9768d83e8a7522221b63d05d7762d028))
- NTOS:MM: Do not let a file be truncated under a mapped view ([fix/mm-truncate-mapped-view](https://github.com/tomassrnka/reactos/commit/8ebff34b520e7ba9f6a9b07efa45ac03cecfd629))
- NTOS:MM: Wait for paging file I/O on a view before freeing it ([fix/mm-unmap-wait-entry](https://github.com/tomassrnka/reactos/commit/9c62d2ecac20b50834d5ed396acb0e54155d4cc0))
- NTOS:MM: Flush decommitted and deleted user pages on the process's processors only ([fix/mm-user-tb-flush-targets](https://github.com/tomassrnka/reactos/commit/956de32e504b6aeaec26b298ab80a0f2c91b1577))
- NTOS:MM: Unlock a VirtualLock'd page made inaccessible ([fix/mm-virtuallock-delete](https://github.com/tomassrnka/reactos/commit/50531cdb0b1fe0ff933a79b4f673f0ea7d12a63f))
- NTOS:MM: Resolve a transition fault without requiring free pages ([fix/mm-zero-free-pages](https://github.com/tomassrnka/reactos/commit/02de0dbbf85cdd08dd1593515f286ffa37d9134e))
- NTOS:MM: Zero free pages through a cached mapping and flush them ([fix/mm-zero-pages-cached](https://github.com/tomassrnka/reactos/commit/c84aea2c9192bbcfa0350169a076123ac4385ec0))
- NTOS:MM: Do not put the active processors into a new process's PEB affinity mask ([fix/process-affinity-peb](https://github.com/tomassrnka/reactos/commit/22f27fd354716bc45e9aabf588c7b1fb667f5d59))
- NTOS:MM: snapshot PagesUsed in the consumer trim branch ([fix/uf22-balancer-trim-target-v2](https://github.com/tomassrnka/reactos/commit/7c2b8266c89e7232e51846874119d86810e615fa))
- NTOS:MM: Mark files with writable sections from a handle as user-mapped ([fix/uf23b-fatloss-evict-v2](https://github.com/tomassrnka/reactos/commit/b68629e9ff8e1b0e19cea74292c8ae42fe484d48))
- NTOS:MM: Do not wait for the balancer while holding AddressCreationLock [pick] ([main/mm-wait-address-lock](https://github.com/tomassrnka/reactos/commit/3e9b5634349286b1f99434efe37fbcff993d1bc9))
- ntoskrnl mm: Mm: retry a transient in-page read before bugchecking ([fix/mm-inpage-retry](https://github.com/tomassrnka/reactos/commit/a2aace7a77d1871a88c4eb10d706b29db7386e16))
- ntoskrnl mm: mm: balance segment reference on MiRosUnmapViewOfSection claim-loop early return ([fix/mm-section-segment-ref-leak](https://github.com/tomassrnka/reactos/commit/30c8a1035c11c23d4492ccc7bc9bebb8489227e4))

</details>

<details><summary>Kernel: Ob, Se, Ps, Ex and registry (38 fixes)</summary>

- CMLIB: fix a security-related bug ([fix/cmlib-amd64-recovery](https://github.com/tomassrnka/reactos/commit/f3c54e419c1251d97df9020f631c0b7abdbed64f))
- CMLIB: Mark every block of a cell dirty ([fix/cmlib-mark-cell-dirty](https://github.com/tomassrnka/reactos/commit/aa9b4df9e81a48029bbb469d3ad40adbc7f85dc8))
- NTOS:CM: fix a security-related bug ([fix/p2-ntoskrnl-misc](https://github.com/tomassrnka/reactos/commit/139849ec47082be3fe2ce9a0b61e3582879920c5))
- NTOS:CM/SDK:CMLIB: Report a failed hive flush ([fix/cm-flush-failure](https://github.com/tomassrnka/reactos/commit/9e9e08af954db07bba5b80e35fc35856f71e77ba))
- NTOS:EX: Restore the default timer resolution on the last release ([fix/ex-timer-resolution](https://github.com/tomassrnka/reactos/commit/0a194e77e4acf15e4d80ed5c99937838853d1510))
- NTOS:EX: fix a security-related bug ([fix/kernel-dst-transition](https://github.com/tomassrnka/reactos/commit/c2cb8b8769f57ae4e3e518db2ff829ea8a3752a5))
- NTOS:EX/NTOS:PS: Report the private commit charge in bytes ([fix/ntos-process-private-bytes](https://github.com/tomassrnka/reactos/commit/5ddcb3f4fc04e80b88203d39ab15dc1f0abee6ec))
- NTOS:FSRTL: Keep paged FILE_LOCK fields out of the range spin lock ([fix/fsrtl-filelock-internal-sync](https://github.com/tomassrnka/reactos/commit/a66dc6138fcde3898b3d77b37da6fb2ce7d60630))
- NTOS:FSRTL: Compare byte-range lock offsets as unsigned ([fix/fsrtl-filelock-signed-compare](https://github.com/tomassrnka/reactos/commit/c566feb82d45b8ea605804603d098c99a9615737))
- NTOS:FSRTL: Lock per-stream contexts by FCB header version, as Windows ([fix/fsrtl-teardown-ctx](https://github.com/tomassrnka/reactos/commit/f2a88140ef58d045c0b92efe837fef5b1051db14))
- NTOS:IO: Keep the end-of-file offset of append-only synchronous writes ([fix/append-write](https://github.com/tomassrnka/reactos/commit/678833d1e2cd9a26c1b099354419977cf393455e)) — fixes [CORE-18789](https://jira.reactos.org/browse/CORE-18789)
- NTOS:IO: Implement FileIoCompletionNotificationInformation ([fix/io-completion-notification-modes](https://github.com/tomassrnka/reactos/commit/8840b64162cbb0b927a94a5384f93eec1474fd51))
- NTOS:IO: Answer FileFsDeviceInformation without the file object lock ([fix/io-getfiletype-deadlock](https://github.com/tomassrnka/reactos/commit/26571b9c7009a3d3174682c897f4e250a1d7f317))
- NTOS:IO/NTOS:OB: Balance object lifetime in the cancel APC and unnamed handle count ([fix/p2-io-ob-lifetime](https://github.com/tomassrnka/reactos/commit/1813c6d6582c06cdc41461cf9e1ab74ae74f47db))
- NTOS:KD: Release the time refresh lock only when the time-slip work item got it ([fix/kd-timeslip-lock](https://github.com/tomassrnka/reactos/commit/cf7d7f60dcda27401aec55d20abaf42c48fe98e2))
- NTOS:KDBG: Keep the full stack limit across a debugger entry ([fix/kdbg-stack-limit](https://github.com/tomassrnka/reactos/commit/b3a6a9a7060cb2cca96a2b47670f1c659119400e))
- NTOS:LPC: fix a security-related bug ([fix/lpc-callback-validation](https://github.com/tomassrnka/reactos/commit/23a80a6f1048b62944fd93adb6c29a16a01227c7))
- NTOS:LPC: Free the reply message of an exiting thread ([fix/lpc-exit-thread](https://github.com/tomassrnka/reactos/commit/70c406400be146088d8d86dbc3c7766d59b7f013))
- NTOS:LPC: Match every reply-semaphore release with exactly one wait ([fix/lpc-reply-wait-window](https://github.com/tomassrnka/reactos/commit/fa6075cc9248e861523581dc3830a04c0e694e04))
- NTOS:MM: Flush the address of a trimmed working set entry ([fix/kernel-misc](https://github.com/tomassrnka/reactos/commit/42c2b80fdc279413e663583ae0da0e9cb7c23659))
- NTOS:OB: Initialise the handle returned by NtCreateDirectoryObject ([fix/ob-createdir-handle](https://github.com/tomassrnka/reactos/commit/308273b4c3ef1f759ca711b6484a084dc7a38ab9))
- NTOS:OB: Unlock the handle entry on a write reference type mismatch ([fix/ob-write-ref-unlock](https://github.com/tomassrnka/reactos/commit/d8edb96ddbbcf441ad67a894243e5924a43c3244))
- NTOS:PNP: Protect the user-mode PnP event queue with a lock ([fix/pnp-event-queue-lock](https://github.com/tomassrnka/reactos/commit/6b2cd8c63f14e3b38cb96a2c80840580e9b4abf5))
- NTOS:PO/KERNEL32: Implement power request objects ([fix/kernel32-power-requests](https://github.com/tomassrnka/reactos/commit/7b8f54832589c8d8b04d788452aab39b1e9fb700))
- NTOS:PS: Answer ThreadAmILastThread from the active thread count ([fix/exit-last-thread](https://github.com/tomassrnka/reactos/commit/15c4dc745c8ca6182ac4cae16f94b266cab1e43c))
- NTOS:PS: Grant PROCESS_QUERY_LIMITED_INFORMATION and accept it for the limited query classes ([fix/openprocess-limited-query-v2](https://github.com/tomassrnka/reactos/commit/b09cf3c11cbc6b250d4854068dc7084a96791630))
- NTOS:PS: Attach to the target process when writing another thread's impersonation TEB info ([fix/teb-impersonation-attach](https://github.com/tomassrnka/reactos/commit/ab8f9a2713f8753ceb792c13ee5816d4cca2cc38))
- NTOS:SE: fix a security-related bug ([fix/se-accesscheck-capture](https://github.com/tomassrnka/reactos/commit/6f5b0176e0fc03b040d38774ec9b26988e5d4ca4))
- NTOS:SE: Honour the auto-inheritance flags in SeAssignSecurityEx ([fix/se-auto-inherit](https://github.com/tomassrnka/reactos/commit/0b474e55aa48cb0b4a26756c4aca2c7522a591ed))
- NTOS:SE: Copy the restricting SIDs into a filtered token ([fix/se-restricted-sids-copy](https://github.com/tomassrnka/reactos/commit/fa3207cff016188f520adabe48a9e356296eafcd))
- NTOS:SE/NTOS:OB: fix a security-related bug ([fix/se-token-integrity-label](https://github.com/tomassrnka/reactos/commit/f6cde1a5ae669896581906f80884117b02e48f93))
- NTOS:SE/WOW64: fix a security-related bug ([fix/se-token-logon-sid](https://github.com/tomassrnka/reactos/commit/14b2bc93270c37c90a880637ffe1ad80306eae39))
- PS/CONSRV: fix a security-related bug ([fix/kernel-handle-list-inherit](https://github.com/tomassrnka/reactos/commit/57435902888c7551f885dd1af0103bb5396a2b54))
- SDK:CMLIB: Stop the dirty-block walks at the hive's last block ([fix/cmlib-dirty-walk-bound-v2](https://github.com/tomassrnka/reactos/commit/cbe683b1f081a3326bd3ff0f31eb84834b538bcb))
- SDK:CMLIB: Flush a hive's base block before writing its bins ([fix/cmlib-flush-base-block](https://github.com/tomassrnka/reactos/commit/cf19e20e3160e48198257906c8f6a923c8c47816))
- SDK:CMLIB: Refuse a hive with a cell of size 0 or past its bin ([fix/cmlib-zero-size-cell](https://github.com/tomassrnka/reactos/commit/8f93ed53ef28cade2f593ea029aea63879da3893))
- SDK:CMLIB/NTOS:CM: Serialize hive cell allocation and keep replaced block lists ([fix/cmlib-concurrent-hive-growth](https://github.com/tomassrnka/reactos/commit/b2b6bc5d6976cbec540b53d9b9b75bf3fc0ccab6))
- SDK:CMLIB/NTOS:CM: fix a security-related bug ([fix/cmlib-hive-log-rules](https://github.com/tomassrnka/reactos/commit/c6d3de9f435c65a3c7159da99403b3f09da0d646))

</details>

<details><summary>I/O manager and file-system libraries (2 fixes)</summary>

- HIDPARSER: Clear the description on failure; no report ID byte for absent reports ([fix/hidparse-description](https://github.com/tomassrnka/reactos/commit/1ba7d06b275deca11a9c23270a1726cf72e286b7))
- SPTILIB: Clear the output tail of the caller's IRP, not of the new one ([fix/sptilib-output-buffer-irp](https://github.com/tomassrnka/reactos/commit/e2ab8a90d8cfd81f8d58480e362b7e20d3a51bf4))

</details>

<details><summary>NTFS (ntfsng) (11 fixes)</summary>

- BOOTDATA: Register untfs.dll as the NTFS format provider ([fix/ntfs-format-provider](https://github.com/tomassrnka/reactos/commit/3031af1bd64329554c6206829ce30886c6d80123))
- NTFSLIB: Write the cluster bitmap in pieces of about 1 MB ([fix/ntfslib-bitmap-chunked-write](https://github.com/tomassrnka/reactos/commit/f5cdaa7286e4dfa031fdb4c98097cb6c3c88f484))
- NTFSNG: Pass PnP requests on to the storage device, dismount at its removal ([fix/dmshrink-pnp-dup](https://github.com/tomassrnka/reactos/commit/b5c321d32a30bbfe38f500a836679a7a54cea75e))
- NTFSNG: Round-2: fix release lock regression + harden diagnostic gating ([fix/fs-release-hygiene](https://github.com/tomassrnka/reactos/commit/5362f3fa6946938a90802f5d9927bfed19938a7d))
- NTFSNG: Read whole sectors when a device read is not sector-aligned ([fix/ntfsng-aligned-device-read](https://github.com/tomassrnka/reactos/commit/a2d853aae7801bdfe0f6e54c17ed40f09f4a066f))
- NTFSNG: fix a security-related bug ([fix/ntfsng-managevolume-privgate](https://github.com/tomassrnka/reactos/commit/678f85e01acc4d9e01cf484a47dde732aeeab7e5))
- NTFSNG: fix a security-related bug ([fix/ntfsng-relative-open-fcb](https://github.com/tomassrnka/reactos/commit/0a713d4fc1afb42f0938920158c1c621da9d3d8c))
- NTFSNG: Report FILE_PERSISTENT_ACLS later, reparse points on NTFS 3.x only ([fix/ntfsng-report-reparse-support](https://github.com/tomassrnka/reactos/commit/e9ccb06e9669556989d23741d47761270473f2cf))
- NTFSNG: fix a memory-safety bug ([fix/ntfsng-security-cluster](https://github.com/tomassrnka/reactos/commit/ed8d0f1c52373051775e91bc1181cd4a56b36096))
- NTFSNG: fix a security-related bug ([fix/ntfsng-volume-open-dacl-parity](https://github.com/tomassrnka/reactos/commit/41329bf981e67f2b987c7b16dfafc6eeebb82fa5))
- NTFSNG: read an immutable cluster size on the paging-file path ([fix/uf22-ntfsng-pagefile-divzero-v2](https://github.com/tomassrnka/reactos/commit/27cb71f7d881c49563c58f59cbf8d77b3550483f))

</details>

<details><summary>FAT (3 fixes)</summary>

- FASTFAT: Purge at shutdown also when a fat write failed ([fix/fastfat-fat-write-race](https://github.com/tomassrnka/reactos/commit/ebef9d5d6b43b5094cd5d343127952a08220cdf4))
- FASTFAT: Answer FileAttributeTagInformation ([fix/fs-allinfo-attrtag](https://github.com/tomassrnka/reactos/commit/207fd41338d8055ee95f3151fbe32073fea162fb))
- FORMAT: Stop at a wrong volume label and at the end of input ([fix/fat32-format](https://github.com/tomassrnka/reactos/commit/5e368205524a96f2acfbf8549e0668134f3625fe))

</details>

<details><summary>Btrfs and other file systems (7 fixes)</summary>

- BTRFS: Refuse a volume with an unfinished block group tree conversion ([fix/btrfs-block-group-tree](https://github.com/tomassrnka/reactos/commit/f51001308c2d645e1caf38712d99a70a39c984c5))
- BTRFS: Write the whole free space info item on 32-bit ([fix/btrfs-free-space-info-size](https://github.com/tomassrnka/reactos/commit/ff658f47d352117cb475eeec9cf5cb4e8a2d12f5))
- BTRFS: Sync to v1.10 ([fix/btrfs-sync-1.10-main](https://github.com/tomassrnka/reactos/commit/54e35c388a2617297197655bdbae650d6e0092f5))
- BTRFS: Small robustness fixes from the filesystem P2 sweep ([fix/p2-fs](https://github.com/tomassrnka/reactos/commit/63296a3096539a1ec636327d6ff24a740b0d07a0))
- NPFS: Fix the name, index number and short buffers of FileAllInformation ([fix/npfs-query-info](https://github.com/tomassrnka/reactos/commit/d8473eaba5fafe7f7986b6ab97efe3292fa8e4cd))
- NTFS: Accept NTFS volumes on GPT partitions ([fix/gpt-volumes](https://github.com/tomassrnka/reactos/commit/a378470c20f90576e3735219d5b62f6fa74d1c57))
- VFATLIB/CHKDSK: Do not report a mounted volume's normal state as damage ([fix/vfat-chkdsk-live-volume](https://github.com/tomassrnka/reactos/commit/9b6a1295c54d784d2af1b65ee9433395f6c74dc6))

</details>

<details><summary>win32k, GDI and USER (49 fixes)</summary>

- BOCHSMP: Map the frame buffer write-combined ([fix/framebuffer-write-combined](https://github.com/tomassrnka/reactos/commit/156732102a7b6a605684a6898ad2260234efc68d))
- FRAMEBUF: Never let the engine draw past the shadow ([fix/framebuf-shadow-surface-v2](https://github.com/tomassrnka/reactos/commit/748681206bbdf0ea6983e5e1fd124342a62f5a4b))
- GDI32: Size theme image margins and resize theme images as Windows does ([fix/gdi32-drawstream-margins](https://github.com/tomassrnka/reactos/commit/89b767567c0ff8c9ce19fd329e45f0f84b782a15))
- GDI32: Let GdiDrawStream draw a theme image another DC holds ([fix/gdi32-drawstream-shared-bitmap](https://github.com/tomassrnka/reactos/commit/189604cc1265688d919016140862c522de94451f))
- GDI32: Form the size of a PolyPatBlt batch in a wide type ([fix/gdi32-polypatblt-batch-size](https://github.com/tomassrnka/reactos/commit/8fbf70bde27f9f83afe450038f2d6025de1fde74))
- GDI32/WIN32K: GetCharacterPlacementW applies pair kerning on its own ([fix/uf29-extent-no-kerning-v3](https://github.com/tomassrnka/reactos/commit/2b4dac4d0de94d795e152144c97f1d26e6cd4f8f))
- LOCALSPL: State the limit of the SetJob priority rollback ([fix/p2-win32k-deadcode](https://github.com/tomassrnka/reactos/commit/22ed80445abd61f7424806265b5a1ce0935b6efc))
- NTUSER: Track a right press in the caption only on the frame's own caption ([fix/nc-rbutton-hittest](https://github.com/tomassrnka/reactos/commit/4ba6ba7595077021b0b3aeb66aa23f7947f3ac19))
- NTUSER: fix a security-related bug ([fix/ntuser-buildhwndlist-probe](https://github.com/tomassrnka/reactos/commit/c173bc99cd9bedc032e8415040bdd1fc4ebe6ca2))
- NTUSER: UpdateLayeredWindow changes geometry and content with no callback in between ([fix/ntuser-ulw-cross-thread](https://github.com/tomassrnka/reactos/commit/a99b05ff921782487c1237822bfd23a5e4b0a01f))
- NTUSER: PrintWindow: pin the render DC; keep the image's palette ([fix/ntuser-wmprint-printwindow](https://github.com/tomassrnka/reactos/commit/b39cfa0a0585c182e097a653467d9108adbb2236))
- NTUSER: Erase the frame of a frame change in place in screen coordinates ([fix/win32k-framechanged-screen-region](https://github.com/tomassrnka/reactos/commit/65fdcb4306f4ab6641b71bf3e03df3dffcd2a6bd)) — fixes [CORE-15934](https://jira.reactos.org/browse/CORE-15934)
- NTUSER/USER32_APITEST: Do not make a queue of another desktop the foreground queue ([fix/win32k-foreground-other-desktop](https://github.com/tomassrnka/reactos/commit/1ff6944f68ff918042d5d06dd800359d2a5dcabb))
- NTUSER/USER32_APITEST: Flash state, not WNDS_ACTIVEFRAME, keeps the flashed frame active ([fix/win32k-ncpaint-active-frame](https://github.com/tomassrnka/reactos/commit/4c6f350834763955417f6ce9d8fe3c8b99729546))
- NTUSER/USER32_APITEST: Drawing the caption and the frame leaves the client area alone ([fix/win32k-ncpaint-exclude-client](https://github.com/tomassrnka/reactos/commit/bf5d0fd9dce4ebb87b4c31177a9cbf6e0c888781))
- NTUSER/USER32_APITEST: Restore a minimized or maximized window from its screen rectangle ([fix/win32k-restore-ncframe-v3](https://github.com/tomassrnka/reactos/commit/cb1910d389a0169c107690778b5ca29dff3471ac))
- SYSMAIN: Firefox: make the Windows 7 SP1 version answer a default that explicit layers override ([fix/text-rendering-firefox](https://github.com/tomassrnka/reactos/commit/8277c8301d1f124c70a198b7a901b6f98efcee64))
- USER32: Load PNG compressed icon images from resources ([fix/user32-loadimage-png-icon](https://github.com/tomassrnka/reactos/commit/407806fa3899b9f9b7fc3f83caace44d04000c3f))
- USER32: Add SetCoalescableTimer, IsWindowArranged, RegisterSuspendResumeNotification and GetAutoRotationState, export the rest of the Windows 8 ones ([fix/user32-missing-exports](https://github.com/tomassrnka/reactos/commit/a8c2f482da03ca9526977743f8cc85848d141eee))
- USER32: The padded border of SystemParametersInfo and SystemParametersInfoForDpi follows the frame ([fix/user32-padded-border-metric](https://github.com/tomassrnka/reactos/commit/0bc52fb7a40e36c77a2c70c4248adfc776e97a8d))
- USER32: Keep the allocation checks of MsgiUnicodeToAnsiMessage ([fix/user32-wtoa-message-inline](https://github.com/tomassrnka/reactos/commit/964e32d499655cd2b9b7283938a0b4a96d00b1f6))
- USP10/USER32_APITEST: Draw fallback-font text in EDIT controls ([fix/edit-linked-glyphs](https://github.com/tomassrnka/reactos/commit/b18ffc61b543038558a8735c3f6ce041d2658fff))
- UXTHEME: fix a security-related bug ([fix/win32k-apihook-unload-once](https://github.com/tomassrnka/reactos/commit/5b4fc134ebb88ff525dd1dcccb68b3d35391b2b7))
- WIN32K: Keep wake bits truthful so MWMO_INPUTAVAILABLE waits can block ([fix/msgwait-wake-bits](https://github.com/tomassrnka/reactos/commit/9f999748cee6d78eb03d41e7d78629a1f9a02c19))
- WIN32K: Update the visible region of a DC that another process holds when a window moves ([fix/win32k-dce-foreign-visrgn](https://github.com/tomassrnka/reactos/commit/5acfab119bdd5109c81fc0007f611b8f0f8d4fbd))
- WIN32K: Serialize the monitor list of a graphics device ([fix/win32k-enumdisplaydevices-race](https://github.com/tomassrnka/reactos/commit/4074aef101b19cf57289ff12f210aa9ace3652ea))
- WIN32K: Pointer: show it after a switch also for a driver that hands it to the engine; convert only with a translation ([fix/win32k-pointer-flicker-v2](https://github.com/tomassrnka/reactos/commit/4a50965f493ed8f843ce897749374a1eee6899f9))
- WIN32K: Do not read the destination in StretchBlt when the ROP does not use it ([fix/win32k-stretchblt-dest-read](https://github.com/tomassrnka/reactos/commit/ffe18e86bbd70248c7f70084d3c83f23b7549a81))
- WIN32K/GDI32_APITEST: Add a Debug-only region allocation failure injection and tests ([fix/win32k-region-alloc-failure](https://github.com/tomassrnka/reactos/commit/0a806e7d568b87cfffbbcca0b1e37a2ef417d84b))
- WIN32K/GDI32_APITEST: fix a security-related bug ([fix/win32k-text-draw-lone-surrogate](https://github.com/tomassrnka/reactos/commit/3c80cc64a67806106a045898ffe9609f790958fa))
- WIN32K/GDI32_APITEST: Measure text extents from the glyph advance, not from a rendered bitmap ([fix/win32k-text-extent-no-raster](https://github.com/tomassrnka/reactos/commit/075d582c0b680524d9a3190372a610161963bb20))
- WIN32K/USER32: Post WM_MOUSEHWHEEL for horizontal wheel input ([fix/win32k-mousehwheel-v2](https://github.com/tomassrnka/reactos/commit/9307a750d9a2c90d23496501ed896c2b2971d6b9))
- WIN32K:GDI: fix a security-related bug ([fix/dwrite-fontface-from-hdc-private](https://github.com/tomassrnka/reactos/commit/13d9ac63bf9bbd8526f9518ca36cdca6a893b6ea))
- WIN32K:NTGDI: Keep the bitmap of a DC marked as selected across RestoreDC ([fix/gdi-restoredc-bitmap-owner](https://github.com/tomassrnka/reactos/commit/d1f31e82de197c7619325ed638588a5a6087602e))
- WIN32K:NTGDI: Free a font when its resource is removed ([fix/win32k-font-creator-ref-removal](https://github.com/tomassrnka/reactos/commit/fdfe0d4c8a458af4a2261ce1efe5d7e6deed4219))
- WIN32K:NTUSER: Keep the full pointer-sized menu item data ([fix/ntuser-menu-itemdata-64bit](https://github.com/tomassrnka/reactos/commit/038224c1aac4f40874db3193737f7824f6f13a68))
- WIN32K:NTUSER: Do not truncate the module handle of a WinEvent hook on x64 ([fix/ntuser-winevent-hook-module-x64](https://github.com/tomassrnka/reactos/commit/7d8438b08b15e5878018c694d465ccc8a0707cf4))
- WIN32K:NTUSER: Judge hung threads as Windows does ([fix/win32k-hung-window-v3](https://github.com/tomassrnka/reactos/commit/7c3c5f60801f73b9909db27414651a90b37ff67f))
- WIN32K:NTUSER: Zero the KBDLLHOOKSTRUCT before it is filled ([fix/win32k-kbdhook-zero-fill](https://github.com/tomassrnka/reactos/commit/6bd9d8242a8a99ed24d0f8897121ebc31a7b9219))
- WIN32K:NTUSER: Implement SPI_GETWHEELSCROLLCHARS and SPI_SETWHEELSCROLLCHARS ([fix/win32k-wheel-scroll-chars](https://github.com/tomassrnka/reactos/commit/1e51c969ad44e733c335feea6ef3a0cd9b95120b)) — fixes [CORE-18390](https://jira.reactos.org/browse/CORE-18390)
- WIN32SS:DIB: Do not step a negative-height StretchBlt flip through the source stride ([fix/win32k-stretchblt-negative-height-v2](https://github.com/tomassrnka/reactos/commit/5cdbec32ed6d86bc5c798fbd1ed6fea9a4ef591c))
- WIN32SS:NTGDI: Release the SharedFace creator reference for private fonts ([fix/win32k-font-sharedface-leak-v2](https://github.com/tomassrnka/reactos/commit/e8af6e5b83438ab486b985151c5b896e131f034e))
- WIN32SS:NTUSER: Store the mouse sonar and vanish settings in UserPreferencesMask ([fix/ntuser-mouse-vanish-sonar](https://github.com/tomassrnka/reactos/commit/1b8f087eb2c9242a3aaae649c9dc13c2e3606434))
- WIN32SS:NTUSER: Free a withdrawn sent message instead of parking it for the receiver ([fix/uf19-attach-queue-v2](https://github.com/tomassrnka/reactos/commit/822498ef95c84613c52b5b861fc65bddc489a865))
- WIN32SS:NTUSER: Fail the process callout when the process is terminating ([fix/win32k-process-terminating](https://github.com/tomassrnka/reactos/commit/9ad27b19d5de58129c2acf58cdea8b62ea155810))
- WINED3D: Release the adapter when OpenGL adapter initialisation fails ([fix/wined3d-gl-adapter-leak](https://github.com/tomassrnka/reactos/commit/f92138dc6b8def54d70ad54d6e24defbe0ffb477))
- WINED3D/DDRAW: Do not show the no 3D notice for DirectDraw ([fix/ddraw-no3d-sw-gl](https://github.com/tomassrnka/reactos/commit/79d7677c61177b27a6804e83e857590d3524f5d5))
- WINED3D/DDRAW: Fail quietly without OpenGL 2.0, as Windows does without a 3D driver ([fix/wined3d-quiet-no-3d](https://github.com/tomassrnka/reactos/commit/90dc6656728b7ec9d05fe02af742853e9d5d9c7e))
- WINHTTP/QUARTZ: Do not call a function inside HRESULT_FROM_WIN32 ([fix/winerror-hresult-from-win32-single-eval](https://github.com/tomassrnka/reactos/commit/86231bed87f9721f5a8b9dc9f760eccf90350c56))

</details>

<details><summary>Console and csrss (7 fixes)</summary>

- CONSRV: Bring the cursor into the view after output that only moved it ([fix/consrv-view-follow-cr](https://github.com/tomassrnka/reactos/commit/fa882940169ec0faa2f0cb3d3a29b3950fe4862c))
- CONSRV: Limit the console window repaints while output streams ([fix/consrv-view-follow-repaint-limit](https://github.com/tomassrnka/reactos/commit/9e1bce9eb5980a44c303a80df12d516b6c758fbb))
- CONSRV: fix a security-related bug ([fix/consrv-write-stall-v2](https://github.com/tomassrnka/reactos/commit/ee428ea768d77588e9031faf50b4a82cf6d48f7b))
- CONSRV/KERNEL32: csrss survives a client crash in console control-event delivery [pick] ([main/csrss-survive-alone](https://github.com/tomassrnka/reactos/commit/6d7a4195dbb36b8aa2fb5652ddf22ad07fc08370))
- CSRSRV: fix a memory-safety bug ([fix/csrsrv-thread-refcount](https://github.com/tomassrnka/reactos/commit/748b6642ff52444d0f8e2a8523ed859e0f9319ad))
- WINLOGON: Keep EWX_FORCE for the logoff that starts a shutdown ([fix/shutdown-hang-with-force](https://github.com/tomassrnka/reactos/commit/f38398670646b8633c00bab1122a6f94ceeb030a))
- WINSRV: Accept hard errors that come without a CSR thread ([fix/winsrv-harderror-null-thread](https://github.com/tomassrnka/reactos/commit/a4290dcbf54bba7259eb70e68b788542ea04e295))

</details>

<details><summary>WoW64 (36 fixes)</summary>

- CMAKE: Keep WoW64 table generation out of default native builds ([fix/wow64-native-i386-stubs](https://github.com/tomassrnka/reactos/commit/427fc062520d7e3b17ff659b3069c161e2e53d29))
- KERNEL32: Accept the DEP mitigation attribute from a WOW64 guest on amd64 ([fix/kernel32-wow64-dep-attribute](https://github.com/tomassrnka/reactos/commit/7656641d8873930456bf62bf09a7dceb394a5cb7))
- MSI: Treat a failed window enumeration like an incomplete list ([fix/msi-wow64-ca-honest](https://github.com/tomassrnka/reactos/commit/30b77862d886031f32fa6c5a93011c462c8a9da3))
- NTDLL: Initialize the native TLS list in WoW64 processes ([fix/wow64-tc32-tlslist](https://github.com/tomassrnka/reactos/commit/c8ac8e964823995f0372e99d61140a4ed1045552))
- NTDLL/WOW64: Keep the guest APC routine out of the callee's home space ([fix/wow64-ff32-apc-home-space](https://github.com/tomassrnka/reactos/commit/029c1d6e96cac503d72d1e0f0ad09364afd70b83))
- NTDLL/WOW64: Give the guest system call stubs the NT 5.2 x64 WOW64 layout ([fix/wow64-guest-stub-layout](https://github.com/tomassrnka/reactos/commit/0e7370f326e999ed65f7dd791462344ad87de2d6))
- NTOS:KE: Load the WoW64 FS base of a thread attached to another process ([fix/wow64-tc32-cmteb](https://github.com/tomassrnka/reactos/commit/6a317eb5731f82f6c6f6f873245359b3637cb378))
- NTOS:PS: Declare the WoW64 flag of PspSetProcessTlsInformation only where it is used ([fix/wow64-ff32-dynamic-tls](https://github.com/tomassrnka/reactos/commit/5f549a72d6f88fb0f25ff3e42df4f5f12a47a86d))
- NTUSER: Keep the client area when the frame changes in place ([fix/wow64-tc32-lister-framechanged](https://github.com/tomassrnka/reactos/commit/e124449b5127881897433df4335e711f0408a26d))
- RTL: Report the 32-bit modules of a WOW64 process in the debug buffer ([fix/tasklist-wow64-modules](https://github.com/tomassrnka/reactos/commit/483f4066939838fd03aa888fc74d12d1a82c959e))
- SYSWOW64: Install dwrite.dll and user32_vista.dll for 32-bit programs ([fix/wow64-ff32-syswow64-deps](https://github.com/tomassrnka/reactos/commit/a97cf79b13568e0c6d2a6db4897b3331170a2c9f))
- USER32: Align the MsgWaitForMultipleObjectsEx wait structure ([fix/wow64-idle-msgloop-spin](https://github.com/tomassrnka/reactos/commit/34fc0328e8efc8fc43c4af3d36c316e9d87a9f04))
- WOW64: Translate console screen buffer, code page, language and shutdown CSR calls ([fix/wow64-console-csr-batch](https://github.com/tomassrnka/reactos/commit/ae268e8250677a0c6e6bd80d04be4918dce1d763))
- WOW64: Check and copy the guest attribute values before creating the process ([fix/wow64-ff32-attribute-list](https://github.com/tomassrnka/reactos/commit/556dfced79703616d04199baef8f59413c1290e0))
- WOW64: Grow the guest stack before any local of the exception dispatch is written ([fix/wow64-ff32-exception-stack-limit](https://github.com/tomassrnka/reactos/commit/5d6a5dd48e6cfae42e620e2884c82ac119b42712))
- WOW64: Give the first guest thread a growable stack with a guard page ([fix/wow64-first-thread-stack-guard](https://github.com/tomassrnka/reactos/commit/ed5408f5a26ecb9b3e9e2ed33243ca2075ae324c))
- WOW64: Read the job information arguments once ([fix/wow64-job-information](https://github.com/tomassrnka/reactos/commit/38b135e2ab8bba9bc60fa22250077980ccfdbd74))
- WOW64: Translate NtPowerInformation for the fixed-size records ([fix/wow64-power-information](https://github.com/tomassrnka/reactos/commit/c4e9fc60a4a44c30a806618d3174e19e7dfb418c))
- WOW64: Translate NtQueryInformationProcess image file name classes ([fix/wow64-process-image-name](https://github.com/tomassrnka/reactos/commit/1c8ab233ecce06819e25febdbced51c7845178e5))
- WOW64: Translate more NtQueryInformationProcess classes ([fix/wow64-query-information-process](https://github.com/tomassrnka/reactos/commit/fef0409cfb8d6a30cd13881b9a411044a282cc6c))
- WOW64: Translate NtQueryVirtualMemory(MemorySectionName) ([fix/wow64-query-virtual-memory](https://github.com/tomassrnka/reactos/commit/7eb62166fbfe8187e13cf6cb58ea42dd4893802d))
- WOW64: Translate NtRaiseHardError ([fix/wow64-tc32-crashreport](https://github.com/tomassrnka/reactos/commit/c9c523c631b56034f622ec27f36c93805d64503e))
- WOW64: fix a security-related bug ([fix/wow64-token-dacl-label](https://github.com/tomassrnka/reactos/commit/ac2fecbf6d6654c59df726552d8d7f4e5da8c1a6))
- WOW64: Name the untranslated request in the unsupported diagnostics ([fix/wow64-unsupported-diag](https://github.com/tomassrnka/reactos/commit/52134ff90e129f02148c4b8f10ccb8ccec9b960e))
- WOW64/USER32: Review follow-up for the idle message loop test ([fix/wow64-idle-loop-followups](https://github.com/tomassrnka/reactos/commit/c57e3f7cbae890d97d1a29106ae6a354d3fc97ec))
- WOW64WIN: Keep the transition guard set on a popped callback frame ([fix/wow64-callback-return-intransition](https://github.com/tomassrnka/reactos/commit/1cc03ad63d92ef02ce15d6234fc67c0c1cd6b3e4))
- WOW64WIN: Guard checked-build-only services instead of dropping them ([fix/wow64-p3-l1-gen-dbg](https://github.com/tomassrnka/reactos/commit/133d8804260790900b2d9d63b33d8c98b4a2a6a5))
- WOW64WIN: fix a security-related bug ([fix/wow64-p3-l2-ime-probe](https://github.com/tomassrnka/reactos/commit/610e5567882d0e33c528e1db8b2dcfc9c53ee87a))
- WOW64WIN: Never forward an unreadable guest structure in its 32-bit layout ([fix/wow64-p3-l3-display](https://github.com/tomassrnka/reactos/commit/23aeabdaa0e5f340b691be9acab65f0bf711455c))
- WOW64WIN: Snapshot guest GDI structures and fix EnumObjects copy-back ([fix/wow64-p3-l4-gdi-extra](https://github.com/tomassrnka/reactos/commit/044bc060ca0bce2d902c1ef31430f4658abcc4f4))
- WOW64WIN: Copy back every mouse point the kernel fills ([fix/wow64-p3-l5-mousemove](https://github.com/tomassrnka/reactos/commit/c0bd55ad421aa59a1f49d20022e596de66293c9c))
- WOW64WIN: fix a security-related bug ([fix/wow64-p3-l6-rawinput](https://github.com/tomassrnka/reactos/commit/55bb168a2160a3003166dbd39867efa92562c65a))
- WOW64WIN: Refuse native HID preparsed data for 32-bit callers ([fix/wow64-p3-l7-misc-preparsed](https://github.com/tomassrnka/reactos/commit/b0b34895a40a1dc26c665e2610345a55d245d49b))
- WOW64WIN: Translate WM_MEASUREITEM and WM_DRAWITEM ([fix/wow64-tc32-ownerdraw](https://github.com/tomassrnka/reactos/commit/45d2e03ecf23a12b6cb75a638d77ce7fb80ee394))
- wow64win: wow64win: list the 39 bucket services in the generator's manual set [pick] ([main/wow64win-bucket-manual-set](https://github.com/tomassrnka/reactos/commit/9502f19be6c2b8b45e1947fb856f47ef2347d060))
- wow64win: wow64win: single generator regen after s1-s5 bucket merges; drop three reconciled duplicate thunks [pick] ([main/wow64win-stage1c-regen](https://github.com/tomassrnka/reactos/commit/1f53c219d25ef0bc67ed893e668c2aaf1ad19674))

</details>

<details><summary>Networking (57 fixes)</summary>

- AFD: Wait for in-flight transport requests before freeing a closed socket ([fix/afd-close-inflight](https://github.com/tomassrnka/reactos/commit/5466bb0f712709c4df4d09ec499dd912cf0d51f7))
- AFD: Complete a cancelled connect only after its transport connect, close the old endpoint on listen ([fix/afd-connect-cancel](https://github.com/tomassrnka/reactos/commit/bc2ba8b5023356cde36e6c108d0748017f8aa022))
- AFD: Issue the first listen from a work item and retry failed listens ([fix/afd-listen](https://github.com/tomassrnka/reactos/commit/b49924191a5557a904dbec84a0de6296b75d006c))
- AFD: Honour the listen backlog ([fix/afd-listen-backlog](https://github.com/tomassrnka/reactos/commit/a8f46b6d34404d7f5d78a930824fc78f7f6862b1))
- AFD: Release the buffers of a cancelled datagram receive ([fix/afd-recvdatagram-cancel-cleanup](https://github.com/tomassrnka/reactos/commit/6c16f86142e989c7f1d746a49e5327229ee81d6a))
- AFD: Scatter received datagrams over all buffers of the request ([fix/afd-recvfrom-multibuf](https://github.com/tomassrnka/reactos/commit/a52f46350bd0062d3cbbae0d96e57926ef946b24))
- AFD: fix a memory-safety bug ([fix/afd-security-core](https://github.com/tomassrnka/reactos/commit/eb2fbb213354746b55f9ba797d7550cf27eda473))
- AFD: Complete a select request whose poll could not be allocated ([fix/afd-select-alloc-failure](https://github.com/tomassrnka/reactos/commit/99d435d6ae181e65453cc738ba263c33f9b9d0c6))
- AFD: Check select requests and reference only sockets ([fix/afd-select-handles](https://github.com/tomassrnka/reactos/commit/bb23ac1a8fc3d338d28ba64ede20d933c6e2ad5e))
- AFD: Allocate the receive work item before opening the address ([fix/afd-udp-recv-rearm](https://github.com/tomassrnka/reactos/commit/fd9d19a90d7bc0986a8a706d790a2bc781925905))
- AFD/MSAFD: Put the AcceptEx initial data at the start of the buffer ([fix/afd-acceptex-initial-data-m8b1](https://github.com/tomassrnka/reactos/commit/892b4a8838dc9c255876177b77957f0debf8aeb8))
- AFD/MSAFD: Fail a second ConnectEx instead of freeing the buffer the first one uses ([fix/afd-connectex-overlap](https://github.com/tomassrnka/reactos/commit/21aa363d94e8545008cd6782b8f8e7226ce1d433))
- E1000: Reclaim every finished transmit descriptor ([fix/e1000-tx-reclaim-all](https://github.com/tomassrnka/reactos/commit/af06b043205de5ccfc9c5ad977419a152aba3904))
- IPHLPAPI: GetAdaptersAddresses: fill the interface luid and report the Windows 7 structure size ([fix/iphlpapi-adapters-luid-bestinterface](https://github.com/tomassrnka/reactos/commit/f19ca93b6da89a28a9c28f76a7069dd66ef11d11))
- IPHLPAPI: Implement GetBestRoute2 for IPv4 ([fix/iphlpapi-getbestroute2](https://github.com/tomassrnka/reactos/commit/9b7647ff5bc0ad55e28bbd6837ecc0be560bbcd1))
- IPHLPAPI: The Vista IP table getters fail instead of reporting success with no table ([fix/iphlpapi-ip-table-not-supported](https://github.com/tomassrnka/reactos/commit/1e2402a2b71015e4bf6b4e6f7fa7c2fdd8ed4aa3))
- IPHLPAPI: Add NotifyIpInterfaceChange and CancelMibChangeNotify2 ([fix/iphlpapi-ipinterface-change](https://github.com/tomassrnka/reactos/commit/d59e978ea5685811565942feb8d84b99d5e89bd3))
- IPHLPAPI: Add NotifyRouteChange2 and NotifyUnicastIpAddressChange ([fix/iphlpapi-notify-route-change2](https://github.com/tomassrnka/reactos/commit/2dc47e66875e763adbdeabf77b9fdc77b2c5002d))
- IPHLPAPI/IPHLPAPI_APITEST: List every TCP and UDP socket in the table functions ([fix/iphlpapi-tcpudp-all-rows](https://github.com/tomassrnka/reactos/commit/79bc675ad5b2e3295ca581d8724393bc5f67a7fd))
- MSAFD: Return INVALID_SOCKET from every WSPAccept failure exit ([fix/msafd-accept-invalid-socket](https://github.com/tomassrnka/reactos/commit/138bb32622cfd59f2ed210559dac63c53f08e939))
- MSAFD: Time out a receive only where it can be cancelled alone ([fix/msafd-socket-timeouts](https://github.com/tomassrnka/reactos/commit/73ca54872d03355f6f43e1a40d28e048ec4575a3))
- MSAFD: Fail rejected SIO_UDP_CONNRESET requests also when overlapped ([fix/net-udp-connreset-v2](https://github.com/tomassrnka/reactos/commit/bede2eac13fc886f4a9084c3a54623354e92cfd1))
- MSAFD: Leave the byte count of a failed send unchanged ([fix/tcpip-abortive-close](https://github.com/tomassrnka/reactos/commit/2641a1980a3f7153d059345253bfcc9cc078c0ae))
- MSAFD/WS2_32_APITEST: Let SO_UPDATE_CONNECT_CONTEXT run after getpeername ([fix/msafd-getpeername-connect](https://github.com/tomassrnka/reactos/commit/210e02b15f9c024692fec7996e725fa1566f927a))
- NDIS: Do not send queued packets from the interrupt DPC ([fix/ndis-dpc-send-budget](https://github.com/tomassrnka/reactos/commit/dc35710e39a62b1e84559c5e3945f801502ff9ff))
- NDIS: Pass the miniport's own context to MiniportHalt ([fix/ndis-halt-context](https://github.com/tomassrnka/reactos/commit/1686b1f7044eafc889512ecda69fafd60fcd2acb))
- NDIS: fix a security-related bug ([fix/p2-net](https://github.com/tomassrnka/reactos/commit/6576554d9f48d48619ab8b3287a3f3704fe6e146))
- NET: NET SHARE: accept printer and serial ports and the other paths Windows takes ([fix/net-share-device-path](https://github.com/tomassrnka/reactos/commit/3dc79c9398072e6e38de3ed82f1502c93e611f9c))
- NET: net start lists the started services and reports results like Windows ([fix/net-start-list](https://github.com/tomassrnka/reactos/commit/662f5a001535dd8a331bdfcddc7d44d4bf91b78e))
- NET: Implement NET VIEW and rewrite NET SHARE to match Windows ([fix/net-view-share](https://github.com/tomassrnka/reactos/commit/be71022d1f9459615d71b48d728fe0e66869d4ef))
- NETAPI32: Add NetGetAadJoinInformation and NetFreeAadJoinInformation ([fix/netapi32-missing-exports](https://github.com/tomassrnka/reactos/commit/8be6c1874c060109867562f1d38291c7cb03884e))
- NETCFGX: Fix the indentation of a trace in ExtractParameterValue ([fix/netcfgx-gcc13-indent](https://github.com/tomassrnka/reactos/commit/3f9d46eccc526aef1f177fb6169a247218ab27c0))
- RPCRT4: Let the reading thread cancel its own pipe read on server stop ([fix/rpcrt4-np-close-read](https://github.com/tomassrnka/reactos/commit/7cd8533b469dad676db9768852ad4d38c1992c63))
- RPCSS: Create and signal Global\ScmCreatedEvent ([fix/rpcss-scm-created-event](https://github.com/tomassrnka/reactos/commit/262f964815abdba01d7294a5b2b3cc787e241072))
- schannel: fix a security-related bug ([fix/schannel-cert-check-ordering](https://github.com/tomassrnka/reactos/commit/af2e84dfcd003864ca77d6a0d0f832c9d8634da8))
- SCHANNEL: fix a security-related bug ([fix/schannel-server-cert-validation](https://github.com/tomassrnka/reactos/commit/5f4ab40f29a24218f06ab1265c1d114c43737023))
- SRVSVC: NetrServerGetInfo: allocate the server name on its own ([fix/net-localgroup-server-info-v2](https://github.com/tomassrnka/reactos/commit/3f76980ef57f2d691829c8fd47c2d8330ce64760))
- TCPIP: Do not revive a closing address file in AddrFindShared ([fix/p2-tcpip-memsafety](https://github.com/tomassrnka/reactos/commit/dd52ec799acec04d05a816712a0c6467c47abbe1))
- TCPIP: Do not revive a dying address file in the receive search ([fix/tcpip-addrfile-ref](https://github.com/tomassrnka/reactos/commit/28babc677f4cc89034508d3fe2789d0f47e313d2))
- TCPIP: Do not touch a request the cancel routine does not own ([fix/tcpip-cancel-request](https://github.com/tomassrnka/reactos/commit/bf2b7021a565c38311837d5df4708c6e6354079f))
- TCPIP: Report exhausted ephemeral TCP ports as WSAENOBUFS ([fix/tcpip-ephemeral-port](https://github.com/tomassrnka/reactos/commit/53312e67db49bc57f3c5e12a2a44c8aaf5c405ab))
- TCPIP: fix a security-related bug ([fix/tcpip-icmp-options](https://github.com/tomassrnka/reactos/commit/57f4d2849c173853199fb8242f006b0f94f71027))
- TCPIP: Take only the ICMP replies that answer the echo request ([fix/tcpip-icmp-reply-match](https://github.com/tomassrnka/reactos/commit/6269b14ad2925bec9522f7a41ff44be2c561699a))
- TCPIP: fix a memory-safety bug ([fix/tcpip-ipv4-reasm-bounds](https://github.com/tomassrnka/reactos/commit/c5b6d432b5f00fb4915cf2d32bef2dec6bdc8759))
- TCPIP: fix a security-related bug ([fix/tcpip-ipv4-reasm-total](https://github.com/tomassrnka/reactos/commit/10ef9a6a9707f86ef3776da715a2d88b9d535d15))
- TCPIP: Keep the unspecified address of an address file across a connect ([fix/tcpip-listener-after-cancel](https://github.com/tomassrnka/reactos/commit/d01a34546ae7f8a69b559c94bcf9e2085d8df947))
- TCPIP: Coalesce media status changes in one worker per adapter ([fix/tcpip-media-status-order](https://github.com/tomassrnka/reactos/commit/9c7bd667dc069db2762fe6b277d9f700e0e27fb7))
- TCPIP: Return TCP ports in network order from the lwIP glue ([fix/tcpip-peer-port-order](https://github.com/tomassrnka/reactos/commit/c9e5bed199737326a0a1ce9bb62c4e2ecc81b381))
- TCPIP: Publish an interface's three TDI entities together ([fix/tcpip-socket-scale](https://github.com/tomassrnka/reactos/commit/44338a8e1cada51df6c9fcf57eaacb35359793e4))
- TCPIP: Let a SYN reopen a connection in TIME-WAIT ([fix/tcpip-timewait-reopen](https://github.com/tomassrnka/reactos/commit/3ef0e081c0bcd1fe56880cb38c9c44edf23704f0))
- TCPIP/AFD: Discard unread data when the peer resets the connection ([fix/tcpip-abortive-close-data](https://github.com/tomassrnka/reactos/commit/f1468a9a209c7a86013149ccbbedc966f8ab41d6))
- WININET: Fail a stalled read after the receive timeout, and report a damaged gzip body like Windows ([fix/wininet-gzip-stall-v2](https://github.com/tomassrnka/reactos/commit/7d45027d8a9f1570f8bd219acde7f72e555f9a60))
- WS2_32: gethostname: fail instead of returning an untouched or unterminated name ([fix/ws2_32-gethostname-errors](https://github.com/tomassrnka/reactos/commit/bec84ed077129dedd83f4c67aebdb862e6cf1740))
- WS2_32: Add GetHostNameW and GetAddrInfoExCancel ([fix/ws2_32-missing-exports](https://github.com/tomassrnka/reactos/commit/7350867149eddb98956033bc44eaaa07846e59d0))
- WS2_32: Size the query-set buffer with the alignment the copy uses ([fix/ws2_32-queryset-align](https://github.com/tomassrnka/reactos/commit/9453b74cc229d27cb75167ca12fe33195180942c))
- WS2_32: Retry WSAPoll only for a confirmed stale socket, and without waiting ([fix/ws2_32-wsapoll](https://github.com/tomassrnka/reactos/commit/57a1c6a72e5dd75b3e4ad6064a6dff5d155552e6))
- WS2_32: WSASendMsg: reserve the completion context before sending, leave the event alone when a routine is given ([fix/ws2_32-wsasendmsg](https://github.com/tomassrnka/reactos/commit/46ccaf4e64896c91ea89e92095c464a6fbf0fd24))

</details>

<details><summary>Audio and multimedia (6 fixes)</summary>

- MMSYS: Keep the volume tray icon when OK changes nothing ([fix/mmsys-tray-icon-unchanged](https://github.com/tomassrnka/reactos/commit/ac57e989fb963aeae9f51407f94ac2f003251223))
- SNDREC32: Do not hang on exit when the recording device failed to open ([fix/sndrec32-close-hang](https://github.com/tomassrnka/reactos/commit/abdd107b6613a3cfe213901db55ed50a75114804))
- SNDREC32: Say once that there is no recording device ([fix/sndrec32-no-input-device-v3](https://github.com/tomassrnka/reactos/commit/4db60c2901a2b03d982db1f733279a55c9ee7fb3))
- WINMM: Load the multimedia drivers on first use, not in DllMain ([fix/winmm-lazy-drivers](https://github.com/tomassrnka/reactos/commit/f10b55a7ff07e55c2dcaacd83b69ebbb09ee42b8))
- WINMM: Hold the system timer resolution while timeBeginPeriod is active ([fix/winmm-timebeginperiod-resolution](https://github.com/tomassrnka/reactos/commit/6693aa6d63a6e2a0a34073e5408c1e47309db0bf))
- WINMM: Read the QPC frequency once in timeGetTime ([fix/winmm-timegettime-div0](https://github.com/tomassrnka/reactos/commit/f2301e5030a1f60126d2a391b6d8511255fda11b))

</details>

<details><summary>Storage, USB, PnP and drivers (6 fixes)</summary>

- ATAPI: Flush the write cache after a FUA write on disks without NCQ ([fix/atapi-fua-flush](https://github.com/tomassrnka/reactos/commit/8360dcbfdfb5a8eee876be05988a0d4326570e3f))
- hid: fix a security-related bug ([fix/hidparser-descriptor-bounds-r2](https://github.com/tomassrnka/reactos/commit/237085375d07b0d1cf45238899e6a82fd5a0575c))
- MOUHID/NTUSER: Fix absolute pointer scaling and the move to (0, 0) ([fix/abs-pointer-mapping](https://github.com/tomassrnka/reactos/commit/188ca9fa0bfcc9988f164e9508758f0966253cb9))
- MOUNTMGR: Link one stored volume name, and keep offline and saved links right ([fix/mountmgr-stored-links](https://github.com/tomassrnka/reactos/commit/1c7dd9992ef828296054e96dbfca5338b1c49416))
- PCI: Make BAR-sizing failure handling safe against the two-pass query ([fix/pci-bar-sizing-decode-off](https://github.com/tomassrnka/reactos/commit/91f3e15da57697c293e1a24d7a3f8e70c3e35980))
- USBUHCI: Poll interrupt endpoints at their period ([fix/usbuhci-drag-interrupt-poll](https://github.com/tomassrnka/reactos/commit/e02645c587f2c15894e44e58911c9a39f99eeaf5))

</details>

<details><summary>Shell, explorer and common controls (50 fixes)</summary>

- BROWSEUI: Keep the last results and the final status of a finished search ([fix/shellfind-search-complete](https://github.com/tomassrnka/reactos/commit/4b089240d68d289edb6c003eb078aa093c7ab2ae))
- BROWSEUI/EXPLORER: Keep the taskbar toolbars across logons ([fix/taskbar-toolbars-persist-v3](https://github.com/tomassrnka/reactos/commit/32d75e0450ce5d6e90b68a1cdc4a9bc61e6ced04))
- BROWSEUI/FONTEXT: Search: list the items of the Fonts folder ([fix/shellfind-font-folder-results-v2](https://github.com/tomassrnka/reactos/commit/c56e3d5307a30e75516230022181244aa92c9468)) — fixes [CORE-16444](https://jira.reactos.org/browse/CORE-16444)
- BROWSEUI/SHELL32: Ask for a disk when opening an empty drive ([fix/explorer-insert-disk](https://github.com/tomassrnka/reactos/commit/cbe26fad60ee04a40f3881ac105a56a7b72c6383))
- CMD: Keep the console cursor size at the prompt ([fix/cmd-cursor-size](https://github.com/tomassrnka/reactos/commit/29ecd7336e27ab511a44e7db162d0ea448c7a73b))
- CMD: Add the ftype internal command ([fix/cmd-ftype](https://github.com/tomassrnka/reactos/commit/01737a9fe0c2596838dbc2443416b0a7fd644dec))
- CMD: Start the external programs of a pipeline directly ([fix/cmd-pipe-direct-exec](https://github.com/tomassrnka/reactos/commit/9876d73dcef72b890ae424af9395f8a4f60393b8))
- CMD: Write redirected output in the console code page by default ([fix/cmd-redirect-ansi](https://github.com/tomassrnka/reactos/commit/25a3be3914f44d06b7225f2e21cdac91bb22e5a6)) — fixes [CORE-14795](https://jira.reactos.org/browse/CORE-14795)
- CMD: Run only files with a PATHEXT extension for a name without one ([fix/cmd-search-extensionless](https://github.com/tomassrnka/reactos/commit/a2cf87a2385c45912696d67519e596a4174dca56)) — fixes [CORE-12604](https://jira.reactos.org/browse/CORE-12604)
- CMD: start: give the program its name as typed ([fix/cmd-start-command-line](https://github.com/tomassrnka/reactos/commit/34efc0ffe44ab8d3e79ce4730433613d36226fe0))
- COMCTL32: Implement ImageList_WriteEx ([fix/comctl32-missing-exports-v2](https://github.com/tomassrnka/reactos/commit/f264240c6f5daa5b83af68d62709bcd47fc447e5))
- COMCTL32: Keep the wizard caption on a page that has no title ([fix/comctl32-propsheet-title](https://github.com/tomassrnka/reactos/commit/d99b70541d359a58f062d3fac4840812991db9c6)) — fixes [CORE-20687](https://jira.reactos.org/browse/CORE-20687)
- COMCTL32: Export TaskDialog and TaskDialogIndirect, number the version 6 icon exports like Windows ([fix/comctl32-v6-ordinals](https://github.com/tomassrnka/reactos/commit/1c5a339c22147105eff7af161e14549f8ed1d938))
- COMCTL32: Send a full NMITEMACTIVATE for Enter in the list-view ([fix/listview-enter-itemactivate](https://github.com/tomassrnka/reactos/commit/07fdcc5f1a9b76e72936eab9ed1e5c438021afd1))
- COMCTL32: Toolbar: forward the notifications of its child controls to the parent ([fix/toolbar-forward-child-notify](https://github.com/tomassrnka/reactos/commit/424776c69fca2c96b33248738e0de4acc0eb3b77))
- COMDLG32: fix a memory-safety bug ([fix/comdlg32-canonical-path-overflow](https://github.com/tomassrnka/reactos/commit/9fb0c558b8b2a09f7083a5660573972032ffdd6d))
- COMDLG32: Add the default extension only to a name that has none ([fix/comdlg32-defext-keep-extension](https://github.com/tomassrnka/reactos/commit/a14b21ad5a83a3be5ef1e0b441a10b1ebb6343a2)) — fixes [CORE-20756](https://jira.reactos.org/browse/CORE-20756)
- COMDLG32: Fall back to the personal folder as documented ([fix/comdlg32-save-dialog-folder](https://github.com/tomassrnka/reactos/commit/78de99821d70c7bd25887ab176c36efcec67d14f))
- EXPLORER: Make "Show Quick Launch" show the Quick Launch toolbar ([fix/explorer-show-quick-launch](https://github.com/tomassrnka/reactos/commit/96117a2f8ff0b6ccfc18269339cbc75a3f22782a))
- EXPLORER: Never wait for another process's window on the shell thread ([fix/shell-no-blocking-sends](https://github.com/tomassrnka/reactos/commit/d81ea6eb3f95b2f6600bca20d06ea246bab68a0a))
- EXPLORER: Notice a full-screen window that moves or resizes and keeps its handle ([fix/shell-tray-topmost-fullscreen](https://github.com/tomassrnka/reactos/commit/2f00c35e48e26beb6ee237603e059f7b5c240313))
- IEFRAME: Save an Internet shortcut whose icon properties are still held open ([fix/ieframe-urlsave-open-properties](https://github.com/tomassrnka/reactos/commit/cdd934436dee6004aa97110932e20b026ea55f6a))
- KERNEL32: Fail foreign-machine images on 64-bit without a hard error ([fix/kernel32-amd64-foreign-image](https://github.com/tomassrnka/reactos/commit/dbdf7c33a0ac0a766bfdc4ff9d95e1160466329f))
- NETSHELL: Report that Repair is not supported from the folder menu ([fix/netshell-status-repair](https://github.com/tomassrnka/reactos/commit/0a72aefb849cef7d6285b8a16d4a7ce7eb944b35))
- NETSHELL/APPHELP: fix a security-related bug ([fix/p2-dlls-shell-print](https://github.com/tomassrnka/reactos/commit/430df20cb91478e490f7a9f057b7d3ef8e44627c))
- NTOBJSHEX: Show a Properties page for NT objects ([fix/ntobjshex-properties-v2](https://github.com/tomassrnka/reactos/commit/311117ce937b7257722c836c45b58fc92f640d6d))
- PROPSYS: Implement VariantCompare ([fix/propsys-variantcompare-v2](https://github.com/tomassrnka/reactos/commit/21d06586e5b66fd81fb92d7f27d52ee8ffbed027))
- SHELL32: Report a failed rename in a folder view ([fix/defview-rename-error-box](https://github.com/tomassrnka/reactos/commit/6e90e9541ff0abbcfc13150ffb2fdc76b108c79c))
- SHELL32: Count the size of items of other folders in the status bar ([fix/defview-statusbar-item-size](https://github.com/tomassrnka/reactos/commit/3920df73de1ded54bb9440a930d4d2c061299802))
- SHELL32: Let the Properties of several files change their attributes ([fix/filedefext-multi-attributes](https://github.com/tomassrnka/reactos/commit/b9d7f580d05dce1bffc0e1ccac6e62fdc1e6cd37))
- SHELL32: Implement IApplicationAssociationRegistration ([fix/shell32-app-assoc-registration](https://github.com/tomassrnka/reactos/commit/bea734aeaab150ce307138feaaea91dfa10ff4c6))
- SHELL32: Show no error box when the user cancels a copy ([fix/shell32-copy-cancel-silent](https://github.com/tomassrnka/reactos/commit/b5b1098edf879f0cffe4b8eb4f28185b7acea289))
- SHELL32: Do not set an unchanged drive label on OK ([fix/shell32-drive-label-unchanged-m](https://github.com/tomassrnka/reactos/commit/600f4fe60bf9a91057f7dd270100f807ab947c8d))
- SHELL32: Report the result of the drive Tools buttons ([fix/shell32-drive-tools-honest](https://github.com/tomassrnka/reactos/commit/869e1175f5026bd9c46de8b780ef42bc30919bee))
- SHELL32: Pass the ShellExecuteEx directory to the context menu handler ([fix/shell32-exec-context-menu-directory](https://github.com/tomassrnka/reactos/commit/8d089d7697b4e47b9b0937297caf9c76e75397f4)) — fixes [CORE-19855](https://jira.reactos.org/browse/CORE-19855)
- SHELL32: Fonts folder shortcut: bind the font shell extension ([fix/shell32-fonts-folder-fontext](https://github.com/tomassrnka/reactos/commit/524f53e771c54432145d5057100651b7bfd0e2a2))
- SHELL32: Expand a default-user known folder value that starts with another variable ([fix/shell32-known-folder-defs](https://github.com/tomassrnka/reactos/commit/4f392b70c80e21a9bce7fcc18d84d70b4d1b4937))
- SHELL32: Implement SHGetKnownFolderPath and related functions ([fix/shell32-known-folder-path](https://github.com/tomassrnka/reactos/commit/3dcc19baa955b0e42c674e34a6982586e1019b2d))
- SHELL32: Make Yes the default of the Empty Recycle Bin question ([fix/shell32-recycle-confirm-default](https://github.com/tomassrnka/reactos/commit/bb582f86133556ef9e57fc6f9907eaddb11439d8))
- SHELL32: Honor CMIC_MASK_NOASYNC for Recycle Bin item properties ([fix/shell32-recycle-item-props](https://github.com/tomassrnka/reactos/commit/0e4bacb8262d56113e935a9b4743e877af07537c))
- SHELL32: Keep the icon and save the comment of a shortcut ([fix/shell32-shortcut-comment-icon](https://github.com/tomassrnka/reactos/commit/976378b860678d57e67456d2d7838a3e5f7d809f))
- SHELL32: Implement SHGetStockIconInfo and stop the NT 6 stubs from killing callers ([fix/shell32-vista-stubs](https://github.com/tomassrnka/reactos/commit/f64c9661ff0aa852bca62bf244ff53a40dd2e53e))
- SHELL32: Add SHGetPropertyStoreForWindow ([fix/shell32-window-property-store](https://github.com/tomassrnka/reactos/commit/1e64c43806f3836e56d6b3d803a04d609abdebc9))
- SHELL32: Do not set folder attributes when reading them failed ([fix/shellpath-attr-read-check](https://github.com/tomassrnka/reactos/commit/abe63f45ba6f98b8f360bbf0054bebbaf655147f))
- SHELL32/USRMGR: Run as on program shortcuts; enable Reset Password ([fix/shortcut-runas-reset-password-v2](https://github.com/tomassrnka/reactos/commit/4dc803a91336157d4947a697746c9d34b9baeb7c))
- URLMON: fix a memory-safety bug ([fix/browser-heap-corruption](https://github.com/tomassrnka/reactos/commit/78dc648bad2eab7eb3d5d8d3373f3382d6c24f7f))
- USER32: Keep icon identifiers apart from the handle array on 64-bit ([fix/pickicon-sparse-grid](https://github.com/tomassrnka/reactos/commit/1b7a175ec8d46c549293eecc254c3eb76f33568c))
- UXTHEME: Free the window contexts on uahStop, not on uahShutdown ([fix/uxtheme-defer-hook-shutdown-cleanup](https://github.com/tomassrnka/reactos/commit/81255dca8fd43023c7812736c3b343e5888b3fad))
- UXTHEME: Add BeginPanningFeedback, UpdatePanningFeedback and EndPanningFeedback ([fix/uxtheme-panning-feedback](https://github.com/tomassrnka/reactos/commit/dec7e1677609bbc0bce395124732ffe46a062e91))
- UXTHEME: Number the named functions as Windows 10 does ([fix/uxtheme-win10-ordinals](https://github.com/tomassrnka/reactos/commit/f192df9f4f2e2d7afc6cc2a84e1dd96e9d0a971f))

</details>

<details><summary>Setup and boot (24 fixes)</summary>

- BOOTDATA: Default "Show Control Panel in My Computer" to checked ([fix/folderopts-cpl-default](https://github.com/tomassrnka/reactos/commit/54d788b35991637e1a763ad99e544dacfc1c3f0c))
- BOOTDATA/KERNEL32_APITEST: Drop internal project names from comments ([fix/remove-internal-names](https://github.com/tomassrnka/reactos/commit/fddaeeeb42574801bf9e034c9af72f127c3281ec))
- DEVMGR: Report a failure to enable or disable a device ([fix/devmgr-disable-honest](https://github.com/tomassrnka/reactos/commit/6c08280a20a363c2597fb20736fc49f2748c3f46))
- DEVMGR: Give the focus to OK when the problem button loses it ([fix/devmgr-troubleshoot-button](https://github.com/tomassrnka/reactos/commit/29d0fb267d0a851b838b3e75595cb5daad26403e))
- FREELDR: Apply a hive log only to the interrupted write it belongs to ([fix/freeldr-hive-log-rules](https://github.com/tomassrnka/reactos/commit/4cd2c3bee221a4ceb2984f6f092bab6ee57d4954))
- FREELDR: Read hive log dirty blocks from after the dirty vector ([fix/freeldr-log-offset](https://github.com/tomassrnka/reactos/commit/c6cc7434638a2507f66558d8fa331ffd05e95151))
- FREELDR: Check the NTFS attribute list context allocation ([fix/freeldr-ntfs-attribute-extents](https://github.com/tomassrnka/reactos/commit/ddf5d247a53391230bff61ece6fc573a9b858855))
- FREELDR: fix a memory-safety bug ([fix/freeldr-stall-calibration](https://github.com/tomassrnka/reactos/commit/a8f56d53e7c3ee0f4ba259070c04e4c9528e1ec1))
- HDWWIZ: Fail the search when a device that has a name does not answer ([fix/hdwwiz-search-honest-v3](https://github.com/tomassrnka/reactos/commit/369699aa19c15874ab1bb1184dba12085fc1952e))
- RAPPS: Say once that a 32-bit installer cannot run on 64-bit without WoW64 ([fix/rapps-amd64-32bit-installer](https://github.com/tomassrnka/reactos/commit/966166d99131653b77d241d95aaeafc7664e2ed6))
- RAPPS: Skip packages whose dependency did not install ([fix/rapps-dependency-failure](https://github.com/tomassrnka/reactos/commit/c3e6f1c79b3c0edf645fc38c5030a9596d64b897))
- RAPPS: Retry a busy server and report the HTTP status ([fix/rapps-download-retry](https://github.com/tomassrnka/reactos/commit/e3d89a394ede8e12ae51215415ec4ea97ab1b947))
- RAPPS: Do not call SHGetKnownFolderPath on NT5 ([fix/rapps-generate-nt5](https://github.com/tomassrnka/reactos/commit/856b15254e6ffc56c2bffaf3168c74585b74cc15))
- RAPPS: Count a cancelled installer as not completed ([fix/rapps-installer-exit-code](https://github.com/tomassrnka/reactos/commit/9a03473d8ee0385bf898a1998a0391c4cb227295))
- RAPPS: Ask before running a download that is a console program ([fix/rapps-standalone-program](https://github.com/tomassrnka/reactos/commit/1019fddfaa14cf122748573554c234ac298bae32))
- REACTOS/BOOTDATA: Default to NTFS in the graphical installer too ([fix/guisetup-ntfs-default](https://github.com/tomassrnka/reactos/commit/d23861e424f5e045b260803aa93f834dc41c209e))
- SETUPAPI: fix a security-related bug ([fix/setupapi-device-property-v2](https://github.com/tomassrnka/reactos/commit/9135a160b628c31ea6f007363579e978c7cdc738))
- SETUPAPI: Implement the disk space list drive and INF section functions ([fix/setupapi-diskspace-sections](https://github.com/tomassrnka/reactos/commit/9645971598c8eaae1ff54bd8624c12c9054c9515))
- SETUPAPI: Implement SetupAddToDiskSpaceList and SetupQuerySpaceRequiredOnDrive ([fix/setupapi-diskspace-sync](https://github.com/tomassrnka/reactos/commit/c522fad70bc26b130de19eddd5b22a0c26d9dec1))
- SETUPAPI: Ask for a restart when a running device gets its driver again ([fix/setupapi-reinstall-running-device](https://github.com/tomassrnka/reactos/commit/d057ed442fdf782345fe7be7bc40e1109fc8b2bb))
- SETUPAPI: Do not ask for a restart after the first install of a device that keeps its drivers ([fix/setupapi-started-first-install](https://github.com/tomassrnka/reactos/commit/b944a87d8faa7516fb6f6cba5fa03e5acb3ef393))
- SYSDM: Show the small dump for a CrashDumpEnabled value the dialog does not know ([fix/sysdm-crashdump-values](https://github.com/tomassrnka/reactos/commit/be277623784ec99ef8c35aff3a1d3ed106ca4fa9))
- SYSDM: Grey out the Error Reporting button ([fix/sysdm-error-reporting-disabled](https://github.com/tomassrnka/reactos/commit/70573d89f357da63f9d49bbddf8e3f627c48f915))
- SYSDM: Implement the processor scheduling radios of Performance Options ([fix/sysdm-processor-scheduling](https://github.com/tomassrnka/reactos/commit/c52abed52647e4aefb97c70a1ee8f92e91515fd4))

</details>

<details><summary>Printing (6 fixes)</summary>

- COMDLG32/NOTEPAD: Report a printer device context that cannot be created ([fix/print-dc-failure-honest](https://github.com/tomassrnka/reactos/commit/ce0e343058ed60392f955cb17ceef74063603fe8))
- LOCALSPL: Schedule a job only through the handle that added it ([fix/localspl-addjob-other-handle-v2](https://github.com/tomassrnka/reactos/commit/cd0aa56fc4eec29df6233f50c76aeddeec13d08b))
- LOCALSPL: Keep every job at its own sorted place in the printer queue ([fix/localspl-delete-printer](https://github.com/tomassrnka/reactos/commit/ed076b39229e3c9638326f78a5040673c1c626cf))
- LOCALSPL/WINSPOOL: Print a document into the output file given to StartDocPrinter ([fix/localspl-print-to-file](https://github.com/tomassrnka/reactos/commit/77168743aa3164d52c73300174089da4b2a6de9a))
- SHELL32: Add a context menu for printers in the Printers folder ([fix/printers-folder-verbs](https://github.com/tomassrnka/reactos/commit/cedadb919efc5d7b6d698da601c882529d54de4f))
- WINSPOOL: AbortPrinter: keep a spooled document whose abort failed ([fix/printers-abortprinter-v3](https://github.com/tomassrnka/reactos/commit/a2a7b18ca2364c7b108791cb1bd7c47fb632bf19))

</details>

<details><summary>Services, security and profiles (22 fixes)</summary>

- ADVAPI32: Implement CloseTrace ([fix/advapi32-closetrace](https://github.com/tomassrnka/reactos/commit/ec804b17f9953a92b766836a1855a07ec01c11c4))
- ADVAPI32: Parse the S-R-I-S string of ConvertStringSidToSidW strictly ([fix/advapi32-convert-string-sid](https://github.com/tomassrnka/reactos/commit/9bb9005936e7f4f25b4672c3eeaf7e4200f30a2c)) — fixes [CORE-8717](https://jira.reactos.org/browse/CORE-8717)
- ADVAPI32: Fail EncryptFile instead of reporting success without encrypting ([fix/advapi32-encryptfile-honest](https://github.com/tomassrnka/reactos/commit/28eb46379e5730660776a315a809efe5ae710af8))
- ADVAPI32: Return the key class from RegEnumKeyExA ([fix/advapi32-enumkey-class](https://github.com/tomassrnka/reactos/commit/6e9774c0a4af6d4a70753af57e6f3eb2c7e7a495))
- ADVAPI32: fix a security-related bug ([fix/advapi32-equaldomainsid](https://github.com/tomassrnka/reactos/commit/4c04275525dc1845043fa1a235112e16a07d1541))
- ADVAPI32: fix a security-related bug ([fix/advapi32-regenumvalue-terminator](https://github.com/tomassrnka/reactos/commit/baa909277716f4d7ed941cb96dcb1633aed04b1c))
- ADVAPI32: fix a security-related bug ([fix/advapi32-reggetvalue-expand](https://github.com/tomassrnka/reactos/commit/e07c5b358855d4b886a839d9e729bf556d3573c3))
- ADVAPI32/ADVAPI32_APITEST: Export the event provider functions ([fix/advapi32-event-provider](https://github.com/tomassrnka/reactos/commit/e7fcfc3800b191c95b9539f8ce3232ef5b796dcf))
- ADVAPI32/ADVAPI32_APITEST: fix a security-related bug ([fix/sddl-rights-numeric](https://github.com/tomassrnka/reactos/commit/cb3ced5fdc24a7c14a1f079d839be480f7494862))
- LSASRV: Unlink the deregistered logon context, not its successor ([fix/lsa-deregister](https://github.com/tomassrnka/reactos/commit/f3e1aa617e65578f0a416483a27a2e620a08f2da))
- NTDLL/ADVAPI32: Add EtwEventSetInformation and EnableTraceEx2 ([fix/advapi32-missing-exports](https://github.com/tomassrnka/reactos/commit/c9a266ba16dfb6f6484b54167cf5346324ad0bd6))
- NTMARTA/ADVAPI32_APITEST: Honour ACE inheritance flags and inherit from the parent on unprotected DACLs ([fix/ntmarta-inherit-acl](https://github.com/tomassrnka/reactos/commit/9e0655f183f9bfce772c89a73f9890f247dee133))
- NTMARTA/ADVAPI32_APITEST: fix a security-related bug ([fix/ntmarta-setentries-oldacl](https://github.com/tomassrnka/reactos/commit/3c877f3af0c9b7755d82bd000f01339f4d55f9c9))
- SERVICES: Store the FailureActions value in the layout Windows uses ([fix/hivesys-recovery-defaults](https://github.com/tomassrnka/reactos/commit/fb22433006f43f240e98c5baf6e630153fc85bf4))
- SERVICES: Start every own-process service in a process of its own ([fix/scm-own-process-image-v2](https://github.com/tomassrnka/reactos/commit/e6a8c02b33259d860353d2e9ebfb283527f3f0c7))
- TELNETD: Clear the stop flag of a run after the shutdown flag ([fix/telnetd-stop](https://github.com/tomassrnka/reactos/commit/45c1061970877993e385a3a538f596e6909e261c))
- UMPNPMGR: Initialize the notification lock before starting threads ([fix/umpnpmgr-init-order](https://github.com/tomassrnka/reactos/commit/5f3430ad626cf47dd20493fdc909cba8ac6476ed))
- UMPNPMGR: Report no dock station when there is no docking information ([fix/umpnpmgr-no-dock-info](https://github.com/tomassrnka/reactos/commit/55314903197aa04deab83dcbf84280f6105d4471))
- USERENV: Remove directory trees through handles ([fix/userenv-deleteprofile-check](https://github.com/tomassrnka/reactos/commit/bc71ac61a515ae307d411b64925a3eceb7c9e577))
- USERENV: Create the legacy profile junctions on NTFS ([fix/userenv-legacy-profile-junctions](https://github.com/tomassrnka/reactos/commit/ad207d52732f16d534a5c9734d930f745e564082))
- USERENV: Add DeriveAppContainerSidFromAppContainerName ([fix/userenv-missing-exports](https://github.com/tomassrnka/reactos/commit/1fc4df69931e47827d8ba735b40092dbef271676))
- USERENV/SHELL32: Take upstream's Vista profile layout series as it is ([fix/upstream-users-profile-layout](https://github.com/tomassrnka/reactos/commit/2c5c6e27bae09219a62dca251e9fb88b8786f80c))

</details>

<details><summary>User-mode system DLLs (67 fixes)</summary>

- CMAKE: Regenerate sdk/cmake/baseaddress.cmake: the i386 image of kernel32 no longer fits below ntdll ([fix/kernel32-volume-mountpoint-r1015](https://github.com/tomassrnka/reactos/commit/fe236f8237f13f2867886b629ff3c166e7ffe60b))
- CRT: Do not give DLLs a TLS directory they do not need ([fix/dll-no-crt-tls-directory](https://github.com/tomassrnka/reactos/commit/d8804816b3a429774d1bc5a633a709703a85a1e8))
- KERNEL32: EnumUILanguages: list the installed UI languages only ([fix/enumuilanguages-installed](https://github.com/tomassrnka/reactos/commit/2f8e0a9bff28ac8c4e004bb94448f3a914b8461d))
- KERNEL32: Always release gcsAppCert after the application certification setup ([fix/kernel32-appcert-lock-race](https://github.com/tomassrnka/reactos/commit/ebfec408b94dd24d8f01873bf8419927424fa93e))
- KERNEL32: Application recovery registration; restart settings as measured on Windows ([fix/kernel32-application-recovery](https://github.com/tomassrnka/reactos/commit/9e9b4f561da6132d4ae2a283d8f6a538595e4cb7))
- KERNEL32: Implement application restart registration ([fix/kernel32-application-restart](https://github.com/tomassrnka/reactos/commit/632cc8034617b0b37c1363e672e32208f73efb34)) — fixes [CORE-16061](https://jira.reactos.org/browse/CORE-16061)
- KERNEL32: Implement CreateEventExA/W and CreateMutexExA/W ([fix/kernel32-create-event-mutex-ex](https://github.com/tomassrnka/reactos/commit/5c957f560d794d11c7f4f4f25486cb256afb808f))
- KERNEL32: fix a security-related bug ([fix/kernel32-createfile-sqos](https://github.com/tomassrnka/reactos/commit/a2fadb3ff7902345aee8990e422b29594da1582d))
- KERNEL32: Implement CreateRemoteThreadEx ([fix/kernel32-createremotethreadex](https://github.com/tomassrnka/reactos/commit/bbf2938ad7ce70d6fefbba21b22fde7ad4353db4))
- KERNEL32: Implement QueryIdleProcessorCycleTime, QueryProcessCycleTime and QueryThreadCycleTime ([fix/kernel32-cycle-time](https://github.com/tomassrnka/reactos/commit/6e58a31aa1bc23c471a993d05b5b58752d5c05bb))
- KERNEL32: GetDynamicTimeZoneInformation: empty key names, result of GetTimeZoneInformation ([fix/kernel32-dynamic-time-zone](https://github.com/tomassrnka/reactos/commit/30577d877dc7f5f2805a62e7e0551d48f5727e60))
- KERNEL32: Honour FILE_FLAG_FIRST_PIPE_INSTANCE in CreateNamedPipeW ([fix/kernel32-first-pipe-instance](https://github.com/tomassrnka/reactos/commit/6687a834ee527baca7a2e976ea7a121018ae4237))
- KERNEL32: Read an I64 FormatMessage argument as 64 bits on 32-bit systems ([fix/kernel32-formatmessage-int64](https://github.com/tomassrnka/reactos/commit/a96335568d79b05b0ee8277326c64ab540f7d2ac)) — fixes [CORE-17317](https://jira.reactos.org/browse/CORE-17317)
- KERNEL32: Report the binary type of an image as Windows does ([fix/kernel32-getbinarytype](https://github.com/tomassrnka/reactos/commit/071d816c830882ed1ce3974554b145aab0b7881f))
- KERNEL32: Implement GetLogicalProcessorInformationEx ([fix/kernel32-logical-processor-ex](https://github.com/tomassrnka/reactos/commit/6b368cd4a50bc155918509e7d24b1101db5026c1))
- KERNEL32: Accept the mitigation policy process attribute ([fix/kernel32-mitigation-policy-attribute](https://github.com/tomassrnka/reactos/commit/a6012db2b2193e7db45041c3c83f232bf2eb6775))
- KERNEL32: Refuse PROC_THREAD_ATTRIBUTE_HANDLE_LIST instead of emulating it ([fix/kernel32-proc-thread-attribute-list](https://github.com/tomassrnka/reactos/commit/5856f2c746d1bd91135f82656f404852ff257115))
- KERNEL32: Sync profile.c with Wine 11.19 and map INI files through IniFileMapping ([fix/kernel32-profile-sync](https://github.com/tomassrnka/reactos/commit/9b644b68fd6cd4cde766f50f9efd43dd874e9690)) — fixes [CORE-16273](https://jira.reactos.org/browse/CORE-16273)
- KERNEL32: Implement ResolveLocaleName ([fix/kernel32-resolvelocalename](https://github.com/tomassrnka/reactos/commit/3e7e6a20743232576e687809b2db4148cdf89215))
- KERNEL32: Implement SetEnvironmentStringsW ([fix/kernel32-setenvironmentstrings](https://github.com/tomassrnka/reactos/commit/c1d8b0170fff8c93d25c772aa6c5b7624e38c399))
- KERNEL32: Implement SetFileInformationByHandle ([fix/kernel32-setfileinformationbyhandle](https://github.com/tomassrnka/reactos/commit/5476c605eebefed0b657b0e421a516187590856c))
- KERNEL32: Implement SetThreadErrorMode and GetThreadErrorMode ([fix/kernel32-thread-error-mode](https://github.com/tomassrnka/reactos/commit/476b3569f4db7c053fa3320d0a24aba12ed0a5b4))
- KERNEL32: GetTimeZoneInformationForYear: do not read a year value of more than 44 bytes ([fix/kernel32-timezone-for-year](https://github.com/tomassrnka/reactos/commit/4bc508d10a607ef311acf3fd734ded619be54c79))
- KERNEL32: Export the token functions that Windows 7 exports ([fix/kernel32-token-forwarders](https://github.com/tomassrnka/reactos/commit/4b0645fc9fd881560309dffab3447b7b1dfdc249))
- KERNEL32: Add WerRegisterRuntimeExceptionModule and WerUnregisterRuntimeExceptionModule ([fix/kernel32-wer-runtime-exception-module](https://github.com/tomassrnka/reactos/commit/3919a3b6cda59c8f0ca4bb2b12eefee5772d0bd8))
- KERNEL32: Add GetMaximumProcessorGroupCount, GetProcessGroupAffinity and SetWaitableTimerEx ([fix/kernel32-win7-thread-processor](https://github.com/tomassrnka/reactos/commit/af41ff507ad67fe3544d3e595814360f26b818ca))
- KERNEL32: Take the parent UI languages from the locale data ([fix/preferred-ui-languages](https://github.com/tomassrnka/reactos/commit/29dfa9100e15c09dd51c95cb6388b41418f7be00))
- KERNEL32: Report fail fast exceptions like unhandled ones ([fix/raisefailfast-report](https://github.com/tomassrnka/reactos/commit/b4ad2a8faf5ee0bc961fd43529e68a5e062a48ff))
- KERNEL32/KERNEL32_APITEST: Do not zero the tail of WIN32_FIND_DATA in FindFirstFile/FindNextFile ([fix/findfirstfile-keep-tail](https://github.com/tomassrnka/reactos/commit/89940c947af3d1880a6feb8153c80050488394c4))
- KERNEL32/KERNEL32_APITEST: Share the process UI languages with kernel32_vista, test the full language list and processors above 31 ([fix/kernel32-missing-exports](https://github.com/tomassrnka/reactos/commit/3a8e80a936801f7275423d13978679d9d2a2d968))
- KERNEL32/KERNELBASE: Export WaitOnAddress and WakeByAddress* from kernelbase only ([fix/waitonaddress-kernelbase-only](https://github.com/tomassrnka/reactos/commit/d8c1e90568699adcc0b3ea29f8fa3b418430952a))
- KERNEL32_VISTA: Implement GetFileMUIInfo ([fix/kernel32-getfilemuiinfo](https://github.com/tomassrnka/reactos/commit/3df03b4f16e589eaa58acfd66d3d60334ec4d28c))
- KERNEL32_VISTA: Require a file handle for the absorbed priority hint class ([fix/p2-usermode-kernel32](https://github.com/tomassrnka/reactos/commit/ed4147866471311babb9cb2cbdf20a0f455ffdf1))
- KERNELBASE: fix a security-related bug ([fix/kernelbase-interrupt-time](https://github.com/tomassrnka/reactos/commit/b9a5f86893da916be39012b6cf1b2794324fb85b))
- KERNELBASE: PathCanonicalizeW: only treat a whole ".." segment as a parent reference ([fix/shlwapi-path-dotdot-prefix](https://github.com/tomassrnka/reactos/commit/2264a530e0fc1ef84956ed1c9f945407adbb4670)) — fixes [CORE-20129](https://jira.reactos.org/browse/CORE-20129)
- NTDLL: Dynamic TLS: guarded header reads, a failed thread frees its stack ([fix/ldr-dynamic-tls](https://github.com/tomassrnka/reactos/commit/b539ffb4b75aa1e1b23835233aa34f0ea5bcf7f5))
- NTDLL: Unlink the FLS data of an exiting thread before freeing it ([fix/ldr-fls-before-tls-v3](https://github.com/tomassrnka/reactos/commit/54178f6d7d2902ecc990ac584143b473f238b12a))
- NTDLL: Honor the loadFrom path of a manifest file element in DLL redirection ([fix/ntdll-actctx-loadfrom](https://github.com/tomassrnka/reactos/commit/53e2c8a0ee496debe89db83ff5ae52e794d8a368)) — fixes [CORE-19707](https://jira.reactos.org/browse/CORE-19707)
- NTDLL: RtlQueryPerformanceFrequency always writes its output; relax the time test ([fix/qpc-unbiased-interrupt-time](https://github.com/tomassrnka/reactos/commit/766e6db02f95c8661c5b94a542b122499ca84c75))
- NTDLL: Heap walk: the ranges of a segment must be in ascending order and inside it, so a damaged list ends the walk ([fix/rtl-walk-heap](https://github.com/tomassrnka/reactos/commit/c56c49b05883800046399da8057b9edb82e03fff))
- NTDLL/ADVAPI32: fix a security-related bug ([fix/advapi32-add-mandatory-ace](https://github.com/tomassrnka/reactos/commit/929225ff09bbdb998800c4bcc62fece7213b5781))
- NTDLL/ADVAPI32: Implement the manifest-based event provider functions ([fix/etw-event-provider](https://github.com/tomassrnka/reactos/commit/f547ebc518fd00ac4e8b4e3043d16a4c44665fd1))
- NTDLL/KERNEL32: Restore the fiber checks inverted by the NT5-isms change ([fix/msvcrt-thread-detach-free](https://github.com/tomassrnka/reactos/commit/d71feff634d5d6d592adbf11b4d8e65b11e4ad35))
- NTDLL/KERNEL32: List the heaps of a process for the Toolhelp heap functions ([fix/toolhelp-heaps](https://github.com/tomassrnka/reactos/commit/cbb0effeb2e31732187fa26ca7625da71453ff14))
- NTDLL/NTDLL_APITEST: Report only the API set contracts the loader resolves, keep the shared UTF-8 decoder as it was ([fix/ntdll-missing-exports](https://github.com/tomassrnka/reactos/commit/d715d0f379c47e0ef31a8a3f3a4fc9f77dec7430))
- PSEH: Do not let a call before an x64 GCC try block return into it ([fix/x64-gcc-pseh](https://github.com/tomassrnka/reactos/commit/31d8fba88a49b0f102b188531158e7805cab3877))
- RTL: Do not lose condition variable wake-ups ([fix/rtl-condvar-lost-wakeups](https://github.com/tomassrnka/reactos/commit/dac3a7287a64f24773f96dcdb0ab8b428760d50f))
- RTL: Claim static critical section debug entries atomically ([fix/rtl-critsect-debuginfo-race](https://github.com/tomassrnka/reactos/commit/a3b056366437e59a14bf85254ee1d235ff9475b7))
- RTL: Copy environment variable values whole or not at all in RtlExpandEnvironmentStrings ([fix/rtl-expandenv-small-buffer](https://github.com/tomassrnka/reactos/commit/e0450ae0550c6b84e4db375648db42b5ad3612ae))
- RTL: Fix the cached walk in RtlGetElementGenericTable ([fix/rtl-generictable-walk](https://github.com/tomassrnka/reactos/commit/8463844c8667f974dac8d4eb7b925d672617a291))
- RTL: Do not commit past the end of a handle table reservation ([fix/rtl-handle-table-end](https://github.com/tomassrnka/reactos/commit/cb7ca8ea6316025b274a0a133f52610686c11dc2))
- RTL: Fill the whole tail pattern on 64-bit ([fix/rtl-heap-tail-pattern](https://github.com/tomassrnka/reactos/commit/0acb926f2639d71d737e14170eeb41e8131f856e))
- RTL: Answer RtlIsRangeAvailable for a window with Start above End ([fix/rtl-range-available-reversed](https://github.com/tomassrnka/reactos/commit/dd6742567c840bed31952921d4f0991425fa3fb7))
- RTL: fix a security-related bug ([fix/rtl-remote-modules-bound](https://github.com/tomassrnka/reactos/commit/6b332ccfa6a48f8633dde1dd56a0ea0f8a5d89b9))
- RTL: Block SRW lock waiters on a keyed event instead of sleeping in ticks ([fix/rtl-srw-backoff-convoy](https://github.com/tomassrnka/reactos/commit/3f120eca95f9fb81ed88837adde147f5e8fa99c1))
- RTL: Let SRW lock waiters give the processor away ([fix/rtl-srw-wait-backoff](https://github.com/tomassrnka/reactos/commit/2c0fc8da3ab60d83aaa142d5d05e7cd55f079fbe))
- RTL: Take the SRW wait block lock only on a lock word that has wait blocks ([fix/rtl-srw-wait-block-lifetime](https://github.com/tomassrnka/reactos/commit/45258a10b8666c8cf97e9a6b82fb1640819f6b9f))
- RTL: Do not start a wait callback after the wait was cancelled ([fix/rtl-wait-deregister](https://github.com/tomassrnka/reactos/commit/423a581b9a2a4095bfc45f6ea251c7800e6d57c0))
- RTL: Fix the shared count encoding in RtlTryAcquireSRWLockShared ([fix/srw-try-shared](https://github.com/tomassrnka/reactos/commit/52979b8c3b7c8f85d0d87793247693bc89cfbde3))
- RTL/NTDLL: Implement RtlAddGrowableFunctionTable, RtlGrowFunctionTable and RtlDeleteGrowableFunctionTable ([fix/ntdll-growable-function-tables](https://github.com/tomassrnka/reactos/commit/7850c61328b5806dfd6e223f82e7244dde06cd77))
- RTL/x64: Set EXCEPTION_TARGET_UNWIND after a collided unwind ([fix/rtl-amd64-collided-unwind](https://github.com/tomassrnka/reactos/commit/1290624eb8c02cb5523ab6e9bc7937b5ff396931))
- SDK/KERNEL32: Use the Windows layouts of the Toolhelp structures ([fix/toolhelp-x64-structs](https://github.com/tomassrnka/reactos/commit/9a33b3af64d0e605cd4e0ec600238196b24d1d95))
- SDK:RTL: Fail a NULL string default in RtlQueryRegistryValues ([fix/rtl-query-registry-default](https://github.com/tomassrnka/reactos/commit/056dfc4b60d6d0fa3ddff900b227b4347ef94bbe))
- UCRTBASE: Raise the exceptions of the C99 math functions ([fix/ucrtbase-c99-math](https://github.com/tomassrnka/reactos/commit/0ef691b3bc72d3578a9f8f4ac0fc8dd1370c965f))
- UCRTBASE: Load the x87 control word through the environment image ([fix/ucrtbase-fenv](https://github.com/tomassrnka/reactos/commit/faed9f5f2128561bcfdb00439118e982e3768574))
- UCRTBASE: lgamma(0) raises divide-by-zero; the _o_ gamma and logb variants leave errno alone ([fix/ucrtbase-gamma-logb](https://github.com/tomassrnka/reactos/commit/cfab89b8d75b10648d4da589b907078393e53a91))
- UCRTBASE: Report the math errors of the legacy functions like Windows ([fix/ucrtbase-math-errno](https://github.com/tomassrnka/reactos/commit/e8d8502ab207f1281855253f7a02f8210786c834))

</details>

<details><summary>Browser, COM and Wine imports (14 fixes)</summary>

- APISETS: Resolve the WinRT apisets to combase ([fix/apisets-winrt-combase](https://github.com/tomassrnka/reactos/commit/5d2d3b66f0a4cde0eb29aaa2a68bb1827d4d8407))
- CRYPTBASE: Add cryptbase.dll forwarder DLL ([fix/cryptbase-forwarder](https://github.com/tomassrnka/reactos/commit/ed7e480a123842e879f066d018f1690ddf7227c3))
- IFMON: Show the lease of a DHCP adapter in "show config" ([fix/ifmon-show-dhcp-config](https://github.com/tomassrnka/reactos/commit/8c740f01d7eff86f308c0bec93518fc9d4cd8713))
- KTMW32/NTDLL: Add ktmw32 and fail transactions cleanly ([fix/ktmw32-transactions](https://github.com/tomassrnka/reactos/commit/83fb6fec74596288346c30904b8c4b27dd369962))
- MSHTML: Read the raw HTTP headers as the ANSI text they are ([fix/mshtml-http-headers-ansi](https://github.com/tomassrnka/reactos/commit/e18fda464e0d89d4fd23cdb635d51f378ace5b9f))
- MSI: Give a package with ALLUSERS the all-users Start Menu and Desktop folders ([fix/msi-allusers-folders](https://github.com/tomassrnka/reactos/commit/e8b4ed38f2a539cc558592269f8ef21941dbd590))
- MSI: Publish the first feature to the texts that follow a SelectionTree ([fix/msi-seltree-initial-events](https://github.com/tomassrnka/reactos/commit/98e789b488c183ebd2247c8a6b8da788afa1b8c9))
- MSI/MSIEXEC: Log why a package was refused before its log opened ([fix/msi-early-failure-log-v2](https://github.com/tomassrnka/reactos/commit/f2ad212695e51bcaf196c865dbb1ffb89f88a753))
- NETID: Disable the Network ID button ([fix/netid-network-id-disabled](https://github.com/tomassrnka/reactos/commit/3d0bceb827a0e12ffd13e7066adc5b6ad58c6acd))
- PSAPI/PSAPI_APITEST: Read the lists of a WOW64 process again when they grow, signal the test child from the child ([fix/psapi-missing-exports](https://github.com/tomassrnka/reactos/commit/5784e0ef523d0369bbbaba549daa4d65ca0b4683))
- SHCORE/APISETS: Add GetFeatureVariant, RecordFeatureUsage, RecordFeatureError and UnsubscribeFeatureStateChangeNotification ([fix/shcore-missing-exports](https://github.com/tomassrnka/reactos/commit/721903e0b7470ddd28d40ffeae0b752835efa639))
- TDH/SDK: Import tdh.dll from Wine-11.19 ([fix/tdh-import](https://github.com/tomassrnka/reactos/commit/d926ceed6230ab36a49ad5ede0120607d675c83c))
- UIAUTOMATIONCORE: Add UiaDisconnectAllProviders ([fix/uiautomationcore-missing-exports-v2](https://github.com/tomassrnka/reactos/commit/9ede0086b67defee8dc138b4fad4ba4c0b41b249))
- URLMON: Run protocol tasks that were queued during a client Read ([fix/urlmon-read-task-queue-v2](https://github.com/tomassrnka/reactos/commit/a29b8321636bc32727a587d4de188a481f0bcef5))

</details>

<details><summary>Applications and tools (33 fixes)</summary>

- ACCESS: Fill the High Contrast color scheme list ([fix/access-high-contrast-schemes](https://github.com/tomassrnka/reactos/commit/63fd56cc41c87b0aca9974d24ab976b7f021306e))
- ACGENRAL: Import uxtheme only when a shim uses it ([fix/msvcrt-lock-recursion](https://github.com/tomassrnka/reactos/commit/4c76ce831d34ca5156e50d197e56da0ca174209b))
- APPWIZ/SYSSETUP: Provide Wine Gecko on amd64 ([fix/gecko-amd64](https://github.com/tomassrnka/reactos/commit/46307bf334cfccbde4256f05e7b58c08f73e5468))
- AT: Take the rest of the command line as the command ([fix/at-command-rest-of-line](https://github.com/tomassrnka/reactos/commit/6968d144c1fd716f27fe9f8a680342ce5301a683))
- CACLS: Refuse volumes without ACLs and report errors ([fix/cacls-fat-no-acls](https://github.com/tomassrnka/reactos/commit/e04c672cb6bb86d957198a53db561ca322765f3b))
- CHARMAP: Implement Search in the advanced view ([fix/charmap-search](https://github.com/tomassrnka/reactos/commit/56f44646a65550f33669e10fc56ba4847c23ca0d))
- CLEANMGR: Do not ask to delete files when nothing is ticked ([fix/cleanmgr-nothing-selected](https://github.com/tomassrnka/reactos/commit/38e2895d22f61df9982a9f34e4ba91c00c07d1ab))
- DISKMGMT: Shrink Volume shrinks the partition in place ([fix/dmshrink-dm-grow-r1015b](https://github.com/tomassrnka/reactos/commit/2e58894df95e0b7fcb1bfad10e9047e761b179b2))
- DISKPART: Clear the volume focus on create, keep numbers across all reads ([fix/diskpart-create-selects-volume](https://github.com/tomassrnka/reactos/commit/796b99bcb4ba607afef934df41c5ee562f30d6a6))
- DISKPART: Do not wait for a volume after resizing a partition without one ([fix/p2-usermode-cli](https://github.com/tomassrnka/reactos/commit/e6733a6857c725a0b921f2e14691bb84bbf7f77a))
- FIND: Never match an empty search string ([fix/find-empty-string](https://github.com/tomassrnka/reactos/commit/296151085b3e95812f86aaf67e0f6a2d25bf2a1d))
- FINDSTR: Scan only new input for line ends and report standard input read errors ([fix/findstr](https://github.com/tomassrnka/reactos/commit/2fe3b3380bcd457b088dcf6bf1e0ab0422f03e0c))
- JOY: Disable the Troubleshoot button ([fix/joy-troubleshoot-disabled](https://github.com/tomassrnka/reactos/commit/fb01838d7f518d045c796db681b38c36739234dd))
- KERNEL32/NTMARTA: fix a security-related bug ([fix/usermode-security-cluster](https://github.com/tomassrnka/reactos/commit/8d80b221ffa46d32cbe0371047cef8c07b13f39b))
- MAGNIFY: Disable the Help button, there is no help file ([fix/help-unavailable-disabled](https://github.com/tomassrnka/reactos/commit/5a03092962cdd75111e756a8604d5c4da784d3ec)) — fixes [CORE-20067](https://jira.reactos.org/browse/CORE-20067)
- MSCONFIG: Disable the startup selection that cannot be saved ([fix/msconfig-general-honest](https://github.com/tomassrnka/reactos/commit/f585dde006332f39cb379c5793d53d609e43fb2f))
- MSI/MSIEXEC: Report a package that cannot be opened as Windows does ([fix/msi-open-failure-codes](https://github.com/tomassrnka/reactos/commit/106fbcdf6c7b5c716d2ac08ac87d9ee297dc86e1))
- NOTEPAD: Warn before saving Unicode text as ANSI ([fix/notepad-ansi-save-warning](https://github.com/tomassrnka/reactos/commit/b6c4742886fbf175fe7e4aceb7faa8fde8e0117f))
- NOTEPAD: Make Replace All linear and undoable in one step ([fix/notepad-replace-all-linear](https://github.com/tomassrnka/reactos/commit/17d264ab5f802ed8d3eb19dcf8fd096d4917d188))
- REGEDIT: Pass the key path to the permissions dialog ([fix/regedit-permissions](https://github.com/tomassrnka/reactos/commit/31278d89eddbe5b1a9a1f3375dd38fbe8adcc75e))
- SCHEDSVC: Check stored schedules, arm the timer in UTC of the start date, lock at start ([fix/at-hang](https://github.com/tomassrnka/reactos/commit/c28c72a3d7c0e808c14f9e891a578eee139d0620))
- SCHTASKS: Report that the Task Scheduler is not available instead of doing nothing ([fix/schtasks-honest](https://github.com/tomassrnka/reactos/commit/123341247be5be783e6bed00697c6b53f61e0ac4))
- SERVMAN: Show the pending and paused service states ([fix/servman-pending-status](https://github.com/tomassrnka/reactos/commit/34ecb64433e275aa28e826afaf46ed01fffa135d))
- SERVMAN: fix a security-related bug ([fix/servman-recovery-save](https://github.com/tomassrnka/reactos/commit/b7be8dfcb79e844049455323d63a354f8f0cb678))
- SORT: Sort the named file and take the Windows options ([fix/sort-file-argument-v2](https://github.com/tomassrnka/reactos/commit/63cf20763a07a700317eb06090fd68cebfede961))
- TASKLIST: Accept the largest 64-bit filter value ([fix/tasklist-switches-v3](https://github.com/tomassrnka/reactos/commit/4467c08999ac00edaeda84bc5940ad4d318952dc))
- TASKMGR: Do not close on Enter ([fix/taskmgr-enter](https://github.com/tomassrnka/reactos/commit/015aab0b0d0c04e30caf28483a5c1d47c98eb083))
- TASKMGR: Show one CPU usage history graph per processor ([fix/taskmgr-per-cpu-graphs](https://github.com/tomassrnka/reactos/commit/049839771a03e3c57e49dd237615acb6dd3030b9))
- TELEPHON: Say that Phone and Modem Options is not available ([fix/telephon-not-available](https://github.com/tomassrnka/reactos/commit/b8a7abedfd969f0e07862669c39e5ca2dc255ef8))
- TELNET: Install telnet.ini and telnet.cfg ([fix/telnet-install-config](https://github.com/tomassrnka/reactos/commit/b5e19db1e584c47709cc4a7b25b1e65b1ead7206)) — fixes [CORE-12498](https://jira.reactos.org/browse/CORE-12498)
- TIMEOUT: End the wait only on a key press, not on a key release ([fix/timeout-wait](https://github.com/tomassrnka/reactos/commit/0e95215b82c61dc1769d86996d174694b4fd2d04))
- USER32: IsDialogMessageW: Stop the mnemonic search after one full pass ([fix/user32-dialog-accel-loop](https://github.com/tomassrnka/reactos/commit/ae102eefbd17144f8697093eed28a3aca34b7727))
- WORDPAD: Say that printing is not available instead of blank pages ([fix/wordpad-print-honest](https://github.com/tomassrnka/reactos/commit/af1dbff8c6d67cc6e938c5b02b9447c3ec3391cf))

</details>

<details><summary>Build, toolchain and API surface (10 fixes)</summary>

- APISETS: fix a security-related bug ([fix/apisets-real-hosts](https://github.com/tomassrnka/reactos/commit/b229be0aa30a1833363d455f6adc203ed950df35))
- APISETS: Host the shcore contracts that shcore implements in shcore.dll ([fix/apisets-shcore-hosts](https://github.com/tomassrnka/reactos/commit/b4d75640680f176f52ca9db5cb1bbedfea07acc2))
- APISETS/NTDLL: Ignore the minor version and the manifest when resolving API sets ([fix/apisets-minor-manifest](https://github.com/tomassrnka/reactos/commit/3ad4c066e85b6ac494e793c5791f594fc8c35825))
- include: About: keep the upstream version lines and fit the Nitro lines ([fix/nitro-version-string](https://github.com/tomassrnka/reactos/commit/75a6134fb6aa46ad3625efae9c13880d7cf90bbb))
- PSDK/CABINET_APITEST: Do not pack FDINOTIFICATION to 4 bytes on x64 ([fix/cabinet-fdi-x64-layout](https://github.com/tomassrnka/reactos/commit/2a6e7f5b31bef3c04a393d458c5f06adf736af9b))
- RTL/KERNEL32: Implement RtlFormatMessageEx and format messages through it ([fix/formatmessage-rtl-v2](https://github.com/tomassrnka/reactos/commit/9edb16a53513c90c1ed2b2e56517118fb7670f5b))
- RTL/TZLIB: Identify the current time zone by its key name alone ([fix/auto-dst-disabled](https://github.com/tomassrnka/reactos/commit/67694f3c2b21eafce9b6c68702b53914ce61ec03))
- SDK: Name the revision only from a build tag ([fix/version-revision-date-tag](https://github.com/tomassrnka/reactos/commit/b814861d5826e502b904b750df456046a1d78e0b))
- SDK:MC: Print whole addresses in the hard error texts on 64-bit ([fix/hard-error-address-amd64](https://github.com/tomassrnka/reactos/commit/d61f0afecf3951e52fa740c3b4761e59ae3bf312))
- VCRUNTIME: Fix _AddressOfReturnAddress for x64 GCC ([fix/x64-gcc-return-address](https://github.com/tomassrnka/reactos/commit/2aa7d91b4dcf0b8f081681c4ef361885f4e79374))

</details>

<details><summary>Tests only (26 fixes)</summary>

- ADVAPI32_APITEST: tests for a security-related fix [test-only] ([fix/logon-user-processes](https://github.com/tomassrnka/reactos/commit/1f5ab40693d094654ee904f131bca926fdff82cd))
- APITESTS: Link WoW64 services in static RTL tests [test-only] [pick] ([main/wow64-rtl-unittest-link](https://github.com/tomassrnka/reactos/commit/7426aa673f03210c7d54b155b8d4ee380e14c6bc))
- CRT_APITEST: strlen: clear the direction flag before calling ok() [test-only] ([fix/rosautotest-pipe-nul-m](https://github.com/tomassrnka/reactos/commit/27a1ad93210ba2a9fd7d00c95ae9d186f6eeb0ea))
- GDI32_APITEST: tests for a security-related fix [test-only] ([fix/d3dkmt-dcmem-probe](https://github.com/tomassrnka/reactos/commit/66bdb43cacd68820b0ece7d7422d9f506c850cc4))
- GDI32_WINETEST: Skip the font name checks for FontSubstitutes aliases [test-only] ([fix/gdi32-font-alias-families](https://github.com/tomassrnka/reactos/commit/957f72a208e45f2e05f06529dee6fb27f27d99ee))
- KERNEL32_APITEST: Expect the NTSTATUS in the BindIoCompletionCallback error code [test-only] ([fix/rtl-iocompletion-callback](https://github.com/tomassrnka/reactos/commit/b02e5a87455b7c2342ea00178375634d203cae20))
- KMTESTS:CC: Run the CcLazyWriteDelete test on ReactOS only [test-only] ([fix/cc-lazywrite-dirty-list](https://github.com/tomassrnka/reactos/commit/91f2c6249db853396d8040fe88c34593138e3615))
- KMTESTS:CC: Test concurrent exclusive pins in CcPinRead [test-only] ([fix/cc-pin-lost-race](https://github.com/tomassrnka/reactos/commit/3097d087d36e4c9d419d966c887cdb5266608a90))
- KMTESTS:CC: Do not pass NULL for required CcCopyRead parameters [test-only] ([fix/kmtest-cccopyread-contract](https://github.com/tomassrnka/reactos/commit/484b23f8e0669c582bccdae06214e95c0cb7b247))
- KMTESTS:IO: Call IoFastQueryNetworkAttributes from a system thread [test-only] ([fix/io-deletefile-handle-leak](https://github.com/tomassrnka/reactos/commit/0887a658f96d41351d0a2270c2946c0c9c47c194))
- MSI_WINETEST: Restore the end of test_selectiontree_events [test-only] [pick] ([main/msi-seltree-merge-fix](https://github.com/tomassrnka/reactos/commit/b6a4289b8b292596c158d52c5cc24ce280deffb2))
- NTDLL_APITEST: Do not expect a missed change for a watcher without a buffer [test-only] ([fix/fsrtl-notify-delete](https://github.com/tomassrnka/reactos/commit/eb8bb780232b2e6f2e5d27792738d55e3aaf7ceb))
- NTDLL_APITEST: tests for a security-related fix [test-only] ([fix/io-write-mdl-status](https://github.com/tomassrnka/reactos/commit/9a8182da2fc4699dd83dd976a56af8058351d610))
- NTDLL_APITEST: BadSegmentFault: accept the faulting instruction as the address [test-only] ([fix/ke-amd64-badss-record-leak](https://github.com/tomassrnka/reactos/commit/0ca6e3ef5b0609b23db6bd1f1b6c54570fb8e0cb))
- NTDLL_APITEST: tests for a security-related fix [test-only] ([fix/timezone-privilege](https://github.com/tomassrnka/reactos/commit/bca92924137c9aaffea0c7c89691bbd028aa263e))
- NTFS_APITEST: Expect what Windows does on the empty-name junction reopen [test-only] ([fix/ntfsng-relopen-junction-acl](https://github.com/tomassrnka/reactos/commit/23614592a13c90e022c1d13f18af787051d25fd9))
- ROSAUTOTEST: Keep the --list for a module pattern that ends in "test" [test-only] ([fix/rosautotest-named-test](https://github.com/tomassrnka/reactos/commit/3aa9063d912a8970cc74089e0df617a7e2726626))
- ROSAUTOTEST: Do not lose the rest of a run to a pipe left open by a test [test-only] ([fix/rosautotest-unique-pipe](https://github.com/tomassrnka/reactos/commit/9ebb183b5479b072e3b06356e382f02ed71cde0f))
- SETUPAPI_APITEST: tests for a security-related fix [test-only] ([fix/pnproot-double-free](https://github.com/tomassrnka/reactos/commit/7814ea2076dfdddd3c3c5aaf0406b8a488645fdc))
- stackoverflow-amd64-frame: tests for a security-related fix [test-only] ([fix/stackoverflow-amd64-frame](https://github.com/tomassrnka/reactos/commit/d3855aa29e0afa2af91be4cbafc76cddabc496f8))
- stackoverflowthread-t1-timeout: tests for a security-related fix [test-only] ([fix/stackoverflowthread-t1-timeout](https://github.com/tomassrnka/reactos/commit/a9524479ba1002b57379c4559b6b52a2dec8f475))
- USERENV_APITEST: Delete read-only, hidden and system entries in the DeleteProfile test [test-only] ([fix/userenv-deleteprofile-test-attrs](https://github.com/tomassrnka/reactos/commit/44a0a189344bafb96d8f47787be4e2d9adec5887))
- USERENV_APITEST: DeleteProfile: use todo_if(is_reactos()) for the loaded-hive check [test-only] ([fix/userenv-deleteprofile-todo](https://github.com/tomassrnka/reactos/commit/12a4463321e093a864ffc3285017d3a6d507d098))
- WIN32U_APITEST: tests for a security-related fix [test-only] ([fix/dxg-createsurface-probe](https://github.com/tomassrnka/reactos/commit/b3aadc547daf08981de5d91c4ec916ab02471798))
- WOW64_APITEST: Give the win32k test stubs a native i386 path [test-only] ([fix/wow64-test-stubs-native-gate](https://github.com/tomassrnka/reactos/commit/cc6d95668f6aef81f142cf0f6d78efc51a52aa86))
- WS2_32_WINETEST: Query the completion notification modes before closing the socket [test-only] ([fix/ws2-winetest-iocn-query](https://github.com/tomassrnka/reactos/commit/7ecaa32d7b71dc142c2e36c53a736b8bc18d4b28))

</details>


### Limitations

- Firefox sandbox: on 64-bit it starts at default levels but restrictions are not enforced (a test that tries to create files in 5 places: all denied on Windows 10, all allowed here). Do not treat the build as sandboxed. 32-bit Firefox runs with the sandbox off.
- Firefox ESR 140 and later do not fully run yet: `cryptbase.dll` is now provided, but they still need `QueryUnbiasedInterruptTimePrecise`, which ReactOS does not implement. ESR 128 is the tested version. The Firefox installers (NSIS) only run with the Windows 10 compatibility layer set.
- Known flaky areas: 32-bit Firefox https tab sometimes never loads (2 of about 8 runs); 64-bit Firefox parent spun at 95.7% CPU in 1 of 8 runs; about 3 of 45 boots hang in ipconfig before the network is up; the full test run shows 16 regressions against upstream, no bugchecks.
- WoW64 is amd64 only and opt-in; some 32-bit system calls are not thunked yet (about 250 debug messages per Firefox run), some console calls from 32-bit console programs fail, CreateProfile fails in 32-bit processes, and the 32-bit Total Commander installers use `Program Files`.
- NTFS: no repair tool beyond journal replay; a junction in the middle of a path returns an error; persistent-ACL support is not reported.
- SMP: more processors do not always mean more throughput (8 versus 4 vCPUs was within noise on Intel and not measurable on AMD hosts); cached parallel reads fall as vCPUs rise; i386 on AMD hosts is far slower than amd64 because every interrupt-priority change exits to the host.
- Display: the shadow framebuffer is one paged-pool block (about 33 MB at 4K); 3200x2400 can break into the kernel; 8 bpp and real VESA hardware are untested.
- Memory: private memory is not paged out, so heavy memory pressure can stall instead of failing cleanly.
- Btrfs: volumes spanning several disks (RAID, multi-device) are not mounted; Check Disk is not supported.
- Installing onto Btrfs from the graphical setup hangs or fails.
- Some reported kernel issues are still under investigation.

### Changes since the previous Nitro

This is the first Nitro release, so there is no previous Nitro to compare with. The complete list of what it contains is the Bug fixes section above (one line per fix branch) together with the tracks. For context, relative to the last internal changelog (main `44e1aa62b1d`, 2026-10-18):
- The upstream base moved from `6ba28adc72c` to `ac1ab9e3043` (two merges, not rebases); upstream's own C:\Users layout was adopted.
- Main moved on by six integration batches, to `23dd8f7e336`.
- The first patch series has 47 topics; a branding patch set is kept aside, and reactos#9618 is merged upstream and no longer counts as a fork change.
- Six experimental branches (an NTFS check tool, several Windows 6 API sets, a software OpenGL) were closed without merging.
