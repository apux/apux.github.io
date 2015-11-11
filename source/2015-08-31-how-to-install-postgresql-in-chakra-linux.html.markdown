---
title: How to Install Postgresql in Chakra Linux
date: 2015-08-31 03:49 UTC
tags: [postgresql, chakra]
---

The first thing we need to do is install the package `postgresql`.

    $ sudo pacman -S postgresql

Now, we can run the server by executing the service as root:

    $ sudo systemctl start postgresql.service
    Job for postgresql.service failed because the control process exited with error code. See "systemctl status postgresql.service" and "journalctl -xe" for details.

It seems that our server couldn't start. New versions of postgresql in chakra
needs a process to be run previously. To do that, we need to login as postgres
user.

    $ sudo su postgres

Now, we need to initialize the database with the configuration we want. Change
es_MX to en_US for example.

    [postgres]$ initdb --locale es_MX.UTF-8 -E UTF8 -D '/var/lib/postgres/data'

Given that we are logged in as postgres user, we can create the rest of the users right
now. The easiest way to do that is interactively.

    [postgres]$ createuser --interactive

Now, as normal a user (not postgres), we can start our server.

    $ sudo systemctl start postgresql.service

If we don't see any errors, it means that the server is running and accepting
connections. Enjoy!
