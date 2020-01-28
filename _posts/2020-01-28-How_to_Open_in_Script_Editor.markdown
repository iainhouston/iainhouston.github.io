---
title: How to "Open in Script Editor"
date: 2020-01-28 09-23-38
layout: post
comments: true
categories: ['Scripting']
---

I was wondering how people show their AppleSript in their web pages and, by clicking a link, have it open in your Mac's Script Editor.

This is what I discovered (read back to front if you like!):

1.	You create a link in your web page in this form:    

	````
	<a href = "applescript://com.apple.scripteditor?action=new&script=
	"your script" 
	>Click to open in Script Editor</a>
	````
	
2.	where `your script` above is URL encoded    

3.	where you can URL encode your script with this command as follows:    

	```
	cat myscript.applescript | \
	python3 -c "import urllib.parse, sys; print(urllib.parse.quote(sys.stdin.read()))" \
	> myscript.urlencoded
	```
	
4.	Where `myscript.applescript` above is exported **as text** from Script Editor


So: tasks in order:

1.	Export as text  

2.	URL Encode  

3.	Paste URL encoded text between "quotes" into the `a` tag


To Do
-----  

See how to make the displayed code the link, rather than separate the link from the code as I have done here.