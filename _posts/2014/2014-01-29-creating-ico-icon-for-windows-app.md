---
title: Creating an ICO icon file for your Windows app
layout: post
categories: windows
description: "A quick walkthrough of how to convert PNGs into a proper ICO file for your Windows app."
---

Every Windows app needs an icon. Even yours.

If you are a developer then this task probably sounds simpler than it may turn out to be. First off, a lot of graphics programs ([Paint.NET][paint], for example) don't support saving as an ICO file, the format needed by Windows.

Don't sweat. I've got you covered. Here is the developer cheat sheet for creating an app icon file.

> **Pro tip:** ICO files can contain _multiple_ icons. Often they will contain multiple versions of the same icon in different sizes. For example, a 64px and a 16px icon, each optimized for their respective size.
>
> [Wikipedia][ico] has a lot more interesting info on the ICO format.

1. Create your icon. Make it square. Make it high res. Remember, you can always size down but it is harder to size up. I like to go with a minimum of 256px.
2. Export your icon to PNG files at multiple sizes. My minimum recommendation would be the following: 16px, 32px, 48px, 64px, 96px.
3. Use [PNGGauntlet][pnggauntlet] or [PNGOut][pngout] to compress those PNGs. You won't lose quality but you will shed a lot of wasted bytes.

	<img alt="Screenshot of PNGGauntlet" src="{{site.url}}/assets/forposts/creating-ico-files/pnggauntlet.png" />

4. Use [Icon Maker][iconmaker] to create your ICO file. Windows supports a single ICO file with multiple sizes embedded and that's what we want to do. Create a single ICO file with each of your specifically sized PNGs in it. You could also just drag in the largest PNG (96px for example) and trust Windows to resize it nicely.

	<img alt="Screenshot of Icon Maker" src="{{site.url}}/assets/forposts/creating-ico-files/iconmaker.png" />
5. Save out the ICO file and use it in your application.

That's it! The biggest trick is knowing the tools. Icon Maker is a life saver. You should also understand what size icon(s) to use so that your app is well represented throughout the Windows UI. There is a [good thread on StackOverflow][so] with more details on this.

I'm not an expert in this area so I'm sure there are things I'm missing to further optimize your icon. Add any of your own tips in the comments!

[ico]: http://en.wikipedia.org/wiki/ICO_(file_format)
[paint]: http://www.getpaint.net/
[so]: http://stackoverflow.com/a/3244679/31308
[pnggauntlet]: http://pnggauntlet.com/
[pngout]: http://advsys.net/ken/util/pngout.htm
[iconmaker]: http://www.inedo.com/downloads/icon-maker