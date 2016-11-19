Ignite is partially [Redis](http://redis.io/) compliant which enables users to store and retrieve distributed data from Ignite cache using any Redis compatible client.

In version 1.8.0 only the following commands are supported
- ECHO
- PING
- QUIT
- GET
- MGET (limitation: not null values returned for non-existing keys)
- SET (limitation: without key expiration)
- MSET
- INCR
- DECR
- INCRBY
- DECRBY
- APPEND
- STRLEN
- GETSET
- SETRANGE
- GETRANGE
- DEL (limitation: number of submitted keys returned)
- EXISTS
- DBSIZE

Cluster nodes accepts Redis requests listening on a particular socket. By default each Ignite node is listening for incoming requests on {{\[host\]:11211}}. You can override the host and port using ConnectorConfiguration class