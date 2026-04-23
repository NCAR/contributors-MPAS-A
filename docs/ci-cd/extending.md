# Extending CI

CI changes belong in
[NCAR/MPAS-Model-CI](https://github.com/NCAR/MPAS-Model-CI), not
MPAS-Model.

## Guiding principles

1. **Reuse before duplicating.** New parallel job logic almost always
   means something should be factored into a reusable workflow or
   composite action.
2. **Centralize variable settings** in
   [`ci-config.env`](https://github.com/NCAR/MPAS-Model-CI/blob/master/.github/ci-config.env)
   rather than hard-coding in YAML.
3. **Match existing trigger policy.** GPU and self-hosted-runner workflows
   stay `workflow_dispatch` only.
4. **Update this contributors guide** for changes contributors should know
   about.

## Adding a test resolution

```bash
# 1. Build the archive
tar czf 60km.tar.gz namelist.atmosphere streams.atmosphere <mesh files>

# 2. Create a release with the archive as an asset
gh release create testdata-60km-v1 --repo NCAR/MPAS-Model-CI 60km.tar.gz

# 3. Add to ci-config.env
RELEASE_TESTDATA_60KM=testdata-60km-v1

# 4. Point the workflow(s) at the new resolution
```

## Adding a BFB caller

Copy `bfb-io.yml`, `bfb-decomp.yml`, or one of the GPU variants. Rename,
set the trigger, and edit the `variants` array. For GPU BFB, also set
`gpu: 'true'` and `compiler: nvhpc`. Don't add `pull_request` triggers to
GPU workflows.

## Adding a compiler / MPI combination

1. Confirm an `hpcdev` image exists for the combination
   ([Docker Hub tags](https://hub.docker.com/r/ncarcisl/hpcdev-x86_64/tags)).
   Add `CONTAINER_COMPILER_*` if the tag uses a versioned name.
2. Add `MAKE_TARGET_<compiler>` if the Makefile target name differs.
3. Copy an existing caller and change the `compiler`/`mpi` inputs.
4. Trigger policy: CPU + MPICH → `push`/`pull_request`; anything new or
   known-flaky → `workflow_dispatch` until it stabilizes.

## Adding a unit test

Place pFUnit tests under `tests/unit/` in MPAS-Model-CI. They run as part
of `unit-tests.yml`. Follow existing tests like
`test_spline_interpolation.pf`.

## Cleanup

Anything that produces large artifacts should be removed by a cleanup job
or get a short retention period on upload.
