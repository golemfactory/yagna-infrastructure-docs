# Golem net

## Abstract 

This document describes the high-level architecture of the proposed **golem-net** module, its feature set, responsibilities and interop with **golem-core** \(via the **net API**\).

## Overview 

* 1. **golem-net** is a modular networking stack library, which implements and exposes the **net API**,
  2. layers of the networking stack do not correspond to the OSI model directly and may span multiple layers of that model,
  3. each layer of the networking stack is abstracted and independent from the previous layer,
  4. layers may consist of multiple components serving same domain functions,
  5. components within a layer implement a common interface \(trait\), which prevents coupling by design,
  6. components within a layer are independent of each other and replaceable,
  7. the library exposes a negotiation mechanism for choosing the corresponding module within a layer \(i.e. protocol upgrades\),
  8. application protocols implemented with **golem-net** are agnostic of connection establishment, transport and routing,
  9. the **golem-net** library provides at least one component implementation for each layer of the stack,
  10. **golem-net** additionally provides at least one IPC integration per supported platform,
  11. the library provides an optional Python integration layer built on top of **golem-net**,
  12. **golem-net** can be compiled and utilized on modern Linux, macOS and Windows operating systems,
  13. the library is implemented in Rust. 

## Components 

* 1. OSI layer 1-4\(+\) protocols:
     * hardware agnostic,
     * link layer protocol agnostic,
     * network protocols:
       1. IPv4 \(**must have**\),
       2. IPv6 \(**must have**\),
       3. SDP \(**could have**\),
     * reliable transport protocols,
       1. TCP \(**must have**\),
       2. uTP on UDP \(**should have**\),
       3. WebSockets \(**should have**\) upgrade on TCP,
  2. NAT traversal:
     * message relaying,
     * automatic port forwarding via UPnP,
     * provides the means to integrate more specialized NAT traversal mechanisms,
  3. cryptography:
     * message signing and encryption with ECIES \(ephemeral keys\),
  4. multiplexing:
     * application protocols per connection \(**must have**\),
     * listening addresses, network, and transport protocols per node \(**should have**\),
  5. routing:
     * Kademlia-based \(**must have**\),
     * relay-based \(**must have**\),
     * provides the means to integrate custom routing algorithms,
  6. discovery:
     * node discovery, e.g. Kademlia \(**must have\)**,
     * service discovery \(**should have**\),
     * resource discovery \(**could have**\)**,**
  7. protocols:
     * base \(id and protocol negotiation\) \(**must have**\),
     * DHT protocol \(e.g. Kademlia\) \(**must have**\),
     * resource transfer \(**should have**\),
     * VPN \(**could have**\),
  8. flow control / QoS mechanisms for multiplexed streams. 

## Message format 

Messages exchanged by the built-in protocols the library are \(de\)serialized with Protobuf. This does not impose the serialization method for higher-level \(i.e. application\) protocols, however this method of serialization is highly recommended.

**Rust message structs cannot implement application logic \(i.e. serialization, encryption, define application constants\)**.

## net API

This section describes the public programming interface exposed by the **golem-net** library.

| **Name** | **Arguments** | **Return value** | **Description** |
| :--- | :--- | :--- | :--- |
| configure | NetworkOptions | Result&lt;\(\), Error&gt; | Change p2p configuration |
| listen | Address | FutureResult&lt;\(\), Error&gt; | Listen on Address |
| dial | Address | FutureResult &lt;impl ChannelTrait, Error&gt; | Connect to Address. Can be a multicast group |
| unicast | Message, Address | FutureResult&lt;\(\), Error&gt; | Sends a message in a “fire and forget” fashion |
| broadcast | Message | FutureResult&lt;\(\), Error&gt; | Broadcast a message |
| close | \(\) | FutureResult&lt;\(\), Error&gt; | Terminate |

### Address ****

**net API** uses a common addressing mechanism \(e.g. IPFS multiaddr\), consisting of:  


* * * networking protocol and connection details \(e.g. an IPv4 address\)
    * transport protocol and connection details \(e.g. port within an established TCP session\)
    * application protocol and connection details \(e.g. HTTP and resource URI\)
    * optional relay node information

  
