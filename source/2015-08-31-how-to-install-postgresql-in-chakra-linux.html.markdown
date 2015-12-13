---
title: How to Install Postgresql in Chakra Linux
date: 2015-08-31 03:49 UTC
tags: [postgresql, chakra]
---

One of the basic packages we need as developers is a  Database Management
System (DBMS). The one I use most is Postgresql and here I'm going to show how
to install it in Chakra Linux that probably will work on other Arch derived
distros. This installation is for development purpose only.

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
`es_MX` to en_US for example.

    [postgres]$ initdb --locale es_MX.UTF-8 -E UTF8 -D '/var/lib/postgres/data'

Now, as normal a user (not postgres), we can start our server.

    $ sudo systemctl start postgresql.service

If we don't see any errors, it means that the server is running and accepting
connections.

As postgres user, we can create the rest of the users right now. The easiest
way to do that is interactively.

    [postgres]$ createuser --interactive

We can now create a new database.

    $ createdb <DBNAME> -U <USERNAME>

Change _<USERNAME>_ with the user name you just created and <DBNAME> with the
actual database name. Note that the option `-U` should be capital letter.

We can access our database by typing:

    $ psql <DBNAME> -U <USERNAME>

We should be in postgresql prompt now and that means we succeed. Type `\q` to exit.

If we reboot our system now, we will see that postgresql doesn't start
automatically. This is because we haven't enabled it. Let's check it:

    $ systemctl is-enabled postgresql.service
    disabled

As we can see, it is disabled for autoloading at start. We can enable it easily:

    $ sudo systemctl enable postgresql.service
    $ systemctl is-enabled postgresql.service
    enabled

And we have our postgresql server ready. Enjoy!
