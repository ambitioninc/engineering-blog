Contributing
============
If you'd like to contribute a post, please fork this repo and make a Pull Request on Github!


Jekyll
------
This site is built with Jekyll. If you need a primer, check out [this one](https://www.andrewmunsell.com/tutorials/jekyll-by-example/tutorial)

Two handy commands to know are
* `jekyll serve --watch`
* `jekyll build --watch`

If you run two tabs with each of these running in the blog's directory, you can easily see the changes by going to http://localhost:4000

Posts
-----
Posts can be added to the `_posts` directory and should have the format `yyyy-mm-dd-title.md`.


Frontmatter
-----------
The Frontmatter in Jekyll is a header to your markdown file specifying a few configuration options.
```
---
title: Welcome to the Engineering Blog!
layout: post
author: Micah Hausler
comments: true
tags: AmbitionSales
---
```
* Title is required
* Layout is `post` for posts and `default` for other pages
* Author is requried for posts
* Comments are optional: only enabled if set to true
* Tags are optional: Creates twitter hashtag links for each tag you add


Styling
-------
You are welcome to make the styling better. I'm starting this off with Twitter Bootstrap
