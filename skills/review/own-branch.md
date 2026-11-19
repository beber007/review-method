# Your own branch: reviews before a merge or a push

This file is part of the skill `review`. `runs.md`, next to it, says how a run starts,
how its findings are checked, and how the log is written.

## 0. Read REVIEW.md, and run the checks

Read `REVIEW.md` and `CLAUDE.md` at the root of the repository.

`REVIEW.md` is the file that the Code Review service of Anthropic reads. The local
`/code-review` reads `CLAUDE.md`, not `REVIEW.md`. So the `CLAUDE.md` at the root of
the repository must hold a line `@REVIEW.md`, alone on its line, which brings the file
in. If that line or that file is missing, say so, and offer to add it.

If `REVIEW.md` is missing, say so. Offer to write one from `REVIEW-template.md`, the
file next to this one, filled from the repository's history: its reverts, its fixes,
its issues. Do not invent its failure cases: ask the user for those you cannot find.
Until it exists, every change that is not documentation alone takes the Deep tier.

If `REVIEW.md` lacks a section of the template, for instance one written for an
earlier version of this plugin, say which ones, and offer to add them.

Run the commands of the section "Checks before the reviews", and show their output. If
one fails, fix it, or tell the user, before any review starts.

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
So before the first full review of a change, and before the full reviews of new work
(see "What to review" in `SKILL.md`), start one fresh agent. It reads the commits that
no pass read yet: the log (`runs.md`) holds a line `texts` for each pass, with its
commit. There is no such pass for documentation alone, nor for the change of someone
else, where the agent that reads what the runs miss covers the texts. Give it the
range, the commit messages, the text of the pull request if there is one, `CLAUDE.md`
and `REVIEW.md`, and nothing else. It reads each statement that the change adds to a
text, and checks it against the code: each figure, each name, each "always", "only" or
"never", each claim on what another commit does. For each fault it gives the text, the
code that contradicts it, and a tag, as in `runs.md`. Give its findings to a second
fresh agent, which checks each one as the first agent of `runs.md` does: the author's
reading is not a check. Fix what holds, as step 4 does, and commit. Then start the
full review.

## 2. Run a full review

Commit everything first, the change and each fix, and leave nothing uncommitted. A run
reviews the range of commits it is given, but it reads the files as they are on disk.

Start the runs of `/code-review` and of the profiles as `runs.md` says: three runs of
`/code-review` for Deep, one for Standard and for documentation alone, and one run for
each profile, except for documentation alone. Then check their findings as `runs.md`
says.

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

Make one run of `/code-review` (`runs.md`) with the range of the fixes alone as
`TARGET`, such as `abc1234..HEAD`, where `abc1234` is the last commit before them.
Then start the first agent of the check of `runs.md`, without the second. Give it what
the run found, the findings the fixes answer, and the range of the fixes. It checks
each finding; that each fix removes its fault; and each statement the fixes add to a
text, against the code. It still tags as Pre-existing only what the whole change did
not bring.

This agent is not optional, even when a finding looks already checked or plain from
the code: you wrote the fixes, so your own reading of them is not a check. Tag each
finding as `runs.md` says. A false statement that a fix adds to a text is Important,
as in a full review: it does not become a Nit because it is short or easy to fix.

- If it finds something Important: fix it (step 4). Then run a full review (step 2),
  except for documentation alone: it stops once the fix is made, with no other review.
- If it finds nothing Important: fix its Nits, and stop.

## 6. When to stop

- The reviews stop at the first full review with nothing Important, once a review of
  its Nits' fixes has read them.
- After three full reviews in a row that each found something Important, stop. Ask the
  user whether the approach is the right one.

## 7. The end

Tell the user the reviews are done, with what each one found, and the commit the last
review read. What they cover is tied to that commit: a new commit needs new reviews.
Two cases do not:

- the history cleaned, with the same files in the end;
- the branch moved onto a newer main branch, without a conflict.

### Before a push

Do not push and do not merge: the user does. Set the ref of the branch (see "What to
review" in `SKILL.md`), then stop at the report.

### With a merge

Merge only with the user's agreement. That agreement covers the same commit, with the
same two cases.
