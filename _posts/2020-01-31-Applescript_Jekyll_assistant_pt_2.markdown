---
title: Applescript Jekyll assistant (pt 2)
date: 2020-01-31 18-27-11
layout: post
comments: false
categories: ['Scripting']
---

Following on from [Applescript Jekyll assistant (pt 1)](/scripting/2020/01/27/Applescript_assistant_for_creating_Jekyll_blog_posts_pt_1.html) we're using a `.plist` to record the properties of the users' blog posts that persist for the lifetime of the blog.

The script allows the user to choose where she keeps he Jekyll blog posts and move it around fer directory structure over time.

Today's chunk reflects a better understanding of handling file references  in either `alias` (class `alias`) or `POSIX file` (class `text`) 

More chunks soon!

<a href = "applescript://com.apple.scripteditor?action=new&script=
--%20these%20two%20properties%20are%20immutable%20%0Aproperty%20homeFolder%20%3A%20the%20POSIX%20path%20of%20%28path%20to%20home%20folder%29%0Aproperty%20plistPath%20%3A%20the%20POSIX%20path%20of%20%28homeFolder%20%26%20%22.makepost.plist%22%29%0A%28%2A%0A%09We%20keep%20three%20persistent%20settings%20in%20%27makepost.plist%27%3A%0A%09%0A%09%27chosenBlogPath%27%20is%20the%20path%20to%20the%20folder%20where%20Jekyll%20expects%20blog%20posts%20to%20be.%0A%09This%20is%20typically%20the%20%27_posts%27%20subdirectory%20in%20the%20Jekyll%20project%20files.%0A%09%0A%09%27chosenCategories%27%20is%20a%20list%20of%20the%20user%27s%20Categories%20from%20which%20the%20user%20can%20choose%0A%09and%20which%20will%20appear%20in%20the%20frontmatter%20of%20the%20newly-to-be-created%20blog%20post%27s%20%27markdown%27%0A%09%0A%09%27chosenScriptFilePath%27%20is%20the%20path%20to%20where%20the%20user%20has%20stored%20%27makepost.py%27%0A%09which%20is%20written%20to%20be%20used%20as%20a%20shell%20command%20for%20creating%20new%20Jekyll%20blog%20posts%0A%09%0A%2A%29%0A--%20These%204%20properties%20of%20our%20Script%20Object%20are%20mutable%0A%0A--%20not%20persistent%20across%20runs%0Aproperty%20chosenTitle%20%3A%20%22Untitled%22%0A%0A--%20persistent%20across%20runs%0Aproperty%20chosenBlogPath%20%3A%20the%20POSIX%20path%20of%20homeFolder%0Aproperty%20chosenCategories%20%3A%20%7B%22Blog%22%2C%20%22Scripting%22%7D%0Aproperty%20chosenScriptFilePath%20%3A%20the%20POSIX%20path%20of%20%28homeFolder%20%26%20%22bin/makepost.py%22%29%0A%0A%28%2A%09%0A%09Initialisation%0A%09--%20--%20--%20--%20--%0A%0A%09The%20plist%2C%20and%20thus%20the%20three%20persistent%20settings%20above%20may%20not%20yet%20exist%0A%09in%20which%20case%20we%20have%20bootstrapped%20them%20with%20some%20sensible%20initial%20values.%0A%09%0A%09Notes%3A%0A%09%2A%20AppleScript%20aliases%20have%20to%20refer%20to%20existing%20files%0A%09%0A%09%2A%20POSIX%20files%20and%20paths%20need%20not%20refer%20to%20existing%20files%20or%20directories%0A%09%0A%09%2A%20The%20AppleScript%20path%20of%20a%20POSIX%20path%20is%20%22text%22%0A%2A%29%0A%0A%0A--%20overwite%20the%20above%20persistent%20properties%20if%20the%20user%20already%20has%20a%20.plist%0AinitialisePersistentSettings%28%29%0A%0A%0A%28%2A%0A%09Next%20we%20ask%20the%20user%0A%091.%20What%20the%20title%20of%20her%20blog%20is%20going%20to%20be%3B%0A%09%09and%20from%20this%2C%20we%20will%20derive%20the%20Jekyll-conformant%20file%20name%20for%20the%20post%0A%092.%20Which%20Folder%20she%20uses%20for%20posts%0A%09%09in%20her%20Jekyll-conformant%20Folder%20structure%0A%2A%29%0A--%20overwite%20the%20above%20properties%20as%20the%20user%20chooses%0AmanageTitleAndBlogpath%28%29%0A%0A%0A--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%0Alog%20%22chosenTitle%3A%20%22%20%26%20chosenTitle%0Alog%20%22chosenBlogPath%3A%20%22%20%26%20chosenBlogPath%0Alog%20%22chosenScriptFilePath%3A%20%22%20%26%20chosenScriptFilePath%0Arepeat%20with%20n%20from%201%20to%20the%20length%20of%20chosenCategories%0A%09log%20%22chosenCategories%20%28%22%20%26%20n%20%26%20%22%29%3A%20%22%20%26%20item%20n%20of%20chosenCategories%0Aend%20repeat%0A--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%0A%0A%0A--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%0A%28%2A%0A%09Handlers%20/%20subroutines%0A%2A%29%0A--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20%0A%0Ato%20manageTitleAndBlogpath%28%29%0A%09%0A%09--%20Reduce%20the%20length%20of%20the%20path%20for%20displaying%20as%20the%20legend%20in%20button%201%0A%09set%20pathClue%20to%20crypticPath%28chosenBlogPath%29%0A%09set%20chosenTitle%20to%20%22Untitled%22%0A%09%0A%09--%20We%27ll%20cycle%20the%20path-choosing%2C%20not%20forgetting%20the%20title-choosing%20each%20cycle%2C%20until%20user%20is%20happy%20with%20her%20choice%0A%09repeat%0A%09%09set%20%7BOKButton%2C%20chosenTitle%7D%20to%20%7Bbutton%20returned%2C%20text%20returned%7D%20of%20%28display%20dialog%20%22Choose%20the%20Title%20of%20your%20new%20post%22%20default%20answer%20chosenTitle%20with%20title%20%22Makepost%22%20buttons%20%7BpathClue%2C%20%22Other%22%7D%20default%20button%201%29%0A%09%09%0A%09%09if%20OKButton%20is%20pathClue%20then%0A%09%09%09--%20stay%20with%20the%20blogpath%20that%20we%20read%20from%20user%27s%20settings%20file%0A%09%09%09exit%20repeat%0A%09%09else%0A%09%09%09chooseBlogPath%28%29%0A%09%09%09copy%20crypticPath%28chosenBlogPath%29%20to%20pathClue%0A%09%09end%20if%0A%09%09%0A%09end%20repeat%0A%09%0Aend%20manageTitleAndBlogpath%0A%0Ato%20chooseBlogPath%28%29%0A%09%0A%09--%20User%20chooses%20her%20%20blog%20posts%20Folder%0A%09set%20chosenBlogPath%20to%20the%20POSIX%20path%20of%20%28choose%20folder%20with%20prompt%20%22Choose%20the%20path%20to%20your%20blog%20posts%20Folder%22%20default%20location%20chosenBlogPath%29%0A%09%0A%09--%20record%20chosenBlogPath%20in%20makepost%27s%20persistent%20settings%0A%09tell%20application%20%22System%20Events%22%0A%09%09tell%20property%20list%20file%20plistPath%0A%09%09%09set%20value%20of%20property%20list%20item%20%22blogPath%22%20to%20chosenBlogPath%0A%09%09end%20tell%0A%09end%20tell%0Aend%20chooseBlogPath%0A%0Aon%20crypticPath%28fullPath%29%20--%20%28String%29%20as%20String%0A%09set%20splitPath%20to%20splitText%28fullPath%2C%20%22/%22%29%0A%09set%20lastBit%20to%20%7B%7D%0A%09copy%20item%20%28%28count%20of%20splitPath%29%20-%202%29%20of%20splitPath%20to%20end%20of%20lastBit%0A%09copy%20%22/%22%20to%20end%20of%20lastBit%0A%09copy%20item%20%28%28count%20of%20splitPath%29%20-%201%29%20of%20splitPath%20to%20end%20of%20lastBit%0A%09%0A%09return%20%28%22Place%20in%20.../%22%20%26%20lastBit%20as%20text%29%20%26%20%22%20%3F%22%0Aend%20crypticPath%0A%0Ato%20splitText%28theText%2C%20theDelimiter%29%0A%09set%20AppleScript%27s%20text%20item%20delimiters%20to%20theDelimiter%0A%09set%20theTextItems%20to%20every%20text%20item%20of%20theText%0A%09set%20AppleScript%27s%20text%20item%20delimiters%20to%20%22%22%0A%09return%20theTextItems%0Aend%20splitText%0A%0A%0Ato%20initialisePersistentSettings%28%29%0A%09%0A%09if%20not%20FileExists%28plistPath%29%20then%0A%09%09%0A%09%09--%20create%20the%20plist%20with%20bootstrap%20settings%20now%2C%20and%20for%20recording%20the%20user%27s%20choices%20later%0A%09%09tell%20application%20%22System%20Events%22%0A%09%09%09%0A%09%09%09--%20Have%20a%20look%20at%20the%20%27Mac%20Automation%20Scripting%20Guide%27%20for%20more%20on%20this%20%27tell%27%20block%0A%09%09%09%0A%09%09%09--%20Create%20an%20empty%20property%20list%20dictionary%20item%0A%09%09%09set%20theParentDictionary%20to%20make%20new%20property%20list%20item%20with%20properties%20%7Bkind%3Arecord%7D%0A%09%09%09%0A%09%09%09--%20Create%20a%20new%20property%20list%20file%20using%20the%20empty%20dictionary%20list%20item%20as%20contents%0A%09%09%09set%20thePropertyListFilePath%20to%20plistPath%0A%09%09%09%0A%09%09%09set%20thePropertyListFile%20to%20make%20new%20property%20list%20file%20with%20properties%20%7Bcontents%3AtheParentDictionary%2C%20name%3AthePropertyListFilePath%7D%0A%09%09%09%0A%09%09%09tell%20property%20list%20items%20of%20thePropertyListFile%0A%09%09%09%09%0A%09%09%09%09--%20use%20bootstrapped%20settings%20of%20these%20properties%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20list%20key%20for%20accessing%20the%20users%27%20blog%20categories%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Alist%2C%20name%3A%22catsList%22%2C%20value%3AchosenCategories%7D%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20string%20key%20for%20accessing%20the%20path%20to%20the%20user%27s%20blog%20posts%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Astring%2C%20name%3A%22blogPath%22%2C%20value%3AchosenBlogPath%7D%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20string%20key%20for%20accessing%20the%20path%20to%20the%20python%20script%20%27makepost.py%27%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Astring%2C%20name%3A%22scriptFilePath%22%2C%20value%3AchosenScriptFilePath%7D%0A%09%09%09%09%0A%09%09%09end%20tell%0A%09%09end%20tell%0A%09%09%0A%09else%0A%09%09%0A%09%09--%20overwrite%20the%20bootstrapped%20values%20of%20these%20persistent%20properties%0A%09%09tell%20application%20%22System%20Events%22%0A%09%09%09%0A%09%09%09tell%20property%20list%20file%20plistPath%0A%09%09%09%09set%20chosenBlogPath%20to%20value%20of%20property%20list%20item%20%22blogPath%22%0A%09%09%09%09set%20chosenCategories%20to%20value%20of%20property%20list%20item%20%22catsList%22%0A%09%09%09%09set%20chosenScriptFilePath%20to%20value%20of%20property%20list%20item%20%22scriptFilePath%22%0A%09%09%09end%20tell%0A%09%09%09%0A%09%09end%20tell%0A%09%09%0A%09end%20if%0A%09%0Aend%20initialisePersistentSettings%0A%0Aon%20FileExists%28theFile%29%20--%20%28String%29%20as%20Boolean%0A%09--%20Convert%20the%20file%20to%20a%20string%0A%09set%20theFile%20to%20theFile%20as%20string%0A%09tell%20application%20%22System%20Events%22%0A%09%09if%20exists%20file%20theFile%20then%0A%09%09%09return%20true%0A%09%09else%0A%09%09%09return%20false%0A%09%09end%20if%0A%09end%20tell%0Aend%20FileExists%0A
">Open in Script Editor</a>

