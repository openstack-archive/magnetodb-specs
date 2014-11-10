..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================================================
MagnetoDB Management API part for tenant specific configuration
===============================================================

managment-api_

.. _managment-api:
   https://blueprints.launchpad.net/magnetodb/+spec/tenant-storage-configuration-api

Problem description
===================

No we have no abilities for managing tenant storage configuration
(like replication factor for example). Default configuration can be specified
in magnetodb-api.conf and will be used for each tenant

---------
Use Cases
---------
We want to have use storage with different configration for each tenant


Proposed change
===============

I suggest to add REST API for creating storage for tenant with specific
configuration. Also method for deleting storage and describing storage are
required. Cloud admin or user shoud create storage first and only then
try to create tables there.

------------
Alternatives
------------

Use static per-tenant configuration in magnetodb-api.conf. This approach
hides all storage options from user. Also keep in mind please that in future
maybe we will add multible storage (table container with specific options)
support per single tenant 

-----------------
Data model impact
-----------------

tenant meta infortion repository is required. Now we have table meta
infortion repository. We need or improve existing one or add new repo

---------------
REST API impact
---------------

Set tenant storage configuration
--------------

Method type
```````````
PUT

Normal http response code(s)
````````````````````````````
200

Expected error http response code(s)
````````````````````````````````````
400

URL for the resource
````````````````````
v1/{tenant_id}/config

Request Parameters
``````````````````
Parameters should be specified in request's body which
is json with tenant's configuration

Response Syntax
```````````````
json with applied configuration

Get tenant storage configuration
--------------

Method type
```````````
GET

Normal http response code(s)
````````````````````````````
200

Expected error http response code(s)
````````````````````````````````````
400

URL for the resource
````````````````````
v1/{tenant_id}/config

Request Parameters
``````````````````

Response Syntax
```````````````
json with applied configuration

Drop tenant storage configuration
--------------

Method type
```````````
DELETE

Normal http response code(s)
````````````````````````````
200

Expected error http response code(s)
````````````````````````````````````
400

URL for the resource
````````````````````
v1/{tenant_id}/config

Request Parameters
``````````````````

Response Syntax
```````````````
json with dropped configuration

---------------
Security impact
---------------

--------------------
Notifications impact
--------------------

Other end user impact
---------------------

------------------
Performance Impact
------------------

---------------------
Other deployer impact
---------------------

----------------
Developer impact
----------------

Implementation
==============

-----------
Assignee(s)
-----------

Primary assignee:
  <None>

Other contributors:
  <None>

----------
Work Items
----------

Dependencies
============

Testing
=======

Documentation Impact
====================
New API method should be documented

References
==========

