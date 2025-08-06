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
