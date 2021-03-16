---
title: "Modern CLI Applications: Part 1"
date: 2020-01-08T16:20:00+01:00
tags: ["cli", "workflow"]
categories: ["showcase"]
---

Following the [post](/blog/colorize-your-cli/) about colorizing the CLI,
this time you will discover some interesting CLI application to enhance your
workflow. For each app you will see its usage in real world examples and
a comparison with the other applications. This showcase will be split over
different posts with 4 applications each.

Ready? Let's begin the journey!

***Prerequisites:*** _Basic knowledge about Unix command line and its
applications_.

***Notes:*** _For each software listed here, a link to its
[repology](https://repology.org) page will be attached; this way you can
check if it has been packaged for your distribution of choice and use the
system package manager to install it_.

## exa

|
-|-
Homepage: | https://the.exa.website/
Repository: | https://github.com/ogham/exa
Written in: | _Rust_
Repology: | https://repology.org/project/exa/
Dependencies: | _libgit2_ (optional)

`ls` is the most used application while interacting with the CLI. For this
reason the first application is its modern alternative: `exa`.

The invocation is the same as `ls`:

![](/img/modern-cli-applications-1/exa-1.png "Exa invocation with no arguments")

Yay, exa printed out some colors. However, ls can do the same when using
`--colors=auto`. Let's try some advanced option.

![](/img/modern-cli-applications-1/exa-2.png
        "exa invoked with advanced options")

The options used above are self-documenting, but if you want to know more about
them, just call the help page of exa ( `exa --help` ).

You can personalize further how exa looks by adding more arguments.
For example this is my personal preference:

![](/img/modern-cli-applications-1/exa-3.png
        "exa invoked with my personal options")

### Shortcut

Wrapping up all the features highlighted above, here it is a function to use
as `ls` alternative (here it is mapped to `l` but feel free to map it
according to your preference):

```bash
$ alias l='exa --icons --all --git --git-ignore --group-directories-first'
```

You can notice that `--long` have been removed, so that it can be added when you need call `l`:

```bash
$ l --long
```

You can also use its abbreviated form `-l` or add another alias `ll`:

```bash
$ alias ll='l --long'
```

## delta

|
-|-
Repository: | https://github.com/dandavison/delta
Written in: | _Rust_
Repology: | https://repology.org/project/git-delta/versions
Dependencies: | _libgit2_

The next application is a git diff viewer: `delta`.

![](/img/modern-cli-applications-1/delta-1.png
        "git diff output")
![](/img/modern-cli-applications-1/delta-2.png
        "delta output")

Delta takes git output, formats it in a stylish way and add syntax
highlighting. It supports many git commands like the following (and more):

- `git diff`
- `git show`
- `git log -p`
- `git stash show -p`
- `git reflog -p`
- `git add -p`

To add `delta` to `git`, add the following to your `~/.gitconfig` file:

```
[core]
    pager = delta

[interactive]
    diffFilter = delta --color-only
```

### Themes

`delta` supports many themes; you can see the complete list by running:

```bash
$ delta --list-syntax-themes
```

After you have found one that you like or that pairs well with your terminal
theme, add it to the config (`~/.gitconfig`):

```
[delta]
    syntax-theme = Monokai Extended
```

Plus and minus color have to be chosen manually. Here I use the ones for
the Monokai theme (in `delta` section as used above):

```
    plus-color = "#ff6188"
    minus-color = "#a9dc76"
```

### Side by side

![](/img/modern-cli-applications-1/delta-3.png
        "delta with side-by-side option enabled")

To enable side by side comparison, add the following to `~/.gitconfig`:

```
[delta]
    side-by-side = true
```

### Bonus: color-moved

Recent versions of git (>= v2.17, April 2018) supports new `--color-moved`
argument that will highlight lines that have been moved with a different
style than the one used with added/removed lines. `delta` will automatically
decect when this feature has been enabled, and modify its output accordingly.
All you have to do is enable `--color-moved` in git by default, by adding the
following to your `~/.gitconfig`:

```
[diff]
    colorMoved = default
```

![](/img/modern-cli-applications-1/delta-4.png
        "delta when colorMoved has been enabled in git")

You can also customize the colors by adding:

```
[color "diff"]
    oldMoved = "#ab9df2"
    newMoved = "#ffd866"
```

***Notes:*** _`delta` detect `color-moved` usage by reading raw colors from
git output. If it causes problem, disable this feauture in `delta`_.

```
[delta]
    inspect-raw-lines = false
```

## dog

|
-|-
Homepage: | https://dns.lookup.dog/
Repository: | https://github.com/ogham/dog
Written in: | _Rust_
Repology: | https://repology.org/project/dog-dns/versions
Dependencies: | _OpenSSL_ (optional)

How many of you use `dig` to perform a DNS query from the terminal? And how
many of you use the newer `drill` to do the same? This application is aimed
for you both!

`dog` is **a modern DNS client for the CLI**, written by the developer of `exa`.

![](/img/modern-cli-applications-1/dog-1.png
        "dog output for example.com domain")

It includes some cool features like emitting the output as JSON or printing
the time spent for the request.

![](/img/modern-cli-applications-1/dog-2.png
        "jq pretty printing dog's JSON output")

![](/img/modern-cli-applications-1/dog-3.png
        "dog printing the time spent for the request")

It also let you choose which protocol to use between the following:

- _DNS over UDP (default)_
- _DNS over TCP_
- _DNS-over-TLS_
- _DNS-over-HTTPS_

![](/img/modern-cli-applications-1/dog-4.png
        "dog printing the time spent for the request using DNS over TCP")

## mandown

|
-|-
Repository: | https://github.com/Titor8115/mandown
Written in: | _C_
Repology: | https://repology.org/project/mandown/versions
Dependencies: | _ncurses_, _libxml2_

How many times have you came across a Markdown and had to read it as plain text?
The syntax is easy and looks good _when rendered_. However, using projects like
[livedown] just to read a 100 line Markdown file is overkill.

[livedown]: https://github.com/shime/livedown

`Mandown` is **a Markdown pager for the command line**.

Let's create a simple Markdown file to test mandown with. It will be called
`HelloWorld.md`:

```md
## Hello world!

I am _fairly_ simple **Markdown** file.

- [ ] todo1
- [x] todo2

### Credits

Thanks to everyone reading this post :). ]
```

Now we can use mandown to render it:

```
$ mdn HelloWorld.md
```

![](/img/modern-cli-applications-1/mandown-1.png
        "mandown rendering HelloWorld.md")

As of version 1.0.1 (_16 July 2020_), mandown support for the majority of
HTML tags is in development. It will work best on Markdown only files, for now.

## Conclusions

That's all for this post. Did you already know any of the application
highlighted above? Do you know another alternative or application you want
to see in the next posts? Comment below!

