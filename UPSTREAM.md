# Relationship to upstream ReactOS

reactos-nitro tracks ReactOS master and merges it regularly. We keep our changes as small and self-contained as we can, so the patch set stays easy to rebase and our fixes stay easy for the ReactOS project to pick up. Where upstream is actively building the same thing, we say so here and explain where we differ.

This is written in good faith from the public ReactOS commit history; it is our reading, not a statement from the ReactOS project.

**Last synced with ReactOS master:** [`e46f4bb1a1e`](https://github.com/reactos/reactos/commit/e46f4bb1a1ea7c722f941231565144fe0c286d3e) (2026-10-25, weekly sync #4; 59 upstream commits since `f53b1d15601`). `nitro-2026.10-5` is built on this commit; the patch series applies to it unchanged.

## What upstream is actively working on that overlaps with us

- **NTFS write and install.** Upstream has added experimental NTFS write and NTFS install support to its own `ntfs` driver (e.g. [#9316](https://github.com/reactos/reactos/pull/9316), [#9319](https://github.com/reactos/reactos/pull/9319), [#9700](https://github.com/reactos/reactos/pull/9700)) and is still improving it.
- **SMP / APIC.** Ongoing HAL work (for example x2APIC support).
- **USB3.** A new USB3 stack is being built (xHCI/UCX, [#9623](https://github.com/reactos/reactos/pull/9623)).
- **MMC.** A new Microsoft Management Console implementation is in progress.
- **Shell.** Continuing shell work (for example IShellItem2, [#9692](https://github.com/reactos/reactos/pull/9692)).

Several of these are areas where we also have work. Our intent is to converge with upstream, not to compete: as upstream's version of a feature matures, we expect to drop ours in favour of it.

## Why our NTFS is a separate driver

Upstream's own NTFS driver now writes, and write is **enabled by default on a normal install** (the default system hive sets the enabling registry value, [#9319](https://github.com/reactos/reactos/pull/9319)). But it is still labelled experimental — the registry value is literally named `MyDataDoesNotMatterSoEnableExperimentalWriteSupportForEveryNTFSVolume` — several operations are still unimplemented, and it has no journaling or crash/power-cut recovery.

We wanted NTFS that is safe to use as the default file system **now**. So reactos-nitro uses `ntfsng`, a driver built on the Linux kernel's `fs/ntfs`, running through a compatibility layer, with journaling and power-cut recovery. It is write-enabled by default and is the default file system for new installs.

The trade-off is honest: this is a larger divergence from upstream than a small patch would be, and it is a different codebase from upstream's driver. We took that cost deliberately, to have dependable read/write NTFS today rather than wait for the experimental driver to mature. If upstream's NTFS reaches the same reliability, re-converging on it is on the table.

## Disk Management

Our `diskmgmt.msc` is a standalone application, not built on upstream's in-progress MMC. When upstream's MMC is ready, porting Disk Management onto it is planned.
