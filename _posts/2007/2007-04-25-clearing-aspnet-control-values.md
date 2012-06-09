---
title: Clearing ASP.NET control values
layout: post
categories: asp.net
---

Have you ever needed to clear the values of a bunch of TextBoxes on your ASP.NET page? Not all the TextBoxes, but maybe all the ones in a certain Panel. Or maybe you need to reset all the DropDownLists too?

This is a method I wrote to do just that. It clears the values of all child controls under a given parent. Enjoy.

<pre data-language="generic">
public static void ClearControlValues(Control parentCtrl, bool recursive)
{
 ClearControlValues(parentCtrl, recursive, new List&lt;Control&gt;(0));
}

public static void ClearControlValues(Control parentCtrl, bool recursive, List&lt;Control&gt; ignoreControls)
{
 foreach (Control ctrl in parentCtrl.Controls)
 {
     bool skipMe = false;

     foreach (Control ignoreCtrl in ignoreControls)
     {
         if (ctrl.ID == ignoreCtrl.ID)
         {
             skipMe = true;
             break;
         }
     }

     if (skipMe)
         continue;

     if (ctrl.GetType() == typeof(TextBox))
         ((TextBox)ctrl).Text = string.Empty;

     if (ctrl.GetType() == typeof(DropDownList))
         ((DropDownList)ctrl).ClearSelection();

     if (ctrl.GetType() == typeof(RadioButton))
         ((RadioButton)ctrl).Checked = false;

     if (ctrl.GetType() == typeof(CheckBox))
         ((CheckBox)ctrl).Checked = false;

     if (ctrl.GetType() == typeof(RadioButtonList))
         ((RadioButtonList)ctrl).ClearSelection();

     if (ctrl.GetType() == typeof(CheckBoxList))
         ((CheckBoxList)ctrl).ClearSelection();

     if (ctrl.GetType() == typeof(ListBox))
         ((ListBox)ctrl).ClearSelection();


     if (recursive == true)
     {
         ClearControlValues(ctrl, true, ignoreControls);
     }
 }
}
</pre>