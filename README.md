# reactos-nitro

**reactos-nitro is an unofficial patch set for [ReactOS](https://reactos.org).** It is not affiliated with or endorsed by the ReactOS project. ReactOS is a trademark of the ReactOS project.

> **Do not report problems from this build to the ReactOS project.** Report them here, in this repository's issues.
>
> Like ReactOS itself, this is alpha-quality software. Don't use it on a machine or disk holding data you care about.

**Last synced with ReactOS master:** [`e46f4bb1a1e`](https://github.com/reactos/reactos/commit/e46f4bb1a1ea7c722f941231565144fe0c286d3e) (2026-10-25, sync #4). We merge upstream regularly; this line tracks the most recent merge. (`nitro-2026.10-5` is based on this commit; `nitro-2026.10-2` to `-4` on `f53b1d15601`; `nitro-2026.10-1` on the earlier [`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043).)

**Docs:** [CHANGELOG](CHANGELOG.md) · [BENCHMARKS](BENCHMARKS.md) · [UPSTREAM](UPSTREAM.md)

reactos-nitro is a large set of fixes and features maintained on top of upstream ReactOS master. Each release ships:

- **the patch series**: one patch per topic, plus the exact upstream ReactOS commit it applies to;
- **build scripts** that apply the series to a clean upstream checkout and build the ISOs;
- **ready-to-install ISOs** for amd64 (with WoW64) and i386;
- **the full source tree** of the release, tagged in the [development fork](https://github.com/tomassrnka/reactos).

See [UPSTREAM.md](UPSTREAM.md) for how reactos-nitro relates to upstream ReactOS and why our NTFS is a separate driver.

## Goals

- **Usable for daily work:** real applications installed and running reliably, not only demos.
- **Fast:** responsive on modern multi-core machines and virtual machines.
- **Secure:** fix the security problems we find, and say plainly where protection is still missing.
- **Actively developed:** frequent releases, each one tested and documented.
- **In sync with upstream ReactOS:** track ReactOS master closely and merge it regularly. Keep our changes small and self-contained where possible, so the patch set stays easy to rebase and our fixes are easy to port.

## Status

Latest build: **`reactos-nitro-2026.10-5`** (2026-10-29), on upstream base `e46f4bb1a1e` — the modern-app release: Windows 10 is the default reported identity; Chrome (software rendering) and Claude Desktop run; 32-bit Firefox renders at the default sandbox again (32-bit + 64-bit side by side); Firefox 158 and PowerShell 7 run; plus Task Scheduler 2.0, a guest-clock fix and a memory commit cap. Each release ships an amd64-with-WoW64 ISO, an i386 ISO, a source tarball, and the patch series; see [CHANGELOG.md](CHANGELOG.md) for per-release detail and known limitations.

## Highlights

Over 750 fixes and several feature tracks on top of upstream ReactOS:

- **32-bit Windows programs on 64-bit ReactOS (WoW64).** Total Commander runs 32-bit and 64-bit side by side in one session, and Firefox runs 32-bit and 64-bit side by side at its default sandbox setting. Upstream ReactOS has no WoW64.
- **Modern apps.** Windows 10 identity by default; Chrome (software rendering) and Claude Desktop run; Firefox 158; PowerShell 7; Task Scheduler 2.0; an Optional Features manager.
- **NTFS read/write**, with a new driver built on the Linux kernel's `fs/ntfs` (imported from Linux v7.3-rc6), plus journal and power-cut recovery. NTFS is the default file system for new installs.
- **Btrfs**: WinBtrfs 1.10 (upstream ships 1.8.1).
- **Multi-processor support on x86 and amd64**, installed by default on ACPI machines. It is tested from 1 to 16 vCPUs, with multi-hour soaks on AMD and Intel hosts.
- **Smoother desktop.** A shadow framebuffer makes window dragging smooth at a fraction of the CPU cost. USB tablet polling and pointer fixes also help.
- **Networking**: a reworked receive path and dozens of TCP/IP, AFD and Winsock fixes.
- **Setup and user profiles**: the setup program preselects NTFS. The C:\Users layout includes Windows-style legacy folder links, and profile deletion works.
- **Kernel and memory manager hardening**: commit charging with a commit limit, plus many Mm, cache-manager and scheduler fixes.
- **Desktop and tools**: LCD text rendering, bundled open fonts, crash reports with a Windows-style dialog, and Disk Management.

A preliminary [BENCHMARKS.md](BENCHMARKS.md) (draft) compares reactos-nitro against upstream ReactOS.

A full list of changes, one line per fix, is in [CHANGELOG.md](CHANGELOG.md) .

## Known limitations

- WoW64 is amd64 only.
- **Firefox sandbox.** On 64-bit, the sandbox process starts but its restrictions are **not** fully enforced on ReactOS. 32-bit Firefox renders at the default sandbox setting.
- **Software rendering only.** Chrome runs with software compositing (no DirectComposition, no hardware GPU acceleration).
- **32-bit MFC applications** fail at their first window (WoW64 global-hook gap).
- **Memory commit is capped at about physical RAM** until private memory is pageable; Firefox 128 x64 thrashes at the cap.
- **.NET Framework 4.x** managed applications do not run yet.
- Known open security hardening items (local-privilege class) remain; do not expose this to untrusted local users.
- Some 32-bit applications leave redraw artifacts when dragged over other windows.
- Tested Firefox versions: ESR 128 and Firefox 158. Newer versions are expected to work but are untested.
- Everything in upstream ReactOS's own list of known issues also applies.

## Identifying a reactos-nitro build

The desktop shows `ReactOS (Nitro) - <version>`. The About dialog also says *"Unofficial build, not supported by the ReactOS project"*.

## Licence and credits

reactos-nitro is distributed under the same licences as ReactOS: mainly GPL-2.0-or-later, with per-file licences kept as they are in the source. Upstream copyright notices are kept intact.

Credit goes to:

- the [ReactOS project](https://reactos.org) and its contributors, whose operating system this builds on;
- the authors of the Linux kernel's `fs/ntfs` driver;
- [WinBtrfs](https://github.com/maharmstone/btrfs) by Mark Harmstone;
- the [Wine project](https://www.winehq.org), whose code ReactOS and this patch set reuse.
