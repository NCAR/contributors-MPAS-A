# Getting Started

## What MPAS-A is

MPAS-A is the non-hydrostatic atmosphere component of the
[Model for Prediction Across Scales](https://mpas-dev.github.io/) family. All
MPAS models share a centroidal Voronoi tessellation for the horizontal mesh
and a common framework providing data types, MPI, parallel I/O, time
management, and block decomposition.

NCAR maintains the atmosphere model; LANL maintains ocean, land-ice, and
sea-ice. The code is BSD-licensed.

## Prerequisites

You'll want familiarity with Git/GitHub, modern Fortran (2003/2008), and
one MPI implementation (MPICH, OpenMPI, MVAPICH2, or MPT).

Build dependencies: NetCDF ≥ 4.4, Parallel-NetCDF ≥ 1.8, PIO 2.x; METIS
optional for offline mesh partitioning. As of v8.4.0, the CMake build
supports linking against an external **ESMF** library
(`MPAS_ESMF`) and uses `manage_externals` to fetch pinned tags of
[MMM-physics](https://github.com/NCAR/MMM-physics) and
[UGWP](https://github.com/NOAA-GSL/UGWP).

The
[MPAS-A User's Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide.html)
covers environment setup, mesh preparation, and running the model.

## Fork and clone

```bash
# Fork MPAS-Dev/MPAS-Model on GitHub, then:
git clone https://github.com/<your-username>/MPAS-Model.git
cd MPAS-Model
git remote add upstream https://github.com/MPAS-Dev/MPAS-Model.git
```

Branch from `develop` for new work — see
[Development Workflows](workflows.md).
