---
title: 'Git: Best Practices For Better Collaboration'
date: 2021-12-04T12:01:00.001-08:00
draft: false
aliases: [ "/2022/12/6-git-bestpractices.html" ]

# post thumb
image: "https://datawrangler.mo.cloudinary.net/images/featured-post/post17.jpg"

categories:
  - "Informal"

# meta description
description: "Compilation of the best git tips and tricks to boost your productivity"

tags:
  - "Best Practices"
  - "Development"
  - "Git"

# post type
type: "featured"
---

Git - the ultimate version control system: has been an indispensable component to the developers. This article gives an overview of the best practices you can follow, to boost your productivity and gain back the lost hours.

* Always have individual branches for your environments. The developers can create branches out of the dev environment and work on their respective branches👩‍💻👨‍💻.

```
main
|
|------dev
        |--- helsinki/feature1
        |--- tokyo/feature2
                |--- nairobi/feature3 
```

* Protect 🛡️ the main and dev branches so that no one accidentally pushes or deletes the branches. Even if the branches are not protected, never 🛑 push directly to main and dev branches.

Work on your own feature branch 🌳 that is branched out of dev branch.

```
> git checkout dev
> git checkout -b helsinki/feature-2
```

* Commit and push your changes 🟩 to the remote branch 🌲 before you end your day, every evening 🗓️.

> Devices can be replaced but the lost code can never be.

* Rebase your changes 🟩 with your source branch 🌲 before you begin the day 🌅, to avoid merge conflicts.

> Feature Branch ← Source Branch (Rebase)

```
> git branch
dev
branch1
> git checkout branch1
> git rebase dev
Successfully rebased and updated refs/heads/branch1.
```

* In case of conflicts, DO NOT PANIC! 🤯🙅🏻

Why do the conflicts occur?

Merge conflicts may occur if competing changes are made to the same line of a file or when a file is deleted that another person is attempting to edit.

a. You clone from a Main branch
b. Your colleague clones from the same branch
c. He pushes his commit and merges the changes in his branch to the main branch
d. You push your commit and when trying to merge your change with the main branch, 🟥 Merge conflict occurs.

```
Feature branch → Source Branch
```

Merge conflicts can occur while rebasing too:

```
> git rebase main
Auto-merging main.py
CONFLICT (content): Merge conflict in main.py
error: could not apply cce2b78... added another statement
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply cce2b78... added another statement

> git status
interactive rebase in progress; onto bbf1e2e
Last command done (1 command done):
   pick cce2b78 added another statement
No commands remaining.
You are currently rebasing branch 'branch2' on 'bbf1e2e'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
    both modified:   main.py

> nano main.py

> git commit -m 'made changes'

> git status

interactive rebase in progress; onto bbf1e2e
Last command done (1 command done):
   pick cce2b78 added another statement
No commands remaining.
You are currently editing a commit while rebasing branch 'branch2' on 'bbf1e2e'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)

> git rebase
 --continue
Successfully rebased and updated refs/heads/branch2.

> git push origin branch2
Merge branch 'branch2' of https://github.com/datawrangl3r/conflict into branch2
To https://github.com/datawrangl3r/conflict.git
 ! [rejected]        branch2 -> branch2 (non-fast-forward)
error: failed to push some refs to 'https://github.com/datawrangl3r/conflict.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

> git pull origin branch2
hint: Pulling without specifying how to reconcile divergent branches is
hint: discouraged. You can squelch this message by running one of the following
hint: commands sometime before your next pull:
hint:
hint:   git config pull.rebase false  # merge (the default strategy)
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint:
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
From https://github.com/datawrangl3r/conflict
 * branch            branch2    -> FETCH_HEAD
Merge made by the 'recursive' strategy.

> git push origin branch2
```

* Always delete ❌ your branch upon completion of a feature request.

*Originally Published in: https://dev.to/sathyasarathi90/git-best-practices-for-better-collaboration-34ff*