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

First think to do is clone the existing repository from GitHub.

```bash
git clone https://github.com/darrylcauldwell/darrylcauldwell.github.io.git
cd darrylcauldwell.github.io
```

If I check there is single branch named 'master'.

```git
git branch

* master
```

First step is to create new branch to work in and switch to it.

```
git checkout -b blog-branch

git branch

master
* blog-branch
```

Add files,  make changes within branch etc, .

```
On branch blog-branch
Your branch is up to date with 'origin/blog-branch'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        content/post/git-branch.md

no changes added to commit (use "git add" and/or "git commit -a")
```

Once we're happy with changes to the branch approve the changes and add the files to the index.

```
git add content/post/git-branch.md

git status                        
On branch blog-branch
Your branch is up to date with 'origin/blog-branch'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   content/post/git-branch.md
``` 

Then commit the changes to the local repository.

```
git commit -m 'these are changes I made'

[blog-branch 8b566a8] these are changes I made
 1 file changed, 51 insertions(+), 1 deletion(-)
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

So I make some changes, commit then and try and push using same command as earlier.

```
git commit -m 'these are changes I made'

[blog-branch 7ec976e] these are changes I made
 1 file changed, 25 insertions(+), 15 deletions(-)

git push origin master
Everything up-to-date
```

Interesting we can see a file was changed but when we try and push it suggests nothing to push. 

```
git push

Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 8 threads
Compressing objects: 100% (10/10), done.
Writing objects: 100% (10/10), 2.41 KiB | 2.41 MiB/s, done.
Total 10 (delta 7), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (7/7), completed with 3 local objects.
To https://github.com/darrylcauldwell/darrylcauldwell.github.io.git
   bf9a51c..7ec976e  blog-branch -> blog-branch
```