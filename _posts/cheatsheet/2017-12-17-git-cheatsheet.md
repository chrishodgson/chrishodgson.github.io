---
layout: post
title: Git Cheatsheet
summary: 
categories: [cheatsheet]
tags: [git]
links:
lastmod: 2017-12-17T07:31:30+00:00 
---

0. Rename local branch (not yet pushed to origin)
    - `git branch -m new-name` From same branch 

0. Rename local and remote branch (already pushed to origin) 
    - `git branch -m new-name` From same branch 
    - `git push origin :old-name new-name` Delete old-name remote branch and push new-name local branch
    - `git push origin -u new-name` reset upstream branch for new-name local branch
    
0. `git branch --set-upstream-to origin/BRANCH` set upstream branch for local branch before doing a push with `git push origin -u BRANCH`
