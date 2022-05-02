---
title: Debugging Drupal with Xdebug and Atom
layout: post
permalink: /atom_xdebug_client/
categories: ['DevOps', 'Drupal']
---

### Summary. 

[Atom](https://atom.io) is an excellent debugging client when connected to a remote Drupal (with Xdebug 3 + PHP 8.1) and sufficient for my Drupal dev needs. Here's how I set it up.  

This is admitedly *opinionated* in the sense that it intentionally focuses on my particular simple uses case and in light of most web articles still relating to Xdebug 2 and older versions of PHP.  

I hope it is useful to you and others with similar needs to mine.

### Motivation 

I try to avoid programming Drupal custom modules and themes if I can, but occasionally it is necessary to dust off the development tools, then try to remember what I was doing a year ago when I last had to do this. At which point I find out that my JetBrains subscription to PHPStorm IDE has expired and needs £65 to renew it for a year.  

So I am motivated to get the free of charge tools I already have installed to do the IDE tasks I need, chiefly editing and with some debugging and inspection using Atom as an Xdebug client

## PHPStorm and Atom  

Atom does the job for me.

PHPStorm is very useful and as I have blogged previously, has been a reliable and fully-featured Xdebug client. Because of the architecture and history of Drupal and the fact that up-to-date documentation for this fantastic Open Source project, although greatly improved over the last 12 years, can often be hit and miss or just very low level, then often the only way to really understand how to program contrib modules and themes is to explore key data structures and execution paths with an IDE (like PHPStorm for example).  

So PHPStorm is a proprietary product which developers find invaluable and worth paying the annual subscription for, especially if they are contributing to Drupal on a daily or frequent schedule, but for the occasional Drupal Developer / DevOP who has to dip in on a less frequent basis, PHPStorm can be more than is needed and cost more than is warranted. It has every conceivable IDE bell and whistle out of the box plus extensive and excellent help resources.  

But PHPStorm is not easy to set up for Xdebug. There are many PHPStorm-specific concepts to grasp when configuring it and so, having done this several times over the last decade, I am of the opinion that the learning curve for a passable understanding of PHPStorm's execution model and its integrated value-added and layers and mighty IDE facilities is actually more demanding than that of setting up Xdebug from scratch. Unless, of course you can justify this investment in time and money in relation to the scale of the project in which you and your team are engaged.

Developed by GitHub and contributors, free of charge and a delight to use is Atom, a really first class editor with additional plug in packages that together give me   sufficient and effective functionaity that I need of an IDE. 

### Objectives 

Providing that Xdebug on the dev server is set up with the correct `php.ini` configuration, Atom can be set up as an Xdebug client really easily so I thought that now I have a working system, I would record what is needed put to together a development setup with these following items of note in my component stack (May 2022)  

# Server-side

*   Drupal 9.3.12  

*   Apache 2.4.41 running with PHP-fpm and APCu cacheing on Ubuntu 20.04 Linux. (In the past I haven't reliably been able to use Xdebug with Nginx and PHP-fpm)  

*   PHP 8.1 - (PHP 8.0 and later require Xdebug 3)  

*   Xdebug 3 which has different, simpler `php.ini` settings than previously in Xdebug 2

# Client-side

*   Ansible, Vagrant, and my fork of Jeff Geerling's Drupal-VM which I use to provision both the Live server and the virtual Dev machine.  

*   MacOS (Monterey 12) as the *host* machine in which Atom is the Xdebug client and PHP Xdebug running in the *guest* virtual dev machine 

*   Parallels Desktop providing the virtualisation for the virtual Ubuntu Linux guest dev machine


So we want to have:

1.  a connection between Xdebug running on the remote testing virtual server and 

1.  the Xdebug client running on our host machine as an Atom plugin. 

1.  We also want a browser set up with the ability to effectively switch Xdebug step tracing on and off for each http request it makes to the remote server.

## Atom setup. 

1.  I assume that you have the latest [Atom](https://atom.io) installed.

1.  Install Atom's PHP-Debug package.  
    You will be asked to install several other dependent packages. I ended up disabling the optional packages like PHP-IDE which caused errors as it seemed to be out of sync with PHP-Debug and didn't offer me any extra functionality of spending time on investigating the error.  (There *is* one dependent package that is required.)
    Note that [PHP-Debug's GitHub page](https://github.atom.io/packages/php-debug) has example `php.ini` settings for Xdebug 2. Ignore these. We'll be using Xdebug 3 because it is compatible with PHP 8.1 (more on this below)
    
1.  Configure Atom's PHP-Debug settings.  
    1.  Path mappings. Drupal programs are in subfolders of `/var/www/drupal/web` of the guest-remote VM whereas I am editing them in the host-local subdirectories of `/Users/iainhouston/bradford-abbas.uk/web` and so the Atom PHP-Debug path mapping is a JSON dictionary expression like mine:
    
        ```json
        [{"remote":"/var/www/drupal/web","local":"/Users/iainhouston/bradford-abbas.uk/web"}]
        ```  
        
     1. Port    
        The *default* for Xdebug 3 is 9003 but I set mine to 9000 on the remote and left Atom's PHP-Debug default 9000 on the client.  
     
     1. IDE Key. 
        In Xdebug 3 this is just another *trigger*. In any case I set PHP-Debug's IDE key as `xdebug-atom` and will specify this as the trigger on the remote VM.  
        
## Remote testing server setup
       
1.  Install Xdebug on the remote / guest VM  
    I used `sudo apt install php8.1-xdebug`
       
1.  Configure `php.ini` on remote. 
    `apt install` will have created `/etc/php/8.1/fpm/conf.d/20-xdebug.ini` containing only `zend_extension=xdebug.so`

    Edit `/etc/php/8.1/fpm/conf.d/20-xdebug.ini` as follows:  
        
    ```ini
    ; file /etc/php/8.1/fpm/conf.d/20-xdebug.ini
    
    [XDebug]
    zend_extension=xdebug.so
    
    xdebug.mode=debug
    xdebug.start_with_request=trigger
    xdebug.trigger_value=xdebug-atom
    xdebug.discover_client_host=true
    xdebug.client_port=9000
    xdebug.log=/tmp/xdebug.log
    ```    
    
    For good measure you should make `/etc/php/8.1/apache2/conf.d/20-xdebug.ini` identical to `/etc/php/8.1/fpm/conf.d/20-xdebug.ini`
    
    For a full explanation of these settings look at Xdebug's [Upgrade Guide](https://xdebug.org/docs/upgrade_guide) for Xdebug 2 -> 3 and [Xdebug docs](https://xdebug.org/docs/).

1.  Restart the remote system's web server.  
    AFAIK it is sufficient to `sudo systemctl restart php8.1-fpm` but for good measure `sudo systemctl restart apache2` also.  
    Maybe someone would be kind enough to comment below as to whether restarting Apache is really necessary when using PHP-fpm. I don't think that it is.
    
## Browser setup  

I use Firefox Developer Edition but I know that similar [Xdebug Helper](https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/) extensions exist for Chrome and [for Safari](https://apps.apple.com/us/app/xdebug-key/id1441712067?mt=12).  

1.  Install [Xdebug Helper for Firefox by BrianGilbert](https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/)  

1.  Edit the extension's settings to use the IDE Key `xdebug-atom`

If you mainly want to set breakpoints and inspect variables then I think the concepts of `IDE Key` and `trigger` are somewhat merged in Xdebug 3, so I leave the profile triggers and trace triggers blank.

## Set up breakpoints  

*"Move the cursor to a line you want to break on and set a breakpoint by pressing `Alt+F9`. 
If everything worked correctly, you can now use the various buttons/commands to step through the script."*

The Atom client's *PHP Console* indicates when the server is connected. The *Debug* panel lists the breakpoints you have set and allows you to inspect the values of all variables at such breakpoints.

**Caveat:** I need to double-check the <span style="text-decoration: underline;">exact</span> configuration values when I get back from holiday but I think they are pretty much correct although you'd be well advised to check the [Xdebug docs](https://xdebug.org/docs/) in the meantime. (May 1st 2022)