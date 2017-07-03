!NADA Changes
=============

0.9 - Thu Apr  4 14:51:06 BRT 2013
----------------------------------

* Added support for SQLite database.

0.6 - Tue Jan 29 16:28:14 BRST 2013
----------------------------------
* Included script to clean-up database unnecessary old 
  data

0.5 - Wed Jan 23 17:54:23 BRST 2013
----------------------------------
* Fixed parsing of metrics that doesn't contain minimum
  and maximum especified on perfdata

0.4 - Thu Jan  3 20:27:33 BRST 2013
----------------------------------
* Added some memory allocation protections
* Removed ugly homebrewed string copy function
* Migrating from realloc() to calloc()
* Using strncpy() instead of strcpy()
* Added section `How to help?' into README file

0.3 - Tue Aug 21 18:52:48 BRT 2012
----------------------------------
* Fixed float cast at tolerance calc
* Added option to avoid boundary limit below zero
* Added exponential smoothing as a valid algorithm to calcule
  baselines
* Now using NAGIOS_HOSTNAME and NAGIOS_SERVICEDESC at database
  level

0.2 - Mon Jul  9 21:07:55 BRT 2012
----------------------------------
* Splitted history table into two different entities to avoid
  unneeded info replication
* Added sazonality as a variable option

0.1-alpha
---------
* Initial release
