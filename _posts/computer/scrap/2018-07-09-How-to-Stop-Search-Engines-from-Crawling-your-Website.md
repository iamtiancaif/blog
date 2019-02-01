---
layout: post
title:  "How to Stop Search Engines from Crawling your Website"
categories: computer
tags:  computer scrap
author: "Jacob Nicholson"
source: "https://www.inmotionhosting.com/support/website/restricting-bots/how-to-stop-search-engines-from-crawling-your-website"
---

* content
{:toc}


In order for your website to be found by other people, **search engine** crawlers, also sometimes referred to as bots or spiders, will **crawl** your website looking for updated text and links to update their search indexes.

How to Control search engine crawlers with a robots.txt file
------------------------------------------------------------

Website owners, can instruct search engines on how they should crawl a website, by using a **robots.txt** file.

When a search engine crawls a website, it requests the **robots.txt** file first and then follows the rules within.

It's important to know **robots.txt** rules don't have to be followed by bots, and they are a guideline.

For instance to [set a Crawl-delay for Google](http://www.inmotionhosting.com/support/website/google-tools/setting-a-crawl-delay-in-google-webmaster-tools) this must be done in the [Google Webmaster tools](http://www.inmotionhosting.com/support/website/google-tools/introduction-to-google-webmaster-tools).

For bad bots that abuse your site you should look at [how to block bad users by User-agent in .htaccess](http://www.inmotionhosting.com/support/website/website-troubleshooting/block-unwanted-users-from-your-site-using-htaccess#block-by-user-agent).

Edit or create robots.txt file
------------------------------

The **robots.txt** file needs to be at the root of your site. If your domain was **example.com** it should be found:

**On your website**:

	http://example.com/robots.txt

**On your server**:

	/home/userna5/public_html/robots.txt

You can also [create a new file](http://www.inmotionhosting.com/support/website/how-to/create-new-file) and call it **robots.txt** as just a plain-text file if you don't already have one.

Search engine User-agents
-------------------------

The most common rule you'd use in a **robots.txt** file is based on the **User-agent** of the search engine crawler.

Search engine crawlers use a **User-agent** to identify themselves when crawling, here are some common examples:

**Top 3 US search engine User-agents**:

	Googlebot
	Yahoo! Slurp
	bingbot

**Common search engine User-agents blocked**:

	AhrefsBot
	Baiduspider
	Ezooms
	MJ12bot
	YandexBot
<!--more-->

Search engine crawler access via robots.txt file
------------------------------------------------

There are quite a few options when it comes to controling how your site is crawled with the **robots.txt** file.

The **User-agent:** rule specifies which User-agent the rule applies to, and ***** is a wildcard matching any User-agent.

**Disallow:** sets the files or folders that are **not allowed** to be crawled.

Here are some of the most common uses of the **robots.txt** file:

>[Set a crawl delay for all search engines](https://www.inmotionhosting.com/support/website/restricting-bots/how-to-stop-search-engines-from-crawling-your-website#crawl-delay)
>
>[Allow all search engines to crawl website](https://www.inmotionhosting.com/support/website/restricting-bots/how-to-stop-search-engines-from-crawling-your-website#allow-all)
>
>[Disallow all search engines from crawling website](https://www.inmotionhosting.com/support/website/restricting-bots/how-to-stop-search-engines-from-crawling-your-website#disallow-all)
>
>[Disallow one particular search engines from crawling website](https://www.inmotionhosting.com/support/website/restricting-bots/how-to-stop-search-engines-from-crawling-your-website#disallow-one)
>
>[Disallow all search engines from particular folders](https://www.inmotionhosting.com/support/website/restricting-bots/how-to-stop-search-engines-from-crawling-your-website#disallow-folders)
>
>[Disallow all search engines from particular files](https://www.inmotionhosting.com/support/website/restricting-bots/how-to-stop-search-engines-from-crawling-your-website#disallow-files)
>
>[Disallow all search engines but one](https://www.inmotionhosting.com/support/website/restricting-bots/how-to-stop-search-engines-from-crawling-your-website#disallow-all-but-one)

**Set a crawl delay for all search engines**:

If you had 1,000 pages on your website, a search engine could potentially index your entire site in a few minutes.

However this could cause [high system resource usage](http://www.inmotionhosting.com/support/website/server-usage/what-is-high-system-resource-usage) with all of those pages loaded in a short time period.

A **Crawl-delay:** of **30** seconds would allow crawlers to index your entire 1,000 page website in just **8.3 hours**

A **Crawl-delay:** of **500** seconds would allow crawlers to index your entire 1,000 page website in **5.8 days**

You can set the **Crawl-delay:** for all search engines at once with:

	**User-agent:** *****
	**Crawl-delay:** **30**

**Allow all search engines to crawl website**:

By default search engines should be able to crawl your website, but you can also specify they are **allowed** with:

	**User-agent:** *****
	**Disallow:** 

**Disallow all search engines from crawling website**:

You can **disallow** any search engine from crawling your website, with these rules:

	**User-agent:** *****
	**Disallow:** **/**

**Disallow one particular search engines from crawling website**:

You can **disallow** just one specific search engine from crawling your website, with these rules:

	**User-agent:** **Baiduspider**
	**Disallow:** **/**

**Disallow all search engines from particular folders**:

If we had a few directories like **/cgi-bin/**, **/private/**, and **/tmp/** we didn't want bots to crawl we could use this:

	**User-agent:** *****
	**Disallow:** **/cgi-bin/**
	**Disallow:** **/private/**
	**Disallow:** **/tmp/**

**Disallow all search engines from particular files**:

If we had a files like **contactus.htm**, **index.htm**, and **store.htm** we didn't want bots to crawl we could use this:

	**User-agent:** *****
	**Disallow:** **/contactus.htm**
	**Disallow:** **/index.htm**
	**Disallow:** **/store.htm**

**Disallow all search engines but one**:

If we only wanted to allow **Googlebot** access to our **/private/** directory, and disallow all other bots we could use:

	**User-agent:** *****
	**Disallow:** **/private/**

	**User-agent:** **Googlebot**
	**Disallow:**

When the **Googlebot** reads our **robots.txt** file, it will see it is not disallowed from crawling any directories.  






