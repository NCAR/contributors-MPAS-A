# GPU & Profiling

GPU testing splits into three tiers, each with a different cost and
trigger.

## Tier 1: compile-only (every PR)

`compile-nvhpc-cuda-mpich.yml` builds with `openacc: 'true'` inside the
NVHPC + MPICH + CUDA container on a GitHub-hosted runner — no GPU, no
model run. Catches OpenACC syntax errors and toolchain regressions early.

## Tier 2: GPU ECT and BFB on CIRRUS (dispatch only)

`test-gpu-mpich.yml` and `test-gpu-openmpi.yml` run a full ECT cycle on
`CIRRUS-4x8-gpu` self-hosted runners (`_test-gpu.yml`, OpenACC build).
`bfb-decomp-gpu.yml`, `bfb-io-gpu.yml`, and `bfb-nvhpc-cpu-vs-gpu.yml`
extend this to BFB. All dispatch-only — running unreviewed code on
self-hosted runners is a security boundary.

## Tier 3: Nsight Systems profiling (dispatch only)

`profile-gpu-nsight.yml` produces an Nsight Systems trace on CIRRUS:

1. Build OpenACC MPAS-A in the CUDA container; download a test case
   (default 240 km).
2. Override only `config_run_duration` — never `config_dt`, which is set
   by science. Default `0_01:00:00` is three timesteps for the 240 km case
   (`config_dt = 1200 s`).
3. Run `nsys profile --trace=cuda,nvtx,osrt --stats=true` around `mpirun`.
4. Upload `.nsys-rep` and `nsys stats` text with **3-day** retention.

The `setup-nsight-systems` action prefers an `nsys` already in the image
and otherwise installs `nsight-systems-cli` from NVIDIA's devtools repo,
caching RPMs (bump `NSYS_CLI_CACHE_VERSION` to invalidate).

For other resolutions, set `run_duration` to at least one timestep at that
case's `config_dt`.

## Conventions for new GPU code

- **Batch host↔device transfers.** v8.4.0 reorganized the dycore so most
  fields move once at the start and end of a dynamics step rather than per
  routine. Follow this pattern.
- **Use `present(...)` clauses** so the runtime fails loudly if a field
  wasn't transferred.
- **Use GPU-aware MPI** for halo exchanges in performance-critical paths
  (`config_gpu_aware_mpi = .true.` in `&development`).
- **Don't `!$acc parallel loop` trivial loops** — directive overhead can
  exceed the work.

Validate any GPU port with ECT first, then measure performance. For exact
CPU/GPU equivalence, add a GPU BFB caller — see
[Validation](validation.md#gpu-and-cpu-vs-gpu-bfb).
