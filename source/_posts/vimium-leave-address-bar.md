title: How to leave focus from Chrome address bar without mouse
tags:
  - vim
  - chrome
date: 2016-03-07 23:45:05
---

As a vimer, I have been using a plugin called [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en) in chrome for a long time. It keeps me to play with the browser in vim hotkey conventions, therefore I don't need mouse or touchpad to scroll/jump link/history back etc, however there is one thing sometimes annoys me, the **address bar**.
<!-- more -->  

Although most of the time you can use `o` or `O` to open URL or history in current tab or new tab, `Ctrl-L` to highlight the address bar and modifying the URL is not a rare case.
Here comes the issue, when the address bar is highlighted(i.e in input mode), unless you click the `enter` to reload the whole page the vimium is not active, neither can you expect to press `esc` to back to the page.

In such situation, to get focus back to the page, you have 3 choices:

1. use mouse to click the page(html body), this is the most straightforward one but I don't want to move my fingers from keyboard, so it passed.
2. use tab, it can move focus to the next control in browser and eventually you can focus on html body and continue the vimium key shortcuts.
However, if you are a guy as me has lots of plugins installed, you have to tab multiple times to get the focus on page. It's no ideal solution.
![](/images/chrome-plugins.png)
3. This one is a bit tricky but it works. `Cmd-F`(Or `Ctrl-F` at Windows) to trigger the search box, and input any character(the simplest one is `blank`) then `esc`, it brings you back to web page as well, but this solution has two cons:
  * It needs 3 key-strokes, not so concise.
  * If you are in the middle of the page(by scrolling or whatever way), then the combination above will always bring you back to the top of page. Quite annoying, isn't it?

Thanks to the [@ReyCharles](http://superuser.com/users/106061/reycharles) 's answer [here](http://superuser.com/questions/324266/google-chrome-mac-set-keyboard-focus-from-address-bar-back-to-page/324267#324267), got the Holy Grail as follows,

1. In search engines setting, add a new one.
2. Name the new engine with whatever you like, I named it as `leaveAddressBar`.
3. A simple hot key for `Keyword`, as the illustration I used `u`.
4. In `URL` field, input `javascript:`, this is the point which moves focus back to the page.
![](/images/search-engine.png)

Done, now feel free to fiddle the address bar without mouse!