```applescript
-- these two properties are immutable 
property homeFolder : the POSIX path of (path to home folder)
property plistPath : the POSIX path of (homeFolder & ".makepost.plist")
(*
	We keep three persistent settings in 'makepost.plist':
	
	'chosenBlogPath' is the path to the folder where Jekyll expects blog posts to be.
	This is typically the '_posts' subdirectory in the Jekyll project files.
	
	'chosenCategories' is a list of the user's Categories from which the user can choose
	and which will appear in the frontmatter of the newly-to-be-created blog post's 'markdown'
	
	'chosenScriptFilePath' is the path to where the user has stored 'makepost.py'
	which is written to be used as a shell command for creating new Jekyll blog posts
	
*)
-- These 4 properties of our Script Object are mutable

-- not persistent across runs
property chosenTitle : "Untitled"

-- persistent across runs
property chosenBlogPath : the POSIX path of homeFolder
property chosenCategories : {"Blog", "Scripting"}
property chosenScriptFilePath : the POSIX path of (homeFolder & "bin/makepost.py")

(*	
	Initialisation
	-- -- -- -- --

	The plist, and thus the three persistent settings above may not yet exist
	in which case we have bootstrapped them with some sensible initial values.
	
	Notes:
	* AppleScript aliases have to refer to existing files
	
	* POSIX files and paths need not refer to existing files or directories
	
	* The AppleScript path of a POSIX path is "text"
*)


-- overwite the above persistent properties if the user already has a .plist
initialisePersistentSettings()


(*
	Next we ask the user
	1. What the title of her blog is going to be;
		and from this, we will derive the Jekyll-conformant file name for the post
	2. Which Folder she uses for posts
		in her Jekyll-conformant Folder structure
*)
-- overwite the above properties as the user chooses
manageTitleAndBlogpath()


-- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG --
log "chosenTitle: " & chosenTitle
log "chosenBlogPath: " & chosenBlogPath
log "chosenScriptFilePath: " & chosenScriptFilePath
repeat with n from 1 to the length of chosenCategories
	log "chosenCategories (" & n & "): " & item n of chosenCategories
end repeat
-- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG --


-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
(*
	Handlers / subroutines
*)
-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

to manageTitleAndBlogpath()
	
	-- Reduce the length of the path for displaying as the legend in button 1
	set pathClue to crypticPath(chosenBlogPath)
	set chosenTitle to "Untitled"
	
	-- We'll cycle the path-choosing, not forgetting the title-choosing each cycle, until user is happy with her choice
	repeat
		set {OKButton, chosenTitle} to {button returned, text returned} of (display dialog "Choose the Title of your new post" default answer chosenTitle with title "Makepost" buttons {pathClue, "Other"} default button 1)
		
		if OKButton is pathClue then
			-- stay with the blogpath that we read from user's settings file
			exit repeat
		else
			chooseBlogPath()
			copy crypticPath(chosenBlogPath) to pathClue
		end if
		
	end repeat
	
end manageTitleAndBlogpath

to chooseBlogPath()
	
	-- User chooses her  blog posts Folder
	set chosenBlogPath to the POSIX path of (choose folder with prompt "Choose the path to your blog posts Folder" default location chosenBlogPath)
	
	-- record chosenBlogPath in makepost's persistent settings
	tell application "System Events"
		tell property list file plistPath
			set value of property list item "blogPath" to chosenBlogPath
		end tell
	end tell
end chooseBlogPath

on crypticPath(fullPath) -- (String) as String
	set splitPath to splitText(fullPath, "/")
	set lastBit to {}
	copy item ((count of splitPath) - 2) of splitPath to end of lastBit
	copy "/" to end of lastBit
	copy item ((count of splitPath) - 1) of splitPath to end of lastBit
	
	return ("Place in .../" & lastBit as text) & " ?"
end crypticPath

to splitText(theText, theDelimiter)
	set AppleScript's text item delimiters to theDelimiter
	set theTextItems to every text item of theText
	set AppleScript's text item delimiters to ""
	return theTextItems
end splitText


to initialisePersistentSettings()
	
	if not FileExists(plistPath) then
		
		-- create the plist with bootstrap settings now, and for recording the user's choices later
		tell application "System Events"
			
			-- Have a look at the 'Mac Automation Scripting Guide' for more on this 'tell' block
			
			-- Create an empty property list dictionary item
			set theParentDictionary to make new property list item with properties {kind:record}
			
			-- Create a new property list file using the empty dictionary list item as contents
			set thePropertyListFilePath to plistPath
			
			set thePropertyListFile to make new property list file with properties {contents:theParentDictionary, name:thePropertyListFilePath}
			
			tell property list items of thePropertyListFile
				
				-- use bootstrapped settings of these properties
				
				-- Add a list key for accessing the users' blog categories
				make new property list item at end with properties {kind:list, name:"catsList", value:chosenCategories}
				
				-- Add a string key for accessing the path to the user's blog posts
				make new property list item at end with properties {kind:string, name:"blogPath", value:chosenBlogPath}
				
				-- Add a string key for accessing the path to the python script 'makepost.py'
				make new property list item at end with properties {kind:string, name:"scriptFilePath", value:chosenScriptFilePath}
				
			end tell
		end tell
		
	else
		
		-- overwrite the bootstrapped values of these persistent properties
		tell application "System Events"
			
			tell property list file plistPath
				set chosenBlogPath to value of property list item "blogPath"
				set chosenCategories to value of property list item "catsList"
				set chosenScriptFilePath to value of property list item "scriptFilePath"
			end tell
			
		end tell
		
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


