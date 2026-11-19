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

## What to review

The user gives no argument, or one: an issue key such as `PROJ-123`, or a branch.
The commit of the last review of a branch is kept in the ref
`refs/review-method/<branch>`, where `<branch>` is the name without the remote, such
as `feature/x` for `origin/feature/x`. The target branch is always taken on the
remote, such as `origin/main`, after a fetch. A ref is shared by all the worktrees of the
repository, takes a `/` in its name, and keeps the commit from the garbage collector.

- **No argument**: the current branch, written by the user. If there are changes not
  committed, ask the user to commit them first. In a detached `HEAD`, there is no
  branch: run the full reviews, from step 0, and keep no ref. Else read the ref:
  - No ref: run the full reviews, from step 0.
  - `HEAD` is the commit of the ref: say that nothing changed since the last review,
    and stop.
  - `HEAD` descends from it: ask the user once whether the new commits only fix the
    last review, or add new work. Fixes: run step 0, then a review of fixes alone
    (step 5) on `<ref>..HEAD`, with the findings of the last review if this
    conversation has them, else without. New work: run the full reviews.
  - `HEAD` does not descend from it: compare the change of the branch then and now,
    each from its merge base with the target branch, with
    `git diff <base> <commit> | git patch-id --stable`, which ignores the line
    numbers. The same patch id: the history was
    cleaned, or the branch moved onto a newer target branch (step 7). Say that the
    last review still covers it, move the ref to `HEAD`, and stop. The same merge base
    but another diff: make a commit on top of the ref with the tree of `HEAD`
    (`git commit-tree HEAD^{tree} -p <ref>`), write in its message how each commit
    message changed, and keep it with the ref `refs/review-method-tmp/fixes`. Run a
    review of fixes alone on `<ref>..<that commit>`, then delete that ref. The
    review read the tree of `HEAD`, so the ref of the branch then moves to `HEAD`.
    Another merge base and another diff: run the full reviews.
- **An issue key or a branch**: fetch, then find the branch with
  `git branch -a --list "*<argument>*"`. A local branch and its remote copy are one
  branch: take the local one when it exists. If several branches still match, ask
  the user which one. If none matches, say so and stop.
  - The author: list the commits of the branch that the target branch does not hold,
    with `git log --no-merges origin/<target>..<branch>`. If the user's
    `git config user.email` is the author of each one, it is the user's branch: if it
    is not the current branch, ask the user to check it out, then go on as with no
    argument.
  - Otherwise, it is the change of someone else: go to step 8, on the commits after
    its ref if there is one and `<branch>` descends from it, else on the whole pull
    request.

At the end of the reviews, in every way they end (steps 7 and 8), set
`refs/review-method/<branch>` to the commit that the last review read, never to a
commit made for the review: `git update-ref refs/review-method/<branch> <commit>`.

## 0. Read REVIEW.md, and run the checks

When the user reviews the change of someone else, go to step 8 now, whatever
`REVIEW.md` says: it changes how the checks below run.

Read `REVIEW.md` and `CLAUDE.md` at the root of the repository.

`REVIEW.md` is the file that the Code Review service of Anthropic reads. The local
`/code-review` reads `CLAUDE.md`, not `REVIEW.md`. So the `CLAUDE.md` at the root of
the repository must hold a line `@REVIEW.md`, alone on its line, which brings the file
in. If that line or that file is missing, say so, and offer to add it.

If `REVIEW.md` is missing, say so. Offer to write one from
`${CLAUDE_SKILL_DIR}/REVIEW-template.md`, filled from the repository's history: its
reverts, its fixes, its issues. Do not invent its failure cases: ask the user for
those you cannot find. Until it exists, every change that is not documentation alone
takes the Deep tier.

If `REVIEW.md` lacks a section of the template, for instance one written for an
earlier version of this plugin, say which ones, and offer to add them.

Run the commands of the section "Checks before the reviews", and show their output.
If one fails, fix it, or tell the user, before any review starts.

