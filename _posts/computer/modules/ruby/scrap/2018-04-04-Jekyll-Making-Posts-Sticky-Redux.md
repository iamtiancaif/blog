---
layout: post
title:  "Jekyll: Making Posts Sticky Redux"
categories: computer
tags:  computer jekyll
author: web
source: https://tamouse.github.io/swaac/jekyll/2017/09/04/jekyll-making-posts-sticky-redux/
---

* content
{:toc}



Sep 4, 2017 • 


[Last time](https://tamouse.github.io/swaac/2013/09/28/jekyll-making-posts-sticky/), I talked about using categories to make a post sticky. This eats up a category, though, and can make the post’s permalink look silly.

Use a Page Variable
-------------------

This time, we’re just going to use a page variable to do it.

In your posts that you want to be sticky, add a page variable to the front matter:
    ---
    layout: post
    sticky: true
    # ...
    ---

Then in your page where you want the sticky posts to appear first, use the following template:
 {% raw %} 
    {% for post in site.posts %}{% if post.sticky %}
    * [{{post.title}}]({{post.url}}) {{post.date|date_to_string}}
    {% endif %}{% endfor %}

    {% for post in site.posts %}{% unless post.sticky %}
    * [{{post.title}}]({{post.url}}) {{post.date|date_to_string}}
    {% endunless %}{% endfor %}
 {% endraw %} 
If you wanted the sticky posts to show up again in the lower section, just remove the `unless-endunless` condition in the second loop.

* * *

Use a Collection
----------------

The above solution is still a bit crude, and doesn’t really lend itself to pagination.

Perhaps a better solution to try for is set up a collection of sticky posts and deal with them completely separately from your regular posts.
In `_config.yml`
    collections:
      - important

Create the sticky posts in the `_important` folder like regular posts.

The file you want to show the sticky and regular post then can have the following construction:
	{% raw %} 
    {% for post in site.important %}
    * [{{post.title}}]({{post.url}}) {{post.date|date_to_string}}
    {% endfor %}
    
    {% for post in site.posts %}
    * [{{post.title}}]({{post.url}}) {{post.date|date_to_string}}
    {% endfor %}
	{% endraw %} 
<!--more-->
* * *

Works with Jekyl 3.5.2. YMMV.




Collections
===========
https://jekyllrb.com/docs/collections/

Not everything is a post or a page. Maybe you want to document the various methods in your open source project, members of a team, or talks at a conference. Collections allow you to define a new type of document that behave like Pages or Posts do normally, but also have their own unique properties and namespace.

Using Collections[Permalink](https://jekyllrb.com/docs/collections/#using-collections "Permalink")
--------------------------------------------------------------------------------------------------

To start using collections, follow these 3 steps:

*   [Step 1: Tell Jekyll to read in your collection](https://jekyllrb.com/docs/collections/#step1)
*   [Step 2: Add your content](https://jekyllrb.com/docs/collections/#step2)
*   [Step 3: Optionally render your collection’s documents into independent files](https://jekyllrb.com/docs/collections/#step3)

### Step 1: Tell Jekyll to read in your collection[Permalink](https://jekyllrb.com/docs/collections/#step1 "Permalink")

Add the following to your site’s `_config.yml` file, replacing `my_collection` with the name of your collection:

    collections:
    - my_collection
    

You can optionally specify metadata for your collection in the configuration:

    collections:
      my_collection:
        foo: bar
    

Default attributes can also be set for a collection:

    defaults:
      - scope:
          path: ""
          type: my_collection
        values:
          layout: page
    

##### Gather your collections3.7.0

You can optionally specify a directory to store all your collections in the same place with `collections_dir: my_collections`.

Then Jekyll will look in `my_collections/_books` for the `books` collection, and in `my_collections/_recipes` for the `recipes` collection.

##### Be sure to move posts into custom collections directory

If you specify a directory to store all your collections in the same place with `collections_dir: my_collections`, then you will need to move your `_posts` directory to `my_collections/_posts`. Note that, the name of your collections directory cannot start with an underscore (\`_\`).

### Step 2: Add your content[Permalink](https://jekyllrb.com/docs/collections/#step2 "Permalink")

Create a corresponding folder (e.g. `<source>/_my_collection`) and add documents. YAML front matter is processed if the front matter exists, and everything after the front matter is pushed into the document’s `content`attribute. If no YAML front  matter is provided, Jekyll will not generate the file in your collection.

##### Be sure to name your directories correctly

The folder must be named identically to the collection you defined in your `_config.yml` file, with the addition of the preceding `_` character.

### Step 3: Optionally render your collection’s documents into independent files[Permalink](https://jekyllrb.com/docs/collections/#step3 "Permalink")

If you’d like Jekyll to create a public-facing, rendered version of each document in your collection, set the `output` key to `true` in your collection metadata in your `_config.yml`:

    collections:
      my_collection:
        output: true
    

This will produce a file for each document in the collection. For example, if you have `_my_collection/some_subdir/some_doc.md`, it will be rendered using Liquid and the Markdown converter of your choice and written out to `<dest>/my_collection/some_subdir/some_doc.html`.

If you wish a specific page to be shown when accessing `/my_collection/`, simply add `permalink: /my_collection/index.html` to a page. To list items from the collection, on that page or any other, you can use:

    {% for item in site.my_collection %}
      <h2>{{ item.title }}</h2>
      <p>{{ item.description }}</p>
      <p><a href="{{ item.url }}">{{ item.title }}</a></p>
    {% endfor %}
    

##### Don't forget to add YAML for processing

Files in collections that do not have front matter are treated as [static files](https://jekyllrb.com/docs/static-files) and simply copied to their output location without processing.

Configuring permalinks for collections[Permalink](https://jekyllrb.com/docs/collections/#permalinks "Permalink")
----------------------------------------------------------------------------------------------------------------

If you wish to specify a custom pattern for the URLs where your Collection pages will reside, you may do so with the [`permalink` property](https://jekyllrb.com/docs/permalinks/):

    collections:
      my_collection:
        output: true
        permalink: /:collection/:name
    

### Examples[Permalink](https://jekyllrb.com/docs/collections/#examples "Permalink")

For a collection with the following source file structure,

    _my_collection/
    └── some_subdir
        └── some_doc.md
    

each of the following `permalink` configurations will produce the document structure shown below it.

*   Default Same as `permalink: /:collection/:path`.
    
        _site/
        ├── my_collection
        │   └── some_subdir
        │       └── some_doc.html
        ...
        
    
*   `permalink: pretty` Same as `permalink: /:collection/:path/`.
    
        _site/
        ├── my_collection
        │   └── some_subdir
        │       └── some_doc
        │           └── index.html
        ...
        
    
*   `permalink: /doc/:path`
    
        _site/
        ├── doc
        │   └── some_subdir
        │       └── some_doc.html
        ...
        
    
*   `permalink: /doc/:name`
    
        _site/
        ├── doc
        │   └── some_doc.html
        ...
        
    
*   `permalink: /:name`
    
        _site/
        ├── some_doc.html
        ...
        
    

### Template Variables[Permalink](https://jekyllrb.com/docs/collections/#template-variables "Permalink")

VARIABLE

DESCRIPTION

`:collection`

Label of the containing collection.

`:path`

Path to the document relative to the collection's directory.

`:name`

The document's base filename, with every sequence of spaces and non-alphanumeric characters replaced by a hyphen.

`:title`

The `:title` template variable will take the `slug` [front matter](https://jekyllrb.com/docs/frontmatter/)variable value if any is present in the document; if none is defined then `:title` will be equivalent to `:name`, aka the slug generated from the filename.

`:output_ext`

Extension of the output file. (Included by default and usually unnecessary.)

Liquid Attributes[Permalink](https://jekyllrb.com/docs/collections/#liquid-attributes "Permalink")
--------------------------------------------------------------------------------------------------

### Collections[Permalink](https://jekyllrb.com/docs/collections/#collections "Permalink")

Each collection is accessible as a field on the `site` variable. For example, if you want to access the `albums` collection found in `_albums`, you’d use `site.albums`.

Each collection is itself an array of documents (e.g., `site.albums` is an array of documents, much like `site.pages` and `site.posts`). See the table below for how to access attributes of those documents.

The collections are also available under `site.collections`, with the metadata you specified in your `_config.yml` (if present) and the following information:

VARIABLE

DESCRIPTION

`label`

The name of your collection, e.g. `my_collection`.

`docs`

An array of [documents](https://jekyllrb.com/docs/collections/#documents).

`files`

An array of static files in the collection.

`relative_directory`

The path to the collection's source directory, relative to the site source.

`directory`

The full path to the collections's source directory.

`output`

Whether the collection's documents will be output as individual files.

##### A Hard-Coded Collection

In addition to any collections you create yourself, the `posts` collection is hard-coded into Jekyll. It exists whether you have a `_posts` directory or not. This is something to note when iterating through `site.collections` as you may need to filter it out.

You may wish to use filters to find your collection:`{{ site.collections | where: "label", "myCollection" | first }}`

##### Collections and Time

Except for documents in hard-coded default collection `posts`, all documents in collections you create, are accessible via Liquid irrespective of their assigned date, if any, and therefore renderable.

However documents are attempted to be written to disk only if the concerned collection metadata has `output: true`. Additionally, future-dated documents are only written if`site.future` _is also true_.

More fine-grained control over documents being written to disk can be exercised by setting`published: false` (_`true` by default_) in the document's front matter.

### Documents[Permalink](https://jekyllrb.com/docs/collections/#documents "Permalink")

In addition to any YAML Front Matter provided in the document’s corresponding file, each document has the following attributes:

VARIABLE

DESCRIPTION

`content`

The (unrendered) content of the document. If no YAML Front Matter is provided, Jekyll will not generate the file in your collection. If YAML Front Matter is used, then this is all the contents of the file after the terminating `---` of the front matter.

`output`

The rendered output of the document, based on the `content`.

`path`

The full path to the document's source file.

`relative_path`

The path to the document's source file relative to the site source.

`url`

The URL of the rendered collection. The file is only written to the destination when the collection to which it belongs has `output: true` in the site's configuration.

`collection`

The name of the document's collection.

`date`

The date of the document's collection.

Accessing Collection Attributes[Permalink](https://jekyllrb.com/docs/collections/#accessing-collection-attributes "Permalink")
------------------------------------------------------------------------------------------------------------------------------

Attributes from the YAML front matter can be accessed as data anywhere in the site. Using the above example for configuring a collection as `site.albums`, you might have front matter in an individual file structured as follows (which must use a supported markup format, and cannot be saved with a `.yaml` extension):

    title: "Josquin: Missa De beata virgine and Missa Ave maris stella"
    artist: "The Tallis Scholars"
    director: "Peter Phillips"
    works:
      - title: "Missa De beata virgine"
        composer: "Josquin des Prez"
        tracks:
          - title: "Kyrie"
            duration: "4:25"
          - title: "Gloria"
            duration: "9:53"
          - title: "Credo"
            duration: "9:09"
          - title: "Sanctus & Benedictus"
            duration: "7:47"
          - title: "Agnus Dei I, II & III"
            duration: "6:49"
    

Every album in the collection could be listed on a single page with a template:

    
    {% for album in site.albums %}
      <h2>{{ album.title }}</h2>
      <p>Performed by {{ album.artist }}{% if album.director %}, directed by {{ album.director }}{% endif %}</p>
      {% for work in album.works %}
        <h3>{{ work.title }}</h3>
        <p>Composed by {{ work.composer }}</p>
        <ul>
        {% for track in work.tracks %}
          <li>{{ track.title }} ({{ track.duration }})</li>
        {% endfor %}
        </ul>
      {% endfor %}
    {% endfor %}





