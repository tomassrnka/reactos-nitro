# Benchmarks — reactos-nitro vs ReactOS master

> **Not re-measured for 2026.10-5.** These numbers come from a development build that predates `nitro-2026.10-1` (see below) and have not been re-measured for this release; they do not reflect the Windows 10 identity default, the memory commit cap or other changes since.
>
> **DRAFT / preliminary.** These numbers were measured on a development build (batch 87, `23dd8f7e336`), **not** the tagged `2026.10-1` release, and will be re-run on the release build before launch. Two outliers (marked †) are large enough that they need an independent re-check. Read this as a direction, not final figures.

Measured 2026-10-19 on one AMD EPYC 9254 host under QEMU/KVM, 4 vCPU / 2 GB, both sides built Release from the same sources.

- **A — ReactOS master:** upstream [`ac1ab9e3043`](https://github.com/reactos/reactos/commit/ac1ab9e3043) (the reactos-nitro base).
- **B — reactos-nitro:** fork `main` [`23dd8f7e336`](https://github.com/tomassrnka/reactos/commit/23dd8f7e336).

## The two things that don't compare

- **amd64: upstream could not be measured.** The upstream amd64 build does not get through setup — it bugchecks with `STOP 0x71 SESSION5_INITIALIZATION_FAILED`, 2 of 2 attempts. reactos-nitro installs and runs on amd64. So there is no amd64 A-vs-B row; the amd64 column below is nitro only.
- **SMP: upstream runs on one CPU.** Upstream's setup installs the uniprocessor kernel, and its multiprocessor kernel, swapped in by hand, still sees one CPU and then stops. So the real comparison below is upstream uniprocessor vs nitro multiprocessor — which is the point.

**Stability:** reactos-nitro finished all 26 counted runs across i386/amd64, FAT/NTFS and 1/4 vCPU with no crash or hang.

## i386, quiet host (the real A-vs-B comparison)

Medians of 5 runs. "B/A" above 1 means nitro is faster; for latency, lower is better. Cases where nitro is **slower** are marked.

| Area | Test | ReactOS master | reactos-nitro | B/A |
|---|---|---|---|---|
| SMP | CPU throughput, 4 threads (Mops/s) | 687 (1 CPU) | 2744 | **4.0×** |
| SMP | CPU throughput, 2 threads | 687 (1 CPU) | 1376 | 2.0× |
| Memory | memcpy (MB/s) | 27,688 | 28,112 | 1.0× (same) |
| Memory | memset (MB/s) | 51,895 | 52,493 | 1.0× (same) |
| Memory | demand-zero page faults (faults/s) | 1,961,140 | 1,933,550 | 1.0× (same) |
| Memory | VirtualAlloc/Free 64 KiB (ops/s) | 227,125 | 1,640,700 | 7.2× |
| Memory | VirtualAlloc + touch + Free (ops/s) | 533 | 56,340 | **106× †** |
| Disk (FAT) | sequential write + flush (MB/s) | 22.3 | 56.4 | 2.5× |
| Disk (FAT) | buffered read (MB/s) | 26.3 | 291 | **11× †** |
| Disk (FAT) | small-file create (files/s) | 423 | 3,299 | 7.8× |
| Disk (FAT) | small-file delete (files/s) | 36,958 | 35,318 | 1.0× (same) |
| Disk (FAT) | uncached read (MB/s) | 708 | 665 | 0.94× (**slower**) |
| Network | HTTP download (MB/s) | 23.5 | 23.4 | 1.0× (same) |
| Network | requests/s, 1 connection | 473 | 410 | 0.87× (**slower**) |
| Network | requests/s, 4 parallel | 473 | 701 | 1.5× |
| SMP | cross-CPU wake latency, unpinned (µs) | n/a (1 CPU) | 9.6 | slower than A's single-CPU 5.0 |

† The 106× and 11× figures reflect a slow path in the upstream build and are pending an independent re-check before they're treated as final.

## NTFS

_Note: upstream ReactOS has experimental NTFS write (off by default, partial, unjournaled). The upstream-vs-nitro NTFS comparison was not run and is pending re-measurement. The table below is reactos-nitro FAT vs NTFS only._

| Test (reactos-nitro) | FAT | NTFS |
|---|---|---|
| buffered read (MB/s) | 267 | 2,477 |
| sequential write (MB/s) | 55.1 | 53.1 |
| small-file create (files/s) | 3,008 | 2,394 |
| small-file delete (files/s) | 33,300 | 9,657 |

## Honest caveats

- Preliminary pre-2026.10-1 dev build, not any release tag; not re-measured for 2026.10-5; re-run pending.
- No amd64 A-vs-B (upstream amd64 doesn't install).
- Upstream NTFS write (experimental, off by default) was not benchmarked against nitro's ntfsng; that comparison is pending.
- The SMP comparison is upstream-uniprocessor vs nitro-multiprocessor.
- Under heavy host load, nitro degraded more than upstream on the network tests (down to 0.54× on single-connection requests/s). At 1 vCPU, nitro was slower on download, requests/s and uncached read.
- Disk tests measure the guest file system and cache manager over the host page cache, not raw device speed.
