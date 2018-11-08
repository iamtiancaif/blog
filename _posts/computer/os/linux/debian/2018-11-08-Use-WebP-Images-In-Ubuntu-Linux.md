---
layout: post
title:  "Use WebP Images In Ubuntu Linux"
categories: computer
tags:  computer linux debian
author: web
source: "https://itsfoss.com/webp-ubuntu-linux/"
---

* content
{:toc}


![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-0.jpg)

_Brief: This guide shows you how to view WebP images in Linux and how to convert WebP images to JPEG or PNG format. _

What is WebP?
-------------

It’s been over five years since Google introduced [WebP file format](https://developers.google.com/speed/webp/) for images. WebP provides lossy and lossless compression and WebP compressed files are around 25% smaller in size when compared to JPEG compression, Google claims.

Google aimed WebP to become the new standard for images on the web but I don’t see it happening. It’s over five years and it’s still not adopted as a standard except in Google’s ecosystem. But as we know, Google is pushy about its technologies. Few months back Google changed all the images on Google Plus to WebP.

If you download those images from Google Plus using Google Chrome, you’ll have WebP images, no matter if you had uploaded PNG or JPEG. And that’s not the problem. The actual problem is when you try to open that files in Ubuntu using the default GNOME Image Viewer and you see this error:

> Could not find XYZ.webp  
> Unrecognized image file format

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-1.png)

GNOME Image Viewer doesn’t support WebP images

In this tutorial, we shall see

*   how to add WebP support in Linux
*   list of programs that support WebP images
*   how to convert WebP images to PNG or JPEG
*   how to download WebP images directly as PNG images

How to view WebP images in Ubuntu and other Linux
-------------------------------------------------

[GNOME Image Viewer](https://wiki.gnome.org/Apps/EyeOfGnome), the default image viewer in many Linux distributions including Ubuntu, doesn’t support WebP images. There are no plugins available at present that could enable GNOME Image Viewer to add WebP support.

This means that we simply cannot use GNOME Image Viewer to open WebP files in Linux. A better alternative is [gThumb](https://wiki.gnome.org/Apps/gthumb) that supports WebP images by default.

To install gThumb in Ubuntu and other Ubuntu based Linux distributions, use the command below:

    sudo apt-get install gthumb

<!--more-->

Once installed, you can simply rightly click on the WebP image and select gThumb to open it. You should be able to see it now:

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-2.jpeg)

WebP image in gThumb

### Make gThumb the default application for WebP images in Ubuntu

For Ubuntu beginners, if you like to make gThumb the default application for opening WebP files, just follow the steps below:

Step 1: Right click on the WebP image and select Properties.

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-3.jpg)

Select Properties from Right Click menu

Step 2: Go to Open With tab, select gThumb and click on Set as default.

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-4.png)

Make gThumb the default application for WebP images in Ubuntu

### Make gThumb the default applications for all images

gThumb has a lot more to offer than Image Viewer. For example, you can do simple editing, add color filters to the images etc. Adding the filter is not as effective as XnRetro, the dedicated tool for [adding Instagram like effects on Linux](https://itsfoss.com/add-instagram-effects-xnretro-ubuntu-linux/), but the basic filters are available.

I liked gThumb a lot and decided to make it the default image viewer. If you also want to make gThumb the default application for all kind of images in Ubuntu, follow the steps below:

Step 1: Open System Settings

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-5.jpeg)

Step 2: Go to Details.

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-6.jpeg)

Step 3: Select gThumb as the default applications for images here.

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-7.png)

Alternative programs to open WebP files in Linux
------------------------------------------------

It is possible that you might not like gThumb. If that’s the case, you can choose one of the following applications to view WebP images in Linux:

*   [XnView](http://www.xnview.com/en/xnviewmp/#downloads) (Not open source)
*   GIMP with unofficial WebP plugin that can be installed via [this PPA](https://launchpad.net/~george-edison55/+archive/ubuntu/webp) that is available until Ubuntu 15.10. I’ll cover this part in another article.
*   [Gwenview](https://userbase.kde.org/Gwenview)

Convert WebP images to PNG and JPEG in Linux
--------------------------------------------

There are two ways to convert WebP images in Linux:

*   Command line
*   GUI

### Using command line to convert WebP images in Linux

You need to install WebP tools first. Open a terminal and use the following command:

    sudo apt-get install webp

#### Convert JPEG/PNG to WebP

We’ll use cwebp command (does it mean compress to WebP?) to convert JPEG or PNG files to WebP. The command format is like:

cwebp -q \[image\_quality\] \[JPEG/PNG\_filename\] -o \[WebP\_filename\]

For example, you can use the following command:

    cwebp -q 90 example.jpeg -o example.webp

### Convert WebP to JPEG/PNG

To convert WebP images to JPEG or PNG, we’ll use dwebp command. The command format is:

dwebp \[WebP\_filename\] -o \[PNG\_filename\]

An example of this command could be:

    dwebp example.webp -o example.png

### Using GUI tool to convert WebP to JPEG/PNG

  

For this purpose, we will use XnConvert which is a free but not open source application. You can download the installer files from their website:

Note that XnConvert is a powerful tool that you can use for batch resizing images. However, in this tutorial, we shall only see how to convert a single WebP image to PNG/JPEG.

Open XnConvert and select the input file:

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-8.jpeg)

In the Output tab, select the output format you want it to be converted. Once you have selected the output format, click on Convert.

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-9.jpeg)

That’s all you need to do to convert WebP images to PNG, JPEg or any other image format of your choice.

Download WebP images as PNG directly in Chrome web browser
----------------------------------------------------------

Probably you don’t like WebP image format at all and you don’t want to install a new software just to view WebP images in Linux. It will be a bigger pain if you have to convert the WebP file for future use.

An easier and less painful way to deal with is to install a Chrome extension Save Image as PNG. With this extension, you can simply right click on a WebP image and save it as PNG directly.

![](/image/scrap/2018-11-08-Use-WebP-Images-In-Ubuntu-Linux-10.jpg)

Saving WebP image as PNG in Google Chrome

[Get Save Image as PNG extension](https://chrome.google.com/webstore/detail/save-image-as-png/nkokmeaibnajheohncaamjggkanfbphi?utm_source=chrome-ntp-icon)

### What’s your pick?

I hope this detailed tutorial helped you to get WebP support on Linux and helped you to convert WebP images. How do you handle WebP images in Linux? Which tool do you use? From the above described methods, which one did you like the most?





