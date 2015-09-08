---
layout: post
title: PHPUnit and Travis
date: 2015-09-09 03:13:44.000000000 -05:00
categories:
- DevOps
tags: []
status: publish
type: post
published: false
---

I've been getting more and more interested in unit testing lately, especially continuous integration to help spot bugs immediately while the errant code is fresh in your mind.

<!-- how do we install PHPUnit -->
[https://phpunit.de/](PHPUnit) is a unit testing framework for PHP.  Unit testing is a way of testing particular chunks of your code to make sure they're behaving in a way that you expect.

<!-- How do we create a unit test -->
The documentation mentions an 'autoload' file.  This is simple a file that declares how to find the definition of a class if it hasn't been found yet.  [http://php.net/manual/en/language.oop5.autoload.php](refer to the php.net documentation).  As per their recommendation, we'll create two directories for our project, one called 'tests' to hold our tests and one called 'src' to hold all of our source code.

```
abbadon:test jklem$ phpunit --bootstrap src/autoload.php test/
```

Next we'll want to add our phpunit.xml file.  When travis runs a test it defaults to 'phpunit' (if you're testing a php application).  Since there are no arguments, phpunit will look for the xml file.

```
<phpunit bootstrap="src/autoload.php">
  <testsuites>
    <testsuite name="muhtests">
      <directory>tests</directory>
    </testsuite>
  </testsuites>
</phpunit>
```

<!-- how do we get Travis to run said tests -->
To start using travis we'll want to create a .travis.yml to tell it how to test our project.  The following is an example ripped straight from their website:

```
language: php
php:
  - 5.4
  - 5.5
  - 5.6
  - hhvm
  - nightly
```

This is pretty self explanitory, all of the versions of php we want to test for are spelled out.  HHVM isn't a version of PHP, rather it's a JIT compiler for PHP that's apparently becoming pretty popular.  

