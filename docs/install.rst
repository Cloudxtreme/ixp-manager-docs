.. _installing-ixp-manager

Installation
============

We've tried out best to make installing IXP Manager as easy as possible (within the time constraints
allotted to us as an open source project). But, beware, there's no such thing as *an IXP in a box* and
a number of skills are required in the true NetDevOps fashion.

The installation of IXP Manager doesn't end when you have the web interface up and running (which is
mostly what we cover in this section) - that's really just the beginning. Completing the installation is
about all the features that work around and via IXP Manager such as route server configurations, graphing,
peer to peer graphs via sflow, etc. Each of these require different amounts of effort and are covered in
their individual feature pages.

Requirements
------------

IXP Manager tries to stay current in terms of technology. Typically, this means some element of framework
refresh(es) every couple of years and other more incremental package upgrades with minor version upgrades.
As well as the obvious reasons for this, there is also the requirement to prevent developer apathy - insisting
on legacy frameworks and packages that have been EOL'd provides a major stumbling block for bringing on new
developers and contributors.

The current requirements for the web application are:

- a Linux / BSD host. Windows may be possible but is unsupported as we have not developed or run IXP Manager on this platform.
- MariaDB / MySQL version 5.5.4 or later.
- Apache. Nginx may be possible, but it is untested and unsupported.
- PHP >= 5.6. Note that IXP Manager will not run on older versions of PHP and will require PHP 7 before year end 2015.
- Memcached - optional but highly recommended.

To complete the installation using the included config/scripts, you will also need to have installed:

- git - distributed revision control system (e.g.: apt-get install git)
- memcache extension module for PHP5 (e.g.: apt-get install php5-memcache)
- SNMP extension module for PHP5 (e.g.: apt-get install php5-snmp)
- php-mbstring - may be built into the PHP core
- php-gettext - may be built into the PHP core
- php-iconv - may be built into the PHP core
- php-ctype - may be built into the PHP core
- php-json - may be built into the PHP core
- php-pdo_mysql

TODO: script to check dependancies.

Install Composer
----------------

IXP Manager utilises `Composer`_ to manage its dependencies. First, download a copy of the composer.phar.
Once you have the PHAR archive, you can either keep it in your local project directory or move to `/usr/local/bin`
to use it globally on your system.

.. _Composer: http://getcomposer.org/

Install IXP Manager
-------------------

If you are using IXP Manager v3, please note that there is no in place upgrade path for v4. V4 must be installed
in parallel / in place of your v3 versoin. When you complete the installation of v4, you can then point Apache
to the new v4 directory (or just move v3 out of the way and replace it with v4).
