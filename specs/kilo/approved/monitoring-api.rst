..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
MagnetoDB monitoring api
========================

Launchpad: monitoring-api_

.. _monitoring-api:
   https://blueprints.launchpad.net/magnetodb/+spec/monitoring-api

The API for exposing usage statistic for users, external monitoring or
billing tools.
::

 +--------------+                     +-----------------+
 |              |                     |                 |
 |  Ceilometer  |                     |                 |
 |  Horizon     | HTTP/monitoring API |                 |
 |  collectd,   +--------------------->  magnetodb+api  |
 |  statsd,     |                     |                 |
 |  nagios...   |                     |                 |
 |              |                     |                 |
 +--------------+                     +-----------------+


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

1. Introduce monitoring API available on the same port as data API by path
   /monitoring/...
2. Implement single method for tables detailes /monitoring.
3. Implement filters by table_name, table_id, project_id.


------------
Alternatives
------------

Implement as part of data API. However it gives access to data itself for
monitoring tools.


-----------------
Data model impact
-----------------

No data is stored or cached.


---------------
REST API impact
---------------

Table usage details
-------------------

Method type
```````````

GET

Normal http response code(s)
````````````````````````````

200

Expected error http response code(s)
````````````````````````````````````

500



URL for the monitoring resource
```````````````````````````````

v1/monitoring?project_id=my_project_id&table_name=my_table_name


Request Parameters
``````````````````

Filters should be passed as URL parameters.

**project_id**
   * Returns data filtered by specified project_id
   * Type: string
   * Required: No

**table_id**
   * Returns data for table with specified table_id
   * Type: string
   * Required: No

**table_name**
   * Returns data filtered by specified table_name
   * Type: string
   * Required: No



Response Syntax
```````````````

::

        [
            {
                "project_id": "123",
                "table_id": "1234",
                "table_name": "my_table",
                "usage_detailes": {
                    "metric1": 1003432,
                    "metric2": 3000
                }
            },
            ...
        ]


---------------
Security impact
---------------

* authorization is performed by user's token
* authorization can be performed by token with specific role with
  permission to call monitoring API


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
  <ominakov>

Other contributors:
  <achudnovets>


----------
Work Items
----------

1. Define Monitoring API on wiki.
2. Write interface for API.
3. Update documentation.


Dependencies
============

None


Testing
=======

None


Documentation Impact
====================

* Monitoring API section should be added to documentation_.

.. _documentation:
   http://magnetodb.readthedocs.org/en/latest/api_reference.html


References
==========

None
