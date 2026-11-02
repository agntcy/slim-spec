---
title: "Secure Low-Latency Interactive Messaging (SLIM)"
abbrev: "agent-slim"
category: info

docname: draft-mpsb-agntcy-slim-03
submissiontype: independent
number:
date: 2026-11-02
consensus: false
v: 3
# area:
# workgroup: Independent Submission
keyword:
 - AI
 - Agentic AI
 - Communications
 - Realtime

author:

-
    fullname: Luca Muscariello
    organization: Cisco
    email: lumuscar@cisco.com
-
    fullname: Michele Papalini
    organization: Cisco
    email: micpapal@cisco.com
-
    fullname: Mauro Sardara
    organization: Cisco
    email: msardara@cisco.com

-
    fullname: Sam Betts
    organization: Cisco
    email: sambetts@cisco.com

informative:
  SLIMRPC:
    title: "SLIMRPC Reference"
    author:
      - name: AGNTCY
    target: https://docs.agntcy.org/slim/slim-rpc/
  DID-Key:
    title: "The did:key Method v0.7: A DID Method for Static Cryptographic Keys"
    date: 2025-03-26
    author:
      - name: W3C Credentials Community Group
    target: https://w3c-ccg.github.io/did-method-key/
--- abstract

This document specifies the Secure Low-Latency Interactive Messaging
(SLIM), a protocol designed to support real-time interactive AI applications at
scale. SLIM provides a name-routed messaging data plane and session layer for
agent protocols such as A2A and MCP. The data plane carries Protocol Buffers
messages over streaming gRPC on HTTP/2 or WebSocket, while the session layer
provides point-to-point and group communication, configurable reliable delivery,
and optional end-to-end encryption using MLS. SLIM also defines link negotiation
between routing nodes and control messages for session establishment and group
membership.

--- middle

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Introduction

## Summary

The Secure Low-Latency Interactive Messaging (SLIM) protocol addresses the
unique communication requirements of modern AI agentic applications by providing
a secure, scalable, and efficient messaging infrastructure. SLIM combines the
reliability and performance of gRPC with secure messaging and group
communication, creating a comprehensive solution for interactive AI application
communication.

At its core, SLIM consists of three primary components: data plane routing
nodes that forward messages based on metadata, session-layer clients that manage
secure group state and reliability, and application endpoints that publish and
receive encrypted content. The protocol leverages Message Layer Security (MLS)
for end-to-end encryption, ensuring that messages remain confidential even when
passing through intermediate nodes or experiencing TLS termination along the
communication path.

The architecture is built around a distributed network of routing nodes, each
maintaining connection and subscription state to route messages by hierarchical
name. An optional, separate control plane configures and manages routing nodes;
it is not in the message path. The session layer runs at application endpoints
and manages session state, reliability, and group membership.

SLIM employs hierarchical names of the form
`organization/namespace/service/id`. The first three components support
aggregation, anycast discovery, and routing; the final identifier distinguishes
an application instance. The identifier is derived from the application's
authenticated identity for routing and identity binding. DIDs may be used as
application identities, but are not required as the wire representation of a
SLIM name.

The protocol includes a session layer that abstracts messaging and, when
enabled, MLS operations from applications. It provides point-to-point and group
sessions. SLIMRPC builds request/response and streaming RPC over SLIM sessions
(see {{SLIMRPC}}); it is an application-layer facility rather than a data-plane
message type.

SLIM supports application authentication and authorization through configured
identity providers and session policies. MLS-based group encryption and
identity binding are optional capabilities; the security properties provided
depend on deployment configuration.

# Protocol Overview

guaranteeing end-to-end security and low-latency communication.
SLIM is designed to work as a messaging layer for applications running as
workloads in a data center, a browser, or a mobile device.

SLIM carries Protocol Buffers messages over streaming gRPC using HTTP/2, or over
WebSocket. The transport is hop-by-hop and may be protected with TLS;
transport-layer encryption is independent of session-layer security. When
enabled for a session, MLS {{!RFC9420}} {{!RFC9750}} encrypts application content
at the endpoints. Routing nodes can then forward the encrypted content without
learning its plaintext. MLS is optional; deployments that do not enable it rely
on the configured transport security.

