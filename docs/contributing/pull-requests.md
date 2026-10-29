# Pull Requests & Review

## Opening a PR

Before you open a PR, check that:

- The feature or fix is complete.
- You've tested the change and can describe how — see [Testing](testing.md).
- The User's Guide is updated if you changed namelist options, stream
  defaults, or build flags.

## Description template

```markdown
## Summary
What this PR changes, in 2–3 sentences.

## Type
- [ ] Bug fix · [ ] Feature · [ ] Performance · [ ] Docs · [ ] Refactor

## Testing
- [ ] Tests you ran, and how you know the change is correct
- [ ] Performance numbers attached (if optimization)

## Related issues
Closes #NNN
```

If you add namelist or output changes, please mention the new group/option
names and their defaults — downstream users search for exactly these.

## Reviewing

When you review, look at:

- **Correctness** — logic matches stated intent? Edge cases? Units?
- **Tests** — does the test actually exercise the change? Could it pass
  even if the change were reverted?
- **Integration** — does it break Registry parsing, default streams, or
  existing namelist defaults?
- **Documentation** — public interfaces have FORD docstrings; namelist
  additions are reflected in the User's Guide PR or issue.

Specific comments are the most helpful: quote the line and propose a change.
Reserve "blocking" for correctness and test gaps, and label style nits as nits.

## After review

Please respond to each comment, either with a code change or a short
explanation of why the existing approach works. Keep fix commits visible
rather than squashing review feedback into one "address comments" commit.
**Avoid force-pushing** unless you're rebasing onto a moved `develop`, and
leave a note if you do.