The section "The end of the reviews" of `REVIEW.md` says how the reviews end: with a
merge, or before a push. If the section is missing, they end with a merge, unless the
user asks for the reviews before a push.

## 1. Choose the tier

The section "How deep, by risk" of `REVIEW.md` lists the paths or the kinds of change
of each tier.

- **Deep**: three runs in each full review.
- **Standard**: one run in each full review.
- **Documentation alone**, a change of the documentation and nothing else, as
  `REVIEW.md` defines it: one full review of one run, then one review of its fixes.
  Fix what each one finds. Then stop.

A change that touches two tiers takes the deeper one.

### Before the first full review: the texts

In the late reviews, most Important findings are often texts that the code does not
keep: a commit message, a comment, a Kconfig help. Each one costs a full review more.
So before the first full review of a change, and before the full reviews of new
work (see "What to review"), start one fresh agent. It reads the commits that no pass
read yet: the log (end of step 3) holds a line `texts` for each pass, with its
commit. There is no such pass for documentation alone, nor in step 8, where the
agent of step 3 that reads what the runs miss covers the texts. Give it the range, the commit messages, the text of the pull
request if there is one, `CLAUDE.md` and `REVIEW.md`, and nothing else. It reads each
statement that the change adds to a text, and checks it against the code: each
figure, each name, each "always", "only" or "never", each claim on what another
commit does. For each fault it gives the text, the code that contradicts it, and a
tag, as step 3 does. Give its findings to a second fresh agent, which checks each one
as the first agent of step 3 does: the author's reading is not a check. Fix what
holds, as step 4 does, and commit. Then start the full review.

## 2. Run a full review

Commit everything first, the change and each fix, and leave nothing uncommitted. A run
reviews the range of commits it is given, but it reads the files as they are on disk.

A run is one call of `/code-review` in a new session, from the root of the
repository:

```sh
claude -p "/code-review high --max-findings default TARGET" \
    --allowedTools Read Grep Glob "Bash(git diff:*)" "Bash(git log:*)" \
    "Bash(git show:*)" TOOLS --output-format text > OUT
```

- `TARGET` is the whole change, as a range of local commits, such as
  `origin/main...HEAD`. Before a push, it starts at the main branch the change will be
  merged into, not at the remote copy of the branch, which may hold an older version
  of the change.
- `TOOLS` are those of the section "Commands a review may run" of `REVIEW.md`, such as
  `"Bash(make:*)"`. Without them, a run can read the code but not run it.
- `--max-findings default` undoes a limit set in an earlier run. For documentation
  alone, pass `--max-findings all` instead: there, every Nit of style counts.
- `OUT` is a file outside the repository, so that no report gets committed.

A new session reads the `CLAUDE.md` of the directory it starts in, and so `REVIEW.md`.
A call of the skill from your own session would read the `CLAUDE.md` of the directory
your session started in, which may be another one.

A run takes minutes. Start it in the background, and wait until it ends before you
read its file. Give the run nothing else: not your view of the change, not what an
earlier review found. For the Deep tier, start the three runs at once, each with its
own output file. If the runs may build, give each one its own worktree of the same
commit, so that they do not write over each other's files.

### The runs of the reviewer profiles

The section "Reviewer profiles" of `REVIEW.md` may list profiles: each one a file that
says what one reviewer of the team checks, from the reviews they gave. A profile may
stay outside the repository, when it names people or quotes reviews that must not be
committed.

Each full review adds one run for each profile, whatever the tier, except for
documentation alone. Such a run is not `/code-review`: it is a new session given the
profile and the range, from the root of the repository:

```sh
claude -p "$(cat PROFILE)

Review the commits TARGET as the reviewer that this profile describes. Read the
code, the commit messages and what the code calls. For each fault, give the file, the
line, and a concrete case: what is given, what comes out wrong." \
    --allowedTools Read Grep Glob "Bash(git diff:*)" "Bash(git log:*)" \
    "Bash(git show:*)" TOOLS --output-format text > OUT
```

