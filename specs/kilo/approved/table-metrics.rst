..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================
Integration with StatsD 
=======================

Launchpad: statsd_table_metrics_

.. _statsd_table_metrics:
   https://blueprints.launchpad.net/magnetodb/+spec/statsd-tables-metrics

Mechanism for pushing table-related metrics (table size in bytes,
item count, etc) to StatsD.
::

+------------------+             +-------------------+         +----------+
|                  |             |                   |         |          |
|                  |      HTTP   |                   |  UDP    |          |
|  Monitoring API  | <-----------+  Stats collector  +-------> |  StatsD  |
|                  |             |                   |         |          |
|                  |             |                   |         |          |
+------------------+             +-------------------+         +----------+


Problem description
===================

---------
Use Cases
---------

As a magnetodb administrator I need to push table stats to external monitoring
service(StatsD).


Proposed change
===============

1. Introduce new service for periodically sending stats to StatsD.
2. Use MagnetoDB monitoring API for collecting table stats.
3. Push collected data to StatsD server.

------------
Alternatives
------------

1. Place interaction with StatsD in MagnetoDB source code. However integration
   with StatsD is special case so it's better don't add unnecessary
   dependencies to MagnetoDB source code.

2. Use async task executor for sending data to StatsS. Currently async task
   executor contains code for creating and deleting tables. Placing code
   for interaction with external (and optional) sytem seems not a good
   idea.
3. CronD task. Pros: simple to implement. Cons: minimum interval 1 min.
   This way will add new dependency to external system (crond).

-----------------
Data model impact
-----------------

No data is stored or cached.


---------------
REST API impact
---------------

None

---------------
Security impact
---------------

* TBD

--------------------
Notifications impact
--------------------

None


---------------------
Other end user impact
---------------------

None


------------------
Performance Impact
------------------

None


---------------------
Other deployer impact
---------------------

None


----------------
Developer impact
----------------

None


Implementation
==============

Oslo provides utils for periodic tasks: periodic_task_

.. _periodic_task: https://github.com/openstack/oslo-incubator/blob/master/openstack/common/periodic_task.py


-----------
Assignee(s)
-----------

Primary assignee:
  <achudnovets>


----------
Work Items
----------

1. Write interface for StatsD.
2. Update documentation.


Dependencies
============

https://blueprints.launchpad.net/magnetodb/+spec/monitoring-api-url-refactoring


Testing
=======

None


Documentation Impact
====================

* StatsD integration section should be added to documentation_.

.. _documentation:
   http://magnetodb.readthedocs.org/en/latest/api_reference.html


References
==========

None
