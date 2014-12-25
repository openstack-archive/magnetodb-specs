..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================
Global Secondary Index support
==============================


https://blueprints.launchpad.net/magnetodb/+spec/global-index-support

Global Secondary Index is essential for making a distributed database
functionally complete.

Problem description
===================

MagnetoDB doesn't have support for global secondary indices yet. This blueprint
aims to solve that.

Use Cases
----------

A person wants to index data based on a column which is not a part of the
primary key, and for the same purpose he requires Global Index.


Proposed change
===============

The proposal is to use additional Cassandra tables which will act as global
indices for the original table. These tables will not be visible through the
MagnetoDB API layer. Information of these new tables, which will act as GSI
will be kept in table_info table.

When an insert operation is performed on a table which has global secondary
index created over it, we will need to perform insertions in both the tables --
the primary table and the table which is acting as the GSI.


Alternatives
------------

An alternative is to use batch operation for inserts. It comes with it's own
limitations, e.g. no conditional support in case of batch operations. There are
also some failure scenarios with batch operations which make them slightly
difficult to use, e.g. if the batch insert fails, it will be retried again till
it succeeds, and we have no control if we want to abort that retry.

Data model impact
-----------------

New entries to table_info table will be added to accommodate associated global
secondary index tables. A new column will be added to table_info table, which
will be a map; keys will be GSI index name, and values will be UUID which will
be the Cassandra table name for corresponding GSI.


REST API impact
---------------

API of create table and list tables will be modified. Response syntax of almost
all the API operations will change; will include information regarding global
secondary index. It will be very similar to DynamoDB's API request and response
syntax types for global secondary index.

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

Support in python-magnetodbclient for this feature will need to be added.

Performance Impact
------------------

Batch operations have some performance penalty, so an insert will take longer
if the table has many global secondary indexes, since we'll have to do as many
inserts in Cassandra.

Other deployer impact
---------------------

None

Developer impact
----------------

For cloud application developers, GSI feature will be available.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  rushiagr (Rushi Agrawal)

Other contributors:
  vivekd (Vivek Dhayal)
  ajayaa (Ajaya Agrawal)


Work Items
----------

* Create additional tables for GSI, update table_info table
* Cache table information as much as possible in magnetoDB layer
* Update python-magnetodbclient to include GSI support

Dependencies
============

None

Testing
=======

Comprehensive unit and integration (Tempest) tests will be added.

Documentation Impact
====================

Documentation for GSI will need to be added.


References
==========

* DynamoDB GSI doc
  http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html

* Cassandra batch operations doc
  http://www.datastax.com/documentation/cql/3.1/cql/cql_reference/batch_r.html
