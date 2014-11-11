..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
MagnetoDB backup/restore API
============================

Launchpad: backup-restore-api_

.. _backup-restore-api:
   https://blueprints.launchpad.net/magnetodb/+spec/backup-restore-api

The API for creating and managing backups in MagnetoDB.

Problem description
===================

---------
Use Cases
---------

As a MagnetoDB user I want to create a backup of a table, list available
backups, get status of and information about a backup, delete backups,
restore data from backup, list available restore jobs and get information
about a restore job.

Proposed change
===============

1. Introduce backup API available on the same port as data API by path
   /v1/{tenant_id}/backup/{table_name}...
2. Introduce restore API available on the same port as data API by path
   /v1/{tenant_id}/restore/{table_name}...

------------
Alternatives
------------

-----------------
Data model impact
-----------------

Data models for storing information about backup and restore jobs should be introduced.

---------------
REST API impact
---------------

Create backup
-------------

'Create backup' is an asynchronous method. In case of success it returns
immediately with 202 status. That means that backup job was enqueued.
This job's status then can be checked via 'describe backup' method call.

**Method type**

POST

**Normal http response code(s)**

202

**Expected error http response code(s)**

403
404
500


**URL for the resource**

/v1/{tenant_id}/management/{table_name}/backup

**Request Syntax**

::

    {
        "name": "string",
        "strategy": "object"
    }


**Request Elements**

*name*
   * Name of a backup to create
   * Type: String
   * Required: Yes

*strategy*
   * A strategy to use for the backup.
     Matching strategy will be used for restoring from this backup
   * Type: Object
   * Required: No

**Response Syntax**

::

    {
        "id": "UUID",
        "name": "string",
        "table_name": "string",
        "status": "string",
        "start_datetime": "string",
        "strategy": "object",
        "links": [
            {
                "rel": "self",
                "href": "url"
            },
            {
                "rel": "location",
                "href": "url"
            }
        ]
    }


**Response Elements**

*id*
   * Backup ID
   * Type: UUID

*name*
   * Backup name
   * Type: String

*table_name*
   * Table name
   * Type: String

*status*
   * Backup status
   * Type: String

*start_datetime*
   * Date and time when backup operation was started
   * Type: String

*strategy*
   * Strategy used for the backup
   * Type: Object

*links*
   * Links to the backup info and to the backup location
   * Type: Array of objects

List backups
------------

**Method type**

GET

**Normal http response code(s)**

200

**Expected error http response code(s)**

404
500


**URL for the resource**

/v1/{tenant_id}/management/{table_name}/backups


**Request Parameters**

Parameters should be provided via URL.

*exclusive_start_backup_name*
   * The last backup name evaluated in previous operation
   * Type: String
   * Required: No

*limit*
   * A maximum number of the items to return
   * Type: int
   * Required: No


**Response Syntax**

::

        {
            "last_evaluated_backup_id": "UUID",
            "backups": [
                {
                    "id": "UUID",
                    "name": "string",
                    "table_name": "string",
                    "status": "string",
                    "start_datetime": "string",
                    "finish_datetime": "string",
                    "strategy": "object",
                    "links": [
                        {
                            "rel": "self",
                            "href": "url"
                        },
                        {
                            "rel": "location",
                            "href": "url"
                        }
                    ]
                }
            ]
        }


**Response Elements**

*last_evaluated_backup_id*
   * The ID of the last backup in the current page of results
   * Type: String

*backups*
   * Array of the backup info items
   * Type: Array of objects



Describe backup
---------------

**Method type**

GET

**Normal http response code(s)**

200

**Expected error http response code(s)**

404
500


**URL for the resource**

/v1/{tenant_id}/management/{table_name}/backups/{backup_id}



**Request Syntax**

None

**Request Elements**

None


**Response Syntax**

::

    {
        "id": "UUID",
        "name": "string",
        "table_name": "string",
        "status": "string",
        "start_datetime": "string",
        "finish_datetime": "string",
        "strategy": "object"
        "links": [
            {
                "rel": "self",
                "href": "url"
            },
            {
                "rel": "location",
                "href": "url"
            }
        ]
    }

**Response Elements**

*id*
   * Backup ID
   * Type: UUID

*name*
   * Backup name
   * Type: String

*table_name*
   * Table name
   * Type: String

*status*
   * Backup status
   * Type: String

*start_datetime*
   * Date and time when backup operation was started
   * Type: String

*finish_datetime*
   * Date and time when backup operation was finished
   * Type: String

*strategy*
   * Strategy used for the backup
   * Type: Object

*links*
   * Links to the backup info and to the backup location
   * Type: Array of objects


Delete backup
-------------

**Method type**

DELETE

**Normal http response code(s)**

200

**Expected error http response code(s)**

403
404
500


**URL for the resource**

/v1/{tenant_id}/management/{table_name}/backups/{backup_id}


**Request Syntax**

None

