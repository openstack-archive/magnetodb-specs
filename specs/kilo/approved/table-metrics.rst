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


Problem description
===================

---------
Use Cases
---------

As a magnetodb administrator I need to push table stats to external monitoring
service(StatsD). 


Proposed change
===============

1. Introduce new task for async task executor.
2. Use MagnetoDB monitoring API for collecting table stats.
3. Push collected data to StatsD server.

------------
Alternatives
------------

Place interaction with StatsD in MagnetoDB source code. However integration
with StatsD is special case so it's better don't add unnecessary dependences
to MagnetoDB source code.


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

None


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
