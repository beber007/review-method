# Runs, checks and the log

This file is part of the skill `review`. The mode files (`own-branch.md`,
`review-only.md`) say when to use each part.

## A run of /code-review

A run is one call of `/code-review` in a new session, from the root of the repository:

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

## A run of a reviewer profile

The section "Reviewer profiles" of `REVIEW.md` may list profiles: each one a file that
says what one reviewer of the team checks, from the reviews they gave. A profile may
stay outside the repository, when it names people or quotes reviews that must not be
committed.

Each full review adds one run for each profile, whatever the tier, except for
documentation alone (see `own-branch.md`). Such a run is not `/code-review`: it is a
new session given the profile and the range, from the root of the repository:

```sh
claude -p "$(cat PROFILE)

Review the commits TARGET as the reviewer that this profile describes. Read the
code, the commit messages and what the code calls. For each fault, give the file, the
line, and a concrete case: what is given, what comes out wrong." \
    --allowedTools Read Grep Glob "Bash(git diff:*)" "Bash(git log:*)" \
    "Bash(git show:*)" TOOLS --output-format text > OUT
```

Start these runs with the others, each with its own output file. Their findings go to
the check below with the others: a profile does not make a finding hold.


## Check the findings, and read what the runs miss

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

## The log of who found what

After each check of findings (in a full review, a review of fixes or a review only),
and after the pass on the texts, add one line per finding to
`review-method/findings.tsv` under `git rev-parse --git-common-dir`. The file stays
out of the repository. Write the header line when the file is new. The columns,
separated by tabs:

- `date`, `branch`, `commit` (the commit that was reviewed);
- `review`: `texts`, `full-1`, `full-2` and so on, `fixes`, or `review-only`;
- `tag`: `Important`, `Nit`, `Pre-existing`, `Dropped` (shown wrong, without a
  concrete case, or a Nit beyond "Cap the nits"), or `None`: one line with this tag
  when the check kept and dropped nothing, so that each review has a line;
- `found_by`: every run or agent that reported it, separated by commas:
  `code-review-1` to `code-review-3`, `profile-<file name without .md>`,
  `missed-agent` (the second agent above), `check-agent` (the third agent above),
  `fixes-agent` (the agent of a review of fixes), `texts-agent` (the pass on the
  texts);
- `summary`: the fault in a few words, with no tab.

In a review only too, the log is written: it is outside the repository and the pull
request, so it is not a change.

When the user asks, or at the end of the tenth review in the log and of each tenth
after it (a review is one value of `branch`, `commit` and `review`), say for each run
or agent how many findings it reported that were kept (`Important`, `Nit` or
`Pre-existing`), and how many of them no other run reported. Count the runs
`code-review-1` to `code-review-3` together: they are the same review, run three
times. A run that never finds a kept finding alone is a candidate to remove: tell the
user, and do not remove it yourself.
