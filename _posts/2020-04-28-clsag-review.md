---
layout: post
title: CLSAG Review
---

### Overview

This review is for two implementations of CLSAG in Monero:

- [moneromooo](https://github.com/moneromooo-monero/bitmonero/tree/clsag)
- [sarang](https://github.com/SarangNoether/monero/tree/clsag-device)
- [kenshamir](https://github.com/crate-crypto/CLSAG) (cross-reference)

The [CLSAG whitepaper](https://eprint.iacr.org/2019/654) was used for reference and algorithm specification.

No major implementation bugs were found after conducting a manual code-review, and performing tool-assisted tests.

Both implementations accurately follow the protocol described in the whitepaper. Optimizations to point operations
and other cryptographic details should be reviewed further by domain experts in C and cryptographic optimization.

A constant-time variant of `CLSAG_Gen` and `verRctCLSAGSimple` are recommended for hardware wallet hardening.

Wallet files are encrypted to disk using ChaCha20 without Poly1305 MACs. Adding Poly1305 for AEAD protection is recommended for wallet hardening.

A minor discrepancy exists between the implementation and whitepaper regarding the ordering of serialized signature parameters.
The ordering discrepancy is not directly a security bug, but could lead to third-party implementation differences resulting in false-negative signature verification.


### Version Information


Review is based on Sarang Noether's and moneromooo's forks of Monero:

- Repo: [https://github.com/SarangNoether/monero](https://github.com/SarangNoether/monero)
- Branch: [https://github.com/SarangNoether/monero/tree/clsag-device](https://github.com/SarangNoether/monero/tree/clsag-device)
- Head commit SHA1 hash: `f0190c7a3080517ebd5be34ee763c0685b26fbb9`

- Repo: [https://github.com/moneromooo-monero/bitmonero](https://github.com/moneromooo-monero/bitmonero)
- Branch: [https://github.com/moneromooo-monero/bitmonero/tree/clsag](https://github.com/moneromooo-monero/bitmonero/tree/clsag)
- Head commit SHA1 hash: `f84273308588da7e07217135768a6fe060edf59f`


### Methodology

Testing took place over two weeks, using both manual and tool-assisted review. Focus for manual review was primarily on validating implementation and specification, while fuzzing was used to stress-test the new CLSAG APIs.

The [CLSAG fuzzing harness](https://github.com/SarangNoether/monero/pull/2) was derived from an existing test, and modification for CLSAG was relatively straightforward (a nice win for people interested in such things).

Other verification tooling like KLEE would be a great addition to the Monero test ecosystem, though no formal verification tools were used in this review.


### Findings


#### Timing Side-channels

Many of the internal point operations in `CLSAG_Gen` utilize variable-time implementations.
At the time of writing `addKeys_aGbBcC` and `addKeys_aAbBcC` use `ge_triple_scalarmult_base_vartime` and `ge_triple_scalarmult_precomp_vartime`, respectively.
Variable-time cryptographic functions are vulnerable to timing side-channel attacks, potentially leaking secret key material.

In the reference implementation (desktop, mobile), side-channel timing attacks are likely orders of magnitude harder to exploit than other attacks for recovering key material.
Given the low-likelihood of exploitation, a constant-time variant of CLSAG will likely provide neglible improvements compared to other hardening measures.

However, in the hardware wallet context, side-channel timing attacks are one of the main attack vectors for recovering key material. In is this researcher's humble opinion, a
constant-time implementation of CLSAG is paramount for harware wallet implementations. Ed25519-donna is a curve variant that supports constant-time point operations, and is used by
well-audited libraries like [libSodium](https://github.com/jedisct1/libsodium). It is recommended to implement the constant-time variant of CLSAG using libSodium, or a similarly well-audited cryptographic library.

#### ChaCha20 without Poly1305 MAC

Wallet JSON files are encrypted to disk using [ChaCha20 encryption](monero/src/wallet/wallet2.cpp:3914) without a Poly1305 MAC.
ChaCha20 is secure to use on it's own in certain contexts; however, adding a Poly1305 MAC for AEAD (Authenticated Encryption w/ Additional Data) protects against chosen ciphertext attacks.
Since the changes required to add Poly1305 are very minimal, and the security gains are non-trivial, it is recommended to use ChaCha20-Poly1305 to encrypt wallet files.


### Auxilliary Findings

#### Timing Side-channels

In [kenshamir](https://github.com/kenshamir) and [kevaundray](https://github.com/kevaundray)'s Rust
implementation of [CLSAG](https://github.com/crate-crypto/CLSAG), Ristretto is used for EC point operations. Ristretto relies on [subtle](https://github.com/dalek-cryptography/subtle) for constant-time comparison of `u8` bytes and slices.

Using the [haybale-pitchfork](https://github.com/PLSysSec/haybale-pitchfork) crate, an integration test
is included with this report for testing constant-time violations.

At the time of this report draft, the test fails with one violation originating with `std::ops::BitXor`. A [separate issue/PR](https://github.com/dalek-cryptography/subtle/pull/69) has been opened with [subtle](https://github.com/dalek-cryptography/subtle) crate.


### Notes


#### Signature Serialization

In the [whitepaper](https://eprint.iacr.org/2019/654), CLSAG signatures are serialized as: S = (c0, s0, s1...sn-1, I, D)

In the [implementation](https://github.com/SarangNoether/monero/blob/clsag-device/src/ringct/rctSigs.cpp#L313), CLSAG signatures are serialized as: S = (s0, c0, s1...sn-1, I, D)

The difference is negligible, and does not present any security risks in the Monero implementation.
Since the signature parameters are serialized and deserialized in the same order, there is no risk for incorrect interpretation.

Serialization order agreement might become important when interacting with third-party implementations, which may only consult the whitepaper.

It is recommended to have the same ordering between implementation and specification to reduce chances of interoperability bugs.

At the time of writing, no other major/minor bugs were found in the Monero CLSAG implementation. The code quality is superb, and very readable (a big win for auditors!).


#### Appreciation

Working with moneromooo, Sarang Noether, and the Monero Research Lab was a pleasure. They were very helpful and informative when I had questions, and were very responsive
to the few findings listed in this report. Definitely looking forward to working together with them in the future.

