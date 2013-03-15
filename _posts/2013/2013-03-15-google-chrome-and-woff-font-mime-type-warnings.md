---
title: Google Chrome and WOFF font MIME type warnings
layout: post
categories: css
description: "If you are getting warnings like 'Resource interpreted as Font but transferred with MIME type font/x-woff' in Google Chrome then it means your web server is serving the font with the wrong MIME type. I'll show you how to fix that."
---

If you are getting warnings like **Resource interpreted as Font but transferred with MIME type font/x-woff** in Google Chrome then it means your web server is serving the font with the wrong MIME type.

It took awhile for the WOFF font format to get an approved MIME type so there are several variations hanging around.

* `font/x-woff`
* `font/woff`
* `application/x-font-woff`

Thank God an official type [was finally decided on](http://www.w3.org/TR/WOFF/#appendix-b), `application/font-woff`.

Unfortunately, Google Chrome (as of 26 beta) still expects you to use `application/x-font-woff` and doesn't recognize `application/font-woff` as a valid MIME type. This [has been fixed](https://bugs.webkit.org/show_bug.cgi?id=111418) in the WebKit trunk but hasn't been merged into Chrome yet. The [Chromium bug report](https://code.google.com/p/chromium/issues/detail?id=178823) has been closed so it should be coming soon.

Until it is merged into a Chrome release you have a couple of options.

1. You can switch to `application/x-font-woff` which will fix the Chrome warnings but isn't the approved MIME type. You'll need to change it again later.
2. You can switch to `application/font-woff` which will continue to cause warnings in Chrome but is the approved MIME type. Obviously the warnings will go away once the WebKit fix has been merged into Chrome but this will take a few months to make it into a release.

Since the warnings are only annoying I would recommend switching to the approved MIME type right away. The warnings will resolve themselves in time.

### IIS Express

The steps to fix this will vary by web server but for IIS Express this is how you'd do it.

1. Open `~/Documents/IISExpress/config/applicationhost.config`
2. Search for **".woff"** and you'll find a line like this:

        <mimeMap fileExtension=".woff" mimeType="font/x-woff" />
3. Change **mimeType** to `application/font-woff`

        <mimeMap fileExtension=".woff" mimeType="application/font-woff" />
4. Restart your IIS Express site.