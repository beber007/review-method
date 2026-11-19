# Review only: the change of someone else

This file is part of the skill `review`. `runs.md`, next to it, says how a run starts,
how its findings are checked, and how the log is written.

The user reviews a pull request that someone else wrote, and wants points to write
their own comments. Change nothing: no edit, no commit, no push, no build unless the
user asks, and nothing written in the pull request. The author fixes the change, so
there is no fix and no new round: one pass only.

1. Fetch, and add a detached worktree of the head of the pull request, outside the
   user's working tree. Do not check out anything in the user's tree. When the head of
   the pull request has no `CLAUDE.md` or no `REVIEW.md`, copy the user's file into
   the worktree, so that the runs read it. When its `CLAUDE.md` has no line
   `@REVIEW.md`, add the line to the copy in the worktree. When the pull request
   changes one of them, keep its version and tell the user.
2. Start every command and every run below from the root of the worktree: a run reads
   the files on disk, and the user's tree holds other code. An agent starts where this
   session started, so give each agent the path of the worktree, and tell it to read
   the files there.
3. Run the checks of "Checks before the reviews" that only read, such as a lint. A
   failure is a finding, not something to fix.
4. Start, at once, as `runs.md` says: one run of `/code-review`, whatever the tier,
   unless the user asks for more, and one run for each reviewer profile. `TARGET` is
   the range of the pull request, from its merge base with its target branch. Give the
   runs no command of `TOOLS` that builds, unless the user asks.
5. Check the findings and read what the runs miss, as `runs.md` says, with the text of
   the pull request. Tell the agents not to build. Keep only what an agent showed with
   a concrete case. The reasons for a dropped finding go to the user, not to the pull
   request.
6. Give the user a short summary, in the user's language:
   - Important, then Nit, within "Cap the nits". For each: the file and line, the case
     in one or two sentences, and a short question in the language of the pull
     request, in plain words, that the user can take or rewrite.
   - Pre-existing, apart: not for this pull request.
   - The points dropped, one line each with the reason, so that the user does not
     raise them.
   - The commit that was reviewed.
7. Set the ref of the branch to the commit that was reviewed (see "What to review" in
   `SKILL.md`). This is not a change to the pull request. Delete the copied files, and
   check with `git status` in the worktree that nothing else is left. Then remove it
   with `git worktree remove`. If it holds files that the review made, such as a
   cache, use `--force`; otherwise ask the user.

If the author pushes new commits, run this file again on the range of the new commits,
or on the whole pull request if its history was rewritten.
