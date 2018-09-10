---
title: Getting started with execline scripting
date: 2018-09-04T22:15:00+01:00
categories: ["tutorial", "programming"]
tags: ["s6", "scripting"]
draft: false
---

## Index

- [Preface](#preface)
- [Overview](#overview)
- [Words and whitespaces](#words-and-whitespaces)
- [Execution](#execution)
- [Pipes](#pipes)
- [Script arguments](#script-arguments)
- [Multiple execution](#multiple-execution)
- [Variable management](#variable-management)
- [Conditional execution](#conditional-execution)
- [Iteration](#iteration)
- [`stderr` and `stdout` redirection](#stderr-and-stdout-redirection)
- [Ambivalent block syntax](#ambivalent-block-syntax)

## Preface

In this post, the `execline` scripting language will be outlined: we will learn anything you need to get started.

_**Prerequisites**: basic understanding of shell scripting, and environment; access to unix shell and `execline` installed._

_**Note**: all the scripts have been tested with `execline` version 2.5.0.1._

## Overview

`execline` is scripting language written with security and simplicity in mind.
Its goal is to provide a predictable syntax while remaining portable and easy on resources.

[Why not just use /bin/sh?](http://skarnet.org/software/execline/dieshdiedie.html)

Let's have a look at it.

```
importas home HOME cd $home ls
```

A single chain load as an entire script; it seems really weird, doesn't it?

`execline` relies completely on chain loading: it simply executes the first word and uses the
remaining ones as `argv`. This makes `execline` perfect for critical scripts
like init and rc scripts. It is primary used as scripting language by the [`s6`](http://skarnet.org/software/s6/) supervision suite and the [`s6-rc`](http://skarnet.org/software/s6-rc/) service manager.

## Words and whitespaces

An `execline` script is a string, parsed as words with whitespaces in between.</br> And `execline` parser consider _spaces_, _tabs_, _newlines_ and _carriage returns_ as whitespace.

Understanding that, we can rewrite the previous script in a more friendly way, by adding newlines here and there:

```
importas home HOME

cd $home

ls
```

In each line we find a program and its respective arguments. `ls` is an external program while `cd` and `importas` are two built-ins:

> `cd dir prog...`

> cd changes the current working directory to `dir` and then executes `prog...`

> `importas variable envvar prog...`

> `importas` fetches the value of `envvar` in the environment and import it in the literal `variable`. It then executes `prog...`

So what the script does is, in order:

- get `HOME` value from the environment and assign its value to `home` literal
- change the directory to the `home` value
- call `ls`

_**Note**: `execline` doesn't expand `~` to the home directory, so `cd ~` won't work._

`execline` built-ins take an arbitrary number of arguments and then execute the next program. The words left are the arguments of this new program executed. Therefore we can generalize the syntax as:

```
built-in args prog...
```

where `args` can be any number of arguments.

## Execution

We have our script ready to be executed. But how do we execute it?

There are two ways:

- calling execline directly
- using a shebang

Create a new file named `ls_home` and the following code in it:

```
importas home HOME cd $home ls
```

Now execute the following command in a *nix shell:

```bash
execlineb -P ls_home
```

Yay! We have executed the script by calling `execlineb` directly.
Though this could be a little cumbersome, expecially when we need to add arguments to the `execlineb` call.
Indeed, shebangs are often chosen for simplicity.

Using your editor of choice, open `ls_home` and add the following as first line:

```bash
#/bin/exexlineb -P
```

Now that we've added the shebang, we have to make the file executable; get back to the shell and execute:

```bash
chmod +x ls_home
```

Now execute it:

```
./ls_home
```

This shebang will be used from now and omitted for shortness, unless noted.

## Pipes

Another feature used heavily by shell scripting languages are [pipes](http://www.linfo.org/pipe.html).
This is an example of pipes in `execline`:

```
pipeline { ls }

wc -l
```

`pipeline` is a built-in that execute the chain load inside the block delimited by `{}`
and uses the block output as input for the program at the end of it.
This script display how many folders and files there are in the current directory.
The equivalent in bash would be:

```bash
#!/bin/sh
ls | wc -l
```

We can also use the `-w` flag to reverse the verse of the pipe. This is the pipe above looks like if we use `-w` flag:

```
pipeline -w { wc -l }

ls
```

Consider now three pipes one after another:

```
# Returns all included headers in the current directory
pipeline { grep -R "#include .*"}

# sort the output of grep, uniq needs its input sorted
pipeline { sort }

# Remove all the results that appear twice
pipeline { uniq }

wc -l
```

This script, which we will call `uniq_results`, displays how many unique C/C++ headers are used in the current directory.

## Script arguments

The string passed to grep is hardcoded, it means that the script usefulness is limited. To improve `uniq_results` we will read and use
arguments passed to it.

```
#!/bin/execlineb -S1

# Gives us all the results in the current directory
pipeline { grep -R $1 }

# sort the output of grep, uniq needs its input sorted
pipeline { sort }

# Remove all the results that appear twice
pipeline { uniq }

wc -l
```

`$1` has taken the place of the hardcoded string: grep will search for whatever string we use as the script first argument. As in `bash`, positional arguments are called by using `$` followed by the number of the positional argument wanted.

The last difference between this script and the previous one is in the shebang: the `-P` flag has been replaced by the flag `-S nmin`.
The latter tells `execlineb` the minimum positional parameters that the script needs to run.

Another useful flag is `-s nmin`. It works like `-S`, being `$@` value the only difference:

- with `-S`, it contains all arguments
- with `-s`, it contains all arguments minus the first nmin

There are other powerful and flexible ways to handle positional parameters; `execline` documentation explains [them](http://skarnet.org/software/execline/el_pushenv.html) in details. Have also a look at [`shift`](http://skarnet.org/software/execline/shift.html) and [`dollarat`](http://skarnet.org/software/execline/dollarat.html) programs.

## Multiple execution

We often need to execute external programs in a row, but `execline` doesn't allow the use of `newlines`, that are considered whitespace, nor semicolons. It instead provides two different built-ins: [`foreground`](http://skarnet.org/software/execline/foreground.html) and [`background`](http://skarnet.org/software/execline/background.html).

> `foreground { prog1... } prog2...`

> `foreground` reads `prog1` in a block. It executes it, waits for it to complete and then executes `progs2`

> `background { prog1... } prog2...`

> `background` reads a `prog1...` command in a block, spawns a child executing `prog1...` and then execs into `prog2...`

```
foreground { sleep 10 }

echo "I waited 10 seconds."
```

This is an example where the script wait ten seconds and echo the message we choosed. `foreground` could also be used several times in a row.

## Variable management

> `execline` maintains no state, and thus has no real variables, it provides such a substitution facility via substitution commands

As stated in the `execline` documentation, there no variables. All we have is the substitution facility, which will be used in the next examples.

> `define variable value prog...`

> `define` replaces the literal `variable` with `value`, then executes `prog...`

`define` remind us of an assignment operator, which is what it exactly does.

```
define home /home/foo

mkdir ${home}/scripts
```

_**Note**: when a literal is preceded or followed by other strings, it needs to be wrapped inside curly braces as the example above; otherwise the `execline` parser won't apply substitution._

Another powerful source is the environment, as we have already seen with `importas`. We can use it to store values and retrieve them later on, just as we would do with real variables. `define` assigns a value to a variable, thus this value cannot be calculated at runtime. In this last case, we should use [`backtick`](http://skarnet.org/software/execline/backtick.html) along with `importas`.

> `backtick variable { prog1... } prog2...`

> `backtick` reads `prog1...,` spawns a child executing it and saves its output, then it assigns this output to `variable` literal

Suppose we have to create a file named as the current date, and this file should be store inside the `.cache` folder of the user calling the script:

```
# Get the date and store it inside DATE variable
backtick DATE { date +%d%m%y }

# Import DATE var from environment into date literal
importas date DATE

# Do the same for HOME
importas home HOME

# Create the file
touch ${home}/.cache/used-${date}
```

## Conditional execution

Another thing that we often do is a testing a condition and executing something in case the condition was false, something else if it is true.

`exeline` provides four different built-ins: [`if`](http://skarnet.org/software/execline/if.html), [`ifelse`](http://skarnet.org/software/execline/ifelse.html), [`ifte`](http://skarnet.org/software/execline/ifte.html) and [`ifthenelse`](http://skarnet.org/software/execline/ifthenelse.html). In this tutorial we will only see the first two.

> `if { prog1... } prog2..`

> if reads prog1... in a block. It executes it, wait for it to complete and then:

> - If `prog1` exits a non-zero status, `if` exits 1.
> - Else `if` execs into `prog2`.

This is a simple built-in, it just execute a program, that could also be a chain loads of programs, and then continue with the script execution only if the exit status is zero.

```
importas home HOME

if -n { ls -d ${home}/.cache/used }

mkdir ${home}/.cache/used
```

This is a simple script that check if the directory `~/.cache/used` exists, if it doesn't there is an attempt to create it. We would also need sometimes to check if `mkdir` successfully exit and it can do done by wrapping the last line with another `if`.

We negate the result of the `if` block, hence the `-n` flag:

> `-n` : negate the test (exit on true, exec into `prog2` on false)

Let's improve the scripts above: we create the directory `~/.cache/used` if it does not exist, and then we create a file with the current date in this directory. The script exit with code 1 if a file with the same date has already been created. We will use [`ifelse`](http://skarnet.org/software/execline/ifelse.html):

> `ifelse { prog1... } { prog2... } prog3...`

> - ifelse reads prog1..., it forks and executes it, then waits for it to complete.
> - If `prog1` exits with a return code equal to 0, `ifelse` execs into `prog2`.
> - Else `ifelse` execs into `prog3`.

```
importas home HOME

foreground
{
    if -n { ls -d ${home}/.cache/used }

    mkdir ${home}/.cache/used
}

backtick DATE { date +%d%m%y }

importas date DATE

ifelse -n
# Run this block and check its exit code
{
    ls -d ${home}/.cache/used/${date}
}

# If the previous block exit with non zero code then 
# this block is executed (because of the -n flag in ifelse)
{
    touch ${home}/.cache/used/${date}
}

# Else the script continues here
exit 1
```

We're wrapping the `if` because we need only a little part of the script to depend on that if; we want to execute the rest of the script, which is after the `foreground`, regardless of the result of the `if`.

## Iteration

[`loopwhilex`](http://skarnet.org/software/execline/loopwhilex.html) is the `execline` equivalent of `while`:

> `loopwhilex prog...`

> - `loopwhilex` runs `prog...` as a child process and waits for it to complete. 
> - As long as prog exits zero, loopwhile runs it again. 
> - `loopwhilex` then exits with the last `prog` invocation's exit code.

Since this is pretty straightforward, we will skip an example and we will instead focus on [`forstdin`](http://skarnet.org/software/execline/forstdin.html) and [`forbacktickx`](http://skarnet.org/software/execline/forbacktickx.html), which are the `execline` equivalent of `for` construction.

> `forstdin variable loop...`

> - `forstdin` reads its standard input, splitting it automatically.
> - For every argument `x` in the split output, `forstdin` runs `loop...` as a child process, with `variable=x` added to its environment.

We have to remember that, even if the variable is added to the environment, we still need to import it in a literal, using [`importas`](http://skarnet.org/software/execline/importas.html), this time adding `-u` flag:

> `-u` : Unexport. `envvar` will be removed from the environment after the substitution.

Note also that `forstdin` reads from the standard input, so `pipeline` is needed here.

```
# Print all the files and directory, and use them as
# input for the next program
pipeline { ls }

# For every files and directory printed,
# execute the following block, after replacing FOLDER
# with the name of file/directory
forstdin FOLDER

# Import the environmental variable into the literal folder
importas -u folder FOLDER

# Check if it is a folder, so we skip files
if { ls -d $folder }

# Create the empty file
touch ${folder}/active
```

This script create an empty file named active for each folder in the current directory.

Using `forbacktickx` instead, the script above could have been written like this:

```
forbacktickx FOLDER
{
    ls
}

# Import the environmental variable into the literal folder
importas -u folder FOLDER

# Check if it is a folder, so we skip files
if { ls -d $folder }

# Create the empty file
touch ${folder}/active
```

## stdout and stderr redirection

As you could guess, there is no redirection operator; instead `execline` provides us `fdmove` to redirect file [descriptor](https://en.wikipedia.org/wiki/File_descriptor).

> `fdmove fdto fdfrom prog...`

> `fdmove` moves the file descriptor number `fdfrom`, to number `fdto`, then execs into `prog` with its arguments.
> `-c` : duplicate `fdfrom` to `fdto` instead of moving it; do not close `fdfrom`.

We don't usually need to close the file descriptor, so the `-c` flag is needed.

To redirect `stderr` to `stderr` add the following before `prog`:

```
fdmove -c 2 1
```

To redirect `stdout` to `stderr` invert the file descriptor above:

```
fdmove -c 1 2
```

## Ambivalent block syntax

There are two ways to describe a block; the first is curly braces, and this is the one that we've used in this tutorial.

The second is to delimit only the end of a block with `""`.

```
pipeline ls "" wc -l
```

is equivalent to

```
pipeline { ls } wc -l
```

## Ending

Thank you for reading this getting started tutorial for this interesting scripting language. 

The great and complete [documentation](http://skarnet.org/software/execline/) references other programs provided and a deep explanation of the design and the [grammar](http://skarnet.org/software/execline/grammar.html).

You can find some `execline` scripts in the `s6-rc` [example services](https://github.com/skarnet/s6-rc/tree/master/examples/source).

