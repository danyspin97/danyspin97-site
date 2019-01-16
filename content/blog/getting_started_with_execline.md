---
title: Getting started with execline scripting
date: 2018-09-04T22:15:00+01:00
categories: ["tutorial", "programming"]
tags: ["s6", "scripting"]
draft: false
---

## Preface

In this post, the `execline` scripting language will be outlined: we will learn anything you need to get started.

_**Prerequisites**: basic understanding of shell scripting, and environment; access to unix shell and `execline` installed._

_**Note**: all the scripts have been tested with `execline` version 2.5.0.1._

## Overview

`execline` is scripting language written with security and simplicity in mind.
Its goal is to provide a predictable syntax while remaining portable and easy on resources.

Why not just use /bin/sh?[^1]

Let's have a look at it.

```
importas home HOME cd $home ls
```

A single chain load as an entire script; it seems really weird, doesn't it?

`execline` relies completely on chain loading: it simply executes the first word and uses the
remaining ones as `argv`. This makes `execline` perfect for critical scripts
like init and rc scripts. It is primary used as scripting language by the `s6`[^2] supervision suite and the `s6-rc`[^3] service manager.

## Words and whitespaces

An `execline` script is a string, parsed as words with whitespaces in between.</br> And `execline` parser consider _spaces_, _tabs_, _newlines_ and _carriage returns_ as whitespace.

Understanding that, we can rewrite the previous script in a more friendly way, by adding newlines here and there:

```
importas home HOME

cd $home

ls
```

In each line we find a program and its respective arguments. `ls` is an external program while `cd` and `importas` are two builtins. The difference will be discussed in details later.

> `cd dir prog...`

> cd changes the current working directory to `dir` and then executes `prog...`

> `importas variable envvar prog...`

> `importas` fetches the value of `envvar` in the environment and import it in the literal `variable`. It then executes `prog...`

So what the script does is, in order:

- get `HOME` value from the environment and assign its value to `home` literal
- change the directory to the `home` value
- call `ls`

_**Note**: `execline` doesn't expand `~` to the home directory, so `cd ~` won't work._

`execline` builtins take an arbitrary number of arguments and then execute the next program. The words left are the arguments of this new program executed. Therefore we can generalize the syntax as:

```
builtin args prog...
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
execlineb ls_home
```

Yay! We have executed the script by calling `execlineb` directly.
Though this could be a little cumbersome, expecially when we need to add arguments to the `execlineb` call.
Indeed, shebangs are often chosen for simplicity.

Using your editor of choice, open `ls_home` and add the following as first line:

```bash
#!/bin/exexlineb
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

## Builtin and external commands

When we call a command in the shell, it usually searches in `$PATH` for a matching one, executing the first that it finds.

But for some command, usually the most used, the shell usually provide its own replacement; for example bash provides `type`, `cd` and `test` (and many others). These commands are called _shell builtins_. It means that if we call `test`, bash `test` implementation will be executed instead of `/usr/bin/test`. `type` cannot be called outside of bash because is a builtin and it doesn't have a respective `/usr/bin/type`.

_**Note**: in bash, you can know if a command is a builtin or not by using `type` followed by the command._

So for the bash shell, we can say that everything we call is either a _builtin_ or a program in `$PATH`. The latter are called external commands.

`execlineb` does not provide any builtin; instead, it provides its utilities (such as `importas` and `cd`) in `$PATH`. We will call everything that is provided by `execline` as builtin, and everything else an external command.

## Blocks

`execlineb` treats the arguments as one big `argv`, but there are some cases where we need to extract two command lines from it. Blocks have this purpose.

From the documentation[^4]:

> execline commands that need more than one linear set of arguments use blocks.

> In `execlineb` scripts, blocks are delimited by braces. They can be nested. 

## Redirection

As you could guess, there is no redirection operator; instead `execline` provides `fdmove`[^5] to move file descriptors[^6] and `redirfd`[^7] to redirect a file descriptor to a file.

Integer value | Name | <stdio.h> file stream
--- | --- | ---
0 | Standard input | stdin
1 | Standard output | stdout
2 | Standard error | stderr

> `fdmove fdto fdfrom prog...`

> `fdmove` moves the file descriptor number `fdfrom`, to number `fdto`, then execs into `prog` with its arguments.
> `-c` : duplicate `fdfrom` to `fdto` instead of moving it; do not close `fdfrom`.

We don't usually need to close the file descriptor, so the `-c` flag is needed.

To redirect `stderr` to `stdout` add the following before `prog...`:

```
fdmove -c 2 1
```

To redirect `stdout` to `stderr` invert the file descriptors above:

```
fdmove -c 1 2
```

> `redirfd` redirects the file descriptor number `fd` to file, then execs into `prog...`. 

> Options
> : -r : open file for reading.
> : -w : open file for writing, truncating it if it already exists.
> : -a : open file for appending, creating it if it doesn't exist. 

Shell equivalent
: `redirfd -r n file prog...` is equivalent to `prog... n< file`
: `redirfd -w n file prog...` is equivalent to `prog... n> file`
: `redirfd -a n file prog...` is equivalent to `prog... n>> file`

To append the current date in a file in shell syntax, we would have done:

```bash
date >> date_saved
```

The equivalent in `execlineb` scripting is:

```bash
redirfd -a 1 date_saved

