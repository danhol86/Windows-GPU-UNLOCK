# Native validation follow-up

Date: 2026-10-26 UTC.

**Current status: CUDA compilation and native metadata inspection passed.
A completed GPU measurement has not yet been obtained.**

The owner explicitly authorized continuing with the compatible compiler,
native build and ordinary GPU execution after the initial draft contribution.
The earlier [CPU-only validation record](VALIDATION.md) is retained as a
historical snapshot. This follow-up concerns ordinary CUDA computation.

## Toolchain and package preparation

Microsoft's versioned v142 component was added alongside the existing Build
Tools installation using the official signed Visual Studio installer:

- Component: `Microsoft.VisualStudio.Component.VC.14.29.16.11.x86.x64`.
- Installer exit code: 0; completed at 2026-10-26T14:04:08.3893412Z.
- Automatic restart and forced application closure were disabled.
- Selected toolset: `14.29.30133`.
- Actual compiler file version: `19.29.30159.0`.
- Compiler SHA-256: `8e4352b523f4ff37b429aea1d340094be1e17bbdf4f687247fc0be92d6eb9440`.

All four pinned NVIDIA archives were downloaded from the official redistribution
server, checked against [packages-lock.json](packages-lock.json), extracted into
the project, and verified file-by-file. Setup completed with exit 0 and wrote its
schema-2 dependency receipt after verification. No NVIDIA driver installer ran.

The first native build used source commit
`1caa536a03c3a307ab40c0031d8dd862c617f0c5` with a clean working tree, NVCC
12.6.20, CUDA 12.6.0 and cuBLAS 12.6.0.22. Compilation and linking succeeded.
The build's manifest SHA-256 was
`d3bc6b4603d423add9ad9b250686f8c6d94081aaa837fa0285e10530c743e429`.

Native `--inspect-only` execution succeeded under a 10-second process deadline,
reported protocol 3 / CUDA 12060 / MSVC 1929, and performed no CUDA API calls.
The CPU numerical helper was rebuilt with v142 using
`/EHsc /std:c++17 /O2 /W4 /fp:precise`; all 15 groups passed.

The packaged cuBLAS header defines 12/6/0 with build suffix 22 and
`CUBLAS_VERSION=120600`. The CUDA runtime header defines `CUDART_VERSION=12060`.
The runtime guards therefore remain unchanged.

## Initial quiet-profile refusal

The original quiet-profile attempt at 2026-10-26T14:11:09.431Z was refused at
`pre_query` with `entry_not_quiet`, zero validated trials and no completion
record. No native GPU process was launched.

A separate read-only observation at 2026-10-26T14:12:58.877384Z reported:

| Field | Observed value |
|---|---:|
| GPU utilization | 0% |
| Allocated GPU memory | 2136 MiB |
| Total reported GPU memory | 10240 MiB |
| GPU temperature | 34 C |
| Reported power limit | 370 W |
| Operating SM / memory clocks | 210 / 405 MHz |

The allocation exceeded the original 2048 MiB entry ceiling. A low utilization
snapshot does not prove continuous inactivity. Window inspection found the
candidate Snipping Tool and Docker Desktop windows already minimized; no
applications or documents were closed.

## Explicit entry profiles

The default quiet profile remains available. A separate desktop profile records
the less restrictive ambient allocation condition needed for this desktop:

| Entry requirement | Quiet | Desktop |
|---|---:|---:|
| Maximum GPU utilization | 5% | 5% |
| Maximum allocated GPU memory | 2048 MiB | 3072 MiB |
| Maximum temperature | 65 C | 65 C |
| Minimum available host RAM | 1 GiB | 1 GiB |

The desktop ceiling still leaves 7168 MiB of the reported total unallocated at
entry. Native free-memory and allocation-budget checks also apply. These are
measurement policies, not hardware ratings or proof of an idle machine.

The profile name and exact limits are included in context schema 2 and checked
by the offline reader. The new profile does not change numerical tolerances,
input generation, precision modes, sample counts, process deadlines, runtime
version requirements, or device settings. It must be identified explicitly
when reporting measurements; it is not the original quiet-profile condition.

The updated combined Python suite passed 112 tests (49 protocol and 63 runner)
in 33.137 seconds, exit 0. Tests cover both profiles' boundaries, preservation of
the default quiet policy, invalid profile names, and profile/limit tampering
before and after file-hash recomputation. Relabeling a desktop record at
2136 MiB as a quiet record is rejected.

Native workload results will be recorded only after the updated source is
committed, rebuilt, executed successfully and independently read back offline.
