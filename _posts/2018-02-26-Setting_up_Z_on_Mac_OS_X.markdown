---
title: Setting up Z on Mac OS X
date: 2018-02-26 23-19-56
layout: post
comments: true
categories: ['Formal Specifications']
---

How to set up LaTeX and the Fuzz type checker to prepare your Z specifications on MacOS High Sierra

Downloads - Required software
------------------------------
-  The LaTeX part of this is [The MacTeX-2017 Distribution](http://tug.org/mactex/)   
-  The Z-specific LaTeX package; the fonts; and the Fuzz type-checker are all downloaded from [Mike Spivey's web server](http://spivey.oriel.ox.ac.uk/mike/fuzz/fuzz-3.4.1.tgz) in a single gzipped tar file containing the source code and documentation.
-  Apple's XCode from the Mac  App Store.
-  The Gnu `gcc` compiler proper - rather than XCode's gcc alias to its more contemporary llvm compiler. Use `brew install gcc` to get the HomeBrew version which gives us the executable - as of writing - `gcc-7`
-  The Gnu `awk` rather than the BSD one: `brew install gawk`

Installing MacTeX
------------------
[MacTeX](http://tug.org/mactex/)  is a standard Mac OS X package so in installed in the standard way.  

It comes with two main apps: *TexShop*, the LaTeX editor; and the *Tex Live Utility* which is a kind of package manager and preferences editor.

Once you've fired up TexShop - the editor / IDE - immediately "check for updates" .
Similarly, *TeX Live Utility* will most probably display an error until after you also complete "check for updates".

#### Now you can typeset your Z spec

The MacTeX distribution comes with the `zed-csp` package already pre-installed, so you can start writing your Z spec without further ado (see below for using `zed-csp`)  

So some of the following is actually redundant. You can stop here if all you want is to typeset a Z spec.

In my case I  also want to have the fuzz type-checking program built, and it will do no harm to have the zed fonts and style file installed again locally.

Installing the Z bits
----------------------
Now we have a working LaTeX environment, and we want to proceed to installing `fuzz`. The same Makefile that builds the Fuzz type checker executable also installs the `fuzz` LaTeX package

So we need to adapt the Makefile to our OS X's BSD Unix basis as Mike's default Makefile is set up for a Linux distro.

-  Edit the `Makefile` in the downloaded fuzz root directory. For me this is `/Users/iainhouston/Downloads/fuzz-3.4.1/Makefile`. We use the customisation directories referenced by the MacTeX package. These are in separate directories from the MacTeX app and will remain intact after any updates to MacTeX.   

```sh
TEXDIR = /Users/iainhouston/Library/texmf/tex
MFDIR = /usr/local/texlive/texmf-local/fonts/source/local
```

-  Edit the `Makefile` in the  fuzz `src` directory to be specific about the compilers we're using. For me this is `/Users/iainhouston/Downloads/fuzz-3.4.1/src/Makefile`. Update just these two lines so that OS X can find the C compiler (`gcc-7`) and the C preprocessor (`cpp`).

```sh
# CC=gcc
CC=gcc-7

# CPP=/lib/cpp  
CPP=/usr/bin/cpp
```

-	Edit line 291 of `src/zscan.l` to comment out a duplicate / redundant declaraton that will otherwise cause a compilation error.  

```sh
/* EXTERN char *strncpy(); */
```
<!-- \* -->

Now we can follow Mike Spivey's instructions and, from the fuzz root directory (for me `/Users/iainhouston/Downloads/fuzz-3.4.1`) we do:  

```sh
make 				# build the executable
make test 			# test the build went OK
sudo make install	# distribute all the LaTeX bits and the executable properly
sudo texhash		# tell MacTeX of the Z package
```

Testing the whole thing
-----------------------
We want to ensure that we can:

-  Typeset a Z specification relying upon all the Z LaTeX additions being in the common locations rather than in the same directory as the specification source.
- Type check a Z specification

So we'll copy `example.tex` and `tut.tex` from the downloaded `tex` directory to a separate one and use the TexShop GUI to open and typeset these files and use `fuzz example.tex` and `fuzz -t tut.tex` to show that the type checker is doing its thing.

`fuzz example.tex` should report one error and `fuzz -t tut.tex` should produce no errors but report a lot of types of global definitions.

Using the `zed-csp` pre-installed package
------------------------------------------

Some kind person has created a [template here (Overleaf)](https://www.overleaf.com/14040901jsfzfpqvhfmf#/54458206/) that demonstrates the use of the pre-installed `MacTeX` package `zed-csp`.

```latex
\documentclass[12pt]{article}
\usepackage{zed-csp}
\begin{document}
\title{Typesetting Z specifications with \texttt{zed-csp}}
\author{}\date{}
\maketitle
\thispagestyle{empty}

\begin{schema}{PhoneDB}
known: \power NAME \\ phone: NAME \pfun PHONE
\where
known = \dom phone
\end{schema}
.
.
.
\end{document}
```

The Fuzz type checker is not concerned whether you choose to use the `fuzz` or the `zed-csp` package.  

Incidentally, you'll notice that the [Overleaf example](https://www.overleaf.com/14040901jsfzfpqvhfmf#/54458206/) is not strictly type correct and needs a couple of lines added, in any case it's not pretending to say anything much sensible but suffices to demonstrate the pre-installed `MacTeX` package:

```latex
.
.
%% \begin{zed} [NAME, PHONE, RESOURCE] \end{zed}
.
.
%%unchecked
\begin{gendef}[X,Y]
first: X \cross Y \fun X
.
.
```