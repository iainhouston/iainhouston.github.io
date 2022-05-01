---
title: Debugging Drupal with Xdebug and Atom
date: 2022-04-30 
layout: post
comments: false
categories: ['DevOps', 'Drupal']
---

# Summary. 

[Atom](https://atom.io) is an excellent debugging client when connected to a remote Drupal (with Xdebug 3 + PHP 8.1) and sufficient for my Drupal dev needs. Here's how I set it up.

# Motivation 

I try to avoid programming Drupal custom modules and themes but occasionally it is necessary to dust off the development tools, try to remember what I was doing a year ago when I last had to do this. At which point I find out that my JetBrains subscription to PHPStorm IDE has expired and needs £65 to renew it for a year.  

So I am motivated to get the free of charge tools I already have installed to do the IDE tasks I need, chiefly editing and with some debugging and inspection using Atom as an Xdebug client

## PHPStorm and Atom  

Atom does the job for me.

PHPStorm is very useful and as I have blogged previously, has been a reliable and useful Xdebug client. Because of the architecture and history of Drupal and the fact that up-to-date documentation for this fantastic Open Source project, although greatly improved over the last 12 years, can often be hit and miss, the only way to really understand how to program contrib modules and themes is to explore key data structures and execution paths with an IDE (like PHPStorm for example).  

PHPStorm is a proprietary product which developers find invaluable and worth paying the annual subscription for, especially if they are contributing to Drupal on a daily or frequent schedule, but for the occasional Drupal Developer / DevOP who has to dip in on a less frequent basis, PHPStorm can be more than is needed and cost more than is warrented. It has every conceivable IDE bell and whistle out of the box.

Developed by GitHub and contributors, free of charge and a delight to use is Atom, an editor with additional plug in packages that together give me all the sufficient but effective functionaity I need of an IDE. 

# Objectives 

But in the past I have found Atom difficult to set up as an Xdebug client so I thought that now I have a working setup, I would record what I did in to put together a development setup with the following items notable in the component stack (April 2022)  

*   Drupal 9.3.12  

*   Apache 2.4.41 running with PHP-fpm and APCu cacheing on Ubuntu 20.04 Linux (I couldn't reliably use Xdebug with Nginx)  

*   PHP 8.1 - this requires Xdebug 3 which has different, simpler php.ini settings than previously in Xdebug 2

*   Ansible, Vagrant, and my fork of Jeff Geerling's Drupal-VM used to provision the virtual Dev machine  

*   MacOS Mojave as the host machine in which Atom is the Xdebug client with PHP Xdebug running in the virual dev machine 

*   Parallels Desktop providing the virtualisation for the virtual Ubuntu Linux guest dev machine

# Atom setup

1.  I assume that you have the latest [Atom](https://atom.io) installed.

1.  Install Atom's PHP-Debug package.  
    You will be asked to install several other dependent packages. I ended up disabling the optional packages like PHP-IDE which caused errors as it seemed to be out of sync with PHP-Debug and didn't offer me I considered worthy of spending time on investigating the error.  
    Note that [PHP-Debug's GitHub page](https://github.atom.io/packages/php-debug) has example `php.ini` settings for Xdebug 2. Ignore these. We'll be using Xdebug 3 because it is compatible with PHP 8.1 (more on this below)
    
1.  Configure PHP-Debug settings.  
    1.  Path mappings. Drupal programs are in subfolders of `/var/www/drupal/web` of the guest-remote VM whereas I am editing them in the host-local subdirectories of `/Users/iainhouston/bradford-abbas.uk/web` and so the Atom PHP-Debug path mapping is a JSON dictionary expression like mine:
    
        ```json
        [{"remote":"/var/www/drupal/web","local":"/Users/iainhouston/bradford-abbas.uk/web"}]
        ```  
        
     1. Port    
        The default Xdebug 3 is 9003 but I set mine to 9000 on the remote and left PHP-Debug's default 9000 on the client.  
     
     1. IDE Key. 
        In Xdebug 3 this is just another *trigger*. In any case I set PHP-Debug's IDE key as `xdebug-atom` and will specify this as the trigger on the remote VM.   
       
1.  Configure `php.ini` on remote.  

    This is the template I have in my Ansible task. Note the how the `zend_extension file name` is constructed. I will show my actual expansion below
    
***Hmmm jinja { and } are disappearing inGitHub Markdown extension!***
        
```django
; target file /etc/php/8.1/fpm/conf.d/20-xdebug.ini

[XDebug]
zend_extension="{{ php_xdebug_module_path }}/xdebug-{{ php_xdebug_version }}.so"

xdebug.max_nesting_level={{ php_xdebug_max_nesting_level }}

xdebug.mode={{ php_xdebug_mode }}
xdebug.start_with_request={{ php_start_with_request }}
xdebug.trigger_value={{ php_xdebug_trigger_value }}
xdebug.discover_client_host={{ php_xdebug_discover_client_host }}
xdebug.client_port={{ php_xdebug_client_port }}
xdebug.log={{ php_xdebug_log }}
```    

    More to follow (May 1st)      
        

**more to follow**