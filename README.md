!NADA 
=====

## !Not ADAptive Thresholds

Version: 0.09

## WARNING!
### THIS PROJECT IS COMPLETELY EXPERIMENTAL!
#### Feedback is welcome
---

What is !NADA?
--------------

!NADA is a brand new project which intents to insert baseline adaptive thresholds
to Nagios(R) or Icinga monitoring frameworks. By "adaptive" I mean a threshold which
may change through time, accordingly to a given resource behaviour.

This project is intended those ones, who have already did a question like: 
"How can I avoid false positives when monitoring a given server that every 
Monday has a higher load average than during the other days?"


How does it work?
-----------------

!NADA requires a MySQL database running or an SQLite installation together with 
Nagios(R)/Icinga. It encapsulates your check plugin, parses and stores performance
data into DB, calculates the standard deviation and creates two new metrics, pointing 
to the top and bottom of your baseline. If collected value overflow (up or down) the 
baseline, !NADA change the plugin return code to CRITICAL thus causing Nagios(R)/Icinga
to alert.

!NADA' standard behaviour assumes that you are using a week sazonality, if it's
not appropriate, please may the source be with you.

Let's explain how it works by a simple example:

If a given check occurs just now (let's say: Monday at 11:07 PM), !NADA will
retrieve the last one hundred Monday =~ 11:07 PM check results from DB. It 
will then calculate the stardard deviation using these one hundred check 
results and make a good baseline to current check.


OK, how can I configure this thing?
-------------------------------------

Let's suppose you have a Nagios(R)/Icinga command configuration like this:

    define command{
        command_name    check_disk
        command_line    $USER1$/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
    }

You just need to change command to the following and you are ready:

    define command{
        command_name    check_disk_baseline
        command_line    /path/to/nada $USER1$/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
    }


Configure is easy, how about compile?
-------------------------------------

At these early stages I have not prepared any how-to on compilation, however
it's pretty direct and simple.

You'll need just a C compiler (of course) and mysql-devel package installed 
into your system (if you're gonna use MySQL, otherwise you need sqlite-devel). 
I left a simple `Makefile` together with the project, so if you want to adapt 
it to your system, feel free to do it, but don't forget to send me a diff. ;)

MySQL compilation (as root)

    make mysql
    make install

SQLite compilation (as root)

    make sqlite
    make install

I have sucessfully built it with:

* gcc version 4.6.3 20120306 (Red Hat 4.6.3-2) (GCC)
* Linux 3.4.2-1.fc16.x86_64 
* mysql-devel-5.5.24-1.fc16.x86_64
* sqlite-devel-3.7.7.1-1.fc16.x86_64

and also

* gcc version 4.1.2 20080704 (Red Hat 4.1.2-52)
* Linux 2.6.18-308.1.1.el5PAE
* mysql-devel-5.0.95-1.el5_7.1
* sqlite-devel-3.3.6-6

Un-installation (as root)

    make uninstall


How about configuring MySQL?
----------------------------

You will find a .sql file together with the package. You basically need to run:

    mysql -u root -p -A < database-creation.sql 


How can I configure MySQL database user/password?
-------------------------------------------------

You can change the user, password and host to your MySQL server by editing
baseline.ini at root source directory before installing or, after running 
"make install", edit it under /opt/nada/.

Important - baseline.ini explained
----------------------------------

* `minentries=X`

    If !NADA couldn't find at least X entries on its database, it will
    not apply baseline calculation to determine check state. In this 
    specific case, !NADA will return the same code returned to it by
    plugin execution. As we need a reasonable amount of historic data
    to be able to calculate the standard deviation, I chose to let it 
    at user choise what's the minimun number of entries before start to
    actually change plugin's output. X must be an integer.

    Example:
    If you set this value to 10, using !NADA in a service which checks
    at every 5 minutes, it will take =~ 10 weeks(considering a sazonality
    value of 7 - see bellow) to actually start applying baseline 
    calculation.

    Otherwise if you have a service wich checks at every minute, !NADA 
    probably will start to calculate the baseline within =~ 2 weeks(again
    considering a sazonality value equal to 7). This  is an algorithm side 
    effect: internally, on query execution, it applies by default a 
    tolerance of 5 minutes when retrieveng data of last  weeks, so, for 
    example, a execution today Fri 14:30:00 will fetch data from all last
    Fridays between 14:28:30 and 14:32:30.

* `maxentries=X`

    The opposite of the option above. This determines the maximum 
    number of historic data retrieved to calculate baseline. Pay 
    attention to the performance issues implied with this specific 
    option. X must be an integer.

* `sazonality=X`

    With this option, you specify in days how long your service sazonality
    is. So, for example, if your monitored service has a behaviour which
    repeats itself every day(a server which load has a considerable 
    increase every midday) you may define X to 1. In the other hand, if 
    your service has an increase, let's say, every monday, you may define 
    sazonality to 7(in other words, one week). X must be an integer.

* `tolerance=X`

    After standard deviation calc, a tolerance index is applied before
    define resource's top and bottom limits. X must be an integer, 
    and it should represent an acceptable percentage tolerance on
    monitored resource usage.

* `allownegatives=[yes|no]`

    This options defines if after the calc for deviation has been made,
    the value for bottom boundary will be able to remain below 0(zero).
    If this option is defined to 'no', and the bottom boundary reamains
    below zero, bottom line will become 0.

* `baselinealgorithm=[exponential_smoothing|standard_deviation]`

    Define algorithm to baseline calculation. Avaible algorithms by now
    are "Standard Deviation" or "Simple Exponential Smoothing", try
    both to see which fits better to your monitored resource.

`[DATABASE]`

* `host=localhost`
    MySQL server's IP address.
    This parameter is ignored in case of SQLite.

* `user=root`
    MySQL user with write/read permissions to `nada` database.
    This parameter is ignored in case of SQLite.

* `password=mypass`
    MySQL users's password.
    This parameter is ignored in case of SQLite.

* `dbname=nada`
    For MySQL, specify database name(default nada).
    For SQLite, it points to valid path where SQLite gonna be
    created(i.e. dbname=/tmp/nada)


Database Management
-------------------

Included with this package there's an executable that should be scheduled to
run once a day in your crontab. This executable clean all data that NADA 
doesn't need anymore, avoiding dabase uncontrolled grow.

To correctly schedule database clean up process, just use line below on your
crontab configuration:

    5 0 * * * root /opt/nada/purge-db-data >/dev/null 2>/dev/null


Why in the hell C?
------------------

The right answer is: ***because I like it***.
You may find a lot of resources pointing that <put you hype language here> is
far better than C, but really, really, common! I was just trying to experiment
my C in the real world and try to help community :-D

If you want to implement a brand new shining !NADA version on a 'hype' language,
please remember to advise me, so I can point a link here to your project.


How to help?
------------

To see what's going on with this small project, please look at the 
[CHANGES](CHANGES.md) and [TODO](TODO.md) files, and if you're willing to help in any way, 
please contact-me and I'll certainly have something for you to do.

If you help out, you'll end up on the [THANKS](THANKS.md) page!

Caveats
-------

Some well known issues.

* Expression tree is too large (maximum depth 1000)

     NADA uses extensively database and, in case of SQLite, there's a limitation
     on query depth. Try to decrease baseline.ini's `maxentries' parameter to a 
     value below 500 and see if the problem persists.

     For further details on this issue: http://www.sqlite.org/limits.html
