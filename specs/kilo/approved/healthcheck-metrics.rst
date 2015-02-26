..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================
MagnetoDB Health Check Metrics
==============================

Launchpad: healthcheck-metrics_

.. _healthcheck-metrics:
   https://blueprints.launchpad.net/magnetodb/+spec/healthcheck-metrics

Health check metrics.

Problem description
===================

---------
Use Cases
---------

To monitor MagnetoDB operation status, MagnetoDB admin users need real time
visibility into the health status of MagnetoDB API server and its dependencies
including Keystone, Cassandra and RabbitMQ.

The health check metrics should be collected at each MagnetoDB API server node.

High Level Sequence Diagram
===========================
::

               +-----------+                  +--------+          +----------------------------+
               | MagnetoDB |                  | StatsD |          | Graphite or other backends |
               +-----------+                  +--------+          +----------------------------+
   --------------\ |                               |                         |
  |Periodic call:|-|                               |                         |
  | /healthcheck | |                               |                         |
   --------------- |                               |                         |
                   | --------------\               |                         |
                   |-| Sampling    |               |                         |
                   | ---------------               |                         |
                   |                               |                         |
                   | StatsD Gauge stats            |                         |
                   |------------------------------>|                         |
                   |           (UDP)               | --------------------\   |
                   |                               |-| Aggregate gauges  |   |
                   |                               | |                   |
                   |                               | | Hold for 1 min    |   |
                   |                               | ---------------------   |
                   |                               |                         |
                   |                               | Flush to backend (TCP)  |
                   |                               |------------------------>|
                   |                               | (e.x. Graphite Carbon)  | -------------------\
                   |                               |                         |-| Store metrics in |
                   |                               |                         | | Graphite Whisper |
                   |                               |                         | | or InfluxDB, etc |
                   |                               |                         | | Further aggregate|
                   |                               |                         | --------------------
                   |                               |                         |


Proposed Change
===============


1) Refactor health_check.py to have detailed response body for major components 

Periodically call /healthcheck?fullcheck=true API and report metrics to StatsD
server usinga configurable sample rate. The periodic interval should be
configurable.

Health check metrics are all gauges.

2) Introduce a generic periodic task runner as a service to run different
periodic statistics collecting and metrics emitting tasks, including the above
health check metrics collector.

3) Add service launcher as an added standalone service and launch the above
periodic task runner service. The service launcher should be generic so future
services such as table statistics metrics service can be added easily.

The following metrics will be collected at node level:

- magnetodb.health.overall (gauge)
- magnetodb.health.api (gauge)
- magnetodb.health.cassandra (gauge)
- magnetodb.health.rabbitmq (gauge)
- magnetodb.health.keystone (gauge)

------------
Alternatives
------------
Instead of using StatsD, the refactored notification driver can be used to send
a newly defined event type which has only StatsD metrics as the delivery method.

-----------------
Data model impact
-----------------
No impact.


---------------
REST API impact
---------------
No impact. Metrics will be exposed through StatsD.


---------------
Security impact
---------------

No impact. Metrics are collected in periodic task runner service, and exposed
through StatsD. The monitoring framework's own security mechanism will be used
to secure metrics collected.


--------------------
Notifications impact
--------------------

No impact.


---------------------
Other end user impact
---------------------

No impact.


------------------
Performance Impact
------------------

Performance impact should be minimal if StatsD is used. The metrics sent to
StatsD are through UDP.


---------------------
Other deployer impact
---------------------

StatsD server will need to be deployed and configured. If StatsD server is not
configured or unavailable, no health check metrics will be generated.


----------------
Developer impact
----------------

No impact.


Implementation
==============


-----------
Assignee(s)
-----------

Primary assignee:
  <unassigned>

Other contributors:
  <unassigned>


----------
Work Items
----------

1) Create periodic task runner as a service. The period interval should be
configurable.

2) Define a periodic task to call health check API, gather the results and
perform sampling with a configurable sample rate, and generate StatsD metrics.

3) Add service launcher in bin/magnetodb-api-server and
bin/magnetodb-api-server-gunicorn and launch the above periodic task runner
service.

Dependencies
============

StatsD will be needed for health check metrics to be collected. StatsD is
optional. If no StatsD is configured, no health check metrics will be generated.


Testing
=======

None


Documentation Impact
====================

Published health check metrics should be added to documentation_.

.. _documentation:
   http://magnetodb.readthedocs.org/en/latest/api_reference.html


References
==========

None
