# Conduit

Conduit is a peer-to-peer coordination and settlement protocol designed for secure, fault-tolerant message exchange between untrusted peers over unreliable networks.

The project focuses on protocol correctness, explicit failure handling, and clear architectural trade-offs rather than throughput maximisation or feature breadth. Conduit is intended as a systems and networking exercise, not a production-ready marketplace.

## Motivation

Many peer-to-peer systems conflate application logic, trust assumptions, and transport concerns, making it difficult to reason about failure modes and correctness under real network conditions. Conduit aims to separate these concerns by providing a minimal coordination layer that can operate under packet loss, latency, and peer churn without relying on central infrastructure.

The primary goal is to explore how untrusted peers can reliably exchange signed messages and coordinate simple settlement workflows in a hostile but non-anonymous network environment.

## Design Goals

- Explicit protocol state machines with well-defined transitions

- Secure peer identity based on public-key cryptography

- Fault tolerance under packet loss, reordering, and duplication

- Idempotent message handling and replay protection

- Minimal trusted assumptions and no central coordinator

## Non-Goals

- Conduit does not provide anonymity or traffic obfuscation beyond standard transport encryption.

- Conduit does not implement a full marketplace, including listings, pricing logic, or reputation systems.

- Conduit does not guarantee exactly-once delivery; message handlers are designed to be idempotent.

- Conduit does not aim to maximise throughput; predictable behaviour under latency and peer churn is prioritised.

- Conduit does not attempt to fully mitigate Sybil attacks beyond basic admission controls.

## Architecture Overview

Conduit is structured as a small set of composable layers:

- *Transport Layer:* Responsible for unreliable message delivery (e.g. UDP) and basic connection management.

- *Identity Layer:* Handles peer identity, key management, and message signing.

- *Protocol Layer:* Defines message formats, versioning, replay protection, and state machines.

- *Coordination Layer:* Implements offer exchange and settlement handshakes.

Each layer is intentionally narrow in scope and communicates through explicit interfaces.

## Networking Model
The protocol is designed to operate over unreliable transport and assumes:
- Messages may be dropped, duplicated, or reordered
- Peers may disconnect or restart at any time
- Network latency may vary significantly between peers
Correctness is achieved through explicit retries, timeouts, and idempotent message processing rather than transport-level guarantees.

## Security Model

- All protocol messages are signed using long-lived peer identities

- Session-level keys may be rotated without changing peer identity

- Replay protection is enforced at the protocol layer

Out of scope attacks and limitations are documented explicitly in the Non-Goals section.

## Implementation Notes

- Implemented in Rust with an emphasis on memory safety and explicit ownership

- Binary message formats with versioned envelopes

- No heap allocations on critical protocol paths where feasible

- Unsafe code is avoided unless justified by measurable benefit

## Protocol Specification

This section provides a high-level, implementation-oriented description of the Conduit protocol. It is intentionally concise and focuses on observable behaviour rather than internal optimisations.

### Terminology

- *Peer:* A participating node identified by a long-lived public key

- *Identity:* A peer's public/private key pair used for signing protocol messages

- *Session:* A temporary association between two peers for exchanging protocol messages

- *Envelope:* A versioned, signed wrapper around a protocol message

### Message Envelope

All protocol messages are transmitted inside a signed envelope with the following logical fields:

- Protocol version

- Message type

- Sender identity

- Monotonic message identifier

- Payload

- Signature

The envelope format is stable and versioned to allow forward-compatible changes.

### Transport Assumptions

Conduit assumes an unreliable transport:

- Messages may be dropped, duplicated, or reordered

- No delivery, ordering, or liveness guarantees are provided by the transport

- The protocol does not rely on transport-level reliability for correctness.

### Sessions

Sessions are established opportunistically when peers exchange valid signed messages. There is no explicit connection handshake; session state exists solely to support retry logic, replay protection, and timeouts.

Session loss or restart does not imply protocol failure.

### Message Handling

- All messages are processed idempotently

- Duplicate messages are detected using sender identity and message identifiers

- Messages that cannot be validated or parsed are discarded without response

### Retry and Timeout Behaviour

Peers may retransmit messages if acknowledgements are not received within a bounded timeout. Timeouts are advisory and tuned for correctness rather than throughput.

### Offer and Settlement Coordination

The protocol supports a minimal coordination workflow:

1. A peer publishes an offer message

2. A counterparty responds with an acceptance message

3. Both peers exchange settlement confirmation messages

Each step is explicitly acknowledged and may be retried safely.

### Failure Handling

The protocol explicitly tolerates:

- Peer restarts at any point in the workflow

- Duplicate or delayed messages

- Partial completion of coordination flows

Peers may abandon in-progress workflows after bounded retry attempts.

### Security Properties

Conduit provides:

- Message authenticity via digital signatures

- Replay protection at the protocol layer

- Explicit trust boundaries between peers

It does not provide anonymity, confidentiality beyond transport encryption, or global ordering guarantees.

## Protocol Invariants

The following invariants are maintained by all conforming implementations:

- A peer never acts on an unauthenticated or unsigned message

- Protocol state transitions are monotonic and cannot be rolled back by message replay

- Reprocessing the same message does not produce duplicate side effects

- Loss of session or in-memory state does not violate protocol correctness

- No protocol decision depends on transport-level ordering or delivery guarantees

## Project Status

Conduit is an experimental project under active development. The protocol, APIs, and internal structure are subject to change as design trade-offs are explored and documented.

## License

This project is licensed under the Apache License version 2.0.
