.. _installing-ixp-manager:

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

- a Linux / BSD host.
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


Upgrading from V3
-----------------

If you are using IXP Manager v3, please note that there is no in place upgrade path for v4. V4 must be installed
in parallel / in place of your v3 versoin. When you complete the installation of v4, you can then point Apache
to the new v4 directory (or just move v3 out of the way and replace it with v4).

It is the intention of the authors to provide a step by step guide for this when we actually assist a non-INEX
installation on this upgrade (INEX is already running v4).

Get the IXP Manager Source
--------------------------

The code for IXP Manager is maintained on GitHub and the canonical repositorying is `inex/IXP-Manager`_. When
installing IXP Manager, you need to decide if you should fork the repository or just clone the canonical one.

.. _inex/IXP-Manager: https://github.com/inex/IXP-Manager

Forking vs. Cloning
+++++++++++++++++++

First, you need to understand what forking and cloning mean:

- Forking means creating your own copy of INEX's repository. GitHub have an excellent explanation which you should
  `read here <https://help.github.com/articles/fork-a-repo>`_. You then clone from your own forked repository.
- Cloning is like an SVN checkout (yes, it's a lot more than that, but that analogy is sufficient here for now). You
  can either clone from INEX's canonical repository or you can clone from your own forked repository.

Terminology: when we say forking we mean forking and cloning from your own repository and when we say cloning we mean
cloning from INEX's canonical repository.

So, which should you do?

- If you plan to just use INEX's official upstream IXP Manager without diverging from INEX's version, just clone
  from INEX directly.
- If you plan to develop your own features (whether or not they are accepted into the canonical repository) then fork
  your own version.
- If you plan to develop your own features and work with INEX to get them into the canonical repository, then clone
  from INEX for your live production version and fork for your development version.
- If you plan to skin IXP Manager for your local version and want to keep these skinned pages in a GitHub repository,
  then fork and clone from your fork.

If you're not a developer and you just want to get the bloody thing working and keep it updated with minimal fuss.
What should you do? **Easy, don't fork and clone from INEX directly.**

In this document, we'll proceed as if you were cloning from the canonical repository.

Install from GitHub
+++++++++++++++++++

Log into the server where you wish to install IXP Manager. Move to the directory where you wish to store the source.
Note that it should not be checked out into any web exposed directory (e.g. do not checkout to ``/var/www``). In my case,
I'm going to use ``/usr/local/ixp`` so I:

::

  cd /usr/local
  git clone https://github.com/inex/IXP-Manager.git ixp


Initial Setup and Dependancies
------------------------------

Now that you have the code, you need to set up a database, enter basic configuration parameters, etc.


Database Setup
++++++++++++++

Use whatever means you like to create a database and user for IXP Manager. For example:

::

  $ mysql -u root -p
  CREATE DATABASE `ixp` CHARACTER SET = 'utf8mb4' COLLATE = 'utf8mb4_unicode_ci';
  GRANT ALL ON `ixp`.* TO `ixp`@`127.0.0.1` IDENTIFIED BY 'password';
  GRANT ALL ON `ixp`.* TO `ixp`@`localhost` IDENTIFIED BY 'password';
  FLUSH PRIVILEGES;

In the directory where you cloned IXP Manager (``/usr/local/ixp`` in this example), create a file called ``.env`` with:

::

  DB_HOST=127.0.0.1
  DB_DATABASE=ixp
  DB_USERNAME=ixp
  DB_PASSWORD=password

Dependencies
++++++++++++

As mentioned above, we are using *composer* for dependency management - you must now install dependancies via:

::

  composer install
