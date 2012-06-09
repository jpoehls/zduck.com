---
layout: post
title: Installing Python Pygments on Windows
categories: python windows
---

While I was updating this blog to use [Jekyll](http://www.jekyllrb.com),
I wanted to enable some nice syntax highlighting for any code snippets I may post.
Jekyll provides support for this through [Pygments](http://pygments.org/) which is
a pretty mature and widely used syntax highlighter built on Python.

I find it a little ironic that I need to install Python to enable a feature in a Ruby program.
Though it is nice to see that the Jekyll team isn't afraid to leverage great tools even if they
aren't written in Ruby.

I should note that this is my first experience doing anything with Python. That said, here
are the steps I fumbled through to install Pygments. This is officially the first
Python egg I've ever hatched.

What the heck is an egg? Eggs are one way to package up
a group of Python scripts for easy distribution. This makes 
eggs the equivelant of [Ruby gems](http://en.wikipedia.org/wiki/RubyGems).

To create or install eggs you need to have
[easy_install](http://pypi.python.org/pypi/setuptools) installed.

Here are the steps I took to do that.

1. Download ez_setup.py from [http://peak.telecommunity.com/dist/ez_setup.py](http://peak.telecommunity.com/dist/ez_setup.py)
   > **Note:** The official documenation for easy_install can be found at
   > [http://peak.telecommunity.com/DevCenter/EasyInstall](http://peak.telecommunity.com/DevCenter/EasyInstall).

2. From the command line run: `python ez_setup.py`.
   This will put an easy_install.exe file (and related scripts) in your `C:\Python2x\Scripts` folder.

   > **Note:** If your Python scripts folder is not already in your
   > <a href="http://en.wikipedia.org/wiki/PATH_(variable)">PATH environment variable</a> you
   > may want to add it. This will make it easier to run scripts from that directory later on.

Now that easy_install is available, I can download and install the Pygments egg that I need.

1. Download the latest Pygments egg for your version of Python from [http://pygments.org/download](http://pygments.org/download)

2. From the command line run: `easy_install Pygments-x.y.z-pyX.egg`.

3. This will put add a 'pygmentize' script to your default Python scripts folder (probably C:\Python2x\Script).

And that's it. Of course after going through all of that I discovered that
there is [a bug](http://github.com/mojombo/jekyll/issues/#issue/76) in
the current release of Jekyll that breaks its pygments support on Windows.
Fortunately someone has fixed this bug in [a fork of Jekyll](http://github.com/mojombo/jekyll/issues/#issue/76)
but I haven't gotten around to trying that out yet. So for now I have no fancy syntax highlighting, but I did learn
how to install an egg!

You can find much more knowledgable information on Python Eggs
[here](http://mrtopf.de/blog/python_zope/a-small-introduction-to-python-eggs/)
and [here](http://mxm-mad-science.blogspot.com/2008/02/python-eggs-simple-introduction.html).