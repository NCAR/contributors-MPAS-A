# Commit Messages

The commit message is the only durable record of *why* a change exists.
The diff shows what changed; the message explains everything else.

Treat the first ~80 characters as if they'll appear in `git log --oneline`
forever. Use searchable terms in there, things like module or routine names,
compiler/version when relevant, kind of change, etc.

Start the subject line with a prefix that says what kind of change it is:
`debug:` for a bug fix, `optimize:` for a performance or memory improvement,
`feature:` for new functionality, or another label that fits. This answers the
"what did you actually do?" question up front and makes `git log --oneline`
easy to scan.

The body should answer:

- What was the original problem or requirement?
- What did you actually do? Is this an optimization or a new feature or a debug?
- What testing did you perform so that you're confident it's correct?
- What's the measurable impact, based on your testing? (memory saved, speedup, test that now passes)

## Example

```
debug: fix memory leak in mpas_atm_core temperature cleanup (gfortran-11.3.0)

The temperature array in mpas_atm_core_finalize was conditionally double-
deallocated when both physics and dynamics finalize ran in the same
process, leaking ~2 GB/hour on long simulations.

Discovered on a 72-hour CONUS_12km case on Derecho with 1024 ranks.
Valgrind pointed at the second deallocate; the first had already cleared
the pointer via shared cleanup logic. Guarding the second deallocate with
`allocated()` fixes it.

Validated on the same case: memory now flat, and output passes a
bit-for-bit comparison against the baseline.
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
