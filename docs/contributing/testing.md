# Testing

Document how you tested in the PR description. Add or extend automated tests
where this project keeps them (see **Unit tests** below).

## MPAS-Model vs MPAS-Model-CI

Two different GitHub repositories are involved:

| | [**MPAS-Dev/MPAS-Model**](https://github.com/MPAS-Dev/MPAS-Model) | [**NCAR/MPAS-Model-CI**](https://github.com/NCAR/MPAS-Model-CI) |
|---|----------------------------------|----------------------------------|
| **Role** | The atmosphere model source you edit | CI only: workflows, Actions, test archives, pFUnit harness |
| **Your contribution** | Fork, branch, and open the **PR here** | You normally do **not** open model PRs in this repo |
| **GitHub Actions** | This repo has **no** workflow definitions under `.github/workflows/` | **All** automated builds, ECT/BFB runs, and pFUnit jobs are defined **here** |

So: Fortran and Registry changes live in **MPAS-Model**. **MPAS-Model-CI**
clones that repository at a ref you (or a maintainer) choose, then builds
and runs checks in containers.

## Local testing (before you open a PR)

Run the smallest case that exercises your change (see the
[User's Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide.html)).
If the change should be science-neutral, plan for **ECT**; if it must be
bitwise identical across configurations, plan for **BFB**. What those
mean in CI is described in
[Validation (ECT & BFB)](../ci-cd/validation.md).

## Automated checks (GitHub)

Jobs run from **MPAS-Model-CI**, not from your MPAS-Model PR automatically.

Typical ways your branch gets exercised:

1. **Manual dispatch** — In
   [MPAS-Model-CI → Actions](https://github.com/NCAR/MPAS-Model-CI/actions),
   run a workflow (e.g. **GNU+MPICH (CPU)**) and, where offered, set inputs
   for repository and ref.
2. **[Cross-repo test](https://github.com/NCAR/MPAS-Model-CI/blob/master/.github/workflows/test-cross-repo.yml)** —
   `mpas-repository` + `mpas-ref` point at your fork and PR branch (or any
   commit). Same pattern exists on **Ensemble Consistency Test (ECT)** for
   standalone ECT.
3. **Pushes to MPAS-Model-CI** — When NCAR updates the CI repo itself
   (`master` / `develop`), subset workflows run against the **default**
   model checkout configured in that workflow (not your PR unless inputs
   say otherwise).

When a job runs against your ref, it commonly includes multi-compiler
**MPICH** builds with a 3-member **ECT**, **NVHPC+CUDA compile-only** (no
GPU), and **pFUnit** via `unit-tests.yml`. OpenMPI variants, full GPU ECT,
BFB, and Nsight are usually **dispatch-only**. Details:
[Workflows & Triggers](../ci-cd/workflows.md).

## Unit tests (pFUnit)

The pFUnit sources and `tests/unit/` tree live **only in MPAS-Model-CI**,
not inside MPAS-Model. They run in that repo’s `unit-tests.yml`. To add a
test, follow examples like `test_spline_interpolation.pf` and coordinate a
PR against **NCAR/MPAS-Model-CI** (see [Extending CI](../ci-cd/extending.md)).

## Test types (terminology)

| Type | What it does | Use for |
|------|--------------|---------|
| **Unit** | Tests one subroutine or module | Operators, kernels, utilities |
| **Integration** | Runs a small case end-to-end | Anything touching the time loop or namelist |
| **ECT** | Statistical consistency across a perturbed ensemble | Science-neutral changes |
| **BFB** | Bit-for-bit comparison between configurations | I/O paths, MPI decomposition, refactors that must be exactly equivalent |
| **Performance** | Wall-clock and scaling | Optimization PRs |
