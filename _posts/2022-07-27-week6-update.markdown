---
layout: post
title:  "GSoC 2022 Week 6 Update"
date:   2022-07-27 21:00:00 +0200
categories: gsoc update
---
This update covers the work done in the 6th week of the coding period as well as what is expected to be done by the end of the 7th week.

## Last Week's Progress

### Support for floating point images
The Tiff format specification supports images with floating-point values for pixels. This can be configured by setting the SampleFormat Tag value to 3.

In this week, support for reading floating-point images was added to the read function implemented in the previous weeks. The function now supports single-precision (32-bit) floating-point images as well as double-precision (64-bit) ones.

There exists, however, Tiff images with what is called half-precision (16-bit) floating-point pixel data. The data in these images is encoded according to the IEEE 754-2008 half-precision floating-point standard. This type of images is hard to support as there isn't a standard type in C++ that implements this data type.

So, since these images are not very common, support for this type of images will be considered after the main parts of the project are done.

### Implementing the functions for writing tests
As discussed in the last post, writing tests for the new classdef which generate their own images for testing needed more functions to be implemented.

The bare minimum of functions needed for writing the first tests were `setTag` and `writeEncodedStrip`. With only these two function and `read`, we can write tests that cover most of the cases for stripped Tiff images.

The implementation for `setTag` covers only the scalar tag types for now, since it is simple to handle and allows for a lot to be done. Handling the rest of the tag types will be implemented when the tests for all the cases available are done.

The implementation for `writeEncodedStrip` is nearly complete except for BiLevel images which are not yet implemented. This will be implemented in the next week so we can write tests for Bi-Level images.

### Writing tests for implemented functions
Now that we have functions that can be used to generate different types of Tiff images, I started writing tests for generating and reading images to check for different cases and different failure modes.

The tests that are written so far can be found at the end of [Tiff.m](https://hg.octave.org/octave-libtiff/file/tip/scripts/io/Tiff.m).

## Next Week's Objectives
- Add more tests to cover all corner cases
- Complete implementing functions to test the remaining cases
- Add documentation for the implemented functions
