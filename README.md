# review-method

A plugin for Claude Code. It runs the reviews of a change before it is merged, and
stops them at a clear point.

## The problem

A common way to review a pull request with agents is to review it again and again,
until a review finds nothing. On a real repository, two things went wrong.

- The last rounds found style only. A fix of style could lead to a new finding, and
  the rounds went on.
- The reviewer was told what the earlier reviews had found. A reviewer told that a
  change is sound tends to agree.

## The method

- **A reviewer sees the code, not the author's view.** It gets the issue, the diff,
  the pull request's text and the repository's rules. Nothing else.
- **Each finding has a tag.** Important: a wrong behaviour or a false statement. Nit:
  style. Pre-existing: there before the change, so it goes to an issue.
- **Each finding has a proof.** A file, a line, and a concrete case that goes wrong.
- **The depth follows the risk.** Three reviewers for the code that costs most when it
  breaks, one for the rest, a lighter review for documentation alone.
- **A fresh agent checks each Important finding** before it is fixed. A wrong one is
  dropped.
- **The reviews stop** at the first full review with nothing Important, once its nits
  are fixed and a review of those fixes has read them. After three full reviews in a
  row with something Important, the agent stops and asks the user whether the approach
  is right.
- **The agreement to merge covers one commit.** A new commit needs a new agreement.

The skill, [`skills/review/SKILL.md`](skills/review/SKILL.md), holds the method step by
step.

## What each repository adds

The method needs to know what failure looks like in your repository. Put a
`REVIEW.md` at its root, from
[`skills/review/REVIEW-template.md`](skills/review/REVIEW-template.md). Keep it to a
page. Fill it from real faults: reverts, fixes, incidents. A list of generic risks
helps less.

[`examples/escapement-REVIEW.md`](examples/escapement-REVIEW.md) is one filled in, for
a small real-time operating system.

## Install

In Claude Code:

```
/plugin marketplace add beber007/review-method
/plugin install review-method@review-method
```

Then ask Claude to run the reviews of a pull request before its merge, or type
`/review-method:review`. A bare `/review` runs the review built into Claude Code, not
this one.

## Where it comes from

The method follows the code review of the Claude Code team, described in
[How the Claude Code team does code review](https://claude.dev/blog/solving-code-review-at-anthropic/):
tagged findings, a depth by risk, a step back after three rounds, an agreement tied
to a commit. It was first used on [Escapement](https://github.com/beber007/escapement),
in November 2026.

## License

MIT, see [`LICENSE`](LICENSE).
