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
restore data from backup, list available restore jobs and fet information
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

**Method type**

POST

**Normal http response code(s)**

202

**Expected error http response code(s)**

403
404
500


**URL for the resource**

/v1/{tenant_id}/backup/{table_name}

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
   * Name of a strategy to use for the backup
   * Type: String
   * Required: No


**Response Syntax**

::

    {
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
                "rel": "storage",
                "href": "url"
            }
        ]
    }


**Response Elements**

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

List backups
------------

Method type
```````````

GET

Normal http response code(s)
````````````````````````````

200

Expected error http response code(s)
````````````````````````````````````

404
500


URL for the resource
````````````````````

/v1/{tenant_id}/backup/{table_name}


Request Parameters
``````````````````

Parameters should be provided via URL.

**exclusive_start_backup_name**
   * The last backup name evaluated in previous operation
   * Type: String
   * Required: No

**limit**
   * A maximum number of the items to return
   * Type: int
   * Required: No


Response Syntax
```````````````

::

        {
            "last_evaluated_backup_name": "string",
            "backups": [
                {
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
                            "rel": "storage",
                            "href": "url"
                        },
                    ]
                }
            ]
        }


Response Elements
`````````````````

**last_evaluated_backup_name**
   * The name of the last backup in the current page of results
   * Type: String

**backups**
   * Array of the backup info items
   * Type: Array of objects



Describe backup
---------------

Method type
```````````

GET

Normal http response code(s)
````````````````````````````

200

Expected error http response code(s)
````````````````````````````````````

404
500


URL for the resource
````````````````````

/v1/{tenant_id}/backup/{table_name}/{backup_name}


Request Syntax
``````````````

None

Request Elements
````````````````

None


Response Syntax
```````````````

::

    {
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
                "rel": "storage",
                "href": "url"
            }
        ]
    }

Response Elements
`````````````````

**name**
   * Backup name
   * Type: String

**table_name**
   * Table name
   * Type: String

**status**
   * Backup status
   * Type: String

**start_datetime**
   * Date and time when backup operation was started
   * Type: String

**finish_datetime**
   * Date and time when backup operation was finished
   * Type: String

**strategy**
   * Strategy used for the backup
   * Type: Object

**links**
   * Links to the backup info and to the backup location
   * Type: Array of objects


Delete backup
-------------

Method type
```````````

DELETE

Normal http response code(s)
````````````````````````````

200

Expected error http response code(s)
````````````````````````````````````

403
404
500


URL for the resource
````````````````````

/v1/{tenant_id}/backup/{table_name}/{backup_name}


Request Syntax
``````````````

None

Request Elements
````````````````

None


Response Syntax
```````````````

::

    {
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
                "rel": "storage",
                "href": "url"
            }
        ]
    }

Response Elements
`````````````````

**name**
   * Backup name
   * Type: String

**table_name**
   * Table name
   * Type: String

**status**
   * Backup status
   * Type: String

**start_datetime**
   * Date and time when backup operation was started
   * Type: String

**finish_datetime**
   * Date and time when backup operation was finished
   * Type: String

**strategy**
   * Strategy used for the backup
   * Type: Object

**links**
   * Links to the backup info and to the backup location
   * Type: Array of objects


Restore from backup
-------------------

Method type
```````````

POST

Normal http response code(s)
````````````````````````````

202

Expected error http response code(s)
````````````````````````````````````

403
404
500


URL for the resource
````````````````````

/v1/{tenant_id}/restore/{table_name}


Request Syntax
``````````````

::

    {
        "name": "string"
    }


Request Elements
````````````````

**name**
   * Name of a backup to restore from
   * Type: String
   * Required: Yes


Response Syntax
```````````````

::

    {
        "id": "string"
        "backup_name": "string",
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


Response Elements
`````````````````

**id**
   * Restore job Id
   * Type: String

**backup_name**
   * Backup name
   * Type: String

**table_name**
   * Table name
   * Type: String

**status**
   * Restore status
   * Type: String

**start_datetime**
   * Date and time when restore operation was started
   * Type: String

**finish_datetime**
   * Date and time when restore operation was finished
   * Type: String

**links**
   * Links to the restore job info and to the source backup
   * Type: Array of objects


List restore jobs
-----------------

Method type
```````````

GET

Normal http response code(s)
````````````````````````````

200

Expected error http response code(s)
````````````````````````````````````

404
500


URL for the resource
````````````````````

/v1/{tenant_id}/restore/{table_name}


Request Parameters
``````````````````

Parameters should be provided via URL.

**exclusive_start_restore_job_id**
   * The last restore job Id evaluated in previous operation
   * Type: String
   * Required: No

**limit**
   * A maximum number of the items to return
   * Type: int
   * Required: No


Response Syntax
```````````````

::

        {
            "last_evaluated_restore_job_id": "string",
            "restore_jobs": [
                {
                    "id": "string"
                    "backup_name": "string",
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


Response Elements
`````````````````

**last_evaluated_backup_name**
   * The Id of the last restore job in the current page of results
   * Type: String

**backups**
   * Array of the restore job info items
   * Type: Array of objects


Describe restore job
--------------------

Method type
```````````

GET

Normal http response code(s)
````````````````````````````

200

Expected error http response code(s)
````````````````````````````````````

404
500


URL for the resource
````````````````````

/v1/{tenant_id}/backup/{table_name}/{restore_job_id}


Request Syntax
``````````````

None


Request Elements
````````````````

None


Response Syntax
```````````````

::

    {
        "id": "string"
        "backup_name": "string",
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


Response Elements
`````````````````

**id**
   * Restore job Id
   * Type: String

**backup_name**
   * Backup name
   * Type: String

**table_name**
   * Table name
   * Type: String

**status**
   * Restore status
   * Type: String

**start_datetime**
   * Date and time when restore operation was started
   * Type: String

**finish_datetime**
   * Date and time when restore operation was finished
   * Type: String

**links**
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

Data integrity only guaranteed on per item basic, that is if batch
update is running during back up process, some items may get updated
but others may don’t. But no item can get ‘half-updated’.


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

