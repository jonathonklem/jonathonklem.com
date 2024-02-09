---
layout: post
title: Go & React & MongoDB, OH MY!
date: 2024-02-09 03:13:44.000000000 -05:00
categories:
- development
tags: []
status: publish
type: post
published: true
---

While tidying things up and redoing the Jekyll install, I was reminded of [this post from 2016](/development/releasing-apps).  I'm not sure if I was just incredibly green or if the process has gotten significantly easier but releasing apps to the App and Play store is not nearly as big an ordeal as it once was. Thankfully, xcode even manages keys for you so iOS is arguably easier to work than Android.  However, deploying Android apps is still a straight-forward process.

In it's infinite wisdom, the YouTube algorithm has been promoting a lot of developer channels such as [The Primeagen](https://www.youtube.com/@ThePrimeTimeagen). At first it seemed harmless, but slowly it started to reignite my passion for learning new tech.  Slowly I became convinced that I needed to dive head-first into some React and Go code. 

Thanks to [this article](https://www.kantega.no/blogg/running-go-and-react-on-aws-using-lambda) I also decided to throw caution to the wind and just Leroy Jenkins my way into some serveless architecture as well. 

Eventually, I ended up with an architecture that looked like this:

<img src="/assets/architecture.jpg" style="max-width: 100%">

You can see some promo videos of what the app does [here](https://youtu.be/ydSN48bS5js) and [here](https://youtu.be/k8vZI_3OW4g).  Additionally you can check the website out as [atfgundb.com](https://atfgundb.com).

This was an incredibly fun project to work on.  Some of the major challenges as someone who only had a little experience with React and no experience with Go:
- **Marshalling JSON in Go** - I've been spoiled by PHP's `json_decode()` and weak typing, however I know the weaker typing is a double-edged sword so I appreaciate Go's stronger (but still not oppressively so) typing.
- **Communicating with Auth0 and handling JWT tokens** - This was a new one for me.  Once I understood some of the basic concepts, the Auth0 API wasn't hard to work with, but initially all of the concepts of token claims and audiences were entirely foreign.
- **Communicating state in a larger React application** - I'm not a stranger to `useState()` and React hooks, however, this project involved several components that all worked in tandem.  This was my first introduction to Contexts with React.  I still think I could improve the way this works a bit more, but it's significantly better than how I first started shoving things into one giant component and passing the various set() functions to child elements.
- **Thinking in a NoSQL manner** - Coming from MySQL, I love joins and seek to leverage them whereever possible.  Additionally, I find MySQL aggregate functions like `SUM()`, `COUNT()`, `AVG()`.. very intuitive.  It was a learning experience to transition to Mongo's way of handling this.  I got pretty familiar with `$match`, `$group`, `$lookup`, and `$project`

After overcoming all of the original engineering challenges, I realized it's no good just having my app out there in the ether.  I want people to install it and use it.  So I started running an Adwords campaign as well as doing Facebook promotions.  The only way to verify that the adspend is worthwhile is to guage installs and user interaction.  So now I needed a way to track user installs and verify that users were active.  Additionally I needed a way to handle cases of malicious activity and to prevent abuse.

So then I added a management system written in Laravel...  I typically favor CakePHP for my MVC/CRUD needs, however, Laravel has been growing in popularity for quite some time.  After using it, I can see why.  I really like CakePHP but Laravel's Artisan Console is amazing.  I'm especially fond of the tinker function.  Maybe an analog to this exists in CakePHP or the upcoming CakePHP 5, but I'm not aware of it.  Rather than having to throw print_r()s or other debugging methods into code you're trying to flesh out, tinker lets you call functions directly and observe their output.

I also prefer the Larvel method for handling cron tasks.  Rather than calling them manually from outside the application, you can document everything and when it needs to run in app/Console/Kernel.php.  This is handy because you can actually track changes.

All-in-all this was an incredibly fun project to work on and I look forward to learning more as I maintain it.  If you're looking for a solution to track your ammo, gun collection, range sessions, and maintenance, head over to [atfgundb](https://atfgund.com) and download the app or use the web based version!
