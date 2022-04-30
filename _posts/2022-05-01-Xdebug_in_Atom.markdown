---
title: Debugging Drupal with Xdebug and Atom
date: 2022-05-01 11-32-27
layout: post
comments: false
categories: ['DevOps', 'Drupal']
---

## Motivation. 

I try to avoid programming Drupal custom modules and themes but occasionally it is necessary to dust off the development tools, try to remember what I was doing a year ago when I last had to do this. At which point I find out that my JetBrains subscription to PHPStorm IDE has expired and needs £65 to renew it for a year.  

PHPStorm is very useful and as I have blogged previously, has been a reliable and useful Xdebug client. Because of the architecture and history of Drupal and the fact that the documentation for this fantastic Open Source project can often be hit and miss, the only way to really understand how to program contrib modules and themes is to explore key data structures and execution paths with an IDE like PHPStorm.  

PHPStorm is a proessionslly-dveloped proprietary product which many developers will find invaluable and worth paying the annual subscription for, especially if they are contributing to Drupal on a daily or frequent schedule, but for the occasional Drupal Developer / DevOP who has to dip in on a less frequent basis, PHPStorm can be more than is needed and cost more than is warrented.  

No less professionally developed by GitHub and contributors, free of charge and a delight to use is Atom, an editor with additional plug in packages that together give me all the effective functionaity of an IDE. But in the past I have found Atom difficult to set up as an Xdebug client so I thought that now I have a working setup, I would record what I did in to put together a development setup with the following notable in the component stack (April 2022)  

*   Drupal 9.3.12  

*   Apache 2.4.41 running with PHP-fpm and APCu cacheing on Ubuntu 20.04 Linux (I couldn't reliably use Xdebug with Nginx)  

*   PHP 8.1 - this requires Xdebug 3 which has different, simpler php.ini settings than previously in Xdebug 2

*   Ansible, Vagrant, Parallels Desktop and my fork of Jeff Geerling's Drupal-VM used to provision the virtual Dev machine  

*   MacOS Mojave as the host machine in which Atom is the Xdebug client with Xdebug running in the virual dev machine 

**more to follow**