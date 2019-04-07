---
layout: post
title:  "Launching and Bootstrapping a server on EC2"
date:   2015-02-24 13:00
categories: chef devops
---

In the [previous post]({{page.previous.url}}) we setup a chef server and set up our workstation to control that server and EC2 instances on
Amazon.  In this post we will install and manage some Chef "cookbooks" and and actually use knife to start up and
configure an EC2 instance.  Once your chef repo has been set up and knife ec2 has been proven to work we can get going
on our server setup.

##Install the necessary cookbooks
First we must prepare the cookbook directory, it must be a git repo:
   {% highlight bash %}
   $ cd cookbook
   $ git init
   $ git add ./
   $ git commit -a -m "initial commit"
   {% endhighlight %}

   Now we can install the cookbooks that we need, an index of these cookbooks are kept in the
   [chef 'supermarket'](https://supermarket.chef.io/).

   {% highlight bash %}
   $ cd ~/chef-repo

   #install the apt and apache cookbooks from the chef 'site'
   $ knife cookbook site install apt
   $ knife cookbook site install apache2
   {% endhighlight %}

If you now display the contents of the 'cookbooks' directory you will note that it now contains cookbooks for not only
'apt' and 'apache2' but also 'iptables' and 'logrotate' which the apache2 cookbook is dependant upon.
 {% highlight console %}
 $ ls -l cookbooks/
 total 8
 drwxr-xr-x  10 rmartin  staff  340 Feb 23 20:28 apache2
 drwxr-xr-x  12 rmartin  staff  408 Feb 23 20:27 apt
 -rw-r--r--@  1 rmartin  staff  588 Feb 20 23:50 chefignore
 drwxr-xr-x   9 rmartin  staff  306 Feb 23 20:28 iptables
 drwxr-xr-x  22 rmartin  staff  748 Feb 23 20:28 logrotate
  {% endhighlight %}

## Create a new role
Roles serve as templates for a single function that exists across an organization, a virtual machine can have one or
many roles.  (For instance it could be both a web server and a caching server or a database server.):

 {% highlight bash %}
   $ knife role create webserver
 {% endhighlight %}

Knife will now start up your chosen editor, vim, with a template for a role. This template is JSON.  What we must do is
fill in the "run_list" with all of the recipes needed to turn a blank machine into a web server.  We can add
the recipes here but for demonstration purposes we will add one new recipe that will contain all of the others.
The run list is a comma delimited string containing the recipes and other roles:
`"run_list": ['recipe[cookbook_name::recipe_name],role[role_name]']` :

 {% highlight json %}
{
  "name": "webserver",
  "description": "",
  "json_class": "Chef::Role",
  "default_attributes": {

  },
  "override_attributes": {

  },
  "chef_type": "role",
  "run_list": ["recipe[mycookbook::webserver]"],
  "env_run_lists": {

  }
}
{% endhighlight %}

##Create a cookbook
Now that we have a role we will need to complete the cookbook and recipe to fill that out. Again, we turn to knife:

{% highlight console %}
  $ knife cookbook create mycookbook

  $ ls -l cookbooks/mycookbook
  total 24
  -rw-r--r--  1 rmartin  staff   467 Feb 24 11:25 CHANGELOG.md
  -rw-r--r--  1 rmartin  staff  1480 Feb 24 11:25 README.md
  drwxr-xr-x  2 rmartin  staff    68 Feb 24 11:25 attributes
  drwxr-xr-x  2 rmartin  staff    68 Feb 24 11:25 definitions
  drwxr-xr-x  3 rmartin  staff   102 Feb 24 11:25 files
  drwxr-xr-x  2 rmartin  staff    68 Feb 24 11:25 libraries
  -rw-r--r--  1 rmartin  staff   284 Feb 24 11:25 metadata.rb
  drwxr-xr-x  2 rmartin  staff    68 Feb 24 11:25 providers
  drwxr-xr-x  3 rmartin  staff   102 Feb 24 11:25 recipes
  drwxr-xr-x  2 rmartin  staff    68 Feb 24 11:25 resources
  drwxr-xr-x  3 rmartin  staff   102 Feb 24 11:25 templates
{% endhighlight %}

As you can see we have created the entire infrastructure for a cookbook.  You will define your recipes in the 'recipes'
directory, adding any supporting files libraries, definitions etc into their appropriate directories.  We will keep this
demonstration simple and add a simple recipe that sets up apache and adds a test html file that we can call.

In your `cookbooks/mycookbook/recipes` directory create a new file webserver.rb open this file in your favorite
text editor.  This file will contain the code for your webserver recipe, Add the following code:

{% highlight ruby%}
#we can keep some shared resources on S3 and copy them when needed.  This works great for install files.
S3_URL = "https://s3.amazonaws.com/astrology-test/images/"

#Here we are telling Chef to install the apache2 server with a package manager: apt-get on ubuntu, yum on centOS
package 'apache2'

#We installed it now lets start it up.
service 'apache2' do
  action [:start, :enable]
end

#Lets write some content to a file to test the web server
file '/var/www/html/index.html' do
  content '<html>
  <body>
    <h1>hello world!!</h1>
<img src="image.gif"/>
  </body>
</html>'
end

#Download a file from S3 and save it.
  remote_file "/var/www/html/image.gif" do
    source "#{S3_URL}hello.gif"
  end
{% endhighlight %}

This code is written in Ruby but is specifically the Chef Recipe "Domain Specific Language" (DSL).  This language is
quite rich, allowing you do do many different things.  This is just a "taste" of what it possible.  You can see more
here: [https://docs.chef.io/chef/dsl_recipe.html](https://docs.chef.io/chef/dsl_recipe.html).


Also you will need to add your dependencies to the metadata file `cookbook/mycookbook/metadata.rb`:

{% highlight ruby%}
name             'mycookbook'
maintainer       'YOUR_COMPANY_NAME'
maintainer_email 'YOUR_EMAIL'
license          'All rights reserved'
description      'Installs/Configures mycookbook'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '0.1.0'
depends "apache2"
depends "apt"
{% endhighlight %}


## Push the cookbooks to the Chef server in bulk:
We can now do a bulk upload to the Chef server of all of the cookbooks in our local repo.  When you do this Chef will
run a basic "lint" test on the cookbooks and metadata and will inform you of any errors.
{% highlight bash %}
  $ knife upload cookbooks
{% endhighlight %}

## Spin up a Web server with knife:
At last! Everything is in place so that we can "spin up" a basic webserver using our knife "oneliner"
(don't worry we will break this down):

&nbsp;
{% highlight bash %}
  $ knife ec2 server create   --availability-zone us-east-1b   --node-name webserver_test_1   --flavor t1.micro   --image ami-84562dec   --run-list "role[webserver]" -G sec_my_webserver --fqdn webserver_test_1.windsorweb.com --ssh-user ubuntu
{% endhighlight %}
&nbsp;

* So the first part you get, right? `knife ec2 server create` We are creating a new server on ec2.
* `--availability-zone us-east-1b` The "Availability Zone" is the data center we are creating the instance in, in this
case the "b" datacenter in Virginia.
* `--node-name webserver_test_1` This names the "node" in the chef server so that it can track it.
* `--flavor t1.micro` The AWS EC2 size and type of instance.  You can get a list with `knife ec2 flavor list`
* `--image ami-84562dec` This is the name of the image that you wish to launch.  This is a standard ubuntu 14.04 instance.
* `--run-list "role[webserver]"` The "run list is the list of roles that you want the server to take and also additional
recipes
* `-G sec_my_webserver` This is the name of security group to use. (Make sure port 22 it is accessible from your ip address.)
* `--fqdn webserver_test_1.windsorweb.com` This is the full domain name for your server it get created with this name.
* `--ssh-user ubuntu` this is the ssh user that knife uses to communicate with your server.  by default the ubuntu
instances are created with an 'ubuntu' user that has full admin privileges.

## Update, Improve, Refactor your server
Now that we have a basic server up and running, the basic work flow to improve upon it is easy.

1. Add to your Recipe and add other recipes to your cookbook.
2. Run: `knife cookbook upload mycookbook` This uploads your cookbook to the chef server.
3. Run: `knife ssh "role:webserver" "sudo chef-client" --ssh-user ubuntu`

This last command is important you can use the `knife ssh` to run arbitrary shell commands on your remote server.  In
this case we are running the "chef-client" command on every server with a role of 'webserver'.

{% highlight bash %}
  $ knife ssh "role:web_server" "sudo chef-client" --ssh-user ubuntu
{% endhighlight %}


##Security
There are several key points to remember in regards to security:

1. Don't allow your server to be ssh accessible from IP addresses other than your own.
2. In a production environment best practice would be to set up a VPC and connect to it via VPN.  Allowing access to
your machines from your load balancer or other local Private IP addresses withing your VPC.
3. It is important to remember that the contents of your .chef directory includes some very sensitive data including
the keys to your AWS kingdom.  Be sure to add the entire .chef folder to your .gitignore file.

###Chef Docs

* [Set up a multi node cluster on EC2](https://learn.chef.io/legacy/starter-use-cases/multi-node-ec2/)
* [Knife configuration file settings](https://docs.chef.io/config_rb_knife.html)
* [Knife role parameters](https://docs.chef.io/knife_role.html)
* [Knife cookbook command](https://docs.chef.io/knife_cookbook.html)
* [Knife ec2 plugin](https://docs.chef.io/plugin_knife_ec2.html)
* [The official listing of chef cookbooks](https://supermarket.chef.io/)
* [EC2 availability zones](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)