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

For example, today's chunk addresses using macOS's  `plist` to persist the user's preferences / settings between blog post creation sessions.

I'll also publish 

References
==========

[These Apple web pages](https://developer.apple.com/library/archive/documentation/LanguagesUtilities/Conceptual/MacAutomationScriptingGuide/index.html) (*Mac Automation Scripting Guide*) are a really nice resource to start with.

[This comprehensive AppleScript Language Guide](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html#//apple_ref/doc/uid/TP40000983-CH208-SW1) is essential, and quite unusually and frustratingly organised until you get to know your way around it.


Today's chunk
=============

<a href = "applescript://com.apple.scripteditor?
action=new&script=%28%2A%0A%09We%20keep%20three%20persistent%20settings%20in%20%60makepost.plist%60%3A%0A%09%0A%09%60chosenBlogPath%60%20is%20the%20path%20to%20the%20folder%20where%20Jekyll%20expects%20blog%20posts%20to%20be.%0A%09This%20is%20typically%20the%20%60_posts%60%20subdirectory%20in%20the%20Jekyll%20project%20files.%0A%09%0A%09%60chosenCategories%60%20is%20a%20list%20of%20the%20user%27s%20Categories%20from%20which%20the%20user%20can%20choose%0A%09and%20which%20will%20appear%20in%20the%20frontmatter%20of%20the%20newly-to-be-created%20blog%20post%27s%20%60markdown%60%0A%09%0A%09%60chosenScriptFilePath%60%20is%20the%20path%20to%20where%20the%20user%20has%20stored%20%60makepost.py%60%0A%09which%20is%20written%20to%20be%20used%20as%20a%20shell%20command%20for%20creating%20new%20Jekyll%20blog%20posts%0A%09%0A%2A%29%0Aglobal%20chosenBlogPath%2C%20chosenCategories%2C%20chosenScriptFilePath%0A%0A%28%2A%09%0A%09The%20plist%2C%20and%20thus%20the%20three%20settings%20may%20not%20yet%20exist%0A%09in%20which%20case%20we%20bootstrap%20them%20with%20some%20sensible%20values.%0A%09%0A%09Note%3A%0A%09AppleScript%20aliases%20have%20to%20refer%20to%20existing%20files%0A%09POSIX%20files%20and%20paths%20need%20not%20refer%20to%20existing%20files%0A%2A%29%0Aproperty%20homeFolder%20%3A%20the%20POSIX%20path%20of%20%28path%20to%20home%20folder%29%20--%20as%20string%0Aproperty%20scriptFilePath%20%3A%20the%20POSIX%20path%20of%20%28homeFolder%20%26%20%22bin/makepost.py%22%29%0Aproperty%20plistPath%20%3A%20the%20POSIX%20path%20of%20%28homeFolder%20%26%20%22.makepost.plist%22%29%0A%0A--%20the%20Title%20for%20the%20blog%20will%20%20chosen%20by%20the%20user%20imminently%20%0A--%20%28US%20users%20will%20choose%20their%20Title%20momentarily%20%3B-%29%0Aglobal%20chosenTitle%0A%0AinitialisePersistentSettings%28%29%0A%0Ato%20initialisePersistentSettings%28%29%0A%09%0A%09if%20not%20FileExists%28plistPath%29%20then%0A%09%09%0A%09%09--%20create%20the%20plist%20with%20bootstrap%20settings%20now%2C%20and%20for%20recording%20the%20user%27s%20choices%20later%0A%09%09tell%20application%20%22System%20Events%22%0A%09%09%09%0A%09%09%09--%20Have%20a%20look%20at%20the%20%60Mac%20Automation%20Scripting%20Guide%60%20for%20more%20on%20this%20%60tell%60%20block%0A%09%09%09%0A%09%09%09--%20Create%20an%20empty%20property%20list%20dictionary%20item%0A%09%09%09set%20theParentDictionary%20to%20make%20new%20property%20list%20item%20with%20properties%20%7Bkind%3Arecord%7D%0A%09%09%09%0A%09%09%09--%20Create%20a%20new%20property%20list%20file%20using%20the%20empty%20dictionary%20list%20item%20as%20contents%0A%09%09%09set%20thePropertyListFilePath%20to%20plistPath%0A%09%09%09%0A%09%09%09set%20thePropertyListFile%20to%20make%20new%20property%20list%20file%20with%20properties%20%7Bcontents%3AtheParentDictionary%2C%20name%3AthePropertyListFilePath%7D%0A%09%09%09%0A%09%09%09tell%20property%20list%20items%20of%20thePropertyListFile%0A%09%09%09%09--%20create%20bootstrap%20settings%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20list%20key%20for%20accessing%20the%20users%27%20blog%20categories%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Alist%2C%20name%3A%22catsList%22%2C%20value%3A%7B%22Blog%22%2C%20%22Scripting%22%7D%7D%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20string%20key%20for%20accessing%20the%20path%20to%20the%20user%27s%20blog%20posts%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Astring%2C%20name%3A%22blogPath%22%2C%20value%3AhomeFolder%7D%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20string%20key%20for%20accessing%20the%20path%20to%20the%20python%20script%20%60makepost.py%60%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Astring%2C%20name%3A%22scriptFilePath%22%2C%20value%3AscriptFilePath%7D%0A%09%09%09%09%0A%09%09%09end%20tell%0A%09%09end%20tell%0A%09%09%0A%09else%0A%09%09log%20%22plist%20available%20for%20reading%22%0A%09%09tell%20application%20%22System%20Events%22%0A%09%09%09--%20Have%20a%20look%20at%20the%20%60Mac%20Automation%20Scripting%20Guide%60%20for%20more%20on%20this%20%60tell%60%20block%0A%09%09%09%0A%09%09%09tell%20property%20list%20file%20plistPath%0A%09%09%09%09set%20chosenBlogPath%20to%20value%20of%20property%20list%20item%20%22blogPath%22%0A%09%09%09%09set%20chosenCategories%20to%20value%20of%20property%20list%20item%20%22catsList%22%0A%09%09%09%09set%20chosenScriptFilePath%20to%20value%20of%20property%20list%20item%20%22scriptFilePath%22%0A%09%09%09end%20tell%0A%09%09end%20tell%0A%09%09%0A%09%09log%20%22chosenBlogPath%3A%20%22%20%26%20chosenBlogPath%0A%09%09log%20%22chosenScriptFilePath%3A%20%22%20%26%20chosenScriptFilePath%0A%09%09repeat%20with%20n%20from%201%20to%20the%20length%20of%20chosenCategories%0A%09%09%09log%20%22chosenCategories%20%28%22%20%26%20n%20%26%20%22%29%3A%20%22%20%26%20item%20n%20of%20chosenCategories%0A%09%09end%20repeat%0A%09%09%0A%09end%20if%0Aend%20initialisePersistentSettings%0A%0Aon%20FileExists%28theFile%29%20--%20%28String%29%20as%20Boolean%0A%09--%20Convert%20the%20file%20to%20a%20string%0A%09set%20theFile%20to%20theFile%20as%20string%0A%09tell%20application%20%22System%20Events%22%0A%09%09if%20exists%20file%20theFile%20then%0A%09%09%09return%20true%0A%09%09else%0A%09%09%09return%20false%0A%09%09end%20if%0A%09end%20tell%0Aend%20FileExists%0A">You can open this in Script Editor.</a>

             
```applescript           
(*
	We keep three persistent settings in 'makepost.plist`:
	
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