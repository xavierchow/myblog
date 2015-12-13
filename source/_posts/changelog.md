title: It's time to make a changelog
date: 2015-12-13 22:01:33
tags: git
---

## What's changelog and why use it?
Changelog records the history changes for a development, it lets users or contributors easily find what's changed in the certain release or tag.
Usually it containes records as fix bugs, features, deprecated or something else. Normally it's named with `CHANGELOG.md`, `History.md` etc. although the best convension I think is `CHANGELOG.md`.
Here is a pretty well post with comprehensive description for **changelog**. http://keepachangelog.com/ and from there you can also find why use changelog.
<!-- more -->  

## How? 
There are serveral approaches for creating changelog.

### Manually 
Absolutelly in this way your changelog will be most valuable as long as you care it. The cons is it costs some extra effort. For me, as I'm a lazy developer, I don't like this way.

### git changelog
This is a command in [git-extra](https://github.com/tj/git-extras/blob/master/Commands.md#git-changelog). It dumps the commits into the changelog with a tag name and is very easy to use. However, just wrapping development commits into a changelog also makes it meanningless because most of time it ends up with trival information such as `fix typo`. 

{% blockquote http://keepachangelog.com/ %}
As is the difference between good comments and the code itself, so is the difference between a change log and the commit log: one describes the why, the other the how.
{% endblockquote %}

### github-changelog-generator
If your project is highly relied on github features, (github-changelog-generator)[] is a good choice. It automatically collects the issues, pull requests, tags and labels 
into the changelog. The cons is that don't forget to make issues for you notable changes, bug fix and also have to label them, actually I don't think it's a cons because
thosse activities mentioned above are after all the best practises you should adhere to, right?

## Conclusion
No matter which approache you use, the important point is that you care about changelog and take care of it. You will have prifited from it.


