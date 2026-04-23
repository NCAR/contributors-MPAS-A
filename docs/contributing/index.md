# Contributing Overview

This section is for developers working on MPAS-A itself — adding physics,
modifying the dycore, optimizing modules, or extending the model. If
you're trying to *build and run* an existing release, the
[MPAS-A User's Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide.html)
is a better starting point.

The pages below follow a typical contributor's path, from first checkout
to merged PR.

## Orientation

- [Getting Started](getting-started.md) — prerequisites, build dependencies,
  fork & clone.
- [Codebase Structure](codebase.md) — top-level layout, `core_atmosphere`,
  the Registry.

## Daily work

- [Development Workflows](workflows.md) — branches, features, bug fixes,
  optimization, GPU acceleration.
- [Code Standards](standards.md) — Fortran conventions, FORD docstrings,
  formatting.
- [Things Not to Do](things-not-to-do.md) — common patterns to avoid.
- [Testing](testing.md) — what CI runs and what to add for new code.
- [Commit Messages](commits.md) — what to write in `git log`.

## Review and help

- [Pull Requests & Review](pull-requests.md) — opening and reviewing PRs.
- [Getting Help](help.md) — issue templates, support forum, code of conduct.

## CI/CD

For what runs in CI, validation strategy, and how to extend it, see the
[CI/CD section](../ci-cd/index.md). CI itself lives in
[NCAR/MPAS-Model-CI](https://github.com/NCAR/MPAS-Model-CI).
