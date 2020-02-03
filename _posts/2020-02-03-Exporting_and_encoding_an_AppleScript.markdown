---
title: Exporting and encoding an AppleScript
date: 2020-02-03 13-15-48
layout: post
comments: false
categories: ['Scripting']
---

Further to me recent post ["A Folder Action to encode a script to be Opened in Editor"](/URLEncodeAppleScript), I have updated the [GitHub Gist](https://gist.github.com/iainhouston/0fcf1db99544bef5ce77853c9c47af08) noting that

>	AppleScript Editor exports a decompiled version of your compiled script which may
>	contain line continuation characters that the Python urllib.parse finds unpalatable. 
>	Thus, instead of merely catenating the .AppleScript into the output file, we do 
>	`iconv -f iso-8859-1 -t UTF-8 " & scriptInputName ...`  
>	to create a UTF-8 file that Python is happy with.