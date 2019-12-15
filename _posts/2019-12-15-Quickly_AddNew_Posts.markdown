---
title: Quickly add a new post
date: 2019-12-15 14:49:10
categories: ["Blogging"]
layout: post
---

Quickly add a new post
===============

There must be a million scripts written by Jekyll bloggers to create a new post in the format Jekyll requires.

Here's mine; which obviously needs to be updated to accept arguments for parameter variables that are currently hard coded. Just represents a bit of digging into `zsh`'s' Parameter Expansion functionality.

```sh
#!/bin/zsh

autoload -U regexp-replace

TITLE="Today's Title is tremend*& £s!"
FILENAME=$TITLE
regexp-replace FILENAME "[^[:alnum:]]" "_"
regexp-replace FILENAME "_{2,}" "_"
regexp-replace FILENAME "_+$" ""
regexp-replace FILENAME "^_+" ""

BLOGPATH='/Users/iain/WebDev/iainhouston.github.io/_posts/'

cat > ${BLOGPATH}`date "+%Y-%m-%d"`-${FILENAME}.markdown << EOF
---
title: ${TITLE}
date: `date "+%Y-%m-%d %H:%M:%S" `
layout: post
---

$TITLE
===============

Your text about ${TITLE} starts here
EOF
```
