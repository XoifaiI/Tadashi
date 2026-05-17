# Tadashi

Cryptographic test vectors for cases the canonical suites skip.

## Why

Most existing test suites are organized around correctness or conformance:
they verify that a correct implementation passes, but they don't say what
specific bug each failing test would expose. Tadashi inverts that, every
non-KAT vector exists to catch one named implementation bug.

- **One vector, one bug.** Every non-KAT vector states explicitly which
  implementation bug it's designed to catch via a `targets` field.
- **Provenance for every vector.** `source` + `notes` say where it came from,
  who generated it, and what other implementation agrees with the output.
- **Cross-validated.** Outputs are cross checked against at least two
  independent implementations before inclusion.
- **Standard JSON shape.** Test cases use the conventional
  `tcId / comment / flags / result` fields so existing runners can consume
  Tadashi files with the `targets` field ignored.

## Layout

```
testvectors/                  # the JSON files
SCHEMA.md                     # JSON schema docs
```

## Schema

Each file is a JSON document with one or more test groups. Each test case has
the standard `tcId / comment / flags / result` plus a Tadashi-specific
`targets` field listing the bug categories that test exists to catch. See
`SCHEMA.md`.

## Status

| Algorithm | Status |
|---|---|
| ChaCha20 | done (17 tests) |
| XChaCha20 | done (13 tests) |
| Poly1305 | done (15 tests) |
| ChaCha20-Poly1305 | done (15 tests) |
| XChaCha20-Poly1305 | done (17 tests) |
| AES-GCM | done (33 tests) |
| SHA-224 | done (15 tests) |
| SHA-256 | done (15 tests) |
| SHA-384 | done (15 tests) |
| SHA-512 | done (15 tests) |
| SHA3-224 | done (11 tests) |
| SHA3-256 | done (11 tests) |
| SHA3-384 | done (11 tests) |
| SHA3-512 | done (11 tests) |
| SHAKE128 | done (13 tests) |
| SHAKE256 | done (13 tests) |
| BLAKE2b | done (16 tests) |
| BLAKE3 | done (22 tests) |
| HMAC-SHA256 | done (12 tests) |
| HMAC-SHA512 | done (12 tests) |
| HKDF-SHA256 | done (9 tests) |
| HKDF-SHA512 | done (9 tests) |
| KMAC128 | done (12 tests) |
| KMAC256 | done (12 tests) |
| Ed25519 | done (50 tests) |
| X25519 | done (27 tests) |

## License

Apache 2.0. See `LICENSE`.
