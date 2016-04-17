.. _features-graphing:

Graphing
======================

The biggest new feature available at launch in IXP Manager v4 is a new graphing system called *Grapher*.

*Grapher* is a complete rewrite of all previous graphing code and includes:

- API access to graphs and graph statistics
- multiple backends (such as MRTG, sflow) with dynamic resolution of appropriate backend
- configuration generation where required
- consistent and flexible OOP design

To date, we've developed three reference backend implementations:

1. ``dummy`` - a dummy grapher that just provides a placeholder graph for all possible graph types;
2. ``mrtg`` - MRTG graphing using either the log or rrd backend. Use cases for MRTG are L2 interface statistics for bits / packets / errors / discards per second. Aggregate graphs for customer LAGs, overall customer traffic, all traffic over a switch / infrastructure / the entire IXP are all supported.
3. ``sflow`` - while the MRTG backend looks at layer 2 statistics, sflow is used to provide layer 3 statistics such as per protocol (IPv4/6) graphs and peer to peer graphs.

In a typical production environment, you'd implement both MRTG and sflow to provide the complete set of features.

Configuration
-------------

There is only a handful of configuration options required and these can be seen with documentation in ``config/grapher.php`` (remember to put your own local changes in ``.env``).

The only global (non-backend specific) options are:

* ``backend`` - in a typical production environment this would be ``"mrtg|sflow"`` which means try the MRTG backend first and then sflow. We ship with this set as ``"dummy"`` so you can see sample graphs working out of the box.
* ``cache`` - as the industry standard is to graph at 5min intervals, the cache settings do not regenerate / reload / reprocess log / rrd / image files if we have cached them and they are less than 5mins old. This is enabled by default which is the recommended setting.

Backend specific configuration and set-up instructions can be found in their own sections.


Backends
---------

Mrtg
+++++

Overview
%%%%%%%%%

MRTG is particularly efficient in the SNMP sense as, irrespective of how many times an interface is referenced for different graphs, it is only polled once per run.

Per-second graphs are generated for bits, packets, errors and discards at 5min intervals.

IXP Manager's Grapher system can use MRTG to poll switches and create traffic graphs for:

* **Aggregate IXP and Infrastructure Graphs**
  
  The MRTG script creates aggregate graphs for the entire IXP as well as per-infrastructure graphs. These graphs are available from the Statistics menu under Overall Peering Graphs. Also, the graphs on the admin dashboard are the monthly versions of these and will appear on the dashboard when configured as above.

* **Switch Aggregate Graphs**
  
  These are defined and built automatically from the switches you have defined. These graphs are the aggregate of all peering ports. These graphs are available from the Statistics menu under Switch Aggregate Graphs.

* **Inter-Switch / Trunk Graphs**
  
  The above will generate graphs for all your customer ports and aggregate graphs for all items mentioned at the start of this page except inter-switch graphs. There is currently no way to be able to create a sane definition of these in the IXP Manager database, so we chicken out and let each IXP do it manually. Simplicity r00lz.
  
  FIXME: document the best way to do this
  
  These graphs will be available in the Statistics menu under Inter-Switch / PoP Graphs.

* **Customer Graphs**
  
  MRTG creates per port, per LAG and aggregate graphs for each member / customer.


Configuration
%%%%%%%%%%%%%%%%%%%%%

You need to install some basic packages for MRTG to work - on Ubuntu for example, install:

::

  apt-get install libconfig-general-perl libnetaddr-ip-perl mrtg

You also need a folder to store all MRTG files. For example:

::

  mkdir -p /srv/mrtg

In your ``.env``, you need to set the following options:

::

  # the database type to use - either log or rrd
  GRAPHER_BACKEND_MRTG_DBTYPE="rrd"
  # where to store log/rrd/png files as created above. This is from the perspective
  # of the mrtg daemon so should also be local
  GRAPHER_BACKEND_MRTG_WORKDIR="/tmp"
  # where to find the WORKDIR above from IXP Manager's perspective. This can be a
  # local directory or a URL to remote web server
  GRAPHER_BACKEND_MRTG_LOGDIR="http://collector.example.com/mrtg"


You can now generate a MRTG configuration by executing a command such as:

::

  # output to stdout:
  ./artisan grapher:generate-configuration -B mrtg
  # output to a named file
  ./artisan grapher:generate-configuration -B mrtg -O /tmp/mrtg.cfg.candidate

You could also combine a syntax check before putting the resultant file live. Here's a complete example that could be run via cron:

::
  
  #! /bin/sh
  
  APPLICATION_PATH=/srv/ixp
  
  # Synchronise configuration files
  ${APPLICATION_PATH}/artisan grapher:generate-configuration -B mrtg -O /tmp/mrtg.cfg.candidate
  
  /usr/bin/mrtg --check /tmp/mrtg.cfg.candidate                 \
    && /bin/mv /tmp/mrtg.cfg.candidate /etc/mrtg/mrtg.cfg

Note that our header template starts MRTG as a daemon. On FreeBSD, MRTG comes with an initd script by default and you can kick it off on boot with something like the following in rc.conf:

::

  mrtg_daemon_enable="YES"
  mrtg_daemon_config="/etc/mrtg/mrtg.cfg"

However, on Ubuntu it does not but it comes with a /etc/cron.d/mrtg file which kicks it off every five minutes (it will daemonise the first time and further cron jobs will have no effect). If you use this method, you will need to have your periodic update script restart / stop the daemon when the configuration changes.

To start and stop it via standard initd scripts on Ubuntu, use `an initd script such as this <https://github.com/inex/IXP-Manager/blob/master/tools/runtime/mrtg/ubuntu-mrtg-initd>`_ (`source <http://www.iceflatline.com/2009/08/how-to-install-and-configure-mrtg-on-ubuntu-server/>`_):

::

  cp $APPLICATION_PATH/tools/runtime/mrtg/ubuntu-mrtg-initd /etc/init.d/mrtg
  chmod +x /etc/init.d/mrtg
  update-rc.d mrtg defaults
  /etc/init.d/mrtg start

Remember to disable the default cron job for MRTG on Ubuntu!

Customising the Configuration
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

Inserting Traffic Data Into the Database
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
