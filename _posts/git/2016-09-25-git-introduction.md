---
layout: post
title:  "Git introduction"
description: 'Introduction to git comands and basic workflow'
slug: 'git-introduction'
date:   2016-09-25 12:01:06 +0300
categories: git
tags: [github, git, version control]
icon: git
---

When you start a new project you need an easy way to track your changes and I want to introduce you to **GIT**.
In this post I will just talk about the basics of it.

### What is GIT?
Git is a distributed source controll repository. Today is by far, the most widely used modern version control system in
the world. Git was created in 2005 by Linus Torvalds, the creator of Linux operating system. Git is used as version
control system for all kind of software projects, commercial as well as open source.

### Why to use git?

1. **Save time** - Git is lightning fast. Use your time for something more useful than waiting for your version control
system to get back to you.
2. **Work offline** - With git you can work while you're on the move, on you local machine. You can add files, create
commits, merge branches, check the history.
3. **Undo mistake** - Because we are humans we do mistakes, but with git you can "undo" almost any kind of mistake. You
can correct your last commit if you forgot to add a file or a change, revert an entire commit because there is a bug in
it, and when you delete something and you go crazy because you actualy needed that file, restore it from the "Reflog".
4. **Don't worry** - As you already may remark git gives you that confidence that you can't mess things up. This is
because git mostly adds data and rarely will actualy delete.

### How to install and configure git?

Most people are using git from the command line and because day by day I'm using linux (Ubuntu dist), in this tutorial
I'll show you all the command line tools you need to know, so that you can begin using git for your own projects.
Because I usually work on linux, I'll just show you how to install git on Ubuntu, but don't worry, if you
google it, I'm pretty sure you'll find how you can install it for your operating system.

```
$ sudo apt-get update
$ sudo apt-get install git
$ git --version
$ git version 2.7.4
```

Like most of the command line tools git has a help system that you can use by typing `$ git help` into your terminal. A
list with commands will be printed for you, but if you need to be more specific you can use `$ git help` and then the
command you need more information about. For example `$ git help config` will print the help for `config` command.

The first step when you install git is to set up some basic configurations. You have to configure your username
and email, so that when you create commits with awesome code, to take some credit for it.

```
$ git config --global user.name 'Iulian Popa'
$ git config --global user.email 'iulyanpopa@gmail.com'
```

Also in order to have some nice colors on our command line we have to configure git with the following command.

```
$ git config --global color.ui true
```

To get started with git, let's create a new directory `$ mkdir blog && cd blog` and run the command `$ git init` in
that directory. This command will just initialize the local git repository for us, it's not stored up on any server it's
just on your local computer, precisely in that `.git` directory.

### Basic Git Workflow

One of the most useful git command is `$ git status`. This command helps you see what files have your changed since your
last commit. Soon you'll realize that the `status` command will be your **best friend** when you work with git.

Let's say that you want to create a `README.md` file as your first file into your project `$ touch README.md`. Then if
you check the status, you'll see that you have a new untracked file.

```
On branch master
Your branch is up-to-date with 'origin/master'.
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	README.md

nothing added to commit but untracked files present (use "git add" to track)
```

In order for git to know that it needs to track that new file, you need to make use of another command as it is specified
in the last message from the status output: `$ git add`. The `add` command will add the file to the staging area, and if
you check again the status you'll see that the file it's actually staged.

```
$ git add README.md
$ git status

On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   README.md

```

> **Note!** The `$ git status` command will output a lot of info about your changes. After a while you'll know what
you're doing and you may want to have a more simple output. You can accomplish that with `-s` **silent** flag on status
command.

```
# The `A` character in front of the file name means that the file was staged.
$ git status -s
A README.md
```

After we've staged the file we are ready to create our first **commit**.

```
$ git commit -m 'Create readme file.'
[master 6f874cb] Create readme file
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md

$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
nothing to commit, working directory clean
```

As you can see if you check the status, we are on the branch `master` and there is nothing else to commit, there are no
other changes or files that we modified and we didn't commit.

We will talk about branching in another post, for now, all you need to understand is that at the moment we have only one
timeline named `master`.

Next I'll let you create a **LICENSE** file and also modify the **README.md** file. Then check the status, stage the
changes and create a new commit. Now that you've created the second commit I want to show you another command that helps
you to check the history of the commits, by listing all the commits.

```
$ git log

commit 6f874cb07a20c86213f00d033f308c2ffe595bd9
Author: Iulian Popa <iulyanpopa@gmail.com>
Date:   Tue Sep 27 22:38:11 2016 +0300

    Add license and finish readme file

commit 5a637eb97558289cc9955880672a49d78d230dca
Author: Iulian Popa <iulyanpopa@gmail.com>
Date:   Tue Sep 27 22:21:27 2016 +0300

    Create readme file

```

As you can imagine, when you're working on a real project, the commit messages are very important, that's why you need
to try to be as descriptive as possible to what they do.

