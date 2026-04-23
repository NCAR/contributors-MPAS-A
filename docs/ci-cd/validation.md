# Validation: ECT and BFB

Two correctness checks that answer different questions:

| Check | Question | Use for |
|-------|----------|---------|
| **ECT** | Did this change move the model outside its noise envelope? | Refactors, optimizations, GPU ports |
| **BFB** | Are these two configurations producing identical NetCDF data? | I/O path, MPI decomposition, refactors that should be exactly equivalent |

## Ensemble Consistency Test (ECT)

ECT compares a small ensemble of perturbed short runs against a precomputed
**ensemble summary** with [PyCECT](https://github.com/NCAR/PyCECT). It does
**not** require bit-for-bit reproducibility — it catches changes that move
behavior outside the round-off-level noise envelope. Reference:
Price-Broncucia et al. (2025),
[doi:10.5194/gmd-18-2349-2025](https://doi.org/10.5194/gmd-18-2349-2025).

In CI:

1. Build double precision with SMIOL.
2. Run **3 ensemble members** in parallel, each with **4 MPI ranks**,
   applying an O(10⁻¹⁴) perturbation to initial `theta`.
3. Validate against the stored summary using PyCECT pinned to `PYCECT_TAG`.

Defaults: 120 km mesh, summary from the `RELEASE_ECT` release, exclusion
list at `.github/data/ect_excluded_vars.txt`.

ECT runs in two places:

- **Inside `_test-compiler.yml`**, alongside each compiler/MPI build, on
  every push and PR.
- **Standalone (`ect-test.yml`)** on every push to `master` and on
  dispatch — GCC + OpenMPI on the 120 km mesh from a cached spin-up
  restart, runnable against an external `mpas-repository`/`mpas-ref` for
  testing arbitrary commits.

### Running ECT outside containers

ECT can be reproduced on Derecho, Cheyenne, or anywhere with PyCECT. You
need the 120 km test archive (matching `RELEASE_TESTDATA_120KM`), the
ensemble summary (`RELEASE_ECT`), and the `perturb_theta.py` /
`trim_history.py` scripts in
[`.github/actions/run-perturb-mpas/`](https://github.com/NCAR/MPAS-Model-CI/tree/master/.github/actions/run-perturb-mpas).
Build double precision, copy the case into N working directories, perturb
`theta`, run, trim history, then run `pyCECT.py` against the summary.

## Bit-for-bit (BFB)

BFB compares NetCDF *variable data* (not raw bytes — harmless metadata
differences are ignored) using
[`compare-bfb-nc.py`](https://github.com/NCAR/MPAS-Model-CI/blob/master/.github/scripts/compare-bfb-nc.py).

The reusable `_test-bfb.yml` takes a `variants` JSON array, one entry per
run:

| Field | Required | Meaning |
|-------|----------|---------|
| `id` | yes | Unique slug (used for artifact paths) |
| `ranks` | yes | MPI process count |
| `use_pio` | no | `true` builds with PIO; default SMIOL |
| `openacc` | no | `true` builds with OpenACC |
| `label` | no | Human-readable description |
| `resolution`, `run_duration` | no | Per-variant overrides |

The variant at `reference_index` (default 0) is the reference; others are
compared against it. Variants sharing build flags reuse one executable.
Default mesh: **240 km**.

### GPU and CPU-vs-GPU BFB

- `bfb-decomp-gpu.yml` — 1 vs 4 ranks on CIRRUS (NVHPC + OpenACC).
- `bfb-io-gpu.yml` — SMIOL vs PIO at 4 ranks on CIRRUS.
- `bfb-nvhpc-cpu-vs-gpu.yml` — NVHPC build with vs without OpenACC; makes
  CPU/GPU divergence visible. Same MPI, ranks, and resolution.

All GPU BFB workflows are dispatch-only.

### Adding a BFB test

Copy an existing caller (`bfb-io.yml`, `bfb-decomp.yml`, or one of the GPU
variants), rename, set the trigger, and edit `variants`.