date
```

`redirfd` can also be used to redirect a file descriptor to `/dev/null`, discarding unwanted output:

```bash
redirfd -w 1 /dev/null prog...
```

## Pipes

Another feature used heavily by shell scripting languages are pipes[^8].
This is an example of pipes in `execline`:

```
pipeline { ls }

wc -l
```

`pipeline` is builtin that take two linear argument, therefore blocks are used. It executes the chain load inside the block
and save its output. Then the second command line argument, that is `wc -l` here, is executed and the saved output is used as input.

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
pipeline { grep -R "#include <.*>" }

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

The last difference between this script and the previous one is in the shebang, where the flag `-S nmin` has been added.
The latter tells `execlineb` the minimum positional parameters that the script needs to run.

Another useful flag is `-s nmin`. It works like `-S`, being `$@` value the only difference:

- with `-S`, it contains all arguments
- with `-s`, it contains all arguments minus the first nmin

There are other powerful and flexible ways to handle positional parameters; `execline` documentation explains them[^9] in details. Have also a look at `shift`[^10] and `dollarat`[^11] programs.

## Multiple execution

We often need to execute external programs in a row, but `execline` doesn't allow the use of `newlines`, that are considered whitespace, nor semicolons. It instead provides two different commands: `foreground`[^12] and `background`[^13].

> `foreground { prog1... } prog2...`

> `foreground` reads `prog1` in a block. It executes it, waits for it to complete and then executes `progs2`

> `background { prog1... } prog2...`

> `background` reads a `prog1...` command in a block, spawns a child executing `prog1...` and then execs into `prog2...`

Relating back to the `sh` syntax:

- `foreground { a } b` is the equivalent to `a ; b`

- `background { a } b` is the equivalent of `a & v`

```
foreground { sleep 10 }

echo "I waited 10 seconds."
```

This is an example where the script wait ten seconds and echo the message we choosed. `foreground` can also be used several times in a row.

## Variable management

> `execline` maintains no state, and thus has no real variables

While somewhat of a misnomer, execline emulates variable management in other scripting languages via its substitution mechanism, which we will use in the next examples.

> `define variable value prog...`

> `define` replaces the literal `variable` with `value`, then executes `prog...`

`define` remind us of an assignment operator, which is what it exactly does.

```
define home /home/foo

