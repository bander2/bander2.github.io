---
layout: post
title: "Drupal 8 Development with Docker"
date: "2017-04-26 12:08:32 -400"
tags: drupal, devops, docker
---

I want to show you how I get up and running quickly with a Drupal 8 development
environment. You need [Docker](https://docs.docker.com/engine/installation/).

First bring up a database container:

```
$ docker run -d --name drupal_db \
    -v $(pwd)/.data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=root \
    -e MYSQL_DATABASE=drupal \
    -e MYSQL_USER=drupal \
    -e MYSQL_PASSWORD=drupal \
    mysql
```

Then bring up the web container and link it to the database:

```
$ docker run -d --name drupal_web \
    --link drupal_db:drupal_db \
    -p 80:80 \
    -v $(pwd)/drupal:/var/www/drupal \
    banderson/drupal
```

You can see the Dockerfile for the image
[on my github account](https://github.com/hcpss-banderson/docker-drupal).
Basically, it installs PHP, Apache, [Composer](https://getcomposer.org/),
[Drush](http://www.drush.org/en/master/) (with a @dev alias for your site) and
[Drupal Console](https://drupalconsole.com/).

At this point you have a website capable of serving Drupal, but not Drupal
itself. So let's get it:

```
$ docker exec -it drupal_web composer \
    create-project drupal-composer/drupal-project:~8.0 . \
    --stability dev \
    --no-interaction
```

And let's install Drupal with Drush:

```
$ docker exec -it drupal_web drush @dev site-install -y \
    --account-pass=admin \
    --db-url=mysql://drupal:drupal@drupal_db/drupal
```

Now you should have a fresh Drupal install at [localhost](http://localhost). The
admin password is *admin*. All the Drupal files will be located in ./drupal.
