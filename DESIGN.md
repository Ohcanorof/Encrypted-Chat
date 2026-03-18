# Encrypted Chat Research Project - Design Document

## 1. Objective
Build a Python-based end-to-end encrypted (E2EE) chat application where an untrusted relay server forwards messages but never sees plaintext. The client performs all key management and encryption/decryption locally.

This project also includes a research component so it is not only an application build. The research focuses on evaluating modern encryption techniques used in secure messaging and verifying security properties through controlled tests.

Primary goals:
- Implement a working E2EE 1-to-1 chat system (two clients + relay server).
- Implement replay and tamper defenses and verify them with automated tests.
- Benchmark modern AEAD encryption choices used today and document tradeoffs.

Optional goal:
- Add a client-side AI safety feature that runs locally and never sends plaintext to the server.

## 2. Scope

### 2.1 Functional Requirements
Client features:
- Connect to relay server and register a unique username.
- List online users.
- Send and receive 1-to-1 messages.
- Establish a secure session with another user and exchange encrypted messages.
- Reject messages that fail authentication or fail replay/counter rules.

Server features:
- Accept TCP connections.
- Register usernames.
- Relay messages to the intended recipient using the "to" field.
- Never decrypt or inspect plaintext.

### 2.2 Non-Functional Requirements
- Cross-platform development (Windows + VS Code primary).
- Reproducible runs and tests.
- Clear logging and error handling.
- Test suite for protocol correctness, cryptography behavior, and threat simulations.
- Research artifacts: benchmark results and a short report section in the README.

## 3. Threat Model

### 3.1 Attacker Capabilities
- The relay server is untrusted and may attempt to read, modify, replay, reorder, or drop packets.
- A network attacker may intercept, replay, or tamper with packets.
- The attacker does not break modern cryptographic primitives.

### 3.2 Defended Attacks
- Eavesdropping: prevented by encryption.
- Ciphertext tampering: detected by AEAD authentication.
- Header tampering (from/to/counter): detected by authenticating headers using AEAD AAD.
- Replay of old packets: rejected using a per-peer counter policy.

### 3.3 Out of Scope (Unless Added as Stretch Goals)
- Full Signal Double Ratchet (forward secrecy and post-compromise security).
- Metadata privacy against traffic analysis.
- Group chat and multi-device sync.

## 4. Architecture Overview

### 4.1 Components
1. Relay Server (Untrusted)
- Maintains a mapping of username -> socket connection.
- Forwards messages based on "to".
- Performs no encryption or decryption.

2. Client (Trusted)
- Handles key generation and session establishment.
- Encrypts outgoing messages and decrypts incoming messages.
- Maintains per-peer counters for replay protection.
- Provides a CLI interface for sending and receiving messages.

3. Common Protocol Layer
- Defines JSON message schemas.
- Implements length-prefixed message framing for TCP.
- Validates messages and converts between objects and JSON.

4. Research Tools
- Benchmark runner for measuring encryption performance.
- Attack harness for replay and tamper simulations.

## 5. Protocol and Crypto Design

### 5.1 Transport and Message Framing
- TCP sockets are used for transport.
- Messages are encoded as UTF-8 JSON.
- Each JSON message is sent with a 4-byte big-endian length prefix to define message boundaries.

### 5.2 Message Types
Register:
- type: "register"
- username: string

List:
- type: "list"

Message (encrypted):
- type: "msg"
- from: string
- to: string
- counter: integer
- nonce: string (encoded)
- ciphertext: string (encoded)
- aad: string or structured fields (implementation detail)

Bye:
- type: "bye"

Error/OK:
- type: "error" or "ok"
- message: string

### 5.3 Key Establishment
Baseline approach:
- Use X25519 (ECDH) to derive a shared secret between two clients.
- Use HKDF to derive symmetric keys from the shared secret.
- Store session state per peer.

Note: A full identity verification system is not required for the baseline. A trust-on-first-use model can be documented. Manual fingerprint verification can be added as a stretch feature.

### 5.4 Message Encryption
Use an AEAD cipher to provide confidentiality and integrity:
- AES-GCM
- ChaCha20-Poly1305

The project will implement one as the baseline and include the second for benchmarking and comparison.

### 5.5 Authenticating Headers with AAD
To prevent tampering with metadata, the following fields are authenticated as Associated Authenticated Data (AAD):
- from
- to
- counter

If any of these fields are modified in transit, decryption must fail.

### 5.6 Replay Protection
- Each session maintains:
  - send_counter (increment on each send)
  - expected_recv_counter (monotonic check on receive)
- The client rejects any message with a counter less than the expected value.
- For the baseline, messages are assumed in-order. Out-of-order handling can be added later.

## 6. Research Component

## 6.1 Research Track A: Benchmarking Modern AEAD Ciphers
Research question:
- How do AES-GCM and ChaCha20-Poly1305 compare in latency and throughput in this chat protocol?

