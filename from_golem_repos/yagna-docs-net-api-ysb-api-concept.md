# Service Bus API

This document describes the Service Bus (GSB) API and its purpose in Yagna.

## Table of contents

- [Service Bus API](#yagna-service-bus-api)
  - [Table of contents](#table-of-contents)
  - [Concepts](#concepts)
    - [Net API](#net-api)
    - [Channel](#channel)
    - [Service](#service)
    - [Service address](#service-address)
    - [Service registration](#service-registration)
    - [Service relaying](#service-relaying)
    - [GSB API module](#ysb-api-module)
  - [Authorization](#authorization)
  - [Authentication and accounting](#authentication-and-accounting)
  - [API definition](#api-definition)
    - [Protocol Buffers](#protocol-buffers)
  - [API implementation details](#api-implementation-details)
    - [Message definition](#message-definition)
    - [(De)serialization](#deserialization)
    - [Transport](#transport)
    - [Encapsulated payload serialization](#encapsulated-payload-serialization)
    - [Message routing](#message-routing)

## Concepts

### Net API

An access point to the Yagna Network. Abstracts away concepts like network addresses, transport and discovery.
Nodes are reachable _only_ by their network identifier. The underlying implementation is primarily responsible
for establishing communication channels between peers.

### Channel

A [PubSub](https://en.wikipedia.org/wiki/Publish–subscribe_pattern) pattern implemented as a network overlay.
Channels will be executed as multiplexed logical channels within the Yagna protocol. The protocol will enable multiple
different callers per registered method.

**The multiple-caller aspect is out of scope of the PoC** and will only support a single caller.

Channels are mapped to service names and created via registering a service within the Service Bus API module.

Each node is responsible for authorizing calls coming from the network. Typically, a requestor manages this kind of
service authorization, i.e. by remotely populating whitelists on providers' Yagna daemons.

### Service

Any entity that benefits from exposing its interface on the Yagna Network. Usually an external processes or a
(built-in) Yagna daemon module.

Foreseen service types:

- an API implementation provider, e.g. a third-party Market API implementation external to the Yagna daemon
- an ExeUnit instance, awaiting and responding to requestor's remote calls
- an application endpoint exposed on the Yagna Network, providing functionality beyond the scope of daemon and
SDK-provided capabilities; e.g. an FTP service

### Service address

A string that consists of node's id and the registered service name.

### Service registration

An API call to the GSB API module which binds a given service name string to the registered service process and
route any incoming messages to that service. The latter is performed by calling an appropriate interface method,
which is required to be implemented by the service.

Registration shares the lifetime of a "connection" between the GSB API module and the registered service.

### Service relaying

A state of exposing a Service interface directly on the Yagna network. The interface may be called by any third
party who posesses the knowledge of that service's address.

The prerequisite for relaying is to register a service within the GSB API module. In consequence, messages addressed
to that service will be routed to that service by the GSB API module. Responses are routed back to the caller either
as a single reply or a stream.

### GSB API module

A module within the Yagna daemon exposing the Service Bus API.

## Authorization

**Service authorization in the Yagna daemon is out of scope of the PoC version of Yagna.**

Requires a service authorization API in the Yagna daemon (not defined).

## Authentication and accounting

**Currently there are no plans to include service accounting and authentication in the PoC / MVP versions of
Yagna.**

Proposal: services can only be _authorized_ within the Yagna daemon. Accounting will be handled internally by
the Yagna daemon.

## API definition

Due to the PubSub nature of the API, two different services need to be implemented by the GSB API provider
and the registering service.

Note: authorization is not included in the scope of this proposal.

### Protocol Buffers

```protobuf
syntax = "proto3";

package GSB_API;

/* Exposed by Service Bus API implementation */
service Bus {
  /* Register a service within the bus */
  rpc Register (RegisterRequest) returns (RegisterReply);

  /* Call a local or remote service method */
  rpc ServiceCall (ServiceCallRequest) returns (CallReply);
}

/* Exposed by registering services */
service Service {
  rpc Call (CallRequest) returns (CallReply);
}

enum RegisterReplyCode {
  OK = 0;
  BAD_REQUEST = 400; // e.g. invalid name
  CONFLICT = 409;  // already registered
}

enum ServiceReplyCode {
  OK = 0;
  SERVICE_FAILURE = 500;  // e.g. service did not respond in time
}

enum ServiceReplyType {
  FULL = 0;  // a single response or end of stream
  PARTIAL = 1;  // i.e. a streaming response
}

message RegisterRequest {
  string service_id = 1;
}

message RegisterReply {
  RegisterReplyCode code = 1;
  string message = 2;  // in case of errors
}

message ServiceCallRequest {
  string address = 1;
  string request_id = 2;
  bytes data = 3;
}

message CallRequest {
  string request_id = 1;
  bytes data = 2;
}

message CallReply {
  string request_id = 1;
  ServiceReplyCode code = 2;
  ServiceReplyType type = 3;
  bytes data = 4;
}
```

## API implementation details

This section describes implementation hints and requirements for modules exposing the Service Bus API.

### Message definition

According to the `protobuf` specification, as seen in the [example above](#protocol-buffers).

### (De)serialization

Provided by `protobuf` libraries.

The binary `data` field within the messages must be serialized with [msgpack](https://msgpack.org/index.html).

### Transport

GSB API modules utilize a `nanomsg-next-gen` library with 2 transports enabled:

- IPC: unix sockets (Linux, macOS) and Named Pipes (Windows)
- TCP: all supported operating systems

Per language libraries:

- Rust: [runng](https://github.com/jeikabu/runng)
- C: [nanomsg-next-gen](https://github.com/nanomsg/nng)
- Python: [pynng](https://github.com/codypiersall/pynng)

### Encapsulated payload serialization

### Message routing

Provided by `nanomsg-next-gen` libraries.