Start these runs with the others, each with its own output file. Their findings go to
step 3 with the others: a profile does not make a finding hold.

## 3. Check the findings, and read what the runs miss

Merge the findings of the runs: two findings on the same fault are one.

Then start two fresh agents at once. Give each the whole change (the range of the full
review, even after a review of fixes), the issue the change answers, the pull
request's text (before a push, the commit messages), `CLAUDE.md` and `REVIEW.md`. Give
them nothing else: not your view of the change, nor of the findings.

- **The first checks the findings**, if there are any. Give it the merged findings
  too. With more than ten, share them out between several such agents. For each
  finding, it checks whether it holds, by reading the code and running what it can.
- **The second reads what the runs miss.** Give it no finding: a reviewer told what an
  earlier one found tends to look no further. The runs look for faults in the code,
  more than in the texts or the tools. So it reads each statement that the change adds
  to a comment, a document or the pull request's text, against the code; each change
  to a tool, a build file or the CI, for what it now skips or refuses; and whether the
  change does what its issue asks.

Each agent gives, for each fault, a concrete case: what is given, what comes out
wrong. It tags it, as the section "What Important means here" of `REVIEW.md` says:

- **Important**: a wrong behaviour, or a false statement;
- **Nit**: style, wording, an unclear referent;
- **Pre-existing**: not brought by the whole change.

Once every agent is done, drop what "Do not report" covers. Keep no more Nits in all
than "Cap the nits" allows: the ones with the clearest case first. For documentation
alone, keep them all, unless `REVIEW.md` caps them there too.

Each Important fault that the second agent found goes to a third fresh agent, given
the fault and the code, which checks whether it holds, as the first agent does.

A finding an agent shows wrong is dropped, and does not count as found. A finding
without a concrete case is dropped too. Write the reason in the pull request, or tell
the user if there is none.

### The log of who found what

After each check of findings (here, in step 5 and in step 8), and after the pass on
the texts, add one line per finding to `review-method/findings.tsv` under
`git rev-parse --git-common-dir`. The file stays out of the repository. Write the
header line when the file is new. The columns, separated by tabs:

- `date`, `branch`, `commit` (the commit that was reviewed);
- `review`: `texts`, `full-1`, `full-2` and so on, `fixes`, or `review-only`;
- `tag`: `Important`, `Nit`, `Pre-existing`, `Dropped` (shown wrong, without a
  concrete case, or a Nit beyond "Cap the nits"), or `None`: one line with this tag
  when the check kept and dropped nothing, so that each review has a line;
- `found_by`: every run or agent that reported it, separated by commas:
  `code-review-1` to `code-review-3`, `profile-<file name without .md>`,
  `missed-agent` (step 3), `check-agent` (the third agent of step 3), `fixes-agent`
  (step 5), `texts-agent`;
- `summary`: the fault in a few words, with no tab.

In step 8 too the log is written: it is outside the repository and the pull request,
so it is not a change.

When the user asks, or at the end of the tenth review in the log and of each tenth
after it (a review is one value of `branch`, `commit` and `review`), say for each run
or agent how many findings it reported that were kept (`Important`, `Nit` or
`Pre-existing`), and how many of them no other run reported. Count the runs
`code-review-1` to `code-review-3` together: they are the same review, run three
times. A run that never finds a kept finding alone is a candidate to remove: tell the
user, and do not remove it yourself.

## 4. Fix

- **Important**: fix it with its whole class, each other place of the same fault in
  the change. Where the repository has tests, write a test that fails on it first.
- **Nits**: fix them all together, in one commit.
- **Pre-existing**: do not fix it here. Propose an issue to the user.

Commit each fix. Then:

- For documentation alone, after its full review: run one review of the fixes (step
  5), whatever the full review found.
- If the full review found something Important: run a full review again (step 2).
- If it found Nits only: run one review of those fixes alone (step 5).
- If it found nothing: stop (step 6).

