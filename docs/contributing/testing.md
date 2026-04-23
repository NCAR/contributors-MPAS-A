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

## Automated testing (MPAS-Model-CI)

Automated testing is being developed in
[**NCAR/MPAS-Model-CI**](https://github.com/NCAR/MPAS-Model-CI), a **GitHub
fork** of [**MPAS-Dev/MPAS-Model**](https://github.com/MPAS-Dev/MPAS-Model)
where workflows, containers, and test assets live separately from the
upstream model tree. For what runs today, how triggers are wired, and how
to invoke jobs, see the [**CI/CD overview**](../ci-cd/index.md) in this
guide.

Start jobs from
[MPAS-Model-CI Actions](https://github.com/NCAR/MPAS-Model-CI/actions), or
see [Cross-repo testing](../ci-cd/workflows.md#cross-repo-testing) to run
against a specific fork and ref.