Until now we've been working with just our own local repository on our computer. It's time to push this code out to the
cloud, so others can use it or collaborate on it. Because git doesn't take care of access control other services as
Github, Bitbucket or Gitorious have been created. The former one needs to be hosted by you. Mainly what those services
provide for you is a way to specify that this user has access to just this repository or this group of users have access
to this repository. I encourage you to use [Github](http://github.com) or [Bitbucket](http://bitbucket.com) as they are
hosted services and free for public repositories. Bitbucket is free also for private repositories.

In order for you to share your local repository with other users you should "push" it to, let's say, Github (the most
popular). You can accomplish this with `$ git push` command. To use the command you need first to create a Github
account and a new repository on Github. When you create a new repository on Github you'll have a repository link that
looks like this `https://github.com/iulyanp/git-tutorial.git`. The next step is to tell git about your remote
repository you just created on Github.

```
$ git remote add origin https://github.com/iulyanp/elixir-bundle.git
```
> **Note!** Usually we are referring to our official repository as **origin**, that's why I named it like that.

If you want to see a list of remotes that git knows about you can run:

```
$ git remote -v

origin	git@github.com:iulyanp/git-tutorial.git (fetch)
origin	git@github.com:iulyanp/git-tutorial.git (push)
```

Awesome! Now let's push our code to that repository from Github `$ git push` command. To do that we have to specify the
remote repository name which is "origin" and the branch name which off course is "master".
Don't worry, we'll talk about branches in another post.

```
# Push the master branch to github
## origin = remote repository
## master = the 'master' branch that we want to push to Github
$ git push origin master
```

> **Note!** When you run this command git is asking you for your Github credentials.

Congratulations, your first repository is now up on Github, go and refresh the Github repository page and you'll see
your files there.
Now let's imagine you're working within a team, with your friend and he wants to contribute to your repository. In order
for him to get your code from Github on his local computer, first you have to give him access to the repository from the
Github repository settings. He will need then to use another git command `$ git clone` to copy your code. So he will go
on Github page copy the link to your repository and clone it locally.

```
$ git clone https://github.com/iulyanp/git-tutorial.git awesome-project
```

The clone command will copy the entire project from Github repository into *awesome-project* directory on his local
computer, because he had specified the directory where he wanted the repository to be cloned.
Git would create a new directory named **git-tutorial** and copy the project in it if you omit the last parameter in the
command. Now he has your project on his computer, he can change/add/delete files, create commits and push them up to
Github.

After he pushed some changes and you're back working on the project first thing you need to do is to check if you have
any changes on Github that are not on your computer. With git you just have to run `$ git pull` command and you're back
on track.

```
$ git pull origin master
```
Now you can see you have also your friend code on your local computer and you can continue working.

> **Note!** You'll find out soon that not all the time life it's that easy and sometimes you'll get conflicts in your files
because you've worked locally and your friend modified the same files you changed on your machine.
In this situation, when you try to pull the changes on your computer, you'll see some *merge conflicts*.
You have to solve every single conflict, take all the files that contain conflicts and fix them one by one.
Then use `$ git add` and `$ git commit` to commit those solved conflicts, push the changes on Github and you're ready to
code again.

### Other tips

#### Git configuration

```
# Show colors improve readability
$ git config --system color.ui true

# Open the global configuration file in a text editor for manual editing
$ git config --global –-edit

#Create a shortcut for a Git command.
$ git config --global alias.<alias-name> <git-command>

#Configure the git status in silance mod. -b flag will show you also the branch you're on.
$ git config --global alias.sb “status -sb”
```

#### Git add

```
# When you want to stage all the changes you can use
$ git add --all

# When you want to stage all the changes you can use
$ git add .

# when you want to add everything from a directory
$ git add directory/

# when you want to add all .php files from the current directory
$ git add *.php

# when you want to add all th .php files from the entire project
$ git add "*.php"

$ git commit -am 'Stage and commit all files: tracked and untracked'
```

#### Git log

```
# Display your git logs in a more readable way and create an alias for it
## --oneline = prints the log just in one line
## --decorate = decorates the log with the branches
## --all =
## --graph = creates a nice litle graphic for you in the console
$ git config --global alias.lg “log --oneline --decorate --all –-graph”
```

#### Git commit

```
# Auto stage the files have been modified or deleted and include them into the commit.
$ git commit -a

# Amend a commit with a new file or change you forgot to include previously.
$ git commit --amend
```

### Conclusions

Congratulations, you learned the basic git workflow. You can go and start using git on your awesome projects.

#### Git basic workflow

1. Add/edit files
2. Stage the changes
3. Create a new commit
4. Repeat from step 1 :)
5. Push your code on remote repositories
6. Pull others code on your computer
7. Repeat from step 1 :)

With time you'll learn to trust git and you'll not worry that you could mess things up.
Keep in mind that whatever you do, whatever bad you might think you broken your project with git it almost never the case.
If you liked this post and you learned something from it or maybe it's the other way around, don't hesitate to leave me
a feedback.
