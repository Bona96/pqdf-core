# Security Policy

`pqdf-core` is **pre-alpha and has not been independently audited**. It is not yet suitable for protecting real evidence. Security reports are still very welcome, and they directly shape the design. For the current understanding of risks and assumptions, read [THREAT_MODEL.md](THREAT_MODEL.md).

## Supported versions

| Version | Supported |
| :-- | :-- |
| `main` branch | ✅ Fixes land here |
| Tagged pre-releases (`0.x`) | ❌ Upgrade to `main` or the latest tag |

Once a stable release exists, this table will list the supported release lines.

## Reporting a vulnerability

**Please do not open a public issue, pull request, or discussion for a suspected vulnerability.**

Use either channel:

1. **GitHub private vulnerability reporting (preferred).** Go to the repository's **Security** tab and choose **Report a vulnerability**, or open <https://github.com/Bona96/pqdf-core/security/advisories/new>.
2. **Email:** mnuwabiine@gmail.com with the subject line `pqdf-core security`. If you need an encrypted channel, say so in a first message without technical details and we will arrange one.

### What to include

- A description of the issue and its security impact
- The affected component (Rust core, napi-rs bindings, Node.js service, SDK, serialization spec) and commit or version
- Steps to reproduce, a proof of concept, or a failing test
- For side-channel reports: the measurement setup, hardware, and statistics (for example, dudect output)
- Whether you plan to publish, and any deadline you are working to

## What to expect

This project is maintained by a single developer, so these are best-effort targets:

| Step | Target |
| :-- | :-- |
| Acknowledge your report | within 5 days |
| Initial assessment and severity | within 14 days |
| Fix or mitigation plan | shared as soon as it is known |
| Coordinated public disclosure | up to 90 days after the report, sooner if a fix ships earlier |

If a report is declined, you will get an explanation. If you disagree, you are free to disclose publicly after the 90-day window.

## Scope

**In scope**

- Bugs in the cryptographic implementation or its integration, including incorrect outputs and failures against known-answer tests
- Secret-dependent timing or memory-access behaviour in secret-handling code
- Secret keys or ephemeral secrets reaching the JavaScript heap, logs, or error messages
- Failure to zeroize secret material
- Memory-safety problems at the FFI boundary (napi-rs bindings)
- Ambiguities in canonical serialization that allow signature bypass or evidence substitution
- Verification logic that accepts invalid signatures, stale algorithms, or malformed bundles
- Vulnerable or malicious dependencies with a plausible exploit path in this project

**Out of scope**

- Theoretical cryptanalysis of ML-DSA, ML-KEM, or SLH-DSA themselves (please report to NIST or the relevant standards forum)
- Social engineering, physical access, or compromise of a capture device before evidence reaches the system
- Volumetric denial of service
- Vulnerabilities in third-party dependencies with no exploit path here (report upstream)
- Findings that only apply to features documented as planned and not yet implemented
- Hardware attacks such as power analysis, EM analysis, or fault injection

## Safe harbour

If you act in good faith, avoid privacy violations and data destruction, only test against your own deployments of this software, and give us reasonable time to respond, we will not pursue or support any legal action against you for your research.

## Credit

Reporters are credited in the security advisory and release notes unless they prefer to stay anonymous. Please tell us how you would like to be named.
