# RM9 kernel CI (pineapple / SM8650, android14-6.1)

## What this builds
The RedMagic 9 Pro kernel in this repo using Qualcomm's **kleaf/Bazel** build
system, producing a `dist/` with the kernel `Image`, `*.dtb`/`dtbo.img`, and
(where the tree generates them) `boot.img` / `vendor_boot.img` / `*.img` /
`*.ko`.

This repo only carries the **device sources**:
`kernel_platform/{common,msm-kernel,qcom}` + top-level `external/ kernel/ vendor/`.
The kleaf tooling and prebuilts are **not** committed; CI fetches them.

## How the workspace is assembled
`.github/workflows/build-kernel.yml` runs `repo` against the trimmed manifest
[`rm9_pineapple.xml`](./rm9_pineapple.xml), which pins the missing kleaf
projects (`build/`, `prebuilts/`, `external/`, `tools/`, `bootable/`) to CLO
drop:

```
AU_LINUX_KERNEL.PLATFORM.3.0.C1.00.00.00.057.010
(git.codelinaro.org/clo/la/kernelplatform/manifest)
```

This is the **earliest** CLO pineapple android14-6.1 release, and its pinned
clang prebuilt is exactly **`clang-r487747c`**, matching this tree's
`kernel_platform/common/build.config.constants`. The manifest's own
`kernel/common` and `kernel/qcom` projects are **removed** so our sources stay
authoritative; the three `<linkfile>` entries they'd create
(`WORKSPACE`, `build_with_bazel.py`, `.source_date_epoch_dir`) are recreated by
the workflow after `repo sync`.

## Build command
```
cd kernel_platform
./build_with_bazel.py -t pineapple gki --skip abl --lto=thin
# output: kernel_platform/out/msm-kernel-pineapple-gki/dist/
```

## Running it
Actions → **Build RM9 Kernel (pineapple GKI)** → *Run workflow*.
Inputs: `variant` (`gki`/`consolidate`), `lto` (`thin`/`full`/`none`).

## Known caveats
- **Disk/time:** a full kleaf pineapple build is heavy. The workflow frees
  ~40 GB on the hosted runner first; if it still runs out of disk, use a larger
  runner. `--lto=thin` keeps RAM/time reasonable (`full` is much slower).
- **`common` is labelled 6.1.25** while the kleaf tooling comes from the
  earliest available pineapple drop (6.1.68-era). The kleaf API is stable across
  android14-6.1 (KMI generation 11) and the clang matches, so this is expected
  to build; if a kleaf/common API mismatch appears, that's the first thing to
  revisit.
- **No SukiSU / no SUSFS** yet — this is the stock baseline.
