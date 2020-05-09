---
title: "Working with large files in Python"
description: "Recently I started on a pet project to explore a large data-set (32 million users) which was leaked to explore performance in Python. This…"
date: "2015-12-17T16:57:05.331Z"
categories: []
published: true
canonical_link: https://medium.com/@Rapchik/working-with-large-files-in-python-d9b3850de425
redirect_from:
  - /working-with-large-files-in-python-d9b3850de425
---

Recently I started on a pet project to explore a large data-set (32 million users) which was leaked to explore performance in Python. This was a weekend study and hence is not very thorough but a mere overview of how Python allow’s multiple approaches to a problem but not all the approaches provide a fitting solution.

The way the files are split, we have 2 key files that we will work with. One file which has user info (username, geo-coordinates, etc) and is a 13 GB file. The other file stores email addresses and is around 1.6 GB. Not BigData scale but fair enough for testing performance on an core 2 duo Mac Book Pro (2009).

My goal with the project was to see how fast we can get a City from the geo-coordinates and find the corresponding email address from the other file.

First I started looking for different solutions to get location from geo-coordinates, initially I used this open source [CountryChecker](https://github.com/che0/countries) module built using the wonderful [OSGEO GDAL/OGR](http://www.osgeo.org/gdal_ogr) python bindings. The library uses the GDAL framework to test coordinates against a provided shape file which contains world data with boundaries for cities/countries. I used a world data set provided by [thematicmapping](http://thematicmapping.org/downloads/world_borders.php) for testing against coordinates.

After getting the country, we go through the entire file containing list of email addresses to find the email against the id of the user’s location.

The results for this approach as you can guess, were pathetic. Its a classic O(n\*n) with a huge bottle neck being file I/O as we would be going through the entire email file for each user + the shape file too. It took the app 30 minutes to go through 30,000 users of the 32 million. It would have become slower as we went down the list as the emails would be lower down in the second file so we obviously need to optimize this.

First and foremost, reading through the entire email file for each of the 32 million users in the first file is not a very bright idea. As the emails are listed sequentially preceded by the user ID, we do have an idea of the line number where we can find the email address for the specific user.

The way file systems work is that we can open a file, and seek to any specific byte offset we want in the file, but we cannot jump to a line as we do not know where each line ends. How about we go through the file once and save the byte offset for each line ending? We would be going through the entire file but only once, after which we will have the magical power of jumping to the exact line number we want and closing the final!

Python has a built-in function that allows just this called [LineCache](https://docs.python.org/2/library/linecache.html) but the implementation uses readlines() to read the file to cache. That means that python would load the entire file into memory at once before going through the file data to figure out where the offsets are for each line ending. This approach would be a bottleneck for memory and hung my poor old mac for 15 minutes on the email file before running out of memory.

To counter this I created [my version of LineCache](https://github.com/umarniz/PythonDataTools) that reads files line by line and gets the offset at the end of each line and stores it in a .cache file.

Now we can jump to the middle of any file, but we still have a problem. The files we have are SQL Dumps and do not perfectly correlate to each other. So if file 1 has a user on line 20,000 the email file could have the email for that user at line 19,600 as some users are missing from each file. We know that the user we need to find in file 2 is going to be somewhere around the line on which the user was in file 1 as there can only be so many missing entries. So we can translate the **somewhere around** to code if we implement some form of fuzzy matching.

What I did was to jump to the line in the email file and get the user ID on that line and then check against the user id we actually need. If the user ID we need is greater than the user ID we have, we need to go a few lines ahead otherwise go a few lines behind till we get to the user.

With fuzzy matching + line caching, I ran a quick test to see how the performance was at this stage. It took ~45 seconds to make a line cache on first run. After 30 minutes we had checked over 2 million users. Not bad at all, but we need moarrr!

Second major point of failure for us would be the bottle neck involved checking the geo-coordinates against geo-coordinates of all the countries in the world to find what country it is and then find what city it is. The easiest solution for that was to invert the test. As we have 1 city/country we are looking for and millions of points to test, I first calculated the bounding box of the city were looking for and added a fast inexpensive test to check if the point is even inside the bounding box that contains the city before passing the data to OSGEO.

After adding this broadphase test, it took my MBP 2 minutes to test the complete dataset of 32 million users against their location and find their email addresses.
