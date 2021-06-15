---
layout: post
title: Procedure to renew SSL Certificate annually
categories: ["DevOps", "Drupal", "Parish Councils"]
---

Our webserver is an Apache2 httpd with `php7.4-fpm` on Ubuntu Linux. Recently our Domain Registrar / SSL Certificate provider was taken over and completely changed the process for renewing our SSL Certificate.

### Generate the CSR

A Certificate Signing Request (CSR) is generated with:  

      cd
      openssl req -new -newkey rsa:2048 -nodes -keyout bradford-abbas.uk.key -out bradford-abbas.uk.csr

-   Country Code must be `GB` (and not `UK` for Ukraine)  

- Challenge password must be blank

- Common Name for us `www.bradford-abbas.uk` as it will also allow `bradford-abbas.uk:443`

I moved the resulting `.key` file to the `private` directory  

    sudo mv bradford-abbas.uk.key /etc/ssl/private  

and gave user and group read-only permissions to the key.

### Check CSR encryption

    openssl req -in bradford-abbas.uk.csr -noout -text

should show our Common Name and country Code correctly

### Send CSR to SSL provider

They should deal with Geotrust to obtain a certificate and / or authorisation request email-outs. Sent to them via a support ticket.

### Receive Our Certificate and Intermediate CA's from provider

We had to ask them specifically for the Zip file containing Intermediate and domain name certificate.

And make a bundle thus:  

    cat www_bradford-abbas_uk.crt \
      SectigoRSADomainValidationSecureServerCA.crt \
      USERTrustRSAAAACA.crt \
      AAACertificateServices.crt > bradford-abbas.ca-bundle

Order above is important. So now we have:

      total 48
      drwx------@ 7 iainhouston  staff   224B 15 Jun 16:37 .
      drwx------+ 7 iainhouston  staff   224B 15 Jun 12:32 ..
      -rw-rw-r--@ 1 iainhouston  staff   1.5K  1 Jan  2004 AAACertificateServices.crt
      -rw-rw-r--@ 1 iainhouston  staff   2.1K  2 Nov  2018 SectigoRSADomainValidationSecureServerCA.crt
      -rw-rw-r--@ 1 iainhouston  staff   1.9K 12 Mar  2019 USERTrustRSAAAACA.crt
      -rw-r--r--  1 iainhouston  staff   7.7K 15 Jun 16:37 bradford-abbas.ca-bundle
      -rw-rw-r--@ 1 iainhouston  staff   2.2K 15 Jun 00:00 www_bradford-abbas_uk.crt

and `bradford-abbas.ca-bundle` is the certificate file that Apache should be pointed to.