SLIM routing nodes negotiate each inter-node link before processing routed
messages. Link negotiation exchanges a link identifier, SLIM version, node
identity, and connection type. Peers can derive a per-link key for header
integrity; deployments can also negotiate hybrid post-quantum key material.

Authentication and authorization are handled at the application level and can be
managed in a decentralized or federated way or a mix of both.

In SLIM there are three main communication elements: routing nodes (data plane),
session-layer clients, and application endpoints that publish and receive
messages.

A producer (also called a "publisher") is an endpoint that encapsulates content
in SLIM messages for transport within the network of routing nodes. A session
may use MLS to encrypt application content for its members. The publisher sends
messages to a routable hierarchical name; routing nodes use name prefixes and
subscription state to select forwarding connections without inspecting
application content.

A routable name is a name prefix that is stored in a forwarding table (FIB).
This enables requests to reach the producer and fetch a response, if one exists.

~~~
 +-------------+         +---------------------+         +-------------+
 | Producer 1  |         |                     |         | Consumer 1  |
 +-------------+         |    Routing Node     |         +-------------+
                         |                     |<------->| Consumer 2  |
 +-------------+         |                     |         +-------------+
 | Producer 2  |-------->|                     |<------->| Consumer 3  |
 +-------------+         +---------------------+         +-------------+

          |                        ^   ^   ^
          |                        |   |   |
          |                        |   |   |
          |                        |   |   |
          v                        |   |   |
 +------------------------+        |   |   |
 | Group Moderator        |<-------+---+---+
 | (application or        |
 | channel manager)       |
 +------------------------+

 Legend:
 - Producers publish to topics via routing nodes.
 - Consumers subscribe to topics via routing nodes.
- The moderator manages group membership and distributes MLS updates
  when MLS is enabled.
