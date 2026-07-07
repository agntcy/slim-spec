---
title: "An Overview of Messaging Systems and Their Applicability to Agentic AI"
abbrev: "agntcy-messaging-eco"
category: info

docname: draft-mpsb-agntcy-messaging-02
submissiontype: independent
number:
date: 2026-07-07
consensus: false
v: 3
# area: Applications
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
  AMQP:
    title: "OASIS Advanced Message Queuing Protocol (AMQP) 1.0 Specification"
    author:
        - name: OASIS
    target: https://www.oasis-open.org/standards#amqp
  MQTT:
    title: "OASIS MQTT Version 5.0 Specification"
    author:
      - name: OASIS
    target: https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
  NATS:
    title: "NATS Documentation"
    author:
      - name: Synadia Communications
    target: https://docs.nats.io/
  Kafka:
    title: "Apache Kafka Documentation"
    author:
      - name: Apache Software Foundation
    target: https://kafka.apache.org/documentation/
  gRPC:
    title: "gRPC Documentation"
    author:
      - name: CNCF
    target: https://grpc.io/docs/
  SLIM:
    title: "AGNTCY SLIM Specification"
    author:
      - name: AGNTCY
    target: https://spec.slim.agntcy.org
  A2A:
    title: "Agent2Agent (A2A) Protocol"
    author:
      - name: Google
    target: https://google.github.io/A2A/
  MCP:
    title: "Model Context Protocol (MCP) Specification"
    author:
      - name: Anthropic
    target: https://modelcontextprotocol.io/
  SRPC:
    title: "SLIM RPC (SRPC) Reference"
    author:
      - name: AGNTCY
    target: https://github.com/agntcy/slim/blob/main/data-plane/slimrpc-compiler/README.md
  ACP:
    title: "Agent Client Protocol (ACP)"
    author:
      - name: Agent Client Protocol Community
    target: https://agentclientprotocol.com/get-started/introduction

--- abstract

Agentic AI systems require messaging infrastructure that supports real-time
collaboration, high-volume streaming, and dynamic group coordination across
distributed networks. Traditional protocols like AMQP {{AMQP}}, MQTT {{MQTT}}, and NATS {{NATS}} address
some requirements but fall short on security, particularly regarding
post-compromise protection and forward secrecy essential for autonomous
agents handling sensitive data.

This document analyzes six messaging protocols—AMQP, MQTT, NATS, AMQP over
WebSockets, Kafka, and AGNTCY SLIM—across dimensions critical for GenAI agent
systems: streaming performance, delivery guarantees, security models, and
operational complexity. We examine how each protocol's design decisions impact
agentic AI deployments, from lightweight edge computing scenarios to large-scale
multi-organizational collaborations.

AGNTCY SLIM emerges as a purpose-built solution, integrating Message Layer
Security (MLS) {{!RFC9420}} with gRPC {{gRPC}} over HTTP/2 {{!RFC9113}} to
provide end-to-end encryption with forward secrecy, efficient streaming, and OAuth-based
authentication {{!RFC6749}}. Unlike transport-layer security approaches, SLIM's
MLS implementation ensures secure communication even through untrusted
intermediaries while supporting dynamic
group membership changes essential for collaborative AI agents.

--- middle

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Introduction

When designing a multi-agent system for generative AI, the messaging layer
becomes a critical piece of infrastructure. GenAI agents—built with frameworks
like LangGraph, AutoGen, or LlamaIndex—often need to collaborate in real time,
exchange high volumes of streaming data (e.g., token-by-token outputs), and
coordinate complex tasks such as voting or consensus. Moreover, security
requirements extend well beyond basic TLS; in scenarios where agents share
sensitive models or partial computations, post-compromise security and robust
end-to-end encryption are essential.

In practice, a designer of such systems will require a protocol that efficiently handles one-to-many or
many-to-many communication, supports dynamic membership (with agents joining or
leaving on the fly), and scales to accommodate a “forest” of agents spread
across global networks. Some protocols excel at ultra-low-latency,
high-throughput streaming—critical for continuous token streams or aggregated
embeddings—while others emphasize strong consistency and durability.
Additionally, advanced cryptographic features such as automatic key rotation and
forward secrecy are vital when compromised credentials must not enable an
attacker to decrypt future communications.

