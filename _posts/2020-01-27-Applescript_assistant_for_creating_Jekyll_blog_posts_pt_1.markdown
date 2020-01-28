---
title: Applescript Jekyll assistant (pt 1)
date: 2020-01-27 23-48-15
layout: post
comments: true
categories: ['Scripting']
---

Introduction
============

I'm posting chunks of the Applescript as I develop it. This will encourage me to add more explanatory comments in the script than I might otherwise do. 

My aim is to share things I've discovered about Applescript.

I already have a working version of the Applescript Jekyll assistant. This series of posts record the process of refactoring  the working script as I discover ways of making the Applescript simpler. 

For example, today's chunk addresses macOS's using `plist` to persist the user's preferences / settings between blog post creation sessions.

References
==========

[These Apple web pages](https://developer.apple.com/library/archive/documentation/LanguagesUtilities/Conceptual/MacAutomationScriptingGuide/index.html) (*Mac Automation Scripting Guide*) are a really nice resource to start with.

[This comprehensive AppleScript Language Guide](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html#//apple_ref/doc/uid/TP40000983-CH208-SW1) is essential, and quite unusually and frustratingly organised until you get to know your way around it.


Today's chunk
=============

You can copy this into the Script Editor and run it.

             
```applescript           
(*
	We keep three persistent settings in `makepost.plist`:
	
	`chosenBlogPath` is the path to the folder where Jekyll expects blog posts to be.
	This is typically the `_posts` subdirectory in the Jekyll project files.
	
	`chosenCategories` is a list of the user's Categories from which the user can choose
	and which will appear in the frontmatter of the newly-to-be-created blog post's `markdown`
	
	`chosenScriptFilePath` is the path to where the user has stored `makepost.py`
	which is written to be used as a shell command for creating new Jekyll blog posts
	
*)
global chosenBlogPath, chosenCategories, chosenScriptFilePath

(*	
	The plist, and thus the three settings may not yet exist
	in which case we bootstrap them with some sensible values.
	
	Note:
	AppleScript aliases have to refer to existing files
	POSIX files and paths need not refer to existing files
*)
property homeFolder : the POSIX path of (path to home folder) -- as string
property scriptFilePath : the POSIX path of (homeFolder & "bin/makepost.py")
property plistPath : the POSIX path of (homeFolder & ".makepost.plist")

-- the Title for the blog will  chosen by the user imminently 
-- (US users will choose their Title momentarily ;-)
global chosenTitle

initialisePersistentSettings()

to initialisePersistentSettings()
	
	if not FileExists(plistPath) then
	
		-- create the plist with bootstrap settings now, and for recording the user's choices later
		tell application "System Events"
		
			-- Have a look at the `Mac Automation Scripting Guide` for more on this `tell` block

			-- Create an empty property list dictionary item
			set theParentDictionary to make new property list item with properties {kind:record}
			
			-- Create a new property list file using the empty dictionary list item as contents
			set thePropertyListFilePath to plistPath
			
			set thePropertyListFile to make new property list file with properties {contents:theParentDictionary, name:thePropertyListFilePath}
			
			tell property list items of thePropertyListFile
				-- create bootstrap settings
				
				-- Add a list key for accessing the users' blog categories
				make new property list item at end with properties {kind:list, name:"catsList", value:{"Blog", "Scripting"}}
				
				-- Add a string key for accessing the path to the user's blog posts
				make new property list item at end with properties {kind:string, name:"blogPath", value:homeFolder}
				
				-- Add a string key for accessing the path to the python script `makepost.py`
				make new property list item at end with properties {kind:string, name:"scriptFilePath", value:scriptFilePath}
				
			end tell
		end tell
		
	else
		log "plist available for reading"
		tell application "System Events"
		-- Have a look at the `Mac Automation Scripting Guide` for more on this `tell` block

			tell property list file plistPath
				set chosenBlogPath to value of property list item "blogPath"
				set chosenCategories to value of property list item "catsList"
				set chosenScriptFilePath to value of property list item "scriptFilePath"
			end tell
		end tell
		
		log "chosenBlogPath: " & chosenBlogPath
		log "chosenScriptFilePath: " & chosenScriptFilePath
		repeat with n from 1 to the length of chosenCategories
			log "chosenCategories (" & n & "): " & item n of chosenCategories
		end repeat
		
	end if
end initialisePersistentSettings

on FileExists(theFile) -- (String) as Boolean
	-- Convert the file to a string
	set theFile to theFile as string
	tell application "System Events"
		if exists file theFile then
			return true
		else
			return false
		end if
	end tell
end FileExists
```

Python access to plist
======================

At a later stage `makepost.py` will want to read the `.plist` file.  

Here's a proof of concept showing how the Python `plistlib` can be used to access it.


```python
#!/usr/bin/python3

import os
import plistlib

fileName=os.path.expanduser('~/.makepost.plist')

if os.path.exists(fileName):

	with open(fileName, 'rb') as fp:
		pl = plistlib.load(fp, fmt=None, use_builtin_types=False)

	print("\nThe plist's contents \n{}\n" .format(pl))

	if 'blogPath' in pl:
		print('blogPath value is "{}"\n'.format(pl['blogPath']))
	else:
		print('There is no blogPath key in the plist\n')

else:
	print("{} does not exist".format(fileName))


```