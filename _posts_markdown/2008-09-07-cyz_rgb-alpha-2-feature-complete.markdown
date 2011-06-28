--- 
wordpress_id: 5
layout: post
title: CYZ_RGB alpha 2 - feature complete
date: 2008-09-07 19:03:00 +00:00
wordpress_url: http://caprazzi.net/?p=5
---
<div style="text-align: justify;"><span style="font-family: courier new;">The second release of CYZ_RGB, an alternative firmware for </span>[BlinkM](http://thingm.com/products/blinkm)<span style="font-family: courier new;">, is now ready for download.</span><br /><br /><span style="font-family: courier new;"><span style="font-weight: bold;">The latest release implements all the features of the original firmware.</span><br /><br />The program has been slightly tested at a functional level (sitting there and watching the lights go on end off), but has no unit tests.</span><br /><br /><span style="font-family: courier new;">I usually write loads of tests, possibly before the actual code; this time I was wandering in a completely new area and I knew most of the code was going to be highly disposable. Now that all hurdles have been surpassed, the refactoring stage will go hand-to-hand with writing a thorough test suite. Wish me good luck.</span><br /><br /><span style="font-family: courier new;">Go read the </span>[details](http://code.google.com/p/codalyze/wiki/CyzRgb)<span style="font-family: courier new;">.</span><br /><br /><span style="font-family: courier new;font-size:100%;">Goals for next release:<br /></span>- <span style="font-family: courier new;font-size:100%;">Write a comprehensive test suite</span>
- <span style="font-family: courier new;font-size:100%;">Decouple cyz_cmd from cyz_rgb</span>
- <span style="font-family: courier new;font-size:100%;">Decouple usiTwiSlave from cyz_cmd</span>
- <span style="font-family: courier new;font-size:100%;">Reduce binary footprint (now 4072b of 4096 available)</span>
- <span style="font-family: courier new;font-size:100%;">Add ability to write 2 scripts </span>
<span style="font-family: courier new;">-Matteo</span><br /><span style="font-family: courier new;"></span></div>
