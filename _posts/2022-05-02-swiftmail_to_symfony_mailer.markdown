---
title: From Swiftmailer to Symfony Mailer
layout: post
permalink: /swift_to_symfony/
categories: ['DevOps', 'Drupal']
---

Drupal's Swiftmailer module is no longer supported as our site's Status Report reminds us and also tells us that we must switch to Drupal's Symfony Mailer. So what to do?

Summary  
=======

We switched from Drupal's `mailsystem` and `swiftmailer` combination to Drupal's new [Symfony Mailer](https://www.drupal.org/project/symfony_mailer) (`symfony_mailer`) on both Live and Dev servers whilst finding a simple solution to retaining our  use of Mailhog to capture outgoing email on our dev system. 

Although I've gone into a lot of detail below, making the switch really was quite straightforward, especially if you've done any Drupal theming before.

Why is this important to us?  
============================

In addition to Drupal's immediate reasons for encouragine sites to switch from Swiftmailer to Symfony Mailer there are seemingly other good reasons to switch. Looking at the [Symfony Mailer docs](https://symfony.com/doc/current/mailer.html) you can see that Symfony is looking at the broader picture by integrating into its services, some of the things like DKIM signing, that we have to manage in detail ourselves. If not already included in Drupal's implementation, it seems reasonable to expect inclusion in due course.

On our site we don't use any third party mailing system like Mailchimp (although maybe we will at some point) but we send email directly from our server's `postfix` MTA. Testing the functionality and appearance of our simplenews issues is important to us. 

To do this we use Mailhog (and thus `mhsendmail`) to capture emails on our dev system. The Dev system runs in a Parallels virtual machine provisioned using our fork of [Jeff Geering's Drupal-VM](https://www.drupalvm.com) which has [Mailhog provisioning built in](https://github.com/geerlingguy/ansible-role-mailhog).  

Our system  deploys several `simplenews` "issues" (distribution lists), key to the functioning of our community website. We send a low volume of emails but each one is important to us as several of our content types fulfil a statutary requirement.   

We were keen to switch our Simplenews and site emails from Mail System and Swiftmailer to Symfony Mailer so long as we coould do so with  as few differences between Live and Dev configuration as possible.

History - and how we abandoned our previous approach
----------------------------------------------------

Our dev system we used to have   `php.ini` provisioned with: 

```ini
sendmail_path=/opt/mailhog/mhsendmail
``` 

and on our our live system the `sendmail_path` was left as the default sendmail binary. In this way we could maintain an identical Drupal configuration between dev and live systems thus eliminating potential errors by minimising any configuration differences. 

Since Symfony Mailer's *Native* Transport would use whatever was configured in the PHP `sendmail_path` this seemed the way to go.  

Were we to use the default `sendmail` binary on our Dev system, attempts to send email into the ether would not be successful as connecting MTAs would reject them as they would be eminating from the private subnet IP adddress of the Dev virtual machine.

So, that *was* our approach which we tried to maintain as we switched to Symfony Mailer after which mailing produced no errors yet neither did it produce any email output! We tried various combinations of -t -bs -i `mhsendmail` parameters with varying degrees of failure. (Turns out that `mhsendmail` ignores all these flags anyway).

I should note that to have got this far in trying Symfony Mailer we had to have gone through the [installation](https://www.drupal.org/docs/contributed-modules/symfony-mailer-0/getting-started#s-installation) process. More on this later when we talk about Symfony Mailer *Policies*.

So, in the end, we could not meet our objective of having both dev and live Symfony Mail configuration use *Native* Transport. This is the approach we abandoned as it would seem that, in the nature of the `sendmail`-style interaction between the two processes (Mailhog and Symfony), each  has different expectations of the other.  

BTW you can check that `mailhog` is working at any time as mail sent from the Linux `mail` command  should correctly invoke `mhsendmail` and messages then routed to the Mailhog Web UI for inspection. e.g.

```bash
$ mail -s"Test Subject" -aFrom:webmaster@my.website.uk anyemail@me.com <<< "Test Body"
``` 

Our solution
============

We *do* have a workable solution that allows us to trap emails in Mailhog in dev and actually send messages via the *Sendmail* transport on the live system and the *SMTP* Transport on the Dev system, both using Drupal Symfony Mailer and with the same config settings in both.

How we do it is this: the *default* Transport in the GUI is permanently set to `sendmail` but we also have a (non-default) SMTP Transport configuration set to Host 127.0.0.1 and Port 1025 (Mailhog's port).  

So the *Sendmail* configuration gets picked up in the Live system however  it gets  overriden by *SMTP* in the dev system because we do a config override in our dev's `settings.php` thus:

```php
$config['symfony_mailer.settings']['default_transport'] = 'smtp';
```  

Theming 
=======

We are talking about formatting the HTML messages generated by Simplenews when we send a node as a test message to one recipient or to all recipients of an issue / distribution list.  

We use Mailhog to be able to capture the email message and see whether it has the correct HTML and is formatted correctly according to our CSS.

Inspecting outgoing email
-------------------------

First we need to enable TWIG Debugging in `services.yml`:  

```yaml
parameters:
  twig.config:
    debug: true 
```

Then, in the source markup of outgoing emails, we will see the names of the TWIG templates that Drupal is looking for when it is rendering a node.  

```html
<!— THEME DEBUG —>
<!— THEME HOOK: ‘node’ —>
<!— FILE NAME SUGGESTIONS:
   * node—501–email-html.html.twig
   * node—501.html.twig
   x node—article—email-html.html.twig
   * node—article.html.twig
   * node—email-html.html.twig
   * node.html.twig 
   —>
<!— BEGIN OUTPUT from ‘themes/contrib/our_theme/templates/node—article—email-html.html.twig’ —>

<div class=“mailpix”>
  <img class=“mailpix_img” 
    src=“http://my.website.uk/themes/contrib/our_theme/images/email_hero.jpg” 
    alt=“My Web Site banner hero image” 
    style=“max-width: 100%; border: 0; height: auto; outline: none;” >
</div>

<p class=“link-to-page” 
    style=“font-family: serif; font-size:18px; line-height: 26px; text-align: center; width: 100%; margin: 0;”>

<a href=“http://my.website.uk/node/501” 
    style=“text-decoration: none; font-family: Arial,sans-serif; font-size: 12px; line-height: 16px; color: #0d77b5;”>
    You can view this message in our web browser
</a>
</p>
.
.
.

<!-- END OUTPUT from 'themes/contrib/our_theme/templates/node—article—email-html.html.twig' -->
``` 

Inspecting the above *DEBUG* information, there are several things to note here:  

1.  Symfony Mailer has its own template naming conventions, so we need to rename the templates that we previously used with Swiftmail (more below).
1.  Theme functions corresponding to theme template names will also need to be renamed. (more below).
1. The suggested template names will appear following the *FILE NAME SUGGESTIONS* line with an _x_ against the one actually chosen. 
1. The *BEGIN OUTPUT* line will show whether the template was chosen from our own theme or one of its base themes.
1. CSS from our theme’s `email.css` is injected inline to the html as  email clients generally only handle inline styling and have no means of including external CSS files.
1. [Symfony Mailer’s installation docs](https://www.drupal.org/docs/contributed-modules/symfony-mailer-0/getting-started#s-installation) describe how `<our theme's>.libraries.yml` is configured to point to `email.css`


Renaming theme templates
------------------------  
 
Symfony Mailer has its own template naming conventions, so we will need to rename the templates that we used with Swiftmail. For example, after switching to Symfony Mailer our site has templates named like this:  

`node--article--email-html.html.twig` which is for our *Article* content type node being displayed in its `email-html` view.

And now we will probably need Xdebug to see exactly what data we are being passed in `$variables` so at this point I will refer those of you who, like me are without  PHPStorm by choice or necessity, to a previous post where I describe how to simply deploy [PHP 8.1's Xdebug 3](https://iainhouston.com/atom_xdebug_client/)  with my editor of choice, Atom (April 2022).  

Then we have `email.html.twig` which takes the place of `swiftmail.html.twig` which is used to wrap the _body_ of our simplenews messages. (I think we based the following Microsoft client-friendly HTML on a helpful Mailchimp blog post):  

```html
<!DOCTYPE html>
<html lang="en" xmlns:o="urn:schemas-microsoft-com:office:office">
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  <!--[if mso]>
  <xml>
    <o:OfficeDocumentSettings>
      <o:PixelsPerInch>96</o:PixelsPerInch>
    </o:OfficeDocumentSettings>
  </xml>
  <![endif]-->
</head>
<body>
<!--[if mso]>
<table role="presentation" border="0" cellpadding="0" cellspacing="0" style="margin:0 auto; width:600px;"><tr><td>
<![endif]-->
<div class="body-wrapper">
{% raw %}{{ body }}{% endraw %}
</div>
<!--[if mso]>
</td></tr></table>
<![endif]-->
</body>
</html>
```

I would like to refer you to the [Drupal Symfony Mailer documentation pages](https://www.drupal.org/docs/contributed-modules/symfony-mailer-0/getting-started#s-installation) for more info on theme file naming

Drupal Symfony Mailer Policy markup
-----------------------------------

**TBD** *The purpose and functionality of Symfony Mailer's *Policy* markup and how they simplify templates?*

As I mentioned before, it is necessary to import previously-used `simplenews` and `swiftmail` config settings into Symfony Mailer as a step in its [installation process](https://www.drupal.org/docs/contributed-modules/symfony-mailer-0/getting-started#s-installation).

I haven't completely got my head around the thinking and practice behind everything the  Drupal module `symfony_mailer` does with its *Policies* but we can see that they get created when we follow the installation instructions and import   existing configuration data. And we can see that some pretty clever analysis is going on. For example, since Symfony Mailer is concerned with all the component parts of a mail message (*Body, From, Subject* etc.), it creates *Policy* markup configurations for each of those components as they occur in our existing Simplenews configuration YAML.  

In the example below see what Symfony Mailer has picked up from Simplenews during import:  

```yaml
# file symfony_mailer.mailer_policy.simplenews_newsletter.node.councillors.yml
uuid: 9b952dfc-...
langcode: en
status: true
dependencies:
  config:
    - simplenews.newsletter.councillors
id: simplenews_newsletter.node.councillor
configuration:
  email_from:
    addresses:
      -                                                                                                                                                                   
        value: clerk@bmy.website.uk
        display: 'Our Parish Clerk'
```

... whilst retaining a dependency on the Simplenews configuration item `simplenews.newsletter.councillors`

```yaml
# file simplenews.newsletter.councillors.yml
uuid: 581c5a46...
langcode: en
status: true
dependencies: {  }                                                                                                                                                        
name: Councillors
id: councillors
description: 'This list of  email addresses are "subscribed" by the Parish Clerk and is used to summons councillors to meetings; circulate documents, etc.'
format: html
priority: <0></0>
receipt: true
from_name: 'Our Parish Clerk'
subject: '[[simplenews-newsletter:name]] [node:title]'
from_address: clerk@my.website.uk
hyperlinks: true
allowed_handlers: {  }                                                                                                                                                    
new_account: 'off'
access: default
weight: 0
```
And Drupal's Symfony Mailer GUI allows us to edit what we have imported.

![Editing Simplenews Policy](/assets/images/SimplenewsPolicy.png)


