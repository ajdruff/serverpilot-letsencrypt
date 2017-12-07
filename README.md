# serverpilot-letsencrypt
Add Let's Encrypt Certificate to Server Pilot's Free Plan



# Overview

Server Pilot doesn't provide any documentation on installing let's encyrpt yourself, since its trying to nudge you to their paid plan, which includes it.

Instead, there are a number of scripts and random posts that provide various solutions. My preference was to use certbot and manual configuration. 

I mostly followed [knynkwl/letsencrypt.md](https://gist.github.com/knynkwl/7f20d57b39979e19ae0e98465eab7ece)



## References:

* [Installing Let's Encrypt with Cerbot on DigitalOcean & ServerPilot](https://gist.github.com/knynkwl/7f20d57b39979e19ae0e98465eab7ece) 
* [Improve SSL Performance with Nginx](https://serverfault.com/q/770464/395483)
* [Certbot User Guide](https://certbot.eff.org/docs/using.html)
* [Enable SSL for Nginx (linode docs)](https://www.linode.com/docs/security/ssl/enable-ssl-for-https-configuration-on-nginx/)
* [Which Lets Encrypt Plugin Should You Use (standalone,webroot,nginx)?]()
* [Certbot](https://certbot.eff.org/)
* [HSTS and Let's Encrypt](https://timkadlec.com/2016/01/hsts-and-lets-encrypt/)

## Certificate Installation

### Step 1 - Create the Nginx SSL Configuration File

Configure your SSL configuration file for Nginx. If you skip or forget this step, you will get a 'refusal to connect' type error when you attempt to use https to access your site.

***configuration file taken from :*** [knynkwl/letsencrypt.md](https://gist.github.com/knynkwl/7f20d57b39979e19ae0e98465eab7ece)


**download the nginx ssl configuration file:**

    sudo cd /etc/nginx-sp/vhosts.d/
    wget https://raw.githubusercontent.com/ajdruff/serverpilot-letsencrypt/ma ster/nginx-sp.ssl.conf -O wordpress.ssl.conf

>Replace `wordpress` with the name of your Server Pilot app if you are using something different.

Search and Replace the downloaded file to replace values specific for your site. 


    sed -i 's|APP_NAME|wordpress|g;s|DOMAIN_NAME|example.com|g' wordpress.ssl.conf


>Replace 'wordpress' and 'example.com' with values matching your app name and domain name. Do not include www in your domain name. If you are using subdomains, you'll need to edit the file appropriately.



### Step 2 - Install certbot

Certbot, formerly letsencrypt, is a Let's Encrypt client developed by [EFF](https://certbot.eff.org/about/).

Go to [Certbot's install page](https://certbot.eff.org/) and choose 'none of the above' for software, and Ubuntu 6.0.4 for os, or whatever the OS you installed Server Pilot on.

You'll then get the commands needed to install certbot. For Ubuntu 6.0.4, its

    sudo apt-get update
    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt-get update
    sudo apt-get install certbot 

run each command individually (not as a script or paste them all into the command line), since there are prompts;optionally you can use the -y option.


Now do a test run of a staging only certificate:

    sudo certbot certonly --dry-run --staging --webroot -w /srv/users/serverpilot/apps/wordpress/public -d example.com -d www.example.com


(optional) If all goes well, you can then just install the staging certificate if you like:

    sudo certbot certonly --staging --webroot -w /srv/users/serverpilot/apps/wordpress/public -d example.com -d www.example.com

This should result in your being able to to use https, but you'll get 'untrusted' prompts from your browser, since its essentially a 'self-signed' certificate.

Staging certificates are good for testing, or development sites. Note that every attempt at installing a production certificate counts against your limit of 5 for the week, so play around with the --dry-run and --staging plugins until everything looks like its working.

For production use, use the following:


    sudo certbot certonly --webroot -w /srv/users/serverpilot/apps/wordpress/public -d example.com -d www.example.com

>be sure to place which ever domain you'd like to appear on your certificate first when you list your domains.


### Step 3 - Configure for autorenew

`certbit renew` will renew all certificates within 30 days of expiring.


    crontab -e

add the following to crontab:

        # letsencrypt-auto renew command every Tuesday at 5:30 am (2  in crontab is Tuesday)
        30 5 * * 2 /usr/bin/certbot renew --pre-hook "service nginx-sp stop" --post-hook "service nginx-sp start" >> /var/log/letsencrypt-renew.log



#### Check if Need Renewal

    certbot renew


### Step 4  - Force HTTPS


Download nginx-sp.custom.conf :

    sudo cd /etc/nginx-sp/vhosts.d/
    wget https://raw.githubusercontent.com/ajdruff/serverpilot-letsencrypt/master/nginx-sp.custom.conf  -O wordpress.custom.conf

Change to a file name that includes the nme of your application, if not WordPress. Note that this will overwrite any file with the same name.

Edit the file place holders with your domain name:

    sed -i 's|DOMAIN_NAME|example.com|g' wordpress.custom.conf


### Step 5  - Force Redirect of www to Non-WWW (optional)

If you want to force redirect of www.example.com to example.com, do this :

add the following server block to APP_NAME.custom.conf, replacing your `example.com` with your domain

        server {
        #redirect www to non-ww
            listen              443 ssl;
            server_name www.example.com;
          # LetsEncrypt Certs
          ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
          ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
            return 301 https://example.com$request_uri;
        }


remove `www.example.com` from wordpress.ssl.conf from the server_name directive so the start of the file looks like this :


    server {
      listen 443 ssl http2;
      listen [::]:443 ssl http2;
      server_name example.com;

### Final Check

Check your installation with Qualys SSL Labs Free Check:

[https://www.ssllabs.com/ssltest/index.html]
(https://www.ssllabs.com/ssltest/index.html)




### Troubleshooting


#### Key and Certificate Checks

For the following checks, set a shell variable equal to your domain.

    export DOMAIN=example.com

This will allow you to copy and paste the commands into your shell without any editing.

### Directory Locations

**Certificates and keys:**

    /etc/letsencrypt/live/$DOMAIN/

**Certificate Requests (CSRs)**

    /etc/letsencrypt/csr/



##### Server Key Checks

    openssl rsa -in /etc/letsencrypt/live/$DOMAIN/privkey.pem -check | grep RSA

* what this will do: check your private key. without the grep, it will dump your key to stdout
* expected response: RSA key Ok


##### CSR Checks


     openssl req -text -noout -verify -in /etc/letsencrypt/csr/0001_csr-certbot.pem

>The `0000_csr-certbot.pem` may be incremented to 0001 or higher. To see a full list of certificate requests, `ls /etc/letsencrypt/csr/`


* what this will do: check your certificate request.
* expected response: verify OK

##### Cert and Key Match Checks

Copy and paste all of the following lines into your bash shell. They will calculate a checksum of the certificate and key for comparison.


    CERT_CHECKSUM=$(openssl x509 -noout -modulus -in /etc/letsencrypt/live/$DOMAIN/cert.pem| openssl md5)
    KEY_CHECKSUM=$(openssl rsa -noout -modulus -in /etc/letsencrypt/live/$DOMAIN/privkey.pem| openssl md5)
    [[ $CERT_CHECKSUM == $KEY_CHECKSUM ]] && echo Match


 * what this will do: Verify that the key was used correctly for your certificate
* expected response: `Match`


##### Server Certificate Checks:

    openssl x509 -in /etc/letsencrypt/live/$DOMAIN/cert.pem -text -noout

* what this will do: Check validity of your Certificate
* expected response: Prints the Certificate to your terminal.


##### SSL Client Checks

**verify that subject alternate names are present**


    openssl s_client -connect $DOMAIN:443 | openssl x509 -noout -text | grep DNS:

* what this will do: Connect over http to your site and check the validity of your certificate
* expected response: Prints the Certificate to your terminal.

To check that alternate names have been correctly set:

    openssl s_client -connect $DOMAIN:443 | openssl x509 -noout -text | grep DNS:

Chrome will require that at least one alternate names be set (e.g., www.example.com in addition to examplel.com)


#### Redirect Troubleshooting

After applying the 'force https' and 'redirect to non-www' changes, you may have issues with trying to get everything working. 


**Restart Nginx after all configuration changes:**

    service nginx-sp restart


**Use wget to troubleshoot**


    wget --no-hsts http://www.example.com/ | grep Location:


**Turn Off HSTS while troubleshooting**

[HSTS](https://timkadlec.com/2016/01/hsts-and-lets-encrypt/)

HSTS can make it seem like your redirects are working when they aren't, so turn it off when you are troubleshooting using wget with the `--no-hsts` flag.

We also don't want to completely depend on HSTS.
We still want to force using redirects in case HSTSgets misconfigured, we want to make sure we are still enforcing https.

##### Things to Look For While Troubleshooting

* are there excessive redirects?
* are the redirects working properly
* are you getting a message that HSTS policy is being enforced?

##### Check Syntax of Configuration Files

Check Syntax for Nginx configuration:

    sudo /opt/sp/nginx/sbin/nginx -T

Check Syntax for Apache:


    sudo /opt/sp/apache/bin/apachectl -t





## Contributors

Andrew Druffner <andrew@nomstock.com>

## Attributions

Original SSL configuration taken from: [knynkwl/letsencrypt.md](https://gist.github.com/knynkwl/7f20d57b39979e19ae0e98465eab7ece)
