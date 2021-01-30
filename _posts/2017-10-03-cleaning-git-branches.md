---
toc: false
layout: post
description: How to clean git branches which aren't active anymore
categories: []
title: Cleaning merged Git branches
---
Doing some git house cleaning is something that gets neglected more often than not. With zero-cost branches in git (unlike svn), most developers creates zillions of branches during the course of their projects. Most git workflows see creation of every unit of work being put under its own branch. This means individual branches for features, bug fixes, hot fixes etc.

It's very desirable to keep our git repos clean of branches that are no longer useful/relevant. Have too many unnecessary branches also pollutes git logs since it's more difficult to locate a particular branch's head when there are hundreds of those.

Here are a couple of really quick shell commands to tame this situation

## Finding all the merged branches

First, we switch to `master` and bring in all the updates. This is **important** if we don't want to lose branches that haven't yet been merged to `master`.

```bash
git checkout master
git pull origin master
```

Then, we can list all the merged branches, both local and remote with:

```bash
git branch --merged
git branch -r --merged
```

This gives some good info of the stuff we're going to delete. We need to make sure there's nothing in here that we *don't* want to delete.

It should be noted that `master` is also listed amongst merged branches. We obviously don't want to delete that. Good ol' `grep` to the rescue.

```bash
git branch --merged | grep -E -v "(^\*|master)"
git branch -r --merged | grep -E -v "(^\*|master)"
```

The resulting lists of branches will be our candidates for deletion.

## Deleting all the merged branches

Now that we know what to delete:

```bash
git branch --merged | grep -E -v "(^\*|master)" | xargs git branch -d

git branch -r --merged | grep -E -v "(^\*|master)" | sed 's/origin\///' | xargs git push origin --delete
```

Note the addition of `sed` in our pipeline for deletion of remote branches. That's because we need to strip the `origin/` part from the names to make them compatible with the consequent `git push`.

Hope this makes it easier to keep the branch population under control.

Have a comment? Reply on [@akshagrwl](https://twitter.com/akshagrwl/status/915170192904335360).
