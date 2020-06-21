---
title: Swift Playgrounds and Working Copy
date: 2020-06-21 15-56-00
layout: post
comments: true
categories: ['Programming', 'Blog', 'DevOps']
---


[Working Copy](https://workingcopyapp.com) is a terrific app. It's a lot more than an IOS Git Client.

I am using Working Copy for a number of tasks, and one I am keen to develop more, is doing bits of development of Swift IOS and Mac programs on my iPad Pro as [John Sundell](https://www.swiftbysundell.com/articles/review-swift-playgrounds-30-for-ipad/) seems to do. 

BTW I am writing this post to my iainhouston.github.io account using the built-in editor of Working Copy. Commit; Push; and its published.

The author Anders' [documentation](https://workingcopyapp.com/manual/extending-ios) is very thorough, however I had a few problems when trying to manage commits of my Swift Playground.

No, Swift Playgrounds are not compatible between IOS and Xcode on Mac and Yes, I still have to extract the Swift from the IOS playground bundle to work on it in Xcode but ... hopefully this will improve over time.

***You may find this useful***

So I was floundering around until I discovered that what *does* work for me is this:  

1.  Using an iPad window split between Working Copy and Playgrounds; I Drag and Drop a Playground from Swift Playground’s choosing page into a repo in Working Copy when I want it to be version controlled  
1.  Then, when I want to work on the Swift program: I start from Working Copy  
1.  Right click / long hold on the repo’s highest level  
1.  Choose Files App (Takes me to Working Copy’s files)  
The version-controlled Playground is there with its Swift icon - **that’s reassuring**!  
1.  Double click / tap the Playground’s icon  
1.  Answer "Open Anyway" to Swift Playground’s alert "may have changed"  
1.  Carry on working in the Playground  
1.  Save to Files ("..."  in the RH top corner >> Share >> Save to Files) and see the screenshot below
1.  Return to Working Copy to do a commit  
 
Repeat from 2. above

![Save dialogue from Working Copy](/assets/images/saving_playground.jpeg)