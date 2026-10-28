# Threat Model: pqdf-core

> **Status:** draft skeleton. Sections marked ❓ are open questions to resolve before Milestone 1 closes. Update this file whenever the design changes.

**Legend:** ✅ addressed by design · 🟡 planned or partially addressed · ❓ open question

## 1. Purpose and scope

`pqdf-core` provides post-quantum signing and key encapsulation for digital evidence. This document states what the project tries to protect, from whom, and what it explicitly does not claim.

**In scope**

- Integrity and authenticity of evidence bundles and custody records
- Long-term verifiability of signatures against quantum-capable adversaries
- Confidentiality of evidence in transit and at rest where ML-KEM is used
- Isolation of cryptographic operations and secret keys from the JavaScript runtime

**Out of scope**

- Compromise of the capture device before evidence reaches the system
- Physical attacks and hardware side channels (power, EM, fault injection)
- Determining legal admissibility in any jurisdiction
- Denial-of-service resilience beyond basic rate limiting
- Human factors such as coercion or social engineering of operators

## 2. Assets

| ID | Asset | Why it matters |
| :-- | :-- | :-- |
| A1 | Evidence content and its hash | The thing being protected |
| A2 | Signing keys (ML-DSA) | Compromise allows forged evidence |
| A3 | KEM keys (ML-KEM) and derived secrets | Compromise breaks confidentiality |
| A4 | Custody log and timestamps | Establish *when* and *who* |
| A5 | Canonical serialization rules | Ambiguity here breaks every signature |
| A6 | Verification results | Must not be spoofable by the service |

## 3. Adversaries

| ID | Adversary | Capability | Goal |
| :-- | :-- | :-- | :-- |
| T1 | Passive archiver (HNDL) | Records traffic and stored ciphertext today, has a quantum computer later | Decrypt evidence retroactively |
| T2 | Future forger | Quantum computer, access to archived classical signatures | Forge or repudiate historical evidence |
| T3 | Network attacker | Active man-in-the-middle on API traffic | Tamper with or replay submissions |
| T4 | Malicious insider | Legitimate API credentials | Alter or delete evidence, backdate records |
| T5 | Compromised Node.js host | Remote code execution in the service process | Exfiltrate keys, sign arbitrary data |
| T6 | Co-tenant or local timing attacker | Can measure timing of operations | Recover secret key bits via side channels |
| T7 | Storage attacker | Read/write access to the database or object store | Modify stored bundles silently |

## 4. Trust boundaries

1. **Client ⇄ Node.js API:** untrusted network, authenticated and encrypted transport.
2. **Node.js service ⇄ Rust core:** the FFI boundary. Treated as a trust boundary even though both run on the same host. ❓ Decide whether secret keys are held only as opaque handles inside Rust.
3. **Service ⇄ evidence store:** the store is untrusted for integrity (verified via signatures and hashes).
4. **Verifier ⇄ service:** the verifier must not need to trust the ingest service. 🟡

## 5. Threats and mitigations

| # | Threat | Adversary | Mitigation (design) | Status |
| :-- | :-- | :-- | :-- | :-- |
| 1 | Harvested ciphertext decrypted later | T1 | ML-KEM-based encapsulation instead of RSA/ECDH key exchange | 🟡 |
| 2 | Classical signatures forged after quantum break | T2 | ML-DSA signatures; hybrid and re-signing support | 🟡 |
| 3 | Replay or tampering of submissions | T3 | Authenticated transport, nonces, signed receipts | 🟡 |
| 4 | Insider rewrites or backdates records | T4 | Append-only custody log with hash chaining, external timestamps | 🟡 |
| 5 | Keys stolen from a compromised Node.js process | T5 | Secret keys never enter the JS heap, opaque handles, `zeroize` | ❓ |
| 6 | Signing oracle abuse by compromised service | T5 | Signing policy in Rust (what may be signed), rate limits, audit log | ❓ |
| 7 | Timing leakage of secret-dependent operations | T6 | Constant-time design in Rust, dudect-style tests in CI | 🟡 |
| 8 | Secrets lingering in memory, swap, or core dumps | T5, T6 | Zeroization, memory locking, disabled core dumps | 🟡 |
| 9 | Silent modification of stored bundles | T7 | Signature and hash verification on read, independent verifier | 🟡 |
| 10 | Canonicalization ambiguity causes signature bypass | T3, T4 | Single normative spec (RFC 8785 or deterministic CBOR), published test vectors | 🟡 |
| 11 | Malformed input crashes or corrupts the FFI layer | T3 | Strict parsing, `cargo-fuzz` targets, bounded allocations | 🟡 |

## 6. Cryptographic assumptions and failure modes

- **Assumptions:** the module-lattice problems underlying ML-KEM and ML-DSA remain hard against classical and quantum adversaries, and the chosen implementation is correct.
- **If an algorithm is weakened:** algorithm identifiers in every signature let operators re-sign affected evidence with a replacement scheme. 🟡
- **If a signing key is compromised:** revocation, key-rotation procedure, and how previously signed evidence is treated. ❓
- **Signing mode:** FIPS 204 allows hedged (randomized) and deterministic signing. Which one is used, and why, must be documented. ❓
- **Hash function for evidence:** SHA-256 or SHA3-256, to be fixed in the spec. ❓

## 7. Side channels and implementation risks

- **Timing:** secret-dependent branches and memory access are the primary risk. Rust does not guarantee constant-time behaviour, so this is verified empirically, not assumed.
- **Memory residue:** ephemeral secrets must be zeroized, and compiler optimizations must not elide the writes.
- **Fault attacks and power/EM analysis:** out of scope for the software-only threat model.
- **Dependencies:** cryptographic crates and native build tooling are part of the trusted computing base. Pin versions, audit them, publish an SBOM. 🟡

## 8. Security properties targeted

| Property | Target | Evidence of achievement |
| :-- | :-- | :-- |
| Integrity | Any change to evidence or metadata is detectable | Signature verification, test vectors |
| Authenticity | Signatures bind evidence to a key holder | Known-answer tests |
| Long-term verifiability | Signatures remain checkable after classical schemes fail | PQC algorithms, re-signing procedure |
| Confidentiality at rest and in transit | Resistant to HNDL | ML-KEM, protocol review |
| Independent verifiability | Verification does not require trusting the service | Standalone verifier |

## 9. Open questions

1. Are secret keys generated and stored only inside Rust, and what is the key-custody story (file, OS keystore, HSM)?
2. Where do trustworthy timestamps come from, and how are they verified post-quantum?
3. How are keys rotated and revoked without invalidating past evidence?
4. What is the policy for hybrid signatures during migration?
5. How is multi-party custody (several collectors, one case) represented in the bundle?
6. What does an independent verifier need to ship with (CLI, WASM, offline mode)?

## 10. Review log

| Date | Change | Author |
| :-- | :-- | :-- |
| | Initial skeleton | |