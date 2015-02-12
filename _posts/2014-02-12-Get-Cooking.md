---
layout: post
title:  "Get Cooking with Chef!"
date:   2015-02-07 00:33:02
categories: Chef
---

This is from my November 214 talk at "Hack Sonoma County"

##Config Management What are our choices?

###Beanstalk or Engineyard?
* Easy
* Locks you in to thier infrastructure
* limited configurability
* Supports limited number of platforms
###Puppet
* Open source
* Enterprise version
* can work as stand alone
###Chef
* Open source Chef
* Totally free
* Gotta set up a server
* There is a recipe for  that!
###Hosted Chef
* Free for up to five nodes plans after that
* No need to set up and manage another server
* Walks you though set up of server and client.
###Amazon ops works.
* uses Chef receipes
* Ties you to amazon
* Can't do windows

##Chef Overview
###Prerequisites:
1. Set up a an open source Chef server or hosted Chef
1. Install chef client or gem install chef
1. clone chef-repo
1. git clone https://github.com/opscode/chef-repo.git
1. add .chef dir to repo and add .pem and knife files. (search for "chef-repo")
1. test by listing users
   `knife user list`
1. install the knife-ec2 plugin
1. install the necessary cookbooks (installs depenencies).
   ```
   cd chef-repo
   knife cookbook site install apt
   knife cookbook site install apache2
   ```
1. Create a new cookbook for this project
   `knife cookbook create rob_server_test`
1. push the cookbooks to the Chef server in bulk:
   `knife upload cookbooks`
1. Spin up the server with knife:
   `knife ec2 server create   --availability-zone us-east-1b   --node-name webserver_test_3   --flavor m3.medium   --image ami-98aa1cf0   --run-list "role[web_server]" -G RailsApp --fqdn webserver_test_3.windsorweb.com --ssh-user ubuntu`
1. Re-Run chef
   `knife ssh "role:web_server" "sudo chef-client" --ssh-user ubuntu`


