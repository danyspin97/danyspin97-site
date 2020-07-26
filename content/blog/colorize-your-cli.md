---
title: "Colorize your CLI"
date: 2020-07-26T15:30:00+01:00
categories: ["showcase"]
tags: ['cli']
hacker_news_id: "23957325"
---

***Prerequisites***: _basic understanding of CLI and of common programs_.

The _[Command Line Interface]_ (abbreviated `CLI` for the rest of the post) is 
the swiss knife of many developers and sysadmins out there. Unix systems provide
a settings layer that can be rapidly accessed using some CLI utilities.

[Command Line Interface]: https://en.wikipedia.org/wiki/Command-line_interface

And that's just the tip of the iceberg. The CLI allows us to do almost
everything without leaving the comfort of the keyboard. From browsing files and directories (using [nnn] or [ranger]) to reading and sending emails (using
_[aerc]_).

[aerc]: https://aerc-mail.org/
[nnn]: https://github.com/jarun/nnn
[ranger]: https://ranger.github.io/

However, most programs we use:
- disable colors by default, sticking to the same default since their initial
  release. E.g.: **ls** and **grep** accept a `--color` argument for enabling colored output, _or_
- do not implement colors at all, e.g., **df**

There are a lot of references on the web regarding this topic, like [Color
output in console] page in the _Arch Linux_ wiki and various Stack Overflow
[questions](https://unix.stackexchange.com/questions/148/colorizing-your-terminal-and-shell-environment).
They propose multiple solutions that sometimes overlap with each other. One
must go through all the documentation, choose carefully and follow the
installation procedure, and this can become tedious in a short amount of time.

[Color output in console]: https://wiki.archlinux.org/index.php/Color_output_in_console

In this post, we will **setup a colourised** `CLI` **using the programs
described** above. This will be a comprehensive guide and there will be (almost)
no need to read any documentation.

Please note that this is based on my personal setup, based on my personal
preferences. Feel free to skip/change anything you dislike, as it won't impact
the rest of the guide.

## Starship

This is what the bash prompt looks like out of the box:

![](/img/colorize-your-cli/bash-prompt.png)

The first step into our colored journey is to making this prompt both more
meaningful and better to look at.

There are a lot of prompt plugins out there. However, most of them work only
in a single shell or require the installation of a framework like `Oh-My-<shell>`. What I have instead chosen is a shell agnostic prompt:
[**starship**](https://starship.rs/).

**starship** is a fast (really fast!) prompt written in Rust that works in all
shells (including exotic shells such as Ion).

![](/img/colorize-your-cli/starship-prompt.png)

Let's have a look at our new prompt inside a git repository:

![](/img/colorize-your-cli/starship-git.png)

</br>

**starship** is available in the following
[distributions](https://repology.org/project/starship/versions).
If your distribution isn't listed, you can run:

```
$ curl -fsSL https://starship.rs/install.sh | bash
```

To enable it in `bash`, add the following to the end of `~/.bashrc`:
```bash
eval "$(starship init bash)"
```

For enabling it in different shells, like `zsh` or `fish`, follow the
[getting started guide](https://starship.rs/guide/#%F0%9F%9A%80-installation)
on the official site.

## Generic Colourizer

Let's take as example a well known and widely used program: `df`.

![](/img/colorize-your-cli/df-plain.png)

As you can see, it's clear but plain and somehow hard to follow. Let's use
[**Generic Colourizer**](https://github.com/pengwynn/grc) to improve its output:

![](/img/colorize-your-cli/df-grc.png)

Now it seems like a total different program!

As we have just seen, **Generic colourizer** is a wrapper that uses regex to
add colors and modifiers (**bold**, _italic_) to a program's output.
Among the supported programs there are `id`, `env`, `sysctl` and `mount`.

You can install it using the package manager, as it is available in almost every
[distribution](https://repology.org/project/grc/versions).

To enable it in `bash`, add the following at the end of your `~/.bashrc`:

```
[[ -s "/etc/grc.bashrc" ]] && source /etc/grc.bashrc
```

To enable it in `zsh` or `fish`, respectively source `/etc/grc.zsh` or
`/etc/grc.fish`.

These configuration files will add aliases to the supported programs. I.e.,
next time you run `df`, the shell will automatically run `grc df`.

## ls

`ls` command is one of the commands that by default do not use colors, although
they are avaiable by using `--color` argument.

![](/img/colorize-your-cli/ls.png)

Simply add an alias in your config. For `bash` users, add the following in
your `~/.bashrc`:

```bash
alias ls='ls --color=auto'
```

## grep

![](/img/colorize-your-cli/grep.png)

As for ls, add an alias to `grep --color=auto`.

```
# ~/.bashrc

alias grep='grep --color=auto'
```

## diff

![](/img/colorize-your-cli/diff.png)

As for ls, add an alias to `diff --color=auto`.

```
# ~/.bashrc

alias diff='diff --color=auto'
```

## man

![](/img/colorize-your-cli/colored-man.png)

To add colors to `man`, we need to add colors to `less`. In `bash`, this can be
achieved by creating a file `~/.config/less/termcap` with the following content:

```sh
export LESS_TERMCAP_mb=$(tput bold; tput setaf 2) # green
export LESS_TERMCAP_md=$(tput bold; tput setaf 6) # cyan
export LESS_TERMCAP_me=$(tput sgr0)
export LESS_TERMCAP_so=$(tput bold; tput setaf 3; tput setab 4) # yellow on blue
export LESS_TERMCAP_se=$(tput rmso; tput sgr0)
export LESS_TERMCAP_us=$(tput smul; tput bold; tput setaf 7) # white
export LESS_TERMCAP_ue=$(tput rmul; tput sgr0)
export LESS_TERMCAP_mr=$(tput rev)
export LESS_TERMCAP_mh=$(tput dim)
export LESS_TERMCAP_ZN=$(tput ssubm)
export LESS_TERMCAP_ZV=$(tput rsubm)
export LESS_TERMCAP_ZO=$(tput ssupm)
export LESS_TERMCAP_ZW=$(tput rsupm)
export GROFF_NO_SGR=1         # For Konsole and Gnome-terminal
```

And then in your `.bashrc`:

```sh
# Get color support for 'less'
export LESS="--RAW-CONTROL-CHARS"

# Use colors for less, man, etc.
[[ -f ~/.config/less/termcap ]] && . ~/.config/less/termcap
```

If you are using `fish`, add the following to your `~/.config/fish/config.fish`:

```fish
set -xU LESS_TERMCAP_md (printf "\e[01;31m")
set -xU LESS_TERMCAP_me (printf "\e[0m")
set -xU LESS_TERMCAP_se (printf "\e[0m")
set -xU LESS_TERMCAP_so (printf "\e[01;44;33m")
set -xU LESS_TERMCAP_ue (printf "\e[0m")
set -xU LESS_TERMCAP_us (printf "\e[01;32m")In 
set -xU LESS "--RAW-CONTROL-CHARS"
```

## highlight

When reading a file using `less`, the output is the plain file, as we were
reading the file through `cat`.

![](/img/colorize-your-cli/less.png)

This can be improved by by passing the file through
[**highlight**](http://www.andre-simon.de/doku/highlight/highlight.php).

![](/img/colorize-your-cli/highlight.png)

We now have **syntax highlighting**!

`highlight` can be found in almost all [distributions](https://repology.org/project/highlight/versions). After
installing it using your package manager, add the following to your `~/.bashrc`:

```sh
export LESSOPEN="| /usr/bin/highlight %s --out-format xterm256 --force"
```

By adding `--line-numbers`, highlight will also show line numbers.
You can also customize its style by adding `--style <style>`; for the
screenshot, the `molokai` theme has been used. Please read the
[documentation](http://www.andre-simon.de/doku/highlight/highlight.php)
for further customizations.

## Bonus for bash users: ble.sh

`bash`, contrary to its alternatives `zsh` and `fish`, **does not have a
powerful line editing**. By default the command you write won't be highlighted
and the completion won't be shown.

![](/img/colorize-your-cli/bash-no-blesh-1.png)
![](/img/colorize-your-cli/bash-no-blesh-2.png)

[**Bash Line Editor**](https://github.com/akinomyoga/ble.sh) (also called
`ble.sh`) is a flexible plugin that makes bash closer to the other two shells.

Let's have a look at the exact two situations above and the difference `ble.sh`
does.

![](/img/colorize-your-cli/bash-ble-1.png)
![](/img/colorize-your-cli/bash-ble-2.png)

To install `ble.sh` run the following commands:

``` sh
$ cd /tmp
$ git clone --recursive https://github.com/akinomyoga/ble.sh.git
$ cd ble.sh
$ make
$ make install
```

`ble.sh` will be installed inside `~/.local/share/blesh`.

Now add the following lines at the top of your `~/.bashrc`:

```sh
[[ $- == *i* ]] && source ~/.local/share/ble.sh --noattach
```

And the following lines at the end of the same file:

```bash
((_ble_bash)) && ble-attach
```

## Conclusion

After setting up the above programs/aliases you will have a different
experience when using your CLI.

In this post we have mostly used **common command line programs**, although
_modern alternatives exist_, e.g., [ripgrep] as alternative to `grep`. Those
will be probably covered in another post.

[ripgrep]: https://github.com/BurntSushi/ripgrep

Did you like the configuration presented through this post? Do you have other
way of adding colors to your CLI? _Comment below and let me know_!