## 5. Run a review of fixes alone

Make one run (step 2) with the range of the fixes alone as `TARGET`, such as
`abc1234..HEAD`, where `abc1234` is the last commit before them. Then start the first
agent of step 3, without the second. Give it what the run found, the findings the
fixes answer, and the range of the fixes. It checks each finding; that each fix
removes its fault; and each statement the fixes add to a text, against the code. It
still tags as Pre-existing only what the whole change did not bring.

This agent is not optional, even when a finding looks already checked or plain from
the code: you wrote the fixes, so your own reading of them is not a check. Tag each
finding as step 3 does. A false statement that a fix adds to a text is Important, as in
a full review: it does not become a Nit because it is short or easy to fix.

- If it finds something Important: fix it (step 4). Then run a full review (step 2),
  except for documentation alone: it stops once the fix is made, with no other review.
- If it finds nothing Important: fix its Nits, and stop.

## 6. When to stop

- The reviews stop at the first full review with nothing Important, once a review of
  its Nits' fixes has read them.
- After three full reviews in a row that each found something Important, stop. Ask
  the user whether the approach is the right one.

## 7. The end

Tell the user the reviews are done, with what each one found, and the commit the last
review read. What they cover is tied to that commit: a new commit needs new reviews.
Two cases do not:

- the history cleaned, with the same files in the end;
- the branch moved onto a newer main branch, without a conflict.

### Before a push

Do not push and do not merge: the user does. Set the ref of the branch (see "What to
review"), then stop at the report.

### With a merge

Merge only with the user's agreement. That agreement covers the same commit, with the
same two cases.

## 8. Review only: the change of someone else

The user reviews a pull request that someone else wrote, and wants points to write
their own comments. Change nothing: no edit, no commit, no push, no build unless the
user asks, and nothing written in the pull request. The author fixes the change, so
steps 4 to 7 do not apply, and there is one pass only.

1. Fetch, and add a detached worktree of the head of the pull request, outside the
   user's working tree. Do not check out anything in the user's tree. When the head
   of the pull request has no `CLAUDE.md` or no `REVIEW.md`, copy the user's file into
   the worktree, so that the runs read it. When its `CLAUDE.md` has no line
   `@REVIEW.md`, add the line to the copy in the worktree. When the pull request
   changes one of them, keep its version and tell the user.
2. Start every command and every run below from the root of the worktree: a run
   reads the files on disk, and the user's tree holds other code. An agent starts
   where this session started, so give each agent the path of the worktree, and tell
   it to read the files there.
3. Run the checks of "Checks before the reviews" that only read, such as a lint. A
   failure is a finding, not something to fix.
4. Start, at once: one run of `/code-review` (step 2), whatever the tier, unless the
   user asks for more, and one run for each reviewer profile. `TARGET` is the range
   of the pull request, from its merge base with its target branch. Give the runs no
   command of `TOOLS` that builds, unless the user asks.
5. Check the findings and read what the runs miss, as step 3 does, with the text of
   the pull request. Tell the agents not to build. Keep only what an agent showed
   with a concrete case. The reasons for a dropped finding go to the user, not to the
   pull request.
6. Give the user a short summary, in the user's language:
   - Important, then Nit, within "Cap the nits". For each: the file and line, the case
     in one or two sentences, and a short question in the language of the pull
     request, in plain words, that the user can take or rewrite.
   - Pre-existing, apart: not for this pull request.
   - The points dropped, one line each with the reason, so that the user does not
     raise them.
   - The commit that was reviewed.
7. Set the ref of the branch to the commit that was reviewed (see "What to
   review"). This is not a change to the pull request. Delete the copied files, and
   check with `git status` in the worktree that nothing else is left. Then remove it with `git worktree remove`. If it holds files
   that the review made, such as a cache, use `--force`; otherwise ask the user.

If the author pushes new commits, run this step again on the range of the new
commits, or on the whole pull request if its history was rewritten.
