---
title: A Folder Action to encode a script to be Opened in Editor
date: 2020-01-31 07-05-04
layout: post
comments: true
permalink:  URLEncodeAppleScript
categories: ['Scripting']
---

To use:  

*	Place this script in `~/Library/Scripts/Folder Action Scripts/`

*	Use *Folder Actions Setup* to associate this script with a particular folder  
	(Do: Control-click a folder in *Finder* >> *Services* >> *Folder Actions Setup*)

<a href = "applescript://com.apple.scripteditor?action=new&script=
property%20done_foldername%20%3A%20%22URL%20Encoded%20Scripts%22%0A%0Aon%20adding%20folder%20items%20to%20this_folder%20after%20receiving%20these_items%0A%09--%20If%20the%20result%20folder%20doesn%27t%20exist%2C%20then%20create%20it%0A%09tell%20application%20%22Finder%22%0A%09%09if%20not%20%28exists%20folder%20done_foldername%20of%20this_folder%29%20then%0A%09%09%09make%20new%20folder%20at%20this_folder%20with%20properties%20%7Bname%3Adone_foldername%7D%0A%09%09%09set%20current%20view%20of%20container%20window%20of%20this_folder%20to%20list%20view%0A%09%09end%20if%0A%09%09set%20the%20target_folder%20to%20folder%20done_foldername%20of%20this_folder%0A%09%09log%20%22info%20for%20this_folder%3A%20%22%20%26%20%28this_folder%29%0A%09end%20tell%0A%09%0A%09--%20%20Process%20each%20file%20newly-added%20to%20this_folder%0A%09try%0A%09%09repeat%20with%20i%20from%201%20to%20number%20of%20items%20in%20these_items%0A%09%09%09set%20this_item%20to%20item%20i%20of%20these_items%0A%09%09%09set%20the%20item_info%20to%20the%20info%20for%20this_item%0A%09%09%09if%20%28alias%20of%20the%20item_info%20is%20false%20and%20the%20type%20identifier%20of%20the%20item_info%20is%20%22com.apple.applescript.text%22%29%20or%20%28the%20name%20extension%20of%20the%20item_info%20is%20%22applescript%22%29%20then%0A%09%09%09%09tell%20application%20%22Finder%22%0A%09%09%09%09%09--%20LOOK%20FOR%20EXISTING%20MATCHING%20ITEMS%20IN%20THE%20DESTINATION%20FOLDER%0A%09%09%09%09%09--%20IF%20THERE%20ARE%20MATCHES%2C%20THEN%20RENAME%20THE%20CONFLICTING%20FILES%20INCREMENTALLY%0A%09%09%09%09%09my%20resolve_conflicts%28this_item%2C%20target_folder%29%0A%09%09%09%09%09--%20MOVE%20THE%20ITEM%20TO%20THE%20DESTINATION%20FOLDER%0A%09%09%09%09%09set%20the%20target_file%20to%20%28move%20this_item%20to%20the%20target_folder%20with%20replacing%29%20as%20alias%0A%09%09%09%09%09%0A%09%09%09%09%09set%20target_folderName%20to%20the%20POSIX%20path%20of%20%28target_folder%20as%20string%29%20as%20string%0A%09%09%09%09%09set%20the%20scriptInputName%20to%20target_folderName%20%26%20the%20text%20of%20%28name%20of%20%28info%20for%20target_file%29%29%0A%09%09%09%09%09set%20the%20scriptInputExtension%20to%20%22.%22%20%26%20%28the%20text%20of%20%28name%20extension%20of%20%28info%20for%20target_file%29%29%29%0A%09%09%09%09%09set%20the%20scriptTrimName%20to%20my%20trimText%28scriptInputName%2C%20scriptInputExtension%2C%20%22end%22%29%0A%09%09%09%09%09set%20the%20scriptOutputName%20to%20scriptTrimName%20%20%26%20%22.urlencoded%22%0A%09%09%09%09%09%0A%09%09%09%09%09set%20the%20scriptInputName%20to%20the%20quoted%20form%20of%20scriptInputName%0A%09%09%09%09%09set%20the%20scriptOutputName%20to%20the%20quoted%20form%20of%20scriptOutputName%0A%09%09%09%09end%20tell%0A%09%09%09%09--%20PROCESS%20THE%20ITEM%0A%09%09%09%09process_item%28scriptInputName%2C%20scriptOutputName%29%0A%09%09%09else%0A%09%09%09%09tell%20application%20%22Finder%22%0A%09%09%09%09%09set%20myErrorMessage%20to%20%22Expecting%20%22%20%26%20scriptTrimName%20%26%20%22%20to%20have%20been%20exported%20as%20Text%20%22%0A%09%09%09%09%09activate%0A%09%09%09%09%09display%20dialog%20myErrorMessage%20buttons%20%7B%22Cancel%22%7D%20default%20button%201%20giving%20up%20after%20120%0A%09%09%09end%20tell%0A%09%09%09end%20if%0A%09%09end%20repeat%0A%09on%20error%20error_message%20number%20error_number%0A%09%09if%20the%20error_number%20is%20not%20-128%20then%0A%09%09%09tell%20application%20%22Finder%22%0A%09%09%09%09activate%0A%09%09%09%09display%20dialog%20error_message%20buttons%20%7B%22Cancel%22%7D%20default%20button%201%20giving%20up%20after%20120%0A%09%09%09end%20tell%0A%09%09end%20if%0A%09end%20try%0Aend%20adding%20folder%20items%20to%0A%0Aon%20resolve_conflicts%28this_item%2C%20target_folder%29%0A%09tell%20application%20%22Finder%22%0A%09%09set%20the%20quotedName%20to%20quoted%20form%20of%20POSIX%20path%20of%20this_item%0A%09%09log%20%22quotedName%3A%20%22%20%26%20quotedName%0A%09%09set%20the%20file_name%20to%20the%20name%20of%20this_item%0A%09%09if%20%28exists%20document%20file%20file_name%20of%20target_folder%29%20then%0A%09%09%09set%20file_extension%20to%20the%20name%20extension%20of%20this_item%0A%09%09%09if%20the%20file_extension%20is%20%22%22%20then%0A%09%09%09%09set%20the%20trimmed_name%20to%20the%20file_name%0A%09%09%09else%0A%09%09%09%09set%20the%20trimmed_name%20to%20text%201%20thru%20-%28%28length%20of%20file_extension%29%20%2B%202%29%20of%20the%20file_name%0A%09%09%09end%20if%0A%09%09%09set%20the%20name_increment%20to%201%0A%09%09%09repeat%0A%09%09%09%09set%20the%20new_name%20to%20%28the%20trimmed_name%20%26%20%22.%22%20%26%20%28name_increment%20as%20string%29%20%26%20%22.%22%20%26%20file_extension%29%20as%20string%0A%09%09%09%09if%20not%20%28exists%20document%20file%20new_name%20of%20the%20target_folder%29%20then%0A%09%09%09%09%09--%20rename%20to%20conflicting%20file%0A%09%09%09%09%09set%20the%20name%20of%20document%20file%20file_name%20of%20the%20target_folder%20to%20the%20new_name%0A%09%09%09%09%09exit%20repeat%0A%09%09%09%09else%0A%09%09%09%09%09set%20the%20name_increment%20to%20the%20name_increment%20%2B%201%0A%09%09%09%09end%20if%0A%09%09%09end%20repeat%0A%09%09end%20if%0A%09end%20tell%0Aend%20resolve_conflicts%0A%0A--%20this%20sub-routine%20processes%20files%20%0Aon%20process_item%28scriptInputName%2C%20scriptOutputName%29%0A%09--%20NOTE%20that%20the%20variable%20this_item%20is%20a%20file%20reference%20in%20alias%20format%20%0A%09--%20FILE%20PROCESSING%20STATEMENTS%20GOES%20HERE%20%0A%09try%0A%09%09with%20timeout%20of%203%20seconds%0A%09%09%09set%20shellCommand%20to%20%22echo%20%27%3Ca%20href%20%3D%20%5C%22applescript%3A//com.apple.scripteditor%3Faction%3Dnew%26script%3D%27%20%3E%20%22%20%26%20scriptOutputName%20%26%20%22%20%26%26%20cat%20%22%20%26%20scriptInputName%20%26%20%22%20%7C%20python3%20-c%20%5C%22import%20urllib.parse%2C%20sys%3B%20print%28urllib.parse.quote%28sys.stdin.read%28%29%29%29%5C%22%20%3E%3E%20%22%20%26%20scriptOutputName%20%26%20%22%20%26%26%20echo%20%27%5C%22%3EOpen%20in%20Script%20Editor%3C/a%3E%27%20%3E%3E%20%22%20%26%20scriptOutputName%0A%09%09%09do%20shell%20script%20shellCommand%0A%09%09end%20timeout%0A%09on%20error%20error_message%0A%09%09tell%20application%20%22Finder%22%0A%09%09%09activate%0A%09%09%09display%20dialog%20error_message%20buttons%20%7B%22Cancel%22%7D%20default%20button%201%20giving%20up%20after%20120%0A%09%09end%20tell%0A%09end%20try%0Aend%20process_item%0A%0Aon%20trimText%28theText%2C%20theCharactersToTrim%2C%20theTrimDirection%29%0A%09set%20theTrimLength%20to%20length%20of%20theCharactersToTrim%0A%09if%20theTrimDirection%20is%20in%20%7B%22beginning%22%2C%20%22both%22%7D%20then%0A%09%09repeat%20while%20theText%20begins%20with%20theCharactersToTrim%0A%09%09%09try%0A%09%09%09%09set%20theText%20to%20characters%20%28theTrimLength%20%2B%201%29%20thru%20-1%20of%20theText%20as%20string%0A%09%09%09on%20error%0A%09%09%09%09--%20text%20contains%20nothing%20but%20trim%20characters%0A%09%09%09%09return%20%22%22%0A%09%09%09end%20try%0A%09%09end%20repeat%0A%09end%20if%0A%09if%20theTrimDirection%20is%20in%20%7B%22end%22%2C%20%22both%22%7D%20then%0A%09%09repeat%20while%20theText%20ends%20with%20theCharactersToTrim%0A%09%09%09try%0A%09%09%09%09set%20theText%20to%20characters%201%20thru%20-%28theTrimLength%20%2B%201%29%20of%20theText%20as%20string%0A%09%09%09on%20error%0A%09%09%09%09--%20text%20contains%20nothing%20but%20trim%20characters%0A%09%09%09%09return%20%22%22%0A%09%09%09end%20try%0A%09%09end%20repeat%0A%09end%20if%0A%09return%20theText%0Aend%20trimText
">Click to open the below script in Script Editor</a>

