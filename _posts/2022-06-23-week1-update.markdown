---
layout: post
title:  "GSoC 2022 Week 1 Update"
date:   2022-06-23 16:05:00 +0200
categories: gsoc update
---
This is my first weekly update for GSoC 2022 Project with GNU Octave "Improve TIFF image support".

The main objective right now is to add a classdef called "Tiff" to Octave similar to the one with the same name in matlab and with similar or identical functionality.

## This Week's Objectives

### Set up a hosting repostiroy forked from Octave

With the help of the Octave community, I got a hosting repository on fosshost which can be found [here](https://hg.octave.org/octave-libtiff/). I set up a bookmark in the repository with the name "gsoc-libtiff" to contain all the commits relevant to this project.

### Add the Tiff classdef to the Octave build system

Adding the Tiff classdef to the Octave build system required adding two main modifications to the file tree:  
- Adding a Tiff.m file under scripts/io to define the classdef and adding an entry in the module.mk file for the new file.
- Adding a \_\_tiff\_\_.cc under libintrep/dldfcn to serve as an oct file for all the functions that will be defined from the C++ side, and adding an entry in "module-files" file for this new file.

And now the classdef is successfuly added to the build system of Octave, which means that it can be used by anyone by cloning the repo and building Octave like usual.

It is worth noting that there are some more modifications still required to fully integrate this classdef  into the code base and support all the systems that Octave does support, namely to the configure.ac file. These modifications will be done in the next week.

### Implement getTag and setTag functions

Now that the classdef is added to Octave, it was time to implement an interface on top of libtiff to start using it. The bare minimum functions needed were a constructor that opens the file and a function to close a file.
```matlab
a = Tiff("example.tif")
a.close()
```
I faced an issue regarding how to keep the file pointer around once it is constructed, you can read about the issue in detail and the possible solutions in the forum thread [here](https://octave.discourse.group/t/tying-a-c-pointer-to-an-octave-object/2865).

The first real feature of the class that I started working on this week was getTag. At first, I was able to implement a simple version of the function that always assumes the data of the tag is a 32-bit unsigned integer. This worked for many of the simple tags like ImageWidth, ImageLength, and so on.
```matlab
getTag(a, "ImageWidth")
getTag(a, 256)
getTag(a, Tiff.TagID.ImageWidth)
```

However, the format specification for Tiff, as well as the libtiff library, include support for tags with many different data types and support for tags with multiple values as well. Unfortunately, libtiff library doesn't provide a generalized interface to retrieve the data for these tags. Instead, each tag requires some special arguments passed to TIFFGetField in order to retrieve it correctly.  You can see that in the [documentation of libtiff](https://libtiff.gitlab.io/libtiff/man/TIFFGetField.3tiff.html):
```cpp
int TIFFGetField(TIFF *tif, ttag_t tag, ...)
```
```
The type and number of values returned is dependent on the tag being requested.
```

With a long list of tags and their corresponding arguments in the rest of the page.
This means that a lot of work needs to be done to handle all of these special cases and develop my own generalized interface that getTag can use.

## Next Week's Objectives
- Finish implementing getTag for all the supported tags.
- Implement setTag.
- Modify configure.ac to correctly handle the new classdef.