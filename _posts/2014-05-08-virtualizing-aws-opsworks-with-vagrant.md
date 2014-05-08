---
layout:  post
title:   Virtualizing AWS OpsWorks with Vagrant
image:   opsworks-vagrant.png
---

Having recently made the switch from my old PaaS provider to [AWS OpsWorks](https://aws.amazon.com/opsworks/), I've resolved to take this opportunity to update my local development workflow along with it. My peers have been extolling the virtues of [Vagrant](http://vagrantup.com/) for years, so it's time to finally see what all the fuss is about.

{{ more }}

### Table of Contents
{:.no_toc}

* Table of Contents Placeholder
{:toc}

## Why Vagrant?

Vagrant is an awesome tool for local web development. With a few Chef cookbooks Vagrant can create a portable, consistant development environment to mirror your production server down to the last detail. No longer do you have to worry that an app which runs perfectly on your OSX machine's LAMP stack will break down when you push it to EC2 due to subtle differences in your production server's configuration. No longer do you have to juggle multiple versions of PHP or Python or Ruby on your local dev machine so that you can work on multiple projects with different dependencies.

Vagrant encloses your app within its own virtualized develpment environment, isolating it from your operating system and any other external influences and perfectly mimicing the environment in which you intend to deploy it.

### Virtualizing OpsWorks

OpsWorks uses [Chef-Solo](http://docs.opscode.com/chef_solo.html) to provision its components, so to emulate the service we will simply take the same built-in chef recipes (which Amazon provides on [GitHub](https://github.com/aws/opsworks-cookbooks)) and run them through Vagrant.  The resulting workflow will resemble the following diagram (thanks [@buccolo](https://twitter.com/buccolo)):

![Varant Workflow](/img/posts/opsworks-vagrant-diagram.png)

## Getting Started

For this walkthrough I'm going to follow Amazon's OpsWorks tutorial ["Getting Started: Create a Simple PHP Application Server Stack"](http://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted.html) and show you how to mirror the entire stack on a local machine step by step.  If you have not done so, you may want to follow their tutorial first.

There are a few minor caveats to note here.  First, Amazon Linux is not available outside of AWS so for true environment parity we need to use Ubuntu LTS 12.04 which is available universally. Make sure to select it as your *default operating system* in [step 2.1](http://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted-simple-stack.html).  Also, in this example I'm going to be using chef-solo version 11.10.  The default on OpsWorks is currently 11.4, but you can change this by clicking the "advanced" button as you create your stack.

![OpsWorks Stack Configuration](/img/posts/opsworks-vagrant-add-stack.png)

> **Note:** All of the code used in this tutorial can be found on GitHub at [https://github.com/pixelcog/opsworks-local/](https://github.com/pixelcog/opsworks-local/).  If you want to skip ahead and play around with it you can simply clone it, install the dependencies, and run `vagrant up`

## Virtualizing "Simple PHP App" Step 1 & 2

Still here?  Good.

After having completed the OpsWorks tutorial through step 2 your simple stack should look like this:

![OpsWorks Example Stack](/img/posts/opsworks-vagrant-php_walkthrough_arch_2.png)

This is what we're going to emulate first.

If you haven't done so already, you'll need to download Vagrant from from the [official website](http://downloads.vagrantup.com/) and install it, then you'll need to do the same for [VirtualBox](https://www.virtualbox.org/wiki/Downloads). Both are free tools.

Now let's create a directory to house this project.

	mkdir opsworks-local && cd opsworks-local
	mkdir dev ops

Now, lets clone Amazon's [built-in chef cookbooks](https://github.com/aws/opsworks-cookbooks) and the "[simple php](https://github.com/amazonwebservices/opsworks-demo-php-simple-app)" app from the OpsWorks tutorial into our project directory.

	git clone -b release-chef-11.10 https://github.com/aws/opsworks-cookbooks.git ops/opsworks-cookbooks
	git clone -b version1 https://github.com/amazonwebservices/opsworks-demo-php-simple-app.git dev/simple-php

In this example I'm using the *Chef 11.10* branch of opsworks-cookbooks. As I mentioned earlier this should correspond to the Chef version you use on your OpsWorks stack.

Next, we'll grab our virtualbox image of Ubuntu 12.0, install the [chef omnibus](https://github.com/schisamo/vagrant-omnibus/) plugin, and initialize our config file.

	vagrant box add precise64 http://files.vagrantup.com/precise64.box
	vagrant plugin install vagrant-omnibus
	touch Vagrantfile

Open up Vagrantfile with your text editor of choice and let's add some lines.

	Vagrant.configure("2") do |config|
	  
	  # Set our operating system
	  config.vm.box = "precise64"
	  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
	
	  # Ensure we are using the correct Chef version
	  config.omnibus.chef_version = "11.10.0"
	
	  # Configure our basic app settings
	  app_name = "simple-php"
	  app_type = "php"
	  app_root = ""
	
	  # Share our app's directory with the guest machine
	  config.vm.synced_folder "dev/#{app_name}", "/home/vagrant/app/#{app_name}"
	
	  # Forward port 80 so we can see our work
	  config.vm.network "forwarded_port", guest: 80, host: 8080

These app settings correspond to the values set in the "Add App" page on OpsWorks (see [step 2.4](http://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted-simple-app.html) of their tutorial):

![OpsWorks "Add App" Page](/img/posts/opsworks-vagrant-add-app.png)

Next, we get to the meat of the configuration, the provisioners. I've found that maunally refreshing the apt cache first is necessary otherwise the *opsworks_initial_setup* recipe tends to fail.
	  
	  # Ensure our apt cache is fresh
	  config.vm.provision "shell", inline: "apt-get update > /dev/null"

Now we get to the Chef provisioner. The following lets Vagrant know where the opsworks cookbooks are located, and then also defines a "roles" path. More on this in a moment.

	  # Provision our machine
	  config.vm.provision "chef_solo", id:"chef" do |chef|
	    chef.cookbooks_path = "ops/opsworks-cookbooks"
	    chef.roles_path = "ops/opsworks-roles"
	
	    chef.add_role "php-app"
	
	    chef.json = {
	      :deploy => {
	        app_name => {
	          :application => app_name,
	          :application_type => app_type,
	          :document_root => app_root,
	          :scm => {
	            :scm_type => "git",
	            :repository => "/home/vagrant/app/#{app_name}"
	          },
	          :domains => [app_name],
	          :memcached => {},
	          :database => {}
	        }
	      },
	      :opsworks => {
	        :ruby_stack => "ruby",
	        :stack => {:name => "TestStack"},
	        :layers => {}
	      }
	    }
	  end
	end

You can find the full Vagrantfile [here](https://github.com/pixelcog/opsworks-local/blob/version1/Vagrantfile).

### Roles and Attributes

The `chef.json` hash defines the attributes we pass into Chef when provisioning. This is like the "Custom JSON" field in your OpsWorks stack settings, though OpsWorks also silently adds its own attributes corresponding to your stack's current configuration — The `deploy` key contains the information needed to tell our deploy recipes where to find the app source, and the `opsworks` key contains a few other required attributes so the opsworks recipes don't complain.

OpsWorks "Layers" are essentially the same thing as Chef's "Roles".  In them we define a run-list of recipes and default attributes for them. Each of the OpsWorks built-in layers have a set of default recipes. If you select a PHP layer in the OpsWorks console and click on "recipes", you'll see something like the following:

![PHP App Default Recipes](/img/posts/opsworks-vagrant-default-recipes-php.png)

Vagrant has no concept of OpsWorks' *"[Lifecycle Events](http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-events.html)"*, so in order to provision and deploy our app, we must run the recipes within the *Setup*, *Deploy*, and *Configure* events in that order.

Let's create our "php-app" role now.

	mkdir ops/opsworks-roles
	touch ops/opsworks-roles/php-app.json

Open up *php-app.json* in your editor of choice and add the following:

	{
	  "name": "php-app",
	  "description": "OpsWorks recipe run-list for the PHP app layer",
	  "default_attributes": {
	  },
	  "run_list": [
	    "recipe[opsworks_initial_setup]",
	    "recipe[mysql::client]",
	    "recipe[dependencies]",
	    "recipe[opsworks_ganglia::client]",
	    "recipe[mod_php5_apache2]",
	    "recipe[deploy::default]",
	    "recipe[deploy::php]",
	    "recipe[opsworks_ganglia::configure-client]",
	    "recipe[php::configure]"
	  ],
	  "chef_type": "role",
	  "json_class": "Chef::Role"
	}

Note that I've left out *'ssh_host_keys'*, *'ssh_users'*, *'ebs'*, and *'agent_version'* because they don't apply here.  There is no "opsworks-agent" installed here, nor do we have EBS volumes, and we don't need to add any additional users to our development box.

If you're lazy, you can clone all of this from GitHub at:  
[https://github.com/pixelcog/opsworks-local/tree/version1](https://github.com/pixelcog/opsworks-local/tree/version1)

### Hello World

Now that we have configured our Vagrant stack, lets run it and test it.

	vagrant up

It'll take a few minutes to provision everything, so go grab a coffee. If all goes well, you can then type in [localhost:8080](http://localhost:8080) into your browser and voilà!

![Successful Vagrant Implementation](/img/posts/opsworks-vagrant-success-v1.png)

Make sure to run `vagrant halt` and `vagrant destroy` when you are done to clean up and remove the virtual disk image.

## Virtualizing "Simple PHP App" Step 3 - Adding a MySQL Layer

Well that was neat and all, but it's not really much of an app...  Things get a lot more interesting when you add a database layer and a second virtualized machine.

After you complete [step 3](http://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted-db.html) of the OpsWorks tutorial your stack should look like this:

![OpsWorks Example Stack](/img/posts/opsworks-vagrant-php_walkthrough_arch_3.png)

How do we model all of this in Vagrant? First, lets update our simple-php app to *version2* and clone the [opsworks custom cookbooks](https://github.com/amazonwebservices/opsworks-example-cookbooks) example from the tutorial into our 'ops' directory.

	cd dev/simple-php
	git checkout version2
	cd ../..
	git clone -b master git@github.com:amazonwebservices/opsworks-example-cookbooks.git ops/opsworks-example-cookbooks

Next, lets add a new role declaration for the MySQL layer.  Here are the OpsWorks default recipes for the built-in MySQL layer:

![MySQL Layer Default Recipes](/img/posts/opsworks-vagrant-default-recipes-mysql.png)

Create a new file at 'ops/opsworks-roles/db-master.json' and add the following:

	{
	  "name": "db-master",
	  "description": "OpsWorks recipe run-list for the MySQL layer",
	  "default_attributes": {
	  },
	  "run_list": [
	    "recipe[opsworks_initial_setup]",
	    "recipe[mysql::client]",
	    "recipe[dependencies]",
	    "recipe[opsworks_ganglia::client]",
	    "recipe[mysql::server]",
	    "recipe[deploy::mysql]",
	    "recipe[opsworks_ganglia::configure-client]"
	  ],
	  "chef_type": "role",
	  "json_class": "Chef::Role"
	}

Again, we're leaving out out *'ssh_host_keys'*, *'ssh_users'*, *'ebs'*, and *'agent_version'*.

Now, lets update our Vagrantfile. First, as per the tutorial, change the `app_root` setting to "web". Then, remove the `config.vm.network` declaration since we cannot forward one port from multiple machines. Next, update our chef provisioner to the following:

	  # Provision our machine
	  config.vm.provision "chef_solo", id:"chef" do |chef|
	    chef.cookbooks_path = ["ops/opsworks-cookbooks","ops/opsworks-example-cookbooks"]
	    chef.roles_path = "ops/opsworks-roles"
	
	    chef.json = {
	      :deploy => {
	        app_name => {
	          :application => app_name,
	          :application_type => app_type,
	          :document_root => app_root,
	          :scm => {
	            :scm_type => "git",
	            :repository => "/home/vagrant/app/#{app_name}"
	          },
	          :domains => [app_name],
	          :memcached => {},
	          :database => {
	            :host => "10.10.10.20",
	            :database => app_name,
	            :username => "root",
	            :password => "correcthorsebatterystaple",
	            :reconnect => true
	          }
	        }
	      },
	      :opsworks => {
	        :ruby_stack => "ruby",
	        :stack => {:name => "TestStack"},
	        :layers => {
	          "php-app" => {
	            "instances" => {
	              "php-app1" => {"private-ip" => "10.10.10.10"}
	            }
	          },
	          "db-master" => {
	            "instances" => {
	              "db-master1" => {"private-ip" => "10.10.10.20"}
	            }
	          }
	        }
	      },
	      :dependencies => {
	        :debs => {
	          "curl" => "latest"
	        }
	      },
	      :mysql => {
	        :server_root_password => "correcthorsebatterystaple",
	        :tunable => {:innodb_buffer_pool_size => "256M"}
	      }
	    }
	  end

The settings in this chef provisioner block will be the default for both layers of the stack.  Here we've added the *opsworks-example-cookbooks* directory to our `cookbooks_path`, and fleshed out our custom JSON hash.  Notably we've expanded `opsworks.layers` to identify our virtual machines' private IP addresses and populated `deploy.database` to include the connection settings and the IP address of our database layer. The password can be [whatever you want](http://xkcd.com/936/) as long as its the same in both `deploy.database` and `mysql`.

The `innodb_buffer_pool_size` setting must be a value which will fit in memory of the virtualized environment (VirtualBox's default is 512MB).  Also we've included `curl` as a dependency since it is needed by our custom cookbooks.

This is really just a quick-and-dirty imitation of just the OpsWorks attributes which are required for this example to work.  If you wanted to be really thorough, you could flesh out `opsworks.instance`, `opsworks.stack`, `opsworks.layers`, and `deploy` with all of the attributes OpsWorks uses in production.  I've compiled an example attribute dump from a deploy lifecycle event on OpsWorks [here](https://gist.github.com/mikegreiling/63b6a74fb4d466262671#file-attributes-json).

After the chef provisioner block, we can define our virtual machines:

	  # Define our app layer
	  config.vm.define "app" do |layer|
	
	    layer.vm.provision "chef_solo", id:"chef" do |chef|
	      chef.add_role "php-app"
	      chef.add_recipe "phpapp::appsetup"
	    end
	    
	    # Forward port 80 so we can see our work
	    layer.vm.network "forwarded_port", guest: 80, host: 8080
	    layer.vm.network "private_network", ip: "10.10.10.10"
	  end
	  
	  # Define our database layer
	  config.vm.define "db" do |layer|
	
	    layer.vm.provision "chef_solo", id:"chef" do |chef|
	      chef.add_role "db-master"
	      chef.add_recipe "phpapp::dbsetup"
	    end
	    
	    layer.vm.network "private_network", ip: "10.10.10.20"
	  end

Here, we've created two virtual machines, *app* and *db*, and assigned IP addresses to them on a private network.  The app machine is assigned the "*php-app*" role followed by our custom recipe, and the db machine is assigned the "*db-master*" role followed by its own custom recipe (as in [step 3.4](http://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted-db-lifecycle.html) of the OpsWorks tutorial).

The `id:"chef"` part is important here because it specifies that we are overriding the existing chef provisioner rather than adding a new one.

The complete Vagrantfile can be found [here](https://github.com/pixelcog/opsworks-local/blob/version2/Vagrantfile).

### Hello World — Part 2

We're now ready to boot our environment.

	vagrant up

Wait a few minutes for the provisioner to do its thing and if all goes well, you can then go to [localhost:8080](http://localhost:8080) and be greeted with something similar to this:

![Successful Vagrant Implementation v2](/img/posts/opsworks-vagrant-success-v2.png)

Huzzah!

## Local Development and Deployment

Now the way deployment works here is pretty straight forward.  You can make any set of changes to the app contained within *dev/app/simple-php* and run `git commit` to modify the repository. Then you can run `vagrant provision app` to deploy the changes. This will re-execute all of the recipes on the *app* instance and apply all necessary bootstraping through the custom deploy recipes.

	cd dev/simple-php
	echo "<?php phpinfo(); ?>" > web/info.php
	git add -A && git commit -m "add phpinfo script"
	vagrant provision app

This provision process will go much faster the second time around since the only thing that is changing is the app deployment. Any changes you want to see reflected in the development environment must be committed to the repository first since the deploy recipe just takes whatever is in the repository HEAD.

Alternatively if you want to have your changes reflected immediately on the development server *without* committing and provisioning, you can log into the server and symlink the shared folder to the live environment

	vagrant ssh app
	ln -sfn /home/vagrant/app/simple-php /srv/www/simple-php/current

Note that when you do this, you'll have to do your bootstraping manually (essentially all of the stuff contained in the "*[phpapp::appsetup](https://github.com/amazonwebservices/opsworks-example-cookbooks/blob/master/phpapp/recipes/appsetup.rb)*" custom recipe). In the case of this app, you'd need to do the following (still logged into the guest machine):

	cd /srv/www/simple-php/current
	curl -sS https://getcomposer.org/installer | php
	php composer.phar install --no-dev

Then add a the file "db-connect.php" with the following:

	<?php 
	  define('DB_NAME', 'simple-php');
	  define('DB_USER', 'root');
	  define('DB_PASSWORD', 'correcthorsebatterystaple');
	  define('DB_HOST', '10.10.10.20');
	  define('DB_TABLE', 'urler');
	?>

Now you can modify whatever you want and have the changes reflected immediately.

Each of these deployment options are nice for different scenarios. When actively developing your app, saving yourself from reprovisioning each time you want to test it out is quite handy. However the former option gives you an opportunity to debug your deploy recipes if you have any.

## Going Forward / Final Notes

I'll admit that this implementation is a bit messy. Getting the required attributes to be just right involved a considerable amount of trial and error.  However this should be a good starting block for anyone looking to virtualize their OpsWorks stack for local development.

In the future I would really like to see a Vagrant plugin developed which can look at a Vagrantfile config and automatically generate the [OpsWorks custom attributes](https://gist.github.com/mikegreiling/63b6a74fb4d466262671) to match.  As it stands, there is a bit of manual synchronization involved, and if you write a custom recipe which relies on additional custom attributes, you'll need to add them to the JSON yourself.

Perhaps I will endeavor to write such a plugin if I ever find the free time.

I will follow this post up with some additional tips and tricks as I experiment more with the platform.