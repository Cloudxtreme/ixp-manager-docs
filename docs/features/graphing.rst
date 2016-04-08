.. _features-graphing

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

There is only a handful of configuration options required and these can be seen with documenantion in ``config/grapher.php`` (remembering to put your own changes in ``.env``).

The only global (non-backend specific) options are:

* ``backend`` - in a typical production environment this would be ``"mrtg|sflow"`` which means try the MRTG backend first and then sflow. We ship with this set as ``"dummy"`` so you can see sample graphs working out of the box.
* ``cache`` - as the industry standard is to graph at 5min intervals, the cache settings do not regenerate / reload / reprocess log / rrd / image files if we have cached them and they are less than 5mins old. This is enabled by default which is the recommended setting.

Backend specific configuration and set-up instructions can be found in their own sections.


Backends
---------

Mrtg
+++++

IXP Manager's Grapher system can use MRTG to poll switches and create traffic graphs for:

* aggregate (total) traffic over the entire exchange (all infrastructures);
* aggregate graphs for each infrastructure;
* aggregate graphs for each switch;
* aggregate graphs for inter-switch links; and
* aggregate and per port graphs for each member / customer.

MRTG is particularly efficient in the SNMP sense as, irrespective of how many times an interface is referenced for different graphs, it is only polled once per run.

Per-second graphs are generated for bits, packets, errors and discards at 5min intervals.

MRTG Configuration
%%%%%%%%%%%%%%%%%%%%%

In all cases below, ``$APPLICATION_PATH`` is the base directory of your IXP Manager installation.

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

However, you need to complete additional actions from the Integrating with IXP Manager section below.

You also need to set up a cron job to regenerate the configuration periodically (see below). Note that our header template starts MRTG as a daemon (see below also).
