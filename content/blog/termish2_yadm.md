---
title: "Termish[2] = yadm"
date: 2018-04-10T15:30:18+02:00
categories: ["termish"]
tags: ["dotfiles", "terminal"]
draft: false
---

_Dotfiles_: the files whom dictate pretty much anything on your workstation,
from vi[m] behavior to desktop customization. If you had never heard of
dotfiles or you're not familiar with their concept,
[this][stack overflow dotfiles] is a little explanation.

But what if your laptop and data got lost? Rewrite everything from scratch would be a real hell: indeed they are usually stored on Github or whatever so that restoring or porting to a fresh install requires just 2 clicks.

It seems simple, doesn't it? Well, it is. And simple problems can be resolved in a dozen of different ways. Dotfiles make no difference: people have distinct way to manage configuration files ([this][dotfiles handler] is only a glance) and use from symlinks to rsync and everything in between.

# Yet Another Dotfiles Manager

Yadm is a simple program: there are no symlinks and it doesn't use rsync either. It just clone your repository into your home and let you use git commands to manage it. Yea, you can commit, merge and whatever from yadm.

# How it works

<script src="https://asciinema.org/a/168409.js" id="asciicast-168409" async></script>

At the first call you can either clone the remote repository or init a new one, then you can use all git commands, from commit to rebase.

```bash
yadm init
yadm add <your dotfiles>
yadm commit
```

Adding a remote repository and pushing to it its trivial, so I'll skip it (you can find more in the [getting started] of the official documentation).
Instead, let's focus on specific features of yadm, like encryption and alternate files.

## [Encryption]

Both symmetric and asymmetric encryption can be used, with symmetric as default. It is a pretty simple method, with all files encrypted together, and it's perfect for sensible data like ssh or OpenGPG keys. However, pushing sensible data to a public repository, even if encrypted, is _discouraged_.

## [Alternate files]

Alternate files are a powerful way to have the same dotfiles in different computers that may run different Os or need only a different `.bashrc`. You can use up to 4 different matches (_CLASS_, _OS_, _HOSTNAME_, _USER_) to differentiate your files.

## [Bootstrap]

You are installing dotfiles but you also want to install all vim plugins so you don't have to do it manually? Bootstrap is the perfect tool for the job. It will execute commands right after the dotfiles repositories has been cloned.

# Features

- No symlinks
- Few dependencies
- Alternate files between hosts
- Encryption
- Use git internally

# Installation

[yadm] is available in almost every distro.

# Conclusion

[yadm] is a reliable program that will manage your dotfiles with as little effort as possible, both if you're an advanced user or a novice. Its commands are the same as git, which mean you don't need to learn everything new. Its encrypt, alternate and boostrap features make it flexible and powerful. If you need a tool to manage dotfiles, this is definitely the best.

[stack overflow dotfiles]: https://askubuntu.com/questions/94780/what-are-dot-files
[dotfiles handler]: https://dotfiles.github.com/index.html#general-purpose-dotfile-utilities
[Homemaker]: https://github.com/FooSoft/homemaker
[yadm]: https://thelocehiliosan.github.io/yadm/
[getting started]: https://thelocehiliosan.github.io/yadm/docs/getting_started
[Alternate files]: https://thelocehiliosan.github.io/yadm/docs/alternates
[Encryption]: https://thelocehiliosan.github.io/yadm/docs/encryption
[Bootstrap]: https://thelocehiliosan.github.io/yadm/docs/bootstrap
