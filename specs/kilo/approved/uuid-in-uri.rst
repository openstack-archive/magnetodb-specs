..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Add UUID in uri
========================

Launchpad: table-uuid-in-uri_

.. _table-uuid-in-uri:
   https://blueprints.launchpad.net/magnetodb/+spec/table-uuid-in-uri

Implement table resource lookup by UUID, not only name

Problem description
===================

---------
Use Cases
---------

UUID is used as an intristic identifier of table. We need to expose in for
Ceilometer purposes. Resource lookup by UUID is closer for OpenStack
and defines resource unambiguously.

Proposed change
===============

1. Add primary table lookup by UUID in any uri, that contains table name or UUID
2. List table response will contain table UUID
3. Describe table will contatin table UUID

------------
Alternatives
------------

-----------------
Data model impact
-----------------

No data is stored or cached.


---------------
REST API impact
---------------

Table name in any request except 'create table' can be substituted by table UUID.
Additionally 'list table' and 'describe table' responses will contain table UUID.


URL for the resource
````````````````````

    v1/{tenant_id}/data/tables/{table_uuid|table_name}


Response Syntax
```````````````

List table:
::
        {
            "last_evaluated_table_name": "string",
            "tables": [
                {
                    "rel": "string",
                    "href": "url",
                    "uuid": "string"
                }
            ]
        }

Describe table:
::
        {
            "table": {
                "attribute_definitions": [
                    {
                        "attribute_name": "string",
                        "attribute_type": "string"
                    }
                ],
                "creation_datetime": "number",
                "item_count": "number",
                "key_schema": [
                    {
                        "attribute_name": "string",
                        "key_type": "string"
                    }
                ],
                "local_secondary_indexes": [
                    {
                        "index_name": "string",
                        "index_size_bytes": "number",
                        "item_count": "number",
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
                        }
                    }
                ],
                "links": [
                    {
                        "href": "url",
                        "rel": "self"
                    }
                ],
                "uuid": "string",
                "table_name": "string",
                "table_size_bytes": "number",
                "table_status": "string"
            }
        }


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
  <aostapenko>

Other contributors:
  <None>


----------
Work Items
----------

1. Update interface for data and monitoring API.
2. Update documentation.


Dependencies
============

None


Testing
=======

None


Documentation Impact
====================

* Updated API section should be added to documentation_.

.. _documentation:
   http://magnetodb.readthedocs.org/en/latest/api_reference.html


References
==========

