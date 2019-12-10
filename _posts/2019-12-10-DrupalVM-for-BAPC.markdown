---
layout: post
title:  "Developing and maintaining a Drupal site with Drupal-VM"
date:   2019-12-10
categories: devops Drupal
permalink: /drupalbapc/
---

We use our fork of [Jeff Geerling's Drupal-VM](https://www.drupalvm.com) to keep our *development* and *live* sites' environments exactly in sync.

-  The operating system is at exactly the same level

-  The required software components are exactly the same

-  The configuration of both operating system and software components are the same.

You can see how I've used Drupal-VM [here at bradford-abbas.uk](https://github.com/iainhouston/bradford-abbas.uk).

Three convenience shell commands do much of the day-to-day heavy lifting for us:


```sh
√ bradford-abbas.uk % cdbadev
updateLiveCode - Code and Config to Live site
cloneLive2Dev  - Clone Live Database and Files to Dev site
safecex        - Safe export of Dev site's configuration
endev          - Enable development modules in Dev site
```


Quick start
========

1. Clone this [GitHub repo](https://github.com/iainhouston/drupal-vm) to `~/bradford-abbas.uk`

2. Add the following alias to your `~/.bashrc` (or `~/.zshrc` as appropriate):

  ```sh
  alias cdbadev="cd ~/bradford-abbas.uk && source ./scripts/badev/dev_aliases.sh"
  ```

3. `$ source ~/.bashrc` (or `~/.zshrc` as appropriate)

4. `$ cdbadev` =>

  ```sh
updateLiveCode - Code and Config to Live site
cloneLive2Dev  - Clone Live Database and Files to Dev site
safecex        - Safe export of Dev site's configuration
endev          - Enable development modules in Dev site
  ```

Regular maintenance
===============

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
*  Drupal configuration YAML from most recent  `drush @badev cex`   
   *but note that it doesn't run a*  `drush @balive cim`

So you run `updateLiveCode` when any of these have been updated and tested on the local development site.

Important configuration files
======================

These are in the following directories:

* The `config` directory

    This is where the Drupal configuration `.yml` are kept under `git` version control. After you have run `updateLiveCode` you then run `drush @balive cim` to import the new configuration into the live site.

* The `drush` directory

    *  `drush/sites` contains the drush alias definitions for `@balive` and `@badev`  


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

*  Note that the *Project root* is `.../bradford-abbas.uk` where you cloined this repo.  

  *  `composer.json`

  *  `Vagrantfile`

  Note that this loads the `Vagrantfile` in `vendor/iainhouston/drupal-vm` but also sets some important local `ENV`ironmen=t variables including some Ansible argumments that you might have overlooked

* `vendor`

  This is where `composer` installs all the required libraries including the Symfony PHP modules etc. for Drupal, and other non-PHP libraries - including `iainhouston/drupal-vm`

Development:
===============

I used our fork of Jeff Geerling's Drupal-VM to create a local Drupal server per his [tutorial](https://www.jeffgeerling.com/blog/2017/soup-nuts-using-drupal-vm-build-local-and-prod).

Just a few notes to expand on Jeff's excellent directions:


Provisioning the development site
-----------------

I do my development on a Mac but Jeff describes [here](http://docs.drupalvm.com/en/latest/getting-started/installation-windows/) how its done on a Windows 10 machine.


1. **Required software**

  Our local environment (at the time of writing is shown by  using our helper alias `checkVersions`:

    ```sh
    √ bradford-abbas.uk % checkVersions
    PHP 7.4.0 (cli) (built: Nov 29 2019 16:18:44) ( NTS )
    Copyright (c) The PHP Group
    Zend Engine v3.4.0, Copyright (c) Zend Technologies
        with Zend OPcache v7.4.0, Copyright (c), by Zend Technologies
    Composer version 1.9.1 2019-11-01 17:20:17
    Vagrant 2.2.6
    VirtualBox 6.0.14r133895
    ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.x86_64-darwin19]
    ansible 2.9.1
      config file = None
      configured module search path = ['/Users/iainhouston/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
      ansible python module location = /usr/local/Cellar/ansible/2.9.1/libexec/lib/python3.7/site-packages/ansible
      executable location = /usr/local/bin/ansible
      python version = 3.7.5 (default, Nov 30 2019, 08:22:32) [Clang 11.0.0 (clang-1100.0.33.12)]
    NodeJS Version v13.2.0
    npm Version 6.13.1
    vagrant-auto_network (1.0.3, global)
      - Version Constraint: > 0
    vagrant-hostsupdater (1.1.1.160, global)
      - Version Constraint: > 0
    vagrant-vbguest (0.21.0, global)
      - Version Constraint: 0.21
    Developer Edition of Mozilla Firefox 72.0b1
    ```

    **Errors?** My experience over several years of using Drupal-VM shows that unexplained provisioning errors can often disappear after you are sure you have upgraded to the latest of each of `ansible`; `vagrant`; and `VirtalBox`.    

2. **Key environment variable**

    If you think you're provisioning a live server rather than a development one, or vice versa, ensure that the `DRUPALVM_ENV` environment variable is correctly set by issuing vagrant commands in the form: `DRUPALVM_ENV=vagrant vagrant up` and `DRUPALVM_ENV=vagrant vagrant provision`. Keep and eye on `echo $DRUPALVM_ENV`: that caught me out.

3. **Drush:**

    Getting `drush` right has taken a lot of my bandwidth over various releases. I now take the approach of using a single, locally intalled `drush` that is aliased to the `vendor/bin` directory on the host machine; uses `drush/sites` for this website's aliases, and, because NFS has the devlopment server accessing exacly the same `drush` binary for execution in host and guest machines i.e. in MacOS host and Linux guest.


Provisioning
========

Managing the secrets file
---------------------------

This is where the passwords for drupal admin, drupal db; and mysql root;  are encoded.

```
ansible-vault create  vm/secrets.yml
ansible-vault view  vm/secrets.yml
ansible-vault edit  vm/secrets.yml
```

Encrypting SSL key and certificate.</a>
-----------

<a name="enc_ssl">Like this:</a>

```
ansible-vault encrypt  vm/certs/SSL.crt
```


Initialising the AWS EC2 live server
-------------------------------

This step is required before we can provision the live server with all the software we require.

1. Enable `root` login on EC2 server. This assumes that we have the AWS EC2 server account's private key in `~/.ssh/BAPC-2.pem` on the Mac

    ```
    ssh ubuntu@remote.server.uk -i ~/.ssh/BAPC-2.pem # from local
    sudo emacs /root/.ssh/authorized_keys # on remote
    ```  

2. On EC2 server: Install Python (and editor).

    Ansible needs this to work its provisioning magic. `sudo apt install python` installs Python 2.7 (and `emacs` if you're not comfortable with `vi`).

3. On EC2 server: remove the preamble  

      remove the preamble before the string `ssh-rsa` in `/root/.ssh/authorized_keys`

4. On local control machine: Create `vars.yml`

      Create `vendor/iainhouston/drupal-vm/examples/prod/bootstrap/vars.yml` per the tutorial creating a new admin account (`webmaster`) on the server with the password recorded in `Vault PW` here on the Mac.

5. On local control machine: run the 'init' playbook.  

    ```
    ansible-playbook -i vm/inventory vendor/iainhouston/drupal-vm/examples/prod/bootstrap/init.yml -e "ansible_ssh_user=root"
    ```

    We should now have created `webmaster` and be able to `ssh webmaster@remote.server.uk` and thence `sudo`  things using the password recorded in `Vault PW` here on the Mac.  

6. On EC2 server:  revert the preamble

    revert the preamble before the string `ssh-rsa` in `/root/.ssh/authorized_keys` to prevent anyone logging into `root` directly.


Provisioning the Production server
==================================

If there are Ansible tasks that we need and are not already covered in  Drupal-VM's roles and tasks:

Add any additional tasks not in Drupal VM's provisioning roles and tasks
-------------------------------------------

In `prod.config.yml` we added ...

```
# setup DKIM; Place certs etc.
# path relative to current playbook.yml
pre_provision_tasks_dir: "{{ config_dir }}/pre_provision_tasks/*"
post_provision_tasks_dir: "{{ config_dir }}/post_provision_tasks/*"
```

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
