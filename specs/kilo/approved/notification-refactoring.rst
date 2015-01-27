================================================
MagnetoDB Notification Refactoring Specification
================================================
With the introduction of StatsD metrics, MagnetoDB now has multiple ways of
sending events. Previous events delivered through oslo messaging are no longer
sufficient. We will use an event registry to store all information about events.
The event notifier will use the above information to decide how to deliver the
notification.

All existing notifications/events will be refactored accordingly.

Event Naming Convention
=======================

- event for API request: 

  - EVENT_TYPE_REQUEST_<REQUEST_NAME>
  - EVENT_TYPE_REQUEST_<REQUEST_NAME>_ERROR_xxx

- event for async task:

  - EVENT_TYPE_TASK_<TASK_NAME>_START
  - EVENT_TYPE_TASK_<TASK_NAME>_END
  - EVENT_TYPE_TASK_<TASK_NAME>_ERROR

- event for others

  - EVENT_TYPE_<FUNCTION_NAME>
  - EVENT_TYPE_<FUNCTION_NAME>_START
  - EVENT_TYPE_<FUNCTION_NAME>_END
  - EVENT_TYPE_<FUNCTION_NAME>_ERROR

Event Changes
=============

create table
------------
- TABLE_CREATE_START

  - event name: EVENT_TYPE_TABLE_CREATE_START
  - location: storage manager
  - new event name: EVENT_TYPE_TASK_TABLE_CREATE_START
  - notification level: audit
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: messaging
  - notes: changed

- TABLE_CREATE_END

  - event name: EVENT_TYPE_TABLE_CREATE_END
  - location: storage manager, async task executor
  - new event name: EVENT_TYPE_TASK_TABLE_CREATE_END
  - notification level: audit
  - new location: n/a
  - metrics name: mdb.task.create_table
  - delivery mechanism: messaging, statsd
  - notes: changed

- TABLE_CREATE_ERROR

  - event name: EVENT_TYPE_TABLE_CREATE_ERROR
  - location: storage manager, async task executor
  - new event name: EVENT_TYPE_TASK_TABLE_CREATE_ERROR
  - notification level: error
  - new location: n/a
  - metrics name: mdb.task.create_table.error
  - delivery mechanism: messaging, statsd
  - notes: changed

- REQUEST_TABLE_CREATE

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_TABLE_CREATE
  - notification level: audit
  - new location: middleware
  - metrics name: mdb.req.create_table
  - delivery mechanism: messaging, statsd
  - notes: new

- REQUEST_TABLE_CREATE_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_TABLE_CREATE_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.create_table.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_TABLE_CREATE

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_TABLE_CREATE
  - notification level: audit
  - new location: middleware
  - metrics name: mdb.dynamo.create_table
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_TABLE_CREATE_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_TABLE_CREATE_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.create_table.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

delete table
------------

- TABLE_DELETE_START

  - event name: EVENT_TYPE_TABLE_DELETE_START
  - location: storage manager
  - new event name: EVENT_TYPE_TASK_TABLE_DELETE_START
  - notification level: audit
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: messaging
  - notes: changed

- TABLE_DELETE_END

  - event name: EVENT_TYPE_TABLE_DELETE_END
  - location: storage manager, async task executor
  - new event name: EVENT_TYPE_TASK_TABLE_DELETE_END
  - notification level: audit
  - new location: n/a
  - metrics name: mdb.task.delete_table
  - delivery mechanism: messaging, statsd
  - notes: changed

- TABLE_DELETE_ERROR

  - event name: EVENT_TYPE_TABLE_DELETE_ERROR
  - location: storage manager, async task executor
  - new event name: EVENT_TYPE_TASK_TABLE_DELETE_ERROR
  - notification level: error
  - new location: n/a
  - metrics name: mdb.task.delete_table.error
  - delivery mechanism: messaging, statsd
  - notes: changed

- REQUEST_TABLE_DELETE

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_TABLE_DELETE
  - notification level: audit
  - new location: middleware
  - metrics name: mdb.req.delete_table
  - delivery mechanism: messaging, statsd
  - notes: new

- REQUEST_TABLE_DELETE_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_TABLE_DELETE_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.delete_table.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_TABLE_DELETE

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_TABLE_DELETE
  - notification level: audit
  - new location: middleware
  - metrics name: mdb.dynamo.delete_table
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_TABLE_DELETE_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_TABLE_DELETE_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.delete_table.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

