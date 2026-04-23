# Commit Messages

The commit message is the only durable record of *why* a change exists.
The diff shows what changed; the message explains everything else.

Treat the first ~80 characters as if they'll appear in `git log --oneline`
forever — because they will. Get grep-able terms in there: module or
routine names, compiler/version when relevant, kind of change.

The body should answer:

- What was the original problem or requirement?
- What did you actually do?
- What surprised you while debugging? (compiler quirks, environment, edges)
- What's the measurable impact? (memory saved, speedup, test that now passes)

## Example

```
Fix memory leak in mpas_atm_core temperature cleanup (gfortran-11.3.0)

The temperature array in mpas_atm_core_finalize was conditionally double-
deallocated when both physics and dynamics finalize ran in the same
process, leaking ~2 GB/hour on long simulations.

Discovered on a 72-hour CONUS_12km case on Derecho with 1024 ranks.
Valgrind pointed at the second deallocate; the first had already cleared
the pointer via shared cleanup logic. Guarding the second deallocate with
`allocated()` fixes it.

Validated on the same case (memory now flat); all standard ECT subsets pass.
```

## Searching history

Well-structured messages turn `git log --grep` into a real tool:

```bash
git log --grep="mpas_atm_core"          # changes to a module
git log --grep="gfortran"               # compiler-specific work
git log --grep="OpenACC\|GPU\|CUDA"     # GPU-related changes
```

## Anti-patterns

- `Fix bug` / `Update` / `Changes` — communicates nothing
- A 200-line diff under a one-line subject — the body is where value lives
- Unrelated changes in one commit — splits the history's signal
- Authorship comments inside the code (`! Modified by ABC on …`) — git tracks that