Below, we compare six messaging protocols—AMQP, MQTT, NATS, AMQP over
WebSockets, Kafka, and AGNTCY SLIM (Secure Low-Latency Interactive
Messaging)—across dimensions that matter for GenAI agent systems: streaming
performance, delivery guarantees, flexible pub/sub patterns, agent coordination,
security (including end-to-end encryption and zero-trust support), and
real-world adoption.


# Protocol Analysis for Agentic AI Systems

The following sections provide detailed analysis of each messaging protocol in
the context of agentic AI requirements.

## Traditional Enterprise Messaging: AMQP

The Advanced Message Queuing Protocol (AMQP), most commonly implemented through
RabbitMQ, represents the gold standard for enterprise messaging systems. AMQP's
strength lies in its sophisticated message routing capabilities through
exchanges, queues, and routing keys, enabling complex message flow patterns
essential for enterprise applications.

For agentic AI systems, AMQP offers several advantages. Its support for both
at-least-once and exactly-once delivery semantics (particularly in AMQP 1.0)
ensures reliable message delivery between AI agents, which is crucial when
agents are coordinating critical tasks or sharing expensive computational
results. The protocol's durable queue support means that agent messages can
persist across system restarts, preventing loss of important coordination data.

However, AMQP's enterprise focus comes with trade-offs. The protocol carries
higher overhead due to its rich feature set, which may impact performance in
high-frequency agent communication scenarios. Streaming capabilities require
extensions like RabbitMQ Streams, adding complexity to deployments focused on
real-time agent collaboration.

Authentication in AMQP relies on traditional enterprise mechanisms like SASL,
LDAP, and Kerberos, which integrate well with existing corporate identity
systems but may not align with modern cloud-native authentication patterns
preferred in AI infrastructure.

## IoT-Optimized Messaging: MQTT

Message Queuing Telemetry Transport (MQTT) emerged from the IoT world with a
focus on lightweight, efficient communication over constrained networks. Its
topic-based publish-subscribe model maps naturally to many agent communication
patterns, where agents subscribe to topics representing different types of
events or data streams.

MQTT's three Quality of Service levels (QoS 0, 1, and 2) provide flexibility in
balancing performance versus reliability. For agentic AI systems, QoS 0
(at-most-once) works well for frequent status updates or non-critical
notifications, while QoS 2 (exactly-once) ensures critical agent coordination
messages are delivered reliably.

The protocol's very low overhead makes it attractive for scenarios involving
large numbers of lightweight AI agents or edge computing deployments where
bandwidth is constrained. However, MQTT's IoT heritage shows in its limitations
for agentic AI use cases. Native streaming support requires broker extensions,
and message-level security typically relies entirely on transport-layer TLS
rather than end-to-end encryption.

MQTT's authentication mechanisms, while sufficient for IoT devices, may not
provide the sophisticated identity and access management features required for
complex multi-agent AI systems involving different trust domains.

## Cloud-Native Messaging: NATS

NATS represents a modern approach to messaging designed for cloud-native
architectures. Its lightweight design and support for multiple communication
patterns—publish-subscribe, request-reply, and queue groups—make it particularly
well-suited for microservices-based AI agent deployments.

The protocol's core at-most-once delivery semantics align well with scenarios
where AI agents can tolerate occasional message loss in favor of high
performance. For use cases requiring stronger guarantees, NATS JetStream
provides at-least-once delivery and streaming capabilities, though this requires
additional infrastructure complexity.

NATS's optional broker architecture offers interesting deployment flexibility
for agentic AI systems. While most deployments use a broker for efficiency, the
protocol can support peer-to-peer communication, potentially enabling direct
agent-to-agent communication in specialized scenarios.

Authentication in NATS includes modern options like JWT tokens and NKey
cryptographic authentication, aligning better with cloud-native security
practices. However, like MQTT, NATS relies primarily on transport-layer security
rather than providing built-in end-to-end message encryption.

## Browser Integration: AMQP over WebSockets

AMQP over WebSockets addresses a specific deployment challenge: enabling
browser-based AI agents or user interfaces to participate in AMQP-based agent
coordination systems. This approach tunnels standard AMQP protocols through
WebSocket connections, allowing web applications to overcome firewall
restrictions and network topology limitations.

For agentic AI systems that include web-based components—such as user-facing AI
assistants that need to coordinate with backend AI agents—this protocol variant
provides a bridge between browser environments and enterprise messaging
infrastructure. The WebSocket Secure (WSS) transport ensures encrypted
communication from browser to broker.