Experimental variables:
- Cipher choice: AES-GCM vs ChaCha20-Poly1305
- Message sizes: 32B, 256B, 1KB, 10KB, 100KB
- Message counts: 100, 1,000, 10,000

Metrics:
- Encrypt time and decrypt time (ms)
- End-to-end encrypt+send and receive+decrypt latency (ms)
- Throughput (messages/sec)

Outputs:
- CSV or JSON results
- Summary table(s) and plots
- Discussion of tradeoffs and misuse risks (nonce reuse, error handling)

## 6.2 Research Track B: Threat Testing and Attack Harness
Research question:
- Which attacks are detected and how reliably? Where do naive designs fail?

Attack simulations:
- Replay a previously captured packet
- Flip bits in ciphertext
- Modify from/to/counter
- Reorder messages
- Drop messages (simulate packet loss)

Metrics:
- Detection rate for each attack type
- False positive rate (rejecting valid messages)
- Clarity of error reporting

Outputs:
- Automated tests (pytest)
- A short threat test report section in README

## 6.3 Optional Track C: Client-Side AI Safety Feature
AI must run locally and never send plaintext to the server.

Possible features:
- Warning for phishing or credential theft patterns
- PII leak warning before sending a message
- Local chat summarization for user convenience

Evaluation options:
- Accuracy on a small labeled dataset (precision/recall)
- Latency overhead per message

If implemented, AI is treated as a secondary feature and is isolated from core cryptography code.

## 7. Project Milestones
1. Plaintext prototype: relay server and two clients exchange plaintext messages.
2. Protocol stabilization: message framing and schema validation.
3. Crypto v1: key establishment and encrypted messaging with one AEAD cipher.
4. Security: counter-based replay protection and tamper detection.
5. Research: implement benchmark runner and attack harness; collect and document results.
6. Optional: implement client-side AI safety feature and evaluate it.

## 8. Repository and File Structure
encrypted-chat/
- DESIGN.md
- README.md
- requirements.txt
- src/
  - common/
    - protocol.py        (length-prefixed JSON framing)
    - messages.py        (message dataclasses/schemas)
    - errors.py          (custom exceptions)
    - utils.py           (helpers)
  - crypto/
    - kex.py             (X25519 + HKDF)
    - aead.py            (AEAD wrappers and selection)
    - session.py         (session keys and counters)
    - nonce.py           (nonce helpers)
  - server/
    - server.py          (relay server entry)
    - registry.py        (username mapping)
  - client/
    - client.py          (client entry)
    - ui.py              (CLI parsing and commands)
    - receiver.py        (receive loop thread)
    - store.py           (optional persistence)
  - research/
    - benchmark.py       (benchmark runner)
    - attack_harness.py  (attack simulations)
- tests/
  - test_protocol.py
  - test_crypto.py
  - test_replay.py
  - test_attacks.py

## 9. UML-Oriented Class Outline

### Common/Protocol
ProtocolFramer
- send(sock, message_dict) -> None
- recv(sock) -> message_dict

Message Types (dataclasses or lightweight classes)
- RegisterMessage(username)
- ListMessage()
- ChatMessage(from_user, to_user, counter, nonce, ciphertext)
- OkMessage(message)
- ErrorMessage(message)

### Crypto
AEADCipher (interface)
- encrypt(key, nonce, plaintext, aad) -> ciphertext
- decrypt(key, nonce, ciphertext, aad) -> plaintext

AESGCMCipher, ChaCha20Poly1305Cipher
- implements AEADCipher

KeyExchange
- create_keypair() -> (private_key, public_key)
- derive_shared_secret(private_key, peer_public_key) -> bytes
- derive_session_keys(shared_secret) -> (send_key, recv_key)

Session
- peer: string
- send_key: bytes
- recv_key: bytes
- send_counter: int
- expected_recv_counter: int
- encrypt_message(plaintext, header_fields) -> (nonce, ciphertext)
- decrypt_message(nonce, ciphertext, header_fields) -> plaintext
- check_counter(counter) -> bool

### Server
ClientRegistry
- register(username, socket) -> bool
- unregister(username) -> None
- get(username) -> socket or None
- list_users() -> list[str]

RelayServer
- start(host, port)
- handle_client(socket)
- relay_message(chat_message_dict)

### Client
ChatClient
- connect(host, port)
- register(username)
- list_users()
- send(to_user, plaintext)
- receive_loop()

CommandParser (optional)
- parse(line) -> command/action

### Research
BenchmarkRunner
- run(cipher, sizes, counts) -> results
- save_results(path)

AttackHarness
- replay(packet)
- tamper_ciphertext(packet)
- tamper_header(packet)
- reorder(packets)
- run_all() -> report

## 10. Deliverables
- Working E2EE chat demo with an untrusted relay server.
- Automated tests including replay and tamper simulations.
- Benchmark results comparing AEAD cipher choices.
- Documentation: this design document and a README containing setup, usage, and research findings.