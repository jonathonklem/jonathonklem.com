---
layout: post
title: Up And Running With Vagrant And Docker
date: 2014-10-18 22:04:17.000000000 -05:00
categories:
- DevOps
tags: []
status: publish
type: post
published: true
---

[Check out my later blog post for more accurate and up-to-date information on docker](/devops/docker-update/)

It's not uncommon for a new developer to join a project and be required to set up a local dev environment.  Depending on the company, there may be little instructions about how they want him to do this.  More often than not the most straight-forward way to get this dev environment up and running is to install an XAMP server, then download the latest files from production and copy over the database information.  After all the information is in place, this will usually require a couple hours of seeing a broken website and going back and forth checking error messages and installing required modules.

Other than the wasted time of setting up this dev environment, the problem with this scenario is that it's error prone and even if the dev gets the site running there's no assurance that he's using the same version of the modules that are being ran on the live site which can cause problems down the road.  There's also the issue of copying over the database information.  If it's a production server that has ecommerce, there's a chance that the dev may end up copying live user information to an insecure location.

Vagrant and docker help end the problems of human error, version mismatch, and keeping production data safe.  [Vagrant](https://www.vagrantup.com/) is a tool that uses Ruby to build and provision virtual machines.  It requires [Virtual Box](https://www.virtualbox.org/) to spin these machines up.  The great thing about Vagrant is you can automate the set up and provisioning process so that a developer can pull some scripts from a repository and all they need to get the development environment running is to type this single command:

```vagrant up```

There are two public repositories that will be used in this tutorial located [here](https://github.com/jonathonklem/vagrant_provisioning) and [here](https://github.com/jonathonklem/fake_website).  The second one (fake_website) actually gets cloned in the docker container we're going to set up, so the only one you need to clone is the vagrant_provisioning repository.  If you have virtual box, git, and vagrant installed on your computer you can run these three commands to spin up the dev environment:

```
git clone https://github.com/jonathonklem/vagrant_provisioning.git
cd vagrant_provisioning
vagrant up
```

<p>The `vagrant up` command will take a while to run. It has to download several image files and clone a couple repositories. Once it's completed though, you will be able to open up a browser and navigate to http://10.10.10.10 and you should see this:</p>

!["vagrant-server-screenshot"](/assets/images/vagrant-server-screenshot.png)

Now that we see what the finished product looks like, let's check out why it works.  *Note: While there is no set-in-stone way you must use docker, it should be noted that we're using it in a less 'correct' way in this article.  Technically with how I'm using Docker it would have made just as much sense to just set up Apache in the Vagrant machine, however, I want to introduce docker and some basics about it because in a future article I will describe the 'correct' process of setting up volumes and having multiple docker containers communicate with each other.*

The main file that gets everything going is our 'Vagrantfile'.  It's a simple text file with some Ruby code in it to define what base image we're going to use and what type of provisioning we'll use.  We also set the IP address.   In our example we're using 'shell provisioning' which is just a bash script that runs the commands necessary to set up the system.  You can also use ansible, puppet, and chef to provision your virtual machines.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # set our base image
  config.vm.box = "trusty"
  config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"

  # set our IP
  config.vm.network "private_network", ip: "10.10.10.10"

  # Tell Vagrant how we're going to provision our virtual machine
  # here we're copying a file over and using a simple shell script
  config.vm.provision "file", source: "Dockerfile", destination: "Dockerfile"
  config.vm.provision "file", source: "connectToDocker.sh", destination: "connectToDocker.sh"
  config.vm.provision "shell", path:   "provision.sh"
end
```

The great thing about Vagrant is you can easily switch base images, so instead of using trusty we can use an earlier version of Ubuntu or a different distro entirely.  After setting the ip address, we copy our Dockerfile and our custom script for connecting to the running docker container to the virtual machine. The last line tells Vagrant the script to run to provision the system.

The next step is to provision the system.  The steps we'll take to provision the system are:

+ Update Ubuntu base system
+ Install packages we need (docker and git)
+ Build our docker container
+ Run the Apache docker container in daemon mode

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- JonKlem Docker Ad -->
<ins class="adsbygoogle"
     style="display:inline-block;width:728px;height:90px"
     data-ad-client="ca-pub-6493519324100090"
     data-ad-slot="1392066444"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

Here is our provisioning script:

```
#!/bin/bash
# Author: Jon Klem &lt;jonathonklemp@gmail.com&gt;
# Written for Ubuntu Trusty, should be adaptable to other distros.

## Variables
HOME=/root
DOCKERSCRIPT=/vagrant/connectToDocker.sh
cd $HOME

apt-get update -qq
apt-get install -yq git docker.io

# intall ns enter to enter our docker container
docker run -v /usr/local/bin:/target jpetazzo/nsenter

# build our docker container
docker build -t apache /home/vagrant

# run it in daemon mode
docker run -d -p 80:80 -t apache /usr/sbin/httpd -D FOREGROUND

# make sure our script is executable
chmod +x $DOCKERSCRIPT
```

In the first steps, we set up some variables that we use later on in provisioning our system, then we update our package manager with

```apt-get update -qq```

and install the necessary packages with

```apt-get install -yq git docker.io```

Then, we build our Docker container from the Dockerfile that we copied into /home/vagrant with our Vagrantfile. The -t argument specifies what the 'tag' that we're going to give to our docker image. Docker functions similar to Vagrant in that it allows you to specify a base image and then provision it. Unlike Vagrant, which uses a shell script to provision the system, so there's not any special syntax. Docker files are slightly different. To run a command, it follows this syntax:

```RUN command```

If you check out the Dockerfile, you'll also see that we start out with a base image:

```FROM centos:centos5```

This lets Docker know to download their standard centos5 image and build on top of that.

After building our Docker container, we install "nsenter", by running

```docker run -v /usr/local/bin:/target jpetazzo/nsenter```

The interesting aspect about docker is that the machine's are not persistent. For example, you can run a command

```touch myfile```

and then leave the docker container. When you come and run that docker image 'myfile' will not be there. You start with a base image and you can run commands inside this small virtual machine but the changes aren't saved unless you 'commit' them. Docker has a good tutorial on their website <a title="Try Docker" href="https://www.docker.com/tryit/">here</a>.  I highly recommend using that tutorial to get a feel for the limitations of docker.

Normally when running something in docker, it will run and then exit once it's done.  We start up apache in the docker container by running it in 'daemon' mode.

```docker run -d -p 80:80 -t apache /usr/sbin/httpd -D FOREGROUND```

'-d' specifies daemon mode.  The '-p 80:80' option tells it to open up port 80.  The '-t' option specifies the tag of the docker image that we want to run, after that we specify the command.  By default httpd will fork and run in the background, but due to the limitations with docker we have to tell httpd to run in the foreground.

Lastly we make our connectToDocker.sh executable. This file is used to connect to the running docker container. Due to the way docker works, if we try to run that docker image, we'll get a new container and any commands we run or changes won't affect the apache instance that's running in daemon mode. This is where 'nsenter' comes into effect. You have to give nsenter the id of the running docker instance to connect to that instance. There are 4 main steps necessary to connect with nsenter:

+ Get the docker ID of the running instance (using `docker ps -q`)
+ Then we need to get the process ID of the running docker container (using `docker inspect --format '{{.State.Pid}}' [the docker id]`)
+ Finally we enter the running instance (using nsenter --target [the process id] --mount --uts --ipc --net --pid)

Once you're inside the running container, you can use any commands as if you're ssh'd into that live machine.  Normally with docker you would make changes to the base machine and then commit the docker image, however with this example situation we're using the more familiar git route.  You can make changes to the files in /var/www/html and then commit and push those changes.  Since we're cloning a repository when we provision the docker image if you push these changes to master they will persist even if you spin up a new server elsewhere.

Now that I've explained what the provision script is doing and how Vagrant works, let's revisit spinning up this server and working with it.  In this example the Vagrant file and provision script are in a git repository, so the first step is to pull those files from the repo:

``` git clone https://github.com/jonathonklem/vagrant_provisioning.git ```

Once we have the files, we'll enter the directory and run `vagrant up` which will download the necessary images and run the provisioning script:

```
cd vagrant_provisioning
vagrant up
```

Again, this command will take some time, but thankfully it's automated and doesn't require any hands-on manipulation. Once it's actually running, again you can check it in the browser by visiting http://10.10.10.10 in your browser. To shell into this running virtual machine use:

```vagrant ssh```

This will connect you to the machine as if you're ssh'd in. You can also use putty to connect to the machine. The username and password will both be 'vagrant' by default.

Once you're inside your running virtual machine, to connect to the running docker instance you can navigate to the vagrant home directory:

```
cd /home/vagrant
./connectToDocker.sh
```

This will run the commands we discussed earlier and connect to the running docker instance and give you a shell.

From here you can hack away at the system.  The glorious part about vagrant is you can stop the virtual machine with

```
vagrant halt
```

And then you can completely remove it from your system, as if you never ran `vagrant up` with this command

```
vagrant destroy
```

If you have any questions or comments send emails to [jonathonklem@gmail.com](mailto:jonathonklem@gmail.com).  In upcoming articles I'll go further into properly setting up docker containers to work together.
