---
title: "Devember 2019: First ten days"
date: 2019-12-12T23:00:00+01:00
categories: ["development"]
tags: ["devember", "dlang"]
---

After 10 long days, I’m ready to share my new journey with Dlang.

## Build system

D documentation was really interesting and the language seemed promising.
But I neened to start somewhere and I’ve chosen to start by creating the
boilerplate files and building them; after all I want to create a **shared
library** (_libtt_) and an **executable linked to it** (_tt_). This is straightforward in C/C++ but I have no idea about D.

Looking at the build systems with D support, there are **dub**[^6], dlang package manager shipped
alongside the compiler, and **meson**[^2], which has seen a huge increase in adoption over the last year, despite not having reached its 1.0 release.

I have chosen meson, because I am already familiar and support my use case
really well. The problem was that D support for meson is really lacking and the
language itself has rarely used at a system level. I had to
discover and use some _“hacks”_ to properly **build** and obtain a **working executable**.
And this took most of my time during these first days.

I will work more during the next weeks to improve both meson and the D compilers,
but it will require a lot of time.

## Parsing 

Back at the fun part of this Devember; one of the first thing to develop is
the **environment file** and **service parser**.

For the environment file, that allows the service to be _configurable_, and
the service files themself, I need a parsing library. After some search, I’ve come
across on a new one called **libconfini**[^7]. It is really **minimal** and **well
documented**. Now I just have to call its method from D.

_“Interfacing with C”_[^12] documentation refers to the deimos project:
d sources generated from C headers, which **allow D programs to use C libraries**.
Creating a wrapper is straightforward thanks to **dstep**[^13], and the result works
out of the box flawlessy. You can see d sources to use libconfini in
**libconfini-d**[^14].

**Note**: _There is another way to use C libraries from D which is dpp[^10], a wrapper over the dmd compiler._


An example environment file is:

```
VERBOSITY=1
STAGE2=/etc/tt/stage2
```

libconfini parsing works by defining a `IniFormat`, describing what is allowed and
what not, the characters used for delimiting keys from values, and other
configuration flags. Here I have created a function to get the `IniFormat`:

```d
import confini;

IniFormat get_env_format() {
    IniFormat env_format;

    env_format.delimiter_symbol = IniDelimiters.INI_EQUALS;
    env_format.case_sensitive = 1;
    env_format.semicolon_marker = IniCommentMarker.INI_IGNORE;
    env_format.hash_marker = IniCommentMarker.INI_IGNORE;
    env_format.multiline_nodes = IniMultiline.INI_NO_MULTILINE;
    env_format.section_paths = IniSectionPaths.INI_NO_SECTIONS;
    env_format.no_single_quotes = 1;
    env_format.no_double_quotes = 0;
    env_format.no_spaces_in_names = 1;
    env_format.implicit_is_not_empty = 1;

    return env_format;
}
```

**Note**: _To discover more about the IniFormat and libconfini, browse its online docs[^15] or its man pages._

To parse a file, the function `load_ini_path` is used:

```c
  int load_ini_path (
               const char * path,
               IniFormat format,
               IniStatsHandler f_init,
               IniDispHandler f_foreach,
               void * user_data
           )
```

where `path` is the path of the file, `format` contains the parsing rules,
`f_foreach` is a callback function that actually process the parsed key/value
pairs and `user_data` is some data that is kept over the parsing.
`f_init` won’t be used so let’s skip it for now.

The problem here is that we have to pass as `f_foreach` **a D function as a C callback**, so we have to match the signature (in
D) `extern (C) const int function(IniDispatch *, void*)`.

I’ve tried using a closure and adding the tag `extern (C)`, but despite my
efforts, it doesn’t work. An idea appears in my mind: what if I **add an alias to
this signature**?

```
extern (C) alias IniCallback =
    const int function(IniDispatch* dispatch, void* v_null);
```

It acually compiles. Now let’s add a function that return this exact signature:

```
extern (C) IniCallback get_env_parse_callback() {
    return (IniDispatch* dispatch, void* env_p) {
        return 0;
    };
}
```

And then call `load_ini_path` from our testing main:

```
import std.stdio : writeln ;
import std.string : toStringz ;

int main() {
    if (load_ini_path(
                toStringz("/path/to/my/env"),
                get_env_format(),
                null,
                get_env_parse_callback(),
                null)) {
        writeln("Something went wrong");
        return 1;
    }

   return 0;
}
```

And it works! We have been able to use the parsing library libconfini from
D code. And withouth too much of works, the result is pritty much readable.

**Note**: _We use the method `char* toStringz(string)` to convert a D string
into a C string._

This parsed line that we have in `f_foreach` should be processed and saved
somewhere so that we can later load its values into a service environment.
The suited data structure is an **associative array** which is also really simple to
use in D:

```
string[string] myass;
```

Thouth this array can’t be directly passed to `void* user_data` (I’ve done way many tests, and the cast from `void*` to `string[string]` always fail). So the
easiest work around is to to create a struct containing the associative array:

```
struct EnvWrapper {
    string[string] hash;
}
```

Now our `main` become like this:

```
int main() {
    EnvWrapper env;

    if (load_ini_path(
                toStringz(path),
                get_env_format(),
                null,
                get_env_parse_callback(),
                &env)) {
        writeln("Something went wrong");
        return 1;
    }
    
    return 0;
}
```

And our callback:

```
extern (C) IniCallback get_env_parse_callback() {
    return (IniDispatch* dispatch, void* env_p) {
        ini_unquote(dispatch.data, dispatch.format);
        ini_string_parse(dispatch.value, dispatch.format);
        auto env = cast(EnvWrapper*)env_p;
        auto key = to!string(dispatch.data);
        auto val = to!string(dispatch.value);
        env.hash[key] = val;
        return 0;
    };
}
```

The functions `ini_unquote` and `ini_string_parse` are used to sanitize (i.e. 
removing escaping chars) and removing the quotes according to the format.

This time the conversion is the other way, from C strings to D strings using
`to!string`.

### Conclusion

I have much fun writing this piece of code, way less fun writing the
files for the build system. I hope that this can be improved because it’s really
a shame to have such unexpressed potential.

On the next days I’ll write the good for the service parser too, and actually
experiment with the **service dependencies checker**, which I think will be one of
the critical parts.

_Stay tuned!i_

[^6]: https://github.com/dlang/dub
[^2]: https://mesonbuild.com
[^7]: https://github.com/madmurphy/libconfini/
[^12]: https://dlang.org/spec/interfaceToC.html
[^13]: https://github.com/jacob-carlborg/dstep
[^14]: https://github.com/danyspin97/libconfini-d
[^15]: https://madmurphy.github.io/libconfini/html/index.html