list table
----------

- REQUEST_TABLE_LIST

  - event name: EVENT_TYPE_TABLE_LIST
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_TABLE_LIST
  - notification level: debug
  - new location: middleware
  - metrics name: mdb.req.list_table
  - delivery mechanism: messaging, statsd
  - notes: changed

- REQUEST_TABLE_LIST_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_TABLE_LIST_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.list_table.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_TABLE_LIST

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_TABLE_LIST
  - notification level: debug
  - new location: middleware
  - metrics name: mdb.dynamo.list_table
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_TABLE_LIST_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_TABLE_LIST_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.list_table.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

put item
--------

- REQUEST_PUTITEM

  - event name: EVENT_TYPE_DATA_PUTITEM
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_PUTITEM
  - notification level: info
  - new location: middleware
  - metrics name: mdb.req.put_item
  - delivery mechanism: messaging, statsd
  - notes: changed

- PUTITEM_START

  - event name: EVENT_TYPE_DATA_PUTITEM_START
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- PUTITEM_END

  - event name: EVENT_TYPE_DATA_PUTITEM_END
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- PUTITEM_ERROR

  - event name: EVENT_TYPE_DATA_PUTITEM_ERROR
  - location: storage manager
  - new event name: n/a
  - notification level: error
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- REQUEST_PUTITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_PUTITEM_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.put_item.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- REQUEST_PUTITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_PUTITEM
  - notification level: info
  - new location: middleware
  - metrics name: mdb.dynamo.put_item
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_PUTITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_PUTITEM_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.put_item.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

delete item
-----------

- REQUEST_DELETEITEM

  - event name: EVENT_TYPE_DATA_DELETEITEM
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_DELETEITEM
  - notification level: info
  - new location: middleware
  - metrics name: mdb.req.delete_item
  - delivery mechanism: messaging, statsd
  - notes: changed

- DELETEITEM_START

  - event name: EVENT_TYPE_DATA_DELETEITEM_START
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- DELETEITEM_END

  - event name: EVENT_TYPE_DATA_DELETEITEM_END
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- DELETEITEM_ERROR

  - event name: EVENT_TYPE_DATA_DELETEITEM_ERROR
  - location: storage manager
  - new event name: n/a
  - notification level: error
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- REQUEST_DELETEITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_DELETEITEM_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.delete_item.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_DELETEITEM

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_DELETEITEM
  - notification level: info
  - new location: middleware
  - metrics name: mdb.dynamo.delete_item
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_DELETEITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_DELETEITEM_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.delete_item.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

batch write
-----------

- BATCHWRITE_START

  - event name: EVENT_TYPE_DATA_BATCHWRITE_START
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- BATCHWRITE_END

  - event name: EVENT_TYPE_DATA_BATCHWRITE_END
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- REQUEST_BATCHWRITE

  - event name: n/a
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_BATCHWRITE
  - notification level: info
  - new location: middleware
  - metrics name: mdb.req.batch_write
  - delivery mechanism: messaging, statsd
  - notes: new

- REQUEST_BATCHWRITE_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_BATCHWRITE_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.batch_write.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_BATCHWRITE

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_BATCHWRITE
  - notification level: info
  - new location: middleware
  - metrics name: mdb.dynamo.batch_write
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_BATCHWRITE_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_BATCHWRITE_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.batch_write.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

batch read
----------

- BATCHREAD_START

  - event name: EVENT_TYPE_DATA_BATCHREAD_START
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- BATCHREAD_END

  - event name: EVENT_TYPE_DATA_BATCHREAD_END
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- REQUEST_BATCHREAD

  - event name: n/a
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_BATCHREAD
  - notification level: info
  - new location: middleware
  - metrics name: mdb.req.batch_read
  - delivery mechanism: messaging, statsd
  - notes: new

- REQUEST_BATCHREAD_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_BATCHREAD_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.batch_read.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_BATCHREAD

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_BATCHREAD
  - notification level: info
  - new location: middleware
  - metrics name: mdb.dynamo.batch_read
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_BATCHREAD_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_BATCHREAD_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.batch_read.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

update item
-----------

