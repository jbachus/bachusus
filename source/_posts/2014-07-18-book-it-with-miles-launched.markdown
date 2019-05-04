---
layout: post
title: "Book it with Miles launched!"
date: 2014-07-18 10:04:51 -0600
comments: true
categories: 
---
I launched my airline miles award booking service, Book it with Miles, today!  You can see it at [http://bookitwithmiles.com](http://www.bookitwithmiles.com).

Anyway, some of the technical details.  It was built using Jekyll, a static site generator, and I used the theme bundled with Jekyll.  It is hosted on Amazon AWS, specifically on S3 and CloudFront CDN for speed around the world (since it has an international audience).  I chose zopim for a live chat and created a basic form-to-email script to email me the results of the contact form.  After being contacted, I will work with the client on either a Campfire or HipChat chat room, then collect payment with PayPal.