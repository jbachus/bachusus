---
layout: post
title: "Automated EC2 Snapshots"
date: 2015-01-12 15:28:01 -0700
comments: true
categories: 
---
I've been running several VMs on Amazon's EC2 through their AWS services and need to run automated backups just in case something happens.  Luckily, they have a nice API to make this really easy.

EBS snapshots are incremental, so they don't take a ton of space if you're doing daily snapshots.  Root snapshots should only be made while the host is down, but user data should be partitioned off of the root filesystem anyway.

We just have two volumes to snapshot - /home and /data.  One of those does house MySQL database files, so we want to have that flushed while the snapshot is running.  I found a tool called ec2-consistent-snapshot that makes it easy to freeze the filesystems, flush the DB, and take the snapshot.

Step 1: Install the tool  
```bash
    yum --enablerepo=epel install perl-Net-Amazon-EC2 perl-File-Slurp perl-DBI perl-DBD-MySQL perl-Net-SSLeay perl-IO-Socket-SSL perl-Time-HiRes perl-Params-Validate perl-DateTime-Format-ISO8601 perl-Date-Manip perl-Moose ca-certificates
	git clone https://github.com/alestic/ec2-consistent-snapshot.git ec2-consistent-snapshot
```

Step 2: Gather your data  
Gather your volume IDs from the AWS console and inventory any database instances, usernames, passwords, etc.  Create your .awscredentials file and .my.cnf with your credentials for AWS and MySQL.

Step 3: Create the scripts  
```bash
ec2-consistent-snapshot \
 --freeze-filesystem /home \
 --region ap-southeast-1 \
 --description "Snapshot for home partition $(date +'%Y-%m-%d %H:%M:%S')" \
 vol-XXXXXXXX
 
 ec2-consistent-snapshot \
 --freeze-filesystem /data \
 --region ap-southeast-1 \
 --mysql \
 --description "Snapshot for data partition $(date +'%Y-%m-%d %H:%M:%S')" \
 vol-XXXXXXXX
```
 
 Step 4: Put it in a script and schedule it in cron  
 
 Easy!