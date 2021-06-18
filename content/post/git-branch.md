+++
title = "GitHub Pages Git Branching Workflow"
date = "2021-06-18"
description = "GitHub Pages Git Branching Workflow"
tags = [
    "github"
]
categories = [
    "blogging"
]
thumbnail = "clarity-icons/code-144.svg"
+++
Five years ago I moved my [personal blog to GitHub Pages](/posts/jekyll). Up to now, my workflow has been to clone to local repository make changes directly to master branch, build static website files, test locally and then push all changes upstream for GitHub pages to build. This simple workflow has worked perfectly however recently I find myself working across multiple devices and on multiple posts simultaneously. 

As such I have moved to use GitHub branching in my workflow to assist meet the demands of my evolved working situation.

## Clone GitHub repository and start local instance

```bash
git clone https://github.com/darrylcauldwell/darrylcauldwell.github.io.git
cd darrylcauldwell.github.io
hugo server
```

Open browser to locally running copy of site typically http://localhost:1313/

## Create branch for new post OR edit of existing

List all current branches

```git
git branch

* master
```
Create new branch to work in and switch to it

```
git checkout -b blog-branch
```

List all current branches and note active branch

```
git branch

master
* blog-branch
```

Add files,  make changes within branch,  it could be easy to lose track of what changes you've made so we can view changes.

```
git status
```

Once we're happy with changes to the branch approve the changes and add the files to the index.

```
git add .
``` 

Then commit the changes to the local repository.

```
git commit -m 'these are changes I made'
```

Once happy push the branch to remote upstream repository on GitHub.

```
git push origin master
```

I can continue to work on this local repository branch and keep pushing.  However if I delete the local repository or switch devices I might need to create clone from GitHub again.

```
git clone https://github.com/darrylcauldwell/darrylcauldwell.github.io.git

Cloning into 'darrylcauldwell.github.io'...
remote: Enumerating objects: 7361, done.
remote: Counting objects: 100% (5996/5996), done.
remote: Compressing objects: 100% (1761/1761), done.
remote: Total 7361 (delta 2975), reused 5678 (delta 2660), pack-reused 1365
Receiving objects: 100% (7361/7361), 66.43 MiB | 5.85 MiB/s, done.
Resolving deltas: 100% (3667/3667), done.
```

However when I check branches of local repository I see only 'master'.

```
git branch  
* master
```

If I run same command to view all I can see the remote branch.

```
git branch -a

* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/blog-branch
  remotes/origin/master
```

If I try to switch to the branch the client is intelligent enough to link the local and remote and mark to track.

```
git checkout blog-branch

Branch 'blog-branch' set up to track remote branch 'blog-branch' from 'origin'.
Switched to a new branch 'blog-branch'

git branch -a           

* blog-branch
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/blog-branch
  remotes/origin/master
```

