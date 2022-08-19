---
layout: post
title:  "GSoC 2022 Week 9 Update"
date:   2022-08-18 16:00:00 +0200
categories: gsoc update
---
This update covers the work done in the 9th week of the coding period as well as what is expected to be done by the end of the 10th week.

## Last Week's Progress

### Moved internal functions to corefcn
Previously, the internal functions file for the `Tiff` class `__tiff__.cc` was implemented in libinterp/dldfcn to be a dynamically loaded library.

However, this might cause errors on some systems as the LibTIFF library already exists in other places of Octave and it doesn't give any advantage either as the library is already loaded at start-up as part of other packages.

So the file was moved to libinterp/corefcn instead and necessary modifications were made to the relevant `Module.mk` file.

### Finished implementing setTag
So far, the implementation for setTag was a bare-minimum implementation that handles only singular-value tags to enable writing tests for other methods.

In this week, the implementation of setTag was completed and it now supports setting tags with a singular-value and array-value, as well as tags that require special case for its value such as `ColorMap`. Some bugs were also fixed in handling the return types for some special tags.
```matlab
img = Tiff ("example.tif", "w");
setTag (img, "ImageWidth", 10);
setTag (img, "YCbCrCoefficients", [1, 1.5, 2]);
setTag (img, "Software", "Octave");
```

### Implemented custom read modes (readRGBAImage/Strip/Tile)
LibTIFF provides a uniform interface for retrieving the pixel data of any Tiff image in RGBA format. This means that any image with any number of channels, any bit depth and any memory organization can be handled and converted internally by LibTIFF to 8-bit four channels (RGBA) data.

This is useful for purposes of previewing the image without dealing with the internal represntation of the image and the values for the different tags describing how the image pixel data should be processed because all of that is processed internally.

The disdvantage is that the user loses information and possibly precision from the original pixel data of the image, so cases that require accuracy in processing the pixel data should use the `read` interface instead.

The three methods from this interface `readRGBAImage`, `readRGBAStrip` and `readRGBATile` were implemented to retrieve the data of the image, a strip or a tile respectively.
```matlab
[RGB, alpha] = readRGBAImage (t)
[RGB, alpha] = readRGBAStrip (t, row)
[RGB, alpha] = readRGBATile (t, row, col)
```

Note that `readRGBAStrip` can only be used for stripped images and `readRGBATile` can only be used for tiled images.

### Implemented directory manipulation methods
A Tiff image file can contain multiple images in what is called directories. Each directory has one image as well as metadata about this image in the form of tags. This means that each image in the file can have its own format and their need not be any similarity between images in the same Tiff file as long as each of them is in its own directory.

To read and write as well as process different directories in the image file, it was required to implement functions that expose this functionality to the user.

The following methods were implemented:
 - `currentDirectory`: get the 1-based index of the current directory.
 - `lastDirectory`: get whether the current directory is the last one.
 - `nextDirectory`: switch to the next directory if exists.
 - `setDirectory`: switch to a directory with the given index.
 - `writeDirectory`: write the contents of the current directory to file.
 - `rewriteDirectory`: write the modified contents of the current directory to file.

## Next Week's Objectives
- Implement Sub-directory reading and writing
- Handle all remaining bugs / missing features in the Tiff class
- Work on modifying the imread/imwrite interface to use the new Tiff class
