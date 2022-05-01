---
title: Debugging Drupal with Xdebug and Atom
date: 2022-04-30 
layout: post
comments: false
categories: ['DevOps', 'Drupal']
---

## Summary. 

[Atom](https://atom.io) is an excellent client to a remote Drupal (with Xdebug 3 + PHP 8.1) and sufficient for my Drupal dev needs. Here's how I set it up.

## Motivation 

I try to avoid programming Drupal custom modules and themes but occasionally it is necessary to dust off the development tools, try to remember what I was doing a year ago when I last had to do this. At which point I find out that my JetBrains subscription to PHPStorm IDE has expired and needs £65 to renew it for a year.  

So I am motivated to get the free of charge tools I already hae installed to do the IDE tasks I need, mainly debugging and inspection using Atom as an Xdebug client

# PHPStorm and Atom  

Atom does the job for me.

PHPStorm is very useful and as I have blogged previously, has been a reliable and useful Xdebug client. Because of the architecture and history of Drupal and the fact that the documentation for this fantastic Open Source project can often be hit and miss, the only way to really understand how to program contrib modules and themes is to explore key data structures and execution paths with an IDE like PHPStorm.  

PHPStorm is a proprietary product which developers find invaluable and worth paying the annual subscription for, especially if they are contributing to Drupal on a daily or frequent schedule, but for the occasional Drupal Developer / DevOP who has to dip in on a less frequent basis, PHPStorm can be more than is needed and cost more than is warrented. It has every conceivable IDE bell and whistle out of the box.

Developed by GitHub and contributors, free of charge and a delight to use is Atom, an editor with additional plug in packages that together give me all the sufficient but effective functionaity I need of an IDE. 

## Objectives 

But in the past I have found Atom difficult to set up as an Xdebug client so I thought that now I have a working setup, I would record what I did in to put together a development setup with the following items notable in the component stack (April 2022)  

*   Drupal 9.3.12  

*   Apache 2.4.41 running with PHP-fpm and APCu cacheing on Ubuntu 20.04 Linux (I couldn't reliably use Xdebug with Nginx)  

*   PHP 8.1 - this requires Xdebug 3 which has different, simpler php.ini settings than previously in Xdebug 2

*   Ansible, Vagrant, and my fork of Jeff Geerling's Drupal-VM used to provision the virtual Dev machine  

*   MacOS Mojave as the host machine in which Atom is the Xdebug client with PHP Xdebug running in the virual dev machine 

*   Parallels Desktop providing the virtualisation for the virtual Ubuntu Linux guest dev machine

**more to follow**