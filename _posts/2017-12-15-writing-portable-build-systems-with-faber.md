---
comments: true
layout: post
categories: software
---

##### Introduction

Faber is a project that evolved out of the need for a modern, portable build
tool. Even today most existing tools are either tied to a specific
platform (e.g., "GNU Make", "nmake"), programming language (e.g. "ant",
"Rake"), or project (e.g. "boost.build").
A few projects have tried to break out of these bounds (notably "CMake"
and "SCons"), but are overly complex, obscure, or inefficient.

One aspect that sets Faber apart from all of the above is its use of
Python rather than a domain-specific language, to define the build logic.

In this post I will illustrate a few specific design aspects that
allow the build logic (here expressed in "fabscripts") to become
platform-agnostic.


##### artefacts, actions, rules

Faber, just as any other build system, is concerned about building
things, which we'll call _artefacts_. They are updated using _actions_,
which describe how certain "target" artefacts are built, typically
from "source" artefacts. The connection from "source" to "target" is
expressed using _rules_. Let's consider a very simple example:

```python
compile = action('compile', 'g++ -c -o $(<) $(>)')
link = action('link', 'g++ -o $(<) $(>)')

obj = rule(compile, 'hello.o', source='hello.cpp')
exe = rule(link, 'hello', source=obj)
```

This forms a very simple pipeline such that invoking `faber hello`
will first compile 'hello.o' from 'hello.cpp', then link 'hello'
from 'hello.o'. (The syntax of the action commands above should be
intuitively clear: "$(<)" and "$(>)" denote the target and source 
artefacts, respectively.)

This is the fundamental structure of all build systems, i.e. some
file defines a set of artefacts and the way to update them, then
the build tool can figure out the exact chain of actions that needs
to be executed for the desired artefact to be updated.

While the above is extremely simple, both notationally as well as
conceptually, it unfortunately isn't very portable. Not all systems
may have a compiler called "g++" installed. Calling conventions, i.e.
the way to pass optional and non-optional arguments, may differ, too.
And finally, the names used for the built artefacts may also be
platform-dependent. While on Linux, "hello" may be a fine name for 
an executable, on Windows we may prefer to name it "hello.exe".

So how can we add the required flexibility to the above script to
make it portable while preserving the conceptual simplicity ?

##### encapsulating actions into tools

While the conceptual steps to compile the "hello" binary are the
same on all platforms, the exact actions can vary. The "compile" and
"link" actions are carried out by a compiler, but the exact spelling
of the commands depend both on the compiler, as well as on the platform.

Faber defines _tools_ to encapsulate concrete actions like the above.
For example, it defines a "cxx" tool, with (member) actions "compile"
and "link" (as well as a few others), that lets us rewrite the above
build script as

```python
from faber.tools.cxx import cxx

obj = rule(cxx.compile, 'hello.o', source='hello.cpp')
exe = rule(cxx.link, 'hello', source=obj)
```

"cxx.compile" now is an _abstract action_, which faber will try to
match with a concrete implementation, depending on the platform and
build parameters. Calling `faber hello` may actually perform the
same action as before (if `g++` is detected), or it may execute
`cl`, if it is being executed on Windows and a `MSVC` toolchain is
detected.
Users may also specify on the command line which tool to pick (if
multiple compilers are available), for example by running 
`faber cxx.name=msvc`.
Further, tools may be configured in a config file by instantiating
the appropriate compiler classes, such as::

```python
mingwxx = gxx(command='x86_64-w64-mingw32-g++', features=target(arch='w64'))
```

With that, `faber target.arch=w64` will find this MinGW compiler
and cross-compile the code for Windows 64.

##### parametrizing actions with features

Our original example above used extremely simple 'compile' and 'link'
commands. Real commands include different compiler flags (from header
search paths to code-generation options) that need to be added.

Faber abstacts such parameters into (mostly) platform-agnostic _features_
which will be mapped to tool-specific flags:

```python
from faber.tools.cxx import cxx, include, define

obj = rule(cxx.compile, 'hello.o', source='hello.cpp',
           featuers=(include('/search/path'), define('ANSWER=42'))
exe = rule(cxx.link, 'hello', source=obj)
```

In this example, we add an "include" and a "define" feature specifically
to the "hello.o" artefact. We can also define them globally, for example
by invoking `faber include=/search/path define=ANSWER=42`.

As you can guess, the "include" feature adds a header search path
(which maps to an `-I` option for `g++` on Linux, and `/I` for `cl` on Windows),
and the "define" feature adds a macro definition.

##### using implicit rules

So far we have used explicit rules to define the build pipeline
from "hello.cpp" to "hello". Typically, a project consists of many source
files that all need to be compiled with the same (or at least, similar)
options.

Artefacts have a _type_ (e.g. "cxx", "obj", "bin"), which is either 
explicitly specified, or inferred from their name. Tools may define 
_implicit rules_, which map between _artefact types_. I.e., rather than 
calling a rule to update a specific target artefact (such as "hello.o")
from a given source artefact (such as "hello.cpp"), a tool calls an 
implicit rule which defines how to update a target artefact type (such as "obj")
from a source artefact type (such as "cxx").

Compiler instances will declare their respective "compile" and "link"
actions to build "obj" from "cxx" sources, and "bin" from "obj", respectively,
which faber can use to chain multiple implicit rules to build e.g. a "bin"
from "cxx" sources.

In fact, this process is so common that faber encapsulates all this into
higher-order artefacts. We can thus eliminate even more platform-specific
code by rewriting the above as

```python
from faber.artefacts.binary import binary

hello = binary('hello', 'hello.cpp')
```

"binary" is derived from "artefact". Rather than being created by
invoking a "rule", it is constructed directly, using name and a source
arguments. Note there is no action argument any longer.
Invoking `faber hello` on this fabscript will still perform the
expected actions, as can be seen by the output:

```
gxx.compile hello.o
gxx.link hello
...made 2 artefacts...
```

Can you guess what the following fabscript does ?


```python
from faber.artefacts.library import library
from faber.artefacts.binary import binary

greet = library('greet', 'greet.cpp')
hello = binary('hello', ['hello.cpp', greet])
```

Hint: here is the output `faber hello` may produce:

```
gxx.compile greet.o
gxx.archive greet
gxx.compile hello.o
gxx.link hello
...made 4 artefacts...
```

##### Conclusion

In this post I have tried to give a quick overview
of how to write portable build logic that works
on different platforms, with different compilers.

For an in depth look please refer to Faber's 
[documentation](http://stefanseefeld.github.io/faber).

Faber is a very young project. As such, it will
lack quite a number of features, or support for
more tools or platforms. Oh, and there may be 
one or two bugs left. ;-)

The source code is [here](http://github.com/stefanseefeld/faber).

I would welcome any contribution !
