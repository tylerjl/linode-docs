---
author:
  name: Tyler Langlois
  email: docs@linode.com
contributor:
  name: Tyler Langlois
  link: https://github.com/tylerjl
description: 'An introductory tutorial for using Puppet to manage the configuration of a Linux server.'
keywords: 'linux tips,configration management,puppet,provisioning,automation,devops'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
published: ''
modified: Tuesday, May 12, 2015
modified_by:
  name: Tyler Langlois
title: Configuration Management with Puppet
---

Managing any more than a few servers can quickly become daunting without a unified, repeatable way of installing and configuring software. Configuration management is an excellent way to abstract this process into simple directives, and [Puppet](https://puppetlabs.com/) is one of the most commonly used tools around.

In this guide, we'll explore how to use Puppet to manage various aspects of a Linux server. The first few steps will explain how to do so on separate distributions, but after we get puppet running, the steps will be the same! This is part of the beauty of configuration management.

Installing Puppet
-------------------

Whether you're on an `rpm` based distribution like CentOS or an `deb` based one such as Ubuntu, Puppet Labs provides package repositories to make installing and upgrading puppet easier. Puppet Labs has [instructions regarding how to install each](https://docs.puppetlabs.com/guides/puppetlabs_package_repositories.html#open-source-repositories), but here's the abbreviated versions:

### RPM Based Distributions

If you're on a CentOS machine, determine which major version the operating system is (you can usually look in `/etc/redhat-release` to find it.) Use that major version in the following command. For example, if you are running CentOS 6:

    sudo rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-el-6.noarch.rpm

This command will install the Puppet Labs yum repository in `/etc/yum.repos.d`. The format is similar for Fedora: find the major version, and use a command like the following example, which installs the Puppet Labs repo for Fedora 20:

    sudo rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-fedora-20.noarch.rpm

### DEB Based Distributions

Whether you are running Debian or Ubuntu, find the code name of the release first (on Ubuntu, you can find this by issuing the command `lsb_release -a`, or looking at the file `/etc/debian_version` on a regular Debian installation.) Then, using that code name, issue the following commands (in this example, we're setting up repositories for a Ubuntu Precise Pangolin installation):

    wget https://apt.puppetlabs.com/puppetlabs-release-precise.deb
    sudo dpkg -i puppetlabs-release-precise.deb
    sudo apt-get update

These commands:

1. Download the release .deb package
2. Install it into the dpkg database
3. Refresh the list of available packages on your system

You now have puppet packages avilable for installation!

### Installing The Puppet Package

With the Puppet package repository set up, installing the needed packages is straightforward.

On an RPM-based Linux distribution:

    sudo yum install puppet

Or on a Debian or Ubuntu operating system:

    sudo apt-get install puppet

Basic Puppet Configuration
--------------------------

Puppet can configure your system based upon command-line arguments, but the canonical method of telling puppet how to configure your systems is to create puppet `.pp` files in a `manifests/` directory. Create that directory, and a puppet file to store the configuration that we'll be writing.

    sudo mkdir -p /etc/puppet/manifests
    sudo touch /etc/puppet/manifests/site.pp

From this point on, the following examples should be entered into `/etc/puppet/manifests/site.pp`.

Managing Built-In Resources
---------------------------

In Puppet, there are many types of built-in resources that Puppet already knows how to manage (to see every type that puppet natively supports, read [the puppet type reference documentation](https://docs.puppetlabs.com/references/latest/type.html).) We'll look at a few simple examples using some common resource types.

### Creating and Managing Users

By 'declaring' a user resource, puppet ensures that your system is in the state you define in your `.pp` files. For example, to make sure that your system has the user `alice` with some specific settings, put the following in `/etc/puppet/manifests/site.pp`:

    user { 'alice':
        ensure => 'present',
        managehome => true,
    }

`'ensure'` tells puppet to make sure the user is there, and `'managehome'` indicates puppet should make sure the user's home directory is present and configured with the right permissions. Then tell puppet to apply the manifest:

    sudo puppet apply /etc/puppet/manifests/site.pp

You'll notice output indicating that puppet has created the user. Try executing `ls /home/` to see if a home directory for `alice` has been created.

We have a user on our system managed by puppet! What if we want to change something about the user later on? If we edit the existing manifest to add an option for the user resource, puppet will find the existing user and only make the changes needed. In this example, we'll add the `'password'` attribute (this is a `crypt`-hashed password for the string `'hunter2'`):

    user { 'alice':
        ensure => 'present',
        managehome => true,
        password => '$6$Le.iA5hzHnpksO$kzgNvcajDXjZ9ccFW4idsXvR8e/qg0OYGRDSWPJHpmlEduRuhsjrwiueWlK.9KSfSx30qYMbbyrRgxzmaM3ST/',
    }

And re-apply the manifest:

    sudo puppet apply /etc/puppet/manifests/site.pp

Puppet doesn't recreate the user, just edits the existing user to add the password `'hunter2'`. This is one of puppet's strengths: it evaluates the current state of the system and only performs operations that are required by comparing the *catalog* (the results of compiling the manifest) against the current state of the operating system. If you were to re-run the `apply` command, puppet would run quickly and not modify the system at all, because the `alice` user's state matches the one you're asking for in the puppet manifest.

### Installing Packages

Puppet can also a package is in the state you ask it to be in, whether installed, absent, or kept up-to-date. For example, append the following `package` resource to the end of your manifest file:

    package { 'git':
        ensure => 'latest',
    }

And re-run puppet apply:

    sudo puppet apply /etc/puppet/manifests/site.pp

This will take a little longer, as behind the scenes, puppet is running the necessary commands to install `git`. If you were to re-run the manifest, puppet would ensure that the package is installed and at the latest version for your distribution.

Using Puppet Modules
--------------------

Thus far we've only managed resources that come pre-bundled with puppet itself. However, the [Puppet Forge](https://forge.puppetlabs.com/) hosts a multitude of *modules* that extend puppet to be able to easily manage many other types of resources.

This makes managing webservers, daemons, and many types of server software much simpler. the following examples will set up docker on your server.

**Note**: Docker is a relatively fast-moving project, so if you encounter problems, you may need to ensure your system is up-to-date. You can follow the [package management guide](docs/tools-reference/linux-package-management) to see how to keep your system updated.

### Setting up Docker

First, the docker puppet module needs to be installed. The following command installs the `docker_platform` module from puppetlabs:

    sudo puppet module install puppetlabs/docker_platform

By default, puppet installs both the module and its dependencies into the default location `/etc/puppet/modules`, where puppet can reference them when you mention them in your manifests.

### Updating the Manifest

Now that we have the puppet module available, we can use the resources it provides to us. The simple line:

    include docker

In the `/etc/puppet/manifests/site.pp` file indicates that docker should be set up on the system. Run the apply command to set up docker:

    sudo puppet apply /etc/puppet/manifests/site.pp

You should see puppet set up docker for you - check that the service is running with your distribution's service manager (`service`, `systemctl`, `upstart`, etc.)

### Configuring Docker

The docker module we installed can also manage some configuration aspects of docker. For example, we can tell puppet to ensure a docker image is installed, and run a container using that image.

    include docker

    docker::image { 'ubuntu':
        require => Class['docker'],
    }

    docker::run { 'docker_test':
        image => 'ubuntu',
        command => '/bin/bash -c "while true ; do echo The time is `date`. ; sleep 10 ; done"',
        require => Docker::Image['ubuntu'],
    }

Note a few new things about our `site.pp`:

1. The `require` lines indicate that the resource it's inside of needs the resource it's pointing at before being run. In this example, we can't pull an image until docker is installed, and we can't run a container with that image until we have the image. In many cases, puppet is smart enough to get the order right without us telling it so, but in this case, we explicitly tell puppet that some resources depend on others. For example, puppet will not configure the `docker::run` resource if the `docker::image` resource fails for one reason or another.
2. The `include` syntax is a little different than the others - this just means that we're defining the module but aren't passing it any parameters and are trusting the module's defaults.

Try running your new manifest with the docker resources:

    sudo puppet apply /etc/puppet/manifests/site.pp

Note that this may take some time for docker to pull the image you've defined.

If everything went smoothly, you should have an ubuntu container up and running, printing the date every ten seconds. Try watching the logs:

    sudo docker logs -f docker-test

Let's change the `docker::run` resource to ensure that the container is *not* running anymore to clean up after ourselves:

    docker::run { 'docker_test':
        running => false,
        image => 'ubuntu',
        command => '/bin/bash -c "while true ; do echo The time is `date`. ; sleep 10 ; done"',
        require => Docker::Image['ubuntu'],
    }

Note the `running => false` line. Re-apply the manifest will halt the container.

Let's summarize the strengths of this setup:

* There is a single file defining the setup we want: a user with a password, git installed and up-to-date, and a docker container up and running.
* All of these resources are *cross-distribution*, that is, your `site.pp` can be applied to Fedora, CentOS, or Ubuntu without modifications and should still end up with the same end state.
* The only knowledge we had to acquire was to install puppet, run puppet, and write a puppet manifest. The details of user commands, software packages, and how to start and run docker are handled by puppet under the hood.

Going Further
-------------

This example is a simple deployment of *masterless* puppet: we did not set up a puppet *master*, which allows you to 
