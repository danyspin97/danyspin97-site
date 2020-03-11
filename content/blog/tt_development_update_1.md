---
title: "tt: Development update #1"
slug: "tt-development-update-1"
date: 2020-03-10T17:52:32+01:00
draft: false
---

Devember as long finished, but not all the projects started during the challenge did the same. This is the case for ***tt***[^3], the init/rc manager that _strives to replace systemd_.

***Notes***: _If you have do not know what _tt_ is, I really invite you to read the previous two posts[^1][^2] on the matter, as this one is a continuation. Nevetherless it is not a requirement for reading this post_.

//for the lazy and for all the other ones that could not read it, the next section is a recap_.

## Starting point

In my last update I have written about using [libconfini] to parse an environment file with the a simple key/value format. This file is read at runtime by services and will be used for configuration.

[libconfini]: https://github.com/madmurphy/libconfini/

Example file:

```
FOO="BAR"
```

This file follows the [INI file] specification and libconfini was suited as a parser.

[INI file]: https://en.wikipedia.org/wiki/INI_file

## Service format

I started developing the next part: the service file parser. To do so, the format needed to be defined first.

What is tt under the hood? Put it simple, it is a wrapper around s6-rc: it takes one or more services files and convert into [s6 service directories]; tt service file should mostly follow s6 directory structure. obarun (66 creator) has defined such a [format][66 frontend] before me, so let’s have a look at it.

[s6 service directories]: https://skarnet.org/software/s6/servicedir.html
[66 frontend]: https://web.obarun.org/software/66/frontend.html

`mount-fstab` file from [66-exherbo]:

[66-exherbo]: https://gitlab.exherbo.org/exherbo-misc/66-exherbo/-/tree/master

```ini
[main]
@type = oneshot
@name = mount-fstab
@description = “Parse and mount all the devices listed in /etc/fstab”
@user = ( root )
@depends = ( fsck mount-rwfs )

[start]
@build = auto
@execute = (
    foreground { s6-echo -- “Starting oneshot: mount-fstab” }
    
    foreground { mount --all }
    
    s6-echo -- “Done oneshot: mount-fstab”
)
```

The file is straigthforward to read if you are used to s6 service directory. There is a `[main]` section that defines basic service information and `[start]` section for this oneshot.

I do not like the keys starting with the hat character (`@`) but overall it is a good start. tt service format has been defined [here][tt service format] and it is somehow similar.

[tt service format]: https://github.com/DanySpin97/tt/issues/2

tt service example:

```
[main]
name = nginx
polish_name = Nginx server
description = "Start and supervise the nginx http server”
type = longrun

[start]
build = auto
execute = (
    ifelse -x { test -d /var/lib/www }
    { exit 1 }
    { nginx --no-daemon }
)

[options]
depends = ( connman iptables )
```


***Notes***: _A lot of changes and new keys are already been either introduced or [proposed][tt provider proposal] to enhance tt format. 66 format has just been the inspiration and the start but tt is_ ***not*** _a mere copy_.

[tt provider proposal]: https://github.com/DanySpin97/tt/issues/9

## Discarding libconfini

This file respect the INI format except for the `execute` key, which contains a multiline string with no escapes character. This is not supported by INI parser such as libconfini.

libconfini supports multiline strings when the newline is escaped. From its [example][libconfini multiline example]:

[libconfini multiline example]: https://github.com/madmurphy/libconfini/blob/master/examples/ini_files/self_explaining.conf#L40

```ini
multi-line	=	this is\
			a multi-line value!
```

The most important parts of a service file are its scripts which could be one or more per file and each spans at least a few lines. Personally, I consider the requirement of an escape character at the end of every line too much tedious. After all I will mantain many services myself and I want it to be as easier as possible, withouth this big drawback. Imagine having the parser tell you "_look, you forgot to add an escape line, now go back and search for it_" every now and then.

This is the reason I had to **discard libconfini as the parsing library**.

I considered other format like TOML, which supports the following:

```toml
multi-line = """
    multiline value
    spanning two lines"""
```

However, I wasn’t satisfied with its syntax. In the end I choosed to write a custom parser.

## Valid values

### Parsing code

The file is divided in sections, and each service type have different sections. Each section contains key/value pairs (with or without the quoted value) and code.

Some strictness is required when parsing code, otherwise we could interpret code as token of the service format. Consider the following code:

```bash
execute=( x=$(cat /etc/fstab)
   echo $x
)
```

The parser could misleadingly treat the last parenthesis as the closing token of the execute key, providing and the next line would be considered an invalid line. If the parser counts parenthesis, then this will work.
However only unquoted parenthesis should be counted, and the one quoted should no be considered. This exponentially increase the parser complexity and opens up to a tons of bugs and edge cases.

If we only consider the following two examples everything become easy and reliable to parse:

```
execute = "/usr/bin/foo -d"

execute = (
    x=$(cat /etc/fstab)
    echo $x
)
```

The first one will be parsed as a one-line key/value pair; the latter will be treated as a multiline code, with no possible cases of the code itself being treated as the format token.

