# Purpose

This repository contains the blog, theme and source files needed to build the static site hosted at https://darrylcauldwell.github.io.

# Usage Steps

1. If not already added [install Hugo](https://gohugo.io/overview/installing/)
2. Clone these two repository
```bash
git clone https://github.com/darrylcauldwell/darrylcauldwell.github.io.git
```
3. Switch to repository and start Hugo 
```bash
cd darrylcauldwell.github.io
hugo server
```
4. Browser to locally running copy of site typically http://localhost:1313/
5. Add or edit any aspect typically within /content/post
6. The content changes will dynamically update the ocally running copy of site
7. Once your happy with changes build static files, the following command will create static site files in /public 
```bash
hugo --destination ~/Development/darrylcauldwell.github.io/docs
```
8. Git commit and push all changes
9. Check to ensure https://darrylcauldwell.github.io reflects changes made