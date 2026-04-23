# CI/CD Overview

CI for MPAS-A lives in
[**NCAR/MPAS-Model-CI**](https://github.com/NCAR/MPAS-Model-CI),
independent of MPAS-Model so CI changes don't require touching the model.
These pages summarize what CI does for contributors; for infrastructure
work, edit MPAS-Model-CI directly.

## What CI runs

All jobs are defined and executed in **this CI repository**. They clone
**MPAS-Dev/MPAS-Model** (or another fork) at a chosen ref when a workflow
runs — including on `workflow_dispatch` with cross-repo inputs.

| Capability | Notes |
|------------|-------|
| Multi-compiler builds | GCC, Intel oneAPI, NVHPC inside `hpcdev` containers |
| CPU subset testing | MPICH + ECT when workflows run (see [Workflows](workflows.md)) |
| Ensemble Consistency Test (ECT) | Statistical validation via [PyCECT](https://github.com/NCAR/PyCECT) |
| Bit-for-bit (BFB) tests | NetCDF variable comparison across configurations |
| GPU (OpenACC) | Full ECT on **CIRRUS** (dispatch); NVHPC+CUDA compile-only on CI-repo push/PR |
| GPU profiling | Nsight Systems traces on dispatch |
| Coverage & unit tests | GCC coverage to Codecov; pFUnit across GCC 12/13/14 |

## Layout

| Path in MPAS-Model-CI | Purpose |
|------|---------|
| [`.github/workflows/`](https://github.com/NCAR/MPAS-Model-CI/tree/master/.github/workflows) | Caller workflows + reusable templates (`_*.yml`) |
| [`.github/actions/`](https://github.com/NCAR/MPAS-Model-CI/tree/master/.github/actions) | Composite actions (build, run, download, validate) |
| [`.github/ci-config.env`](https://github.com/NCAR/MPAS-Model-CI/blob/master/.github/ci-config.env) | Image templates, release tags, MPI flags, ECT/BFB defaults |
| `tests/` | pFUnit unit-test infrastructure |

Reusable workflows (prefixed `_`) hold the job logic; caller workflows
(e.g. `test-gcc-mpich.yml`) are thin shims that pass inputs and define
triggers. The workflow YAML and `ci-config.env` are the source of truth —
if these pages disagree with them,
[file an issue](https://github.com/NCAR/contributors-MPAS-A/issues).