However, the additional protocol layers (AMQP within WebSockets) introduce
higher overhead compared to native AMQP or other lightweight protocols. This
makes AMQP over WebSockets primarily suitable for scenarios where browser
integration is essential rather than for high-performance agent-to-agent
communication.

## High-Throughput Streaming: Apache Kafka

Apache Kafka {{Kafka}} represents a fundamentally different approach to messaging, based
on distributed commit logs rather than traditional message queues. This
architecture provides exceptional throughput and built-in streaming capabilities
that align well with certain agentic AI use cases.

Kafka's partition-based topic model enables massive horizontal scaling, making
it suitable for AI systems that need to process large volumes of training data,
model updates, or inference results across distributed agent networks. The
platform's native streaming capabilities through Kafka Streams and ksqlDB provide
powerful tools for real-time processing of agent-generated data.

The protocol's built-in persistence across distributed clusters ensures that
agent communication history is preserved and can be replayed, which is valuable
for AI systems that need to audit agent decisions or retrain models based on
historical interactions. Consumer groups enable multiple agents to process
different partitions of the same topic concurrently, supporting parallel AI
workloads.

However, Kafka's strengths come with complexity costs. The requirement for a
distributed cluster infrastructure may be overkill for simpler agent
coordination tasks. While Kafka provides exactly-once semantics through
transactions, the default at-least-once delivery may require additional
deduplication logic in agent implementations.

Kafka's security model, while comprehensive, relies primarily on transport-layer
encryption and broker-based access controls rather than end-to-end message
encryption, which may not meet the security requirements of AI systems handling
sensitive model data or proprietary algorithms.

## Next-Generation Agent Messaging: SLIM

AGNTCY SLIM {{SLIM}} (Secure Low-Latency Interactive Messaging) represents a
purpose-built protocol for modern agentic AI systems, designed to address the
specific security, performance, and coordination requirements that existing
protocols cannot fully satisfy.

SLIM is intended as a transport layer for agent protocols like A2A {{A2A}},
MCP {{MCP}}, and ACP {{ACP}}. It handles secure routing, group messaging, and
end-to-end encryption so protocol
implementations can focus on agent semantics. A registration-based model lets
agents become reachable through the SLIM network without exposing server ports,
while only routing nodes need to be publicly reachable. This simplifies
deployment for agents behind NATs and firewalls.

SLIM's foundation on gRPC over HTTP/2 and HTTP/3 provides several immediate
advantages for AI agent communication. The binary protocol buffer wire format
minimizes serialization overhead while supporting both binary and text data
types essential for AI workloads. HTTP/2's multiplexing capabilities allow a
single connection to carry multiple concurrent agent conversations, reducing
connection overhead in systems with many interacting agents.

SLIM is architected as a distributed system with a clear separation of
concerns:

- **Data plane**: Routes messages across SLIM nodes using only metadata for
  efficient forwarding and topology management.
- **Session layer**: Provides reliable delivery, MLS-based end-to-end
  encryption, and secure group management (create, invite, join, remove).
- **Control plane**: Orchestrates routing nodes, configuration, and
  administrative operations.

Routing nodes run only the data plane, keeping infrastructure lightweight,
while language bindings include the data-plane client plus the session layer
for full security and reliability.

The protocol's quality of service model explicitly addresses the diverse
communication patterns found in agentic AI systems. Fire-and-forget messaging
supports high-frequency status updates and non-critical notifications, while
reliable exactly-once delivery ensures critical coordination messages and
expensive computational results are never lost. This extends consistently across
request-reply patterns and streaming communications.

SLIM's security model is intentionally two-tiered. At the network layer,
HTTP/2 with TLS 1.3 provides hop-by-hop transport security, integrating
naturally with existing infrastructure—load balancers, API gateways, and
observability tools all operate on TLS-protected HTTP/2 connections without
requiring access to message content. At the content layer, MLS provides an
application-layer encryption envelope that protects message payloads
end-to-end, independently of how many TLS connections are traversed along the
path. Routing nodes see only the channel name and an encrypted blob; even after
terminating a TLS session, an intermediate node cannot access agent data.

