..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
MagnetoDB Request Metrics
============================

Launchpad: request-metrics_

.. _request-metrics:
   https://blueprints.launchpad.net/magnetodb/+spec/request-metrics

Real time request metrics.

Problem description
===================

---------
Use Cases
---------

To proactively address MagnetoDB operational issues, MagnetoDB admin users need
real time visibility into the request metrics data on each API node, including:

- number of requests completed
- number of requests in processing
- number of errors
- average latency
- median latency
- minimum latency
- maximum latency
- average requests per second
- distribution of request latencies, such as: "50%","66%","75%","80%","90%","95%","98%","99%","100%".
- request size in bytes
- response size in bytes

Each of the above metrics should be collected at node level, API endpoint level,
and tenant level.

--------
Glossary
--------

- request metrics

  Measurements of MagnetoDB API requests, including counter based metrics such
  as count, error count, and timer based metrics, such as latency

- API node level metrics
  Request metrics collected for the whole API node, such as count of requests
  for API node

- API endpoint metrics
  Request metrics collected for a specific MagnetoDB REST API for an API node,
  such as count of requests for get_item calls for an API node

- tenant metrics
  Request metrics collected for a specific tenant for an API node. It can be
  further broken down by MagnetoDB REST APIs, such as the count of requests for
  a tenant on an API node, which further breaks down into count of different API
  endpoint calls for that tenant on that API node


High Level Sequence Diagram
===========================
::

               +-----------+                  +--------+          +----------------------------+
               | MagnetoDB |                  | StatsD |          | Graphite or other backends |
               +-----------+                  +--------+          +----------------------------+
   --------------\ |                               |                         |
   | API request |-|                               |                         |
   --------------- |                               |                         |
                   | --------------\               |                         |
                   |-| Sampling    |               |                         |
                   | ---------------               |                         |
                   |                               |                         |
                   | StatsD Timer, Counter request |                         |
                   |------------------------------>|                         |
                   |           (UDP)               | --------------------\   |
                   |                               |-| Aggregate counters|   |
                   |                               | | Aggregate timers  |   |
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


1) Report all request metrics to StatsD server using a sample rate for node
level/API endpoint level/tenant level.

Request metrics are either counters or timing data (in units of milliseconds).
StatsD will expand timing data to min, max, avg, count, and 90th percentile per
timing metric. Further aggregation can be performed at StatsD's backend to get
additional metrics.

For example, number of requests completed can be calculated using the request
latency timer. In Graphite it can be represented as:
count(mdb.req.latency)

Another example, number of requests per minute (RPM) can be
calculated using count(mdb.req.latency) in 1 minute interval:
summarize(sumSeries(mdb.req.latency.count), "1min")

2) Introduce a new middleware to collect request metrics at API node level, and
tenant level. API node level request metrics are for all tenants except
monitoring API requests. Tenant level request metrics are API node level request
metrics further broken down by individual tenants.

Middleware seems a natural place to collect the request metrics data at API node
level. We propose to introduce a new middleware to all API node services,
including REST API and streaming.

3) Update API endpoint request controllers to collect request metrics at API
endpoint level and tenant level. The change should not be intrusive to API
controller logic and should be easily turned on/off via configuration.

API endpoint level request metrics are API node level request metrics broken
down by API endpoints, and further by tenants.

Admin users may need visibility into requests for each API endpoint. The API
endpoint request controller can use StatsD client to send corresponding
metrics to StatsD server.

Initially we will focus on request measurements on node wide metrics, API
endpoints and tenant metrics on each API node. Later on, we will expand
to cover Cassandra request metrics.

The following metrics will be collected at node level:

- mdb.req.latency (timer)
- mdb.req.size_bytes (counter)
- mdb.req.resp_size_bytes (counter)
- mdb.req.error (counter)

Optionally, the following metrics can be collected at at node level for each
tenant:

- mdb.req.{TENANT}.latency (timer)
- mdb.req.{TENANT}.size_bytes (counter)
- mdb.req.{TENANT}.resp.size_bytes (counter)
- mdb.req.{TENANT}.error (counter)

The following metrics can be optionally collected at API endpoint:

- mdb.req.{API_ENDPOINT}.latency (timer)
- mdb.req.{API_ENDPOINT}.error (counter)
- mdb.req.{API_ENDPOINT}.size_bytes (counter)
- mdb.req.{API_ENDPOINT}.resp.size_bytes (counter)

