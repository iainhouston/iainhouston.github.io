---
title: The Significance  of CICS 
layout: post 
permalink: /CICS_Significance/
categories: ['CICS', 'Software Development']
---

I hope IBM will forgive me for reproducing this article they published in 2009 and which is as true now, I expect, as it was then. I was motivated to post this after having met up with my newly rediscovered good friend and first line manager of the RDO project in CICS 1.7, Chris Nix. 

I worked on CICS from 1976 for several years. At some time in 1980? I proposed an addition to the system that allowed users to add, remove, update and delete "resources" such as files, application programs, queues etc. *dynamically*, in other words without having to re-assemble definition tables and recycle the system which was the norm for systems like CICS and IMS at that time. So we developed a CICS *transaction program* we named CEDA for *Dynamic Allocation of resoures* which you might just make out in the screenshot below. We called the projects RDO for *Resource Definition Online* I wrote several thousand lines of PL/S to implement the CEDA transaction. This was the first bit of CICS to be released (in CICS 1.7) that was written in a high level language. Previously all the CICS programs were written in (the wonderful, and anything but Basic) IBM /370 Basic Assembler Language. Later releases contained *CICS Restructure* code in PL/S. Other members of the RDO team were Ken Davies (who did the difficult bits of interprocess syncronisation if I recall), John Stainforth, David Bowdler amongst others.

## CICS: Securing Online Transactions

If asked to list the world’s bestselling software, it is unlikely that many people would be able to name IBM ® CICS ® , the Customer Information Control System. In fact, this revolutionary product ranks as one of the top 35 technologies that shaped the industry, according to Computerworld magazine. It may well be IBM’s best kept secret.

![CICS Screenshot (#1)](/assets/images/CICS_screenshot.jpg)

## At work billions of times every day

CICS applications are necessary for all kinds of businesses. Many financial institutions conduct hundreds of millions of transactions every day using CICS; providing banking, investment, securities brokerage and other services, not to mention ATMs communicating with banks. CICS is at work when delivery drivers or warehouse employees scan a package and communicate tracking information to a customer or website. Systems for online trading, electronic ticketing, payroll, order entry and processing, and retail distribution all rely on CICS.

If you have ever withdrawn money from an ATM, taken out an insurance policy, paid a utility bill or shopped in a large retail store, then you have most likely used CICS. In all likelihood, you probably use CICS every week. And that’s really what CICS is all about—processing the world’s transactions. Securely and reliably. Day in and day out. Phil Manchester’s quote in Personal Computer Magazine sums it up:
*“CICS is probably the most successful piece of software of all time. … It is the mainstay of business computing throughout the world. … Millions of users unknowingly activate CICS every day, and if it were to disappear the world economy would grind to a halt.”*

CICS was initially delivered around the time astronauts first landed on the moon. It was rapidly adopted by many of the world’s largest banking, insurance, telecommunications, retail, manufacturing, utilities and government organizations to run their core applications. More than 40 years after CICS was first made generally available, IBM’s commitment to deliver innovative and reliable CICS-based technologies means the world’s largest enterprises still trust CICS to run the applications that run their businesses today.

*“Forty years young, with at least 40 more years to go, CICS is the backbone of so many companies around the world. When I think of CICS, I think of industry-leading transaction processing. I think of something that is really at the very core of IBM’s value proposition. CICS is clearly one of IBM’s flagship products and a great part of our portfolio.”*  
Steve Mills, Senior Vice President and Group Executive, Software and Systems, IBM July 2009

Before CICS was invented, most application programs used batch processing, which originally involved physically loading batches of punched cards into computers for processing. Over time, this evolved into computer programs processing batches of digital data, usually overnight. CICS provided the capability to process transactions instantaneously; in other words, real-time transactions.

CICS also provided a collection of standard general-purpose programs, which were delivered as functions that customers could include in their own applications. With CICS providing capabilities such as security, recovery and scalability, IT programmers were free to focus on developing applications that advanced their businesses. No longer did each and every programmer have to spend time developing common capabilities that made up the “middle” layer between their applications and the computers on which they ran. The value was immediate: more powerful, more reliable applications, developed more quickly and more easily. This radical concept rapidly became the multi-billion dollar software market segment that is now known as “middleware.” And even today, CICS is a market leader.

The origins of CICS date back to the mid-1960s, when IBM’s Data Processing Division assembled a small team to figure out what needed to be done to improve telephone customer service. Company employees at the time had to look up paper customer records in a filing cabinet to be able to process them. Companies such as Michigan Bell Telephone, for example, wanted to see a customer’s telephone records on a screen while a company representative was talking to the customer on the phone. A feature like that would dramatically improve telephone service.

Ben Riggins, an IBM systems engineer, joined the IBM team with his ideas for a general online customer service application. The news soon spread that IBM had a prototype solution, and queries poured in from nearly every type of industry including airlines, utilities and financial services. Five years later, worldwide responsibility for CICS moved to the IBM development laboratory in Hursley, United Kingdom. Under the direction of Malcolm Beaver, the manager of the Hursley Programming Center at the time, IBM Hursley soon became an increasingly important center of innovation in the IT industry.
In July 1976, just two years after taking over the CICS mission, Ken Davies, a senior IBM programmer, led a team of developers in designing and implementing a major new innovation in the CICS product: the High Level Programming Interface (HLPI). This elegant replacement for the more complex macro-based capabilities allowed programmers to access CICS services and resources more easily. This new method of invoking CICS capabilities meant that writing and enhancing CICS applications, and upgrading the IT infrastructure that ran them, could all be done in a fraction of the time it used to take. Programmers who were untrained in CICS were becoming productive within weeks rather than months. This was revolutionary in the complex software industry of the 1960s, and is still pretty impressive today. HLPI technology remains at the heart of all CICS applications to this day.

Throughout the rapid economic development of the last 40 years, CICS has been the backbone of many of the world’s largest companies; continually evolving to enable these companies to grow to their full potential. Through the opening of the financial markets in the 1980s, the explosion of Internet commerce in the 1990s, and realizing the promise of service-oriented architecture in the 2000s, CICS has led the way.

Back in 1969, the problem for business was how to securely process thousands of transactions per day. Volume has grown exponentially, and the problem now is how to process thousands of transactions per second.
Each and every day, CICS secures tens of billions of online transactions. So next time you go shopping, spare a thought for IBM’s best kept secret.