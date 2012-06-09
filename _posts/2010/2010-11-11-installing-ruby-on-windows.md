---
layout: post
title: Installing Ruby (and the DevKit) on Windows
categories: ruby windows
---

Here's my recipe for a painfree installation of Ruby on Windows.

1. Use the RubyInstaller from [www.ruby-lang.org](http://www.ruby-lang.org/en/downloads/).
   I used Ruby 1.9.2-p0.
2. Install the Ruby DevKit by following the instructions in the [Installation Overview](https://github.com/oneclick/rubyinstaller/wiki/development-kit).
   I used version DevKit-4.5.0-20100819-1536-sfx.

The DevKit is needed so that you can install gems that need to build native extensions.
Without it when you try to `gem install jekyll` you will see an error like this:

<pre>
ERROR:  Error installing jekyll:
ERROR: Failed to build gem native extension.

'make' is not recognized as an internal or external command,
operable program or batch file.
</pre>

After installing DevKit `gem install jekyll` will work like a champ.

Good luck!