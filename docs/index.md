# MPAS-A Contributors Guide

A practical reference for developers working on the
[Model for Prediction Across Scales – Atmosphere](https://github.com/MPAS-Dev/MPAS-Model)
(MPAS-A): adding physics, modifying the dycore, optimizing modules, or
extending the model.

If you're trying to *build and run* an existing release, the
[MPAS-A User's Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide.html)
is a better starting point.

The guide is organized into two sections, accessible from the sidebar:

- **Contributing** — getting set up, codebase layout, conventions, testing,
  commits, and pull requests.
- **CI/CD** — what runs in CI, how validation works, and how to extend it.
  CI itself lives in [NCAR/MPAS-Model-CI](https://github.com/NCAR/MPAS-Model-CI).

## Latest release: v8.4.0

[v8.4.0](https://github.com/MPAS-Dev/MPAS-Model/releases/tag/v8.4.0)
(April 2026) adds capabilities most relevant to contributors:

- **Large-Eddy Simulation** — diagnostic-TKE 3-D Smagorinsky and 1.5-order
  prognostic TKE, following WRF.
- **CheMPAS-A foundation** — `core_atmosphere/chemistry/musica/` stubs and a
  `&musica` namelist group, active when linked against
  [MUSICA-Fortran](https://github.com/NCAR/musica).
- **GPU-aware MPI halo exchanges** — opt-in via `config_gpu_aware_mpi` in
  `&development`.
- **Batched GPU data movement** — most fields transfer once per dynamics step
  instead of per routine.
- **Online mesh partitioning** at startup with PT-Scotch.
- **External ESMF** — new `MPAS_ESMF` build option; CMake now uses
  `manage_externals` to pin tags of MMM-physics and UGWP.
- **Configurable hybrid vertical coordinate** — `config_hybrid_coordinate` and
  `config_hybrid_top_z` in `&vertical_grid`.
- **Smaller default history files** — PV diagnostics dropped from the default
  output stream (~20% smaller).
