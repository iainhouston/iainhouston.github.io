---
title: Applescript Jekyll assistant (pt 5)
date: 2020-02-03 11-32-27
layout: post
comments: false
categories: ['Scripting']
---

In which we finally get to create our blog post and open it in BBEdit.

Following on from our previous chunk ([Applescript Jekyll assistant (pt 4)
](/scripting/2020/02/02/Applescript_Jekyll_assistant_pt_4.html))

Questions arising from this exercise:  

1.	Was AppleScript the most appropriate tool to achieve our Applescript Jekyll assistant?

2.	Since there was relatively little sending a receiving of Apple Events, and quite a lot of string manipulation, would be have been better off using the Javascript alternative in Script Editor?  

3.	Would we have been better off doing most of our stuff in a Python program and calling it from a minimal AppleScript that mostly dealt with the inter-application Apple Events?

Image of end result with body text selected to edit:  

![Image of end result](/assets/images/Screenshot_2020-02-03_at_11_36.png)


<a href = "applescript://com.apple.scripteditor?action=new&script=
--%20these%20two%20properties%20are%20immutable%20%0Aproperty%20homeFolder%20%3A%20the%20POSIX%20path%20of%20%28path%20to%20home%20folder%29%0Aproperty%20plistPath%20%3A%20the%20POSIX%20path%20of%20%28homeFolder%20%26%20%22.makepost.plist%22%29%0A%28%2A%0A%09We%20keep%20three%20persistent%20settings%20in%20%27makepost.plist%27%3A%0A%09%0A%09%27chosenBlogPath%27%20is%20the%20path%20to%20the%20folder%20where%20Jekyll%20expects%20blog%20posts%20to%20be.%0A%09This%20is%20typically%20the%20%27_posts%27%20subdirectory%20in%20the%20Jekyll%20project%20files.%0A%09%0A%09%27availableCategories%27%20is%20a%20list%20of%20the%20user%27s%20Categories%20from%20which%20the%20user%20can%20choose%0A%09and%20which%20will%20appear%20in%20the%20frontmatter%20of%20the%20newly-to-be-created%20blog%20post%27s%20%27markdown%27%0A%09%0A%09%27chosenScriptFilePath%27%20is%20the%20path%20to%20where%20the%20user%20has%20stored%20%27makepost.py%27%0A%09which%20is%20written%20to%20be%20used%20as%20a%20shell%20command%20for%20creating%20new%20Jekyll%20blog%20posts%0A%09%0A%2A%29%0A--%20These%205%20properties%20of%20our%20Script%20Object%20are%20mutable%0A%0A--%20persistent%20across%20runs%0Aproperty%20chosenBlogPath%20%3A%20the%20POSIX%20path%20of%20homeFolder%0Aproperty%20availableCategories%20%3A%20%7B%22Blog%22%2C%20%22Scripting%22%7D%0Aproperty%20chosenScriptFilePath%20%3A%20the%20POSIX%20path%20of%20%28homeFolder%20%26%20%22bin/makepost.py%22%29%0A%0A--%20not%20persistent%20across%20runs%0Aproperty%20chosenTitle%20%3A%20%22Untitled%22%0Aproperty%20chosenCategories%20%3A%20availableCategories%0A%0A%28%2A%09%0A%09Initialisation%0A%09--%20--%20--%20--%20--%0A%0A%09The%20plist%2C%20and%20thus%20the%20three%20persistent%20settings%20above%20may%20not%20yet%20exist%0A%09in%20which%20case%20we%20have%20bootstrapped%20them%20with%20some%20sensible%20initial%20values.%0A%09%0A%09Notes%3A%0A%09%2A%20AppleScript%20aliases%20have%20to%20refer%20to%20existing%20files%0A%09%0A%09%2A%20POSIX%20files%20and%20paths%20need%20not%20refer%20to%20existing%20files%20or%20directories%0A%09%0A%09%2A%20The%20AppleScript%20path%20of%20a%20POSIX%20path%20is%20%22text%22%0A%2A%29%0A--%20overwite%20our%20bootstrapped%20properties%20if%20the%20user%20already%20has%20a%20.plist%0AinitialisePersistentSettings%28%29%0A%0A%0A%28%2A%0A%09Next%2C%20we%20ask%20the%20user%0A%091.%20What%20the%20title%20of%20her%20blog%20is%20going%20to%20be%3B%0A%09%09and%20from%20this%2C%20we%20will%20derive%20the%20Jekyll-conformant%20file%20name%20for%20the%20post%0A%092.%20Which%20Folder%20she%20uses%20for%20posts%0A%09%09in%20her%20Jekyll-conformant%20Folder%20structure%0A%2A%29%0A--%20overwite%20our%20chosenTitle%20and%20chosenBlogPath%20properties%20as%20the%20user%20chooses%0AchooseTitleAndBlogpath%28%29%0A%0A%0A%28%2A%0A%09Ask%20the%20user%20to%20choose%20which%20blog%20categories%20the%20post%20is%20to%20be%20part%20of.%0A%09getCats%28%29%20will%20deal%20with%20non-existent%20.blogcats%0A%2A%29%0Aset%20chosenCategories%20to%20chooseCategories%28%29%0A%0A%28%2A%0A%09An%20insanity%20check%21%0A%09Always%20worthwhile%21%0A%09...%20for%20prolonged%20sanity%0A%2A%29%0AsanityCheck%28%29%0A%0A%28%2A%0A%09We%20must%20be%20about%20ready%20to%20invoke%20Makepost.py%0A%2A%29%0A--%20on%20shell%20script%20error%3A%20AppleScript%20will%20exit%20after%20having%20reported%20makepost%27s%20STDERR%20text%0Aset%20resultFromMakepost%20to%20do%20shell%20script%20buildshellCommand%28%29%0A%0A--%09edit%20the%20newly-created%20post%20in%20BBEdit%20if%20she%20has%20it%20installed%0Aif%20applicationIsInstalled%28%22BBEdit%22%29%20then%0A%09editInBBEdit%28resultFromMakepost%29%0Aelse%0A%09display%20alert%20%C3%82%0A%09%09%22Your%20blog%20post%20file%20has%20been%20written%20to%20%22%20message%20resultFromMakepost%0Aend%20if%0A%0A%0A--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%0Alog%20%22chosenTitle%3A%20%22%20%26%20chosenTitle%0Alog%20%22chosenBlogPath%3A%20%22%20%26%20chosenBlogPath%0Alog%20%22chosenScriptFilePath%3A%20%22%20%26%20chosenScriptFilePath%0Arepeat%20with%20n%20from%201%20to%20the%20length%20of%20availableCategories%0A%09log%20%22availableCategories%20%28%22%20%26%20n%20%26%20%22%29%3A%20%22%20%26%20item%20n%20of%20availableCategories%0Aend%20repeat%0Arepeat%20with%20n%20from%201%20to%20the%20length%20of%20chosenCategories%0A%09log%20%22chosenCategories%20%28%22%20%26%20n%20%26%20%22%29%3A%20%22%20%26%20item%20n%20of%20chosenCategories%0Aend%20repeat%0Alog%20%22buildshellCommand%3A%20%22%20%26%20buildshellCommand%28%29%0Alog%20%22resultFromMakepost%3A%20%22%20%26%20resultFromMakepost%0A--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%20DEBUG%20--%0A%0A%0A--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%0A%28%2A%0A%09Handlers%20/%20subroutines%0A%2A%29%0A--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20--%20%0A%0Ato%20editInBBEdit%28fileToBeEdited%29%0A%09--%09edit%20the%20newly-created%20post%20in%20BBEdit%20if%20she%20has%20it%20installed%0A%09%0A%09tell%20application%20%22BBEdit%22%0A%09%09activate%0A%09%09open%20POSIX%20file%20chosenBlogPath%20--%20the%20%22project%20window%22%0A%09%09open%20POSIX%20file%20fileToBeEdited%20opening%20in%20project%20window%201%0A%09%09tell%20text%20of%20front%20text%20window%0A%09%09%09find%20%22%5EReplace.%2A%24%22%20options%20%7Bstarting%20at%20top%3Atrue%2C%20search%20mode%3Agrep%2C%20wrap%20around%3Afalse%2C%20backwards%3Afalse%7D%20with%20selecting%20match%0A%09%09end%20tell%0A%09%09%0A%09end%20tell%0A%09%0Aend%20editInBBEdit%0A%0Aon%20applicationIsInstalled%28appRef%29%20--%20%28String%29%20as%20Boolean%0A%09--%20Thanks%20to%20Stackoverflow%20user%20scolfax%0A%09--%20https%3A//stackoverflow.com/questions/14297644/check-if-an-application-exists-using-applescript%0A%09set%20shellScript%20to%20%22osascript%20-e%20%22%20%26%20%28%22id%20of%20application%20%5C%22%22%20%26%20appRef%20%26%20%22%5C%22%22%29%27s%20quoted%20form%20%26%20%22%20%7C%7C%20osascript%20-e%20%22%20%26%20%28%22id%20of%20application%20id%20%5C%22%22%20%26%20appRef%20%26%20%22%5C%22%22%29%27s%20quoted%20form%20%26%20%22%20%7C%7C%20%3A%22%0A%09--%20%09e.g.%3A%20osascript%20-e%20%27id%20of%20application%20%22BBEdit%22%27%20%7C%7C%20osascript%20-e%20%27id%20of%20application%20id%20%22BBEdit%22%27%20%7C%7C%20%3A%09%0A%09--%20%09NB%20The%20command%20to%20the%20right%20of%20the%20%7C%7C%20%28logical%20OR%29%20is%20executed%20only%20when%20the%20first%20command%20fails%0A%09--%09If%20one%20of%20the%20commands%20succeeds%2C%20the%20result%20will%20be%20a%20string%20identifying%0A%09--%09either%20the%20application%20%22BBEdit%22%20or%20the%20application%20id%20%22BBEdit%22%0A%09set%20appBundleId%20to%20do%20shell%20script%20shellScript%0A%09return%20%28appBundleId%20%C2%AD%20%22%22%29%0Aend%20applicationIsInstalled%0A%0Ato%20buildshellCommand%28%29%0A%09--%20%20the%20%60--cryptic%60%20option%20was%20designed%20especially%20for%20us%3A%20%0A%09--%20%60makepost.py%60%20is%20quiet%20except%20for%20printing%20the%20file%20name%20of%20the%20newly-created%20post%0A%09return%20%22python3%20%22%20%26%20chosenScriptFilePath%20%26%20space%20%26%20quoted%20form%20of%20chosenTitle%20%26%20space%20%26%20makeCatArgs%28chosenCategories%29%20%26%20space%20%26%20%22%20%20--cryptic%20%22%0Aend%20buildshellCommand%0A%0Ato%20sanityCheck%28%29%0A%09%0A%09set%20blogFolderPath%20to%20chosenBlogPath%0A%09if%20not%20FolderExists%28chosenBlogPath%29%20then%0A%09%09%0A%09%09--%20this%20shoudn%27t%20happen%20as%2C%20in%20the%20%60choose%20folder%60%20dialogue%2C%20the%20user%20pointed%20to%20an%20existing%20folder%0A%09%09if%20button%20returned%20of%20%28display%20dialog%20%22The%20folder%20of%20blog%20posts%20%22%20%26%20blogFolderPath%20%26%20%22%20does%20not%20%28or%20no%20longer%29%20exists%22%20buttons%20%7B%22Choose%20again%22%2C%20%22Cancel%22%7D%20default%20button%201%20cancel%20button%202%20with%20title%20%22Missing%20folder%20for%20blog%20posts%22%20with%20icon%20caution%29%20is%20%22Choose%20again%22%20then%0A%09%09%09set%20chosenBlogPath%20to%20homeFolder%0A%09%09%09chooseBlogPath%28%29%0A%09%09else%0A%09%09%09error%20%22Error%3A%20The%20folder%20of%20blog%20posts%20%22%20%26%20blogFolderPath%20%26%20%22%20does%20not%20%28or%20no%20longer%29%20exists%22%0A%09%09end%20if%0A%09end%20if%0A%09%0A%09--%20Ensure%20we%20know%20in%20which%20directory%20the%20Python%20script%20%60makepost.py%60%20has%20been%20installed%0A%09%0A%09if%20not%20FileExists%28chosenScriptFilePath%29%20then%0A%09%09%0A%09%09--%20get%20the%20user%20to%20tell%20us%20where%20to%20find%20the%20executable%0A%09%09--%20either%20she%20hasn%27t%20told%20us%20yet%2C%20or%20she%20may%20have%20moved%20it%20since%20last%20use%0A%09%09set%20selectedScriptFilePath%20to%20%C3%82%0A%09%09%09%28choose%20file%20with%20prompt%20%22Please%20locate%20makepost.py%20in%20the%20directory%20where%20it%20has%20been%20installed%22%20of%20type%20%7B%22public.python-script%22%7D%29%0A%09%09set%20chosenScriptFilePath%20to%20the%20POSIX%20path%20of%20selectedScriptFilePath%0A%09%09persistChosenScriptFilePath%28chosenScriptFilePath%29%0A%09end%20if%0A%09%0Aend%20sanityCheck%0A%0Ato%20chooseCategories%28%29%0A%09%0A%09repeat%0A%09%09%0A%09%09choose%20from%20list%20availableCategories%20with%20title%20%22Choose%20Blog%20Categories%22%20with%20prompt%20%22Choose%20one%20or%20more%20Blog%20Categories%22%20OK%20button%20name%20%22Save%22%20cancel%20button%20name%20%22Edit%22%20default%20items%20%7B%22Blog%22%7D%20with%20multiple%20selections%20allowed%0A%09%09%0A%09%09if%20the%20result%20is%20not%20false%20then%0A%09%09%09return%20result%0A%09%09else%0A%09%09%09set%20availableCategories%20to%20editCats%28%29%0A%09%09end%20if%0A%09%09%0A%09end%20repeat%0A%09%0Aend%20chooseCategories%0A%0Ato%20editCats%28%29%0A%09--%20Ask%20the%20user%20to%20edit%20her%20blog%20categories%0A%09%0A%09--%20turn%20the%20list%20availableCategories%20into%20a%20comma-separated%20string%20for%20her%20to%20edit%0A%09set%20availableCatString%20to%20convertListToString%28availableCategories%2C%20%22%2C%22%29%0A%09set%20%7BcatButton%2C%20editedCategories%7D%20to%20%7Bbutton%20returned%2C%20text%20returned%7D%20of%20%28display%20dialog%20%22Add%20and%20delete%20comma-separated%20values.%20Best%20not%20to%20have%20too%20many%21%22%20default%20answer%20availableCatString%20buttons%20%7B%22Save%22%2C%20%22Cancel%22%7D%20default%20button%201%20cancel%20button%202%20with%20title%20%22Edit%20your%20Categories%22%29%0A%09--%20AppleScript%20will%20exit%20if%20user%20presses%20the%20%22Cancel%22%20button%0A%09%0A%09--%20%20just%20in%20case%20we%20have%20a%20zero%20length%20%22list%22%0A%09if%20editedCategories%20is%20%22%22%20then%20set%20editedCategories%20to%20%22Blog%22%0A%09%0A%09--%20sanitise%20the%20new%20category%20%22list%22%20which%20is%20a%20string%0A%09--%20make%20it%20a%20list%0A%09set%20editedCategoriesList%20to%20convertStringToList%28editedCategories%2C%20%22%2C%22%29%0A%09%0A%09--%20remove%20any%20leading%20or%20trailing%20spaces%20of%20each%20list%20item%0A%09set%20trimmedEditedCategoriesList%20to%20trimItems%28editedCategoriesList%29%0A%09%0A%09--%20so%20we%20can%20set%20a%20default%20in%20the%20choose%20cats%20dialogue%0A%09if%20trimmedEditedCategoriesList%20does%20not%20contain%20%22Blog%22%20then%0A%09%09copy%20%22Blog%22%20to%20end%20of%20trimmedEditedCategoriesList%0A%09end%20if%0A%09%0A%09set%20sortedTrimmedEditedCategoriesList%20to%20sortList%28trimmedEditedCategoriesList%29%0A%09%0A%09--%20write%20the%20newly-edited%2C%20and%20sorted%2C%20list%20of%20categories%20to%20the%20plist%0A%09persistCats%28sortedTrimmedEditedCategoriesList%29%0A%09%0A%09return%20sortedTrimmedEditedCategoriesList%0A%09%0Aend%20editCats%0A%0Ato%20convertListToString%28theList%2C%20theDelimiter%29%0A%09set%20AppleScript%27s%20text%20item%20delimiters%20to%20theDelimiter%0A%09set%20theString%20to%20theList%20as%20string%0A%09set%20AppleScript%27s%20text%20item%20delimiters%20to%20%22%22%0A%09return%20theString%0Aend%20convertListToString%0A%0Ato%20convertStringToList%28theText%2C%20theDelimiter%29%20--%20%28String%29%20as%20list%0A%09set%20AppleScript%27s%20text%20item%20delimiters%20to%20theDelimiter%0A%09set%20theListItems%20to%20every%20text%20item%20of%20theText%0A%09set%20AppleScript%27s%20text%20item%20delimiters%20to%20%22%22%0A%09return%20theListItems%0Aend%20convertStringToList%0A%0Ato%20trimItems%28inList%29%0A%09%0A%09set%20outList%20to%20%7B%7D%0A%09repeat%20with%20i%20from%201%20to%20%28count%20%28items%20of%20inList%29%29%0A%09%09set%20trimmedText%20to%20trimText%28%28item%20i%20of%20inList%29%2C%20%22%20%22%2C%20%22both%22%29%0A%09%09if%20trimmedText%20is%20not%20%22%22%20then%0A%09%09%09if%20countInstancesOfItemInList%28outList%2C%20trimmedText%29%20is%200%20then%0A%09%09%09%09copy%20trimmedText%20to%20end%20of%20outList%0A%09%09%09end%20if%0A%09%09end%20if%0A%09end%20repeat%0A%09%0A%09return%20outList%0A%09%0Aend%20trimItems%0A%0Ato%20trimText%28theText%2C%20theCharactersToTrim%2C%20theTrimDirection%29%0A%09set%20theTrimLength%20to%20length%20of%20theCharactersToTrim%0A%09if%20theTrimDirection%20is%20in%20%7B%22beginning%22%2C%20%22both%22%7D%20then%0A%09%09repeat%20while%20theText%20begins%20with%20theCharactersToTrim%0A%09%09%09try%0A%09%09%09%09set%20theText%20to%20characters%20%28theTrimLength%20%2B%201%29%20thru%20-1%20of%20theText%20as%20string%0A%09%09%09on%20error%0A%09%09%09%09--%20text%20contains%20nothing%20but%20trim%20characters%0A%09%09%09%09return%20%22%22%0A%09%09%09end%20try%0A%09%09end%20repeat%0A%09end%20if%0A%09if%20theTrimDirection%20is%20in%20%7B%22end%22%2C%20%22both%22%7D%20then%0A%09%09repeat%20while%20theText%20ends%20with%20theCharactersToTrim%0A%09%09%09try%0A%09%09%09%09set%20theText%20to%20characters%201%20thru%20-%28theTrimLength%20%2B%201%29%20of%20theText%20as%20string%0A%09%09%09on%20error%0A%09%09%09%09--%20text%20contains%20nothing%20but%20trim%20characters%0A%09%09%09%09return%20%22%22%0A%09%09%09end%20try%0A%09%09end%20repeat%0A%09end%20if%0A%09return%20theText%0Aend%20trimText%0A%0Ato%20getCats%28%29%0A%09%0A%09tell%20application%20%22System%20Events%22%0A%09%09%0A%09%09tell%20property%20list%20file%20plistPath%0A%09%09%09set%20availableCategories%20to%20value%20of%20property%20list%20item%20%22catsList%22%0A%09%09end%20tell%0A%09%09%0A%09end%20tell%0A%09%0Aend%20getCats%0A%0Ato%20persistChosenScriptFilePath%28theFilePath%29%0A%09%0A%09tell%20application%20%22System%20Events%22%0A%09%09%0A%09%09tell%20property%20list%20file%20plistPath%0A%09%09%09set%20value%20of%20property%20list%20item%20%22scriptFilePath%22%20to%20theFilePath%0A%09%09end%20tell%0A%09%09%0A%09end%20tell%0A%09%0Aend%20persistChosenScriptFilePath%0A%0Ato%20persistCats%28theCategoriesList%29%0A%09%0A%09tell%20application%20%22System%20Events%22%0A%09%09%0A%09%09tell%20property%20list%20file%20plistPath%0A%09%09%09set%20value%20of%20property%20list%20item%20%22catsList%22%20to%20theCategoriesList%0A%09%09end%20tell%0A%09%09%0A%09end%20tell%0A%09%0Aend%20persistCats%0A%0Ato%20chooseTitleAndBlogpath%28%29%0A%09%0A%09--%20Reduce%20the%20length%20of%20the%20path%20for%20displaying%20as%20the%20legend%20in%20button%201%0A%09set%20pathClue%20to%20crypticPath%28chosenBlogPath%29%0A%09set%20chosenTitle%20to%20%22Untitled%22%0A%09%0A%09--%20We%27ll%20cycle%20the%20path-choosing%2C%20not%20forgetting%20the%20title-choosing%20each%20cycle%2C%20until%20user%20is%20happy%20with%20her%20choice%0A%09repeat%0A%09%09set%20%7BOKButton%2C%20chosenTitle%7D%20to%20%7Bbutton%20returned%2C%20text%20returned%7D%20of%20%28display%20dialog%20%22Choose%20the%20Title%20of%20your%20new%20post%22%20default%20answer%20chosenTitle%20with%20title%20%22Makepost%22%20buttons%20%7BpathClue%2C%20%22Other%22%7D%20default%20button%201%29%0A%09%09%0A%09%09if%20OKButton%20is%20pathClue%20then%0A%09%09%09--%20stay%20with%20the%20blogpath%20that%20we%20read%20from%20user%27s%20settings%20file%0A%09%09%09exit%20repeat%0A%09%09else%0A%09%09%09chooseBlogPath%28%29%0A%09%09%09copy%20crypticPath%28chosenBlogPath%29%20to%20pathClue%0A%09%09end%20if%0A%09%09%0A%09end%20repeat%0A%09%0Aend%20chooseTitleAndBlogpath%0A%0Ato%20chooseBlogPath%28%29%0A%09%0A%09--%20User%20chooses%20her%20%20blog%20posts%20Folder%0A%09set%20chosenBlogPath%20to%20the%20POSIX%20path%20of%20%28choose%20folder%20with%20prompt%20%22Choose%20the%20path%20to%20your%20blog%20posts%20Folder%22%20default%20location%20chosenBlogPath%29%0A%09%0A%09--%20record%20chosenBlogPath%20in%20makepost%27s%20persistent%20settings%0A%09tell%20application%20%22System%20Events%22%0A%09%09tell%20property%20list%20file%20plistPath%0A%09%09%09set%20value%20of%20property%20list%20item%20%22blogPath%22%20to%20chosenBlogPath%0A%09%09end%20tell%0A%09end%20tell%0Aend%20chooseBlogPath%0A%0Aon%20crypticPath%28fullPath%29%20--%20%28String%29%20as%20String%0A%09set%20splitPath%20to%20splitText%28fullPath%2C%20%22/%22%29%0A%09set%20lastBit%20to%20%7B%7D%0A%09copy%20item%20%28%28count%20of%20splitPath%29%20-%202%29%20of%20splitPath%20to%20end%20of%20lastBit%0A%09copy%20%22/%22%20to%20end%20of%20lastBit%0A%09copy%20item%20%28%28count%20of%20splitPath%29%20-%201%29%20of%20splitPath%20to%20end%20of%20lastBit%0A%09%0A%09return%20%28%22Place%20in%20.../%22%20%26%20lastBit%20as%20text%29%20%26%20%22%20%3F%22%0Aend%20crypticPath%0A%0Ato%20splitText%28theText%2C%20theDelimiter%29%0A%09set%20AppleScript%27s%20text%20item%20delimiters%20to%20theDelimiter%0A%09set%20theTextItems%20to%20every%20text%20item%20of%20theText%0A%09set%20AppleScript%27s%20text%20item%20delimiters%20to%20%22%22%0A%09return%20theTextItems%0Aend%20splitText%0A%0A%0Ato%20initialisePersistentSettings%28%29%0A%09%0A%09if%20not%20FileExists%28plistPath%29%20then%0A%09%09%0A%09%09--%20create%20the%20plist%20with%20bootstrap%20settings%20now%2C%20and%20for%20recording%20the%20user%27s%20choices%20later%0A%09%09tell%20application%20%22System%20Events%22%0A%09%09%09%0A%09%09%09--%20Have%20a%20look%20at%20the%20%27Mac%20Automation%20Scripting%20Guide%27%20for%20more%20on%20this%20%27tell%27%20block%0A%09%09%09%0A%09%09%09--%20Create%20an%20empty%20property%20list%20dictionary%20item%0A%09%09%09set%20theParentDictionary%20to%20make%20new%20property%20list%20item%20with%20properties%20%7Bkind%3Arecord%7D%0A%09%09%09%0A%09%09%09--%20Create%20a%20new%20property%20list%20file%20using%20the%20empty%20dictionary%20list%20item%20as%20contents%0A%09%09%09set%20thePropertyListFilePath%20to%20plistPath%0A%09%09%09%0A%09%09%09set%20thePropertyListFile%20to%20make%20new%20property%20list%20file%20with%20properties%20%7Bcontents%3AtheParentDictionary%2C%20name%3AthePropertyListFilePath%7D%0A%09%09%09%0A%09%09%09tell%20property%20list%20items%20of%20thePropertyListFile%0A%09%09%09%09%0A%09%09%09%09--%20use%20bootstrapped%20settings%20of%20these%20properties%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20list%20key%20for%20accessing%20the%20users%27%20blog%20categories%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Alist%2C%20name%3A%22catsList%22%2C%20value%3AavailableCategories%7D%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20string%20key%20for%20accessing%20the%20path%20to%20the%20user%27s%20blog%20posts%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Astring%2C%20name%3A%22blogPath%22%2C%20value%3AchosenBlogPath%7D%0A%09%09%09%09%0A%09%09%09%09--%20Add%20a%20string%20key%20for%20accessing%20the%20path%20to%20the%20python%20script%20%27makepost.py%27%0A%09%09%09%09make%20new%20property%20list%20item%20at%20end%20with%20properties%20%7Bkind%3Astring%2C%20name%3A%22scriptFilePath%22%2C%20value%3AchosenScriptFilePath%7D%0A%09%09%09%09%0A%09%09%09end%20tell%0A%09%09end%20tell%0A%09%09%0A%09else%0A%09%09%0A%09%09--%20overwrite%20the%20bootstrapped%20values%20of%20these%20persistent%20properties%0A%09%09tell%20application%20%22System%20Events%22%0A%09%09%09%0A%09%09%09tell%20property%20list%20file%20plistPath%0A%09%09%09%09set%20chosenBlogPath%20to%20value%20of%20property%20list%20item%20%22blogPath%22%0A%09%09%09%09set%20availableCategories%20to%20value%20of%20property%20list%20item%20%22catsList%22%0A%09%09%09%09set%20chosenScriptFilePath%20to%20value%20of%20property%20list%20item%20%22scriptFilePath%22%0A%09%09%09end%20tell%0A%09%09%09%0A%09%09end%20tell%0A%09%09%0A%09end%20if%0A%09%0Aend%20initialisePersistentSettings%0A%0Aon%20FileExists%28theFile%29%20--%20%28String%29%20as%20Boolean%0A%09--%20Convert%20the%20file%20to%20a%20string%0A%09set%20theFile%20to%20theFile%20as%20string%0A%09tell%20application%20%22System%20Events%22%0A%09%09if%20exists%20file%20theFile%20then%0A%09%09%09return%20true%0A%09%09else%0A%09%09%09return%20false%0A%09%09end%20if%0A%09end%20tell%0Aend%20FileExists%0A%0Aon%20FolderExists%28theFolder%29%20--%20%28String%29%20as%20Boolean%0A%09set%20theFolder%20to%20theFolder%20as%20string%0A%09tell%20application%20%22System%20Events%22%0A%09%09if%20exists%20folder%20theFolder%20then%0A%09%09%09return%20true%0A%09%09else%0A%09%09%09return%20false%0A%09%09end%20if%0A%09end%20tell%0Aend%20FolderExists%0A%0Aon%20sortList%28theList%29%0A%09set%20theIndexList%20to%20%7B%7D%0A%09set%20theSortedList%20to%20%7B%7D%0A%09repeat%20%28length%20of%20theList%29%20times%0A%09%09set%20theLowItem%20to%20%22%22%0A%09%09repeat%20with%20a%20from%201%20to%20%28length%20of%20theList%29%0A%09%09%09if%20a%20is%20not%20in%20theIndexList%20then%0A%09%09%09%09set%20theCurrentItem%20to%20item%20a%20of%20theList%20as%20text%0A%09%09%09%09if%20theLowItem%20is%20%22%22%20then%0A%09%09%09%09%09set%20theLowItem%20to%20theCurrentItem%0A%09%09%09%09%09set%20theLowItemIndex%20to%20a%0A%09%09%09%09else%20if%20theCurrentItem%20comes%20before%20theLowItem%20then%0A%09%09%09%09%09set%20theLowItem%20to%20theCurrentItem%0A%09%09%09%09%09set%20theLowItemIndex%20to%20a%0A%09%09%09%09end%20if%0A%09%09%09end%20if%0A%09%09end%20repeat%0A%09%09set%20end%20of%20theSortedList%20to%20theLowItem%0A%09%09set%20end%20of%20theIndexList%20to%20theLowItemIndex%0A%09end%20repeat%0A%09return%20theSortedList%0Aend%20sortList%0A%0Aon%20countInstancesOfItemInList%28theList%2C%20theItem%29%0A%09set%20theCount%20to%200%0A%09repeat%20with%20a%20from%201%20to%20count%20of%20theList%0A%09%09if%20item%20a%20of%20theList%20is%20theItem%20then%0A%09%09%09set%20theCount%20to%20theCount%20%2B%201%0A%09%09end%20if%0A%09end%20repeat%0A%09return%20theCount%0Aend%20countInstancesOfItemInList%0A%0A%0Aon%20makeCatArgs%28catsList%29%20--%20%28%7BString%7D%20-%3E%20%7BString%7D%0A%09%0A%09set%20resultCats%20to%20%7B%7D%0A%09set%20catPrefix%20to%20%22--category%3D%27%22%0A%09%0A%09repeat%20with%20i%20from%201%20to%20%28count%20%28items%20of%20catsList%29%29%0A%09%09set%20properCat%20to%20%28catPrefix%20%26%20%28item%20i%20of%20catsList%29%20%26%20%22%27%20%22%29%0A%09%09copy%20properCat%20to%20end%20of%20resultCats%0A%09end%20repeat%0A%09%0A%09return%20resultCats%0A%09%0Aend%20makeCatArgs%0A
">Open in Script Editor</a>

