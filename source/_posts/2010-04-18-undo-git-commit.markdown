---
layout: post
title: "Undo Git Commit"
date: 2010-04-18 19:02
comments: false
categories: [Git]
---

I just made a commit to a public project and pushed the changes up to github.  Then I realized that I left some confidential information in one of the files.  Searching online lead to a couple of different ways to fix this (git rebase, and git filter-branch), neither of which were working well for me.

I finally figured out that doing the following would work as long as the confidential information were only included in the last commit.

First update the file and remove the confidential info.

    git add my-file-with-confidential-info
    git commit --amend

(this will add what's in your current staging area to the last commit)

    git push origin +master

(force push your changes to github)

Hope this helps someone else.