This two-tier approach—HTTP/2 for network security, MLS for content
security—gives SLIM a zero-trust intermediary property: routing nodes can
forward messages through untrusted infrastructure while maintaining full payload
confidentiality. Protocols that rely solely on transport-layer security require
trust in all intermediaries, since each hop terminates TLS and has visibility
into message content. For agentic AI deployments spanning multiple organizations
or transiting third-party infrastructure, this distinction is critical.

The protocol's authentication model demonstrates particular innovation in
addressing agentic AI security requirements. By transporting MLS credentials and
cryptographic proofs within OAuth bearer tokens over HTTP/2, SLIM achieves
several important properties:

- **Interoperability**: Leverages standard HTTP/2 and OAuth libraries, reducing
  implementation complexity and improving compatibility with existing
  infrastructure
- **Scalability**: Single persistent HTTP/2 connections efficiently carry many
  MLS-secured messages between agents
- **Immediate revocation**: Malicious or compromised agents can be immediately
  ejected by revoking their OAuth tokens without requiring complex ratchet
  tree rebalancing operations

SLIM's naming system is hierarchical and DID-inspired, for example:
`organization/namespace/service/instance`. This supports anycast routing (to any
available instance), unicast routing (to a specific instance), and service
discovery without hardcoded endpoints or external registries. The structure maps
cleanly to organizational boundaries and multi-tenant deployments.

The protocol's support for both broker-based and peer-to-peer operation offers
deployment flexibility. While broker-based operation provides efficiency for
multi-party group communications typical in agent coordination scenarios,
peer-to-peer capabilities enable direct agent-to-agent communication when
appropriate. SLIM exposes two session types that map to common agent patterns:
point-to-point sessions for tool calls and group sessions for coordination and
broadcast.

SLIM provides multi-language bindings. Python and Go bindings are available
today, with JavaScript/TypeScript, C#, and Kotlin in progress, enabling
heterogeneous agent systems to interoperate on the same transport.

# Security Analysis

Security requirements for agentic AI systems extend well beyond the capabilities
provided by traditional messaging protocols. The autonomous nature of AI agents,
combined with their access to sensitive data and computational resources,
creates unique threat models that messaging infrastructure must address.

**Post-Compromise Security**: In traditional systems, credential compromise
typically requires immediate revocation and re-authentication. However, AI
agents may operate for extended periods with limited human oversight. SLIM's
MLS implementation provides forward secrecy, ensuring that compromise of
current credentials cannot decrypt past communications, and post-compromise
security, guaranteeing that future communications remain secure even after
credential compromise.

**Quantum-Safe Cryptography**: As quantum computing advances threaten current
cryptographic standards, AI systems—which may operate for years with the same
cryptographic keys—need protection against future quantum attacks. SLIM's
MLS implementation provides this protection and, when combined with
post-quantum cipher suites, also defends against future quantum attacks; traditional
protocols rely entirely on classical cryptographic assumptions that may become
vulnerable.

**Multi-Domain Operations**: Agentic AI systems often span multiple
organizational and security domains, with agents from different organizations
collaborating on shared tasks. Traditional protocols typically assume trust in
messaging infrastructure, but SLIM's end-to-end encryption ensures secure
communication even when messages transit through potentially untrusted
intermediaries.

**Dynamic Group Membership**: AI agent groups frequently change as agents join
collaborations, complete tasks, or become unavailable. MLS's efficient group
key management handles these membership changes while maintaining security
properties, unlike approaches that require complete cryptographic context
regeneration.

# Performance Characteristics

The performance characteristics of messaging protocols significantly impact the
behavior and capabilities of agentic AI systems, particularly as the number of
agents and frequency of interactions scale.

**Latency Sensitivity**: Many AI agent interactions are latency-sensitive,
particularly in real-time decision-making scenarios or when agents are
coordinating time-critical tasks. SLIM's HTTP/2 foundation provides header
compression and multiplexing that reduce per-message overhead, while the binary
protocol buffer encoding minimizes serialization costs.

**Throughput Requirements**: Large-scale agentic AI systems may involve
thousands of agents generating substantial message volumes. While protocols
like Kafka excel at raw throughput, they may introduce latency through their
log-based architecture. SLIM balances throughput and latency through efficient
connection reuse and optional reliability levels.

**Connection Efficiency**: Traditional protocols often require separate
connections for each communication pattern or security context. SLIM's
connection multiplexing allows a single HTTP/2 connection to handle diverse
communication patterns between agents, reducing resource consumption and
connection establishment overhead.

