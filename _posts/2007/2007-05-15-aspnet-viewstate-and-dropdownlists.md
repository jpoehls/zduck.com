---
title: ASP.NET ViewState and DropDownLists
layout: post
categories: asp.net
---

Did you know ViewState is the root of all evil? Ok, maybe not all evil but certainly 90% of all ASP.NET page bloat. This isn't a strike against .NET, just a sign of how powerful it really is. Any platform that can enable developers to screw up so easily and never realize it has to be very powerful indeed. To borrow a quote from another blog I was reading&#0133;

> ASP.NET is so powerful that it can enable you to be an incredibly bad programmer FASTER THAN EVER.  
> Don't program by coincidence.

### Background

Yesterday I wanted to disable ViewState on one of my DropDownLists. Reason? The drop-down has 3000 items in it. Yes, I know, sick and wrong... It's bad enough that the drop-down itself is so large but with ViewState every item in the drop-down essentially gets sent to the client twice! How else do you think ViewState remembers what items were in your drop-down when you do a PostBack? It's very handy, but for large lists it can significantly increase the size of the page you are sending to the client.

Just to give you some idea, by disabling ViewState on my drop-down I saved nearly 200 kB. That's significant bandwidth savings, especially for those last couple of dial-up users out there.

### Problem

Now that ViewState is disabled for my drop-down I have a few issues:

1. ASP.NET pulls the SelectedValue property of the DropDownList out of ViewState. Guess what? No ViewState == no SelectedValue.
2. Even though `AutoPostBack` still works, the `SelectedIndexChanged` event has lost its firing pin.
3. My drop-down now loses all of its items after I any post-back occurs. Suprise, it also forgets which items was selected.

### Solution

1. Despite ASP.NET's attempts to deceive you into thinking that everything that matters must be a Property of the control (i.e. `SelectedValue`)... ASP.NET is just a fancy wrapper around HTTP and thus any values that are set from the client are received by the server via the `Request.Form` or `Request.QueryString`.

    Since these are POST-backs, we're interested in `Request.Form`. Now we just need to know that ASP.NET uses the control's `UniqueID` property to name the control on the client, this means that `Request.Form[myDropDownCtrl.UniqueID]` will give me the value of that drop-down list as it was sent from the client. This is the exact same as `myDropDownCtrl.SelectedValue`. Bingo!

2. There are ways to get the `SelectedIndexChanged` event to fire again but I found an easier work-around for my needs. This also covers #3.

<pre data-language="generic">
protected string MySelectedValue
{
 get
 {
   return (ViewState["MySelectedValue"] == null) ? null : ViewState["MySelectedValue"].ToString();
 }
 set
 {
   ViewState["MySelectedValue"] = value;
 }
}

protected void Page_Load(object sender, EventArgs e)
{
 // Check if the selectedValue has changed.
 string selectedValue = Request[myDropDownCtrl.UniqueID];
 if (selectedValue != MySelectedValue)
 {
   // Value has changed, do whatever you would do in the SelectedIndexChanged event here.
 }
 MySelectedValue = selectedValue;

 // TODO: Populate the drop-down control
 PopulateMyDropDownCtrl();

 // set the set the selectedValue on the drop-down
 if (selectedValue != null)
 {
   myDropDownCtrl.Items.FindByValue(selectedValue).Selected = true;
 }
}
</pre>

_I didn't actually test this exact code, so&#0133; this code comes as-is, untested and without any guarantees. Let me know if there is a bug and I'll correct it._