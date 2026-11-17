---
name: review
description: Run the reviews of a pull request or a branch before its merge, until a clear point where they stop. Fresh agents see the code, not the author's view; each finding is tagged Important, Nit or Pre-existing, with a proof; the depth follows the risk set in the repository's REVIEW.md. Use when the user asks to run the reviews before a merge, or to write a REVIEW.md.
---

# Reviews before a merge

This method runs the reviews of a change until it can be merged, then stops. It
replaces reviewing again and again until a review finds nothing, which fails in two
ways:

- The last rounds find style only. A fix of style can lead to a new finding, and the
  rounds go on.
- A reviewer told what an earlier review found, or that the change is sound, tends to
  agree.

The method is the same in every repository. What failure looks like differs: each
repository says it in a file `REVIEW.md` at its root.

## 0. Read REVIEW.md, and run the checks

Read `REVIEW.md` at the root of the repository, and `CLAUDE.md` if there is one.

If `REVIEW.md` is missing, say so. Offer to write one from
`${CLAUDE_SKILL_DIR}/REVIEW-template.md`, filled from the repository's history: its
reverts, its fixes, its issues. Do not invent its failure cases: ask the user for
those you cannot find. Until it exists, every change that is not documentation alone
takes the Deep tier.

Run the checks that `REVIEW.md` lists under "Checks that come before the reviews",
and show their output. If one fails, fix it, or tell the user, before any review
starts.

## 1. Choose the tier

`REVIEW.md` lists the paths or the kinds of change of each tier.

- **Deep**: three reviewers in each full review, at once. Each looks at one side:
  - the logic of the code;
  - the facts that the texts state (comments, documentation, the pull request);
  - the tools and the CI that build and test the code.
- **Standard**: one reviewer in each full review.
- **Documentation alone**: one full review, then one review of its fixes. Fix what
  each one finds. Then stop.

A change that touches two tiers takes the deeper one.

## 2. Brief a full review

Start each reviewer as a fresh agent. Give it:

- the issue or the request the change answers;
- how to get the whole diff (for instance `git diff origin/main...HEAD`);
- the pull request's text, if there is one;
- `CLAUDE.md` and `REVIEW.md`.

Do not give it your view of the change. Do not give it what an earlier review found.

Ask for a report that starts with the number of findings. Each finding gives a file
and a line, a concrete case (what is given, what comes out wrong), and a tag:

- **Important**: a wrong behaviour, or a false statement.
- **Nit**: style, wording, an unclear referent.
- **Pre-existing**: not brought by the diff.

A finding without a concrete case is not one.

## 3. Check each Important finding

Before fixing an Important finding, start one more fresh agent. Give it the finding
and the code, and ask whether it holds. A finding it shows wrong is dropped, and does
not count as found. Write the reason in the pull request, or tell the user if there
is none.

## 4. Fix

- **Important**: fix it with its whole class, each other place of the same fault in
  the diff. Where the repository has tests, write a test that fails on it first.
- **Nits**: fix them all together, in one commit.
- **Pre-existing**: do not fix it here. Propose an issue to the user.

Then:

- For documentation alone: run one review of the fixes (step 5), whatever the full
  review found.
- If the full review found something Important: run a full review again (step 2).
- If it found Nits only: run one review of those fixes alone (step 5).
- If it found nothing: stop (step 6).

## 5. Brief a review of fixes alone

Start one fresh agent. Give it the diff of the fixes and the findings they fix. It
checks each fix and what the fix changed around it, not the whole change again.

- If it finds something Important: check it (step 3) and fix it (step 4). Then run a
  full review (step 2), except for documentation alone: it stops once the fix is made.
- If it finds nothing Important: fix its Nits, and stop.

## 6. When to stop

- The reviews stop at the first full review with nothing Important, once a review of
  its Nits' fixes has read them.
- After three full reviews in a row that each found something Important, stop. Ask
  the user whether the approach is the right one.

## 7. The merge

Tell the user the reviews are done, with what each one found. Merge only with the
user's agreement. That agreement covers the commit it was given on: a new commit needs
a new agreement. Two cases do not:

- the history cleaned, with the same files in the end;
- the branch moved onto a newer main branch, without a conflict.
