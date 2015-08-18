---
layout: post
title: RAD with CakePHP
date: 2015-03-16 02:14:20.000000000 -05:00
categories:
- Programming
tags: []
status: draft
type: post
published: false
meta:
  _edit_last: '1'
author:
  login: jonathon
  email: jonathonklem@gmail.com
  display_name: jonathon
  first_name: ''
  last_name: ''
excerpt: !ruby/object:Hpricot::Doc
  options: {}
---
<p>There are several PHP frameworks available, and I've worked with CodeIgniter, Zend, Symfony, and CakePHP. Of these, CakePHP has recently become my favorite. There has been a lot already written about CakePHP and there's an online book available <a href="http://book.cakephp.org/2.0/en/index.html" target="_blank">here</a>.  This article will not serve as a comparison, but a brief introduction to one of the more impressive features with cake--code generation with the cake console application.  It is the first part in a series on development with CakePHP.</p>
<p>CakePHP really excels with CRUD (Create Read Update Delete) applications.  With cake, you are spared from the monotony of writing a lot of the boiler plate code for creating forms and storing that information in the database.  It even goes a step further and can handle user authentication and more advanced functionality, but that's outside the scope of this article.</p>
<p>For the purposes of this article, we're going to develop a box inventory management system.  In our example problem, we want to keep track of boxes in our attic and the contents within them.  We'll start off with our database:</p>
<p>[insert pic of database]</p>
<p>&nbsp;</p>
