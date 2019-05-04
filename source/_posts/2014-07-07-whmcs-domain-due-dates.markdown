---
layout: post
title: "WHMCS Domain Due Dates"
date: 2014-07-07 11:27:37 -0600
comments: true
categories: 
---

###Problem###
I use [WHMCS](http://www.whmcs.com) as my billing system for the small web hosting company I own and operate.  One annoying thing about WHMCS is that the due date for domains is set by default to the expiration date.  I also like to process domain renewals manually since it is a very low margin product for me and making a few mistakes of renewing a domain that was not paid for can erase my profit.  Thus, I want to give myself a little buffer time between the deadline for payment and the expiration of the domain name.

###Solution###
Since WHMCS is a database-driven application, I can simply update the database with the dates I'd like to use for the invoice date and due date.  Since the application already syncs expiration dates with the registrar, I can use that to base my dates off of.  Looking at the database, there's a table called tbldomains with columns labeled expirydate, nextduedate, and nextinvoicedate.  These are the columns we will be working with.  In my case, I want the invoice date to be 1 month before the expiry date and the due date to be 2 weeks before the expiry date.  Thus, I simply execute the following SQL:  

``` sql
update tbldomains set nextinvoicedate=DATE_SUB(expirydate, INTERVAL 1 MONTH), nextduedate=DATE_SUB(expirydate, INTERVAL 2 WEEK);
```

In order to run this every day, I want to set up a cron job to run my query on the appropriate database.  The ```mysql -e``` flag will do what I want, but I don't want my password in plaintext inside the crontab, so I create a .my.cnf inside my home directory with the following contents:  

```
[client]
password=your_password
```

To make sure only you can read and edit it, make sure to chmod 600 the file as well.  My final cron command looks like so:  
``` bash
/usr/bin/mysql -udb_user db_name -e "update tbldomains set nextinvoicedate=DATE_SUB(expirydate, INTERVAL 1 MONTH), nextduedate=DATE_SUB(expirydate, INTERVAL 2 WEEK);" > ~/duedate.log 2>&1
```