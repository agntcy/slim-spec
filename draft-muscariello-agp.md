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

         +-------------------+
         |     Application   |
         +-------------------+
         |   AGP Services    |
         +-------------------+
         |     Pub/Sub      |
         +-------------------+
         |      gRPC        |
         +-------------------+
         |      HTTP/2      |
         +-------------------+

## Core Components

* Gateway Nodes: Handle routing and message distribution
* Topics: Named channels for pub/sub communication
* Streams: Bidirectional communication channels
* Services: Application-specific RPC definitions


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations


The Agent Directory Protocol relies on several security mechanisms to ensure the
integrity, authenticity, and privacy of directory records:

## Record Signatures

All agent directory records MUST be digitally signed by the producing agent. The
signature covers:

* The complete set of OASF attributes
* The agent's capabilities description
* Any additional metadata including timestamps
* Version information

Signatures enable consumers to verify the authenticity and integrity of records
independent of their location in the DHT.

## Location Independence

Agent directory records are location-independent - their trust is derived from
cryptographic signatures rather than network location. This means:

* Records can be cached and replicated across the DHT
* Consumers can verify records regardless of the serving node
* Man-in-the-middle attacks are prevented through signature verification
* Trust is bound to cryptographic identities rather than network addresses

## Key Management

Agents MUST generate and maintain cryptographic key pairs following these requirements:

* Use of asymmetric cryptography (e.g., Ed25519) for signing
* Private keys MUST be properly secured by agents using hardware security modules where available
* Public keys are distributed as part of agent records
* Key rotation procedures MUST be supported and documented
* Revocation mechanisms MUST be provided

## DHT Security

The DHT implementation MUST provide:

* Node authentication to prevent Sybil attacks
* Secure routing to prevent record tampering
* Replication policies to ensure availability
* Access controls for record updates
* Protection against eclipse attacks
* Rate limiting of requests
* Peer reputation tracking

## Transport Security

All protocol interactions MUST use secure transport with:

* Mutual TLS authentication between nodes
* Perfect forward secrecy
* Strong cipher suites as defined in TLS 1.3
* Certificate-based authentication
* Revocation checking

Implementations MUST NOT support:

* Plaintext communications
* Weak cipher suites
* Older TLS versions

## Privacy Considerations

The protocol implements privacy protection through:

* Minimal attribute disclosure
* Encrypted record contents
* Anonymous routing capabilities
* Pseudonymous agent identities
* Access control mechanisms

## Operational Security

Implementers MUST consider:

* Regular key rotation schedules
* Secure bootstrapping procedures
* Node authentication policies
* Resource exhaustion protections
* Monitoring and alerting systems
* Incident response procedures


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
