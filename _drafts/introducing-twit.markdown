---
layout: post
title: Introducing TWIT
tags: golang, devops
---
Stop using [sed](https://www.gnu.org/software/sed/manual/sed.html) and
[awk](https://www.gnu.org/software/gawk/manual/gawk.html) to modify files! We
are human beings for God's sake! You don't have to pretend that you like writing
regular expressions!

Because now you have TWIT, the CLI **T**ool for **W**r**I**ting **T**emplates.
You can just template those file *without* a template engine.

For example; let's say I want to provision docker image with a Drupal install.
I want the database configuration to be configurable at runtime. In the end,
I'll want a settings.php file that looks like this:

```php
<?php
$base_url = 'http://mydrupal.dev';

$databases['default']['default'] = array(
  'driver' => 'mysql',
  'database' => 'myradsite',
  'username' => 'drupal_guy',
  'password' => 'my<Strange>/\password',
  'host' => 'drupal_db',
  'prefix' => '',
);
```

I define my environment variable:

```
FROM drupal:7

ENV BASEURL "http://www.drupal.dev"
ENV DBNAME drupal
ENV DBUNAME drupal
ENV DBPASSWD drupal
ENV DBHOST localhost

COPY docker-entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

But the work of configuring Drupal at runtime will be done with the
/entrypoint.sh bash script.

One way to approach the problem is to copy a default version of settings.php and
use sed or awk to modify it:

```bash
#!/bin/bash

# /entrypoint.sh

# Copy the example file
cp sites/default/default.settings.php sites/default/settings.php

# Find the line in the file with the example base_url and replace it.
sed -i "s@# $base_url = 'http://www.example.com';@$base_url = '$BASEURL';@" sites/default/settings.php

# Write all the lines that define the db connection.
echo "$databases['default']['default'] = array(" >> sites/default/settings.php
echo "  'driver' => 'mysql'," >> sites/default/settings.php
echo "  'database' => '$DBNAME'," >> sites/default/settings.php
echo "  'username' => '$DBUNAME'," >> sites/default/settings.php
echo "  'password' => '$DBPASSWD'," >> sites/default/settings.php
echo "  'host' => '$DBHOST'," >> sites/default/settings.php
echo "  'prefix' => ''," >> sites/default/settings.php
echo ");" >> sites/default/settings.php
```

Think that will work? I doubt it. Is it easy to understand what's happening?
Nope.

Let's try it with TWIT. First we'll create a template in the
[go template format](https://golang.org/pkg/text/template/):

{% comment %}
    Got some template stuff that might confuse liquid. Hence "raw"..
{% endcomment %}
{% raw %}
```php
<?php
$base_url = '{{ .base_url }}';

$databases['default']['default'] = array(
  'driver' => 'mysql',
  'database' => '{{ .database }}',
  'username' => '{{ .username }}',
  'password' => '{{ .password }}',
  'host' => '{{ .host }}',
  'prefix' => '',
);
```
{% endraw %}

Copy it by adding this to the Dockerfile:

```
COPY settings.php.tpl /srv/templates/settings.php.tpl

# And install TWIT
RUN wget https://github.com/bander2/twit/releases/download/1.1.0/twit-linux-amd64 \
        -O /usr/local/bin/twit \
    && chmod u+x /usr/local/bin/twit
```

Then the /entrypoint.sh script can look like this:

```bash
twit \
    /srv/templates/settings.php.tpl \
    /var/www/html/sites/default/settings.php \
    -p '{"base_url": "'$BASEURL'", "database": "'$DBNAME'", "username": "'$DBUNAME'", "password": "'$DBPASSWD'", "host": "'$DBHOST'", }'
```

Will it work? Yes. Is it clear? Yes.
