# Things Not to Do

A short cheat sheet of patterns that show up in PRs and slow them down.

## In the code

**Don't narrate every line.** Comments explain *why*; the code shows what.

```fortran
! Bad
temperature = 25.0   ! Set temperature to 25
```

**Don't sign your modifications.** Git already records who changed what.

```fortran
! Bad
! Modified by J.S. on 2024-01-15
```

**Don't hard-code magic numbers** —
[use named constants](standards.md#magic-numbers).

**Don't ignore compiler warnings** — they almost always indicate real
issues.

**Don't reformat unrelated code** in the same PR as a logic change. The
reviewer can't see what actually changed.

## In Git

**Don't commit directly to `master` or `develop`.** Always work on a topic
branch and open a PR.

**Don't write vague commit messages.** `"fix"`, `"update"`, `"changes"` are
useless six months later — see [Commit Messages](commits.md).

**Don't force-push to shared branches.** Force-push only on your own topic
branch, and only before substantive review.

## In review

**Don't submit a PR with no description.** Reviewers shouldn't reverse-engineer
your intent.

**Don't dismiss feedback without engaging.** "It's fine" or "that's how it's
always worked" aren't responses. Either fix it or explain why.

**Don't optimize prematurely.** A `!$acc parallel loop` around a 3-iteration
loop adds maintenance burden for zero benefit. Profile first.

**Don't skip tests for "small" changes.** Small changes are exactly where
silent regressions hide.
