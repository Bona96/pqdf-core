# Contributing to pqdf-core

Thank you for taking an interest. This project handles cryptography and evidence integrity, so reviews and critiques are as valuable as code. Cryptographers, Rust developers, and digital forensics practitioners are especially welcome.

> **Found a vulnerability?** Do not open a public issue. Follow [SECURITY.md](SECURITY.md).

## Ways to help

- **Review:** read the [threat model](THREAT_MODEL.md) and challenge its assumptions or open questions.
- **Test:** add known-answer tests, fuzz targets, or timing tests (see below).
- **Spec work:** help define the canonical serialization format and evidence-bundle spec.
- **Forensics input:** tell us how evidence is actually collected, stored, and presented in practice.
- **Docs:** clarify setup, architecture, and design decisions.
- **Code:** pick an issue labelled `good first issue` or `help wanted`.

## Before you start

1. Search existing issues and discussions.
2. For anything beyond a small fix, **open an issue first** and describe what you want to change and why. This avoids wasted work.
3. Changes to the serialization format, algorithm choices, or trust boundaries need a short written proposal in the issue before code.

## Development setup

Requirements: Rust stable (plus nightly for fuzzing) and a current Node.js LTS.

```bash
git clone https://github.com/Bona96/pqdf-core.git
cd pqdf-core

# Rust core
cargo fmt --all
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features

# Node.js service and SDK (once they exist)
npm ci
npm run build
npm test
```

CI runs the same checks, plus `cargo audit` and `cargo deny`. Run them locally before pushing if you can:

```bash
cargo install cargo-audit cargo-deny --locked
cargo audit
cargo deny check
```

## Rules for cryptographic code

These apply to anything that touches secret material or signature and key-encapsulation logic. Pull requests that break them will be asked to change.

1. **No home-grown primitives.** Use vetted, maintained implementations of the NIST algorithms. New dependencies need a short justification in the pull request.
2. **No secret-dependent branches or indexing.** Use the `subtle` crate for comparisons and selections on secret data.
3. **Zeroize secrets.** Types that hold secret material must implement or wrap `zeroize` behaviour, and must not derive `Debug`, `Display`, or `Clone` without good reason.
4. **No secrets in logs, errors, or panics.** Error messages must not contain key material or secret-dependent values.
5. **Keep `unsafe` minimal.** Core crates should use `#![forbid(unsafe_code)]`. Any `unsafe` (for example at the FFI boundary) needs a `// SAFETY:` comment and a test that exercises it.
6. **Secret keys stay in Rust.** Do not add APIs that return raw secret key bytes to JavaScript. Use opaque handles.
7. **Algorithm identifiers everywhere.** Signatures and keys carry their algorithm so evidence can be re-signed later.

## Testing expectations

| Change | Required |
| :-- | :-- |
| Algorithm or parameter change | NIST known-answer tests (ACVP vectors) updated and passing |
| New parser or deserializer | A `cargo-fuzz` target and malformed-input unit tests |
| Serialization or bundle format change | Updated spec text and test vectors |
| Secret-handling code | Zeroization test where practical |
| Bug fix | A regression test that fails without the fix |

**Timing tests.** Statistical constant-time tests (dudect-style) are slow and noisy on shared runners. Name them `timing_*` and mark them `#[ignore]`. They run in the nightly workflow, not on every pull request:

```bash
cargo test --release --workspace -- --ignored timing_ --nocapture
```

## Pull request checklist

- [ ] Linked to an issue (or the change is trivially small)
- [ ] `cargo fmt`, `cargo clippy`, and `cargo test` pass
- [ ] Tests added or updated, including vectors or fuzz targets where the table above requires them
- [ ] No new secret-dependent control flow, and secrets are zeroized
- [ ] Documentation, spec, and [THREAT_MODEL.md](THREAT_MODEL.md) updated if behaviour or assumptions changed
- [ ] Commits are signed off (see below)

Changes to the Rust core, the FFI boundary, or serialization receive extra scrutiny and may take longer to review. That is expected.

## Commits and licensing

- Use clear, imperative commit messages. [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`, `test:`) are welcome but not required.
- Sign off each commit with `git commit -s`. This adds a `Signed-off-by` line certifying the [Developer Certificate of Origin](https://developercertificate.org/).
- By contributing, you agree that your contributions are licensed under the project's terms: **MIT OR Apache-2.0**, at the user's option.

## Conduct

Be respectful and constructive. Critique ideas and code, not people. A `CODE_OF_CONDUCT.md` based on the [Contributor Covenant](https://www.contributor-covenant.org/) will define expectations and how to report problems.
