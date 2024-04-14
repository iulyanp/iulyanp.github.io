---
layout: post
title:  "Git branch"
description: 'Introduction to git brancing'
slug: 'git-branch'
date:   2016-10-24 12:01:06 +0300
category: git
tags: [git, branch, github, version control]
icon: code-fork
---


When we are working with git, it's always a good idea to use branches to encapsulate our work until we finish the development. Up until now we had worked only into the base branch named `master` but it's time to learn how to make our own branches.
All we have to do to is to create a branch, create commits to that new branch and when we are ready we can merge it back into the `master` branch.

### What is a branch?

Technically a branch is nothing else but a pointer to a specific commit. When you create a branch a pointer is created and every time you commit something on that branch the pointer moves to the new commit. In other words a branch is used to group and isolate a bunch of commits.

### How to create a branch?

In order to create a new branch named `some-feature` we use:

```
$ git branch some-feature
```

The branch is created but we are not on that branch yet. In order to go on that branch and start working on some new awesome feature, all we need to do is to check it out.

```
$ git checkout some-feature
```

> Note! If you want to create and jump directly on the new branch you can use `$ git checkout -b some-feature`.

### Renaming a branch

There will be a time when you will notice you have wrongly named the branch and you'll want to rename it.

```
$ git branch -m old-name new-name
```
> Note! If you want to rename the branch you're currently on you just have to run `$ git branch -m new-name` and you'll have it renamed.

### Merging a branch

After we finish working on the new feature we want to get the branch and merge it into the `master` branch.

```
$ git checkout master
$ git merge some-feature
```

### Delete a branch.

Now that we merged the branch into `master` we want to delete it. To do that we can use:

```
$ git branch -d some-feature
```

This would not work if we didn't merged the branch, because git is trying to protect us from deleting a branch which might contain some changes we need.
But if we know what we do and we are sure that we want to delete the branch we can use `-D` flag and the branch will be deleted even if it wasn't merged.

```
$ git branch -D some-feature
```

### Conclusion

Git branching functionality is very powerfull, if we want to test someting out and after that delete everything we can create a new branch, work there until we are satisfied and after that delete the branch and nothing will be saved.

Hope you liked it and you're ready for more git insides because in the next post we will talk about `$ git stash` command, so stay tuned.