**Streaming Performance**: AI agents frequently exchange streaming data—such as
token-by-token language model outputs or real-time sensor data. SLIM's native
gRPC streaming support over HTTP/2 provides efficient bidirectional streaming
without the overhead of connection-per-stream approaches.

# Deployment and Operational Considerations

The operational characteristics of messaging protocols significantly impact the
total cost of ownership and operational complexity of agentic AI systems.

**Infrastructure Requirements**: Traditional enterprise protocols like AMQP
require dedicated message broker infrastructure with high availability and
clustering capabilities. Kafka requires even more complex distributed
infrastructure. SLIM's optional broker architecture allows deployments to scale
infrastructure complexity with system requirements.

**Monitoring and Observability**: Debugging distributed agentic AI systems
requires comprehensive visibility into agent communications. SLIM's foundation
on standard HTTP/2 infrastructure enables use of existing observability tools
and practices, while proprietary protocols may require specialized monitoring
solutions.

**Integration with Cloud Services**: Modern AI deployments increasingly rely on
cloud services for scalability and managed operations. SLIM's HTTP/2 foundation
integrates naturally with cloud load balancers, API gateways, and observability
services, while specialized messaging protocols may require additional
integration layers.

**Compliance and Auditing**: AI systems in regulated industries require
comprehensive audit trails and compliance capabilities. SLIM's structured topic
hierarchy and optional message persistence support regulatory requirements,
while the end-to-end encryption provides compliance with data protection
regulations.

# RPC in Agentic Protocols and Relationship to Messaging

Agentic AI systems are predominantly built around Remote Procedure Call
(RPC)-oriented protocols. A2A {{A2A}}, the Model Context Protocol (MCP) {{MCP}},
and the Agent Client Protocol (ACP) {{ACP}} all expose synchronous
request/response semantics: a caller issues a structured request, awaits a
response within a bounded time, and receives an explicit result or error. This
design provides well-defined interfaces, typed contracts, and composable error
handling that make agents discoverable and interoperable across heterogeneous
systems.

However, pure point-to-point RPC does not scale to all coordination demands of
multi-agent systems. Fan-out invocations, asynchronous event delivery, streaming
responses, dynamic group membership, and multi-domain security are capabilities
that RPC protocols delegate to the underlying transport. This section examines
how RPC and messaging interoperate in agentic systems, and how SLIM's SRPC
capability provides a purpose-built solution.

## Agentic Protocols Are RPC-Oriented

The three leading agentic coordination protocols each define an RPC-based
interaction model:

**Model Context Protocol (MCP)** defines a JSON-RPC interface between LLM hosts
and tools or resources. A host invokes tools by name with typed parameters and
receives typed results or streaming token outputs. MCP is designed for
synchronous tool calls with optional server-side streaming for partial results.

**Agent2Agent (A2A)** is an HTTP-based protocol for delegating tasks between
agents. An orchestrator submits task requests to worker agents and receives
structured results or a stream of progress events. Agent capabilities are
advertised via Agent Cards, enabling dynamic discovery without pre-configuration.

**Agent Client Protocol (ACP)** standardizes JSON-RPC communication between
code editors and coding agents, operating over stdio for local agents or HTTP
and WebSocket for remote agents. It enables editors to invoke coding agents and
receive structured responses, following a pattern analogous to the Language
Server Protocol for language tools.

All three share a fundamental structure: a caller invokes a named callee with
parameters and awaits a response within a bounded time. SLIM is designed to
serve as the transport layer beneath all three.

## RPC vs. Messaging: Synchronous vs. Asynchronous

- **RPC (A2A, MCP, ACP)**: The caller issues a request and blocks or awaits a
  timely response. Semantics emphasize tightly scoped operations with bounded
  latency and explicit error contracts.
- **Messaging (AMQP, MQTT, NATS, Kafka, SLIM)**: Decoupled producers and
  consumers communicate via topics, subjects, or queues. Delivery can be
  one-to-one, one-to-many, or many-to-many, with loose coupling, buffering,
  and retries. Producers are not inherently blocked by consumers.

In practice, agentic applications need both: synchronous tool invocations for
interactivity and asynchronous channels for streaming output, progress,
coordination, and fan-out/fan-in patterns.

## Challenges of Running RPC over Messaging

