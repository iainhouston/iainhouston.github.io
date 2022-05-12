---
title: Debugging Drupal with Xdebug and Atom
layout: post
permalink: /atom_xdebug_client/
categories: ['DevOps', 'Drupal']
---  

# Summary
 
Atom's docs and most web articles  still refer  to  __Xdebug 2__ configuration and older versions of PHP rather than the current versions of both.


[Atom](https://atom.io) is an excellent debugging client when connected to a remote Drupal (with __Xdebug 3__ and  __PHP 8.1__) and sufficient for my Drupal dev needs. Here's how I set it up.

This is  *opinionated* advice drawn from my experience with my simple use case. I hope it is useful to you and others with similar needs to mine.

# Motivation

I aim to do the minimum Drupal programming of custom modules and themes that I can, but occasionally it is necessary to dust off the development tools and then try to remember what I was doing those months ago when I last had to.  

So I need to get the tools I already have to do the "IDE" tasks I need, chiefly editing and with some debugging and inspection using Atom as an Xdebug client.

# Objectives

Providing that Xdebug on the dev server is set up with the correct `php.ini` configuration, Atom can be set up as an Xdebug client really easily so I thought that I would record what is needed put to together a development setup with thes following items in my component stack.  

This my current set-up (May 2022)  

## Server-side

*   __Drupal__ 9.3.12  

*   __Apache__ 2.4.41 running with PHP-fpm and APCu cacheing on Ubuntu 20.04 Linux. (In the past I haven't reliably been able to use Xdebug with Nginx and PHP-fpm)  

*   __PHP 8.1__ - (PHP 8.0 and later require Xdebug 3)  

*   __Xdebug 3__ which has different, simpler `php.ini` settings than previously was the case in _Xdebug 2_

## Client-side

*   __MacOS__ (Monterey 12) as the *host* machine in which Atom is the Xdebug client and PHP Xdebug running in the *guest* virtual dev machine

*   __Parallels__ Desktop providing the virtualisation for ...

*   the virtual __Ubuntu__ Linux guest dev machine


So we want to have:

1.  a connection between Xdebug running on the remote testing virtual server and

1.  the Xdebug client running on our host machine as an Atom plugin.

1.  We also want a browser set up with the ability to switch Xdebug step tracing on and off for each http request it makes to the remote server.

# Atom setup   

Atom (developed by GitHub and contributors) is free of charge and a delight to use. It has (contributed) plugin packages that together can provide IDE-like functionality for many Drupal dev tasks.

1.  I assume that you have the latest [Atom](https://atom.io) installed.

1.  Install Atom's PHP-Debug package.  
    You will be asked to install several other dependent packages. I ended up disabling the optional packages  IDE-PHP and ATOM-IDE-UI which caused errors as they seemed to be out of sync with PHP-Debug and didn't seem to offer any extra functionality worthy of spending time investigating the error.  There *is* one dependent package that is required ATOM-DEBUG-UI.
    Note that [PHP-Debug's GitHub page](https://github.atom.io/packages/php-debug) has example `php.ini` settings for _Xdebug 2_. Ignore these. We'll be using _Xdebug 3_ because it is compatible with PHP 8.1 (more on this below)

1.  Configure Atom's PHP-Debug settings.  
    1.  Path mappings. Drupal programs are in the `themes/contrib` and `modules/contrib` subfolders of `/var/www/drupal/web` of the guest-remote VM whereas I am editing them in the host-local subdirectories of `/Users/me/mysite/web` and so the Atom PHP-Debug path mapping is a JSON dictionary expression like this:

        ```json
        [{"remotePath":"/var/www/drupal/web","localPath":"/Users/me/mysite/web"}]
        ```  

     1. Port    
        The *default* for Xdebug 3 is 9003 but I set mine to 9000 on the remote and left Atom's PHP-Debug default 9000 on the client.  

     1. IDE Key.
        In Xdebug 3 this is just another *trigger*. In any case I set PHP-Debug's IDE key as `xdebug-atom` and will specify this as the trigger on the remote VM.  

# Remote testing server setup

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
    xdebug.discover_client_host=True
    xdebug.client_port=9000
    xdebug.log=/tmp/xdebug.log
    ```    

    For good measure you should make `/etc/php/8.1/apache2/conf.d/20-xdebug.ini` identical to `/etc/php/8.1/fpm/conf.d/20-xdebug.ini`

    For a full explanation of these settings look at Xdebug's [Upgrade Guide](https://xdebug.org/docs/upgrade_guide) for Xdebug 2 -> 3 and [Xdebug docs](https://xdebug.org/docs/).

1.  Restart the remote system's web server.  
    AFAIK it is sufficient to `sudo systemctl restart php8.1-fpm` but for good measure `sudo systemctl restart apache2` also.  
    Maybe someone would be kind enough to comment below as to whether restarting Apache is really necessary when using PHP-fpm. I don't think that it is.

# Browser setup  

I use [Firefox Developer Edition](https://www.mozilla.org/en-GB/firefox/developer/) but I know that similar [Xdebug Helper](https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/) extensions exist for Chrome and [for Safari](https://apps.apple.com/us/app/xdebug-key/id1441712067?mt=12).  

1.  Install [Xdebug Helper for Firefox by BrianGilbert](https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/)  

1.  Edit the extension's settings to use the IDE Key `xdebug-atom`

If you mainly want to set breakpoints and inspect variables then I think the concepts of `IDE Key` and `trigger` are somewhat merged in Xdebug 3, so I leave the profile triggers and trace triggers blank.

# Set up breakpoints  

In Atom *"Move the cursor to a line you want to break on and set a breakpoint by pressing `Alt+F9`.
If everything worked correctly, you can now use the various buttons/commands to step through the script."*

The Atom client's *PHP Console* panel indicates when the server is connected. The *PHP Debug* panel lists the breakpoints you have set and allows you to inspect the values of variables at such breakpoints.

# A note on PHPStorm  

I have been a JetBrains subcriber for many years but don't need PHPStorm at this time. PHPStorm is a superb product and very useful and, as I have blogged previously, is not only a useful Xdebug client, but a fully-fledged IDE with extensive and excellent help resources.  

But PHPStorm is not easy to set up for Xdebug. There are several PHPStorm-specific concepts to grasp when configuring it and so, having done this several times over the last decade, I am of the opinion that the learning curve is actually more demanding than that of setting up Xdebug from scratch. So,  you can justify your investment in time and money in relation to the scale of the project in which you and your team are engaged.

