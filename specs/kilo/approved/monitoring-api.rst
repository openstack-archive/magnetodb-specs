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
2. Implement REST method for list of tables /{tenant_id}/monitoring/tables.
3. Table usage details /{tenant_id}/monitoring/table/{table_name}.


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



URL for the resource
````````````````````

v1/{tenant_id}/monitoring/table/{table_name}

Response Syntax
```````````````

::

        {
            'size': 1003432,
            'item_count': 3000
        }


List tables
-----------

Method type
```````````

GET

Normal http response code(s)
````````````````````````````

200

Expected error http response code(s)
````````````````````````````````````

500


URL for the resource
````````````````````

v1/{tenant_id}/monitoring/tables

Request Parameters
``````````````````

Parameters should be provided via GET query string.

**exclusive_start_table_name**
   * The first table name that this operation will evaluate.
   * Type: string
   * Required: No

**limit**
   * A maximum number of the items to return.
   * Type: int
   * Required: No

Response Syntax
```````````````

::

        {
            "last_evaluated_table_name": "string",
            "tables": [
                {
                    "rel": "string",
                    "href": "url"
                }
            ]
        }

Response Elements
`````````````````

**last_evaluated_table_name**
   * The name of the last table in the current page of results.
   * Type: String

**tables**
   * Array of the table info items
   * Type: array of structs


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

https://review.openstack.org/#/c/122330/
