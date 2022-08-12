---
layout: post
title:  "GSoC 2022 Week 8 Update"
date:   2022-08-10 16:00:00 +0200
categories: gsoc update
---
This update covers the work done in the 8th week of the coding period as well as what is expected to be done by the end of the 9th week.

## Last Week's Progress

### Implementing the write method
In the last weeks, priority was given to methods that helps in writing tests that cover all types of Tiff images. After that was done, the most important method to implement was the `write` method as it is the main interface for the user to write pixel data without the need to format the data into tiles or strips or channels.

The function was implemented to handle all the types and cases of the Tiff images. An example usage of the function is as follows:
```matlab
img = Tiff ("example.tif", "w");
setTag (img, struct(
    "ImageLength", 10,
    "ImageWidth", 10,
    "BitsPerSample", 8
));
data = uint8 (randi ([0,255], [10, 10]));
write(img, data);
img.close();
```

### Implementing readEncodedStrip and readEncodedTile
Next in the interface is the two methods `readEncodedStrip` and `readEncodedTile` which allows reading a specific strip or tile.

The methods take an 1-based index for a strip or tile and returns the pixel data that belong to said strip or tile.
```matlab
stripData = readEncodedStrip (t, stripNumber)
tileData = readEncodedTile (t, tileNumber)
```

### Adding class structures for tag value enums
Many of the tags supported in Tiff, take a set of predefined values that depend on the type of the tag. The Tiff interface defines a set of structures thatserve as enums for these tags.

All the structures were added to the Tiff class to allow an easier and more readable method of setting these tags.
```matlab
setTag(img, "Compression", Tiff.Compression.None);
```
All structures can be found at the beginning of the class in [Tiff.m](https://hg.octave.org/octave-libtiff/file/tip/scripts/io/Tiff.m).

## Next Week's Objectives
- Move internal functions to corefcn
- Finish implementing `setTag`
- Implement custom read modes (readRGBAImage/Strip/Tile)
- Implement directory manipulation methods
