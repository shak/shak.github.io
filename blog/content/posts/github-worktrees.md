---
title: "Github Worktrees"
date: 2017-12-10T19:31:57Z
draft: false
tags: ["Github", "Development", "Tools"]
categories: ["Development"]
---

Recently I needed another copy of the git repository I was working on but it seemed like an awful lot of work cloning it another folder and then switching back and forth and fetching/pushing to keep it all up-to-date.

<!--more-->

This is where `git worktree` can be a very useful feature.

Git "worktrees" gives you a full working copy of your repository that you can treat like any other clone? So why are "worktrees" better than simply cloning repositories?

A few reasons:

- "worktrees" are hard linked, so they are really light-weight and unlike clones they don't pull the entire repository locally.
- You can access locally committed changes across different "worktrees". With cloned repositories you'd need to push the changes upstream and then fetch in a different clone where those changes are required.
- You can switch between "worktrees" without affecting changes in your current repo. So all the goodness of clones but without the additional overhead.

#### How to

To create a "worktree" simply run:

```sh
# git worktree add -b <new-branch> <directory> <upstream>/<branch>
```

[Documentation on git branch can be found here](https://git-scm.com/docs/git-worktree)

### Use Cases

Following are some use-cases from my personal experiences to give you an idea when and how can "worktrees" be useful.

#### Running tests

Before I had access to CI testing services on my testable repositories, I would create a "worktree" for branch that was ready to be merged, `cd` into the worktree directory and run the tests. In the meantime I would `cd` back to my working directory and carry on working while the tests ran in the background.

Once tests finished and I was happy with the outcome, I would merge the branch and simply delete the worktree directory. This is especially useful if you are low on `inodes` or `storage` or if that is a concern (as it was in my case).

I still sometimes do that when I want to run the tests in the background so I can continue working on code.

#### Building "Github Pages"

Those who use Github Pages would know that it is possible to publish your site from the `gh-pages` branches. Normally, to deploy changes to your Github Pages you would switch the `gh-pages` branch, build the site and then push up.

With worktrees, I create a "worktree" for the `gh-pages` branch in the `dist` folder. Then I can compile/build my site directly into the `dist` folder, `cd` into it and then push it up which will always update the `gh-pages`.
