# review-method

A plugin for Claude Code. It runs the reviews of a change before it is merged, or
before it is pushed, and stops them at a clear point.

## The problem

A common way to review a pull request with agents is to review it again and again,
until a review finds nothing. On a real repository, two things went wrong.

- The last rounds found style only. A fix of style could lead to a new finding, and
  the rounds went on.
- The reviewer was told what the earlier reviews had found. A reviewer told that a
  change is sound tends to agree.

## The method

- **The reviewer is `/code-review`**, the review built into Claude Code. Each run is a
  new session, started in the repository. It does not see the author's conversation,
  so it does not get the author's view.
- **The depth follows the risk.** Three runs for the code that costs most when it
  breaks, one for the rest. Two runs on the same diff find different things, and
  their union finds more.
- **A fresh agent checks each finding.** It needs a concrete case that goes wrong, or
  it is dropped. The agent tags each finding that holds. Important: a wrong behaviour
  or a false statement. Nit: style. Pre-existing: there before the change, so it goes
  to an issue.
- **Another fresh agent reads what the runs miss**, without their findings: the texts
  against the code, the tools and the CI, and the change against its issue.
- **The reviews stop** at the first full review with nothing Important, once its nits
  are fixed and a run on those fixes has read them. After three full reviews in a row
  with something Important, the agent stops and asks the user whether the approach is
  right.
- **Reviewer profiles add runs.** A profile says what one reviewer of the team checks,
  from the reviews they gave. Each full review adds one run that reviews the change
  as that reviewer would. Its findings are checked like the others.
- **The reviews cover one commit.** A new commit needs new reviews. Before a push, the
  agent stops at the report, and the user pushes. With a merge, the agent merges only
  with the user's agreement, given on that commit.
- **A review of someone else's change changes nothing.** One pass in a separate
  worktree, the findings checked as above, and a short summary of points, each with
  a question the reviewer can take for their own comments.

The skill, [`skills/review/SKILL.md`](skills/review/SKILL.md), holds the method step by
step.

## What each repository adds

The method needs to know what failure looks like in your repository. Put a
`REVIEW.md` at its root, from
[`skills/review/REVIEW-template.md`](skills/review/REVIEW-template.md). It follows the
format that the Code Review service of Anthropic reads. Keep it to a page. Fill it from
real faults: reverts, fixes, incidents. A list of generic risks helps less.

The local `/code-review` reads `CLAUDE.md`, not `REVIEW.md`. So add this line, alone,
to your `CLAUDE.md`:

```
@REVIEW.md
```

[`examples/escapement-REVIEW.md`](examples/escapement-REVIEW.md) is one filled in, for
a small real-time operating system. On 17 November 2026, we ran `/code-review` again on
an old change of that repository, [commit
`98d8051`](https://github.com/beber007/escapement/commit/98d8051). Without this file,
three runs missed both faults the change had brought. With it, three runs together found
both.

## Install

In Claude Code:

```
/plugin marketplace add beber007/review-method
/plugin install review-method@review-method
```

Then ask Claude to run the reviews of a pull request before its merge, or of a branch
before its push, or type `/review-method:review`. A bare `/review` runs the review
built into Claude Code, not this one.

## Where it comes from

The method follows the code review of the Claude Code team, described in
[How the Claude Code team does code review](https://claude.dev/blog/solving-code-review-at-anthropic/):
tagged findings, a depth by risk, a step back after three rounds, an agreement tied
to a commit. It was first used on [Escapement](https://github.com/beber007/escapement),
in November 2026.

## License

MIT, see [`LICENSE`](LICENSE).