mkdir ${home}/scripts
```

### Variable Substitution

When a literal is followed by other strings, it needs to be wrapped inside curly braces; otherwise the `execline` parser won't apply substitution.

The substitution will be applied here:

```bash
define FOO blah
echo $FOO
```

but won't be applied here:

```bash
define FOO blah
echo $FOO-$FOO
```

because the first literal must be wrapped inside the braces:

```bash
define FOO blah
echo ${FOO}-$FOO
```

_**Note**: in `$FOO$FOO` both variables will be substituted._

#### Quoting

> the following is the most complex part of the whole language

_**Note**: Feel free to skip this paragraph if you don't intend to use the quoting mechanism._

There are two possible cases: the variable is either substituted or not substituted. Both will be highlighted here, in order.

> if `${FOO}` is preceded by `2*n` backslashes (an even number), the whole sequence will be replaced with `n` backslashes, followed by the substituted value.

As an example, consider the following script:

```bash
define FOO blah
echo \\${FOO}
```

> If `${FOO}` is preceded by `2*n+1` backslashes (an odd number), the whole sequence will be replaced with `n` backslashes, followed by the literal `${FOO}`.

Consider again the script above, this time with an odd number of backlashes:

```bash
define FOO blah
echo \${FOO}
```

`${FOO}` will not be substituted and its output will be `${FOO}` literal. If we used three backlash (`2*n + 1` with `n=1`), the output would be `\${FOO}`, that is `n` backslashes and the literal.

`${FOO}` will be substituted with its value, and the `2*n` (where `n=1` in the example) backlashes will be substituted with `n` backlashes.

### Environmental variables

Since execline takes heavy advantage of chainloading, we can use the environment as a true variable store, as we have already seen with `importas`.

_**Note**: environment variables are real variables, they are just external to the substitution mechanism until brought in via `importas` (or an execline utility that does automatic substitution)._

We used `define` to emulate the assignment operator, assigning a value to a literal; thus this value cannot be calculated at runtime. If we need to assign the output of a command to a variable, we should use the environment instead.

`backtick`[^14] is the builtin used to set an envorimental variable from another command output:

> `backtick variable { prog1... } prog2...`

> `backtick` reads `prog1...,` spawns a child executing it and saves its output, then it assigns this output to `variable` in the environment.

Suppose we have to create a file named as the current date, and this file should be store inside the `.cache` folder of the user calling the script:

```
# Get the date and store it inside DATE variable
backtick DATE { date +%d%m%y }

# Import DATE var from environment into date literal
importas date DATE

# Do the same for HOME
importas home HOME

# Create the file
touch ${home}/.cache/used-$date
```

## Conditional execution

Another thing that we often do is a testing a condition and doing different things depending on its result (or exit code in shell scripting). In another words we execute something in case the condition was false, something else if it was true.

`exeline` provides four different builtins: `if`[^15], `ifelse`[^16], `ifte`[^17] and `ifthenelse`[^18]. In this tutorial we will only see the first two.

> `if { prog1... } prog2..`

> if reads prog1... in a block. It executes it, wait for it to complete and then:

> - If `prog1` exits a non-zero status, `if` exits 1.
> - Else `if` execs into `prog2`.

This is a simple builtin, it just execute a program, that could also be a chain loads of programs, and then continue with the script execution only if the exit status is zero.

_**Note**: to test that a file exists the POSIX utility `test`[^19] will be used in the following examples._

```
importas home HOME

if -n { test -d ${home}/.cache/used }

mkdir ${home}/.cache/used
```

This is a simple script that check if the directory `~/.cache/used` exists, if it doesn't there is an attempt to create it. We would also need sometimes to check if `mkdir` successfully exit and it can do done by wrapping the last line with another `if`.

We negate the result of the `if` block, hence the `-n` flag:

> `-n` : negate the test (exit on true, exec into `prog2` on false)

By wrapping the `if` with a `foreground` call, we can execute the rest of the script, regardless of the result of the first block executed by the`if`:

```bash
importas home HOME

foreground {
    if -n { test -d ${home}/.cache/used }

    mkdir ${home}/.cache/used
}

# This line will be executed regardless of the exit status in the if above
echo The script finished the execution
```

Let's improve the scripts above: we create the directory `~/.cache/used` if it does not exist, and then we create a file with the current date in this directory. The script exit with code 1 if a file with the same date has already been created. We will use `ifelse`[^20]:

> `ifelse { prog1... } { prog2... } prog3...`

> - ifelse reads prog1..., it forks and executes it, then waits for it to complete.
> - If `prog1` exits with a return code equal to 0, `ifelse` execs into `prog2`.
> - Else `ifelse` execs into `prog3`.

```
importas home HOME

