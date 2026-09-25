# pqdf-core: Post-Quantum Digital Forensics

## The Problem
Standard digital evidence platforms rely on classical signatures (RSA, ECDSA) susceptible to Harvest-Now-Decrypt-Later (HNDL) state surveillance. Implementing post-quantum cryptography (PQC) directly in JavaScript runtimes is insecure; Just-In-Time (JIT) compilation and unpredictable garbage collection break constant-time execution, exposing the algorithms to timing side-channel attacks.

## Hybrid Architecture
This project bridges rigorous cryptographic security with web-scale accessibility by isolating cryptographic operations from the Node.js runtime:

1. **Rust Cryptographic Core:** Standardized lattice algorithms (ML-DSA, ML-KEM) are implemented in Rust. This guarantees memory safety, strict constant-time execution, and secure zeroization of ephemeral keys.
2. **N-API Native Bindings:** The Rust core is compiled into native Node.js addons via `napi-rs` to execute computationally heavy lattice operations without blocking the JavaScript event loop.
3. **Node.js Wrapper:** A lightweight REST and gRPC API layer handles deterministic JSON serialization, evidence ingest, routing, and PQC timestamp verification.

## Development Roadmap
* **Milestone 1:** Rust bindings for ML-DSA and ML-KEM & Canonical Serialization Specification.
* **Milestone 2:** Node.js microservice architecture with gRPC API endpoints and database adapters.
* **Milestone 3:** Next.js independent verification web portal and lightweight `@pqdf/sdk` client package.
* **Milestone 4:** Automated fuzzing, hardware memory benchmarking, and Docker image deployment.