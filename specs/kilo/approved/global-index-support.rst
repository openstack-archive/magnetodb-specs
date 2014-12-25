..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Example Spec - The title of your blueprint
==========================================


https://blueprints.launchpad.net/magnetodb/+spec/global-index-support

Global Secondary Index is essential for making a distributed database
functionally complete.

Problem description
===================

MagnetoDB doesn't have support for global secondary indices yet. This blueprint
aims to solve that.

Use Cases
----------

A person wants to cluster data based on a column which is not a part of the
primary key. Let's take keystone token as an example. Tokens will be stored
with token UUID as the primary hash key. But to support deleting all tokens for
a particular user, one would need some way to 'cluster' information so that
he/she retrieves all tokens UUIDs for a particular user. Global secondary
indices provide exactly that.

Proposed change
===============

The proposal is to use additional Cassandra tables which will act as global
indices for the original table. These tables will not be visible through the
MagnetoDB API layer. Information of these new tables, which will act as GSI
will be kept in table_info table.

When an insert operation is performed on a table which has global secondary
index created over it, we will need to perform insertions in both the tables --
the primary table and the table which is acting as the GSI. We will use logged
batch for such an insertion.

Logged batches can have two failure scenarios -- either update of the internal
batchlog table can fail, or after update to batchlog succeeds, one or more
inserts of the batch operation fails. In the second failure scenario, Cassandra
ensures that the operation will be retried again, and 'eventually' it will
succeed. Currently we will just throw an internal
server in case of any of these two errors, but we'll need to think about better
way to handle this in the future.

Global secondary index has another unique property that hash and range primary
keys can also have duplicates. This functionality will not handled under this
blueprint, but should be implemented in future.


Alternatives
------------

Not use batch operations at all, or use unlogged batches. The advantage would
be we would be better able to handle error cases (if an insert fails, it will
not be retried, and be done in an 'eventual' fashion, which might be
undesirable in some cases). The disadvantage is that in case of partial updates
(e.g. update to primary table is succeessful but not to GSI table), we'll need
to do the error handling in MagnetoDB API server (python code)

Data model impact
-----------------

New entries to table_info table will be added to accommodate associated global
secondary index tables.

REST API impact
---------------

CreateTable and UpdateTable API will be affected, and will now include
parameters to create and modify global secondary index.


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
if the table has many global secondary indexes. Also, we would like to keep the
table information as much cached as possible, so that we don't incur a round
trip to Cassandra cluster before each insert. Similarly for updates and
deletes.

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
