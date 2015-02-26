---
layout: post
title:  "Configuration Management Overview"
date:   2015-02-24 10:00
categories: devops
---

###Beanstalk, Engineyard or Heroku?
These choices are very easy to set up and scale easily and automatically.  Using these services by-pass the need for
a configuration management framework.  You should not theoretically need to worry about servers and set up.
This is a good choice for a fairly vanilla application. However, configuration is limited, you are pretty much stuck with what they give you.  Also the
platforms are limited to Java, PHP, .NET, Node.js, Python, and Ruby.

###Puppet Labs: Puppet
Puppet, is a configuration management system. The basic setup consists of "Puppet Master" server and a client on each
of the machines being managed.  It can also work as a stand alone with local files.  In normal operation the client program
is run periodically and runs the setup scripts.  thus if some configuration setting gets changed, just wait a half hour,
and it will be set right again. You can purchase the enterprise version and get more/better support
or you can go alone with the open source version.

###Devops: Chef
Chef is another configuration management system works almost the same way a Puppet with nearly identical features.  It
too has an enterprise version and an open source version, but additionally there is a "Hosted" version of Chef that is
totally free for you to get started with up to five nodes.  (After that you will need to pay or set up your own server.)
An additional benefit to the hosted version is that is walks you through the setup of the server and client.

###Chef vs Puppet
As I noted earlier the feature of these two competing products is almost identical however there are a few key differences
that I have found that I think give Chef an edge.

####Chef's web interface
I am not sure if puppet has one but the chef server web interface makes browsing available configuration settings a easy.
You can also do a limited about of editing.

####Knife the all purpose command line tool.
Loading up the puppet server and managing the  scripts is all done via source control like git or svn.  Not so with knife
knife is a highly configurable all purpose commandline tool for managing Chef 'recipes' and uploading them to the chef
server.  It may seem a little overwhelming at first but knife is incredibly powerful and makes dealing the chef
recipes, servers, and clients very script-able.  The power of knife will be shown in the next couple of posts.

####Order of execution
Chef will execute all order in the order given, you are guaranteed that.  This means though that there there is an implied
dependency command 'b' will only start once command 'a' is finished.  If command 'a' fails then all processing halts.
With Puppet if command 'a' fails it will keep on working on command 'b' and in fact may finish 'b' before 'a' even starts.
With Puppet the order of execution is not guaranteed.  In practice this means that all dependencies MUST be spelled out
in the puppet scripts.  This can seem like an advantage, but I find the the implied dependency of the guaranteed order of
execution of Chef to be more intuitive and you are less likely to have a partially configured machine.

#### Amazon uses Chef!
Perhaps the best reason in Chef's favor!

###Amazon ops works.
Our last configuration management system is the AWS "opsWorks" system.  This is basically Amazon's version of a chef
server it works a little differently but it uses actual standard Chef Recipes.  It it does lock you  into using AWS.
And it currently does not support windows.