```applescript
-- these two properties are immutable 
property homeFolder : the POSIX path of (path to home folder)
property plistPath : the POSIX path of (homeFolder & ".makepost.plist")
(*
	We keep three persistent settings in 'makepost.plist':
	
	'chosenBlogPath' is the path to the folder where Jekyll expects blog posts to be.
	This is typically the '_posts' subdirectory in the Jekyll project files.
	
	'availableCategories' is a list of the user's Categories from which the user can choose
	and which will appear in the frontmatter of the newly-to-be-created blog post's 'markdown'
	
	'chosenScriptFilePath' is the path to where the user has stored 'makepost.py'
	which is written to be used as a shell command for creating new Jekyll blog posts
	
*)
-- These 5 properties of our Script Object are mutable

-- persistent across runs
property chosenBlogPath : the POSIX path of homeFolder
property availableCategories : {"Blog", "Scripting"}
property chosenScriptFilePath : the POSIX path of (homeFolder & "bin/makepost.py")

-- not persistent across runs
property chosenTitle : "Untitled"
property chosenCategories : availableCategories

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
-- overwite our bootstrapped properties if the user already has a .plist
initialisePersistentSettings()


(*
	Next, we ask the user
	1. What the title of her blog is going to be;
		and from this, we will derive the Jekyll-conformant file name for the post
	2. Which Folder she uses for posts
		in her Jekyll-conformant Folder structure
*)
-- overwite our chosenTitle and chosenBlogPath properties as the user chooses
chooseTitleAndBlogpath()


(*
	Ask the user to choose which blog categories the post is to be part of.
	getCats() will deal with non-existent .blogcats
*)
set chosenCategories to chooseCategories()

(*
	An insanity check!
	Always worthwhile!
	... for prolonged sanity
*)
sanityCheck()

(*
	We must be about ready to invoke Makepost.py
*)
-- on shell script error: AppleScript will exit after having reported makepost's STDERR text
set resultFromMakepost to do shell script buildshellCommand()

--	edit the newly-created post in BBEdit if she has it installed
if applicationIsInstalled("BBEdit") then
	editInBBEdit(resultFromMakepost)
else
	display alert Â
		"Your blog post file has been written to " message resultFromMakepost
end if


-- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG --
log "chosenTitle: " & chosenTitle
log "chosenBlogPath: " & chosenBlogPath
log "chosenScriptFilePath: " & chosenScriptFilePath
repeat with n from 1 to the length of availableCategories
	log "availableCategories (" & n & "): " & item n of availableCategories
end repeat
repeat with n from 1 to the length of chosenCategories
	log "chosenCategories (" & n & "): " & item n of chosenCategories
end repeat
log "buildshellCommand: " & buildshellCommand()
log "resultFromMakepost: " & resultFromMakepost
-- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG -- DEBUG --


-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
(*
	Handlers / subroutines
*)
-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

to editInBBEdit(fileToBeEdited)
	--	edit the newly-created post in BBEdit if she has it installed
	
	tell application "BBEdit"
		activate
		open POSIX file chosenBlogPath -- the "project window"
		open POSIX file fileToBeEdited opening in project window 1
		tell text of front text window
			find "^Replace.*$" options {starting at top:true, search mode:grep, wrap around:false, backwards:false} with selecting match
		end tell
		
	end tell
	
end editInBBEdit

on applicationIsInstalled(appRef) -- (String) as Boolean
	-- Thanks to Stackoverflow user scolfax
	-- https://stackoverflow.com/questions/14297644/check-if-an-application-exists-using-applescript
	set shellScript to "osascript -e " & ("id of application \"" & appRef & "\"")'s quoted form & " || osascript -e " & ("id of application id \"" & appRef & "\"")'s quoted form & " || :"
	-- 	e.g.: osascript -e 'id of application "BBEdit"' || osascript -e 'id of application id "BBEdit"' || :	
	-- 	NB The command to the right of the || (logical OR) is executed only when the first command fails
	--	If one of the commands succeeds, the result will be a string identifying
	--	either the application "BBEdit" or the application id "BBEdit"
	set appBundleId to do shell script shellScript
	return (appBundleId ­ "")
end applicationIsInstalled

to buildshellCommand()
	--  the `--cryptic` option was designed especially for us: 
	-- `makepost.py` is quiet except for printing the file name of the newly-created post
	return "python3 " & chosenScriptFilePath & space & quoted form of chosenTitle & space & makeCatArgs(chosenCategories) & space & "  --cryptic "
end buildshellCommand

to sanityCheck()
	
	set blogFolderPath to chosenBlogPath
	if not FolderExists(chosenBlogPath) then
		
		-- this shoudn't happen as, in the `choose folder` dialogue, the user pointed to an existing folder
		if button returned of (display dialog "The folder of blog posts " & blogFolderPath & " does not (or no longer) exists" buttons {"Choose again", "Cancel"} default button 1 cancel button 2 with title "Missing folder for blog posts" with icon caution) is "Choose again" then
			set chosenBlogPath to homeFolder
			chooseBlogPath()
		else
			error "Error: The folder of blog posts " & blogFolderPath & " does not (or no longer) exists"
		end if
	end if
	
	-- Ensure we know in which directory the Python script `makepost.py` has been installed
	
	if not FileExists(chosenScriptFilePath) then
		
		-- get the user to tell us where to find the executable
		-- either she hasn't told us yet, or she may have moved it since last use
		set selectedScriptFilePath to Â
			(choose file with prompt "Please locate makepost.py in the directory where it has been installed" of type {"public.python-script"})
		set chosenScriptFilePath to the POSIX path of selectedScriptFilePath
		persistChosenScriptFilePath(chosenScriptFilePath)
	end if
	
end sanityCheck

to chooseCategories()
	
	repeat
		
		choose from list availableCategories with title "Choose Blog Categories" with prompt "Choose one or more Blog Categories" OK button name "Save" cancel button name "Edit" default items {"Blog"} with multiple selections allowed
		
		if the result is not false then
			return result
		else
			set availableCategories to editCats()
		end if
		
	end repeat
	
end chooseCategories

to editCats()
	-- Ask the user to edit her blog categories
	
	-- turn the list availableCategories into a comma-separated string for her to edit
	set availableCatString to convertListToString(availableCategories, ",")
	set {catButton, editedCategories} to {button returned, text returned} of (display dialog "Add and delete comma-separated values. Best not to have too many!" default answer availableCatString buttons {"Save", "Cancel"} default button 1 cancel button 2 with title "Edit your Categories")
	-- AppleScript will exit if user presses the "Cancel" button
	
	--  just in case we have a zero length "list"
	if editedCategories is "" then set editedCategories to "Blog"
	
	-- sanitise the new category "list" which is a string
	-- make it a list
	set editedCategoriesList to convertStringToList(editedCategories, ",")
	
	-- remove any leading or trailing spaces of each list item
	set trimmedEditedCategoriesList to trimItems(editedCategoriesList)
	
	-- so we can set a default in the choose cats dialogue
	if trimmedEditedCategoriesList does not contain "Blog" then
		copy "Blog" to end of trimmedEditedCategoriesList
	end if
	
	set sortedTrimmedEditedCategoriesList to sortList(trimmedEditedCategoriesList)
	
	-- write the newly-edited, and sorted, list of categories to the plist
	persistCats(sortedTrimmedEditedCategoriesList)
	
	return sortedTrimmedEditedCategoriesList
	
end editCats

to convertListToString(theList, theDelimiter)
	set AppleScript's text item delimiters to theDelimiter
	set theString to theList as string
	set AppleScript's text item delimiters to ""
	return theString
end convertListToString

to convertStringToList(theText, theDelimiter) -- (String) as list
	set AppleScript's text item delimiters to theDelimiter
	set theListItems to every text item of theText
	set AppleScript's text item delimiters to ""
	return theListItems
end convertStringToList

to trimItems(inList)
	
	set outList to {}
	repeat with i from 1 to (count (items of inList))
		set trimmedText to trimText((item i of inList), " ", "both")
		if trimmedText is not "" then
			if countInstancesOfItemInList(outList, trimmedText) is 0 then
				copy trimmedText to end of outList
			end if
		end if
	end repeat
	
	return outList
	
end trimItems

to trimText(theText, theCharactersToTrim, theTrimDirection)
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

to getCats()
	
	tell application "System Events"
		
		tell property list file plistPath
			set availableCategories to value of property list item "catsList"
		end tell
		
	end tell
	
end getCats

to persistChosenScriptFilePath(theFilePath)
	
	tell application "System Events"
		
		tell property list file plistPath
			set value of property list item "scriptFilePath" to theFilePath
		end tell
		
	end tell
	
end persistChosenScriptFilePath

to persistCats(theCategoriesList)
	
	tell application "System Events"
		
		tell property list file plistPath
			set value of property list item "catsList" to theCategoriesList
		end tell
		
	end tell
	
end persistCats

to chooseTitleAndBlogpath()
	
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
	
end chooseTitleAndBlogpath

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
				make new property list item at end with properties {kind:list, name:"catsList", value:availableCategories}
				
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
				set availableCategories to value of property list item "catsList"
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

on FolderExists(theFolder) -- (String) as Boolean
	set theFolder to theFolder as string
	tell application "System Events"
		if exists folder theFolder then
			return true
		else
			return false
		end if
	end tell
end FolderExists

on sortList(theList)
	set theIndexList to {}
	set theSortedList to {}
	repeat (length of theList) times
		set theLowItem to ""
		repeat with a from 1 to (length of theList)
			if a is not in theIndexList then
				set theCurrentItem to item a of theList as text
				if theLowItem is "" then
					set theLowItem to theCurrentItem
					set theLowItemIndex to a
				else if theCurrentItem comes before theLowItem then
					set theLowItem to theCurrentItem
					set theLowItemIndex to a
				end if
			end if
		end repeat
		set end of theSortedList to theLowItem
		set end of theIndexList to theLowItemIndex
	end repeat
	return theSortedList
end sortList

on countInstancesOfItemInList(theList, theItem)
	set theCount to 0
	repeat with a from 1 to count of theList
		if item a of theList is theItem then
			set theCount to theCount + 1
		end if
	end repeat
	return theCount
end countInstancesOfItemInList


on makeCatArgs(catsList) -- ({String} -> {String}
	
	set resultCats to {}
	set catPrefix to "--category='"
	
	repeat with i from 1 to (count (items of catsList))
		set properCat to (catPrefix & (item i of catsList) & "' ")
		copy properCat to end of resultCats
	end repeat
	
	return resultCats
	
end makeCatArgs
```