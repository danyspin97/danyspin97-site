---
title: "Termish[1] = ddgr"
date: 2018-02-25T10:32:38+01:00
categories: ["termish"]
tags: ["terminal", "search engine", "duckduckgo"]
draft: false
---

What is the most common thing we do each day? We browse the internet using a search engine, from _[Google]_ to _[DuckDuckGo]_.

But what's the process?

Open the browser, fire up the search and wait for results; then take the mouse and click on it, or if there is some vim-like plugin installed, press f and wait for popups.

It is too long for something we do so often, isn't it?

There are two different terminal utilities which improve this simple course of action, respectively for _[Google]_ and _[DuckDuckGo]_ : **[googler]** and **[ddgr]**. </br> We are going to talk about the latter.

# Overview

_[ddgr]_ is a python3 script, ~1800 lines of code as of version `1.2`, which take your query and show you the results. It doesn't waste _precious_ space on your screen: there are only your prompt and you.

## How it works

There are two ways to call _ddgr_: with or without the search query as an argument. Both lead to the prompt, which is called _omniprompt_, where you can:

- select an index to open a result
- choose to open either in a text-based or GUI browser
- go back and forth between the result pages
- use DuckDuckGo _[bangs]_ or site specific search

</br>
If you call the utility without the search query, you'll need to insert it afterwards and you cannot take advantage of the various search options, like "I'm feeling Ducky" or time based search. However, this way it can be easily called from a window manager like i3/Sway:

~~~i3
bindsym $mod+c exec alacritty -e ddgr
~~~

</br>
But lets _ddgr_ talk for itself. Here's an asciinema:

<script src="https://asciinema.org/a/151849.js" id="asciicast-151849" async></script>

## Features

Quoting the exhaustive list of [features] from readme:

- Fast and clean (no ads, stray URLs or clutter), custom color
- Designed to deliver maximum readability at minimum space
- Specify the number of search results to show per page
- Navigate result pages from omniprompt, open URLs in browser
- Search and option completion scripts for Bash, Zsh and Fish
- DuckDuckGo Bang support (along with completion)
- Open the first result directly in browser (as in I'm Feeling Ducky)
- Non-stop searches: fire new searches at omniprompt without exiting
- Keywords (e.g. filetype:mime, site:somesite.com) support
- Limit search by time, specify region, disable safe search
- HTTPS proxy support, Do Not Track set, optionally disable User Agent
- Support custom url handler script or cmdline utility
- Comprehensive documentation, man page with handy usage examples
- Minimal dependencies


## Installation

_[ddgr]_ is available pretty much in every distro as shown [here][1].

# Conclusion

_[ddgr]_ is really a must for people who use DuckDuckGo and want a clean terminal utility; it worked flawlessly for me in the past months, both in features and stability. Plus, it can encode the results in JSON format, making effortless the implementation in other scripts.

<iframe src="https://fosstodon.org/@danyspin97/99604238432944546/embed" class="mastodon-embed" style="max-width: 100%; border: 0" width="400"></iframe><script src="https://fosstodon.org/embed.js" async="async"></script>

[Google]: https://google.com
[DuckDuckGo]: https://duckduckgo.com
[googler]: https://github.com/jarun/googler
[ddgr]: https://github.com/jarun/ddgr
[Jarun]: https://github.com/jarun
[bangs]: https://duckduckgo.com/bang
[features]: https://github.com/jarun/ddgr#features
[1]: https://github.com/jarun/ddgr#from-a-package-manager
