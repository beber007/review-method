# Review instructions

The first four sections are for each review. The last five are for the plugin
[`review-method`](https://github.com/beber007/review-method).

<!-- One page at most. Each line here should come from a real fault of this
repository: a revert, a fix, an incident, an issue. Delete the comments once done.

The first four sections follow the format of the Code Review service of Anthropic,
which reads this file. The local /code-review reads it too, once CLAUDE.md holds the
line @REVIEW.md. -->

## What Important means here

<!-- The faults that have cost this repository, each in one line, with its issue or
commit when there is one. Examples of the kind:
- A migration that cannot run backwards (#123).
- A text that lies: a figure not checked against the code.
- A tool or a CI job that now skips, without a word, what it ran before. -->

Important means a wrong behaviour, or a false statement. Here, that is above all:

-

Style, naming and rewording are Nit at most.

## Cap the nits

Report at most five Nits per review, except for a change of documentation alone:
there, report every one.

## Do not report

<!-- Generated files, vendored code, lock files, what the CI already checks. -->

-

## Always check

<!-- The rules a review must check on every change, such as "each new route has a
test". Keep the ones the history shows are missed. -->

-

## How deep, by risk

<!-- For the plugin review-method. The paths or kinds of change of each tier. A
change that touches two tiers takes the deeper one. -->

- **Deep**, three runs of the review:
- **Standard**, one run: anything else.
- **Documentation alone**, a change of Markdown files only: one run, then one run on
  its fixes.

## Commands a review may run

<!-- For the plugin review-method. The commands a run of /code-review may use, as
rules of --allowedTools, such as "Bash(make:*)". Reading the code and git are always
allowed. -->

-

## Checks before the reviews

<!-- For the plugin review-method. The commands that must pass before a review
starts: tests, build, lint. -->

-

## Reviewer profiles

<!-- For the plugin review-method. Optional. One file for each reviewer of the team
whose reviews a run should follow: what they check, each point from a review they
gave, with its issue. A file that names people or quotes internal reviews can stay
outside the repository: give its absolute path. Delete the section if there is none. -->

-

## The end of the reviews

<!-- For the plugin review-method. "With a merge": the agent merges, with the user's
agreement. "Before a push": the agent stops at the report, and the user pushes. The
review of someone else's pull request always ends with a summary, with no change
made, whatever this section says. -->

With a merge.
