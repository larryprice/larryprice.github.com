---
layout: post
title: "Git out the Way - Rebase Workflow"
date: 2014-05-25 15:44:55 -0400
comments: true
categories: git
---

We use [git](http://www.git-scm.com/) on my current project, and we used to always use remote branches. After doubling the number of developers touching the repo, we found that remote feature branches led to merge conflicts, stale branches, and hidden code. We've switched away from using remote feature branches, favoring instead to commit directly to `origin/master`, making remote feature branches the exception. We do this using a simple method called `rebase`.

What does it mean to `rebase`?

When you rebase your local changes onto another branch, your changes become the head of that branch. For instance, say I start working off remote branch `master`, specifically changeset `M2`. I then make some changes and commit locally, which I'll note as changeset `L`. While I was making my changes, some other developer pushed to remote branch `master`, resulting in changeset `M3`.

```
M1 -> M2 -> M3
        \
         ->L
```

This is where `rebase` comes in. I want to rebase my local commit on top of `M3`, which will make it the new head of `master`.

```
$ git rebase origin/master
```

```
M1 -> M2 -> M3 -> L
```

Now I can keep working locally. Until I push my local `master` to `origin`, every time I `rebase` will cause all of my commits to rebase on the head of `master`.

I like to keep as close to `origin/master` as possible since we have a fairly active repository. If I want to make sure I have the bleeding edge of `master`, sometimes I don't have my local changes in working order when I want to rebase. Some people like to stash and unstash local changes, but that takes too much effort for me. I prefer just to use `git commit --amend` to constantly update the last changes I committed. You can only amend to an unpushed local commit, otherwise you'd be changing history remotely (which is bad). Using the `amend` flag allows me to keep one solid local changeset around until I'm comfortable with pushing it to `origin`. Also note that using `amend` also allows you to change the commit message of the last commit.

What happens when you need to use a remote branch?

Git's best-kept secret is `merge --squash`, a wonderful flag that gives you all the benefits of a `rebase` (linear history, fewer changesets), without all the hassle of a big fat rebase merge (rebasing infrequently can lead to merges that will haunt you in your sleep).

```
$ git branch
* local-branch
master
$ git merge master
$ git checkout master
$ git merge local-branch --squash
```

This will give you a single changeset on the head of `master` containing all the changes you made in `local-branch`, including resolution of merge conflicts. `merge --squash` has eased a lot of the headaches my team had when stuck doing rebase merges previously.
