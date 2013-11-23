---
layout: post
title:  Recovering a Corrupted iPhoto Library
image:  recover-corrupted-photo-library.png
---

My hard drive RAID was damaged recently during a move and much of its contents were lost or corrupted. This included my iPhoto library with six years of personal photos. In order to recover my library I had to carefully combine several collections of partial backups with data recovered from the faulty hard drive.

{{ more }}

To accomplish this, I created a Python script to parse meta-data, find duplicates, detect corrupt images, and canonize the best-available copy of each photo. I've since managed to reconstruct a best-possible version of my iPhoto library using all available source material.

I'm putting the source code for this script on [GitHub](https://github.com/mikegreiling/photosort.py/). Read on for details on how I used it to recompile my library.

## Picking Up the Pieces

Before I could do anything I needed to compile all of the source material available to me which could be used to piece my library back together. First I utilized [Data Rescue 3](http://www.prosofteng.com/products/data_rescue.php) from Prosoft Engineering (which I highly recommend) to restore files from the failed hard drive, then I used [PhoneView](https://www.ecamm.com/mac/phoneview/) to recover synced iPhoto libraries from my iOS devices. There are other tools that will work for this as well.

Here's what I ended up with:

1. **Partly-recovered data from iPhoto** — Following the crash I cloned the faulty hard drive and used Data Rescue 3 to collect what I could from my corrupted RAID array. The result was a giant folder of thousands of rescued JPG files, some of which were from my iPhoto library, some of which were unrelated image files found on the hard drive. Many of these files were partly corrupted.

2. **iPhone photo library** — Months before the crash, I had synced my entire photo library (minus videos) to my iPhone. Unfortunately, these are all compressed copies of the originals. To save space, iTunes shrinks most images to a maximum width/height of 2048px. For example, 4:3 aspect ratio photos taken with the iPhone 4S are reduced from 3264x2448 to 2048x1536, a **60% loss in resolution**! Still, it kept the meta-data intact and at the very least I had a baseline of low-res photos to start with, complete with album and event lists.

3. **iPad photo library** — I had also synced a few photo albums to my iPad. Sadly, this did not include my full library, but the photos in this collection appear to be full resolution. So I'll want to go with these copies whenever possible.

4. **iCloud Photo Stream** — Sadly, although my iPhone deceptively shows 1000 photos in the photo stream "album", that is just what my phone has cached (at low resolution). What is actually stored on iCloud are photos from the last 30 days. This amounted to only 80 or so photos, all of which were still in my camera roll anyway; not so useful.

5. **Old iPhoto Library** — As luck would have it, I also had a four-year old copy of my iPhoto library on an old hard drive. At the very least I'd have all of these pre-2009 photos at full resolution, and it had videos too.

6. **Various Vacation Photos** — For a few vacations I went on, I had given a copy of all of my photos to family members and to my girlfriend. Once retrieved, I had these in full resolution as well.

## Putting It All Together

Now, I have several collections of photos, some overlapping, some corrupted, all at varying resolutions. I needed a programatic way to sort all of them, weed out corrupted files, detect duplicates at different resolutions, and canonize the best-available version of each photo in the library. Oh, and I want to preserve the albums they were sorted into as well.

To do this I had to get my hands dirty with Python. In my free time over the course of several weeks I put together a script to help me with this process. This being my first Python project since learning the language, the source code might not be the most elegant, but for what it's worth I've put it up on [GitHub](https://github.com/mikegreiling/photosort.py/).

### Installing Dependencies

This script has three main dependencies:
- [exiv2](http://exiv2.org) — to manipulate EXIF data
- [jpeginfo](https://github.com/tjko/jpeginfo) — to programmatically detect image corruption
- [pil](http://www.pythonware.com/products/pil/) — the Python Imaging Library (or any of its forks)

Assuming you use [Homebrew](http://brew.sh) on your Mac (and you should...) installing these is as simple as:

	brew install exiv2 jpeginfo
	pip install pillow

If you don't have pip installed, you'll need to install it with `sudo easy_install pip`. Linux users can probably achieve the same result with a simple `apt-get` or other flavor package manager.

### Running The Script

The way this script works is by sorting image sources into "buckets". You provide a source directory with `./photosort.py sort src dest -n bucket-name`. The script will walk through all files within the directory and identify images and non-images, and then further subcategorize them by their meta-data. These files are either copied `-c` or moved `-m` to a folder within 'dest'.

Example:

	[dest]/bucket-name
	    /exif
	        /2010
	            /03
	            /05
	                /2010.05.17 23.34.21-0 f6ce 2056x1536 OK fba6d3 [IMG_0129.JPG].bucket-name.jpg
	                /2010.05.19 18.14.02-4 78ea 2056x1536 OK e2c82f [IMG_0130.JPG].bucket-name.jpg
	                /2010.05.22 20.03.01-0 f6ce 2056x1536 OK 2bac41 [IMG_0131.JPG].bucket-name.jpg
	            /...
	        /2011
	            /...
	        /2012
	            /...
	    /noexif
	        /2011
	            /...
	    /noimage
	        /...

Now, the filenames are generated using the available photo EXIF meta-data and attributes such as image resolution. They get sorted into subfolders of the 'exif' folder corresponding to the year and month each photo was taken. Photos without EXIF data are sorted into 'noexif' and the file's last-modified attribute is used instead of the EXIF date-taken attribute.

The filename structure breaks down as follows:

	2010.05.22 20.03.01-0 f6ce 2056x1536 OK 2bac41 [IMG_0131.JPG].bucket-name.jpg
	|exif-datetime|hdr|extra|resolution|integrity|hash|[original-filename]|source(s)|.ext

### Combining Sources

When subsequent sources are added, the script first searches the existing buckets within 'dest' for meta-data matches using this filename structure. If a meta-match is found it will add it to the matched image's bucket. If no match is found it will sort it into the source image's own bucket.

When a match is found, the higher quality image will become the new canonical image, and the lower quality image will be either discarded or placed in `[bucket]/exif/low` if the `--preserve` flag is provided.

If a higher quality image is found, but shows potential signs of corruption it is sorted into `[bucket]/exif/alt` for manual review.

Finally, since EXIF data is only accurate to the second, occasionally you might encounter multiple photos taken within one second of each other. If a metadata match is made to multiple images, the match is sorted into `[bucket]/exif/unk` to be manually reviewed.

### Manual Review 

Once all photo sources have been sorted, you can manually review the files within 'alt' and 'unk'. You can canonize these images manually with the 'replace' command `./photosort.py replace src dest`. This will associate image at 'src' with 'dest', move 'src' to 'dest', and move 'dest' to the bucket's 'low' folder.

### Restoring Original Filenames

After you're satisfied with the organization, you can restore the original filenames with the 'restore' command: `./photosort.py restore src`. This will revert (in place) all of the filenames within the src directory to their original filename.

You can then add the images within the sorted buckets back into iPhoto or your library manager of choice.

### Fixing Photo Dates

When no EXIF data is present, iPhoto will use the filesystem date-modified attribute. However, occasionally an image will still have TIFF data embedded that corresponds with a later time and iPhoto will use that instead. If you'd rather have it go with the filesystem date-modified attribute you can overwrite the TIFF data with the 'fix' command: `./photosort.py fix src`. This will modify the metadata in each of the files within the 'src' directory to match the file's date-modified attribute.

I found that when I copied all of the photos from my iPhone with PhoneView, the date-modified attribute was more reliable than the TIFF data which seemed to correspond to the date the file was created rather than the date the photo was taken.

### Preserving Album Collections (Sort Order Matters)

Preserving album collections is pretty straight forward. You just need to be mindful about the order in which you sort the sources. PhoneView lets you export individual albums or entire libraries from your phone, so what I did was export the albums I wanted to preserve, then I exported the full library (knowing this would also include the photos in the albums).

As long as you sort the albums first, the duplicate photos in the full-library export will be recognized and sorted accordingly. If you sort the full library first and the albums second, the photos in the albums will all be seen as duplicates instead and their buckets will be empty since they are just a subset of what's already been processed.

One caveat is if you have photos which appear in multiple albums, they will only end up being sorted into the first album that gets processed, so this trick only works to preserve mutually-exclusive albums.

I ended up sorting everything with the following commands:

	./photosort.py sort ~/sources/album/jens-photos ~/sorted
	./photosort.py sort ~/sources/album/screensaver ~/sorted
	./photosort.py sort ~/sources/iphone-lib ~/sorted -n canon
	./photosort.py sort ~/sources/ipad-lib ~/sorted -n canon-ipad
	./photosort.py sort ~/sources/norway-vacation ~/sorted -n norway
	./photosort.py sort ~/sources/old-lib ~/sorted -n canon-old
	./photosort.py sort ~/sources/file-recovery ~/sorted -n recovered

Since the albums "jens-photos" and "screensaver" were subsets of "iphone-lib" I sorted them first.

## Voila!

Hopefully someone else out there will benefit from this tool. I know it saved me a hell of a lot of time that would otherwise be spent manually sorting through folders of photos to piece my library back together.

Again you can find it at [github.com/mikegreiling/photosort.py](https://github.com/mikegreiling/photosort.py/) and I'd appreciate any comments, pull-requests, or suggestions for improvement.
