..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================================
Cassandra storage driver redesign for native Cassandra LSI support
==================================================================

Launchpad: cassandra-backend-implementation-redesign_

.. _cassandra-backend-implementation-redesign:
   https://blueprints.launchpad.net/magnetodb/+spec/cassandra-backend-implementation-redesign


Problem description
===================

Current LSI implementation for Cassandra storage driver doesn't use any
Cassandra support for this feature. It makes index update operations too slow
becase it requires reading of old indexed value before write and in addition
these two operation (read and write) should be executed atomicaly.

Proposed change
===============

In this blueprint I suggested to write cassandra extension
for MagnetoDB-specific LSI support. Then it makes possible to
execute cassandra query "CREATE CUSTOM INDEX ..." on table creation stage
And then put and update data without doing index update staff on MagnetoDB side


------------
Alternatives
------------

Leave current implementation as is

-----------------
Data model impact
-----------------

Now we store indexes in the same table with data. We need to remove indexes
and leave only data here

---------------
REST API impact
---------------

None

---------------
Security impact
---------------

None

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

Should make performance for operations with indexes much faster


---------------------
Other deployer impact
---------------------

Building java library with custom cassandra LSI implementation and its
deploing to cassandra/lib directory is required for each cassandra node
before node bootstrapping

----------------
Developer impact
----------------

Examples for queries:

CREATE TABLE u_table1(
    u_hash_key int,
    u_range_key text,
    u_indexed_field_1 text,
    u_indexed_field_2 int,
    u_not_indexed_field text,
    query_properties text,
    PRIMARY KEY (u_hash_key, u_range_key)
);

CREATE CUSTOM INDEX ON u_table1(query_properties) USING
    'com.mirantis.magnetodb.cassandra.db.index.MagnetoDBLocalSecondaryIndex'
    WITH OPTIONS = {'query_properties_field':'true'};

CREATE CUSTOM INDEX ON u_table1(u_indexed_field_1) USING
    'com.mirantis.magnetodb.cassandra.db.index.MagnetoDBLocalSecondaryIndex'
    WITH OPTIONS = {'query_properties_field':'false'};

CREATE CUSTOM INDEX ON u_table1(u_indexed_field_2) USING
    'com.mirantis.magnetodb.cassandra.db.index.MagnetoDBLocalSecondaryIndex'
    WITH OPTIONS = {'query_properties_field':'false'};

SELECT * FROM test.u_table1 WHERE u_hash_key=1 and u_indexed_field_2>=3 and
    query_properties='reversed:true' ALLOW FILTERING;

Implementation
==============

We need to:
1. change Create/Delete table logic - not only table but its indexes too should be created
2. remove all logic for updating indexes - it will be done
   at cassandra side
3. change query from index CQL query format

-----------
Assignee(s)
-----------

Primary assignee:
  <dukhlov>

Other contributors:
  <None>


Dependencies
============

None


Testing
=======

None


Documentation Impact
====================

None


References
==========

None
