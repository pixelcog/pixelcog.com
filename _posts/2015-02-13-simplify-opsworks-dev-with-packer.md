---
layout:  post
title:   Simplify OpsWorks Development With Packer
image:   opsworks-vagrant-packer.png
---

After publishing a long-overdue update to the "[Virtualizing AWS OpsWorks with Vagrant]({% post_url 2014-05-08-virtualizing-aws-opsworks-with-vagrant %})" tutorial from last year, I decided to make a second post detailing some optimizations I have made to my own local development workflow.

{{ more }}

## What's New?

Right around the time I published my first tutorial on local OpsWorks development, which relied on Vagrant's [chef solo provisioner](http://docs.vagrantup.com/v2/provisioning/chef_solo.html), OpsWorks switched to a custom ['chef zero' implementation](http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-chef11.html) which allowed for features like Chef search, data bags, and Berkshelf.  As soon as they updated their built-in cookbooks to start relying on these features, my chef solo workflow stopped working unless I started implementing an increasing number of hacky workarounds.

To remedy this, I have resorted to installing the OpsWorks "[instance agent](http://docs.aws.amazon.com/opsworks/latest/userguide/agent.html)" itself, which is AWS's own provisioner that gets installed on each OpsWorks EC2 instance.  Instead of calling the chef solo provisioner through Vagrant, I could now pass some JSON in to the VM's `opsworks-agent-cli` command to kickstart chef zero.  I created a [small shell script](https://github.com/pixelcog/opsworks-vm/blob/master/opsworks/opsworks) which will install opsworks-agent and run the provisioner after performing a quick sanity check on the JSON attributes.  This is all detailed in the [revised tutorial]({% post_url 2014-05-08-virtualizing-aws-opsworks-with-vagrant %}).

## Taking It a Step Further

![Packer.io Logo](/img/posts/opsworks-packer-logo.png)

This provisioner method works wonderfully, but it still takes upwards of 15 minutes to provision a machine the first time you run `vagrant up` because it has to download and install the agent, add all of the cookbooks to the cache, and run all of the initial setup recipes before it can even get to installing and running your app.

To make this process quicker, I've created [Packer](http://packer.io/) templates for Ubuntu 12.04 and 14.04 pre-loaded with OpsWorks and pre-configured with my provisioner shell script. It is housed on GitHub under the name "[opsworks-vm](https://github.com/pixelcog/opsworks-vm)".

### Using Packer

Packer will take an Ubuntu install disk image, create a base box, install some optimizations for whichever virtualization software you specify (both VMWare or VirtualBox are supported), install the OpsWorks agent, run the OpsWorks agent once with the base recipes enabled to warm up the cache and pre-install the primary dependencies, pre-cache some common community cookbooks through Berkshelf, and then bundle the resulting image with my provisioner shell script.

In the end, using a pre-loaded box shaved off about 6 minutes from the time it took to provision the example project used in the [previous post]({% post_url 2014-05-08-virtualizing-aws-opsworks-with-vagrant %}).

## Try It Yourself!

To start creating your own OpsWorks VMs, download the latest version of Packer from [packer.io](https://www.packer.io/downloads.html), then clone 'opsworks-vm' and run one of the built-in rake tasks.

    git clone https://github.com/pixelcog/opsworks-vm.git
    cd opsworks-vm
    rake build install

> **Note**: If you prefer to build Ubuntu 12.04 rather than 14.04, you must provide the `ubuntu1204` argument (e.g. `rake build[ubuntu1204]`).  
> If you prefer to use VMWare for your guest VM, you can build a box for it by prefixing the commands with `vmware:` (e.g. `rake vmware:build`).  

This process will take several minutes to complete.  When it is finished, you will now have a box called `ubuntu1404-virtualbox` installed through Vagrant.

To use the box, create a new Vagrantfile like so:

    Vagrant.configure("2") do |config|
      config.vm.box = "ubuntu1404-opsworks"
      config.vm.provision :opsworks, type: 'shell', args: 'path/to/dna.json'
    end

The `dna.json` file path is set relative to the Vagrantfile and should contain any JSON data you wish to send to OpsWorks Chef.

For example:

    {
      "deploy": {
        "my-app": {
          "application_type": "php",
          "scm": {
            "scm_type": "git",
            "repository": "path/to/my-app"
          }
        }
      },
      "opsworks_custom_cookbooks": {
        "enabled": true,
        "scm": {
          "repository": "path/to/my-cookbooks"
        },
        "recipes": [
          "recipe[opsworks_initial_setup]",
          "recipe[dependencies]",
          "recipe[mod_php5_apache2]",
          "recipe[deploy::default]",
          "recipe[deploy::php]",
          "recipe[my_custom_cookbook::configure]"
        ]
      }
    }

If the `scm.repository` attributes for either the deployments or custom cookbooks represent a local path, it will be automatically copied into the VM and treated by the OpsWorks Chef as if it were a remote repository.

For a more detailed look at the provisioner, read my [previous post on the subject]({% post_url 2014-05-08-virtualizing-aws-opsworks-with-vagrant %}).  Also see the [example project](https://github.com/pixelcog/opsworks-vm/tree/master/example) included in opsworks-vm.

### Customizing Your Vagrant Box

If you wish to pre-load anything else in your Vagrant box, modify the [opsworks.sh file](https://github.com/pixelcog/opsworks-vm/blob/master/provision/opsworks.sh).  In it you can change which cookbooks get pre-cached by Berkshelf, and you can specify a list of recipes to run prior to compiling the box.

## Conclusion

Once again, you can clone this project and check out some example use cases on [GitHub](https://github.com/pixelcog/opsworks-vm/).

I've found this technique extremely useful in my local development workflow, and I hope it can save some other people some time and headaches as well.  Happy coding!
