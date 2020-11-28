--- 
layout: post  
title:  Overview of the Parish Council website  
categories: ["Parish Councils", "Drupal"]  

permalink: /bapcoverview/
--- 


# The three servers: Live, Staging, Development


-  The *Live Server* is the public-facing web server. We "rent" ours from [SimplyHosting](https://www.simplyhosting.com).  

-  The *Staging Server* is really only used when we have to change servers. Once we can see that we have migrated the code, database, static files and settings from the Live server correctly, the Staging Server becomes the Live Server.

-  The Development Server is where we make changes to modules and settings and test that these can me safely migrated to the Live Server

## The Development Server

This runs on an Ubuntu virtual guest machine within the Mac host.  

The [GitHub repo iainhouston/bradford-abbas.uk](https://github.com/iainhouston/bradford-abbas.uk) is cloned into the *project directory* `~/bradford-abbas.uk`.   
 
When we change to the project directory (`~/bradford-abbas.uk`) we then `vagrant up` which does two things:  

1.  Provisions a virtual development server

    -  Its an Ubuntu Linux machine 
    
    -  It is allocated an IP Address and URL in `/etc/hosts` 
    
    -  It has the latest PHP code from Drupal and  contributed both by us and from others
    
    -  It has an (as yet empty) MySQL database for the Drupal CMS  
    
2.  Shares our (host) project directory with the `/var/www/drupalvm` directory of the (guest)  virtual development server.  

This is so that we can use `git` and an IDE / editor like `PHPStorm` to manage changes to the program code and configuration settings in the host development environment whilst, at the same time, this code and those settings can be served by Apache HTTP Server ("httpd") as a website on the virtual development server.

Although we do sometimes `ssh` into the virtual development machine, it would not be practicable to run an IDE in the Linux virtual environment - it has no GUI installed - and thus we use  Mac GUI tools to change manage the code and settings on the Mac. 

The (host) project directory is shared with the (virtual guest) web document directory's parent using NFS (Sun's Network File System) which is present on both the guest Linux and host MacOS's BSD Unix systems.

### Using the Development Server

As yet - after having just provisioned the Development Server - there is program code but no database or static files. Ignoring the case where we are starting the website from scratch, we need to populate - clone - the Development Server's database and static files from the Live Server.

We do this as follows:  

1.  we run `cdba9dev` which  

    -  `cd`'s us to the project directory (`~/bradford-abbas.uk`)  
    
    -  exports shellvariables and sets aliases for utilty commands  
    
    `cdba9dev` is defined in `~/.zshrc` as follows !!!!  
    
2.  we run `cloneLiveToDev` to populate the Development Server's database and static files from the Live Server.

We can now brows the Development version of the Parish Council website using the URL shown in the commentary generated from the `vagrant up` step above.