Layering RPC semantics onto an asynchronous messaging substrate introduces
several non-trivial challenges:

- **Request/response correlation**: Messaging systems are decoupled by design
  and provide no direct return path. Implementing RPC requires correlating
  requests and responses using unique identifiers and temporary reply channels.
- **Latency and ordering**: Messaging layers may introduce variable delivery
  latency and do not guarantee strict ordering, which can conflict with
  synchronous RPC contracts.
- **Error propagation**: Messaging systems may buffer, retry, or drop messages,
  making it difficult to propagate errors and timeouts in a way that matches
  RPC error contracts.
- **Streaming and multiplexing**: Supporting streaming RPC (server streaming,
  client streaming, bidirectional) over messaging requires careful management
  of stream lifecycles, backpressure, and multiplexing multiple logical RPCs
  over shared channels.
- **Security and authorization**: Ensuring that only authorized callers can
  invoke specific methods, and that all messages are authenticated and
  encrypted end-to-end, is more complex in a distributed, group-based
  messaging environment.

SRPC resolves all of these challenges natively, as described below.

## When Asynchronous Feels Synchronous

Asynchronous transports can provide an interactive, RPC-like experience when:

- A request message carries a correlation ID and a reply-to destination.
- The callee publishes a response to the reply destination within a short SLA.
- Client libraries surface responses as futures or promises and manage
  timeouts and retries.

This pattern underpins agent UIs where a user triggers an action and expects
prompt, possibly streaming, results.

## Bridging Patterns: RPC over Messaging

- **Request/Reply over Pub/Sub**: Implement RPC by publishing a command event
  and awaiting a correlated reply event (applicable to AMQP, NATS, MQTT, and
  SLIM topics).
- **Streaming RPC**: Use bidirectional streams (gRPC over SLIM HTTP/2/3) to
  deliver token streams, partial results, or progress updates while retaining
  an RPC caller experience.
- **Sagas and CQRS**: For multi-step workflows across agents, coordinate via
  asynchronous orchestration with idempotency keys, correlation/causation IDs,
  and compensating transactions.
- **Backpressure and Flow Control**: Prefer streaming transports (HTTP/2/3,
  gRPC) or messaging systems with flow control when returning large or
  continuous results.

## SLIM RPC (SRPC)

SRPC layers request/response semantics directly onto SLIM's secure messaging
fabric, addressing all of the challenges enumerated above:

- **Correlation and reply routing**: SRPC manages correlation IDs and reply
  channel lifecycle automatically, with no application-level plumbing required.
- **Idempotency and deduplication**: SRPC idempotency keys make retries safe
  without duplicating side effects.
- **Ordering and synchronization**: Lightweight ordering guarantees for both
  request/response and streaming patterns.
- **All four gRPC interaction patterns**: Unary, server streaming, client
  streaming, and bidirectional streaming are all supported, enabling direct
  mapping of A2A, MCP, and ACP interaction models onto SLIM.
- **Integrated security**: Every SRPC call inherits SLIM's MLS end-to-end
  encryption and OAuth-based authorization without additional configuration.

See the SRPC specification for details {{SRPC}}.

## Advantages of SLIM for Agentic Protocols

SLIM augments the point-to-point RPC model of A2A, MCP, and ACP with
capabilities that are difficult to achieve over plain request/response transports:

- **Scatter-gather RPC**: Invoke a single RPC across many agents simultaneously
  (by topic, group, or label) and aggregate responses (first-success, quorum,
  or all-success) using correlation IDs.
- **Group addressing and dynamic membership**: Target MLS-secured groups; add
  or remove agents without reconfiguring endpoints or updating caller code.
- **Streaming responses**: Return partial results or token streams from each
  agent over a single multiplexed connection.
- **Idempotency and safe retries**: SRPC idempotency keys enable robust retry
  without duplicating effects—critical for expensive or stateful agent
  operations.
- **QoS, deadlines, and backpressure**: Apply delivery guarantees, per-call
  timeouts, and flow control to avoid overload while maintaining interactivity.
- **End-to-end security**: MLS encryption and OAuth-based policy apply
  uniformly across both RPC and messaging channels.
- **Observability**: Correlation and causation IDs, combined with standard
  HTTP/2 transport, enable distributed tracing and per-agent metrics with
  existing tooling.

