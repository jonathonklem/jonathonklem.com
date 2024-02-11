---
layout: post
title: DKIM/SPF and Cloudflare
date: 2024-02-09 03:13:44.000000000 -05:00
categories:
- development
tags: []
status: publish
type: post
published: true
---

In response to the new dkim/spf requirements implemented by Google and Yahoo, I've been having to do a lot more work with DNS recently.

Thankfully tools such as [mxtoolbox](https://mxtoolbox.com/spf.aspx) and [dmarcian](https://dmarcian.com/dmarc-inspector/) make inspecting and generating SPF and DMARC records a breeze.  However, this weekend there was an issue that was stumping me. 

Emails were not going through and we kept getting `DKIM = did not pass\n550-5.7.26  SPF [x.y.z] with ip: [a.b.c.d] = did not pass\n550-5.7.26\n550-5.7.26` however, the domain name AND ip address were both explicitly provided in the SPF record and our DKIM was passing.  There were no syntax errors.

It turns out that because it was behind Cloudflare, a little extra configuration was required.  I only discovered this after looking at [atfgundb's](https://atfgundb.com) DNS since it is also using Cloudflare.  I hadn't had any problems over there sending transactional emails.  It turns out that we needed to include this in our SPF record: `include:_spf.mx.cloudflare.net`.

As soon as that was included, emails started working.  This isn't the most involved of fixes/updates, but I couldn't find this documented anywhere on Cloudflare and google was not helping.  Hopefully this information can help someone in the future who may be experiencing the same problem.
