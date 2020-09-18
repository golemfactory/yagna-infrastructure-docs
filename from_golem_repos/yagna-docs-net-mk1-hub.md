# Table of contents

- [Table of contents](#table-of-contents)
  - [Description](#description)
  - [Architecture](#architecture)
  - [Message Format](#message-format)
    - [Message Components](#message-components)
    - [Message Destination](#message-destination)
      - [Prefix](#prefix)
      - [Address](#address)
        - [Node Address](#node-address)
        - [Broadcast Address](#broadcast-address)
      - [Destination Module and Function](#destination-module-and-function)
      - [Example Destinations](#example-destinations)
    - [Payload](#payload)
    - [Reply To](#reply-to)
    - [Request ID](#request-id)
    - [Message Type](#message-type)
  - [Message Handling](#message-handling)
    - [Requests](#requests)
    - [Responses and Errors](#responses-and-errors)
  - [Hub](#hub)
    - [Technology stack](#technology-stack)
    - [Specification](#specification)

## Description

YagnaNet is a module that is responsible for network communication and discovery between Yagna nodes.
The first implementation called YagnaNet Mk1 uses a centralized server to allow communication between nodes.

## Architecture

YagnaNet module receives messages from two sources:

- Service Bus (GSB)
- Other nodes in the network

After receiving a message, YagnaNet module may put it on the Service Bus if it is addressed to the current node,
send it to the centralized server which forwards it to the YagnaNet module of the destination node (YagnaNet Mk1) or
use P2P network to send it to the destination node (YagnaNet Mk2).

## Message Format

### Message Components

Every message contains following parts:

| Message Part Name | Example |
|--|--|
| [Destination](#message-destination) | /net/0x123/market-api/get-offers |
| [Payload](#payload) | { "max-offers": 50 } |
| [Reply To](#reply-to) | 0x789 |
| [Request ID](#request-id) | 1574244629 |
| [Message Type](#message-type) | REQUEST |

### Message Destination

#### Prefix

Messages addressed to YagnaNet module must start with `/net/`.

#### Address

The messages could be sent to a given node address or to a broadcast address.

##### Node Address

A Yagna node address, e.g. `0x123...`.
A message with an address like this should be delivered to the given node.
In the YagnaNet Mk1 implementation it is sent to the centralized server, which sends it directly to the destination
node. The YagnaNet Mk2 implementation should allow P2P messages without a centralized server; sometimes the messages
will need to travel between many nodes before reaching the destination node.

##### Broadcast Address

When a messages is sent to a broadcast address, it can be delivered to more than one node in the network.
The broadcast address is a string `broadcast` (which means all nodes in the network)
optionally followed by a `:N` (which means all nodes that are not further than N hops
from the originating node - this can be used to send broadcast messages only to a part of the network).

#### Destination Module and Function

The next part of the message should be the destination module name followed by the method name, e.g. 
`market-api/get-offers`.

#### Example Destinations

| Destination | Description |
|--|--|
| /net/0x123/market-api/get-offers | Get offers from the Market API module on node 0x123. |
| /net/broadcast:5/payment/get-payment-method | Get payment methods from nodes that are not further than N hops from the originating node. |

### Payload

The payload depends on the destination module and method. It could contain method parameters encoded in JSON
(for example: `[ "method-name": "param" ]`) or binary data. The YagnaNet module does not check payload content format.

### Reply To

Specifies node address that send this message. It is automatically added by YagnaNet module so that the reply could
be sent to the originating node.

### Request ID

Request ID is necessary to pair a request with response. It should be a unguessable random number.

### Message Type

| Message Type | Description |
|--|--|
| Request | Remote method call |
| Response | Reply to a remote method call |
| Error | Error message |

## Message Handling

### Requests

When YagnaNet receives a network message prefixed with `/net/NODE_ID/`, where NODE_ID is the current node identifier,
the message is put on the Service Bus without the `/net/NODE_ID/` prefix, so that modules subscribed to this type
of message receive it.

If the message is prefixed with `/net/NODE_ID/`, where NODE_ID is different from the current node identifier,
the message (in YagnaNet Mk1 version) is forwarded to the centralized server (hub) which relays it to the destination
node.

### Responses and Errors

TODO

## Hub

An HTTP(S) server and a centralized predecessor of the MK2 P2P network. Provides means to exchange messages between
network peers via HTTP requests.

Functionality provided by the hub:

- unicasting messages
- broadcasting messages with a TTL
- polling for messages
- peer authentication
- authorization of requests

Note: can be implemented as a WSS server.

### Technology stack

The programming language used in this project will be [Rust](https://www.rust-lang.org/). The newest stable version of
Rust compiler (rustc) should compile all source code without errors.

For the HTTP (/ WS) server code, [Actix Web 1.0](https://actix.rs) will be used.

### Specification

[OpenAPI specification](net-mk1-hub-openapi.yaml)
