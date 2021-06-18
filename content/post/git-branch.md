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
Five years ago I moved my [personal blog to GitHub Pages](/post/jekyll). Up to now, my workflow has been to clone to local repository make changes directly to master branch, build static website files, test locally and then push all changes upstream for GitHub pages to build. This simple workflow has worked perfectly however recently I find myself working across multiple devices and on multiple posts simultaneously. 

As such I have moved to use GitHub branching in my workflow to assist meet the demands of my evolved working situation.

## Create new branch for post

First think to do is clone the existing repository from GitHub.

```bash
git clone https://github.com/darrylcauldwell/darrylcauldwell.github.io.git
cd darrylcauldwell.github.io
```

If I check there is single branch named 'master'.

```bash
git branch

* master
```

First step is to create new branch to work in and switch to it.

```bash
git checkout -b blog-branch

git branch

master
* blog-branch
```

Add files,  make changes within branch etc, .

```bash
On branch blog-branch
Your branch is up to date with 'origin/blog-branch'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        content/post/git-branch.md

no changes added to commit (use "git add" and/or "git commit -a")
```

Once we're happy with changes to the branch approve the changes and add the files to the index.

```bash
git add content/post/git-branch.md

git status                        
On branch blog-branch
Your branch is up to date with 'origin/blog-branch'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   content/post/git-branch.md
``` 

Then commit the changes to the local repository.

```bash
git commit -m 'these are changes I made'

[blog-branch 8b566a8] these are changes I made
 1 file changed, 51 insertions(+), 1 deletion(-)
```

Once happy push the branch to remote upstream repository on GitHub.

```bash
git push origin blog-branch
```

## Make changes to existing branch

The workflow is slightly different if I have created a branch and want to return to work on it.  To simulate this if I delete the folder with local repository and create new clone from GitHub again.

```bash
git clone https://github.com/darrylcauldwell/darrylcauldwell.github.io.git

Cloning into 'darrylcauldwell.github.io'...
remote: Enumerating objects: 7361, done.
remote: Counting objects: 100% (5996/5996), done.
remote: Compressing objects: 100% (1761/1761), done.
remote: Total 7361 (delta 2975), reused 5678 (delta 2660), pack-reused 1365
Receiving objects: 100% (7361/7361), 66.43 MiB | 5.85 MiB/s, done.
Resolving deltas: 100% (3667/3667), done.
```

When creating new clone only the 'master' branch, we can see the branch on remote/origin.

```bash
git branch -a

* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/blog-branch
  remotes/origin/master
```

When look to switch local respository to branch with identical name the client is intelligent enough to link the local and remote and mark to track.

```bash
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

From here we can continue making changes, commit the files to local repository and then try and push them to remote as before.

## In-branch blog build and testing

So far I have been updating blog content in markdown files. The Hugo engine takes these markdown files and creates all of the pages required for static website. Prior to posting it is good to test the post formatting looks good, the image links work etc. From within the branch we can run Hugo and use browser to test.

```bash
hugo server

Start building sites … 

                   | EN   
-------------------+------
  Pages            | 242  
  Paginator pages  |  27  
  Non-page files   |   0  
  Static files     | 274  
  Processed images |   0  
  Aliases          |  86  
  Sitemaps         |   1  
  Cleaned          |   0  

Built in 205 ms
Watching for changes in /Users/darrylcauldwell/Development/darrylcauldwell.github.io/{archetypes,content,layouts,static,themes}
Watching for config changes in /Users/darrylcauldwell/Development/darrylcauldwell.github.io/config/_default, /Users/darrylcauldwell/Development/darrylcauldwell.github.io/config/_default/menus
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

In my configuration GitHub pages is configured to use the /docs folder.  When we've tested and all looks good can then look to rebuild the static website files into that folder.

```bash
hugo --destination ~/Development/darrylcauldwell.github.io/docs

Start building sites … 

                   | EN   
-------------------+------
  Pages            | 242  
  Paginator pages  |  27  
  Non-page files   |   0  
  Static files     | 274  
  Processed images |   0  
  Aliases          |  86  
  Sitemaps         |   1  
  Cleaned          |   0  

Total in 263 ms
```

As we can see from the output adding a single post actually rebuilds many pages. It isn't really practical to add each to repository manually but we can use wildcard and add all.

```bash
git add .
git commit -m 'new post on github branching'

[blog-branch e202461] new post on github branching
 189 files changed, 4147 insertions(+), 2184 deletions(-)
 create mode 100644 docs/post/git-branch/index.html

git push origin blog-branch

Enumerating objects: 646, done.
Counting objects: 100% (646/646), done.
Delta compression using up to 8 threads
Compressing objects: 100% (277/277), done.
Writing objects: 100% (382/382), 202.55 KiB | 4.71 MiB/s, done.
Total 382 (delta 192), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (192/192), completed with 79 local objects.
To https://github.com/darrylcauldwell/darrylcauldwell.github.io.git
   da04b9c..e202461  blog-branch -> blog-branch
```

So at this stage we've got out local and remote branch in sync.

```bash
git branch -a -v  

* blog-branch                e202461 new post on github branching
  master                     fb51479 heading formats
  remotes/origin/HEAD        -> origin/master
  remotes/origin/blog-branch e202461 new post on github branching
  remotes/origin/master      fb51479 heading formats
```

## Merge blog post branch to master
Once we're happy that the new post branch is good we can look to merge that into 'master' and be processed by GitHub Pages.

As this is single user repository I could just use 'git merge' to merge the branch. Merging branches in multi-user repository the merge might come via a merge pull request which allows for change review. In the past I'd typically made pull requests via the web UI. I recently found the new git cli 'gh' has capability to create pull-request.

```bash
gh pr create --title "New post about github branching" --body "New post about github branching with github pages" -H blog-branch

Creating pull request for blog-branch into master in darrylcauldwell/darrylcauldwell.github.io
https://github.com/darrylcauldwell/darrylcauldwell.github.io/pull/2
```

As pull requests require independant review I cannot approve my own pull request but I can add some comments.

![GitHub Pull Request Review](/images/git-branch-pr.png)

I can then look to merge.

![GitHub Pull Request Merge](/images/git-branch-pr.png)

When we're merged into 'master' branch can look to remove the branch both locally and remotely.

```bash
git branch -d blog-branch
git push origin -d blog-branch

To https://github.com/darrylcauldwell/darrylcauldwell.github.io.git
 - [deleted]         blog-branch

git branch -a

* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```