### Multiple values

By writing a custom parser, tt format has also gained the possibility to give keys with multiple values their own rules. Let’s take for example dependencies declaring:

```
depends = ( foo bar )

depends = (
    foo bar )

depends = ( foo
    bar
)
```

Each of the previous examples is valid in tt. This increases the different type of keys to three: _key/value pairs_, _code values_ and ***multiple values spanning in multiple lines***.

Code parser and multiple values parser are **ambigous**, so the section can **allow only one of these two types**. This is intended as its easier to remember the similar sintax instead of using different tokens. These is possibility that this syntax could change in the future if a section must have all three different kinds.

## Engineering the parser

<img src='https://g.gravizo.com/svg?
@startuml;
abstract ServiceDirector;
abstract OptionsBuilder;
interface SectionBuilder;
ServiceParser -- ServiceDirector;
ServiceParser -- ParserFactory;
ParserFactory -- BundleDirector;
ParserFactory -- LongrunDirector;
ParserFactory -- OneshotDirector;
ServiceDirector <-- BundleDirector;
ServiceDirector <-- LongrunDirector;
ServiceDirector <-- OneshotDirector;
ServiceDirector -- SectionBuilder;
SectionBuilder <|-- MainSectionBuilder;
SectionBuilder <|-- EnvironmentBuilder;
SectionBuilder <|-- ScriptBuilder;
ScriptBuilder <|-- LoggerScriptBuilder;
SectionBuilder <|-- OptionsBuilder;
OptionsBuilder <|-- BundleOptionsBuilder;
OptionsBuilder <|-- LongrunOptionsBuilder;
OptionsBuilder <|-- OneshotOptionsBuilder;
@enduml;
'>

The UML diagram above shows the class design that parse a single service file. Implementation details have been left out for the scope of this post; this include the class SystemServicesParser that gets a file location from a service name and the entire `libtt.parser.line`/`libtt.parser.word` modules that are used to parse the file line per line.

The `ServiceParser` load the lines of the file in memory and parses its first section (which must be a `[main]` section) and get its type. Then the `ParserFactory` returns a `ServiceDirector` class based on the type parsed.

The `ServiceDirector` derived classes map each section to the respective `SectionBuilder`. When a new section is found, its builder is instancieted, along with an empty data class or a pointer. For example a Script pointer is passed to ScriptBuilder; the latter receive its lines from the Director and then proceed to parse them. When it has finished, a properly configured and instatiated Script is assigned to the pointer.

At the end of the parsing, the `ServiceDirector` devired classes instantiate their service using the data that builders have filled.

Validation checks are performed in the data itself, except for volatile data; in another words data that is parsed from the service file but do not translate 1:1 into the various tt data classes. To elaborate with an example: the logger script can be manually defined to specify a custom location, but this data isn’t then saved into the `LoggerScript` class. It instead used to generate the execute script and then discarded. This is one of the few cases where the validation check is performed into the builder itself.

This is my first attempt at writing a custom parser and I am satisfied with the current result. This design I have choosen makes it easy to re use components (`EnvironmentBuilder` will be used to parse environment files) and is easy to test.

## Parser CLI

The parser is functional and parse complete services. It has a 75% unittest coverage so many bugs have been already been spotted and many else will be caught in a short time.

For testing purpose I have adding the possibility to try it using tt CLI.
Let’s first build tt. A working installation of ldc2 (LLVM dlang compiler) and mesonbuild are required; they can be installed using the package manager of your distribution.

```bash
$ git clone https://github.com/danyspin97/tt
$ meson build && cd build && ninja
$ ./src/tt/tt parse -f ../examples/mount-fstab
```

### Performance

Working systems tend to have more than 50 services available. It is therefore critical that the parsing do take few time to complete.

Estimating the time needed using the POSIX utility `time` shows us the following:

```bash
$ time ./src/tt/tt parse -q -f ../examples/mount-fstab
```

Considering all the overhead of instantiating a lot of classes, this is quite good. Using different threads to parse a service each, it will avoid an exponentially time increase when many services needs to be parsed.

### Binary Size

tt strive to be minimal, so its library and executable should be really small. At the time of writing the stripped library weighs `1.4M` and the stripped executable `292k`; I am satisfied with these values, because, even if D is a compiled language, it still a high level one.

## Conclusion

Thanks for reading this post and for following tt development. Top priority for tt is not to be running in the least time, but to be solid and well-tested, avoiding the rush and the technical debt that follows.

Special thanks to [@Cogitri](https://github.com/Cogitri) for all its contributions on the CLI and improving libtt code quality, as well for all his insights on the format and issues. Thanks also for the new contributor [@pac85](https://github.com/pac85) which have worked on tt service serialation module (more on this in the next updates!).

tt development also happens in the #tt freenode channel. Feel free to join.

[^1]: https://danyspin97.org/blog/devember-2019-rewriting-66/
[^2]: https://danyspin97.org/blog/devember-2019-first-ten-days/
[^3]: https://github.com/danyspin97/tt

