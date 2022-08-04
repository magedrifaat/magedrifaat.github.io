---
layout: post
title:  "GSoC 2022 Week 7 Update"
date:   2022-08-04 16:00:00 +0200
categories: gsoc update
---
This update covers the work done in the 7th week of the coding period as well as what is expected to be done by the end of the 8th week.

## Last Week's Progress

### Finish implementing writeEncodedStirp
Last week the method `writeEncodedStrip` was implemented to support writing stripped images of all bit depths and configurations except for logical images as they require additional logic for packing/unpacking the bit data into separate array cells.

This week, this logic was added and the method now supports writing all types of stripped images with any configurations.

An example usage for logical images is as follows:
```matlab
img = Tiff ("test.tif", "w");
setTag (img, struct ("ImageWidth", 10, "ImageLength", 10));
data = logical (randi ([0,1], 10, 10));
writeEncodedStrip (img, 1, data);
img.close ();
```
The size of the `data` matrix should be the expected size of the strip. If the size is smaller than the strip size, the matrix is padded with zeroes to reach the correct size. If the size is bigger, the data is truncated and a warning is given to the user.

### Implementing writeEncodedTile
As discussed in earlier posts, Tiff images can be organized either as strips or as tiles. The `writeEncodedStrip` function allows writing images of the first type only, so implementing `writeEncodedTile` method is as important so that we can write tests that cover all types of Tiff images.

The signature for writeEncodedTile is:
```matlab
writeEncodedTile (t, tileNumber, imageData)
```
Similar to `writeEncodedStrip`, the `tileNumber` is a 1-based index of the tile in the image where tiles are numbered left-to-right top-to-bottom. And `imageData` is a matrix holding the data of the tile. The matrix must be of a datatype compatible with the bit-depth of the image and should have the same dimensions configured for the tiles of the image otherwise the data will be padded with zeroes or truncated.

### Implementing utility functions
Handling stripped or tiled images require additional information that can be calculated from the image parameters like the number of strips, number of tiles, and whether or not the image is tiled.

But LibTIFF provides functions that does the calculations as well as the error handling internally. So it is useful to implement methods that directly uses these functions.

Five methods were implemented which are:
 - `tf = isTiled (t)`: get whether the image is a tiled image.
 - `numStrips = numberOfStrips (t)`: get the number of strips in the image.
 - `numTiles = numberOfTiles (t)`: get the number of tiles in the image.
 - `stripNumber = computeStrip (t, row, plane)`: get the index of the strip containing the given row, with an optional plane argument for separate planes images.
 - `tileNumber = computeTile (t, [row,col], plane)`: get the index of the tile containing the given cell defined by row and col, with an optional plane argument for separate planes images.

These methods make writing code that handles Tiff images simpler and more readable.

### Writing tests to cover all corner cases
Now that we have methods that allow writing and reading Tiff images of all cases and configurations, we can write tests that check for the corner cases and the different combinations of image configurations as well as cases when the functions should fail and produce an error.

A lot of tests were written to cover as much of the code paths as possible. I used the guide at the end of the [Tests](https://wiki.octave.org/Tests) wiki to follow the best practices as much as possible.

All tests are at the end of the [Tiff.m](https://hg.octave.org/octave-libtiff/file/tip/scripts/io/Tiff.m) file.

## Next Week's Objectives
- Implement the `write` method
- Implement `readEncodedStrip`and `readEncodedTile`
- Add class structures for tag value enums
