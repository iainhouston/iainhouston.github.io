---
title: Applescript assistant for creating Jekyll blog posts (pt 1)
date: 2020-01-27 23-48-15
layout: post
categories: ['Scripting']
---

Applescript assistant for creating Jekyll blog posts (pt 1)
===============
             
```osascript           
-- We keep three persistent settings in `makepost.plist`:
global chosenBlogPath, chosenCategories, chosenScriptFilePath

(*	
	The plist, and thus the three settings may not yet exist
	in which case we bootstrap them with some sensible values.
	
	Note:
	AppleScript aliases have to refer to existing files
	POSIX files and paths need not refer to existing files*)
property homeFolder : the POSIX path of (path to home folder) as string
property scriptFilePath : the POSIX path of (homeFolder & "bin/makepost.py")
property plistPath : the POSIX path of (homeFolder & ".makepost.plist")

-- the Title for the blog will  chosen by the user imminently 
-- (US users will choose their Title momentarily :-)
global chosenTitle

initialisePersistentSettings()

to initialisePersistentSettings()
	
	if not FileExists(plistPath) then
		-- create plist for bootstrap settings now, and user's choices later
		tell application "System Events"
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
			tell property list file plistPath
				set chosenBlogPath to value of property list item "blogPath"
				set chosenCategories to value of property list item "catsList"
				set chosenScriptFilePath to value of property list item "scriptFilePath"
			end tell
		end tell
		log "chosenBlogPath: " & chosenBlogPath
		log "chosenCategories: " & chosenCategories
		log "chosenScriptFilePath: " & chosenScriptFilePath
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
