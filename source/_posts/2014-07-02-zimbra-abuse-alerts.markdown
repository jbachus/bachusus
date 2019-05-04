---
layout: post
title: "Zimbra Abuse Alerts"
date: 2014-07-02 09:58:38 -0600
comments: true
categories: 
---
###Why?###
I administer several [Zimbra](http://www.zimbra.com) servers for one of my clients located in Asia.  Since the client computers typically don't adhere to the best security practices - like system updates and antivirus software - a recurring problem is that user email accounts are routinely compromised and used to send spam.  I have taken many precautions like preventing brute force login attempts, enabling very strict SMTP restrictions, forcing SSL/SASL on all connections, and most recently enabling policyd to prevent any email floods that might indicate a compromised client.  Policyd is configured to divert email volume over 100 messages per hour to the held queue in Postfix, so I want to monitor the held queue and lock any accounts whose email ends up in this queue.  I also want notification when this happens, or if something falls through the cracks and the deferred queue grows to over 100 messages, which is usually indicitive of a compromised account.  One additional requirement is that the alerts need to be sent through a separate SMTP server so they still get delivered in the event of a backlog on the local SMTP server.

###Gathering Tools###
Since the stock `mail` command can't send mail to a remote SMTP server, I needed to find a simple tool to make this easy.  I could have written something in Perl or PHP, but I knew there was something simple out there.  After searching google and finding a few tools: ssmtp and mailx, I didn't find them in my yum repo and didn't want to maintain separate packages, so I decided to search my yum repositories.  A simple `yum search smtp` came up with a tool called simply `email` that would fit my needs.  A simple `yum install email` got me up and running and the man page has all the information I need.

###Configuring and Testing `email`###
The default configuration file for `email` is at /etc/email/email.conf.  I wanted to connect directly to the Gmail SMTP server to ensure delivery, so I set SMTP_SERVER to 'smtp.gmail.com', the SMTP_PORT to 587, filled out the MY_NAME and MY_EMAIL variables, set USE_TLS to 'true', commented out signature and address book files, and finally set SMTP_AUTH to 'LOGIN', filled out my SMTP_AUTH_USER and SMTP_AUTH_PASS to my Gmail credentials, and saved the file.  Be sure to `chmod 600 /etc/email/email.conf` as well to protect your credentials.  In order to test that everything worked, I ran `echo "Test123" | email -s "Test email" myaddress@gmail.com` and verified that it was sent successfully and appeared in my inbox.

###Set Postfix to Print SASL User in Headers###
In order to more easily keep track of which SASL username is compromised, we need to configure Postfix to print this information in the message headers.  This can be done using the smtpd_sasl_authenticated_header config value.  Since Zimbra wraps the Postfix configs, we have to set it like so:
``` bash
zmlocalconfig -e postfix_smtpd_sasl_authenticated_header=yes
#Restart Postfix to apply changes
zmmtactl restart
#Validate config variable is set
postconf -v | grep smtpd_sasl_authenticated_header
> smtpd_sasl_authenticated_header = yes
```

###Simple Alert Script###
This could be improved, especially tying in to a nagios or other monitoring/alerting system, but this does fine for my needs at the moment.  I set it up to run every hour in my crontab.
``` bash
#!/bin/bash

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

MAXDEFERRED=100
CURDEFERRED=`find /opt/zimbra/data/postfix/spool/deferred -type f | wc -l`

CURHELD=`find /opt/zimbra/data/postfix/spool/hold -type f | wc -l`

if [[ $CURDEFERRED -gt $MAXDEFERRED ]]; then
  email -b -s "Server has $CURDEFERRED deferred messages in the queue" admin@domain.com
fi

if [ $CURHELD -gt 0 ]; then
  find /opt/zimbra/data/postfix/spool/hold -type f | xargs postcat > /tmp/heldmsg.txt
  SASL_SENDER=`grep "Authenticated sender:" /tmp/heldmsg.txt`
  #Send the full held queue to the admin
  email -s "Server has $CURHELD held messages in the queue" admin@domain.com < /tmp/heldmsg.txt

  #Lock the account if we have the username
  if [ -n "${SASL_SENDER}" ]; then
    #Extract the logins
    grep "Authenticated sender:" /tmp/heldmsg.txt | sed 's/[^@]* \([a-zA-Z0-9.]*@[^ ]*\).*)/\1/' | uniq > /tmp/disableaccts
    #Lock the accounts in Zimbra
    cat /tmp/disableaccts | xargs -i su - zimbra -c "/opt/zimbra/bin/zmprov ma {} zimbraAccountStatus locked"
    #Send a notification email
    cat /tmp/disableaccts | xargs -i email -b -s "Server has locked {} for sending messages too fast" admin@domain.com
    #Restart Postfix to force reauthentication
    su - zimbra -c "zmmtactl restart"
  fi
#Cleanup
rm -f /tmp/heldmsg.txt
rm -f /tmp/disableaccts
fi
```


