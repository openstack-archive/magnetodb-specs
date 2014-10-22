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


Problem description
===================

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

Response::

        {
            'size': 1003432,
            'item_count': 3000
        }



Alternatives
------------

Implement as part of data API. However it gives access to data itself for
monitoring tools.



Data model impact
-----------------

None



REST API impact
---------------

Method type
```````````

GET

Normal http response code(s)
````````````````````````````

200

Expected error http response code(s)
````````````````````````````````````

500

Type: "JolokiaError"

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


Security impact
---------------

* authorization is performed by user's token
* authorization can be performed by token with specific role with
  permission to call monitoring API
* MagnetoDB API node should have access to Jolokia HTTP endpoint


Notifications impact
--------------------

None


Other end user impact
---------------------

None


Performance Impact
------------------

Now we have POC for count items in table
(https://review.openstack.org/#/c/118557/) but this implementation is very
expensive (we scan all our data to get one counter). So we need change
concept and use Cassandra JMX to get approximate number of items and also
size of CF on disk.


Other deployer impact
---------------------

* Jolokia component is used for collecting data from Cassandra

  * Cassandra start script should be updated with jolokia-jvm-agent_

* MagnetoDB API node should have access to Jolokia HTTP endpoint

.. _jolokia-jvm-agent:
   http://www.jolokia.org/agent/jvm.html


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


Assignee(s)
-----------

Primary assignee:
  <ominakov>

Other contributors:
  <achudnovets>
  <SpyRay>


Work Items
----------

1. Define Monitoring API on wiki.
2. Write interface for API.
3. Implement calls from MagnetoDB to Jolokia agent.
4. Update documentation.


Dependencies
============

Devstack integration update


Testing
=======

Tempest tests for this functional is not possible to implement, as we get
the approximate values ​​of the size and number of keys from the database.


Documentation Impact
====================

* Monitoring API section should be added to documentation_.
* Deployment_ guide should be updated.

.. _documentation:
   http://magnetodb.readthedocs.org/en/latest/api_reference.html

.. _Deployment:
   http://magnetodb.readthedocs.
   org/en/latest/admin_guide.html#installation-guide


References
==========

Jolokia_

.. _Jolokia:
   http://www.jolokia.org/

https://review.openstack.org/#/c/122330/