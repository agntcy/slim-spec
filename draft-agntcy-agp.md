---
###
# Description of Agent Gateway Protocol
###
title: "Agent Gateway Protocol"
abbrev: "agent-gw"
category: info

docname: draft-muscariello-agp-latest
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
  github: agntcy/agp
  latest: https://verbose-adventure-1pnqvyr.pages.github.io/

author:
 -
    fullname: Luca Muscariello
    organization: Cisco
    email: lumuscar@cisco.com


informative:


--- abstract


This document specifies the Agent Gateway Protocol (AGP), a protocol designed to
support real-time interactive AI applications at scale. AGP extends gRPC with
publish-subscribe capabilities to enable efficient many-to-many communication
patterns between AI agents. The protocol provides mechanisms for connection
management, stream multiplexing, and flow control while maintaining
compatibility with existing gRPC deployments.

--- middle

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Introduction


As AI systems become more sophisticated and interconnected, there is a growing need
for protocols that can support real-time interactive applications at scale. The Agent
Gateway Protocol (AGP) addresses this need by:

* Extending gRPC with publish-subscribe patterns
* Supporting bidirectional streaming between agents
* Enabling efficient many-to-many communication
* Maintaining backward compatibility with gRPC

## Protocol Overview

AGP builds on gRPC's core features while adding:

* Native support for pub/sub messaging patterns
* Enhanced stream multiplexing capabilities
* Real-time event notification system
* Dynamic topic creation and management

# Architecture

## Protocol Layers

~~~
         +-------------------+
         |     Application   |
         +-------------------+
         |   AGP Services    |
         +-------------------+
         |     Pub/Sub       |
         +-------------------+
         |      gRPC         |
         +-------------------+
         |      HTTP/2       |
         +-------------------+
~~~
{: #fig-stack title="Protocol Stack."}

## Core Components

* Gateway Nodes: Handle routing and message distribution
* Topics: Named channels for pub/sub communication
* Streams: Bidirectional communication channels
* Services: Application-specific RPC definitions

### Gateway Nodes

Gateway Nodes are essential components of the Agent Gateway Protocol (AGP) architecture. They handle routing and message distribution between agents and manage the communication infrastructure. Gateway Nodes are composed of two main tables: the connection table and the subscription table.

#### Connection Table

The connection table maintains interfaces with neighboring nodes and local applications. It is responsible for:

* Establishing and managing connections with other Gateway Nodes
* Maintaining active connections with local applications
* Handling connection setup, teardown, and error recovery

The connection table entries include:

* Node ID: Unique identifier for the neighboring node or local application
* Connection Status: Current status of the connection (e.g., active, inactive, error)
* Connection Parameters: Details such as IP address, port, and security credentials

#### Subscription Table

The subscription table is used to map topic subscriptions to neighboring nodes. It manages the distribution of messages based on topic subscriptions and ensures efficient routing of pub/sub messages. The subscription table entries include:

* Topic: The name of the topic to which the subscription applies
* Subscriber Node IDs: List of node IDs that have subscribed to the topic
* Subscription Status: Current status of the subscription (e.g., active, inactive)

The subscription table is responsible for:

* Managing topic subscriptions from local applications and neighboring nodes
* Routing messages to the appropriate subscribers based on topic subscriptions
* Handling subscription updates, additions, and removals
* Ensuring efficient and reliable message delivery

By maintaining these tables, Gateway Nodes facilitate seamless communication and message distribution in the AGP network, enabling real-time interactive AI applications at scale.

# Security Considerations


The Agent Gateway Protocol (AGP) relies on the Messaging Layer Security (MLS) protocol
to provide end-to-end security for group communications between agents.

## MLS Integration

AGP uses MLS for the following security properties:

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

AGP with MLS provides:

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