- REQUEST_UPDATEITEM

  - event name: EVENT_TYPE_DATA_UPDATEITEM
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_UPDATEITEM
  - notification level: info
  - new location: middleware
  - metrics name: mdb.req.update_item
  - delivery mechanism: messaging, statsd
  - notes: changed

- REQUEST_UPDATEITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_UPDATEITEM_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.update_item.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_UPDATEITEM

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_UPDATEITEM
  - notification level: info
  - new location: middleware
  - metrics name: mdb.dynamo.update_item
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_UPDATEITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_UPDATEITEM_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.update_item.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

get item
--------

- REQUEST_GETITEM

  - event name: EVENT_TYPE_DATA_GETITEM
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_GETITEM
  - notification level: info
  - new location: middleware
  - metrics name: mdb.req.get_item
  - delivery mechanism: messaging, statsd
  - notes: changed

- GETITEM_START

  - event name: EVENT_TYPE_DATA_GETITEM_START
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- GETITEM_END

  - event name: EVENT_TYPE_DATA_GETITEM_END
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- REQUEST_GETITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_GETITEM_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.get_item.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_GETITEM

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_GETITEM
  - notification level: info
  - new location: middleware
  - metrics name: mdb.dynamo.get_item
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_GETITEM_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_GETITEM_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.get_item.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

query
-----

- REQUEST_QUERY

  - event name: EVENT_TYPE_DATA_QUERY
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_QUERY
  - notification level: info
  - new location: middleware
  - metrics name: mdb.req.query
  - delivery mechanism: messaging, statsd
  - notes: changed

- QUERY_START

  - event name: EVENT_TYPE_DATA_QUERY_START
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- QUERY_END

  - event name: EVENT_TYPE_DATA_QUERY_END
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- REQUEST_QUERY_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_QUERY_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.query.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_QUERY

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_QUERY
  - notification level: info
  - new location: middleware
  - metrics name: mdb.dynamo.query
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_QUERY_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_QUERY_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.query.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

scan
----

- REQUEST_SCAN

  - event name: EVENT_TYPE_DATA_SCAN
  - location: storage manager
  - new event name: EVENT_TYPE_REQUEST_SCAN
  - notification level: info
  - new location: middleware
  - metrics name: mdb.req.scan
  - delivery mechanism: messaging, statsd
  - notes: changed

- SCAN_START

  - event name: EVENT_TYPE_DATA_SCAN_START
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- SCAN_END

  - event name: EVENT_TYPE_DATA_SCAN_END
  - location: storage manager
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: n/a
  - notes: removed

- REQUEST_SCAN_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_REQUEST_SCAN_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.req.scan.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

- DYNAMO_SCAN

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_SCAN
  - notification level: info
  - new location: middleware
  - metrics name: mdb.dynamo.scan
  - delivery mechanism: messaging, statsd
  - notes: new

- DYNAMO_SCAN_ERROR

  - event name: n/a
  - location: n/a
  - new event name: EVENT_TYPE_DYNAMO_SCAN_ERROR_xxx
  - notification level: error
  - new location: middleware
  - metrics name: mdb.dynamo.scan.error.xxx
  - delivery mechanism: messaging, statsd
  - notes: xxx is the http error code, new

streaming
---------

- STREAMING_PATH_ERROR

  - event name: EVENT_TYPE_STREAMING_PATH_ERROR
  - location: streaming server
  - new event name: n/a
  - notification level: error
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: messaging
  - notes: no change

- STREAMING_DATA_START

  - event name: EVENT_TYPE_STREAMING_DATA_START
  - location: streaming server
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: messaging
  - notes: no change

- STREAMING_DATA_END

  - event name: EVENT_TYPE_STREAMING_DATA_END
  - location: streaming server
  - new event name: n/a
  - notification level: info
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: messaging
  - notes: no change

- STREAMING_DATA_ERROR

  - event name: EVENT_TYPE_STREAMING_DATA_ERROR
  - location: streaming server
  - new event name: n/a
  - notification level: error
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: messaging
  - notes: no change

rate limiting
-------------

- REQUEST_RATE_LIMITED

  - event name: EVENT_TYPE_REQUEST_RATE_LIMITED
  - location: rate limit middleware
  - new event name: n/a
  - notification level: audit
  - new location: n/a
  - metrics name: n/a
  - delivery mechanism: messaging
  - notes: no change

