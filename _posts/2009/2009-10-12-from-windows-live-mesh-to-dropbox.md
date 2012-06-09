---
layout: post
title: From Windows Live Mesh to Dropbox
categories: opinions
---
The other night I received a recommendation to try out Dropbox.
If you don't know what Dropbox is, here's the short of it.
Post-install, you'll see a "My Dropbox" folder on your desktop
(or wherever you chose to place it) and anything you put in there gets
automagically sync'd with every other computer you have Dropbox installed on.

From random documents and OneNote Notebooks to my [Keepass](http://www.keepass.info)
Database and books I'm reading, I have quite a few files that I like to
have access to no matter which computer I'm on. I could use a thumb drive
for this... and I have in the past, but the convenience of having everything
synchronized automatically for me and not having to remember to carry
around the thumb drive is too much to pass up.

For the past 6 months or so I've been using Windows Live Mesh to
synchronize these files between my computers. Before that I was
using Windows Live Sync. I'm actually still using Live Sync for
some things (more on that later) but I have made the switch from Mesh
to Dropbox. Bottom line, Dropbox "just works" and it keeps me in the
loop while it's doing it. Let me explain.

## Windows Live Mesh

This is Microsoft's vision for your personal cloud.
You get 5GB of online storage known as your Live Desktop.
You get file synchronization between your computers (just like Live Sync).
And you get remote desktop baked in.

Mesh is still a beta product and things seem to be moving slowly
on it judging by the sparse updates and lack of news via the blogging community.

<a class="img lightbox" href="{{site.url}}/assets/forposts/livemesh-dropbox/mesh_live_desktop.jpg">
  <img src="{{site.url}}/assets/forposts/livemesh-dropbox/mesh_live_desktop_small.jpg" /></a>

The best part of Mesh is that in addition to syncing folders
across your computer, you can *optionally* have it sync the folder
to your Live Desktop. Anything in your Live Desktop is available from any
web connected computer and this is very handy. Don't miss the key word though.
**Optional**. This is very important because I may have 20+ GB of music that
I want to sync between my computers, but I don't want that eating
up my 5GB of cloud space. With Mesh, this is easily handled by configuring
exactly which devices you want to sync each folder with. Guess what?
Your Live Desktop shows up as a device!

<a class="img lightbox" href="{{site.url}}/assets/forposts/livemesh-dropbox/mesh_folder_share_config.png">
  <img src="{{site.url}}/assets/forposts/livemesh-dropbox/mesh_folder_share_config.png" /></a>

What's the catch? Well for me the whole syncing part was too opaque.
I never felt like I really 'knew' if my latest changes had been sync'd
yet or not. Even with the fancy News sidebar that Mesh attaches to each
of your sync'd folders.

<a class="img lightbox" href="{{site.url}}/assets/forposts/livemesh-dropbox/mesh_log.jpg">
  <img src="{{site.url}}/assets/forposts/livemesh-dropbox/mesh_log_small.jpg" /></a>

All too often I would update files and trust Mesh to sync them only
to find out later that Mesh never picked up on the changes. Maybe it's
just the fancy UI they've given it, but I don't feel like the News pane
is giving me all the details about which files have been uploaded, when,
and which ones are still in progress.

Maybe if Mesh had been faithful in keeping my files in sync I wouldn't
care about seeing these details. Unfortunately it failed me and now I want
to see everything.

I only used the built in remote desktop a couple of times but I was
impressed that it worked pretty well, even when the computer I was connecting
to was behind a proxy. 

## Dropbox

Whereas Mesh and Live Sync allow you to synchronize multiple folders
between your computers, Dropbox only syncs one folder. Your My Dropbox folder.
This was a small adjustment for me to make but it was worth it. 

Dropbox is completely cross platform with support for Windows, Mac, Linux,
iPhone and other mobile phones. I only care about Windows and the iPhone though.

I get 2GB free from Dropbox which is plenty for my critical files.
Unlike Mesh, you can't choose to _not_ have some files sync'd to the cloud.
Everything in your My Dropbox goes to the cloud, as well as your other computers.
This is the only thing that I see Mesh having an upper hand in and its
actually the reason I'm still using Live Sync for some things.

The Windows client for Dropbox is perfect. I get status overlays on all
of the files and folders in My Dropbox that indicates whether they are
up-to-date or downloading (+1 for acting like TortoiseSVN!). I get real-time
balloon messages from my notification area whenever a file is changed, even
when it's on another computer. For instance if I update a file on my desktop,
I'll get a notification as soon as my laptop receives the update. This does
wonders for making me feel "in the loop" and confident that my files are being
kept up-to-date.

<a class="img lightbox" href="{{site.url}}/assets/forposts/livemesh-dropbox/dropbox_status_overlays.png">
  <img src="{{site.url}}/assets/forposts/livemesh-dropbox/dropbox_status_overlays.png" /></a>

Dropbox has the cloud part covered by letting you login and access all your
files via the browser. On top of this, they also keep past versions of your
files (think shadow copy) as backups. 

The iPhone application (free) is a real work of art. I expected slow and
clunky and I got fast and simple. Once you login you see a list of all your
files and folders alphabetized. If its a file type that the iPhone supports
(i.e photos or Office docs) you can open it up right on your phone. You can
also add photos to your dropbox from your iPhone.

Of course all of these apps offer great support for sharing files with
your friends and colleagues, but Dropbox offers one stand out feature here.
In addition to being able to share specific folders with friends, they give
you a built-in Public folder that you can drop any file into, then right-click
on the file to select an option to Copy a Public Link to that file. You can
send this link to anyone and they'll be able to access the file. This enables
a very quick workflow for sharing documents.

## Windows Live Sync

This one has been around for awhile and (unlike Mesh) its *not* a beta.
Also unlike Mesh, this one hasn't lost my confidence. 

There is no cloud storage with Live Sync. It only syncs directly between
your computers peer-to-peer style. This means to stay up to date your
computers really need be online all the time. When you change a file on
computer A it checks if computer B is online and if so it will upload
directly from A to B. If computer B is offline, it doesn't get the update.
When computer B comes back online it will look for updates from A, if A is
offline it won't get updates until it comes back.

Oh, and it has an activity log:

<a class="img lightbox" href="{{site.url}}/assets/forposts/livemesh-dropbox/live_sync_activity.png">
  <img src="{{site.url}}/assets/forposts/livemesh-dropbox/live_sync_activity.png" /></a>

No cloud storage means no piddly size limitations. Well, sort of.
You are limited to syncing 20 folders (top-level folders, this doesn't
include children) with up to 20,000 files in each. I'm not sure what the
per-file size limits are.

I mentioned above that I'm still using Live Sync in addition to Dropbox.
This is for 2 reasons:

1. I have a `C:\tools` folder that I want to sync. I don't want this
   to become `C:\My Dropbox\tools`.
2. I have some larger files (like music) that I don't care if its
   stored in the cloud, I just want it sync'd. If I used Dropbox
   for this my free storage limits would have run out within the first hour...
   
At the end of the day, if you need to sync files between computers,
I highly recommend either Dropbox or Windows Live Sync. As for Live Mesh?
This is admittedly still a BETA product and it acts like one. I'd wait until
it's more mature before going this route.