title: Bring Emacs to the foreground on macOS Catalina
date: 2020-06-20 14:07:16
tags:
---

I have been using Emacs for a few months, I always keep one instance and use a global shortcut key `cmd  + e` to open or switch it to the foreground.
I did it with [Karabiner-Element](https://karabiner-elements.pqrs.org/) key mapping
```
    {
        "from": {
            "key_code": "e",
            "modifiers": {
                "mandatory": [
                    "right_command"
                ]
            }
        },
        "to": [
            {
                "shell_command": "open -a 'emacs'"
            }
        ],
        "type": "basic"
    }


```


Recently I upgraded the macOS to Catalina and found the `open -a emacs` can not bring it to foreground anymore. I spent hours searching with google until I found this post(https://spin.atomicobject.com/2019/12/12/fixing-emacs-macos-catalina/).

<!-- more -->  

As explained in the post, I never noticed the binary in Emacs.app is actually a ruby script which was working before Catalina, but now the OS fails to bring it to the front as the binary doesn't match the ruby script.

I tried the workaround mentioned in the [post](https://spin.atomicobject.com/2019/12/12/fixing-emacs-macos-catalina/) above and figured out I need to use Emacs-x86_64-10_10 instead.
```
% cd /Applications/Emacs.app/Contents/MacOS_
% mv Emacs Emacs-launcher
% mv Emacs-x86_64-10_10 Emacs
% cd /Applications/Emacs.app/Contents/
% rm -rf _CodeSignature
```

However, I still get some dependencies errors after the renaming,
```
Warning: arch-dependent data dir '/Users/build/workspace/Emacs-Multi-Build/label/mavericks/emacs-source/nextstep/Emacs.app/Contents/MacOS/libexec/': No such file or directory
Warning: arch-independent data dir '/Users/build/workspace/Emacs-Multi-Build/label/mavericks/emacs-source/nextstep/Emacs.app/Contents/Resources/etc/': No such file or directory
Warning: Lisp directory '/Users/build/workspace/Emacs-Multi-Build/label/mavericks/emacs-source/nextstep/Emacs.app/Contents/Resources/lisp': No such file or directory
Error: charsets directory not found:
/Users/build/workspace/Emacs-Multi-Build/label/mavericks/emacs-source/nextstep/Emacs.app/Contents/Resources/etc/charsets
Emacs will not function correctly without the character map files.
Please check your installation!
```

After another round of searching, I ended up with a solution according to the suggestion here: https://github.com/caldwell/build-emacs/issues/57.
I put the following alias to my zshrc and everything returns to normal as before.

`alias emacs=/Application/Emacs.app/Contents/MacOS/Emacs`


Hopefully, this post would also help anybody who is suffering the same wacky issue as me.

