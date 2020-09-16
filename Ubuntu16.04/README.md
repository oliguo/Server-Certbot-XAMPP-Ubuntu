# Ubuntu 16.04 (XAMPP)
```
I met the case I could not do the same logic by '--apachectl',
and certbot said it is depreciated, then I found the version is not latest on 16.04,
and I cannot upgrade the OS to 18.04, so I have to try another way to do generate cert automatically.
```

## --manual --manual-auth-hook --manual-cleanup-hook
```
Yup, we need to use the above parameters to do our jobs after I found it is helpful to generate on 16.04.
```

## Let's start, install certbot first and to do the same steps like on 18.04
```
Please check the point 3,4,5,6,7
```

## XAMPP Virtual Host on my case

### Webroot Path
```
/opt/lampp/htdocs/abcd.com/
```

### Virtual Host Config
```
/opt/lampp/etc/extra/httpd-vhosts.conf

<VirtualHost *:80>
DocumentRoot  "/opt/lampp/htdocs/abcd.com"
ServerName abcd.com
<Directory "/opt/lampp/htdocs/abcd.com">
        Options Includes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
</Directory>
ErrorLog "/opt/lampp/htdocs/abcd.com/domain_error_log"
ErrorDocument 404 '404'
</VirtualHost>
```

### .Well-Known
```
/opt/lampp/htdocs/abcd.com/.well-known/acme-challenge
```

## 1. Important Steps when we will use --manual-auth-hook --manual-cleanup-hook by creating execute file(.sh)
### 1.1 Create authenticator.sh
```
sudo nano /opt/lampp/htdocs/abcd.com.authenticator.sh
#----- copy and paste -----
#!/bin/bash
echo $CERTBOT_VALIDATION > /opt/lampp/htdocs/Project/abcd.com/.well-known/acme-challenge/$CERTBOT_TOKEN
#----- copy and paste -----

sudo chmod -R 777  /opt/lampp/htdocs/abcd.com.authenticator.sh
```
### 1.2 Create cleanup.sh
```
sudo nano /opt/lampp/htdocs/abcd.com.cleanup.sh
#----- copy and paste -----
#!/bin/bash
rm -f /opt/lampp/htdocs/abcd.com/.well-known/acme-challenge/$CERTBOT_TOKEN
#----- copy and paste -----

sudo chmod -R 777  /opt/lampp/htdocs/abcd.com.cleanup.sh
```

## 2. We can run command to do now
```
sudo certbot certonly --manual \
 --preferred-challenges=http \
 --manual-auth-hook /opt/lampp/htdocs/abcd.com.authenticator.sh \
 --manual-cleanup-hook /opt/lampp/htdocs/abcd.com.cleanup.sh \
 -d abcd.com
```

### 3. Here is a running log when it works
```
root@ubuntu:/home/oli# certbot certonly --manual  --preferred-challenges=http  --manual-auth-hook /opt/lampp/htdocs/abcd.com.authenticator.sh  --manual-cleanup-hook /opt/lampp/htdocs/abcd.com.cleanup.sh  -d abcd.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for abcd.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/abcd.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/abcd.com/privkey.pem
   Your cert will expire on 2020-08-24. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

root@ubuntu:/home/oli# 
```

### 4. Then we do the renew test and it works and we can set it on the crontab weekly for checking
```
certbot renew --dry-run

root@ubuntu:/home/oli# certbot renew --dry-run
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/abcd.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator manual, Installer None
Starting new HTTPS connection (1): acme-staging-v02.api.letsencrypt.org
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for abcd.com
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed without reload, fullchain is
/etc/letsencrypt/live/abcd.com/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/abcd.com/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
root@ubuntu:/home/vtladmin# 
```

### 5. Finally, we need update our httpd-vhosts-le-ssl.conf
```
sudo nano /opt/lampp/etc/extra/httpd-vhosts-le-ssl.conf

<IfModule mod_ssl.c>
<VirtualHost *:443>
DocumentRoot  "/opt/lampp/htdocs/abcd.com"
ServerName abcd.com
<Directory "/opt/lampp/htdocs/abcd.com">
        Options Includes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
</Directory>
ErrorLog "/opt/lampp/htdocs/abcd.com/domain_error_log"
ErrorDocument 404 '404'

SSLCertificateFile /etc/letsencrypt/live/abcd.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/abcd.com/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>
```

### 6. Then restart XAMPP ! And Set up Crontab. Done !!!
```
sudo /opt/lampp/lampp restart


sudo crontab -e
------------
0 0 * * 0 certbot renew
------------
```
