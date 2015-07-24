---
layout: post
title: Bash Scraping
date: 2015-03-20 03:01:01.000000000 -05:00
categories:
- Bash
tags: []
status: publish
type: post
published: true
---

Recently I wanted to scrape a dictionary list for import to <a href="https://wordpress.org/plugins/name-directory/" target="_blank">this name directory plugin</a>. Ordinarily I use PHP for scraping web pages (possibly covered in a future blog entry), but this time the task was relatively easy and the page had dictionary list tags (&lt;dl&gt;&lt;dt&gt;...&lt;/dt&gt;&lt;dd&gt;...&lt;/dd&gt;&lt;/dl&gt;) so I thought it would be easier to use already available text manipulation tools to build the CSV file for import.

The first task was to use wget to download the page:

```
wget http://example.com/glossary.php
```

After this, we have to do two steps to pull out the terms and the definitions each.  First, we have to find the &lt;dt&gt;&lt;/dt&gt; or &lt;dd&gt;&lt;/dd&gt; strings.  After we have that information, we'll remove any markup and add quotes to the beginning and end of the lines.

```
grep -o '<dd>.*</dd>' glossary.php | sed -e 's/<[^>]*>//g;s/.*/"&amp;"/' > definitions.txt
grep -o '<dt>.*</dt>' glossary.php | sed -e 's/<[^>]*>//g;s/.*/"&amp;"/' > terms.txt
```

Grep pulls out the text that we're interested in.  After that we pipe the output to sed, where we strip out anything that begins with '<' and ends with '>'.  Using '.*' would be greedy. A pattern like '<.*>' would match as much as it could and would remove everything between the beginning and ending HTML tags including the tags themselves.  By using '[^>]*' we're telling sed to continue UNTIL it encounters a '>' character.  The second command (s/.*/"&amp;"/) we use makes sure to add quotes to the beginning and end of the lines.

Lastly, we'll mash the two files together and match up each line from the files and separate them with commas (for our CSV).  Thankfully there's already a command for this called 'paste'.  There's a '-d' option that lets us specify the delimiter.

```
paste -d, term.txt definition.txt > output.csv
```

Now, output.csv has all the data we want to import with the format: "term","definition".

That's it!  Short and sweet.  Enjoy!
