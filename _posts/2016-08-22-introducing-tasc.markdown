---
layout: post
title: "TASC: Let the Gerbils Assemble Your Project For You"
date: 2016-08-22 18:06:17 -0400
tags: golang, devops
---
Sometimes you just need a way to assemble some source code from various sources.
[Composer](https://getcomposer.org/) and
[Drush make](http://www.drush.org/en/master/make/) are awesome but they
only work for Drupal or Composer projects.

I needed something that would work for any kind of project and that's why I
wrote [TASC](https://github.com/HCPSS/tasc).

## What is TASC?

TASC is a generic **T**ool for **A**ssembling **S**ource **C**ode. Like a horde
of gerbils hypnotized to do your bidding, it fetches code from github, remote
zip archives and local directories to dutifully assemble your
finished project. It allows you to automate building your project so that you
can quickly provision development environments and run deployments.

Let's use a Wordpress site as an example. Let's say that we have a very simple
Wordpress blog which has a contributed theme and a single plugin. To assemble
the site with TASC, you simply write a "manifest". A manifest describes your
finished project and is a YAML file like this:

```yaml
# manifest.yml
projects:
  - provider: git
    source: https://github.com/WordPress/WordPress.git
    version: tags/4.6
    tags: ["blocking", "sticky"]

  - provider: zip
    source: https://downloads.wordpress.org/theme/basic.1.1.6.zip
    rename: basic
    destination: wp-content/themes

  - provider: zip
    source: https://downloads.wordpress.org/plugin/wordpress-seo.3.4.2.zip
    rename: wordpress-seo
    destination: wp-content/plugins
```

The project can be assembled with:

```
tasc -manifest=./manifest.yml -destination=/var/www/html
```

## Why Go?

TASC is written in the [Go](https://golang.org/) programming language. Why did I
choose Go? To reduce dependencies.

I originally wrote [TASC in PHP](https://github.com/hcpss-banderson/tasc). I was
building PHP projects and so I could count on the availability of PHP. But as I
started down some DevOps avenues (like docker) I found myself in situations
where I wanted to assemble my PHP source code without PHP. So I rewrote TASC in
Python.

But again, I ran into limitations as Python was not always installed and I
didn't want to install it just to assemble my code.

[Go](https://golang.org/) is a compiled language which is dead simple to
distribute. What does that mean? It means that in order to run TASC, all you
need is the ~8MB binary. That's it. No dependencies.

## Best Friends with Docker!

TASC and [Docker](https://www.docker.com/) are buds! Because TASC is a simple
binary, it can be easily downloaded and removed in your Dockerfile:

```
FROM ubuntu

COPY manifest.yml /srv/manifest.yml

RUN buildDeps="wget git" \
    && apt-get update \
    && apt-get install -y $buildDeps ca-certificates --no-install-recommends
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && wget https://github.com/HCPSS/tasc/releases/download/v0.2.2/tasc-linux-amd64 \
        -O /usr/local/bin/tasc --secure-protocol=TLSv1 \
    && chmod u+x /usr/local/bin/tasc \
    && tasc -manifest=/srv/manifest.yml -destination=/var/www/html \
    && rm /usr/local/bin/tasc \
    && apt-get purge -y --auto-remove $buildDeps
```

## Getting TASC

Because TASC is written in Go, all you have to do is pick up a binary from the
[releases](https://github.com/HCPSS/tasc/releases). Or if you have Go installed:

```
go get github.com/HCPSS/tasc
```
