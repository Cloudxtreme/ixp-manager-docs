.. upgrade-from-v3:

Upgrade from V3
<<<<<<< HEAD
================
=======
===============
>>>>>>> e2c1dbc85fdfdca207934479217ee06813fd6465

If you are using IXP Manager v3, please note that there is no in place upgrade path for v4. V4 must be installed
in parallel / in place of your v3 version. When you complete the installation of v4, you can then point Apache
to the new v4 directory (or just move v3 out of the way and replace it with v4).

**This documentation was written when v3 was the master branch and v4 was in its own development branch.
Ensure this is still the case as you follow these instructions!**

**THESE ARE DRAFT NOTES FROM A TEST UPGRADE. THESE WILL BE FLESHED OUT SOON.**


Duplicating the Database
--------------------------

As we're installing in parallel, we want to duplicate the database for testing as follows.
When granting permissions on the new database, use your existing IXP Manager database
credentials for ease of configuration.

::

  mysql -u root -pXXX -e 'CREATE DATABASE ixp4;'
  mysqldump -u root -pXXX ixp | mysql -u root -pXXX ixp4
  mysql -u root -pXXX -e 'GRANT ALL ON ixp4.* TO `ixp`@`localhost` IDENTIFIED BY "YYY";'
  # and test:
  mysql -u ixp -pYYY -h localhost ixp4


Get the IXP Manager Source
--------------------------

The code for IXP Manager is maintained on GitHub and the canonical repositorying is `inex/IXP-Manager`_.

Log into the server where you wish to install IXP Manager. Move to the directory where you wish to store the source.
Note that it should not be checked out into any web exposed directory (e.g. do not checkout to ``/var/www``). In my case,
I'm going to use ``/usr/local/ixp`` which will be referred to as ``$IXPROOT`` below. So I:

::

  cd /usr/local
  git clone https://github.com/inex/IXP-Manager.git ixp4
  git checkout v4   ### NB: current v3 is the master branch and v4 has it's own. This could be reversed unexpectedly.

You now need to create the text file ``$IXPROOT/.env`` containing database details from above:

::

  DB_HOST=localhost
  DB_DATABASE=ixp4
  DB_USERNAME=ixp
  DB_PASSWORD=XXX

And then run:

::

  composer update


Initial Configuration Tasks
---------------------------

New Local Settings / Configuration (Laravel)
++++++++++++++++++++++++++++++++++++++++++++

As v4 is a version in migration from older libraries like Zend Framework to Laravel, we need to do a bit of setup
for both frameworks. First Laravel: we'll essential set defaults in ``$IXPROOT/.env`` which will be used by files in
``$IXPROOT/config``. Feel free to look at those files and add ``.env`` parameters to suit your own environment.

First, the application key needs to be set and unique. Execute the following command and you should see a similar
output (but a different key):

::

  ./artisan key:generate
  Application key [XjWtsiPl_CHANGE_ME_CHANGE_ME] set successfully.

Check ``.env`` and if the key has not been set, add the following to the ``.env`` file by copying the key above:

::

  APP_KEY=XjWtsiPl_CHANGE_ME_CHANGE_ME


**NB: Where possible, place local changes into ``.env`` rather than changing the config files as these files are
under version control. See `Laravel's documentation on this`_ and email the mailing list for help.**

.. Laravel's documentation on this: http://laravel.com/docs/5.1/installation#configuration

Here are some hints on what you might want to set:

``config/app.php``
  Set: ``APP_URL`` in ``$IXPROOT/.env``.

``config/cache.php``
    Either leave ``CACHE_DRIVER`` as ``file`` or set otherwise in ``$IXPROOT/.env``.

``config/identity.php``
  Copy ``config/identity.php.dist`` to ``config/identity.php`` and edit for your IXP *(essentially copy
  from your v3 application/configs/application.ini)*.

``config/mail.php``
  Set appropriate options for mail relay. Specifically ``MAIL_DRIVER``, ``MAIL_HOST`` and ``MAIL_PORT``.

Migrate Old Zend Framework Settings
+++++++++++++++++++++++++++++++++++

Copy your IXP v3 `application/configs/application.ini` file to your new v4 `application/configs` and edit as follows:

1. Change the database name:

::

  -resources.doctrine2.connection.options.dbname   = 'ixp'
  +resources.doctrine2.connection.options.dbname   = 'ixp4'

2. Change paths as follows:

::

  -includePaths.osssnmp    = APPLICATION_PATH "/../library/OSS_SNMP.git"
  -includePaths.osslibrary = APPLICATION_PATH "/../library/OSS-Framework.git"
  +includePaths.osslibrary = APPLICATION_PATH "/../vendor/opensolutions/oss-framework/src/"

  -pluginPaths.OSS_Resource = APPLICATION_PATH "/../library/OSS-Framework.git/OSS/Resource"
  +pluginPaths.OSS_Resource = APPLICATION_PATH "/../vendor/opensolutions/oss-framework/src/OSS/Resource"

  -resources.smarty.plugins[] = APPLICATION_PATH "/../library/OSS-Framework.git/OSS/Smarty/functions"
  -resources.smarty.plugins[] = APPLICATION_PATH "/../library/Smarty/plugins"
  -resources.smarty.plugins[] = APPLICATION_PATH "/../library/Smarty/sysplugins"
  +resources.smarty.plugins[] = APPLICATION_PATH "/../vendor/opensolutions/oss-framework/src/OSS/Smarty/functions"
  +resources.smarty.plugins[] = APPLICATION_PATH "/../vendor/smarty/smarty/libs/plugins"
  +resources.smarty.plugins[] = APPLICATION_PATH "/../vendor/smarty/smarty/libs/sysplugins"

3. Change Doctrine2 options as follows:

::

  -resources.doctrine2cache.path               = "/usr/share/php/Doctrine/ORM"
  -resources.doctrine2cache.autoload_method    = "pear"
  +resources.doctrine2cache.autoload_method    = "composer"
  +resources.doctrine2cache.namespace          = 'ixp4'

  -resources.doctrine2.models_path        = APPLICATION_PATH
  -resources.doctrine2.proxies_path       = APPLICATION_PATH "/Proxies"
  -resources.doctrine2.repositories_path  = APPLICATION_PATH
  -resources.doctrine2.xml_schema_path    = APPLICATION_PATH "/../doctrine/schema"
  +resources.doctrine2.models_path        = APPLICATION_PATH "/../database"
  +resources.doctrine2.proxies_path       = APPLICATION_PATH "/../database/Proxies"
  +resources.doctrine2.repositories_path  = APPLICATION_PATH "/../database"
  +resources.doctrine2.xml_schema_path    = APPLICATION_PATH "/../database/xml"

Update the Database Schema
++++++++++++++++++++++++++

View the required changes with:

::

  ./artisan d2b:schema:update --sql

And apply with:

::

  ./artisan d2b:schema:update --commit



Apache
++++++

::

  Alias /ixp4 /srv/ixp4/public
  <Directory /srv/ixp4/public>
      Options FollowSymLinks
      AllowOverride None
      Require all granted

      SetEnv APPLICATION_ENV production

      RewriteEngine On
      RewriteCond %{REQUEST_FILENAME} -s [OR]
      RewriteCond %{REQUEST_FILENAME} -l [OR]
      RewriteCond %{REQUEST_FILENAME} -d
      RewriteRule ^.*$ - [NC,L]
      RewriteRule ^.*$ /ixp4/index.php [NC,L]
</Directory>

File System Permissions
+++++++++++++++++++++++

::

  chown -R www-data: var/ storage/ database/Proxies
