---
name: review
description: Run the reviews of a pull request or a branch before its merge, or before its push, until a clear point where they stop. Each review is a run of the built-in /code-review, more than one where the risk calls for it, plus one run for each reviewer profile that REVIEW.md lists; each finding is checked, tagged Important, Nit or Pre-existing, and fixed or sent to an issue. The depth follows the risk set in the repository's REVIEW.md. Use when the user asks to run the reviews before a merge or a push, to review someone else's pull request without changing it, or to write a REVIEW.md.
---

# Reviews before a merge or a push

This method runs the reviews of a change until it can be merged, or pushed, then
stops. It replaces reviewing again and again until a review finds nothing, which fails
in two ways:

- The last rounds find style only. A fix of style can lead to a new finding, and the
  rounds go on.
- A reviewer told what an earlier review found, or that the change is sound, tends to
  agree.

The reviewer is the review built into Claude Code, `/code-review`. Each run is a new
session of Claude Code, started in the repository. It does not see this conversation,
so it does not get the author's view. This method decides how many runs to make,
checks what they find, and says when to stop.

One run is not enough on code that costs a lot when it breaks. Two runs on the same
diff find different things. Their union finds more than either.

## The files of this skill

Read only the file that the case needs, from `${CLAUDE_SKILL_DIR}`:

- `own-branch.md`: the reviews of the user's own branch, before a merge or a push.
  Also when the user asks to write a `REVIEW.md` (its step 0).
- `review-only.md`: the review of the pull request of someone else, with no change
  made.
- `runs.md`: how a run starts, how its findings are checked, and the log. Both files
  above use it.
- `REVIEW-template.md`: the template of `REVIEW.md`.

## What to review

The user gives no argument, or one: an issue key such as `PROJ-123`, or a branch. The
commit of the last review of a branch is kept in the ref
`refs/review-method/<branch>`, where `<branch>` is the name without the remote, such
as `feature/x` for `origin/feature/x`. The target branch is always taken on the
remote, such as `origin/main`, after a fetch. A ref is shared by all the worktrees of
the repository, takes a `/` in its name, and keeps the commit from the garbage
collector.

- **No argument**: the current branch, written by the user. If there are changes not
  committed, ask the user to commit them first. In a detached `HEAD`, there is no
  branch: run the full reviews (`own-branch.md`, from step 0), and keep no ref. Else
  read the ref:
  - No ref: run the full reviews (`own-branch.md`, from step 0).
  - `HEAD` is the commit of the ref: say that nothing changed since the last review,
    and stop.
  - `HEAD` descends from it: ask the user once whether the new commits only fix the
    last review, or add new work. Fixes: run step 0 of `own-branch.md`, then a review
    of fixes alone (its step 5) on `<ref>..HEAD`, with the findings of the last review
    if this conversation has them, else without. New work: run the full reviews
    (`own-branch.md`, from step 0).
  - `HEAD` does not descend from it: compare the change of the branch then and now,
    each from its merge base with the target branch, with `git diff <base> <commit> |
    git patch-id --stable`, which ignores the line numbers. The same patch id: the
    history was cleaned, or the branch moved onto a newer target branch (step 7 of
    `own-branch.md`). Say that the last review still covers it, move the ref to
    `HEAD`, and stop. The same merge base but another diff: make a commit on top of
    the ref with the tree of `HEAD` (`git commit-tree HEAD^{tree} -p <ref>`), write in
    its message how each commit message changed, and keep it with the ref
    `refs/review-method-tmp/fixes`. Run a review of fixes alone on `<ref>..<that
    commit>`, then delete that ref. The review read the tree of `HEAD`, so the ref of
    the branch then moves to `HEAD`. Another merge base and another diff: run the full
    reviews (`own-branch.md`, from step 0).
- **An issue key or a branch**: fetch, then find the branch with `git branch -a --list
  "*<argument>*"`. A local branch and its remote copy are one branch: take the local
  one when it exists. If several branches still match, ask the user which one. If none
  matches, say so and stop.
  - The author: list the commits of the branch that the target branch does not hold,
    with `git log --no-merges origin/<target>..<branch>`. If the user's `git config
    user.email` is the author of each one, it is the user's branch: if it is not the
    current branch, ask the user to check it out, then go on as with no argument.
  - Otherwise, it is the change of someone else: follow `review-only.md`, on the
    commits after its ref if there is one and `<branch>` descends from it, else on the
    whole pull request.

At the end of the reviews, in every way they end, set `refs/review-method/<branch>` to
the commit that the last review read, never to a commit made for the review: `git
update-ref refs/review-method/<branch> <commit>`.
