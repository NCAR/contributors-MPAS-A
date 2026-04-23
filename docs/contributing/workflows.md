# Development Workflows

## Branches

Work happens on a topic branch off `develop`:

```bash
git checkout develop
git pull upstream develop
git checkout -b feature/short-descriptive-name
```

Use prefixes (`feature/`, `bugfix/`, `optimize/`, `gpu/`) so the purpose is
obvious from `git branch -a`.

## New features

Identify which cores, modules, and Registry entries you'll touch before
writing code. Mirror existing patterns: new physics schemes follow the
existing ones in `core_atmosphere/physics/`; new diagnostics live under
`diagnostics/`.

Default namelist values should preserve current behavior. Adding a field
to the default `output` stream needs justification — v8.4.0 *removed* PV
diagnostics and cut history file size by ~20%.

User-facing changes need an update to the
[User's Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide.html);
internal interfaces use [FORD](standards.md#ford-documentation). Tests are
covered in [Testing](testing.md).

## Bug fixes

Reproduce the bug with a minimal failing case before fixing it. Note the
compiler and version in the commit message if it's compiler-specific. Make
the smallest change that works — don't bundle refactors. Add a regression
test or, at minimum, a CI variant that would have caught it.

## Optimization

Profile first. Don't change anything for a hotspot worth less than ~5% of
total time. Establish a baseline (runtime, memory, scaling on a
representative case), optimize one axis at a time, and measure after each
change. Verify correctness with ECT or BFB —
[Validation](../ci-cd/validation.md).

## GPU acceleration (OpenACC)

The dycore is fully ported to OpenACC. v8.4.0 added two patterns worth
following in new GPU code:

- **Batched data movement.** Most fields transfer once at the start and end
  of a dynamics step rather than per routine.
- **GPU-aware MPI halo exchanges.** Set `config_gpu_aware_mpi = .true.` in
  `&development` when linked against a GPU-aware MPI to skip host staging.

Use explicit `enter data`/`exit data` regions at routine boundaries with
`present(...)` clauses on parallel loops:

```fortran
!$acc parallel loop collapse(2) present(field, mesh)
do iCell = 1, nCells
    do k = 1, nVertLevels
        ! computation
    end do
end do
!$acc end parallel loop
```

Build with `OPENACC=true`. Validate against the CPU output with ECT, then
measure speedup. The `compile-nvhpc-cuda-mpich` workflow checks OpenACC
syntax on every PR without a GPU; full GPU runs are dispatch-only —
[GPU & Profiling](../ci-cd/gpu.md).
