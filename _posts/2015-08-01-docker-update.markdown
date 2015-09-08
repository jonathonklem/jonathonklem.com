---
layout: post
title: My Docker Info Is Antiquated
date: 2015-08-01 03:13:44.000000000 -05:00
categories:
- DevOps
tags: []
status: publish
type: post
published: true
---

I've received a decent amount of traffic on an article I wrote earlier about docker [here](/devops/up-and-running-with-vagrant-and-docker/) which is now pretty much completely useless.  This article was written shortly after Ohio Linux Fest and I was super stoked to mess around with these sweet new tools.

Some of what I wrote about has become obsolete, other parts were just a little half-baked.  I've come to this conclusion after having worked with docker in more detail.

In the aforementioned article I made the mistake of pushing everything into one docker container--treating it exactly like a virtual machine and loading up an Apache and a MySQL daemon.

The glory of docker is that you can isolate your daemons and have them function independently of one another in their own separate containers.  The proper way to set this up would be to have a container for Apache, MySQL and then the Data volume itself.  Rather than having to upgrade a daemon, you can create a new container with the latest version and test it out then hook up the proper volumes and remove the old container. 

First we're going to create the MySQL container:

```
{%raw%}docker run --name master-mysql -e MYSQL_ROOT_PASSWORD=muhPassword -d mysql{%endraw%}
```

Now we'll create our data only container:

```
docker run -v /var/www/html --name web-data busybox
```

By using -v we're exposing a volume at /var/www/html.  This means that another container can use this volume if we supply the right parameters.

Next we start up our WordPress container:

```
{%raw%}docker run --name wordpress-site -e WORDPRESS_DB_PASSWORD=muhPassword --link master-mysql:mysql --volumes-from web-data -p 8080:80 -d wordpress{%endraw%}
```

This command uses '-e' to set the environment variable so the container knows how to authenticate to our MySQL database.  The --link option connects our WordPress container to the MySQL container as if they're networked together.  The --volumes-from option tells the WordPress container to use the volume-only container "web-data", which will automatically "mount" it at /var/www/html.  Lastly the -p option maps the host's port 8080 to the container's port 80 so we can connect to it from the host's browser (or elsewhere). Just like in the MySQL example the "-d" flag is used to run it in detached mode.

If you navigate to localhost:8080 you should see the WordPress install page.  We now have a functioning containerized WordPress site!  

If you make changes to the database, those changes will be lost unless you do a `docker commit master-mysql` to save the state of the database.  The volume-only container will not use a layered file system, so you don't have to commit changes made to it.  With Docker volumes, the volume only persists so long as there is a running container using it.  You can use `docker export` and `docker import` to backup and restore your docker volume:

```
docker export --output="volume.tar" web-data
```

```
cat volume.tar | docker import - new_image
```

Lastly, nsenter is no longer necessary to connect to a running docker container.  Now there is a native `docker exec` that can be used to shell in to your container:

```
docker exec -it wordpress-site bash
```

In my next article I'll break down the process of managing these docker containers and go further into detail about backup, restoration, and upgrading.
