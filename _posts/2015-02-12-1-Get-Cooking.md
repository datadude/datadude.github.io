---
layout: post
title:  "Chef Overview"
categories: chef devops
date:   2015-02-24 11:00

---
Our Goal is to set up a chef server then use knife and the knife-ec2 plugin to spin up ec2 instances
and configure EC2 intances of a linux server with a one line command from your local terminal.  This one line command
will:

1. Start up an EC2 instance from a designated image on EC2
2. Run the chef "bootstrap" commands in order to install the basic chef client.
3. Check in with the Desinated chef server.

The chef server serves a "Go To" repository for any chef client that we configure.  It holds the "Recipes" for
how to build out a server that has the "Role" that we assign it.  The chef client communicates with the chef server via
normal HTTP protocol.   The chef client will

1. Idenitify itself to the server using a secure key.
2. pull down its roles and recipes from the server.
3. compile a "runlist" of recipes that it needs to run.
4. execute the run list in order until it run into an error or it finishes the run list.


![EC2 and Chef]({{site.url}}/assets/images/ec2_and_chef.jpg)


###Prerequisites:

1. An AWS account with access to EC2 and S3

