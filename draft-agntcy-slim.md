---
###
# Description of SLIM
###
title: "Secure Interactive Low-Latency Interactive Messaging (SLIM)"
abbrev: "agent-slim"
category: info

docname: draft-agntcy-slim-latest
submissiontype: independent
number:
date:
consensus: false
v: 3
area: Applications
workgroup: Independent Submission
keyword:
 - AI
 - Agentic AI
 - Communications
 - Realtime
venue:
  group: WG
  type: Working Group
  mail: discussion@agntcy.org
  github: agntcy/slim
  latest: https://spec.slim.agntcy.org

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

normative:
  DID-W3C:
    title: "Decentralized Identifiers (DIDs) v1.0"
    date: 2022-07-19
    author:
      - name: W3C Credentials Community Group
    target: https://www.w3.org/TR/did-core/
  NI-Registry:
    title: "Named Information Hash Algorithm Registry"
    date: 2013-08-01
    author:
      - name: IANA
    target: https://www.iana.org/assignments/named-information/named-information.xhtml
  DID-Methods:
    title: "Known DID Methods in the Decentralized Identifier Ecosystem"
    date: 2025-04-29
    author:
      - name: W3C Credentials Community Group
    target: https://www.w3.org/TR/did-extensions-methods/


informative:
  CID-Spec:
    title: "CID (Content IDentifier) Specification"
    author:
      - name: Multiformats Community
    target: https://github.com/multiformats/cid
  DID-Key:
    title: "The did:key Method v0.7: A DID Method for Static Cryptographic Keys"
    date: 2025-03-26
    author:
      - name: W3C Credentials Community Group
    target: https://w3c-ccg.github.io/did-method-key/
  DID-ATProto:
    title: "Decentralized Identifiers (DIDs) in the AT Protocol"
    author:
      - name: Bluesky/AT Protocol Community
    target: https://atproto.wiki/en/wiki/reference/identifiers/did
  DID-Web:
    title: "The did:web Method Specification"
    author:
      - name: W3C Credentials Community Group
    target: https://w3c-ccg.github.io/did-method-web/

--- abstract

This document specifies the Secure Low-Latency Interactive Real-Time Messaging
(SLIM), a protocol designed to support real-time interactive AI applications at
scale. SLIM leverages gRPC and adds publish-subscribe capabilities to enable
efficient many-to-many communication patterns between AI agentic applications
(AI models, tools and data). The protocol provides mechanisms for connection
management, stream multiplexing, and flow control while maintaining
compatibility with existing gRPC deployments.

--- middle

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Introduction


As AI systems become more sophisticated and interconnected, there is a growing need
for protocols that can support real-time interactive applications at scale.


## Protocol Overview

SLIM is designed to work as a messaging layer for applications running as
workloads in a data center, but also running in a browser or mobile device while
guaranteeing end-to-end security and low-latency communication. SLIM leverages
HTTP/2 end to end as a thin waist of the communication stack and avoids the need
to create message transcoding along the path. By leveraging message encryption
via MLS {{!RFC9420}} {{!RFC9750}}, TLS connection termination along the path
does not negatively affect
confidentiality. Authentication and authorization are handled at the application
level and can be managed in a decentralized or federated way or a mix of both.

In SLIM there are three main communication elements: intermediate nodes equipped
with message queues, message producers and message consumers.

A producer (also called a "publisher") is an endpoint that encapsulates content
in SLIM messages for transport within the SLIM message network of nodes. A
producer MUST belong to an MLS group to encrypt messages that can be decrypted
by message consumers who are members of the same group, as specified by the MLS
protocol. Once a SLIM message is encrypted, it can be published under a routable
name, which is human-readable and hierarchical. This routable channel name is
used by intermediate nodes to store and forward messages within the same
channel, allowing consumers to retrieve messages using this name.

A routable name is a name prefix that is stored in a forwarding table (FIB).
This enables requests to reach the producer and fetch a response, if one exists.