foreground
{
    if -n { test -d ${home}/.cache/used }

    mkdir ${home}/.cache/used
}

backtick DATE { date +%d%m%y }

importas date DATE

ifelse -n
# Run this block and check its exit code
{
    test -d ${home}/.cache/used/$date
}

# If the previous block exit with non zero code then 
# this block is executed (because of the -n flag in ifelse)
{
    touch ${home}/.cache/used/$date
}

# Else the script continues here
exit 1
```

## Iteration

`loopwhilex`[^21] is the `execline` equivalent of `while`:

> `loopwhilex prog...`

> - `loopwhilex` runs `prog...` as a child process and waits for it to complete. 
> - As long as prog exits zero, loopwhile runs it again. 
> - `loopwhilex` then exits with the last `prog` invocation's exit code.

Since this is pretty straightforward, we will skip an example and we will instead focus on `forstdin`[^22] and `forbacktickx`[^23], which are the `execline` equivalent of `for` construction.

> `forstdin variable loop...`

> - `forstdin` reads its standard input, splitting it automatically.
> - For every argument `x` in the split output, `forstdin` runs `loop...` as a child process, with `variable=x` added to its environment.

We have to remember that, even if the variable is added to the environment, we still need to import it in a literal, using `importas`[^24], this time adding `-u` flag:

> `-u` : Unexport. `envvar` will be removed from the environment after the substitution.

_**Note**: `forstdin` reads from the standard input, so `pipeline` is needed here._

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
if { test -d $folder }

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
if { test -d $folder }

# Create the empty file
touch ${folder}/active
```

## Ambivalent block syntax

There are two ways to describe a block; the first is curly braces, and this is the one that we've used in this tutorial.

The second is to delimit only the end of a block with `""`.

```
pipeline { ls } wc -l
```

is equivalent to

```
pipeline ls "" wc -l
```

## Ending

Thank you for reading this getting started tutorial for this interesting scripting language. 

The great and complete documentation[^25] references other programs provided and a deep explanation of the design and the grammar[^26].

You can find some `execline` scripts in the `s6-rc` example services[^27] and in the s6-rc services[^28] used in Exherbo.

EDIT: Thanks to Cogitri[^29] and Heliocat[^30] for their suggestions on improving this guide.

[^1]: http://skarnet.org/software/execline/dieshdiedie.html
[^2]: http://skarnet.org/software/s6/
[^3]: http://skarnet.org/software/s6-rc/)
[^4]: https://skarnet.org/software/execline/el_semicolon.html
[^5]: http://skarnet.org/software/execline/fdmove.html
[^6]: https://en.wikipedia.org/wiki/File_descriptor
[^7]: http://skarnet.org/software/execline/fdmove.html
[^8]: http://www.linfo.org/pipe.html
[^9]: http://skarnet.org/software/execline/el_pushenv.html
[^10]: http://skarnet.org/software/execline/shift.html
[^11]: http://skarnet.org/software/execline/dollarat.html
[^12]: http://skarnet.org/software/execline/foreground.html
[^13]: http://skarnet.org/software/execline/background.html
[^14]: http://skarnet.org/software/execline/backtick.html
[^15]: http://skarnet.org/software/execline/if.html
[^16]: http://skarnet.org/software/execline/ifelse.html
[^17]: http://skarnet.org/software/execline/ifte.html
[^18]: http://skarnet.org/software/execline/ifthenelse.html
[^19]: https://www.computerhope.com/unix/test.htm
[^20]: http://skarnet.org/software/execline/ifelse.html
[^21]: http://skarnet.org/software/execline/loopwhilex.html
[^22]: http://skarnet.org/software/execline/forstdin.html
[^23]: http://skarnet.org/software/execline/forbacktickx.html
[^24]: http://skarnet.org/software/execline/importas.html
[^25]: http://skarnet.org/software/execline/
[^26]: http://skarnet.org/software/execline/grammar.html
[^27]: https://github.com/skarnet/s6-rc/tree/master/examples/source
[^28]: https://gitlab.exherbo.org/exherbo-misc/s6-exherbo
[^29]: https://github.com/Cogitri
[^30]: http://heliocat.net/

