---
layout: post
title: Notepad++ window switcher plugin
---

For quite some time, [Notepad++](http://notepad-plus.sourceforge.net/uk/site.htm) has become my favourite text editor. I wrote a small plugin to fit it even more with my coding habbits. If you want to switch to a certain window with a shortcut and/or open/switch to the corresponding source/header file for the current document, then this plugin might do what you want.

This is the initial version of the __Window Switcher__ Notepad++ plugin.


###What does it do

* it allows window switching bound to a shortcut. Default shortcuts are set to <kbd>ALT+1<kbd> ... <kbd>ALT+0</kbd>
* it switches between the corresponding `.h`/`.c`/`.cpp` file through the means of a shortcut. The default shortcut is <kbd>ALT</kbd>+<kbd>O</kbd>

###How to install

1. copy the plugin corresponding to your Notepad++ version into the Notepad++\plugins directory. Use `NppWindowSwitcherAnsi.dll` for ANSI version and `NppWindowSwitcherUnicode.dll` for UNICODE.
2. configure the shortcuts
3. if you want to use the default shortcuts, be aware of the fact that <kbd>ALT</kbd>+<kbd>1</kbd> .. <kbd>ALT</kbd>+<kbd>8</kbd> are bound by default to _collapse current level_ functionality. Go to `Settings -> Shortcut mapper -> Main Menu` and change the shortcuts so they point to something else. I remaped them to <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>1</kbd> ... <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>8</kbd>
4. enjoy

Download the plugin [here]({{ site.url }}/assets/NppWindowSwitcher.zip).