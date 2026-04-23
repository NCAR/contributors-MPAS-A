# Testing

Did you test your changes? Please at least include the test in your commit/PR documentation, and try to contribute the test if possible.

## Test types

| Type | What it does | Use for |
|------|--------------|---------|
| **Unit** | Tests one subroutine or module | Operators, kernels, utilities |
| **Integration** | Runs a small case end-to-end | Anything touching the time loop or namelist |
| **ECT** | Statistical consistency across a perturbed ensemble | Science-neutral changes |
| **BFB** | Bit-for-bit comparison between configurations | I/O paths, MPI decomposition, refactors that must be exactly equivalent |
| **Performance** | Wall-clock and scaling | Optimization PRs |

See [Validation (ECT & BFB)](../ci-cd/validation.md) for what those checks
actually compare.

## Unit tests

[pFUnit](https://github.com/Goddard-Fortran-Ecosystem/pFUnit) tests live
under `tests/unit/` in
[MPAS-Model-CI](https://github.com/NCAR/MPAS-Model-CI), run via
`unit-tests.yml` across GCC 12/13/14. Follow existing tests like
`test_spline_interpolation.pf` for style.

## What CI runs on your PR

Automatic on push and PR to `master` or `develop`:

- **CPU subsets** — GCC, Intel, NVHPC builds with MPICH plus 3-member ECT
- **NVHPC + CUDA compile-only** — catches GPU toolchain breakage without a GPU
- **pFUnit unit tests**

Pushes to `master` also run **coverage**. OpenMPI variants, full GPU ECT,
BFB, and Nsight profiling are dispatch-only.

Branches matching `hackathon-*` auto-run the CPU ECT subsets — the easiest
way to run the full gauntlet before opening a PR. Otherwise trigger from
[MPAS-Model-CI Actions](https://github.com/NCAR/MPAS-Model-CI/actions).
