# Containers & Configuration

## Container images

All builds use
[`ncarcisl/hpcdev-x86_64`](https://hub.docker.com/r/ncarcisl/hpcdev-x86_64).
Image names come from templates in
[`ci-config.env`](https://github.com/NCAR/MPAS-Model-CI/blob/master/.github/ci-config.env),
resolved by the `resolve-container` action.

| Class | Template (current `26.02` line) |
|-------|---------------------------------|
| CPU | `almalinux9-{compiler}-{mpi}-26.02` |
| GPU | `almalinux9-{compiler}-{mpi}-cuda-26.02` |

Compiler names sometimes need translation (e.g. `gcc` → `gcc14`) via
`CONTAINER_COMPILER_*` mappings. Intel images may be pinned to an older
line when toolchain regressions break newer ones; comments in
`ci-config.env` track current state.

Inside containers, source `/container/config_env.sh` before any MPI build
or run; activate Miniforge with
`eval "$(conda shell.bash hook)" && conda activate base` if you need it.

## `ci-config.env`

Single source of truth for everything that varies across compilers and
resolutions:

| Variable family | Purpose |
|-----------------|---------|
| `CONTAINER_IMAGE*`, `CONTAINER_COMPILER_*` | Image templates and name mappings |
| `MAKE_TARGET_*`, `*_EXTRA_MAKE_FLAGS` | Compiler-specific build settings |
| `OPENMPI_RUN_FLAGS`, `MPICH_RUN_ENV_*` | MPI runtime settings |
| `RELEASE_TESTDATA_*`, `RELEASE_ECT` | GitHub release tags for assets |
| `ECT_*`, `PYCECT_TAG` | ECT resolution, perturbation, exclusion list, tools |
| `BFB_*` | BFB resolution, duration, run timeout |
| `NSYS_CLI_CACHE_VERSION` | Cache key for the Nsight Systems CLI install |

Editing this file is almost always cheaper than editing a workflow.

## Test data

Test meshes, ECT summaries, and ECT spin-up restarts live as **GitHub
release assets** on
[NCAR/MPAS-Model-CI](https://github.com/NCAR/MPAS-Model-CI/releases),
versioned independently. Current tags include `testdata-240km-v1`,
`testdata-120km-v1`, `ect-summary-v1`, `ect-restart-v1`.

The `download-testdata` action looks up `RELEASE_TESTDATA_{RES}`
(resolution uppercased, `-` → `_`), downloads `{resolution}.tar.gz` from
the matching release, caches it, and extracts. Each archive's
`namelist.atmosphere` carries model defaults; workflows override only what
they need (typically `config_run_duration`).

## Known issues

- **NVHPC + OpenMPI**: model exits 134 (SIGABRT) with 4 ranks on
  GitHub-hosted runners. MPICH is the primary PR gate; affected jobs run
  `continue-on-error`.
- **NVHPC and Intel `mpi_f08`**: broken against MPI in current `hpcdev`
  images; both use `MPAS_MPI_F08=0`.
- **Intel oneAPI**: image may be pinned to an older line until upstream
  regressions clear.

## Common debugging knobs

- **Intel stack size**: `ulimit -s unlimited`
- **OpenMPI in containers**: runs with `--allow-run-as-root --oversubscribe`
  via `OPENMPI_RUN_FLAGS`; `run-mpas` applies these automatically.
- **PIO vs SMIOL**: toggle with `USE_PIO2` / `PIO_ROOT` build flags.
