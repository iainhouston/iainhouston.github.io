---
title: From Swiftmailer to Symfony Mailer
layout: post
permalink: /swift_to_symfony/
categories: ['DevOps', 'Drupal']
---

Drupal's Swiftmailer module is no longer supported as our site's Status Report keeps on reminding us and also that we must switch to Drupal's Symfony Mailer. So what to do?

Summary  
=======

We switched from Drupal's `mailsystem` and `swiftmailer` combination to Drupal's new Symfony Mailer on our Live and Dev servers whilst finding a simple solution to retaining our use use of `mailhog` on our dev system. 

Why is this important to us?  
============================

Our system  deploys several `simplenews` "issues" (distribution lists) in our role of supporting the functioning of our community. 

We send a low volume of emails but each one is important to us as many of our content types fulfil a statuary requirement.   

We don't use any third party mailing system like Mailchimp (although maybe we should at some point) but, for the time being, testing the functionality and appearance of our simplenews issues is important to us.

We were keen to switch our Simplenews and site emails from Mail System and Swiftmailer to Symfony Mailer with as little disruption as possible.

We use mailhog (and thus `mhsendmail`) to capture emails on our dev system running in a Parallels virtual machine provisioned using our fork of [Jeff Geering's Drupal-VM](https://www.drupalvm.com) which has mailhog provisioning built in.

History - and how we abandoned our approach
-------------------------------------------

On our dev system,  when provisioned, configured `php.ini` >> `sendmail_path=/opt/mailhog/mhsendmail` and on our our live system `sendmail_path` specified the default sendmail binary. In this way we could maintain an identical Drupal configuration between dev and live systems thus eliminating potential errors by minimising any configuration differences.

Well, that *was* our approach, but switching to Symfony Mailer whilst retaining our current php.ini settings and choosing SM’s Native Transport produced no errors yet neither did it produce any email output! We tried various combinations of -t -bs -i (mh)sendmail parameters with varying degrees of failure.

`mailhog` was working as mail sent from the Linux `mail` command  correctly invoked `mhsendmail` and messages are routed to the mailhog UI for inspection.

It is necessary to import previously-used `swiftmail` config settings into Symfony Mailer as a step in its [installation](https://www.drupal.org/docs/contributed-modules/symfony-mailer-0/getting-started#s-installation) process. We could verify that we had imported our swiftmailer configuration OK as, since mailhog uses port 1025, we can send our simplenews email messages to mailhog by using Symfony Mailer’s SMTP Transport specifying host 127.0.0.1 and port 1025.

But we could not meet our objective of having both dev and live Symfony Mail configuration use Native Transport. This is the approach we abandoned as it would seem that, in the nature of the `sendmail`-style interaction between the two processes (mailhog and symfony), each  has different expectations from the other.

Our solution
============

We *do* have a workable solution that allows us to trap emails in mailhog in dev and actually send messages via the `sendmail` transport on the live system both using Drupal Symfony Mailer and with the same config settings in both.

How we do it is this: the **default** Transport in the GUI is permanently set to `sendmail` but we also have a (non-default) SMTP Transport configuration set to Host 127.0.0.1 and Port 1025 (mailhog's port).  

So the `sendmail` configuration gets picked up in the Live system however  it gets  overriden by `smtp` in the dev system because we do a config override in our dev's `settings.php` thus:

```php
$config['symfony_mailer.settings']['default_transport'] = 'smtp';
```  

Theming 
=======

To be added shortly.  

Changes to preprocessing function names  
---------------------------------------

And why it was necessary (and possible) to deploy [PHP 8.1's Xdebug 3 with Atom](https://iainhouston.com/atom_xdebug_client/).  

Changes to templates  
--------------------

And the purpose and functionality of Symfony Mailer's *Policy* markup and how they simplify templates?

