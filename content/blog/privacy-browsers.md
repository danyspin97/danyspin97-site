---
title: "Web Browsers and Privacy in 2019"
date: 2019-01-24T21:50:07+01:00
draft: false
---

An important piece in everyday life is the browser, considering that the new trend is having every data and application accessible in the cloud.

Among the browsers landscape, the choice is actually finite because develop and maintain one is expensive. But, which should you choose? One that offer speed or one that is secure? Does it respect your privacy?

In the current post, we will showcase browsers with a user-friendly GUI which both novice and expert can use. Firefox and Google Chrome will be highlighted as well as their forks and brand new browser; attenction is paid to the consumer privacy.

***Prerequisistes***: _None. Both beginners and advanced users can enjoy this post; it tries to explain every technology that is needed to discuss the various browsers_.

![](https://upload.wikimedia.org/wikipedia/commons/e/e1/StatCounter-browser-ww-monthly-201707-201707-map.png)
*Browser Market Share Worldwide 2017*

## Google Chrome [^1]


Google Chrome has been the most used browsers in the last year with a wide gap from the other alternatives. Why? Because rendering a page is really fast and it is integrated with Google services.

But it’s a Google application, you can’t expect that it will respect your privacy; settings don’t help either. And it isn’t a libre browser.

### Chromium [^2]

The rendering engine (the part that read the HTML page and print out the formatted output) of Google Chrome is called Chromium and is distributed as a separate software. It’s basically a Google Chrome minus _some_ extensions and plugins. Privacy is a little better but it’s still _Googlish_.

#### Iridium [^32]

Here we are at the first privacy-aware browser: Iridium is a fork of Chromium, private and secure by default. The project has been released recently, but it seems already abandoned: the latest commit to its codebase is dated in November 2018. No fixes or security update since then.

#### ungoogled-chromium [^3]

_ungoogled-chromium_, is a fork of chromium with privacy improvements where _every_ Google trace has been removed. It is a drop-in replacement so you can just download the browser and start using the same interface you’re used to.

Download![^4]

## Mozilla Firefox [^5]

Another pillar of privacy-aware browsers is Mozilla Firefox. It is fast (especially with the new Quantum engine[^8]) and easy to use. The only problem is that Mozilla often makes opinionated choices [^6][^7]. But Firefox it’s open source and it would be a huge waste to waste its code; this is why we will overview some of its forks.

It’s important to state that Firefox isn’t privacy-aware by default, but the opt-out and relevant settings are just few clicks away, and all the sneaky extensions enabled by default (like Mr. Robot extensions[^7]) can be disabled.

If you still want to use Firefox, follow PrivacyToolsIO guide[^15].

Download![^5]

### Basilisk [^12]

![](https://www.basilisk-browser.org/bas-logo-300w.png)

Basilisk is a Firefox fork made right before the Quantum update [^8] to demonstrate the Unified XUL Platform[^13][^20]. But what is XUL[^19][^14] and why should you care about?

> XUL (XML User Interface Language) is Mozilla's XML-based language for building user interfaces of applications like Firefox.

It is a subset of XML made for creating add ons that change Firefox user interface; Mozilla deprecated XUL-based add ons at the end of the 2017[^18]. Basilisk developers still believe in this technology and want to improve it.

It uses a fork of the Firefox rendering engine

The project seems interesting, but after a bit of digging, I’ve found this issue[^21] in the OpenBSD work in progress ports tree. It has been opened by Basilisk developers (they also develop Pale Moon[^22]) to make the port comply  with the licence, and the terms used are not polite at all. Package and ports maintainers are often volunteers so there is no point in making such an approach for whatever reason. Because of this, Basilisk developers doens’t seem worthy of trust, I suggest to use some more trustworthy project instead.

***Note***: _A OpenBSD port is a set of files (_Makefiles_) that describes where to fetch the source of a software and how to compile/install it. The pre-compiled binaries offered by OpenBSD are built using the ports_.

### GNU Icecat [^9]

![](https://www.gnu.org/software/gnuzilla/icecat-128.png)

Icecat browser for people who really care about their privacy but also cares about code freedom. It is based on Firefox ESR (Extended Support Release) and consists in a set of patches that remove all non-free code and enhance privacy.

Some security extensions like LibreJS[^10] and Searxes’ Third-party Request Blocker[^11] are shipped with Icecat but they can be disabled with a click in the homepage, restoring the usual surfing experience (these two extensions break many sites).

In a fancy GNU style, Icecat allows a limited set of extensions, the free ones; indeed Stallman wouldn’t approve non-free extensions!

Download version _60.3.0_![^16] \(File: icecat-60.3.0.en-US.gnulinux-x86_64.tar.bz2)

### Librefox [^24]

![](https://raw.githubusercontent.com/intika/Librefox/master/capture.png)

As strange as it sounds, Librefox is not a fork. It instead consists in a bunch of configuration files applied to vanilla Firefox. Privacy extensions and sane settings are shipped with it, uneeded features are removed (like crash reporter and updater).

The first version of Librefox has been relased in November 2018, so it’s a pretty new and interesting project. As of January 2019, Librefox distributes the configurations files that need to be applied manually to vanilla Firefox, by unpacking them in the installation directory. Developers stated that a complete build is planned[^25].

Download![^26]

### Pale Moon [^23]

Pale Moon is a fork of Firefox dated back in the 2009. It supports XUL and offers the old Firefox UI. The rendering engine is the same of Basilisk so they are almost the same in terms of speed.

For reason stated above about its developers, I suggest using other more trustworthy alternatives.

### Waterfox [^27]

Waterfox’s philosophy is close to Librefox, but dates back in the 2011; it has a dedicated website with all the relative informations and the full build download.

It allows every plugin (unlike Icecat) but its version lags behind: at the time of writing the last release is 56.2.6, while Firefox latest stable release is 64.0.2.

Download![^28]

## GNOME Web [^29]

![](https://wiki.gnome.org/Apps/Web?action=AttachFile&do=get&target=epiphany-3-24.jpg)

GNOME Web, formerly known as Ephiphany, is the browser of choice for GNOME desktop and Elementary OS. It features a clean interface (which pair perfectly with the above desktops) and uses Webkit[^30] (Apple open-source browser engine). The latter is also its biggest flaw: Webkit it’s slower than its competitors (both Firefox browser engine and Chromium).

Another worth mentioning feature is the desktop integration with GNOME:

![](https://wiki.gnome.org/Apps/Web?action=AttachFile&do=get&target=epiphany-web-apps.jpg)

Install it from your distro repository!

## Falkon [^31]

![](https://www.falkon.org/images/screenshot.png)

Falkon, which changed name in the latest release, is the former Qupzilla, a Qt based browser made by KDE; it uses Chromium engine.

Download! [^32]

## Konqueror [^35]

![](https://docs.kde.org/trunk5/en/applications/konqueror/konqorg.png)

Another browser which uses Qt GUI library and that’s developed by the KDE team. Its peculiarity is its support for both Chromium engine and KHtml[^36] engine. Right-click and then choose “Render with KHtml” or “Render with WebEngine”, this is all it takes to switch between the rendering engines.

The KHtml browser engine was choosed by Apple as starting point for its WebKit, but it’s currently unusable because of inability to render Html5 properly.

Install it from your distro repository!

## Midori [^37]

![](https://upload.wikimedia.org/wikipedia/commons/9/95/Midori_Screenshot.png)

Midori is a lightweight browser, originally developed for the XFCE desktop (indeed its UI reminds this desktop); it uses WebKit.

## Notes about add ons

Firefox has recently deprecated support for its extensions API and then switched to an API conforming to Chrome extensions; as a result, Chrome and Firefox (including their forks) share the extension format. This is not true for all the other browsers listed in this post, which any of them support its own format.

### Bonus: Privacy-aware addons [^17]

[^1]: https://www.google.com/chrome/
[^2]: https://www.chromium.org/
[^3]: https://github.com/Eloston/ungoogled-chromium
[^4]: https://github.com/Eloston/ungoogled-chromium#downloads
[^5]: https://www.mozilla.org/en-US/firefox/new/
[^6]: https://www.reddit.com/r/privacytoolsIO/comments/abfgj5/mozilla_responds_to_bookingcom_snippet_concerns/
[^7]: https://www.ghacks.net/2017/12/16/firefox-looking-glass-extension-what-it-is/
[^8]: https://blog.mozilla.org/blog/2017/11/14/introducing-firefox-quantum/
[^9]: https://www.gnu.org/software/gnuzilla/
[^10]: https://www.gnu.org/software/librejs/
[^11]: https://searxes.danwin1210.me/
[^12]: https://www.basilisk-browser.org/
[^13]: https://github.com/MoonchildProductions/UXP
[^14]: https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XUL
[^15]: https://www.privacytools.io/#about_config
[^16]: http://ftp.gnu.org/gnu/gnuzilla/60.3.0/
[^17]: https://www.privacytools.io/#addons
[^18]: https://tech.slashdot.org/story/17/02/17/1635216/mozilla-will-deprecate-xul-add-ons-before-the-end-of-2017
[^19]: https://en.wikipedia.org/wiki/XUL
[^20]: http://thereisonlyxul.org/
[^21]: https://github.com/jasperla/openbsd-wip/issues/86
[^22]: http://danyspin97.org/blog/web-browsers-and-privacy-in-2019/#pale-moon
[^23]: https://www.palemoon.org/
[^24]: https://librefox.org/
[^25]: https://github.com/intika/Librefox/issues/55
[^26]: https://github.com/intika/Librefox/releases
[^27]: https://www.waterfoxproject.org/
[^28]: https://www.waterfoxproject.org/en-US/waterfox/new/?scene=1
[^29]: https://wiki.gnome.org/Apps/Web/
[^30]: https://webkit.org/
[^31]: https://www.falkon.org
[^32]: https://www.falkon.org/download/
[^33]: https://iridiumbrowser.de/
[^34]: https://iridiumbrowser.de/downloads/source
[^35]: https://konqueror.org/features/browser.php
[^36]: https://api.kde.org/frameworks/khtml/html/index.html
[^37]: https://www.midori-browser.org/
