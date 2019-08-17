---
layout: post
title:  "Developing and maintaining a Drupal site with Drupal-VM"
date:   2019-08-17 20:10:41 +0100
categories: devops Drupal 
permalink: /drupalbapc/
---

**Undergoing revision**

**Summary**: In  praise of [Jeff Geerling's Drupal-VM](https://www.drupalvm.com) with which we keep our *development* and *live* sites' environments exactly in sync.

We use [Jeff Geerling's Drupal-VM](https://www.drupalvm.com) to keep our *development* and *live* sites' environments exactly in sync.

-  The operating system is at exactly the same level

-  The required software components are exactly the same

-  The configuration of both operating system and software components are the same.

You can see how I've used Drupal-VM [here at bradford-abbas.uk](https://github.com/iainhouston/bradford-abbas.uk). (What follows is the GitHub repo's REAME file.)

Three convenience shell commands do much of the heavy lifting for us:


```
updateLiveCode - Code and Config to Live site
rsyncp2dfiles  - Get latest files from live site
sqldumpDev     - Get latest SQL from live site
```


Quick start
========

1. Clone the GitHub repo to `~/bradford-abbas.uk`

2. Add the following alias to your `~/.bashrc` (or `~/.bash_aliases` as appropriate):

  ```
  alias cdbadev="cd ~/bradford-abbas.uk && source ./scripts/badev/.dev_aliases"
  ```

3. `$ source ~/.bashrc`

4. `$ cdbadev` =>

  ```
  updateLiveCode - Code and Config to Live site
  rsyncp2dfiles  - Get latest files from live site
  sqldumpDev     - Get latest SQL from live site
  ```

Regular maintenance
===============

Pushing updated stuff to the live site
-----------------------

`updateLiveCode` is shorthand for:

```
DRUPALVM_ENV=prod \
ansible-playbook vendor/geerlingguy/drupal-vm/provisioning/playbook.yml \
--inventory-file=vm/inventory \
--tags=drupal   \
--extra-vars="config_dir=$(pwd)/vm" \
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

  Where the shortcut / convenience commands (see above) are defined.


* The `vm` directory

  * `vm/certs`

    When a new SSL key / certificate pair are downloaded (via LCN, our registrar) they are encrypted here with `ansible-vault` ([see below](#enc_ssl)).

  * `vm/post_provision_tasks`

  * `vm/pre_provision_tasks`

  * `vm/templates`  

      Nginx web server settings for `bradford-abbas.uk` are declared in `vm/templates/nginx-ssl-vhost.j2`

*  The `web` directory

    This is Drupal's *docroot*

*  Project root: `~/bradford-abbas.uk`  

  *  `composer.json`

  *  `Vagrantfile`

Development:
===============

I used Jeff Geerling's Drupal-VM to create a local Drupal server per his [tutorial](https://www.jeffgeerling.com/blog/2017/soup-nuts-using-drupal-vm-build-local-and-prod).

Just a few notes to expand on Jeff's excellent directions:


Provisioning the development site
-----------------

I do my development on a Mac but Jeff describes [here](http://docs.drupalvm.com/en/latest/getting-started/installation-windows/) how its done on a Windows 10 machine.


1. **Required software** Local environment at this time:

    ```
    python --version #==> Python 2.7.14
    ansible --version #==> ansible 2.4.2.0
    vagrant --version #==> Vagrant 2.0.1
    VBoxManage --version #==> Virtual Box 5.2.4r119785
    composer --version  #==> ... 1.6.2 2018-01-05 15:28:4
    ```

    **Errors?** My experience over several years of using Drupal-VM shows that unexplained provisioning errors can often disappear after you are sure you have upgraded to the latest of each of `ansible`; `vagrant`; and `VirtalBox`.

2. **Key environment variable** Ensure that the `DRUPALVM_ENV` environment variable is correctly set by issuing vagrant commands in the form: `DRUPALVM_ENV=vagrant vagrant up` and `DRUPALVM_ENV=vagrant vagrant provision`. Keep and eye on `echo $DRUPALVM_ENV`: that caught me out.

3. **Drush:**
  Getting `drush` right took a lot of my bandwidth as older versions  and the latest Drupal 8.4 have different dependencies on Symfony packages. So I really had to persist with `composer` etc. to get a working vagrant-based development setup.   

  I am using drush 9.0.0-rc2. I encountered problems with both 8.1.15 and 9.0.0-rc2, but suppose Moshe Weitzman will be fixing 9.0.0-rc2 (I raised issues for some of the following)  

  1. Run these commands *inside* the vagrant VM (or prod server) - regardless of whether you're using `drush` 8.1.15 or ~9.0  - only one of the aliases can be remote when running the `(sql-|r)sync` command. e.g.:  

    ```
    vagrant ssh
    drush rsync  @balive:%files @self:%files --exclude-paths=sync:css:js:php
    drush sql-sync  @balive @self
    drush @self updb
    ```

    Note that - at this time - neither of the above commands work properly.

    But ...  

  2. **Workarounds:** Right now drush 9 (9.0.0-rc2) doesn't properly rsync to the correct destination (`drush rsync  @balive:%files @self:%files`) and `drush sql-dump  @balive` produces extraneous text in the SQL dump.  

    *  Since `drush rsync` is not working as it used to, I am doing  

       ```
       rsync -avz wpbapc:/var/www/drupal/web/sites/default/files/ \
       ./web/sites/default/files/ \
       --exclude=js --exclude=php --exclude=css
       ```  

    *  Edit the output of `drush sql-dump  @balive > tmp.sql` to remove extraneous text string.   

    * Issues raised on Github [`rsync`](https://github.com/drush-ops/drush/issues/3306) and [`sql-sync`](https://github.com/drush-ops/drush/issues/3305)

  3. **Drush Launcher:**  
    To use `drush` globally (at `drush:~9.0`) we install the
    [Drush Launcher](https://github.com/drush-ops/drush-launcher)
    instead of the  `drush` php executable which is locally installed
    in the `vendor` directory.  (See [next section](#dlaunch))


<a name=dlaunch>Drush Launcher</a>
---------

In `vm/config.yml` we declare the *launcher* as the executable rather than the `drush` executable itself

```
drush_launcher_version: "0.5.0"
drush_phar_url: https://github.com/drush-ops/drush-launcher/releases/download/{{ drush_launcher_version }}/drush.phar
```

It's pretty much as simple as that. OK, well, we also needed to convert
our alias files into `.yml`. The conversion command didn't work
properly. For example, the ssh options were  not as described
[on the drush github repo](https://github.com/drush-ops/drush/blob/master/examples/example.site.yml).

**Rationale:** I had unexplained errors with `drush/drush:8.1.15`
so switched to `drush/drush:~9.0`.
This means that I also required `ansible` to install  the
[Drush Launcher](https://github.com/drush-ops/drush-launcher).  

Provisioning
========

From Development to Production
--------------

Again; I followed Jeff's blog. However, if you want to go back to provisioning a local server after you've provisioned your remote prod server, you'll have to account for the presence of any files you encrypted with `ansible-vault`.

Once we have created and encrypted a `secrets.yml` file for production; should we wish to reprovision our vagrant development server, we will need to supply our vault password to Ansible which vagrant will only do after we have added this to our delegate `Vagrantfile` located in the project root.

```
ENV['DRUPALVM_ANSIBLE_ARGS'] = '--ask-vault-pass'
```


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

2. On EC2 server: Install Python (and editor). Ansible needs this to work its provisioning magic. `sudo apt install python` installs Python 2.7 (and `emacs` if you're not comfortable with `vi`).

3. On EC2 server: remove the preamble before the string `ssh-rsa` in `/root/.ssh/authorized_keys`

4. On local control machine: Create `vendor/geerlingguy/drupal-vm/examples/prod/bootstrap/vars.yml` per the tutorial creating a new
admin account (`webmaster`) on the server with the password recorded in `Vault PW` here on the Mac.

5. On local control machine: run the 'init' playbook.

  ```
  ansible-playbook -i vm/inventory vendor/geerlingguy/drupal-vm/examples/prod/bootstrap/init.yml -e "ansible_ssh_user=root"
  ```

  We should now have created `webmaster` and be able to `ssh webmaster@remote.server.uk` and thence `sudo`  things using the password recorded in `Vault PW` here on the Mac.

6. On EC2 server: revert the preamble before the string `ssh-rsa` in `/root/.ssh/authorized_keys` to prevent anyone logging into `root` directly.

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

To use an extra ansible task file, configure the path to the file (relative to `provisioning/playbook.yml`).

Run the provisioning playbook
-----------

```
DRUPALVM_ENV=prod ansible-playbook \
-i vm/inventory vendor/geerlingguy/drupal-vm/provisioning/playbook.yml \
-e "config_dir=$(pwd)/vm" \
--become --ask-become-pass \
--ask-vault-pass
```


A note about provisioning a staging server
--------------------------------

Couldn't make Drupal VM deal with `DRUPALVM_ENV=staging` and `staging.inventory` etc. so we're sticking to a `DRUPALVM_ENV=prod`-only approach and will switch staging->live in due course either by provisioning a new EC2 server or just reprovisioning this one after having changed relevant `inventory` settings and `prod.config.yml` and DNS settings.
