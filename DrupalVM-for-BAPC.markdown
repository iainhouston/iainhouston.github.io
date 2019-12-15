---
layout: page
title:  "Developing and maintaining a Drupal site with Drupal-VM"
date:   2019-12-13
categories:
  - "DevOps"
  - "Drupal"

permalink: /drupalbapc/
---

I'm still working on this. Apologies for yet incomplete bits.

Introduction
============

I used [our fork of Jeff Geerling's Drupal-VM](https://github.com/iainhouston/drupal-vm) to create both a local development Drupal server and a live production server very much along the lines of his tutorial which he wrote for Drupal MidCamp in March 2017. So you might like to go [there](https://www.jeffgeerling.com/blog/2017/soup-nuts-using-drupal-vm-build-local-and-prod) if I have omitted a detail you want to follow.

We use Drupal-VM to keep our *development* and *live* sites' environments exactly in sync.

-  The operating system is at exactly the same level

-  The required software components are exactly the same

-  The configuration of both operating system and software components are the same.

The following describes our use of Drupal-VM using the `refactorvm`
 branch of our GitHub repo  [iainhouston/bradford-abbas.uk](https://github.com/iainhouston/bradford-abbas.uk)

The major section  [Important Drupal-VM configuration files](#VMconfig) below gives further information about our oparticular use of Drupal-VM so that you can see how I've used Drupal-VM to build and maintain [our website here at bradford-abbas.uk](https://bradford-abbas.uk).  



Applicability to other UK Parish Councils
-----------------------------------------

Another Parish Council could fork our GitHub repo and follow the steps below to buid their own website and manage their own Meetings, Agendas, Minutes and other Documents; distribute News Articles; summons Councillors to attend Mettings and so on.  

There is very little (a couple? of Ansible string variables in `vm/*config.yml` with website names) that are peculiar to Bradford Abbas Parish Council. Everything else that follows could be used to build a Parish Council website that conforms to the [Transparency Code for Smaller Authorities](https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/388541/Transparency_Code_for_Smaller_Authorities.pdf) which came into law in 2014.   

Costs
------

Drupal-VM uses only [free, Open Source software](#required_soft). The inescapable costs of our website are:  

*  the small monthly cost of the Amazon EC2 web server;

*  the annual renewal of our domain name;

*  the annual renewal of our SSL Certificate;

*  the annual renewal of our data protection (ICO) certificate which we'd need even if we didn't have a website.

Typical use cases
==================

There are three typical use cases that use  steps described in one or more of the major sections  below.

1.  Most typically, deploying updates to the live server

    Deploying updates to the live server after having applied a security update to a Drupal core or contributed module, or an update we've made to our Drupal theme:

    1.  Set up the local development environment. We only need to do this once on a Mac where we haven't been doing our website's development work before.  

    2.  Do some development and testing.

    3.  Deploy the updated and tested system to the live server.


2. Less typically, switch live servers.  

    This is one that I have done recently to upgrade the live server to the a more recent Linux distro and a later PHP verdsion so as to accomodate a more recent Drupal version.  

    1.  Set up a new live prodction server

    2.  Clone the live server to the local development server

    3.  Do some development

    4. Deploy the updated and tested system to the new live server.  

3.  Theme development: css and html changes

    As part of either of the "Do some development" steps above: use the theme's development toolchain to create a new git-tagged version of `iainhouston/pellucid_monoset`

Set up the local development environment
========

1. Clone this [GitHub repo](https://github.com/iainhouston/drupal-vm) to `~/bradford-abbas.uk`

2. Add the following alias to your `~/.zshrc` (or `~/.bashrc` as appropriate):

  ```sh
  alias cdbadev="cd ~/bradford-abbas.uk && source ./scripts/badev/dev_aliases.sh"
  ```

3. `$ source ~/.zshrc` (or `~/.bashrc` as appropriate)

4. `$ cdbadev` =>

    ```sh
    updateLiveCode - Code and Config to Live site
    cloneLive2Dev  - Clone Live Database and Files to Dev site
    safecex        - Safe export of Dev site's configuration
    endev          - Enable development modules in Dev site
    ```    

    We have written several convenience shell commands to do much of the day-to-day heavy lifting for us. As above, we are reminded of four of them when we switch to our `~/bradford-abbas.uk` directory. `cdbadev` also assigns several key exports and aliases.

5.  Install the libraries and a working Drupal

    `composer install` will download all the libraries. These include the Symfony PHP libraries, upon which Drupal is built; Drupal's core and contributed modules; other non-PHP libraries like our own Ansible rôles and tasks in `iainhouston/drupal-vm`, and our own Drupal theme `iainhouston/pellucid_monoset`

    `composer install` also places the appropriate Drupal contributed and core modules in the appropriate directories in the `web` directory which it creates for us (please ensure you don't start out with `~/bradford-abbas/web` present, or `composer install` will not create the Drupal site.)

    You can reassure yourself that the installation has been successful and will shortly, after we have [fired up the development VM](#fire_up), respond to `http` requests by the presence of `~/bradford-abbas/web/autoload.php` and of the presence of `~/bradford-abbas/web/index.php`.

    `composer install` creates `web/autoload.php` so that running Drupal modules have access to the libraries in `web/../vendor/` via `web/../vendor/autoload.php` and `vendor/composer/autoload_real.php` which  can then load any of the PHP classes etc. it discovers in the `vendor/` packages.

6.  Verify `webmaster`'s access to the Live Server  

    Satisfy yourself that the `Host` settings in your `~/.ssh/config` settings match the host names in `scripts/badev/dev_aliases.sh` and that you have the correct key pair `IdentityFile` provided when [the AWS EC2 Server was set up](#live_init).  Do this by `ssh wpbapc` success.  

    ```
    Host wpbapc
         ForwardX11 no
         User webmaster
         Hostname bradford-abbas.uk
         PreferredAuthentications publickey
         IdentityFile  ~/.ssh/BAPC-2.pem
    ```

    For example,  `cdbadev` does `export LIVE_SSH_ALIAS="wpbapc"` and several scripts in `scripts/badev` refer to `$LIVE_SSH_ALIAS`

    This access is needed in the next step.  

    <a name="fire_up"></a>

7.  Fire up the development VM  

    `vagrant up` will also provision the VM if it does not previously exist, otherwise Vagrant will just boot it up and manage the IP Addresses in `/etc/hosts` and the NFS shared directories in `/etc/exports`

8.  Sync the Live Drupal to the Development Drupal  

    `cloneLive2Dev` clones the live database and the static files (uploaded images, PDFs etc.) to the development Drupal in the VM.

    Drush will not copy/clone between to remote servers. We (the host controller Mac) are dumping the live SQL into a newly-named file in `vm/saved_sql/live` for later use if required, and importing it into `@badev`, the development Drupal.  

    For this reason, too, `cloneLive2Dev`  `rsync`s the static files from `wpbapc`'s access in the live server to the controlling host's `/web/sites/default/files/` rather than using `drush core:rsync`. NFS makes these files immediately available to the development server's VM.  

9.  Make changes to the site's Theme

    *  Verify you have `NodeJS` installed per [Required Software](#required_soft)above

    *  `cdt` will change directories to `web/themes/contrib/pellucid_monoset`

    *  From `web/themes/contrib/pellucid_monoset` do `npm install` to install a local `gulp` (a `NodeJS` based toolchain)

    *  `npx gulp` will use the locally-installed `gulp` to fire up Firefox and  `BrowserSync` and inject CSS into Firefox-rendered pages as gulp watches for changes to various source files and automatically triggers Sass compiles and `drush` cache clears as appropriate.  

    Our Druopal Theme module - `iainhouston/pellucid_monoset` - in `web/themes/contrib/pellucid_monoset` is under version control separately from `iainhouston/bradford-abbas.uk`. A git tag must be pushed to GitHub when a new version is developed to ensure that this latest version is picked up when `composer update iainhouston/pellucid_monoset` is run next in the project folder `~/bradford-abbas.uk`


[//]: # (' Markdown workaround)


Deploy the updated and tested system to the new live server.
===============

The static data files (`sites/default/files/`) and the database on the server are not affected by the first step.


Pushing updated stuff to the live site
-----------------------

`updateLiveCode` is shorthand for:

```sh
DRUPALVM_ENV=prod \
ansible-playbook vendor/iainhouston/drupal-vm/provisioning/playbook.yml \
--inventory-file=vm/inventory \
--tags=drupal   \
--extra-vars="config_dir=$(pwd)/vm" \
--skip-tags=test_only \
--become --ask-become-pass --ask-vault-pass
```

This doesn't re-provision the live server, it does just those playbook tasks - those with `tags: ['drupal']` - required to update the following:

*  Drupal core and contributed modules (via `composer.json`)
*  Our SSL key and certificate
*  Drupal configuration YAML from most recent local  `drush @badev cex` (and thus `safecex`). But note that it doesn\'t run a  `drush @balive cim`

So you run `updateLiveCode` to deploy to the live server after any of these have been updated and tested on the local development site.


Development:
===============


Provisioning the development site
-----------------

I do my development on a Mac but Jeff describes [here](http://docs.drupalvm.com/en/latest/getting-started/installation-windows/) how its done on a Windows 10 machine.

<a name="required_soft"></a>

1. **Required software**

    Our local environment (at the time of writing is shown by  using our helper alias `checkVersions`:    

    ```sh
    PHP 7.4.0 (cli) (built: Nov 29 2019 16:18:44) ( NTS )
    Copyright (c) The PHP Group
    Zend Engine v3.4.0, Copyright (c) Zend Technologies
        with Zend OPcache v7.4.0, Copyright (c), by Zend Technologies
    Composer version 1.9.1 2019-11-01 17:20:17
    Vagrant 2.2.6
    VirtualBox 6.0.14r133895
    ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.x86_64-darwin19]
    ansible 2.9.1
    ...
    NodeJS Version v13.2.0
    npm Version 6.13.1
    vagrant-auto_network (1.0.3, global)
      - Version Constraint: > 0
    vagrant-hostsupdater (1.1.1.160, global)
      - Version Constraint: > 0
    vagrant-vbguest (0.21.0, global)
      - Version Constraint: 0.21
    Developer Edition of Mozilla Firefox 72.0b
    ```    

    Errors ahead?: My experience over several years of using Drupal-VM shows that unexplained provisioning errors can often disappear after you are sure you have upgraded to the latest of each of `ansible`; `vagrant`; and `VirtalBox`.    

2. **Key environment variable**

    If you think you're provisioning a live server rather than a development one, or vice versa, ensure that the `DRUPALVM_ENV` environment variable is correctly set by issuing vagrant commands in the form: `DRUPALVM_ENV=vagrant vagrant up` and `DRUPALVM_ENV=vagrant vagrant provision`. Keep and eye on `echo $DRUPALVM_ENV`: that caught me out.

3. **Drush:**

    Getting `drush` right has taken a lot of my bandwidth over various releases. I now take the approach of using a single, locally installed `drush` that is aliased to the `vendor/bin` directory on the host machine; uses `<project root>/drush/sites` locally for this website's aliases, and, because NFS has the devlopment server accessing exacly the same `drush` binary for execution in both host and guest machines - i.e. in MacOS host and Linux guest VM -  we know we're at exactly the same `drush` relesase: host (a.k.a.controller); dev server; and live server.


Encrypted secrets
========

Managing the Production Server's secrets file
---------------------------

This is where the passwords for Drupal admin, drupal db; and mysql root on the Live Production Server  are encoded. (The development server uses plain text values from `vagrant.config.yml`.)

```
ansible-vault create  vm/prodn_secrets.yml
ansible-vault view  vm/prodn_secrets.yml
ansible-vault edit  vm/prodn_secrets.yml
```

Encrypting SSL key and certificate.
-----------

<a name="enc_ssl"></a>

```
ansible-vault encrypt  vm/certs/SSL.crt
```


The Live Production Server
==========================

<a name="live_init"></a>

Initialising the AWS EC2 live server
-------------------------------

These steps are required before we can run Ansible to automatically provision the live server with all the software we require.

1. Store the one-time-generated security key pair associated the AWS EC2 Server in `~/.ssh/BAPC-2.pem`

2. Enable `root` login on EC2 server. This assumes that we have the AWS EC2 server account's private key in `~/.ssh/BAPC-2.pem` on the Mac

    ```
    ssh ubuntu@remote.server.uk -i ~/.ssh/BAPC-2.pem # from local
    sudo emacs /root/.ssh/authorized_keys # on remote
    ```  

3. On EC2 server: Install Python (and editor).

    Ansible needs this to work its provisioning magic. `sudo apt install python` installs Python 2.7 (and `emacs` if you're not comfortable with `vi`).

4. On EC2 server: remove the preamble  

      remove the preamble before the string `ssh-rsa` in `/root/.ssh/authorized_keys`

5. On local control machine: Create `vars.yml`

      Create `vendor/iainhouston/drupal-vm/examples/prod/bootstrap/vars.yml` per the tutorial creating a new admin account (`webmaster`) on the server with the password recorded in `Vault PW` here on the Mac.

6. On local control machine: run the 'init' playbook.  

    ```
    ansible-playbook -i vm/inventory vendor/iainhouston/drupal-vm/examples/prod/bootstrap/init.yml -e "ansible_ssh_user=root"
    ```

    We should now have created `webmaster` and be able to `ssh webmaster@remote.server.uk` and thence `sudo`  things using the password recorded in `Vault PW` here on the Mac.  

7. On EC2 server:  revert the preamble

    revert the preamble before the string `ssh-rsa` in `/root/.ssh/authorized_keys` to prevent anyone logging into `root` directly.


Provisioning the Production server
==================================

Additional provisioning rôles
-------------------------------------------

There are several Ansible tasks that are peculiar to our setup and are not already provided by  Drupal-VM's roles and tasks:

```
# setup DKIM; Place SSL cert etc.
pre_provision_tasks_dir: "{{ config_dir }}/pre_provision_tasks/*"
post_provision_tasks_dir: "{{ config_dir }}/post_provision_tasks/*"
```

<a name="prodn_provision"></a>

Run the provisioning playbook
-----------

```sh
DRUPALVM_ENV=prod ansible-playbook \
vendor/iainhouston/drupal-vm/provisioning/playbook.yml \
--become --ask-become-pass \
--ask-vault-pass
--inventory-file=vm/inventory \
--extra-vars="config_dir=$(pwd)/vm" \
--skip-tags=test_only \
--become --ask-become-pass --ask-vault-pass
```

Note that we don't use `--tags=drupal`, because, in this case,  we require *all* the Provisioning tasks to be run.

<a name="VMconfig"></a>

Important Drupal-VM configuration files
======================

These are in the following directories:

* The `config` directory  

    `config/sync` is where the Drupal configuration `.yml` files are kept under `git` version control. After you have run `updateLiveCode` you then run `drush @balive cim` to import the new configuration into the live site.  

    New configuration settings can occur both as a result of changes you make to Drupal Content Types etc., and changes to updated / newly installed Drupal Corre and Contributed modules.

    When you run `safecex` then `config/sync` is replaced by configuration `.yml` files exported from the development Drupal's current database into the `git` change managemnt.

* The `drush` directory  

    `drush/sites` contains the drush alias definitions for `@balive` and `@badev`  

* The `scripts` directory

  * `badev` Where the shortcut / convenience commands (see above) are defined.

  * `composer` contains a `composer` initialisation script that will install a basic Drupal site. Very useful. Make sure your `web` directory doesn't exists, though, before you run `composer install`

* The `vm` directory

  * `vm/certs`

    When a new SSL key / certificate pair are downloaded (via LCN, our registrar) they are encrypted here with `ansible-vault` ([see below](#enc_ssl)).

  * `vm/post_provision_tasks` and `vm/pre_provision_tasks`

    As expected, Ansible tasks that extend the functionality of `drupal-vm` for our needs. For example: installing and configuring DKIM's digital 'signing' of emails

  * `vm/templates`  

    `jinja2` templates for files to be customised by Ansible before boing copied into a server during provisioning.

*  The `web` directory

    This is Drupal's *docroot*. Make sure it **doesn't** exist before you run `composer install` as will be the case first thing after having cloned this repo.

    Note that the *Project root* is `.../bradford-abbas.uk` where you cloied this repo.  

*  `composer.json`  

    Updated by all the `composer require ...` and `composer update ...` we've done over the lifetime of the project. It is used both to establish our development and live production Drupal systems and to deploy updates through `git`.   

    In the `require-dev` section we have packages that are not required on the live production server (e.g. `drupal/devl` and `iainhouston/drupal-vm`)

*  `Vagrantfile`

    Note that this loads the `Vagrantfile` in `vendor/iainhouston/drupal-vm` but also sets some important local `ENV`ironmen=t variables including some Ansible argumments that you might have overlooked

* `vendor`

    This is where `composer` installs all the required libraries including the Symfony PHP modules etc. for Drupal, and other non-PHP libraries - including `iainhouston/drupal-vm`  

*  `tmp` and `private_files`  

    These directories are created by Ansible in the project folder - i.e. a level above the Drupal docroot (where the `index.php` sits). Ansible does this in two circumstances.

    1. On the devlopment server as a side-effect of provisioning with `vagrant up` or `vagrant provision`.

    2. On the live production server as a side-effect of provisioning  with `ansible-playbook` (See [Run the provisioning playbook](#prodn_provision))

    These two directories are purely for use by a running Drupal system
