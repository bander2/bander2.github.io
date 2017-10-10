---
layout: post
title: From FTP to MAMP to Vagrant to Docker
date: "2017-02-23 18:06:17 -0400"
tags: devops, mamp, docker, vagrant
---

I, like many PHP developers, started out writing PHP files locally and uploading
them to a development server to run. When I found out that Dreamweaver could
automatically upload my file when I saved it - 200% productivity gain!

Believe it or not, there are even better ways to develop PHP. I am going to
go over a list of (IMHO) better and better ways to develop PHP applications and
websites.

## Why Develop Locally

The biggest jump you'll get in productivity is moving from running your PHP
application on a development server to running it on your local machine. Why?

  1. Updates to your files are immediately reflected on the server.
  2. No more worrying about the inevitable problems syncing libraries.
  3. No more worrying about the integrity of the development server which may be
     used by multiple developers.

## Why Use MAMP?

[MAMP](https://www.mamp.info/en/) stands for Mac (or My), Apache, MySQL and PHP.
What I am going to say about MAMP is also relevant to
[XAMPP](https://www.apachefriends.org/) and
[WAMP](http://www.wampserver.com/en/). These are all packages that make it easy
to install PHP, Apache, and MySQL on your local machine.

The great thing about MAMP is that it's easy. In addition to MySQL and Apache,
you get multiple versions of PHP, phpMyAdmin, and a control panel to change
settings.

So, whats wrong with MAMP?

  1. It's difficult to configure if you need to. Let's say you need to install a
     PHP package that it does not come with? Don't even try. It's a nightmare.
  2. You get multiple versions of PHP, but let's say you change the
     configuration of one of them, your are stuck with that configuration for
     all your projects that use that version of PHP.
  3. However you configure PHP, Apache and MySQL, it is going to be impossible
     to configure them in a way that is even similar to the production
     environment.
  4. It's difficult to add other technologies to your stack. Like Memcache,
     Redis or Logstash

## Why Switch to Vagrant?

[Vagrant](https://www.vagrantup.com/) is a tool for building virtual machines
(VMs). With vagrant, instead of having a single server (your local development
box) for all your projects, each project can have it's own virtual server, with
it's own copy of PHP configured just for it.

Pros:

  1. You can configure your development environment to be more like, or exactly
     like, your production environment.
  2. You can more easily experiment with other technologies. If you want Redis,
     you just install Redis. NGINX? No problem.
  3. Different projects can use different configurations for PHP.

Cons:
  1. You no longer get phpMyAdmin automatically. You can set it up, but it's not
     automatic. Personally, once I started using MySQL's CLI client, I never
     looked back.
  2. The virtual server can take a long time to provision. You are generally
     starting with a pretty bare Linux distribution and installing all the
     packages you need via a shell script or an automation tool like
     [Ansible](https://www.ansible.com/). Sometimes you need to compile
     something from source. I've had servers that took 40 minutes to provision.
     This is especially galling when you are trying to troubleshoot something in
     the provisioning process itself (so you have to run it over and over).
  3. It is possible to use one vagrant project to bring up multiple virtual
     machines, but orchestrating them is not easy and they take up a lot of disk
     space.
  4. You need to know how to install and configure all the software you are
     going to use.

## Why Switch to Docker?

In my mind, [Docker](https://www.docker.com/) solves most of the Vagrant issues.
You can think of Docker like small, light weight, savable, VMs. Docker is built
around the idea of smaller, independent, VMs (which docker calls containers)
that interact with each other instead of one monolithic VM that has all the
software on it, like Vagrant. So instead of one container with PHP, Apache, and
MySQL on it, you would have one with PHP and Apache and one with MySQL.

Pros:
  1. Because Docker was built with orchestration in mind, it comes with a nice
     tool called [docker-compose](https://docs.docker.com/compose/overview/)
     which can be used to configure multiple containers to interact.
  2. Docker makes it much easier to save your containers in various states and
     store them remotely. This means you will spend way less time building VMs
     and you don't need to store them when you are not using them.
  3. The [Docker Hub](https://hub.docker.com/) has tons of images ready to be
     used. That means I can start using Elasticsearch without ever having to
     figure out how to install it or the JDK it runs on.
