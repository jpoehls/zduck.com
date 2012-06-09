---
layout: post
title: How to Recover Deleted Files in TFS
categories: tfs coding
---

**Disclaimer:** I didn't work for this knowledge. Google provided
it for me on a silver platter from the [TFS Forums](http://social.msdn.microsoft.com/Forums/en-US/tfsversioncontrol/thread/0836d05c-79f6-4fa9-9adf-75248cc314ff).

So you just found out that last week while you were cleaning house in your project's
TFS source control that you deleted something that was still being used.

**No problem!**

Here's the fast track to recovering those deleted items.

1. In Visual Studio, go to `Tools` | `Options` | `Source Control` | `Visual Studio Team Foundation`
2. Check the box to `Show deleted items in the Source Control Explorer`

	 <a class="img lightbox" href="{{site.url}}/assets/forposts/tfs-deleted-files/tfs_deleted_files_option.png">
	 	<img src="{{site.url}}/assets/forposts/tfs-deleted-files/tfs_deleted_files_option.png" width="400" /></a>
	 
3. Open up Source Control Explorer (`View` | `Other Windows` | `Source Control Explorer`)
4. Browse to where the deleted items used to live.

	 <img src="{{site.url}}/assets/forposts/tfs-deleted-files/tfs_deleted_files_showing.png" />
	 
5. You see all those faded folders with x's on them? Those are the now undead
	 folders living among the rest of your source. If you are looking for a specific
	 file instead of a folder, you can see those as well. In the list of tiles they
	 will be grayed out and in the `Latest` column will say `Deleted`.
6. After you locate what you want to undelete, right-click on the item and click Undelete.

	 <img src="{{site.url}}/assets/forposts/tfs-deleted-files/tfs_deleted_files_context_menu.png" />
	 
7. That's it! The resurrection is now part of your pending changes.
   
Note that I wrote these steps for Visual Studio 2008 and TFS 2008. Older and newer versions beware.