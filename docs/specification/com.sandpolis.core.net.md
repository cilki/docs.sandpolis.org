# Networking Module

## Sandpolis Protocol

Sandpolis uses a custom binary protocol based on
[protocol buffers](https://developers.google.com/protocol-buffers) for all
inter-instance network communications. By default, the server listens on TCP
port **8768**.

Most communication happens over TCP connections among instances and the server,
but the server can also coordinate direct TCP or UDP "sessions" between any two
instances that need to transfer high-volume or low-latency data.

## Streams

Many operations require real-time data for a short-lived or long-lived session.

All streams have a _source_ and a _sink_ and can exist between any two instances
(a stream where the source and sink reside on the same instance is called a
_local stream_). The source's purpose is to produce _stream events_ at whatever
frequency is appropriate for the use-case and the sink's purpose is to consume
those stream events.

### Multicasting

Stream sources can push events to more than one sink simultaneously. This is
called multicasting and can save bandwidth in situations where multiple users
request the same resource at the same time.

## Messages

| Message                 | Sources           | Destinations      | Description                                                              |
| ----------------------- | ----------------- | ----------------- | ------------------------------------------------------------------------ |
| RQ_Session              | `client`, `agent` | `server`          |
| RS_Session              | `server`          | `client`,`agent`  |
| RQ_AddConnection        | `client`          | `server`          |
| RQ_CoordinateConnection | `server`          | `client`, `agent` |
| EV_NetworkChanged       | `server`          | `client`, `agent` | Indicates that some node in the network has changed in connection status |
| RQ_InstallPlugin        | `client`          | `server`          | Request that a new plugin be installed                                   |
| RQ_STStream             | `client`, `agent` | `server`          | Request a new state tree sync stream                                     |
| EV_STStreamData         | `server`          |
| RQ_CloseStream          |                   |                   | Request that a stream be closed                                          |

## Session

Clients and agents maintain an ephemeral session which consists of a session
identifier and authentication state.

Session identifiers are 4-byte unsigned integers that have the instance type and
instance flavor encoded in them.

```
 0         1         2           3
 012345678901234567890123 45678 901
[        Base CVID       | FID |IID]
```

### RQ_CoordinateConnection

Request that the receiving instance establish a new connection to the given
host. The receiver should attempt the connection as soon as possible.

| Field          | Type   | Requirements        | Description                                       |
| -------------- | ------ | ------------------- | ------------------------------------------------- |
| host           | string | An IP address       | The connection host                               |
| port           | int32  | A valid port number | The connection port                               |
| protocol       | string | `tcp` or `udp`      | The connection protocol                           |
| encryption_key | bytes  | 64 bytes            | The initial encryption key for the new connection |

### RQ_STStream

| Field     | Type            | Requirements                                 | Description |
| --------- | --------------- | -------------------------------------------- | ----------- |
| stream_id | int32           |
| oid       | string          |                                              |
| whitelist | repeated string |                                              |
| direction | string          | "upstream", "downstream", or "bidirectional" |