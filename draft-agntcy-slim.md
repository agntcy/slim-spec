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


informative:


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
ValidateThe connection table serves as a fundamental data structure within the
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
disconnections or network issues at both the client and node levels. `


#### Subscription Table

The subscription table is used to map topic subscriptions to neighboring nodes.
It manages the distribution of messages based on topic subscriptions and ensures
efficient routing of pub/sub messages. The subscription table entries include:

* Topic: The name of the topic to which the subscription applies
* Subscriber Node IDs: List of node IDs that have subscribed to the topic
* Subscription Status: Current status of the subscription (e.g., active,
inactive)

The subscription table is responsible for:

* Managing topic subscriptions from local applications and neighboring nodes
* Routing messages to the appropriate subscribers based on topic subscriptions
* Handling subscription updates, additions, and removals
* Ensuring efficient and reliable message delivery

By maintaining these tables, Messaging Nodes facilitate seamless communication and
message distribution in a SLIM network, enabling real-time interactive AI
applications at scale.

# Security Considerations


SLIM relies on the Messaging Layer Security (MLS) protocol
to provide end-to-end security for group communications between agents.

## MLS Integration

SLIM uses MLS for the following security properties:

* End-to-end encryption for all agent communications
* Forward secrecy and post-compromise security
* Group key management and membership changes
* Scalable group messaging security

## Authentication and Identity

Each agent MUST:

* Maintain cryptographic identities compatible with MLS
* Use certified credentials for initial authentication
* Validate peer credentials during connection establishment
* Support credential revocation and rotation

## Group Security

MLS provides the following guarantees for agent groups:

* Continuous group key updates
* Secure member addition and removal
* Protection against message forgery
* Perfect forward secrecy for all messages


## Operational Security

Implementations MUST:

* Maintain secure key storage
* Support MLS epoch advancement
* Implement proper credential management
* Monitor for security events
* Support secure group state recovery

## Privacy Considerations

SLIM with MLS provides:

* Metadata protection
* Group membership privacy
* Participant anonymity options
* Traffic analysis resistance

## Implementation Requirements

Implementations MUST NOT:

* Use non-MLS encryption schemes
* Support downgrades to less secure modes
* Allow plaintext communication
* Skip credential verification
* Allow plaintext communication
* Skip credential verification
