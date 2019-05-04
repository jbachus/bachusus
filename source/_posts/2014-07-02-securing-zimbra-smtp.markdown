---
layout: post
title: "Securing Zimbra SMTP and Installing policyd for Throttling"
date: 2014-07-02 15:46:50 -0600
comments: true
categories: 
---

###Why?###
Running an email server is a constant battle against spammers and hackers.  I've enabled many settings and installed several tools to help prevent these attacks on Zimbra servers I administer.  I'm documenting them here so I don't forget!

###Some Local Configs###
``` bash
# switch to zimbra user
su - zimbra
# immediately fail messages to over quota inboxes
zmprov mcf zimbraLmtpPermanentFailureWhenOverQuota TRUE
# retrying bounced messages for 5 days is excessive
zmlocalconfig -e postfix_bounce_queue_lifetime=1d
# I want to see the SASL username in the headers
zmlocalconfig -e postfix_smtpd_sasl_authenticated_header=yes
```
Also, in the Zimbra admin, under "Global Settings", I have "reject_non_fqdn_sender" and "reject_unknown_sender_domain" enabled.  I also use the b.barracudacentral.org RBL.

###Enabling policyd for Throttling###
Let's enable policyd through Zimbra's handy provisioning:
``` bash
zmprov ms <servername> +zimbraServiceEnabled cbpolicyd
```

Wait a few minutes for provisioning to finish.  After that, we want to enable the Web UI.  This must be done as root:
``` bash
cd /opt/zimbra/httpd/htdocs/ && ln -s ../../cbpolicyd/share/webui
```

After that, edit the ./webui/includes/config.php.  Comment out anything that's not commented out and then make this the only active line:
``` php
$DB_DSN="sqlite:////opt/zimbra/data/cbpolicyd/db/cbpolicyd.sqlitedb";
```

If you don't use spellcheck, httpd may be down.  Start it with a `zmhapachectl start` as the zimbra user.  Then you should be able to navigate to http://hostname:7780/webui/index.php.  You'll want to secure this or leave apache down after you're finished configuring it.

###Policyd Configuration for Throttling###
I'm still adjusting my configuration, but to get a basic throttling setup going, I did the following:

1.  In Policies>Main, I disabled all policies except "Default System Policy" and "Default Outbound System Policy"
2.  For the "Default Outbound System Policy", I modified the Members to make a Source of %internal_ips and Destination of any.
3.  In Policies>Groups, edit the Members of the internal_ips group and enter your subnet as the only member.
4.  In Quotas>Configure, I set up two quotas, one for Sender:user@domain and one for SASLUsername.  Both of mine are configured to dump excess email into the Hold queue.  Once those are set up, set the limit for each one.  Everything you create will default to disabled and must be edited to change that to enabled.
5.  Monitor your hold queue very closely.  I provided a script at <a href="/blog/2014/07/02/zimbra-abuse-alerts/" target="_blank">/blog/2014/07/02/zimbra-abuse-alerts/</a> that I'm using.

