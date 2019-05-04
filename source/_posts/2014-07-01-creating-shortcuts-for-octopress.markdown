---
layout: post
title: "Creating Shortcuts for Octopress"
date: 2014-07-01 19:48:20 -0600
comments: true
categories: 
---
###Why?###
Now that I have [Octopress](http://www.octopress.org) installed along with the [s3_website](https://github.com/laurilehmijoki/s3_website) plugin so that I can easily publish to Amazon S3 and CloudFront, I wanted to make it a little easier to do common tasks like creating a new blog post or page, generate the static pages, and publish the site to the web.

###Prompting for Page and Post Names###
To ensure all appropriate characters are converted for the URL names, I wanted to have the new post and new page tools prompt for the title rather than passing it as an argument, which can sometimes cause problems in parsing special characters or spaces along with making aliasing the commands much more complicated.  Luckily the new_post function already has this ability, but the new_page tool does not, so it needs to be added to the Rakefile.  Here's the diff to add that to the Octopress Rakefile:
``` diff
diff --git a/Rakefile b/Rakefile
index 7c15332..654cc54 100644
--- a/Rakefile
+++ b/Rakefile
@@ -122,12 +122,16 @@ end
 # usage rake new_page[my-new-page] or rake new_page[my-new-page.html] or rake new_
 desc "Create a new page in #{source_dir}/(filename)/index.#{new_page_ext}"
 task :new_page, :filename do |t, args|
+  if args.filename
+    pagename = args.filename
+  else
+    pagename = get_stdin("Enter a name for your page: ")
+  end
   raise "### You haven't set anything up yet. First run `rake install` to set up a
-  args.with_defaults(:filename => 'new-page')
   page_dir = [source_dir]
-  if args.filename.downcase =~ /(^.+\/)?(.+)/
+  title = pagename
+  if pagename.downcase =~ /(^.+\/)?(.+)/
     filename, dot, extension = $2.rpartition('.').reject(&:empty?)         # Get f
-    title = filename
     page_dir.concat($1.downcase.sub(/^\//, '').split('/')) unless $1.nil?  # Add p
     if extension.nil?
       page_dir << filename
```
###Shell Shortcuts###
In addition, I wanted to create some terminal shortcuts for creating a new post, page, for generating the static pages, and finally for publishing the changes to the web.  To do that, since I use bash as my shell, I simply added the following to my ~/.profile:
``` bash
	alias newpost="rake new_post"  
	alias newpage="rake new_page"  
	alias generate="rake generate"  
	alias publish="s3_website push --site=public"  
```
Now every step can be accomplished with a single word so I no longer have to remember the syntax and commands, even though they aren't terribly complex anyway.  Remember, you must source your new .bashrc file to activate it after making any changes.  This is simply done by executing `. ~/.bashrc`.