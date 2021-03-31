+++
title = "Migrate Wordpress to GitHub Pages"
date = "2016-06-29"
description = "Migrate Wordpress to GitHub Pages"
tags = [
    "jekyll"
]
categories = [
    "blogging"
]
thumbnail = "clarity-icons/code-144.svg"
+++

There are the steps I used to migrate my blog from Wordpress to GitHub Pages,  first I will try and describe why I am doing this.

## Migrate To Learn
Learning new technical skills is straight forwards but while building any new skill building a muscle memory for command syntax is only achieved by taking as many opportunities to exercise the muscle as possible.

I found at my old job I was more and more using distributed source control more and more to help my day to day work, since changing job this required much less. The goal at my employer is to use this in the near future so editing my personal blog is a good way to keep learning and building knowledge with github. By migrating my blog to GitHub Pages and Jekyll, publishing a blog post becomes committing changes and pushing them to GitHub.

Albeit distributed source control is used much less in my new role writting infrastructure configuration code is still very much required. I've moved away from Sublime Text and been starting to use the excellent Visual Studio Code at work. I write much less code at home on Mac and struggle with the shortcut key differences. By maintaining my personal blog as a series of files which I use a code editor will help me with this.

While working with Ansible I use YAML formatted files, using Jeykll and passing variables seems a great way to keep using and learning.

## Migration Overview

1. Install Jekyll
2. Create new Jekyll project
3. Create GitHub Pages site
4. Export Wordpress posts as Jekyll
5. Test posts on Jekyll (Local)
6. Commit exported posts to GitHub Pages

## Install Jekyll
The [Jekyll install](https://jekyllrb.com/docs/installation) requires Ruby and Ruby Gems be in place before starting. 
These are included with OS X so we can just pen terminal and run the following to install.

```bash
sudo gem install jekyll
sudo gem install jekyll-paginate
```

## Create new Jekyll project

A Jekyll project is just a collection of files so first its worth creating a folder to house the project then running a Jekyll command 
to create the framework files.

```bash
mkdir Website
cd Website/
jekyll new darrylcauldwell.com
```

## Create GitHub Pages site

Create a personal GitHub repository called *projectname*.github.io,  for example <*>darrylcaudwell.github.io<*>.

Initialise your Jekyll website project folder as a local git repository, add all the files to the local repository, link the local 
git repository to github.com, create first commit and then push this to master.

```bash
git init
git add .
git remote add github git@github.com:darrylcauldwell/darrylcauldwell.github.io
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/darrylcauldwell/darrylcauldwell.github.io.git
git push -u origin master
```

By performing this commit should trigger the creation of your SSL secured GitHub Pages website in my example the URL is.
https://darrylcauldwell.github.io

## Export Wordpress posts as Jekyll

So by this stage we have the website running with our chosen theme and we're now ready to migrate across all old posts from wordpress.

Within existing wordpress site install the ['WordPress2Jekyll'](https://wordpress.org/plugins/wp2jekyll/) plugin and use this to export the site as a zip file.

```bash
cd yourproject
jekyll build
git add .
git commit -m "imports old blog posts"
git push -u origin master
```

## Redirect URL
Once your happy that your website is fully funcional on the URL *yourproject*.github.io you can then add a DNS alias to point your domain
name to the new site.  After a few days if your still happy you can then remove your account from previous hosting company.