~~~
 +-------------+         +---------------------+         +-------------+
 | Producer 1  |         |                     |         | Consumer 1  |
 +-------------+         |   Messaging Node    |         +-------------+
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
 | MLS Authentication     |<-------+---+---+
 | Service                |
 +------------------------+

 Legend:
 - Producers publish to topics at the Messaging Node.
 - Consumers subscribe to topics at the Messaging Node.
 - MLS Authentication Service handles group authentication and key management.
 - Encryption group coincides with the topic identifier.
~~~
{: #fig-general-arch title="Main components of the SLIM architecture."}

Secure group members are clients as described in {{!RFC9750}} which can write
messages as producers or read messages as consumers. Most of the time, clients
are able to read and write messages in the same secure group. Clients join
secure groups as described in the MLS standard {{!RFC9750}} via an
authentication service and by exchanging messages via the delivery service. In
the SLIM architecture, the SLIM nodes constitute the infrastructure that is
responsible for delivering messages in a secure group via a logical SLIM
channel. MLS commit messages are exchanged directly using the SLIM messaging
nodes.

### Messaging Nodes

Messaging nodes are fundamental components of the SLIM architecture that serve
as specialized message queues. They fulfill several critical functions in the
messaging infrastructure. At their core, nodes efficiently route messages
between connected clients using intelligent routing algorithms while handling
the distribution and delivery of messages across the network infrastructure.

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
{: #fig-node-network title="SLIM messaging node network topology."}

The node architecture relies on two essential data structures that work in
concert. The connection table forms the foundation for tracking all active
client connections and their states, maintaining crucial metadata about each
connected client. Alongside it, the subscription table manages topic
subscriptions and implements message filtering rules, determining which messages
should be delivered to which clients.

Through this dual-table architecture, messaging nodes can effectively coordinate
message delivery while maintaining optimal system performance. The connection
and subscription mechanisms work together seamlessly to ensure reliable message
routing, proper client tracking, and efficient subscription management across
the distributed system. Each node operates autonomously while participating in
the broader network, creating a resilient and scalable messaging infrastructure.



#### Connection Table

The connection table serves as a fundamental data structure within the
SLIM messaging node architecture, maintaining a comprehensive registry of both
client-to-node and node-to-node connections. Each entry in the table contains
essential metadata about connected endpoints, including their unique
identifiers, connection timestamps, authentication status, and current state
information.

For client connections, the table tracks end-user applications that connect to
receive or send messages through the system. For node connections, it maintains
the network fabric topology by recording inter-node relationships and routing
paths. This dual-purpose nature enables SLIM to manage both the edge
connectivity with clients and the internal communication infrastructure between
nodes.

Connection states are dynamically tracked and updated to reflect the real-time
status of each endpoint. This includes monitoring whether clients or nodes are
actively connected, temporarily disconnected, or in various intermediate states.
The table maintains crucial session information such as endpoint capabilities,
protocol versions, and quality of service parameters that influence message
handling.

By maintaining this detailed connection state, the table enables efficient
routing decisions across the entire network fabric. It provides each messaging
node with immediate access to both client and node status information, allowing
for rapid determination of message delivery paths and handling of
connection-related events. The connection table also plays a vital role in
system reliability by tracking connection health and enabling quick detection of
disconnections or network issues at both the client and node levels.

A connection table maps location-independent channel names to connections to
remote nodes. The mapping is used to forward messages towards nodes that can
either route messages or consume them in case consumers are directly connected
to the node.

Channel names are encoded as human-readable hierarchical names for efficient
table lookup operations.


#### Subscription Table and Matching

The subscription table is used to map channel subscriptions to neighboring
nodes. It manages the distribution of messages based on subscriptions and
ensures efficient delivery of messages.
A message carries the data to be delivered as well as the channel name and the
address locator of the message producer.

The control plane manages the configuration and updates of the connection and
subscription tables.

~~~
+-------------------------------------------------------------+
|                    SLIM Message Structure                   |
+-------------------------------------------------------------+
| Channel Name   | Address Locator   |      Data Payload       |
+-------------------------------------------------------------+
| "/foo/bar"     | 192.0.2.10:12345 |   { ... application ... |
|                |                  |         data ... }      |
+-------------------------------------------------------------+

Legend:
- Channel Name: Identifies the logical channel/topic for routing.
- Address Locator: Specifies the producer's network address.
- Data Payload: Contains the actual message content.
~~~
{: #fig-message-structure title="SLIM message structure carrying channel name,
address locator, and data."}

### Control Plane

The control plane is responsible for the management and orchestration of
SLIM messaging nodes and their interconnections. It handles the configuration,
provisioning, and monitoring of nodes, ensuring that the messaging
infrastructure operates smoothly and efficiently.

Key functions of the control plane include:

- **Node Discovery and Registration**: New messaging nodes discover
  each other and register their presence with the control plane. This
  enables the control plane to maintain an up-to-date view of the
  messaging infrastructure.

- **Configuration Management**: The control plane distributes
  configuration updates to messaging nodes, including connection and
  subscription table updates. This ensures consistent and correct
  routing behavior across the node network.

- **Monitoring and Analytics**: The control plane collects and
  analyzes telemetry data from messaging nodes, providing insights
  into system performance, message flow, and potential issues.

- **Fault Detection and Recovery**: In case of node failures or
  network issues, the control plane detects faults and initiates
  recovery procedures, such as rerouting messages or reallocating
  resources.

- **Security and Access Control**: The control plane manages
  security policies, authentication, and authorization of nodes and
  clients, ensuring a secure messaging environment.


By centralizing these management functions, the control plane enhances
the overall reliability, security, and performance of the SLIM messaging
infrastructure. It enables efficient scaling, dynamic reconfiguration,
and proactive maintenance of the node network.

### Session Layer

Clients connect to messaging nodes via a session layer.


## Naming Considerations

SLIM requires several types of identifiers, including channel names, client
names, and client locators.
A channel name identifies a messaging group and must be routable; that is, it
must include a globally unique network prefix that can be aggregated for
scalable lookups and message forwarding.
A group in SLIM is an MLS group with a moderator client responsible for adding
and removing group members. The moderator is identified by a cryptographic
public key as defined in MLS {{!RFC9750}}, and in SLIM, also by a decentralized
identifier derived as the hash of the public key {{DID-W3C}}.
By naming entities with hashes {{!RFC6920}}, SLIM achieves secure and globally
unique naming, enabling the creation of permissionless systems where channel
names and client names can be distributed across administrative boundaries. W3C
DIDs are optional but can be used when hash links are employed and conform to
the Named Information {{!RFC6920}} standard, referencing the IANA registry
{{NI-Registry}}.

SLIM routable name prefixes and client names can use different did methods which
will have different resolution systems such as did:web {{DID-Web}}, did:key
{{DID-Key}} and did:plc {{DID>ATProto}}, see {{DID-Methods}} for well known did
methods.




## Deployment Considerations

# Security Considerations

Security is a paramount concern for SLIM, given the sensitive nature of
the data being transmitted and the need for reliable access control.
SLIM inherits security features from MLS, gRPC, and TLS, but also
introduces new mechanisms to address its unique requirements.

## Authentication and Authorization

Authentication and authorization in SLIM are handled at the application
level, leveraging the capabilities of the underlying MLS groups. Clients
must authenticate themselves to the MLS Authentication Service, which
issues credentials that are used to sign messages. These credentials
are then used by other clients to verify the authenticity of the
messages and the identity of the sender.

Authorization policies determine what actions an authenticated client
is allowed to perform, such as publishing or subscribing to specific
channels. These policies are enforced by the messaging nodes, which
check the client's credentials and the requested operation against the
configured policies.

## Confidentiality and Integrity

Confidentiality and integrity of messages are ensured through end-to-end
encryption using MLS. Messages are encrypted by the producer before
being sent and can only be decrypted by consumers that are members of
the same MLS group. This ensures that even if messages are intercepted
in transit, they cannot be read or tampered with by unauthorized
parties.

