---
layout: post
title: Arduino Weather Display
date: 2015-06-01 19:56:18.000000000 -05:00
categories:
- Circuits
- Programming
tags: []
status: publish
type: post
published: true
---

Keeping with the tradition of doing temperature related things with the Arduino, I decided that I was going to finally get my LCD display working and make it extra fancy by hooking it up to the internet.

Unlike with [the Twilio project](http://www.jonathonklem.com/blog/useless-arduino-hacking/), we will be fetching information and displaying it on our device.  To fetch the information, we use [World Weather Online's API](http://www.worldweatheronline.com/api/).  One limitation I quickly discovered was the Arduino does not have enough RAM to be able to download the entire XML response and parse it out.  To get around this, I introduced a PHP middleware script that parses out the information we want from the XML response and returns them in a format that is more manageable for our Arduino (the format is pipe delimited).  All of the code associated with this project can be found [in this github repo](https://github.com/jonathonklem/TemperatureDisplayer).

Here is what our circuit looks like:

![Schematic](https://jonathonklem.com/assets/images/schematic.jpg)

![Breadboard](https://jonathonklem.com/assets/images/breadboard.jpg)

The ethernet shield that I'm using uses the [SPI bus](http://www.arduino.cc/en/Reference/SPI) which consists of pins 10-13.  Even though they're physically available they are being used by the ethernet shield.  It should also be noted that I'm using a 4 line LCD display.  The 4 line LCD has the same pin layout as the 2 line LCD and uses the same functions from the LCD library.

Our process can be described as:

1. Connect to internet
2. Grab output of PHP middleware
  1. Use cURL to access the World Weather Online's API
  2. Parse the XML
  3. Return the information we want in a format more digestible for the Arduino
3. Parse the output and display on the LCD
4. Pause
5. Go to 2

Again, all of the code can be found  [in this github repo](https://github.com/jonathonklem/TemperatureDisplayer).  Rather than adding much explanation on the blog post, I tried to be as descriptive as possible in the comments.

Here is what the finished hardware looks like:

![Arduino Lights on](https://jonathonklem.com/assets/images/arduino-lights-on.jpg)

![Arduino Lights Off](https://jonathonklem.com/assets/images/arduino-lights-off.jpg)
