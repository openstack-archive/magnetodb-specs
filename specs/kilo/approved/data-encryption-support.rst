..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Data encryption support
========================

Launchpad: data-encryption-support_

.. _data-encryption-support:
   https://blueprints.launchpad.net/magnetodb/+spec/data-encryption-support

Supporting of data encryption


Problem description
===================

The aim of this blueprint is having secure stored data on drive. This means
that if malefactor get access to our hard disk for example, he will have no
ability to read tenant's data.

This is pretty big task and it touches few components let us go throw them


---------
Use Cases
---------

User wants to encrypt tenant's data using secret key


Proposed change
===============

This change requires management api implemented first for providing per tenant
options (encryption key)

To have securely stored data we need to have:

- Data encrypted
- Key for encryption not stored on the same drive (at least without
  encryption)

Also one more important point. MagnetoDB allows us to perform range queries.
This means that MagnetoDB requires keys which could be compared one to another
to store this keys ordered.

If we are using third-party storage
(Cassandra for example) we need to implement encryption between Cassandra and
our drive. It seems that the most suitable for this task is file
based encryption file system (like eCryptFS for Linux).

For this case we need:

- implement magnetodb-cassandra-agent. And run it on each Cassandra node
  for mounting and umounting eCryptFS partitions
- improve async-task-executor for performing management-api requests
  (in scope of management-api implementation)
- integrate with Barbican for storing secrets for encryption

------------
Alternatives
------------

Use storage internal mechanism for data encryption


-----------------
Data model impact
-----------------

It looks like we need to move our tenant/table info repository out from data
storage (Cassandra) to some dedicated storage (MySQL for example)


---------------
REST API impact
---------------

Doesn't require changing of existing API


---------------
Security impact
---------------

Secrets shouldn't be stored in persistent storages like hard drive, only
in memory. If we need to get secret we suold ask external keymanagement API
(Barbican)


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

eCryptFS is fast but will consume some CPU with depends on alhorithm and its
paramenters (like key length) choosen

---------------------
Other deployer impact
---------------------

Deploing of magnetodb-cassandra-agent is required for each Cassandra node




-----------
Assignee(s)
-----------

Primary assignee:
  <None>


----------
Work Items
----------

1. Improve management-api implementation for handling encryption property
   corectly
2. move out tenant/table info repository to dedicated storage
3. Implement magnetodb-cassandra-agent
4. Integrate with Barbican


Dependencies
============

management-api implementation


Testing
=======

set of tempest tests will cover this functionality


Documentation Impact
====================

Deployment procedure should be updated.
New storage options for encryption should be described
