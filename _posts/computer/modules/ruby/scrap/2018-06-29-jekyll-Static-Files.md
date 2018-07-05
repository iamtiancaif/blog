---
layout: post
title:  "jekyll Static Files"
categories: computer
tags:  computer jekyll copy
author: web
source: "https://jekyllrb.com/docs/static-files/"
---

* content
{:toc}


In addition to renderable and convertible content, we also have static files.

A static file is a file that does not contain any YAML front matter. These include images, PDFs, and other un-rendered content.

They’re accessible in Liquid via `site.static_files` and contain the following metadata:

|VARIABLE|DESCRIPTION|
|--- |--- |
|**file.path**|The relative path to the file, e.g. /assets/img/image.jpg|
|**file.modified_time**|The `Time` the file was last modified, e.g. 2016-04-01 16:35:26 +0200|
|**file.name**|The string name of the file e.g. image.jpg for image.jpg|
|**file.basename**|The string basename of the file e.g. image for image.jpg|
|**file.extname**|The extension name for the file, e.g. .jpg for image.jpg|

Note that in the above table, `file` can be anything. It’s simply an arbitrarily set variable used in your own logic (such as in a for loop). It isn’t a global site or page variable.

Add front matter to static files
---------------------------------------------------------------------------------------------------------------------------------

Although you can’t directly add front matter values to static files, you can set front matter values through the [defaults property](https://jekyllrb.com/docs/configuration/#front-matter-defaults) in your configuration file. When Jekyll builds the site, it will use the front matter values you set.

Here’s an example:

In your `_config.yml` file, add the following values to the `defaults` property:

    defaults:
      - scope:
          path: "assets/img"
        values:
          image: true
    

This assumes that your Jekyll site has a folder path of `assets/img` where you have images (static files) stored. When Jekyll builds the site, it will treat each image as if it had the front matter value of `image: true`.

Suppose you want to list all your image assets as contained in `assets/img`. You could use this for loop to look in the `static_files` object and get all static files that have this front matter property:

    {% assign image_files = site.static_files | where: "image", true %}
    {% for myimage in image_files %}
      {{ myimage.path }}
    {% endfor %}
    

When you build your site, the output will list the path to each file that meets this front matter condition.

















