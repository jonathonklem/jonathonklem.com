---
layout: post
title: PHPUnit and Travis
date: 2015-09-09 03:13:44.000000000 -05:00
categories:
- DevOps
tags: []
status: publish
type: post
published: true
---

I've been getting more and more interested in unit testing lately, especially continuous integration to help spot bugs immediately while the errant code is fresh in your mind.  Unit testing is a way of testing particular chunks of your code to make sure they're behaving in a way that you expect.  [PHPUnit](https://phpunit.de/) is a unit testing framework for PHP.  Continuous integration is a method of automatically testing your code whenever a new commit is made to your repository (normally the master branch).  The idea behind continuous integration is that it's easier to fix bugs while they're fresh in our mind, so every commit is tested and if a test fails you know exactly which commit introduced the problem.

First we'll start off with PHPUnit.  Their quickstart guide (located [here](https://phpunit.de/getting-started.html)) does a good job of laying out the process and we'll create two directories for our project, one called 'tests' to hold our tests and one called 'src' to hold all of our source code like they recommend.  

In our src directory, we'll have our autoload.php file, which will be used to find class definitions ([refer to the php.net documentation for more info](http://php.net/manual/en/language.oop5.autoload.php)):

```
<?php
	spl_autoload_register(
		function($class) {
			if (file_exists("src/$class.php")) { 
				include "src/$class.php";
			}
		}
	);
```

Nothing too exciting here.  Next we'll move on to our Quadrilateral class (Quadrilateral.php):

```
<?php
class Quadrilateral {
	private $_width, $_height;
	public function __construct($width=1,$height=1) {
		$this->_width 	= $width;
		$this->_height	= $height;
	}
	public function getArea() {
		return $this->_width * $this->_height;
	}
	public function getHeight() { 
		return $this->_height;
	}
	public function getWidth() {
		return $this->_width;
	}
	public function setWidth($width) {
		$this->_width = $width+2;
	}
	public function setHeight($height) {
		$this->_height = $height;
	}
}

```

You'll notice there is an issue with our setWidth() method.  We're adding 2 to the entered width for no apparent reason.  This is to help demonstrate how PHPUnit can be used to find bugs.  Let's look at our QuadrilateralTest.php file in the test directory:

```
<?php
class QuadrilateralTest extends PHPUnit_Framework_TestCase
{
    function testArea()
    {
        $instance = new Quadrilateral(3, 2);

        $this->assertEquals($instance->getArea(), 6);
    }

		function testSetHeight() {
				// defaults to 1x1
        $instance = new Quadrilateral();

				// should now be 1x5
				$instance->setHeight(5);

        $this->assertEquals($instance->getArea(), 5);
		}

		function testSetWidth() {
				// defaults to 1x1
        $instance = new Quadrilateral();

				// should now be 7x1
				$instance->setWidth(7);

        $this->assertEquals($instance->getArea(), 7);
		}
}
?>
```

In this file we've defined 3 tests with the 3 functions.  In all three of these tests we're making assertions.  In the first we create an instance of Quadrilateral and pass it a width and a height.  We then make sure that the height is what we expect (3*2 should = 6).  In the second test we're creating an instance with the default width and height and then setting the height.  In the third test we're setting the width.  In both the second and the third test, we make an assertion about the area.

Next we'll want to add our phpunit.xml file.  When travis runs a test it defaults to 'phpunit' (if you're testing a php application).  Since there are no arguments, phpunit will look for the xml file.  This can also be demonstrated in your shell by running `phpunit` in the directory:

```
<phpunit bootstrap="src/autoload.php">
  <testsuites>
    <testsuite name="muhtests">
      <directory>tests</directory>
    </testsuite>
  </testsuites>
</phpunit>
```

The structure is pretty self explanatory.  We make sure to define our test suite and let PHPUnit know where the tests are located and we also tell it where our autoload file is located.  Next if we navigate to our directory and type `phpunit` it PHPUnit will run through our tests:

```
abbadon:test jklem$ phpunit
PHPUnit 4.8.6 by Sebastian Bergmann and contributors.

..F

Time: 99 ms, Memory: 11.75Mb

There was 1 failure:

1) QuadrilateralTest::testSetWidth
Failed asserting that 7 matches expected 9.

/Users/jklem/Documents/test/tests/QuadrilateralTest.php:28

FAILURES!
Tests: 3, Assertions: 3, Failures: 1.

```

Here we're given the name of the test that fails (testSetWidth) and a brief explanation (Failed asserting that 7 matches expected 9).  We would now know that something was up with our setWidth() function and be able to investigate.  For now, we'll leave our code broken so we can see how Travis handles it.


To start using travis we'll have to link our github repo to travis, their documentation outlines the steps (here)[http://docs.travis-ci.com/user/getting-started/].  Next we need to create a .travis.yml to tell it how to test our project. The following is an example ripped straight from their website:

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
Finally, we make sure that our .travis.yml and everything is added to our repo and then push it up to github.  If you log in to your travis dashboard, you'll see the build churning away and it'll be a yellow color.  Once it's done churning away it should turn to red and let you know that the build test failed.  

![travis build fail]({{site.url}}/assets/travis-screenshot.png)

If we go back to line 22 of our Quadrilateral.php file and remove the '+2', commit the change, and push it up, travis will test it again.

![travis build success]({{site.url}}/assets/travis-success.png)

And there you have it.  Continuous integration with PHPUnit and Travis.

