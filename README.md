# Server-Certbot-XAMPP-Ubuntu
Here is a guideline how to use the certbot to help you generate SSL cert and renew it automatically under the XAMPP of Ubuntu 18.04

```
Before writing this guide, 
I was in the trouble about the certbot how to run it well on the XAMPP of Ubuntu,
and did many and many research and read the certbot doc deeply,
I solved the issue, the apache thing by this option --apache-ctl /opt/lampp/bin/apachectl !
```

# Let's start!
## 1. XAMPP Installation
```
I think you have installed the xampp and it runs as well, if you have not installed it, you can check my repo [Server Deployment], it may be help you.
```

## 2. Certbot Installation
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot python-certbot-apache
```

### Note,check the cerbot version
```
apt-cache policy certbot | grep Installed

###Ubuntu 18.04###
  Installed: 0.31.0-1+ubuntu18.04.1+certbot+1
 
###Ubuntu 16.04###
  Installed: 0.27.0-1~ubuntu16.04.1 > This doc tried this version but not working on 16.04
```

## 3. Let's stop the just installed apache2, we make sure it is stopped.
```
sudo service apache2 stop
```

## 4. Comment all ports both 40 and 443 from the file(ports.conf)
```
sudo nano /etc/apache2/ports.conf
------------
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

#Listen 80

<IfModule ssl_module>
#       Listen 443
</IfModule>

<IfModule mod_gnutls.c>
#       Listen 443
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
------------
```

## 5. Let's create the file(httpd-vhosts-le-ssl.conf) which certbot will write the cert information automatically after the verification has been done
```
sudo touch /opt/lampp/etc/extra/httpd-vhosts-le-ssl.conf
```

## 6. Then we add this file(httpd-vhosts-le-ssl.conf) in the httpd.conf of XAMPP
```
sudo nano /opt/lampp/etc/httpd.conf
------------
# Virtual hosts
Include etc/extra/httpd-vhosts.conf
Include etc/extra/httpd-vhosts-le-ssl.conf #Add it here
------------
```
## 7. Retart the XAMPP
```
sudo /opt/lampp/lampp restart
```

## 8. Finally, you can the run command and following the steps of certbot to choose what you want to generate
```
###Ubuntu 18.04###
 sudo certbot --apache-ctl /opt/lampp/bin/apachectl
```

## 9. Here is a running log when it works
```
sudo certbot --apache-ctl /opt/lampp/bin/apachectl
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator apache, Installer apache

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1.: ssl-test.example.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
Obtaining a new certificate
Created an SSL vhost at /opt/lampp/etc/extra/httpd-vhosts-le-ssl.conf
Deploying Certificate to VirtualHost /opt/lampp/etc/extra/httpd-vhosts-le-ssl.conf
Enabling available site: /opt/lampp/etc/extra/httpd-vhosts-le-ssl.conf

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled
https://ssl-test.example.com

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=ssl-test.example.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/ssl-test.example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/ssl-test.example.com/privkey.pem
   Your cert will expire on 2020-06-05. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

## 10. Let's check ours httpd-vhosts-le-ssl.conf
```
cat httpd-vhosts-le-ssl.conf
------------
<IfModule mod_ssl.c>
<VirtualHost *:443>
DocumentRoot  "/opt/lampp/htdocs/SSLTest"
ServerName ssl-test.example.com
<Directory "/opt/lampp/htdocs/SSLTest">
        Options Includes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
</Directory>
ErrorLog "/opt/lampp/htdocs/SSLTest/domain_error_log"
ErrorDocument 404 http://example.com


SSLCertificateFile /etc/letsencrypt/live/ssl-test.example.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/ssl-test.example.com/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>
------------
```

## 12. Restart the XAMPP better again and check the site
```
sudo /opt/lampp/lampp restart
```

## 13. Then we do the renew test and it works and we can set it on the crontab weekly for checking
```
sudo certbot renew --dry-run

Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/ssl-test.example.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator apache, Installer apache
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for ssl-test.example.com
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of apache server; fullchain is
/etc/letsencrypt/live/ssl-test.example.com/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/ssl-test.example.com/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
```

## 14.Set the crontab every Sunday. Done !!!
```
sudo crontab -e
------------
0 0 * * 0 certbot renew
------------
```
