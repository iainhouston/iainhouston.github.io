---
title: Making a post - Part 1 - Explaining a parameter expansion
date: 2019-12-18 18:05:26
layout: post
categories: ["Scripting", 'Blog']
---

Explaining a parameter expansion - Part 1
===============

I had a most enjoyable experience recently as a result of asking a question on the [Unix & Linux StackExchange Q&A Community forum](https://unix.stackexchange.com/questions/557473/what-kind-of-patterns-can-i-use-in-zsh-parameter-expansion/557482) . This is the collaborative internet at its best:

*  I research an experiment with an issue for ... well, a couple of days in this case. I'm a very casual shell script programmer, having written quite a few over the years for automating tasks, but have never got into it more deeply than necessary as shell scripts always seemed a bit arcane in contrast to the programming languages I've been used to: IBM /360 Basic Assembly Language; Python; C++; Modula-2; Go; Java, Coffeescript etc. etc..

*  Someone with a deep knowledge of my problem area - thank you [Stéphane Chazelas!](https://unix.stackexchange.com/users/22565/stéphane-chazelas) - takes the time and trouble to not only answer my question, but to elaborate on it; give some of the history  behind it; suggest alternatives, and so on.

*  I then research  unfamiliar aspects in the answer and learn a lot along the way, deepening my appreciation and respect for shell scripting, and very much enjoying my journey.

Experienced shell script programmers may scoff at my (previous) naïve prejudice avbout shell scripting. Well, there you go! I sincerely hope they enjoyed their original discovery of these aspects of shell scripting as much as I have just done.

Here's my original naïve solution to transforming a phrase into a file name, substituting characters  that I thought would be unsuitable for a filename with underscores; and removing any leading and trailing underscores.

```
regexp-replace FILENAME "[^[:alnum:]]" "_"
regexp-replace FILENAME "_{2,}" "_"
regexp-replace FILENAME "_+$" ""
regexp-replace FILENAME "^_+" ""
```

I didn't want to use `sed` and I didn't yet know about `zsh` (or other shells') parameter expansion.

Then, after discovering parameter expansion (PE) expressions in the form ...  

```
${var//pattern/replacement}
```  

... I found that some of the  *patterns*  I was using in `regexp-replace` didn't work in PE.

```
${name//[^[:alnum:]]/"_"} # Did work
${name//_{2,}/'_'}        # Did not work
```

I looked through `man zsh*` but didn't spot an answer, so asked the question on the [Unix & Linux StackExchange Q&A Community forum](https://unix.stackexchange.com/questions/557473/what-kind-of-patterns-can-i-use-in-zsh-parameter-expansion/557482). I knew that `regexp-replace` used POSIX 1003.2 regular expressions (although not PCRE ones unless compiled in, `man` told me) but what kind of expressions could I use as a *pattern* in PE?

So [Stéphane Chazelas!](https://unix.stackexchange.com/users/22565/stéphane-chazelas) pointed me to the extended globbing (`setopt extendedglob`) in `ksh` and its variations on POSIX 1003.2 regular expressions. For example `regexp-replace name "_{2,}" "_"` simply became `${name//_(#c2,)/_}`.

So now I could do:

```
${${${${name//[^[:alnum:]]/_}//_(#c2,)/_}/%_#}##_#}
```

Reducing four lines of code to one with four nested PEs served to keep my mind of Brexit for a few hours!

But I hadn't grasped the detail of *greedy* PE patterns until nudged in that direction. So I could then do:

```
name=${${${${var//[^[:alnum:]]/_}//_(#c2,)/_}/%_#}##_#}
```   

At the same time Stéphan suggested what he called a hack, but I thought was a rather elegant solution. The fact that it was *three* lines long - not an issue in return for such enjoyment in discovery!

```
words=()
: ${var//(#m)[[:alnum:]]##/${words[1+$#words]::=$MATCH}}
name=${(j:_:)words}
```

This what I discovered by unpacking the unfamiliar (to me) syntax:  

**Discovery 1** Arrays. Like `word` above. I'd read about them in `man zsh*` but I hadn't really connected that a shell script would use them. Doh! At this point I realised that shell scripts could be fully-fledged programs; not just the trivial scripts I had been writing. I'd always used Python for more serious DevOps tasks, but why go to the palaver of managing Python module versions, Python environments etc. when the shell has all the functionality I needed "ready to go".

**Discovery 2** `:` The so-called *POSIX special built-in* `:` (Similar to, but not the same as the *regular built-in* `true`) which can be used as above. As pointed out by a StackOverflow contributor *A useful application for `:` is if you're only interested in using parameter expansions for their side-effects rather than actually passing their result to a command.*

**Discovery 3** `(#m)` *"Set[s] references to the match data elements for the entire string ; this is similar to backreferencing ... The parameters $MATCH, $MBEGIN and $MEND will be set to the string matched and to the indices of the beginning and end of the string, respectively."* So, for each time a  string matching the pattern `[[:alnum:]]` is (greedily `##`) captured in `$MATCH`, it is recorded in the next `word` array element, thus giving us an array of `alnum` "words" from `var`.  Again, the side-effect is initialising `word`, no changes to `var` are made.  

**Discovery 4** `${(j:_:)words}` Joins the "words" of our `words` array together using '_' as a separator
