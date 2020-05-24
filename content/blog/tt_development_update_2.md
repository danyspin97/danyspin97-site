---
title: "tt#dev2: C++ rewrite"
slug: "tt-dev2-cpp-rewrite"
tags: ['tt', 'cpp']
date: 2020-05-24T16:30:00+02:00
draft: false
---

Second update on [tt] development. You can find the first [here](/blog/tt-development-update-1/).

[tt]: https://github.com/danyspin97/tt

The last months with D programming language has been good. Except for the language compiler. There are three different D compilers out there, which are themselves written in D. How do you compile them without a D compiler?<br>
You use an already **compiled compiler executable** (sorry for the pun) for your _system/architecture_. musl is not supported however, so this created a lot of problems for me.

I gave up to Dlang because I don't feel comfortable asking users to go all that
I went through for bootstrapping a D compiler on a _unsupported system_.
tt has been **converted to C++**. Which took time indeed, but wasn't a painful
process after learning how to apply tt design in C++.

## C++ rewrite

What I am about to write in this post is related to: 

- The **rewrite** that gave me the possibility to improve the low-level parser implementation and change some detail of the service format.
- The new language choosen that opened up some new challenges, implementation
  differences and optimizations possibilities.

## Service format update

Arrays are now enclosed by square brackets (`[]`) to avoid ambiguity between an array and a code value. Array and code can now be found in the same section.

```
[options]
depends = [ myval ]
```

Quotes around values have been removed to simplify the parser implementation:

```
[main]
name = mount-fstab
type = oneshot
```

However the quotes are still required in the `[config]` section to emphasize
that key/value pairs in this section will be written into the environment. This
is important because the environment keys are stricter than usual: only
`[A-ZA-z0-9_]` is supported and the first character cannot be a number.
Another advantage is that here trailing spaces can be spot at a first glance,
as they might be a problem with some service/user configuration.

```
[config]
CMDARGS="--debug"
```

## C++17

C++ is a language that have been heavily changed (and improved) in the latest
standard. C++17 standard adds many useful features in its standard library,
such as `std::variant`, `std::optional` and `std::filesystem`. Since _tt_ is a
new codebase and **C++17 support is considered complete** in the latest releases
of both _GCC_ and _Clang_, it seems a good idea to use these features.

### std::optional

