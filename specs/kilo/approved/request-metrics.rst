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
- number of failures* (this need to be measured at load balancer level)
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

Proposed change
===============

1) Report all request metrics to StatsD server using a sample rate for node 
level/API endpoint level/tenant level. 

Request metrics are either counters or timing data (in units of milliseconds). 
StatsD can be utilized to expand timing data to min, max, avg, count, and 90th 
percentile per timing metric. 

2) Introduce a new middleware to collect request metrics at API node level, and 
tenant level.

Middleware seems a natural place to collect the request metrics data at API node
level. We propose to introduce a new middleware to all API node services, 
including REST API and streaming.

3) Update API endpoint request controllers to collect request metrics at API 
endpoint level and tenant level.

Admin users may also need visibility into requests for each API endpoint. The 
API endpoint request controller can use StatsD client to send corresponding 
metrics to StatsD server.

Initially we will focus on request measurements on node wide metrics, API 
endpoints and WSGI processing delays on each API node. Later on, we will expand 
to cover Cassandra request metrics. 

The following metrics will be collected at node level:

- mdb_req_completed
- mdb_req_active
- mdb_req_failures* (this need to be measured at load balancer level)
- mdb_req_error
- mdb_req_avg_latency
- mdb_req_median_latency
- mdb_req_min_latency
- mdb_req_max_latency
- mdb_req_avg_rps
- mdb_req_latency_100
- mdb_req_latency_99
- mdb_req_latency_98
- mdb_req_latency_95
- mdb_req_latency_90
- mdb_req_latency_80
- mdb_req_latency_75
- mdb_req_latency_66
- mdb_req_latency_50
- mdb_req_size_bytes
- mdb_req_resp_size_bytes

- mdb_req_{TENANT}_completed
- mdb_req_{TENANT}_active
- mdb_req_{TENANT}_error
- mdb_req_{TENANT}_avg_latency
- mdb_req_{TENANT}_median_latency
- mdb_req_{TENANT}_min_latency
- mdb_req_{TENANT}_max_latency
- mdb_req_{TENANT}_avg_rps
- mdb_req_{TENANT}_latency_100
- mdb_req_{TENANT}_latency_99
- mdb_req_{TENANT}_latency_98
- mdb_req_{TENANT}_latency_95
- mdb_req_{TENANT}_latency_90
- mdb_req_{TENANT}_latency_80
- mdb_req_{TENANT}_latency_75
- mdb_req_{TENANT}_latency_66
- mdb_req_{TENANT}_latency_50
- mdb_req_{TENANT}_size_bytes
- mdb_req_{TENANT}_resp_size_bytes


The following metrics will be collected at API endpoint:

- mdb_req_{API_ENDPOINT}_completed
- mdb_req_{API_ENDPOINT}_active
- mdb_req_{API_ENDPOINT}_error
- mdb_req_{API_ENDPOINT}_avg_latency
- mdb_req_{API_ENDPOINT}_median_latency
- mdb_req_{API_ENDPOINT}_min_latency
- mdb_req_{API_ENDPOINT}_max_latency
- mdb_req_{API_ENDPOINT}_avg_rps
- mdb_req_{API_ENDPOINT}_latency_100
- mdb_req_{API_ENDPOINT}_latency_99
- mdb_req_{API_ENDPOINT}_latency_98
- mdb_req_{API_ENDPOINT}_latency_95
- mdb_req_{API_ENDPOINT}_latency_90
- mdb_req_{API_ENDPOINT}_latency_80
- mdb_req_{API_ENDPOINT}_latency_75
- mdb_req_{API_ENDPOINT}_latency_66
- mdb_req_{API_ENDPOINT}_latency_50
- mdb_req_{API_ENDPOINT}_size_bytes
- mdb_req_{API_ENDPOINT}_resp_size_bytes

- mdb_req_{API_ENDPOINT}_{TENANT}_completed
- mdb_req_{API_ENDPOINT}_{TENANT}_active
- mdb_req_{API_ENDPOINT}_{TENANT}_error
- mdb_req_{API_ENDPOINT}_{TENANT}_avg_latency
- mdb_req_{API_ENDPOINT}_{TENANT}_median_latency
- mdb_req_{API_ENDPOINT}_{TENANT}_min_latency
- mdb_req_{API_ENDPOINT}_{TENANT}_max_latency
- mdb_req_{API_ENDPOINT}_{TENANT}_avg_rps
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_100
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_99
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_98
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_95
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_90
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_80
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_75
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_66
- mdb_req_{API_ENDPOINT}_{TENANT}_latency_50
- mdb_req_{API_ENDPOINT}_{TENANT}_size_bytes
- mdb_req_{API_ENDPOINT}_{TENANT}_resp_size_bytes


------------
Alternatives
------------
Instead of using StatsD, a pure middleware based approach can be used to gather 
request metrics at node/API endpoint/tenant levels, using Scales which is used 
by Cassandra python driver. No new dependency will be introduced.


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

StatsD server will need to be deployed and configured. 


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
