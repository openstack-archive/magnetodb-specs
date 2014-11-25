..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Add UUID in create, describe, delete table response bodies 
========================

Launchpad: table-uuid_

.. _table-uuid:
   https://blueprints.launchpad.net/magnetodb/+spec/table-uuid

Add table uuid field on table creation

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

1. Table uuid will be generated on table creation. Create, delete, describe table response bodies will contain table uuid


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

Response bodies of create, describe and delete table operations will additionally contain table UUID.


Response Syntax
```````````````

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
                "table_id": "string",
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

1. Update documentation.


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

