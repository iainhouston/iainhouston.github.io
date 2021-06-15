---
layout: post
title: Procedure to renew SSL Certificate annually
categories: ["DevOps", "Drupal", "Parish Councils"]
---

Our webserver is an Apache2 httpd with `php7.4-fpm` on Ubuntu Linux. Recently our Domain Registrar / SSL Certificate provider was taken over and completely changed the process for renewing our SSL Certificate.

###Generate the CSR

A Certificate Signing Request (CSR) is generated with:  

    openssl req -in bradford-abbas.uk.csr -noout -text

I moved the resulting `.csr` file to the `certs` directory  

    sudo mv bradford-abbas.uk.csr /etc/ssl/certs  

and the `.key` file to the `private` directory  

    sudo mv bradford-abbas.uk.key /etc/ssl/private  

and gave user and group read-only permissions to the key.

###Send CSR to SSL provider

They should deal with Geotrust to obtain a certificate and / or authorisation request email-outs.
