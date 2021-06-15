---
layout: post
title: Procedure to renew SSL Certificate annually
categories: ["DevOps", "Drupal", "Parish Councils"]
---

Our webserver is an Apache2 httpd with `php7.4-fpm` on Ubuntu Linux. Recently our Domain Registrar / SSL Certificate provider was taken over and completely changed the process for renewing our SSL Certificate.

###Generate the CSR

A Certificate Signing Request (CSR) is generated with:  

    openssl req -new -newkey rsa:2048 -nodes -keyout bradford-abbas.uk.key -out bradford-abbas.uk.csr

-   Country Code must be `GB` (and not `UK` for Ukraine)  

- Challenge password must be blank

- Common Name for us `www.bradford-abbas.uk` as it will also allow `bradford-abbas.uk:443`

I moved the resulting `.key` file to the `private` directory  

    sudo mv bradford-abbas.uk.key /etc/ssl/private  

and gave user and group read-only permissions to the key.

###Check CSR encryption

    openssl req -in bradford-abbas.uk.csr -noout -text

should show our Common Name and country Code correctly

###Send CSR to SSL provider

They should deal with Geotrust to obtain a certificate and / or authorisation request email-outs. Sent to them via a support ticket.
