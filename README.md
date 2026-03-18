# Encrypted Chat Research Project

## Overview
This project aims to build a Python-based end-to-end encrypted (E2EE) 1-to-1 chat application where an untrusted relay server forwards messages but never sees plaintext. All encryption and decryption happens on the client.

In addition to building the application, this project includes a research component focused on evaluating modern encryption techniques and verifying security properties through controlled tests.

Design details are documented in DESIGN.md.

## Goals
### Core application goals (planned)
- Relay server that accepts TCP connections, registers usernames, and forwards messages based on the recipient field.
- Client that can connect, register, list online users, and send messages to a specific user.
- Length-prefixed JSON message framing for reliable TCP messaging.
- End-to-end encrypted messaging (server only relays ciphertext).
- Replay resistance using a per-peer message counter.
- Tamper detection using AEAD authentication, including authenticating key header fields (from/to/counter) via AEAD AAD.
- Automated tests for protocol behavior and basic threat simulations.

### Research goals (planned)
- Benchmark modern AEAD encryption choices used in secure messaging (AES-GCM vs ChaCha20-Poly1305).
- Measure latency/throughput across different message sizes and message counts.
- Implement a small attack harness to simulate replay and tampering and record outcomes.
- Document results in a short report section (tables and discussion).

## Possible stretch goals (may or may not be completed)
- Better key trust model (fingerprints and manual verification, trust-on-first-use documentation).
- Handling out-of-order messages (baseline assumes in-order delivery).
- Optional client-side AI safety feature that runs locally only (examples: phishing/credential-theft warning, PII leak warning, local summarization). Any AI feature must not send plaintext to the server.
- Persisted session state (saving keys/sessions locally between runs).
- Improved CLI usability (history, nicer commands, configuration file).

## Non-goals (out of scope for the baseline)
- Full Signal Double Ratchet (forward secrecy and post-compromise security).
- Metadata privacy against traffic analysis.
- Group chat and multi-device sync.

## Current status
This repository is in an early, barebones state. Initial development focuses on:
1. Plaintext relay prototype (server + two clients)
2. Protocol framing and schema stabilization
3. Adding cryptography and security checks
4. Adding tests and research tooling (benchmarking and attack harness)

## Project structure (planned)
encrypted-chat/
- README.md
- DESIGN.md
- requirements.txt
- src/
  - common/
  - crypto/
  - server/
  - client/
  - research/
- tests/

## Setup (development)
1. Create and activate a virtual environment.
2. Install dependencies from requirements.txt.

Example commands (Windows PowerShell):
- py -m venv .venv
- .\.venv\Scripts\Activate.ps1
- python -m pip install --upgrade pip
- pip install -r requirements.txt

## Running (planned)
Once implemented, the intended workflow will be:
1. Run the relay server.
2. Run two clients with different usernames.
3. Send messages between clients (plaintext first, then encrypted).

Exact commands and usage will be added as the server/client modules are implemented.

## Design document
See DESIGN.md for:
- Threat model
- Protocol format
- Cryptography plan
- Replay and tamper defenses
- UML-oriented class outlines
- Research plan and metrics

## License
TBD
