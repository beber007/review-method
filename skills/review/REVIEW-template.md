# The reviews before a merge

<!-- One page at most. Each line here should come from a real fault of this
repository: a revert, a fix, an incident, an issue. Delete the comments once done. -->

The reviews of this repository follow the method of the plugin `review-method`. This
file holds what is local.

## What failure looks like here

<!-- The faults that have cost this repository, each in one line, with its issue or
commit when there is one. Examples of the kind:
- A migration that cannot run backwards (#123).
- A text that lies: a figure not checked against the code.
- A tool or a CI job that now skips, without a word, what it ran before. -->

-

## How deep, by risk

<!-- The paths or kinds of change of each tier. A change that touches two tiers takes
the deeper one. -->

- **Deep**, three reviewers:
- **Standard**, one reviewer: anything else.
- **Documentation alone**: one full review, then one review of its fixes.

## What to skip

<!-- Generated files, vendored code, lock files: what a reviewer need not read. -->

-

## Checks that come before the reviews

<!-- The commands that must pass before a review starts: tests, build, lint. -->

-
