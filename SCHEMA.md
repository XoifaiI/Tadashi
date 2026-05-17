# Tadashi JSON schema

JSON based, with one non standard field, `targets` on each test case.
Runners that don't know about `targets` should ignore it.

## Top level

```json
{
  "algorithm": "CHACHA20",
  "generatorVersion": "Tadashi 0.1.0",
  "schema": "tadashi_stream_cipher_v1.json",
  "header": ["Free-form prose explaining what this file covers."],
  "notes": {
    "FlagName": {
      "bugType": "...",
      "description": "What this flag means.",
      "effect": "What a buggy implementation would do."
    }
  },
  "numberOfTests": 42,
  "testGroups": [ /* see below */ ]
}
```

## Test group

Each group fixes parameters (key size, nonce size, etc.). Tests within a
group share these.

```json
{
  "type": "ChaCha20Encrypt",
  "keySize": 256,
  "ivSize": 96,
  "tests": [ /* test cases */ ]
}
```

## Test case

```json
{
  "tcId": 12,
  "comment": "Plaintext spans exactly one block boundary (64 bytes).",
  "targets": ["BlockBoundary", "LastBlockHandling"],
  "flags": ["BlockBoundary"],
  "source": {
    "name": "tadashi",
    "version": "0.1.0",
    "crossChecked": ["RustCrypto/chacha20"]
  },
  "key": "0001020304050607...",
  "iv": "000000000000004a00000000",
  "counter": 1,
  "msg": "...",
  "ct": "...",
  "result": "valid"
}
```

### Fields

- `tcId` (int): Sequential, unique within the file.
- `comment` (string): One sentence on what the test does.
- `targets` (array of string): Bug categories this test exists to catch. **Tadashi-specific.** KAT (correctness) tests have an empty array.
- `flags` (array of string): Conventional flag categories describing test characteristics.
- `source` (object): Provenance. **Tadashi-specific.**
  - `name`: Where the vector came from (`rfc8439`, `tadashi`, `libsodium`, etc.).
  - `version`: Source version if applicable.
  - `crossChecked`: Other implementations that produced the same output.
- Input/output fields are algorithm-specific (see per-algorithm comments).
- `result`: `"valid"`, `"invalid"`, or `"acceptable"`. `acceptable` is reserved
  for genuine spec ambiguity where two well-defined implementation choices both
  conform to a published spec (e.g. RFC 7748 small-order X25519 PKs: rbx-crypto
  returns the all-zero shared per the original Curve25519 contract, libsodium-
  strict rejects). A consumer of an `acceptable` test should pass if its impl
  matches the recorded `shared` (or equivalent output field) OR if it cleanly
  rejects. The `comment` and `flags` document the two camps.

## Targets vocabulary

A test's `targets` lists bug categories. Standard vocabulary:

### Encoding targets (apply to every algorithm)
- `HexByteOrder`, input bytes are asymmetric (first byte ≠ last byte; bytes monotonically increase or use distinct values). Catches hex parsers that reverse byte order on parse or emit.
- `HexNibbleOrder`, each byte has high nibble ≠ low nibble (e.g. `0x12, 0x34, 0x56, ...`). Catches hex parsers that swap nibbles within a byte.
- `MixedCaseHex`, same vector encoded with both upper- and lowercase hex; result must be identical. Catches case-sensitive parsers.

### Stream-cipher targets
- `BlockBoundary`, plaintext length is exactly N·block_size; bug if implementation hangs at boundary.
- `LastBlockHandling`, plaintext length is N·block_size + k for 0 < k < block_size.
- `EmptyPlaintext`, zero-length plaintext; bug if implementation panics.
- `CounterEndianness`, counter value chosen to expose byte-order bugs.
- `NonceEndianness`, nonce value chosen to expose byte-order bugs.
- `KeyEndianness`, key value chosen to expose byte-order bugs.
- `ZeroInputs`, all-zero key/nonce/plaintext exposing keystream-only state.
- `InitialStateConstants`, uses values that depend on Sigma/Tau constants being correct.

### MAC targets
- `EmptyMessage`, empty MAC input.
- `BlockBoundaryMac`, MAC input length is N·block_size.
- `RClamping`, Poly1305 r-key bytes that get clamped; bug if clamping is wrong.
- `FinalReduction`, h close to 2^130 − 5 boundary.
- `PadAddition`, h + pad addition with carry overflow.
- `ModTruncation`, final mod-2^128 truncation drops the right bits.

### AEAD targets
- `LengthEncoding`, length fields large enough to expose 4-byte-vs-8-byte LE confusion.
- `AadOnly`, empty plaintext, non-empty AAD.
- `PlaintextOnly`, non-empty plaintext, empty AAD.
- `BothEmpty`, empty AAD AND empty plaintext (only the length block goes into MAC).
- `AadPaddingZero`, AAD length is exact multiple of 16; tests that no extra padding block is added.
- `CtPaddingZero`, CT length is exact multiple of 16; same.
- `TagForgery`, single-bit-flipped tag; tests verifier rejection.
- `TagTruncation`, caller asks for shorter tag than spec mandates; tests rejection.
- `CounterStart`, encryptor must start at counter 1, not 0 (block 0 is for MAC key).
- `WrongMacKey`, MAC key derived from wrong counter; tests detection.

Add new targets as needed; each should be one specific bug category, not a class.
