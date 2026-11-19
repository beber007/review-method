---
name: review
description: Run the reviews of a pull request or a branch before its merge, or before its push, until a clear point where they stop. Each review is a run of the built-in /code-review, more than one where the risk calls for it, plus one run for each reviewer profile that REVIEW.md lists; each finding is checked, tagged Important, Nit or Pre-existing, and fixed or sent to an issue. The depth follows the risk set in the repository's REVIEW.md. Use when the user asks to run the reviews before a merge or a push, or to write a REVIEW.md.
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

## 0. Read REVIEW.md, and run the checks

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

Do not push and do not merge: the user does. Stop at the report.

### With a merge

Merge only with the user's agreement. That agreement covers the same commit, with the
same two cases.
