---
layout: post
title:  "Setting up a Chef Server 12"
date:   2015-02-24 12:00
categories: chef devops
---


You can download and install a version of the  chef server inside your Amazon VPC.  The quickest way to get
started is to spin up an ubuntu instance download the chef debian package and install from that.

Start by going to the AWS Console and Launching an instance of ubuntu 14.04 ami-9a562df2 and ssh into it, note the ip
address you will need that later.


{% highlight bash %}
# switch to home folder
$ cd ~
# change the host name to a fqdn
$ sudo -i echo chef.your-doman.net > /etc/hostname


# write your public ip address and fqdn to your hosts file
# or you can set up dns for your server but you need a fqdn
$ sudo -i echo <your public ip> chef.your-doman.net > /etc/hosts


# Download the Chef Server Package
$ wget https://web-dl.packagecloud.io/chef/stable/packages/ubuntu/trusty/chef-server-core_12.0.3-1_amd64.deb

# Install the Chef Server
$ sudo dpkg -i chef-server*

# reconfigure the service for your machine
$ sudo chef-server-ctl reconfigure
{% endhighlight %}



Run the following command to create an administrator:
{% highlight bash %}
$ chef-server-ctl user-create user_name first_name last_name email password --filename FILE_NAME
{% endhighlight %}

An RSA private key is generated automatically. This should be saved to a safe location. The --filename option will save the RSA private key to a specified path.
For example:

{% highlight bash %}
$ chef-server-ctl user-create stevedanno Steve Danno steved@chef.io abc123 --filename /path/to/file.key
{% endhighlight %}

Run the following command to create an organization:

{% highlight bash %}
$ chef-server-ctl org-create short_name full_organization_name --association_user user_name --filename FILE_NAME
{% endhighlight %}

The organization name must begin with a lower-case letter or digit, may only contain lower-case letters, digits, hyphens, and underscores, and must be between 1 and 255 characters. For example: chef.

The full organization name must begin with a non-white space character and must be between 1 and 1023 characters. For example: Chef Software, Inc..

The --association_user option will associate the user_name with the admins security group on the Chef server.

An RSA private key is generated automatically. This should be saved to a safe location. The --filename option will save the RSA private key to a specified path.

For example:

{% highlight bash %}
$ chef-server-ctl org-create chef Chef Software, Inc. --association_user stevedanno --filename /path/to/file.key`
{% endhighlight %}

*Note:* As of version 12 you now need to install the web management interface separately. This is  now a premium feature
that you will need to pay for if you use it longer than 30 days
{% highlight console %}
sudo chef-server-ctl install opscode-manage
{% endhighlight %}

References:

 * [http://cloudacademy.com/blog/getting-started-with-chef-on-amazon-aws/](http://cloudacademy.com/blog/getting-started-with-chef-on-amazon-aws/)
 * [https://docs.chef.io/install_server.html](https://docs.chef.io/install_server.html)

##Hosted Chef
Another very good option is to use the hosted Chef services available from Ops code.  You can get started for free
managing up to 5 nodes.  Start by going here: [https://manage.chef.io/signup](https://manage.chef.io/signup).
You will find that the setup screens are quite helpful and will walk you through the process of setting up an
organization, and installing the chef client application. Just start by downloading the "starter kit".  This will
generate a new key pair for you. This key pair will automatically be put in the chef repo you are downloading.

## Install the Chef client.
There are two main ways to get up and running.  If you are a frequent user of Ruby and don't mind adding a few new gems,
you can simply install the chef gem.
{% highlight bash %}
$ gem install chef
{% endhighlight %}

The recommended method, however, is to install the [Chef Development Kit.](http://downloads.chef.io/chef-dk/).
Download and install the version for your operating system. __(most *nix systems it installs in /opt/chef)__.
Now, validate that it is properly installed with the command:
{% highlight bash %}
$ chef validate
{% endhighlight %}

## Clone the Chef Repo
In order to manage chef nodes you must have a copy of the chef-repo.  If you are using Hosted Chef and downloaded and
unzipped the "Starter Kit" you should be good to go.  Otherwise you should clone the chef-repo from github and add some
config files:
{% highlight bash %}
# clone repo
$ git clone https://github.com/chef/chef-repo.git

# create the .chef dir.
$ cd chef-repo
$ mkdir .chef
$ cd .chef

#copy the key file created when you added the user and org to the server
$ cp /path/to/my_keyfile/username.pem ./
$ cp /path/to/my_keyfile/orgname.pem ./

#install knife
$ gem install knife

#create a knife.rb configuration file
$ cd ..
$ knife configure
{% endhighlight %}

Now open your favorite editor and edit the .chef/knife.rb file.

As a minimum you will need:
{% highlight ruby linenos %}
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                "your_username"
client_key               "#{current_dir}/your_username.pem"
validation_client_name   "your_org_name-validator"
validation_key           "#{current_dir}/your_org_name-validator.pem"
chef_server_url          "https://api.opscode.com/organizations/your-org-name"
syntax_check_cache_path  "#{ENV['HOME']}/.chef/syntaxcache"
cookbook_path            ["#{current_dir}/../cookbooks"]
knife[:aws_access_key_id] = 'ACCESSKEYID' #Random all caps
knife[:aws_ssh_key_id] = 'your amazon keyname' #the name of your keyfile you are using for access to ec2 servers
knife[:aws_secret_access_key] = 'Your_Amazon_secret_access_key'
knife[:editor] = "/usr/bin/vim"
{% endhighlight %}

Again this should mostly be automatically set up by the starter kit if you used it.
Be sure to note the 'aws_' params abve and replace them with yours.

Now confirm that you are set up correctly and install the knife-ec2 plugin:
{% highlight bash %}
$ knife user list

#now set up the knife-ec2 client
# if you used the installer to install the chef client:
$ /opt/chef/embedded/bin/gem install knife-ec2

# if you used 'gem install chef':
$ gem install knife-ec2

#check to make sure it is installed and configured properly:
$ $ knife ec2 flavor list
{% endhighlight %}

References:

* [knife configuration parameters.](https://docs.chef.io/config_rb_knife.html)
* [knife-ec2 plugin docs.](https://docs.chef.io/plugin_knife_ec2.html)
* [Managing Amazon EC2 Instances with knife.](http://www.agileweboperations.com/amazon-ec2-instances-with-opscode-chef-using-knife)


