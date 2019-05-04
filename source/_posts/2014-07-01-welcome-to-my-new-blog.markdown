---
layout: post
title: "Welcome to My New Blog"
date: 2014-07-01 13:06:38 -0600
comments: true
categories: 
---
###Why?###
For a long time, I've wanted a place to put random snippets of my work or things I've learned for the day.  __Something simple__.  I could have put up a Wordpress blog fairly easily, but I wanted something more of a minimalist approach.  __Something fast and secure__.  Something I wouldn't have to endlessly maintain and fix.  __I finally decided to go static__.

###Behind the Scenes###
I searched for a while for a simple static blog and eventually settled on [Octopress](http://www.octopress.org) which is written in Ruby and has a mature codebase with enough contributions to make it easy to get up and running quickly.  To fill the requirement of being maintenance free and fast, I decided to publish the blog to a bucket on [Amazon's S3](http://aws.amazon.com/s3/) service and use that bucket as a source for [Amazon's CloudFront](http://aws.amazon.com/cloudfront/) CDN.  This would make the blog extremely quick anywhere in the world.  It would be mirrored all over the world and each user would be directed to the fastest mirror for them.  The DNS is also hosted on [Amazon's Route 53](http://aws.amazon.com/route53/) DNS service which ensures quick name resolution all around the world as well.  I also settled on the [Whitespace](https://github.com/lucaslew/whitespace) theme which, while quite minimalist already, will require some further optimization for my needs.

###What I'll Be Writing About###
This site will be used more as a personal and professional repository of things I've learned and want to share.  Of course, my main interests of travel, technology, and entrepreneurialism will be most prevalent.  More travel-related topics will be posted at [Miles of Adventure](http://www.milesofadventure.com), which is another blog I'm working on.  I will be discussing more about the how and why of my various projects on this site, especially my journey of self-employment and attempt to start several businesses.  Please feel free to follow my writing and send along any comments and feedback that you feel would be helpful.  I'm always looking for feedback.