From [cppreference](https://en.cppreference.com/w/cpp/utility/optional):

> The class template std::optional manages an optional contained value, i.e. a value that may or may not be present.

Let's see its usage from `tt/data/script.hpp`:

```cpp
namespace tt {
class Script {
    // Methods
    // ...
private:
    Type type_;
    std::string execute_;
    std::optional<std::string> user_;
    std::optional<std::string> group_;
};
} // namespace tt
```

The class `Script` **may optionally contain** a defined `user` or `group`.
`std::optional` wrap up this feature nicely. We may check and get its value by
doing:

```cpp
if(user_) {
    std::cout << user_->get() << std::endl;
}
```

### std::filesystem::path

From [cppreference](https://en.cppreference.com/w/cpp/filesystem/path):

> Objects of type path represent paths on a filesystem.

This class is used to handle services paths and extensions. From
`src/data/service_impl.cpp`:

```cpp
void tt::ServiceImpl::ValidatePath() const {
    std::filesystem::path file(path());
    if (file.stem() != name()) {
        throw ServiceNameDoNotMatchFileExecption();
    }
}
```

This method check that the name of the service is the same as the name of the
file without extension.

### std::variant

From [cppreference](https://en.cppreference.com/w/cpp/utility/variant):

> The class template std::variant represents a type-safe union.

`std::variant` is used in tt to store either a `Bundle`, a `Longrun` or a
`Oneshot` services, which are all instances of the base service file,
`ServiceImpl`:

```cpp
using Service = std::variant<Bundle, Longrun, Oneshot>;
```

Instead of storing a `std::vector` of pointers to `ServiceImpl`, the
`std::variant` can be stored. This change of the data structures might lead to
**better** [**data
locality**](https://gameprogrammingpatterns.com/data-locality.html)
and therefore **better performance**. This is important especially when we
iterate over this vector multiple time; the `DependencyGraph` class may need to
iterate repeatedly over all the services enabled and is the component that will
most benefit from using `std::variant`.

***Notes***: _The `DependencyGraph` class will be explained in depth in the
next update_.

### std::visit

From [cppreference](https://en.cppreference.com/w/cpp/utility/variant/visit):

> Applies the visitor vis to the variants vars.

The perfect companion for `std::variant` is `std::visit` that let us use **the
visitor patter over this container**.

Two examples of its usage in the tt codebase. From
`src/dependency_graph/dependency_graph.cpp`:

```cpp
    auto get_name = [](auto &service) { return service.name(); };
    const auto name = std::visit(get_name, service);
```

Here we use a closure to get the name of the service. `ServiceImpl` has
a public `string name();` method so all the three classes contained in the
`std::variant` have it.

From `include/tt/parser/service/dependency_reader`:

```cpp
class DependencyReader {
public:
    std::vector<std::string> dependencies() { return dependencies_; }

    void operator()(const Bundle &bundle);
    void operator()(const Longrun &longrun);
    void operator()(const Oneshot &oneshot);

private:
    std::vector<std::string> dependencies_;
};
```

From `src/parser/service/services_parser.cpp`:

```cpp
    tt::DependencyReader dep_reader;
    std::visit(dep_reader, service);
```

In this example, the object `dep_reader` is used to visit the `std::variant`.
An object is used because it will be used later to access the dependencies read
from the service.

## External libraries

The **C++ standard library has different features compared to the Dlang one**.
Specifically, these are the features that I have lost from the switch:

- Good command line arguments parser
- JSON serialization
- Logging
- String formattation
- Unit testing

None of the above functionalities are trivial to implement, so I have to resort
on either **include libraries into tt codebase** or **link to external
libraries**.

### Taywe/args
For parsing command line arguments I have chosen [Taywe/args]() header-only
library. I need a parser that is able to declare subcommands and assign
specific options to them. Most cli arguments parser that I have tried to declare a static object and let the developer reuse that in different classes/sources.
_Taywe/args_ lets you instead **configure the subcommand called by the user at runtime**, which is really similar to what was done in Dlang.

From file `main.cpp` (simplified code):
```cpp
int main(int argc, char *argv[]) {
    args::ArgumentParser parser("tt init/rc manager.");
    args::Command tt_parse(parser, "parse",
                        "Parse one or more services for testing purposes.",
                            &tt::ParseCommand::Dispatch;
                        );
    }
    
    try {
        parser.ParseCLI(argc, argv);
    } catch (args::Help &e) {
        std::cout << parser;
    } catch (args::Error &e) {
        cerr << e.what() << endl << parser;
        return 1;
    }
    return 0;
```

`tt::ParseCommand::Dispatch` will be called when `parse` subcommand has been
requested by the user. This method will take a `args::Subparser&` argument
that will configure all options specific to the subcommand and reparse all
the cli argumnts left.

### Catch2

[Catch2](https://github.com/catchorg/Catch2) is a powerful unit-tests framework
that has a wide adoption among C++ Open Source projects. The only disadvantage
is that the testing isn't internal to the class but it is instead declared in
a separate source file. This is not strictly related to this awesome library
but to the language itself. For this reason, all the unit tests on private
methods have been dropped.

From `test/parsr/lin/section_line_parser.cpp`:
```cpp
TEST_CASE("SectionLineParser") {
    SECTION("parse valid section") {
        auto parser = tt::SectionLineParser("[foo]");
        REQUIRE(parser.IsLineValid());
    }

    SECTION("parse invalid section") {
        auto parser = tt::SectionLineParser("foo");
        REQUIRE(!parser.IsLineValid());
    }
```

## Other libraries

The above libraries have been already integrated into the codebase. For the
other features missing I have been considered other libraries such as
[fmt](https://github.com/fmtlib/fmt) and
[spdlog](https://github.com/gabime/spdlog). However, they will be introduced
in next updates as they have been integrated and used properly.

## Static analisys and tools

C++ is a widely adopted language, so there are many tools to ease its
development. The rewrite ensured a better meson support (yay!) and the
possibility to run static analisys tools.

Among the static analisys tools used locally there are
[_clang-tidy_](https://clang.llvm.org/extra/clang-tidy/) and
[_oclint_](https://github.com/oclint/oclint). They showed a lot of improvements
for the codebase and some logic error too. _oclint_ also check **path complexity**,
**method complexity** and **file complexity**, warning you about things that
should be refactored. In addition, some static analizer run remotely thanks to
[LGTM](https://lgtm.com/), [CodeFactory](https://www.codefactor.io/) and
[Codacy](https://www.codacy.com/).

However, **coverage is not measured any more** due to various errors on _CodeCov_.
I have also considered [Coveralls](https://coveralls.io/), but generating the
coverage from meson isn't straightforward either. I am planning to readd this
in the future.

C++ suffers from
[Undefined Behaviour](https://en.cppreference.com/w/cpp/language/ub),
especially when the program is compiled with (aggressive) optimizations enabled.
_tt_ should be able to run even when compiled with aggressive optimizations, so
I am closely following the static analizers' warnings and I have enabled the
_undefined behaviour sanitizer_ when running the test in the _CI_ (thanks meson
for being awesome).

