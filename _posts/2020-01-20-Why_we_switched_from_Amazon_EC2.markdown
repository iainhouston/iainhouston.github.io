---
title: Why we switched from Amazon EC2
date: 2020-01-20 10-35-53
layout: post
comments: true
categories: [ 'DevOps']
---

Why we switched from Amazon EC2
===============

Summary
--------

Everything on our newly EC2-hosted website was singing along merrily when ... the complete subnet, of which our Elastic IP address was a member, was blacklisted by UCEPROTECTL3. According to their website, our IP address was not <em>individually</em> blacklisted, but because someone else in this subnet had issued 10's of thousands of spam emails, we got to be blacklisted. I didn't think this was what I was signing up for when I opened our Amazon AWS account.  

Effects
-------

More specifically; all our outgoing email connections were being timed out because we are blacklisted. We rely upon emails to communicate between all the Councillors in our Council.

Outcome
-------

I have now switched to a UK-based provider (LCN) with immediate perceivable benefits to our email delivery (small in volume but important to us).

Re answer from Amazon (See below): in my experience Amazon's statement is just not true. We had many hours of delays to Apple, Gmail and Yahoo! recipients, and eventually messages were in the postqueue for more than a day before I took the decision to switch suppliers. We just couldn't rely that messages would be delivered in a timely fashion. 

So; not my experience: => "This will have no major impact to any large ISP and minimal to no impact to small to mid-size ISPs and recipient domains." Hmmmm.

Quote from Amazon
------------------

*AWS is currently aware of an issue in which the UCEProtect blacklist has placed a section of Amazon IPs into the level 3 category. This is resulting in emails originating from an AWS IP as spam for b2b recipient domains who use the UCEProtect blacklist feed as part of their filtering. This will have no major impact to any large ISP and minimal to no impact to small to mid-size ISPs and recipient domains. We are working with UCEProtect to correct this situation.*
