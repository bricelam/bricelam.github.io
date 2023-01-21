---
layout: post
title: Image Metadata using WinRT on .NET
tags: miscellaneous
---

Did you know you can tap into a whole world of Windows Runtime (WinRT) APIs using .NET 6? No really, APIs for [media transcoding](https://learn.microsoft.com/uwp/api/windows.media.transcoding) and [composition](https://learn.microsoft.com/windows/uwp/audio-video-camera/media-compositions-and-editing), [face detection](https://learn.microsoft.com/windows/uwp/audio-video-camera/detect-and-track-faces-in-an-image), [OCR](https://learn.microsoft.com/uwp/api/windows.media.ocr), [speech recognition](https://learn.microsoft.com/windows/apps/design/input/speech-recognition), and [text-to-speech](https://learn.microsoft.com/uwp/api/windows.media.speechsynthesis) are just sitting there waiting to be used by your apps. In this post, we'll use some of these APIs to answer the Who, When, and Where of an image file.

Using WinRT
===========
Before you can access the WinRT namespaces, you need to reference them. Do this by updating your project's target framework (TFM) to be Windows-specific. Inside your project file, and add *-windows10.0.17763* to it.

```xml
<TargetFramework>net6.0-windows10.0.17763</TargetFramework>
```

In the same way that using the *-android* or *-ios* lets you access those platforms' native APIs, this lets you access the Windows ones.

When
====
The first question we want to answer using the WinRT APIs is: When was a photo taken?

There are lots of ways to access this information, but in my opinion, the best is also one of the simplest.

```cs
var file = await StorageFile.GetFileFromPathAsync(path);
var properties = await file.GetBasicPropertiesAsync();
var dateTaken = properties.ItemDate;
Console.WriteLine($"Date Taken: {dateTaken}");
```

The [BasicProperties.ItemDate](https://learn.microsoft.com/uwp/api/windows.storage.fileproperties.basicproperties.itemdate) API will give you "the most relevant date for the item". For photos, that means the date it was taken. I prefer this API because you don't have to worry about how the metadata is stored. It will look everywhere it can to find the information.

Where
=====
The next question we want to answer is: Where was the photo taken?

For this, we'll use [GeotagHelper](https://learn.microsoft.com/uwp/api/windows.storage.fileproperties.geotaghelper). Like ItemDate above, this will look everywhere it can for the location.

```cs
var geotag = await GeotagHelper.GetGeotagAsync(file);
var latitude = geotag?.Position.Latitude;
var longitude = geotag?.Position.Longitude;
Console.WriteLine($"Location: {latitude},{longitude}");
```

Who
===
The previous APIs were nice and simple. They're also general-purpose--they work just as well on video files. But to answer the question *Who is this a photo of?* we'll need to dive deeper into the metadata APIs.

Photo apps like [Picasa](https://en.wikipedia.org/wiki/Picasa), [Windows Live Photo Gallery](https://en.wikipedia.org/wiki/Windows_Photo_Gallery) (may they both rest in peace), and [digiKam](https://www.digikam.org/) let you tag people in your photos the same way you would when posting on social media. This information gets embedded into the image's metadata.

If all you need are the names, you can use [ImageProperties.PeopleNames](https://learn.microsoft.com/uwp/api/windows.storage.fileproperties.imageproperties.peoplenames).

```cs
var imageProperties = await file.Properties.GetImagePropertiesAsync();
var peopleNames = imageProperties.PeopleNames;
```

Unfortunately, this doesn't tell you which name belongs to which face. To get the corresponding rectangle on the image, we need to do some intense querying of the metadata.

There are two main ways this information is stored. The first metadata standard for it was Microsoft Photo (you guessed it, made popular by our beloved Windows Live Photo Gallery). Later, the big tech companies came together as the Metadata Working Group to create [another standard](https://xkcd.com/927/) that "fixed" all their complaints about the first one. So, now we always have two places to look instead of one.

Here's a method to query the metadata using [BitmapDecoder.BitmapProperties](https://learn.microsoft.com/uwp/api/windows.graphics.imaging.bitmapdecoder.bitmapproperties). Fun fact, this API is backed by the [Windows Imaging Component](https://learn.microsoft.com/windows/win32/wic/-wic-about-windows-imaging-codec) (or WIC) so every image format imaginable is supported. Well, so long as long as you have a codec for it installed anyway.

```cs
static async Task<IReadOnlyList<(string Name, Rect Area)>> GetPeopleAsync(IStorageFile file)
{
    var people = new List<(string Name, Rect Area)>();
    using var stream = await file.OpenReadAsync();

    BitmapDecoder decoder;
    try
    {
        decoder = await BitmapDecoder.CreateAsync(stream);
    }
    catch
    {
        return Array.Empty<(string, Rect)>();
    }

    // Microsoft Photo
    var regionList = (BitmapPropertiesView?)(await decoder.BitmapProperties.GetPropertiesAsync(new[] { "/xmp/MP:RegionInfo/MPRI:Regions" })).Values.SingleOrDefault()?.Value;
    if (regionList is not null)
    {
        foreach (var region in (await regionList.GetPropertiesAsync(Enumerable.Empty<string>())).Values.Select(p => (BitmapPropertiesView)p.Value))
        {
            var name = (string)(await region.GetPropertiesAsync(new[] { "/MPReg:PersonDisplayName" })).Values.Single().Value;

            var rectangle = (string?)(await region.GetPropertiesAsync(new[] { "/MPReg:Rectangle" })).Values.SingleOrDefault()?.Value;
            if (rectangle is null)
                continue;

            var rectangleParts = rectangle.Split(',', StringSplitOptions.TrimEntries);
            var x = double.Parse(rectangleParts[0]);
            var y = double.Parse(rectangleParts[1]);
            var w = double.Parse(rectangleParts[2]);
            var h = double.Parse(rectangleParts[3]);

            people.Add((name, new Rect(x, y, w, h)));
        }
    }

    // Metadata Working Group
    const string mwgRs = @"http\:\/\/www.metadataworkinggroup.com\/schemas\/regions\/";
    regionList = (BitmapPropertiesView?)(await decoder.BitmapProperties.GetPropertiesAsync(new[] { $"/xmp/{mwgRs}:Regions/{mwgRs}:RegionList" })).Values.SingleOrDefault()?.Value;
    if (regionList is not null)
    {
        foreach (var region in (await regionList.GetPropertiesAsync(Enumerable.Empty<string>())).Values.Select(p => (BitmapPropertiesView)p.Value))
        {
            var name = (string?)(await region.GetPropertiesAsync(new[] { $"/{mwgRs}:Name" })).Values.SingleOrDefault()?.Value;
            if (name is null)
                continue;

            const string stArea = @"http\:\/\/ns.adobe.com\/xmp\/sType\/Area";
            var cx = double.Parse((string)(await region.GetPropertiesAsync(new[] { $"/{mwgRs}:Area/{stArea}#:x" })).Values.Single().Value);
            var cy = double.Parse((string)(await region.GetPropertiesAsync(new[] { $"/{mwgRs}:Area/{stArea}#:y" })).Values.Single().Value);
            var w = double.Parse((string)(await region.GetPropertiesAsync(new[] { $"/{mwgRs}:Area/{stArea}#:w" })).Values.Single().Value);
            var h = double.Parse((string)(await region.GetPropertiesAsync(new[] { $"/{mwgRs}:Area/{stArea}#:h" })).Values.Single().Value);

            // Note, x and y represent the center of the rectangle in this format.
            // We normalize it to left and top instead so it matches Rect.
            people.Add((name, new Rect(cx - (w / 2.0), cy - (h / 2.0), w, h)));
        }
    }

    return people;
}
```

As you can see, these APIs aren't the friendliest to work with, but with a lot of casting and superfluous constructs, we're able to get the information we need.

Now we can use this method to get the people in our photo.

```cs
var people = await GetPeopleAsync(file);
Console.WriteLine("People:");
foreach (var person in people)
    Console.WriteLine($"    {person.Name} [{person.Area}]");
```

One interesting thing to note is that the x, y, width, and height values are always between 0 and 1. That's because they're percentages of the photo's actual with and height. This allows the metadata values to remain the same even if the image is resized. If you want to draw the rectangles, you'd have to multiply the values by the image's width and height.

```cs
var rectToDraw = new Rect(
    faceArea.X * image.Width,
    faceArea.Y * image.Height,
    faceArea.Width * image.Width,
    faceArea.Height * image.Height);
```

What's next?
============
Hopefully this has given you a taste of all the untapped power inside Windows just waiting to be used by your .NET apps. Let me know if there are other areas you'd like to see covered, and let me know about all the cool APIs that I should be using in my apps. Happy coding!