**Request Elements**

None


**Response Syntax**

::

    {
        "id": "UUID",
        "name": "string",
        "table_name": "string",
        "status": "string",
        "start_datetime": "string",
        "finish_datetime": "string",
        "strategy": "object",
        "links": [
            {
                "rel": "self",
                "href": "url"
            },
            {
                "rel": "location",
                "href": "url"
            }
        ]
    }

**Response Elements**

*id*
   * Backup ID
   * Type: UUID

*name*
   * Backup name
   * Type: String

*table_name*
   * Table name
   * Type: String

*status*
   * Backup status
   * Type: String

*start_datetime*
   * Date and time when backup operation was started
   * Type: String

*finish_datetime*
   * Date and time when backup operation was finished
   * Type: String

*strategy*
   * Strategy used for the backup
   * Type: Object

*links*
   * Links to the backup info and to the backup location
   * Type: Array of objects


Restore from backup
-------------------

'Restore from backup' is an asynchronous method. In case of success it returns
immediately with 202 status. That means that restore job was enqueued.
This job's status then can be checked via 'describe restore job' method call.

**Method type**

POST

**Normal http response code(s)**

202

**Expected error http response code(s)**

403
404
500


**URL for the resource**

/v1/{tenant_id}/management/{table_name}/restore


**Request Syntax**

::

    {
        "id": "UUID",
        "source": "string"
    }


**Request Elements**

*id*
   * ID of a backup to restore from
   * Type: UUID
   * Required: Yes


*source*
   * URL of a data source to restore from
   * Type: String
   * Required: Yes

Elements *name* and *source* can not be set simultaneously.

**Response Syntax**

::

    {
        "id": "UUID",
        "backup_id": "UUID",
        "table_name": "string",
        "status": "string",
        "start_datetime": "string",
        "links": [
            {
                "rel": "self",
                "href": "url"
            },
            {
                "rel": "source",
                "href": "url"
            }
        ]
    }


**Response Elements**

*id*
   * Restore job ID
   * Type: UUID

*backup_id*
   * Backup ID
   * Type: UUID

*table_name*
   * Table name
   * Type: String

*status*
   * Restore status
   * Type: String

*start_datetime*
   * Date and time when restore operation was started
   * Type: String

*links*
   * Links to the restore job info and to the source backup
   * Type: Array of objects


List restore jobs
-----------------

**Method type**

GET

**Normal http response code(s)**

200

**Expected error http response code(s)**

404
500


**URL for the resource**

/v1/{tenant_id}/management/{table_name}/restore


**Request Parameters**

Parameters should be provided via URL.

*exclusive_start_restore_job_id*
   * The last restore job Id evaluated in previous operation
   * Type: String
   * Required: No

*limit*
   * A maximum number of the items to return
   * Type: int
   * Required: No


**Response Syntax**

::

        {
            "last_evaluated_restore_job_id": "string",
            "restore_jobs": [
                {
                    "id": "UUID"
                    "backup_id": "UUID",
                    "table_name": "string",
                    "status": "string",
                    "start_datetime": "string",
                    "finish_datetime": "string",
                    "links": [
                        {
                            "rel": "self",
                            "href": "url"
                        },
                        {
                            "rel": "source",
                            "href": "url"
                        }
                    ]
                }
            ]
        }


**Response Elements**

*last_evaluated_backup_name*
   * The ID of the last restore job in the current page of results
   * Type: String

*backups*
   * Array of the restore job info items
   * Type: Array of objects


Describe restore job
--------------------

**Method type**

GET

**Normal http response code(s)**

200

**Expected error http response code(s)**

404
500


**URL for the resource**

/v1/{tenant_id}/management/{table_name}/restore/{restore_job_id}



**Request Syntax**

None


**Request Elements**

None


**Response Syntax**

::

    {
        "id": "UUID"
        "backup_id": "UUID",
        "table_name": "string",
        "status": "string",
        "start_datetime": "string",
        "finish_datetime": "string",
        "links": [
            {
                "rel": "self",
                "href": "url"
            },
            {
                "rel": "source",
                "href": "url"
            }
        ]
    }


**Response Elements**

*id*
   * Restore job ID
   * Type: UUID

*backup_id*
   * Backup ID
   * Type: UUID

*table_name*
   * Table name
   * Type: String

*status*
   * Restore status
   * Type: String

*start_datetime*
   * Date and time when restore operation was started
   * Type: String

*finish_datetime*
   * Date and time when restore operation was finished
   * Type: String

*links*
   * Links to the restore job info and to the source backup
   * Type: Array of objects


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

1. Define Backup/Restore API.
2. Update documentation.


Dependencies
============

None


Testing
=======

None


Documentation Impact
====================

* Backup/Restore API section should be added to documentation_.

.. _documentation:
   http://magnetodb.readthedocs.org/en/latest/api_reference.html


References
==========

https://review.openstack.org/133933