(Also published as a [GitHub Gist](https://gist.github.com/iainhouston/0fcf1db99544bef5ce77853c9c47af08) which I keep up to date as necessary)

```applescript
property done_foldername : "URL Encoded Scripts"

on adding folder items to this_folder after receiving these_items
	-- If the result folder doesn't exist, then create it
	tell application "Finder"
		if not (exists folder done_foldername of this_folder) then
			make new folder at this_folder with properties {name:done_foldername}
			set current view of container window of this_folder to list view
		end if
		set the target_folder to folder done_foldername of this_folder
		log "info for this_folder: " & (this_folder)
	end tell
	
	--  Process each file newly-added to this_folder
	try
		repeat with i from 1 to number of items in these_items
			set this_item to item i of these_items
			set the item_info to the info for this_item
			if (alias of the item_info is false and the type identifier of the item_info is "com.apple.applescript.text") or (the name extension of the item_info is "applescript") then
				tell application "Finder"
					-- LOOK FOR EXISTING MATCHING ITEMS IN THE DESTINATION FOLDER
					-- IF THERE ARE MATCHES, THEN RENAME THE CONFLICTING FILES INCREMENTALLY
					my resolve_conflicts(this_item, target_folder)
					-- MOVE THE ITEM TO THE DESTINATION FOLDER
					set the target_file to (move this_item to the target_folder with replacing) as alias
					
					set target_folderName to the POSIX path of (target_folder as string) as string
					set the scriptInputName to target_folderName & the text of (name of (info for target_file))
					set the scriptInputExtension to "." & (the text of (name extension of (info for target_file)))
					set the scriptTrimName to my trimText(scriptInputName, scriptInputExtension, "end")
					set the scriptOutputName to scriptTrimName  & ".urlencoded"
					
					set the scriptInputName to the quoted form of scriptInputName
					set the scriptOutputName to the quoted form of scriptOutputName
				end tell
				-- PROCESS THE ITEM
				process_item(scriptInputName, scriptOutputName)
			else
				tell application "Finder"
					set myErrorMessage to "Expecting " & scriptTrimName & " to have been exported as Text "
					activate
					display dialog myErrorMessage buttons {"Cancel"} default button 1 giving up after 120
				end tell
			end if
		end repeat
	on error error_message number error_number
		if the error_number is not -128 then
			tell application "Finder"
				activate
				display dialog error_message buttons {"Cancel"} default button 1 giving up after 120
			end tell
		end if
	end try
end adding folder items to

on resolve_conflicts(this_item, target_folder)
	tell application "Finder"
		set the quotedName to quoted form of POSIX path of this_item
		log "quotedName: " & quotedName
		set the file_name to the name of this_item
		if (exists document file file_name of target_folder) then
			set file_extension to the name extension of this_item
			if the file_extension is "" then
				set the trimmed_name to the file_name
			else
				set the trimmed_name to text 1 thru -((length of file_extension) + 2) of the file_name
			end if
			set the name_increment to 1
			repeat
				set the new_name to (the trimmed_name & "." & (name_increment as string) & "." & file_extension) as string
				if not (exists document file new_name of the target_folder) then
					-- rename to conflicting file
					set the name of document file file_name of the target_folder to the new_name
					exit repeat
				else
					set the name_increment to the name_increment + 1
				end if
			end repeat
		end if
	end tell
end resolve_conflicts

-- this sub-routine processes files 
on process_item(scriptInputName, scriptOutputName)
	-- NOTE that the variable this_item is a file reference in alias format 
	-- FILE PROCESSING STATEMENTS GOES HERE 
	try
		with timeout of 3 seconds
			set shellCommand to "echo '<a href = \"applescript://com.apple.scripteditor?action=new&script=' > " & scriptOutputName & " && cat " & scriptInputName & " | python3 -c \"import urllib.parse, sys; print(urllib.parse.quote(sys.stdin.read()))\" >> " & scriptOutputName & " && echo '\">Open in Script Editor</a>' >> " & scriptOutputName
			do shell script shellCommand
		end timeout
	on error error_message
		tell application "Finder"
			activate
			display dialog error_message buttons {"Cancel"} default button 1 giving up after 120
		end tell
	end try
end process_item

on trimText(theText, theCharactersToTrim, theTrimDirection)
	set theTrimLength to length of theCharactersToTrim
	if theTrimDirection is in {"beginning", "both"} then
		repeat while theText begins with theCharactersToTrim
			try
				set theText to characters (theTrimLength + 1) thru -1 of theText as string
			on error
				-- text contains nothing but trim characters
				return ""
			end try
		end repeat
	end if
	if theTrimDirection is in {"end", "both"} then
		repeat while theText ends with theCharactersToTrim
			try
				set theText to characters 1 thru -(theTrimLength + 1) of theText as string
			on error
				-- text contains nothing but trim characters
				return ""
			end try
		end repeat
	end if
	return theText
end trimText
```