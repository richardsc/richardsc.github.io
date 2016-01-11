---
layout:  post
title: "Forking and syncing branches with git and github"
published:  true
author: "Clark Richards"
date: 2015-02-03
categories: [git]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

When forking a branch on github, it was not entirely clear to me how to sync branches other than master (e.g. to make a pull request). The following eventually seemed to work:

## Set upstream remotes

First, you need to make sure that your fork is set up to track the original repo as `upstream` (from [here](https://help.github.com/articles/configuring-a-remote-for-a-fork/)):

List the current remotes:

```bash
$ git remote -v
# origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
# origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```

Specify a new remote upstream repository that will be synced with the fork.

```bash
$ git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```

Verify the new upstream repository you’ve specified for your fork.

```bash
$ git remote -v
# origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
# origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
# upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
# upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
```

## Syncing a fork

Now you’re ready to sync changes! See [here](https://help.github.com/articles/syncing-a-fork/) for more details on syncing a “main” branch:

Fetch the branches:

```bash
$ git fetch upstream
# remote: Counting objects: 75, done.
# remote: Compressing objects: 100% (53/53), done.
# remote: Total 62 (delta 27), reused 44 (delta 9)
# Unpacking objects: 100% (62/62), done.
# From https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY
#  * [new branch]      master     -> upstream/master
```

Check out your fork’s local master branch.

```bash
$ git checkout master
# Switched to branch 'master'
```

Merge the changes from upstream/master into your local master branch. This brings your fork’s master branch into sync with the upstream repository, without losing your local changes.

```bash
$ git merge upstream/master
# Updating a422352..5fdff0f
# Fast-forward
#  README                    |    9 -------
#  README.md                 |    7 ++++++
#  2 files changed, 7 insertions(+), 9 deletions(-)
#  delete mode 100644 README
#  create mode 100644 README.md
```

## Syncing an upstream branch

To sync upstream changes from a different branch, do the following (from [here](https://codedocean.wordpress.com/2015/02/03/forking-and-syncing-branches-with-git-and-github/)):

```bash
git fetch upstream                            #make sure you have all the upstream changes
git checkout --no-track upstream/newbranch    #grab the new branch but don't track it
git branch --set-upstream-to=origin/newbranch #set the upstream repository to your origin
git push                                      #push your new branch up to origin
```
