# reactos-nitro

**reactos-nitro is an unofficial patch set for [ReactOS](https://reactos.org).** It is not affiliated with or endorsed by the ReactOS project. ReactOS is a trademark of the ReactOS project.

> **Do not report problems from this build to the ReactOS project.** Report them here, in this repository's issues.
>
> Like ReactOS itself, this is alpha-quality software. Don't use it on a machine or disk holding data you care about.

**Last synced with ReactOS master:** [`f53b1d15601`](https://github.com/reactos/reactos/commit/f53b1d1560120c529763b18963fdee68f39118af) (2026-10-19, sync #3). We merge upstream regularly; this line tracks the most recent merge. (The `nitro-2026.10-1` release is based on the earlier [`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043), 2026-10-18.)

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

Latest build: **`reactos-nitro-2026.10-4`** (2026-10-24), on upstream base `f53b1d15601` — the app-compatibility release: Claude Desktop (Electron/Chromium) runs, renders, and connects online, and Firefox-ESR 32-bit + 64-bit render side-by-side on a single amd64+WoW64 system, plus a kernel security batch. Each release ships an amd64-with-WoW64 ISO, an i386 ISO, a source tarball, and a cumulative over-base patch; see [CHANGELOG.md](CHANGELOG.md) for per-release detail and known limitations.

## Highlights

Over 520 fixes and several feature tracks on top of upstream ReactOS:

- **32-bit Windows programs on 64-bit ReactOS (WoW64).** Total Commander runs 32-bit and 64-bit side by side in one session, and Firefox ESR runs 64-bit; 32-bit Firefox launches but renders only with its content sandbox disabled (see [Known limitations](#known-limitations)). Upstream ReactOS has no WoW64.
- **NTFS read/write**, with a new driver built on the Linux kernel's `fs/ntfs` (imported from Linux v7.3-rc6), plus journal and power-cut recovery. NTFS is the default file system for new installs.
- **Btrfs**: WinBtrfs 1.10 (upstream ships 1.8.1).
- **Multi-processor support on x86 and amd64**, installed by default on ACPI machines. It is tested from 1 to 16 vCPUs, with multi-hour soaks on AMD and Intel hosts.
- **Smoother desktop.** A shadow framebuffer makes window dragging smooth at a fraction of the CPU cost. USB tablet polling and pointer fixes also help.
- **Networking**: a reworked receive path and dozens of TCP/IP, AFD and Winsock fixes.
- **Setup and user profiles**: the setup program preselects NTFS. The C:\Users layout includes Windows-style legacy folder links, and profile deletion works.
- **Kernel and memory manager hardening**: commit charging with a commit limit, plus many Mm, cache-manager and scheduler fixes.
- **Desktop and tools**: LCD text rendering, bundled open fonts, crash reports with a Windows-style dialog, and Disk Management.

A preliminary [BENCHMARKS.md](BENCHMARKS.md) (draft) compares reactos-nitro against upstream ReactOS.

A full list of changes, one line per fix, is in [CHANGELOG.md](CHANGELOG.md) (draft for the first release).

## Known limitations

- WoW64 is amd64 only.
- **Firefox sandbox.** On 64-bit, the sandbox process starts but its restrictions are **not** enforced on ReactOS. On 32-bit (WoW64), the sandbox broker cannot spawn Firefox's child processes, so with the sandbox at its default setting **32-bit Firefox opens but web pages stay blank**. To browse in 32-bit Firefox you must disable its content sandbox (`security.sandbox.content.level = 0`), which is a security downgrade. 64-bit Firefox renders normally.
- Some 32-bit applications leave redraw artifacts when dragged over other windows.
- Firefox ESR 140 and later need components that ReactOS does not provide yet. ESR 128 is the tested version.
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
