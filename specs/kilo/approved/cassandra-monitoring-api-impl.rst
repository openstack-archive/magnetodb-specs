..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Cassandra implementation of monitoring api
==========================================

Implementation of monitoring API for Cassandra using
Jolokia HTTP-JMX bridge.
::

 +-----------+        +-------------------+
 |           |  HTTP  |                   |  JMX
 | MagnetoDB +--------> jolokia-jvm-agent +-------+
 |           |        |                 +-+-------v--------+
 +-----------+        +-----------------+                  |
                                        |     Cassandra    |
                                        |                  |
                                        +------------------+




Problem description
===================

---------
Use Cases
---------

As a magnetodb user I need to know how much data I have in table. As a
magnetodb administrator I need to know now much space is used with user's
table. As a accountant department I need to know how big user's table is
in order to create a bill.


Proposed change
===============

1. Add Jolokia HTTP-JMX bridge to Cassandra deployment.
2. Implement calls from MagnetoDB to jolokia-jvm-agent.


------------
Alternatives
------------

Get count of items in table using CQL (SELECT COUNT(*) FROM CF LIMIT 100000).
Use nodetool cfstats through ssh for getting all metrics from Cassandra.


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

* MagnetoDB API node should have access to Jolokia HTTP endpoint


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

Now we have POC for count items in table
(https://review.openstack.org/#/c/118557/) but this implementation is very
expensive (we scan all our data to get one counter). So we need change
concept and use Cassandra JMX to get approximate number of items and also
size of CF on disk.


---------------------
Other deployer impact
---------------------

* Jolokia component is used for collecting data from Cassandra

  * Cassandra start script should be updated with jolokia-jvm-agent_

* MagnetoDB API node should have access to Jolokia HTTP endpoint

.. _jolokia-jvm-agent:
   http://www.jolokia.org/agent/jvm.html


----------------
Developer impact
----------------

None


Implementation
==============

Current concept:

1. At Cassandra node we run JMX-HTTP bridge agent (Jolokia_).
2. Client goes to MagnetoDB with REST-like interface
   (.../{tenant_id}/monitoring/table/{table_name}).
3. MagnetoDB goes to Jolokia agent via HTTP, get JMX-metrics from Cassandra
   and returns them to client.


-----------
Assignee(s)
-----------

Primary assignee:
  <ominakov>

Other contributors:
 | <achudnovets>
 | <SpyRay>


----------
Work Items
----------

1. Implement calls from MagnetoDB to Jolokia agent.


Dependencies
============

Devstack integration update


Testing
=======

Tempest tests for this functional is not possible to implement, as we get
the approximate values ​​of the size and number of keys from the database.


Documentation Impact
====================

* Deployment_ guide should be updated.

.. _Deployment:
   http://magnetodb.readthedocs.
   org/en/latest/admin_guide.html#installation-guide


References
==========

Jolokia_

.. _Jolokia:
   http://www.jolokia.org/

https://review.openstack.org/#/c/122330/