..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================================
Global Secondary Index API specification
========================================


https://blueprints.launchpad.net/magnetodb/+spec/global-index-support

Global Secondary Index is essential for making a distributed database
functionally complete. This spec talks about the specification of the API.

Problem description
===================

MagnetoDB doesn't have support for global secondary indices yet. This blueprint
aims to lay down the API specification for the same. This is so that once the
GSI support is present in Cassandra, we have the API interface defined
properly. The desire is to use this API specification to create a backend for
all OpenStack projects one-by-one, without waiting for GSI support in Cassandra
to land.

Use Cases
----------

A person wants to index data based on a column which is not a part of the
primary key, and for the same purpose he requires Global Index.


Proposed change
===============

In this section, changes to API requests and responses will be described.

* Create Table API request:

Addition to the API request:

..
    "global_secondary_indexes": [
        {
            "index_name": "string",
            "key_schema": [
                {
                    "attribute_name": "string",
                    "key_type": "string"
                }
            ],
            "projection": {
                "nonkey_attributes": [
                    "string"
                ],
                "projection_type": "string"
            },
        }
    ],

Addition to API response:

..
    "global_secondary_indexes": [
            {
                "index_name": "string",
                "index_size_bytes": "number",  # unused
                "index_status": "string",     # unused
                "item_count": "number",       # unused
                "key_schema": [
                    {
                        "attribute_name": "string",
                        "key_type": "string"
                    }
                ],
                "projection": {
                    "non_key_attributes": [
                        "string"
                    ],
                    "projection_type": "string"
                },
            }
        ],

Note: Keys for whom 'unused' comments are written could be empty till the
implementation of the same is in place. Alternatively, we can remove those keys
from the API completely.

* Delete Table API.

No change in the request API format. Response API format same as create table
API.

* Update table API

No change in the request API format. Response API format same as create table
API. Note that currently we're not expecting in Cassandra the functionality of
updating Global Index 'after' the table has been created.

* List tables API
  No change to request and response API format

* Put item API
  No change to request and response API format

* Get item API
  No change to request and response API format

* Update item API
  No change to request and response API format

* Delete item API
  No change to request and response API format

* Batch write item API
  No change to request and response API format

* Batch get item API
  No change to request and response API format

* Query API
  No change to request and response API format

* Scan API
  No change to request and response API format


Alternatives
------------

None

Data model impact
-----------------

N/A

REST API impact
---------------

Changes to API described in 'Problem Description' section.
It will be very similar to DynamoDB's API request and response
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

N/A

Other deployer impact
---------------------

None

Developer impact
----------------

For cloud application developers, GSI API will be available

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

N/A

Dependencies
============

None

Testing
=======

N/A

Documentation Impact
====================

Documentation for GSI will need to be added.


References
==========

* DynamoDB GSI doc
  http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html
