---
title: "Fancy Vim Plugins"
date: 2018-12-24T12:30:00+01:00
tags: [ vim, vimplugins ]
draft: false
best: true
---

***Prerequisites:*** _basic knowledge of vim editor, minimal plugin configuration_.

I read a lot of plugin posts over the internet back when I begun using vim. While I still find them supportive for starters, they suggest the same plugins over and over.

Today I will talk about a bunch of unknown plugins which give the editor a modern look and feel. I call them _“unknown”_ because the number of stars these projects received doesn’t go above **1000**.

I will include installation commands for `vim-plug`[^1] plugin manager and optional configuration useful for the users. You can use your plugin manager of choice by replacing `Plug` with whatever it uses.

## 1. colorizer[^2]

Have you read an HTML color code and asked yourself which color represented? In this case, `colorizer` will save your day. I enjoy it the most when working on small CSS files and on dotfiles.

![](/img/fancy_vim_plugins/colorizer.jpg)

Installation:

```vim
Plug 'lilydjwg/colorizer'
```


## 2. rainbow[^3]

Doesn't matter which language you’re writing in, nested parentheses and braces give developers headaches; matches highlighting don’t help much either.

`rainbow` highlights the parentheses and braces based on their nesting level; as a result, the window will be hightlighted with “rainbow” colors, hence the plugin name.

But it’s easier to see with some lisp code:

![](/img/fancy_vim_plugins/rainbow_screenshot.png)

Installation:

```vim
Plug 'luochen1990/rainbow'
```

`rainbow` can the toggled using `:RainbowToggle`; to enable the plugin at startup add:

```vim
let g:rainbow_active = 1
```

## 3. vim-illuminate[^4]

`vim-illuminate` highlights the words in the current buffer matching the one under the cursor, letting you glance at once the local references of a variable or method.

![](/img/fancy_vim_plugins/vim-illuminate.gif)

```vim
Plug 'RRethy/vim-illuminate'
```

The default configuration shipped is sane and use `cursorline` as highlight group. Since I prefer a highlight group little more constrast with the background, I set `Visual` in place of `cursorline`:

```vim
hi link illuminatedWord Visual
```

## 4. vim-smooth-scroll[^6]

Window scroll in vim comply to the overall efficiency. However, scroll isn’t the nicest feature to expect from a modern editor, expecially when we run many instances of Chrome at the same time (I’m pointing at you Electron!). `vim-smooth-scroll` provides exactly this feature:

![](/img/fancy_vim_plugins/vim-smooth-scroll.gif)

```vim
Plug 'terryma/vim-smooth-scroll'
noremap <silent> <c-u> :call smooth_scroll#up(&scroll, 0, 2)<CR>
noremap <silent> <c-d> :call smooth_scroll#down(&scroll, 0, 2)<CR>
noremap <silent> <c-b> :call smooth_scroll#up(&scroll*2, 0, 4)<CR>
noremap <silent> <c-f> :call smooth_scroll#down(&scroll*2, 0, 4)<CR>
```

You can customize _dinstance_, _duration_ and _speed_ by changing the respective parameters of `smooth_scroll#` methods. I personally prefer a value of `5` for _duration_:

```vim
noremap <silent> <c-u> :call smooth_scroll#up(&scroll, 5, 2)<CR>
```

## 5. vim-search-pulse

`vim-search-pulse` add a tiny and peculiar feature: it pulses every time you scroll the search results by pressing `n`/`N`.

![](/img/fancy_vim_plugins/vim-search-pulse.mp4)

```vim
Plug 'inside/vim-search-pulse'
```

The pulse duration can be customized to user’s taste.

```vim
let g:vim_search_pulse_duration = 200
```

## bonus: semshi[^5]

***Note***: _the following plugin works in Neovim but doesn’t support Vim 8 due to a missing equivalent highlighting API, sorry folks! Have you considered upgrading to neovim?_

`semshi` provides semantic highlighting for Python. Semantic highlighting directly follow syntax’s one in color coding, coloring fixed words _and_ all the remaining words. Analizing the code, it rapidly gives each word a color based on its type, with a much better result than before.

With semshi
: ![After](/img/fancy_vim_plugins/w-semshi.jpg)

Without semshi
: ![Before](/img/fancy_vim_plugins/wo-semshi.jpg)

This plugin is fast, static and works on single files; it means that it doesn’t need to know how your project structure and can be invoked anywhere.

[^1]: https://github.com/junegunn/vim-plug
[^2]: https://github.com/lilydjwg/colorizer
[^3]: https://github.com/luochen1990/rainbow
[^4]: https://github.com/RRethy/vim-illuminate
[^5]: https://github.com/numirias/semshi
[^6]: https://github.com/terryma/vim-smooth-scroll
[^7]: https://github.com/inside/vim-search-pulse
