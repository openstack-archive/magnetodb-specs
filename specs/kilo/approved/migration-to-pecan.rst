..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
MagnetoDB migration to pecan
============================

Launchpad: migration-to-pecan_

.. _migration-to-pecan:
   https://blueprints.launchpad.net/magnetodb/+spec/migration-to-pecan

Migration to pecan framework.

Problem description
===================

Now in MagnetoDB are  used several separate libs for API implementation, it
would be useful to do migration to pecan lib which is widespread in openstack
projects.

Proposed change
===============

Refactor MagnetoDB API within migration to pecan lib.

------------
Alternatives
------------

None

-----------------
Data model impact
-----------------

None

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
  <idegtiarov>

----------
Work Items
----------

1. Refactor api.v1.app_factory by the use of pecan lib.
2. Remove common.wsgi.Router.

Dependencies
============

None

Testing
=======

Unittest

Documentation Impact
====================

None

References
==========

* `Docs for pecan`_

Example of using pecan in openstack projects API:

* `Pecan using in ceilometer API`_
* `Pecan using in ironic API`_

.. _Docs for pecan: http://pecan.readthedocs.org/
.. _Pecan using in ceilometer API: https://github.com/openstack/ceilometer/blob/master/ceilometer/api/controllers/v2.py
.. _Pecan using in ironic API: https://github.com/openstack/ironic/blob/master/ironic/api/controllers/v1/driver.py

