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
count(mdb.req.timing)

Another example, number of requests per minute (RPM) can be
calculated using count(mdb.req.timing) in 1 minute interval:
summarize(sumSeries(mdb.req.timing.count), "1min")

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

- mdb.req.timing (timer)
- mdb.req.timing.error.400 (timer)
- mdb.req.timing.error.403 (timer)
- mdb.req.timing.error.404 (timer)
- mdb.req.timing.error.500 (timer)
- mdb.req.size_bytes (counter)
- mdb.req.resp_size_bytes (counter)

Optionally, the following metrics can be collected at at node level for each
tenant:

- mdb.req.timing.{TENANT} (timer)
- mdb.req.timing.error.400.{TENANT} (timer)
- mdb.req.timing.error.403.{TENANT} (timer)
- mdb.req.timing.error.404.{TENANT} (timer)
- mdb.req.timing.error.500.{TENANT} (timer)
- mdb.req.size_bytes.{TENANT} (counter)
- mdb.req.resp_size_bytes.{TENANT} (counter)

The following metrics can be optionally collected at API endpoint:

- mdb.req.timing.{API_ENDPOINT} (timer)
- mdb.req.timing.error.400.{API_ENDPOINT} (timer)
- mdb.req.timing.error.403.{API_ENDPOINT} (timer)
- mdb.req.timing.error.404.{API_ENDPOINT} (timer)
- mdb.req.timing.error.500.{API_ENDPOINT} (timer)
- mdb.req.size_bytes.{API_ENDPOINT} (counter)
- mdb.req.resp_size_bytes.{API_ENDPOINT} (counter)

- mdb.req.timing.{API_ENDPOINT}.{TENANT} (timer)
- mdb.req.timing.error.400.{API_ENDPOINT}.{TENANT} (timer)
- mdb.req.timing.error.403.{API_ENDPOINT}.{TENANT} (timer)
- mdb.req.timing.error.404.{API_ENDPOINT}.{TENANT} (timer)
- mdb.req.timing.error.500.{API_ENDPOINT}.{TENANT} (timer)
- mdb.req.size_bytes.{API_ENDPOINT}.{TENANT} (counter)
- mdb.req.resp_size_bytes.{API_ENDPOINT}.{TENANT} (counter)

- mdb.req.table_create_task.timing (timer)
- mdb.req.table_create_task.timing.error (timer)

- mdb.req.table_create_task.timing.{TENANT} (timer)
- mdb.req.table_create_task.timing.error.{TENANT} (timer)

Note that for table_create and table_delete API calls, since these are
asynchronous operations, the timing metrics are only latency measurements when
responses are returned, not when table operations are complete. Separate request
timing metrics are collected in async_task_executor. In order to get the
end-to-end latencies for table create/delete operations, user can use the StatsD
backend such as Graphite to correlate timing metrics on API nodes and timing
metrics on async_task_executor node.

async_task_executor metrics can be implemented through a customized notification
driver for StatsD. Existing notification API calls will be continue to be used
to send counter and timer metrics to StatsD server. This way code change can be
avoided or minimized by reusing the notification API call code.

The following are derived metrics. * means metrics can be derived from StatsD
and + means metrics can be derived from monitoring backend such as Graphite.

- mdb.req.completed+
- mdb.req.rpm+
- mdb.req.timing_avg*
- mdb.req.timing_median*
- mdb.req.timing_min*
- mdb.req.timing_max*
- mdb.req.timing_100*
- mdb.req.timing_99*
- mdb.req.timing_98*
- mdb.req.timing_95*
- mdb.req.timing_90*
- mdb.req.timing_80*
- mdb.req.timing_75*
- mdb.req.timing_66*
- mdb.req.timing_50*

- mdb.req.{TENANT}.completed+
- mdb.req.{TENANT}.rpm+
- mdb.req.{TENANT}.timing_100*
- mdb.req.{TENANT}.timing_99*
- mdb.req.{TENANT}.timing_98*
- mdb.req.{TENANT}.timing_95*
- mdb.req.{TENANT}.timing_90*
- mdb.req.{TENANT}.timing_80*
- mdb.req.{TENANT}.timing_75*
- mdb.req.{TENANT}.timing_66*
- mdb.req.{TENANT}.timing_50*

- mdb.req.{API_ENDPOINT}.completed+
- mdb.req.{API_ENDPOINT}.rpm+
- mdb.req.{API_ENDPOINT}.timing_100*
- mdb.req.{API_ENDPOINT}.timing_99*
- mdb.req.{API_ENDPOINT}.timing_98*
- mdb.req.{API_ENDPOINT}.timing_95*
- mdb.req.{API_ENDPOINT}.timing_90*
- mdb.req.{API_ENDPOINT}.timing_80*
- mdb.req.{API_ENDPOINT}.timing_75*
- mdb.req.{API_ENDPOINT}.timing_66*
- mdb.req.{API_ENDPOINT}.timing_50*

- mdb.req.{API_ENDPOINT}.{TENANT}.completed+
- mdb.req.{API_ENDPOINT}.{TENANT}.rpm+
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_100*
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_99*
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_98*
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_95*
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_90*
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_80*
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_75*
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_66*
- mdb.req.{API_ENDPOINT}.{TENANT}.timing_50*

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
