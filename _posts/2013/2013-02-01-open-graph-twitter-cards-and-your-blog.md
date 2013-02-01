---
title: Open Graph, Twitter Cards, and your blog
layout: post
categories: web
description: "Have you ever wondered why some web pages have a great preview when linked on Twitter, Facebook, or Google+ while others look so lacking? It's all about the metadata."
---

Have you ever wondered why some web pages have a great preview when linked on Twitter, Facebook, or Google+ while others look so lacking?

<style>
	.carousel {
		width: 600px;
		margin: 0 auto;
	}
	.carousel img {
		display: inline-block;
		margin-bottom: 20px;
	}
</style>

<div class="carousel">
	<img alt="Screenshot of Facebook link previews" src="{{site.url}}/assets/forposts/open-graph-and-twitter-cards/fb-vs.png" style="width: 450px" />

	<img alt="Screenshot of Google+ link previews" src="{{site.url}}/assets/forposts/open-graph-and-twitter-cards/g-vs.png" style="width: 450px" />

	<img alt="Screenshot of Twitter link previews" src="{{site.url}}/assets/forposts/open-graph-and-twitter-cards/tw-vs.png" style="width: 450px" />
</div>

<div style="clear: both;"></div>

What you are seeing are formats such as the [Open Graph protocol](http://ogp.me/) and [Twitter Cards](https://dev.twitter.com/docs/cards) at work.

Open Graph is an open source protocol originally created by Facebook that allows your web page to describe itself using meta-tags. When people link to your web page on Facebook or Google+, these meta-tags will be parsed and used to display a snippet of content from your web page. Without these tags, the parser has to guess what to display.

Twitter Cards let you attach a summary to any tweet that contains a link to your web page. Twitter Cards use their own set of meta-tags but will fallback to using the Open Graph meta-tags in some cases.	

## Adding Open Graph

Adding Open Graph tags to your pages is an easy way to enhance their display on both Google+ and Facebook.

### ...to your home page

Open Graph requires you to import a namespace of sorts. You do this by adding a `prefix` attribute to your `<html>` tag.

<pre data-language="html">
&lt;html prefix="og: http://ogp.me/ns#"&gt;
</pre>

Every page is required to have a set of basic properties. Your blog's home page should have the following meta-tags inside the `<head>` tag.

<pre data-language="html">
&lt;meta property="og:type" content="website" /&gt;
&lt;meta property="og:title" content="Zduck - Joshua Poehls" /&gt;
&lt;meta property="og:image" content="http://zduck.com/logo.png" /&gt;
&lt;meta property="og:url" content="http://zduck.com" /&gt;
</pre>

The `og:url` property is the canonical URL of the current page. If there are multiple URLs a user could use to visit your page (e.g. www.zduck.com and zduck.com), this should contain the preferred URL.

### ...to your posts

The properties on your individual post pages will be slightly different. Open Graph calls a blog post an _article_ and we'll need to import the namespace for the [article object type](http://ogp.me/#type_article).

<pre data-language="html">
&lt;html prefix="og: http://ogp.me/ns# article: http://ogp.me/ns/article#"&gt;
</pre>

<pre data-language="html">
&lt;meta property="og:type" content="article" /&gt;
&lt;meta property="og:site_name" content="Zduck - Joshua Poehls" /&gt;
&lt;meta property="og:title" content="Title of my blog post" /&gt;
&lt;meta property="og:image" content="http://zduck.com/mypost/image.png" /&gt;
&lt;meta property="og:url" content="http://zduck.com/mypost" /&gt;
&lt;meta property="og:description" content="One or two sentence description or summary of my blog post" /&gt;
&lt;meta property="article:published_time" content="2013-01-28" /&gt;
</pre>

`article:published_time` needs to be an [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) formatted timestamp. It'll look something like `2013-01-28` or `2013-01-28T00:37Z` if you include the UTC time.

If you use categories or tags then you can include properties for those as well.

<pre data-language="html">
&lt;meta property="article:tag" content="Category 1" /&gt;
&lt;meta property="article:tag" content="Category 2" /&gt;
</pre>

Facebook [requires](https://developers.facebook.com/docs/technical-guides/opengraph/built-in-objects/#article) your `og:image` to be at least 50px by 50px but prefers a minimum of 200px by 200px. It is a good idea to create a separate copy of your image specifically sized for Facebook.

## Adding Twitter Cards

Twitter Cards have their own set of meta-tags that you are required to add to your page. Unlike Open Graph, you aren't required to import the namespace prefix on your page.

<pre data-language="html">
&lt;meta name="twitter:card" content="summary" /&gt;
&lt;meta name="twitter:title" content="Title of my blog post" /&gt;
&lt;meta name="twitter:url" content="http://zduck.com/mypost" /&gt;
&lt;meta name="twitter:image" content="http://zduck.com/mypost/twitter-image.png" /&gt;
&lt;meta name="twitter:description" content="One or two sentence description or summary of my blog post" /&gt;
</pre>

If you have a Twitter account then you can link to it as the author of the article.

<pre data-language="html">
&lt;meta name="twitter:creator" content="@JoshuaPoehls" /&gt;
</pre>

And if you have an account that represents your blog you can link to it as the publisher of the article.

<pre data-language="html">
&lt;meta name="twitter:site" content="@MyBlog" /&gt;
</pre>

Twitter Cards require a `twitter:url` but will fallback to `og:url` if it isn't found. Twitter also has fallbacks for `twitter:image` to `og:image`, `twitter:title` to `og:title`, and `twitter:description` to `og:description`. If you already have the Open Graph properties on your page then there isn't much point in duplicating them for Twitter unless you want to change how the information is displayed on Twitter specifically.

Twitter requires your images to be 120px by 120px and will resize and crop any image that is too large. It is a good idea  to have a copy of your image sized for Twitter and use the `twitter:image` tag in addition to `og:image`.

_Remember that to light up Twitter Cards for your website you have to [apply for participation](https://dev.twitter.com/docs/cards) with Twitter._

## Further Reading

Both the Open Graph protocol and Twitter Cards support a lot more than what I've covered. These formats have a lot to offer beyond the world of blogging. Do yourself a favor and skim the [Open Graph protocol](http://ogp.me/) and [Twitter Card documentation](https://dev.twitter.com/docs/cards) for an idea of all they can do.

Also, while Google+ supports the Open Graph protocol it actually prefers [schema.org microdata](http://schema.org) for [building its snippets](https://developers.google.com/+/plugins/snippet/). Google, Yahoo!, Bing, and other search engines also use schema.org microdata to better represent your pages in search results. That is a topic for another time.

Facebook also [has documentation](https://developers.facebook.com/docs/technical-guides/opengraph/) regarding their use of the Open Graph protocol.

## Testing Tools

Make sure you've implemented the meta-tags correctly by using a few tools to check your pages.

* [Twitter Card Preview](https://dev.twitter.com/docs/cards/preview)
* [Google Rich Snippets](http://www.google.com/webmasters/tools/richsnippets)
* [Facebook Debugger](https://developers.facebook.com/tools/debug)

## Bonus Tip

You can [claim your domain](https://developers.facebook.com/docs/insights/#claiming_a_domain) with Facebook and use their [Insights](www.facebook.com/insights) dashboard to see how well your links are doing.