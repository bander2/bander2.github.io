---
layout: post
title: "Logging to the Database Sucks"
date: 2017-01-23 19:07:00 -0400
tags: moodle
---
Most PHP web applications log events to the database. Usually that's ok but I
recently found myself with a [Moodle](https://moodle.org) installation with a
20GB database, 97% of which was taken up by the logs. That made it almost
impossible to run conventional mysqldump backups or provision development
environments.

That's why I made the
[File Logstore](https://github.com/HCPSS/moodle-logstore_file) for Moodle. Just
like it sounds, it's a Moodle plugin that writes Moodle event logs to a file.
The settings are simple:

![File logstore settings](/assets/file-log-settings.png)

The logstore writes each log event as a JSON encoded object. For example:

```json
{ "eventname": "\\core\\event\\calendar_event_created", "component": "core", "action": "created", "etc...": "etc..." }
{ "eventname": "\\core\\event\\course_viewed", "component": "core", "action": "viewed", "etc...": "etc..." }
{ "eventname": "\\core\\event\\message_sent", "component": "core", "action": "sent", "etc...": "etc..." }
```

If you don't like that, it's ok. You can template the log lines by overriding
the file_log.mustache template in your theme. For example you could format your
logs as csv rows:

{%raw%}
```mustache
{{! theme/logstore_file/templates/file_log.mustache }}
{{{ eventname }}},{{{ component }}},{{{ action }}}
```
{%endraw%}

gives you:

```csv
"\\core\\event\\calendar_event_created","core",created"
"\\core\\event\\course_viewed",core",viewed"
"\\core\\event\\message_sent","core",sent"
```

Or you could do XML or whatever you want!
