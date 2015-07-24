---
layout: post
title: Jekyll and Git Hooks
date: 2015-07-19 03:13:44.000000000 -05:00
categories:
- DevOps
tags: []
status: publish
type: post
published: true
---

I've been using WordPress for a while now and I've always enjoyed how easy it is to develop custom themes and plugins for it.  Recently a couple of colleagues expressed concerns with using WordPress for some projects that I'm involved with.  WordPress is quick and easy to get up and running and if you need some modifications it's easy to edit themes and create plugins, however, I've recently been introduced to Jekyll and with the help of git hooks I would argue that it's even easier to add and manage content with Jekyll and Git Hooks than with WordPress AND it has the added benefit of not requiring a database or even PHP.

[Jekyll](http://jekyllrb.com/) is one of many [static-site generator](https://www.staticgen.com/).  It's written in Ruby and requires no server-side logic.  You install Jekyll on your local machine and you use it to compile several configuration files and template files into a static web site.  The Jekyll website has really good documentation on installing it on your computer for Mac, Windows, and Linux, so that won't be covered in this article.

After installing Jekyll, I browsed for a theme to get started.  I found one site with several Jekyll themes [here](http://jekyllthemes.org/).  The glorious part about these Jekyll themes is that they are the functioning website (not just a piece to be added) and most of them are hosted on Github.

For our Linux User Group ([website](http://www.evlug.com) - [Github](https://github.com/EV-LUG)), we started out with the [minimal-block theme](https://github.com/drvy/minimal-block).  The first step was to clone the repo:

```
abbadon:test jklem$ git clone https://github.com/drvy/minimal-block
 Cloning into 'minimal-block'...
 remote: Counting objects: 99, done.
 remote: Total 99 (delta 0), reused 0 (delta 0), pack-reused 99
 Unpacking objects: 100% (99/99), done.
 Checking connectivity... done.
abbadon:test jklem$ cd minimal-block/
abbadon:minimal-block jklem$ ls
 404.html README.md _includes _posts contact feed.xml preview.png
 LICENSE.md _config.yml _layouts about css index.html
```

The [Jekyll site](http://jekyllrb.com/docs/structure/) has excellent documentation about the directory structure, so I won't go into detail but it's incredibly easy to get going.  All blog posts are added in `_posts` as markdown files.  If you want to create additional pages, you create a directory and add an index.html file to that directory (in the minimalist-block example they have a 'contact' and 'about' page).

After you've made any changed and added any content you want, you can test things out with the following:

```
abbadon:minimal-block jklem$ jekyll serve
```

This builds all your files and then listens on localhost:4000 so you can view things in your browser.  If you don't want to preview anything, you can simply build your site with the following:

```
abbadon:minimal-block jklem$ jekyll build
```

Once the site is built, by default Jekyll puts all your static files in the _site directory, which you can then manually transfer to your server (if you'd like).  I found the magic was in using a git hook to automagically publish the site.

You can create git hooks by adding them in '.git/hooks/'.  There are specific names that are expected and all the files have to be executable to be used.  There's some more awesome documentation [here](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks).  For our purposes we want to use 'post-commit' and it's a simple shell script:

```
#!/bin/sh

# --destination is optional, if it's ommitted, the default _site folder will be used
jekyll build --source /path/to/your/jekyllsite/ --destination /path/to/your/output/site/

#  if you're hosting your site on an s3 bucket you'd use the following:

# aws s3 sync /path/to/your/output/site/ s3://yourBucket

#  the above assumes the aws's CLI tool has been configured
#  there's some easy-to-follow documentation at this url:
#  http://docs.aws.amazon.com/cli/latest/userguide/installing.html

# you can also scp the files over
scp -rv /path/to/your/jekyllsite/* youruser@yourserver.com:/path/to/webroot

# The sky's the limit, if you can script it you can deploy it.  It doesn't have to be just scp or aws
```

You want to make sure that your 'post-commit' script is executable.  The above example uses scp to deploy the site and has comments for using aws cli, but you can use any utility you'd like.  Once that's in place, all you have to do is add any changes to your repository and then commit them.  After the commit is finished, the post-commit script is executed and this will push your site to the interwebs.  Magick!

I plan on converting my blog to a static site (so I don't have to worry about maintaining WordPress anymore).  I'm eyeing [Pelican](http://blog.getpelican.com/) over Jekyll for my blog personally since it can import existing WordPress content.  But that's content for a different post.