These capabilities let A2A, MCP, and ACP interactions scale beyond one-to-one
invocations, enabling broadcast queries, coordinated multi-agent actions, and
efficient collection of results in real time.

## Security Implications

- **Unified identity**: OAuth tokens are reused across RPC calls and messaging
  channels for consistent policy enforcement. An agent's identity and
  authorization scope apply uniformly to tool calls and subscription channels.
- **End-to-end encryption**: MLS-backed channels ensure that RPC requests,
  responses, and streaming payloads are encrypted end-to-end, independently
  of transport hops. Routing nodes cannot inspect RPC call content.
- **Caller authentication**: SRPC carries MLS group membership proofs with
  each call, allowing callees to verify that the caller is an authorized group
  member before executing the requested operation.

## Guidance: When to Choose What

- Use **RPC (A2A, MCP, ACP)** for low-latency point operations with immediate
  feedback and well-defined error contracts.
- Use **Messaging** for broadcast/fan-out, decoupling, retries, buffering,
  and multi-party coordination.
- Use **SRPC over SLIM** for any of the above when end-to-end security,
  scatter-gather invocation, streaming responses, or multi-organization
  deployment is required.

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
|---------|---------------------|-------|------|
| **Protocol Type** | AMQP tunneled through WebSockets | Distributed commit log, high-throughput pub/sub | Secure low-latency interactive messaging |
| **Transport** | WebSockets over TLS | TCP (optionally TLS) | gRPC (over HTTP/2-HTTP/3) |
| **Message Model** | Same as AMQP (depends on the broker's AMQP model) | Topics with partitions, consumer groups, offset-based consumption | Hierarchical names (`org/namespace/service/instance`), point-to-point and group sessions |
| **QoS / Delivery** | Same as AMQP | At-least-once default; exactly-once possible via transactions | At-most-once (fire-and-forget) or exactly-once (reliable); applies consistently to request/reply and streaming interactions |
| **Streaming** | Same as AMQP if broker supports streaming | Native log-based streaming (Kafka Streams, ksqlDB, etc.) | Native gRPC support via HTTP/2/3 client streaming, server streaming |
| **Persistence** | Same as AMQP | Built-in: messages persist on disk across clusters | Not supported |
| **Protocol Overhead** | Higher (AMQP + WebSockets handshake) | Moderate (custom binary protocol, but optimized for high throughput) | Low: binary Protocol Buffers wire format; native support for both binary (bytes) and text payloads |
| **Broker Required** | Yes | Yes (distributed cluster) | Yes for multi-party group sessions; peer-to-peer also supported |
| **Authentication** | Same as AMQP (broker-based) | SASL/PLAIN, SASL/SCRAM, Kerberos, OAuth | OAuth bearer tokens carrying MLS credentials and cryptographic proofs; standard HTTP/2 and OAuth libraries for interoperability; immediate revocation by invalidating OAuth tokens |
| **Transport Security** | WSS (WebSocket Secure) | TLS | TLS |
| **Message Security** | Same as AMQP (depends on the broker's encryption at rest/in-transit) | TLS in-flight encryption, optional at-rest encryption (broker config) | MLS end-to-end encryption with forward secrecy and post-compromise security, independent of transport |
| **Binary or Text** | Binary AMQP frames over WebSockets | Binary protocol (common payloads: Avro, JSON, Protobuf) | Binary or Text |
| **Use Cases** | Browser-based apps needing AMQP behind firewalls | High-throughput data pipelines, streaming analytics, event sourcing | Group messaging, one-to-many, many-to-many, cloud-native microservices, real-time communications, streaming |
| **Real-World Usage** | Less common, mainly for browser/firewall scenarios using RabbitMQ or similar | Extremely widespread across industries; de facto standard for large-scale event streaming | New entrant; reference implementations in Python and Go available; early production deployments |

# Security Considerations

This document is an informational comparison of messaging protocols for agentic
AI systems. Security properties specific to each protocol are discussed in the
Security Analysis section of this document. Readers deploying any
of the protocols described SHOULD consult the security considerations sections
of the respective protocol specifications.

Of the protocols analyzed, only SLIM provides application-layer end-to-end
encryption via MLS {{!RFC9420}}, forward secrecy, and post-compromise security
independent of transport-layer security. All other protocols rely on
transport-layer TLS, which does not protect message content from trusted
intermediaries.

# IANA Considerations

This document has no IANA actions.
