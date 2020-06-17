# Lelantus Cryptographic Review

## Summary

Over a period of ~20 days, I reviewed the cryptographic properties of the Lelantus protocol<br>
defined in original IACR pre-print [1](https://eprint.iacr.org/2019/373.pdf), a draft revision, and the published revision [2](https://lelantus.io/a-new-design-for-anonymous-and-confidential-cryptocurrencies.pdf).

Findings in this report are for the protocol described in the published revision only, since<br>
it addresses all findings from the IACR pre-print and draft revision.

## Caveats

I am not a PhD researcher, and do not consider myself a professional cryptanalyst.<br>
I have experience implementing, and sometimes breaking, cryptographic protocols.<br>
I took on this project pro-bono as a learning experience, and to assist other auditors with further research.

Any mistakes are fully my own, and findings should be weighed against these caveats.

That said, I have given my best effort to accurately review the Lelantus protocol.

## Generalized Pedersen Commitments

To the best of my understanding, the generalization of Pedersen commitments is<br>
correct, and maintains the security properties of the original scheme.

Independence of the generators is key to the binding property, as a known discrete-log relation<br>
among the generators would allow a malicious prover to commit to the same serial number<br>
and/or value multiple times. In addition to being independent of each individual generator, each<br>
generator must also be independent of the product of the other two generators<br>
(which essentially form the second generator in classic Pedersen commitments).

Formal cryptographic analysis of Pedersen commitments was performed by Roberto Metere<br>
and Chaugyu Dong in 2017 [3](https://arxiv.org/pdf/1705.05897.pdf), using an extension of [EasyCrypt](https://github.com/EasyCrypt/easycrypt).
Further extending their work to include proofs for generalized Pedersen commitments could be valuable as another means for ensuring
the correctness of the general construction.

## Proof of knowledge of discrete logarithm representation

The construction for proving knowledge of discrete logarithm representation is correct, and secure<br>
given the assumptions in the paper.

One possibility for a malicious prover to break the scheme is to set `x = 0`, which breaks the<br>
binding property (any exponent would verify, requiring no knowledge of discrete log representation).

Since the Fiat-Shamir heuristic is used to generate the non-interactive challenge `x`, the<br>
likelihood of `x = 0` is negligible. Alternatively, it would require a malicious prover to find a<br>
preimage to the hash function used for the Fiat-Shamir heuristic, which is also neglible given a<br>
cryptographically secure hash function.

## Bulletproofs

Since Bulletproofs are being used unmodified from their original construction, findings are<br>
deferred to formal audits conducted by other researchers.

## Mint

During the minting process, a random blinding factor `x` is generated using the Fiat-Shamir<br>
heuristic. Similar to the use of the Fiat-Shamir heuristic for proving knowledge of discrete<br>
logarithm representation, `x = 0` is negligible.

If another implementation chooses not to use the Fiat-Shamir heuristic to generate `x`, a check<br>
must be performed to ensure `x != 0` to preserve the binding property of the Mint construction.

## Spend

During output generation, blinding factors `x0, ..., xn` are randomly sampled using a cryptographically
secure pseudo-random number generator (CSPRNG). The likelihood of one of the blinding factors being zero
is negligible, but an extra check may be advisable to ensure the binding property.

If blinding factors are sampled using the Fiat-Shamir heuristic, similar reasoning as other<br>
uses applies, i.e. `x = 0` is neglible.

The rest of the construction appears to be correct and offers the claimed security properties.

## One-out-of-many proof for double-blinded commitments

The extension of Groth and Kohlweiss' one-out-of-many proofs [4](https://eprint.iacr.org/2014/764.pdf) over double-blinded commitments<br>
appears correct, and does not appear to introduce new assumptions or security vulnerabilities.

Only one value for the challenge parameter `x` breaks the binding property, i.e. `x = 0`.

Since the challenge is generated with the Fiat-Shamir heuristic, the likelihood of `x = 0` is neglibile.

Given that the generation of `Gk` and `Qk` rely on multiplication by `h2^gamma[k]` and<br>
`h2^-gamma[k]`, respectively, more analysis is needed to ensure no subtle changes to security<br>
properties are introduced.

For example, verification includes commitments of the form `Comm(0, rho[k], gamma[k] + tau[k])`,<br>
which enables different `gamma[k]` and `tau[k]` to sum to the same exponent. It is unclear if<br>
the sum of blinding factors has any affect on the security properties of the proof.

## Verify and Receive

Both the Verify and Receive constructions appear correct, and to provide the claimed security<br>
properties.

For the Receive construction, some care may need to be taken during implementation to ensure<br>
constant-time operations involving secret data. For example, coins identified to be addressed<br>
to the recipient undergo extra processing, which may open timing side-channels for an attacker.

## Acknowledgement

Over the weeks of the review, the insight, feedback, and discussion with Reuben Yap and<br>
Aram Jivanyan is very much appreciated. They both made the review process very enjoyable and productive.

The Lelantus papers are very well structured, and conducive to deeper research through the<br>
referenced material. The ideas presented in the paper itself are complex, but communicated in a<br>
clear, concise presentation. I learned a lot through this process, and look forward to possible<br>
opportunities to work with the Zcoin team in the future.

## References

1. [Lelantus IACR preprint](https://eprint.iacr.org/2019/373.pdf)
2. [Lelantus: A New Design for Anonymous and Confidential Cryptocurrencies](https://lelantus.io/a-new-design-for-anonymous-and-confidential-cryptocurrencies.pdf)
3. [Automated Cryptographic Analysis of the Pedersen Commitment Scheme](https://arxiv.org/pdf/1705.05897.pdf)
4. [One-out-of-Many Proofs: Or How to Leak a Secret and Spend a Coin](https://eprint.iacr.org/2014/764.pdf)