In particular, the address might only include the application protocol + ID and rely solely on node / service discovery.

### NetworkOptions

* * * listening addresses
    * transport protocols
    * relevant custom intervals
    * TBD

### ChannelTrait 

| **Name** | **Arguments** | **Return value** | **Description** |
| :--- | :--- | :--- | :--- |
| send | Message | FutureResult&lt;\(\), Error&gt; | Send a message |
| close | \(\) | FutureResult&lt;\(\), Error&gt; | Close the channel |
| state | \(\) | FutureResult &lt;impl ChannelStateTrait, Error&gt; | Retrieve channel state info |
| metrics | \(\) | FutureResult &lt;impl ChannelMetricsTrait, Error&gt; | Retrieve channel metrics |

  
 Implemented by Channel and MulticastChannel structs.

### MulticastChannel \(impl ChannelTrait\) methods

| **Name** | **Arguments** | **Return value** | **Description** |
| :--- | :--- | :--- | :--- |
| unicast | Message, Address | FutureResult&lt;\(\), Error&gt; | Send a message to node |

### ChannelState

* * * GUID
    * Node structure:
      1. ID
      2. List of open channels
      3. Means of discovery
      4. Routing details

  1. ChannelMetrics
     * TBD: per connection \(transport\) metrics
     * TBD: summarized connection metrics for node 

## High-level architecture

External logic communicates with **golem-net** via an asynchronous, event-based system. The library allows for interfacing multiple, various inter-thread and inter-process communication mechanisms and allows them to operate simultaneously.  


* 1. IPC interface \(**net API**\)

  
**golem-net** provides an IPC layer supporting macOS, Linux and Windows platforms. The default implementation is based on Unix sockets and named pipes in case of Windows.  


| net evt publisher → | → IPC tx | → | → IPC rx | → poll evt |
| :--- | :--- | :--- | :--- | :--- |
| net control layer ← | ← IPC rx | ← | ← IPC tx | ← submit evt |
| ↑↓ |  |  |  | ↑↓ |
| Channel |  |  |  |  |
| Networking stack | External logic |  |  |  |
| Networking process | External process |  |  |  |

* 1. Rust native interface \(**net API**\)

     Networking and the external logic communicate through MPSC \(Multi-Producer, Single Consumer\) queues.

| net evt publisher → | → net API MPSC tx | → | → net API MPSC rx | → poll evt |
| :--- | :--- | :--- | :--- | :--- |
| net control layer ← | ← net API MPSC rx | ← | ← net API MPSC tx | ← call evt |
| ↑↓ |  |  |  | ↑↓ |
| Channel | External logic |  |  |  |
| Networking stack |  |  |  |  |
| Networking thread |  |  |  |  |

* 1. Rust Python bindings

Builds on Rust’s native interface, placing external logic in a separate thread. That thread may block \(i.e. lock the GIL\) without interrupting the networking thread.  


| net evt publisher → | → MPSC net tx | → | → MPSC net rx | → poll evt |
| :--- | :--- | :--- | :--- | :--- |
| net control layer ← | ← MPSC py rx | ← | ← MPSC py tx | ← submit evt |
| ↑↓ |  |  |  | ↑↓ |
| Channel | Session |  |  |  |
| Networking stack | Python bindings \(external logic\) |  |  |  |
| Networking thread | Python integration thread |  |  |  |

Integration with **golem-core** may be done as a full reimplementation of a networking stack or a basic one, passing binary blobs from and to the session logic written in Python.  


* * * Re-implementation
      1. full network stack
      2. includes session management
      3. calls services written in Python \(i.e. TaskServer, TaskManager, etc.\)
      4. requires for Python services to have a more abstract interface 
    * Basic integration
      1. full network stack
      2. passes binary blobs to and from session management code written in Python
      3. requires an additional integration layer between Channels and sessions in Python
      4. requires changes to connection management in Python \(e.g. dropping connections in PeerSession and TaskSession\)

## Testing

