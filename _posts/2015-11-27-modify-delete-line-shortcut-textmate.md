---
layout: post
title:  "Modify the 'Delete Line' shortcut in TextMate"
date:   2015-11-27 
categories: dev tools
---


By default, [TextMate](https://macromates.com/) binds the *Delete Line* action from the Text bundle to **^&#8679;K**, but I much prefer the **&#8984;D** combination. Unfortunately, this is not configurable from TextMate's preferences.

Despite the lack of GUI options, TextMate's key binding can be modified using the [Cocoa preference file](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/EventOverview/TextDefaultsBindings/TextDefaultsBindings.html), which allows to assign a different "low-level" function to a key. However, this is not really convenient to create a shortcut mapped to a "Menu" action.

### OS X Shortcuts

Instead, OS X provides an easy way to create shotcuts for any application.

Go to *System preferences*, *Keyboard* and open the *Shortcuts* tab. Select *All applications* then click the `+` icon. Use the following configuration:

![screenshot]({{ site.baseurl }}/assets/2015-11-27-textmate-shortcut.png){: .img-medium}

If the new shortcut collides with an existing one, the resulting behavior might be unexpected. Usually, shortcuts using the **&#8984;** key work well enough for me.
