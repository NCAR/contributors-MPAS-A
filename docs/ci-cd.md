# MPAS-A CI/CD System

This page documents the Continuous Integration and Continuous Deployment (CI/CD) infrastructure for MPAS-A, based on the work in the [MPAS-Model-CI repository](https://github.com/NCAR/MPAS-Model-CI).

## Overview

The MPAS-A CI/CD system provides automated testing infrastructure to:

- **Build MPAS-A** across multiple compiler and MPI combinations
- **Run test cases** to validate model correctness
- **Validate results** against reference outputs
- **Support GPU acceleration** testing with OpenACC

The system leverages GitHub Actions workflows and NCAR's containerized development environments to ensure consistent, reproducible builds and tests.

---

## Architecture

### Workflow Structure

The CI/CD pipeline consists of three main jobs that run sequentially:

```mermaid
graph LR
    A[Build Job] --> B[Run Job]
    B --> C[Validation Job]
    C --> D[Cleanup Job]
```

1. **Build Job**: Compiles MPAS-A with various compiler/MPI/IO combinations
2. **Run Job**: Executes the 240km test case with built executables
3. **Validation Job**: Compares output logs against reference standards
4. **Cleanup Job**: Removes temporary artifacts

### Container Environment

The CI system uses NCAR CISL development containers, which provide pre-configured environments with:

- Compilers (GCC, Intel oneAPI, NVIDIA HPC SDK)
- MPI implementations (MPICH, OpenMPI)
- I/O libraries (PIO, NetCDF, Parallel-NetCDF)
- CUDA toolkit (for GPU builds)

Container images follow the naming convention:
```
ncarcisl/cisldev-{arch}-{os}-{compiler}-{mpi}[-cuda]:devel
```

For example: `ncarcisl/cisldev-x86_64-almalinux9-nvhpc-mpich3-cuda:devel`

---

## Build Matrix

The CI system tests MPAS-A across multiple configurations:

### Supported Compilers

| Compiler | Target | GPU Support |
|----------|--------|-------------|
| GCC (gfortran) | `gfortran` | No |
| Intel oneAPI | `intel` | No |
| NVIDIA HPC SDK | `nvhpc` | Yes (OpenACC) |

### Supported MPI Implementations

| MPI | Environment Variable |
|-----|---------------------|
| MPICH | `MPI_IMPL=mpich` |
| OpenMPI | `MPI_IMPL=openmpi` |

### I/O Options

| I/O Library | Configuration |
|-------------|---------------|
| PIO | `USE_PIO2=true`, `PIO_ROOT=/container/pio` |
| SMIOL | `USE_PIO2=false` |

### GPU Options

| Mode | Configuration |
|------|---------------|
| CPU only | `OPENACC=false` (default) |
| GPU (CUDA) | `OPENACC=true` (nvhpc only) |

---

## Workflows

### Main Build and Run Workflow

The primary workflow (`CIRRUS_build_run240.yml`) performs comprehensive testing:

```yaml
name: (CIRRUS, cisldev) - Build MPAS-A and run 240km test case

on:
  workflow_dispatch:
    inputs:
      os:
        description: 'Base OS'
        type: choice
        required: true
        default: almalinux9
        options:
          - almalinux8
          - almalinux9
          - almalinux10
          - leap
          - tumbleweed
          - noble
```

#### Build Job Configuration

The build job uses a strategy matrix to test multiple configurations:

```yaml
strategy:
  fail-fast: false
  max-parallel: 4
  matrix:
    compiler: [ nvhpc, oneapi, gcc ]
    mpi: [ mpich3, openmpi ]
    gpu: [ cuda, nogpu ]
    io: [ smiol, pio ]
    arch: [ x86_64 ]

    exclude:
      - compiler: gcc
        gpu: cuda
      - compiler: oneapi
        gpu: cuda
```

!!! note "GPU Build Exclusions"
    GPU builds with CUDA are only supported with the NVIDIA HPC SDK (`nvhpc`) compiler, as it provides OpenACC support. GCC and Intel oneAPI builds skip the CUDA configuration.

#### Run Job Configuration

The run job downloads built executables and runs the 240km test case:

```yaml
strategy:
  matrix:
    compiler: [ nvhpc, oneapi, gcc ]
    mpi: [ mpich3, openmpi ]
    gpu: [ cuda, nogpu ]
    io: [ smiol, pio ]
    num_procs: [ 1 ]
```

---

## Build Process

### Build Script

The `build_mpas.sh` script handles compilation:

```bash
#!/usr/bin/env bash
set -ex

# Source common configuration
source ${SCRIPTDIR}/build_common.cfg

# Map compiler family to MPAS make target
case "${COMPILER_FAMILY}" in
    "aocc"|"clang") compiler_target="llvm" ;;
    "gcc")          compiler_target="gfortran" ;;
    "oneapi")       compiler_target="intel" ;;
    "nvhpc")        compiler_target="nvhpc" ;;
esac

# Build MPAS-A
make ${compiler_target} CORE=atmosphere --jobs ${MAKE_J_PROCS:-$(nproc)}
```

### Build Configuration

Environment variables control the build:

| Variable | Description | Example |
|----------|-------------|---------|
| `COMPILER_FAMILY` | Compiler being used | `gcc`, `nvhpc`, `oneapi` |
| `USE_PIO2` | Enable PIO library | `true` or `false` |
| `PIO_ROOT` | Path to PIO installation | `/container/pio` |
| `OPENACC` | Enable OpenACC for GPU | `true` or `false` |
| `MAKE_J_PROCS` | Parallel make jobs | Number or `$(nproc)` |

---

## Test Case: 240km Global

The CI system uses a 240km resolution global test case for validation.

### Run Script

The `run_mpas_240km.sh` script executes the test:

```bash
#!/usr/bin/env bash
set -ex

# Parse arguments
NUM_PROCS="${1:-${NUM_PROCS:-1}}"
RUN_DURATION="${2:-${RUN_DURATION:-0_06:00:00}}"
RESTART_INTERVAL="${3:-${RESTART_INTERVAL:-0_06:00:00}}"

# Set MPI flags for OpenMPI in containers
if [ "${MPI_IMPL}" = "openmpi" ]; then
    MPI_FLAGS="${MPI_FLAGS} --allow-run-as-root"
fi

# Extract test case
tar xzf .github/workflows/240km.tar.gz
mv 240km 240km_${NUM_PROCS}
cd 240km_${NUM_PROCS}/

# Link executable
ln -sf ../atmosphere_model .

# Configure run duration
sed -i "s/config_run_duration = '[^']*'/config_run_duration = '${RUN_DURATION}'/" namelist.atmosphere

# Set stack size for Intel Fortran
ulimit -s unlimited 2>/dev/null || true

# Run the model
mpirun -n "$NUM_PROCS" $MPI_FLAGS ./atmosphere_model
```

### Run Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `NUM_PROCS` | 1 | Number of MPI processes |
| `RUN_DURATION` | `0_06:00:00` | Simulation duration (DD_HH:MM:SS) |
| `RESTART_INTERVAL` | `0_06:00:00` | Restart file output interval |

---

## Validation

### Log Comparison

The validation job compares run outputs against reference standards:

```yaml
- name: Compare logs against reference
  run: |
    python .github/workflows/validation/compare_logs.py \
      logs \
      .github/workflows/validation/reference_log.atmosphere.0000.out
```

### Validation Criteria

The validation system checks:

- **Completion status**: Model ran to completion without errors
- **Output values**: Key diagnostic values match reference within tolerance
- **Timing information**: Run completed within expected timeframe

---

## Artifacts

### Build Artifacts

Each build produces an executable artifact:

```yaml
- name: Upload atmosphere_model executable
  uses: actions/upload-artifact@v4
  with:
    name: atmosphere_model-${{ matrix.compiler }}_${{ matrix.mpi }}_${{ matrix.gpu }}_${{ matrix.io }}
    path: atmosphere_model
```

Artifact naming convention: `atmosphere_model-{compiler}_{mpi}_{gpu}_{io}`

### Run Artifacts

Each run produces log files:

```yaml
- name: Upload log files
  uses: actions/upload-artifact@v4
  with:
    name: mpas_240km_{num_procs}rank_logs_{compiler}_{mpi}_{gpu}_{io}
    path: |
      240km_*/log.*
```

### Cleanup

Executable artifacts are cleaned up after validation to save storage:

```yaml
- name: Delete all executable artifacts
  uses: geekyeggo/delete-artifact@v5
  with:
    name: atmosphere_model-*
    failOnError: false
```

---

## Running Workflows

### GitHub Actions (Standard Runners)

For workflows using standard GitHub-hosted runners:

1. Navigate to the repository's **Actions** tab
2. Select the desired workflow
3. Click **Run workflow**
4. Choose the base OS from the dropdown
5. Click **Run workflow** to start

### CIRRUS (NCAR HPC Runners)

For workflows using NCAR's CIRRUS infrastructure:

1. Workflows run on the `CIRRUS-4x8-gpu` runner group
2. These provide access to GPU resources for CUDA testing
3. Higher resource limits for memory-intensive builds

!!! warning "Runner Access"
    CIRRUS runners require appropriate access permissions. Contact NCAR CISL for access to HPC-connected GitHub runners.

---

## Extending the CI System

### Adding a New Compiler

1. Ensure the compiler is available in CISL containers
2. Add the compiler to the workflow matrix:
   ```yaml
   compiler: [ nvhpc, oneapi, gcc, NEW_COMPILER ]
   ```
3. Update `build_mpas.sh` with the compiler target mapping:
   ```bash
   "new_compiler") compiler_target="new_target" ;;
   ```

### Adding a New Test Case

1. Prepare the test case data as a tarball
2. Add the tarball to `.github/workflows/`
3. Create a new run script or modify `run_mpas_240km.sh`
4. Update the workflow to include the new test

### Adding Validation Checks

1. Add reference outputs to `.github/workflows/validation/`
2. Update `compare_logs.py` with new validation criteria
3. Add appropriate tolerances for numerical comparisons

---

## Troubleshooting

### Common Build Issues

#### Stack Size Errors (Intel Fortran)

```bash
# Solution: Set unlimited stack size
ulimit -s unlimited
```

#### OpenMPI Root User Errors

```bash
# Solution: Add --allow-run-as-root flag
mpirun --allow-run-as-root -n 4 ./atmosphere_model
```

#### PIO Not Found

```bash
# Solution: Set PIO environment variables
export PIO_ROOT=/container/pio
export USE_PIO2=true
```

### Debugging Failed Runs

1. **Check the workflow logs** in GitHub Actions
2. **Download log artifacts** for detailed output
3. **Review the environment** using the "Interrogate Runtime Environment" step output
4. **Compare against reference** logs for expected values

---

## Related Resources

- [MPAS-Model-CI Repository](https://github.com/NCAR/MPAS-Model-CI)
- [NCAR CISL Docker Hub](https://hub.docker.com/u/ncarcisl)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Ben Kirk's Container Workflows](https://github.com/benkirk/demo_github_actions)

---

## Contributing to CI/CD

Improvements to the CI/CD system are welcome! When contributing:

1. **Test locally** using the CISL containers when possible
2. **Use workflow_dispatch** for manual testing before automating triggers
3. **Document changes** in workflow comments and this guide
4. **Consider resource usage** - minimize unnecessary builds/runs
5. **Follow the matrix pattern** for adding new configurations

For questions about the CI/CD system, please open an issue in the [MPAS-Model-CI repository](https://github.com/NCAR/MPAS-Model-CI/issues).
