# Pull Requests & Review

## Opening a PR

Before pushing, confirm:

- The feature or fix is complete (no `TODO` markers in code paths reviewers
  care about).
- Tests added or updated; evidence attached (local runs and/or
  **MPAS-Model-CI** dispatch / cross-repo runs) — see [Testing](testing.md).
- The User's Guide is updated if you changed namelist options, stream
  defaults, or build flags.

## Description template

```markdown
## Summary
What this PR changes, in 2–3 sentences.

## Type
- [ ] Bug fix · [ ] Feature · [ ] Performance · [ ] Docs · [ ] Refactor

## Testing
- [ ] MPAS-Model-CI subset / cross-repo run passed (or equivalent evidence)
- [ ] ECT (if science-affecting) / BFB (if exactly equivalent)
- [ ] Performance numbers attached (if optimization)

## Related issues
Closes #NNN
```

For namelist or output changes, mention new namelist group/option names and
their defaults — that's exactly what downstream users grep for.

## Reviewing

When reviewing, check:

- **Correctness** — logic matches stated intent? Edge cases? Units?
- **Tests** — does the test actually exercise the change? Could it pass
  even if the change were reverted?
- **Integration** — does it break Registry parsing, default streams, or
  existing namelist defaults?
- **Documentation** — public interfaces have FORD docstrings; namelist
  additions are reflected in the User's Guide PR or issue.

Be specific in comments — quote the line and propose the change. Reserve
"blocking" feedback for correctness and test gaps; mark style nits as such.

## After review

Address each comment with either a code change or a reply explaining why
the existing approach is correct. Don't squash review feedback into a
single "address comments" commit — keep fix commits visible. **Don't
force-push** unless rebasing onto a moved `develop`, and leave a note when
you do.
