# Smokeview — chraibi fork

This is a fork of [firemodels/smv](https://github.com/firemodels/smv)
maintained for the [pyFDS-Evac](https://github.com/PedestrianDynamics/pyFDS-Evac)
project. It diverges from upstream on one point: **per-particle avatar
rendering and rotation for Lagrangian particle files (`.prt5`)**.

## Why this fork will not track upstream

The `&EVAC` namelist was removed from FDS, and most of Smokeview's
agent-rendering code was removed along with it. Upstream commit
[`b0dc2a69d`](https://github.com/firemodels/smv/commit/b0dc2a69d)
("smokeview source: remove particle azimuth and elevation orientation
angles", 2026-04-22) deletes the `col_azimuth` / `col_elevation`
fields from `partclassdata` and the `AZIMUTH` / `ELEVATION` shortlabel
recognition in `readsmvfile.c` — the hooks we need to face a rendered
avatar in the direction its corresponding agent is walking.

Glenn Forney's position on
[firemodels/smv#2597](https://github.com/firemodels/smv/issues/2597)
is that per-particle rotation should be re-introduced via a new FDS
particle quantity type, not via Smokeview code. That is a reasonable
long-term path but depends on changes outside this repository and is
not compatible with our short-term goal of driving Smokeview from
existing `.prt5` outputs.

We therefore **do not plan to merge upstream `master` back into this
fork's `master`** once the divergence with `b0dc2a69d` is in place.
Ad-hoc cherry-picks of non-conflicting upstream work are possible but
not automatic.

## What this fork adds on top of stock Smokeview

Against base `84b123316` (upstream `master` at 2026-04-21):

| Commit | Change |
|---|---|
| `0d1bae185` | `fix(IOpart): use memory-backed stream in CreatePartBoundFile` — so PRT5 bounds scanner doesn't collapse `ntimes` to 1 when `numtypes > 0` (`fread_mv` zero-copy path requires the memory branch). |
| `475f806f0` | `feat(IOpart): apply per-particle AZIMUTH as a rotation in DrawPart` — reinstates per-particle `glRotatef` driven by the `AZIMUTH` quantity column, replacing the old FDS+Evac `CLASS_OF_HUMANS` reader that was removed with `&EVAC`. |
| `abb688bce` | `fix(threader): null out caller's slot on ThreadJoin` — prevents `assert(*thiptr == NULL)` aborts on re-entrant thread slots. |
| `9683c1e8e` | `fix(IOpart): read AZIMUTH from rvals, not broken fvars_dep unmap` — bypasses the `CopyDepVals` quantization path whose `partpropdata.valmin/valmax` sentinels are never populated without a case `.ini`, which otherwise collapsed every per-particle AZIMUTH to `0°`. |
| `63918b8e0` | `fix(IOpart): join prior SortAllPartTags before re-init in FinalizePartLoad` — removes the `ThreadInit` assertion abort hit on any PRT5 large enough that the background sort is still running when the cleanup path reopens the slot. |

All five are confined to particle-rendering code; none should affect
slice, boundary, 3-D smoke, isosurface, or geometry paths.

## Build

Same as upstream:

```bash
cd Build/<platform>  # e.g. osx_intel_64 or linux_intel_64
./make_smv.sh
```

or with CMake from a build directory:

```bash
cmake -S . -B build-fork -DCMAKE_BUILD_TYPE=Release
cmake --build build-fork --target smokeview -j$(nproc)
```

The binary lands at `build-fork/smokeview` (CMake path) or at the
usual `Build/<platform>/<name>` path (legacy shell scripts).

## Running with pyFDS-Evac

`run.py` with `--smv-export --smv-with-azimuth` writes a PRT5 that
uses the rotation path this fork re-enables. Against stock Smokeview
6.10.x the same file plays frame 0 only and never rotates; against
this fork it plays to completion and each avatar turns to face its
direction of travel.

See
[PedestrianDynamics/pyFDS-Evac `docs/smv-avatars.md`](https://github.com/PedestrianDynamics/pyFDS-Evac/blob/main/docs/smv-avatars.md)
for the full protocol and a minimal reproducer.

## Upstream links

- Smokeview (upstream): <https://github.com/firemodels/smv>
- Upstream issues: <https://github.com/firemodels/smv/issues>
- Upstream discussion: <https://groups.google.com/forum/#!forum/fds-smv>
- FDS-SMV website: <https://pages.nist.gov/fds-smv/>

## License

Inherits upstream Smokeview's license. See the upstream repository for
details.
