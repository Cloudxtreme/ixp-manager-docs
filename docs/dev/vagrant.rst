.. _dev-api

Vagrant
=================

To aid development (as well as allowing easier evaluation), we use `Vagrant`_ from v4 onwards.

.. Vagrant: https://www.vagrantup.com/


Quick Vagrant
-------------

If you want to get IXP Manager with Vagrant up and running quickly, follow these steps:

1. Install Vagrant (see: http://docs.vagrantup.com/v2/installation/index.html)
2. Install VirtualBox (see: https://www.virtualbox.org/)
3. Clone IXP Manager to a directory and checkout the v4 development branch:

   ::

     git clone https://github.com/inex/IXP-Manager.git ixpmanager
     cd ixpmanager
     git checkout v4

4. The following is optional but recommended if you can as you may require your GitHub login details to get past GitHub's rate
limits. See `Composer's install instructions <https://getcomposer.org/download/>`_ if you don't have it. There may
be a database-related error which can safely be ignored.

   ::

     composer update

5. Spin up a Vagrant virtual machine:

   ::

     vagrant up

6. Access IXP Manager on: http://localhost:8088/

7. Log in with one of the following username / passwords:

   - Admin user: ``vagrant / vagrant1``
   - Customer Admin: ``as112 / as112as112``
   - Customer User: ``asii2user / as112as112`

Please see Vagrant's own documentation for a full description of how to use it fully. To access the virtual machine
that the above has spun up, just run the following from the ``ixpmanager`` directory:

::

  vagrant ssh

You'll find the ``ixpmanager`` directory mounted under ``/vagrant``, you can ``sudo su -`` and you can access MySQL via:

::

  mysql -u root -ppassword ixp

If you prefer to use phpMyAdmin, you'll find it at http://localhost:8088/phpmyadmin and you can log in with ``root / password``.


Database Details
----------------

Spinning up Vagrant in the above manner loads a sample database from ``ixpmanager/database/vagrant-base.sql``. If you
have a preferred development database, place a bzip'd copy of it at ``ixpmanager/ixpmanager-preferred.sql.bz2`` before
step 5 above.
