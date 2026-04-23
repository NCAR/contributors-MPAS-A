# Workflows & Triggers

A typical CPU subset run:

```mermaid
graph LR
    A[caller workflow] --> B[_test-compiler.yml]
    B --> C[resolve container] --> D[build] --> E[3 ECT members] --> F[PyCECT validate] --> G[cleanup]
```

Caller workflows declare triggers and pass `compiler`/`mpi`/`precision` to
the reusable `_test-compiler.yml` (or `_test-gpu.yml`). Build, run, and
validation logic live in the reusable workflows and composite actions.

## Trigger policy

| Class | Trigger | Notes |
|-------|---------|-------|
| `test-*-mpich` | push/PR to `master`, `develop`, `hackathon-*` | Primary PR gate |
| `test-*-openmpi` | dispatch | OpenMPI in containers is noisy |
| `test-gpu-*` | dispatch | Self-hosted CIRRUS — security boundary |
| `compile-nvhpc-cuda-mpich` | push/PR | OpenACC/CUDA toolchain check, no GPU |
| `ect-test` | push to `master`; dispatch | Standalone ECT (GCC + OpenMPI, 120 km, GitHub-hosted) |
| `bfb-*` (CPU) | dispatch; some on push to `hackathon-*` | Bit-for-bit |
| `bfb-*-gpu`, `bfb-nvhpc-cpu-vs-gpu`, `profile-gpu-nsight` | dispatch | Inherit GPU policy |
| `coverage` | push to `master` | GCC coverage to Codecov |
| `unit-tests` | push/PR | pFUnit across GCC 12/13/14 |

Branches matching **`hackathon-*`** auto-run the CPU ECT subsets — useful
for running the full gauntlet before opening a PR. Exact `on:` blocks live
in each workflow YAML.

## Cross-repo testing

`test-cross-repo.yml` accepts `mpas-repository` and `mpas-ref` inputs to
run the multi-compiler subset against any MPAS-Model commit (a fork, PR
branch, or upstream `develop`). The `checkout-mpas-source` action clones
the target source and overlays `.github/` from MPAS-Model-CI. Same
mechanism is built into `ect-test.yml` so a standalone ECT can be
dispatched against an external commit.

## Artifacts

Builds, history files, and logs are uploaded as GitHub Actions artifacts.
A cleanup job removes temporary artifacts at the end of each run. Profiler
output is kept for **3 days**.
