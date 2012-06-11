---
title: Syntax highlighting for Nginx in VIM
layout: post
categories: vim nginx
---

Thanks to [Evan Miller](http://www.vim.org/scripts/script.php?script_id=1886), adding VIM syntax highlighting for Nginx config files is a breeze.

First, install VIM if you haven't already. On Arch Linux, it goes like this:

<pre>
>pacman -Sy vim
</pre>

Create a folder for your VIM syntax files.

<pre>
>mkdir -p ~/.vim/syntax/
</pre>

Download the syntax highlighting plugin.

<pre>
>curl http://www.vim.org/scripts/download_script.php?src_id=14376 -o ~/.vim/syntax/nginx.vim
</pre>

Add it to VIM's file type definitions. Make sure to adjust the path to your Nginx installation if you need to.

<pre>
>echo "au BufRead,BufNewFile /etc/nginx/conf/* set ft=nginx" >> ~/.vim/filetype.vim
</pre>

Now enable syntax highlighting in your .vimrc file.

<pre>
>echo "syntax enable" >> ~/.vimrc
</pre>

That's it. Now you'll have nice colors when you edit your Nginx configs with VIM!

<pre>
>vim /etc/nginx/conf/nginx.conf
</pre>

<p><img alt="Screenshot of VIM with syntax highlighting in an Nginx config file." src="{{site.url}}/assets/forposts/nginx-vim-colors.png" /></p>