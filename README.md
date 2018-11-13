# FreeBSD Graphics Blog

[freebsddesktop.github.io](https://freebsddesktop.github.io)

## How-to

### Step 1: Add Author

If you are new, add yourself as author in [_data/authors.yml](https://github.com/FreeBSDDesktop/freebsddesktop.github.io/blob/master/_data/authors.yml).

### Step 2: Write post

Create a new file in `_posts` folder.  
Filename syntax is `"YYYY-MM-DD-my-title.md"`

If desired, add an optional header at the top of the file.
```
---
title: My Title
author: johalun
---
```

If no title is supplied, title will be derived from the filename.  
If no author is supplied, the post will be anonymous.  
  
Write the post using Github markdown language.

### Step 3: Publish

Github pages will automatically rebuild the blog web site when this repository is updated (i.e. file saved using Github web UI or committed from command line). The new post will appear on the [blog web site](https://freebsddesktop.github.io) within 10 minutes (usually within one minute).


## Misc

More information here: [Using Jekyll with GitHub Pages](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/).
