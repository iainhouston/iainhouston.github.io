---
layout: page
title:  "Developing and maintaining a our Parish Council website"
date:   2020-11-27
categories:
  - "DevOps"
  - "Drupal"

permalink: /simplerbapc/
---

The three servers: Live, Staging, Development
=============================================

The Development Server
----------------------

This runs on an Ubuntu virtual guest machine on the Mac host.  

The [repo](https://github.com/iainhouston/bradford-abbas.uk) is cloned into the *project directory*.   

When we change to the project directory (`~/bradford-abbas.uk`) we can `vagrant up` which will do two things:  

1.  Provision a virtual development server

    -  Its an Ubuntu Linux machine 
    
    -  It has an (as yet empty) MySQL database for the Drupal CMS  
    
2.  Shares our (host) project directory with the `/var/www/drupalvm` directory of the (guest)  virtual development server.  

This is so that we can use `git` and an IDE / editor like `PHPStorm` to manage changes to the program code and configuration settings in the host development environment whilst, at the same time, this code and settings can be served by Apache HTTP Server ("httpd") as a website.

Although we do sometimes `ssh` into the virtual development machine, it would not be practicable to run an IDE in the Linux virtual environment - it has no GUI - and thus we use  Mac GUI tools to change manage the code and settings on the Mac. 

The (host) project directory is shared with the (virtual guest) web document directory's parent using NFS (Sun's Network File System).


  