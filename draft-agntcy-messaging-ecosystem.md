---
###
# Description of Messaging Ecosystem
###
title: "An Overview of Messaging Systems and Their Applicability to Agentic AI"
abbrev: "agncty-messaging-eco"
category: info

docname: draft-agntcy-messaging-ecosystem-latest
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



--- abstract

When designing a multi-agent system for generative AI, the messaging layer
becomes a critical piece of infrastructure. GenAI agents—built with frameworks
like LangGraph, AutoGen, or LlamaIndex—often need to collaborate in real time,
exchange high volumes of streaming data (e.g., token-by-token outputs), and
coordinate complex tasks such as voting or consensus. Moreover, security
requirements extend well beyond basic TLS; in scenarios where agents share
sensitive models or partial computations, post-compromise security and robust
end-to-end encryption are essential.

In practice, you’ll want a protocol that efficiently handles one-to-many or
many-to-many communication, supports dynamic membership (with agents joining or
leaving on the fly), and scales to accommodate a “forest” of agents spread
across global networks. Some protocols excel at ultra-low-latency,
high-throughput streaming—critical for continuous token streams or aggregated
embeddings—while others emphasize strong consistency and durability.
Additionally, advanced cryptographic features such as automatic key rotation and
forward secrecy are vital when compromised credentials must not enable an
attacker to decrypt future communications.

Below, we compare six popular messaging protocols—AMQP, MQTT, NATS, AMQP over
WebSockets, Kafka, and the emerging AGP (Agent Gateway Protocol)—across
dimensions that matter for GenAI agent systems: streaming performance, delivery
guarantees, flexible pub/sub patterns, agent coordination, security (including
end-to-end encryption and zero-trust support), and real-world adoption.


--- middle

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Introduction

# Comparison

Table 1 provides a detailed comparison of three popular messaging protocols commonly considered for agent communication systems:

| Feature | AMQP (e.g. RabbitMQ) | MQTT | NATS |
|---------|----------------------|------|------|
| **Protocol Type** | Message queueing (queues/exchanges) | Lightweight pub/sub for IoT | Lightweight messaging (pub/sub, req/reply, queue groups) |
| **Transport** | TCP (optionally TLS) | TCP (optionally TLS) | TCP (optionally TLS) |
| **Message Model** | Queues, exchanges, routing keys | Topic-based | Subjects (pub/sub), queue groups, request/reply |
| **QoS / Delivery** | At-least-once, exactly-once (AMQP 1.0) | QoS 0 (at-most-once), 1, 2 (exactly-once) | At-most-once (core), at-least-once with JetStream |
| **Streaming** | Via extensions/plugins (e.g. RabbitMQ Streams) | Not native (requires broker extensions) | Native with JetStream |
| **Persistence** | Yes (durable queues) | Broker-dependent | Optional via JetStream |
| **Protocol Overhead** | Higher (rich feature set) | Very low | Very low |
| **Broker Required** | Yes | Yes | Optional (but common) |
| **Authentication** | User/password, SASL (e.g., LDAP, Kerberos) | Username/password or custom tokens | NKey, JWT, token, user/password |
| **Transport Security** | TLS | TLS | TLS |
| **Message Security** | Typically broker-level or plugin-based encryption | Usually none at message level; rely on TLS | None in core (TLS in transit), JetStream can encrypt at rest |
| **Binary or Text** | Binary framing | Binary framing | Text-based protocol (core), binary clients available |
| **Use Cases** | Enterprise messaging, financial transactions, RPC | IoT, mobile, sensor networks | Cloud-native microservices, real-time communications |
| **Real-World Usage** | Very widely used via RabbitMQ (top open-source broker) in enterprises of all sizes | Dominant in IoT ecosystems; supported by many device/broker vendors | Gaining traction in cloud-native (CNCF project), used by major tech companies |

Table 2 extends the comparison to include additional protocols relevant to modern agentic AI systems:

| Feature | AMQP over WebSockets | Kafka | SLIM |
|---------|---------------------|-------|-----|
| **Protocol Type** | AMQP tunneled through WebSockets | Distributed commit log, high-throughput pub/sub | AGP Spec |
| **Transport** | WebSockets over TLS | TCP (optionally TLS) | gRPC (over HTTP/2-HTTP/3) |
| **Message Model** | Same as AMQP (depends on the broker's AMQP model) | Topics with partitions, consumer groups, offset-based consumption | Topics based on organization, namespace, agent types etc. |
| **QoS / Delivery** | Same as AMQP | At-least-once default; exactly-once possible via transactions | Fire&Forget unreliable (at-most-once), unreliable and reliable (exactly-once). This extends to request/reply and streaming as well. |
| **Streaming** | Same as AMQP if broker supports streaming | Native log-based streaming (Kafka Streams, KSQL, etc.) | Native gRPC support via HTTP/2/3 client streaming, server streaming. Notice that Server Sent Events (SSE) with HTTP/1.1 cannot carry binary nor compressed data. |
| **Persistence** | Same as AMQP | Built-in: messages persist on disk across clusters | Not supported |
| **Protocol Overhead** | Higher (AMQP + WebSockets handshake) | Moderate (custom binary protocol, but optimized for high throughput) | Low: Wire format uses protocol buffer. Supports also binary (byte type in protobuf) |
| **Broker Required** | Yes | Yes (distributed cluster) | Yes for efficient multi-party. P2P is also possible. |
| **Authentication** | Same as AMQP (broker-based) | SASL/PLAIN, SASL/SCRAM, Kerberos, OAuth | Transports MLS credentials and proofs inside OAuth bearer tokens over HTTP/2. This gives you: Interoperability: Leverage standard HTTP/2 and OAuth libraries. Scalability: One persistent HTTP/2 connection carries many MLS messages. Immediate revocation: Eject bad actors by revoking their OAuth tokens—no need to rebalance the ratchet tree first. |
| **Transport Security** | WSS (WebSocket Secure) | TLS | TLS |
| **Message Security** | Same as AMQP (depends on the broker's encryption at rest/in-transit) | TLS in-flight encryption, optional at-rest encryption (broker config) | MLS (Quantum safe, Secure end-to-end, even across insecure hops, post-compromise security) |
| **Binary or Text** | Binary AMQP frames over WebSockets | Binary protocol (common payloads: Avro, JSON, Protobuf) | Binary or Text |
| **Use Cases** | Browser-based apps needing AMQP behind firewalls | High-throughput data pipelines, streaming analytics, event sourcing | Group messaging, one-to-many, many-to-many, Cloud-native microservices, real-time communications, streaming |
| **Real-World Usage** | Less common, mainly for browser/firewall scenarios using RabbitMQ or similar | Extremely widespread across industries; de facto standard for large-scale event streaming | New Entrant, low |