- Group data and control messages use SLIM channels.
~~~
{: #fig-general-arch title="Main components of the SLIM architecture."}

authentication service and by exchanging messages via the delivery service. In
the SLIM architecture, the SLIM nodes constitute the data-plane infrastructure
Group members are application clients that can send and receive messages on a
group channel. A moderator invites and removes members. When MLS is enabled,
the session layer exchanges MLS updates over SLIM channels; routing nodes
provide message delivery and do not act as an MLS authentication service.

## Routing Nodes (Data Plane)

Routing nodes are fundamental components of the SLIM architecture that forward
messages using metadata without inspecting payloads. They fulfill several
critical functions in the messaging infrastructure. At their core, nodes
efficiently route messages between connected clients while handling the
distribution and delivery of messages across the network infrastructure.

~~~
    Producer A              Producer B              Producer C
        |                       |                       |
        v                       v                       v
   +----------+            +----------+            +----------+
   |  Node 1  |<---------->|  Node 2  |<---------->|  Node 3  |
   +----------+            +----------+            +----------+
        ^                       ^                       ^
        |                       |                       |
        v                       v                       v
   +----------+            +----------+            +----------+
   |  Node 4  |<---------->|  Node 5  |<---------->|  Node 6  |
   +----------+            +----------+            +----------+
        ^                       ^                       ^
        |                       |                       |
    Consumer X             Consumer Y             Consumer Z

Legend:
- Each Node maintains connection and subscription tables
- Bidirectional arrows represent inter-node communication paths
- Producers and Consumers connect to their local nodes
- Messages are routed through the node network based on subscriptions
~~~
{: #fig-node-network title="SLIM routing node network topology."}

The node architecture relies on two essential data structures that work in
concert. The connection table forms the foundation for tracking all active
client connections and their states, maintaining crucial metadata about each
connected client. Alongside it, the subscription table manages topic
subscriptions and implements message filtering rules, determining which messages
should be delivered to which clients.

Through this dual-table architecture, routing nodes can effectively coordinate
message delivery while maintaining optimal system performance. The connection
and subscription mechanisms work together seamlessly to ensure reliable message
routing, proper client tracking, and efficient subscription management across
the distributed system. Each node operates autonomously while participating in
the broader network, creating a resilient and scalable messaging infrastructure.



### Connection Table

The connection table tracks client-to-node and node-to-node connections and
provides the connection references used for message forwarding.

For client connections, the table tracks end-user applications that connect to
receive or send messages through the system. For node connections, it maintains
the network fabric topology by recording inter-node relationships and routing
paths. This dual-purpose nature enables SLIM to manage both the edge
connectivity with clients and the internal communication infrastructure between
nodes.

disconnections or network issues at both the client and node levels.
Connection state is updated as connections are established and closed. Link
negotiation identifies inter-node connections before they are used for routed
traffic.

A connection table maps location-independent channel names to connections to
remote nodes. The mapping is used to forward messages towards nodes that can
either route messages or consume them in case consumers are directly connected
to the node.

Channel names are encoded as human-readable hierarchical names for efficient
table lookup operations.


### Subscription Table and Matching

The subscription table is used to map channel subscriptions to neighboring
nodes. It manages the distribution of messages based on subscriptions and
ensures efficient delivery of messages. A message carries a SLIM header and
content. The header includes source and destination names, identity and version
information, a time-to-live (TTL), and forwarding metadata. Session messages
additionally carry a session type, session-message type, session identifier,
and message identifier. Content is either an application payload or a
session/control payload. Subscription and unsubscription requests can request a
direct acknowledgement by supplying a non-zero subscription identifier.

Routing state can be configured locally or distributed by a control plane.

~~~
+-------------------------------------------------------------+
|                    SLIM Message Structure                   |
+-------------------------------------------------------------+
| SLIM Header    | Session Header    |         Content         |
+-------------------------------------------------------------+
| names, TTL,    | type, session ID, | application or control  |
| forwarding info| message type, ID  | payload                 |
+-------------------------------------------------------------+

Legend:
- SLIM Header: Identifies endpoints and carries routing and integrity
  fields.
- Session Header: Present for session-layer messages.
- Content: Application data or a session/control payload.
~~~
{: #fig-message-structure title="SLIM message structure carrying channel name,
forwarding metadata, session information, and content."}

## Control Plane

The control plane is a separate management component and is not required in the
message path. A deployment can configure routing nodes directly or use a
control plane to register nodes, manage topology and routes, distribute
configuration, and monitor operation. Control-plane topology may be
configuration-managed or API-managed.

When a control plane is used, node registration can be restricted to nodes
authorized for the claimed routing domain. Deployments can also partition
topology into routing domains to isolate routes.

## Session Layer

The session layer serves as a critical abstraction component that bridges
application frameworks with the underlying SLIM messaging infrastructure. It
provides a unified interface that simplifies the complexity of secure messaging
while handling the details of MLS client operations and message distribution.

### Core Responsibilities

The session layer encapsulates several key functionalities:

**MLS Client Operations**: When MLS is enabled for a session, the layer handles
MLS encryption and decryption, key management, and group membership operations.

**Channel Management**: It provides seamless channel subscription and
unsubscription capabilities, managing the lifecycle of channel memberships and
maintaining subscription state across connection interruptions or node
failures.

**Message Abstraction**: The session layer abstracts message passing between
applications and the SLIM message distribution network, handling message
formatting, routing, and delivery confirmation while providing simple send and
receive primitives to applications.

**Configuration Abstraction**: It eliminates the need for applications to
manage complex configuration details required to connect to SLIM nodes,
automatically handling node discovery, connection establishment, and
subscription management.

### API Design Principles

The session layer API is designed with the following principles:

- Simplicity: Applications interact with the messaging system through
  intuitive publish/subscribe operations without needing to understand the
  underlying MLS or routing complexities.

- Asynchronous Operations: All messaging operations are designed to be
  non-blocking, supporting high-performance applications.

- Framework Agnostic: The API provides language bindings and framework
  adapters for various application development environments, ensuring broad
  compatibility across different technology stacks.

- Error Handling: Comprehensive error reporting and recovery mechanisms
  help applications handle communication issues and authentication failures.

### Session Management

The session layer manages state for point-to-point and group communication.
Group sessions use distinct data and control channel names. The session layer
handles discovery, join, leave, group updates, acknowledgements, and (when MLS
is enabled) MLS messages over these channels. Delivery retries and
acknowledgements are configurable; they are not guaranteed for every session.

Participants can transition between online and offline states without being
removed from a group. Participants periodically send one-way heartbeats, and
each member independently tracks peer liveness. A returning participant can
rejoin; when MLS is enabled, key material is updated as needed to reconcile an
epoch mismatch. Session-state persistence is optional and can restore group
state after a process restart.

## Naming Considerations

the Named Information {{!RFC6920}} standard, referencing the IANA registry
tables, which can aggregate multiple names under name prefixes such as
SLIM names identify applications and channels and are used for routing and
service discovery. A name has the form:

```
organization/namespace/service/id
```

The first three components are application-selected and support aggregation.
The final component identifies an application instance and is derived from its
authenticated identity. For example, an identity may be a `did:key` as defined
in {{DID-Key}}. A name ID is a routing identifier, not proof of identity; when
identity binding is enabled, the session layer verifies the peer identity and
checks its binding to the advertised name.

The three-component prefix `organization/namespace/service` addresses a service
for anycast discovery. Adding the final ID addresses an instance. A group
channel uses the same four-component form. Its data and control channel IDs are
derived from the moderator identity and channel name; they share the same first
three components. The moderator is the group participant authorized to invite
and remove members.


## Deployment Considerations
SLIM helps the deployment of agentic AI applications where a combination of
data streams, tools and LLMs are combined to create a distributed multi-agent
systems that can solve problems via AI.

These applications work as SLIM clients and MAY expose a service to the secure
group. The channel and client naming structure combined allows for service
discovery capabilities by binding the application a specific application
namespace and service name.

# RPC in Agentic Protocols and Relationship to Messaging

Most agent-facing interfaces in use today—such as A2A and the Model Context
Protocol (MCP)—are Remote Procedure Call (RPC) oriented. They expose synchronous
request/response semantics for tool invocation, resource listing, and capability
execution. This section clarifies how RPC relates to asynchronous messaging and
how the two paradigms interoperate in agentic systems.

## RPC vs. Messaging: Synchronous vs. Asynchronous

- **RPC (A2A, MCP)**: The caller issues a request and blocks or awaits a timely
response. Semantics emphasize tightly scoped operations (for example, “call tool
X with parameters Y”) with bounded latency and explicit error contracts.
- **Messaging (AMQP, MQTT, NATS, Kafka, SLIM)**: Decoupled producers and
consumers communicate via topics, subjects, or queues. Delivery can be
one-to-one, one-to-many, or many-to-many, with loose coupling, buffering, and
retries. Producers are not inherently blocked by consumers.

In practice, agentic applications need both: synchronous tool invocations for
interactivity and asynchronous channels for streaming output, progress,
coordination, and fan-out/fan-in patterns.

## Challenges of RPC over Messaging

Bridging synchronous RPC semantics with asynchronous messaging infrastructure
introduces several challenges:

- **Request/Response Correlation**: Messaging systems are inherently decoupled
and do not guarantee a direct response path. Implementing RPC requires
correlating requests and responses, often using unique identifiers and temporary
reply channels.
- **Latency and Ordering**: Messaging layers may introduce variable delivery
latency and do not guarantee strict ordering, which can complicate synchronous
RPC expectations.
- **Error Handling**: Messaging systems may buffer, retry, or drop messages,
making it difficult to propagate errors and timeouts in a way that matches RPC
contracts.
- **Streaming and Multiplexing**: Supporting streaming RPC (e.g., bidirectional
or server/client streaming) over messaging requires careful management of stream
lifecycles, backpressure, and multiplexing multiple logical RPCs over shared
channels.
- **Security and Authorization**: Ensuring that only authorized parties can
invoke or respond to RPCs, and that all messages are authenticated and
encrypted, is more complex in a distributed, group-based messaging environment.

SLIMRPC addresses these challenges with an RPC abstraction over the SLIM
session layer, as described in the following section.

## Remote Procedure Calls over SLIM (SLIMRPC)

SLIMRPC is an application-layer RPC facility that uses SLIM sessions as its
transport. It supports the four common RPC patterns: unary, client-streaming,
server-streaming, and bidirectional streaming.

### Overview

Applications define services using Protocol Buffers and invoke their methods
through generated client and server stubs. SLIMRPC security and delivery
properties are those of the underlying SLIM session and its configuration.

### Key Features

- **Streaming RPC Support**: SLIMRPC supports all four RPC patterns, allowing
for flexible data exchange patterns such as real-time data feeds, interactive
sessions, and collaborative workflows.
- **Session Reuse and Multiplexing**: Concurrent RPC calls share a persistent
SLIM session. Each call has a unique RPC ID, allowing the server to dispatch
new calls and route subsequent stream messages to the right handler.
- **Multicast RPC (mRPC)**: A client constructs a group channel with the names
of the servers to call. On the first RPC call, the SLIMRPC channel lazily
creates a multicast session and invites those servers through the SLIM session
layer. The client application acts as the group moderator. The RPC request is
sent after session establishment; invitations are not part of each RPC call.
The request is delivered to the group, and responses are returned with the
source member identified. The response stream completes when all members have
finished; a member error does not prevent other members' responses.
- **Protocol Buffers Integration**: Service definitions and message schemas are
specified using Protocol Buffers, enabling strong typing and interoperability
across languages and platforms.

### Architecture

SLIMRPC libraries and generated stubs manage RPC stream lifecycles over SLIM
sessions. Service and method names, the RPC ID, and call direction are carried
as message metadata; the server subscribes to its base SLIM name rather than a
distinct routed name for each method.

### Benefits for Agentic Applications

By providing streaming RPC capabilities, SLIMRPC enables agentic AI applications
and distributed systems to:

- Orchestrate complex, multi-step workflows across multiple agents and services.
- Exchange large or continuous data streams efficiently and securely.
- Implement interactive, real-time collaboration between distributed components.
- Simplify integration with existing gRPC-based tools and ecosystems.

SLIMRPC makes it possible to build robust, scalable, and secure agentic
applications that fully leverage the power of SLIM's messaging and group
communication model.

### SLIMRPC Method Dispatch

The server subscribes to its base SLIM name. Each initial RPC message carries
the service name, method name, and a unique RPC ID in metadata. Subsequent
messages in a streaming call carry the RPC ID, which the server uses to dispatch
them to the active handler. Multiple calls therefore share a session without
encoding method-specific names in the SLIM routing name. For multicast calls,
the channel targets a group and responses can be received from multiple
members.

# Security Considerations

SLIM's security properties depend on deployment configuration. TLS protects a
transport hop when enabled. MLS {{!RFC9420}} {{!RFC9750}}, when enabled for a
session, provides end-to-end protection for application content; intermediate
routing nodes then see only routing metadata and ciphertext. Without MLS,
application content is not protected from routing nodes beyond any transport
encryption in use.

Link negotiation is required before an inter-node connection processes routed
messages. When peers negotiate header integrity, implementations MUST verify
the integrity field before forwarding or accepting a protected message.
Implementations MUST reject replayed session control messages when replay
protection is enabled. MLS implementations MUST validate KeyPackages and apply
epoch transitions according to {{!RFC9420}}; removed members MUST NOT be able to
decrypt messages sent after their removal.

When identity binding is enabled, implementations MUST bind a peer's advertised
name to its verified identity. A name ID alone is not an authentication
credential. Implementations that persist session state MUST protect the stored
MLS state and associated credentials against unauthorized access.

# IANA Considerations

This document has no IANA actions.

