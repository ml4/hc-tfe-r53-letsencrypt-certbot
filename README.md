![Development status](https://img.shields.io/badge/status-PRD-D00.svg?style=for-the-badge)
![Certbot version](https://img.shields.io/badge/certbot-1.1.6-003b5b.svg?style=for-the-badge)

# hc-tfe-r53-letsencrypt-certbot

Repository which automates the setup and management of SSL certs for domains managed in Route53 when creating TFE instances which need them.

The rationale is:
* Free SSL certs
* Configured and managed as automatically as possible
* Simple reproducible setup a la github
* Use Route53 for domain mgmt
* Follow Matthew Sanabria's great treatment of using the .pem outputs: https://www.terraform.io/docs/enterprise/install/installer.html

Let's Encrypt (https://letsencrypt.org/donate/) produces certs with a lifespan of 90 days.  Good for experimentation and learning, and the service will email you when they are due to expire, but it's even more reason to manage the setup as automatically as possible.  Hacking together stuff from a couple of places:

To get a cert from certbot quickly for ea.demos.io using Route53

* Note the certbot call is different to Li0nel's cos we have a subdomain to handle.  Copy paste as needed to setup.
* See https://hackernoon.com/easy-lets-encrypt-certificates-on-aws-79387767830b for more
* From the GITROOT:

```shell
brew install certbot

PREFIX="${HOME}/Documents/letsencrypt"
mkdir -p ${PREFIX}
MASTERDOMAIN=yourdomain.io
SUBDOMAIN=mysubdomain.${MASTERDOMAIN}
EMAIL=you@email.com
certbot certonly \
--non-interactive \
--manual          \
--manual-auth-hook "./auth-hook.sh UPSERT ${MASTERDOMAIN}" \
--manual-cleanup-hook "./auth-hook.sh DELETE ${MASTERDOMAIN}" \
--preferred-challenge dns \
--config-dir "${PREFIX}" \
--work-dir "${PREFIX}" \
--logs-dir "${PREFIX}" \
--agree-tos \
--manual-public-ip-logging-ok \
--domains ${SUBDOMAIN} \
--email ${EMAIL}
```

* For the first run, one gets output such as

```
Saving debug log to /Users/ml4/Documents/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for ml4.ea.demos.io
Running manual-auth-hook command: ./auth-hook.sh UPSERT ea.demos.io
Waiting for verification...
Cleaning up challenges
Running manual-cleanup-hook command: ./auth-hook.sh DELETE ea.demos.io
Non-standard path(s), might not work with crontab installed by your operating system package manager

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /Users/ml4/Documents/letsencrypt/live/ml4.ea.demos.io/fullchain.pem
   Your key file has been saved at:
   /Users/ml4/Documents/letsencrypt/live/ml4.ea.demos.io/privkey.pem
   Your cert will expire on 2020-05-23. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /Users/ml4/Documents/letsencrypt. You
   should make a secure backup of this folder now. This configuration
   directory will also contain certificates and private keys obtained
   by Certbot so making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

 ```

* The files you're interested in are in ${PREFIX}/live/<domain> which, for TFE, is the privkey1.pem and the fullchain1.pem which is a cat of cert1.pem and chain1.pem in the right order.

* For a renewal, certbot finds the cert in your filesystem from the previous run.  You'll get output such as:

Saving debug log to /Users/ml4/Documents/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Cert is due for renewal, auto-renewing...
Renewing an existing certificate
Performing the following challenges:
dns-01 challenge for ml4.ea.demos.io
Running manual-auth-hook command: ./auth-hook.sh UPSERT ea.demos.io
Waiting for verification...
Cleaning up challenges
Running manual-cleanup-hook command: ./auth-hook.sh DELETE ea.demos.io

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /Users/ml4/Documents/letsencrypt/live/ml4.ea.demos.io/fullchain.pem
   Your key file has been saved at:
   /Users/ml4/Documents/letsencrypt/live/ml4.ea.demos.io/privkey.pem
   Your cert will expire on 2020-07-23. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

