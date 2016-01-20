---
layout: post
title: PDF Shenanigans
date: 2016-01-20 03:13:44.000000000 -05:00
categories:
- Info
tags: []
status: publish
type: post
published: true
---

It's been several months since my last post.  I figured it was about time to take a breather and get some words out there.  

Recently I've been working on an SaaS project that involves completing and signing several documents.  It was decided that PDFs were the best way to store this information, mostly due to government requirements.  I've dealt with generating PDFs in the past and it's been a cludgy process but fairly straight forward.  This was the first time I had worked on a project that involved filling in a PDF form and affixing a signature to it.

Previously we were generating these "filled" forms by outputing HTML, but because a lot of these government forms are ridiculously tedious with a bunch of tiny checkboxes and seals and all that, we weren't able to get an exact match.  Our process was made more difficult by the fact that it was a Dreamhost shared hosting environment.  

I would love to be told I did this incorrectly and that there's an easier way.  In the end the process required:

1. Use pdftk on my local machine to enumerate the field titles 
    * `pdftk file.pdf dump_data_fields`
2. Use [php-pdftk](https://github.com/mikehaertl/php-pdftk) to merge text fields and save the file out to a temporary "merged" PDF.
    * This can be installed with composer: `composer require mikehaertl/php-pdftk`
2. Take the signature data URI and convert it to a temporary image file
    * Base64 decoding the string and storing to a file
3. Flatten the PDF
    * Involving the command-line pdftk utility
4. Use PDFI to put the image in its place.
    * Take the flattened PDF, load it up as a background for the new PDF and place the signature image on the signature line

My issues with this solution are that it requires 2 PDF libraries and using system() to call a command line program on the server.  Moreover, since we were in a shared hosting environment where I wasn't able to install the necessary binary to flatten the PDF, so I had to unpack a debian package, place the binary and library files on the server, and then use a shell script to call it: 

```
#!/bin/bash

export PATH=~/bin:$PATH;
export LD_LIBRARY_PATH=~/lib:/usr/local/lib:$LD_LIBRARY_PATH;

~/bin/pdftk.target "$@"
```

(Courtesy of [this forum post](https://discussion.dreamhost.com/thread-100286-page-2.html) ) 

If given more time, I'd like to fork an existing PDF library and roll all of this into 1 self-contained solution.  Even perusing stack overflow there doesn't appear to be a good all-encompassing solution for what I imagine has to be a very basic problem.  

I'm curious if any of you handful of blog readers out there have had the pleasure of working with PDFs in PHP.  If so, please leave a comment below.
