---
title: Jekyll recipes for your blog's meta-tags
layout: post
categories: jekyll
published: false
description: "Have you ever wondered why some web pages have a great preview when linked on Twitter, Facebook, or Google+ while others look so lacking? It's all about the metadata."
---

Adding [Open Graph protocol](http://ogp.me) and [Twitter Card](https://dev.twitter.com/docs/cards) meta-tags to your blog is really easy. If you use [Jekyll](http://jekyllrb.com/) and perhaps [Github Pages](http://pages.github.com) for your blog then it is even easier with these recipes.

Remember that these recipes aren't complete. You'll need more than what I show here to correctly enhance your blog with Open Graph protocol or Twitter Card metadata. Read up on [my other post]({{site.url}}/2013/open-graph-twitter-cards-and-your-blog/) for the rest of what you need.

## Basics

<pre data-language="html">
&lt;meta property="og:title" content="&#123;&#123; page.title &#125;&#125;" /&gt;
&lt;meta property="og:url" content="&#123;&#123; site.url &#125;&#125;&#123;&#123; page.url &#125;&#125;" /&gt;
</pre>

## Categories

<pre data-language="html">
&#123;% for category in page.categories %&#125;

    &lt;meta property="article:tag" content="&#123;&#123; category &#125;&#125;" /&gt;

&#123;% endfor %&#125;
</pre>

## Timestamp

<pre data-language="html">
&lt;meta property="article:published_time" content="&#123;&#123; page.date | date_to_xmlschema &#125;&#125;" /&gt;
</pre>

## Description

I define a special `description` variable in the YAML front-matter of posts that I want to enter a specific description for. Other posts I'm perfectly OK with simply using the beginning portion of the body as the description. I handle that scenario by using the `description` variable if it exists and otherwise use the first 200 characters of the body, truncated at the word boundary.

<pre data-language="html">
&#123;% if page.description %&#125;

    &lt;meta name="description" content="&#123;&#123; page.description | truncatewords: 200 | strip_newlines | escape_once &#125;&#125;" /&gt;

&#123;% else %&#125;

    &lt;meta name="description" content="&#123;&#123; content | strip_html | truncatewords: 200 | strip_newlines | escape_once &#125;&#125;" /&gt;

&#123;% endif %&#125;
</pre>

## Images

I use an `image` variable in the front-matter to specify the image I want used to represent my post. Not all posts have an image so it is optional. This image should be at least 50px by 50px but [Facebook prefers](https://developers.facebook.com/docs/technical-guides/opengraph/built-in-objects/#article) 200px by 200px or larger.

<pre data-language="html">
&#123;% if page.image %&#125;

    &lt;meta property="og:image" content="&#123;&#123; site.url &#125;&#125;/assets/&#123;&#123; page.image &#125;&#125;" /&gt;

&#123;% endif %&#125;
</pre>

Since Twitter Cards want a 120px by 120px image, I like to provide a special version of the image instead of letting Twitter crop my original. I do this by specifying another front-matter variable.

<pre data-language="html">
&#123;% if page.twitter_card_image %&#125;

    &lt;meta name="twitter:image" content="&#123;&#123; site.url &#125;&#125;/assets/&#123;&#123; page.twitter_card_image &#125;&#125;" /&gt; 

&#123;% endif %&#125;
</pre>