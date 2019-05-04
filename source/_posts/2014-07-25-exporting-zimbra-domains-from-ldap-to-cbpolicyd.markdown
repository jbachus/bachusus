---
layout: post
title: "Exporting Zimbra domains from LDAP to cbpolicyd"
date: 2014-07-25 10:37:02 -0600
comments: true
categories: 
---

###The Problem###
In the ongoing fight against compromised user accounts, I set up cbpolicyd to rate limit outgoing emails from my servers.  However, since I never set up a list of local domains, I occasionally get false positives triggered by users either moving many messages to spam (and automatically being forwarded to the spam autolearn address) or sending to local users.  Since the list of domains can change often, I wanted a way to populate this list on a schedule.

###Devising a Solution###

First, I started with the question of how to extract the list of domains from LDAP.  To get the ldap password, you have to be logged in as zimbra and run ```zmlocalconfig -s | grep ldap```.  Since ldapsearch outputs in LDIF format, some text sanitization had to be done to leave only the list of domains.  The final product of this sanitization resulted in this:  
```bash
ldapsearch -H ldap://<domain name>:389 -LLL -w <ldap password> -D "uid=zimbra,cn=admins,cn=zimbra" "(objectClass=zimbraDomain)" zimbraDomainName | grep -e "zimbraDomainName: " | sed -e 's/zimbraDomainName: //g' | sort | uniq
```

Next, I had to work on getting this list populated into the sqlite database used by cbpolicyd.  First I had to locate where this database was stored by looking in /opt/zimbra/conf/cbpolicyd.conf at the [database] parameter.  Once I had this located, I needed to check what group ID I was using, so logged in and checked like so:  
```bash
$ sqlite3 cbpolicyd.sqlitedb
SQLite version 3.6.20
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> select * from policy_groups;
1|internal_ips|0|
2|internal_domains|0|
```
It looks like my policy group ID is 2, so I'll use that in my scripts.

I created a bash script that first gets the list of domains and redirects the output to a temporary file.  Then it deletes any existing policy group members from the internal_domains group, and finally re-populates it with the domains from the ldap list (the domains need to be prefixed by @).  In order to avoid any locking issues, I'm shutting down cbpolicyd while I delete and repopulate the domain list.  Then I delete the temp file.

###The Final Script###

```bash
#!/bin/bash
PATH=/opt/zimbra/bin:/opt/zimbra/postfix/sbin:/opt/zimbra/openldap/bin:/opt/zimbra/snmp/bin:/opt/zimbra/rsync/bin:/opt/zimbra/bdb/bin:/opt/zimbra/openssl/bin:/opt/zimbra/java/bin:/usr/sbin:/usr/kerberos/bin:/usr/local/bin:/bin:/usr/bin

#Generate domain list
ldapsearch -H ldap://<domain name>:389 -LLL -w <password> -D "uid=zimbra,cn=admins,cn=zimbra" "(objectClass=zimbraDomain)" zimbraDomainName | grep -e "zimbraDomainName: " | sed -e 's/zimbraDomainName: //g' | sort | uniq > /opt/zimbra/data/cbpolicyd/curdomains

#Shut down cbpolicyd to avoid locking issues
zmcbpolicydctl stop

#Purge existing internal_domains entries
sqlite3 /opt/zimbra/data/cbpolicyd/db/cbpolicyd.sqlitedb "DELETE FROM policy_group_members WHERE PolicyGroupID='<Policy Group ID>'";

#Repopulate the db with the domain list
for a in `cat /opt/zimbra/data/cbpolicyd/curdomains`; do
  B="@"$a
    sqlite3 /opt/zimbra/data/cbpolicyd/db/cbpolicyd.sqlitedb "INSERT INTO policy_group_members VALUES (NULL,'<Policy Group ID>','$B', '0', NULL)"
done

#We're done with the db, start cbpolicyd
zmcbpolicydctl start

#Delete our temp file
rm /opt/zimbra/data/cbpolicyd/curdomains
```

###Implementation###
I have this running in cron daily which should be sufficient for the frequency domains are added and deleted for my servers.