- mdb.req.{API_ENDPOINT}.{TENANT}.latency (timer)
- mdb.req.{API_ENDPOINT}.{TENANT}.error (counter)
- mdb.req.{API_ENDPOINT}.{TENANT}.size_bytes (counter)
- mdb.req.{API_ENDPOINT}.{TENANT}.resp_size_bytes (counter)

Note that for table_create and table_delete API calls, since these are asynchronous
operations, the latency metrics are only latency measurements when responses are
returned, not when table operations are complete. Separate request latency
metrics will be collected in async_task_executor to reflect the true latencies
for table create/delete operations as below:

- mdb.req.table_create_task.latency (timer)
- mdb.req.table_create_task.error (counter)
- mdb.req.table_create_task.{TENANT}.latency (timer)
- mdb.req.table_create_task.{TENANT}.error (counter)

And optionally, the above metrics can be collected at tenant level:

- mdb.req.table_delete_task.latency (timer)
- mdb.req.table_delete_task.error (counter)
- mdb.req.table_delete_task.{TENANT}.latency (timer)
- mdb.req.table_delete_task.{TENANT}.error (counter)


The following are derived metrics. * means metrics can be derived from StatsD
and + means metrics can be derived from monitoring backend such as Graphite.

- mdb.req.completed+
- mdb.req.rpm+
- mdb.req.latency_avg*
- mdb.req.latency_median*
- mdb.req.latency_min*
- mdb.req.latency_max*
- mdb.req.latency_100*
- mdb.req.latency_99*
- mdb.req.latency_98*
- mdb.req.latency_95*
- mdb.req.latency_90*
- mdb.req.latency_80*
- mdb.req.latency_75*
- mdb.req.latency_66*
- mdb.req.latency_50*

- mdb.req.{TENANT}.completed+
- mdb.req.{TENANT}.rpm+
- mdb.req.{TENANT}.latency_100*
- mdb.req.{TENANT}.latency_99*
- mdb.req.{TENANT}.latency_98*
- mdb.req.{TENANT}.latency_95*
- mdb.req.{TENANT}.latency_90*
- mdb.req.{TENANT}.latency_80*
- mdb.req.{TENANT}.latency_75*
- mdb.req.{TENANT}.latency_66*
- mdb.req.{TENANT}.latency_50*

- mdb.req.{API_ENDPOINT}.completed+
- mdb.req.{API_ENDPOINT}.rpm+
- mdb.req.{API_ENDPOINT}.latency_100*
- mdb.req.{API_ENDPOINT}.latency_99*
- mdb.req.{API_ENDPOINT}.latency_98*
- mdb.req.{API_ENDPOINT}.latency_95*
- mdb.req.{API_ENDPOINT}.latency_90*
- mdb.req.{API_ENDPOINT}.latency_80*
- mdb.req.{API_ENDPOINT}.latency_75*
- mdb.req.{API_ENDPOINT}.latency_66*
- mdb.req.{API_ENDPOINT}.latency_50*

- mdb.req.{API_ENDPOINT}.{TENANT}.completed+
- mdb.req.{API_ENDPOINT}.{TENANT}.rpm+
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_100*
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_99*
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_98*
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_95*
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_90*
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_80*
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_75*
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_66*
- mdb.req.{API_ENDPOINT}.{TENANT}.latency_50*

------------
Alternatives
------------
Instead of using StatsD, a pure middleware based approach can be used to gather
request metrics at node/API endpoint/tenant levels, using Scales which is used
by Cassandra python driver. No new dependency will be introduced.

Another option can be: create a notification driver based on StatsD, using oslo
messaging's notification mechanism to send metrics request to StatsD. This way
MagnetoDB can use the existing notification mechanism to send metrics to StatD,
hence no new middleware is introduced.

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

No impact. Metrics are collected in middleware/API endpoint controller
directly, and exposed through StatsD.


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
configured or unavailable, no request metrics will be generated.


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

1) Create middleware to collect node and/or tenant level metrics.
2) API endpoint controllers need to be updated to collect API endpoint and tenant level request metrics.
3) Update documentation to list all request metrics to be published.


Dependencies
============

StatsD will be needed for request metrics to be collected. StatsD is optional.
If no StatsD is configured, no request metrics will be generated.


Testing
=======

None


Documentation Impact
====================

Published request metrics should be added to documentation_.

.. _documentation:
   http://magnetodb.readthedocs.org/en/latest/api_reference.html


References
==========

None
