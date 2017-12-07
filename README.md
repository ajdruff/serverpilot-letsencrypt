# serverpilot-letsencrypt
Add Let's Encrypt Certificate to Server Pilot's Free Plan



#Overview

Server Pilot doesn't provide any documentation on installing let's encyrpt yourself, since its trying to nudge you to their paid plan, which includes it.

Instead, there are a number of scripts and random posts that provide various solutions. My preference was to use certbot and I mostly followed [knynkwl/letsencrypt.md](https://gist.github.com/knynkwl/7f20d57b39979e19ae0e98465eab7ece)



##References:

* [Installing Let's Encrypt with Cerbot on DigitalOcean & ServerPilot](https://gist.github.com/knynkwl/7f20d57b39979e19ae0e98465eab7ece) 
* [Improve SSL Performance with Nginx](https://serverfault.com/q/770464/395483)
* [Certbot User Guide](https://certbot.eff.org/docs/using.html)
* [Enable SSL for Nginx (linode docs)](https://www.linode.com/docs/security/ssl/enable-ssl-for-https-configuration-on-nginx/)
* [Which Lets Encrypt Plugin Should You Use (standalone,webroot,nginx)?]()
* [Certbot](https://certbot.eff.org/)

##Step 1 - Create the Nginx SSL Configuration File

Configure your SSL configuration file for Nginx. If you skip or forget this step, you will get a 'refusal to connect' type error when you attempt to use https to access your site.

configuration file taken from : [knynkwl/letsencrypt.md](https://gist.github.com/knynkwl/7f20d57b39979e19ae0e98465eab7ece)


download the nginx ssl configuration file:

    sudo cd /etc/nginx-sp/vhosts.d/
    wget https://raw.githubusercontent.com/ajdruff/serverpilot-letsencrypt/ma ster/nginx-sp.ssl.conf -O wordpress.ssl.conf

>Replace `wordpress` with the name of your Server Pilot app if you are using something different.

Search and Replace the downloaded file to replace values specific for your site. 


    sed -i 's|APP_NAME|wordpress|g;s|DOMAIN_NAME|example.com|g' wordpress.ssl.conf


>Replace 'wordpress' and 'example.com' with values matching your app name and domain name. Do not include www in your domain name. If you are using subdomains, you'll need to edit the file appropriately.



##Step 2 - Install certbot

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






##Testing


##Contributors

Andrew Druffner <andrew@nomstock.com>

##Attributions

Original SSL configuration taken from: [knynkwl/letsencrypt.md](https://gist.github.com/knynkwl/7f20d57b39979e19ae0e98465eab7ece)
