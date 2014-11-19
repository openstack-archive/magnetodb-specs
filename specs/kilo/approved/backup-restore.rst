..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
MagnetoDB backup/restore
========================

Launchpad: backup-restore_

.. _backup-restore:
   https://blueprints.launchpad.net/magnetodb/+spec/backup-restore

Creating and managing backups in MagnetoDB.

Problem description
===================

---------
Use Cases
---------

As a MagnetoDB user I want to backup data in a table and the table structure
and to restore the table and data.

Proposed change
===============

Introduce per table backup/restore functionality in MagnetoDB.

------------
Alternatives
------------

Use Trove for backup/restore.

-----------------
Data model impact
-----------------

Data models for storing information about backup and restore
jobs should be introduced. As much metadata for a backup as
possible should be stored along with the backed up data.
This metadata should be enough to restore the data into
a freshly deployed MagnetoDB.
Also the metadata should be stored in internal MagnetoDB storage
to allow faster access for reading and modification.


---------------
REST API impact
---------------

Backup and restore functionality should be available to a user
through corresponding RESTful API. The API should have a possibility
for a user to choose backup details through providing a *strategy*.
The strategy should define such options as

* backup type - full, incremental
* data format - DB-agnostic (slower, cross-backend),
  native (faster, backend-specific)
* location of the backed up data - internal (Swift), 
  external (provide URL, credentials, etc.)

The strategy should be optional. Default strategy should provide
sane values, for example, full, DB-agnostic backup with data stored to
stock Swift.

Backup/restore API: spec_

.. _spec:
    http://magnetodb-specs.readthedocs.org/en/latest/specs/kilo/approved/backup-restore-api.html

---------------
Security impact
---------------

* authorization is performed by user's token
* authorization can be performed by token with specific role with
  permission to call backup/restore API


--------------------
Notifications impact
--------------------

Create backup, delete backup and restore from backup operations
sould send notifications when operation is started and finished
and in case of error.


---------------------
Other end user impact
---------------------

Data integrity only guaranteed on per item basis, that is if batch
update is running during back up process, some items may get updated
but others may not. But no item can get ‘half-updated’.


While some implementations of backup/restore may require table
(or even tenant) to be locked, general rule of thumb is, data
are not locked during backup and restore.

In case of data being locked corresponding table should be put 
in MAINTENANCE state and all data access requests should be responsed
with 423 'Locked' error code.

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
  <unassigned>

Other contributors:
  <unassigned>


----------
Work Items
----------


Dependencies
============

None


Testing
=======

None


Documentation Impact
====================


References
==========
http://magnetodb-specs.readthedocs.org/en/latest/specs/kilo/approved/backup-restore-api.html

https://review.openstack.org/137010

