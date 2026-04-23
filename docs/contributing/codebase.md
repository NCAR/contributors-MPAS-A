# Codebase Structure

## Top-level layout

```
MPAS-Model/
├── src/
│   ├── driver/        # Stand-alone driver
│   ├── external/      # External software (ESMF stubs, etc.)
│   ├── framework/     # Shared framework: data types, MPI, time
│   ├── operators/     # Shared mathematical operators
│   ├── tools/         # Registry parser, input generation
│   └── core_*/        # Individual model cores
├── testing_and_setup/
└── default_inputs/
```

Browse the live tree: [MPAS-Dev/MPAS-Model](https://github.com/MPAS-Dev/MPAS-Model/tree/develop) (`develop`).

## `core_atmosphere`

Live sources under [`src/core_atmosphere/`](https://github.com/MPAS-Dev/MPAS-Model/tree/develop/src/core_atmosphere) on GitHub.

```
core_atmosphere/
├── mpas_atm_core.F           # Main atmosphere core
├── mpas_atm_core_interface.F # Interface to the framework
├── chemistry/                # CheMPAS-A scaffolding (new in v8.4.0)
│   └── musica/               # MUSICA-Fortran integration stubs
├── diagnostics/              # Diagnostic field modules
├── dynamics/                 # Dycore (mesh, vertical solver, transport)
└── physics/                  # Parameterizations and physics drivers
```

The `chemistry/` subtree is scaffolding for CheMPAS-A; modules under
`musica/` are stubs that activate only when MPAS is linked against
[MUSICA-Fortran](https://github.com/NCAR/musica). The `&musica` namelist
group is added to `namelist.atmosphere` in that build. Options in
`namelist.atmosphere` (and other model namelist groups) are documented in
the [MPAS-A User's Guide, Appendix B: Model namelist options](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide/appB_model_namelist.html).

## Two executables

A shared codebase produces two programs:

- **`atmosphere_model`** — the forecast model (dynamics + physics)
- **`init_atmosphere_model`** — initial and lateral boundary conditions

Code that needs to differ is conditionalized in `Registry.xml`.

## The Registry

[`Registry.xml`](https://github.com/MPAS-Dev/MPAS-Model/blob/develop/src/core_atmosphere/Registry.xml)
(one per core) is the single source of truth for namelist options,
packages, dimensions, variables, streams, and structs. The build-time
parser generates Fortran from it. Adding a namelist option, field, or
output variable almost always starts with a Registry edit.
